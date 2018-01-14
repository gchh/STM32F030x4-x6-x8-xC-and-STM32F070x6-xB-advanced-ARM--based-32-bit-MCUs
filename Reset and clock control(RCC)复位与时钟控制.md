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
