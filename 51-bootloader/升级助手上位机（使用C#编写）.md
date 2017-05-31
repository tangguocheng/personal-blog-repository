---
title: 升级助手上位机（使用C#编写）
date: 2016-07-26 16:33:21
toc: true
categories: C#
tags: C# 上位机 串口
---
bootloader写完之后，使用C#写了一个配套的升级助手。不得不说C#真的是一个很优秀的语言，而**visual studio**也完全可以称之为最强IDE，没有之一！配合[MSDN](https://msdn.microsoft.com/zh-cn/dn308572.aspx)，花了一个星期熟悉C#,然后两周不到完成了51升级助手的开发。

## 主界面
![51升级助手](http://o9kzgz0kz.bkt.clouddn.com/51bootloader.jpg)

> 特点:
1. 支持多种不同的通信协议（主要用于升级前的握手）
	- 用户自定义协议
	- IEC62056-21 C模式
	- DL/T645-07
2. 配合51bootloader完成用户代码的更新
	- 支持原生Intel HEX-80格式
	- 支持设定用户代码段地址范围
	- 支持自动重发
	- 自动剔除全0xFF数据，减少发送数据量
	
> 使用到的功能模块:
1.串口通信(使用C#官方serialPort类库);
2.文件读写(使用C#官方File类库);
3.设置文件(使用xml保存);
4.定时器(用于通信时序控制);

## 整体思路
首先上位机升级助手打开本地`hex`文件，对hex文件进行格式化，方便后面的数据传输。然后下位机在接收到升级指令后会判断是否符合升级条件，如果符合，则跳转到`bootloader`程序中准备接收升级数据，否则忽略升级指令，并返回响应帧通知上位机。
在升级的过程中，双方遵循同一个校验算法，下位机校验数据通过，通过`IAP`写入`flash`，否则请求重发。
下位机检测到处理到用户代码段的最后一帧数据，发送升级完成指令，下位机接收校验通过，通过长跳转指令调到用户代码，完成程序升级。
因为下位机`bootloader`采用查询的方式通信，为了提高可靠信，及降低出错时的重传代价，一次传输的代码字节限制在16个字节，所以需要格式化`hex-80`文件
```cpp
private bool handleHexFile(string filePath)
{
    string[] Hex=System.IO.File.ReadAllLines(filePath);
    string[] formatHex= System.IO.File.ReadAllLines(Application.StartupPath + "\\tempHexFile.txt");
    uint dateAddr = 0;
    uint dateLen = 0;
    foreach (string hexLine in Hex)
    {
        if (!string.IsNullOrEmpty(hexLine))
        {
            if (hexLine.Substring(0, 1) == ":")          // hex-80文件以冒号":"开头
            {
                dateLen = Convert.ToUInt32(hexLine.Substring(1,2).Trim(),16);
                dateAddr = Convert.ToUInt32(hexLine.Substring(3, 4).Trim(),16);
                if (dateLen == 0)
                {
                    continue;
                }
                uint modLine = dateAddr / 16;
                uint modByte = dateAddr % 16 + 1;
                uint modLen = dateLen;
                string[] tempStr=null;
                tempStr = formatHex[modLine].Trim().Split(' ');
                
                for (int i = 0; i < modLen*2; i+=2)
                {
                    tempStr[modByte] = hexLine.Substring(9 + i, 2).Trim();
                    if (modByte>=16)
                    {
                        string temp = null;
                        for (int n = 0; n < tempStr.Length; n++)
                        {
                            temp += (tempStr[n].TrimStart() + ' ');
                        }
                        formatHex[modLine] = temp;
                       
                        modByte = 0;
                        modLine++;
                        tempStr = formatHex[modLine].Trim().Split(' ');
                    }
                    modByte++;
                }
                string temp2 = null;
                for (int n = 0; n < tempStr.Length; n++)
                {
                    temp2 += (tempStr[n].TrimStart() + ' ');
                }
                formatHex[modLine] = temp2;

            }
            else
            {
                tbInformation.AppendText("empty");
                return false;
            }
        }    
    }
    updateFile = formatHex;
    System.IO.File.WriteAllLines(Application.StartupPath + "\\HexFile.txt", formatHex);
    return true;
}
```
除此之外，根据通信协议完成通讯部分的代码设计，一个可用的升级助手基本完成。