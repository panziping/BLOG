# STM32：UART-应用层程序

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

```c
/*****************  发送一个字节 **********************/
void Usart_SendByte( USART_TypeDef * pUSARTx, uint16_t ch)
{
    /* 发送一个字节数据到USART */
    USART_SendData(pUSARTx,ch);
    /* 等待发送数据寄存器为空 */
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET);
}

/****************** 发送8位的数组 ************************/
void Usart_SendArray( USART_TypeDef * pUSARTx, uint8_t *array, uint16_t num)
{
    uint8_t i;

    for(i=0; i<num; i++)
    {
        /* 发送一个字节数据到USART */
        Usart_SendByte(pUSARTx,array[i]);
    }
    /* 等待发送完成 */
    while(USART_GetFlagStatus(pUSARTx,USART_FLAG_TC)==RESET);

}

/*****************  发送字符串 **********************/
void Usart_SendString( USART_TypeDef * pUSARTx, char *str)
{
    unsigned int k=0;
    do
    {
        Usart_SendByte( pUSARTx, *(str + k) );
        k++;
    } while(*(str + k)!='\0');
    /* 等待发送完成 */
    while(USART_GetFlagStatus(pUSARTx,USART_FLAG_TC)==RESET)
    {}
}

/*****************  发送一个16位数 **********************/
void Usart_SendHalfWord( USART_TypeDef * pUSARTx, uint16_t ch)
{
    uint8_t temp_h, temp_l;
    /* 取出高八位 */
    temp_h = (ch&0XFF00)>>8;
    /* 取出低八位 */
    temp_l = ch&0XFF;
    /* 发送高八位 */
    USART_SendData(pUSARTx,temp_h);
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET);
    /* 发送低八位 */
    USART_SendData(pUSARTx,temp_l);
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET);
}
```







## 总结





---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/ARM](https://blog.csdn.net/zipingpan/category_12627684.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**
