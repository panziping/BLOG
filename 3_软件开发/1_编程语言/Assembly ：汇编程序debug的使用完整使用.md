# Assembly：汇编程序debug的使用完整使用

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。
2. 参考：

   - 《微型计算机原理与接口技术》第三版 孙力娟

---

## 正文

### debug的命令符

| debug命令符 | Explain                                                      |
| ----------- | ------------------------------------------------------------ |
| -a          | 逐行汇编                                                     |
| -u          | 反汇编                                                       |
| -t          | 逐行执行命令                                                 |
| -d          | 显示一定内存单元内容，再次输入将在原显示内容上继续显示下面内存的内容； |
| -q          | 退出debug回到dos状态；                                       |
| -r          | 改变或显示一个或多个寄存器的内容；                           |
| -n          | 命名文件；                                                   |
| -w          | 将已命名文件写入磁盘；                                       |
| -l          | 将程序装载进内存。                                           |


### 具体使用流程

话不多说直接开始，我们以一段最简单例子为例来说明如何使用debug。


```assembly language
.486
DATAS SEGMENT USE16
DATAS ENDS

CODES SEGMENT USE16
    ASSUME CS:CODES,DS:DATAS
START:
    MOV AX,DATAS
    MOV DS,AX	
	MOV BX,1234H
    MOV AH,4CH
    INT 21H
CODES ENDS
    END START 

```

我们将1234H这个数送给BX寄存器看，进行debug可否查看到BX寄存器的变化。

首先我们需要将自己编写的程序放在MASM这个文件夹（ [如何在win10_64位下搭载汇编环境](https://blog.csdn.net/xyisv/article/details/69062382)）下,然后启动DOS。

![](./图库/Assembly (2)：汇编程序debug的使用完整使用/汇编程序debug的使用完整使用-P1.png)
我们使用debug-t命令逐行执行指令。
![](./图库/Assembly (2)：汇编程序debug的使用完整使用/汇编程序debug的使用完整使用-P2.png)

后来发现MASM软件其实内置了调试按钮，比使用DOS更加轻松方便（白弄DOS了？不不不知识还是有用的。）
![](./图库/Assembly (2)：汇编程序debug的使用完整使用/汇编程序debug的使用完整使用-P3.png)

## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/编程语言](https://blog.csdn.net/zipingpan/category_12627795.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**

