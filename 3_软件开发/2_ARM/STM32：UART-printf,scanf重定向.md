# STM32：UART-printf,scanf重定向

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

在C 语言标准库中，fputc 函数是printf 函数内部的一个函数，功能是将字符ch 写入到文件指针file所指向文件的当前写指针位置，简单理解就是把字符写入到特定文件中。我们使用USART 函数重新修改fputc 函数内容，达到类似“写入”的功能。fgetc 函数与fputc 函数非常相似，实现字符读取功能。在使用scanf 函数时需要注意字符输入格式。还有一点需要注意的，使用fput 和fgetc 函数达到重定向C 语言标准库输入输出函数必须在MDK的工程选项把“Use MicroLIB”勾选上，MicoroLIB 是缺省C 库的备选库，它对标准C 库进行了高度优化使代码更少，占用更少资源。

```c
//重定向c库函数printf到串口，重定向后可使用printf函数
int fputc(int ch, FILE *f)
{
    USART_SendData(USART1, (uint8_t) ch);
    while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
    return (ch);
}
//重定向c库函数scanf到串口，重写向后可使用scanf、getchar等函数
int fgetc(FILE *f)
{
    while (USART_GetFlagStatus(USART1, USART_FLAG_RXNE) == RESET);
    return (int)USART_ReceiveData(USART1);
}
```



## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/ARM](https://blog.csdn.net/zipingpan/category_12627684.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**
