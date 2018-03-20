#General-purpose timers(TIM15/16/17)  
STM32F030x4和STM32F030x6没有TIM15。  
##TIM15/16/17简介  
定时器TIM15/16/17包含一个由可编程预分频器驱动的16位自动重载计数器。  
它们可以用于各种目的，包括测量输入信号的脉冲宽度（输入捕获）或者产生输出波形（比较输出，PWM，带死区插入的互补PWM）。  
脉冲宽度和波形周期可以从几微秒到几毫秒之间调节，通过使用定时器的预分频器和RCC时钟控制器预分频器。  
TIM15/16/17是完全独立的，没有共享任何资源。所以它们可以和其他定时器同步操作。  
##TIM15主要特性  
- 16位自动重载向上计数器  
- 16位可编程预分频器可以将计数器时钟从1到65535之间分频（可以在运行中修改）  
- 2个独立通道，用于：  
　- 输入捕获  
　- 比较输出  
　- PWM生成（边沿对齐模式）  
　- 单脉冲模式输出  
- 带可编程死区时间的互补输出（仅通道1）  
- 使用外部信号控制定时器且可实现多个定时器互连的同步电路  
- 重复计数器，用于仅在给定数目的计数周期后更新定时器寄存器  
- 刹车输入，用于将定时器输出信号置于复位状态或一个已知状态  
- 下列事件将产生中断/DMA：  
　- 更新：计数器溢出，计数器初始化（由软件或内部/外部触发器引起）  
　- 触发事件（计数器启动，停止，初始化或由内部/外部触发计数）  
　- 输入捕获  
　- 比较输出  
　- 刹车输入（中断请求）  
![](https://i.imgur.com/dswHgk8.png)  
##TIM16和TIM17的主要特性  
- 16位自动重载的向上计数器  
- 将时钟频率在1到65535之间分频的16位可编程预分频器（可以在运行中修改）  
- 1个通道，用于：  
　- 输入捕获  
　- 比较输出  
　- PWM产生（边沿对齐模式）  
　- 单脉冲模式输出  
- 带有可编程死区时间的互补输出  
- 重复计数器，用于仅在经过给定数目的计数周期后才更新定时器寄存器  
- 刹车输入，用于将输出信号置于复位状态或一个已知状态  
- 下列事件产生中断/DMA：  
　- 更新：计数器溢出  
　- 输入捕获  
　- 比较输出  
　- 刹车输入  
![](https://i.imgur.com/SGIVU0T.png)  
##TIM15/16/17功能描述  
###时基单元  
定时器的主要模块是一个16位计数器以及它的自动重载寄存器。该计数器只可以向上计数。计数器时钟由预分频器提供。  
计数器，自动重载寄存器和预分频器寄存器可以由软件进行读写，即使计数器在运行中。  
时基单元包括：  
- 计数器寄存器TIMx_CNT  
- 预分频器寄存器TIMx_PSC  
- 自动重载寄存器TIMx_ARR  
- 重复计数器寄存器TIMx_RCR  
自动重载寄存器是预装载的。对自动自动重载寄存器进行读写实际是访问预装载寄存器。预装载寄存器的内容是立即传送给影子寄存器还是在每次更新事件UEV时传送给影子寄存器，取决于TIMx_CR1中的自动重载预装载使能位ARPE的设置。当计数器溢出时且TIMx_CR1中的UDIS=0，更新事件就将产生。更新事件也可以由软件设置产生。  
计数器的时钟由预分频器的输出CK_CNT提供，只有当TIMx_CR1中的计算器使能位CEN=1时CK_CNT才有效。  
注意，在TIMx_CR1中的CEN=1一个时钟周期后，计算器才开始计数。  
####预分频器描述  
预分频器可以将计数器时钟频率在1到65535之间任意分频。它是基于一个16位计数器通过一个16位寄存器（TIMx_PSC）控制的。它可以在运行中被改变，因为它具有缓冲。新的分频率将在更新事件时生效。  
图161和图162显示了当在运行中改变预分频器的值时，计数器的行为。  
![](https://i.imgur.com/ICWnTGd.png)  
![](https://i.imgur.com/0Y3YTfr.png)  
###计数模式  
TIM15/16/17的计数器只能向上计数，即从0计数到自动重载值（TIMx_ARR中的值）；然后重新从0开始计数，并产生一个计数器溢出事件。  
如果使用了重复计数器，计数器重复计数达到了重复计数器（TIMx_RCR）设置的次数时，发生更新事件。否则，每次计数器溢出都产生更新事件。  
设置TIMx_EGR中的UG=1（由软件或使用从模式控制器设置）也可以产生更新事件。  
如果TIMx_CR1中的UDIS=1，则更新事件UEV被禁止。这样可以避免在向预装载寄存器中写入新值过程中发生更新事件，导致影子寄存器中写入错误值。在UDIS=0之前，更新事件不会产生。但是，计数器和预分频器的计数器都会重新从0开始计数（分频率不会改变）。另外，如果TIMx_CR1中的URS=1，设置UG=1会产生更新事件UEV，但是不会置位UIF（即不会产生中断或DMA请求）。这是为了避免在清除捕获事件的计数器时，同时产生更新和捕获中断。  
当发生更新事件时，所有寄存器被更新，而TIMx_SR中的UIF是否置位由URS位决定：  
- 重复计数器重载入TIMx_RCR寄存器的值。  
- 自动重载影子寄存器更新为预装载值（TIMx_ARR）。  
- 预分频器的缓冲器重载入预装载值（TIMx_PSC的值）。  
下面的图显示了当TIMx_ARR=0x36时，不同时钟频率下，计数器的行为。  
![](https://i.imgur.com/M40RPo8.png)  
![](https://i.imgur.com/EMKGOAM.png)  
![](https://i.imgur.com/OJMp4Kc.png)  
![](https://i.imgur.com/EXV0YsW.png)  
![](https://i.imgur.com/D0FZVpe.png)  
![](https://i.imgur.com/6Sh28re.png)  
###重复计数器  
时基单元一节介绍了计数器溢出如何产生更新事件UEV。实际上，只有当重复计数器到达0时，才会产生更新事件。这可以用于输出PWM信号。  
这意味着，当TIMx_RCR=N时，计数器溢出N+1次才会产生更新事件，预装载寄存器的值才被载入影子寄存器中（TIMx_ARR,TIMx_PSC,比较模式下的TIMx_CCRx）。  
每次计数器溢出，重复计数器减1。  
重复计数器是自动重载的；重复率由TIMx_RCR定义。当软件设置TIMx_EGR中的UG=1或硬件通过从模式控制器产生更新事件，无论此时重复计数器的值是多少，都会立即发生更新事件，并且重复计数器重载入TIMx_RCR的值。  
![](https://i.imgur.com/3GqlSZa.png)  
###时钟源  
计数器时钟可以由下列时钟源提供：  
- 内部时钟CK_INT  
- 外部时钟模式1：外部输入引脚，仅TIM15可用此模式  
- 内部触发输入ITRx（同样只有TIM15可以选择此模式）：用一个定时器做另一个定时器的预分频器。例如，可以配置TIM1作为TIM15的预分频器。  
####内部时钟源CK_INT  
对于TIM15，如果从模式控制器被禁止（SMS=000），那么TIMx_CR1中的CEN和DIR位，以及TIMx_EGR中的UG位，是实际的控制位，且只能由软件修改（除了UG位是由硬件自动清零）。一旦CEN=1，内部始终CK_INT就提供给预分频器。  
TIM16/17的计数器时钟只由内部时钟提供。  
图170显示了向上计数器在正常模式和不分频的情况下的行为。  
![](https://i.imgur.com/CtWDiI2.png)  
####外部时钟模式1  
此模式只有TIM15可用。当TIMx_SMCR中的SMS=111时，选择此模式。计数器在选定的输入的每一次上升或下降沿计数。  
![](https://i.imgur.com/oNsARM2.png)  
TIM15没有ETR，所以上图出现的ETRF是应该去掉的。  
例如，配置计数器在TI2输入的上升沿计数，步骤如下：  
1. 配置通道2检测TI2输入的上升沿，通过设置TIMx_CCMR1中的CC2S=01，将通道2连接到TI2。  
2. 配置输入滤波带宽，通过设置TIMx_CCMR1中的IC2F[3:0]（如果不需要滤波，则保持IC2F=0000）。  
3. 选择上升沿极性，通过设置TIIMx_CCER中的CC2P=0和CC2NP=0。  
4. 配置定时器使用外部时钟模式1，通过设置TIMx_SMCR中的SMS=111。  
5. 选择TI2作为触发输入源，通过设置TIMx_SMCR中的TS=110。  
6. 启动计数器，通过设置TIMx_CR1中的CEN=1。  
注：捕获预分频器不用于触发操作，所以不需要设置它。  
######Upcounter on TI2 rising edge code example  

	/* (1) Enable the peripheral clock of Timer 15 */
	/* (2) Enable the peripheral clock of GPIOA */
	/* (3) Select Alternate function mode (10) on GPIOA pin 3 */
	/* (4) Select TIM15_CH2 on PA3 by enabling AF0 for pin 3 in GPIOA AFRH register */
	RCC->APB2ENR |= RCC_APB2ENR_TIM15EN; /* (1) */
	RCC->AHBENR |= RCC_AHBENR_GPIOAEN; /* (2) */
	GPIOA->MODER = (GPIOA->MODER & ~(GPIO_MODER_MODER9)) | (GPIO_MODER_MODER9_1); /* (3) */
	GPIOA->AFR[0] |= 0x2 << (3*4); /* (4) */
	/* (1) Configure channel 2 to detect rising edges on the TI2 input by writing CC2S = ‘01’,
           and configure the input filter duration by writing the IC2F[3:0] bits 
           in the TIMx_CCMR1 register (if no filter is needed, keep IC2F=0000).*/
	/* (2) Select rising edge polarity by writing CC2P=0 
           in the TIMx_CCER register (reset value). */
	/* (3) Configure the timer in external clock mode 1 by writing SMS=111
	       Select TI2 as the trigger input source by writing TS=110
	       in the TIMx_SMCR register.*/
	/* (4) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_IC2F_0 | TIM_CCMR1_IC2F_1 | TIM_CCMR1_CC2S_0; /* (1) */
	TIMx->CCER &= (uint16_t)(~TIM_CCER_CC2P); /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS | TIM_SMCR_TS_2 | TIM_SMCR_TS_1; /* (3) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (4) */  
当TI2上出现上升沿时，计数器就计一次数，并且TIF标志置1。  
从TI2出现上升沿到实际计数器计数之间的延时是由TI2输入端的重新同步电路造成的。  
![](https://i.imgur.com/ckwVgA9.png)  
###捕获/比较通道  
