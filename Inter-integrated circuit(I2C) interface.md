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
该接口可以工作在以下四个模式之一：  
- 从发送器  
- 从接收器  
- 主发送器  
- 主接收器  
默认情况下，它工作在从模式。接口在生成起始位后会自动由从模式切换到主模式；当发生仲裁丢失或产生停止位时，则由主模式切换到从模式；从而实现多主模式功能。  
####通信流程  
在主模式，I2C接口启动数据传送并产生时钟信号。一串数据总是从START位开始传送，以STOP位结束。START位和STOP位都在主模式下由软件产生。  
在从模式，I2C接口要识别它自己的地址（7或10位）和广播呼叫地址。广播呼叫地址的检测由软件使能或禁止。保留的SMBus地址也可以由软件使能。  
数据和地址均以8位字节传输，MSB高位在前。紧跟START位的第1个或2个字节是地址（7位地址一个字节，10位地址两个地址）。地址只在主模式下发送。  
在紧跟发送一个字节的8个时钟周期后的第9个时钟脉冲，接收器必须向发送器发送一个应答位。如下图所示。  
![](https://i.imgur.com/QnNMr0h.png)  
应当位由软件使能或禁止。I2C接口的地址可由软件选择。  
###I2C初始化  
####使能和禁止外设  
I2C外设时钟必须由时钟控制器配置和使能。  
然后I2C可由设置I2C_CR1寄存器中的PE位来使能。  
当I2C被禁止（PE=0），I2C执行软件复位：SDA和SCL被释放。  
####噪声滤波器  
在通过设置I2C_CR1寄存器中的PE位使能I2C外设前，用户必须根据需要配置噪声滤波器。默认情况，模拟噪声滤波器在SDA和SCL输入上是打开的。模拟滤波器遵从I2C规范要求：在快速模式和超快速模式，对脉宽在50ns以内尖峰进行抑制。用户可以通过设置ANFOFF=1禁止使用模拟滤波器。通过I2C_CR1中的DNF[3:0]位来配置选择数字滤波器。  
当数字滤波器被使能后，SCL或SDA引脚上的电平保持稳定的时间超过DNF×I2CCLK周期，才会发生内部变化。它可以抑制脉宽从1到15个I2CCLK周期的脉冲。  
![](https://i.imgur.com/E8zdedP.png)  
警告：当I2C被使能后，不允许改变滤波器配置。  
####I2C时序  
在主模式和从模式下，都必须配置时序，以确保正确的数据保持和建立时间。这由I2C_TIMINGR寄存器中的PRESC[3:0]、SCLDEL[3:0]和SDADEL[3:0]位进行配置。  
STM32CubeMX工具在I2C配置窗口中计算并提供I2C_TIMINGR的内容。  
![](https://i.imgur.com/mKNyyHb.png)  
- 当SCL下降沿被内部检测到，在发送SDA输出前会插入一段延时。这个延时t<sub>SDADEL</sub>=SDADEL×t<sub>PRESC</sub>+t<sub>I2CCLK</sub>，其中t<sub>PRESC</sub>=(PRESC+1)×t<sub>I2CCLK</sub>。  
T<sub>SDADEL</sub>会影响数据保持时间t<sub>HD;DAT</sub>。  
SDA输出总延时：  
t<sub>SYNC1</sub>+{[SDADEL×(PRESC+1)+1]×t<sub>I2CCLK</sub>}  
t<sub>SYNC1</sub>持续时间取决于下面的参数：  
- SCL下降沿斜率  
- 使能模拟滤波器，带来的输入延时：t<sub>AF(min)</sub>＜t<sub>AF</sub>＜t<sub>AF(max)</sub>ns。  
- 使能数字滤波器，带来的输入延时：t<sub>DNF</sub>=DNF×t<sub>I2CCLK</sub>。  
- SCL与I2CCLK建立同步引起的延时（2到3个I2CCLK周期）。  
为了桥接SCL下降沿的未定义区域，必须遵循如下条件配置SDADEL：  
{t<sub>f(max)</sub>+t<sub>HD;DAT(min)</sub>-t<sub>AF(max)</sub>-[(DNF+3)×t<sub>I2CCLK</sub>]}/{(PRESC+1)×t<sub>I2CCLK</sub>}≤SDADEL  
SDADEL≤{t<sub>HD;DAT(max)</sub>-t<sub>AF(max)</sub>-[(DNF+4)×t<sub>I2CCLK</sub>]}/{(PRESC+1)×t<sub>I2CCLK</sub>}  
注：只有使能模拟滤波器后，公式中才包含t<sub>AF(min)</sub>和t<sub>AF(max)</sub>。参考器件数据手册了解t<sub>AF</sub>的值。  
t<sub>HD;DAT</sub>的最大值是3.45us，0.9us和0.45us分别对应标准模式，快速模式和超快模式；但是必须比t<sub>VD;DAT</sub>的最大值少一个跳变时间。该最大值只有器件不延长SCL信号的低电平周期（t<sub>LOW</sub>）时才能得到。如果时钟延长SCL，数据必须在建立时间内保持有效，之后才能释放时钟。  
SDA上升沿通常很糟糕时，SDADEL的计算公式变为：  
SDADEL≤{t<sub>VD;DAT(max)</sub>-t<sub>r(max)</sub>-260ns-[(DNF+4)×t<sub>I2CCLK</sub>]}/{(PRESC+1)×t<sub>I2CCLK</sub>}  
注：当NOSTRETCH=0时，改条件会不成立。这是因为器件会根据SCLDEL的值来延长SCL低电平时间以保证建立时间。  
参考表71：I2C-SMBUS规范数据建立和保持时间，了解t<sub>f</sub>,t<sub>r</sub>,t<sub>HD;DAT</sub>和t<sub>VD;DAT</sub>的标准值。  
- 在延时t<sub>SDADEL</sub>，或发送SDA输出后，从机可能不得不延长时钟，因为数据还没有写入I2C_TXDR寄存器中，SCL在建立时间内保持低电平。建立时间t<sub>SCLDEL</sub>=(SCLDEL+1)×t<sub>PRESC</sub>其中t<sub>PRESC</sub>=(PRESC+1)×t<sub>I2CCLK</sub>。  
t<sub>SCLDEL</sub>会影响建立时间t<sub>SU;DAT</sub>。  
为了桥接SDA跳变（上升沿通常为最差的情况）产生的未定义区域，用户必须编程SCLDEL满足以下条件：  
{[t<sub>r(max)</sub>+t<sub>SU;DAT(min)</sub>]/[(PRESC+1)×t<sub>I2CCLK</sub>]}-1 <= SCLDEL  
参考表71：I2C-SMBUS规范数据建立和保持时间，了解t<sub>r</sub>和t<sub>SU;DAT</sub>的标准值。  
使用的SDA和SCL跳变时间值是应用中的值。使用最大值而非标准值会增加SDADEL和SCLDEL计算的约束条件，但能确保任何应用的特性。  
注：在每个时钟脉冲，SCL下降沿被检测到后，I2C主机或从机在接收或发送模式下要延长SCL低电平至少[(SDADEL+SCLDEL+1)×(PRESC+1)+1]×t<sub>I2CCLK</sub>。在发送模式，为了防止SDADEL计数结束而数据还没有写入I2C_TXDR，I2C保持延长SCL低电平直到下次数据写入。然后新的数据MSB发送到SDA输出上，并且SCLDEL开始计数，继续延长SCL低电平以保证数据建立时间。  
如果在从模式NOSTRETCH=1，SCL不会被延长。因此SDADEL必须以保证足够的建立时间的方式编程。  
![](https://i.imgur.com/zpzEtvU.png)  
另外，在主模式下，SCL时钟的高和低电平必须通过I2C_TIMINGR寄存器中的PRESC[3:0],SCLH[7:0]和SCLL[7:0]来配置。  
- 当内部检测到SCL下降沿，在释放SCL输出前会插入一段延时。这段延时t<sub>SCLL</sub>=(SCLL+1)×t<sub>PRESC</sub>其中t<sub>PRESC</sub>=(PRESC+1)×t<sub>I2CCLK</sub>。t<sub>SCLL</sub>影响SCL低电平时间t<sub>LOW</sub>。  
- 当内部检测到SCL上升沿，在强置SCL输出低电平前会插入一段延时。这段延时t<sub>SCLH</sub>=(SCLH+1)×t<sub>PRESC</sub>其中t<sub>PRESC</sub>=(PRESC+1)×t<sub>I2CCLK</sub>。t<sub>SCLH</sub>影响SCL高电平时间t<sub>HIGH</sub>。  
参考I2C主机初始化，了解详情。  
警告：I2C使能后，不允许更改时序配置。  
I2C从机NOSTRETCH模式必须在外设使能前配置。详见I2C从机初始化。  
警告：I2C使能后，不能更改NOSTRETCH的配置。  
![](https://i.imgur.com/X5ia8Pa.png)  
###软件复位  
将I2C_CR1中的PE位清零，会执行软件复位。这种情况下，I2C的SCL和SDA被释放。内部状态机复位，并且通信控制位和状态位恢复为其复位值。但是，配置寄存器不受影响。  
下面列出了受影响的寄存器位：  
1. I2C_CR2寄存器：START,STOP,NACK  
2. I2C_ISR寄存器：BUSY,TXE,TXIS,RXNE,ADDR,NACKF,TCR,TC,STOPF,BERR,ARLO,OVR  
支持SMBus功能时，还会影响到下列寄存器位：  
1. I2C_CR2寄存器：PECBYTE  
2. I2C_ISR寄存器：PECERR,TIMEOUT,ALERT  
PE必须保持为0至少3个APB时钟周期，以执行软件复位。遵循下面的软件顺序可以确保这一点：-写PE=0-检查PE=0-写PE=1。  
###数据传输  
数据传输由发送和接收数据寄存器以及移位寄存器来管理。  
####接收   
SDA输入填充移位寄存器。在第8个SCL脉冲后（当接收到完整的数据字节时），如果I2C_RXDR寄存器是空的（RXNE=0），移位寄存器的值会被拷贝到I2C_RXDR寄存器中。如果RXNE=1，意味着前面接收到数据还没有被读出，SCL被延长低电平直到I2C_RXDR被读取。在SCL第8个和第9个脉冲之间插入延时（应答脉冲之前）。  
![](https://i.imgur.com/ikFPs1y.png)  
####发送  
如果I2C_TXDR寄存器不为空（TXE=0），其内容将在第9个SCL脉冲（应答脉冲）后被拷贝到移位寄存器。然后移位寄存器的内容被移出到SDA线上。如果TXE=1，意味着I2C_TXDR中没有数据，SCL延长低电平直到I2C_TXDR被写入。这个延时是在第9个SCL脉冲后。  
![](https://i.imgur.com/intDJC0.png)  
####硬件传输管理  
I2C硬件中有一个内嵌的字节计数器，可以在各种模式下管理字节传输和结束通信：  
- 主模式下生成NACK，STOP和ReSTART  
- 从模式下控制ACK是否发出  
- 当支持SMBus功能时，产生/检查PEC  
字节计数器在主模式下总是打开的。而在从模式下，字节计数器默认是关闭的，但是可由软件设置I2C_CR2中的SBC（从机字节控制）位来使能。  
要传输的字节数被编程在I2C_CR2寄存器中的NBYTES[7:0]位。如果要传输的字节数（NBYTES）大于255，或一个接收器要控制是否对接收到的数据字节进行应答，则必须通过设置I2C_CR2中的RELOAD位来选择重载模式。在此模式下，完成NBYTES中所编程字节数的数据传输后，TCR标志被置1，并且如果TCIE=1，还将产生中断。SCL在TCR标志置位期间被延长。当NBYTES被写入非零值时，TCR由软件清零。  
在往NBYTTES中写入最后一次传输的字节数之前，必须把RELOAD位清零。  
当主模式下RELOAD=0，可在下面2种模式下使用字节计数器：  
- 自动结束模式（I2C_CR2中的AUTOEND=1）。该模式下，一旦完成NBYTES[7:0]中编程的字节数的数据传输，主机会自动发送STOP位。  
- 软件结束模式（I2C_CR2中的AUTOEND=0）。该模式下，一旦完成NBYTES[7:0]中编程的字节数的数据传输，TC标志被置1，如果TCIE=1还将产生中断。当软件将I2C_CR2中的START或STOP位置1时，TC标志将被清零。当主机要发送RESTART位，必须使用此模式。  
警告：当RELOAD=1时，AUTOEND位将不起作用。  
![](https://i.imgur.com/YmgIcrB.png)  
###I2C从模式  
####I2C从模式初始化  
要在从模式下工作，用户必须至少使能一个从地址。可以使用I2C_OAR1和I2C_OAR2这两个寄存器编写自身的从地址OA1和OA2。  
- 通过设置I2C_OAR1中的OA1MODE位，可将OA1配置为7位地址（默认）或10位地址。  
通过设置I2C_OAR1中的OA1EN位使能OA1。  
- 如果需要额外的从地址，可以配置第二个从地址OA2。通过配置I2C_OAR2中的OA2MSK[2:0]位，可以屏蔽最多7位OA2的低位。因此OA2MSK配置为1到6，将分别只有OA2[7:2]、OA2[7:3]、OA2[7:4]、OA2[7:5]、OA2[7:6]或OA2[7]与接收到的地址进行比较。只要OA2MSK不等于0，OA2的地址比较器就会排除I2C保留地址（0000XXX和1111XXXX），这些地址不会得到应答。如果OA2MSK=7，接收到的所有7位地址（保留地址除外）都会得到应答。OA2始终为7位地址。  
如果保留的地址由特定的使能位启用，如果它们在I2C_OAR1或I2C_OAR2寄存器中被编程且OA2MSK＝0，则这些保留地址会得到应答。  
通过设置I2C_OAR2中的OA2EN位使能OA2。  
- 广播呼叫地址由I2C_CR1中的GCEN位置位来使能。  
当通过I2C其中一个使能的地址寻址到该I2C设备时，ADDR中断状态标志位将置1，如果ADDRIE=1，还将产生中断。  
默认情况下，从机会使用它的时钟延长功能，这意味着在需要时，从机会延长SCL信号的低电平，以便执行软件操作。如果主机不支持时钟延长，I2C必须配置I2C_CR1中的NOSTRETCH=1。  
在ADDR中断后，如果启用了多个地址，用户必须读取I2C_ISR中的ADDCODE[6:0]位，检查是哪个地址匹配。还要检查DIR标识，以获知传输方向。  
####带时钟延长的从模式（NOSTRETCH=0）  
在默认模式下，I2C从机在以下情况下延长SCL时钟：  
- 当ADDR标志被置1时：接收到的地址与其中一个使能的从地址匹配。当软件将ADDRCF位置1时，ADDR标志会被清零，此时，该延长被释放。  
- 在发送时，如果前面的数据发送已经完成而没有新的数据写入I2C_TXDR寄存器，或者如果在ADDR标志被清零（TXE=1）时没有写入第一个数据。当数据写入I2C_TXDR中时，该延长被释放。  
- 在接收时，当新的数据接收已经完成，而I2C_RXDR中的前次的数据还未读取。当I2C_RXDR被读取后，该延长被释放。  
- 在从字节控制模式和重载模式下（SBC=1,RELOAD=1），当TCR=1时，这意味着最后的数据已经被发送。当通过向NBYTES[7:0]中写入非零值将TCR清零时，该延长被释放。  
- 在SCL下降沿被检测到后，I2C延长SCL低电平时间[(SDADEL+SCLDEL+1)×(PRESC+1)+1]×t<sub>I2CCLK</sub>这么长。  
####不带时钟延长的从模式（NOSTRETCH=1）  
当I2C_CR1寄存器中的NOSTRETCH=1时，I2C从机不会延长SCL信号。  
- 当ADDR标志被置1时，SCL时钟不会被延长。  
- 发送时，必须在对应于其传输的第一个SCL脉冲出现之前，向I2C_TXDR中写入数据。否则，会发生下溢，I2C_ISR中的OVR标志被置1，如果I2C_CR1中的ERRIE=1，还将产生中断。当第一次数据传输开始而STOPF位依然为1（还没被清零）时，OVR标志也会被置1。因此，如果在写入下一次发送的第一个数据之后才清除上一次传输的STOPF标志，则必须确保OVR的状态符合条件，即使对于第一个要发送的数据。  
- 在接收时，必须在下一个数据字节的第9个SCL脉冲（ACK脉冲）出现之前读取I2C_RXDR中的数据。否则，会发生上溢，I2C_ISR中的OVR标志被置1，并且如果I2C_CR1中的ERRIE=1，还将产生中断。  
####从机字节控制模式  
为了在从接收模式下实现字节ACK控制，必须通过I2C_CR1中的SBC位置1使能从机字节控制模式。这样才能与SMBus标准兼容。  
为了在从接收模式下实现字节ACK控制，也必须选择重载模式（RELOAD=1）。要控制每一个字节，必须在ADDR中断子程序中将NBYTES初始化为0x01，并且在每接收一个字节后将NBYTES重载为0x01。接收到字节后，TCR为将被置1，延长第8个和第9个SCL脉冲间的低电平。用户可以从I2C_RXDR中读取数据，然后决定是否对其应答（配置I2C_CR2中的ACK位）。通过向NBYTES写入非零值释放SCL延长：发送应答或不应答信号，然后接收下一个字节。  
可以向NBYTES写入大于0x01的值，在这种情况，会连续接收NBYTES个数据。  
注：配置SBC位必须在一下情况：I2C未使能，从机没有被寻址到，或ADDR=1。  
警告：从机字节控制模式和不带时钟延长模式，两者不兼容。当NOSTRETCH=1时不允许置位SBC。  
![](https://i.imgur.com/d3bsxUI.png)  
######I2C configured in slave mode code example  

    /* (1) Timing register value is computed with the AN4235 xls file,
           fast Mode @400kHz with I2CCLK = 48MHz, rise time = 140ns, fall time = 40ns */
    /* (2) Periph enable, address match interrupt enable */
    /* (3) 7-bit address = 0x5A */
    /* (4) Enable own address 1 */
    I2C1->TIMINGR = (uint32_t)0x00B00000; /* (1) */
    I2C1->CR1 = I2C_CR1_PE | I2C_CR1_ADDRIE; /* (2) */
    I2C1->OAR1 |= (uint32_t)(I2C1_OWN_ADDRESS << 1); /* (3) */
    I2C1->OAR1 |= I2C_OAR1_OA1EN; /* (4) */  
####从发送器  
当I2C_TXDR寄存器为空时，将产生发送中断状态（TXIS）。如果I2C_CR1中的TXIE=1，将产生中断。  
当I2C_TXDR寄存器中写入下一个要发送的数据时，TXIS位被清除。  
当收到一个NACK时，I2C_ISR中的NACKF位被置1，并且如果I2C_CR1中的NACKIE=1还将产生中断。从机自动释放SCL和SDA，以便主机可以执行停止和重复起始位的发送。当收到NACK时，TXIS不会被置1。  
当收到停止位并且I2C_CR1中的STOPIE=1时，I2C_ISR中的STOPF标志被置1并且产生中断。在大多数应用中，SBC位通常设置为0。在这种情况下，如果当TXE=0时接收到从机地址（ADDR=1），用户可以选择将I2C_TXDR寄存器中的内容作为第一个字节发送出去，或是，将TXE位置1从而清空I2C_TXDR，然后向其中写入新的数据。  
在从机字节控制模式（SBC=1），在地址匹配（ADDR=1）中断子程序中必须在NBYTES中编写要传输的字节数。在这种情况下，传输期间TXIS事件的数量对应于NBYTES中编写的值。  
警告：当NOSTRETCH=1时，在ADDR=1时，SCL时钟不会延长，这样用户就不能在ADDR子程序中来刷新I2C_TXDR以写入第一个数据字节。第一个要发送的数据字节必须提前写入I2C_TXDR寄存器中：  
- 该数据可以是上一次传输信息的最后一个TXIS事件中写入的数据。  
- 如果这个数据字节不是要发送的那个，可以通过设置TXE=1来清空I2C_TXDR，以写入新的数据字节。STOPF位必须在这些操作后才能被清零，以保证这些操作紧跟地址应答但在第一次数据传输开始前被执行。  
如果当第一次数据传输开始后STOPF位仍为1，将产生下溢错误（OVR标志置1）。  
如果需要一个TXIS事件（发送中断或DMA请求），用户必须将TXE位和TXIS位都置1，才会产生TXIS事件。  
![](https://i.imgur.com/9BvxAsi.png)  
![](https://i.imgur.com/gaB78XM.png)  
![](https://i.imgur.com/6Du5zUl.png)  
######I2C slave transmitter code example  

    uint32_t I2C_InterruptStatus = I2C1->ISR; /* Get interrupt status */
    /* Check address match */
    if ((I2C_InterruptStatus & I2C_ISR_ADDR) == I2C_ISR_ADDR)
    {
        I2C1->ICR |= I2C_ICR_ADDRCF; /* Clear address match flag */
        /* Check if transfer direction is read (slave transmitter) */
        if ((I2C1->ISR & I2C_ISR_DIR) == I2C_ISR_DIR)
        {
            I2C1->CR1 |= I2C_CR1_TXIE; /* Set transmit IT */
        }
    }
    else if ((I2C_InterruptStatus & I2C_ISR_TXIS) == I2C_ISR_TXIS)
    {
        I2C1->CR1 &=~ I2C_CR1_TXIE; /* Disable transmit IT */
        I2C1->TXDR = I2C_BYTE_TO_SEND; /* Byte to send */
    }  
####从接收器  
当I2C_RXDR装满时，I2C_ISR中的RXNE被置1。当读取I2C_RXDR时，RXNE被清零。  
当收到停止位并且I2C_CR1中的STOPIE=1时，I2C_ISR中的STOPF位被置1，并产生中断。  
![](https://i.imgur.com/mWb7cx8.png)  
![](https://i.imgur.com/NIblP7H.png)  
![](https://i.imgur.com/ev3Jv3T.png)  
######I2C slave receiver code example  

    uint32_t I2C_InterruptStatus = I2C1->ISR; /* Get interrupt status */
    if ((I2C_InterruptStatus & I2C_ISR_ADDR) == I2C_ISR_ADDR)
    {
        I2C1->ICR |= I2C_ICR_ADDRCF; /* Address match event */
    }
    else if ((I2C_InterruptStatus & I2C_ISR_RXNE) == I2C_ISR_RXNE)
    {
        /* Read receive register, will clear RXNE flag */
        if (I2C1->RXDR == I2C_BYTE_TO_SEND)
        {
            /* Process */
        }
    }  
###I2C主模式  
####I2C主模式初始化  
在使能外设前，必须通过设置I2C_TIMINGR寄存器中的SCLH和SCLL位配置好I2C主机时钟。  
STM32CubeMX工具在I2C配置窗口计算并提供I2C_TIMINGR内容。  
为了支持多主机情况和从机延长时钟，提供了时钟同步机制。  
为了让时钟同步：  
- 从SCL低电平被内部检测到开始，使用SCLL计数器对时钟低电平进行计数。  
- 从SCL高电平被内部检测到开始，使用SCLH计数器对时钟高电平进行计数。  
I2C经过延时t<sub>SYNC1</sub>后检测自身的SCL低电平，该延时取决于SCL下降沿、SCL输入噪声滤波器（模拟+数字）和SCL与I2CxCLK时钟的同步。一旦SCLL计数器达到I2C_TIMINGR寄存器中SCLL[7:0]编程的值，I2C便释放SCL变为高电平。  
I2C经过延时t<sub>SYNC2</sub>后检测自身的SCL高电平，该延时取决于SCL上升沿、SCL输入噪声滤波器（模拟+数字）和SCL与I2CxCLK时钟的同步。一旦SCLH计数器达到I2C_TIMINGR寄存器中SCLH[7:0]编程的值，I2C就会将SCL拉到低电平。  
因此，主机时钟周期是：  
t<sub>SCL</sub> = t<sub>SYNC1</sub> + t<sub>SYN2</sub> + {[(SCLH+1) + (SCLL+1)] × (PRESC+1) × t<sub>I2CCLK</sub>}  
t<sub>SYNC1</sub>的持续时间取决于以下参数：  
- SCL下降沿  
- 由于使用模拟滤波器，带来的输入延时  
- 由于使用数字滤波器，带来的输入延时：DNF×t<sub>I2CCLK</sub>  
- SCL与I2CCLK时钟进行同步，引起的延时（2到3个I2CCLK时钟周期）  
t<sub>SYNC2</sub>的持续时间取决于以下参数：  
- SCL上升沿  
- 使用模拟滤波器，引入的输入延时  
- 使用数字滤波器，引入的输入延时：DNF×t<sub>I2CCLK</sub>  
- SCL与I2CCLK时钟同步，造成的延时（2到3个I2CCLK时钟周期）  
![](https://i.imgur.com/LrLgape.png)  
警告：为了符合I2C或SMBus规范，主机时钟必须遵循下表的时序：  
![](https://i.imgur.com/vfpGxT0.png)  
注：SCLL也被用于产生t<sub>BUF</sub>和t<sub>SU:STA</sub>时序。SCLH也被用于产生t<sub>HD:STA</sub>和t<sub>SU:STO</sub>。  
####主机通信初始化（地址阶段）  
要启动通信，用户必须在I2C_CR2寄存器中为要寻址的从机编程以下参数：  
- 地址模式（7位或10位）：ADD10  
- 要发送的从机地址：SADD[9:0]  
- 传输方向：RD_WRN  
- 在10位地址读取的情况：HEAD10R位。必须配置HEAD10R位，以指示当传输方向改变时，是必须发送完整的地址序列，还是只要发送地址头就可以。  
- 要发送的字节数：NBYTES[7:0]。如果字节数大于等于255，必须将NBYTES[7:0]初始化为0xFF。  
然后，用户需要将I2C_CR2中的START位置1。当START=1时，上述所有位不允许更改。  
一旦主器件检测到总线空闲（BUSY=0），经过t<sub>BUF</sub>的延时后，主器件自动发送起始位，随后发送从器件地址。  
发生仲裁丢失时，主器件自动切换回从模式，并且如果作为从器件被寻址，还会对自身地址进行应答。  
注：当从机地址已发送到总线上，I2C_CR2中的START位由硬件自动清零，而不管接收到的应答值如何。如果发送仲裁丢失，START位也会被硬件清零。  
在10位地址模式，当首先发送的前7位地址，从器件NACK，主器件会自动重启从地址发送，直到收到从器件发来的ACK。在这里，接收到从器件发来的NACK后，主器件会将ADDRCF置1，以停止从地址发送。  
当START=1时，I2C作为从器件被寻址到（ADDR=1），I2C会从主模式切换为从模式，并且START位在ADDRCF位被置1时被清零。  
注：该步骤同样适用于重复起始位。在这种情况，BUSY=1。  
![](https://i.imgur.com/sR8KzWy.png)  
######I2C configured in master mode to receive code example  

    /* (1) Timing register value is computed with the AN4235 xls file,
           fast Mode @400kHz with I2CCLK = 48MHz, 
           rise time = 140ns, fall time = 40ns */
    /* (2) Periph enable, receive interrupt enable */
    /* (3) Slave address = 0x5A, read transfer, 1 byte to receive, autoend */
    I2C2->TIMINGR = (uint32_t)0x00B01A4B; /* (1) */
    I2C2->CR1 = I2C_CR1_PE | I2C_CR1_RXIE; /* (2) */
    I2C2->CR2 = I2C_CR2_AUTOEND | (1<<16) | I2C_CR2_RD_WRN | (I2C1_OWN_ADDRESS << 1); /* (3) */  
######I2C configured in master mode to transmit code example  

    /* (1) Timing register value is computed with the AN4235 xls file,
           fast Mode @400kHz with I2CCLK = 48MHz, 
           rise time = 140ns, fall time = 40ns */
    /* (2) Periph enable */
    /* (3) Slave address = 0x5A, write transfer, 1 byte to transmit, autoend */
    I2C2->TIMINGR = (uint32_t)0x00B01A4B; /* (1) */
    I2C2->CR1 = I2C_CR1_PE; /* (2) */
    I2C2->CR2 = I2C_CR2_AUTOEND | (1 << 16) | (I2C1_OWN_ADDRESS << 1); /* (3) */   
####主接收器寻址一个10位地址从器件的初始化  