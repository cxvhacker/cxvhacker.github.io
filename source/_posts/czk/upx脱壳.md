---
title: upx脱壳总结
author: czk
categories:
- upx脱壳
tags: 
- 脱壳
date: 2020-11-06

---

​	upx脱壳的方法，从网上能搜索到一大批，UPX脱壳的关键就是找到原始OEP；应用最广的就是ESP定律，即利用堆栈平衡快速找到OEP；原理大家都懂，但是中间经历了哪些过程，你得到的到底是不是OEP，本篇就结合最近的分析印证一下；

<!-- more -->



# 样本信息:

MD5: 3DD764D07B511B6290583840FE31DAAF
SHA1: 198C1EE8B8C570330C2079EAD28C6DF1FBBFEA3D





首先对样本进行查壳；

![image-20201102180228468](/pic/czk/unpackupx\checkpacker.png)

用查壳工具检测到是upx壳，但是具体哪个版本的壳未检测出来；

![image-20201102180415368](/pic/czk/unpackupx\checkpacker_sec.png)

从区块信息上发现，这是个特殊的upx壳，熟悉upx壳的就应该知道，被upx加壳后的程序一般只有三个区块：upx0, upx1, rsrc；



使用最新版的upx对他进行脱壳，脱壳失败；

![image-20201106112808752](/pic/czk/unpackupx\upx396unpack.png)

 upx -d在源码中的实现，是会去第一个区块的头部获取upx信息的(版本，压缩等其他信息)；在这个样本中这些信息被抹除了；

![image-20201106113205574](/pic/czk/unpackupx\upxsec0.png)




IDA中分析：

![image-20201106113418016](/pic/czk/unpackupx\Start.png)

看了这个函数头，能确定这用的是lzma算法加的壳,标出来的边界长度和解密长度都是在对upx逆向熟悉的基础上判断出来的，详细的分析过程不做展开； 重点注意一下开始处的pusha，肯定有呼应的地方；

解密代码执行完毕后面紧跟着:

  call修复

![image-20201106130621303](/pic/czk/unpackupx\repcall.png)



导入表修复

![image-20201106130900663](/pic/czk/unpackupx\repimport.png)



看到最后一个popa，与上面的呼应，平衡堆栈；

然后一个jmp远跳，跳转到入口点![image-20201106131123394](/pic/czk/unpackupx\jmpoep.png)



​	结合以上的分析，能确定ESP定律是能定位到原始入口点的；但从上往下一直找类似最后语句的jmp也是可以定位的；这样我们就可以提取出一段特征码，下次在碰到可以直接搜索特征码，定位到入口点附近；

