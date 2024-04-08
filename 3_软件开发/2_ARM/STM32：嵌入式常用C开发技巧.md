# STM32：嵌入式常用C开发技巧

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

### ifndef

````C
#ifndef __BSP_DRIVE_H
#define __BSP_DRIVE_H
```
```
#endif 
````

**ifndef**是**if not define**的简写，是预处理三种（宏定义，文件包含，条件编译）中的一种——条件编译。

在C语言中，只可以对变量和函数进行一次定义，但是可以对变量和函数进行多次声明。

如果h文件中只进行声明工作，即使不使用**#ifndef**，多个c文件包含同一个h文件也不会报错。但是**#ifdef**的作用域只在单个文件中，如果h文件中定义了全局变量,即使采用**#ifdef**，多个c文件包含同一个h文件还是会出现全局变量重定义的错误。

使用**#ifndef**可以避免下面这种错误。如果在h文件中定义了全局变量，一个c文件包含同一个h文件多次，如果不加**#ifndef**宏定义，会出现变量重复定义的错误；如果加了**#ifndef**，则不会出现这种错误。

### 全局define

在使用MDK5对STM32F1进行开发时，需要设置两个宏定义：**USE_STDPERIPH_DRIVER**，**STM32F10X_HD**。**这两个宏定义全在stm32f10x.h中使用**，在Options中设置。

- USE_STDPERIPH_DRIVER

  ```c
  #ifdef USE_STDPERIPH_DRIVER
    #include "stm32f10x_conf.h"	
  #endif
  ```

  stm32f10x_conf.h:包含STM32F1外设的头文件。

- STM32F10X_HD

  由芯片FLASH大小决定。

  |         启动文件          |                             区别                             |
  | :-----------------------: | :----------------------------------------------------------: |
  |  startup_stm32f10x_ld.s   |        ld:low density，小容量，FLASH容量在16~32K之间         |
  |  startup_stm32f10x_md.s   |      md:medium density，小容量，FLASH容量在64~128K之间       |
  |  startup_stm32f10x_hd.s   |       hd:high density，小容量，FLASH容量在256~512K之间       |
  |  startup_stm32f10x_xl.s   |     xl: extra large，超大容量，FLASH容量在512~1024K之间      |
  |   以上四种都属于基本型    |           STM32F101xx、STM32F102xx、STM32F103x系列           |
  |  startup_stm32f10x_cl.s   | cl:connectivity line devices，互联型，特指STM32F105xx和STM32F107xx系列 |
  | startup_stm32f10x_ld_vl.s |    vl:value line devices,超值型系列，特指STM32F100xx系列     |
  | startup_stm32f10x_md_vl.s |    vl:value line devices,超值型系列，特指STM32F100xx系列     |
  | startup_stm32f10x_hd_vl.s |    vl:value line devices,超值型系列，特指STM32F100xx系列     |

### include

include 有两种格式：

1. #include "stm32f10x.h" //双引号首先在当前工程目录下寻找，如果找不到，则到软件安装目录下寻找。
2. #include <stm32f10x.h> //尖括号直接在软件安装目录下寻找。 

### 寄存器的位操作

1. 变量某位清0	

   ```c
    a &= ~(1<<2);  // 将第二位清零 
   ```

2. 变量某位置1

   ```c
   a |= (1<<2);	// 将第二位置1
   ```

3. 变量某位取反

   ```c
   a ^= (1<<2);	// 将第二位取反
   ```



## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/ARM](https://blog.csdn.net/zipingpan/category_12627684.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**
