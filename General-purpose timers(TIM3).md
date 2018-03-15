#General-purpose timers(TIM3)  
##TIM3简介  
通用定时器包含一个由可编程预分频器提供始终的16位自动重装载计数器。  
它们可以用于各种目的，包括测量输入信号的脉冲宽度（输入捕获）或产生输出波形（输出比较，PWM）。  
脉冲宽度和波形周期可以从几微秒到几毫秒的调节，通过使用定时器预分频器和RCC时钟控制预分频器。  
定时器是完全独立的，没有共享任何资源，因此，它们可以同时进行操作。  
##TIM3主要功能  
通用定时器包括以下功能：  
- 16位（TIM3）向上，向下，上/下自动重装载计数器  
- 16位可编程预分频器（动态设置）将频率分频1到65535分之一作为计数器时钟  
- 4个独立通道，可以用作：  
　- 输入捕获  
　- 比较输出  
　- 生成PWM（边沿或中心对齐模式）  
　- 单脉冲模式输出  
- 如下事件产生中断/DMA请求：  
　- 更新：计数器向上或向下溢出，计数器初始化（由软件或内部/外部触发产生）  
　- 触发事件（由内部/外部触发导致的计数器启动，停止，初始化或计数）  
　- 输入捕获  
　- 比较输出  
- 支持定位用的增量（正交）编码器和霍尔传感器  
- 触发输入作为外部时钟或按周期的电源管理  
![](https://i.imgur.com/OkBYle2.png)  
![](https://i.imgur.com/s52vcH7.png)  
##TIM3功能描述  
###时基单元  
可编程通用定时器的主要模块是一个16位/32位的带自动重载寄存器的计数器。计数器可以向上，向下或双向计数。计数器时钟由预分频器分频得到。  
计数器，自动重载寄存器和预分频器寄存器，可以由软件进行读写。甚至在计数器运行中也可以进行读写。  
时基单元包括：  
- 计数器寄存器（TIMx_CNT）  
- 预分频器寄存器（TIMx_PSC）  
- 自动重载寄存器（TIMx_ARR）  
自动重载寄存器是预装载的。对自动重载寄存器进行读写实际是访问的它的预装载寄存器。根据TIMx_CR1中的自动重载预装载使能位ARPE的设置，预装载寄存器的内容立即写入影子寄存器中，或在每次更新事件UEV时写入。当计数器向上溢出或向下溢出，并且TIMx_CR1中的UDIS=0时，发出更新事件。更新事件也可以由软件设置产生。每种配置都会详细描述更新事件的产生。  
计数器由预分频器输出CK_CNT作为时钟，并且当TIMx_CR1中的CEN=1时使能计数器（参考从模式控制器的描述了解更多计数器使能的信息）。注意，在CEN=1一个时钟周期后，计数器实际的使能信号CNT_EN才置1。  
####预分频器描述  
预分频器可以按1到65535将计数器时钟分频。它是基于一个通过TIMx_PSC控制的16位计数器。它可以动态的改变，因为它带有缓冲。新的分频因子在下面发生更新事件时被采用。  
图89和图90给出了当分频因子改变时，计数器的行为。  
![](https://i.imgur.com/Dt4jpCY.png)  
![](https://i.imgur.com/HkEJFs5.png)  
###计数模式  
####向上计数  
在向上计数模式，计数器从0计数到TIMx_ARR中的值，然后重新从0计数，并且产生计数器溢出事件（更新事件）。  
每次计数器溢出或设置TIMx_EGR中的UG=1（由软件设置或从模式控制器），都会产生更新事件。  
设置TIMx_CR1中的UDIS=1，将禁止更新事件UEV产生。这可以避免在向预装载寄存器写入新值的过程中，发生更新事件，更新影子寄存器。直到UDIS=0，否则不会产生更新事件。然而，计数器依然会从0重新开始计数，同样的预分频器的计数器也正常计数（但是分频因子不会改变）。另外，如果TIMx_CR1中的URS(更新请求源选择)=1，设置UG=1将产生更新事件，但是不是置位UIF（也就不会产生中断或DMA请求）。这是为了避免在清除捕获事件的计数器时产生更新和捕获中断。  
当更新事件发生时，所有寄存器被更新；并且根据URS的选择，决定是否置位UIF。  
- 与分配器的缓冲器载入预装载值（TIMx_PSC里的值）  
- 自动重载影子寄存器的值更新为预装载值（TIMx_ARR的值）  
下面是当TIMx_ARR=0x36时，不同时钟频率下计数器行为的图示。  
![](https://i.imgur.com/XnW8V0t.png)  
![](https://i.imgur.com/9sLD0Yb.png)  
![](https://i.imgur.com/HwFgpXQ.png)  
![](https://i.imgur.com/EGyPayQ.png)  
![](https://i.imgur.com/i1OMA7t.png)  
![](https://i.imgur.com/9MF2MSk.png)  
####向下计数  
向下计数模式，计数器从TIMx_ARR中的自动重载值计数到0,然后重新从自动重载值计数，并产生计数器下溢事件。  
计数器下溢或设置TIMx_EGR中的UG=1（软件设置或从模式控制器），都会产生更新事件。  
设置TIMx_CR1中的UDIS=1，将禁止更新事件UEV产生。这可以避免当向预装载寄存器中写入新值时，发生更新事件，造成影子寄存器更新值错误。在UDIS=0之前，不会产生更新事件。然而，计数器和预分频器的计数器（分频因子不会改变）都会如常工作。  
另外，如果TIMx_CR1中的URS=1，设置UG=1将产生更新事件，但不会置位UIF（也就不会产生中断或DMA请求）。这是为了避免在清除捕获事件的计数器时产生更新和捕获中断。  
当发生更新事件时，所有寄存器被更新，而TIMx_SR中的UIF根据URS的选择决定是否置位。  
- 预分频器的缓冲器载入预装载值（TIMx_PSC的值）  
- 自动重载影子寄存器载入预装载值（TIMx_ARR的值）。注意，自动重载寄存器在计数器重载前被更新，所以下个周期才是期望的结果。  
下面是当TIMx_ARR=0x36时，不同时钟频率下计数器的行为的图示。  
![](https://i.imgur.com/Yvf5FQw.png)  
![](https://i.imgur.com/7McaDuS.png)  
![](https://i.imgur.com/SNSQ6WE.png)  
![](https://i.imgur.com/3Wn5jAm.png)  
![](https://i.imgur.com/QTfsXgg.png)  
####中心对齐模式（向上然后向下计数）  
中心对齐模式，计数器先从0计数到自动重载值TIMx_ARR-1，产生计数器溢出事件；然后从自动重载值TIMx_ARR计数到1，产生计数器下溢事件。然后计数器重新从0向上计数。  
当TIMx_CR1中的CMS≠00时，激活中心对器模式。输出通道的输出比较中断标志置位，当：计数器向下计数溢出（中心对齐模式1，CMS=01），计数器向上计数溢出（中心对齐模式2，CMS=10），计数器向上和向下计数溢出（中心对齐模式3，CMS=11）。  
在中心对齐模式，计数方向位TIMx_CR1中的DIR不能被写。它由硬件更新，指示计数器当前计数方向。  
每次计数器溢出和下溢或设置TIMx_EGR中的UG=1（软件设置或从模式控制器），都会产生更新事件。由UG=1产生的更新事件，将使计数器和预分频器的计数器重新从0计数。  
设置TIMx_CR1中的UDIS=1，将禁止产生更新事件UEV。这可以避免在向预装载寄存器写入新值时，发生更新事件，导致更新到影子寄存器中的值错误。在UDIS=0之前不会再产生更新事件。然而，计数器还会继续正常工作。  
另外，如果TIMx_CR1中的URS=1，则设置UG=1将产生更新事件但不会置位UIF（也不会产生中断或DMA请求）。这是为了避免在清除捕获事件的计数器时产生更新和捕获中断。  
当发生更新事件时，所有寄存器被更新，而TIMx_SR中的UIF根据URS选择是否置位。  
- 预分频器的缓冲器载入预装载值（TIMx_PSC的值）  
- 自动重载影子寄存器更新为预装载值（TIMx_ARR的值）。注意如果计数器向上溢出产生更新事件，自动重载寄存器在计数器重载前更新，所以下个周期才是期望的结果（计数器导入新值）。  
下面是不同时钟频率下计数器行为的图示。  
![](https://i.imgur.com/SUu8QNn.png)  
![](https://i.imgur.com/j2IPpPG.png)  
![](https://i.imgur.com/gXkiTQG.png)  
![](https://i.imgur.com/d1KQC0p.png)  
![](https://i.imgur.com/hUZsUPk.png)  
![](https://i.imgur.com/bc8z0Gw.png)  
###时钟源  
计数器时钟有如下选择：  
- 内部时钟CK_INT  
- 外部时钟模式1:外部输入脚TIx  
- 外部时钟模式2：外部触发输入脚ETR  
- 内部触发输入ITRx：使用一个定时器作为另一个的预分频器，例如，可以配置TIM1作为TIM2的预分频器。  
####内部时钟CK_INT  
如果从模式控制器被禁止（TIMx_SMCR中的SMS=000），那么TIMx_CR1中的CEN和DIR位，以及TIMx_EGR中的UG位就是实际的控制位，只能被软件修改（除了UG位自动清零）。当CEN=1，则CK_INT内部时钟提供给预分频器。  
图108显示了控制电路和向上计数器在一般模式，不分频的情况下的行为。  
![](https://i.imgur.com/cBbJpjf.png)  
####外部时钟模式1  
当TIMx_SMCR中的SMS=111时，使用外部时钟模式1。计数器在选定的输入脚TIx的每次上升沿或下降沿计数。  
![](https://i.imgur.com/GQJjyjd.png)  
举例，配置计数器在输入TI2的上升沿向上计数，步骤如下：  
1. 配置通道2检测TI2输入的上升沿，通过设置TIMx_CCMR1中的CC2S=01  
2. 配置输入滤波器带宽，通过设置TIMx_CCMR1中的IC2F[3:0]（如果不需要滤波器，保持IC2F=0000）  
3. 选择上升沿，通过设置TIMx_CCER中的CC2P=0和CC2NP=0  
4. 配置定时器使用外部时钟模式1，通过设置TIMx_SMCR中的SMS=111  
5. 选择TI2作为时钟源，通过设置TIMx_SMCR中的TS=110  
6. 使能计数器，通过设置TIMx_CR1中的CEN=1  
注意，此模式不需要使用捕获预分频器，所以不必设置它。  
######Upcounter on TI2 rising edge code example  

	/* (1) Enable the peripheral clock of Timer 3 */
	/* (2) Enable the peripheral clock of GPIOA */
	/* (3) Select Alternate function mode (10) on GPIOA pin 7 */
	/* (4) Select TIM3_CH2 on PA7 by enabling AF1 for pin 7 in GPIOA AFRL register */
	RCC->APB1ENR |= RCC_APB1ENR_TIM3EN; /* (1) */
	RCC->AHBENR |= RCC_AHBENR_GPIOAEN; /* (2) */
	GPIOA->MODER = (GPIOA->MODER & ~(GPIO_MODER_MODER7)) | (GPIO_MODER_MODER7_1); /* (3) */
	GPIOA->AFR[0] |= 0x1 << (7*4); /* (4) */
	/* (1) Configure channel 2 to detect rising edges on the TI2 input by writing CC2S = ‘01’, 
           and configure the input filter duration by writing the IC2F[3:0] bits 
           in the TIMx_CCMR1 register (if no filter is needed, keep IC2F=0000).*/
	/* (2) Select rising edge polarity by writing CC2P=0 
           in the TIMx_CCER register (reset value).*/
	/* (3) Configure the timer in external clock mode 1 by writing SMS=111
	       Select TI2 as the trigger input source by writing TS=110
	       in the TIMx_SMCR register.*/
	/* (4) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_IC2F_0 | TIM_CCMR1_IC2F_1 | TIM_CCMR1_CC2S_0; /* (1) */
	TIMx->CCER &= (uint16_t)(~TIM_CCER_CC2P); /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS | TIM_SMCR_TS_2 | TIM_SMCR_TS_1; /* (3) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (4) */  
当TI2出现上升沿时，计数器计数一次，并且TIF置位。  
从TI2出现上升沿到产生计数器时钟之间的延时，取决于TI2输入上的重新同步电路。  
![](https://i.imgur.com/lGoB5KD.png)  
####外部时钟模式2  
设置TIMx_SMCR中的ECE=1，使能外部时钟模式2。  
在外部触发输入ETR引脚上的每次上升沿或下降沿，计数器计数一次。  
![](https://i.imgur.com/WjtFuHp.png)  
举例，计数器每2个ETR上升沿向上计数一次，配置步骤如下：  
1. 因为本例不需要滤波器，所以保持TIMx_SMCR中的ETF[3:0]=0000  
2. 配置预分频器，通过设置TIMx_SMCR中的ETPS[1:0]=01，对ETR信号2分频  
3. 选择检测ETR上升沿，通过设置TIMx_SMCR中的ETP=0  
4. 使能外部时钟模式2，通过设置TIMx_SMCR中的ECE=1  
5. 使能计数器，通过设置TIMx_CR1中的CEN=1  
计数器每2个ETR上升沿计数一次。  
从ETR出现上升沿到产生计数器时钟之间的延时，取决于ETRP信号端的重新同步电路。  
![](https://i.imgur.com/23FYbkP.png)  
###捕获/比较通道  
每个捕获/比较通道都是基于一个捕获/比较寄存器（包含一个影子寄存器），一个捕获输入级（带有数字滤波器，多路复用和预分频器），以及一个比较输出级（带有比较器和输出控制）。  
下面的图给出了一个捕获/比较通道的概览。  
输入级对相应的TIx输入进行采样然后产生滤波后的信号TIxF。然后，通过一个带极性选择的边沿检测器后产生信号TIxFPx，它可以作为从模式控制器的触发输入，或者作为捕获控制。TIxFPx经过预分频器产生信号ICxPS，传输到捕获寄存器。  
![](https://i.imgur.com/905L1JK.png)  
输出级产生一个中间波形用作参考：OCxRef(高电平有效)。输出级末端控制输出极性。  
![](https://i.imgur.com/8oWkxiU.png)  
![](https://i.imgur.com/Qm0oWks.png)  
捕获/比较模块由一个预装载寄存器和它的影子寄存器组成。读写操作仅对预装载寄存器进行。  
在捕获模式，实际是影子寄存器捕获计数器值，然后再拷贝到预装载寄存器。  
在比较模式，预装载寄存器的值拷贝到影子寄存器，然后影子寄存器与计数器比较。  
###输入捕获模式  
在输入捕获模式，在ICx信号上检测到相应的边沿后，捕获/比较寄存器TIMx_CCRx锁存住计数器的值。当发生捕获，TIMx_SR中的CCxIF标志置位，并且产生中断或DMA请求如果它们使能的话。如果当CCxIF=1时，再次发生捕获，那么捕获溢出标志TIMx_SR中的CCxOF置位。CCxIF可以通过软件向其写0或读取TIMx_CCRx寄存器中储存的捕获值，清零。CCxOF需要软件向其写0，清除。  
下面的例子介绍当TI1输入上升沿时TIMx_CCR1如何捕获计数器的值。要实现此目的，需要遵循下面的步骤：  
- 选择有效的输入：TIMx_CCR1必须链接到TI1输入上，因此要设置TIMx_CCMR1中的CC1S=01。一旦CC1S≠00，该通道就配置成输入了，并且TIMx_CCR1寄存器也变为只读。  
- 根据输入信号，配置输入滤波器带宽（如果输入是TIx，则配置TIMx_CCMRx寄存器中的ICxF）。设想，当跳变是，输入信号在5个内部时钟周期内会抖动；那么我们必须设置滤波器带宽长于5时钟周期。我们可以配置TIMx_CCMR1寄存器中的IC1F=0011，以f<sub>CK_INT</sub>为采样频率，连续采样8次，如果8次采样结果一致且是新的电平，则可以确认TI1上信号发生跳变。  
- 选择TI1通道上的有效的跳变边沿：设置TIMx_CCER寄存器中的CC1P=0且CC1NP=0，选择上升沿。  
- 配置输入预分频器。在本例中，我们希望在每一个有效边沿都进行捕获，因此不需要分频，保持TIMx_CCMR1寄存器中的IC1PS=00。  
- 使能捕获计数器值到捕获寄存器：设置TIMx_CCER寄存器中的CC1E=1。  
- 如果需要，设置TIMx_DIER寄存器中的CC1IE=1，使能中断；和/或TIMx_DIER中的CC1DE=1，使能DMA请求。  
######nput capture configuration code example  

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
当发生输入捕获时：  
- TIMx_CCR1获取计数器值。  
- CC1IF=1。如果连续发生至少2次捕获且CC1IF=1，则CC1OF=1。  
- 根据CC1IE的设置，决定是否产生中断。  
- 根据CC1DE的设置，决定是否发出DMA请求。  
######Input capture data management code example  

	/* This code must be inserted in the Timer interrupt subroutine. */
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
	/* Note: This code manages only a single counter overflow. 
    To manage many counter overflows the update interrupt must be enabled
    (UIE = 1) and properly managed. */  
为了除了捕获溢出，建议在读取捕获溢出标志之前读取捕获数据。这是为了避免，在读取捕获溢出标志之后而在读取捕获数据之前的这段时间内，又发生捕获，而造成前次捕获数据丢失。  
注：输入捕获中断和/或DMA请求，可以通过软件设置TIMx_EGR中的CCxG=1，产生。  
###PWM输入模式  
该模式是一种特殊的输入捕获模式。和输入捕获模式的差别是：  
- 有2个ICx信号被映射到同一个TIx输入上。  
- 这2个ICx的有效边沿极性相反。  
- 2个TIxFP信号其中一个选作触发器输入TRGI，并且从模式控制器配置成复位模式。  
举例，测量TI1上的PWM信号的周期（使用TIMx_CCR1）和占空比（使用TIMx_CCR2），步骤如下（取决于CK_INT的频率和预分频器的值）：  
- 为TIMx_CCR1选择有效输入：设置TIMx_CCMR1中的CC1S=01，选择TI1。  
- 为TI1FP1选择有效极性（用于捕获数据到TIMx_CCR1和计数器清零）：设置TIMx_CCER中的CC1P=0以及CC1NP=0，选择上升沿有效。  
- 为TIMx_CCR2选择有效输入：设置TIMx_CCMR1中的CC2S=10，选择TI1。  
- 为TI1FP2选择有效极性（用于捕获数据到TIMx_CCR2）：设置TIMx_CCER中的CC2P=1以及CC2NP=0，选择下降沿有效。  
- 选择有效的触发器输入TRGI：设置TIMx_SMCR中的TS=101，选择TI1FP1。  
- 配置从模式控制器为复位模式：设置TIMx_SMCR中的SMS=100。  
- 使能捕获：设置TIMx_CCER中的CC1E=1以及CC2E=1。  
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
![](https://i.imgur.com/MTWgyS5.png)  
###强制输出模式  
在输出模式（TIMx_CCMRx中的CCxS=00），每一个输出比较信号（OCxREF和OCx）可以直接由软件强置为有效或无效电平，而不依赖输出比较寄存器和计数器的比较结果。  
要强制比较输出信号（OCxREF和OCx）为有效电平，只需设置TIMx_CCMRx中的OCxM=101，这样OCxREF就被强制为高电平（OCxREF的有效电平总是高电平），而OCx的电平和TIMx_CCER中的CCxP的设置相反。  
例如：CCxP=0（OCx高电平有效）=> OCx被强制为高电平。  
OCxREF可以强制为低电平（无效电平），通过设置TIMx_CCMRx中的OCxM=100。  
然而，在该模式下，TIMx_CCRx和计数器的比较依然会进行，相应的标志位也会置位。因此，同样可以产生中断和DMA请求。  
###比较输出模式  
此功能可以用于控制输出波形，或指出设定的一段时间已经到时了。  
当捕获/比较寄存器和计数器2者值相等时，比较输出功能：  
- 相应的输出引脚输出配置的电平，该电平由设置的比较输出模式（设置TIMx_CCMRx中的OCxM来选择）和设置的输出极性（设置TIMx_CCER中的CCxP来选择）决定。如果OCxM=000，则输出引脚保持当前输出，比较结果不会影响它；OCxM=001，则输出引脚输出有效电平；OCxM=010，则输出引脚输出无效电平；OCxM=011，则输出引脚输出发生反转。  
- TIMx_SR中的中断标志位CCxIF置位。  
- 如果TIMx_DIER中的CCxIE=1，则产生中断。  
- 如果TIMx_DIER中的CCxDE=1，TIMx_CR2中的CCDS=0，则发出DMA请求。  
TIMx_CCRx由TIMx_CCMRx中的OCxPE设置是否使用预装载寄存器。  
在比较输出模式，更新事件UEV不会影响OCxREF和OCx输出。  
计时分辨率是计数器的一个计数。比较输出模式也可以用于输出一个单独脉冲（单脉冲模式）。  
比较输出模式配置步骤：  
- 选择计数器时钟（内部，外部，预分频器）。  
- 向TIMx_ARR和TIMx_CCRx写入期望的数据。  
- 如果需要中断和/或DMA请求，置位TIMx_DIER中的CCxIE和/或CCxDE。  
- 选择输出模式。例如，设置OCxM=011,OCxPE=0,CCxP=0和CCxE=1，即是当CNT和CCRx相等时OCx输出反转，并且CCRx不使用预装载寄存器，OCx输出使能并且高电平有效。  
- 使能计数器，设置TIMx_CR1中的CEN=1。  
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
TIMx_CCRx寄存器可以任何时候被改变以控制输出波形，前提是没有使用预装载寄存器（OCxPE=0），否则，发生更新事件UEV时TIMx_CCRx的影子寄存器才被更新为新的值。  
下图是不使用预装载寄存器时，改变TIMx_CCRx的值，发生的情况。  
![](https://i.imgur.com/BOdErOj.png)  
###PWM输出模式  
PWM模式允许你通过配置TIMx_ARR确定频率，TIMx_CCRx确定占空比，产生一个PWM信号。  
每个通道可以独立设置为PWM模式（每个OCx输出一路PWM信号）。当TIMx_CCMRx中的OCxM=110，选择PWM模式1；OCxM=111，选择PWM模式2。设置TIMx_CCMRx中的OCxPE=1，使能TIMx_CCRx的预装载寄存器；并且设置TIMx_CR1中的ARPE=1，使能自动重载预装载寄存器（在向上计数或中心对齐计数模式下）。  
因为预装载寄存器的值只有在发生更新事件时才被传送到影子寄存器中，所以在开始计数前，需要设置TIMx_EGR中的UG=1产生更新事件，初始化所有寄存器。  
OCx的极性是软件设置TIMx_CCER中的CCxP位来选择，可以设置为高电平有效或低电平有效。设置TIMx_CCER中的CCxE=1，使能OCx输出。  
在PWM模式1或模式2，TIMx_CNT和TIMx_CCRx始终在进行比较，确定是否TIMx_CNT≥TIMx_CCRx或TIMx_CNT≤TIMx_CCRx（取决于计数方向）。然而，为了顺应OCREF_CLR功能（OCxREF可以被外部事件通过ETR清除，直到下个PWM周期），OCxREF信号只在下列情况发生：  
- 当比较结果改变时，或  
- 当比较输出模式（TIMx_CCMRx中的OCxM位）从“冻结”（OCxM=000）切换到PWM模式（OCxM=110或111）。  
这样，在定时器运行时，可以通过软件强制PWM输出。  
设置TIMx_CR1中的CMS，可以选择边沿对齐PWM模式或中心对齐PWM模式。  
####PWM边沿对齐模式  
#####向上计数配置  
TIMx_CR1中的DIR=0，计数器向上计数。  
在下面的例子中，我们配置为PWM模式1。当TIMx_CNT＜TIMx_CCRx时，参考PWM信号OCxREF是有效电平（即高电平）；否则，为无效的低电平。如果TIMx_CCRx＞TIMx_ARR，那么TIMx_CNT总是小于TIMx_CCRx，所以OCxREF会一直为高电平。如果TIMx_CCRx=0，那么TIMx_CNT不会小于TIMx_CCRx，所以OCxREF会一直为低电平。  
图118显示了，当TIMx_ARR=8时，一段边沿对齐的PWM波形。  
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
![](https://i.imgur.com/LJNhoME.png)  
#####向下计数配置  
TIMx_CR1中的DIR=1，计数器向下计数。  
在PWM模式1下，当TIMx_CNT≤TIMx_CCRx时，OCxREF是有效的高电平；否则，是无效的低电平。如果TIMx_CCRx≥TIMx_ARR，那么TIMx_CNT永远不会大于TIMx_CCRx，所以OCxREF会一直为高电平。但是，当TIMx_CCRx=0时，因为TIMx_CNT可以为0，所以OCxREF不会一直是低电平。  
####PWM中心对齐模式  
TIMx_CR1中的CMS≠00时，计数器按中心对齐模式计数。不管哪种中心对齐模式，对于OCxREF/OCx信号的影响是一样的。  
CCxIF在向上计数或向下计数，或是两者时置位，取决于CMS配置为哪种中心对齐模式。  
TIMx_CR1中的DIR只可读，其由硬件根据计数器计数方向更新。  
图119显示了，一段中心对齐PWM波形，当：  
- TIMx_ARR=8  
- PWM模式1  
- TIMx_CR1中的CMS=01，选择中心对齐模式1，当计数器向下计数过程中发生比较匹配时，CCxIF置位  
![](https://i.imgur.com/qGtGmzL.png)  
中心对齐模式的使用提示：  
- 当开始进入中心对齐模式时，计数器按当前DIR的值指示的方向继续计数。而且，DIR和CMS不能同时被软件修改。  
- 当计数器运行在中心对齐模式中时，不建议修改计数器的值，因为这可能导致意外的结果。特别是：  
　- 如果写入计数器的值大于自动重载值，即TIMx_CNT＞TIMx_ARR，但是，DIR不会更新。如果，此时计数器是向上计数，它会计数向上计数。  
　- 如果将0或TIMx_ARR的值写入计数器，DIR会更新，但是不会产生更新事件UEV。  
- 使用中心对齐模式安全的方法是：在启动计数器之前，软件设置TIMx_EGR中的UG=1，产生一个更新事件，初始化所有寄存器，并且在计数过程中不修改计数器的值。  
###单脉冲输出模式  
单脉冲模式OPM是PWM输出模式的一个特例。它允许计数器响应一个激励而启动，并在可编程延迟后产生具有可编程长度的脉冲。  
可以通过从模式控制器控制计数器的启动。在比较输出模式或PWM输出模式下，产生输出波形。设置TIMx_CR1中的OPM=1，选择单脉冲输出模式。在单脉冲模式，计数器会在更新事件UEV发生时，自动停止。  
单脉冲只有在比较值和计数器初始值不同时，才能正确产生。在启动前（当定时器等待触发时），配置必须为：  
- 向上计数：CNT<CCRx<ARR（特殊地，0<CCRx）  
- 向下计数：CNT>CCRx  
![](https://i.imgur.com/HH1Wo6U.png)  
举例，当TI2输入脚检测到一个上升沿，延时t<sub>DELAY</sub>后，在OC1上输出一个宽度t<sub>PULSE</sub>的正脉冲。  
使用TI2FP2作为触发器输入：  
- 将TI2FP2映射到TI2，通过设置TIMx_CCMR1中的CC2S=01。  
- 检测TI2FP2上升沿，通过设置TIMx_CCER中的CC2P=0和CC2NP=0。  
- 配置TI2FP2作为从模式控制器的触发TRGI，通过设置TIMx_SMCR中的TS=110。  
- 使用TI2FP2启动计数器，通过设置TIMx_SMCR中的SMS=110（触发模式）。  
OPM波形由比较寄存器的值决定（要考虑时钟频率和计数器预分频器）。  
- t<sub>DELAY</sub>由TIMx_CCR1的值决定。  
- t<sub>PULSE</sub>由自动重载值和比较值的差决定（TIMx_ARR-TIMx_CCR1+1）。  
- 假定，当比较匹配时，想要波形从0跳到1，而当计数器达到自动重载值时，波形从1跳到0。要实现它，需要设置TIMx_CCMR1中的OC1M=111，配置为PWM模式2。可以根据需要配置TIMx_CCMR1中的OC1PE=1和TIMx_CR1中的ARPE使能预装载寄存器。如果使用预装载寄存器，在写入TIMx_CCR1比较值和TIMx_ARR自动重载值后，需要软件设置UG=1产生更新事件，将预装载寄存器中的值拷贝到影子寄存器中；然后等待TI2上的外部触发事件。在此例中，设置CC1P=0。  
在我们的举例中，需要设置TIMx_CR1中的DIR=0和CMS=00。  
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
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1M_0 | TIM_CCMR1_OC1PE
	#if PULSE_WITHOUT_DELAY > 0
	             | TIM_CCMR1_OC1FE
	#endif
	             ; /* (4) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (5) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (6) */
	TIMx->CR1 |= TIM_CR1_OPM | TIM_CR1_ARPE; /* (7) */  
你只想要一个脉冲，所以设置TIMx_CR1中的OPM=1，当计数器从自动重载值翻转到0时，发生更新事件，计数器停止。当OPM=0时，计数器会反复计数。  
####特殊情况：OCx快速使能  
在单脉冲模式，TIx输入上的边沿检测设置CEN启动计数器。然后，计数器和比较值的比较结果促使输出跳换。但是这些操作需要几个时钟周期来完成，这就限制了我们能得到的最小延时t<sub>DELAY</sub>。  
如果你想要以最小延时输出波形，可以设置TIMx_CCMRx中的OCxFE=1。然后，OCxREF（以及OCx）被强制响应激励，而不考虑比较结果；输出的波形和比较匹配时的波形一样。OCxFE只对配置成PWM模式1或2的通道起作用。  
###外部事件清除OCxREF信号  
- 外部触发预分频器保持关闭：TIMx_SMCR中的ETPS[1:0]=00。  
- 禁用外部时钟模式2：TIMx_SMCR中的ECE=0。  
- 外部触发极性ETP和外部触发滤波器ETF根据需要设置。  
######ETR configuration to clear OCxREF code example  

	/* This code is similar to the edge-aligned PWM configuration but it enables
	   the clearing on OC1 for ETRclearing (OC1CE = 1) in CCMR1 (5) and ETR is
	   configured in SMCR (7).*/
	/* (1) Set prescaler to 47, so APBCLK/48 i.e 1MHz */
	/* (2) Set ARR = 8, as timer clock is 1MHz the period is 9 us */
	/* (3) Set CCRx = 4, , the signal will be high during 4 us */
	/* (4) Select PWM mode 1 on OC1 (OC1M = 110),
	       enable preload register on OC1 (OC1PE = 1),
	       enable clearing on OC1 for ETR clearing (OC1CE = 1) */
	/* (5) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1) */
	/* (6) Enable output (MOE = 1) */
	/* (7) Select ETR as OCREF clear source (OCCS = 1),
	       select External Trigger Prescaler off (ETPS = 00, reset value),
	       disable external clock mode 2 (ECE = 0, reset value),
	       select active at high level (ETP = 0, reset value) */
	/* (8) Enable counter (CEN = 1),
	       select edge aligned mode (CMS = 00, reset value),
	       select direction as upcounter (DIR = 0, reset value) */
	/* (9) Force update generation (UG = 1) */
	TIMx->PSC = 47; /* (1) */
	TIMx->ARR = 8; /* (2) */
	TIMx->CCR1 = 4; /* (3) */
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 
                 | TIM_CCMR1_OC1PE | TIM_CCMR1_OC1CE; /* (4) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (5) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (6) */
	TIMx->SMCR |= TIM_SMCR_OCCS; /* (7) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (8) */
	TIMx->EGR |= TIM_EGR_UG; /* (9) */  
图121显示了，当ETRF输入从低变高时，对于OCCE=0/1，OCxREF的变化；其中，定时器TIMx配置为PWM模式1。  
![](https://i.imgur.com/HpgmCnx.png)  
###编码器接口模式  
如果计数器只在TI2边沿计数，则配置TIMx_SMCR中的SMS=001选择编码器模式1；如果计数器只在TI1边沿计数，则配置TIMx_SMCR中的SMS=010选择编码器模式2；如果计数器在TI1和TI2两者的边沿都计数，则配置TIMx_SMCR中的SMS=011选择编码器模式3。  
设置TIMx_CCER中的CC1P和CC2P位，选择TI1和TI2的极性。而CC1NP和CC2NP必须保持为0。如果需要，可以设置输入滤波器。  
TI1和TI2两个输入作为增量编码器的接口。参考表48。假设TIMx_CR1中的CEN=1计数器启动，则计数器在TI1FP1或TI2FP2的每一个有效边沿计数（TI1FP1和TI2FP2是TI1和TI2经过输入滤波器和极性选择后的信号，如果不滤波或不取反，则TI1FP1=TI1,TI2FP2=TI2）。对两个输入的跳变顺序进行评估，产生计数脉冲和方向信号。根据两个输入信号的跳变顺序，计数器向上或向下计数，同时TIMx_CR1中的DIR位由硬件自动置位或清除。无论使用哪种编码器模式，在任一输入（TI1或TI2）的每个跳变时都会重新计算设置DIR位。  
编码器接口模式的工作原理类似使用一个带方向选择的外部时钟。这意味着计数器只在0和TIMx_ARR的值之间连续计数（从0计数到TIMx_ARR还是从TIMx_ARR计数到0，取决于DIR位）。所以开启动计数器前，必须配置TIMx_ARR。同样地，捕获器，比较器，预分频器，触发输出仍正常工作。  
在编码器模式，计数器跟随增量编码器的旋转速度和方向被自动修改，并且计数器的内容始终指示着编码器的位置。计数方向则与相连的编码器旋转方向对应。下表列出了所有可能的组合，假定TI1和TI2不同时变换。  
![](https://i.imgur.com/yNQuKqe.png)  
一个外部增量编码器可以和MCU直接相连，而不需要外部的逻辑电路。但是，通常使用比较器将编码器的差分输出转换为数字信号，这将大大增加抗干扰的能力。编码器表示机械零点的输出，可以与外部中断引脚相连并触发计数器复位。  
图122给出了一个计数器操作的例子，显示了计数信号产生和方向控制。它还显示了当选择两个输入信号边沿都计数时，输入抖动是如何被消除的。当传感器的位置靠近一个转换点时，可能会发生抖动。在这个例子中，我们配置如下：  
- TIMx_CCMR1中的CC1S=01，TI1FP1映射到TI1  
- TIMx_CCMR1中的CC2S=01，TI2FP2映射到TI2  
- TIMx_CCER中的CC1P=0,CC1NP=0，TI1FP1不反相，TI1FP1=TI1  
- TIMx_CCER中的CC2P=0,CC2NP=0，TI2FP2不反相，TI2FP2=TI2  
- TIMx_SMCR中的SMS=011，编码器模式3，计数器在TI1和TI2输入边沿都计数  
- TIMx_CR1中的CEN=1，使能计数器  
######Encoder interface code example  

	/* (1) Configure TI1FP1 on TI1 (CC1S = 01),
	       configure TI1FP2 on TI2 (CC2S = 01) */
	/* (2) Configure TI1FP1 and TI1FP2 non inverted (CC1P = CC2P = 0, reset value) */
	/* (3) Configure both inputs are active on both rising and falling edges (SMS = 011) */
	/* (4) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0 | TIM_CCMR1_CC2S_0; /* (1)*/
	TIMx->CCER &= (uint16_t)(~(TIM_CCER_CC21 | TIM_CCER_CC2P); /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS_0 | TIM_SMCR_SMS_1; /* (3) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (4) */  
![](https://i.imgur.com/EAxHoGd.png)  
图123显示了当TIFP1极性反相（CC1P=1，其他配置相同）后，计数器的行为。  
![](https://i.imgur.com/KKnO0tl.png)  
定时器配置成编码器接口模式，可以提供编码器当前的位置。使用另一个定时器配置成捕获输入模式，测量2个编码器事件的间隔，可以获得速度、加速度、减速度等动态信息。编码器指示机械零点的输出可以用做此目的。根据2个事件的间隔，可以按一个固定的时间读取计数器。如果可能的话，你可以把计数器的值锁存入第三个通道的输入捕获寄存器（捕获信号必须是周期性的，而且能由另一个定时器产生）。如果可能的话，也可以通过实时时钟产生的DMA请求，来读取计数器的值。  
###定时器的输入异或功能  
TIMx_CR2中的TI1S=1，则定时器的三个输入引脚TIMx_CH1,TIMx_CH2,TIMx_CH3通过一个异或门，连接到TI1；即，三个输入引脚上的信号异或后作为通道1的输入信号；同样可以用作定时器的触发或输入捕获等所有输入功能。否则，TI1S=0，则TIMx_CH1作为TI1的输入。  
在TIM1一章中，有此功能用于霍尔传感器接口的例子。  
###定时器和外部触发同步  
TIMx定时器可以和外部触发以下列模式实现同步：复位模式，门控模式和触发模式。  
####从模式：复位模式  
当发生一个触发输入事件时，计数器和它的预分频器被重新初始化。另外，如果TIMx_CR1中的URS=0，则还会产生一个更新事件UEV。那么，所有的寄存器（TIMx_ARR,TIMx_CCRx）都会被更新。  
在下面的例子中，当TI1输入上出现上升沿时，向上计数器被清零：  
- 配置通道1检测TI1的上升沿。配置输入滤波器带宽（在这个例子中，不需要滤波器，所以保持IC1F=0000）。由于捕获预分频器不用于触发操作，所以不需要配置它。TIMx_CCMR1中的CC1S用于选择输入捕获源，本例中设置CC1S=01，选择TI1。在TIMx_CCER中写入CC1P=0,CC1NP=0，验证极性（只检测上升沿）。  
- 在TIMx_SMCR中写入SMS=100，定时器配置为复位模式。在TIMx_SMCR中写入TS=101，选择TI1作为输入源。  
- 在TIMx_CR1中写入CEN=1，启动计数器。  
######Reset mode code example  

	/* (1) Configure channel 1 to detect rising edges on the TI1 input
	       by writing CC1S = ‘01’,
	       and configure the input filter duration by writing the IC1F[3:0] bits
           in the TIMx_CCMR1 register (if no filter is needed, keep IC1F=0000).*/
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
计数器按内部时钟正常计数，直到TI1上出现上升沿。此时，计数器被清零并重新从0开始计数。同时，TIMx_SR中的触发标志位TIF置1。如果TIMx_DIER中的TIE=1或TDE=1，还将产生中断或DMA请求。  
下图显示了当TIMx_ARR=0x36时，在复位模式下计数器的行为。TI1的上升沿和实际计数器复位间的延迟是由TI1输入的重新同步电路引起的。  
![](https://i.imgur.com/XCyHCFw.png)  
####从模式：门控模式  
输入信号的电平使能计数器。  
在下面的例子中，只有当TI1输入是低电平时向上计数器才计数：  
- 配置通道1检测TI1输入低电平。配置输入滤波带宽（在本例中，不需要滤波，所以保持IC1F=0000）。捕获预分频器不用于触发操作，所以不需要配置它。TIMx_CCMR1中写入CC1S=01，选择TI1作为捕获输入源。TIMx_CCER中的CC1P=1,CC1NP=0，确定极性（只检测低电平）。  
- 在TIMx_SMCR中写入SMS=101，定时器配置为门控模式。TIMx_SMCR中写入TS=101，选择TI1作为输入源。  
- 设置TIMx_CR1中的CEN=1，使能计数器（如果CEN=0，在门控模式下，不管输入电平高低，计数器都不会计数）。  
######Gated mode code example  

	/* (1) Configure channel 1 to detect low level on the TI1 input
	       by writing CC1S = ‘01’,
	       and configure the input filter duration by writing the IC1F[3:0] bits
           in the TIMx_CCMR1 register (if no filter is needed, keep IC1F=0000). */
	/* (2) Select polarity by writing CC1P=1 in the TIMx_CCER register */
	/* (3) Configure the timer in gated mode by writing SMS=101
	       Select TI1 as the trigger input source by writing TS=101
	       in the TIMx_SMCR register. */
	/* (4) Set prescaler to 12000-1 in order to get an increment each 250us */
	/* (5) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0; /* (1)*/
	TIMx->CCER |= TIM_CCER_CC1P; /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_0 
                | TIM_SMCR_TS_2 | TIM_SMCR_TS_0; /* (3) */
	TIMx->PSC = 11999; /* (4) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (5) */  
当TI1为低电平时，计数器根据内部时钟开始计数；当TI1变为高电平，计数器就停止。当计数器开始或停止计数时，TIMx_SR中的TIF标志都会被置位。  
从TI1出现上升沿到实际计数器停止计数之间的延时是由TI1输入上的重新同步电路造成的。  
![](https://i.imgur.com/MwhJg3V.png)  
####从模式：触发模式  
所选输入上发生的某一事件启动计数器。  
在下面的例子中，当TI2输入上出现上升沿时，向上计数器开始计数：  
- 配置通道2检测TI2上升沿。配置输入滤波器带宽（在本例中，不需要滤波器，所以保持IC2F=0000）。捕获预分频器不用于触发操作，所以不需要配置它。设置TIMx_CCMR1中额CC2S=01，选择TI2作为输入捕获源。向TIMx_CCER中写入CC2P=0,CC2NP=0，以确定极性（只检测上升沿）。  
- 向TIMx_SMCR写入SMS=110，配置定时器为触发模式。设置TIMx_SMCR中的TS=110，选择TI2作为输入源。  
######Trigger mode code example  

	/* (1) Configure channel 2 to detect rising edge on the TI2 input
	       by writing CC2S = ‘01’,
	       and configure the input filter duration by writing the IC1F[3:0]bits
           in the TIMx_CCMR1 register (if no filter is needed, keep IC1F=0000). */
	/* (2) Select polarity by writing CC2P=0 (reset value) in the TIMx_CCER register */
	/* (3) Configure the timer in trigger mode by writing SMS=110
	       Select TI2 as the trigger input source by writing TS=110
	       in the TIMx_SMCR register. */
	/* (4) Set prescaler to 12000-1 in order to get an increment each 250us */
	TIMx->CCMR1 |= TIM_CCMR1_CC2S_0; /* (1)*/
	TIMx->CCER &= ~TIM_CCER_CC2P; /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1
	            | TIM_SMCR_TS_2 | TIM_SMCR_TS_1; /* (3) */
	TIM1->PSC = 11999; /* (4) */  
当TI2上出现上升沿，计数器根据内部时钟开始开始计数，并且TIF被置1。  
TI2上出现上升沿到实际计数器开始计数之间的延迟是TI2输入上的重新同步电路造成的。  
![](https://i.imgur.com/Gt11o80.png)  
####从模式：外部时钟模式2+触发模式  
外部时钟模式2可以和另一种从模式（除了外部时钟模式1和编码器模式）结合使用。在这种情况下，ETR信号用作外部时钟输入，在复位模式、门控模式或触发模式下，可以选择另一个输入源作为触发输入TRGI。不建议通过设置TIMx_SMCR中的TS来选择ETR作为TRGI。  
在下面的例子中，当TI1上出现上升沿时，向上计数器在ETR的每个上升沿处计数：  
1. 通过如下设置TIMx_SMCR，配置外部触发电路：  
　- ETF=0000：不滤波  
　- ETPS=00：不分频  
　- ETP=0：检测ETR上升沿，并且ECE=1，使能外部时钟模式2  
2. 如下配置通道1，检测TI1上升沿：  
　- IC1F=0000：不滤波  
　- 捕获预分频器不用于触发操作，不需要配置  
　- TIMx_CCMR1中的CC1S=01，选择TI1作为输入源  
　- TIMx_CCER中的CC1P=0,CC1NP=0，确定极性（只检测上升沿）  
3. TIMx_SMCR中的SMS=110，配置定时器为触发模式。TIMx_SMCR中的TS=101，选择TI1作为输入源。  
######External clock mode 2 + trigger mode code example  

	/* (1) Configure no input filter (ETF=0000, reset value)
	       configure prescaler disabled (ETPS = 0, reset value)
	       select detection on rising edge on ETR (ETP = 0, reset value)
	       enable external clock mode 2 (ECE = 1) */
	/* (2) Configure no input filter (IC1F=0000, reset value)
	       select input capture source on TI1 (CC1S = 01) */
	/* (3) Select polarity by writing CC1P=0 (reset value) in the TIMx_CCER register */
	/* (4) Configure the timer in trigger mode by writing SMS=110
	       Select TI1 as the trigger input source by writing TS=101
	       in the TIMx_SMCR register. */
	TIMx->SMCR |= TIM_SMCR_ECE; /* (1) */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0; /* (2)*/
	TIMx->CCER &= ~TIM_CCER_CC1P; /* (3) */
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1
	            | TIM_SMCR_TS_2 | TIM_SMCR_TS_0; /* (4) */
	/* Use TI2FP2 as trigger 1 */
	/* (1) Map TI2FP2 on TI2 by writing CC2S=01 in the TIMx_CCMR1 register */
	/* (2) TI2FP2 must detect a rising edge, write CC2P=0 and CC2NP=0
	       in the TIMx_CCER register (keep the reset value) */
	/* (3) Configure TI2FP2 as trigger for the slave mode controller (TRGI)
	       by writing TS=110 in the TIMx_SMCR register,
	       TI2FP2 is used to start the counter by writing SMS to ‘110'
	       in the TIMx_SMCR register (trigger mode) */
	TIMx->CCMR1 |= TIM_CCMR1_CC2S_0; /* (1) */
	//TIMx->CCER &= ~(TIM_CCER_CC2P | TIM_CCER_CC2NP); /* (2) */
	TIMx->SMCR |= TIM_SMCR_TS_2 | TIM_SMCR_TS_1
	            | TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1; /* (3) */  
当TI1上升沿出现，计数器被使能，并且TIF被置1。计数器将在ETR的上升沿计数。  
ETR出现上升沿到实际计数器计数之间的延时是ETRP输入上的重新同步电路造成的。  
![](https://i.imgur.com/lif4x5f.png)  
###定时器同步  
TIMx定时器在内部连接到一起，以实现定时器同步或链接。当一个定时器配置为主模式时，可以对另一个配置为从模式的定时器的计数器进行复位、启动、停止或提供时钟。  
![](https://i.imgur.com/OD8FEva.png)  
####使用一个定时器作为另一个定时器的预分频器  
例如，你可以将TIM1配置为TIM3的预分频器。参考图128。步骤如下：  
- 配置TIM1为主模式，它可以在每次更新事件UEV发生时输出周期性的触发信号。如果向TIM1_CR2中写入MMS=010，则当每次发生更新事件时，TRGO1上输出上升沿。  
- 将TIM1的TRGO1输出连接到TIM3，TIM3必须配置为使用ITR0作为内部触发的从模式。通过向TIM3_SMCR中写入TS=0000，选择ITR0。  
- 然后将TIM3的从模式控制器配置为外部时钟模式1（向TIMx_SMCR中写入SMS=111）。这样TIM3的时钟将由TIM1周期性触发信号的上升沿（即TIM1计数器溢出）提供。  
- 最后向各自的TIMx_CR1中写入CEN=1，使能两个定时器。必须确保使能TIM3之前TIM1已经使能。  
######Timer prescaling another timer code example  

	/* TIMy is slave of TIMx */
	/* (1) Select Update Event as Trigger output (TRG0) 
           by writing MMS = 010 in TIMx_CR2. */
	/* (2) Configure TIMy in slave mode using ITR1 as internal trigger
	       by writing TS = 000 in TIMy_SMCR (reset value)
	       Configure TIMy in external clock mode 1, by writing SMS=111 in the
	       TIMy_SMCR register. */
	/* (3) Set TIMx prescaler to 47999 in order to get an increment each 1ms */
	/* (4) Set TIMx Autoreload to 999 in order to get an overflow (so an UEV) each second */
	/* (5) Set TIMy Autoreload to 24*3600-1 in order to get an overflow each 24-hour */
	/* (6) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	/* (7) Enable the counter by writing CEN=1 in the TIMy_CR1 register. */
	TIMx->CR2 |= TIM_CR2_MMS_1; /* (1) */
	TIMy->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1 | TIM_SMCR_SMS_0; /* (2) */
	TIMx->PSC = 47999; /* (3) */
	TIMx->ARR = 999; /* (4) */
	TIMy->ARR = (24 * 3600) - 1; /* (5) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (6) */
	TIMy->CR1 |= TIM_CR1_CEN; /* (7) */  
注：如果TIM1的OCxREF用作触发输出（MMS=1xx），它的上升沿将用作TIM3计数器的时钟。  
####用一个定时器使能另一个定时器  
在这个例子中，我们使用TIM1的比较输出1控制TIM3的使能。参考图128的连接。当TIM1的OC1REF为高电平时，TIM3按照分频后的内部时钟开始计数。2个定时器的计数器时钟都由内部时钟CK_INT/3提供（f<sub>CK_CNT</sub>=f<sub>CK_INT</sub>/3）。  
- 配置TIM1为主模式，比较输出1参考信号OC1REF作为触发输出TRGO（TIM1_CR2中的MMS=100）。  
- 配置TIM1的OC1REF波形（配置TIM1_CCMR1寄存器）。  
- 配置TIM3接收来自TIM1的内部触发（TIM3_SMCR中的TS=000）。  
- 配置TIM3工作在门控模式（TIM3_SMCR中的SMS=101）。  
- 使能TIM3，向TIM3_CR1中写入CEN=1。  
- 启动TIM1，向TIM1_CR1中写入CEN=1。  
######Timer enabling another timer code example  

	/* TIMy is slave of TIMx */
	/* (1) Configure Timer x master mode to send its Output Compare 1 Reference
	       (OC1REF) signal as trigger output (MMS=100 in the TIM1_CR2 register). */
	/* (2) Configure the Timer x OC1REF waveform (TIM1_CCMR1 register)
	       Channel 1 is in PWM mode 1 when the counter is less than the
	       capture/compare register (write OC1M = 110) */
	/* (3) Configure TIMy in slave mode using ITR1 as internal trigger
	       by writing TS = 000 in TIMy_SMCR (reset value)
	       Configure TIMy in gated mode, by writing SMS=101 in the
	       TIMy_SMCR register. */
	/* (4) Set TIMx prescaler to 2 */
	/* (5) Set TIMy prescaler to 2 */
	/* (6) Set TIMx Autoreload to 999 in order to get an overflow (so an UEV) each 100ms */
	/* (7) Set capture compare register to a value between 0 and 999 */
	TIMx->CR2 |= TIM_CR2_MMS_2; /* (1) */
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1; /* (2) */
	TIMy->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_0; /* (3) */
	TIMx->PSC = 2; /* (4) */
	TIMy->PSC = 2; /* (5) */
	TIMx->ARR = 999; /* (6) */
	TIMx-> CCR1 = 700; /* (7) */
	/* Configure the slave timer to generate toggling on each count */
	/* (1) Configure the TIMy in PWM mode 1 (write OC1M = 110) */
	/* (2) Set TIMy Autoreload to 1 */
	/* (3) Set capture compare register to 1 */
	TIMy->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1; /* (1) */
	TIMy->ARR = 1; /* (2) */
	TIMy-> CCR1 = 1; /* (3) */
	/* Enable the output of TIMx OC1 */
	/* (1) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1) */
	/* (2) Enable output (MOE = 1) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (1) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (2) */
	/* Enable the output of TIMy OC1 */
	/* (1) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1) */
	/* (2) Enable output (MOE = 1) */
	TIMy->CCER |= TIM_CCER_CC1E; /* (1) */
	TIMy->BDTR |= TIM_BDTR_MOE; /* (2) */
	/* (1) Enable the slave counter first by writing CEN=1 in the TIMy_CR1 register. */
	/* (2) Enable the master counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMy->CR1 |= TIM_CR1_CEN; /* (1) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (2) */  
注：TIM3的计数器时钟和TIM1的计数器时钟是不同步的，此模式只影响TIM3的计数器使能信号。  
![](https://i.imgur.com/8yMJvgO.png)  
在图129的例子中，TIM3的计数器和预分频器在开始计数器前未被初始化，所以计数器从当前值继续计数。在启动TIM1之前，可以复位两个定时器使它们从给定值开始计数；你可以向计数器中写入任何值。向TIMx_EGR中写入UG=1，可以复位定时器。  
在下面的例子中，我们同步TIM1和TIM3。TIM1是主模式，从0开始计数；TIM3是从模式，从0xE7开始计数。2个定时器的预分频器分频率是一样的。当向TIM1_CR1中写入CEN=0禁止TIM1时，TIM3也随即停止。  
- 配置TIM1为主模式，它的计数器使能信号CNT_EN作为触发输出TRGO（设置TIMx_CR2中的MMS=001）。  
- 配置TIM1的OC1REF波形（TIM1_CCMR1）。  
- 配置TIM3接收来自TIM1的内部触发输入（TIM3_SMCR中的TS=000）。  
- 配置TIM3为门控模式（TIM3_SMCR中的SMS=101）。  
- 复位TIM1，向TIM1_EGR中写入UG=1。  
- 复位TIM3，向TIM3_EGR中写入UG=1。  
- 初始化TIM3的计数器为0xE7，向TIM3_CNT写入0xE7。  
- 使能TIM3，向TIM3_CR1中写入CEN=1。  
- 启动TIM1，向TIM1_CR1中写入CEN=1。  
- 停止TIM1，向TIM1_CR1中写入CEN=0。  
######Master and slave synchronization code example  

	/* (1) Configure Timer x master mode to send its enable signal
	       as trigger output (MMS=001 in the TIM1_CR2 register). */
	/* (2) Configure the Timer x Channel 1 waveform (TIM1_CCMR1 register)
	       is in PWM mode 1 (write OC1M = 110) */
	/* (3) Configure TIMy in slave mode using ITR0 as internal trigger
	       by writing TS = 000 in TIMy_SMCR (reset value)
	       Configure TIMy in gated mode, by writing SMS=101 in the
	       TIMy_SMCR register. */
	/* (4) Set TIMx prescaler to 2 */
	/* (5) Set TIMy prescaler to 2 */
	/* (6) Set TIMx Autoreload to 99 in order to get an overflow (so an UEV) each 10ms */
	/* (7) Set capture compare register to a value between 0 and 99 */
	TIMx->CR2 |= TIM_CR2_MMS_0; /* (1) */
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1; /* (2) */
	TIMy->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_0; /* (3) */
	TIMx->PSC = 2; /* (4) */
	TIMy->PSC = 2; /* (5) */
	TIMx->ARR = 99; /* (6) */
	TIMx-> CCR1 = 25; /* (7) */
	/* Configure the slave timer Channel 1 as PWM as Timer to show synchronicity */
	/* (1) Configure the TIMy in PWM mode 1 (write OC1M = 110) */
	/* (2) Set TIMy Autoreload to 99 */
	/* (3) Set capture compare register to 25 */
	TIMy->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1; /* (1) */
	TIMy->ARR = 99; /* (2) */
	TIMy-> CCR1 = 25; /* (3) */
	/* Enable the output of TIMx OC1 */
	/* (1) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1)*/
	/* (2) Enable output (MOE = 1) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (1) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (2) */
	/* Enable the output of TIMy OC1 */
	/* (1) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1) */
	/* (2) Enable output (MOE = 1) */
	TIMy->CCER |= TIM_CCER_CC1E; /* (1) */
	TIMy->BDTR |= TIM_BDTR_MOE; /* (2) */
	/* (1) Reset Timer x by writing ‘1 in UG bit (TIMx_EGR register) */
	/* (2) Reset Timer y by writing ‘1 in UG bit (TIMy_EGR register) */
	TIMx->EGR |= TIM_EGR_UG; /* (1) */
	TIMy->EGR |= TIM_EGR_UG; /* (2) */
	/* (1) Enable the slave counter first by writing CEN=1 in the TIMy_CR1 register.
	       TIMy will start synchronously with the master timer */
	/* (2) Start the master counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMy->CR1 |= TIM_CR1_CEN; /* (1) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (2) */  
![](https://i.imgur.com/HHqWQqm.png)  
####使用一个定时器启动另一个定时器  
在这个例子中，我们用TIM1的更新事件来使能TIM3。参考图128的连接。当TIM1产生一个更新事件，TIM3就按照分频后的内部时钟从当前值（可以非0）开始计数。当TIM3收到触发信号，它的CEN位被自动置1，计数器开始计数直到软件将TIM3_CR1中的CEN清0。2个计数器的时钟都由CK_INT/3得到（f<sub>CK_CNT</sub>=f<sub>CK_INT</sub>/3）。  
- 配置TIM1为主模式，将更新事件UEV作为触发输出TRGO（TIM1_CR2中的MMS=010）。  
- 配置TIM1周期（TIM1_ARR寄存器）。  
- 配置TIM3接收来自TIM1的内部触发输入（TIM3_SMCR中的TS=000）。  
- 配置TIM3为触发模式（TIM3_SMCR中的SMS=110）。  
- 启动TIM1（TIM1_CR1中的CEN=1）。  
![](https://i.imgur.com/AcQDoqY.png)  
在上面的例子，在启动计数前可以初始化两个计数器。  
图132除了触发输出使用TIM1计数使能信号CNT_EN外，其他配置和图131相同。  
![](https://i.imgur.com/WZ3ccHq.png)  
####使用一个外部触发同步启动2个定时器  
在这个例子中，我们设置当TI1上出现上升沿时，TIM1使能；并且TIM3随着TIM1使能而使能。连接参考图128。为了确保计数器对齐，TIM1必须配置为主/从模式（从对应TI1，主对应TIM3）：  
- 配置TIM1为主模式，使用它的使能信号作为触发输出TRGO（TIM1_CR1中的MMS=001）。  
- 配置TIM1为从模式，接收来自TI1的触发输入（TIM1_SMCR中的TS=100）。  
- 配置TIM1为触发模式（TIM1_SMCR中的SMS=110）。  
- 配置TIM1为主/从模式（TIM1_SMCR中的MSM=1）。  
- 配置TIM3为从模式，接收来自TIM1的触发输入（TIM3_SMCR中的TS=000）。  
- 配置TIM3为触发模式（TIM3_SMCR中的SMS=110）。  
######Two timers synchronized by an external trigger code example  

	/* (1) Configure TIMx master mode to send its enable signal
	       as trigger output (MMS=001 in the TIM1_CR2 register). */
	/* (2) Configure TIMx in slave mode to get the input trigger from TI1 by writing TS = 100
	       Configure TIMx in trigger mode, by writing SMS=110
	       Configure TIMx in Master/Slave mode by writing MSM = 1
           in the TIMx_SMCR register. */
	/* (3) Configure TIMy in slave mode to get the input trigger from Timer1 by writing TS = 000
	       Configure TIMy in trigger mode, by writing SMS=110
           in the TIMy_SMCR register. */
	/* (4) Reset Timer x counter by writing ‘1 in UG bit (TIMx_EGR register) */
	/* (5) Reset Timer y counter by writing ‘1 in UG bit (TIMy_EGR register) */
	TIMx->CR2 |= TIM_CR2_MMS_0; /* (1)*/
	TIMx->SMCR |= TIM_SMCR_TS_2 | TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1 | TIM_SMCR_MSM; /* (2) */
	TIMy->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1; /* (3) */
	TIMx->EGR |= TIM_EGR_UG; /* (4) */
	TIMy->EGR |= TIM_EGR_UG; /* (5) */
	/* Configure the Timer Channel 2 as PWM */
	/* (1) Configure the Timer x Channel 2 waveform (TIM1_CCMR1 register)
	       is in PWM mode 1 (write OC2M = 110) */
	/* (2) Set TIMx prescaler to 2 */
	/* (3) Set TIMx Autoreload to 99 in order to get an overflow (so an UEV) each 10ms */
	/* (4) Set capture compare register to a value between 0 and 99 */
	TIMx->CCMR1 |= TIM_CCMR1_OC2M_2 | TIM_CCMR1_OC2M_1; /* (1) */
	TIMx->PSC = 2; /* (2) */
	TIMx->ARR = 99; /* (3) */
	TIMx->CCR2 = 25; /* (4) */
	/* Configure the slave timer Channel 1 as PWM as Timer to show synchronicity */
	/* (1) Configure the TIMy in PWM mode 1 (write OC1M = 110) */
	/* (2) Set TIMy prescaler to 2 */
	/* (3) Set TIMx Autoreload to 99 */
	/* (4) Set capture compare register to 25 */
	TIMy->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1; /* (1) */
	TIMy->PSC = 2; /* (2) */
	TIMy->ARR = 99; /* (3) */
	TIMy-> CCR1 = 25; /* (4) */
	/* Enable the output of TIMx OC1 */
	/* (1) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1)*/
	/* (2) Enable output (MOE = 1)*/
	TIMx->CCER |= TIM_CCER_CC2E; /* (1) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (2) */
	/* Enable the output of TIMy OC1 */
	/* (1) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1)*/
	/* (2) Enable output (MOE = 1)*/
	TIMy->CCER |= TIM_CCER_CC1E; /* (1) */
	TIMy->BDTR |= TIM_BDTR_MOE; /* (2) */  
当TIM1的TI1上出现上升沿，2个计数器按照内部时钟，同步开始计数；并且2个定时器的TIF标志都被置1。  
注：这个例子中，2个定时器在启动前都进行了初始化（设置各自的UG=1）。2个计数器都从0开始计数，但是你可以通过改变任一个计数器寄存器TIMx_CNT，在两者之间插入一个偏移量。你可以看到主/从模式在TIM1的CNT_EN和CK_PSC之间插入了一段延时，以使TIM1和TIM3同步计数。  
![](https://i.imgur.com/pxThtYP.png)  
###调试模式  
当微控制器进入调试模式（ARM Cortex-M0内核停止），TIMx计数器根据DBGMCU模块中的DBG_TIMx_STOP位的设置，正常工作或停止。  
##TIM3寄存器  
外设寄存器支持板子（16位）或字（32位）访问。  
###TIM3控制寄存器1（TIM3_CR1）  
![](https://i.imgur.com/xzRgeYX.png)  
![](https://i.imgur.com/q7tLhsu.png)  
![](https://i.imgur.com/aMrIpEC.png)  
