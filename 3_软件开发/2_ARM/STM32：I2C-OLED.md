# STM32：I2C-OLED

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

OLED即有机发光管(Organic Light-Emitting Diode，OLED)。OLED显示技术具有自发光、广视角、几乎无穷高的对比度、较低功耗、极高反应速度、可用于绕曲性面板、使用温度范围广、构造及制程简单等有点，被认为是下一代的平面显示屏新兴应用技术。

OLED显示和传统的LCD显示不同，其可以自发光，所以不需要背光灯，这使得OLED显示屏相对于LCD显示屏尺寸更薄，同时显示效果更优。

常用的OLED屏幕有蓝色、黄色、白色等几种。屏的大小为0.96寸，像素点为128*64，所以我们称为0.96oled屏或者12864屏。

### OLED屏幕参数

本篇使用的是中景园1.29寸OLED

1. 像素数: 128 x 64
2. 有效面积: 29.42 x 14.7 (mm)
3. 驱动芯片：SSD1315



### 编程指南

本文参考了上文**05-STM32F1 - 串行通信2-I2C（2），应用层程序**中对IIC从设备进行连续读写。由于原厂提供了驱动程序，但是使用的软件IIC，所以对于底层的驱动有所改变，上层应用层的程序，并没有什么变化。

主要是修改了：

```c
void BSP_OLED_Refresh(void); 		//刷新屏幕
void BSP_OLED_Clear(void);			//清除屏幕
void BSP_OLED_ColorTurn(u8 i);		//正反色显示
void BSP_OLED_DisplayTurn(u8 i);	//正反转显示
void BSP_OLED_DisPlay_On(void);		//开启显示
void BSP_OLED_DisPlay_Off(void);	//关闭显示
void BSP_OLED_Init(void)；		   //OLED初始化
```

上述函数都操作了硬件IIC。

```c
void OLED_DrawPoint(u8 x,u8 y,u8 t);
void OLED_DrawLine(u8 x1,u8 y1,u8 x2,u8 y2,u8 mode);
void OLED_DrawCircle(u8 x,u8 y,u8 r);
void OLED_ShowChar(u8 x,u8 y,char chr,u8 size1,u8 mode);

void OLED_ShowString(u8 x,u8 y,char *chr,u8 size1,u8 mode);
void OLED_ShowNum(u8 x,u8 y,u32 num,u8 len,u8 size1,u8 mode);
void OLED_ShowChinese(u8 x,u8 y,u8 num,u8 size1,u8 mode);
void OLED_ScrollDisplay(u8 num,u8 space,u8 mode);
void OLED_ShowPicture(u8 x,u8 y,u8 sizex,u8 sizey,u8 BMP[],u8 mode);
```

显存数组的操作都没有进行改动，依旧使用的是原厂的程序。

```
uint8_t OLED_GRAM[128][8]
```



具体代码参考如下：

```c

#define BSP_OLED_ADDRESS7               0x78
#define OLED_CMD_Addr                   0x00
#define OLED_DATA_Addr                  0x40


uint8_t OLED_GRAM[128][8];

void BSP_OLED_Refresh(void) {
    uint8_t i, n;
    uint8_t OLED_Line[128];
    for (i = 0; i < 8; i++) {
        BSP_I2C_WRByte(0xb0 + i,BSP_OLED_ADDRESS7,OLED_CMD_Addr);
        BSP_I2C_WRByte(0x00,BSP_OLED_ADDRESS7,OLED_CMD_Addr);
        BSP_I2C_WRByte(0x10,BSP_OLED_ADDRESS7,OLED_CMD_Addr);
        for (n = 0; n < 128; n++)
            OLED_Line[n] = OLED_GRAM[n][i];
        BSP_I2C_KeepWrite(OLED_Line,BSP_OLED_ADDRESS7,OLED_DATA_Addr, 128);
    }
}


//清屏函数
void BSP_OLED_Clear(void) {
    uint8_t i, n;
    for (i = 0; i < 8; i++)
        for (n = 0; n < 128; n++)
            OLED_GRAM[n][i] = 0;//清除所有数据
    BSP_OLED_Refresh();//更新显示
}


void BSP_OLED_ColorTurn(u8 i) {
    if (i == 0)
        BSP_I2C_WRByte(0xA6,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//正常显示
    if (i == 1)
        BSP_I2C_WRByte(0xA7,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//反色显示
}

void BSP_OLED_DisplayTurn(u8 i) {
    if (i == 0) {
        BSP_I2C_WRByte(0xC8,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//正常显示
        BSP_I2C_WRByte(0xA1,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
    }
    if (i == 1) {
        BSP_I2C_WRByte(0xC0,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//反转显示
        BSP_I2C_WRByte(0xA0,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
    }
}


void BSP_OLED_DisPlay_On(void) {
    BSP_I2C_WRByte(0x8D,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
    BSP_I2C_WRByte(0x14,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
    BSP_I2C_WRByte(0xAF, BSP_OLED_ADDRESS7,OLED_CMD_Addr);
}

void BSP_OLED_DisPlay_Off(void) {
    BSP_I2C_WRByte(0x8D,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
    BSP_I2C_WRByte(0x10,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
    BSP_I2C_WRByte(0xAE,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
}


void BSP_OLED_Init(void) {
    BSP_I2C_WRByte(0xAE,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--turn off oled panel
    BSP_I2C_WRByte(0x00,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//---set low column address
    BSP_I2C_WRByte(0x10,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//---set high column address
    BSP_I2C_WRByte(0x40,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set start line address  Set Mapping RAM Display Start Line (0x00~0x3F)
    BSP_I2C_WRByte(0x81,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set contrast control register
    BSP_I2C_WRByte(0xCF,BSP_OLED_ADDRESS7, OLED_CMD_Addr);// Set SEG Output Current Brightness
    BSP_I2C_WRByte(0xA1,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--Set SEG/Column Mapping     0xa0左右反置 0xa1正常
    BSP_I2C_WRByte(0xC8,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//Set COM/Row Scan Direction   0xc0上下反置 0xc8正常
    BSP_I2C_WRByte(0xA6,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set normal display
    BSP_I2C_WRByte(0xA8,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set multiplex ratio(1 to 64)
    BSP_I2C_WRByte(0x3f,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--1/64 duty
    BSP_I2C_WRByte(0xD3,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//-set display offset	Shift Mapping RAM Counter (0x00~0x3F)
    BSP_I2C_WRByte(0x00,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//-not offset
    BSP_I2C_WRByte(0xd5,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set display clock divide ratio/oscillator frequency
    BSP_I2C_WRByte(0x80,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set divide ratio, Set Clock as 100 Frames/Sec
    BSP_I2C_WRByte(0xD9,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set pre-charge period
    BSP_I2C_WRByte(0xF1,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//Set Pre-Charge as 15 Clocks & Discharge as 1 Clock
    BSP_I2C_WRByte(0xDA,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set com pins hardware configuration
    BSP_I2C_WRByte(0x12,BSP_OLED_ADDRESS7, OLED_CMD_Addr);
    BSP_I2C_WRByte(0xDB,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set vcomh
    BSP_I2C_WRByte(0x40,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//Set VCOM Deselect Level
    BSP_I2C_WRByte(0x20,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//-Set Page Addressing Mode (0x00/0x01/0x02)
    BSP_I2C_WRByte(0x02,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//
    BSP_I2C_WRByte(0x8D,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set Charge Pump enable/disable
    BSP_I2C_WRByte(0x14,BSP_OLED_ADDRESS7, OLED_CMD_Addr);//--set(0x10) disable
    BSP_I2C_WRByte(0xA4,BSP_OLED_ADDRESS7, OLED_CMD_Addr);// Disable Entire Display On (0xa4/0xa5)
    BSP_I2C_WRByte(0xA6,BSP_OLED_ADDRESS7, OLED_CMD_Addr);// Disable Inverse Display On (0xa6/a7)
    BSP_OLED_Clear();
    BSP_I2C_WRByte(0xAF,BSP_OLED_ADDRESS7,OLED_CMD_Addr);
    BSP_OLED_ColorTurn(0);
    BSP_OLED_DisplayTurn(0);
}

```



```c
//画点
//x:0~127
//y:0~63
//t:1 填充 0,清空

void OLED_DrawPoint(u8 x, u8 y, u8 t) {
    u8 i, m, n;
    i = y / 8;
    m = y % 8;
    n = 1 << m;
    if (t) { OLED_GRAM[x][i] |= n; }
    else {
        OLED_GRAM[x][i] = ~OLED_GRAM[x][i];
        OLED_GRAM[x][i] |= n;
        OLED_GRAM[x][i] = ~OLED_GRAM[x][i];
    }
}

//画线
//x1,y1:起点坐标
//x2,y2:结束坐标
void OLED_DrawLine(u8 x1, u8 y1, u8 x2, u8 y2, u8 mode) {
    u16 t;
    int xerr = 0, yerr = 0, delta_x, delta_y, distance;
    int incx, incy, uRow, uCol;
    delta_x = x2 - x1; //计算坐标增量
    delta_y = y2 - y1;
    uRow = x1;//画线起点坐标
    uCol = y1;
    if (delta_x > 0)incx = 1; //设置单步方向
    else if (delta_x == 0)incx = 0;//垂直线
    else {
        incx = -1;
        delta_x = -delta_x;
    }
    if (delta_y > 0)incy = 1;
    else if (delta_y == 0)incy = 0;//水平线
    else {
        incy = -1;
        delta_y = -delta_x;
    }
    if (delta_x > delta_y)distance = delta_x; //选取基本增量坐标轴
    else distance = delta_y;
    for (t = 0; t < distance + 1; t++) {
        OLED_DrawPoint(uRow, uCol, mode);//画点
        xerr += delta_x;
        yerr += delta_y;
        if (xerr > distance) {
            xerr -= distance;
            uRow += incx;
        }
        if (yerr > distance) {
            yerr -= distance;
            uCol += incy;
        }
    }
}


//x,y:圆心坐标
//r:圆的半径
void OLED_DrawCircle(u8 x, u8 y, u8 r) {
    int a, b, num;
    a = 0;
    b = r;
    while (2 * b * b >= r * r) {
        OLED_DrawPoint(x + a, y - b, 1);
        OLED_DrawPoint(x - a, y - b, 1);
        OLED_DrawPoint(x - a, y + b, 1);
        OLED_DrawPoint(x + a, y + b, 1);

        OLED_DrawPoint(x + b, y + a, 1);
        OLED_DrawPoint(x + b, y - a, 1);
        OLED_DrawPoint(x - b, y - a, 1);
        OLED_DrawPoint(x - b, y + a, 1);

        a++;
        num = (a * a + b * b) - r * r;//计算画的点离圆心的距离
        if (num > 0) {
            b--;
            a--;
        }
    }
}


//在指定位置显示一个字符,包括部分字符
//x:0~127
//y:0~63
//size1:选择字体 6x8/6x12/8x16/12x24
//mode:0,反色显示;1,正常显示
void OLED_ShowChar(u8 x, u8 y, char chr, u8 size1, u8 mode) {
    u8 i, m, temp, size2, chr1;
    u8 x0 = x, y0 = y;
    if (size1 == 8)size2 = 6;
    else size2 = (size1 / 8 + ((size1 % 8) ? 1 : 0)) * (size1 / 2);  //得到字体一个字符对应点阵集所占的字节数
    chr1 = chr - ' ';  //计算偏移后的值
    for (i = 0; i < size2; i++) {
        if (size1 == 8) { temp = asc2_0806[chr1][i]; } //调用0806字体
        else if (size1 == 12) { temp = asc2_1206[chr1][i]; } //调用1206字体
        else if (size1 == 16) { temp = asc2_1608[chr1][i]; } //调用1608字体
        else if (size1 == 24) { temp = asc2_2412[chr1][i]; } //调用2412字体
        else return;
        for (m = 0; m < 8; m++) {
            if (temp & 0x01)OLED_DrawPoint(x, y, mode);
            else OLED_DrawPoint(x, y, !mode);
            temp >>= 1;
            y++;
        }
        x++;
        if ((size1 != 8) && ((x - x0) == size1 / 2)) {
            x = x0;
            y0 = y0 + 8;
        }
        y = y0;
    }
}


//显示字符串
//x,y:起点坐标
//size1:字体大小
//*chr:字符串起始地址
//mode:0,反色显示;1,正常显示
void OLED_ShowString(u8 x, u8 y, char *chr, u8 size1, u8 mode) {
    while ((*chr >= ' ') && (*chr <= '~'))//判断是不是非法字符!
    {
        OLED_ShowChar(x, y, *chr, size1, mode);
        if (size1 == 8)x += 6;
        else x += size1 / 2;
        chr++;
    }
}

//m^n
u32 OLED_Pow(u8 m, u8 n) {
    u32 result = 1;
    while (n--) {
        result *= m;
    }
    return result;
}

//显示数字
//x,y :起点坐标
//num :要显示的数字
//len :数字的位数
//size:字体大小
//mode:0,反色显示;1,正常显示
void OLED_ShowNum(u8 x, u8 y, u32 num, u8 len, u8 size1, u8 mode) {
    u8 t, temp, m = 0;
    if (size1 == 8)m = 2;
    for (t = 0; t < len; t++) {
        temp = (num / OLED_Pow(10, len - t - 1)) % 10;
        if (temp == 0) {
            OLED_ShowChar(x + (size1 / 2 + m) * t, y, '0', size1, mode);
            //OLED_ShowChar(x + (size1 / 2 + m) * t, y, ' ', size1, mode);
        } else {
            OLED_ShowChar(x + (size1 / 2 + m) * t, y, temp + '0', size1, mode);
        }
    }
}

//显示汉字
//x,y:起点坐标
//num:汉字对应的序号
//mode:0,反色显示;1,正常显示
void OLED_ShowChinese(u8 x, u8 y, u8 num, u8 size1, u8 mode) {
    u8 m, temp;
    u8 x0 = x, y0 = y;
    u16 i, size3 = (size1 / 8 + ((size1 % 8) ? 1 : 0)) * size1;  //得到字体一个字符对应点阵集所占的字节数
    for (i = 0; i < size3; i++) {
        if (size1 == 16) { temp = Hzk1[num][i]; }//调用16*16字体
        else if (size1 == 24) { temp = Hzk2[num][i]; }//调用24*24字体
        else if (size1 == 32) { temp = Hzk3[num][i]; }//调用32*32字体
        else if (size1 == 64) { temp = Hzk4[num][i]; }//调用64*64字体
        else return;
        for (m = 0; m < 8; m++) {
            if (temp & 0x01)OLED_DrawPoint(x, y, mode);
            else OLED_DrawPoint(x, y, !mode);
            temp >>= 1;
            y++;
        }
        x++;
        if ((x - x0) == size1) {
            x = x0;
            y0 = y0 + 8;
        }
        y = y0;
    }
}

//num 显示汉字的个数
//space 每一遍显示的间隔
//mode:0,反色显示;1,正常显示
void OLED_ScrollDisplay(u8 num, u8 space, u8 mode) {
    u8 i, n, t = 0, m = 0, r;
    while (1) {
        if (m == 0) {
            OLED_ShowChinese(128, 24, t, 16, mode); //写入一个汉字保存在OLED_GRAM[][]数组中
            t++;
        }
        if (t == num) {
            for (r = 0; r < 16 * space; r++)      //显示间隔
            {
                for (i = 1; i < 144; i++) {
                    for (n = 0; n < 8; n++) {
                        OLED_GRAM[i - 1][n] = OLED_GRAM[i][n];
                    }
                }
                BSP_OLED_Refresh();
            }
            t = 0;
        }
        m++;
        if (m == 16) { m = 0; }
        for (i = 1; i < 144; i++)   //实现左移
        {
            for (n = 0; n < 8; n++) {
                OLED_GRAM[i - 1][n] = OLED_GRAM[i][n];
            }
        }
        BSP_OLED_Refresh();
    }
}

//x,y：起点坐标
//sizex,sizey,图片长宽
//BMP[]：要写入的图片数组
//mode:0,反色显示;1,正常显示
void OLED_ShowPicture(u8 x, u8 y, u8 sizex, u8 sizey, u8 BMP[], u8 mode) {
    u16 j = 0;
    u8 i, n, temp, m;
    u8 x0 = x, y0 = y;
    sizey = sizey / 8 + ((sizey % 8) ? 1 : 0);
    for (n = 0; n < sizey; n++) {
        for (i = 0; i < sizex; i++) {
            temp = BMP[j];
            j++;
            for (m = 0; m < 8; m++) {
                if (temp & 0x01)OLED_DrawPoint(x, y, mode);
                else OLED_DrawPoint(x, y, !mode);
                temp >>= 1;
                y++;
            }
            x++;
            if ((x - x0) == sizex) {
                x = x0;
                y0 = y0 + 8;
            }
            y = y0;
        }
    }
}
```







## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/ARM](https://blog.csdn.net/zipingpan/category_12627684.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**
