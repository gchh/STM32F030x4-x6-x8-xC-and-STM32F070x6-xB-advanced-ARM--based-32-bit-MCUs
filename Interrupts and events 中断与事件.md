#Interrupts and events  
##Nested vectored interrupt controller(NVIC)嵌套向量中断控制器  
###NVIC主要特性  
- 32个可屏蔽中断（不包括16个ARM Cortex-M0中断）  
- 4级可编程优先级（使用2bits设置）   
- 低延时的异常和中断处理  
- 电源管理控制  
- 系统控制寄存器的实现  
NVIC和处理器内核紧密耦合，实现低延时的中断处理和高效地处理晚到的中断。  
所有的中断包括内核异常都由NVIC管理。更多关于异常和NVIC编程的说明，可以参考PM0215编程手册。  
######NVIC initialization example  

	/* (1) Enable Interrupt on ADC */
	/* (2) Set priority for ADC to 2*/
	NVIC_EnableIRQ(ADC1_COMP_IRQn); /* (1) */
	NVIC_SetPriority(ADC1_COMP_IRQn,2); /* (2) */  

###系统定时器SysTick校准值寄存器  
SysTick校准值寄存器值设为6000，SysTick时钟设为6MHz（最大f<sub>HCLK</sub>/8）时，产生一个1ms时间基准。  

###中断和异常向量  
![](https://i.imgur.com/4dqjwA5.png)  
![](https://i.imgur.com/D87OrHQ.png)  
![](https://i.imgur.com/z8fvEfh.png)  

##外部中断和事件控制器（EXTI）  
外部中断和事件控制器（EXTI）管理外部和内部异步事件/中断，向CPU/中断控制器发送事件请求，并向电源管理器发送唤醒请求。  
EXTI管理多达28个外部/内部事件线（21个外部事件线和7个内部事件线）。  
每个外部中断线可以独立选择触发沿，而内部中断总数上升沿触发。一个中断可以一直挂起：如果是外部中断，状态寄存器会指出中断源；一个事件通常是一个简单的脉冲，用于唤醒内核（例如，Cortex-M0 RXEV pin）；对于内部中断，挂起状态由IP核确保，因此不需要特定的标志。可以单独屏蔽每条输入线产生中断或事件；另外，内部中断线只在停止模式被采样。控制器允许软件模拟外部事件，通过配置专门的寄存器，与相应的硬件事件线复用。  
###主要特性  
EXTI主要特性如下：  
- 支持产生多达32个事件/中断请求  
- 每个中断/事件线可独立屏蔽  
- 当系统不是停止模式，自动禁止内部中断/事件线  
- 每个外部中断/事件有专门的状态位  
- 可以模拟所有的外部事件请求  
###EXTI框图  
![](https://i.imgur.com/lOiu8i7.png)  
###事件管理  
STM32F0x0能够处理外部或内部事件唤醒内核（WFE）。唤醒事件可以由以下产生：  
- 在外设控制寄存器使能一个外部中断但不在NVIC中使能，打开Correx-M0系统控制寄存器的SEVONPEND位。当MCU从WFE唤醒后，需要清除EXTI外设中断挂起位和外设NVIC IRQ通道挂起位（在NVIC中断清除挂起寄存器）。  
- 或者，配置外部或内部事件线为事件模式。当CPU从WFE唤醒后，不需要清除相应的挂起位，因为对应的挂起位并没有被置位。  
###功能描述  
要产生外部中断，必须配置好外部中断线，并使能外部中断。根据需要配置上升或下降沿触发寄存器，相应的中断屏蔽寄存器中的中断屏蔽位置1允许中断请求。当在在外边中断线检测到设置的中断触发边沿，将产生一个中断请求，对应的挂起位置1。相应的挂起位写1，将清除该挂起位，舍弃该中断请求。  
对于内部中断，触发沿只能是上升沿。内部中断默认打开，在中断屏蔽寄存器，但内部中断在挂起寄存器没有对应的挂起位。  
要产生事件，需要配置好并使能事件线。根据需要配置上升或下降沿触发寄存器，相应事件屏蔽寄存器的位置1允许事件请求。当事件线上检测到设置的事件触发边沿，将产生一个事件脉冲，事件线相应的挂起位不会置1。  
注意：关联到内部线的中断或事件只有在停机模式才能被触发，如果系统在运行，将不会产生中断/事件。  
######External interrupt selection code example  

	/* (1) Enable the peripheral clock of GPIOA */
	/* (2) Select Port A for pin 0 external interrupt by writing 0000 in
	EXTI0 (reset value)*/
	/* (3) Configure the corresponding mask bit in the EXTI_IMR register */
	/* (4) Configure the Trigger Selection bits of the Interrupt line on
	rising edge*/
	/* (5) Configure the Trigger Selection bits of the Interrupt line on
	falling edge*/
	RCC->AHBENR |= RCC_AHBENR_GPIOAEN; /* (1) */
	//SYSCFG->EXTICR[1] &= (uint16_t)~SYSCFG_EXTICR1_EXTI0_PA; /* (2) */
	EXTI->IMR = 0x0001; /* (3) */
	EXTI->RTSR = 0x0001; /* (4) */
	EXTI->FTSR = 0x0001; /* (5) */
	/* Configure NVIC for External Interrupt */
	/* (1) Enable Interrupt on EXTI0_1 */
	/* (2) Set priority for EXTI0_1 */
	NVIC_EnableIRQ(EXTI0_1_IRQn); /* (1) */
	NVIC_SetPriority(EXTI0_1_IRQn,0); /* (2) */  
####硬件中断选择  
按以下步骤配置一个引脚作为中断源：  
- 配置EXTI_IMR中相应的屏蔽位  
- 配置EXTI_RTSR和EXTI_FTSR选择中断触发边沿  
- 配置映射到EXTI的NVIC IRQ通道的使能和屏蔽位，使EXTI可以被正确识别。  
####硬件事件选择  
按以下步骤配置一个引脚作为事件源：  
- 配置EXTI_EMR相应的屏蔽位  
- 配置EXTI_RTSR和EXTI_FTSR选择事件触发边沿  
####软件中断/事件选择  
任何外部中断/事件线都可以配置成软件中断/事件线。按如下步骤将产生软件中断：  
- 配置EXTI_IMR/EXTI_EMR相应的屏蔽位  
- 配置软件中断寄存器EXTI_SWIER相应的请求位  

###外部和内部中断/事件线映射  
