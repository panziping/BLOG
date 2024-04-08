# Allegro：电路板板框

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。

2. 参考：

   - [小哥Allegro:https://space.bilibili.com/456287853](https://space.bilibili.com/456287853)

---

## 正文

### 新建工程（File->New）

![](图库\Allegro：电路板板框\Allegro：电路板板框-P1.png)

### 设置绘图尺寸，栅格(Setup->Design Parameter)



#### Display

开启栅格，栅格大小5mil

![](图库\Allegro：电路板板框\Allegro：电路板板框-P2.png)

#### Design

单位mil，精度0

绘图尺寸 width :18000mil，height:12000mil（约等于width 457mm，304mm）

![](图库\Allegro：电路板板框\Allegro：电路板板框-P3.png)

### 设置电路板板框（Add->Line）

Class：Board Geometry,SubClass：Outline，线宽：0（嘉立创免费打板10cm*10cm，这里暂时设置这个尺寸约等于3900mil）。

![](图库\Allegro：电路板板框\Allegro：电路板板框-P4.png)

通过输入坐标的方式 x -1950 -1950; ix 3900; iy 3900; ix -3900; iy -3900;

### 电路板倒角（Manufacture->drafting->chamfer,fillet）

电路板角比较锋利，所以需要对四个角进行倒角。有两种选择：

1. chamfer：45°角
2. fillet：圆弧角

这里使用圆弧角，倒角半径为80mils（≈2mm）

进入倒角命令下，用鼠标点击角对应的两条直线，则完成倒角

![](图库\Allegro：电路板板框\Allegro：电路板板框-P5.png)

### 设置允许布线区域（Setup->areas->route keepin）

设置布线离板边距离100mil（2.54mm）

![](图库\Allegro：电路板板框\Allegro：电路板板框-P6.png)

通过输入坐标的方式：x -1850 -1850;ix 3700;iy 3700;ix -3700;iy -3700。

### 设置允许器件放置区域(Setup->areas->package keepin)

生产流水线，夹具等原因，同样器件距离板边也应该有所距离。这里设置同上面一样。

同样可以通过输入坐标的方式：x -1850 -1850;ix 3700;iy 3700;ix -3700;iy -3700。

这里使用另外一种方式Z-copy（Edit->Z-copy）。需要将find中选为shapes，Class选为Package Keepin，Subclass选为all，点击route keepin的shape。

Z-copy中可以对Shape进行放大（Expand），缩小（Contract）。

![](图库\Allegro：电路板板框\Allegro：电路板板框-P7.png)

### 放置安装孔（Place Manaully）

在Advanced Setting中勾选library。

在Placement list中选择mechanical symbol，选择之前画M3机械钻孔。

通过输入坐标的方式放置四个安装孔：x -1750 -1750；x 1750 1750;x -1750 1750;x 1750 -1750。

![](图库\Allegro：电路板板框\Allegro：电路板板框-P8.png)

设置禁止摆放元件区域（Package Keepout）

通过绘制矩形的方式，设置禁止摆放元件区域







## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/Orcad Allegro](https://blog.csdn.net/zipingpan/category_12634775.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**









