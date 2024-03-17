# Intel FPGA (1)：关于quartus如何整理、降低project文件大小

## 前提摘要

1. 个人说明：

   **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“noahpanzzz@gmail.com”**

2. 参考

   - [(原创) 如何彻底删除SOPC Builder所遗留下的code? (SOC) (SOPC Builder) (Nios II)](https://www.cnblogs.com/oomusou/archive/2008/09/20/sopc_builder_project.html)
   - [(原創) 如何將編譯結果，統一放在一個目錄下? (SOC) (Quartus II)](https://www.cnblogs.com/oomusou/archive/2008/09/29/quartus_ii_release.html)
   - [(原創) 如何降低project壓縮檔的大小? (SOC) (Quartus II)](https://www.cnblogs.com/oomusou/archive/2008/09/29/quartus_ii_db.html)

3. 日期：2024-01-

---

## 正文

### (原创) 如何彻底删除SOPC Builder所遗留下的code? (SOC) (SOPC Builder) (Nios II)

**Abstract**
初学者学习SOPC Builder时，会发现尽管我在SOPC Builder移除了某些ip，但project内仍残留该ip的code，随着时间日积月累，垃圾code越来越多，想删除又怕误删了不该删的code，该怎么解决这个问题呢?

**Introduction**
Quartus II的project中档案很多，已经为人所诟病，但最少每个module是自己写的，该不该删除自己很清楚，但加入Nios II与SOPC Builder后，档案众多的问题则更雪上加霜。当我们在SOPC Builder加入某ip后，SOPC Builder会在project根目录下产生code，**但是若我们从SOPC Builder移除某ip后，SOPC Builder并不会替我们从project根目录将之前加入的code删除**，由于这些ip大都是Altera所提供，不是自己写的，自己删除又怕删错档案，经过日积月累，垃圾code将越来越多...。

在此我分享一个project管理技巧，日后就能彻底删除SOPC Builder所遗留下的code。

**Solution
Step 1：
将project的根目录另外开3个子目录：hardware、software、ip**

Nios II EDS预设就会自动建立software目录，SOPC Builder预设也会自动读取ip目录下自己写的ip，**hardware目录是我自己加上的，目的是为了放自己写的Verilog module**。

为什么要这样做呢?

由下图可知，根目录有以3种档案：
1.深蓝色为SOPC Builder所产生的code。
2.紫色为Quartus II project所需的档案与top module。
3.咖啡色为SOPC Builder所需的档案。

也就是说，**SOPC Builder产生的code会放在project的根目录，所以我们就将根目录净空让给SOPC Builder，而将自己写的Verilog module移到hardware目录下，日后若要删除SOPC Builder所产生的code，只要将Quartus II project与SOPC Builder以外的档案(深蓝色部分)全部删除即可，然后重新让SOPC Builder去Generate。**

![](Intel FPGA (1)：关于quartus如何整理、降低project文件大小-P1.jpg) 

**Step 2：
hardware目录下放自己写的Verilog module**

若你是将友晶的范例加上Nios II，则可将原范例的code全部搬到hardware下。

![](Intel FPGA (1)：关于quartus如何整理、降低project文件大小-P2.jpg)

**Step 3：
software目录放Nios II的C code**

![](Intel FPGA (1)：关于quartus如何整理、降低project文件大小-P3.jpg)

**Conclusion**
透过这个小技巧，以后就不用担心SOPC Builder所产生的code杀不干净，而且软体、硬体与ip的目录分得清清楚楚，日后维护也方便。

**See Also**
**《(原创) 如何将编译结果，统一放在一个目录下? (SOC) (Quartus II)》**

---

### (原创) 如何将编译结果，统一放在一个目录下? (SOC) (Quartus II)

**Abstract**
Quartus II预设会将所有档案都放在project的根目录下，导致根目录档案过多，管理不便，若能将编译的结果统一放到其他目录下，将有助于日后管理。

**Introduction
使用环境：Quartus II 8.0**

在**《(原创) 如何彻底删除SOPC Builder所遗留下的code? (SOC) (SOPC Builder) (Nios II)》**中，我曾经提出一种project管理方式，将Verilog code统一放在hardware目录下，将根目录净空，以方便日后好管理SOPC Builder所产生的code，经过学长lishyhan的提醒，Quartus II原来还可指定目录放置编译结果，如此可让project的根目录更加干净。

回想我们使用Visual Studio的经验，一个典型的project，除了自己的code外，Visual Studio还会另开Debug与Release目录，专职放置编译的结果，如下图所示：

![](Intel FPGA (1)：关于quartus如何整理、降低project文件大小-P4.jpg)

我将模仿Visual Studio的方式，新增一个release目录，专门放Quartus II编译的结果。

**Step 1：
建立一个release目录**

![](Intel FPGA (1)：关于quartus如何整理、降低project文件大小-P5.jpg)

**Step 2：
设定编译结果路径**

Assignments -> Settings：Category -> Compilation Process Settings：将Save project output files in specified directory打勾，并设定路径到release下

![](Intel FPGA (1)：关于quartus如何整理、降低project文件大小-P6.jpg)

**Conclusion**
经过如此设定，Quartus II就会将sof、pof等编译结果放到release目录下，原来在project根目录下编译结果的档案还会留着，你可视需要自行删除之。

**See Also**
**《(原创) 如何彻底删除SOPC Builder所遗留下的code? (SOC) (SOPC Builder) (Nios II)》**
**《(原创) 如何降低project压缩档的大小? (SOC) (Quartus II)》**



---



### (原创) 如何降低project压缩档的大小? (SOC) (Quartus II)

**Abstract**
当我们想将Quartus II整个project透过email或msn传给别人时，会希望整个project能尽量的压的最小，该如何最佳化我们的压缩档呢?

**Introduction**
适用版本：Quartus II各版本

有时朋友会将整个Quartus II project透过email或msn传给我一起研究，曾经有个project压缩前有50MB，用WinRAR压缩后还有25MB，非常惊人，经过我的优化后，压缩前剩下22MB ，压缩后仅剩下**2.39MB**，我是怎么办到的呢?

**Step 1：
将db目录下所有档案全部删除**

db目录占了30MB，是project肥大的罪魁祸首，若你用了Smart Compilation模式，db会更大。db目录下的档案，类似C的obj档，是compiler连结所用，只要重新编译就会产生。

**Step 2：
将编译结果仅留\*.sof档，其余可删除**

一般测试时，只会用到*.sof档，其他档案都用不到，请**《参阅[原创) 如何将编译结果，统一放在一个目录下? (SOC) (Quartus II)》**将编译结果统一放在指定目录下，只留下 *.sof档，其余档案皆可删除，若有需要 *.pof档烧入至epcs，只要重新编译就会产生。

**Step 3：
将software下的Debug\obj全部删除**

若有用到Nios II，可将Debug\obj下所有档案删除，这些都是obj档，run as hardware时会重新建立。

**Step 4：
使用7Zip压缩**

WinZip与WinRAR曾经是你我的最爱，但是WinRAR无论在压缩率与压缩速度都无法与**7Zip**相比，这也是为什么我放在blog上的压缩档，都是**7Zip**的7z格式。

**Conclusion**
透过这4个小技巧，就能马上降低project压缩档大小，省下宝贵的时间传输。

**See Also**
**《(原创) 如何将编译结果，统一放在一个目录下? (SOC) (Quartus II)》**

---

## 总结

**本博客中大多数均为原创，欢迎转载，请注明文章出处：[CSDN](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会标注出处，由于时效原因，可能并不是原创作者地址（已经找不到原创作者）。**

