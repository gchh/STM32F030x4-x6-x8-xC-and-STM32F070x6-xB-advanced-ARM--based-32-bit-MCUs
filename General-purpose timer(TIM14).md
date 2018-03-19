#General-purpose timer(TIM14)  
##TIM14简介  
通用定时器TIM14包含一个由可编程预分频器驱动的16位自动重载计数器。  
它可以用于多种用途，包括测量输入信号的脉冲宽度（输入捕获），或者产生输出波形（比较输出，PWM）。  
使用定时器预分频器和RCC时钟控制器预分频器，可以将脉冲宽度和波形周期从几微秒调到几毫秒。  
TIM14是完全独立的，没有共享任何资源，可以和其他定时器同步操作。  
##TIM14主要特性  
- 16位自动重装载向上计数器  
- 16位可编程预分频器，用于在1到65535之间分频时钟频率，可在运行时修改  
- 独立通道，用于：  
　- 输入捕获  
　- 比较输出  
　- PWM输出（边沿对齐模式）  
- 下列事件产生中断：  
　- 更新：计数器溢出，计数器初始化（由软件）  
　- 输入捕获  
　- 比较输出  
![](https://i.imgur.com/1WwVF1W.png)  
##TIM14功能描述  
###时基单元  
定时器的主要部分是一个带自动重载寄存器的16位计数器。该计数器只能向上计数。计数器时钟由预分频器提供。  
计数器，自动重载寄存器和预分频器寄存器可以由软件进行读写，即使计数器在运行中。  
时基单元包括：  
- 计数器寄存器TIMx_CNT  
- 预分频器寄存器TIMx_PSC  
- 自动重载寄存器TIMx_ARR  
自动重载寄存器是预装载的。对自动重载寄存器进行读写，实际是对预装载寄存器访问。取决于TIMx_CR1中自动重载预装载使能位ARPE的设置，预装载寄存器的值立即或在每次更新事件UEV时写入影子寄存器。如果TIMx_CR1中的UDIS=0，则当计数器溢出时产生更新事件。更新事件也可以通过软件产生。  
计数器由预分频器输出CK_CNT驱动，当TIMx_CR1中的CEN=1时才被使能。  
注意，在TIMx_CR1中的CEN置1后，经过1个时钟周期，计数器才真正启动。  
####预分频器描述  
预分频器可以将时钟频率在1到65535之间任意分频。它是基于16位计数器通过16位寄存器（在TIMx_PSC中）控制的。它可以被实时修改，因为它具有缓冲。在发生更新事件时，新写入的预分频器值才被写入影子寄存器，生效。  
图146和图147给出了当预分频器被实时修改时，计数器的行为。  
![](https://i.imgur.com/7kQ5pow.png)  
![](https://i.imgur.com/jyHbUPf.png)  
###计数模式  
TIM14的计数器只能向上计数，从0计数到自动重载值（TIMx_ARR中的值），然后重新从0开始计数，并产生一个计数器溢出事件。  
设置TIMx_EGR中的UG=1，也可以产生一个更新事件。  
如果TIMx_CR1中的UDIS=1，则禁止UEV产生。这样可以避免在向预装载寄存器写入新值过程中，发生更新，导致写入影子寄存器中的值发生错误。在UDIS=0之前，不会有更新事件发生。但是，计数器和预分频器的计数器都会重新从0开始计数（分频率不会改变）。另外，如果TIMx_CR1中的URS=1，则设置UG=1会产生更新事件，但是UIF不会被置1（也就不会产生中断）。这是为了避免在清除捕获事件的计数器时产生更新和捕获中断。  
当发生更新事件时，所有寄存器被更新，并且根据URS位的设置决定TIMx_SR中的UIF是否置位：  
- 自动重载影子寄存器被更新为预装载值（TIMx_ARR）  
- 预分频器的缓冲器重新载入预装载值（TIMx_PSC）  
下面的图显示了，当TIMx_ARR=0x36时，不同时钟频率下的计数器行为。  
![](https://i.imgur.com/aD8DSrT.png)  
![](https://i.imgur.com/wJvO1tv.png)  
![](https://i.imgur.com/xHXVcHz.png)  
![](https://i.imgur.com/Y8PfHE0.png)  
![](https://i.imgur.com/Zvp5kvR.png)  
![](https://i.imgur.com/ZimRAsX.png)  
###时钟源  
计数器时钟由内部时钟源CK_INT提供。  
TIMx_CR1中的CEN位和TIMx_EGR中的UG位是实际的控制位，并且它们只能由软件更改（除了，UG位由硬件自动清零）。一旦CEN=1，内部时钟CK_INT给预分频器提供时钟。  
图153显示了控制电路和向上计数，在正常模式和不分频情况下的行为。  
![](https://i.imgur.com/SwQucrg.png)  
###捕获/比较通道  
每个捕获/比较通道都是建立在一个捕获/比较寄存器（包含一个影子寄存器），一个捕获输入级（带数字滤波器，多路复用和预分频器）和一个输出级（带比较器和输出控制）。  
图154和图156给出了一个捕获/比较通道的概览。  
输入级采样相应的TIx输入产生一个滤波后的信号TIxF。然后，经过一个带极性选择的边沿检测器产生信号TIxFPx，它可以用作从模式控制器的触发输入或捕获命令。该信号先进行预分频（ICxPS），而后再进入捕获寄存器。  
![](https://i.imgur.com/94fvcJ5.png)  
输出级产生一个中间波形作为基准：OCxREF（高电平有效）。链的末端决定最终输出信号的极性。  
![](https://i.imgur.com/Et4176t.png)  
![](https://i.imgur.com/zM2t99w.png)  
捕获/比较模块由一个预装载寄存器和一个影子寄存器组成。读和写总是对预装载寄存器进行。  
在捕获模式下，捕获实际是在影子寄存器中进行的，然后将捕获值拷贝到预装载寄存器。  
在比较模式下，预装载寄存器的值拷贝到影子寄存器中，然后和计数器进行比较。  
###输入捕获模式  
在输入捕获模式下，捕获/比较寄存器TIMx_CCRx用于锁存计数器的值，当检测到相应的ICx信号上发生跳变时。当一个捕获发生时，TIMx_SR中的CCxIF置1，并且产生中断或DMA请求，如果它们被使能的话。如果当CCxIF=1时发生捕获，则TIMx_SR中的重复捕获标志CCxOF被置1。CCxIF可以通过软件向其写0将其清除，或通过读取存储在TIMx_CCRx寄存器中的捕获值将其清除。CCxOF可以通过软件向其写0将其清除。  
下面的例子显示了当TI1上升沿时，如何捕获计数器值到TIMx_CCR1中。要实现它，步骤如下：  
1. 选择有效的输入：TIMx_CCR1必须连接到TI1输入，设置TIMx_CCMR1中的CC1S=01。一旦CC1S≠00，该通道就配置为输入模式，并且TIMx_CCR1寄存器变为只读。  
2. 根据连接到定时器的输入信号，配置输入滤波器带宽（当输入是TIx之一时，设置TIMx_CCMRx中的ICxF位）。假设，输入信号的跳变在最多5个时钟周期内抖动，那么我们必须设置输入滤波带宽长于5个时钟周期。当8个连续采样检测到新电平（以f<sub>DTS</sub>为频率进行采样），我们就可以确认TI1发生跳变。向TIMx_CCMR1中写入IC1F=0011。  
3. 选择TI1上的有效边沿，通过设置TIMx_CCER中的CC1P=0和CC1NP=0，选择上升沿。  
4. 设置输入预分频器。在本例中，我们希望在每次有效边沿执行捕获，所以不需要使用预分频器，保持TIMx_CCMR1中的IC1PS=00。  
5. 使能捕获计数器的值到捕获寄存器，通过设置TIMx_CCER中的CC1E=1。  
6. 如果需要，设置TIMx_DIER中的CC1IE，使能中断。  
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
当发生输入捕获时：  
- TIMx_CCR1在输入有效边沿获取计数器的值。  
- CC1IF被置1。如果至少发生2次连续捕获而且CC1IF没有被清零，则CC1OF也会被置1。  
- 根据CC1IE的设置，产生或不产生中断。  
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
	/* Note: This code manages only a single counter overflow.
       To manage many counter overflows the update interrupt
       must be enabled (UIE = 1) and properly managed. */  
为了除了重复捕获，建议在读取重复捕获标志之前读取捕获数据。这样可以避免丢失，可能发生在读取标志之后与读取数据之前的重复捕获。  
注：设置TIMx_EGR中的CCxG=1，可以产生IC输入捕获中断。  
###强制输出模式  
在输出模式（TIMx_CCMRx中的CCxS=00）下，每个比较输出信号（OCxREF和OCx）可以直接由软件强制设置为有效电平或无效电平，不管输出比较寄存器和计数器的比较结果如何。  
要强制输出比较信号（OCxREF/OCx）为有效电平，只需要向TIMx_CCMRx中写入OCxM=101。然后OCxREF就被强制为高电平（OCxREF总是高电平有效），并且OCx的电平和CCxP设置的极性相反。  
例如：CCxP=0（OCx高电平有效）=> OCx被强制为高电平。  
向TIMx_CCMRx中写入OCxM=100，OCxREF信号被强制为低电平。  
在强制输出模式，TIMx_CCRx的影子寄存器和计数器的比较依然会执行，并且相关的标志位也会被置位；因此也会发生中断。  
###比较输出模式  
此功能用于控制输出波形，或指出一段时间已经过去。  
当捕获/比较寄存器和计数器匹配时，比较输出功能：  
1. 为相应的输出引脚分配一个可编程的值，该值由输出比较模式（TIMx_CCMRx中的OCxM）和输出极性（TIMx_CCER中的CCxP）定义。OCxM=000，匹配时不影响输出电平；OCxM=001，匹配时输出有效电平；OCxM=010，匹配时输出无效电平；OCxM=011，匹配时输出翻转。  
2. TIMx_SR中的CCxIF置1。  
3. 如果TIMx_DIER中的CCxIE=1，则产生中断。  
通过设置TIMx_CCMRx中的OCxPE位，TIMx_CCRx可以选择使不使用预装载寄存器。  
在比较输出模式下，更新事件UEV不会影响OCxREF和OCx的输出。同步的精度可以达到计数器的一个计数周期。比较输出模式也可以用于输出一个单脉冲（在单脉冲模式下）。  
比较输出模式配置步骤：  
1. 选择计数器时钟（内部，外部，预分频器）。  
2. 将相应的数据写入TIMx_ARR和TIMx_CCRx中。  
3. 如果要求中断，设置CCxIE=1。  
4. 选择比较输出模式。例如：  
　- 设置OCxM=011，当CNT匹配CCRx时，OCx输出电平发生翻转。  
　- 设置OCxPE=0，禁用预装载寄存器。  
　- 设置CCxP=0，选择高电平有效。  
　- 设置CCxE=1，使能输出。  
5. 设置TIMx_CR1中的CEN=1，使能计数器。  
######Output compare configuration code example  

	/* (1) Set prescaler to 3, so APBCLK/4 i.e 12MHz */
	/* (2) Set ARR = 12000 -1 */
	/* (3) Set CCRx = ARR, as timer clock is 12MHz, an event occurs each 1ms */
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
TIMx_CCRx寄存器的值可以由软件在任意时刻更新，但，前提是禁用预装载寄存器（OCxPE=0，否则只有发生更新事件时TIMx_CCRx的影子寄存器才被更新）。图157给出了一个例子。  
![](https://i.imgur.com/Z6aRFV3.png)  
###输出PWM模式  
PWM模式允许你产生一个，由TIMx_ARR决定频率和由TIMx_CCRx决定占空比的，PWM信号。  
每个通道可以独立选择PWM模式，通过向TIMx_CCMRx中写入OCxM=110选择PWM模式1或OCxM=111选择PWM模式2。必须设置TIMx_CCMRx中的OCxPE=1和TIMx_CR1中的ARPE=1，使能比较预装载寄存器和自动重载预装载寄存器。  
因为当更新事件发生时，预装载寄存器的值才被拷贝到影子寄存器中，所以在启动计数器之前，要设置TIMx_EGR中的UG=1来初始化所有寄存器。  
OCx的极性可通过TIMx_CCER中的CCxP位来编程为高电平有效或低电平有效。TIMx_CCER中的CCxE位控制OCx输出使能。  
在PWM模式1或模式2下，TIMx_CNT和TIMx_CCRx始终进行比较，以确定是否TIMx_CNT≤TIMx_CCRx。  
因为TIM14中的计数器只能向上计数，所以只能在边沿对齐模式下产生PWM。  
以PWM模式1为例。当TIMx_CNT<TIMx_CCRx时，PWM参考信号OCxREF是高电平，否则为低电平。如果TIMx_CCRx>TIMx_ARR，那么OCxREF将保持为1。如果TIMx_CCRx=0，则OCxREF保持为0。图158显示了当TIMx_ARR=8时PWM的波形。  
![](https://i.imgur.com/L2kijNZ.png)  
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
###调试模式  
当微控制器进入调试模式（Cortex-M0内核停止），TIMx计数器是正常工作还是停止，取决于DBG模块中DBG_TIMx_STOP配置位的设置。  
##TIM14寄存器  
###TIM14控制寄存器1（TIM14_CR1）  
![](https://i.imgur.com/saotSH4.png)  
![](https://i.imgur.com/nrnIidj.png)  
###TIM14中断使能寄存器（TIM14_DIER）  
![](https://i.imgur.com/LK4HYYt.png)  
###TIM14状态寄存器（TIM14_SR）  
![](https://i.imgur.com/q8YSZUV.png)  
![](https://i.imgur.com/GmeueDS.png)  
###TIM14事件生成寄存器（TIM14_EGR）  
![](https://i.imgur.com/emqPUPq.png)  
###TIM14捕获/比较模式寄存器1（TIM14_CCMR1）  
![](https://i.imgur.com/B0Sb1Tf.png)  
![](https://i.imgur.com/gcqii9Z.png)  
![](https://i.imgur.com/Vtzb8S7.png)  
![](https://i.imgur.com/ZhcqwFG.png)  
###TIM14捕获/比较使能寄存器（TIM14_CCER）  
![](https://i.imgur.com/uL1Hj84.png)  
![](https://i.imgur.com/96uag2n.png)  
![](https://i.imgur.com/DIh0iwa.png)  
###TIM14计数器（TIM14_CNT）  
![](https://i.imgur.com/w0rfEbU.png)  
###TIM14预分频器（TIM14_PSC）  
![](https://i.imgur.com/zHxqKKX.png)  
###TIM14自动重载寄存器（TIM14_ARR）  
![](https://i.imgur.com/SIFWr2F.png)  
###TIM14捕获/比较寄存器1（TIM14_CCR1）  
![](https://i.imgur.com/HqfwjJf.png)  
###TIM14选项寄存器（TIM14_OR）  
![](https://i.imgur.com/GoDlIpF.png)  
##TIM14寄存器映射  
![](https://i.imgur.com/vPdgBbq.png)  
![](https://i.imgur.com/6oOiA5r.png)  
![](https://i.imgur.com/Dw8QBh0.png)  
