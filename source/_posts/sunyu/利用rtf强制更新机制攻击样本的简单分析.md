---
title: 利用rtf强制更新机制攻击样本的简单分析
author: 此生汝梦 
categories:
- 漏洞分析
tags: 
- rtf强制更新
- rtf嵌入对象
date: 2020-07-24
excerpt: 利用rtf强制更新机制攻击样本的简单分析，样本利用强制跟新机制连续打开5个带宏excel，利用带宏的恶意excel进行攻击。
---
# 前言
　　本文只陈述一下在分析这个利用样本的过程中的一些发现。
# 分析环境与工具
* 操作系统：
　  windos7 专业版(64位)
* 软件：
　  Office Professional Plus 2019 (Simplified Chinese)
* 工具：
　  oletools

# 样本分析
## 获取样本
    文件MD5-1:7D1347B165972290CACE9E640FC430E7（分析的主体）
    文件MD5-2:49846701BD43FB627D48103181CC3481（对照）
　  可从微步等平台自行下载
　
## 运行现象
直接执行样本

样本1：
![ALt text](/pic/sunyu/rtf强制更新/图片1.png)
![ALt text](/pic/sunyu/rtf强制更新/图片2.png)
![ALt text](/pic/sunyu/rtf强制更新/图片3.png)
整个执行过程就是，双击打开RTF文件，这时会先依次弹出5个带宏的excel文档（offcie2019自带了一些安全措施没有让宏自动运行起来，所以恶意代码没有运行），然后再显示rtf文档的内容。

样本2：
![ALt text](/pic/sunyu/rtf强制更新/图片4.png)
打开后可以看到，使用具有诱导性的内容去诱使用户点击嵌入的恶意程序（这里不对恶意程序作展开，仅仅展示一下非漏洞利用型的rtf恶意文档的形式）。
## 分析
### 寻找嵌入的excel对象
总共有两种方法。

第一种方法：
rtf嵌入对象，在rtf内容中肯定是有图标的，不管整个图标有多小，所以对打开的样本1进行查看。
![ALt text](/pic/sunyu/rtf强制更新/图片5.png)
在页脚处可以看到5个方框一样的东西。
选中其中一个后进行双击打开。
![ALt text](/pic/sunyu/rtf强制更新/图片6.png)
打开了一个带宏的excel，此时这个嵌入的excel对象就找到了

第二种方法:
使用oletools工具集中的rtfobj工具
![ALt text](/pic/sunyu/rtf强制更新/图片7.png)
通过上图中rtfobj的查询结果可以发现，虽然嵌入了5个对象，但这5个对象的MD5一致是同一个东西。

### 提取分析
如果没有对excel中的宏进行反分析处理的话，可以直接在rtf中双击打开excel对象，然后查看宏即可。（下图为手动构建见的rtf中嵌入excel对象添加宏且不对宏进行处理的情况）
![ALt text](/pic/sunyu/rtf强制更新/图片8.png)
但是一般恶意文档的作者为了防止杀毒软件对其扫描都会把宏的源码抹掉，使其无法通过excel软件的查看宏功能查看其具体的内容。
这时就需要使用rtfobj工具的-s all 功能，将rtf中的对象提取出来。
![ALt text](/pic/sunyu/rtf强制更新/图片9.png)
因为5个对象的md5均一致所以这里对其中一个进行分析即可。
直接提取的对象修改成xls时不可取的，excel打不开，会提示格式无效。
![ALt text](/pic/sunyu/rtf强制更新/图片10.png)
尝试使用oletools工具集中的olevba查看其中的宏也不行。
![ALt text](/pic/sunyu/rtf强制更新/图片11.png)
但通过die查看其格式，确为offcie文件。
![ALt text](/pic/sunyu/rtf强制更新/图片12.png)
这就很让人很困惑，但是想到office文件其本质都是压缩包，尝试使用解压缩软件打开它。
![ALt text](/pic/sunyu/rtf强制更新/图片13.png)
确实能打开，对其进行解压后，使用olevba工具对Package进行提取宏，此时便可正确查看宏。
![ALt text](/pic/sunyu/rtf强制更新/图片14.png)

在前面的分析中，还有一个小问题没有解决，就是提取的出来rtf对象并不能直接修改成xls，这是正常的rtf提取对象就这样呢还是样本的作者做过特殊处理呢？
通过自行尝试构建正常的嵌入excel的rtf文档后得出结论，正常的rtf提取出来的对象跟样本提取出来对象结构上类似，而且都不能直接通过修改后缀名的方式进行解析，都需要通过解压对象，然后对package文件进行宏提取才可以。


### 利用rtf强制更新的分析
其实到上面这个样本的分析过程就基本结束了，剩下就是解读VBA脚本看看这个样本干什么的事了，基本没什么难度。
但是这个样本是如何使excle文档在rtf加载时自动运行的呢？要知道一般rtf恶意文档想要加载对象都是需要通过内容去诱导用户点击的。
通过查询资料了解到，rtf有一个强制更新的机制.如下图的objupdate的解释，使其在对象显示前（rtf加载时）强制对其进行更新。
![ALt text](/pic/sunyu/rtf强制更新/图片15.png)
这一点体现到具体文档上就是在加载rtf时，先把加了objupdate标签的对象先运行一下。
![ALt text](/pic/sunyu/rtf强制更新/图片16.png)
使用Notepad++打开rtf样本，可以看到在嵌入的excel对象前，均添加了objupdate标签。
这是office的一个比较坑的机制，目前并没有比较好的办法从根源上阻断它，只能平时注意不要打开来历不明的文档。

# 构建类似的样本
## 构建一个嵌入带宏excel的RTF
首先新建一个RTF，然后插入→对象→带宏的excel。
![ALt text](/pic/sunyu/rtf强制更新/图片17.png)
然后在插入的excel中选择视图→宏→录制宏，然后随便起个名，再停止录制宏，这样就可以再试视图→宏→查看宏中看到一个空白宏
![ALt text](/pic/sunyu/rtf强制更新/图片18.png)
![ALt text](/pic/sunyu/rtf强制更新/图片19.png)
然后再将宏的名修改成auto_open一类特殊宏名，就算完成了。

## 使rtf强制更新excel
使用Notepad++打开刚才保存的rtf,在excel对象前添加objupdate标签即可。
![ALt text](/pic/sunyu/rtf强制更新/图片20.png)

## 使用效果
在打开RTF加载时成功弹出excel.
![ALt text](/pic/sunyu/rtf强制更新/图片21.png)

# 参考链接
http://www.biblioscape.com/rtf15_spec.htm RTF1.5规范