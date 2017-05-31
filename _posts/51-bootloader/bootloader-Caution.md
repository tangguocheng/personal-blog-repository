---
title: bootloader使用注意事项
date: 2016-07-13 17:53:26
toc: true
categories: bootloader
tags: 嵌入式 单片机
---
## keil 链接选项
<center>![keil config](http://o9kzgz0kz.bkt.clouddn.com/kei%E8%AE%BE%E7%BD%AE.jpg)</center>
<!--more-->

```C
// 全局设置
#define D_FLAGADDR      0xFFF0							// 存储全局升级标志（可选）
#define D_MCU           D_V98XX							// MCU类型
#define D_FRAMELEN  	16							    // 每帧的数据长度
#define D_ADRESSMODE    D_ABSOLUTE					    // 寻址模式
#define D_READCHECK		D_FALSE						    // 读出比对
#define D_TIMETH        (30)                            // *50ms
// note* bootloader位于flash中的最后10k_(10或20页)(0xD800-0xFFFF),其中0xFE00页用于保存用户程序的入口地址
#if (D_MCU == D_V98XX)
    #define D_SECTORSUM		0x6c						// 128-20页
    #define D_SECTORSIZE	0x200 						// 每页大小，V98xx是512个字节 中颖7012是1024个字节
#endif
#if (D_MCU == D_SH7012)
    #define D_SECTORSUM		0x36 						// 64-10页
    #define D_SECTORSIZE/	0x400 						// 每页大小，V98xx是512个字节 中颖7012是1024个字节
#endif

#define D_FRAMENUM		(D_SECTORSIZE/D_FRAMELEN)	    // 传输一页数据需要的总帧数
```

其中<code>D_FLAGADDR</code>是作为升级标志的全局标志位，位于<code>xdata</code>区，在用户程序接受到升级指令后需要将该位置写入数据<code>0x55</code>。后续增加对IO口的支持。

## 关于中断处理
> 由于IAP的机制要求中断进行响应，而C51程序中的代码首页放置的就是STARTUP与各个中断入口的地址，一旦对首页0000-0200的512字节进行擦除或者写操作，就会导致IAP中断无法响应MCU一直处于挂起状态，所以为了避免这种情况必须保证首页不被擦除或改写，因此提前将中断入口地址固定，保证后面升级的程序中断入口地址不变。否则中断程序的跳转将指向错误的地址，同时为每个中断函数预留了一定的空间，方便用户添加简单的中断处理代码。

为了实现这点，必须将IAP所在的中断进行中断跳转处理，这会导致中断在响应的时候会慢至少一个`LJMP`指令，但是对于目前主频达到十几兆的增强型51来说是可以忽略不计的。

### 如何实现
因为IAP中断不可擦出，中断服务函数同样应该位于不可擦出段，因此可将该段代码定位在bootloader代码的开始。在IAP中断服务函数中跳转至中断映射代码，同样中断映射代码的函数地址同样需要固定，但是是可擦出的。

- 0x0000-0x03FF: 中断向量表
- 0x0400-0x05FF: 512字节的出厂参数，单片机正常工作所必须的参数，不用于用户代码。
- 0x0600-0xD4FF: 用户代码
- 0xD500-0xD6FF: 中断映射代码
- 0xD700-0xD7FF: 启动代码
- 0xD800-0xFFFF: bootloader代码

## 中断服务函数绝对定位地址

```C
     EXINT0SEG    =0xC500-0xC5FF
     TIMER0SEG    =0xC600-0xCBFF
     EXINT1SEG    =0xCC00-0xCCFF
     TIMER1SEG    =0xCD00-0xCDFF
     TIMER2SEG    =0xCE00-0xCEFF
     UART1SEG     =0xCF00-0xD1FF
    *UARTCFSEG    =0xD200-0xD3FF
     UARTRTCSEG   =0xD400-0xD4FF
     PLLEXINT3SEG =0xD500-0xD5FF
     TIMERASEG    =0xD600-0xD6FF
     POWERSEG     =0xD700-0xD7FF
```
**其中，UARTCFSEG实际并不是由中断向量直接指向，而是中断服务函数的一个子函数，因为UARTCF中断包含IAP中断，所以这部分的代码处在不可擦除区**。

## **非常重要的一点**

因为bootloader是和用户代码放在同一个工程下，对于常量尤其是函数内部的字符串常量，编译器在编译的时候可能会把这部分常量放进用户代码区，导致升级之后这部分变量会被更改。解决方法：

- 尽量少使用常量尤其是字符串常量
- bootloader代码中使用的所有变量，不要使用默认初始值也不要使用声明的同时赋值的用法；