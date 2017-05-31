---
title: 可以间接消抖的按键长按短按识别
date: 2016-09-18 16:29:07
tags: 按键识别
categories: 单片机
---

按键按下时触发按键计时，按键释放结束计时，计时时间T<sub>key</sub> < T<sub>short</sub> 时，判为短按。T<sub>key</sub> > T<sub>long</sub>时，判为长按。按键在触发时需要消除抖动，抖动本质上也是一次极短的短按，因此可以根据短按时间来消除抖动。定时器定时扫描端口，源代码如下：

```C
    if (D_keyDown())                       
    {
        if (x_key.keyTmCnt < D_maxCntNum)
        {
            x_key.keyTmCnt++;
        }
        
        if (x_key.keyTmCnt >= D_timeShortKey)    /* 大于D_minTimeFilter判为有效按键,短按 */
        {
            x_key.shortKeyFlag = D_TRUE;
        }
        
        if (x_key.keyTmCnt >= D_timeLongKey)     /* 长按 */
        {
            x_key.keyState = D_keyPushLong;
            if (x_key.shortKeyFlag == D_TRUE)
            {
                x_key.shortKeyFlag = D_FALSE;    /* 清楚短按标志，防止误触发，很重要 */
            }
        }
    }
    else
    {
        x_key.keyTmCnt = 0;
        if (x_key.shortKeyFlag == D_TRUE)       
        {
            x_key.shortKeyFlag = D_FALSE;
            x_key.keyState = D_keyPushShort;
        }
    }
```