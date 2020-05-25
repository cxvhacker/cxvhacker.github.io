---
title: 驱动人生木马(DTLMiner)样本分析
author: iaB
categories:
- 病毒分析
tags: 
- DTLMiner 
date: 2020-04-05 17:34:46
excerpt: DTLMiner是自2019年以来最为活跃的挖矿木马病毒家族之一,以其顽固性,强大的传播性以及惊人的更新频率所被人们知晓.  现在来让我们来详细了解一下,该病毒的运作过程. 
---

# DTLMiner概述
DTLMiner是一种主要以挖矿为目的木马病毒.该病于2018年末,被黑客利用驱动人生的升级通道进行传播,就此开始大规模爆发.也由此得来驱动人生木马的名称.该病毒初期主要使用永恒之蓝漏洞进行扩散并从互联网下载恶意程序执行,因此也被称作永恒之蓝下载器.驱动人生公司也已经在事件发生后立刻对漏洞进行了修复.  
自病毒爆发起,该病毒的大致时间线如下:  
![ALt text](/pic/iab/DTLMiner_1/time.png)  

可见此病毒更新十分频繁,由最初的python打包的可执行文件逐渐改变为使用powershell代码,实现无文件落地.使用windows计划任务机制进行持久化驻留.并且不断添加新的传播方式.到目前为止该病毒已经包含了永恒之蓝,PASH-THE-HASH,ipc/rdp/mysql密码爆破,震网三代漏洞,传播性异常强大.并且还在检测新的漏洞信息,不排除将来可能被利用的可能.
下图显示了整体的行为流程:  
![ALt text](/pic/iab/DTLMiner_1/97687a31e321fd3f5ace553a43bfdae1.png)  

下面就以最近获取的最新样本进行详细分析.
# 样本分析  
中毒的机器通常最为直观的表现就是CPU利用率过高,打开任务管理器可以发现有大量的powershell进程正在运行.并且在计划任务中发现了许多名称随机的任务. 并且该病毒的所有脚本都是通过多层的编码以及字符串替换进行混淆,下面列出的代码都是经过解码整理过后的代码.  
## 计划任务  
提取导出被添加的随机名称的计划任务可见计划任务的内容:  
![ALt text](/pic/iab/DTLMiner_1/60bce8fc-6995-40e2-877a-12db2f27fd8b.png)
通过解码过后为如下powershell脚本:  
![ALt text](/pic/iab/DTLMiner_1/4b59e76d-488c-46e2-8576-321f31aba294.png)  
这段脚本的目的是下载x.js中的数据执行.
## x.js
该文件先收集获取用户信息,包括计算机名,GUID,mac地址,操作系统内版本,系统位数,以及当前运行病毒时间.  
![ALt text](/pic/iab/DTLMiner_1/7db29e9b-665b-463e-b864-1f87323f5053.png)  
并拼接成finalurl,当利用该url进一步下载执行数据时,将数据上传.  
![ALt text](/pic/iab/DTLMiner_1/d1db3a15-026a-4405-a199-2de65fe3f030.png)  
使用url下载x.jsp并通过RSA校验后执行  
## x.jsp 
这段脚本再次获取用户信息,当前用户权限,guid,mac地址,系统版本,域,显卡,内存大小,移动磁盘,网络磁盘,以及时间戳.    
![ALt text](/pic/iab/DTLMiner_1/4d847695-13a8-4718-90ce-2098713bb522.png)  
如果使用n卡并且为64位系统得情况下,下载nvdg.dat并解压到temp文件夹  
![ALt text](/pic/iab/DTLMiner_1/e20431f4-1f84-43ce-9564-8be67ef20173.png)  
gcf函数返回代码字符串函数返回代码字符串,功能是利用powershell进程下载验证本地文件,md5不相同则下载新的文件:$code是需要执行的代码,$md是下载文件的md5,$fn是下载的文件名/同样是本地文件名  
![ALt text](/pic/iab/DTLMiner_1/cc295154-ca33-43d5-8bf2-05ccf93b767d.png)  
gpa函数返回代码字符串,代码功能是读取文件,并且用0x0a做分割,前半部是直接执行的脚本,后半部分是压缩2进制数据,后半部数据解压后生成$fnam.exe.ori文件,使用cmd复制成$fnam.bin.exe执行  
![ALt text](/pic/iab/DTLMiner_1/gpa.png)
gcode函数返回代码字符串,功能是建立Global互斥体  
![ALt text](/pic/iab/DTLMiner_1/gcode.png)
SIEX的功能是下载代码并执行,但是这种代码数据前174字节是校验的hash值,剩余是被校验的数据  
![ALt text](/pic/iab/DTLMiner_1/bdbcf8e1-7377-41f6-84ac-3eb9dabb46d5.png)
stp则是使用cmd执行$gra传入的命令  
![ALt text](/pic/iab/DTLMiner_1/stp.png)  
根据当前机器的状况,使用相应方式下载,执行if.bin,m6.bin,m6g.bin
(if.bin是病毒的攻击模块,用来进行横向传播,在后面进行分析,m6,m6g则是挖矿程序进行挖矿)     
![ALt text](/pic/iab/DTLMiner_1/6b905f02-35bd-4f52-bf9c-6a191cf95f21.png)  
修改dns 8.8.8.8    9.9.9.9  
![ALt text](/pic/iab/DTLMiner_1/c3f89ef0-f8ce-4a7b-b935-3f1f13f815a0.png)  
下载report.jsp数据执行  
![ALt text](/pic/iab/DTLMiner_1/de51db24-95ce-45fd-acf8-786d57a49d42.png)  
修改网络及防火墙配置,并且利用Gkwidjuq8391j.txt做标记互斥,开65529端口也是为攻击模块进行标记  
![ALt text](/pic/iab/DTLMiner_1/77e292c2-1ca8-4fbc-8f89-40d2d3f0e3f8.png)  
稍微整理一下,这部分代码的主要功能是下载传播模块和挖矿模块执行.为提高成功率,修改了dns以及防火墙配置.并且还下载执行report.jsp  
## report.jsp
这部分代码主要都是跟持久化的计划任务相关.  
产生随机域名  
![ALt text](/pic/iab/DTLMiner_1/host.png)  
首先检查是否存在bluetea这个任务标记,如果存在则不继续后续动作.  
![ALt text](/pic/iab/DTLMiner_1/72b827dd-9918-4ec7-a20c-aef7c64077cf.png)  
不存在则创建随机名称的计划任务.分别有三种位置,根目录中,根目录随机文件夹中,\MicroSoft\Windows\随机文件夹中.  
![ALt text](/pic/iab/DTLMiner_1/task.png)
建立计划任务,并替换任务内容中的变量.  
![ALt text](/pic/iab/DTLMiner_1/6f50e62a-70fd-4872-8b27-344fecec4acf.png)  
计划任务的内容与我们分析开始的计划任务基本一致,下载并执行x.js,并且由上方替换的内容可以看出,该任务会将随机出来的域名添加到host文件中.  
![ALt text](/pic/iab/DTLMiner_1/a0fb59f9-821c-49d8-a543-abbc132c3344.png)  
最后建立bluetea标志.  
![ALt text](/pic/iab/DTLMiner_1/buletea.png)  

## 传播模块(if.bin)
if.bin在x.jsp的代码中被执行,它是该病毒的攻击模块,目前包含永恒之蓝,端口扫描器,CVE-2017-8464,PASS-THE-HASH,mysql/ipc/RDP密码爆破字典,CVE-2019-0708漏洞检测模块.该脚本解码完成后已达1万多行之多.所有功能全部呈模块化.    
永恒之蓝漏洞利用模块  
![ALt text](/pic/iab/DTLMiner_1/f9f7d1a6-e771-4567-9e91-4e28f79dcae6.png)  
漏洞扫描器模块  
![ALt text](/pic/iab/DTLMiner_1/31c0d7df-c4d6-4148-ad8f-cdba811c848d.png)  
CVE-2017-8464漏洞利用模块  
![ALt text](/pic/iab/DTLMiner_1/4f845d8e-bf04-43a4-b1b2-7f1c78e00b17.png)  
Killer模块(这个模块的目的应该是,关闭其他抢占资源的程序)  
![ALt text](/pic/iab/DTLMiner_1/25b84362-456b-4cd6-a274-90ccc7eae516.png)  
PASS-THE-HASH模块  
![ALt text](/pic/iab/DTLMiner_1/0cf7718e-3341-4123-b9d3-e46e2c20e68f.png)  
还会下载Mimikatz和wfree工具(mimi.dat就是以base64编码的Mimikatz工具,wfree是一款通过3389连接远程桌面的工具)  
![ALt text](/pic/iab/DTLMiner_1/4e06e543-888c-4db0-88dd-9004469ae0d3.png)  
自带一些字典,之后会跟抓取的密码进行整合  
![ALt text](/pic/iab/DTLMiner_1/90554522-a940-46ba-a91a-09471d2a0942.png)  
在执行脚本运行后先使用CVE-2017-8464对U盘进行感染  
![ALt text](/pic/iab/DTLMiner_1/f35db156-7113-43af-8d46-570c6c8f6ab7.png)  
会在U盘中产生大量漏洞利用的快捷方式以及而已dll文件  
![ALt text](/pic/iab/DTLMiner_1/lnk.png) 
该漏洞无需用户操作,只要用户打开该盘浏览就会主动加载恶意文件  
关闭一些占用资源的进程服务  
![ALt text](/pic/iab/DTLMiner_1/70844adf-6531-4d15-b48d-a290dc6dffae.png)  
使用PASS-THE-HASH抓取用户名密码进行整合  
![ALt text](/pic/iab/DTLMiner_1/2039e5b7-e92f-4b2e-aa75-52d26742065b.png)  
进行端口扫描  
![ALt text](/pic/iab/DTLMiner_1/c1839dac-bebc-4a9a-854d-addbfc74c92f.png)  
对开放445端口的主机先使用抓取到的Hash进行爆破,再使用永恒之蓝漏洞攻击  
![ALt text](/pic/iab/DTLMiner_1/6d1fdf89-d27a-4fc3-a126-b21df9696dfa.png)  
对开放1433端口的主机进行mysql sa账户爆破  
![ALt text](/pic/iab/DTLMiner_1/6dd71ffc-464d-48cb-85f5-649ab53294ab.png)  
对开放3389的机器进行检测CVE-2019-0708是否存在上报  
![ALt text](/pic/iab/DTLMiner_1/c23e4fc0-205c-4201-a97b-80bc4936fbd0.png)  
进行RDP爆破  
![ALt text](/pic/iab/DTLMiner_1/1019701d-9f2c-4e53-b322-d145d57bd3b3.png)  
当攻击成功后进行的动作.篇幅较长只展开一个比较清晰的展示.  
![ALt text](/pic/iab/DTLMiner_1/c396c00e-258c-41d3-819c-b60c90f5d6b2.png)  
他们最终都是借用cmd命令去执行powershell命令.命令的内容为建立计划任务.去下载执行相应的文件.  
ipc爆破方式就是执行ipc.jsp,永恒之蓝方式就是eb.jsp,rdp爆破的方式就是rdp.jsp以此类推.
而eb.jsp\ipc.jsp\ms.jsp\rdp.jsp虽然名称不同,但是其中代码数据除攻击方式的标志外都是相同的.并且与前面分析的report.jsp相同.最终目的是建立计划任务执行x.js中的代码.  
# 总结梳理:
该病毒所有功能进行分步执行,每一步都进行多层编码进行混淆免杀,几乎所有功能都是利用powershell进程直接通过互联网在下载服务器中代码数据到内存执行,完全避免了文件落地,极大的逃避了安全软件对其的检测,并且只虚要通过控制服务器当中脚本代码,就完全可以控制中毒的电脑进行任何动作(更新,下发文件等).在分析过程中也可以发现每一步下载执行都含有对代码完整性验证的步骤,十分谨慎.攻击方式多,Mimikatz密码爆破加漏洞利用,对拥有大量机器的企业危害十分严重,并且难以彻底清除.  
# 防护建议:  
病毒固然防护,但它也只是利用了暴露出系统的弱点进行攻击,只要做好必要的防范措施并不可怕.针对该病毒有几条防护建议:  
- 不使用445端口的机器,利用防火墙高阻止向445端口连接  
- 安装MS17-010永恒之蓝补丁  
- 安装CVE-2017-8464 LNK 远程执行代码漏洞补丁  
- 安装CVE-2019-0708 RDP服务远程代码执行漏洞补丁  
- 使用高强度的密码,不使用重复密码,并定期更换密码  
- 建议安装如下补丁防止Mimikatz窃取密码:  
https://support.microsoft.com/zh-cn/help/2871997/microsoft-security-advisory-update-to-improve-credentials-protection-a  
并使用以下工具关闭其他能够窃取凭证的路径  
http://download.microsoft.com/download/E/2/D/E2D7C992-7549-4EEE-857E-7976931BAF25/MicrosoftEasyFix20141.mini.diagcab  
- 如果不是必要可将powershell功能禁用