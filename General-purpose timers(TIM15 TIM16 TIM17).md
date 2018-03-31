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
此功能用于控制输出波形或指示一段时间已经过去。  
当捕获/比较寄存器和计数器的内容匹配时，比较输出功能做出以下操作：  
- 相应的输出引脚上输出所配置的值，该值由比较输出模式（由TIMx_CCMRx中的OCxM设置）和输出极性（又TIMx_CCER中的CCxP设置）共同定义。如果OCxM=000，当比较匹配时，不会影响输出；如果OCxM=001，比较匹配时，输出置为有效电平；如果OCxM=010，则比较匹配时，输出置为无效电平。  
- 状态寄存器TIMx_SR中的中断标志位CCxIF置1。  
- 如果TIMx_DIER中的中断使能位CCxIE=1，则产生中断。  
- 如果TIMx_DIER中的DMA使能位CCxDE=1和TIMx_CR2中的捕获/比较DMA选择位CCDS=0，则发出DMA请求。  
TIMx_CCRx寄存器可以选择是否使用预装载寄存器，通过设置TIMx_CCMRx中的OCxPE位。  
在比较输出模式下，更新事件UEV不会影响OCxREF和OCx的输出。同步的精度可以达到计数器的一个计数周期。比较输出模式也可以用于输出单个脉冲（单脉冲模式）。  
比较输出模式的配置步骤：  
1. 选择计数器时钟（内部，外部，预分频器）。  
2. 向TIMx_ARR和TIMx_CCRx中写入所需数据。  
3. 如果需要中断，设置CCxIE=1。  
4. 选择输出模式。例如：  
　- OCxM=011，当CNT=CCRx时，OCx输出翻转  
　- OCxPE=0，禁止预装载寄存器  
　- CCxP=0，选择高电平有效  
　- CCxE=1，使能输出  
5. TIMx_CR1中的CEN=1，启动计数器。  
######Output compare configuration code example  

	/* (1) Set prescaler to 3, so APBCLK/4 i.e 12MHz */
	/* (2) Set ARR = 12000 -1 */
	/* (3) Set CCRx = ARR, as timer clock is 12MHz, an event occurs each 1 ms */
	/* (4) Select toggle mode on OC1 (OC1M = 011),
	       disable preload register on OC1 (OC1PE = 0, reset value) */
	/* (5) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1)*/
	/* (6) Enable output (MOE = 1)*/
	/* (7) Enable counter */
	TIMx->PSC |= 3; /* (1) */
	TIMx->ARR = 12000 - 1; /* (2) */
	TIMx->CCR1 = 12000 - 1; /* (3) */
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_0 | TIM_CCMR1_OC1M_1; /* (4) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (5)*/
	TIMx->BDTR |= TIM_BDTR_MOE; /* (6) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (7) */  
TIMx_CCRx寄存器的值可以随时更新以控制输出波形，但是前提是不使用预装载寄存器（OCxPE=0，否则，只有发生更新事件UEV时，TIMx_CCRx的影子寄存器才被更新）。图178给出了示例。  
![](https://i.imgur.com/pyJjCvu.png)  
###PWM输出模式  
PWM模式用于产生一个PWM信号，该信号由TIMx_ARR的值确定频率，由TIMx_CCRx的值确定占空比。  
每个输出通道可以独立选择PWM模式（每个OCx输出一个PWM信号），通过TIMx_CCMRx中的OCxM=110选择PWM模式1或OCxM=111选择PWM模式2。必须设置TIMx_CCMRx中的OCxPE=1，使能相应的TIMx_CCRx的预载寄存器；以及设置TIMx_CR1中的ARPE=1，使能TIMx_ARR的预载寄存器。  
因为只有发生更新事件时，预载寄存器的值才被拷贝到影子寄存器中，所以在启动计算器之前，必须通过设置TIMx_EGR中的UG=1来初始化所有寄存器。  
OCx输出的极性由TIMx_CCER中的CCxP选择，可以选择高电平有效或低电平有效。TIMx_CCER中的CCxE和CCxNE，以及TIMx_BDTR中的MOE,OSSI和OSSR，共同控制OCx的输出。  
无论PWM模式1还是模式2，TIMx_CNT都要和TIMx_CCRx进行比较，以确定是否TIMx_CCRx≤TIMx_CNT或TIMx_CNT≤TIMx_CCRx（取决于计数方向）。  
根据TIMx_CR1中的CMS位可以设置边沿对齐模式或中心对齐模式产生PWM。但是TIM15/16/17计数器只能向上计数，所以只能产生边沿对齐的PWM信号。  
####PWM边沿对齐向上计数模式  
以PWM模式1为例，当TIMx_CNT＜TIMx_CCRx时，PWM参考信号OCxREF为高电平（有效电平）；否则，为低电平。如果TIMx_CCRx大于TIMx_ARR，OCxREF会一直保持高电平。如果TIMx_CCRx=0，OCxREF会保持为低电平。  
图179显示了当TIMx_ARR=8时边沿对齐PWM的波形。  
![](https://i.imgur.com/AmRKMFy.png)  
######Edge-aligned PWM configuration example  

	/* (1) Set prescaler to 47, so APBCLK/48 i.e 1MHz */
	/* (2) Set ARR = 8, as timer clock is 1MHz the period is 9 us */
	/* (3) Set CCRx = 4, , the signal will be high during 4 us */
	/* (4) Select PWM mode 1 on OC1 (OC1M = 110),
	       enable preload register on OC1 (OC1PE = 1) */
	/* (5) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1)*/
	/* (6) Enable output (MOE = 1)*/
	/* (7) Enable counter (CEN = 1)
	       select edge aligned mode (CMS = 00, reset value)
	       select direction as upcounter (DIR = 0, reset value) */
	/* (8) Force update generation (UG = 1) */
	TIMx->PSC = 47; /* (1) */
	TIMx->ARR = 8; /* (2) */
	TIMx->CCR1 = 4; /* (3) */
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1PE; /* (4) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (5) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (6) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (7) */
	TIMx->EGR |= TIM_EGR_UG; /* (8) */  
###互补输出和死区插入  
通用定时器TIM15/16/17能够输出一路互补信号，并且管理输出的关断和接通瞬间。  
这段时间通常被称为死区，用户必须根据与输出连接的器件及其特性（电平转换的固有延时，开关器件产生的延时...）调整死区时间。  
每一路输出可以独立选择输出极性（主输出OCx或互补输出OCxN）。通过TIMx_CCER中的CCxP和CCxNP来选择极性。  
互补信号OCx和OCxN由几个控制位联合控制：TIMx_CCER中的CCxE和CCxNE，TIMx_BDTR中的MOE,OSSI和OSSR，以及TIMx_CR2中的OISx和OISxN。特别的是，当转换到IDLE状态时（MOE变为0），死区也会被激活。  
同时设置CCxE和CCxNE将插入死区，如果带有刹车电路，还需要设置MOE位。每个通道有一个10位的死区发生器。参考信号OCxREF产生2个输出OCx和OCxN。如果OCx和OCxN高电平有效：  
- OCx输出信号和参考信号OCxREF相同，只是它的上升沿相对参考信号的上升沿有延迟。  
- OCxN输出信号和参考信号OCxREF相反，只是它的上升沿相对参考信号的下降沿有延迟。  
如果延迟大于当前有效输出的宽度（OCx或OCxN），那么相应的脉冲就不会产生。  
下面的图显示了死区发生器的输出信号和参考信号OCxREF的关系（假定，CCxP=0,CCxNP=0,MOE=1,CCxE=1,CCxNE=1）。  
![](https://i.imgur.com/buvTqiD.png)  
![](https://i.imgur.com/iY0jF4J.png)  
![](https://i.imgur.com/TDbdmsL.png)  
每个通道的死区延迟是相同的，它由TIMx_BDTR中的DTG位配置。  
####将OCxREF重定向到OCx或OCxN  
在输出模式（强制，比较输出或PWM）下，通过配置TIMx_CCER中的CCxE和CCxNE位，可以将OCxREF重定向到OCx或OCxN输出。  
这个功能可以用于在一个输出上发送特定的波形（PWM或静态有效电平），而互补输出保持无效电平。或者使两个输出都为无效电平，或者使两个输出互补带死区并且同时都为有效电平。  
注：当只有OCxN被使能（OCxE=0,OCxNE=1）时，它是不互补的，一旦OCxREF变为高电平，OCxN就立即有效。例如，如果CCxNP=0，那么OCxN=OCxREF。另一方面，如果OCx和OCxN都被使能（OCxE=OCxNE=1），OCx在OCxREF为高电平时有效，而OCxN则相反，在OCxREF为低电平时有效。  
###使用刹车功能  
当使用刹车功能时，输出使能信号和无效电平根据额外的控制位（TIMx_BDTR中的MOE,OSSI和OSSR位，TIMx_CR2中的OISx和OISxN位）被修改。但是不管怎样，OCx和OCxN输出不能在同一时刻被设置为有效电平。  
刹车通道的输入（BRK）源可以来自于BKIN引脚上的外部信号，也可以来下列内部源：  
- 内核LOCKUP输出  
- PVD输出  
- SRAM奇偶校验错误信号  
- CSS检测到的时钟失败事件  
当退出复位状态后，刹车电路被禁止，MOE=0。通过设置TIMx_BDTR中的BKE=1,使能刹车功能。刹车输入信号的极性通过设置TIMx_BDTR中的BKP位选择。BKE和BKP可以同时修改。写入BKE和BKP，需要延迟1个APB时钟周期，写入才真正生效。因此，在写操作后，需要等待1个APB时钟周期，才能正确读取写入的位。  
因为MOE下降沿可能是异步的，所以在实际信号（作用于输出）和同步控制位（在TIMx_BDTR中）之间插入了重新同步电路。这就造成了在异步信号和同步信号之间的延迟。特别地，如果在MOE处于低电平时向其写入1，在正确读取它之前，你必须插入一个延时（空指令）。这是因为写入的是异步信号，而读取的却是同步信号。  
当发生刹车时（刹车输入脚上出现选定的电平）：  
- MOE位被异步地清零，将输出置于无效状态、空闲或复位状态（由OSSI位选择）。这个特性在MCU振荡器关闭时同样起作用。  
- 一旦MOE=0，每个输出通道的输出电平将由TIMx_CR2中的OISx位决定。如果OSSI=0，使能输出信号=0；否则，使能输出信号=1。  
- 当使用互补输出时：  
　- 输出首先被置于复位/无效状态（取决于极性）。此操作是异步的，所以即使不向定时器提供时钟，操作依然进行。  
　- 如果定时器时钟仍然在提供，那么死区发生器将重新激活，以便在死区后根据OISx和OISxN位设置的电平来驱动输出。即使在这种情况下，OCx和OCxN也不能同时为有效电平。注意由于要重新同步MOE，死区要比通常情况下长一些（大约2个ck_tim时钟周期）。  
　- 如果OSSI=0，定时器释放使能输出；否则，保持使能输出，或当CCxE和CCxNE其中一个为高时，使能输出变高。  
-  TIMx_SR中的刹车状态标志位BIF置1。如果TIMx_DIER中的BIE=1，将产生一个中断。  
-  如果TIMx_BDTR中的AOE=1，MOE位将在发生更新事件UEV时自动置位。这可以用于整形。否则，MOE保持为低直到向其写入1。这可以用于安全方面，比如可以将电源驱动器的警报、热敏传感器和其他安全器件的输出连接到刹车输入上。  
注：刹车输入是电平有效的。所以，当刹车输入有效时，MOE不能被置1（自动和软件都不行）。同时，BIF标志位不能被清零。  
刹车可以由BRK输入产生，可以编程选择不同极性，并由TIMx_BDTR中的BKE使能。  
除了刹车输入和输出管理，刹车电路还实现了对应用程序的写保护。它允许用户冻结几个配置参数（死区长度，OCx/OCxN极性和被禁止时的状态，刹车使能和极性）。通过TIMx_BDTR中的LOCK位的设置，从3级保护中选择一种。LOCK位只能在MCU复位后被写入一次。  
图183显示了响应刹车的输出行为。  
![](https://i.imgur.com/YNKeYrO.png)  
![](https://i.imgur.com/24tiGu9.png)  
###单脉冲模式  
单脉冲模式（OPM）是前述众多模式的一个特例。在这种模式下，计数器响应一个激励开始计数，并在一段可编程的延时后产生一个脉宽可编程的脉冲。  
可以通过从模式控制器控制计数器启动。在比较输出模式或PWM模式下产生波形。设置TIMx_CR1中的OPM=1，选择单脉冲模式。在此模式，计数器在更新事件UEV发生时自动停止。  
只有比较值和计数器初始值不同时，才能产生一个脉冲。在启动之前（当定时器等待触发时），必须如下配置：  
- 向上计数：CNT<CCRx≤ARR（特别地，0<CCRx）  
- 向下计数：CNT>CCRx  
![](https://i.imgur.com/6v6Rpnj.png)  
举例，在检测到TI2输入脚的上升沿时，延时t<sub>DELAY</sub>之后，在OC1上输出一个脉宽t<sub>PULSE</sub>的正脉冲。  
使用TI2FP2作为触发1：  
- 设置TIMx_CCMR1中的CC2S=01，把TI2FP2映射到TI2上。  
- 设置TIMx_CCER中的CC2P=0，选择检测TI2FP2的上升沿。  
- 设置TIMx_SMCR中的TS=110，选择TI2FP2作为从模式控制器的触发输入（TRGI）。  
- 设置TIMx_SMCR中的SMS=110，选择触发模式，TI2FP2用来启动计数器。  
######One-Pulse mode code example  

	/* The OPM waveform is defined by writing the compare registers */
	/* (1) Set prescaler to 47, so APBCLK/48 i.e 1MHz */
	/* (2) Set ARR = 7, as timer clock is 1MHz the period is 8 us */
	/* (3) Set CCRx = 5, the burst will be delayed for 5 us (must be > 0) */
	/* (4) Select PWM mode 2 on OC1 (OC1M = 111),
	       enable preload register on OC1 (OC1PE = 1, reset value)
	       enable fast enable (no delay) if PULSE_WITHOUT_DELAY is set */
	/* (5) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1) */
	/* (6) Enable output (MOE = 1) */
	/* (7) Write '1 in the OPM bit in the TIMx_CR1 register to stop the counter
	       at the next update event (OPM = 1),
	       enable auto-reload register(ARPE = 1) */
	TIMx->PSC = 47; /* (1) */
	TIMx->ARR = 7; /* (2) */
	TIMx->CCR1 = 5; /* (3) */
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1M_0
	             | TIM_CCMR1_OC1PE
	#if PULSE_WITHOUT_DELAY > 0
	             | TIM_CCMR1_OC1FE
	#endif
	             ; /* (4) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (5) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (6) */
	TIMx->CR1 |= TIM_CR1_OPM | TIM_CR1_ARPE; /* (7) */  
OPM波形由比较寄存器的值决定（要考虑时钟频率和计数器预分频器）。  
- t<sub>DELAY</sub>由TIMx_CCR1寄存器的值定义。  
- t<sub>PULSE</sub>由自动重载值和比较值的差值定义（TIMx_ARR-TIMx_CCR1）。  
- 假定想要在比较匹配时，输出波形从0跳转为1；而当计数器到达自动重载值时，输出波形从1跳转为0。要做到这样，需要设置TIMx_CCMR1中的OC1M=111，选择PWM模式2。如果需要，可以设置TIMx_CR1中的ARPE=1和TIMx_CCMR1中的OC1PE=1，使能相应的预装载寄存器。如果预装载寄存器被使能，在写入TIMx_CCR1和TIMx_ARR后，需要设置UG=1产生一个更新，然后等待TI2上的外部触发事件。在此例中，CC1P=0。  
在本例中，TIMx_CR1中的DIR和CMS位应该保持0（定时器TIM15/16/17无这两位）。  
由于只需要一个脉冲，所以设置TIMx_CR1中的OPM=1，当发生更新事件（当计数器从自动重载值返回0时）时计数器自动停止。  
####特殊情况：OCx快速使能  
在单脉冲模式下，TIx输入的边沿检测设置CEN位以启动计数器。计数器和比较值的比较结果造成输出跳变。但是这些操作需要几个时钟周期，并且这限制了我们能得到的最小延时t<sub>DELAY</sub>。  
如果要以最小的延时输出波形，可以设置TIMx_CCMRx中的OCxFE位。OCxREF（和OCx）被强制响应激励，而不依赖比较结果。其新电平和发生比较匹配时相同。仅当PWM1或PWM2模式下，OCxFE才起作用。  
###TIM15与外部触发同步  
这部分只应用在STM32F030x8,STM32F070xB和STM32F030xC。  
TIM15定时器能够在几种模式下和一个外部触发同步：复位模式，门控模式和触发模式。  
####从模式：复位模式  
当触发输入事件发生时，计数器和它的预分频器被重新初始化。此外，如果TIMx_CR1中的URS=0，还将产生一个更新事件UEV。那么，所有预装载寄存器（TIMx_ARR,TIMx_CCRx）被更新。  
在下面的例子中，TI1输入的上升沿导致向上计数器被清零：  
- 配置通道1检测TI1的上升沿。配置输入滤波带宽（在本例中，不需要滤波，所以保持IC1F=0000）。触发操作不需要使用捕获预分频器，所以不需要设置它。设置TIMx_CCMR1中的CC1S=01，选择TI1作为输入捕获源。设置TIMx_CCER中的CC1P=0，选择上升沿。  
- 设置TIMx_SMCR中的SMS=100，配置定时器为复位模式。设置TIMx_SMCR中的TS=101，选择TI1作为输入源。  
- 设置TTMx_CR1中的CEN=1，启动计数器。  
######Reset mode code example  

	/* (1) Configure channel 1 to detect rising edges on the TI1 input
	       by writing CC1S = ‘01’,
	       and configure the input filter duration by writing the IC1F[3:0]
	       bits in the TIMx_CCMR1 register
           (if no filter is needed, keep IC1F=0000).*/
	/* (2) Select rising edge polarity by writing CC1P=0 in the TIMx_CCER register
	       Not necessary as it keeps the reset value. */
	/* (3) Configure the timer in reset mode by writing SMS=100
	       Select TI1 as the trigger input source by writing TS=101
	       in the TIMx_SMCR register.*/
	/* (4) Set prescaler to 48000-1 in order to get an increment each 1ms */
	/* (5) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0; /* (1)*/
	TIMx->CCER &= (uint16_t)(~TIM_CCER_CC1P); /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_TS_2 | TIM_SMCR_TS_0; /* (3) */
	TIM1->PSC = 47999; /* (4) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (5) */  
计数器按内部时钟正常计数，直到出现TI1上升沿，计数器被清零然后从0开始重新计数。同时，TIMx_SR中的触发标志TIF被置1，并且产生中断或DMA请求（取决于TIMx_DIER中的TIE和TDE的设置）。  
下图显示了当TIMx_ARR=0x36时，在复位模式下，计数器的行为。从TI1上出现上升沿到计数器实际复位之间的延迟，是由TI1输入上的重新同步电路造成的。  
![](https://i.imgur.com/S9mPdj2.png)  
####从模式：门控模式  
计数器由选中的输入电平启动。  
在下面的例子中，当TI1输入低电平时计数器向上计数：  
- 配置通道1检测TI1上的低电平。配置输入滤波带宽（在此例中，不需要滤波，所以保持IC1F=0000）。触发操作不需要捕获预分频器，所以不用配置它。设置TIMx_CCMR1中的CC1S=01，选择TI1作为输入捕获源。设置TIMx_CCER中的CC1P=1，选择低电平。  
- 设置TIMx_SMCR中的SMS=101，配置定时器为门控模式。设置TIMx_SMCR中的TS=101，选择TI1作为触发输入源。  
- 设置TIMx_CR1中的CEN=1，使能计数器（在门控模式下，如果CEN=0，无论触发输入的电平如何，计数器都不会启动计数）。  
######Gated mode code example  

	/* (1) Configure channel 1 to detect low level on the TI1 input
	       by writing CC1S = ‘01’,
	       and configure the input filter duration by writing the IC1F[3:0]
	       bits in the TIMx_CCMR1 register (if no filter is needed, keep IC1F=0000). */
	/* (2) Select polarity by writing CC1P=1 in the TIMx_CCER register */
	/* (3) Configure the timer in gated mode by writing SMS=101
	       Select TI1 as the trigger input source by writing TS=101
	       in the TIMx_SMCR register. */
	/* (4) Set prescaler to 12000-1 in order to get an increment each 250us */
	/* (5) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0; /* (1)*/
	TIMx->CCER |= TIM_CCER_CC1P; /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_0 | TIM_SMCR_TS_2 | TIM_SMCR_TS_0; /* (3) */
	TIMx->PSC = 11999; /* (4) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (5) */  
当TI1为低电平时，计数器按内部时钟开始计数；当TI1变为高电平时，计数器停止计数。TIMx_SR中的TIF标志在计数器启动或停止时都会被置1。  
从TI1出现上升沿到计数器实际停止之间的延时，是由TI1输入上的重新同步电路造成的。  
![](https://i.imgur.com/FZjWx2w.png)  
####从模式：触发模式  
计数器由选中的输入事件启动计数。  
在下面的例子中，计数器在TI2输入上升沿时开始向上计数：  
- 配置通道2检测TI2上的上升沿。配置输入滤波带宽（在本例中，不需要滤波，所以保持IC2F=0000）。触发操作不使用捕获预分频器，所以不需要配置它。设置TIMx_CCMR1中的CC2S=01，选择TI2作为输入捕获源。设置TIMx_CCER中的CC2P=0，选择上升沿。  
- 设置TIMx_SMCR中的SMS=110，配置定时器为触发模式。设置TIMx_SMCR中的TS=110，选择TI2作为触发输入源。  
######Trigger mode code example  

	/* (1) Configure channel 2 to detect rising edge on the TI2 input
	       by writing CC2S = ‘01’,
	       and configure the input filter duration by writing the IC1F[3:0] bits
           in the TIMx_CCMR1 register (if no filter is needed, keep IC1F=0000). */
	/* (2) Select polarity by writing CC2P=0 (reset value) in the TIMx_CCER register */
	/* (3) Configure the timer in trigger mode by writing SMS=110
	       Select TI2 as the trigger input source by writing TS=110
	       in the TIMx_SMCR register. */
	/* (4) Set prescaler to 12000-1 in order to get an increment each 250us */
	TIMx->CCMR1 |= TIM_CCMR1_CC2S_0; /* (1)*/
	TIMx->CCER &= ~TIM_CCER_CC2P; /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1 | TIM_SMCR_TS_2 | TIM_SMCR_TS_1; /* (3) */
	TIM1->PSC = 11999; /* (4) */  
当TI2上出现上升沿时，计数器开始按内部时钟计数器，并且TIF标志被置1。  
从TI2出现上升沿到计数器实际开始计数之间的延时，是由TI2输入上的重新同步电路造成的。  
![](https://i.imgur.com/YDgmTAH.png)  
###定时器同步（TIM15）  
这部分只应用在STM32F030x8,STM32F070xB和STM32F030xC。  
定时器在内部相连，用于定时器同步或链接。参考General-purpose timer(TIM3)中的Timer synchronization章节，了解详情。  
###调试模式  
当MCU进入调试模式（Cortex-M0 core停止），TIMx计数器是继续正常工作还是停止，取决于DBG模块中的DBG_TIMx_STOP的设置。  
##TIM15寄存器  
###TIM15控制寄存器1（TIM15_CR1）  
![](https://i.imgur.com/7RBcOWd.png)  
![](https://i.imgur.com/z7izMto.png)  
###TIM15控制寄存器2（TIM15_CR2）  
![](https://i.imgur.com/saXciEd.png)  
![](https://i.imgur.com/Hniap0S.png)  
###TIM15从模式控制寄存器（TIM15_SMCR）  
![](https://i.imgur.com/8UCNLyQ.png)  
![](https://i.imgur.com/AOOROqm.png)  
![](https://i.imgur.com/1h11fZf.png)  
###TIM15 DMA/中断使能寄存器（TIM15_DIER）  
![](https://i.imgur.com/OJCSVVh.png)  
![](https://i.imgur.com/WjOcjP3.png)  
###TIM15状态寄存器（TIM15_SR）  
![](https://i.imgur.com/j4tcqhq.png)  
![](https://i.imgur.com/EisuauJ.png)  
![](https://i.imgur.com/23ZhSnA.png)  
###TIM15事件发生寄存器（TIM15_EGR）  
![](https://i.imgur.com/QrUtNl0.png)  
![](https://i.imgur.com/4Utt8iS.png)  
###TIM15捕获/比较模式寄存器1（TIM15_CCMR1）  
![](https://i.imgur.com/elyGzRw.png)  
![](https://i.imgur.com/GETRiaU.png)  
![](https://i.imgur.com/QfJ7PXa.png)  
![](https://i.imgur.com/I1MjKya.png)  
![](https://i.imgur.com/1rFzOB3.png)  
![](https://i.imgur.com/nLjxlm3.png)  
###TIM15捕获/比较使能寄存器（TIM15_CCER）  
![](https://i.imgur.com/g2kyFEv.png)  
![](https://i.imgur.com/9BiSf68.png)  
![](https://i.imgur.com/fhrYTpR.png)  
![](https://i.imgur.com/LXuoZsP.png)  
###TIM15计数器（TIM15_CNT）  
![](https://i.imgur.com/dviwbyN.png)  
###TIM15预分频器（TIM15_PSC）  
![](https://i.imgur.com/aHSX3N5.png)  
###TIM15自动重载寄存器（TIM15_ARR）  
![](https://i.imgur.com/tRH5xb6.png)  
###TIM15重复计数寄存器（TIM15_RCR）  
![](https://i.imgur.com/cqDFSd6.png)  
###TIM15捕获/比较寄存器（TIM15_CCR1）  
![](https://i.imgur.com/3Xj8FQx.png)  
###TIM15捕获/比较寄存器（TIM15_CCR2）  
![](https://i.imgur.com/5xemp6v.png)  
###TIM15刹车和死区寄存器（TIM15_BDTR）  
![](https://i.imgur.com/lmRbRQI.png)  
![](https://i.imgur.com/GnRBxbD.png)  
![](https://i.imgur.com/nU90XIT.png)  
![](https://i.imgur.com/JdFmq1i.png)  
![](https://i.imgur.com/Re559YY.png)  
###TIM15 DMA控制寄存器（TIM15_DCR）  
![](https://i.imgur.com/ijDa8Tl.png)  
![](https://i.imgur.com/pgdldZC.png)  
###TIM15 DMA全传输地址寄存器（TIM15_DMAR）  
![](https://i.imgur.com/IboOHJF.png)  
##TIM15寄存器映射  
![](https://i.imgur.com/yL9QBXm.png)  
![](https://i.imgur.com/XuMoW7x.png)  
![](https://i.imgur.com/jPqwR0H.png)  
![](https://i.imgur.com/GdnDtRf.png)  
##TIM16和TIM17寄存器  
###TIM16和TIM17控制寄存器1（TIM16_CR1和TIM17_CR1）  
![](https://i.imgur.com/9Ps6hQp.png)  
![](https://i.imgur.com/oRyysqM.png)  
###TIM16和TIM17控制寄存器2（TIM16_CR2和TIM17_CR2）  
![](https://i.imgur.com/QfonsrJ.png)  
![](https://i.imgur.com/iyzBTMg.png)  
###TIM16和TIM17 DMA/中断使能寄存器（TIM16_DIER和TIM17_DIER）  
![](https://i.imgur.com/DNrDpn5.png)  
###TIM16和TIM17状态寄存器（TIM16_SR和TIM17_SR）  
![](https://i.imgur.com/FHBhi8L.png)  
![](https://i.imgur.com/htt2qsa.png)  
###TIM16和TIM17事件发生寄存器（TIM16_EGR和TIM17_EGR）  
![](https://i.imgur.com/ujGgMFW.png)  
![](https://i.imgur.com/woeC0ve.png)  
###TIM16和TIM17捕获/比较模式寄存器1（TIM16_CCMR1和TIM17_CCMR1）  
![](https://i.imgur.com/yOufGAF.png)  
![](https://i.imgur.com/EwxYIsM.png)  
![](https://i.imgur.com/RNugEHo.png)  
![](https://i.imgur.com/6XASXYV.png)  
![](https://i.imgur.com/45dTrTH.png)  
###TIM16和TIM17捕获/比较使能寄存器（TIM16_CCER和TIM17_CCER）  
![](https://i.imgur.com/N9qVb3i.png)  
![](https://i.imgur.com/fQ4HtzY.png)  
![](https://i.imgur.com/JXVAnYg.png)  
![](https://i.imgur.com/sEynNWj.png)  
注：和OCx和OCxN通道相连的外部I/O引脚的状态，取决于OCx和OCxN通道的状态，以及GPIO和AFIO寄存器。  
###TIM16和TIM17计数器（TIM16_CNT和TIM17_CNT）  
![](https://i.imgur.com/lVM1YBQ.png)  
###TIM16和TIM17预分频器（TIM16_PSC和TIM17_PSC）  
![](https://i.imgur.com/7ALflkz.png)  
###TIM16和TIM17自动重载寄存器（TIM16_ARR和TIM17_ARR）  
![](https://i.imgur.com/OzbYNHw.png)  
###TIM16和TIM17重复计数寄存器（TIM16_RCR和TIM17_RCR）  
![](https://i.imgur.com/LsQgVgx.png)  
###TIM16和TIM17捕获/比较寄存器1（TIM16_CCR1和TIM17_CCR1）  
![](https://i.imgur.com/0lp9AHa.png)  
###TIM16和TIM17刹车和死区寄存器（TIM16_BDTR和TIM17_BDTR）  
![](https://i.imgur.com/HyJF6R2.png)  
![](https://i.imgur.com/EgBRfzN.png)  
![](https://i.imgur.com/itzRLkY.png)  
![](https://i.imgur.com/KWZ8E2A.png)  
###TIM16和TIM17 DMA控制寄存器（TIM16_DCR和TIM17_DCR）  
![](https://i.imgur.com/dkCnmIm.png)  
###TIM16和TIM17 DMA全传输地址（TIM16_DMAR和TIM17_DMAR）  
![](https://i.imgur.com/ALfAOQM.png)  
####如何使用DMA并发操作的例子  
在本例中使用定时器的DMA并发功能，以半字DMA传输，更新CCRx(x=2,3,4)寄存器。  
步骤如下：  
1. 如下配置相应的DMA通道：  
　- DMA通道外设地址为DMAR寄存器的地址  
　- DMA通道内存地址是存储DMA传输给CCRx寄存器的数据的RAM缓冲区的地址  
　- 传输数据个数=3（见下面的注）  
　- 禁止循环模式  
2. 配置DCR寄存器，如下：DBL=3次传输，DBA=0xE  
3. 设置TIMx_DIER中的UDE=1，使能TIMx更新DMA请求  
4. 使能TIMx  
5. 使能DMA通道  
注：在本例中每个CCRx被更新一次。如果每个CCRx寄存器需要更新2次，传输个数应该是6。假设RAM缓冲区包含数据data1,data2,data3,data4,data5,data6。在第一次更新DMA请求时，data1传输给CCR2，data2传输给CCR3，data3传输给CCR4。第二次更新DMA请求时，data4传输给CCR2，data5传输给CCR3，data6传输给CCR4。  
##TIM16和TIM17寄存器映射  
![](https://i.imgur.com/sxS9Iav.png)  
![](https://i.imgur.com/SKUviBB.png)  
![](https://i.imgur.com/XrnbhuS.png)  
