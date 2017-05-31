---
title: STM8概述
toc: true
date: 2016-08-04 11:32:15
categories: STM8
tags: STM8
---
## 特性
> - 电源电压范围：1.8V-3.6V
- 低功耗：
	- 五种低功耗模式
	- 动态功耗：与时钟频率相关 200uA/MHz+330uA
	- IO漏电流：50nA
	- 从停止模式唤醒仅需4.7uS
- 先进的STM8内核
	- 哈佛结构
	- 可达到16MIPS
	- 多达40个外部中断源
- 丰富的外设
	- DMA
	- LCD
	- RTC
	- ADC
	- TIMER
	- CLOCK MANAGEMENT
	- ...

## <a href="http://www.xn--4gqa63c686ta68iba.ren/2016/08/04/STM8/STM8%E4%B9%8BGPIO/" >GPIO </a>
## <a href="" >REST </a>
## <a href="http://www.xn--4gqa63c686ta68iba.ren/2016/08/05/STM8/STM8%E4%B9%8BCLOCK/" >CLOCK </a>
## <a href="" >Memory </a>

以上只是一些我认为是重点的部分的简单的概要，很多操作都可以通过ST公司提供的库函数完成，这些库函数结构合理使用方便，用户完全可以拿过来使用，因此很多操作的细节了解下就好了，出了BUG知道一个方向去解决，其他的交给库函数吧。
*详细细节见* **[STM8L 用户指南](http://obd6jz6in.bkt.clouddn.com/STM8L%20%E7%94%A8%E6%88%B7%E6%8C%87%E5%8D%97.pdf)**