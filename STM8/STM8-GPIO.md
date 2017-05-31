---
title: STM8通用IO(GPIO)
toc: true
date: 2016-08-04 13:12:15
categories: STM8
tags: STM8
---
## 特性
- 每个IO口可单独配置
- 输入模式可配：上拉输入、悬浮输入
- 输出模式可配：推挽输出、开漏输出
- 输入输出数据寄存器独立
- 外部中断独立使能
- 输出斜率可控用以减小EMC噪声
- 管脚复用
- 1.6V-VddioMax直接IO状态稳定
<!--more-->
## 相关寄存器(Register)
主要是6个寄存器`DDR、CR1、CR2、ODR、IDR`,全部为8位一个字节，每一位对应一个IO口。

### DDR(date direction register)
控制IO口输入输出方向，0-输入模式，1-输出模式。

### CR1(port control register #1)
- 输入模式
	- 0：悬浮输入
	- 1：输入上拉
- 输出模式
	- 0：伪开漏输出
	- 1：推挽输出，输出斜率可调(CR2)

### CR2(port control register #2)
- 输入模式
	- 0：外部中断禁止
	- 1：外部中断使能
- 输出模式
	- 0：输出最大速率2MHz，低速模式
	- 1：输出最大速率10MHz，高速模式

### ODR(output date register)
输出寄存器，忘改寄存器写入数据，可改变输出管脚电平状态。

### IDR(input date register)
输入寄存器，读取该寄存器可得到当前管脚电平状态。

*以上只是简单的概要，详细细节见* **[STM8L 用户指南](http://obd6jz6in.bkt.clouddn.com/STM8L%20%E7%94%A8%E6%88%B7%E6%8C%87%E5%8D%97.pdf)**