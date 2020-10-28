---
title: Windows进程注入技术总结(一)
author: iaB
categories:
- Windows
tags: 
- Inject 
date: 2020-10-9 07:30:00
excerpt: windows注入是一个被许多人反复研究的话题,并且目前已经拥有许多技术去将数据从一个进程注入到另一个进程当中.恶意代码常常使用注入技术去获得特权,执行敏感操作以此来获得隐匿性并且绕过安全产品的检测.
---

# 概述  
Windows平台下的进程注入技术一直以来都是一个被人精心研究的话题,并且目前已经拥有许多技术去将数据从一个进程注入到其他进程当中.恶意代码最是常用这种技术去进行敏感操作,以达到隐藏自身,绕过安全产品检测的目的.为了能够全面了解这项技术,我学习了Safebreach Labs曾经在BlackHat大会上所讨论的一个主题,该主题包含了尽可能所有的已知的并且在当前环境(Win10)下可用的注入技术,并进行了总结.由于内容较多,先分块进行分享.
这篇主要是阐述在学习过程中所需要的知识铺垫,一些观点的解释,以及一些传统的经典注入方式.  
# Windows进程注入
## 熟悉的注入种类
Safebreach Labs在个话题中提出了一个"true process inject techniques"的说法,所以在此将注入技术分为三类:  
1. Process spawning  
这类方法是创建一个合法的进程,并在进程开始运行之前挂起并修改它,这类技术可行,但缺少隐蔽性.
2. 进程初始化时注入  
这类方法源于某些进程在开始运行时需要加载某些dll,通常这类方法都会需要权限,并且被 Extension Point Disable Policy 所缓解.
3. 注入到正在运行的进程  
这类技术正是这个主题要进行讨论的,也是被称为"true process inject techniques"的技术.注入到正在运行的进程中通常包含两项子技术:  
a.在目标进程中准备内存空间  
b.执行在目标进程中的逻辑代码.    

## 当下的Windows的环境   
如今使用windows 10 64-bit操作系统的人正在大量增长,这一环境的改变对注入技术也产生了极大的影响,所以我们先来了解一下当下环境.  
### X64  
在x86中,除fastcall的所有调用方式都将参数放到栈上.而x64中的调用约定将前4个参数放到寄存器中(RCX,RDX,R8,R9),其余的参数放到栈上.这使得在设计一个64位的payload时要变得更加困难,因为这些payload在调用一个函数时需要控制更多的寄存器.理论上可以使用popad这类寄存器将数据弹出到寄存器中,然而这个指令在x64模式中不可用.  
### Win10新增的安全机制  
- CFG(Control Flow Guard)  
这是微软对windows(8.1,10)中CFI (Control Flow Integrity)的实现.编译器优先于每个间接调用使用_guard_check_icall来检查调用目标的合法性.合法性也是由编译器提供,是一个为每个模块以16字节对其进行记录的列表,以"bitmap"的方式进行访问.为了让该机制生效,调用模块和被调用必须都支持CFG.
- ACG(Arbitrary Code Guard) 
任意代码保护(ACG)是新加入到Windows操作系统之中的可选缓解措施，该机制主要尝试检查如下行为并进行阻止：
1、现有代码被修改(代码页不能变为可写)
2、在数据段上写入代码并执行(数据不能变为代码)
为了实现上述目标，任意代码保护（ACG）机制会执行以下规则：
内存不能同时具有写入（W）和执行（X）属性。
这项功能防止进程在内存中写入数据并执行,即内存不能同时据有写入和执行属性,也就是调用 VirtualAlloc VirtualProtect等函数无法再使用 PAGE_EXECUTE_READWRITE这类参数.注意,这项策略可以选择开启或关闭,并且VirtualProtectEx的策略使用的是调用者的策略. 
- CIG(Code Integrity Guard)  
代码完整性保护,只允许被Microsoft/微软商店/WHQL签名的模块才能被加载到内存.
- Extension Point Disable Policy  
如果启用了"扩展点禁用"策略，则将阻止使用某些内置的第三方扩展点。该策略阻止了以下扩展点：
AppInit DLL
Winsock分层服务提供程序(LSP)
全局Windows Hook
旧版输入法编辑器(IME)  
这里只挑了部分新增的关键的机制说明,对于Win10其他的保护机制可以参考:  
https://docs.microsoft.com/zh-cn/windows/security/threat-protection/overview-of-threat-mitigations-in-windows-10  

### 对CFG的绕过  
微软提供标准API(SetProcessValidCallTargets)将目标进程中任意地址添加到"白名单"(CFG).它的内部实现是调用tdll!NtSetInformationVirtualMemory函数附带 VmInformationClass= VmCfgCallTargetInformation参数
```cpp
HANDLE p = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_OPERATION, FALSE, process_id);
MEMORY_BASIC_INFORMATION meminfo;
VirtualQueryEx(p, target, &meminfo, sizeof(meminfo));
CFG_CALL_TARGET_INFO cfg;
cfg.Offset = ((DWORD64)target) - (DWORD64)meminfo.AllocationBase;
cfg.Flags = CFG_CALL_TARGET_VALID;
SetProcessValidCallTargets(p, meminfo.AllocationBase, meminfo.RegionSize, 1, &cfg);
```
并且有一个简单的方式去停用Win10 1803当中除CFG以外的其他所有保护机制.微软提供了一个标准API(SetProcessMitigationPolicy)再自己的进程中打开/关闭这些功能.这个函数需要再目标进程中调用,并提供三个参数:
```
HANDLE th=OpenThread(THREAD_SET_CONTEXT, FALSE, thread_id);
ntdll!NtQueueApcThread(th, SetProcessMitigationPolicy, (void*)ProcessDynamicCodePolicy, ((char*)GetModuleHandleA("ntdll")) + 0x20, sizeof(PROCESS_MITIGATION_DYNAMIC_CODE_POLICY));
```
但是这个方法在Windows 1809开始被停用.一旦保护设置（通过SetProcessMitigationPolicy），也无法取消设置 - SetProcessMitigationPolicy返回状态ERROR_ACCESS_DENIED.  
## windows注入的步骤
通常情况下,注入分3步:
- 内存分配
- 内存写操作
- 执行
有时内存分配和内存写入在技术上处于同一个步骤,使用相同的API,有时内存分配时隐式的,有时不能将内存写入与执行分开.  
通常情况下,内存分配和写入要在执行前执行多次.所以在介绍注入技术是将写技术和执行技术分开来介绍,也就是说任意一种写技术都可以与任意一种执行技术相匹配.  

### 关于内存分配的注意事项  
一般情况下,内存写入代码需要目标内存已经被申请.这可以通过两种方式进行:  
1. 注入器进程调用VirtualAllocEx(或者NtAllocateVirtualMemory)在目标进程中申请新的内存空间.这种情况下注入器可以申请可读/可写/可执行.注意,申请的可执行内存默认已经被CFG标记为有效目标.  
2. 注入器进程可以指定目标进程中的一块现有内存区覆盖,这里有几个选择:  
    1. 栈:不管是栈中使用的内存还是栈顶以外的内存都是可读可写的.写入栈需要几个注意事项:(i)TOS(栈顶)当写入栈顶以为的内存时,可能被后续的内部调用或系统调用所覆盖(ii)写入栈顶以前的内存时,这将使现有的堆栈被覆盖.  
    2. Image:一些dll的数据段包含"备用"空间,这些空洞是可读可写的,并且被初始化为0.  
    3. 堆:任何在堆上分配的数据对象,并且地址是对注入器进程已知的理论上都可以被利用.虽然有些内存空间可能由于对象被操纵或释放而被修改或回收.   
  
  
## windows注入技术
下面的说明中尽量将 a.在目标进程中准备内存空间,b.执行在目标进程中的逻辑代码这两项技术分开展示.
1. **经典的WriteProcessMemory写方法**  
  a. 确认目标内存地址已经被申请.(例如使用VirtualAllocEx)  
  b. 使用WriteProcessMemory写入数据.  
````
HANDLE h = OpenProcess(PROCESS_VM_WRITE | PROCESS_VM_OPERATION, FALSE, process_id);
LPVOID target_payload = VirtualAllocEx(h,NULL,sizeof(payload),MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
WriteProcessMemory(h,target_payload,payload,sizeof(payload),NULL);
````
2. **经典的Dll注入执行方式(执行方法)**  
a. 将恶意DLL写入磁盘,DllMain需要包含启动payload的引导代码 
b. 将dll路径写入目标进程 
c. 使用加载dll的方式执行.CreateRemoteThread(…,LoadLibraryA,DLL_path_string) (internal functions NtCreateThreadEx and RtlCreateUserThread can also be used) 
变式1:使用QueueUserAPC代替CreateRemoteThread(线程必须处于警醒状态) 
变式2:在系统dll中寻找一个以0节为的有效路径代替想目标进程内从中写入dll路径,将dll文件以找到的名字写入磁盘,然后LoadLibraryA的参数指向找到的字符串.(例如,ntdll.dll中包含字符串\ntdll\ldrsnap.c,所以把dll放在c:\\ntdll\ldrsnap.c)  
```cpp
HANDLE h = OpenProcess(PROCESS_CREATE_THREAD,FALSE,process_id);
CreateRemoteThread(h,NULL,0,(LPTHREAD_START_ROUTINE)LoadLibraryA,target_DLL_path,0,NULL);
```

3. **CreateRemoteThread执行方法** 
与上一种的不同只是thread直接执行payload
a. 使用任何一种写方法将代码数据写入到内存当中. 
b. 使用CreateRemoteThread执行代码(需要CFG有效目标)  
````
HANDLE h = OpenProcess(PROCESS_CREATE_THREAD, FALSE, process_id);
CreateRemoteThread(h, NULL, 0, (LPTHREAD_START_ROUTINE)target_execution,RCX,0,NULL);
````

4. **APC执行方式**  
线程必须处于惊醒状态.i.e. in one of 5 functions: SleepEx, WaitForSingleObjectEx, WaitForMultipleObjectsEx, SignalObjectAndWait, MsgWaitForMultipleObjectsEx (probably RealMsgWaitForMultipleObjectsEx). 
a. 将数据使用任何一种写方式写入到目标进程当中. 
b. 使用QueueUserAPC/NtQueueApcThread执行目标代码.   
``````
HANDLE h = OpenThread(THREAD_SET_CONTEXT,FALSE,thread_id);
QueueUserAPC((LPTHREAD_START_ROUTINE)target_execution,h,RCX);
//or
ntdll!NtQueueApcThread(h,(LPTHREAD_START_ROUTINE)target_execution,RCX,RDX,R8);
``````

5. **线程劫持执行方法"Suspend-Inject-Resume"** 
a. 使用任何一种写内存的方式将数据写到目标位置,例如,VirtualAllocEx(…,PAGE_EXECUTE_READWRITE)+WriteProcessMemory (not shown). 
b. 使用SetThreadContext去执行代码 (目标线程需要被挂起再恢复) – 将RIP设置为被写入的代码或者一个ROP链位置.  
变式:使用NtQueueApcThread(thread,SetThreadContext,-2,context,NULL)去代替  
但是这种方式需要在payload中去恢复劫持前的环境.  
````
HANDLE t = OpenThread(THREAD_SET_CONTEXT,FALSE,thread_id);
SuspendThread(t);
CONTEXT ctx;
ctx.ContextFlags = CONTEXT_CONTROL;
ctx.Rip = (DWORD64)target_execution;
SetThreadContext(t,&ctx);
ResumeThread(t);
````

6. **使用windows Hook函数去执行**  
a. 将恶意dll文件写入磁盘. 
b. 使用SetWindowsHookEx(…,handle to DLL, thread_id) 这将会使dll加载到进程当中(线程中必须有一个消息循环).有一个不太优雅的方式,将thread_id设为0 会注入所有的进程消息循环当中.  
`````
HMOUDLE h = LoadLibraryA(dll_path);
HOOKPROC f = (HOOKPROC)GetProcAddress(h,"GetMsgProc");
SetWindowsHookExA(WH_GETMESSAGE, f, h, thread_id);
PostThreadMessage(thread_id,WM_NULL,NULL,NULL);
`````


以上写的都是写比较通用的写和执行方式,但是由于太过经典被广泛使用导致被检测的可能性极高,之后还有更多更先进更新颖的方式在学习完后继续分享.  

