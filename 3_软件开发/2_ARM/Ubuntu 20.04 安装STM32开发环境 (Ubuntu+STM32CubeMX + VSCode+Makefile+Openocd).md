# Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + VSCode+Makefile+Openocd)

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。
2. 参考：

   - 

---

## 正文

### 安装开发环境

#### 具体方法：

**Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd**

#### 硬件平台：

- STM32F103RCT6核心版
- CMSIS DAP下载器

#### 安装软件

1. 安装STM32CubeMX软件和VSCode软件。

   网上此类软件的安装方法很多就不一一赘述了。

2. 安装编译工具链gcc-arm-embedded。

   其中包含了用于编译的arm-none-eabi-gcc和用于Debug的arm-none-eabi-gdb。

   下载网址：[https://launchpad.net/gcc-arm-embedded/+download](https://launchpad.net/gcc-arm-embedded/+download)。

   安装步骤：

   - 将工具链 复制 到 /usr/local/arm：

     ```bash
     sudo cp ~/下载/gcc-arm-none-eabi-5_4-2016q3-20160926-linux.tar.bz2 /usr/local/arm/
     ```

   - 解压压缩包：

     ```bash
     sudo tar -vxjf gcc-arm-none-eabi-5_4-2016q3-20160926-linux.tar.bz2
     ```

   - 修改环境变量，使用VI打开/etc/profile：

     ```bash
     sudo vi /etc/profile
     ```

   - 在最后面输入如下内容：

     ```bash
     export PATH=$PATH:/usr/local/arm/gcc-arm-none-eabi-5_4-2016q3/bin
     #输入arm-none-eabi-gcc -v来验证环境是否成功建立
     arm-none-eabi-gcc -v 
     ```

3. 安装GDB Sever，该工具往下用于连接jlink或stlink，往上提供reset，halt，flash等常用功能，用于程序下载和调试。此处选用OpenOCD，使用方便且开源。

   ```bash
   sudo apt-get install openocd
   ```

---

### STM32CubeMX生成STM32工程

具体生成工程不讲述，网上教程很多。注意在Toolchain/IDE中记得选择Makefile。

![](./图库/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)-P1.png)

---

### 使用VSCode打开工程

修改工程中Makefile文件，便于VSCode建立tasks调用程序下载和编译。在Makefile中最后可增加如下语句：

#### Makefile：

```bash
# Use The Tool By Openocd Download the Project
OPENOCD := openocd -f interface/cmsis-dap.cfg \
	-c 'transport select swd' \
	-f target/stm32f1x.cfg 
# download your program
flash: all
	$(OPENOCD) -c init \
		-c 'reset halt' \
		-c 'flash write_image erase $(BUILD_DIR)/$(TARGET).elf' \
		-c 'reset run' \
		-c exit
```

不要直接复制上述，Makefile有自己的语法规则，会导致编译不通过。

- 此时应该在Terminal中可以make flash直接一步编译并下载固件了（需要按住reset在松开进行下载固件）。
- 第一行 -f 表示的是选择cmsis-dap，如果是其他烧写方式可以到OpenOCD的scripts目录下进行查找。

默认安装OpenOCD的路径：/usr/share/openocd/scripts。

如果不是jlink可以到OpenOCD的scripts目录下找（如stlink v2.1则可以改为openocd -f interface/stlink-v2-1.cfg），默认路径为/usr/share/openocd/scripts。

第二行仅支持swd模式，所以加上了，实际如果支持jtag模式是不需要的。

第三行选择的是个人的芯片配置文件，同样可以在OpenOCD的scripts目录下找到，可根据自己选择的芯片进行修改。

#### 修改VSCode中C/C++插件

用VSCode打开STM32CubeMX新建的工程文件夹，将C/C++插件配置一波（其实不配置也没关系，只不过会有很多红色波浪线让人看着非常不舒服），便于智能感知的使用，主要是编译器路径和宏定义的配置。

1. 编译器路径配置

   在Terminal中使用whereis查看gcc-arm-embedded的路径

   如果gcc-arm-embedded安装正常，默认路径应该是/usr/bin/arm-none-eabi-gcc，也可以使用whereis查看：![](./图库/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)-P2.png)


​		VSCode快捷键Ctrl+Shift+p打开，搜索C/C++:Edit Configurations (UI)，即可打开配置界面，在Compile path输入上一步的路径：

​		**C/C++:Edit Configurations (UI)：**

​		![](./图库/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + VSCode+Makefile+Openocd)/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)-P3.png)

2. 宏定义的配置

   主要来源于Makefile文件中所使用的宏定义，将图中USE_HAL_DRIVER和STM32F103xx（记得去除-D，该处的宏定义实际与个人的工程有关，我使用的F103的芯片，所有会有STM32F103xx这一项）填入Defines中：

   **Makefile**：

   ![在这里插入图片描述](./图库/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + VSCode+Makefile+Openocd)/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)-P4.png)	

   **C/C++:Edit Configurations (UI)：**

​		![](./图库/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)-P5.png)

### 调试

每个人调试代码的方式可能都不一样，这里就不赘述了。等之后有机会在出个教程。


### 下载验证

![](./图库/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)/Ubuntu 20.04 安装STM32开发环境 (Ubuntu+STM32CubeMX + Vscode+Makefile+Openocd)-P6.png)



## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**

