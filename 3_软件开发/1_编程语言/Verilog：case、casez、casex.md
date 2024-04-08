# Verilog：case、casez、casex

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。
2. 参考：

   - 《数字电子技术基础》-阎石
   - 《Verilog 数字系统设计教程》夏宇闻
   - 《Verilog HDL 高级数字设计》Michael D.Ciletti

---

## 正文

### case、casez、casex

case的真值表：

| case |  0   |  1   |  x   |  z   |
| :--: | :--: | :--: | :--: | :--: |
|  0   |  1   |  0   |  0   |  0   |
|  1   |  0   |  1   |  0   |  0   |
|  x   |  0   |  0   |  1   |  0   |
|  z   |  0   |  0   |  0   |  1   |

casez的真值表：

| casez |  0   |  1   |  x   |  z   |
| :---: | :--: | :--: | :--: | :--: |
|   0   |  1   |  0   |  0   |  1   |
|   1   |  0   |  1   |  0   |  1   |
|   x   |  0   |  0   |  1   |  1   |
|   z   |  1   |  1   |  1   |  1   |

casex的真值表：

| casex |  0   |  1   |  x   |  z   |
| :---: | :--: | :--: | :--: | :--: |
|   0   |  1   |  0   |  1   |  1   |
|   1   |  0   |  1   |  1   |  1   |
|   x   |  1   |  1   |  1   |  1   |
|   z   |  1   |  1   |  1   |  1   |



**Verilog HDL针对电路的特性提供了case语句的其他两种形式，即casez和casex，这可用来处理比较过程中的不必要考虑的情况。其中casez语句用来处理不考虑高阻值z的比较过程，casex语句则将高阻值z和不定值x都视为不必关心的情况。所谓不必关心的情况，即在表达式进行比较时，不将该位的状态考虑在内。这样，在case语句表达式进行比较时，就可以灵活地设置对信号的某些位进行比较。**

## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/编程语言](https://blog.csdn.net/zipingpan/category_12627795.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**

