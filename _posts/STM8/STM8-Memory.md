---
title: STM8内存(memory)
toc: true
date: 2016-08-08 10:12:15
categories: STM8
tags: STM8
---
以低密度flash内存分配为例(高密度和中等密度只是内存分配及大小不同)，下图是内存映射图。
<center><img src="http://obd6jz6in.bkt.clouddn.com/STM8%E4%BD%8E%E5%AF%86%E5%BA%A6Flash.jpg"></center>
和51单片机一样，STM8也是8位机16位地址总线宽度，设计最大寻址范围也是64Kb，每页128字节，IAP操作以页为单位。整个FLASH被分为数据EEPROM(`DATA EEPROM`)、配置选项(`OPTION BYTES`)、代码区(`CODE FLASH`)。

### 数据EEPROM(`DATA EEPROM`)
数据EEPROM可以用来存储一些应用数据，如软件版本号、公司或者组织信息、作者信息等等。初始时，为了保证数据安全，在IAP模式下这部分是写保护的。写保护可以通过MASS密码序列解除，此时可向EEPROM中写入数据。

### 主程序代码区
在UBC或者私有代码区(`PCODE`)里的程序执行完后跳转到存储在这里执行代码。这里一般存的是用户代码。

### 配置选项(`OPTION BYTES`)
这部分存储的是硬件配置代码，以字节为单位。主要会影响到硬件配置和内存的写保护。这部分的内容可以通过编程器在ICP条件下或者用户代码在IAP条件下改写，**UBC与PCODE不可以更改这部分内容**。

### 用户启动代码区(`UBC`)
这个区域里包含用户IAP升级程序和各类中断向量，芯片在复位时会从RESET中断跳转到执行bootloader代码(这部分代码ST官方提供，用户也可以编写自己的bootloader)。UBC无法通过IAP改写，因为UBC拥有一个更强的二级写保护，这个写保护无法通过MASS密码序列解锁。因此更改这部分的代码只能通过ICP模式(使用 SWIM 接口)。通过配置选项(`OPTION BYTES`)，可以设置UBC的大小(以页为单位)。

### *私有代码区(`PCODE`)*
这部分不是所有型号的STM8芯片都存在的，具体是否有这部分，请参阅所使用芯片的datesheet。PCODE用来存放一些用以驱动硬件的私有代码库。可以在ICP模式下通过`PCODESIZE`配置选项配置其打下，一旦配置完成无法擦除，相应的PCODE区的大小也就固定不可更改了。

*以上只是一些简单的概要，详细细节见* **[STM8L 用户指南](http://obd6jz6in.bkt.clouddn.com/STM8L%20%E7%94%A8%E6%88%B7%E6%8C%87%E5%8D%97.pdf)**