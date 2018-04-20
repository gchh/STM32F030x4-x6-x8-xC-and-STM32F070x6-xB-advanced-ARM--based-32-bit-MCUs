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
###读日历  
####当RTC_CR寄存器中的BYPSHAD控制位清零时  
为正确地读取RTC日历寄存器（RTC_SSR,RTC_TR和RTC_DR），APB时钟频率（f<sub>PCLK</sub>）必须等于或大于RTC时钟频率（f<sub>RTCCLK</sub>）的7倍。这确保了同步机制的安全行为。  
如果APB时钟频率小于RTC时钟频率的7倍，软件必须读日历时间和日期寄存器2次。当2次读取结果相同时，才能确保数据正确。否则，必须再读取一次，再进行判断。在任何情况下，APB时钟频率不能低于RTC时钟频率。  
每次日历寄存器被拷贝到RTC_SSR,RTC_TR和RTC_DR影子寄存器中时，RTC_ISR中的RSF位会被置1。每2个RTCCLK周期会执行一次拷贝。为了确保这3个值来自同一时刻点，在读取RTC_SSR或RTC_TR时会锁定高阶日历影子寄存器中的值，直到读取RTC_DR。为了避免软件读取日历的时间间隔小于2个RTCCLK周期：必须在第一次读日历后由软件将RSF位清零，并且必须软件必须等待RSF置1后才能再次读取RTC_SSR,RTC_TR和RTC_DR寄存器。  
从低功耗模式（停机或待机）唤醒后，RSF位必须由软件清零。要读取RTC_SSR,RTC_TR和RTC_DR寄存器必须等待RSF置1后进行。  
RSF必须在唤醒后，而不是在进入低功耗模式之前，清零。  
系统复位后，软件必须等到RSF置1才能读取RTC_SSR,RTC_TR和RTC_DR。事实上，系统复位会将影子寄存器复位为它们的默认值。  
初始化后，必须等RSF置1后才能读取RTC_SSR,RTC_TR和RTC_DR。  
同步后，必须等RSF置1后才能读取RTC_SSR,RTC_TR和RTC_DR。  
######RTC read calendar code example  

	if((RTC->ISR & RTC_ISR_RSF) == RTC_ISR_RSF)
	{
		TimeToCompute = RTC->TR; /* get time */
		DateToCompute = RTC->DR; /* need to read date also */
	}  
####当RTC_CR寄存器中的BYPSHAD控制位置1时（旁路影子寄存器）  
直接从日历计数器中读取日历寄存器值，因此不需要等待RSF位置1。这在退出低功耗模式（停机或待机）后马上读取非常有用，因为在低功耗模式下影子寄存器不会更新。  
当BYPSHAD位置1时，如果在2次读取寄存器之间出现RTCCLK边沿，则不同寄存器读取的结果可能不一致。另外，在读操作期间如果出现RTCCLK边沿，可能导致某个寄存器的值不正确。软件必须读取所有寄存器2次，并且比较2次读取的结果，以确认数据是否一致和正确。此外，软件可以只比较2次读取日历寄存器得到的结果的最低位。  
注：当BYPSHAD=1时，读日历寄存器的指令需要一个额外的APB周期来完成。  
###复位RTC  
任何可用的系统复位源都将导致日历影子寄存器（RTC_SSR,RTC_TR和RTC_DR）和RTC状态寄存器（RTC_ISR）中的某些位被复位为它们的默认值。  
相反，下列寄存器在发生RTC域复位时被复位为它们的默认值，而不受系统复位影响：RTC当前日历寄存器，RTC控制寄存器（RTC_CR），预分频器寄存器（RTC_PRER），RTC校准寄存器（RTC_CALR），RTC移位寄存器（RTC_SHIFTR），RTC时间戳寄存器（RTC_TSSSR,RTC_TSTR和RTC_TSDR），RTC入侵和复用功能配置寄存器（RTC_TAFCR），唤醒定时器寄存器（RTC_WUTR），闹钟A寄存器（RTC_ALRMASSR,RTC_ALRMAR）。  
另外，当由LSE提供时钟时，如果复位源不是RTC域复位源，则RTC将在系统复位时保持工作（参考复位和时钟控制器的RTC时钟部分，详细列出了不受系统复位影响的RTC时钟源）。当发生RTC域复位时，RTC停止运行，并且所有RTC寄存器被设置为它们的复位值。  
###RTC同步  
RTC可与高精度的远程时钟同步。在读取亚秒字段（RTC_SSR或RTC_TSSSR）后，可以计算出远程时钟和RTC之间精确的时间偏差。然后，RTC可以使用RTC_SHIFTR进行几分之一秒的时钟移位，消除此偏差。  
RTC_SSR包含同步预分频器计数器的值。这样就可以计算分辨率低至1/(PREDIV_S+1)秒的RTC的精确时间。因此，可以通过增加同步预分频器的值（PREDIV_S[14:0]）来提高分辨率。当PREDIV_S=0x7FFF，获得最大的分辨率（30.52us，时钟为32768Hz）。  
然而，保持同步预分频器输出1Hz，意味着增加PREDIV_S必须减小PREDIV_A。这样，异步预分频器输出频率将增加，进而导致RTC动态功耗增加。  
RTC可以使用移位控制寄存器（RTC_SHIFTR）进行微调。写入RTC_SHIFR可以以1/(PREDIV_S+1)秒的分辨率移动（延迟或提前）时钟，最长可以移动1秒。移位操作将SUBFS[14:0]的值加到同步预分频器的计数器SS[15:0]中：使时钟延迟。如果同时ADD1S=1，这导致增加1秒并减去几分之一秒，因此时钟提前。  
警告：在初始化移位操作前，必须确保SS[15]=0，以防发生溢出。  
一旦写RTC_SHIFTR寄存器来初始化移位操作，SHPF标志会由硬件置位，表示正在等待一个移位操作完成。一旦移位操作完成，硬件将清除此标志位。  
警告：该同步功能和参考时钟检测功能不兼容：当REFCKON=1时，固件不能对RTC_SHIFTR写操作。  
###RTC参考时钟检测  
RTC日历的更新可以与参考时钟同步，参考时钟RTC_REFIN通常为市电频率（50或60Hz）。参考时钟RTC_REFIN的精度应高于32.768KHz的LSE时钟。当RTC_REFIN检测使能（RTC_CR中的REFCKON=1），日历仍由LSE提供时钟，而RTC_REFIN用于补偿日历更新频率（1Hz）的偏差。  
每个1Hz时钟边沿会与最近的RTC_REFIN时钟边沿比较（如果在给定的时间窗口内发现一个边沿）。在多数情况下，这两个时钟边沿恰好对齐。当由于LSE时钟的误差造成1Hz时钟对不齐时，RTC移动1Hz时钟一位，以使后来的1Hz时钟边沿能对齐。由于该机制，日历可以和参考时钟一样精确。  
RTC使用32.768kHz晶振产生的256Hz时钟（ck_apre）来检测是否存在参考时钟。在每次日历更新（每1秒）前后的时间窗口中执行检测。检测第一个参考时钟边沿的窗口等于7个ck_apre周期；后来的日历更新窗口减少到3个ck_apre周期。  
每次在窗口中检测到参考时钟，输出ck_apre时钟的异步预分频器都会被重载。当参考时钟和1Hz时钟对齐时，此操作无影响，因为预分频器会在同一时刻被重载。当时钟没有对齐时，重载会将后续的1Hz时钟边沿移动一点点，以使其能和参考时钟对齐。  
如果参考时钟停止（在3个ck_apre周期的窗口内没有出现参考时钟边沿），日历将仅依照LSE时钟继续更新。然后，RTC使用一个围绕ck_spre边沿的7个ck_apre周期的大检测窗口来检测参考时钟。  
当RTC_REFIN检测被使能，PREDIV_A和PREDIV_S必须被设置为它们的默认值：  
- PREDIV_A=0x007F  
- PREDIV_S=0x00FF  
注：RTC_REFIN时钟检测在待机模式下不能使用。  
###RTC高精度数字校准  
RTC频率可以按大约0.954ppm的分辨率进行校准，校准范围从-487.1ppm到+488.5ppm。执行一系列的微调（增加/减少单独的RTCCLK脉冲）进行频率校正。这些微调分布的非常好，因此RTC校准的很好，即使在短时间内持续观察也是如此。  
精密数字校准在大约2<sup>20</sup>个RTCCLK脉冲的一个周期中执行，当输入频率为32768Hz时此周期为32秒。此周期由一个RTCCLK提供时钟的20位计数器cal_cnt[19:0]维持。  
精密校准寄存器（RTC_CALR）指定在32秒周期内要屏蔽的RTCCLK时钟周期数：  
- 位CALM[0]=1，32秒周期内屏蔽1个脉冲。  
- 位CALM[1]=1，屏蔽2个周期。  
- 位CALM[2]=1，屏蔽4个周期。  
- 依此类推，直到CALM[8]=1，屏蔽256个时钟。  
注：CALM[8:0]（RTC_CALR）指定在32秒周期内被屏蔽的RTCCLK脉冲数量。设置CALM[0]=1，在cal_cnt[19:0]=0x80000时，在32秒的周期内只屏蔽一个脉冲；CALM[1]=1，会屏蔽2个周期（当cal_cnt=0x40000和0xC0000时）；CALM[2]=1，屏蔽4个周期（cal_cnt=0x20000/0x60000/0xA0000/0xE0000）；依此类推，CALM[8]=1，屏蔽256时钟（cal_cnt=0xXX800）。  
使用适当的分辨率，CALM可使RTC频率最多减少487.1ppm，而CALP置位可使频率增加488.5ppm。CALP=1会导致，每隔2<sup>11</sup>个RTCCLK周期插入一个额外的RTCCLK脉冲，这意味着每一个32秒周期中增加了512个时钟。  
CALM和CALP配合使用，可以在32秒周期内增加-511到+512个RTCCLK周期，换算成校验范围就是-487.1ppm到+488.5ppm，分辨率（校验步长）约为0.954ppm。  
若给定输入频率F<sub>RTCCLK</sub>，则计算有效校准频率F<sub>CAL</sub>的公式如下：  
F<sub>CAL</sub>=F<sub>RTCCLK</sub> × [1+(CALP×512-CALM)/(2<sup>20</sup>+CALM-CALP×512)]  
####当PREDIV_A<3时的校准  
当异步预分频器的值（RTC_PRER中的PREDIV_A）小于3时，CALP不能置1。如果CALP已经置1，并且PREDIV_A设的值小于3，那么CALP会被忽略，校准按CALP=0操作。  
要在PREDIV_A<3情况下执行校准，同步预分频器的值（PREDIV_S）应该减少以便每一秒可加速8个RTCCLK时钟周期，相当于每个32秒周期加256个时钟周期。因此，仅使用CALM位，就可以在每个32秒周期内有效增加255到256个时钟脉冲（对应的校准范围从243.3ppm到244.1ppm）。  
在标称RTCCLK频率32768Hz下，当PREDIV_A=1（分频系数为2），PREDIV_S应该设为16379而不是16383（少4）。当PREDIV_A=0时，PREDIV_S应该设为32759而不是32767（少8）。  
如果PREDIV_S以这种方式减小，有效校准频率的计算公式如下：  
F<sub>CAL</sub>=F<sub>RTCCLK</sub> × [1+(256-CALM)/(2<sup>20</sup>+CALM-256)]  
在此情况下，如果RTCCLK正好是32768Hz，则CALM[7:0]=0x100（CALM范围的中点）是正确的设置。  
####验证RTC校准  
通过测量RTCCLK的精确频率，计算正确的CALM和CALP值，来确保RTC的精确度。提供了一个可选的1Hz输出，允许用户测量和验证RTC精度。  
在有限的时间间隔内测量RTC的精确频率，可能会导致在测量期间产生最多2个RTCCLK时钟周期的测量误差，取决于数字校准周期与测量周期的对齐方式。  
但是，如果测量周期和校准周期一样长，此测量误差可以被消除。在此情况下，