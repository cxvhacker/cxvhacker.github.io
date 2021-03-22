---
title: IDC去除花指令
author: czk
categories:
- IDA
- IDC
- 花指令
tags: 
- idc
date: 2021-03-19
---


# 概述

  分析时碰到一个样本，添加了花指令，静态反汇编的过程完全被破坏了，不能直观的看到反汇编的结果，一条一条的手动去修改，工作量又太大；这就有了本篇的由来，使用IDC脚本去除花指令。



​	花指令多种多样，本篇罗列四种常见的，展示一下添加花指令后的效果；
<!-- more -->


# 情景1

一般常见的花指令形式为：	

```
	jz		label
	jnz		label
	db  thunkcode
label:
```

中间的thunkcode就是随机机器指令，最后一个thunkcode指令肯定不是单字节指令，类似push,pop一类的, 否则完全无意义了，这样就可以与label后面的代码连接起来组成混淆后的汇编指令，从而欺骗反汇编引擎；

类似的指令块还有jb/jnb、jo/jno、jc/jnc等其他指令组合；



下图左侧的是有花指令的，右侧是把thunkcode( 77390f79开始的2个字节)用90替换后的；

![image-20210318155211057](/pic/czk/idcfixconfusion/jzjnz.png)

可以看到添加花指令后，thunkcode严重干扰了反汇编引擎，以至于我们静态分析时，看不到我们想看的反汇编结果；



# 情景2

花指令形式：

```
	cmp	eax,eax
	jz	label
	db	thunkcode
label:	
```

原理和情景1是一样的，只不过换了跳转指令, 为了增加分析的难度，花指令延伸出了多种形态；

下图为IDA中直接看到的反汇编示例：

![image-20210318160713361](/pic/czk/idcfixconfusion/cmpjz.png)

下图为手动重新分析后的：

![image-20210318160913279](/pic/czk/idcfixconfusion/cmpjzR.png)

81 83作为机器指令码正好可以解析成add 指令；但动态执行时 81 83会被前面的jz跳过去的；



# 情景3

花指令形式：

```
	call label
	db	 thunkcode
label:    
```

花指令示例:

![image-20210318162859863](/pic/czk/idcfixconfusion/call.png)

具体逻辑就不多说了和前面的逻辑是一样的；77390f8d 开始的2字节(29 5a)是thunkcode;



# 情景4

花指令形式：

		jmp	label
		db	thunkcode
	label:  
花指令示例:

![image-20210318163607355](/pic/czk/idcfixconfusion/jmp.png)

77390f8a 开始的1字节(68) 和  77390f8d 开始的2字节(29 5a) 是thunkcode;



以上只罗列了四种花指令块，都是单一指令块，当然还有更多的指令块，这里就不一一列举；添加了花指令可以直接的干扰分析者，一两条可以接受，但如果更多的指令块嵌套组合，反反复复上下来回跳转, 分析者估计都能被恶心的够呛；也就是解释为技术上打败不了你，战术上恶心死你；



恰好本次分析样本时，中间就大量的加入了情景2中的花指令；分析时可以手动的调整修改IDA的解析过程，但同时也增加了工作量，而且代码上下看着也不那么连贯；因为花指令单一并且有规律，所以可以采用IDC脚本去实现自动去除这么干扰指令；



# IDC脚本去情景2花指令块

代码逻辑很简单，直接把idc代码贴这里了；

就是一个循环匹配特征的逻辑，唯一注意的地方就是Byte取出来的是一个字节，特征串中是用2个字符来标识的；对比的时候需要把取出的字节格式化成字符串，然后拆分成单字符与特征字符对比。

```
#include <idc.idc>

static matchBytes(StartAddr, Match)
{
    auto Len, i, PatSub,  SrcSub;
    Len = strlen(Match);
    while (i < Len) {
        PatSub = substr(Match, i, i + 1);
        SrcSub = form("%02X", Byte(StartAddr));
        SrcSub = substr(SrcSub, i % 2, (i % 2) + 1);
        if (PatSub != "?" && PatSub != SrcSub)
        {
            return 0;
        }
        if (i % 2 == 1)
        {
            StartAddr++;
        }
        i++;
    }
    return 1;
}

static main() {
	auto StartVa, EndVa, MatchVa,i;

	StartVa = 	0x41200C;
	EndVa 	= 	0x416B8E;
	
	MatchVa	=	StartVa;

	while(MatchVa < EndVa)
	{
		if (matchBytes(MatchVa, "3B??740?") == 0 )
		{
			MatchVa++;
			continue;
		}

		auto nopSize = Byte(MatchVa+3); //取nop掉的代码长度
		Message("addr:%x,%d\n",MatchVa+3, nopSize);
		if( nopSize > 2 ) //样本只有1和2
		{
			MatchVa++;
			continue;
		}
		for (i = 0; i < nopSize+4; i++)
		{
			PatchByte(MatchVa, 0x90);
			MakeCode(MatchVa);
			MatchVa++;
		}
	}
	
	AnalyzeArea(StartVa, EndVa);
	Message("Clear CMP-JZ Opcode Ok\n");
}
```



# IDC脚本执行

下面看一下脚本运行效果；

![image-20210318170213831](/pic/czk/idcfixconfusion/loadscript.png)

IDA->File->Script file选择编写的idc文件,就可以执行脚本了；

最后看一下执行后的效果：

![image-20210318170407360](/pic/czk/idcfixconfusion/scriptresult.png)



这样就把无意义的干扰指令完全用90给nop掉了。

