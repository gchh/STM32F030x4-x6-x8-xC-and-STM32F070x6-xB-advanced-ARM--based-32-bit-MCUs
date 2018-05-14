#Inter-integrated circuit(I2C) interface  
##简介  
I<sup>2</sup>C总线接口处理微控制器和串行I<sup>2</sup>C总线间的通信。它提供多主机功能，可以控制所有I<sup>2</sup>C总线特定的时序、协议、仲裁和定时。它支持标准模式（Sm），快速模式（Fm）和超快速模式（Fm+）。  
它还与SMBus（系统管理总线）和PMBus（电源管理总线）兼容。  
可以使用DMA减轻CPU负担。  
##I<sup>2</sup>C主要特性  
- 兼容I<sup>2</sup>C总线规范03版：  
　- 从和主模式  
　- 多主机功能  
　- 标准模式（高达100kHz）  
　- 快速模式（高达400kHz）  
　- 超快速模式（高达1MHz）  
　- 7位和10位地址模式  
　- 多个7位从地址（2个从地址，其中一个可屏蔽）  
　- 所有7位地址应答模式  
　- 广播呼叫  
　- 总线上的数据建立和保持时间可编程  
　- 易用的事件管理  
　- 软件复位  
- 带DMA功能的1字节缓冲  
- 可编程的模拟和数字噪声滤波器  
根据产品实现，还可以提供以下附加功能：  
- 兼容SMBus规范2.0版：  
　- 带ACK控制的硬件PEC（数据包错误校验）生成和验证  
　- 命令和数据应答控制  
　- 支持地址解析协议（ARP）  
　- 支持主机和从设备  
　- SMBus报警  
　- 超时和空闲状态检测  
- 兼容PMBus标准1.1版本  
- 独立时钟：选择独立时钟源可以使I2C通信速度不受PCLK更改的影响  
##I2C实现  
本手册描述了I2C1实现的所有特性。I2C2实现的功能有所减少。主要区别如下表。  
![](https://i.imgur.com/cftLN8o.png)  
##I2C功能描述  
除了接收和发送数据，此接口还可以从串行格式转换为并行格式，反之亦然。中断由软件使能或禁止。该接口通过一个数据引脚SDA和一个时钟引脚SCL连接到I<sup>2</sup>C总线。它可以连接标准速度（高达100kHz），快速（高达400kHz）或超快速（高达1MHz）的I<sup>2</sup>C总线。  
该接口也可以通过一个数据引脚SDA和一个时钟引脚SCL连接到SMBus。  
如果支持SMBus功能：还可以使用过额外可选的SMBus报警引脚SMBA。  
###I<sup>2</sup>C框图  
![](https://i.imgur.com/T0Ns2YL.png)  
I2C由独立时钟源提供时钟，使其能独立于PCLK工作。  
独立时钟源可以在下列2个时钟源中选择：  
- HSI：内部高速时钟（默认）  
- SYSCLK：系统时钟  
I2C的I/O口支持20mA输出电流驱动以适应超快速模式的操作。通过设置SYSCFG_CFGR1寄存器中的SCL和SDA驱动能力控制位来使能。  
###I<sup>2</sup>C2框图  
![](https://i.imgur.com/2tNS6z4.png)  
###I2C时钟要求  
I2C内核时钟由I2CCLK提供。  
I2CCLK的周期t<sub>I2CCLK</sub>必须满足以下条件：  
t<sub>I2CCLK</sub> < (t<sub>LOW</sub>-t<sub>filters</sub>)/4并且t<sub>I2CCLK</sub> < t<sub>HIGH</sub>  
其中：  
t<sub>LOW</sub>：SCL低电平时间  
t<sub>HIGH</sub>：SCL高电平时间  
t<sub>filters</sub>：滤波器使能时，由模拟和数字滤波器引起的延时的总和。模拟滤波器最大延时是260ns。数字滤波器延时是DNF×t<sub>I2CCLK</sub>。  
PCLK时钟周期t<sub>PCLK</sub>必须满足下列条件：  
t<sub>PCLK</sub> < 4/3 t<sub>SCL</sub>  
其中t<sub>SCL</sub>：SCL周期  
警告：当I2C内核时钟有PCLK提供时，PCLK必须满足t<sub>I2CCLK</sub>的条件。  
###模式选择  
