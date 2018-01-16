##Reset and clock control(RCC)复位与时钟控制  

###Reset 复位  
有三种复位：系统复位，电源复位和RTC报警复位。  
####Power reset 电源复位  
当有以下事件发生时，会产生电源复位：  
- POR/PDR上电/掉电复位  
- 退出待机模式时  
电源复位会复位所有寄存器的指（PWR_CSR不受影响）。  

####System reset 系统复位  
系统复位会复位除时钟控制与状态寄存器（RCC_CSR）复位标志外所有寄存器的值（包括RCC_CSR除复位标志外的其他位的值）。  
当下列事件发生时，会产生系统复位：  
- NRST引脚低电平（外部复位）  
- 窗口看门狗事件（WWDG复位）  
- 独立看门狗事件（IWDG复位）  
- 软件复位（SW复位）  
- 低功耗管理复位  
- 选项字节装载器复位（FLASH_CR的OBL_LAUNCH置1）  
- 电源复位  
复位与时钟控制状态寄存器RCC_CSR中的复位标志指出了时哪种复位源引起的复位。  
这些复位源都会将NRST引脚拉低并在一定的延时时段内保持低电平。复位入口地址固定在0x00000004。  
内部的复位信号会在NRST上输出，而脉冲发生器会保证每个内部复位源产生至少20uS的复位脉冲。外部复位，是NRST被拉低产生了复位脉冲。  
![](https://i.imgur.com/trC5CjF.png)  
#####Software reset  
ARC Cortex-M0应用中断和复位控制寄存器的SYSRESETREQ位置位，会强制MCU软件复位。参考Cortex-M0技术参考手册了解更多详情。  
#####Low-power management reset  
两种情况会发生低功耗管理复位：  
- 当进入待机模式时产生复位：将用户选项字节的nRST_STDBY位resetting（按选项字节的描述应该是清0），当执行了进入待机模式的指令时，MCU也不会进入待机模式，而是产生复位。  
- 当进入停机模式时产生复位：将用户选项字节的nRST_STOP位resetting（按选项字节的描述应该是清0），当执行了进入停机模式的指令时，MCU也不会进入停机模式，而是产生复位。  
#####Option byte loader reset  
当FLASH_CR的OBL_LAUNCH置位，将产生此复位。此位用于通过软件启动选项字节加载。   

####RTC domain reset  
RTC域复位仅影响LSE，RTC和RTC域控制寄存器（RCC_BDCR）。当下列事件发生时，产生此复位：  
- 由RCC_BDCR的BDRST置位引起的软件复位  
- 上电复位  

###Clocks 时钟   
以下的时钟源可以用来驱动系统时钟（SYSCLK）:  
- HSI 内部高速8MHz RC振荡器时钟  
- HSE 外部高速振荡器时钟  
- PLL时钟  
还有如下几种附加时钟源：  
- 40kHz内部低速RC振荡器（LSI），用于驱动独立看门狗（IWDG），还可以用于从停机或待机模式自动唤醒的RTC时钟。  
- 32.768kHz外部低速晶振（LSE），可以用于RTC时钟（RTCCLK）。  
- 14MHz高速内部RC振荡器（HSI14），专用于ADC。  
每一个时钟源都可以独立开关，如果它没有被使用，以降低功耗。  
有几个预分频器用于配置AHB和APB域的时钟，AHB和APB域的时钟最大频率是48MHz。  
所有的外设时钟由其所在的总线时钟（HCLK-AHB时钟或PCLK-APB时钟）驱动，除了：  
- FLASH接口时钟（FLITFCLK）总是由HSI驱动。  
- 选项字节装载器时钟也总是由HSI驱动。  
- ADC时钟可由软件选择HSI14（运行在最大采样速率），PCLK的2或4分频。  
- USART1时钟可由软件选择系统时钟（SYSCLK）,HSI,LSE或PCLK。  
- I2C1时钟可由软件选择系统时钟（SYSCLK）或HSI。  
- USB时钟由PLL时钟提供。  
- RTC时钟可选择LSE,LSI或是HSE/32。  
- 定时器频率由硬件自动选择，有2种情况：  
　　- 如果APB预分频器是1，则定时器时钟频率和APB域时钟PCLK一样。  
　　- 否则，定时器的时钟频率是APB域时钟PCLK频率的2倍。  
- IWDG时钟总是由LSI驱动。  
RCC将AHB时钟的8分频（HCLK/8）提供给Cortex系统定时器，作为其外部时钟。通过配置SysTick控制和状态寄存器，SysTick可以选择HCLK/8或直接用Cortex时钟（HCLK）作为时钟。  
![](https://i.imgur.com/tP9JwFB.png)  
![](https://i.imgur.com/894Bcuf.png)  

####HSE clock 外部高速时钟  
HSE时钟信号可由以下2种时钟源产生：  
- 外部高速晶振  
- 外部用户高速时钟  
为了减少输出失真和启动稳定时间，晶振和负载电容必须尽可能的靠近MCU振荡器引脚。负载电容的容值必须和所用晶振匹配。  
![](https://i.imgur.com/FphuZN4.png)  
#####外部晶振  
4到32MHz的晶振可以为系统提供精确的主时钟。  
时钟控制寄存器（RCC_CR）的HSERDY位指示HSE时钟是否稳定。启动时，直到HSERDY被硬件置位，HSE才可以使用。如果，时钟中断寄存器（RCC_CIR）的HSERDYIE被置位，则当HSE稳定时，会产生中断。  
HSE时钟可以设置RCC_CR的HSEON位来开关。  
######HSE start sequence code example  

	/**
	* Description: This function enables the interrupt on HSE ready,
	* and start the HSE as external clock.
	*/
	__INLINE void StartHSE(void)
	{
		/* Configure NVIC for RCC */
		/* (1) Enable Interrupt on RCC */
		/* (2) Set priority for RCC */
		NVIC_EnableIRQ(RCC_CRS_IRQn); /* (1)*/
		NVIC_SetPriority(RCC_CRS_IRQn,0); /* (2) */
		/* (1) Enable interrupt on HSE ready */
		/* (2) Enable the CSS
			   Enable the HSE and set HSEBYP to use the external clock
			   instead of an oscillator
			   Enable HSE */
		/* Note : the clock is switched to HSE in the RCC_CRS_IRQHandler ISR */
		RCC->CIR |= RCC_CIR_HSERDYIE; /* (1) */
		RCC->CR |= RCC_CR_CSSON | RCC_CR_HSEBYP | RCC_CR_HSEON; /* (2) */
	}
	/**
	* Description: This function handles RCC interrupt request
	* and switch the system clock to HSE.
	*/
	void RCC_CRS_IRQHandler(void)
	{
		/* (1) Check the flag HSE ready */
		/* (2) Clear the flag HSE ready */
		/* (3) Switch the system clock to HSE */
		if ((RCC->CIR & RCC_CIR_HSERDYF) != 0) /* (1) */
		{
			RCC->CIR |= RCC_CIR_HSERDYC; /* (2) */
			RCC->CFGR = ((RCC->CFGR & (~RCC_CFGR_SW)) | RCC_CFGR_SW_0); /* (3) */
		}
		else
		{
			/* Report an error */
		}
	}  

注意：HSEON置位后，内部稳定计数器需要读到512个HSE时钟脉冲，开启HSE振荡器。在没有晶振连接到MCU的情况下，OSC_IN引脚上过大的噪声也可能导致振荡器启动。一旦振荡器启动，振荡器需要6个HSE时钟脉冲来完成关闭流程。不管任何原因，如果此时在OSC_IN引脚上不再有波动，振荡器将不能关闭，从其他任何用途锁定OSC引脚，并引入了不必要的功耗。为了避免这种情况，强烈建议总是开启时钟安全系统（CSS），在这种情况下，它也能关闭振荡器。  
#####外部时钟源（HSE旁路）  
在这种模式，必须提供一个外部的时钟源；它的频率可以高达32MHz。通过设置时钟控制寄存器（RCC_CR）的HSEBYP和HSEON位，选择使用外部时钟源。OSC_IN必须输入占空比40%-60%（取决于频率，参考数据手册）的外部时钟信号（方波，正弦波或三角波），而OSC_OUT可以作为GPIOS使用。  

####HSI clock 内部高速时钟  
HSI时钟信号由内部8MHzRC振荡器产生，可以用作系统时钟或PLL输入。  
HSI作为时钟源的优点在于：低成本（不需要外部元件），更快的启动时间（比HSE）。缺点在于：即使经过校准，HSI的频率精确度也比HSE差。  
#####校准  
在制造过程中的变化，会导致每一片芯片的RC振荡器的频率都不想同，所以，每片芯片在出厂前校准到1%（25°C）的精度。  
系统复位后，工厂校准值会被载入时钟控制寄存器（RCC_CR）的HSICAL[7:0]位。  
不同的应用电压或环境温度会影响RC振荡器的精度，可以通过时钟控制寄存器（RCC_CR）的HSITRIM[4:0]来调整HSI的频率。  
时钟控制寄存器（RCC_CR）的HSIRDY位指示HSI是否稳定。启动时，直到HSIRDY被硬件置位，HSI才可以使用。  
设置时钟控制寄存器（RCC_CR）的HSION位可以开关HSI。  
HSI还可以用作HSE失效时的备用时钟源。参考时钟安全系统（CSS）。  
可以用HSI驱动MCO多路复用器，然后用定时器14测量HSI信号，用户可以使用此方法观察HSI的频率，并校准。  

####PLL  
PLL可以由HSI或HSE倍频得到。PLL配置（选择时钟源，分频和倍频因子）必须在使能PLL前配置好。一旦PLL使能，配置的这些参数就不能再改变。  
要改变PLL的配置，需要遵循以下步骤：  
- 将PLLON位清零，关闭PLL  
- 等待PLLRDY清零，PLL彻底关闭  
- 改变所需参数  
- 将PLLON置1，打开PLL  
- 等待PLLRDY置位，PLL稳定可以使用  
时钟中断寄存器（RCC_CIR）的PLLRDYIE置位，当PLL稳定时，会触发中断。  
PLL输出频率必须在16-48MHz的范围内。  
######PLL configuration modification code example  

	/* (1) Test if PLL is used as System clock */
	/* (2) Select HSI as system clock */
	/* (3) Wait for HSI switched */
	/* (4) Disable the PLL */
	/* (5) Wait until PLLRDY is cleared */
	/* (6) Set the PLL multiplier to 6 */
	/* (7) Enable the PLL */
	/* (8) Wait until PLLRDY is set */
	/* (9) Select PLL as system clock */
	/* (10) Wait until the PLL is switched on */
	if ((RCC->CFGR & RCC_CFGR_SWS) == RCC_CFGR_SWS_PLL) /* (1) */
	{
		RCC->CFGR &= (uint32_t) (~RCC_CFGR_SW); /* (2) */
		while ((RCC->CFGR & RCC_CFGR_SWS) != RCC_CFGR_SWS_HSI) /* (3) */
		{
			/* For robust implementation, add here time-out management */
		}
	}
	RCC->CR &= (uint32_t)(~RCC_CR_PLLON);/* (4) */
	while((RCC->CR & RCC_CR_PLLRDY) != 0) /* (5) */
	{
		/* For robust implementation, add here time-out management */
	}
	RCC->CFGR = RCC->CFGR & (~RCC_CFGR_PLLMUL) | (RCC_CFGR_PLLMUL6); /* (6) */
	RCC->CR |= RCC_CR_PLLON; /* (7) */
	while((RCC->CR & RCC_CR_PLLRDY) == 0) /* (8) */
	{
		/* For robust implementation, add here time-out management */
	}
	RCC->CFGR |= (uint32_t) (RCC_CFGR_SW_PLL); /* (9) */
	while ((RCC->CFGR & RCC_CFGR_SWS) != RCC_CFGR_SWS_PLL) /* (10) */
	{
		/* For robust implementation, add here time-out management */
	}  

####LSE时钟  
LSE由32.768kHz的晶振或外部时钟信号提供。LSE为RTC或其他定时功能提供了低功耗且高精度的时钟源。  
设置RTC域控制寄存器（RCC_BDCR）的LSEON位可以开关LSE。综合考虑鲁棒性，启动时间和功耗，配置RCC_BDCR的LSEDRV[1:0]，选择LSE的驱动能力。  
RCC_BDCR的LSERDY位指示LSE是否稳定。在启动时，直到LSERDY被硬件置位，LSE才可以使用。如果，时钟中断寄存器（RCC_CIR）的LSERDYIE被置位，则，LSERDY=1时，会产生中断。  
注意：当LSEON被置位，内部稳定计数器需要读到4096个LSE时钟信号脉冲，才开启LSE振荡器。在器件未连接晶振的情况下，OSC32_IN引脚上的强干扰信号也有可能导致振荡器开启。一旦振荡器开启，就需要6个LSE时钟脉冲，才能关闭它。此时无论什么原因OSC32_IN引脚上没有波动，都会导致振荡器无法关闭，OSC32将无法用作其他用途，并且会产生多余的功耗。从这种情况恢复的唯一方法，是通过软件执行RTC域复位。  
#####外部低速时钟源（LSE旁路）  
在这种模式，需要由外部提供一个频率可以高达1MHz的时钟源。可以通过置位RCC_BDCR的LSEBYP和LSEON位，选择使用外部时钟源。OSC32_IN需要输入占空比50%的外部时钟信号（方波，正弦波或三角波），OSC32_OUT可以用作GPIO。  

####LSI内部低速时钟  
LSI是一个低功耗的时钟源，可以在停机或待机模式保持运行，为IWDG和RTC提供时钟信号。LSI时钟频率大概是40kHz。  
可以设置时钟控制与状态寄存器（RCC_CSR）的LSION位，开关LSI。  
RCC_CSR的LSIRDY位指示LSI是否稳定；在LSIRDY硬件置位前，LSI还未稳定，将不可以使用LSI。如果，时钟中断寄存器（RCC_CIR）的LSIRDYIE置位，则LSIRDY=1时，会产生中断。  

####SYSCLK系统时钟  
可以选择以下时钟源作为系统时钟SYSCLK：  
- HSI  
- HSE  
- PLL  
系统复位后，HSI作为SYSCLK。如果直接或间接使用PLL作为SYSCLK，PLL将不能停止。  
时钟切换只有当目标时钟源准备好（时钟经过启动延时后稳定或PLL锁定）才能进行。如果选择了未准备好的时钟源作为系统时钟，那么只有该时钟源准备好后，才真正执行切换到该时钟的操作。RCC_CR寄存器中的状态位指示出了时钟是否准备好以及当前系统采用哪个时钟源作为系统时钟。  

####CSS时钟安全系统  
CSS可以由软件激活。一旦CSS激活，时钟监测器将在HSE启动延时后被使能，当HSE被关闭，时钟监测器也会被关闭。  
一旦检测到HSE故障，HSE将被自动关闭，一个时钟失效事件将被送到高级定时器TIM1和通用定时器（TIM15,TIM16和TIM17）的刹车输入，并产生一个时钟安全系统中断CSSI，通知软件执行抢救工作。CSSI中断连接到ARM Cortex-M0 NMI(不可屏蔽中断)异常向量。  
注意：一旦CSS使能并且发生HSE时钟错误，CSS中断就会产生并且自动产生NMI。在CSS中断挂起位被清零前，NMI会不断执行，因此，NMI中断服务程序必须将RCC_CIR的CSSC置1来清除CSS中断。  
如果HSE直接或间接用作系统时钟（间接意思是：HSE作为PLL输入时钟，而PLL用作系统时钟），HSE故障会导致系统时钟切换到HSI，并关闭HSE。如果HSE作为PLL的时钟，而PLL用作系统时钟，那么HSE故障，也会关闭PLL。  

####ADC时钟  
ADC时钟可以配置ADC_CFGR2选择时钟源，它可以选择专用的14MHzRC振荡器（HSI14）,PCLK/2或PCLK/4。HSI14可以配置成由ADC接口控制打开/关闭（自动关闭），或是一直开启。当PCLK作为ADC时钟时，HSI14不能被ADC打开。  

####RTC时钟  
RTCCLK时钟源可以HSE/32,LSE或LSI，由RCC_BDCR的RTCSEL[1:0]的配置来决定。除非RTC复位，否则此配置不能改变。为了正确操作RTC，RTCCLK频率必须配置成小于PCLK。  

####IWDG时钟  
如果IWDG由硬件选项或软件启动，则LSI被强制打开并且不能关闭，待LSI稳定后，提供给IWDG。  

####时钟输出  
microcontroller clock output(MCO)微控制器时钟输出，允许时钟信号输出到外部MCO引脚。MCO对应的GPIO必须配置成MCO模式。下列的时钟信号可以选择其中一个作为MCO时钟：  
- HSI14  
- SYSCLK  
- HSI  
- HSE  
- PLL/2或PLL（STM32F030x8不支持直接连接PLL）  
- LSE  
- LSI  
可以通过配置时钟配置寄存器（RCC_CFGR）的MCO[3:0]位，来选择。  
######MCO selection code example

	/* Select system clock to be output on the MCO without prescaler */
	RCC->CFGR |= RCC_CFGR_MCO_SYSCLK;  
在STM32F030x4, STM32F030x6, STM32F070x6, STM32F070xB and STM32F030xC器件上，RCC_CFGR寄存器多了一位PLLNODIV，来选择是PLL/2还是PLL连接到MCO。另外RCC_CFGR还增加了MCOPRE[2:0]来配置MCO时钟的几分频输出到MCO引脚。  

####TIM14测量内部/外部时钟  
使用TIM14通道1输入捕获功能可以间接测量所有板上时钟源的频率。  
![](https://i.imgur.com/hxjoiOc.png)  
TIM14的输入捕获通道连接一个GPIO或是MCU的内部时钟，由TIM14_OR寄存器的TI1_RMP[1:0]位选择，如下其中之一：  
- 连接到GPIO  
- 连接RTCCLK  
- 连接HSE/32  
- 连接MCO输出时钟  
######Clock measurement configuration with TIM14 code example  

	/**
	* Description: This function configures the TIM14 as input capture
	* and enables the interrupt on TIM14
	*/
	__INLINE void ConfigureTIM14asInputCapture(void)
	{
		/* (1) Enable the peripheral clock of Timer 14 */
		/* (2) Select the active input TI1,Program the input filter, and prescaler */
		/* (3) Enable interrupt on Capture/Compare */
		RCC->APB1ENR |= RCC_APB1ENR_TIM14EN; /* (1) */
		TIM14->CCMR1 |= TIM_CCMR1_IC1F_0 | TIM_CCMR1_IC1F_1 \
					  | TIM_CCMR1_CC1S_0 | TIM_CCMR1_IC1PSC_1; /* (2)*/
		TIM14->DIER |= TIM_DIER_CC1IE; /* (3) */
		/* Configure NVIC for TIM14 */
		/* (4) Enable Interrupt on TIM14 */
		/* (5) Set priority for TIM14 */
		NVIC_EnableIRQ(TIM14_IRQn); /* (4) */
		NVIC_SetPriority(TIM14_IRQn,0); /* (5) */
		/* (6) Select HSE/32 as input on TI1 */
		/* (7) Enable counter */
		/* (8) Enable capture */
		TIM14->OR |= TIM14_OR_TI1_RMP_1; /* (6) */
		TIM14->CR1 |= TIM_CR1_CEN; /* (7) */
		TIM14->CCER |= TIM_CCER_CC1E; /* (8) */
	}
	Note:The measurement is done in the TIM14 interrupt subroutine.  
#####校准HSI  
将LSE通过MCO连接到TIM14输入捕获通道1，主要的目的就是测量HSI（为此，HSI必须作为系统时钟）。在两个连续LSE时钟信号上升沿之间，计到的HSI时钟脉冲数，据此可以计算出HSI的周期。考虑到LSE高精度的优势（典型的达到百万分之几十），它可以用同样的分辨率来确定内部时钟频率，然后修改内部时钟的频率，补偿因制造过程和/或温度电压差异造成的频率偏差。   
为此，HSI有专门的用户可访问的校准位。  
相对测量的基本概念是（比如HSI/LSE的比率）：测量的精确度和2个时钟间的比率密切相关，比率越高，得到的测量结果越准确。  
如果LSE不能用，HSE/32是比较好的选择为了得到较好的测量精度。  
#####校准LSI  
LSI的校准方法和HSI一样，但是要改变参考时钟。必须将LSI连接到TIM14的输入捕获通道1，然后HSE作为系统时钟，2个LSI信号连续边沿之间所计到的HSE时钟脉冲数，据此可以计算出LSI的周期。  
同样的，HSE/LSI的比率越大，得到的测量结果越准确。  
#####校准HSI14  
由于HSI14频率比较高，不可能精确测量它。解决办法是：将HSE通过PLL倍频大到最大的48MHz作为TIM14时钟，并将HSI14经过分频后（设置较大的分频）作为TIM14输入捕获通道1的输入信号。按此配置，我们会得到27的比率；对于精确的校准，这个比率还是有些低。为了增加测量的精度，可以在多个TIM4周期后再计算HSI14的周期。在这种情况下，使用轮询处理捕获事件是必要的。  

###低功耗模式  
APB外设时钟和DMA时钟可以由软件关闭。  
在休眠模式CPU时钟是关闭的，存储器接口（FLASH和RAM接口）时钟可由软件设置在休眠模式中关闭。如果连接到AHB到APB桥的所有外设时钟被关闭，AHB到APB桥的时钟也将由硬件关闭。  
在停止模式，核心区域的所有时钟被关闭，PLL,HSI,HSI14和HSE也都被关闭。  
在待机模式，核心区域的所有时钟，PLL,HSI,HSI14和HSE也都被关闭。  
在停机模式，设置DBGMCU_CR寄存器的DBG_STOP位；或在待机模式，设置DBGMCU_CR寄存器的DBG_STANDBY位，打开调试功能。  
当系统被中断（停机模式）或复位（待机模式）唤醒，HSI将被选作系统时钟。  
如果正在操作FLASH或APB外设，需要等到操作完成，才能进入低功耗模式。  

####RCC寄存器  
#####时钟控制寄存器（RCC_CR）  
