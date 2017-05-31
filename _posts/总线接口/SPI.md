---
title: SPI
date: 2016-08-03 10:12:15
categories: 总线协议
tags: 
---

参考链接:
- [wikipedia SPI](https://zh.wikipedia.org/zh-cn/SPI) 
- [I<sup>2</sup>C](http://www.xn--4gqa63c686ta68iba.ren/2016/08/09/IIC/)

- SPI与I<sup>2</sup>C总线类似，不同的是I<sup>2</sup>C是半双工，SPI是全双工的数据总线。
- SPI采用四线制：
```
	SCLK（Serial Clock）：串行时钟，由主机发出
	MOSI（Master Output,Slave Input）：主机输出从机输入信号，由主机发出
	MISO（Master Input,Slave Output）：主机输入从机输出信号，由从机发出
	SS（Slave Selected）：片选信号，由主机发出，低电平有效
```
- 与I<sup>2</sup>C类似，沿接收、沿发送，高低位时数据线保持稳定。
- I<sup>2</sup>C采用发送数据中包含地址信息来选择从设备，SPI使用片选信号来选择从设备，因此从标准上来看，I<sup>2</sup>C比SPI的从设备选择更加灵活。
- 使用SPI时，需要明确总线时钟极性与相位。根据实际外设需求设置。