---
title: PEtite2.4脱壳
author: czk
categories:
- PETite2.4脱壳
tags: 
- 脱壳
date: 2020-12-18 
---

	最近由于工作的需要，先是逆向分析了PeTite2.4的壳，了解了壳在程序开头做了那些事情，最后代码实现了自动脱壳；
	关于脱壳的文章，网上一搜一大推，但都是讲解脱壳方法的,分析壳代码的很少；这里做个记录，希望后面温故还能能记忆如新；

<!-- more -->

## PETite2.4壳原理：

​	壳开头会申请一块堆内存, 然后执行解密函数，把主要壳代码解密到内存中，最后转入其中执行主要壳功能；堆代码开头先把解密函数拷贝过来，然后在堆内存尾部取一个解密数组结构，遍历每一个成员，根据里面的源RVA和目的RVA，以及长度，调用解密函数解密；最后处理代码中的E8，E9指令的跳转修复，导入表，原始入口点解析恢复，代码校验一类的事情；





## 1.样本概述

### 1.1 样本信息

大小: 11932 字节
MD5: BCEAEEFF17874D3323237BE240D9C5E4
SHA1: F8A5A778AA7448DBCA3892B6D74648F92F694835
CRC32: 4E0DE219



### 1.2 分析环境和工具

环境：WIN732位

工具: OD,   x32dbg 

## 

## 2.详细分析：

### 2.1 查壳

![image-20201106165858263](/pic/czk/PEtite2.4/packinfo.png)

### 2.2 解密壳代码

   申请0X6D3大小的堆内存，以 EBP+1448h 作为源，0x383h 为长度，进行壳代码的解密；

​	sub_4017BF为解密函数；

​	seg000:00401785处的 0A6d56304h 是PE Header校验CRC；

​	

![image-20201106165803647](/pic/czk/PEtite2.4/start.png)



解密函数：

​	主要逻辑就是从esi取值 异或长度(传进来的ebx)，然后ebx递减，写入edi(edi指向上面申请到的堆内存);

具体的指令就不一一赘述了，直接看我逆成c语言的简版形式, 这样看着方便些；

![image-20201106171129764](/pic/czk/PEtite2.4/decrypt.png)



这里提一下sub_401857函数，很变态，因为用到了adc指令，带进位加法，这段指令很多壳中都有使用，个别论坛里面也有这个函数的c形式，这里我直接搬过来了；

![image-20201106171921612](/pic/czk/PEtite2.4/getbit.png)



下面是整理的简版c代码, 理解个大体逻辑意思吧。

```
int getbit(unsigned int *pcom_dword, unsigned int **ppsrc)
{
    int temp;
    temp = ((*pcom_dword)>>31)&1; //得到 符号位
    (*pcom_dword) <<= 1;
    if(0 != (*pcom_dword))
    {
        *pcom_dword = **ppsrc;
        temp = ((*pcom_dword)>>31)&1;
        (*pcom_dword) <<= 1;
        *pcom_dword += ((unsigned int)*ppsrc >= 0xFFFFFFFC ? 0 : 1);
        (unsigned int)*ppsrc += 4;
    }
    return temp;
}

src	=	0x401448h
declen = 0x383h

sub_401857(psrc, pdst, declen)
{
	index = 0;
	lastlen	=	declen;
	edx 	=	0;
	ecx		=	0;	
	do{		
		*(char*)pdst++	= (*(char*)psrc++ ^ lastlen);
		lastlen--;
__loop:
		if( lastlen <= 0 )
		{
			break;
		}
		if( getbit(&edx, &psrc) ==0 ) 
		{
			continue;
		}
		ecx++;
		do{
			temp = getbit(&edx, &psrc);
			ecx  <<= 1;
			ecx  += temp;
		}while( getbit(&edx, &psrc) )
		ecx -= 3;
		if( ecx < 0 )
		{			
			eax = var14;
			ecx++;
		}
		else{
			eax	= ecx;
			ecx = 5;
			do{
				temp = getbit(&edx, &psrc);
				eax <<= 1;
				eax += temp;
			}while( ecx-- )			
			
			eax = ~eax;
		    ebp += 1 + ((eax > -3A0h) : 0 ? 1);
		    ebp += 1 + ((eax > -3FA0h) : 0 ? 1);
		    var14 = eax;
		}		
		temp = getbit(&edx, &psrc);
		ecx <<=1;
		ecx += temp;
		temp = getbit(&edx, &psrc);
		ecx <<=1;
		ecx += temp;
		if( ecx == 0 )
		{
			ecx++;
			do{
				temp = getbit(&edx, &psrc);
				ecx  <<= 1;
				ecx  += temp;
			}while( getbit(&edx, &psrc) < 0)
			ecx += 2;
		}
		ecx += ebp;
		lastlen	-= ecx;
		if( lastlen < 0 )
			break;
		memcpy(pst, pdst+eax, ecx);
		pdst	+= ecx;
		goto __loop;
		
	}while( 1 );	
__end:
	return ;
}
```



### 2.3 堆壳代码执行

我直接动态调试了，注意此时进入堆时的寄存器以及堆栈， 堆是下面这个样子的:

```
		 001EFBC8   A6D56304 
$ ==>    001EFBCC   00000000  
$+4      001EFBD0   00000000  
$+8      001EFBD4   001EFBF4  
$+C      001EFBD8   001EFBEC  
$+10     001EFBDC   7FFDB000  
$+14     001EFBE0 < 013D1779  92de1c7.EntryPoint
$+18     001EFBE4   00520000  
$+1C     001EFBE8   013D4000  92de1c7.013D4000
$+20     001EFBEC   77973C45  返回到 kernel32.77973C45 自 ???
```

ebp是开头 lea     ebp, [eax-4000h]  指定的模块基址；

其中520000是申请的堆基址



以下是解密后的堆内存中的代码, 其中框中的会把sub_17bf解密函数拷贝到本模块的520383处；0x340是指向解密结构数组的偏移；

![image-20201106173958488](/pic/czk/PEtite2.4/520000start.png)



下面就是以一个结构数组进行代码解密恢复到原始PE处, 有调用520383解密函数，参数ebx, esi, edi都是结构数组中的数据；

![image-20201106174352455](/pic/czk/PEtite2.4/decrypt2.png)



其中结构数组位置在520340处；怕排板乱了直接用图片了，如下:

![image-20201106174807670](/pic/czk/PEtite2.4/520340.png)

这个结构体数据最后一个成员是全零的，未贴上；

当DstRVA和SrcRVA一样时有个交叉处理的，会使用从SrcRVA拷贝copySize大小到临时位置；

这个结构体数组结合上面解密函数就是下面的三个步骤:

```
ebx : 解密长度
esi : SrcRva+ImageBase
edi : DstRva

解密逻辑：
[esi] xor len  -->[edi]

[13d1414h]xor  -->[13d3000h]   解密后剩余清零
[13d1250h]xor  -->[13d2000h]
[13d1000h]xor  -->[13d1000h]
注:其中13d0000是动态调试时样本主PE的加载基址;
```



解密后目的rva块剩余未解密的空间会用0填充；然后加 15h字节，处理下一个数组成员；

![image-20201106175720825](/pic/czk/PEtite2.4/set0.png)



根据结构体数组对象的第四个DstSize成员, 决定需不需要修复 E8, E9指令；

最后一个字节作为控制位，shr ecx,1 进CF标志位，紧接着jnb指令判断CF决定是否跳转;

 这里解密完第三个直接进行E8,e9指令修复了；

![image-20201106180521592](/pic/czk/PEtite2.4/repaire8e9.png)



这段的主要逻辑是修复导入表，其中2B0EFA5Ch是加密的入口点；

其中导入的函数的修复也隐含了入口点的解密，会根据获取到的函数地址范围对2B0EFA5Ch进行运算；

![image-20201106180844112](/pic/czk/PEtite2.4/repaitINT.png)

![image-20201106181102658](/pic/czk/PEtite2.4/c10424.png)

其中调用系统函数时，会构建成下面这个样子调用；

```
push D124963B            

rol dword ptr ss:[esp],59

ret
```



入口点计算中间过程:

![image-20201106181337604](/pic/czk/PEtite2.4/oepoffset.png)



这里有个对PE.NtHeader 的校验:

![image-20201106181824634](/pic/czk/PEtite2.4/checkpeheader.png)



修复FF15call:

![image-20201106181942706](/pic/czk/PEtite2.4/repairff15.png)



最后恢复内存属性，是解密数组结构成员最后一成员指定的；

最下面就是获取最后的OEP了,在edx中，最后的jmp前 push的就是返回地址，调用玩VirtualFree释放完本内存，就跳过去了；至此结束；

![image-20201106182121060](/pic/czk/PEtite2.4/retoep.png)





