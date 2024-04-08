# PadDesigner：Pad Designer 焊盘文件命名规则

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。
2. 参考：

   - [小哥Allegro:https://space.bilibili.com/456287853](https://space.bilibili.com/456287853)

---

## 正文

### 焊盘命名规则

单位使用的是millimeter(毫米)

Shape

1. Circle:圆
2. Square：方形
3. Oblong：椭圆形
4. Rect：矩形
5. Octagon：八边形
6. Shape：自定义形状

---





1. 表贴焊盘命名规则(例如：smd_rect_1r40x1r00，矩形焊盘宽1.40mm，高1.00mm)

| Header | Pad Shape | Width |  x   | Height |
| :----: | :-------: | :---: | :--: | :----: |
|  smd_  |   rect    | 1r30  |  x   |  0r30  |
|  smd_  |  oblong   | 1r30  |  x   |  0r30  |

BEGIN LAYER：表贴层。

SOLDERMASK：阻焊层（开窗，绿油，正常比表贴大0.1mm）。

PASTEMASK：钢网层（刷锡膏，正常与表贴同样大小）。

FILMMASK：预留层（**没用过，无需定义**）。

2. 通孔焊盘命名规则（例如，thru_circle_1r82x1r02,圆形焊盘直径1.82mm，钻孔直径1.02mm）

| Header | Hole Shape | Layer pad diameter |  x   | Drill diameter |
| :----: | :--------: | :----------------: | :--: | :------------: |
| thru_  |   circle   |        1R82        |  x   |      1R20      |

3. 机械钻孔命名规则）

| Header | Hole Shape | Drill diameter |  x   | Drill diameter |
| :----: | :--------: | :------------: | :--: | :------------: |
|  mec_  |   circle   |      1R20      |  x   |      1R20      |

4. flash焊盘命名规则

| Header | Hole Shape | Drill diameter |  x   | Drill diameter |
| :----: | :--------: | :------------: | :--: | :------------: |
| flash_ |   circle   |      0R94      |  x   |      0R94      |
| flash_ |   oblong   |      1R00      |  x   |      2R50      |

5. 过孔via命名规则

   | Header | Outer pad diameter | x    | Inner  drill hole diameter |
   | ------ | ------------------ | ---- | -------------------------- |
   | via_   | 24                 | x    | 12                         |

   



## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/Orcad Allegro](https://blog.csdn.net/zipingpan/category_12634775.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**



