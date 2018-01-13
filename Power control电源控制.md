##Power control(PWR) 电源控制  
###Power supplies 电源  
STM32F030/STM32F070内置电压调节器用以给内部1.8V的电路提供电源。  
STM32F030/STM32F070要求2.4V-3.6V的工作电源电压，以及2.4V-3.6V的模拟供电电压。  
![](https://i.imgur.com/56KqT9N.png)  

####Independent A/D converter supply and reference voltage 独立的A/D转换器供电电源和基准电压  
为了提高转换精度和扩展应用灵活性，ADC使用独立的供电电源，可以单独过滤和屏蔽PCB上的噪声。  
独立的ADC电源V<sub>DDA</sub>和地V<sub>SSA</sub>引脚。  
VDDA供电/参考电压必须高于或等于VDD电压。VDDA可以通过外部的滤波电路连接到VDD。如果VDDA不和VDD连接在一起，必须保证VDDA的电压总是高于或等于VDD的电压；为了确保在上电和掉电工程中这一条件成立，可以在VDD和VDDA之间加一个肖特基二极管。  

####Voltage regulator  电压调节器  
电压调节器在复位后就开始工作，取决于不同的应用模式有三种不同的工作模式。  
- 在Run mode，电压调节器向1.8V域(core,memories和digital peripherals)满功率供电  
- 在Stop mode，电压调节器向1.8V域低功率供电，保持寄存器和SRAM的内容  
- 在Standby moed，电压调节器断电，除了待机电路，寄存器和SRAM的内容全部丢失  

###Power supply supervisor 电源管理器  
####Power on reset(POR)/Power down reset(PDR) 上电复位/掉电复位  
芯片内集成了，确保在电源电压升至或降至2V阈值做出适当操作的，POR上电复位和PDR掉电复位电路。  
当供电电压低于规定的阈值（V<sub>POR</sub>/V<sub>PDR</sub>），器件将维持复位状态。  
- POR上电复位只监测VDD电压。在启动阶段，VDDA电压必然首先到达VPOR阈值，并且VDDA电压高于等于VDD。  
- PDR掉电复位同时监测VDD和VDDA电压。但是，如果应用设计能确保VDDA电压高于等于VDD电压，则可以编程V<sub>DDA_MONITOR</sub>位关闭VDDA电压监测器，以减少功耗。  
更多的有关POR/PDR阈值的细节，可以参考数据手册的电气特性部分。  
![](https://i.imgur.com/Cn02uYB.png)  
当VDD升至VPOR后，Reset保持Temporization的时间后，器件可以正常工作；当VDD/VDDA电压降至VPDR，Reset会马上复位。VPDR比VPOR低40mv。  

###Low-power modes 低功耗模式  
默认情况下，在系统或电源复位后，微控制器工作在Run mode。当CPU不需要保持运行时，有几种低功耗模式可供选择，来节省功耗。用户需要根据功耗，启动时间和唤醒源，选择一个最佳的低功耗模式。  
器件有3种低功耗模式：  
- 休眠模式（Sleep mode）：CPU时钟关闭，所有外设（包括ARM Cortex-M0内核外设，如NVIC,SysTick等）保持运行  
- 停止模式（Stop mode）：所有时钟关闭  
- 待机模式（Standby mode）：1.8V供电域断电  
此外，在运行模式，可以通过以下手段减少功耗：  
- 降低系统时钟频率  
- 关闭未用的APB和AHB外设时钟  

![](https://i.imgur.com/IAVbGfa.png)  
![](https://i.imgur.com/rkxIS4y.png)  

####Slowing down system clocks 降低时钟频率  
在运行模式，可以编程预分频寄存器降低系统时钟（SYSCLK,HCLK,PCLK）的频率；在进入休眠模式前，也可以配置这些预分频器来降低外设时钟的频率。  
参考Clock configuration register(RCC_CFGR)时钟配置寄存器了解更多详情。  

####Peripheral clock gating 关闭外设时钟  
AHB时钟（HCLK）和APB（PCLK）为各个外设和存储器提供时钟，在运行模式，可以随时关闭HCLK和PCLK，以减少功耗。  
为了进一步减少休眠模式的功耗，在执行WFI或WFE指令进入休眠模式前，可以关闭外设时钟。  
外设时钟开关由AHB外设时钟使能寄存器（RCC_AHBENR）,APB外设时钟使能寄存器2（RCC_APB2ENR）和APB外设时钟使能寄存器1（RCC_APB1ENR）控制。  

####Sleep mode 休眠模式  
#####进入休眠模式  
执行WFI(等待中断)或WFE(等待事件)指令可以进入休眠模式。取决于ARM Cortex-M0系统寄存器的SLEEPONEXIT位，有2种进入休眠模式的机制：  
- Sleep-now：如果SLEEPONEXIT=0，一旦执行WFI或WFE指令MCU马上就进入休眠模式。  
- Sleep-on-exit：如果SLEEPONEXIT=1，执行WFI或WFE指令后，MCU从最低优先级的中断服务程序退出后，才进入休眠模式。  
在休眠模式，所有I/O口保持运行模式的状态。  

#####退出休眠模式  
如果是执行WFI指令进入休眠模式，任何嵌套向量中断控制器（NVIC）识别的外设中断都可以唤醒MCU退出休眠模式。  
如果是执行WFE指令进入休眠模式，当任一事件发生时，MCU退出休眠模式。唤醒事件由以下方式产生：  
- 在外设控制寄存器中使能一个中断，但不使能NVIC对应的中断，同时，使能ARM Cortex-M0系统控制寄存器的SEVONPEND位。当MCU从WFE中唤醒时，外设中断挂起位和外设NVIC中断请求通道挂起位（在NVIC中断清除挂起寄存器中）必须被清除。  
- 或者配置一个外部或内部EXTI line处于事件模式。当MCU从WFE中唤醒时，不必清除外设中断挂起位和外设NVIC中断请求通道挂起位，因为唤醒事件对应的挂起位并没有被置位。  
休眠模式因为在进入和退出中断没有浪费时间，所以，所需唤醒时间最短。  
![](https://i.imgur.com/AJJMKLI.png)  

####Stop mode 停机模式  
停止模式是在ARM Cortex-M0深度睡眠基础上结合关闭外设时钟。电压调节器可以被配置工作在正常或低功耗模式。在停机模式，1.8V供电区域的时钟全部停止（但是还在供电1.8V），PLL，HSI和HSE振荡器关闭；SRAM和寄存器中的内容会保留（还有1.8V供电）；所有I/O口保持在运行模式时的状态。  
#####进入停止模式  
为了进一步降低功耗，可以设置电源控制寄存器（PWR_CR）的LPDS位，使电压调节器工作在低功耗模式。  
如果正在进行FLASH编程，则等其结束，MCU才进入停机模式。  
如果正在访问APB区域，也需等其完成，MCU才能进入停机模式。  
在停机模式，可以配置各个控制位来选择以下功能：  
- 独立看门狗（IWDG）：可以通过配置选项字节由硬件启动，或由软件向关键字寄存器（IWDG_KR）写入关键字来启动。IWDG一旦启动，在系统复位前，会一直保持开启。  
- 实时时钟（RTC）：由RTC域控制寄存器（RCC_BDCR）的RTCEN位设置。  
- 内部低速RC振荡器（LSI）：由控制/状态寄存器（RCC_CSR）的LSION位设置。  
- 外部32.768kHz振荡器（LSE）：由RTC域控制寄存器（RCC_BDCR）的LSEON位设置。  
如果在进入停机模式前，没有关闭ADC，在进入停机模式后，ADC依然会产生功耗。可以设置ADC控制寄存器（ADC_CR）关闭ADC。  
#####退出停机模式  
当发生一个中断或事件唤醒MCU退出停机模式时，HSI将被选作为系统时钟。  
当电压调节器被设置为低功耗模式，从停机模式唤醒的时间将增加电压调节器从低功耗模式唤醒的时间。因此，如果在停机模式保持电压调机器正常开启，会缩短唤醒时间，但是相应的会增加停机模式的功耗。  
![](https://i.imgur.com/O1V1XQ5.png)  

####待机模式  
待机模式可以获得最低的功耗，它结合了ARM Cortex-M0深度睡眠和关闭电压调节器。在待机模式，1.8V供电区域（不再提供1.8V供电）,PLL,HSI和HSE全部关闭；SRAM和寄存器的内容也会丢失（因为没有1.8V的供电）。  
#####进入待机模式  
在待机模式，可以配置各个控制位来选择以下功能：  
- 独立看门狗（IWDG）：可以通过配置选项字节由硬件启动，或由软件向关键字寄存器（IWDG_KR）写入关键字来启动。IWDG一旦启动，在系统复位前，会一直保持开启。  
- 实时时钟（RTC）：由RTC域控制寄存器（RCC_BDCR）的RTCEN位设置。  
- 内部低速RC振荡器（LSI）：由控制/状态寄存器（RCC_CSR）的LSION位设置。  
- 外部32.768kHz振荡器（LSE）：由RTC域控制寄存器（RCC_BDCR）的LSEON位设置。  
#####退出待机模式  
当NRST引脚产生外部复位，IWDG复位，WKUPx引脚有个上升沿，或发生RTC事件，MCU将退出待机模式。从待机模式退出后，除了电源控制与状态寄存器（PWR_CSR），其他寄存器都将复位。  
从待机模式唤醒后，程序会像复位后一样重新运行（采样启动模式引脚，载入选项字节，读取复位向量等）。电源控制与状态寄存器（PWR_CSR）的SBF位指示MCU曾处于待机模式。  
![](https://i.imgur.com/wbLiFFi.png)  

#####停机模式中I/O的状态  
在停机模式中，所有的I/O口都是高阻抗状态，除了：  
- Reset复位引脚（始终有效）  
- PC13如果配置为RTC功能，PC14和PC15如果配置成LSE功能  
- WKUPx引脚  

#####调试模式  
默认情况下，在停机和待机模式下，调试是无法使用的；因为此时ARM Cortex-M0内核时钟是关闭的。  
然而，通过设置DBGMCU_CR寄存器的某些位，即使在低功耗模式，也可以调试软件。  

####低功耗模式下的RTC唤醒  
通过RTC报警，RTC可以将MCU从低功耗中唤醒。可以选择三个RTC时钟源中的2个来实现此目的，通过设置RTC域控制寄存器（RCC_BDCR）的RTCSEL[1:0]位：  
- 低功耗的32.768kHz外部晶振（LSE）：该时钟源提供了低功耗（典型条件下消耗小于1uA）且精确的时间基准。  
- 低功耗的内部RC振荡器（LSI）：使用该时钟源的优点在于节省的32.768kHz晶振的费用。内部RC振荡器被设计成增加最小的功耗。  
有了时钟源，要使用RTC报警事件将MCU从停机模式唤醒，还需要如下的配置：  
- 配置EXTI 17上升沿触发  
- 配置RTC产生RTC报警  
将MCU从待机模式唤醒，不需要配置EXTI 17，只需要RTC报警。  

###Power control registers 电源控制寄存器  
可用半字（16-bit）或字（32-bit）操作这些外设寄存器。  
####Power control register(PWR_CR) 电源控制寄存器  
![](https://i.imgur.com/IaCJntH.png)  

####Power control/status register(PWR_CSR) 电源控制与状态寄存器  
![](https://i.imgur.com/BxF9rxG.png)  

###PWE register map  
![](https://i.imgur.com/LxSO1he.png)  
