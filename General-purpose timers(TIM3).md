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
每个捕获/比较通道都是建立在一个捕获/比较寄存器（包含一个影子寄存器），一个捕获输入级（带有数字滤波器，多路复用和预分频器），以及一个比较输出级（带有比较器和输出控制）。  
下面的图给出了一个捕获/比较通道的概览。  
输入级对相应的TIx输入进行采样然后产生滤波后的信号TIxF。然后，通过一个带极性选择的边沿检测器后产生信号TIxFPx，它可以作为从模式控制器的触发输入，或者作为捕获控制。TIxFPx经过预分频器产生信号ICxPS，传输到捕获寄存器。  
![](https://i.imgur.com/905L1JK.png)  
