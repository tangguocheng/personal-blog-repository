---
title: STM32型号命名规则
date: 2016-11-28 11:11:33
tags:
---
## stm32型号命名规则
示例：

```
STM32 | F | 100 | C | 6 | T | 6 | B | XXX
  1     2    3    4   5   6   7   8    9
```

从上面的料号可以看出以下信息：

ST品牌ARM Cortex-Mx系列内核32位超值型MCU，LQFP -48封装 闪存容量32KB 温度范围-40℃-85℃；

1. 产品系列:
STM32代表ST品牌Cortex-Mx系列内核（ARM）的32位MCU；

2. 产品类型: F：通用快闪（Flash Memory）；
L：低电压（1.65～3.6V）；F类型中F0xx和 F1xx系列为2.0～3.6V; F2xx和F4xx系列为1.8～3.6V;W：无线系统芯片,开发版.

3. 产品子系列：
050：ARM Cortex-M0内核；051：ARM Cortex-M0内核；100：ARM Cortex-M3内核，超值型； 101：ARM Cortex-M3内核，基本型； 102：ARM Cortex-M3内核，USB基本型； 103：ARM Cortex-M3内核，增强型； 105：ARM Cortex-M3内核，USB互联网型； 107：ARM Cortex-M3内核，USB互联网型、以太网型； 108：ARM Cortex-M3内核，IEEE802.15.4标准； 151：ARM Cortex-M3内核，不带LCD； 152/162：ARM Cortex-M3内核，带LCD；
205/207：ARM Cortex-M3内核，不加密模块.（备注：150DMIPS，高达1MB闪存/128+4KB RAM，USB OTG HS/FS，以太网，17个TIM，3个ADC，15个通信外设接口和摄像头；）
215/217：ARM Cortex-M3内核，加密模块。（备注：150DMIPS，高达1MB闪存/128+4KB RAM，USB OTG HS/FS，以太网，17个TIM，3个ADC，15个通信外设接口和摄像头；）
405/407：ARM Cortex-M4内核，不加密模块。（备注：MCU+FPU，210DMIPS，高达1MB闪存/192+4KB RAM，USB OTG HS/FS，以太网，17个TIM，3个ADC，15个通信外设接口和摄像头）；
415/417：ARM Cortex-M4内核，加密模块。（备注：MCU+FPU，210DMIPS，高达1MB闪存/192+4KB RAM，USB OTG HS/FS，以太网，17个TIM，3个ADC，15个通信外设接口和摄像头）；

4. 管脚数:
F：20PIN；G：28PIN；K：32PIN；T：36PIN；H：40PIN；C：48PIN；U：63PIN；R：64PIN；O：90PIN；V：100PINQ：132PIN；Z：144PIN； I：176PIN；

5. Flash存存容量:
4：16KB flash；（小容量）; 6：32KB flash；（小容量）;8：64KB flash；（中容量）;B：128KB flash；（中容量）;C：256KB flash；（大容量）;D：384KB flash；（大容量）;E：512KB flash；（大容量）;F：768KB flash；（大容量）;G：1MKB flash；（大容量）

6. 封装:
T：LQFP；H：BGA；U：VFQFPN；Y：WLCSP/ WLCSP64；

7. 温度范围:
6：-40℃-85℃；（工业级）; 7：-40℃-105℃；（工业级）

8. 内部代码:
“A” or blank; A：48/32脚封装；Blank：28/20脚封装;

9. 包装方式：
TR：带卷； XXX：盘装;D：电压范围1.65V – 3.6V且BOR无使能；无特性：电压范围1.8V – 3.6V且BOR使能；
