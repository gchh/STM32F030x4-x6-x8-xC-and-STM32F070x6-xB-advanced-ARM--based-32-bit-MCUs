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
由于MOE下降沿可能是异步的，在输出端的实际信号和同步控制位（在TIMx_BDTR中）之间插入有重新同步电路。这会导致在异步信号和同步信号间有一些的延时。特别地，如果在MOE=0写1，则在正确读取它之前，必须插入一个延时（空指令）。这是因为你写入异步信号而读取同步信号。  
当发生刹车时（刹车输入端出现选定的电平）：  
- MOE被异步清除，输出置于无效状态、空闲或复位状态（有OSSI选择）。这个特性在MCU振荡器关闭时依然有效。  
- 一旦MOE=0，每个输出通道输出电平由TIMx_CR2寄存器中的OISx位决定。如果OSSI=0，定时器释放使能输出（使能输出信号=0）；否则，使能输出信号保持高电平。  
- 当使用互补输出时：  
　- 输出首先被置于复位状态/无效状态（取决于极性）。这是异步操作，即使定时器没有时钟，依然有效。  
　- 如果定时器时钟仍然存在，死区时间发生器将被重新激活，在一段死区时间之后根据OISx和OISxN设置的电平驱动输出端口。即使在这种情况下，OCx和OCxN仍然不能被同时置为有效电平。注意因为重新同步MOE，死区时间比通常要长一点（大约是2个ck_tim时钟周期）。  
　- 如果OSSI=0，定时器释放使能输出；否则OSSI=1，使能输出保持，或一旦CCxE或CCxNE其中一个是1，使能输出变为高电平。  
- 置位刹车状态标志位（TIMx_SR中的BIF位）。如果TIMx_DIER中的BIE=1，则会产生中断。  
- 如果TIMx_BDTR中的AOE=1，则MOE会在下个更新事件UEV发生时自动置1。例如，这可以用来整形。否则如果AOE=0，则MOE保持为0，直到向它写1。这种情况，可以用于安全上；你可以将电源驱动的报警输出，温度传感器或其他任何安全器件，连接到刹车输入上。  
注意：刹车输入是电平有效的。因此，当刹车输入有效时，MOE不能被置位（自动或软件都不行）。同时，状态标志位BIF不能被清除。  
刹车由极性可编程的BRK输入和TIMx_BDTR中的BKE使能位产生。  
除了刹车输入和输出管理外，在刹车电路内部执行写保护以保护应用程序。它允许你冻结几个参数（死区时间长度，OCx/OCxN极性和禁止时的状态，OCxM，刹车使能和极性）的配置。可以通过配置TIMx_BDTR中的LOCK[1:0]选择3中保护等级中的一种。LOCK[1:0]只能在MCU复位后写入一次。  
![](https://i.imgur.com/xqvlb47.png)  
###用外部事件清除OCxREF信号  
当OCREF_CLR_INPUT是高电平（TIMx_CCMRx中的OCxCE=1），OCxREF将被置为低电平直到发生下一次更新事件UEV。该功能只能在输出比较模式和PWM模式使用，在强制模式不能使用。  
根据TIMx_SMCR中的OCCS位，OCREF_CLR_INPUT在OCREF_CLR输入和ETRF(ETR经过滤波器后输出的信号)两者中选择其一。  
如果选择ETRF，则ETR必须按如下配置：  
ETRF为高电平时，给定通道的OCxREF信号被拉低（TIMx_CCMRx中的OCxCE=1）。OCxREF保持低电平直到下次更新事件UEV发生。  
该功能只能在输出比较模式和PWM模式使用，在强制模式不能使用。  
例如，OCxREF信号可以连接到一个比较器的输出，用作电流处理。此种情况，ETR必须如下配置：  
- 外部触发预分频器必须关闭：TIMx_SMCR中的ETPS[1:0]=00。  
- 禁止外部时钟模式2：TIMx_SMCR中的ECE=0。  
- 外部触发极性ETP和外部触发滤波器ETF可以根据需要配置。  
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
	TIMx->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1PE 
                 | TIM_CCMR1_OC1CE; /* (4)*/
	TIMx->CCER |= TIM_CCER_CC1E; /* (5) */
	TIMx->BDTR |= TIM_BDTR_MOE; /* (6) */
	TIMx->SMCR |= TIM_SMCR_OCCS; /* (7) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (8) */
	TIMx->EGR |= TIM_EGR_UG; /* (9) */  
图78显示了对应不同的OCxCE，当ETRF输入变高时，OCxREF的动作。在这个例子中，定时器TIMx处于PWM模式。  
![](https://i.imgur.com/7i0rURX.png)  
注意：对于100%PWM的情况（CCRx>ARR），OCxREF会在下一次计数器溢出时被再次使能。  
###6步PWM生成  
当一个通道使用互补输出时，预装载位有OCxM,CCxE和CCxNE。当发生COM换相事件时，这些预装载位写入影子寄存器相对应的位中。因此，你可以预先编写下一步的配置，并在同一时刻改变所有通道的配置。软件设置TIMx_EGR中的COM=1，或硬件上TRGI的上升沿，可以产生COM事件。  
当COM事件发生时，TIMx_SR中的COMIF标志置位，如果TIMx_DIER中的COMIE=1，则会产生中断；或者TIMx_DIER中的COMDE=1，则会产生DMA请求。  
图79显示了，在3种不同配置下，当发生COM事件时，OCx和OCxN的变化。  
![](https://i.imgur.com/QDUEueg.png)  
###单个脉冲模式  
单脉冲模式OPM是前面众多模式的一个特例。此模式，计数器响应一个激励而启动，在一个可编程的延时之后，产生一个宽度可编程的脉冲。  
计数器启动可以通过从模式控制器控制。可以在输出比较模式或PWM模式下产生波形。通过设置TIMx_CR1中的OPM=1，选择单脉冲模式。这样，计数器将在下个更新事件UEV发生时，自动停止。  
只有当比较值和计数器初始值不同时，才能产生一个脉冲。在计数器启动前（等待触发时），配置必须满足：  
- 向上计数时：CNT<CCRx≤ARR（特别地，0<CCRx）。  
- 向下计数时：CNT>CCRx  
![](https://i.imgur.com/Gu0lWF9.png)  
例如，你需要，在TI2输入引脚上检测到一个上升沿时，经过t<sub>DELAY</sub>延时后，在OC1上输出一个长度t<sub>PULSE</sub>的正脉冲。  
假定使用TI2FP2作为触发1：  
- 设置TIMx_CCMR1中的CC2S[1:0]=01，将TI2FP2映射到TI2  
- 设置TIMx_CCER中的CC2P=0和CC2NP=0，使TI2FP2检测上升沿  
- 设置TIMx_SMCR中的TS[2:0]=110，配置TI2FP2作为从模式控制器的触发输入TRGI  
- 设置TIMx_SMCR中的SMS[2:0]=110，选择触发模式，TRGI的上升沿启动计数器计数；即TI2FP2的上升沿启动计数。  
OPM的波形由比较寄存器的值决定（要考虑时钟频率和计数器预分频器）：  
- TIMx_CCR1寄存器的值决定了t<sub>DELAY</sub>的长短。  
- t<sub>PULSE</sub>=(TIMx_ARR-TIMx_CCR+1)。  
- 假定你想要当CNT=TIMx_CCR时，波形从0变为1；当CNT=TIMx_ARR时，波形从1变为0。为此，你需要设置TIMx_CCMR1寄存器的OC1M[2:0]=111，选择PWM模式2。可以随意设置TIMx_CCMR1中的OC1PE=1或TIMx_CR1中的ARPE=1，使能预装载寄存器。向TIMx_CCR1写入比较值，向TIMx_ARR写入自动重载值，然后将TIMx_EGR中的UG=1产生一个更新事件，然后等待TI2上的外部触发事件。本例中，CC1P=0。  
在我们的举例中，TIMx_CR1中的DIR=0，CMS[1:0]=00。  
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
你只想要1个脉冲，所以TIMx_CR1的OPM需要置1，以在下个更新事件发生时（当计数器从自动装载值翻转到0时，即溢出）停止计数器。当TIMx_CR1中的OPM=0，选择重复模式。  
####特殊情况：OCx快速使能  
在单脉冲模式，TIx的边沿检测电路置位CEN启动计数器计数。计数器和比较值间的比较结果促使输出跳变。但是这些操作需要一些时钟周期去完成，所以这限制了我们能得到的最小t<sub>DELAY</sub>延时。  
如果你想以最小的延时输出波形，可以设置TIMx_CCMRx中的OCxFE=1。此时，OCxREF（以及OCx）被强制直接响应激励，而不考虑比较结果如何；输出的波形和比较匹配时输出的波形一样。OCxFE只有在通道配置成PWM模式1或PWM模式2时，才起作用。  
###编码器接口模式  
如果计数器只在TI2边沿计数，则设置TIMx_SMCR中的SMS[2:0]=001选择编码器接口模式。如果计数器只在TI1边沿计数，则设置TIMx_SMCR中的SMS[2:0]=010选择编码器接口模式。如果计数器在TI1和TI2的边沿都计数，则设置TIMx_SMCR中的SMS[2:0]=011选择编码器接口模式。  
设置TIMx_CCER寄存器中的CC1P和CC2P，选择TI1和TI2的极性。需要时，还可以配置输入滤波器。CC1NP和CC2NP必须保持为0。  
TI1和TI2两个输入作为增量编码器的接口。假设计数器使能（TIMx_CR1中的CEN=1），计数器在每次TI1FP1或TI2FP2有效跳变时工作（TI1FP1和TI2FP2是TI1和TI2经过输入滤波和极性选择后的信号，如果没有滤波或反相，则TI1FP1=TI1，TI2FP2=TI2）。根据2个输入的跳变顺序，产生计数脉冲和方向信号。根据向上的跳变次序或向下的跳变次序，硬件自动对TIMx_CR1中的DIR方向位进行设置。DIR方向位在任一输入端（TI1或TI2）的每次跳变时都会重新计算，不管计数器是依据TI1还是TI2或者同时依据2者计数。  
编码器接口简单来看就像使用了带方向选择的外部时钟。这意味着计数器根据方向，从0计数到TIMx_ARR中的自动装载值，或从TIMx_ARR中的自动装载值计数到0。因此，在开始计数前，要配置TIMx_ARR。同样，捕获器，比较器，预分频器，重复计数器，触发器输出特性依然如常工作。编码器模式和外部时钟模式2是不兼容的，因此不能同时选择这两种模式。  
在此模式，计数器跟随增量编码器的速度和方向自动修改，计数器的内容始终指示着编码器的位置。计数方向对应着连接的传感器旋转方向。下表总结了可能的组合，假定TI1和TI2不在同时跳变。  
![](https://i.imgur.com/Q57j4d7.png)  
一个外部增量编码器可以直接与MCU相连而不需要外部接口逻辑。但是，通常使用比较器将编码器的差动输出转换成数字信号，这将大大增加抗干扰的能力。编码器的第三个输出表示机械零点，可以把它连接到一个外部中断输入端口并触发一次计数器复位。  
图81是一个计数器操作的例子，显示了计数信号的产生和方向控制。同时，显示了当选择2个边沿时，输入抖动如何被抑制的；抖动可能会在传感器的位置靠近一个开关点时产生。对这个例子，我们假定配置如下：  
- CC1S=01（在TIMx_CCMR1中，TI1FP1映射到TI1）  
- CC2S=01（在TIMx_CCMR1中，TI2FP2映射到TI2）  
- CC1P=0（在TIMx_CCER中，TI1FP1不反相，TI1FP1=TI1）  
- CC2P=0（在TIMx_CCER中，TI2FP2不反相，TI2FP2=TI2）  
- SMS=011（在TIMx_SMCR中，2个输入在上升沿和下降沿2个边沿都有效）  
- CEN=1（在TIMx_CR1中，计数器使能）  
######Encoder interface code example  

	/* (1) Configure TI1FP1 on TI1 (CC1S = 01),
	       configure TI2FP2 on TI2 (CC2S = 01) */
	/* (2) Configure TI1FP1 and TI2FP2 non inverted (CC1P = CC2P = 0, reset value) */
	/* (3) Configure both inputs are active on both rising and falling edges (SMS = 011) */
	/* (4) Enable the counter by writing CEN=1 in the TIMx_CR1 register. */
	TIMx->CCMR1 |= TIM_CCMR1_CC1S_0 | TIM_CCMR1_CC2S_0; /* (1)*/
	TIMx->CCER &= (uint16_t)(~(TIM_CCER_CC1P | TIM_CCER_CC2P); /* (2) */
	TIMx->SMCR |= TIM_SMCR_SMS_0 | TIM_SMCR_SMS_1; /* (3) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (4) */  
![](https://i.imgur.com/WCgmiXz.png)  
图82是当TI1FP1反相时的一个计数器操作例子，除了CC1P=1外，其他配置同上个例子。  
当配置成编码器模式时，定时器指示传感器当前的位置。使用另一个定时器配置成捕获模式来测量2个编码器事件的间隔，据此，可以获得（速度，加速度，减速度）动态信息。指示机械零点的编码器输出可以用于此目的。根据2次编码事件间隔的时间，计数器可以按固定的时间读取。如果需要，你可以将计数器的值写入第三方输入捕获寄存器（由另一个定时器产生的捕获信号必须是周期性的）；也可以通过实时时钟产生的DMA请求读取计数器的值。  
###定时器输入异或功能  
TIMx_CR2中的TI1S，允许通道1的输入滤波器的输入连接到一个异或门的输出端。TIMx_CH1,TIMx_CH2和TIMx_CH3是异或门的3个输入脚。  
异或输出能够被用于所有定时器输入功能中，像触发或输入捕获。下面给出了一个此特性用于霍尔传感器接口的例子。  
###霍尔传感器接口  
在图83中，TIM1产生PWM信号驱动马达，另一个定时器TIM3被称为“接口定时器”。3个定时器输入引脚（CC1,CC2,CC3）通过一个异或门连接到TI1输入（通过TIMx_CR2中的TI1S位选择），“接口定时器”捕获此信号。  
从模式控制器配置为复位模式，从输入是TI1F_ED。因此，3个输入其中一个发生跳变，计数器就从0开始重新计数。这将由霍尔传感器输入端的任何变化触发产生一个时间基准。  
在“接口定时器”，捕获/比较通道1配置成捕获模式，捕获信号是TRC。捕获值反映了输入信号2次变化的间隔时间，给出了关于马达速度的信息。  
“接口定时器”可以在输出模式下产生一个脉冲；这个脉冲可以通过触发一个COM事件，来改变TIM1各通道的配置。TIM1用于产生驱动马达的PWM信号。为此，“接口定时器”通道必须配置成在一个可编程的延时之后（在输出比较或PWM模式），产生一个正脉冲；这个脉冲通过TRGO输出被发送给TIM1。  
例如：霍尔输入连接到TIMx定时器，每当霍尔输入发生改变时，经过一段可编程的延时后，改变TIM1的PWM配置。  
- 配置3个定时器输入异或到TI1输入通道：配置TIMx_CR2中的TI1S=1。  
- 编程时基：配置TIMx_ARR为最大值（计数器必须由TI1的变化清零）；设置预分频器来获得最大的计数周期，要比比传感器2个变化间的时间间隔更长。  
- 配置通道1为捕获模式（选择TRC）：TIMx_CCMR1中的CC1S=01；如果需要，可以设置数字滤波器。  
- 配置通道2为PWM模式2，带指定的延时：TIMx_CCMR1中OC2M=111和CC2S=00。  
- 选择OC2REF作为TRGO上的触发输出：TIMx_CR2中的MMS=101。  
在高级控制定时器TIM1中，必须选择正确的ITR输入作为触发输入，定时器被编程产生PWM信号，捕获/比较控制信号是预装载的（TIMx_CR2中的CCPC=1），并且由触发输入控制COM事件（TIMx_CR2中的CCUS=1）。PWM控制位（CCxE,OCxM）在COM事件后的下一步被写入（这可以在由OC2REF上升沿产生的中断的服务子程序来处理实现）。  
![](https://i.imgur.com/X5SLjGI.png)  
###TIMx和外部触发同步  
TIMx定时器可以在几种模式下同外部触发同步：复位模式，门控模式和触发模式。  
####从模式：复位模式  
当一个触发输入事件发生时，计数器和它的预分频器将被重新初始化。此外，如果TIMx_CR1中的URS=0，还将产生一个更新事件UEV；这样所有的预装载寄存器（TIMx_ARR,TIMx_CCRx）都会被更新。  
下面的例子中，TI1输入的上升沿导致计数器清零：  
- 配置通道1检测TI1上的上升沿。配置输入滤波器带宽（在本例中，不需要滤波器，所以保持IC1F=0000）。触发操作不需要使用捕获预分频器，所以不需要配置它。TIMx_CCMR1中的CC1S=01选择输入捕获源。TIMx_CCER中的CC1P=0和CC1NP=0，确定极性，只检测上升沿。  
- TIMx_SMCR中的SMS=100，配置计数器为复位模式。TIMx_SMCR中的TS=101，选择TI1作为输入源。  
- TIMx_CR1中的CEN=1，启动计数器。  
######Reset mode code example  

	/* (1) Configure channel 1 to detect rising edges on the TI1 input by writing CC1S = ‘01’,
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
计数器按照内部时钟开始正常计数，直到出现TI1上升沿，计数器被清零并且重新从0开始计数。同时，TIMx_SR中的TIF触发标志置位，并且产生中断或DMA请求（取决于TIMx_DIER中TIE和TDE的设置）。  
下图显示了当自动重载寄存器TIMx_ARR=0x36时的动作。TI1出现上升沿到实际计数器复位之间的延时取决于TI1输入上的重新同步电路。  
![](https://i.imgur.com/T9m9VZj.png)  
####从模式：门控模式  
计数器根据选择的输入电平启动计数。  
下面的例子中，计数器在TI1输入为低电平时向上计数：  
- 配置通道1检测TI1低电平。配置输入滤波器带宽（在本例中，不需要滤波器，所以保持IC1F=0000）。触发操作不需要捕获预分频器，所以不需要配置它。TIMx_CCMR1中的CC1S=01选择输入捕获源。TIMx_CCER中的CC1P=0和CC1NP=0选择极性，只检测低电平。  
- TIMx_SMCR中的SMS=101选择门控模式。TIMx_SMCR中的TS=101选择TI1作为输入源。  
- TIMx_CR1中的CEN=1，启动计数器（在门控模式，CEN=0时，不论输入电平高低，计数器都不会启动计数）。  
######Gated mode code example  

	/* (1) Configure channel 1 to detect low level on the TI1 input by writing CC1S = ‘01’,
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
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_0 | TIM_SMCR_TS_2 | TIM_SMCR_TS_0; /* (3) */
	TIMx->PSC = 11999; /* (4) */
	TIMx->CR1 |= TIM_CR1_CEN; /* (5) */  
当TI1是低电平时，计数器按照内部时钟开始计数；当TI1变为高电平时，计数器停止计数。TIMx_SR中的TIF标志当计数器开始或停止计数时都会被置位。  
从TI1上升沿到计数器实际停止这之间的延时，取决于TI1输入的重新同步电路。  
![](https://i.imgur.com/iV3rUWO.png)  
####从模式：触发模式  
选择的输入事件，启动计数器。  
在下面的例子中，计数器在TI2输入上升沿时开始向上计数：  
- 配置通道2检测TI2的上升沿。配置输入滤波器带宽（在本例中，不需要使用滤波器，所以保持IC2F=0000）。触发操作不需要捕获预分频器，所以不需要配置它。TIMx_CCMR1中的CC2S=01选择输入捕获源。TIMx_CCER中的CC2P=0和CC2NP=0，选择极性，只检测上升沿。  
- TIMx_SMCR中的SMS=110选择触发模式。TIMx_SMCR中的TS=110选择TI2作为输入源。  
######Trigger mode code example  

	/* (1) Configure channel 2 to detect rising edge on the TI2 input by writing CC2S = ‘01’,
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
当TI2上出现上升沿时，计数器开始按照内部时钟计数，并且TIF标志置位。  
从TI2出现上升沿到计数器实际开始基数，这之间的延时取决于TI2输入的重新同步电路。  
![](https://i.imgur.com/2npiiA9.png)  
####从模式：外部时钟模式2+触发模式  
外部时钟模式2可以和一种从模式一起使用（外部时钟模式1和编码器模式除外）。在这种模式，ETR信号作为外部时钟输入，其他的输入可以被选择作为触发器输入（复位模式，门控模式或触发模式）。不建议设置TIMx_SMCR中的TS位选择ETR作为TRGI。  
在下面的例子中，在TI1上出现上升沿后，计数器按照ETR每个上升沿向上计数一次：  
1. 设置TIMx_SMCR寄存器来配置外部触发输入电路，如下：  
　- ETF=0000：没有滤波  
　- ETPS=00：不用预分频器  
　- ETP=0：检测ETR上的上升沿  
　- ECE=1：使能外部时钟模式2  
2. 配置通道1，检测TI1的上升沿，如下：  
　- IC1F=0000：没有滤波  
　- 触发操作不使用捕获预分频器，不需要配置它  
　- TIMx_CCMR1中CC1S=01，选择输入捕获源TI1  
　- TIMx_CCER中CC1P=0和CC1NP=0，确定极性，只检测上升沿  
3. TIMx_SMCR中的SMS=110，配置定时器为触发模式。TIMx_SMCR中的TS=101，选择TI1作为输入源  
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
	TIMx->SMCR |= TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1 | TIM_SMCR_TS_2 | TIM_SMCR_TS_0; /* (4) */
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
	TIMx->SMCR |= TIM_SMCR_TS_2 | TIM_SMCR_TS_1 | TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1; /* (3) */  
TI1的上升沿使能计数器，并且TIF标志置位；然后，计数器在每个ETR上升沿计数。  
从ETR上升沿到计数器实际开始计数，这之间的延时取决于ETRP输入端的重新同步电路。  
![](https://i.imgur.com/P73qjpe.png)  
###定时器同步  
TIM定时器在内部是相连的，用于定时器同步或链接。参考General-purpose timer(TIM3)中的Timer synchronization章节，了解详情。  
###调试模式  
当微控制器进入调试模式（Cortex-M0内核停止），根据DBG模块中的DBG_TIMx_STOP配置，TIMx计数器可以继续正常工作或停止。  
##TIM1寄存器  
###TIM1控制寄存器1（TIM1_CR1）  
![](https://i.imgur.com/o4uHyAK.png)  
![](https://i.imgur.com/phu5Eox.png)  
![](https://i.imgur.com/BCn5f1F.png)  
###TIM1控制寄存器2（TIM1_CR2）  
![](https://i.imgur.com/yvwkxoh.png)  
![](https://i.imgur.com/qkV6jy3.png)  
![](https://i.imgur.com/LybN2MQ.png)  
###TIM1从模式控制寄存器（TIM1_SMCR）  
![](https://i.imgur.com/TNKVAxI.png)  
![](https://i.imgur.com/Z0wLBM6.png)  
![](https://i.imgur.com/fylVuGw.png)  
![](https://i.imgur.com/1Kj1LLp.png)  
![](https://i.imgur.com/IKLskSx.png)  
###TIM1 DMA/中断使能寄存器（TIM1_DIER）  
![](https://i.imgur.com/Fyo5wAd.png)  
![](https://i.imgur.com/j9SKFeT.png)  
![](https://i.imgur.com/RQdU03Z.png)  
###TIM1 状态寄存器（TIM1_SR）  
![](https://i.imgur.com/hU51Nzr.png)  
![](https://i.imgur.com/ECZ9DwF.png)  
![](https://i.imgur.com/4alzxG1.png)  
###TIM1事件发生寄存器（TIM1_EGR）  
![](https://i.imgur.com/OjobiAc.png)  
![](https://i.imgur.com/mOdTrHW.png)  
![](https://i.imgur.com/g5MPKS3.png)  
###TIM1捕获/比较模式寄存器1（TIM1_CCMR1）  
![](https://i.imgur.com/nXTMKSu.png)  
![](https://i.imgur.com/LXrhvC7.png)  
![](https://i.imgur.com/EkoT2ET.png)  
![](https://i.imgur.com/uyXsJIK.png)  
![](https://i.imgur.com/RWXFVHN.png)  
![](https://i.imgur.com/gQGsgnl.png)  
![](https://i.imgur.com/ash6JIg.png)  
###TIM1捕获/比较模式寄存器2（TIM1_CCMR2）  
![](https://i.imgur.com/L2xD9aE.png)  
![](https://i.imgur.com/ppgPmxF.png)  
![](https://i.imgur.com/me7IXkp.png)  
![](https://i.imgur.com/IBFUOKy.png)  
###TIM1捕获/比较使能寄存器（TIM1_CCER）  
![](https://i.imgur.com/UuA57GC.png)  
![](https://i.imgur.com/oqkLnGn.png)  
![](https://i.imgur.com/CvR4NyR.png)  
![](https://i.imgur.com/jUBULHw.png)  
![](https://i.imgur.com/RKWIxZp.png)  
![](https://i.imgur.com/SB5UPiM.png)  
###TIM1计数器（TIM1_CNT）  
![](https://i.imgur.com/Oex8jNc.png)  
###TIM1预分频器（TIM1_PSC）  
![](https://i.imgur.com/o0nXCgU.png)  
![](https://i.imgur.com/T0fwT63.png)  
###TIM1自动重装载寄存器（TIM1_ARR）  
![](https://i.imgur.com/s3UmnBv.png)  
###TIM1重复计数寄存器（TIM1_RCR）  
![](https://i.imgur.com/rhnwFAV.png)  
###TIM1捕获/比较寄存器1（TIM1_CCR1）  
![](https://i.imgur.com/eAGizLk.png)  
![](https://i.imgur.com/UwKC5Y2.png)  
###TIM1捕获/比较寄存器2（TIM1_CCR2）  
![](https://i.imgur.com/Zc9dmem.png)  
###TIM1捕获/比较寄存器3（TIM1_CCR3）  
![](https://i.imgur.com/aRUv0Ha.png)  
###TIM1捕获/比较寄存器4（TIM1_CCR4）  
![](https://i.imgur.com/3L1VIdU.png)  
###TIM1刹车和死区时间寄存器（TIM1_BDTR）  
![](https://i.imgur.com/pn3mmFr.png)  
![](https://i.imgur.com/a1yJfMm.png)  
![](https://i.imgur.com/0ZRUA2T.png)  
![](https://i.imgur.com/1Usctov.png)  
