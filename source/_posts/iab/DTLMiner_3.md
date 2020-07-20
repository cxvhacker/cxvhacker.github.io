---
title: 驱动人生木马blackball再更新,新增linux以及python传播模块
author: iaB
categories:
- 病毒分析
tags: 
- DTLMiner 
date: 2020-07-6 07:30:00
excerpt: 驱动人生木马又又更新了,本次更新中添加了多个模块,并且部分主要传播方式也发生了改变,让我们一起来一探究竟.
---

# 概述
驱动人生木马再次更新,本次更新中在上个版本中进行检测CVE-2020-0796后,首次利用漏洞进行攻击,还新添加了针对linux的SSH爆破以及Redis爆破的传播方式.并且启用了一项曾经在旧版本出现过的以python进行打包的传播模块.  
下面进行详细分析.对未发生变化得部分粗略概括,对更新得部分进行详细说明.  
# 样本分析
## 计划任务
在中毒机器得计划任务上,除了域名得变动没有什么改变,并且使用了多个域名.  
![ALt text](/pic/iab/DTLMiner_3/schtasks.png)  
## a.jsp
这个模块旧功能包括,收集本地机器信息,下载执行矿机(m6.bin,m6g.bin),下载if.bin(传播模块),设置dns,下载执行report.jsp,if_mail.bin(邮件传播模块).  
新增旧进程关闭代码,再代码开始部分,执行其他操作前将当前正在运行得恶意进程关闭.  
![ALt text](/pic/iab/DTLMiner_3/killprocess.png)  
新增模块Kr.bin,这个模块主要是用来关闭一些敏感得服务,进程,以及其他可能抢夺资源得矿机.稍后进行详细描述.  
![ALt text](/pic/iab/DTLMiner_3/krbin.png)  
最后新增ode.bin模块.此模块用来下载新模块并创建计划任务执行.同样稍后进行详细查看.    
![ALt text](/pic/iab/DTLMiner_3/odebin.png)  
## report.jsp
先看旧模块得更新.  
在report.jsp中,上个版本使用得标记是名称为blackball的计划任务作为标记.而再此版本中此处的标记被更换.blackball并没有停止使用只是用作其他模块当中,后面的分析会看到.  
![ALt text](/pic/iab/DTLMiner_3/flag.png)  
功能上没有变化,使用随机名进行建立计划任务并运行.  
## if.bin
if.bin作为传播模块目的上仍然是进行内网扩散.但是在传播方式上发生了比较大的变化.  
首先对于windows平台,停止使用永恒之蓝的传播方式.在解码出来的脚本中,原来的代码依然存在但是已经被注释掉.  
![ALt text](/pic/iab/DTLMiner_3/blue.png)  
作为替换,使用smbghost漏洞进行攻击.  
![ALt text](/pic/iab/DTLMiner_3/smbghost.png)  
被攻击后的所执行的文件中代码也发生了改变.由图可见执行的是amgh.jsp中的代码.    
![ALt text](/pic/iab/DTLMiner_3/smbcode.png)  
这部分代码先针对性得尝试卸载了一些安全软件.  
![ALt text](/pic/iab/DTLMiner_3/smbcode_1.png)  
然后跟report.jsp相同,建立计划任务,但是当中使用的标志位却是blackball.  
![ALt text](/pic/iab/DTLMiner_3/smbcode_2.png)  
最后还有一个神奇操作.  
![ALt text](/pic/iab/DTLMiner_3/smbcode_3.png)  
这段代码的功能是禁用SMBv3的压缩功能,属于是0796的修补措施.病毒作者再攻击成功后将漏洞关闭.这操作用意让人浮想联翩.  

对1433端口的mysql SA账户的爆破没有改变.不再贴出.  

新增针对linux环境22端口的SSH爆破.  
![ALt text](/pic/iab/DTLMiner_3/ssh.png)  
看一下爆破成功的命令,下载core.png并执行.  
![ALt text](/pic/iab/DTLMiner_3/sshcode.png)  
第二个针对linux的是传播方式是Redis未授权访问漏洞.  
![ALt text](/pic/iab/DTLMiner_3/redis.png)  
攻击成功后执行的命令仍然是下载core.png的代码执行.  
![ALt text](/pic/iab/DTLMiner_3/rediscode.png)  
最后来看一下core.png里面是什么.该文件为一个shell脚本.  
![ALt text](/pic/iab/DTLMiner_3/core.png)  
图中代码可见,主要功能是利用linux的计划任务机制,创建计划任务执行a.asp脚本,和启动矿机.Xll/xr矿池为lplp.ackng.com:444.  
在a.asp中储存的同样是shell脚本.该脚本通过网络上大量已知的IOC来关闭其他挖矿程序,主要是通过进程名,文件名,网络连接的ip作为依据进行删除,重复代码比较多,不全部贴出了.  
![ALt text](/pic/iab/DTLMiner_3/aaspkill.png)  
在kill其他抢夺资源的进程后通过本地的.sshown_hosts记录去重新连接本机登录过的ip进行扩散  
![ALt text](/pic/iab/DTLMiner_3/aaspsshhost.png)  
下载矿机xr.zip并解压到.Xll.  
![ALt text](/pic/iab/DTLMiner_3/aaspdown.png)  
最后清空了日志.  
![ALt text](/pic/iab/DTLMiner_3/aaspclear.png)  
而且还利用
稍微总结一下if.bin模块当前版本使用得传播方式:  
- 利用CVE-2017-8464得lnk文件  
- SMBGHOST漏洞利用  
- MySql爆破  
- SSH爆破(目标为linux)  
- Redis未授权访问漏洞(目标为linux)  
## kr.bin
下面来介绍第一位新伙伴kr.bin.  
主要逻辑非常简单.循环Killer.  
![ALt text](/pic/iab/DTLMiner_3/krbin_1.png)  
到底再Kill什么呢?也很清晰.其实它跟前面linux中提到得脚本kill部分非常相似.  
首先关服务.被关闭得服务:  
```
"xWinWpdSrv", "SVSHost", "Microsoft Telemetry", "lsass", "Microsoft", "system", "Oracleupdate", "CLR", "sysmgt", "\gm", "WmdnPnSN", "Sougoudl","National", "Nationaaal", "Natimmonal", "Nationaloll", "Nationalmll","Nationalaie","Nationalwpi","WinHelp32","WinHelp64", "Samserver", "RpcEptManger", "NetMsmqActiv Media NVIDIA", "Sncryption Media Playeq","SxS","WinSvc","mssecsvc2.1","mssecsvc2.0","Windows_Update","Windows Managers","SvcNlauser","WinVaultSvc","Xtfy","Xtfya","Xtfyxxx","360rTys","IPSECS","MpeSvc","SRDSL","WifiService","ALGM","wmiApSrvs","wmiApServs","taskmgr1","WebServers","ExpressVNService","WWW.DDOS.CN.COM","WinHelpSvcs","aspnet_staters","clr_optimization","AxInstSV","Zational","DNS Server","Serhiez","SuperProServer",".Net CLR","WissssssnHelp32","WinHasdadelp32","WinHasdelp32","ClipBooks"
```
![ALt text](/pic/iab/DTLMiner_3/krbin_2.png)  
再删计划任务.被删除得计划任务:  
```
"my1","Mysa", "Mysa1", "Mysa2", "Mysa3", "ok", "Oracle Java", "Oracle Java Update", "Microsoft Telemetry", "Spooler SubSystem Service","Oracle Products Reporter", "Update service for  products", "gm", "ngm","Sorry","Windows_Update","Update_windows","WindowsUpdate1","WindowsUpdate2","WindowsUpdate3","AdobeFlashPlayer","FlashPlayer1","FlashPlayer2","FlashPlayer3","IIS","WindowsLogTasks","System Log Security Check","Update","Update1","Update2","Update3","Update4","DNS","SYSTEM","DNS2","SYSTEMa","skycmd","Miscfost","Netframework","Flash","RavTask","GooglePingConfigs","HomeGroupProvider","MiscfostNsi","WwANsvc","Bluetooths","Ddrivers","DnsScan","WebServers","Credentials","TablteInputout","werclpsyport","HispDemorn","LimeRAT-Admin","DnsCore","Update service for Windows Service","DnsCore","ECDnsCore"
```
![ALt text](/pic/iab/DTLMiner_3/krbin_3.png)  
第三步删除关闭进程.被关闭得进程关键字:  
```
"SC","WerMgr","WerFault","DW20","msinfo", "XMR*","xmrig*", "minerd", "MinerGate", "Carbon", "yamm1", "upgeade", "auto-upgeade", "svshost",
	"SystemIIS", "SystemIISSec", 'WindowsUpdater*', "WindowsDefender*", "update", 
	"carss", "service", "csrsc", "cara", "javaupd", "gxdrv", "lsmosee", "secuams", "SQLEXPRESS_X64_86", "Calligrap", "Sqlceqp", "Setting", "Uninsta", "conhoste","Setring","Galligrp","Imaging","taskegr","Terms.EXE","360","8866","9966","9696","9797","svchosti","SearchIndex","Avira","cohernece","win","SQLforwin","xig*","taskmgr1","Workstation","ress","explores"
```
![ALt text](/pic/iab/DTLMiner_3/krbin_4.png)  
最后根据连接得ip去挂起进程.目标ip为:  
```
'161.35.107.193:80','66.42.43.37:80','167.99.154.202:80','139.162.80.221:80','128.199.183.160:443'
```
![ALt text](/pic/iab/DTLMiner_3/krbin_5.png)  
## ode.bin
再来介绍第二位新伙伴ode.bin.  
这部分代码主要是下载一个文件并执行.并且使用一个kk4kk.log做标记.  
![ALt text](/pic/iab/DTLMiner_3/ode_1.png)  
将20.dat文件下载到temp目录下的一个随机名称.exe文件.  
![ALt text](/pic/iab/DTLMiner_3/ode_2.png)  
在通过验证后,根据权限不同,使用计划任务通过两种方式去执行.  
![ALt text](/pic/iab/DTLMiner_3/ode_3.png)  
## 20.dat
由于20.dat内容比较多,我们将20.dat定为第三位新伙伴.  
被下载下的文件(20.dat)是一个8M多的文件,该文件是通过pyinstaller打包成exe的py程序.通过使用反编译工具pyinstxtractor+uncompyle6得到一个2M的py脚本.由于其中内部实现的函数特别多我们从执行逻辑得顺序去看.  
### 首先是自身的版本检查.  
检测方式是通过读取一个叫path共享内存的方式,这块内存中存放得数据是 "当前路径"+"\*\*"+"版本"+"$$",如果其中没有内容则代表没有正在运行得进程.  
![ALt text](/pic/iab/DTLMiner_3/dat_1.png)  
如果有,则读取其中"\*\*"与"$$"之间得版本进行比较.如果版本低则进行替换并建立计划任务去执行.非常骚气得是替换得过程中在程序最后追加了随机字符.这应该是在躲避一些安全软件得查杀方式.  
![ALt text](/pic/iab/DTLMiner_3/dat_2.png)  
追加随机字符.ranname()函数是一个获得随机字符串得函数.主要用来创建随机名称.  
![ALt text](/pic/iab/DTLMiner_3/dat_3.png)  
版本检查后在保证单例执行得情况下,初始化了一下需要得基本信息,主要都是一些内置得ip地址,用户名,密码字典.  
单例执行.  
![ALt text](/pic/iab/DTLMiner_3/dat_4.png)  
字典信息.这当然不是全部得字典,后续代码中会通过各种方式去扩充.  
![ALt text](/pic/iab/DTLMiner_3/dat_5.png)  
### 构造payload数据  
功力有限,虽然并不知道payload怎么造,但是看这代码.(怎么这么眼熟?)这不是曾经在旧版本中出现过的永恒之蓝吗.  
![ALt text](/pic/iab/DTLMiner_3/dat_6.png)  
### 查找文件并替换.  
这段只调用了一个函数eb.  
![ALt text](/pic/iab/DTLMiner_3/dat_7.png)  
eb函数中,主要功能是在widnows下寻找两个文件,大小分别为6271136和6271280字节,如果找到,则利用cmd命令行替换.这个目的并不明确,可能是去除旧版本程序.  
![ALt text](/pic/iab/DTLMiner_3/dat_8.png)  
### 扩充字典.  
扩充前先将当前进程路径+".eb"添加到ee2中.这个参数是后面PSEXEC用的.等后面会提及到这个函数.  
![ALt text](/pic/iab/DTLMiner_3/dat_9.png)  
然后通过重定向cmd的输出来获得域用户名,管理员组的用户名,以及域管理员的用户名.  
![ALt text](/pic/iab/DTLMiner_3/dat_10.png)  
![ALt text](/pic/iab/DTLMiner_3/dat_11.png)  
通过整理输出信息,将各个信息添加到之前的字典当中.  
![ALt text](/pic/iab/DTLMiner_3/dat_12.png)  
下面又对刚刚抓取到的域用户名分别用123456和password作为密码进行了进行两次尝试攻击.因为最后的ip为localhost,所以认为这是一种尝试猜密码的行为.在攻击成功后则表示猜测正确并将用户名密码记录.  
![ALt text](/pic/iab/DTLMiner_3/dat_13.png)  
### PSEXEC
因为刚刚提到了PSEXEC这个类,直接在这说明一下.  
这个函数可以理解为windows的那个psexec工具,但是病毒作者进行了封装修改.  
![ALt text](/pic/iab/DTLMiner_3/dat_14.png)  
看作者的用法,这个类直接.run加目标ip就好.run当中都是对协议端口,以及认证等相关信息的初始化.最后再doStuff.    
![ALt text](/pic/iab/DTLMiner_3/dat_15.png)  
doStuff中就很明显了.  
第一个参数中(copyfile)就是要复制到目标机器的文件.  
如果参数中的文件名带.eb则复制为temp\svchost.exe或者windows\temp\svchost.exe  
否则复制为temp\dig.exe或windows\temp\dig.exe  
![ALt text](/pic/iab/DTLMiner_3/dat_16.png)  
第二个参数(exefile)就是要复制到目标机器并执行的文件.目标机器中的具体位置则由fr参数控制.  
![ALt text](/pic/iab/DTLMiner_3/dat_17.png)  
第三个参数(command)是要执行的命令行.在复制第二步的文件时,直接跟随在了启动命令当中.  
![ALt text](/pic/iab/DTLMiner_3/dat_18.png)  
这些启动代码都被写在了tmp.vbs中.在执行过后还会被删除.  
![ALt text](/pic/iab/DTLMiner_3/dat_20.png)  
![ALt text](/pic/iab/DTLMiner_3/dat_19.png)  
总之这个类就是利用收集或抓取到的密码或hash凭证去将本地文件发送到目标机器中,并执行相应的命令.  
### mimiakatz
我们再回到20_dat这个脚本当中.在初步收集完域用户名,管理员组的用户名,以及域管理员的用户名后又调用了mmka.  
![ALt text](/pic/iab/DTLMiner_3/dat_21.png)  
实际上mmka函数就是mimiakatz.它通过内置mimiakatz的powershell代码,直接写到m2.ps1去执行,并通过重定向获得执行结果.  
![ALt text](/pic/iab/DTLMiner_3/dat_22.png)  
通过整理结果去获取抓取到的用户名和ntlm哈希值.一并合入爆破使用的列表当中.    
![ALt text](/pic/iab/DTLMiner_3/dat_23.png)  
### 下载命令线程
开启了一个线程并执行.  
![ALt text](/pic/iab/DTLMiner_3/dat_24.png)  
这个线程中主要功能是下载e.png这个文件,根据后续代码判断,其中储存的应该是命令行,并执行其中的命令.  
![ALt text](/pic/iab/DTLMiner_3/dat_cur.png)  
### 漏洞扫描
最后则是收集ip地址,并对非xp系使用收集到的ip进行了扫描.使用的是scansmb方法.  
收集ip是利用find_ip这个函数,它的原理是使用命令行和指定网站去收集跟本机相关的ip地址.并只获取其中的网络地址.  
使用ipconfig -all命令.  
![ALt text](/pic/iab/DTLMiner_3/dat_25.png)  
使用netstat -na命令.  
![ALt text](/pic/iab/DTLMiner_3/dat_26.png)  
使用http://ip.42.pl/raw和http://jsonip.com ip查询网址.  
![ALt text](/pic/iab/DTLMiner_3/dat_27.png)  
扫描时还对一些ip段进行了过滤.  
![ALt text](/pic/iab/DTLMiner_3/dat_28.png)  
然后对随机ip进行扫描使用的是scansmb2这个方法.  
![ALt text](/pic/iab/DTLMiner_3/dat_29.png)  
而scansmb和scansmb2这两个函数的区别仅仅在于其中对psexec类fr参数的传参不同.  
### scansmb  
这个函数分三大块.针对域使用域用户名密码进行PSEXEC,使用永恒之蓝+PSEXEC,使用其余的用户名密码进行PSEXEC.  
![ALt text](/pic/iab/DTLMiner_3/dat_30.png)  
看一下永恒之蓝的payload.主要就是下载gim.jsp去执行,并修改administrator的密码为k8d3j9SjfS7.gim.jsp跟if.bin模块中smbghost攻击成功后所执行的amgh.jsp相同.    
![ALt text](/pic/iab/DTLMiner_3/dat_31.png)  
下面漏洞攻击成功后所执行的PSEXEC内容.主要是将复制过去的svchost文件和dig文件去重命名成一个随机名称.然后建立计划任务去执行这些程序.  
![ALt text](/pic/iab/DTLMiner_3/dat_32.png)  
由此可以得知,利用永恒之蓝漏洞仅仅是给PSEXEC去创造一个可行的凭证去进行操作.  
# 分析总结
此次更新的全部内容已经介绍完毕了.该病毒从原本的本地文件传播到无文件传播,从仅使用网络传播到添加USB以及邮件传播,从永恒之蓝到最新的SMBGhost,从windows到linux.  这个病毒采用的技术可以说是与时俱进,并且非常完善.至此,该病毒已经集成了,永恒之蓝漏洞,CVE-2017-8464 lnk文件漏洞,SMBGhost漏洞,Redis未授权访问漏洞,钓鱼邮件 并且附带ipc,mysql,ssh爆破于一身,可谓是空前的强大.至于为什么放弃了原本powershell中的永恒之蓝利用方法而采用了旧版更加臃肿的py版本有点让人疑惑.  
# 防护建议
- 安装MS17-010永恒之蓝补丁
- 安装CVE-2017-8464 LNK漏洞补丁
- 安装CVE-2020-0796 smbghost补丁
- 使用高强度的密码,不使用重复密码,并定期更换密码
- 建议安装如下补丁防止Mimikatz窃取密码:  
https://support.microsoft.com/zh-cn/help/2871997/microsoft-security-advisory-update-to-improve-credentials-protection-a  
并使用以下工具关闭其他能够窃取凭证的路径  
http://download.microsoft.com/download/E/2/D/E2D7C992-7549-4EEE-857E-7976931BAF25/MicrosoftEasyFix20141.mini.diagcab
