#GPIO  
每个GPIO口都有4个32-bit配置寄存器（GPIOx_MODER,GPIOx_OTYPER,GPIOx_OSPEEDR和GPIOx_PUPDR），2个32-bit数据寄存器（GPIOx_IDR,GPIOx_ODR）和一个32-bit的置位/复位寄存器（GPIOx_BSRR）。端口A和B还有1个32-bit的锁定寄存器（GPIOx_LCKR）和2个32-bit的复用功能选择寄存器（GPIOx_AFRH和GPIOx_AFRL）。  
在STM32F030xB和STM32F030xC上，端口C和D也有2个32-bit的复用功能选择寄存器（GPIOx_AFRH和GPIOx_AFRL）。  
##GPIO主要特性  
- 输出状态：推挽，开漏，上拉或下拉  
- 输出数据来自输出数据寄存器（GPIOx_ODR），或是外设（复用功能输出）  
- 每个IO可以选速度  
- 输入状态：悬空，上拉或下拉，模拟  
- 输入数据来自输入数据寄存器（GPIOx_IDR），或外设（复用功能输入）  
- 置位/复位寄存器（GPIOx_BSRR）提供了对GPIOx_ODR的按位写访问  
- 锁定机制（GPIOx_LCKR）用来冻结端口A和B的配置  
- 模拟功能  
- 复用功能选择寄存器（最多每个IO有16个复用功能）  
- 每2个时钟周期快速切换的能力  
- 高度灵活的引脚复用：可以作为GPIO使用或其他外设功能  
##GPIO功能描述  
根据数据手册列出的每个IO口的硬件特性，每一个IO口可以单独通过软件设置成下面几种模式：  
- 浮空输入  
- 上拉输入  
- 下拉输入  
- 模拟输入/输出  
- 上拉或下拉开漏输出  
- 上拉或下拉推挽输出  
- 上拉/下拉推挽复用功能输出  
- 上拉/下拉开漏复用功能输出  
每个IO口可以自由编程，但是IO口寄存器必须按字，半字或字节访问。GPIOx_BSRR和GPIO_BRR可以按位读/修改GPIOx_ODR。在这种情况下，在读/修改过程中发生IRQ也不会发生危险。  
![](https://i.imgur.com/1yETYYN.png)  
![](https://i.imgur.com/7vPHrlb.png)  
![](https://i.imgur.com/097pYt0.png)  
###GPIO  
在复位期间或刚复位，复用功能还未打开，并且大多数IO口被配置成浮空输入。  
调试引脚复位后被配置成复用AF PU/AF PD：  
- PA14：SWCLK下拉  
- PA13：SWDIO上拉  
当IO配置成输出脚，写入GPIOx_ODR的值将输出到相应引脚上。因此，可以使用推挽或开漏（ODR=0引脚输出低电平，ODR=1时引脚是高阻态）输出驱动外部电路。  
GPIOx_IDR在每个AHB时钟周期捕捉IO引脚上的数据。  
所有IO口带内部弱上拉和下拉，设置GPIOx_PUPDR寄存器使用或关闭它们。  
###IO口的复位功能和映射  
IO口通过多路复用器连接到内置的外设或模块上，同一时刻只能连接一个复用功能外设。因此，在同一个IO口不能设置相冲突的功能。  
每个IO口有一个多达16个复用功能输入（AF0-AF15）的多路复用器，通过GPIOx_AFRL(for pin0 to 7)和GPIOx_AFRH(for pin8 to 15)寄存器配置IO口功能：  
- 复位后，多路复用器选择AF0；IO口通过配置GPIOx_MODER寄存器选择工作模式  
- 每个引脚具体的复位功能在器件数据手册上有详细介绍  
除了这种灵活的IO复用结构，每个外设还可以映射到不同的IO引脚，这样可以使更小的封装有更多的外设使用。  
配置IO口：  
- 调试功能：在复位后调试相关的引脚由系统立即配置成调试口  
- GPIO：由GPIOx_MODER配置成输出，输入或模式口  
- 外设复用功能：  
　- 配置GPIOx_AFRL或GPIOx_AFRH选择IO所需的AFx功能  
　- 配置GPIOx_OTYPER选择输出类型（推挽或开漏），配置GPIOx_PUPDR选择上拉/下拉输出，配置GPIOx_OSPEEDR选择输出速度  
　- 配置GPIOx_MODER使IO口为复用功能  
- 附加功能：  
　- 对于ADC，在GPIOx_MODER中将IO配置成模拟口，并在ADC寄存器中配置所需功能  
　- 对于附加功能如RTC,WKUPx和振荡器，在相关的RTC,PWR和RCC寄存器中配置。这些功能优先级高于GPIO寄存器配置。  
参考数据手册的“复用功能映射”表，了解详细的IO引脚复用功能。  
###IO口控制寄存器  
每个GPIO端口有4个32-bit寄存器（GPIOx_MODER,GPIOx_OTYPER,GPIOx_OSPEEDR,GPIOx_PUPDR）可以配置16个IO口。GPIOx_MODER选择IO口工作模式（input，output，AF，analog）。GPIOx_OTYPER和GPIOx_OSPEEDR选择输出类型（推挽/开漏）和速度。GPIOx_PUPDR选择上拉/下拉不管IO口方向。  
###IO口数据寄存器  
每个GPIO端口有2个数据寄存器：GPIOx_IDR和GPIOx_ODR。GPIOx_ODR存放输出数据，可读可写。GPIOx_IDR存放输入数据，只可读。  
###IO数据按位处理  
GPIOx_BSRR置位复位寄存器，允许应用程序对GPIOx_ODR的每一位置位或复位。GPIOx_BSRR的有效数据宽度是GPIOx_ODR的2倍。  
GPIOx_ODR的每一bit，对应GPIOx_BSRR的2个bit：BS(i)和BR(i)。BS(i)=1，对应ODR(i)=1；BR(i)=1，对应ODR(i)=0。  
GPIOx_BSRR的任何bit=0，不会影响GPIOx_ODR中相应bit的值。如果GPIOx_BSRR中的BS(i)=1，并且BR(i)=1，而置位操作优先级高于复位操作，所以GPIOx_ODR中相应bit置位。  
GPIOx_BSRR对GPIOx_ODR的置位/复位是一次性的，并不会将GPIOx_ODR中的bit的值锁定。任何时候都可以通过GPIOx_ODR改变每一bit的值。GPIOx_BSRR只是提供按位操作的方式。  
不需要关闭中断，当通过GPIOx_BSRR按位置位/复位GPIOx_ODR时：可以在一个AHB写访问周期改变1或多位。  
###GPIO锁定机制  
通过对GPIOx_LCKR特定的操作步骤，可以冻结端口A和B的控制寄存器：GPIOx_MODER,GPIOx_OTYPER,GPIOx_OSPEEDR,GPIOx_PUPDR,GPIOx_AFRL和GPIOx_AFRH。  
为了写GPIOx_LCKR，必须执行特定写/读步骤。当正确的锁定步骤应用于GPIOx_LCKR的bit16，LCKR[15:0]用于锁定IO口的配置（在执行锁定步骤期间，LCKR[15:0]的值必须保持不变）。当IO口被锁定，IO口的值将不能修改直到MCU复位或外设复位。GPIOx_LCKR的每一位冻结对应的控制寄存器的位。  
在执行锁定步骤时，GPIOx_LCKR必须按字写入；因为bit16要和bits[15:0]同时设置。  
###IO复用功能输入/输出  
一个GPIO可以有多个外设功能，由GPIOx_AFRL和GPIOx_AFRH配置选择。  
######Alternate function selection sequence code example  

	/* This sequence select AF2 for GPIOA4, 8 and 9. This can be easily adapted
	with another port by changing all GPIOA references by another GPIO port,
	and the alternate function number can be changed by replacing 0x04 or 0x02 for
	each pin by the targeted alternate function in the 2 last code lines. */
	/* (1) Enable the peripheral clock of GPIOA */
	/* (2) Select alternate function mode on GPIOA pin 4, 8 and 9 */
	/* (3) Select AF4 on PA4 in AFRL for TIM14_CH1 */
	/* (4) Select AF2 on PA8 and PA9 in AFRH for TIM1_CH1 and TIM1_CH2 */
	RCC->AHBENR |= RCC_AHBENR_GPIOAEN; /* (1) */
	GPIOA->MODER = (GPIOA->MODER & ~(GPIO_MODER_MODER4 | GPIO_MODER_MODER8 | GPIO_MODER_MODER9)) 
				   | GPIO_MODER_MODER4_1 | GPIO_MODER_MODER8_1 | GPIO_MODER_MODER9_1; /* (2) */
	GPIOA->AFR[0] |= 0x04 << GPIO_AFRL_AFRL4_Pos; /* (3) */
	GPIOA->AFR[1] |= (0x02 << GPIO_AFRL_AFRH8_Pos) | (0x02 << GPIO_AFRL_AFRH9_Pos); /* (4) */  
###外部中断/唤醒引脚  
所有端口都有外部中断的能力。作为外部中断使用的引脚，不能配置成模拟口或用作振荡器引脚，需要打开输入触发器。  
###输入配置  
当IO口被配置为输入口：  
- 输出缓存器将关闭  
- 施密特触发器打开  
- 由GPIOx_PUPDR配置上拉/下拉  
- 每个AHB时钟周期，IO引脚上的数据被采样，存入GPIOx_IDR  
- 读取GPIOx_IDR的内容，可以知道IO口的状态  
![](https://i.imgur.com/V2zF1C9.png)  
###输出配置  
当IO口被配置为输出口：  
- 输出缓存器打开：  
　- 开漏模式：输出0，N-MOS打开，引脚低电平；输出1，N-MOS打不开，引脚处于高阻态。（P-MOS没有激活）  
　- 推挽模式：输出0，N-MOS打开，引脚低电平；输出1，P-MOS打开，引脚高电平  
- 施密特触发器打开  
- 由GPIOx_PUPDR配置上拉/下拉  
- 每个AHB时钟周期，IO引脚上的数据会被采集存入GPIOx_IDR  
- 读取GPIOx_IDR的内容，可以了解IO口目前的状态  
- 读取GPIOx_ODR的内容，可以获得最后写入的值  
![](https://i.imgur.com/9kAznMW.png)  
###复用功能配置  
当IO口配置成复用功能：  
- 输出缓冲器可以配置成推挽或开漏模式  
- 输出缓冲器由来自外设的信号驱动  
- 施密特触发器打开  
- 由GPIOx_PUPDR配置上拉/下拉  
- 每个AHB时钟周期，IO引脚上的数据将被采集，存入GPIOx_IDR  
- 读取GPIOx_IDR的数据，可以了解IO口目前的状态  
![](https://i.imgur.com/56wKO2j.png)  
###模拟配置  
当IO口被配置成模拟口：  
- 输出缓冲器关闭  
- 施密特触发器被禁止，实现了模拟IO口的0消耗。施密特触发器的输出被强制为0  
- 上拉和下拉被禁止  
- 读GPIOx_IDR得到的值是0  
######Analog GPIO configuration code example  

	/* (1) Enable the peripheral clock of GPIOA, GPIOB and GPIOC */
	/* (2) Select analog mode for PA1 */
	/* (3) Select analog mode for PB1 */
	/* (4) Select analog mode for PC0 */
	RCC->AHBENR |= RCC_AHBENR_GPIOAEN | RCC_AHBENR_GPIOBEN
	             | RCC_AHBENR_GPIOCEN; /* (1) */
	GPIOA->MODER |= GPIO_MODER_MODER1; /* (2) */
	GPIOB->MODER |= GPIO_MODER_MODER1; /* (3) */
	GPIOC->MODER |= GPIO_MODER_MODER0; /* (4) */  
![](https://i.imgur.com/Vh7dTCN.png)  
###HSE或LSE振荡器引脚用作GPIO  
当HSE或LSE振荡器关闭（复位后的默认状态），相关的引脚可以用作GPIO。  
当HSE或LSE振荡器打开（RCC_CSR的HSEON或LSEON置位开启振荡器），相关的引脚GPIO配置失效。  
当振荡器被配置使用外部时钟，余下的OSC_OUT或OSC32_OUT依然可以用作GPIO。  
###在RTC电源域使用GPIO  
当核心供电域断电（器件进入待机模式），PC13/PC14/PC15会失去GPIO功能。在这种情况下，如果它们的GPIO配置没有被RTC配置旁路，它们被设置成模拟输入模式。  
关于IO由RTC控制的详情，可以参考RTC功能描述。  
##GPIO寄存器  
###GPIO模式寄存器（GPIOx_MODER）(x=A..D,F)  
![](https://i.imgur.com/IGBck1l.png)  
###GPIO输出类型寄存器（GPIOx_OTYPER）(x=A..D,F)  
![](https://i.imgur.com/BSaynr2.png)  
###GPIO输出速度寄存器（GPIOx_OSPEEDR）(x=A..D,F)  
![](https://i.imgur.com/fkfSjih.png)  
###GPIO上拉/下拉寄存器（GPIOx_PUPDR）(x=A..D,F)  
![](https://i.imgur.com/2qEuv4J.png)  
###GPIO输入数据寄存器（GPIOx_IDR）(x=A..D,F)  
![](https://i.imgur.com/hhi28c9.png)  
###GPIO输出数据寄存器（GPIOx_ODR）(x=A..D,F)  
![](https://i.imgur.com/PLHsPvo.png)  
###GPIO置位/复位寄存器（GPIOx_BSRR）(x=A..D,F)  
![](https://i.imgur.com/7ySYl7j.png)  
![](https://i.imgur.com/B0mxQng.png)  
###GPIO配置锁定寄存器（GPIOx_LCKR）(x=A..B)  
![](https://i.imgur.com/yHTmYxL.png)  
![](https://i.imgur.com/IywzbUA.png)  
######Lock sequence code example  

	/**
	Description: This function locks the targeted pins of Port A configuration
	This function can be easily modified to lock Port B
	Parameter: lock contains the port pin mask to be locked*/
	void LockGPIOA(uint16_t lock)
	{
		/* (1) Write LCKK bit to 1 and set the pin bits to lock */
		/* (2) Write LCKK bit to 0 and set the pin bits to lock */
		/* (3) Write LCKK bit to 1 and set the pin bits to lock */
		/* (4) Read the Lock register */
		/* (5) Check the Lock register (optionnal) */
		GPIOA->LCKR = GPIO_LCKR_LCKK + lock; /* (1) */
		GPIOA->LCKR = lock; /* (2) */
		GPIOA->LCKR = GPIO_LCKR_LCKK + lock; /* (3) */
		GPIOA->LCKR; /* (4) */
		if ((GPIOA->LCKR & GPIO_LCKR_LCKK) == 0) /* (5) */
		{
			/* Manage an error */
		}
	}  
###GPIO复用功能低位寄存器（GPIOx_AFRL）(x=A..B)  
![](https://i.imgur.com/2AzYzhK.png)  
###GPIO复用功能高位寄存器（GPIOx_AFRH）(x=A..B)  
![](https://i.imgur.com/UwPUGe5.png)  
###GPIO复位寄存器（GPIOx_BRR）(x=A..D,F)  
![](https://i.imgur.com/Q32TSHW.png)  
##GPIO寄存器映射  
![](https://i.imgur.com/x8jUYUy.png)  
