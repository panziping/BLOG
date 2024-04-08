# STM32：STM32F1启动文件STARTUP

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。
2. 参考：

   - 正点原子
   - 野火
   - ST数据手册

---

## 正文

### 启动文件

stm32启动文件(startup_stm32f10x_hd.s)主要实现了下面五种功能

1. Set the initial SP
2. Set the initial PC == Reset_Handler
3. Set the vector table entries with the exceptions ISR address
4. Configure the clock system and also configure the external SRAM mounted on STM3210E-EVAL board to be used as data memory (optional, to be enabled by user)
5. Branches to __main in the C library (which eventually calls main()).



---

1. 初始化堆栈指针

```assembly
Stack_Size      EQU     0x00000400

                AREA    STACK, NOINIT, READWRITE, ALIGN=3
Stack_Mem       SPACE   Stack_Size
__initial_sp
                                                  
; <h> Heap Configuration
;   <o>  Heap Size (in Bytes) <0x0-0xFFFFFFFF:8>
; </h>

Heap_Size       EQU     0x00000200

                AREA    HEAP, NOINIT, READWRITE, ALIGN=3
__heap_base
Heap_Mem        SPACE   Heap_Size
__heap_limit
```

2. 初始化PC指针

```assembly
__Vectors       DCD     __initial_sp               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler
```

3. 初始化中断向量表

```assembly
; Vector Table Mapped to Address 0 at Reset
                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors
                EXPORT  __Vectors_End
                EXPORT  __Vectors_Size

__Vectors       DCD     __initial_sp               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler
				...
				...
                DCD     PendSV_Handler             ; PendSV Handler
                DCD     SysTick_Handler            ; SysTick Handler

                ; External Interrupts
                DCD     WWDG_IRQHandler            ; Window Watchdog
                DCD     PVD_IRQHandler             ; PVD through EXTI Line detect
                ...
				...
                DCD     DMA2_Channel3_IRQHandler   ; DMA2 Channel3
                DCD     DMA2_Channel4_5_IRQHandler ; DMA2 Channel4 & Channel5
__Vectors_End
```

4. 配置系统时钟

```c
void SystemInit (void)
```

5. 引导程序转至main()函数

```assembly
; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
                IMPORT  __main
                IMPORT  SystemInit
                LDR     R0, =SystemInit
                BLX     R0               
                LDR     R0, =__main
                BX      R0
                ENDP
```



### 堆和栈

**Stack-栈**

在启动文件中，默认栈的大小为0X00000400（1KB）。
栈的作用是用于局部变量，函数调用，函数形参等的开销，栈的大小不能超过内部 RAM 的大小（STM32F103VET6内部RAM大小为64KB）。

**示例：函数内部定义局部变量**

```c
uint8_t receive_data[4096];
```

如果**Stack的大小是1KB**，程序就会进入**HardFault_Handler**()硬件错误中断。



**Heap-堆**

在启动文件中，默认堆的大小为 0X00000200（512B）。
堆主要用来动态内存的分配，像 malloc() 函数申请的内存就在堆上面。这个在 STM32 里面用的比较少。

---



STM32内部有两块存储区，FLASH和RAM。其中FLASH用来一般性数据（程序），RAM用来存储临时性数据（临时变量，局部变量）。

那么FLASH可不可以存储变量数据呢？答案是可以。在变量的前面加上const进行修饰。



## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/ARM](https://blog.csdn.net/zipingpan/category_12627684.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**
