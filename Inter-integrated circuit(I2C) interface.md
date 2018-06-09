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
- 如果从地址是10位格式，可以选择将I2C_CR2中的HEAD10R位清零来发送完整的读序列。这种情况下，主器件在START位置1后自动发送下面的完整序列：(Re)Start + Slave address 10-bit header Write + Slave address 2nd byte +REStart + Slave address 10-bit header Read  
![](https://i.imgur.com/ZjmjnC9.png)  
- 如果主器件要寻址一个10位地址的从器件，向该器件发送数据然后读取该从器件发回的数据，必须首先完成主器件的发送流程。然后，重复起始位置1，10为从地址配置为HEAD10R=1。在这种情况下，主器件发送序列为：ReStart + Slave address 10-bit header Read  
![](https://i.imgur.com/PHuIA4b.png)  
####主器件发送  
在写传输的情况下，在每个字节发送完后，也就是在第9个SCL脉冲收到ACK后，TXIS标志会被置1。  
如果I2C_CR1寄存器中的TXIE位等于1，TXIS事件还会产生一个中断。当I2C_TXDR寄存器写入将要发送的下个数据时，TXIS标志被清零。  
传输过程中发生的TXIS事件个数对应于NBYTES[7:0]编程的值。如果要发送的数据字节总数大于255，必须将I2C_CR2中的RELOAD位置1来选择重载模式。在这种情况，当NBYTES个字节的数据被发送后，TCR标志被置1，并且SCL被拉低直到NBYTES[7:0]写入非零值。  
当收到NACK时，TXIS标志不会被置1：  
- 当RELOAD=0且NBYTES个数据发送完成：  
　- 在自动结束模式（AUTOEND=1），自动发送停止位。  
　- 在软件结束模式（AUTOEND=0），TC标志被置1并且SCL被拉低以便执行软件操作：  
　如果已经配置好适当的从地址和要传输的字节数，将I2C_CR2中的START位置1，来请求发送重复起始位。将START位置1，会将TC标志清零，并在总线上发送起始位。  
　将I2C_CR2中的STOP位置1，来请求发送停止位。将STOP位置1，会将TC标志清零，并在总线上发送停止位。  
- 如果收到NACK：TXIS标志不会置位，并在接收到NACK后自动发送停止位。I2C_ISR寄存器中的NACKF标志置1，如果NACKIE置1，还将产生中断。  
![](https://i.imgur.com/1snEFm1.png)  
![](https://i.imgur.com/3utA0UP.png)  
![](https://i.imgur.com/JVl3KgO.png)  
######I2C master transmitter code example  

    /* Check Tx empty */
    if ((I2C2->ISR & I2C_ISR_TXE) == I2C_ISR_TXE)
    {
        I2C2->TXDR = I2C_BYTE_TO_SEND; /* Byte to send */
        I2C2->CR2 |= I2C_CR2_START; /* Go */
    }  
####主器件接收  
在读传输时，在接收到每一个字节后（即第8个SCL脉冲后）RXNE标志被置1。如果I2C_CR1中的RXIE位被置位，RXNE事件还将产生中断。读取I2C_RXDR时，RXNE标志被清零。  
如果要接收的数据字节总数大于255，必须将I2C_CR2中的RELOAD位置1，选择重载模式。在这种情况下，当NBYTES[7:0]个数据传输完成，TCR标志被置1，而且SCL被拉低直到NBYTES[7:0]写入非零值。  
- 当RELOAD=0并且NBYTES[7:0]个数据已经传输完成：  
　- 自动结束模式（AUTOEND=1），接收完最后一个字节数据，自动发送NACK和停止位。  
　- 软件结束模式（AUTOEND=0），接收完最后一个字节，自动发送NACK，TC标志被置1，SCL被拉低，以便执行下面的软件操作：  
　可以通过将I2C_CR2中的START位置1，并配置适当的从地址和传输的字节数，来请求发送重复起始位。将START位置1，会将TC标志清零，并在总线上发送起始位，其后是从地址。  
　通过将I2C_CR2中的STOP位置1，来请求发送停止位。STOP位置1，会将TC标志清零，并在总线上发送停止位。  
![](https://i.imgur.com/rit4s8S.png)  
![](https://i.imgur.com/pwxk9Nu.png)  
![](https://i.imgur.com/X8dtvT5.png)  
######I2C master receiver code example  

    if ((I2C2->ISR & I2C_ISR_RXNE) == I2C_ISR_RXNE)
    {
        /* Read receive register, will clear RXNE flag */
        if (I2C2->RXDR == I2C_BYTE_TO_SEND)
        {
            /* Process */
        }
    }  
###I2C_TIMINGR寄存器配置示例  
下面的表格提供了如何配置I2C_TIMINGR以获得符合I2C规范的时序的示例。为了获得更精确的配置值，请参考应用笔记：I2C时序配置工具(AN4235)和相关软件STSW-STM32126。  
![](https://i.imgur.com/2rcbgqi.png)  
![](https://i.imgur.com/zoRvFoA.png)  
![](https://i.imgur.com/HOe7QLW.png)  
###SMBus特性  
该部分只针对支持SMBus功能的器件。  
####简介  
系统管理总线（SMBus）是一个双线接口，通过它各个器件之间或与系统其余部分进行通信。它是以I2C操作原理为基础。SMBus为系统和电源管理相关的任务提供了一种控制总线。  
该外设和SMBus规范2.0版兼容（[http://smbus.org](http://smbus.org)）。  
系统管理总线规范针对三种类型的器件。  
- 从器件：接收或响应命令。  
- 主器件：发出命令，产生时钟和终止传输。  
- HOST主机：一个特殊的主器件，提供连接系统CPU的主接口。HOST必须具备主-从器件的功能，并且支持SMBus主机通知协议。一个系统中只允许一个HOST。  
该外设可以配置成主器件或从器件，甚至HOST。  
####SMBUS基于I2C规范2.1版  
####总线协议  
任何给定器件都有十一种可用命令协议。一个器件可以使用任意或全部的协议用于通信。协议分别是：快速命令、发送字节、接收字节、写入字节、写入字、读取字节、读取字、进程调用、块读取、块写入以及块写入-块读取进程调用。这些协议应该由用户软件实现。  
这些协议的更多详细内容，请参考SMBus规范2.0版本（[http://smbus.org](http://smbus.org)）。  
####地址解析协议（ARP）  
SMBus从地址冲突问题可以通过为每个从器件动态分配一个新的唯一的地址来解决。为了分配地址，需要一种机制来区分每个器件，即每个器件必须拥有唯一的器件标识符（UDID）。这个128位的UDID由软件来实现。  
该外设支持地址解析协议（ARP）。SMBus器件的默认地址（0b1100 001）通过I2C_CR1中的SMBDEN位置1来使能。ARP命令应该由用户软件实现。  
在从模式下也将执行仲裁，以支持ARP。  
SMBus地址解析协议更多详细内容，请参考SMBus规范2.0版（[http://smbus.org](http://smbus.org)）。  
####接收命令和数据应答控制  
SMBus接收器必须能对接收到的命令或数据回应NACK。为了在从模式下实现ACK控制，必须将I2C_CR1中的SBC位置1使能从字节控制模式。  
####Host通知协议  
该外设支持HOST通知协议，当需要将I2C_CR1中的SMBHEN位置1。这种情况下，HOST会应答SMBus HOST地址（0b0001 000）。  
使用该协议时，器件作为主器件，HOST作为从器件。  
####SMBus报警  
支持SMBus报警信号。从器件可以通过SMBALERT引脚向HOST发出信号，提示从器件想要通信。HOST会处理报警中断，并通过报警响应地址（0b0001 100）同时访问所有SMBALERT器件。只有那些把SMBALERT引脚拉低的器件会应答报警响应地址。  
如果配置为从设备（SMBHEN=0），通过将I2C_CR1寄存器中的ALERTEN位置1来拉低SMBA引脚。同时使能报警响应地址。  
如果配置为HOST（SMBHEN=1），当在SMBA引脚上检测到下降沿且ALERTEN=1时，I2C_ISR中的ALERT标志被置1。如果I2C_CR1中的ERRIE位为1，此时还会产生中断。如果ALERTEN=0，即使外部SMBA引脚被拉低，内部ALERT也会被认为是高电平。  
如果不需要使用SMBus ALERT且ALERTEN=0，SMBA引脚可以用作标准GPIO。  
####包错误检测  
SMBus规范中引入包错误检测机制，以提高可靠性和通信鲁棒性。包错误检测是通过在每次消息传输结尾增加一个包错误码（PEC）来实现的。PEC是对所有信息字节（包括地址和读/写位）使用CRC-8多项式C(x) = x<sup>8</sup> + x<sup>2</sup> + x + 1计算得到的。  
该外设内嵌有一个硬件PEC计算单元，当接收到的字节和硬件计算的PEC不匹配时，会自动发送NACK。  
####超时  
该外设内嵌一个硬件定时器，以符合SMBus规范2.0版定义的3个超时。  
![](https://i.imgur.com/byJ9gY8.png)  
![](https://i.imgur.com/uWTdUDW.png)  
####总线空闲检测  
如果主器件检测到时钟和数据信号保持高电平的时间t<sub>IDLE</sub>大于t<sub>HIGH,MAX</sub>，就可以认为总线是空闲的。  
该定时参数考虑到了如下情况，一个主器件被动态地添加到总线上，可能没检测到SMBCLK或SMBDAT上状态转换。在这种情况下，主器件必须等待足够长，以确保当前没有传输在进行。该外设支持硬件总线空闲检测。  
###SMBus初始化  
该部分只针对支持SMBus功能的器件。  
为了执行SMBus通信，除了I2C初始化外，一些特定的初始化也要进行。  
####接收命令和数据应答控制（从模式）  
SMBus接收器必须对每个接收到的命令或数据回应NACK。为了在从模式下实现ACK控制，必须将I2C_CR1中的SBC位置1，使能从字节控制模式。  
####特定地址（从模式）  
必要时可以使能特定的SMBus地址。  
- I2C_CR1中的SMBDEN=1，使能SMBus器件默认地址（0b1100 001）。  
- I2C_CR1中的SMBHEN=1，使能SMBus Host地址（0b0001 000）。  
- I2C_CR1中的ALERTEN=1，使能报警响应地址（0b0001 100）。  
####包错误检测  
I2C_CR1中的PECEN位置1，使能PEC计算。借助I2C_CR2中的NBYTES[7:0]来管理PEC传输。必须在I2C使能之前配置PECEN位。  
PEC传输由硬件字节计数器管理，因此SMBus在从模式下必须将SBC位置1。当PECBYTE=1且RELOAD=0，在传输完NBYTES-1个字节的数据后，传输PEC。如果RELOAD=1，PECBYTE将不起作用。  
警告：当I2C已经使能后，不能更改PECEN的配置。  
![](https://i.imgur.com/j4255No.png)  
####超时检测  
I2C_TIMEOUTR中的TIMOUTEN位和TEXTEN位置1，使能超时检测。定时器必须按如下方式编程：即在SMBus规范2.0版规定的最大时间之前检测出超时。  
- t<sub>TIMEOUT</sub>检测  
　为了使能t<sub>TIMEOUT</sub>检测，必须将要检查的t<sub>TIMEOUT</sub>参数对应的定时器重载值写入12位的TIMEOUTA[11:0]。TIDLE位必须清零，以便检测SCL低电平超时。  
　然后将I2C_TIMEOUTR中的TIMOUTEN位置1，使能定时器。如果SCL低电平持续时间超过（TIMEOUTA+1）×2048×t<sub>I2CCLK</sub>，I2C_ISR中的TIMEOUT标志被置1。  
警告：当TIMEOUTEN位置1后，不允许更改TIMEOUTA[11:0]和TIDLE位的配置。  
- t<sub>LOW:SEXT</sub>和t<sub>LOW:MEXT</sub>检测  
　取决于外设配置成主模式还是从模式，配置12位的TIMEOUTB定时器检测t<sub>LOW:SEXT</sub>或t<sub>LOW:MEXT</sub>。因为规范只规定了最大值，所以用户可以为这两个参数选择与之相同的值。  
　然后将I2C_TIMEOUTR中的TEXTEN位置1，使能定时器。如果SMBus外设执行SCL延展的积累时间超过了（TIMEOUTB+1）×2048×t<sub>I2CCLK</sub>并且处在总线空闲检测一节描述的超时间隔中，I2C_ISR中的TIMEOUTT标志被置1。  
警告：当TEXTEN位置1后，TIMEOUTB的配置不允许更改。  
####总线空闲检测  
为了使能t<sub>IDLE</sub>检测，12位的TIMEOUTA[11:0]必须被编程位定时器重载值，以获得t<sub>IDLE</sub>参数。TIDLE位必须置1，为了检测SCL和SDA上的高电平超时。  
然后，I2C_TIMEOUTR中的TIMOUTEN位置1，使能定时器。  
如果SCL和SDA两者保持高电平的时间都超过了（TIMEOUTA+1）×4×t<sub>I2CCLK</sub>，I2C_ISR中的TIMEOUT标志会被置1。  
警告：当TIMEOUTEN位置1后，TIMEOUTA和TIDLE的配置不能更改。  
###SMBus：I2C_TIMEOUTR寄存器配置示例   
- 配置t<sub>TIMEOUT</sub>最长为25ms：  
![](https://i.imgur.com/gUtHHEE.png)  
- 配置t<sub>LOW:SEXT</sub>和t<sub>LOW:MEXT</sub>最长为8ms：  
![](https://i.imgur.com/RTAynbt.png)  
- 配置t<sub>IDLE</sub>最长为50us：  
![](https://i.imgur.com/XaCVdpi.png)  
###SMBus从模式  
该部分只针对支持SMBus功能的器件。  
除了I2C从模式传输管理还需要执行一些额外的软件流程以支持SMBus。  
####SMBus从发送器  
当在SMBus下使用IP时，SBC必须设置为1，以允许在编程的数据字节发送完后发送PEC。当PECBYTE位被置1，NBYTES[7:0]中编程的字节数要包含PEC传输。在这种情况下，TXIS中断的总数是NBYTES-1，如果主器件在NBYTES-1个数据字节传输后请求额外的字节，I2C_PECR寄存器的内容将被自动发送出。  
警告：当RELOAD位置1时，PECBYTE位不起作用。  
![](https://i.imgur.com/k9eIUg8.png)  
![](https://i.imgur.com/JOi3wk8.png)  
####SMBus从接收器  
当在SMBus模式下使用I2C时，SBC必须被置1，以允许在完成编程的数据字节数传输后进行PEC校验。为了对每个字节进行ACK控制，RELOAD位必须置1，选择重载模式。  
为了校验PEC字节，RELOAD位必须被清零，而PECBYTE位置1。在这种情况下，在接收到NBYTES-1个数据后，接收的下一个字节和内部I2C_PECR寄存器的内容进行比较。如果比较不匹配，自动发送一个NACK；比较匹配，则发送ACK。PEC字节接收到后，和其他数据一样被拷贝到I2C_RXDR寄存器中，并且RXNE标志置1。  
如果PEC不匹配，PECERR标志会置1，并且如果I2C_CR1中的ERRIE位为1的话，还将产生中断。  
如果不需要ACK软件控制，用户可以编程PECBYTE=1，在同样的写操作下，将NBYTES设置为连续接收的字节数。在接收到NBYTES-1个字节后，接收到的下一个字节被作为PEC来校验。  
警告：当RELOAD位置1时，PECBYTE位不起作用。  
![](https://i.imgur.com/Bf3GmCn.png)  
![](https://i.imgur.com/ejknHEJ.png)  
####SMBus主发送器  
当SMBus主器件要发送PEC时，在START位置1之前，必须设置PECBYTE位为1，并在NBYTES[7:0]中写入要发送的字节数。这种情况下，TXIS中断的总数是NBYTES-1。因此，当NBYTES=0x01时PECBYTE位置1，I2C_PECR寄存器的内容将被自动发送。  
如果SMBus主器件在PEC之后要发送停止位，就需要选择自动结束模式（AUTOEND=1）。这种情况下，发送PEC后，将自动发送STOP。  
如果SMBus主器件在PEC后要发送重复起始位，就要选择软件结束模式（AUTOEND=0）。这种情况下，发送完NBYTES-1个字节后，自动发送I2C_PECR寄存器的内容，并在PEC发送完后将TC标志置1，SCL被拉伸低电平。必须在TC中断服务子程序中，设置RESTART条件。  
警告：RELOAD=1时，PECBYTE位不起作用。  
![](https://i.imgur.com/LL6KSKN.png)  
####SMBus主接收器  
当SMBus主器件要在接收PEC之后发送STOP时，要选择自动结束模式（AUTOEND=1）。在START位置1之前，必须将PECBYTE位置1，并且设置从地址。这种情况下，在接收到NBYTES-1个字节数据后，下一个接收到的字节自动和I2C_PECR寄存器的内容进行比较。对接收到的PEC，主器件将发送NACK作为响应，跟着发送STOP。  
当SMBus主接收器在接收到PEC后，要发送重复起始位时，必须选择软件结束模式（AUTOEND=0）。在START位置1之前，PECBYTE位必须置1，并设置从地址。这种情况下，在接收到NBYTES-1个字节数据后，接收到的下一个字节将和I2C_PECR寄存器的内容自动进行比较。在接收到PEC后，TC标志被置1，SCL被拉伸低电平。RESTART条件可以在TC中断子程序中设置。  
警告：RELOAD=1时，PECBYTE位不起作用。  
![](https://i.imgur.com/rnqrpJ9.png)  
###错误条件  
以下是可能导致通信失败的错误条件。  
####总线错误（BERR）  
当不是在第9的倍数个SCL时钟脉冲之后，检测到START或STOP时，就发生了总线错误。在SCL为高电平时，SDA上出现边沿，就会检测到START或STOP。  
仅当I2C作为主器件或被寻址的从器件处于传输过程中时，总线错误标志才会置1。例如，在从模式下的寻址阶段，总线错误标志不会被置1。  
在从模式下发生START或RESTART错位时，I2C会像接收到正确的START一样进入地址识别阶段。  
当检测到总线错误，I2C_ISR寄存器中的BERR标志会被置1，如果I2C_CR1中的ERRIE位设置为1，还将产生中断。  
####仲裁丢失（ARLO）  
如果SDA上发送的是高电平，但在SCL上升沿采样SDA得到的却是低电平，就检测到一个仲裁丢失。  
- 在主模式下，如果仲裁丢失在地址阶段、数据阶段和应答阶段被检测到，SDA和SCL都会被释放，START位被硬件清零，并且主模式被自动切换为从模式。  
- 在从模式下，如果在数据阶段和应答阶段检测到仲裁丢失，会停止传输，SCL和SDA被释放。  
当检测到仲裁丢失时，I2C_ISR中的ARLO标志被置1，如果I2C_CR1中的ERRIE位被置1，还会产生中断。  
####上溢/下溢错误（OVR）  
在从模式下，当NOSTRETCH=1，并出现下列情况时，会检测到上溢或下溢错误：  
- 在接收时，RXDR寄存器还未被读取，就接收到一个新字节。新接收到的字节会丢失，并自动发送一个NACK作为应答。  
- 在发送时：  
　- 当应该发送第一个字节数据时，STOPF却为1。如果TXE=0，发送I2C_TXDR寄存器的内容，否则，发送0xFF。  
　- 当应该发送一个新字节时，I2C_TXDR寄存器中却没有写入新数据，此时将发送0xFF。  
当检测到上溢或下溢错误时，I2C_ISR寄存器中的OVR标志被置1，如果I2C_CR1寄存器中的ERRIE位置1，将产生中断。  
####包错误检测错误（PECERR）  
该部分仅针对支持SMBus功能的器件。  
当接收到的PEC字节和I2C_PECR的内容不匹配时，检测到PEC错误。接收到错误的PEC后，自动发送一个NACK作为应答。  
当PEC错误被检测到，I2C_ISR寄存器中的PECERR标志被置1，如果I2C_CR1中的ERRIE位置1，将产生中断。  
####超时错误（TIMEOUT）  
该部分仅针对支持SMBus功能的器件。  
下面任何一种情况都会导致超时错误发生：  
- TIDLE=0并且SCL保持低电平的时间达到了TIMEOUTA[11:0]所定义的时间：这用于检测SMBus超时。  
- TIDLE=1并且SDA和SCL两者保持高电平的时间达到了IMEOUTA[11:0]所定义的时间：这用来检测总线空闲状态。  
- 主器件时钟低电平积累的延伸时间达到了TIMEOUTB[11:0]所定义的时间（SMBus t<sub>LOW:MEXT</sub>参数）。  
- 从器件积累的时钟低电平延伸时间达到了TIMEOUTB[11:0]所定义的时间（SMBus t<sub>LOW:SEXT</sub>参数）。  
当主模式下检测到一个超时时，会自动发送STOP。  
在从模式下检测到超时时，SDA和SCL被自动释放。  
检测到超时后，I2C_ISR寄存器中的TIMEOUT标志被置1，如果I2C_CR1中的ERRIE位为1，将产生中断。  
####报警（ALERT）  
该部分仅针对支持SMBus功能的器件。  
当I2C接口被配置为HOST（SMBHEN=1），SMBA引脚报警检测功能被使能（ALERTEN=1）并且在其上检测到一个下降沿时，ALERT标志才被置1.如果I2C_CR1中的ERRIE位为1，还将产生中断。  
###DMA请求  
####使用DMA进行发送  
DMA（直接存储器存取）可以被用于发送，通过设置I2C_CR1中的TXDMAEN位为1来使能。当TXIS=1时，数据从DMA外设指向的SRAM区域，载入到I2C_TXDR寄存器中。  
只有数据用DMA进行传输。  
- 主模式下：初始化、从地址、方向、字节数和START位都由软件设置（发送的从地址不能使用DMA）。当所有数据要使用DMA进行传输时，在START位置1之前，必须初始化DMA。传输的结束由NBYTES计数器管理。  
######I2C configured in master mode to transmit with DMA code example  

	/* (1) Timing register value is computed with the AN4235 xls file,
	       fast Mode @400kHz with I2CCLK = 48MHz, rise time = 140ns,
	       fall time = 40ns */
	/* (2) Periph enable */
	/* (3) Slave address = 0x5A, write transfer, 2 bytes to transmit, autoend */
	I2C2->TIMINGR = (uint32_t)0x00B01A4B; /* (1) */
	I2C2->CR1 = I2C_CR1_PE | I2C_CR1_TXDMAEN; /* (2) */
	I2C2->CR2 = I2C_CR2_AUTOEND | (SIZE_OF_DATA << 16)
              | (I2C1_OWN_ADDRESS << 1); /* (3) */  
- 从模式下：  
　- NOSTRETCH=0时，所有数据使用DMA传输时，DMA必须在地址匹配事件之前，或在ADDR中断子程序中在ADDR清零之前，完成初始化。  
　- NOSTRETCH=1时，DMA必须在地址匹配事件之前完成初始化。  
- 支持SMBus时：PEC的传输由NBYTES计数器管理。  
如果DMA用于发送，TXIE不需要使能。  
####使用DMA进行接收  
DMA可以用于接收，通过设置I2C_CR1中的RXDMAEN位为1。当RXNE=1时，数据从I2C_RXDR寄存器载入到DMA外设指向的SRAM区域之中。只有数据（包括PEC）利用DMA进行传输。  
- 主模式下，初始化、从地址、方向、字节数和START位由软件设置。当所有数据都使用DMA传输时，DMA必须在START位置1之前初始化。传输结束由NBYTES计数器管理。  
- 从模式下，NOSTRETCH=0时，所有数据使用DMA传输，DMA必须在地址匹配事件之前，或ADDR中断子程序中ADDR标志清零之前，完成初始化。  
- 如果支持SMBus：PEC的传输由NBYTES计数器管理。  
如果DMA用于接收，RXIE不需要使能。  
###调试模式  
当微控制器进入调试模式（内核停止），SMBus超时定时器根据DBG模块中DBG_I2Cx_SMBUS_TIMEOUT配置位选择继续工作或停止。  
##I2C低功耗模式  
![](https://i.imgur.com/ZLPjQp2.png)  
##I2C中断  
![](https://i.imgur.com/wXttNuR.png)  
根据产品的实际功能，所有这些中断事件共享一个中断向量（I2C全局中断），或分位2个中断向量（I2C事件中断和I2C错误中断）。  
为了使能I2C中断，需要执行下列操作：  
1. 配置并使能NVIC中的I2C IRQ通道。  
2. 配置I2C以产生中断。  
![](https://i.imgur.com/X4UnvKo.png)  
##I2C寄存器  
这些外设寄存器按字（32位）访问。  
###控制寄存器1（I2C_CR1）  
![](https://i.imgur.com/FMRnulF.png)  
![](https://i.imgur.com/ck7mPEz.png)  
![](https://i.imgur.com/3yyTlUf.png)  
![](https://i.imgur.com/AIQixNr.png)  
###控制寄存器2（I2C_CR2）  
![](https://i.imgur.com/RJSNgEL.png)  
![](https://i.imgur.com/DZGpF8i.png)  
![](https://i.imgur.com/ZeEvNiW.png)  
![](https://i.imgur.com/poaLi2U.png)  
![](https://i.imgur.com/0BtnT6N.png)  
###本机地址1寄存器（I2C_OAR1）  
![](https://i.imgur.com/O0GOEnG.png)  
![](https://i.imgur.com/ytEctpG.png)  
###本机地址2寄存器（I2C_OAR2）  
![](https://i.imgur.com/6182nsz.png)  
![](https://i.imgur.com/F9dojwK.png)  
###时序寄存器（I2C_TIMINGR）  
![](https://i.imgur.com/qg3ERBA.png)  
![](https://i.imgur.com/204d7vh.png)  
