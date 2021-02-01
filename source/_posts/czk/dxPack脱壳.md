---
title: dxPack脱壳
author: czk
categories:
- 脱壳
- dxPack脱壳
tags: 
- 脱壳
date: 2021-01-29
---

最近又翻了翻前面的脱壳文件，真是一脸懵逼，如果没有IDA中的注释我都不知道我分析过这个文件，趁着还有点印象，把逻辑整理一下；

<!-- more -->


# 样本信息：

```
MD5: 8C2AE7C7D7D62904ED800029A45CA5C8
SHA1: 1141C391385E410B14969530ACBA201A233F335F
CRC32: 2DCBB0DA
```

分析工具：

​	`IDA， xdbg`



# dxPack壳工作逻辑:

这里总结一下整个壳的loader的工作过程；

```
基础数据初始化和获取kerner32.dll函数地址，有VirtualAlloc,VirtualProtect,VirtualFree;
从401000备份0x4d长度的数据到栈中,其中包含源区块数量,解密数据长度,导入表信息等其他信息;
以40104d作为源数据起始位置解密0x9f38长度的数据到申请的堆内存中；
最后从解密后的数据中恢复原pe的数据；其中包含区块表、导入表、各区块数据...
执行入口点前的一些回调函数，例如Tls_callback;
转入原始入口点；
```



# 详细分析:

## 数据初始化	

开头一个jmp，进入loc_427005，loc_427005函数开头主要是初始化一些数据，比如加载基址:400000，代码开始RVA:1000

​	![start](/pic/czk/dxPack/start.png)

紧接着加载kernel32.dll,并获取函数地址保存下来；

![initkernel32](/pic/czk/dxPack/initkernel32.png)

获取的函数有以下三个:

![functions](/pic/czk/dxPack/functions.png)

## 数据备份

然后从401000处拷贝4d字节数据到栈中（EBP-78h），后面使用访问数据时将会使用栈中的这部分数据；

![backup401000](/pic/czk/dxPack/backup401000.png)

下面是copydata(dst, src, len)函数，就是单字节拷贝，第三个参数控制循环次数；

![copyData](/pic/czk/dxPack/copyData.png)

## 数据解密

紧接着从40104d开始解密9f38长度的数据到申请的堆内存中；

![AllocforDecrypt](/pic/czk/dxPack/AllocforDecrypt.png)



解密函数没有什么可说的，就是按一定的方式把数据解出来；  

由于工作中已经实现过了DecryptFun解密函数的c++代码, 这里就不在具体阐述解密步骤了,接着往下说逻辑；



## 解密后的原PE数据恢复

解密完数据后，后面恢复原PE数据的逻辑才是关键的，主要有以下几个过程:

a. 原pe块数据抹除

b.更改区块个数

c.区块表数据恢复

d.对应区块数据恢复

e.DataDirectory中项数据恢复

f.导入表修复

g.入口点恢复



这里先把从401000拷贝到ebp-78h的数据结构展示一下，下面会使用到

```
从401000拷贝4dh->|--ebp-78
			   	 |
			   	 |4dh
			   	 |
			   	 |--ebp-2b
			   	 |

ebp-78 .rsrc:00401000 03              db    3		secnum
ebp-77 .rsrc:00401001 38 9F 00 00     dd 9F38h		declen
ebp-73 .rsrc:00401005 54 A4 00 00     dd 0A454h		decsize
ebp-6F .rsrc:00401009 00 70 02 00     dd 27000h		zerosize
ebp-6b .rsrc:0040100D 00 60 02 00     dd 26000h		importrva
ebp-67 .rsrc:00401011 44 01 00 00     dd 144h		importsize
ebp-63 .rsrc:00401015 40 46 02 00     dd 24640h		oep
ebp-5F .rsrc:00401019 00 00 00 00     dd 0
ebp-5b .rsrc:0040101D 00 00 00 00     dd 0
```

### a. 原pe块数据抹除

上面DecryptFun解密函数执行完毕后，就会从401000开始把数据清零，清零的长度由[ebp-6f]指定；

![Filldata](/pic/czk/dxPack/Filldata.png)



MemsetProc函数很简单，使用第二个参数进行数据填充，先4字节，最后完成后面的不足4字节部分；

![Memsetproc](/pic/czk/dxPack/Memsetproc.png)

### b.更改区块数量

​	先定位到FileHeader，使用[ebp-78]填充FileHeander.NumberOfSections;

![changesecnum](/pic/czk/dxPack/changesecnum.png)

### c.区块表数据恢复

​	这部分有三步，先清空对应的28h的区块表项数据，然后从解密出的堆数据中拼接出来新的数据，最后把新的数据写回；

​	当然了这是个循环，要把所有的区块数据都恢复出来；

![restoreSec](/pic/czk/dxPack/restoreSec.png)

### d.对应区块数据恢复

​	从解密出来的数据中循环恢复对应区块的数据，每个区块恢复的长度由最新的区块表中的SizeOfRawData指定；

![copySecData](/pic/czk/dxPack/copySecData.png)

### e.DataDirectory数据项恢复

​	各个区块数据恢复完毕后，所有数据都已经归位了，剩下的就是修正个别数据的正确性了；

恢复资源表项

​	![ModifySource](/pic/czk/dxPack/ModifySource.png)



### f.导入表修复

​	这部分会遍历导入表，调用LoadLibrary,GetProcAddress修正导入表函数地址；代码较多，逻辑过程不一一说明了；

![RestoreImport](/pic/czk/dxPack/RestoreImport.png)

### g.修复重定位

​	代码中还有这部分的修复逻辑，针对exe应该完全没有必要，估计是壳设计时考虑到了dll一类的；

​	![fixreloc](/pic/czk/dxPack/fixreloc.png)

### h.调用TLs CallBack

​	TLS也是从DataDirectory中获取的，在exe执行入口点前，先调用了一遍；

![TLSCallback](/pic/czk/dxPack/TLSCallback.png)

### i.执行原始入口点



​	原始入口点保存中[ebp-63]处，然后加上基址，跳转进入，到此整个壳恢复逻辑就结束了；

![jmpoep](/pic/czk/dxPack/jmpoep.png)