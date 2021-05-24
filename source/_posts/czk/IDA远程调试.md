---
title: IDA远程调试
author: czk
categories:
- IDA远程调试
tags: 
- IDA远程调试
date: 2021-05-21

---
# 概述

​	最近拿到一个linux挖矿病毒样本；需要快速的知道病毒的一些信息；windows下有一些行为分析工具，能清晰的看到程序做了哪些动作，但是linux上找了半天也没有发现有好的工具；最后只能用IDA分析了；IDA中发现，明文字符串都是垃圾无意义的字符，并且还做为了参数，猜测可能有解密；静态太耗时间，所有就想到了用IDA远程动态调试；

<!-- more -->

# 环境：

```
	工具：IDA 6.8
​	主机: windows 64
​	虚拟机:Ubuntu 64
​	样本：
		壳： UPX(3.95)
		MD5: 0622A0D13D0E86C784A224348E453702
		SHA1: B2ECE977C51ED800A324424167EF3DB346789266
```



# 样本脱壳：

​		因为样本有壳,所以先去upx官网下载下来最新版本，然后执行 upx.exe -d 样本名，进行脱壳；命名脱壳后的程序为crondr_unpack；



# linux中组件配置：

​	在IDA目录下的dbgsrv目录中就是IDA附带的一些调试组件；

![image-20210521105028104](/pic/czk/IDADebug/sbgsrc.png)

linux_server：在Linux计算机上执行的、用于调试32位Linux应用程序的服务器组件。
linux_server64：在64位Linux计算机上执行的、用于调试64位Linux应用程序的服务器组件。

想调试linux的样本就要把对应的组件放到linux系统上去，因为我的虚拟机环境是64位环境，所以把linux_server64拷贝过去，并给他添加执行权限 chmod 777 linux_server64，运行起来；

![image-20210521105623253](/pic/czk/IDADebug/copy2linuxforrun.png)

成功后为listening on 0.0.0.0:23946



# windows下配置IDA：

​	打开空白IDA，找到Debugger->Run->Remote Linux Debugger;

​	![image-20210521110534074](/pic/czk/IDADebug/linuxdebug.png)

![image-20210521110758443](/pic/czk/IDADebug/debuggersetting.png)

设置前一定要先把脱壳后的样本放到linux虚拟机里面去；

其中Application就是待调试的文件的全路径；我这里是

```
/home/uyt62/uyt62/crondr_unpack
```

Directory是存放待调试文件的目录，把上面的路径去掉文件名就可以了

```
/home/uyt62/uyt62
```

Parammeters指定的命令行参数，样本没有，这里不填；

Hostname	远程调试机的ip地址；

​		linux虚拟机中使用ifconfig,查看；

Port： 远程调试服务器的TCP监听端口号； 填入23946；linux_server64启动后回显中的，连蒙带猜也得填这个数；

Password: 远程调试机的密码。  我的是1，默默的填上吧；

全部填好后点击OK,调试起来；成功后将会看到如下窗口；RIP在入口点处；

![image-20210521113500194](/pic/czk/IDADebug/debugSuccess.png)



# 调试结果：

调试起来就能看到动态解密出来的字符串了；

![image-20210521113829783](/pic/czk/IDADebug/DebugResult.png)