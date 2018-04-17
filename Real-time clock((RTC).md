#Real-time clock(RTC)  
##简介  
实时时钟RTC提供用于管理所有低功耗模式的自动唤醒单元。  
RTC是一个独立的BCD定时/计数器。RTC提供一个具有可编程报警功能的日历时钟。  
RTC还包含一个具有中断功能的周期性的可编程的唤醒标志。  
2个32位寄存器包含以BCD码格式表示的秒、分钟、小时（12或24小时制）、星期、日、月、年。亚秒值也是按二进制格式提供。  
每月的天数是28天/29天（闰年）/30天/31天会自动补偿。还可以执行夏令时时间补偿。  
另外的32位寄存器包含可编程的用于报警的亚秒、秒、分、时、星期和日期。  
一个数字校准功能可以用来对晶振精度的偏差进行补偿。  
RTC域复位后，所有的RTC寄存器被保护，以防非正常的写访问。  
不管器件处于何种状态（运行，低功耗或复位状态），只要供电电压在工作电压范围内，RTC就不会停止。  
##RTC主要特性  
RTC模块的主要特性如下：  
- 包含亚秒、秒、分钟、小时（12或24小时制）、星期、日期、月份、年份的日历。  
- 由软件编程的夏令时补偿。  
- 带中断的可编程报警功能。任何日历中的字段组合都可以用来触发报警。  
- 自动唤醒单元产生一个周期性的标志以触发自动唤醒中断。  
- 参考时钟检测：可以使用更精确的第二时钟源（50或60Hz）来提高日历的精确度。  
- 利用亚秒级移位特性与外部时钟精确同步。  
- 数字补偿电路（周期性的计数器校正）：精度为0.95ppm，在几秒的补偿窗口中获得。  
- 用于事件保存的时间戳功能。  
- 具有可配置滤波器和内部上拉的入侵检测事件。  
- 可屏蔽中断/事件：  
　- 闹钟A  
　- 唤醒中断  
　- 时间戳  
　- 入侵检测  
##RTC实时时钟的实现  
![](https://i.imgur.com/Iz9T3aH.png)  
##RTC功能描述  
###RTC框图  
![](https://i.imgur.com/IBg39k9.png)  
![](https://i.imgur.com/CJ0Nt0m.png)  
![](https://i.imgur.com/eDipvsa.png)  
![](https://i.imgur.com/vbe9r59.png)  
RTC包含：  
- 一个闹钟  
- 来自IO口的2个入侵事件  
　- 入侵检测擦除备份寄存器  
- 来自IO口的一个时间戳事件  
- 入侵事件检测可以产生一个时间戳事件  
- 复用功能输出：RTC_OUT可以选择下面2个输出其中之一：  
　- RTC_CALIB：512Hz或1Hz时钟输出（用32.768kHz的外部LSE时）。可以通过设置RTC_CR寄存器中的COE=1，使能这个输出。  
　- RTC_ALARM：闹钟A。可以通过配置RTC_CR寄存器中OSEL[1:0]位选择这个输出。  
- 复位功能输入：  
　- RTC_TS：时间戳事件  
　- RTC_TAMP1：入侵事件检测1  
　- RTC_TAMP2：入侵事件检测2  
　- RTC_REFIN：50或60Hz参考时钟输入  
###RTC控制GPIOs  
RTC_OUT,RTC_TS和RTC_TAMP1被映射到同一个引脚PC13上。  
RTC_ALARM输出的选择通过RTC_TAFCR寄存器如下操作：PC13VALUE位用于选择RTC_ALARM输出配置成推挽或开漏模式。  
当PC13不用作RTC复用功能时，可以通过设置RTC_TAFCR中的PC13MODE=1，强制其为推挽输出模式，而PC13VALUE位决定其输出值。这种情况下，PC13的推挽输出状态和输出值在待机模式下是可以保持的。  
PC13的输出机制遵循表63所示的优先级顺序。  
当PC14和PC15不用作LSE振荡器时，可以通过设置RTC_TAFCR中的PC14MODE=1和PC15MODE=1，强制其为推挽输出模式；其输出值由PC14VALUE和PC15VALUE决定。这种情况下，PC14和PC15的推挽输出状态和输出值在待机模式下是可以保持的。  
PC14和PC15的输出机制遵循表64和表65所示的优先级顺序。  
![](https://i.imgur.com/exByiwo.png)  
![](https://i.imgur.com/ewFCoPq.png)  
###时钟和预分频器  
RTC时钟源RTCCLK由时钟控制器在LSE、LSI和HSE时钟之间选择。  
一个可编程的预分频器产生1Hz的时钟，用于更新日历。为了最大程度的降低功耗，预分频器分割成2个可编程预分频器（参见图192：RTC框图）：  
- 通过RTC_PRER寄存器的PREDIV_A位配置的7位异步预分频器。  
- 通过RTC_PRER寄存器的PREDIV_S位配置的15位同步预分频器。  
注：当2个预分频器都使用时，建议将异步预分频器配置为较高的值，以减少功耗。  
使用32.768KHz的LSE时钟时，异步分频器的分频系数设为128，同步分频器的分频系数设为256，从而得到1Hz（ck_spre）的内部时钟频率。  
最小的分频系数是1，而最大的分频系数是2<sup>22</sup>。  
所以最大的输入频率是4MHz左右。  
f<sub>ck_apre</sub>=f<sub>RTCCLK</sub>/(PREDIV_A+1)  
ck_apre时钟为二进制RTC_SSR亚秒向下计数器提供时钟。当计数器减到0，RTC_SSR将被重载为PREDIV_S的值。  
f<sub>ck_spre</sub>=f<sub>RTCCLK</sub>/((PREDIV_S+1)+(PREDIV_A+1))  
ck_spre时钟用于更新日历或作为16位唤醒自动重载定时器的时基。为了获得较短的超时周期，16位唤醒自动重载定时器可以使用经过可编程的4位异步预分频器分频后的RTCCLK作为时钟。  
###实时时钟和日历  
RTC日历时间和日期寄存器可以通过各自的影子寄存器进行访问的，这些影子寄存器与PCLK(APB时钟)同步。它们也可以直接访问，这样避免了同步期间的等待。  
- RTC_SSR亚秒寄存器  
- RTC_TR时间寄存器  
- RTC_DR日期寄存器  
每2个RTCCLK周期，当前的日历数据被拷贝到对应的影子寄存器中，并且RTC初始化和状态寄存器RTC_ISR中的RSF位置1。在停机和待机模式下，不会执行拷贝。当退出这些模式后，影子寄存器会在最多2个RTCCLK周期后更新。  
当应用程序读日历寄存器时，默认访问的是影子寄存器。设置RTC_CR寄存器中的BYPSHAD控制位置1，也可以选择直接访问日历寄存器。默认情况下，BYPSHAD=0，所以用户访问影子寄存器。  
在BYPSHAD=0读RTC_SSR,RTC_TR或RTC_DR寄存器的情况下，APB时钟频率f<sub>APB</sub>必须至少是RTC时钟频率f<sub>RTCCLK</sub>的7倍。  
影子寄存器通过系统复位来复位。  
###可编程闹钟  
RTC单元提供一个可编程闹钟：Alarm A。  
可编程闹钟功能通过RTC_CR中的ALRAE位使能。如果日历中的亚秒、秒、分钟、小时、日期或星期和闹钟寄存器RTC_ALRMASSR和RTC_ALRMAR中编程的值相匹配时，ALRAF位置1。每个日历段可以通过RTC_ALRMASSR中的MASKSSx和RTC_ALRMAR中的MSKx位独立选为闹钟源。设置RTC_CR中的ALRAIE位，使能闹钟中断。  
警告：如果选择秒字段（RTC_ALRMAR中的MSK1=0），RTC_PRER中的同步预分频器分频系数必须≥3，才能确保闹钟正确运行。  
闹钟A（如果通过RTC_CR中的OSEL[1:0]使能）可以连接到RTC_ALARM输出。RTC_ALARM输出的极性由RTC_CR中的POL位配置。  
###周期性的自动唤醒  
周期唤醒标志由一个16位的可编程自动重载向下计数器产生。唤醒定时器可以扩展为17位。  
唤醒功能通过RTC_CR中的WUTE位使能。  
唤醒定时器的时钟输入可以是：  
- RTCCLK的2，4，8或16分频。当RTCCLK是LSE(32.768KHz)时，唤醒中断周期可以配置为122us到32s之间，且分辨率低至61us。  
- ck_spre（通常为1Hz内部时钟）。当ck_spre频率为1Hz时，唤醒时间从1s到36h左右，分辨率为1s。这一较大的可编程时间范围分成2部分：  
　- 当WUCKSEL[2:1]=10时，从1s到18h。  
　- 当WUCKSEL[2:1]=11时，从18h到36h。这种情况，2<sup>16</sup>添加到16位计数器当前值上。当初始化步骤完成，定时器开始向下计数。当使能唤醒功能，向下计数器在低功耗模式下依然工作。另外，当计数器减到0时，RTC_ISR中的WUTF标志置1，并且计数器自动重载入RTC_WUTR中的重载值。  
WUTF必须由软件清零。  
当周期唤醒中断通过设置RTC_CR中的WUTIE=1使能时，它可以使器件退出低功耗模式。  
周期唤醒标志可以连接到RTC_ALARM输出，需要通过设置RTC_CR中的OSEL[1:0]位来使能。RTC_CR中的POL位配置RTC_ALARM输出的极性。  
系统复位和低功耗模式（休眠、停机和待机）对唤醒定时器没有影响。  
###RTC初始化和配置  
####RTC寄存器访问  
RTC寄存器是32位寄存器。APB接口会在访问RTC寄存器时引入2个等待周期，除了当BYPSHAD=0时读取日历影子寄存器。  
####RTC寄存器写保护  
系统复位后，PWR_CR寄存器中的DBP清零，RTC寄存器被保护以避免非正常的写访问。必须设置DBP=1使能对RTC寄存器的写访问。  
RTC域复位后，所有RTC寄存器被写保护。向写保护寄存器RTC_WPR写入密钥使能RTC寄存器写访问。  
解锁RTC寄存器（除了RTC_TAFCR和RTC_ISR[13:8]）的写保护需要下列步骤：  
1. 向RTC_WPR写入0xCA  
2. 向RTC_WPR写入0x53  
写入错误的关键字将激活写保护。  
保护机制不受系统复位影响。  
####日历初始化和配置  
要编程包括时间格式和预分频器配置在内的初始时间和日期日历值，需要按一下步骤：  
1. 设置RTC_ISR寄存器中的INIT=1，进入初始化模式；日历计数器停止，并且可更新计数器的值。  
2. 轮询RTC_ISR中的INITF位。当INITF=1时，进入初始化阶段模式。这需要大概2个RTCCLK时钟周期（由于时钟同步）。  
3. 编程RTC_PRER寄存器中的2个预分频器的分频系数，产生日历计数器使用的1Hz时钟。  
4. 向影子寄存器（RTC_TR和RTC_DR）中写入初始时间和日期，并且通过RTC_CR中的FMT位配置时间格式（12或24小时制）。  
5. 将INIT位清零，退出初始化模式。然后，自动加载日历计数器的实际值，并在4个RTCCLK时钟周期后计数器重新开始计数。  
当初始化操作完成后，日历开始计时。  
注：系统复位后，应用程序可以读取RTC_ISR中的INITS位来检查日历是否已经初始化。如果INITS=0，表明日历没有初始化，因为年字段是RTC域复位后的默认值（0x00）。  
在初始化后要读取日历，必须在RTC_ISR中的RSF=1之后。  
######RTC calendar configuration code example  

	/* (1) Write access for RTC registers */
	/* (2) Enable init phase */
	/* (3) Wait until it is allow to modify RTC register values */
	/* (4) set prescaler, 40kHz/128 => 312 Hz, 312Hz/312 => 1Hz */
	/* (5) New time in TR */
	/* (6) Disable init phase */
	/* (7) Disable write access for RTC registers */
	RTC->WPR = 0xCA; /* (1) */
	RTC->WPR = 0x53; /* (1) */
	RTC->ISR |= RTC_ISR_INIT; /* (2) */
	while ((RTC->ISR & RTC_ISR_INITF) != RTC_ISR_INITF) /* (3) */
	{
		/* add time out here for a robust application */
	}
	RTC->PRER = 0x007F0137; /* (4) */
	RTC->TR = RTC_TR_PM | Time; /* (5) */
	RTC->ISR &=~ RTC_ISR_INIT; /* (6) */
	RTC->WPR = 0xFE; /* (7) */
	RTC->WPR = 0x64; /* (7) */  
####夏令时  
通过RTC_CR中的SUB1H,ADD1H和BKP位管理夏令时。  
使用SUB1H或ADD1H位，软件可以不通过初始化操作便可以在日历中一次减去一小时或增加一小时。  
另外，软件可以使用BKP位来记录是否执行过此操作。  
####编程闹钟  
要编程或更新可编程闹钟，必须执行类似以下的步骤：  
1. 清零RTC_CR中的ALRAE位，禁止闹钟A。  
2. 配置闹钟A寄存器（RTC_ALRMASSR和RTC_ALRMAR）。  
3. 设置RTC_CR中的ALRAE位为1，重新使能闹钟A。  
注：由于时钟同步的原因，每次RTC_CR寄存器的改变，要在2个RTCCLK时钟周期后才生效。  
######RTC alarm configuration code example  

	/* (1) Write access for RTC registers */
	/* (2) Disable alarm A to modify it */
	/* (3) Wait until it is allow to modify alarm A value */
	/* (4) Modify alarm A mask to have an interrupt each 1Hz */
	/* (5) Enable alarm A and alarm A interrupt */
	/* (6) Disable write access */
	RTC->WPR = 0xCA; /* (1) */
	RTC->WPR = 0x53; /* (1) */
	RTC->CR &=~ RTC_CR_ALRAE; /* (2) */
	while ((RTC->ISR & RTC_ISR_ALRAWF) != RTC_ISR_ALRAWF) /* (3) */
	{
		/* add time out here for a robust application */
	}
	RTC->ALRMAR = RTC_ALRMAR_MSK4 | RTC_ALRMAR_MSK3
	            | RTC_ALRMAR_MSK2 | RTC_ALRMAR_MSK1; /* (4) */
	RTC->CR = RTC_CR_ALRAIE | RTC_CR_ALRAE; /* (5) */
	RTC->WPR = 0xFE; /* (6) */
	RTC->WPR = 0x64; /* (6) */  
####编程唤醒定时器  
要配置或改变唤醒定时器自动重载值（RTC_WUTR中的WUT[15:0]），需要按照以下步骤：  
1. 清零RTC_CR中的WUTE位，禁止唤醒定时器。  
2. 轮询RTC_ISR中的WUTWF位，直到该位为1，以确保访问唤醒自动重载计数器和WUCKSEL[2:0]位被允许。这需要大概2个RTCCLK时钟周期（由于时钟同步）。  
3. 配置唤醒自动重载值WUT[15:0]，并选择唤醒时钟（RTC_CR中的WUCKSEL[2:0]）。设置RTC_CR中的WUTE为1，重新使能唤醒定时器。唤醒定时器重新向下计数。由于时钟同步的原因，在WUTE清零2个RTCCLK时钟周期之后，WUTWF被清零。  
######RTC WUT configuration code example  

	/* (1) Write access for RTC registers */
	/* (2) Disable wake up timerto modify it */
	/* (3) Wait until it is allow to modify wake up reload value */
	/* (4) Modify wake upvalue reload counter to have a wake up each 1Hz */
	/* (5) Enable wake up counter and wake up interrupt */
	/* (6) Disable write access */
	RTC->WPR = 0xCA; /* (1) */
	RTC->WPR = 0x53; /* (1) */
	RTC->CR &= ~RTC_CR_WUTE; /* (2) */
	while ((RTC->ISR & RTC_ISR_WUTWF) != RTC_ISR_WUTWF) /* (3) */
	{
		/* add time out here for a robust application */
	}
	RTC->WUTR = 0x9C0; /* (4) */
	RTC->CR = RTC_CR_WUTE | RTC_CR_WUTIE; /* (5) */
	RTC->WPR = 0xFE; /* (6) */
	RTC->WPR = 0x64; /* (6) */  
