---
title: IIC
toc: true
date: 2016-08-08 16:41:15
categories: 总线协议
tags: 总线协议
---
I<sup>2</sup>C总线是一个多主(从)机、单端、串行通信总线，由飞利浦公司(现在的NXP公司)发明。典型应用是那些短距、低速的应用场合。使用I<sup>2</sup>C是不需要许可费的，但是如果使用NXP提供的地址字段，则需要支付相关费用。

## 总线结构
I<sup>2</sup>C是双向开漏两线制总线，由电阻上拉。SDA是其串行数据线，SCL是总线时钟线，用以提供时钟基准。I<sup>2</sup>C地址空间有7bit和10bit两种，大多数I<sup>2</sup>C提供100kbit/s的通信速率，当然这些都不是标准的一部分，视具体应用而定。鉴于通信帧中包含从机地址、从机应答等信息，实际的数据通信速率会比标称的峰值速率低。

最大的从机数目由地址空间的位数决定，同时总线上的电容大小总和受到400PF的限制，这限制了实际的通信距离只有几米，同时为了保证高阻抗和低噪声，一个共同的地也是潜在的需求，这会限制线路板的设计。

## 物理层
在物理层上，SCL与SDA都是开漏设计，因此需要一个上拉电阻，将总线拉低来设置逻辑"0",悬浮时总线被设置成逻辑"1",是一个线与逻辑，总线上的任何一个设备拉低总线都会导致总线为低，因此当总线为低时，必定有其他设备在使用总线，此时总线为`BUSY`状态，因此设备可以依靠`SDA`避免总线冲突。

当总线空闲时，两条线都是高状态，启动一次传输，可以通过拉低SDA而保持SCL为高发出`START`信号，释放SDA而保持SCL为高将产生`STOP`信号。除了`START`与`STOP`信号，SDA需保持其状态在SCL为低时，仅当SCL为高时允许切换SDA状态。
当SCL为低时，由发送器将SDA状态切换至下一状态，然后主机将SCL设置成高电平，一旦SCL为高，接收器读取SDA状态。

## 总线协议(以24Cxx系列EEPROM为例)
I<sup>2</sup>C总线的一个典型应用即是EEPROM。24Cxx系列存储芯片的操作主要有读写两类(有些芯片还有擦除操作，因为比较特殊，同时写操作同样可以达到同样目的，在此按下不表)。读写操作使用同样的通信协议:先写从机地址，接着传输数据直到结束信号到来。
<center><img src="http://obd6jz6in.bkt.clouddn.com/IIC.jpg"></center>

### 写操作
首先主机发送开始信号`START`唤醒总线上的从机，接着发送从机地址(包含写信息)，从机在接收到之后与自己的地址信息进行比较，如果匹配则准备接收数据，否则忽略数据。接着发送需要写入的数据地址及数据，最后发送停止信号。从机在此期间将执行自己的状态机完成数据的接收与写入。*(需要注意的是24Cxx系列的EEPROM没法跨页写，也就是对于跨页写入的数据需要执行两次写周期)*写入期间，从机一只处在`BUSY`状态，对外界一切信息均不作响应，写入完成后从机响应ACK信号。

### 读操作
与写操作类似，首先主机发送开始信号`START`唤醒总线上的从机，接着发送从机地址(包含写信息)，从机在接收到之后与自己的地址信息进行比较，如果匹配则准备接收数据，否则忽略数据。然后再次发送`START`信号与从机地址(包含读信息)，从机在收到后返回相应数据，主机在收到数据后发送应带`ACK`或者`NCK`通知从机接着接收完成或发送下一字节数据，从机发送完最后一个字节数据后，主机可不响应应答信号。

#### 示例代码
```C
//函数声明
void Delay_EEPROM(void);
void EEPROM_I2C_Init(void);
void EEPROM_I2C_Start(void);
void EEPROM_I2C_Stop(void);
unsigned char EEPROM_ReceiveByte( unsigned char ACK );
unsigned char EEPROM_SendByte( unsigned char Data );
unsigned char I2C_Read( unsigned char* RAM_Addr, unsigned int EEPROM_Addr, unsigned char Lenth );
unsigned char I2C_Write( unsigned int EEPROM_Addr, unsigned char* RAM_Addr, unsigned char Lenth );

/***********************************************************************/
/*  函数名称：Delay_EEPROM                                             */
/*  函数功能：延时子程序                                               */
/***********************************************************************/
void Delay_EEPROM(void)
{
    _NOP();_NOP();
    _NOP();_NOP();
}

/***********************************************************************/
/*  函数名称：EEPROM_I2C_Init                                          */
/*  函数功能：I2C管脚初始化                                            */
/***********************************************************************/
void EEPROM_I2C_Init(void)
{
    IO_EEPROM_SCL_OUT_H;
    IO_EEPROM_SDA_OUT_H;
    IO_EEPROM_SCL_DIR_OUT;
    IO_EEPROM_SDA_DIR_OUT;
}

/***********************************************************************/
/*  函数名称：EEPROM_I2C_Start                                         */
/*  函数功能：启动I2C总线                                              */
/*  描述：SCL为高电平时，SDA发出下降沿信号                             */
/***********************************************************************/
void EEPROM_I2C_Start(void)
{
    IO_EEPROM_SDA_OUT_H;      	//SDA=1
    Delay_EEPROM();
    IO_EEPROM_SCL_OUT_H;      	//SCL=1
    Delay_EEPROM();
    IO_EEPROM_SDA_OUT_L;	        //SDA=0
    Delay_EEPROM();
    IO_EEPROM_SCL_OUT_L;      	//SCL=0
    Delay_EEPROM();
}

/***********************************************************************/
/*  函数名称：EEPROM_I2C_Stop                                          */
/*  函数功能：停止I2C总线                                              */
/*  描述：SCL为高电平时，SDA发出上升沿信号                             */
/***********************************************************************/
void EEPROM_I2C_Stop(void)
{
    IO_EEPROM_SDA_OUT_L;      	//SDA=0
    Delay_EEPROM();
    IO_EEPROM_SCL_OUT_H;      	//SCL=1
    Delay_EEPROM();
    IO_EEPROM_SDA_OUT_H;	    //SDA=1
    Delay_EEPROM();
    
    IO_EEPROM_SDA_DIR_OUT;
    IO_EEPROM_SCL_DIR_OUT;
    IO_EEPROM_SDA_OUT_H;
    IO_EEPROM_SCL_OUT_H;
}

/***********************************************************************/
/*  函数名称：EEPROM_ReceiveByte                                       */
/*  函数功能：接收一个字节数据，并发送应答信号                         */
/*  描述：在SCL高电平时，读取SDA信号                                   */
/*  输入参数：ACK：应答信号                                            */
/*  输出参数：读取数据                                                 */
/***********************************************************************/
unsigned char EEPROM_ReceiveByte( unsigned char ACK )
{
    unsigned char i,Data=0x00;
    
    IO_EEPROM_SDA_DIR_IN;     	   //SDA输入
    for( i=0;i<8;i++ )
    {
        IO_EEPROM_SCL_OUT_H;       //SCL=1
        Delay_EEPROM();
        Data = Data<<1;
        if( IO_EEPROM_SDA_IN)
        {
            Data |= 0x01;
        }
        IO_EEPROM_SCL_OUT_L;       //SCL=0
        Delay_EEPROM();
    }
    
    //发送应答位
    IO_EEPROM_SDA_DIR_OUT;     	   //SDA输出
    if( ACK == 1 )
    {
        IO_EEPROM_SDA_OUT_H;	   //SDA=1
    }
    else
    {
        IO_EEPROM_SDA_OUT_L;	   //SDA=0
    }
    IO_EEPROM_SCL_OUT_H;           //SCL=1
    Delay_EEPROM();
    IO_EEPROM_SCL_OUT_L;           //SCL=0
    
    return Data;
}

/***********************************************************************/
/*  函数名称：EEPROM_SendByte                                          */
/*  函数功能：发送一个字节数据或地址，并判断应答信号                   */
/*  描述：在SCL下降沿时，发送SDA信号                                   */
/*  输入参数：Data：待发送的信息                                       */
/*  输出参数：0为成功，1为失败                                         */
/***********************************************************************/
unsigned char EEPROM_SendByte( unsigned char Data )
{
    unsigned char i,ACK;
    
    for( i=0;i<8;i++ )
    {
        if( Data & 0x80 )
        {
            IO_EEPROM_SDA_DIR_IN;
        }
        else
        {
            IO_EEPROM_SDA_DIR_OUT;
            IO_EEPROM_SDA_OUT_L;
        }
        IO_EEPROM_SCL_OUT_H;      	//SCL=1
        Delay_EEPROM();
        Delay_EEPROM();
        IO_EEPROM_SCL_OUT_L;      	//SCL=0
        Data=Data<<1;
    }
    
    //应答位检查
    IO_EEPROM_SDA_DIR_IN;     	//SDA输入
    Delay_EEPROM();
    IO_EEPROM_SCL_OUT_H;          	//SCL=1
    Delay_EEPROM();
    ACK = IO_EEPROM_SDA_IN;	        //正常情况ACK信号应该为0
    IO_EEPROM_SCL_OUT_L;          	//SCL=0
    IO_EEPROM_SDA_DIR_OUT;     	//SDA输出
    Delay_EEPROM();
    
    return ACK;
}

/***********************************************************************/
/*  函数名称：_E2Pread                                                 */
/*  函数功能：从EEPROM里读数据                                         */
/*  输入参数：RAM_Addr：目的地址                                       */
/*            EEPROM_Addr：源地址                                      */
/*            Lenth：数据长度                                          */
/*  输出参数：0为成功，其余为失败                                      */
/***********************************************************************/
unsigned char I2C_Read( unsigned char* RAM_Addr, unsigned int EEPROM_Addr, unsigned char Lenth )
{
    unsigned char ChipAddr,Addr,ACK,i=0;
    
    ClrWdt();
    EEPROM_I2C_Init();
    EEPROM_I2C_Start();
    
    //ChipAddr = 0xAE;
    ChipAddr = (EEPROM_ICAddr | ((EEPROM_Addr>>13)&0x06));
    while( i < 200 )//判断EEPROM能否正常操作
    {
        if( EEPROM_SendByte(ChipAddr) == 0 )//发送器件地址
        {
            break;
        }
        EEPROM_I2C_Stop();
        EEPROM_I2C_Start();
        i++;
    }
    if( i >= 200 )
    {
        EEPROM_I2C_Stop();
        Flag.Error |= F_EEPROM_Err;//器件不正常
        return 0xFF;
    }
    else
        Flag.Error &= ~F_EEPROM_Err;
    
    Addr = ((unsigned char)(EEPROM_Addr>>8) & 0x3F);
    if( EEPROM_SendByte(Addr) )//发送子地址高位
    {
        EEPROM_I2C_Stop();
        Flag.Error |= F_EEPROM_Err;//器件不正常
        return 0xFF;
    }
    Addr = (unsigned char)EEPROM_Addr;
    if( EEPROM_SendByte(Addr) )//发送子地址低位
    {
        EEPROM_I2C_Stop();
        Flag.Error |= F_EEPROM_Err;//器件不正常
        return 0xFF;
    }
    
    EEPROM_I2C_Start();
    ChipAddr |= 0x01;
    if( EEPROM_SendByte(ChipAddr) )//发送器件地址，读命令
    {
        EEPROM_I2C_Stop();
        Flag.Error |= F_EEPROM_Err;//器件不正常
        return 0xFF;
    }
    while( (Lenth--) > 0 )
    {
        if( Lenth != 0 )
            ACK = 0;
        else
            ACK = 1;
        *RAM_Addr = EEPROM_ReceiveByte(ACK);
        RAM_Addr++;
    }
    EEPROM_I2C_Stop();
    
    return 0;
}

/***********************************************************************/
/*  函数名称：_E2Pwrite                                                */
/*  函数功能：往EEPROM里写数据                                         */
/*  输入参数：EEPROM_Addr：目的地址                                    */
/*            RAM_Addr：源地址                                         */
/*            Lenth：数据长度                                          */
/*  输出参数：0为成功，其余为失败                                      */
/***********************************************************************/
unsigned char I2C_Write( unsigned int EEPROM_Addr, unsigned char* RAM_Addr, unsigned char Lenth )
{
    unsigned char ChipAddr,Addr,i=0;
    
    ClrWdt();
    EEPROM_I2C_Init();
    EEPROM_I2C_Start();
    
    //ChipAddr = 0xAE;
    ChipAddr = (EEPROM_ICAddr | ((EEPROM_Addr>>13)&0x06));
    //EEPROM在完成一次写入命令后要延迟5到10毫秒，通过连续发器件地址，一旦总线正常，即立刻进行下一次总线操作
    while( i < 200 )//判断EEPROM能否正常操作
    {
        if( EEPROM_SendByte(ChipAddr) == 0 )//发送器件地址
        {
            break;
        }
        EEPROM_I2C_Stop();
        EEPROM_I2C_Start();
        i++;
    }
    if( i >= 200 )
    {
        EEPROM_I2C_Stop();
        Flag.Error |= F_EEPROM_Err;//器件不正常
        return 0xFF;
    }
    else
        Flag.Error &= ~F_EEPROM_Err;
    
    Addr = ((unsigned char)(EEPROM_Addr>>8) & 0x3F);
    if( EEPROM_SendByte(Addr) )//发送子地址高位
    {
        EEPROM_I2C_Stop();
        Flag.Error |= F_EEPROM_Err;//器件不正常
        return 0xFF;
    }
    Addr = (unsigned char)EEPROM_Addr;
    if( EEPROM_SendByte(Addr) )//发送子地址低位
    {
        EEPROM_I2C_Stop();
        Flag.Error |= F_EEPROM_Err;//器件不正常
        return 0xFF;
    }
    
    do
    {
        EEPROM_SendByte(*RAM_Addr);
        RAM_Addr++;
    }
    while( (--Lenth) > 0 );
    EEPROM_I2C_Stop();
    
    return 0;
}

```
