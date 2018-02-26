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
