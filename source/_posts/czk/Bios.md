---
title: BIOS逆向分析
author: czk
categories:
- BIOS
- BIOS逆向分析
tags: 
- BIOS
date: 2021-08-20 
---
    
    本篇主要是静态的BIOS代码逆向分析，作为后期的储备知识；

<!-- more -->

# 常见的主板BIOS

目前常见的主板BIOS有Award BIOS、AMI BIOS和Phoenix BIOS三种类型。

## 1.Award BIOS

​	Award BIOS 是由 Award Software 公司开发的 BIOS 产品。在Phoenix公司与Award公司合并前，Award BIOS便被大多数台式机主板采用。两家公司合并后，Award BIOS也被称为Phoenix-Award BIOS。

​	Award BIOS设置界面图：

![image-20210806140349015](/pic/czk\Bios\awardbios_setting)

## 2.AMI BIOS

​	AMI BIOS是AMI公司生产的BIOS。该公司成立于1985年。其生 产的AMI BIOS占据了早期台式电脑的市场，但后来逐渐沉寂，被Award BIOS 将市场夺走。

AMI BIOS设置主界面:

​	![image-20210806140933479](/pic/czk\Bios\AMI_BIOS_setting)

## 3.Phoenix BIOS

​	 Phoenix BIOS 是 Phoenix 公司的产品。从性能和稳定性看，Phoenix BIOS要优于Award和AMI，因此被广泛应用于服务器系统、品牌机和笔记本电 脑上。

Phoenix BIOS设置主界面：

​	![image-20210806141207380](/pic/czk\Bios\Phoenixbios_setting)



详细分析之前先了解一下UEFI：

# UEFI概念

BIOS是英文"Basic Input Output System"的缩略词，直译过来后中文名称就是"基本输入输出系统"。

UEFI全称Unified Extensible Firmware Interface，即“统一的可扩展固件接口”，是一种详细描述全新类型接口的标准，是适用于电脑的标准固件接口，旨在代替BIOS（基本输入/输出系统），UEFI旨在提高软件互操作性和解决BIOS的局限性。

所有的电脑都会有一个BIOS，用于加载电脑最基本的程序代码，担负着初始化硬件，检测硬件功能以及引导操作系统的任务。而UEFI就是与BIOS相对的概念，这种接口用于操作系统自动从预启动的操作环境，加载到一种操作系统上，从而达到开机程序化繁为简节省时间的目的。传统BIOS技术正在逐步被UEFI取而代之，在最近新出厂的电脑中，很多已经使用UEFI，使用UEFI模式安装操作系统是趋势所在。

本篇将会分析新版BIOS代码 UEFI；



# UEFI系统的启动过程

UEFI系统从加电到关机可分为以下七个阶段：

1. **SEC(安全验证)**

2. **PEI(EFI前期初始化)**

3. **DXE（驱动执行环境)**

4. **BDS(BS?)(启动设备选择)**

5. **TSL(操作系统加载前期)**

6. **RT(Run Time)**

7. **AL(系统灾难恢复期)**

其中，前三个阶段为UEFI初始化加载阶段，DXE阶段结束后UEFI环境已经准备完毕。

BDS和TSL是UEFI运行OS Loader的阶段。

OS Loader调用ExitBootServices()服务后系统启动进入RT阶段，操作系统加载后期和操作系统运行期均属于RT阶段。

硬件系统或操作系统出现严重错误不能继续正常运行时，固件会尝试进行修复，此时系统进入AL期。但是PI规范和UEFI规范都没有规定AL期的行为。AL期的系统行为由系统供应商自行定义。



每个阶段具体的功能和流程：

### 1. SEC（Security Phase，安全阶段）阶段

[^引自网文https://blog.csdn.net/qq_39180804/article/details/106089511]: 

SEC（Security Phase）阶段是平台初始化的第一个阶段，计算机系统加电后首先进入这个阶段。

 	1.  SEC阶段的功能：UEFI系统开机或重启后首先进入SEC阶段，SEC阶段系统执行以下四种任务：
       	1.  接收并处理系统启动和重启信号，系统加电信号、系统重启信号、系统运行过程中的异常信号。
       	2.  初始化临时存储区域：系统运行在SEC阶段时，仅CPU和CPU内部资源被初始化，而各种外部设备和内存都没有被初始化。因此系统需要一部分临时内存用于代码和数据的存储，一般称为临时RAM，临时RAM只能位于CPU内部（CPU和CPU内部的资源最先被初始化）。最常用的临时RAM是Cache，通过将Cache设置为no-eviction模式，来把其当成内存使用（此时读取命中则返回Cache中的数据，读取缺失并不会向主存发出缺失事件；写命中时写入Cache，写缺失时也不会向主存发出缺失事件），这种技术称为CAR（Cache As RAM）。
       	3.  SEC阶段是可信系统的根：作为系统启动的第一部分，只有SEC能被系统信任，以后的各个阶段才有被信任的基础。因此，大部分情况下SEC再转交控制权给PEI前可以验证PEI是否可信。
       	4.  传递系统参数给下一阶段：SEC阶段的一切工作都是为PEI阶段做准备的，最重要把系统的控制权转交给PEI，并将SEC阶段的运行信息汇报给PEI。SEC通过将以下信息作为参数传递给PEI的入口程序来向PEI汇报信息：
             	1.  系统当前状态，PEI根据状态值判断系统当前的健康情况。
             	2.  可启动固件（Boot Firmware Volume）的地址和大小，PEI据此判断可用硬件。
             	3.  临时RAM区域的地址和大小。
             	4.  栈的地址和大小。

2. SEC阶段执行流程：

   根据临时RAM是否初始化为界限，SEC阶段分为两部分：临时RAM初始化前称为Reset Vector阶段；临时RAM初始化后调用SEC入口函数从而进入SEC功能区。其流程如下图：

   ![image-20210809144237974](/pic/czk\Bios\powertopei)

   ​	其中Reset Vector的执行流程如下：

   1. 进入固件接口。
   2. 从实模式转换到32位平坦模式。
   3. 定位固件中的BFV（Boot Firmware Volume）。
   4. 定位BFV中的SEC影响。
   5. 如果是64位系统，则从32位模式转换至64位模式。
   6. 调用SEC入口函数。

   ​	

# BIOS第一条指令

​	电脑上电后给CPU RESET引脚发送信号后，CPU从固定内存地址读取指令并运行, 这个固定地址是映射到BIOS芯片的, 因此 RESET后，CPU是直接从BIOS的ROM中读取指令并运行的；80386及以后的x86处理器是从地址 FFFFFFF0开始运行的;  这个在inter开发文档里面有提及在9.1.4 First Instruction Executed中

​		![image-20210806143115024](/pic/czk\Bios\914firstinstruction)

在实模式下,FFFFFFF0超出了1M的寻址范围，上面有给出怎么样访问的；



# BIOS代码Dump

RW - Read & Write utility 是一款硬件读写程序，几乎可以访问所有计算机硬件，包括PCI、SMBIOS、IO、Memory、Bios，这次用它读取主板BIOS物理内存；

先看一下FFFFFFF0h地址处的数据：

![image-20210806152743457](/pic/czk\Bios\FFFFFFF0)

开始两个90(nop)指令然后一个e9 jmp跳转；

结合第一条指令的计算方式决定从FFFF0000处开始dump，一直dump到FFFF-FFFF处, 保存命名为M00000000FFFF0000.bin文件,dump的区间是猜测的，多尝试一下；

![image-20210806153116235](/pic/czk\Bios\FFFF0000dump)



# IDA分析dump出的文件

​	用IDA打开dump出来的bin文件，bin文件直接被识别为BIOS Image类型的文件；

​	注意BIOS开头的代码是运行在16位实模式下的； 

​	开始直接jmp走，值得一提的是此时处于Reset Vector阶段；

![image-20210806154755057](/pic/czk\Bios\startjmp)

![image-20210806154909156](/pic/czk\Bios\FDD0)

```
Reset Vector阶段 关键性逻辑注解
这段代码的主要功能就是判断新式UEFI模式启动还是legacy方式；以及开机自检和转入16位保护模式最后进入32位保护模式；

fninit                                ; clear any pending Floating point exceptions
  ;
  ; Store the BIST value in mm0
  ;
  movd    mm0, eax

  ;
  ; Save time-stamp counter value
  ; rdtsc load 64bit time-stamp counter to EDX:EAX
  ;
  rdtsc
  movd    mm5, edx
  movd    mm6, eax

  xor     eax, eax
  mov     es, ax
  assume es:nothing
  mov     ax, cs
  mov     ds, ax
  assume ds:BIOS16
  mov     ax, 0F000h
  mov     es, ax
  assume es:BIOS16
  mov     al, large byte ptr es:start
  cmp     al, 0EAh        ; 判断入口点是否是EA指令(远跳)  现在BIOS一般采用UEFI模式启动
                          ; 判断EA 应该是检测传统模式(Legacy)
  jz      short loc_FFE04
  mov     dx, 0CF9h
  in      al, dx          ; 从端口0CF9读取一个字节到AL  3个有效位 控制了是Soft/Hard reset模式
  cmp     al, 4           ; Soft Reset
  jnz     short loc_FFE13 ; 开机自检
  mov     dx, 0CF9h
  mov     al, 6           ; Hard Reset  具体参见http://www.ufoit.com/thread-107-1-1.html
  out     dx, al 

  ;
  ; Load the GDT table in GdtDesc
  ;
  mov     esi,  GdtDesc
  DB      66h
  lgdt    [cs:si]

  ;
  ; Transition to 16 bit protected mode
  ;
  mov     eax, cr0                   ; Get control register 0
  or      eax, 00000003h             ; Set PE bit (bit #0) & MP bit (bit #1)
  mov     cr0, eax                   ; Activate protected mode

  mov     eax, cr4                   ; Get control register 4
  or      eax, 00000600h             ; Set OSFXSR bit (bit #9) & OSXMMEXCPT bit (bit #10)
  mov     cr4, eax

  ;
  ; Now we're in 16 bit protected mode
  ; Set up the selectors for 32 bit protected mode entry
  ;
  mov     ax, SYS_DATA_SEL
  mov     ds, ax
  mov     es, ax
  mov     fs, ax
  mov     gs, ax
  mov     ss, ax

  ;
  ; Transition to Flat 32 bit protected mode
  ; The jump to a far pointer causes the transition to 32 bit mode
  ;
  mov esi, 0FFFFFF96h
  jmp   dword far  [cs:si]

```

上面最后的jmp   dword far  [cs:si]是从BIOS_F:FF96取四字节作为目的地址（BIOS32:0000FE70）；

注意上面已经实现从16位到32位代码的转换，所以IDA中分析时也要跟着转换；但是IDA是没有转换的，需要自己手动配置一下section区块信息；具体的配置方法就不详细说明了，展示一些最后的结果；

![image-20210809114217793](/pic/czk\Bios\fe70section)

AD列标识了代码块的位数；



紧接上面进入FE70，主要关注圈住的0000FE98和FEB2两处就行，其他逻辑就是一个单纯的跳转；

![image-20210809160832880](/pic/czk\Bios\fe70)



## 初始化临时存储区域

第一个圈处(FE98)的功能主要是初始化临时存储区域见下面详细分析：

[^以下代码引自：https://blog.csdn.net/robinsongsog/article/details/9964079]: 
[^其中涉及的MTRR见：https://blog.csdn.net/u011137631/article/details/96600130]: 

```
;-----------------------------------------------------------------------------
;
;  Section:     SecCarInit
;
;  Description: This function initializes the Cache for Data, Stack, and Code
;               as specified in the  BIOS Writer's Guide.
;
;-----------------------------------------------------------------------------
SecCarInit    PROC    NEAR    PUBLIC
  ;
  ; Detect Boot Guard Boot
  ;
DetectBootGuard:
  mov     ecx, MSR_BOOT_GUARD_SACM_INFO    ;
  rdmsr
  and     eax, 01h
  jnz     BootGuardNemSetup
SkipDetectBootGuard:

  ;
  ;  Enable cache for use as stack and for caching code
  ;  Ensure that the system is in flat 32 bit protected mode.
  ;
  ;  Ensure that only one logical processor in the system is the BSP.
  ;  (Required step for clustered systems).
  ;
  ;  Ensure all APs are in the Wait for SIPI state.
  ;  This includes all other logical processors in the same physical processor
  ;  as the BSP and all logical processors in other physical processors.
  ;  If any APs are awake, the BIOS must put them back into the Wait for
  ;  SIPI state by issuing a broadcast INIT IPI to all excluding self.
  ;
  mov     edi, APIC_ICR_LO               ; 0FEE00300h - Send INIT IPI to all excluding self
  mov     eax, ORALLBUTSELF + ORSELFINIT ; 0000C4500h - Broadcast INIT IPI
  mov     [edi], eax

@@:
  mov     eax, [edi]
  bt      eax, 12                       ; Check if send is in progress
  jc      @B                            ; Loop until idle

  ;   Ensure that all variable-range MTRR valid flags are clear and
  ;   IA32_MTRR_DEF_TYPE MSR E flag is clear.  Note: This is the default state
  ;   after hardware reset.
  ;
  ;   Initialize all fixed-range and variable-range MTRR register fields to 0.
  ;
   mov   ecx, IA32_MTRR_CAP             ; get variable MTRR support
   rdmsr
   movzx ebx, al                        ; EBX = number of variable MTRR pairs
   shl   ebx, 2                         ; *4 for Base/Mask pair and WORD size
   add   ebx, MtrrCountFixed * 2        ; EBX = size of  Fixed and Variable MTRRs

   xor   eax, eax                       ; Clear the low dword to write
   xor   edx, edx                       ; Clear the high dword to write

InitMtrrLoop:
   add   ebx, -2
   movzx ecx, WORD PTR cs:MtrrInitTable[ebx]  ; ecx <- address of mtrr to zero
   wrmsr
   jnz   InitMtrrLoop                   ; loop through the whole table

  ;
  ;   Configure the default memory type to un-cacheable (UC) in the
  ;   IA32_MTRR_DEF_TYPE MSR.
  ;
  mov     ecx, MTRR_DEF_TYPE            ; Load the MTRR default type index
  rdmsr
  and     eax, NOT (00000CFFh)          ; Clear the enable bits and def type UC.
  wrmsr

  ; Configure MTRR_PHYS_MASK_HIGH for proper addressing above 4GB
  ; based on the physical address size supported for this processor
  ; This is based on read from CPUID EAX = 080000008h, EAX bits [7:0]
  ;
  ; Examples:
  ;  MTRR_PHYS_MASK_HIGH = 00000000Fh  For 36 bit addressing
  ;  MTRR_PHYS_MASK_HIGH = 0000000FFh  For 40 bit addressing
  ;
  mov   eax, 80000008h                  ; Address sizes leaf
  cpuid
  sub   al, 32
  movzx eax, al
  xor   esi, esi
  bts   esi, eax
  dec   esi                             ; esi <- MTRR_PHYS_MASK_HIGH

  ;
  ;   Configure the DataStack region as write-back (WB) cacheable memory type
  ;   using the variable range MTRRs.
  ;
  ;
  ; Set the base address of the DataStack cache range
  ;
  mov     eax, PcdGet32 (PcdTemporaryRamBase)
  or      eax, MTRR_MEMORY_TYPE_WB
                                        ; Load the write-back cache value
  xor     edx, edx                      ; clear upper dword
  mov     ecx, MTRR_PHYS_BASE_0         ; Load the MTRR index
  wrmsr                                 ; the value in MTRR_PHYS_BASE_0

  ;
  ; Set the mask for the DataStack cache range
  ; Compute MTRR mask value:  Mask = NOT (Size - 1)
  ;
  mov  eax, PcdGet32 (PcdTemporaryRamSize)
  dec  eax
  not  eax
  or   eax, MTRR_PHYS_MASK_VALID
                                        ; turn on the Valid flag
  mov  edx, esi                         ; edx <- MTRR_PHYS_MASK_HIGH
  mov  ecx, MTRR_PHYS_MASK_0            ; For proper addressing above 4GB
  wrmsr                                 ; the value in MTRR_PHYS_BASE_0

  ;
  ;   Configure the BIOS code region as write-protected (WP) cacheable
  ;   memory type using a single variable range MTRR.
  ;
  ;   Ensure region to cache meets MTRR requirements for
  ;   size and alignment.
  ;

  ;
  ; Save MM5 into ESP before program MTRR, because program MTRR will use MM5 as the local variable.
  ; And, ESP is not initialized before CAR is enabled. So, it is safe ot use ESP here.
  ;
  movd esp, mm5

  ;
  ; Get total size of cache from PCD if it need fix value
  ;
  mov     eax, PcdGet32 (PcdNemCodeCacheSize)
  ;
  ; Calculate NEM size
  ; Determine LLC size of the code region and data region combined must not exceed the size
  ; of the (Last Level Cache - 0.5MB).
  ;
  ; Determine Cache Parameter by CPUID Function 04h
  ;
  xor     edi, edi

Find_LLC_parameter:
  mov     ecx, edi
  mov     eax, 4
  cpuid
  inc     edi
  and     al, 0E0h                       ; EAX[7:5] = Cache Level
  cmp     al, 60h                        ; Check to see if it is LLC
  jnz     Find_LLC_parameter
  ;
  ; Got L3 parameters
  ;
  ; This Cache Size in Bytes = (Ways + 1) * (Partitions + 1) * (Line_Size + 1) * (Sets + 1)
  ;  = (EBX[31:22] + 1) * (EBX[21:12] + 1) * (EBX[11:0] + 1) * (ECX + 1)
  ;
  mov     eax, ecx
  inc     eax
  mov     edi, ebx
  shr     ebx, 22
  inc     ebx
  mul     ebx
  mov     ebx, edi
  and     ebx, NOT 0FFC00FFFh
  shr     ebx, 12
  inc     ebx
  mul     ebx
  mov     ebx, edi
  and     ebx, 0FFFh
  inc     ebx
  mul     ebx
  ;
  ; Maximum NEM size <= (Last Level Cache - 0.5MB)
  ;
  sub     eax, 512*1024
Got_NEM_size:
  ;
  ; Code cache size = Total NEM size - DataStack size
  ;
  sub     eax, PcdGet32 (PcdTemporaryRamSize)
  ;
  ; Set the base address of the CodeRegion cache range from PCD
  ; PcdNemCodeCacheBase is set to the offset to flash base,
  ; so add PcdFlashAreaBaseAddress to get the real code base address.
  ;
  mov     edi, PcdGet32 (PcdNemCodeCacheBase)
  add     edi, PcdGet32 (PcdFlashAreaBaseAddress)

  ;
  ; Round up to page size
  ;
  mov     ecx, eax                      ; Save
  and     ecx, 0FFFF0000h               ; Number of pages in 64K
  and     eax, 0FFFFh                   ; Number of "less-than-page" bytes
  jz      Rounded
  mov     eax, 10000h                   ; Add the whole page size

Rounded:
  add     eax, ecx                      ; eax - rounded up code cache size

  ;
  ; Define "local" vars for this routine
  ; @todo as these registers are overlapping with others
  ; Note that mm0 is used to store BIST result for BSP,
  ; mm1 is used to store the number of processor and BSP APIC ID,
  ; mm6 is used to save time-stamp counter value.
  ;
  CODE_SIZE_TO_CACHE    TEXTEQU  <mm3>
  CODE_BASE_TO_CACHE    TEXTEQU  <mm4>
  NEXT_MTRR_INDEX       TEXTEQU  <mm5>
  NEXT_MTRR_SIZE        TEXTEQU  <mm2>
  ;
  ; Initialize "locals"
  ;
  sub     ecx, ecx
  movd    NEXT_MTRR_INDEX, ecx          ; Count from 0 but start from MTRR_PHYS_BASE_1

  ;
  ; Save remaining size to cache
  ;
  movd    CODE_SIZE_TO_CACHE, eax       ; Size of code cache region that must be cached
  mov     edi, 0xFFFFFFFF
  sub     edi, eax
  inc     edi
  test    edi, 0xFFFF
  jz      @f
  add     edi, 0x10000
  and     edi, 0xFFFF0000
@@:
  movd    CODE_BASE_TO_CACHE, edi       ; Base code cache address

NextMtrr:
  ;
  ; Get remaining size to cache
  ;
  movd    eax, CODE_SIZE_TO_CACHE
  and     eax, eax
  jz      CodeRegionMtrrdone            ; If no left size - we are done
  ;
  ; Determine next size to cache.
  ; We start from bottom up. Use the following algorythm:
  ; 1. Get our own alignment. Max size we can cache equals to our alignment
  ; 2. Determine what is bigger - alignment or remaining size to cache.
  ;    If aligment is bigger - cache it.
  ;      Adjust remaing size to cache and base address
  ;      Loop to 1.
  ;    If remaining size to cache is bigger
  ;      Determine the biggest 2^N part of it and cache it.
  ;      Adjust remaing size to cache and base address
  ;      Loop to 1.
  ; 3. End when there is no left size to cache or no left MTRRs
  ;
  movd    edi, CODE_BASE_TO_CACHE
  bsf     ecx, edi                      ; Get index of lowest bit set in base address
  ;
  ; Convert index into size to be cached by next MTRR
  ;
  mov     edx, 1h
  shl     edx, cl                       ; Alignment is in edx
  cmp     edx, eax                      ; What is bigger, alignment or remaining size?
  jbe     gotSize                       ; JIf aligment is less
  ;
  ; Remaining size is bigger. Get the biggest part of it, 2^N in size
  ;
  bsr     ecx, eax                      ; Get index of highest set bit
  ;
  ; Convert index into size to be cached by next MTRR
  ;
  mov     edx, 1
  shl     edx, cl                       ; Size to cache

GotSize:
  mov     eax, edx
  movd    NEXT_MTRR_SIZE, eax           ; Save

  ;
  ; Compute MTRR mask value:  Mask = NOT (Size - 1)
  ;
  dec     eax                           ; eax - size to cache less one byte
  not     eax                           ; eax contains low 32 bits of mask
  or      eax, MTRR_PHYS_MASK_VALID     ; Set valid bit

  ;
  ; Program mask register
  ;
  mov     ecx, MTRR_PHYS_MASK_1         ; setup variable mtrr
  movd    ebx, NEXT_MTRR_INDEX
  add     ecx, ebx

  mov     edx, esi                      ; edx <- MTRR_PHYS_MASK_HIGH
  wrmsr
  ;
  ; Program base register
  ;
  sub     edx, edx
  mov     ecx, MTRR_PHYS_BASE_1         ; setup variable mtrr
  add     ecx, ebx                      ; ebx is still NEXT_MTRR_INDEX

  movd    eax, CODE_BASE_TO_CACHE
  or      eax, MTRR_MEMORY_TYPE_WP      ; set type to write protect
  wrmsr
  ;
  ; Advance and loop
  ; Reduce remaining size to cache
  ;
  movd    ebx, CODE_SIZE_TO_CACHE
  movd    eax, NEXT_MTRR_SIZE
  sub     ebx, eax
  movd    CODE_SIZE_TO_CACHE, ebx

  ;
  ; Increment MTRR index
  ;
  movd    ebx, NEXT_MTRR_INDEX
  add     ebx, 2
  movd    NEXT_MTRR_INDEX, ebx
  ;
  ; Increment base address to cache
  ;
  movd    ebx, CODE_BASE_TO_CACHE
  movd    eax, NEXT_MTRR_SIZE
  add     ebx, eax
  ;
  ; if carry happens, means NEM base + size over 4G
  ;
  jc      CodeRegionMtrrdone
  movd    CODE_BASE_TO_CACHE, ebx

  jmp     NextMtrr

CodeRegionMtrrdone:
  ;
  ; Enable the MTRRs by setting the IA32_MTRR_DEF_TYPE MSR E flag.
  ;
  mov     ecx, MTRR_DEF_TYPE            ; Load the MTRR default type index
  rdmsr
  or      eax, MTRR_DEF_TYPE_E          ; Enable variable range MTRRs
  wrmsr

  ;
  ; Enable the logical processor's (BSP) cache: execute INVD and set
  ; CR0.CD = 0, CR0.NW = 0.
  ;
  mov     eax, cr0
  and     eax, NOT (CR0_CACHE_DISABLE + CR0_NO_WRITE)
  invd
  mov     cr0, eax
  ;
  ; Enable No-Eviction Mode Setup State by setting
  ; NO_EVICT_MODE  MSR 2E0h bit [0] = '1'.
  ;
  mov     ecx, NO_EVICT_MODE
  rdmsr
  or      eax, 1
  wrmsr

  ;
  ; Restore MM5 from ESP after program MTRR
  ;
  movd mm5, esp

  ;
  ; Restore MM4 which is Patch Pointer.
  ; Current implementation it's the same with the PcdNemCodeCacheBase + PcdFlashAreaBaseAddress
  ;
  mov     edi, PcdGet32 (PcdNemCodeCacheBase)
  add     edi, PcdGet32 (PcdFlashAreaBaseAddress)
  movd    mm4, edi

  ;
  ; One location in each 64-byte cache line of the DataStack region
  ; must be written to set all cache values to the modified state.
  ;
  mov     edi, PcdGet32 (PcdTemporaryRamBase)
  mov     ecx, PcdGet32 (PcdTemporaryRamSize)
  shr     ecx, 6
  mov     eax, CACHE_INIT_VALUE
@@:
  mov  [edi], eax
  sfence
  add  edi, 64
  loopd  @b

  ;
  ; Enable No-Eviction Mode Run State by setting
  ; NO_EVICT_MODE MSR 2E0h bit [1] = '1'.
  ;
  mov     ecx, NO_EVICT_MODE
  rdmsr
  or      eax, 2
  wrmsr

  jmp     FinishedCacheConfig

  ;
  ; Jump to here when Boot Guard boot and NEM is initialized by Boot Guard ACM
  ;
BootGuardNemSetup:
  ;
  ; Finished with cache configuration
  ;
  ; Configure MTRR_PHYS_MASK_HIGH for proper addressing above 4GB
  ; based on the physical address size supported for this processor
  ; This is based on read from CPUID EAX = 080000008h, EAX bits [7:0]
  ;
  ; Examples:
  ;  MTRR_PHYS_MASK_HIGH = 00000000Fh  For 36 bit addressing
  ;  MTRR_PHYS_MASK_HIGH = 0000000FFh  For 40 bit addressing
  ;
  mov   eax, 80000008h                  ; Address sizes leaf
  cpuid
  sub   al, 32
  movzx eax, al
  xor   esi, esi
  bts   esi, eax
  dec   esi                             ; esi <- MTRR_PHYS_MASK_HIGH

  ;
  ; Configure the DataStack region as write-back (WB) cacheable memory type
  ; using the variable range MTRRs.
  ;
  ;
  ; Find available MTRR
  ;
  CALL_EBP     FindFreeMtrr

  ;
  ; Set the base address of the DataStack cache range
  ;
  mov     eax, PcdGet32 (PcdTemporaryRamBase)
  or      eax, MTRR_MEMORY_TYPE_WB
                                        ; Load the write-back cache value
  xor     edx, edx                      ; clear upper dword
  wrmsr                                 ; the value in MTRR_PHYS_BASE_0

  ;
  ; Set the mask for the DataStack cache range
  ; Compute MTRR mask value:  Mask = NOT (Size - 1)
  ;
  mov  eax, PcdGet32 (PcdTemporaryRamSize)
  dec  eax
  not  eax
  or   eax, MTRR_PHYS_MASK_VALID
                                        ; turn on the Valid flag
  mov  edx, esi                         ; edx <- MTRR_PHYS_MASK_HIGH
  inc  ecx
  wrmsr                                 ; the value in MTRR_PHYS_BASE_0

  ;
  ; Program the variable MTRR's MASK register for WDB
  ; (Write Data Buffer, used in MRC, must be WC type)
  ;

  ;
  ; Find available MTRR
  ;
  CALL_EBP     FindFreeMtrr

FoundAvailableMtrr:
  ;
  ; Program the variable MTRR's BASE register for WDB
  ;
  xor     edx, edx
  mov     eax, WDB_REGION_BASE_ADDRESS OR MTRR_MEMORY_TYPE_WC
  wrmsr

  inc     ecx
  mov     edx, esi                                          ; edx <- MTRR_PHYS_MASK_HIGH
  mov     eax, WDB_REGION_SIZE_MASK OR MTRR_PHYS_MASK_VALID ; turn on the Valid flag
  wrmsr

  ;
  ; One location in each 64-byte cache line of the DataStack region
  ; must be written to set all cache values to the modified state.
  ;
  mov     edi, PcdGet32 (PcdTemporaryRamBase)
  mov     ecx, PcdGet32 (PcdTemporaryRamSize)
  shr     ecx, 6
  mov     eax, CACHE_INIT_VALUE
@@:
  mov  [edi], eax
  sfence
  add  edi, 64
  loopd  @b

  ;
  ; Finished with cache configuration
  ;
FinishedCacheConfig:

  ;
  ; Optionally Test the Region
  ;

  ;
  ; Test area by writing and reading
  ;
  cld
  mov     edi, PcdGet32 (PcdTemporaryRamBase)
  mov     ecx, PcdGet32 (PcdTemporaryRamSize)
  shr     ecx, 2
  mov     eax, CACHE_TEST_VALUE
TestDataStackArea:
  stosd
  cmp     eax, DWORD PTR [edi-4]
  jnz     DataStackTestFail
  loop    TestDataStackArea
  jmp     DataStackTestPass

  ;
  ; Cache test failed
  ;
DataStackTestFail:
  STATUS_CODE (0D0h)
  jmp     $

  ;
  ; Configuration test failed
  ;
ConfigurationTestFailed:
  STATUS_CODE (0D1h)
  jmp     $

DataStackTestPass:

  ;
  ; At this point you may continue normal execution.  Typically this would include
  ; reserving stack, initializing the stack pointer, etc.
  ;

  ;
  ; Temporary Set stack top pointer for C code usage.
  ;
  mov     esp, PcdGet32 (PcdTemporaryRamBase)
  add     esp, PcdGet32 (PcdTemporaryRamSize)
  ;
  ; program resource decoding
  ;
  pushad
  call    EarlyCycleDecoding
  popad
  pushad
  call    SerialIoUartConfiguration
  popad

  ;
  ; After memory initialization is complete, please follow the algorithm in the BIOS
  ; Writer's Guide to properly transition to a normal system configuration.
  ; The algorithm covers the required sequence to properly exit this mode.
  ;
  xor    eax, eax

SecCarInitExit:

  RET_ESI

SecCarInit    ENDP

```

## CallPeiCoreEntryPoint

第二个圈处(FEB2) 见： tianocore\edk2-platforms\Platform\BroxtonPlatformPkg\Common\Library\PlatformSecLib\Ia32\SecEntry.asm

TianoCore是Intel的UEFI接口的开源项目；

```
CallPeiCoreEntryPoint   PROC    NEAR    PRIVATE

  ;
  ; Set stack top pointer
  ;
  mov     esp, (HobStruc PTR ds:[ebp]).StackHeapBase;
  add     esp, (HobStruc PTR ds:[ebp]).StackHeapSize

  ;
  ; Push CPU count to stack first, then AP's (if there is one)
  ; BIST status, and then BSP's
  ;

  ;
  ; Here work around for BIST
  ;
  ; Get number of BSPs
  mov     ch, 01 ; for client we have only one BSP
  movzx   ecx, ch

  ; Save number of BSPs
  push    ecx

GetSBSPBist:
  ; Save SBSP BIST
  movd    eax, mm0
  push    eax

  ; Save SBSP APIC ID
  movd    eax, mm1
  shr     eax, BSPApicIDSaveStart       ; Resume APIC ID
  push    eax


TransferToSecStartup:

  ; Switch to "C" code
  STATUS_CODE (0Ch)
  ;jmp $

  ; ECPoverride: SecStartup entry point needs 4 parameters

  ;
  ; Pass entry point of the PEI core
  ;
  mov     edi, PEI_CORE_ENTRY_BASE      ; 0FFFFFFE0h
  push    DWORD PTR ds:[edi]

  ;
  ; Pass BFV into the PEI Core
  ;
  push (HobStruc PTR ds:[ebp]).IBBBase        ; 0FFFFFFFCh

  ;
  ; Pass stack size into the PEI Core
  ;
  Push (HobStruc PTR ds:[ebp]).StackHeapBase   ;calc TempRamBase

  push (HobStruc PTR ds:[ebp]).StackHeapSize   ;calc TempRamBase

  ;
  ; Pass Control into the PEI Core
  ;
  call SecStartup
CallPeiCoreEntryPoint   ENDP
```

上面代码块末尾处的  SecStartup 有对应的c代码 在 tianocore\edk2\IntelFspWrapperPkg\FspInitPei\SecMain.c 中；

注意一下调用这个函数入栈的第三个参数(HobStruc PTR ds:[ebp]).IBBBase（0FFFFFFFCh），下面有用到，这个是BootFirmwareVolume ；

## SecStartup

在逆向汇编中有些代码直接被编译器优化了没有下面C代码看着直观，直接贴c代码了；

```
/**

  Entry point to the C language phase of SEC. After the SEC assembly
  code has initialized some temporary memory and set up the stack,
  the control is transferred to this function.

  @param[in] SizeOfRam           Size of the temporary memory available for use.
  @param[in] TempRamBase         Base address of tempory ram
  @param[in] BootFirmwareVolume  Base address of the Boot Firmware Volume.
**/
VOID
EFIAPI
SecStartup (
  IN UINT32                   SizeOfRam,
  IN UINT32                   TempRamBase,
  IN VOID                     *BootFirmwareVolume
  )
{
  EFI_SEC_PEI_HAND_OFF        SecCoreData;
  IA32_DESCRIPTOR             IdtDescriptor;
  SEC_IDT_TABLE               IdtTableInStack;
  UINT32                      Index;
  UINT32                      PeiStackSize;

  PeiStackSize = PcdGet32 (PcdPeiTemporaryRamStackSize);
  if (PeiStackSize == 0) {
    PeiStackSize = (SizeOfRam >> 1);
  }

  ASSERT (PeiStackSize < SizeOfRam);

  //
  // Process all libraries constructor function linked to SecCore.
  //
  ProcessLibraryConstructorList ();

  DEBUG ((DEBUG_INFO, "FspPei - SecStartup\n"));

  //
  // Initialize floating point operating environment
  // to be compliant with UEFI spec.
  //
  InitializeFloatingPointUnits ();


  // |-------------------|---->
  // |Idt Table          |
  // |-------------------|
  // |PeiService Pointer |    PeiStackSize
  // |-------------------|
  // |                   |
  // |      Stack        |
  // |-------------------|---->
  // |                   |
  // |                   |
  // |      Heap         |    PeiTemporayRamSize
  // |                   |
  // |                   |
  // |-------------------|---->  TempRamBase

  IdtTableInStack.PeiService = 0;
  for (Index = 0; Index < SEC_IDT_ENTRY_COUNT; Index ++) {
    CopyMem ((VOID*)&IdtTableInStack.IdtTable[Index], (VOID*)&mIdtEntryTemplate, sizeof (UINT64));
  }

  IdtDescriptor.Base  = (UINTN) &IdtTableInStack.IdtTable;
  IdtDescriptor.Limit = (UINT16)(sizeof (IdtTableInStack.IdtTable) - 1);

  AsmWriteIdtr (&IdtDescriptor);

  //
  // Update the base address and length of Pei temporary memory
  //
  SecCoreData.DataSize               = (UINT16) sizeof (EFI_SEC_PEI_HAND_OFF);
  SecCoreData.BootFirmwareVolumeBase = BootFirmwareVolume;
  SecCoreData.BootFirmwareVolumeSize = (UINTN)(SIZE_4GB - (UINTN) BootFirmwareVolume);
  SecCoreData.TemporaryRamBase       = (VOID*)(UINTN) TempRamBase;
  SecCoreData.TemporaryRamSize       = SizeOfRam;
  SecCoreData.PeiTemporaryRamBase    = SecCoreData.TemporaryRamBase;
  SecCoreData.PeiTemporaryRamSize    = SizeOfRam - PeiStackSize;
  SecCoreData.StackBase              = (VOID*)(UINTN)(TempRamBase + SecCoreData.PeiTemporaryRamSize);
  SecCoreData.StackSize              = PeiStackSize;

  DEBUG ((DEBUG_INFO, "BootFirmwareVolumeBase - 0x%x\n", SecCoreData.BootFirmwareVolumeBase));
  DEBUG ((DEBUG_INFO, "BootFirmwareVolumeSize - 0x%x\n", SecCoreData.BootFirmwareVolumeSize));
  DEBUG ((DEBUG_INFO, "TemporaryRamBase       - 0x%x\n", SecCoreData.TemporaryRamBase));
  DEBUG ((DEBUG_INFO, "TemporaryRamSize       - 0x%x\n", SecCoreData.TemporaryRamSize));
  DEBUG ((DEBUG_INFO, "PeiTemporaryRamBase    - 0x%x\n", SecCoreData.PeiTemporaryRamBase));
  DEBUG ((DEBUG_INFO, "PeiTemporaryRamSize    - 0x%x\n", SecCoreData.PeiTemporaryRamSize));
  DEBUG ((DEBUG_INFO, "StackBase              - 0x%x\n", SecCoreData.StackBase));
  DEBUG ((DEBUG_INFO, "StackSize              - 0x%x\n", SecCoreData.StackSize));

  //
  // Initialize Debug Agent to support source level debug in SEC/PEI phases before memory ready.
  //
  InitializeDebugAgent (DEBUG_AGENT_INIT_PREMEM_SEC, &SecCoreData, SecStartupPhase2);

}
```

## SecStartupPhase2

SecStartupPhase2 同样在tianocore\edk2\IntelFspWrapperPkg\FspInitPei\SecMain.c 中

同样便于查看直接贴C代码

```
VOID
EFIAPI
SecStartupPhase2(
  IN VOID                     *Context
  )
{
  EFI_SEC_PEI_HAND_OFF        *SecCoreData;
  EFI_PEI_PPI_DESCRIPTOR      *PpiList;
  UINT32                      Index;
  EFI_PEI_PPI_DESCRIPTOR      LocalSecPpiList[sizeof(mPeiSecMainPpi)/sizeof(mPeiSecMainPpi[0])];
  EFI_PEI_PPI_DESCRIPTOR      AllSecPpiList[FixedPcdGet32(PcdSecCoreMaxPpiSupported)];
  EFI_PEI_CORE_ENTRY_POINT    PeiCoreEntryPoint;

  SecCoreData = (EFI_SEC_PEI_HAND_OFF *) Context;
  //
  // Find Pei Core entry point. It will report SEC and Pei Core debug information if remote debug
  // is enabled.
  //
  FindAndReportEntryPoints ((EFI_FIRMWARE_VOLUME_HEADER *) SecCoreData->BootFirmwareVolumeBase, &PeiCoreEntryPoint);
  if (PeiCoreEntryPoint == NULL)
  {
    CpuDeadLoop ();
  }

  CopyMem (LocalSecPpiList, mPeiSecMainPpi, sizeof(mPeiSecMainPpi));
  PatchTopOfTemporaryRamPpi (LocalSecPpiList, (VOID *)((UINTN)SecCoreData->TemporaryRamBase + SecCoreData->TemporaryRamSize));

  //
  // Perform platform specific initialization before entering PeiCore.
  //
  PpiList = SecPlatformMain (SecCoreData);
  if (PpiList != NULL) {
    //
    // Remove the terminal flag from the terminal Ppi
    //
    CopyMem (AllSecPpiList, LocalSecPpiList, sizeof (LocalSecPpiList));
    for (Index = 0; Index < PcdGet32 (PcdSecCoreMaxPpiSupported); Index ++) {
      if ((AllSecPpiList[Index].Flags & EFI_PEI_PPI_DESCRIPTOR_TERMINATE_LIST) == EFI_PEI_PPI_DESCRIPTOR_TERMINATE_LIST) {
        break;
      }
    }
    AllSecPpiList[Index].Flags = AllSecPpiList[Index].Flags & (~EFI_PEI_PPI_DESCRIPTOR_TERMINATE_LIST);

    //
    // Append the platform additional Ppi list
    //
    Index += 1;
    while (Index < PcdGet32 (PcdSecCoreMaxPpiSupported) &&
           ((PpiList->Flags & EFI_PEI_PPI_DESCRIPTOR_TERMINATE_LIST) != EFI_PEI_PPI_DESCRIPTOR_TERMINATE_LIST)) {
      CopyMem (&AllSecPpiList[Index], PpiList, sizeof (EFI_PEI_PPI_DESCRIPTOR));
      Index++;
      PpiList++;
    }

    //
    // Check whether the total Ppis exceeds the max supported Ppi.
    //
    if (Index >= PcdGet32 (PcdSecCoreMaxPpiSupported)) {
      //
      // the total Ppi is larger than the supported Max
      // PcdSecCoreMaxPpiSupported can be enlarged to solve it.
      //
      CpuDeadLoop ();
    } else {
      //
      // Add the terminal Ppi
      //
      CopyMem (&AllSecPpiList[Index], PpiList, sizeof (EFI_PEI_PPI_DESCRIPTOR));
    }

    //
    // Set PpiList to the total Ppi
    //
    PpiList = &AllSecPpiList[0];
  } else {
    //
    // No addition Ppi, PpiList directly point to the common Ppi list.
    //
    PpiList = &LocalSecPpiList[0];
  }

  //
  // Transfer the control to the PEI core
  //
  ASSERT (PeiCoreEntryPoint != NULL);
  (*PeiCoreEntryPoint) (SecCoreData, PpiList);

  //
  // Should not come here.
  //
  return ;
}
```

注意最后调用了 （*PeiCoreEntryPoint）（）进入第二阶段--PEI；

PeiCoreEntryPoint 是 SecStartupPhase2函数开头调用FindAndReportEntryPoints获取到的，在FindAndReportEntryPoints的内部又调用了FindImageBase,此时传入了BootFirmwareVolumePtr固件地址，返回两个参数(见下面代码)；

```
FindAndReportEntryPoints (
  IN  EFI_FIRMWARE_VOLUME_HEADER       *BootFirmwareVolumePtr,
  OUT EFI_PEI_CORE_ENTRY_POINT         *PeiCoreEntryPoint
  )
{
  EFI_STATUS                       Status;
  EFI_PHYSICAL_ADDRESS             SecCoreImageBase;
  EFI_PHYSICAL_ADDRESS             PeiCoreImageBase;
  PE_COFF_LOADER_IMAGE_CONTEXT     ImageContext;

  //
  // Find SEC Core and PEI Core image base
  //
  Status = FindImageBase (BootFirmwareVolumePtr, &SecCoreImageBase, &PeiCoreImageBase);
  ASSERT_EFI_ERROR (Status);
```

逆向中也是这个过程，但是需要定位PeiCoreEntryPoint 的具体地址是多少；

## 定位PeiCoreEntryPoint

FindImageBase调用中使用的BootFirmwareVolumePtr可以向上追踪到SecStartup (  SizeOfRam,TempRamBase,*BootFirmwareVolume)的第三个参数；再向上追溯到push (HobStruc PTR ds:[ebp]).IBBBase ；IDA中是 push 【FFFFFFFCh】；FFFFFFFCh正好处于start函数地址(FFFFFFF0h)下面; IDA中查看地址处的值为 fffe0000h;

![image-20210810171452766](/pic/czk\Bios\fffe0000)

此时再次使用RW工具从fffe0000h dump数据, 长度定位0x10000h大小；实际上这块可以与上次的dump整合成一次；

解析的过程我自己写了c代码，调用FindImageBase传入新dump下来的bin文件数据，动态获取PeiCoreImageBase；

![image-20210810151722417](/pic/czk\Bios\testfindpeicore)

运行后PeiCoreImageBase相对于buffer最终偏移了0x120；

010中查看偏移0x120中正好是个PE文件；把前面的数据删除掉，保存为PeiCoreImageFFFE0120.bin，此时就获取到了PeiCoreImageBase；

![image-20210810172735231](/pic/czk\Bios\120offset)

PeiCoreEntryPoint的定位，查看c代码中就是正常的通过PE结构获取入口点；这个用IDA打开就能直接看到了,过程就不详细罗列；



好了到这里，SEC阶段就结束了，要进入PEI阶段了；



# PEI（Pre-EFI Initialization，预先EFI初始化）阶段

虽然SEC阶段对CPU和CPU内的资源进行了初始化，但是PEI阶段可用的资源依旧十分有限，该阶段对内存进行初始化，主要功能是为DXE阶段准备执行环境，将所需要传递给DXE的信息组成HOB（Hand Off Block）列表，最终将控制权转交到DXE。

UEFI具有模块化设计的特点，PEI就是一个模块。PEI Image的入口函数调用PEI模块的入口函数PEICore。

1. PEI阶段的功能：

   1. 初始化内存。
   2. 为DXE阶段准备执行环境。

2. PEI划分：

   1. PEI内核（PEI Foundation）：负责PEI基础服务和流程。

   2. PEIM（PEI Module）派遣器：找出系统中的所有PEIM，并根据PEIM之间的依赖关系按顺序执行PEIM。PEI阶段对系统的初始化主要由PEIM完成。

   每个PEIM都是一个独立的模块。通过PEIServices，PEIM可以调用PEI阶段（UEFI？）提供的系统服务。通过调用这些服务，PEIM可以访问PEI内核。PEIM之间的的通信通过PPI（PEIM-to-PEIM Interfaces）完成。

3. PEI阶段执行流程：

   1. 进入PEI入口。
   2. 根据SEC阶段传入的信息初始化PS（PEI Core Service）。

   3. 调度系统中的PEIM（PEI Module），准备HOB列表。

      具体调用的系统中的PEIM有：CPU PEIM（提供CPU相关功能，如进行Cache设置、主频设置等）；平台相关PEIM（初始化内存控制器、I/O控制器等）；内存初始化PEIM（对内存进行初始化，此时内存才可用，之前使用的CPU模拟的临时内存）。

   4. 调用PEIServices得到DEX IPL PPI的Entry服务（即DEXLoadCore）。

      注：PPI与DEX阶段的Protocol类似，每个PPI都是一个结构体，包含有函数指针和变量。每个PPI都有一个GUID。通过PEIServices的LocatePPI服务可以找到GUID对应的PPI实例。

   5. DXELoadCore服务找出并运行DXEImage的入口函数，将HOB列表传递给DXE。



PEI阶段执行流程完整描述：SEC模块找到PEI Image的入口函数 _ModuleEntryPoint（该函数位于MdePkg/Library/PeimEntryPoint/PeimEntryPoint.c）， _ModuleEntryPoint函数最终调用PEI模块的入口函数PEICore（该函数位于MdeModulePkg/Core/Pei/PeiMain/PeiMain.c），进入PEICore后，首先根据从SEC阶段出入的信息设置PEI Core Services，然后调用PEIDispatcher执行系统总的PEIM，在内存初始化完成后，系统切换栈并重新进入PEICore。重新进入PEICore后使用的不再是 临时RAM 而是真正的内存。在所有PEIM执行完成后，调用PEIServices的LocatePPI服务得到DXE IPL PPI，并调用DXE IPL PPI的Entry服务（即DEXLoadCore），找出DEX Image的入口函数，执行DXE Image函数并将HOB列表传递给DXE。

PEI阶段执行流程如下图：

​			![image-20210810181557031](/pic/czk\Bios\PEI阶段流程)





未完待续... ...

















