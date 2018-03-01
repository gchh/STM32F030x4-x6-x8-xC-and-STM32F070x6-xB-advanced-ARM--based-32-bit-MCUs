#Advanced-control timers(TIM1)  
##TIM1介绍  
高级控制定时器TIM1有一个可编程预分频器驱动的16位自动重装载计数器。  
TIM1可以用于测量脉冲宽度（输入捕获），或产生输出波形（输出比较，PWM，嵌入死区时间的互补PWM）。  
通过设置定时器分频器和RCC时钟控制分频器，脉冲宽度和波形周期可以从几微秒到几毫秒调节。  
高级控制定时器TIM1和通用定时器TIMx是完全独立的，它们没有共享任何资源。它们可以同时操作。  
##TIM1主要特性  
- 16位向上，向下，向上/向下自动重载计数器  
- 16位可编程预分频器，可以动态设置，将计数器时钟频率分频为1~65535分之一。  
- 4个独立通道：  
　- 输入捕获  
　- 输出比较  
　- PWM生成（边沿或中心对齐模式）  
　- 单脉冲模式输出  
- 带可编程死区时间的互补输出  
- 用外部信号控制定时器并将多个定时器连接在一起的同步电路。  
- 重复计数器仅在给定的计数器周期后更新定时器寄存器  
- 刹车输入可以将定时器的输出信号置于复位或一个已知状态  
- 以下事件会产生中断/DMA请求：  
　- 更新：计数器向上/向下溢出，计数器初始化（由软件或内部/外部触发）  
　- 触发事件（计数开始，停止，初始化或由内部/外部触发计数）  
　- 输入捕获  
　- 输出比较  
　- 刹车输入  
- 支持用于定位的增量（正交）编码器和霍尔传感器  
- 触发输入用作外部时钟或按周期电流管理  
####TIM1框图  
![](https://i.imgur.com/8hJFtM0.png)  
##TIM1功能描述  
###时基单元  
可编程高级控制定时器的主要部分是一个16位的计数器和相关的自重载寄存器。计数器可以向上计数，向下计数，或是上下双向计数。计数器的时钟由预分频器分频得到。  
计数器，自动重载寄存器和预分频器寄存器可以由软件读写，即使计数器在运行。  
时基单元包括：  
- 计数器寄存器TIMx_CNT  
- 预分频器寄存器TIMx_PSC  
- 自动重装载寄存器TIMx_ARR  
- 重复计数寄存器TIMx_RCR  
自动重装载寄存器是预先载入的。读或写TIMx_ARR将访问预装载寄存器。取决于TIMx_CR1寄存器的自动重载/预装载使能位ARPE，预装载寄存器的内容被立即或在每次更新事件（UEV）时传输到影子寄存器。当计数器溢出并且TIMx_CR1的UDIS=0，将产生更新事件。更新事件也可以由软件产生。  
计数器时钟由预分频器输出CK_CNT提供，仅当TIMx_CR1的计数器使能位CEN=1时CK_CNT才被使能。  
注意，当设置TIMx_CR1的CEN=1，一个时钟周期后，计数器才开始计数。  
####预分频器  
预分频器可以将计数器频率按1到65536之间的任意值分频。它是基于16位计数器通过一个16位寄存器（TIMx_PSC）控制的。由于TIMx_PSC带有缓冲器，所以它能在运行时被改变。新的分频因子将在下个更新事件产生时，被预分频器采用。  
![](https://i.imgur.com/LJiwzkb.png)  
![](https://i.imgur.com/V39P3s3.png)  
###计数器模式  
####向上计数模式  
在向上计数模式，计数器从0计数到自动重装载值（TIMx_ARR里的内容），然后重新从0开始计数并产生一个计数器溢出事件。  
如果使用了重复计数器，当向上计数重复次数达到了重复计数寄存器TIMx_RCR设置的重复次数，才产生更新事件UEV；否则，没有使用重复计数器，每次计数器溢出都会产生更新事件。  
设置TIMx_EGR寄存器的UG位（通过软件或从模式控制器），也可以产生更新事件。  
TIMx_CR1的UDIS=1，可以禁止更新事件UEV。这样可以避免在向预装载寄存器写入新值的过程中，更新影子寄存器。在UDIS=0之前，将不会产生更新事件；但是计数器溢出后依然会从0开始计数，预分频器的计数器也一样（但是，分频因子不会变）。另外，如果TIMx_CR1的URS（更新请求选择）=1，当UG=1，会产生更新事件UEV但不会将UIF置位（不会产生中断或DMA请求）。这是为了避免在清除捕获事件的计数器时，同时产生更新和捕获中断。  
当发生更新事件时，所有寄存器被更新，并且将TIMx_SR的UIF更新标志位置1（取决于URS位）：  
- 重复计数器重新载入TIMx_RCR寄存器的值  
- 自动重装载影子寄存器重新更新为TIMx_ARR寄存器的值  
- 预分频器的缓冲器重新装入TIMx_PSC寄存器的值  
下面是当TIMx_ARR=0x36时，不同时钟频率下，计数器的动作图示。  
![](https://i.imgur.com/LHlKzql.png)  
![](https://i.imgur.com/Eh4Bk9G.png)  
![](https://i.imgur.com/SdiC4na.png)  
![](https://i.imgur.com/MclhOAh.png)  
![](https://i.imgur.com/NDjHYeE.png)  
![](https://i.imgur.com/vKcmfR2.png)  
####向下计数模式  
在向下计数模式，计数器从自动重装载值（TIMx_ARR寄存器的内容）向下计数到0，然后重新从自动重装载值计数并产生一个向下溢出事件。  
如果使用重复计数器，当向下计数次数达到TIMx_RCR设置的重复次数，才会产生更新事件UEV；否则，每次计数向下溢出都会产生更新事件。  
设置TIMx_EGR寄存器的UG位（通过软件或从模式控制器），也可以产生更新事件。  
TIMx_CR1的UDIS=1，可以禁止更新事件UEV。这样可以避免向TIMx_ARR寄存器写入新值过程中发生更新事件，将TIMx_ARR影子寄存器的值更新。但是计数器溢出后依然会从当前自动重载值开始计数，预分频器的计数器从0计数（但是，分频因子不会变）。  
另外，如果TIMx_CR1的更新请求选择位URS=1，设置UG=1，会产生更新事件UEV但是不会置位UIF标志位（不会产生中断或DMA请求）。这是为了避免在清除捕获事件的计数器时，同时产生更新和捕获中断。  
当发生更新事件时，所有寄存器被更新，并且将TIMx_SR的UIF更新标志位置1（取决于URS位）：  
- 重复计数器重新载入TIMx_RCR的值  
- 预分频器的缓冲器重新写入TIMx_PSC的值  
- 自动重装载影子寄存器重新更新为TIMx_ARR的值。注意，如果在计数器更新前，改变TIMx_ARR的值，只有等以后发生更新事件后，才有效。  
下面是当TIMx_ARR=0x36时，不同时钟频率下，计数器的动作图示。  
![](https://i.imgur.com/bCDdKVQ.png)  
![](https://i.imgur.com/EykzCjo.png)  
![](https://i.imgur.com/oKjFyzp.png)  
![](https://i.imgur.com/i0NdWnL.png)  
![](https://i.imgur.com/IALM63l.png)  
####中心对齐模式（向上/向下计数）  
在中心对齐模式，计数器从0计数到自动重装载值（TIMx_ARR的值）-1，产生计数溢出事件；然后计数器从自动重装载值计数到1，产生计数下溢事件。然后从0开始重新计数。  
TIMx_CR1的CMS位不等于'00'时，使用中心对齐模式。配置为输出的通道的输出比较中断标志在以下情况置位：计数器向下计数（中心对齐模式1，CMS='01'），计数器向上计数（中心对齐模式2，CMS='10'），计数器向上和向下计数（中心对齐模式3，CMS='11'）。  
在此模式，TIMx_CR1的DIR方向位不能写入。DIR由硬件更新，指示计数器当前方向。  
更新事件产生在每次计数器溢出，下溢或TIMx_EGR的UG置1。此时，计数器从0重新计数，预分频器计数器也重新从0计数。  
TIMx_CR1的UDIS位设置为1，可以禁止产生UEV更新事件。这样可以避免在向预装载寄存器写入新值时更新影子寄存器。在UDIS清零前，不会有更新事件发生。但是，计数器依然会根据当前的自动重载值，继续向上和向下计数。  
此外，如果TIMx_CR1的更新请求选择位URS=1，将UG置位将产生UEV更新事件而UIF标志不会置位（不会产生中断或DMA请求）。这样就避免了在发生捕获事件清零计数器时，同时产生更新和捕获中断。  
当发生更新事件时，所有寄存器器被更新；TIMx_SR中的UIF更新标志位根据URS位的值，决定是否置位：  
- 重复计数器重新载入TIMx_RCR寄存器的值  
- 预分频器的缓冲器重新载入TIMx_PSC寄存器的值  
- 自动重载活动寄存器重新载入TIMx_ARR寄存器的值。注意，如果因为计数器溢出发生更新，自动重载将在计数器之前被更新，因此在下一个周期才是预期值（计数器被载入新值）。  
![](https://i.imgur.com/mKTzClG.png)  
![](https://i.imgur.com/gzsIr3y.png)  
![](https://i.imgur.com/3BZbPtc.png)  
![](https://i.imgur.com/4UxrKPl.png)  
![](https://i.imgur.com/gmKHjow.png)  
![](https://i.imgur.com/IQwhF3I.png)  
###重复计数器  
实际上，只有当重复计数器达到0时，才会产生更新事件。这个特性对于产生PWM信号很有用。  
这意味着每N次计数器溢出或下溢，数据将从预载入寄存器传入影子寄存器（TIMx_ARR自动重载寄存器，TIMx_PSC预分频器寄存器，还有比较模式下的TIMx_CCRx捕获/比较寄存器），N是TIMx_RCR重复计数器寄存器的值。  
重复计数器递减：  
- 在向上计数模式，每次计数器溢出时  
- 在向下计数模式，每次计数器下溢时  
- 在中心对齐模式，每次计数器溢出或下溢时。虽然这样限制了最大重复数为128个PWM周期，但这样每个PWM周期可以更新2次占空比。当在中心对齐模式下每个PWM周期只刷新一次比较寄存器，由于波形的对称性，最大的分辨率是2xT<sub>ck</sub>。  
重复计数器是自动重加载的；重复速率是由TIMx_RCR寄存器的值定义的。当由软件（TIMx_EGR寄存器的UG位置1）或通过从模式控制器由硬件产生更新事件时，无论重复计数器的值是多少，更新会立即发生，重复计数器会重新载入TIMx_RCR寄存器的内容。  
在中心对齐模式，如果RCR的值是奇数，当RCR寄存器被写入并且计数器启动，在溢出或下溢时会发生更新事件。如果RCR在计数器启动前被写入，在溢出时发生UEV更新事件。如果RCR在计数器开始后被写入，在下溢时发生UEV更新事件。例如RCR=3，RCR被写入后，每4个溢出或下溢产生UEV更新事件。  
######Figure 60. Update rate examples depending on mode and TIMx_RCR register settings  
![](https://i.imgur.com/ud6mkbE.png)  
###时钟源  
计数器时钟可由以下时钟源分频得到：  
- 内部时钟CK_INT  
- 外部时钟模式1：外部输入引脚  
- 外部时钟模式2：外部触发输入ETR  
- 内部触发输入ITRx：使用一个定时器作为另一个定时器的预分频器，例如，可以配置Timer1作为Timer2的预分频器。  
####内部时钟源CK_INT  
如果禁止了从模式控制器(SMS=000)，则CEN，DIR(在TIMx_CR1寄存器中)和UG(在TIMx_EGR寄存器中)就是实际的控制位，并且只能被软件修改（除非UG被自动清零）。一旦CEN=1，内部时钟CK_INT就提供给预分频器作为时钟。  
Figure61显示了不带预分频器时，控制电路和向上计数器在正常模式下的动作。  
![](https://i.imgur.com/7XFKp5F.png)  
####外部时钟源模式1  
当TIMx_SMCR寄存器的SMS=111时，选择此模式。计数器在选定的输入引脚的每个上升沿或下降沿计数。  
![](https://i.imgur.com/nuBmoQq.png)  
例如，要配置向上计数器在TI2输入端的上升沿计数，步骤如下：  
1. 写入TIMx_CCMR1寄存器的CC2S='01'，配置通道2检测TI2输入的上升沿  
2. 写入TIMx_CCMR1寄存器的IC2F[3:0]，配置输入滤波器的带宽（如果不需要滤波器，保持IC2F=0000）  
3. 写入TIMx_CCER寄存器的CC2P=0，选择上升沿  
4. 写入TIMx_SMCR寄存器的SMS=111，配置定时器为外部时钟模式1  
5. 写入TIMx_SMCR寄存器的TS=110，选择TI2作为触发器输入源  
6. 写入TIMx_CR1寄存器的CEN=1，使能计数器   
注：捕获预分频器不用作触发，所以不需要配置它。  
######Upcounter on TI2 rising edge code example  

	/* (1) Enable the peripheral clock of Timer 1 */
	/* (2) Enable the peripheral clock of GPIOA */
	/* (3) Select Alternate function mode (10) on GPIOA pin 9 */
	/* (4) Select TIM1_CH2 on PA9 by enabling AF2 for pin 9 in GPIOA AFRH register */
	RCC->APB2ENR |= RCC_APB2ENR_TIM1EN; /* (1) */
	RCC->AHBENR |= RCC_AHBENR_GPIOAEN; /* (2) */
	GPIOA->MODER = (GPIOA->MODER & ~(GPIO_MODER_MODER9)) | (GPIO_MODER_MODER9_1); /* (3) */
	GPIOA->AFR[1] |= 0x2 << ((9-8)*4); /* (4) */
	/* (1) Configure channel 2 to detect rising edges on the TI2 input by
	       writing CC2S = ‘01’, and configure the input filter duration by
	       writing the IC2F[3:0] bits in the TIMx_CCMR1 register (if no filter
	       is needed, keep IC2F=0000).*/
	/* (2) Select rising edge polarity by writing CC2P=0 in the TIMx_CCER
           register (reset value). */
	/* (3) Configure the timer in external clock mode 1 by writing SMS=111
	       Select TI2 as the trigger input source by writing TS=110
	       in the TIMx_SMCR register.*/
	/* (4) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_IC2F_0 | TIM_CCMR1_IC2F_1 | TIM_CCMR1_CC2S_0; /* (1) */
	TIMx->CCER &= (uint16_t)(~TIM_CCER_CC2P); /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS | TIM_SMCR_TS_2 | TIM_SMCR_TS_1; /* (3) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (4) */  
当TI2上出现上升沿时，计数器计一次数，且TIF标志置位。  
在TI2上升沿和计数器实际时钟间的延时，取决于TI2输入端的重新同步电路。  
![](https://i.imgur.com/3JjrWKu.png)  
####外部时钟源模式2  
当TIMx_SMCR寄存器的ECE=1时，选择此模式。计数器在外部触发输入ETR的每个上升沿或下降沿计数。  
![](https://i.imgur.com/JEjMS92.png)  
例如，要配置向上计数器在ETR的每2个上升沿计一次数，步骤如下：  
1. 因为这个例子不需要滤波器，所以写入TIMx_SMCR寄存器的ETF[3:0]='0000'  
2. 写入TIMx_SMCR寄存器的ETPS[1:0]='01'，设置预分频器  
3. 写入TIMx_SMCR寄存器的ETP=0，选择检测上升沿  
4. 写入TIMx_SMCR寄存器的ECE=1，使能外部时钟模式2  
5. 写入TIMx_CR1寄存器的CEN=1,使能计数器  
计数器将每2个ETR上升沿计数一次。  
######Up counter on each 2 ETR rising edges code example  

	/* (1) Enable the peripheral clock of Timer 1 */
	/* (2) Enable the peripheral clock of GPIOA */
	/* (3) Select Alternate function mode (10) on GPIOA pin 12 */
	/* (4) Select TIM1_ETR on PA12 by enabling AF2 for pin 12 in GPIOA AFRH register */
	RCC->APB2ENR |= RCC_APB2ENR_TIM1EN; /* (1) */
	RCC->AHBENR |= RCC_AHBENR_GPIOAEN; /* (2) */
	GPIOA->MODER = (GPIOA->MODER & ~(GPIO_MODER_MODER12)) | (GPIO_MODER_MODER12_1); /* (3) */
	GPIOA->AFR[1] |= 0x2 << ((12-8)*4); /* (4) */
	/* (1) As no filter is needed in this example, write ETF[3:0]=0000
	       in the TIMx_SMCR register. Keep the reset value.
	       Set the prescaler by writing ETPS[1:0]=01 in the TIMx_SMCR register.
	       Select rising edge detection on the ETR pin by writing ETP=0
	       in the TIMx_SMCR register. Keep the reset value.
	       Enable external clock mode 2 by writing ECE=1 in the TIMx_SMCR register. */
	/* (2) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->SMCR |= TIM_SMCR_ETPS_0 | TIM_SMCR_ECE; /* (1) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (2) */  
在ETR上升沿和计数器实际时钟之间的延时，取决于ETRP信号的重新同步电路。  
![](https://i.imgur.com/r4Hlrht.png)  
###捕获/比较通道  
每个捕获/比较通道包括一个捕获/比较寄存器（包含影子寄存器），一个捕获输入级（数字滤波器，多路复用和预分频器）和一个输出级（比较器和输出控制）。  
输入级采样相应的TIx输入信号，产生一个滤波后的信号TIxF。然后，一个带极性选择的边沿检测器产生TIxFPx信号，它可以用作从模式控制器的触发输入或捕获控制。在输入到捕获寄存器前信号会被分频为ICxPS。  
![](https://i.imgur.com/uVRrHOC.png)  
输出级产生一个中间波形OCxRef(高有效)作为基准。链的末端决定最终输出信号的极性。  
![](https://i.imgur.com/0vDPpKF.png)  
![](https://i.imgur.com/DqdIQK6.png)  
![](https://i.imgur.com/Fszvcbj.png)  
捕获/比较模块由一个预装载寄存器和一个影子寄存器组成。读写仅对预装载寄存器进行。  
在捕获模式，捕获实际发生在影子寄存器，然后再复制到预装载寄存器。  
在比较模式，预装载寄存器的内容复制到影子寄存器，然后影子寄存器的内容和计数器进行比较。  
###输入捕获模式  
在输入捕获模式下，当检测到相应的ICx信号边沿跳变，计数器的值被锁存到捕获/比较寄存器。当捕获发生时，TIMx_SR寄存器相应的CCxIF标志被置1；并且如果使能了中断或DMA，则将产生一个中断或DMA请求。如果在CCxIF=1时发生了捕获，TIMx_SR寄存器的捕获溢出标志CCxOF被置1。可以通过软件写CCxIF=0或读取存储在TIMx_CCRx寄存器的捕获数据，清除CCxIF标志。软件写CCxOF=0，清除CCxOF标志。  
下面的例子说明如何在TI1输入的上升沿捕获计数器的值到TIMx_CCR1中，步骤如下：  
- 选择有效的输入：写TIMx_CCMR1寄存器的CC1S=01，TIMx_CCR1链接到TI1输入。一旦CC1S≠00，通道就会被配置为输入，并且TIMx_CCR1寄存器只能读取。  
- 根据输入到计数器的信号的要求，配置输入滤波器带宽（输入是TIx时，TIMx_CCMRx寄存器的ICxF[3:0]用于配置滤波器）。假设输入信号在跳变时，需要最多5个内部时钟周期才能稳定，那么我们必须将滤波器带宽设置长于5个时钟周期。我们可以配置TIMx_CCMR1寄存器的IC1F=0011，对TI1输入信号连续采样8次（以f<sub>DTS</sub>频率采样），检测到新电平，就可以确认一次跳变。  
- 配置TIMx_CCER寄存器的CC1P=0和CC1NP=0，选择TI1通道上升沿有效。  
- 配置输入预分频器。在本例中，我们希望在每次有效边沿执行捕获，所以禁止预分频器（TIMx_CCMR1寄存器的IC1PS[1:0]=00）。  
- 写TIMx_CCER寄存器的CC1E=1，允许将计数器的值捕获到捕获寄存器TIMx_CCR1中。  
- 如果需要，设置TIMx_DIER寄存器的CC1IE=1使能中断或CC1DE=1使能DMA请求。  
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
当一个输入捕获发生：  
- 在有效边沿，TIMx_CCR1获取计数器的值。  
- 中断标志CC1IF=1。如果发生至少2个连续的捕获，而CC1IF=1未被清除，则CC1OF也会被置1。  
- 如果CC1IE=1则会产生中断。  
- 如果CC1DE=1则会产生DMA请求。  
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
			counter0 = TIMx->CCR1; /* Read the capture counter which clears the CC1IF */
			gap = 1; /* Indicate that the first rising edge has yet been detected */
		}
		else
		{
			counter1 = TIMx->CCR1; /* Read the capture counter which clears the CC1IF */
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
	/*此代码只是管理单个计数器溢出。要管理多个计数器溢出，必须使能更新中断（UIE=1），进行合适的管理*/  
为了处理捕获溢出，建议在读取捕获溢出标志前先读出捕获数据，这是为了避免丢失在读取捕获溢出标志之后和在读取数据之前可能发生的捕获溢出。  
注：通过软件设置TIMx_EGR寄存器中相应的CCxG位，也可以产生输入捕获中断和/或DMA请求。  
###PWM输入模式  
该模式是输入捕获模式的一个特例。除了下列区别，其他操作同输入捕获模式一样：  
- 2个ICx信号被映射到同一个TIx输入。  
- 这2个ICx信号的有效边沿极性相反。  
- 2个TIxFP信号其中一个用作触发输入，而从模式控制器配置成复位模式。  
例如，你可以测量TI1上的PWM信号的周期（TIMx_CCR1寄存器）和占空比（TIMx_CCR2寄存器），步骤如下（取决于CK_INT频率和预分频器值）：  
- 选择TIMx_CCR1的有效输入：写TIMx_CCMR1的CC1S=01，选择TI1。  
- 选择TI1FP1的有效极性（用来捕获数据到TIMx_CCR1和清除计数器）：写TIMx_CCER寄存器的CC1P=0和CC1NP=0，选择上升沿。  
- 选择TIMx_CCR2的有效输入：写TIMx_CCMR1的CC2S=10，选择TI1。  
- 选择TI1FP2的有效极性（用来捕获数据到TIMx_CCR2）：写TIMx_CCER寄存器的CC2P=1和CC2NP=0，选择下降沿。  
- 选择有效的触发输入信号：写TIMx_SMCR寄存器的TS[2:0]=101，选择TI1FP1。  
- 配置从模式控制器为复位模式：写TIMx_SMCR寄存器的SMS[2:0]=100。  
- 使能捕获：写TIMx_CCER寄存器的CC1E=1和CC2E=1。  
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
![](https://i.imgur.com/TfmjOV7.png)  
###强制输出模式  
在输出模式（TIMx_CCMRx寄存器的CCxS[1:0]=00），每个输出比较信号（OCxREF和相应的OCx/OCxN）能够由软件直接强制为有效或无效状态，而不依赖于输出比较寄存器和计数器的比较结果。  
相应的TIMx_CCMRx寄存器写OCxM[2:0]=101，即可强制输出比较信号（OCxREF/OCx）为有效状态。这样OCxREF强制为高电平（OCxREF始终为高电平有效），而OCx得到和TIMx_CCER寄存器中CCxP极性相反的信号。  
例如：设置CCxP=0(OCx高电平有效)=>OCx被强制为高电平。  
TIMx_CCMRx寄存器的OCxM[2:0]=100，强制OCxREF为低电平。  
在该模式下，TIMx_CCRx影子寄存器和计数器的比较依然会进行，相应的标志位也会置位。因此，中断和DMA请求仍然会产生。这将在下节的输出比较模式进行描述。  
###输出比较模式  
该功能可以用于控制输出波形或指示一段给定的时间已经结束。  
当捕获/比较寄存器和计数器的内容相同时，输出比较功能做以下操作：  
- 将输出比较模式（TIMx_CCMRx寄存器的OCxM[2:0]位）和输出极性（TIMx_CCER寄存器的CCxP位）定义的值输出到相应的引脚上。输出引脚保持它的电平（OCxM[2:0]=000），比较结果相同时设置为有效电平（OCxM[2:0]=001）或无效电平（OCxM[2:0]=010），抑或电平翻转（OCxM[2:0]=011）。  
- 置位中断状态寄存器中的标志位（TIMx_SR中的CCxIF位）。  
- 如果使能了相应的中断（TIMx_DIER中的CCxIE=1），则产生中断。  
- 如果使能了相应的DMA（TIMx_DIER中的CCxDE=1，TIMx_CR2中的CCDS选择何时发送DMA请求），则发出DMA请求。  
设置TIMx_CCMRx寄存器中的OCxPE位，可以选择TIMx_CCRx寄存器是否使用预装载寄存器。  
在输出比较模式，更新事件UEV不影响OCxREF和OCx的输出。时间精度是计数器的一个计数周期。输出比较模式也可以用来输出一个单个脉冲（在单脉冲模式下）。  
输出比较模式的配置步骤：  
1. 选择计数器的时钟（内部，外部，预分频器）。  
2. 向TIMx_ARR和TIMx_CCRx写入要求的数据。  
3. 如果要求产生中断，将CCxIE置1。  
4. 选择输出模式。例如：  
　- 设置OCxM[2:0]=011，当CNT=CCRx时，OCx输出电平翻转  
　- 设置OCxPE=0，禁用预装载寄存器  
　- 设置CCxP=0，选择高电平有效  
　- 设置CCxE=1，使能输出  
5. TIMx_CR1中的CEN=1，使能计数器。  
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
TIMx_CCRx任何时候都可以通过软件更新以控制输出波形，前提是没有使用预装载寄存器（OCxPE=0，否则TIMx_CCRx影子寄存器只有在下次发生更新事件UEV时才被更新为新值）。  
![](https://i.imgur.com/h6EOQgM.png)  
###PWM模式  
PWM模式可以产生一个由TIMx_ARR寄存器确定频率、由TIMx_CCRx寄存器确定占空比的信号。  
设置TIMx_CCMRx寄存器的OCxM[2:0]=110（PWM模式1）或OCxM[2:0]=111（PWM模式2），每个通道可以独立设置为PWM模式，一个OCx输出一路PWM信号。必须通过设置TIMx_CCMRx寄存器的OCxPE=1，使能相应的预装载寄存器。然后，通过设置TIMx_CR1寄存器的ARPE=1，使能自动重载的预装载寄存器（在向上计数或中心对齐模式中）。  
当更新事件发生时预装载寄存器的值才会被传送到影子寄存器中，因此在计数器开始计数前，必须通过设置TIMx_EGR寄存器的UG=1使用软件产生更新事件，来初始化所有寄存器。  
OCx的极性可以通过软件设置TIMx_CCER寄存器的CCxP位选择高电平有效或低电平有效。OCx输出由CCxE,CCxNE,MOE,OSSI和OSSR（在TIMx_CCER和TIMx_BDTR寄存器中）组合控制使能。详见TIMx_CCER寄存器的描述。  
在PWM模式（模式1或2），TIMx_CNT和TIMx_CCRx一直进行比较，以确定是否TIMx_CCRx≤TIMx_CNT或TIMx_CNT≤TIMx_CCRx（取决于计数器的计数方向）。  
根据TIMx_CR1寄存器CMS[1:0]位的设置，定时器可以产生边沿对齐或中心对齐的PWM信号。  
####PWM边沿对齐模式  
#####向上计数配置  
当TIMx_CR1寄存器的DIR=0时，选择向上计数。  
下面是一个PWM模式1的例子。当TIMx_CNT＜TIMx_CCRx，参考PWM信号OCxREF为高电平，否则为低电平。如果TIMx_CCRx中的比较值大于TIMx_ARR中的值，则OCxREF会一直为高电平。如果TIMx_CCRx=0，则TIMx_CNT永远不会小于TIMx_CCRx，所以OCxREF会一直为低电平。  
![](https://i.imgur.com/LDy36f2.png)  
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
#####向下计数配置  
当TIMx_CR1寄存器的DIR=1时，选择向下计数。  
在PWM模式1，当TIMx_CNT＞TIMx_CCRx时，参考PWM信号OCxREF是低电平，否则为高电平。如果TIMx_CCRx的值大于TIMx_ARR的值，则OCxREF会一直保持为高电平。该模式不会产生0%的PWM波形（即使TIMx_CCRx=0，OCxREF也不会一直保持低电平）。  
####PWM中心对齐模式  
当TIMx_CR1寄存器的CMS[1:0]≠00（所有其他配置对OCxREF/OCx信号有相同的作用）时，即为中心对齐模式。根据CMS[1:0]的设置，当计数器向上计数时，向下计数时或是两者，比较标志会被置位。TIMx_CR1寄存器中的方向标志位DIR由硬件更新，不能用软件修改它。  
![](https://i.imgur.com/XQc2GG4.png)  
######Center-aligned PWM configuration example  

	/* (1) Set prescaler to 47, so APBCLK/48 i.e 1MHz */
	/* (2) Set ARR = 8, as timer clock is 1MHz and center-aligned counting,
	       the period is 16 us */
	/* (3) Set CCRx = 7, the signal will be high during 14 us */
	/* (4) Select PWM mode 1 on OC1 (OC1M = 110),
	       enable preload register on OC1 (OC1PE = 1) */
	/* (5) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1)*/
	/* (6) Enable output (MOE = 1)*/
	/* (7) Enable counter (CEN = 1)
	       select center-aligned mode 1 (CMS = 01) */
	/* (8) Force update generation (UG = 1) */
	TIMx->PSC = 47; /* (1) */
	TIMx->ARR = 8; /* (2) */
	TIMx->CCR1 = 7; /* (3) */
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1PE; /* (4) */
	TIMx->CCER |= TIM_CCER_CC1E; /* (5) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (6) */
	TIMx->CR1 |= TIM_CR1_CMS_0 | TIM_CR1_CEN; /* (7) */
	TIMx->EGR |= TIM_EGR_UG; /* (8) */  
使用中心对齐模式的建议：  
- 当使用中心对齐模式并启动计数器，计数是按TIMx_CR1中DIR的当前值所确定的方向进行。此外，必须确保软件不能同时修改DIR和CMS位。  
- 不建议在中心对齐模式运行时改写计数器，这可能会导致不可预知的结果。特别是：  
　- 如果写入计数器的值大于自装载值（TIMx_CNT>TIMx_ARR），计数方向不会更新。例如，计数器正向上计数，则它会继续向上计数。  
　- 如果将0或TIMx_ARR的值写入计数器，计数方向会更新，但是不会产生更新事件UEV。  
- 使用中心对齐方式最安全的方法是，在启动计数器之前通过软件设置TIMx_EGR寄存器的UG=1产生一个更新事件，并且在计数过程中不要修改计数器的值。  
###互补输出和死区时间插入  
高级控制定时器TIM1可以输出2个互补的信号，并控制输出瞬间的关断和接通。  
输出关断的这段时间通常称为死区，你需要根据连接到输出上的器件和它们的特性（电平转换的延时，电源开关造成的延时等等）来调整死区时间。  
配置TIMx_CCER寄存器的CCxP和CCxNP位，可以独立选择每一个输出的极性（主输出OCx或互补输出OCxN）。  
互补信号OCx和OCxN由下列控制位组合控制：TIMx_CCER中的CCxE和CCxNE,TIMx_BDTR中的MOE和OSSI以及OSSR位,TIMx_CR2中的OISx和OISxN位。特别地，当转换到IDLE状态（MOE降为0）时，死区时间被激活。  
同时置位CCxE和CCxNE位将插入死区时间，如果存在刹车电路还需要置位MOE。每个通道有一个10位的死区时间发生器，将参考信号OCxREF分成2路输出OCx和OCxN。如果OCx和OCxN设为高有效：  
- OCx信号和参考信号相同，只是OCx的上升沿相对于OCxREF的上升沿有一定的延时。  
- OCxN信号和参考信号相反，只是OCxN的上升沿相对于OCxREF的下降沿有一定的延时。  
如果延时大于OCx或OCxN有效的输出宽度，则不会产生相应的脉冲。  
下面的几张图显示了参考信号OCxREF和死区时间发生器2路输出信号间的关系。这些图例，假设CCxP=0,CCxNP=0,MOE=1,CCxE=1,CCxNE=1。  
![](https://i.imgur.com/5mZwoTu.png)  
每一个通道的死区延时时间是相同的，通过TIMx_BDTR寄存器的DTG[7:0]位设置。  
####重定向OCxREF到OCx或OCxN  
在输出模式（强制输出，输出比较或PWM输出），通过设置TIMx_CCER寄存器的CCxE和CCxNE位，OCxREF可以被重定向到OCx或OCxN。  
这个功能允许在某个输出上输出一个特殊的波形（比如PWM或静态的有效电平），在这个输出的互补输出保持无效电平期间。其他作用如，让2个输出同时处于无效电平，或2个输出输出有效电平并且带死区互补输出。  
注意：当只有OCxN使能（CCxE=0,CCxNE=1）时，它没有互补信号，并且一旦OCxREF是高电平它就立即变为有效电平（与CCxNP相反）。例如，如果CCxNP=0，则OCxN=OCxREF。另一方面，当OCx和OCxN都使能（CCxE=CCxNE=1）时，当OCxREF变为高电平时，OCx也变为有效电平；而OCxN相反，当OCxREF变为低电平时，OCxN变为有效电平。  
###使用刹车功能  
但使用刹车功能时，输出使能信号和无效电平可以根据相应的控制位（TIMx_BDTR中的MOE,OSSI和OSSR，TIMx_CR2中的OISx和OISxN）被修改。在任何情况下，OCx和OCxN输出不能在同一时刻都处于有效电平。  
刹车源BRK既可以是连接在BKIN引脚上的外部信号，也可以是下列内部源之一：  
- 内核LOCKUP输出  
- PVD输出  
- SRAM奇偶校验错误信号  
- 时钟安全系统（CSS）监测器产生的时钟失败事件  
- 比较器的输出  
系统复位后，刹车电路被禁止，MOE=0。可以设置TIMx_BDTR寄存器中的BKE=1，使能刹车功能。设置TIMx_BDTR中的BKR，可以选择刹车输入信号的极性。BKE和BKP可以同时修改。当写BKE和BKP时，需要延时一个APB时钟周期，写入的数据才生效。因此，在写操作后，需要等待一个APB时钟周期才能正确读出写入的位。  
