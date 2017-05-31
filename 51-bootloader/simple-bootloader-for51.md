---
title: 简易bootloader设计思路
date: 2016-07-06 14:42:23
toc: true
categories: bootloader
tags: 嵌入式 单片机
---
作为表计产品，限制于使用环境及产品制造特性（如超声波焊），通常在表计生产完成尤其是送到客户手中后，想要再升级固件程序，如果没有bootloader的支持，难度与代价都很大。因此，为现有平台所有系列的MCU添加bootloader程序是个刚性的需求。最近需要实现V9821的在线升级功能，研究了两天初步有了思路，接下来就是实现了，先记录下初步的思路。

<!-- more -->

实现bootloader的前提条件有：

- IAP功能
- 代码区足够大或存在外部flash

对于51单片机，最大支持64Kb的程序代码，其中包括中断向量、启动代码、用户代码、bootloader代码，因此需要合理分配flash空间。对于vango的V98xx系列单片机，程序存储区地址映射如下图:

<center>![v9821_FLASH](http://o9kzgz0kz.bkt.clouddn.com/V9811_FLASH.jpg)</center>

Flash存储器的**008000h--01FFFFh**的内容根据代码段选择寄存器`CBANK,SFR 0xA0`的配置，以32KB为单位，选择行的映射到程序存储器**0x8000-0xFFFF**区域。其中`common area`为公共代码区，使用时不需要切换`Code Bank`，默认代码段为`Bank1`。

使用时刻根据实际情况考虑是否使用<code>bank switch</code>技术，因为我用于验证的用户代码不大，加上bootloader程序也远达不到64Kb的代码量，因此不分`bank`。

## Flash存储器空间分配
- 0x0000-0x03FF: 中断向量表
- 0x0400-0x05FF: 512字节的出厂参数，单片机正常工作所必须的参数，不用于用户代码。
- 0x0600-0xD5FF: 用户代码
- 0xD600-0xD7FF: 中断映射代码
- 0xD800-0xD8FF: 启动代码
- 0xD900-0xFFFF: bootloader代码

> 其中启动代码、bootloader均位于不可IAP区。**0x000-0x03FF为不可IAP区域，硬件不支持**


## 软件设计思路

1. 上电复位：执行startup，跳转到bootloader代码；
2. 在bootloader中首先检查升级指令是否存在<sup>[1]</sup>，如果不存在跳转到用户代码，反之执行升级 。
3. 用户代码中也可被升级指令切换至bootloader程序；
4. 等待数据（等待时长可设），超时复位到用户代码；
5. 接收升级数据并校验<sup>[2]</sup>。
6. 写入update<sup>[3]</sup>。
7. 全部接收完成并校验通过，擦除用户代码，写入升级数据<sup>[4]</sup>。

## note

1. 可使用xdata区的固定位置的全局变量或者相关IO标志。
2. 使用hex文件格式的校验方式：校验和 = 0x100 - 累加和
3. 如果代码不超过32KB，直接写入update区。如果超过32KB，使用外部存储或者直接擦除用户代码，写入升级数据。
4. 写入的时候以16个字节为一组写入。