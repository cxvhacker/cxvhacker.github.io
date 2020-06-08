---
title: 驱动人生木马再度更新,新增钓鱼邮件传播
author: iaB
categories:
- 病毒分析
tags: 
- DTLMiner 
date: 2020-05-18 20:34:22
excerpt: 在短短一个月中,DTLMiner再次更新,从bluetea版本更新为blackball版本.来详细看一下此次更新做了哪些改动.
---

# 概述
驱动人生木马"喜迎"再次更新,本次更新内容主要包括:  
- bluetea标志更改为blackball标志  
- 新增使用钓鱼邮件传播模块  
- 停止对CVE-2019-0708(BlueKeep)漏洞的检测改为对CVE-2020-0796(SmbGhost)的检测  
- 不再使用随机域名修改host文件

# 样本分析
下面来具体看一下此次版本更新后的变化,对于与旧版本相同部分只简单概述,更新部分进行详细叙述.
## 计划任务
上一个版本中,计划任务中的操作 是利用powershell获取x.js文件中的代码执行,进而获取x.jsp进行.  
而新版本直接下载了一个叫a.jsp的文件中的代码执行,也取消了Lemon_Duck的标志.  
![ALt text](/pic/iab/DTLMiner_2/schtasks.png)
## a.jsp
a.jsp的内容根之前x.jsp的内容大体相同.包括:收集信息(计算机名,guid,操作系统信息,mac地址,用户名,域名,显卡信息,可移动磁盘以及网络磁盘信息);根据系统情况下载矿机(m6.bin,m6g.bin),下载if.bin(传播模块),设置dns,下载report.jsp中的数据执行.  
与之前不同之处在于,新添加了邮箱的检测,以及if_mail.bin(邮件传播模块数据的下载并执行).  
![ALt text](/pic/iab/DTLMiner_2/a_jsp_if_mail_bin.png)
由代码可见,病毒程序通过注册表检测是否安装outlook,如果存在则关闭警告并下载执行if_mail.bin传播模块.  
## report.jsp
report.jsp中的改动很小,只是修改了计划任务的内容,以及检测标志.  
旧版本:  
![ALt text](/pic/iab/DTLMiner_2/task_old.png)
![ALt text](/pic/iab/DTLMiner_2/flag_old.png)
新版本:  
![ALt text](/pic/iab/DTLMiner_2/task_new.png)
![ALt text](/pic/iab/DTLMiner_2/flag_new.png)
建立的任务的任务名以及文件夹名 仍然保持随机字符串的状态.  
## if_mail.bin  
if_mail.bin主要是利用了outlook的com接口.利用你的邮箱去发送给其他人带有恶意文件的钓鱼邮件.恶意文件有两个,一个是readme.doc一个是readme.js.  
先判断是否是管理员权限.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_1.png)
如果拥有管理员权限,则执行这段cmd命令  
![ALt text](/pic/iab/DTLMiner_2/if_mail_2.png)
经过整理如下.功能就是建立pipe管道,等待接收管道中发来得数据并执行.
![ALt text](/pic/iab/DTLMiner_2/if_mail_3.png)
那现在接收数据部分有了,继续执行下面代码发送数据.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_4.png)
如果没有管理员权限,则直接IEX执行.  
这一部分就是根据情况,在两种方式中去选择执行$mail_code得代码.下面看一下这些代码.其中有大片代码是使用编码加密执行得,下面直接解码展开描述.  
前半部是在进行钓鱼邮件中附件中文件的数据得构建.  
doc文件的标题,正文,js文档的内容.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_5.png)
漏洞文件的数据头,ole对象等.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_6.png)
最终在%temp%目录生成readme.doc和readme.js两个文件.  
后半部则是利用com接口,使用你得邮箱去发送邮件.  
首先收集目标邮件地址.分别从联系人,收件箱,发件箱当中遍历.并且获取当前用户的邮箱地址.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_7.png)
将文件readme.js打包成readme.zip,并将获取到的当前邮件地址,联系人收件箱发件箱数量信息上传到服务器.    
![ALt text](/pic/iab/DTLMiner_2/if_mail_8.png)
邮件的内容是从一个集合中随机的.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_9.png)
随机的集合.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_10.png)
最后就是将readme.doc,readme.zip添加到附件,发送给所有收集到的邮件地址,并删除所有记录当中与该邮件有关的记录.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_11.png)
附件中的readme.doc实质上是一个利用了CVE-2017-8570的rtf文件.当用户打开该文件后,会在%temp%目录生成一个随机名称的.sct文件并加载.该文件内部可执行脚本代码.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_12.png)
readme.js文件则是利用大量空白来进行误导迷惑用户,实际代码被存放在最后.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_13.png)
![ALt text](/pic/iab/DTLMiner_2/if_mail_14.png)
两个文档利用的最终目标相同,都是获取7.php的内容执行,并执行bpu函数,这个函数则是在7.php中实现的.  
7.php的代码如下:  
![ALt text](/pic/iab/DTLMiner_2/if_mail_15.png)
如果bpu传入参数是http开头则进行下载文件数据执行的操作.该版本则是用来下载mail.jsp文件当中的数据执行.但是这个函数的功能并不止如此,下半部的代码,还可以使用管理员权限执行任意命令.  
mail.jsp与report.jsp几乎相同,不在重复描述,但是值得一提的是,mail.jsp中出现了新的域名.  
![ALt text](/pic/iab/DTLMiner_2/if_mail_16.png)
## if.bin
作为功能最强的传播模块,并没有过多的改动.  
首先是攻击成功后的payload,从原来的建立计划任务再进一步动作,改为直接使用powershell加载相应的jsp数据执行.这些作为payload的jsp文件中的代码和mail.jsp功能一致.  
![ALt text](/pic/iab/DTLMiner_2/if_bin_1.png)
移动磁盘感染部分将readme.js也添加了进去.  
![ALt text](/pic/iab/DTLMiner_2/if_bin_2.png)
![ALt text](/pic/iab/DTLMiner_2/usb.png)
以前对cve-2019-0708的检测被替换为CVE-2020-0796(SmbGhost)的检测.  
![ALt text](/pic/iab/DTLMiner_2/if_bin_3.png)
RDP爆破部分在爆破成功后,并没有尝试执行payload命令而是在上报信息后结束.  
![ALt text](/pic/iab/DTLMiner_2/if_bin_4.png)
if.bin除去密码字典的更新,模块的改动就是这些了.  
# 总结
这次更新从流程上看精简了执行步骤,从原来的多步间接下载数据执行,改为了直接下载最终数据执行,减少了网络请求次数,提高了执行效率;更新域名来逃避监测软件得检测封锁;添加了新的传播方式,并且还在不断尝试统计新的可能被利用的传播方式.可见该病毒也在跟随着技术的发展不断进步.




