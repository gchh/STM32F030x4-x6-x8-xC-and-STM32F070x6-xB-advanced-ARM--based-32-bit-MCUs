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
每一个捕获/比较通道都是建立在一个捕获/比较寄存器（包含一个影子寄存器），一个捕获输入级（带有数字滤波器，多路复用器和预分频器），以及一个输出级（带有比较器和输出控制）。  
图173到176给出了一个捕获/比较通道的概览。  
输入级采样相应的TIx输入信号产生一个滤波后的信号TIxF。然后，经过一个带极性选择的边沿检测器产生TIxFPx信号，此信号可以作为从模式控制器的触发输入或用作捕获控制。TIxFPx通过预分频器产生信号ICxPS进入捕获寄存器。  
![](https://i.imgur.com/7XfRlB4.png)  
输出级产生一个中间波形作为基准OCxREF（高电平有效）。链的末端决定输出信号的极性。  
![](https://i.imgur.com/5QK82IH.png)  
![](https://i.imgur.com/sQcPhn7.png)  
![](https://i.imgur.com/XH6tzlK.png)  
捕获/比较模块由一个预装载器和影子寄存器组成。读和写都是对预装载寄存器进行的。  
在捕获模式下，捕获实际是在影子寄存器中进行，然后影子寄存器的值拷贝到预装载寄存器。  
在比较模式下，预装载寄存器的内容拷贝到影子寄存器，然后影子寄存器和计数器进行比较。  
###输入捕获模式  
在输入捕获模式，捕获/比较寄存器TIMx_CCRx用来，当检测到相应的ICx信号上的跳变沿后，锁存当前计数器的值。当一个捕获发生时，TIMx_SR中相应的CCxIF标志置1；并且产生中断或DMA请求，如果它们使能的话。如果CCxIF=1时发生捕获，那么TIMx_SR中的重复捕获标志CCxOF会被置1。CCxIF可以通过软件向其写0或读取TIMx_CCRx中的捕获值，将其清零。CCxOF标志通过向其写入0，将其清零。  
下面的例子显示了当TI1输入上升沿时TIMx_CCR1如何捕获计数器。为此，需要步骤如下：  
- 选择有效输入：TIMx_CCR1必须连接到TI1输入，所以设置TIMx_CCMR1中的CC1S=01。一旦CC1S≠00，通道就配置为输入，并且TIMx_CCR1寄存器变为只读。  
- 根据输入信号的特点，配置输入滤波带宽（当输入是TIx时，通过TIMx_CCMRx中的ICxF配置）。假设，输入信号跳变时会在最多5个内部时钟周期内抖动，那么滤波带宽必须长于5个时钟周期。我们可以配置TIMx_CCMR1寄存器中的IC1F=0011，以f<sub>CK_INT</sub>为采样频率，连续采样8次，如果8次采样结果一致且是新的电平，则可以确认TI1上信号发生跳变。  
- 选择TI1的有效边沿，设置TIMx_CCER中的CC1P=0和CC1NP=0选择上升沿。  
- 配置输入预分频器。在本例中，我们希望在每一次的输入上升沿捕获，因此不需要使用预分频器，保持TIMx_CCMR1中的IC1PS=00。  
- 使能捕获，设置TIMx_CCER中的CC1E=1。  
- 如果需要，设置TIMx_DIER中的CC1IE=1和/或CC1DE=1，使能中断和/或DMA请求。  
######Input capture configuration code example  

	/* (1) Select the active input TI1 (CC1S = 01),
	       program the input filter for 8 clock cycles (IC1F = 0011),
	       select the rising edge on CC1 (CC1P = 0, reset value)
	       and prescaler at each valid transition (IC1PS = 00, reset value) */
	/* (2) Enable capture by setting CC1E */
	/* (3) Enable interrupt on Capture/Compare */
	/* (4) Enable counter */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0 | TIM_CCMR1_IC1F_0 | TIM_CCMR1_IC1F_1; /* (1)*/
	TIMx->CCER |= TIM_CCER_CC1E; /* (2) */
	TIMx->DIER |= TIM_DIER_CC1IE; /* (3) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (4) */  
当一个输入捕获发生时：  
- TIMx_CCR1捕获计数器当前的值。  
- CC1IF=1；如果CC1IF=1时发生了捕获，则CC1OF=1。  
- 如果CC1IE=1，则产生中断。  
- 如果CC1DE=1，则产生DMA请求。  
######Input capture data management code example  

	//This code must be inserted in the Timer interrupt subroutine.
	if ((TIMx->SR & TIM_SR_CC1IF) != 0)
	{
		if ((TIMx->SR & TIM_SR_CC1OF) != 0) /* Check the overflow */
		{
			/* Overflow error management */
			gap = 0; /* Reinitialize the laps computing */
			TIMx->SR &= ~(TIM_SR_CC1OF | TIM_SR_CC1IF); /* Clear the flags */
			return;
		}
		if (gap == 0) /* Test if it is the first rising edge */
		{
			counter0 = TIMx->CCR1; /* Read the capture counter which clears the CC1ICF */
			gap = 1; /* Indicate that the first rising edge has yet been detected */
		}
		else
		{
			counter1 = TIMx->CCR1; /* Read the capture counter which clears the CC1ICF */
			if (counter1 > counter0) /* Check capture counter overflow */
			{
				Counter = counter1 - counter0;
			}
			else
			{
				Counter = counter1 + 0xFFFF - counter0 + 1;
			}
			counter0 = counter1;
		}
	}
	else
	{
		/* Unexpected Interrupt */
		/* Manage an error for robust application */
	}
	/* Note:This code manages only a single counter overflow. 
       To manage many counter overflows the update interrupt
       must be enabled (UIE = 1) and properly managed. */  
为了处理重复捕获，建议在读取重复捕获标志前先读取捕获值。这是为了避免，在读取捕获溢出标志之后而在读取捕获数据之前的这段时间内，又发生捕获，而造成前次捕获数据丢失。  
注：设置TIMx_EGR中的CCxG=1，可以产生输入捕获中断和/或DMA请求（如果它们被使能的话）。  
###PWM输入模式（仅TIM15）  
该模式是输入捕获模式的一个特例。除了下列区别，操作与输入捕获模式相同：  
- 2个ICx信号映射到同一个TIx输入。  
- 这2个ICx信号的有效边沿极性相反。  
- 2个TIxFP信号其中一个选作触发输入，并且从模式控制器配置为复位模式。  
例如，测量TI1上的PWM信号的周期（用TIMx_CCR1）和占空比（用TIMx_CCR2），步骤如下（取决于CK_INT的频率和预分频器的值）：  
- 选择TIMx_CCR1的有效输入：TIMx_CCMR1中的CC1S=01，选择TI1。  
- 选择TI1FP1的有效极性（用于TIMx_CCR1捕获和计数器清零）：CC1P=0,CC1NP=0，选择上升沿有效。  
- 选择TIMx_CCR2的有效输入：TIMx_CCMR1的CC2S=10，选择TI1。  
- 选择TI1FP2的有效极性（用于TIMx_CCR2捕获）：CC2P=1,CC2NP=0，选择下降沿有效。  
- 选择有效的触发输入：TIMx_SMCR中的TS=101，选择TI1FP1。  
- 配置从模式控制器为复位模式：TIMx_SMCR中的SMS=100。  
- 使能捕获：TIMx_CCER中的CC1E=1,CC2E=1。  
######PWM input configuration code example  

	/* (1) Select the active input TI1 for TIMx_CCR1 (CC1S = 01),
	       select the active input TI1 for TIMx_CCR2 (CC2S = 10) */
	/* (2) Select TI1FP1 as valid trigger input (TS = 101)
	       configure the slave mode in reset mode (SMS = 100) */
	/* (3) Enable capture by setting CC1E and CC2E
	       select the rising edge on CC1 and CC1N (CC1P = 0 and CC1NP = 0, reset value),
	       select the falling edge on CC2 (CC2P = 1). */
	/* (4) Enable interrupt on Capture/Compare 1 */
	/* (5) Enable counter */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0 | TIM_CCMR1_CC2S_1; /* (1)*/
	TIMx->SMCR |= TIM_SMCR_TS_2 | TIM_SMCR_TS_0 | TIM_SMCR_SMS_2; /* (2) */
	TIMx->CCER |= TIM_CCER_CC1E | TIM_CCER_CC2E | TIM_CCER_CC2P; /* (3) */
	TIMx->DIER |= TIM_DIER_CC1IE; /* (4) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (5) */  
![](https://i.imgur.com/vt0rCFg.png)  
###强制输出模式  
在输出模式（TIMx_SMCR中的CCxS=00）下，每个比较输出信号（OCxREF及相应的OCx/OCxN）可以由软件直接强制为有效或无效电平，而不论输出比较寄存器和计数器的计较结果如何。  
要强制比较输出信号（OCxREF/0Cx）为其有效电平，只需要向TIMx_CCMRx中写入OCxM=101。这样OCxREF就被强制为高电平（因为OCxREF的有效电平总是高电平），而OCx的输出电平和CCxP设置的极性相反。  
例如：CCxP=0（OCx高电平有效）=> OCx被强制为高电平。  
OCxREF信号可以被强制为低电平，通过设置TIMx_CCMRx中的OCxM=100。  
不管怎样，TIMx_CCRx影子寄存器和计数器之间的比较会一直进行下去，并且置位相应的位。因此，同样可以产生中断和DMA请求。  
###比较输出模式  
