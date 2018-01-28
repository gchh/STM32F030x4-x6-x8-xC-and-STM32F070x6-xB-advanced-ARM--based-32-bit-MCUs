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

