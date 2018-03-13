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
