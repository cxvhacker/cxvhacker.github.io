---
title: IDC脚本初上手
author: czk
categories:
- IDA
- IDC
tags: 
- idc
date: 2020-07-10 
---

# 概述

IDC是IDA扩展的一种脚本语言，为什么叫这个名字，估计是它的语法类似于C。

IDA可能很多人都知道，也都用过，但是估计很多人都没有使用过IDC；本人也是仰慕许久，一直未碰到合适的场景练练手，最近分析到一个样本，就实践一下；

<!-- more -->

# 情景

最近分析到一段病毒样本，它会动态加载使用到的模块，然后初始化使用到的模块函数；

![17Module](/pic/czk/idcfirsttry/17Module.png)

loc_40E85D处是个循环, 依次处理起始于004122D8h处的模块数组，44h/4=11h,可以看到处理17个模块；

0040E863h处call进入0040E765h,进入之前edi指向模块信息起始地址;

![GetFunAddrByHashProc](/pic/czk/idcfirsttry/GetFunAddrByHashProc.png)

数组中每个成员指向一个模块信息，每个模块信息第一个成员指向模块名，第二个则是函数个数，第三个则指向要初始化的函数hash数组起始位置；

首先从偏移0处获取模块名地址，调用LoadLibrary加载模块, 根据导出表找到函数名称计算一个hash值，具体计算过程就不展示了，然后与函数hash数组中的hash对比，一样则把函数地址回写到存储hash的地址处；

![funhashs](/pic/czk/idcfirsttry/funhashs.png)

函数地址是直接FF 15 call 地址使用的，直接看地址一会就眼花，把地址重命名成对应函数名就好看多了，但一下子17个模块，每个模块几十个函数，每个函数还要计算hash，这要重命名到猴年马月去，费时费力；能自动批量重命名就好了，这个时候就可以用IDC脚本实现这一过程了；

为了省事直接上OD，动态调试，跳过hash的计算过程，让调试器帮忙填充hash对应的函数地址（当然IDC脚本也能实现hash的计算过程，获取函数地址，先不整这么麻烦）；

![ODresult](/pic/czk/idcfirsttry/ODresult.png)

上面就是调试器填充的结果,把所有数据复制到txt文档；下面是导出的起始片段；

![ODResultExport2Txt](/pic/czk/idcfirsttry/ODResultExport2Txt.png)

从导出结果上看我们需要把第一列的函数地址命名成第三列的函数名；

但导出的数据不完全是要处理的； 

有一个规律，系统函数一般占的都是高地址，可以观察第二列的实际函数地址, 看了整个导出文档，发现需要重命名的地址对应的第二列的实际地址开头都是7，这好办了,直接判断这个；



```
#include <idc.idc>

static main()
{
	auto fp;
	auto filelinestr;
	auto straddr,funstr;
	auto hexaddr;	

	fp = fopen("F:\\modulefunctions.txt","r");
	if(fp == 0)
	{
		Message("open %s failed", "configfile");
		return ;
	}	
	while(1)
	{
		//逐行读取
		filelinestr = readstr(fp);
		if(!value_is_string(filelinestr))
			break;
	
		if(filelinestr[10] == "7")
		{
			straddr	= filelinestr[:8];  //截取前8个  地址
			funstr	= filelinestr[20:]; //截取函数名
			hexaddr	= xtol(straddr);	//函数地址字符串转hex
			Message("%x,%s",hexaddr,funstr);
			MakeName(hexaddr,funstr); //重命名
		}
	}	
	fclose(fp);
}




```

以上就是idc脚本, 语法和C类似, 但并不支持c风格的数组, 所以上面直接是使用分片运算符处理IDC字符串的,局部变量直接auto定义；使用到的readstr 读取文件行内容,Message 输出语句；MakeName重命名函数都是IDC提供支持的函数；具体其他的提供支持的函数可以从网上找资料,这里提供一个参考地址:  [Index of IDC functions](https://www.hex-rays.com/products/ida/support/idadoc/162.shtml)



# IDC脚本执行

下面看一下脚本运行效果；

![IDAFileOpenScriptFile](/pic/czk/idcfirsttry/IDAFileOpenScriptFile.png)

IDA->File->Script file选择编写的idc文件,就可以执行脚本了；

![funhashresult](/pic/czk/idcfirsttry/funhashresult.png)



自动重命名了，大功告成；



