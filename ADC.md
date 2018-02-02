#Analog-to-digital converter(ADC)  
##介绍
12位逐次逼近型模数转换器，有多达19个可复用通道，可测量16个外部和2个内部信号源。每个通道的A/D转换可以按单次，连续，扫描或间断模式执行。ADC转换结果可以左对齐或右对齐存储在16位数据寄存器。  
模拟看门狗允许应用检测输入电压是否超出用户设定的高/低阈值。  
一个有效的低功耗模式的实施，在低频率下实现低功耗。  
##ADC主要特性  
- 高性能  
　- 12位，10位，8位，6位，可配置分辨率  
　- ADC转换时间：12位分辨率-1.0us(1MHz)，10位分辨率-0.93us，分辨率越低转换时间越少。  
　- 自校准  
　- 可编程的采样时间  
　- 具有内置数据一致性的数据对齐  
　- 支持DMA  
- 低功耗  
　- 应用可以降低PCLK频率减少功耗，同时保持最佳的ADC性能。例如，无论PCLK频率是多少，都保持1us的转换时间。  
　- 等待模式：在PCLK低频率时，防止ADC溢出  
　- 自动关闭模式：ADC只在转换期间工作，其他时间自动断电。这将大大减少ADC的功耗。  
- 模拟输入通道  
　- 16个外部模拟输入  
　- 1个内部温度传感器（V<sub>SENSE</sub>）输入  
　- 1个内部参考电压（V<sub>REFINT</sub>）输入  
- 多种启动转换的方式：  
　- 软件启动转换  
　- 由可配置极性的硬件触发（TIM1,TIM3和TIM15的内部定时器事件），启动转换  
- 转换模式  
　- 可以转换一个通道或一系列通道  
　- 单次模式，每一次触发，转换一次  
　- 连续模式，连续不断的转换  
　- 间断模式  
- 采样结束，转换结束，一系列转换结束，模拟看门狗或溢出事件，都可以产生中断  
- 模拟看门狗  
- ADC供电要求：2.4V-3.6V  
- ADC输入范围：V<sub>SSA</sub> ≤ V<sub>IN</sub> ≤ V<sub>DDA</sub>  
##ADC引脚和内部信号  
![](https://i.imgur.com/p3wTrwt.png)  
##ADC功能描述  
![](https://i.imgur.com/7mVPRP2.png)  
###自校准ADCAL  
ADC有校准功能，在校准期间，ADC计算一个用于ADC的校准因子直到ADC断电。应用不能在ADC校准期间使用ADC，必须等校准结束。  
校准需要在开始A/D转换前进行，以消除制造芯片过程中造成的偏移误差。  
校准只能在ADC关闭时（ADEN=0），由软件设置ADCAL=1启动。ADCAL会一直为1，直到校准完成，由硬件清零。校准完成后，可以从ADC_DR寄存器（bits 6:0）读出校准因子。  
校准完成后，即使关闭ADC（ADEN=0），校准因子依然保持。当ADC工作条件改变（V<sub>DDA</sub>是造成ADC偏移误差的主要因素，其次是温度），建议在启动ADC前，重新校准一次。  
校准因子在ADC断电（例如，进入待机模式）后会丢失。  
校准的软件流程：  
1. 确保ADEN=0，DMAEN=0  
2. 设置ADCAL=1  
3. 等待ADCAL=0  
4. 可以从ADC_DR中读取校准因子  
######ADC Calibration code example  

	/* (1) Ensure that ADEN = 0 */
	/* (2) Clear ADEN by setting ADDIS*/
	/* (3) Clear DMAEN */
	/* (4) Launch the calibration by setting ADCAL */
	/* (5) Wait until ADCAL=0 */
	if ((ADC1->CR & ADC_CR_ADEN) != 0) /* (1) */
	{
		ADC1->CR |= ADC_CR_ADDIS; /* (2) */
	}
	while ((ADC1->CR & ADC_CR_ADEN) != 0)
	{
		/* For robust implementation, add here time-out management */
	}
	ADC1->CFGR1 &= ~ADC_CFGR1_DMAEN; /* (3) */
	ADC1->CR |= ADC_CR_ADCAL; /* (4) */
	while ((ADC1->CR & ADC_CR_ADCAL) != 0) /* (5) */
	{
		/* For robust implementation, add here time-out management */
	}  
![](https://i.imgur.com/t4g8n4p.png)  
###ADC开/关控制（ADEN,ADDIS,ADRDY）  
ADC默认关闭（ADEN=0），并处于断电状态。  
在ADC正确转换前，需要一个稳定时间t<sub>STAB</sub>。  
2个控制位用于打开或关闭ADC：  
- ADEN=1打开ADC。当ADC稳定后，ADRDY=1。  
- ADDIS=1关闭ADC，并使ADC断电。当ADC完全关闭，硬件自动清除ADEN和ADDIS。  
ADC转换可以由设置ADSTART=1启动或外部触发事件启动（若触发启动打开）。  
打开ADC的步骤：  
1. 清除ADC_ISR的ADRDY位（向ADRDY位写1）  
2. 设置ADC_CR的ADEN=1  
3. 等待ADC_ISR的ADRDY=1。若ADC_IER中的ADC准备好中断使能位ADRDYIE=1，当ADC准备好时，会产生中断。  
######ADC enable sequence code example  

	/* (1) Ensure that ADRDY = 0 */
	/* (2) Clear ADRDY */
	/* (3) Enable the ADC */
	/* (4) Wait until ADC ready */
	if ((ADC1->ISR & ADC_ISR_ADRDY) != 0) /* (1) */
	{
		ADC1->ISR |= ADC_CR_ADRDY; /* (2) */
	}
	ADC1->CR |= ADC_CR_ADEN; /* (3) */
	while ((ADC1->ISR & ADC_ISR_ADRDY) == 0) /* (4) */
	{
		/* For robust implementation, add here time-out management */
	}  
关闭ADC的步骤：  
1. 检查ADC_CR中的ADSTART=0，确保没有转换正在进行。如果需要，向ADC_CR的ADSTP位写1，停止正在进行的转换，并等待ADSTP=0  
2. 设置ADC_CR的ADDIS=1  
3. 若应用需要，可以等待ADC_CR的ADEN=0，表明ADC完全关闭（当ADEN=0时，ADDIS自动清零）  
4. 向ADC_ISR的ADRDY位写1，将其清零  
######ADC disable sequence code example  

	/* (1) Stop any ongoing conversion */
	/* (2) Wait until ADSTP is reset by hardware i.e. conversion is stopped */
	/* (3) Disable the ADC */
	/* (4) Wait until the ADC is fully disabled */
	ADC1->CR |= ADC_CR_ADSTP; /* (1) */
	while ((ADC1->CR & ADC_CR_ADSTP) != 0) /* (2) */
	{
		/* For robust implementation, add here time-out management */
	}
	ADC1->CR |= ADC_CR_ADDIS; /* (3) */
	while ((ADC1->CR & ADC_CR_ADEN) != 0) /* (4) */
	{
		/* For robust implementation, add here time-out management */
	}  
警告：在ADCAL=1和ADCAL被硬件清零（校准结束）后的4个ADC时钟周期内，ADEN不能被置1。  
![](https://i.imgur.com/m5pvHuZ.png)  
###ADC时钟（CKMODE）  
ADC有双时钟架构，因此ADC可以由独立于APB时钟（PCLK）的ADC异步时钟（HSI14）提供时钟。  
![](https://i.imgur.com/7aHhbQK.png)  
ADC的时钟能够在2个不同的时钟源PCLK和ADC异步时钟间选择：  
a）ADC有一个专用的时钟源，称为ADC异步时钟（HSI14），是独立于APB时钟的。参考RCC部分了解此时钟源的信息。  
选择此时钟源，ADC_CFGR2寄存器的CKMODE[1:0]位必须被清零。  
######ADC Clock selection code example  

	/* This code selects the HSI14 as clock source. */
	/* (1) Enable the peripheral clock of the ADC */
	/* (2) Start HSI14 RC oscillator */
	/* (3) Wait HSI14 is ready */
	/* (4) Select HSI14 by writing 00 in CKMODE (reset value) */
	RCC->APB2ENR |= RCC_APB2ENR_ADC1EN; /* (1) */
	RCC->CR2 |= RCC_CR2_HSI14ON; /* (2) */
	while ((RCC->CR2 & RCC_CR2_HSI14RDY) == 0) /* (3) */
	{
		/* For robust implementation, add here time-out management */
	}
	//ADC1->CFGR2 &= (~ADC_CFGR2_CKMODE); /* (4) */  
b）ADC时钟也可以由ADC总线接口的APB时钟分频（/2或/4）得到。使用此时钟源，ADC_CFGR2的CKMODE[1:0]必须不是"00"。  
选项a)的优势在于有最大的ADC时钟频率。  
选项b)的优势在于可以绕开时钟域的重新同步。这是有用的，当ADC由定时器触发启动，并且应用要求没有任何不确定性的精确触发时（否则，2个时钟域间的重新同步，会产生不确定的触发瞬间）。  
![](https://i.imgur.com/2WUvQj2.png)  
###配置ADC  
在ADC关闭（ADEN=0）情况下，软件写ADC_CR的ADCAL和ADEN位。  
在ADC打开并且没有处在关闭过程中（ADEN=1,ADDIS=0）时，软件写ADC_CR的ADSTART和ADDIS位。  
对于在ADC_IER,ADC_CFGRi,ADC_SMPR,ADC_TR,ADC_CHSELR和ADC_CCR寄存器中的控制位，必须在ADC打开（ADEN=1）并且没有转换正在进行（ADSTART=0）情况下，软件才能写。  
ADC_CR中的ADSTP位，必须在ADC打开（可能正在转换）并且没有处在关闭过程中（ADEN=1,ADDIS=0,ADSTART可能为1）的情况下，才能软件写。  
注意：没有硬件保护阻止软件违反上述规则进行写操作。如果违反写规则，ADC可能会进入一种不确定状态。要从此种情况恢复，必须关闭ADC（清除ADC_CR中所有位）。  
###通道选择（CHSEL,SCANDIR）  
有多达18路的可复用通道：  
- 16个模拟输入引脚（ADC_IN0...ADC_IN15）  
- 2个内部模拟输入（温度传感器，内部参考电压）  
可以转换一个通道或自动扫描一系列通道。  
转换的通道需要在ADC_CHSELR通道选择寄存器设置：每一个模拟输入通道都有一个专用的选择位（CHSEL0...CHSEL18）。  
这些通道的扫描次序可以设置ADC_CFGR1寄存器的SCANDIR位配置：  
- SCANDIR=0：扫描次序从通道0到通道18  
- SCANDIR=1：扫描次序从通道18到通道0  
####温度传感器，V<sub>REFINT</sub>内部通道  
温度传感器连接到通道ADC_IN16。内部参考电压V<sub>REFINT</sub>连接到通道ADC_IN17。  
###可编程的采样时间  
在开始转换前，ADC需要在被测电压源和内置采样电容间建立直接的连接。采样时间必须足够长以便输入电压源对内嵌电容充电到输入电压的水平。  
采用可编程的采样时间可以根据输入电压的输入电阻调整转换的速度。  
ADC采样输入电压需要的ADC时钟周期数可以通过ADC_SMPR寄存器的SMP[2:0]位进行修改。  
可编程的采样时间对所有通道都通用。如果应用需要，软件可以在每次转换间改变和调整采样时间。  
总换算时间计算如下：  
t<sub>CONV</sub>=采样时间 + 12.5 x ADC时钟周期  
例如：当ADC_CLK=14MHz，采样时间是1.5个ADC时钟周期，那么转换总时间是：t<sub>CONV</sub>=1.5+12.5=14个ADC时钟周期=1us  
ADC设置EOSMP标志表示采样结束。  
###单次转换模式（CONT=0）  
在单词转换模式，ADC执行一次序列转换，转换所有选中的通道。将ADC_CFGR1寄存器的CONT设为0，选择此模式。  
转换可以由以下2种方法启动：  
- 将ADC_CR寄存器的ADSTART位置1  
- 硬件触发事件   
在序列转换中，每次转换结束：  
- 转换结果存入16位ADC_DR寄存器  
- 转换结束标志位EOC置位  
- 如果EOCIE=1，将产生中断  
在序列转换结束后：  
- 序列转换结束标志位EOSEQ置位  
- 如果EOSEQIE=1，将产生中断  
然后，ADC将停止，直到有新的外部触发事件或ADSTART再次置1。  
注意：如果只转换一个通道，将转换序列长度设位1。  
###连续转换模式（CONT=1）  
在连续转换模式，当软件或硬件触发事件发生，ADC执行一个序列转换，转换所有选中通道一次，然后自动重新执行相同的序列转换。将ADC_CFGR1的CONT位置1，选择此模式。有以下2种方法启动转换：  
- 将ADC_CR的ADSTART位置1  
- 硬件触发事件  
在序列转换中，每次转换结束：  
- 转换结果存入16位ADC_DR寄存器  
- 转换结束标志位EOC置位  
- 如果EOCIE=1，将产生中断  
在序列转换结束后：  
- 序列转换结束标志位EOSEQ置位  
- 如果EOSEQIE=1，将产生中断  
然后，马上重新开始序列转换，ADC会不停的重复序列转换。  
注意：如果只转换一个通道，将转换序列长度设为1。  
ADC不能即是连续模式又是间断模式：即禁止同时置位DISCEN和CONT。  
###开始转换（ADSTART）  
软件启动ADC转换，通过设置ADSTART=1。  
当ADSTART=1时，转换：  
- 如果EXTEN=00(软件触发)，立即开始  
- 如果EXTEN≠00，在下一个所选硬件触发的有效边沿，开始  
ADSTART也用来指示ADC转换是否正在进行。当ADC空闲（ADSTART=0）时，可以重新配置ADC。  
ADSTART由硬件清零：  
- 软件触发（EXTEN=0）的单次模式(CONT=0)：在序列转换结束（EOSEQ=1）后  
- 软件触发（EXTEN=0）的间断模式（CONT=0,DISCEN=1）：在转换结束（EOC=1）后  
- 在任何情况下，软件调用并执行ADSTP后  
注意：在连续模式（CONT=1），当EOSEQ=1时，ADSTART不会被硬件清零，因为序列转换会自动重新开始。  
选择硬件触发的单词模式（EXTEN=01,CONT=0）时，当EOSEQ=1时，ADSTART不会被硬件清零。这是为了避免需要软件设置ADSTART，以确保不会错过下面的触发事件。  
###时序  
转换开始到结束的时间=设置的采样时间+逐次逼近时间（与转换精度有关）：  
t<sub>ADC</sub> = t<sub>SMPL</sub> + t<sub>SAR</sub> = [1.5<sub>|min</sub> + 12.5<sub>|12bit</sub>] x t<sub>ADC_CLK</sub>  
t<sub>ADC</sub> = t<sub>SMPL</sub> + t<sub>SAR</sub> = 107.1 ns<sub>|min</sub> + 892.8 ns<sub>|12bit</sub> = 1 μs<sub>|min</sub>(for f<sub>ADC_CLK</sub> = 14 MHz)  
![](https://i.imgur.com/9xZmeaG.png)  
![](https://i.imgur.com/4t8pxel.png)  
###停止正在进行的转换（ADSTP）   
软件通过设置ADC_CR寄存器的ADSTP=1，可以停止任何正在进行的转换。这会复位ADC的操作，ADC进入空闲，为新的转换准备。  
当ADSTP置位，任何正在进行的转换被停止，结果被丢弃（当前的转换值不会更新到ADC_DR中）。扫描序列也被停止并复位，这意味着重启ADC将会重新开始新的序列转换。  
一旦停止过程完成，ADSTP和ADSTART将由硬件清零，并且，软件必须等待ADSTART=0后才能开启新的转换。  
![](https://i.imgur.com/gPfAk7j.png)  
##外部触发转换和触发极性（EXTSEL,EXTEN）  
一次转换或一个序列转换可由软件或外部事件（比如定时器捕捉）触发开始。如果EXTEN[1:0]!="0b00"，并且ADSTART=1，则外部事件所选的极性会触发一次转换。  
转换进行期间或ADSTART=0，发生的任何硬件触发都会被忽略。  
![](https://i.imgur.com/yyzdZ6f.png)  
注意：外部触发极性只能在ADC不在转换时（ADSTART=0）改变。  
EXTSEL[2:0]用于在8个外部触发事件间选择。  
![](https://i.imgur.com/Emtkjtg.png)  
软件触发源事件由置位ADC_CR的ADSTART位产生。  
###间断模式（DISCEN）  
这种模式可以由ADC_CFGR1寄存器的DISCEN位置位来设置。  
在间断模式（DISCEN=1），需要硬件或软件触发事件启动序列中的每一次转换。相对的，如果DISCEN=0，只需要一次硬件或软件触发事件就可以启动整个序列的所有转换。  
例如：  
- DISCEN=1，要转换的通道是0,3,7,10  
　- 第一个触发：转换通道0，产生EOC事件  
　- 第二个触发：转换通道3，产生EOC事件  
　- 第三个触发：转换通道7，产生EOC事件  
　- 第四个触发：转换通道10，产生EOC事件和EOSEQ事件  
　- 第五个触发：转换通道0，产生EOC事件  
　- 第六个触发：转换通道3，产生EOC事件  
　- ...  
-  DISCEN=0，转换的通道是0,3,7,10  
　-  第一个触发：整个序列被转换：通道0,3,7,10。每次转换会产生一次EOC事件，最后一次会同时产生EOSEQ事件。  
　-  后面任何触发都会重新开始整个序列的转换，通道0,3,7,10  
注意：ADC不能同时设置为间断和连续模式：禁止同时设置DISCEN=1和CONT=1。  
###可编程分辨率-快速转换模式  
降低ADC转换分辨率可以获得更快的转换时间。  
通过ADC_CFGR1寄存器的RES[1:0]位设置选择12,10,8或6位分辨率。如果应用不要求高精度，可以降低精度来加快转换时间。  
注意：当ADEN=0时，才能改变RES[1:0]。  
不管精度选择多少，转换结果都是12位的，未使用的低位补零。  
低精度减少逐次逼近步骤所需的转换时间，如下表所示：  
![](https://i.imgur.com/QB7TF9E.png)  
###转换结束，采样结束（EOC和EOSMP）  
转换结束标志EOC指示ADC转换结束。  
当新的转换数据存入ADC_DR中可被读取，ADC_ISR寄存器的EOC位马上被置位。如果ADC_IER的EOCIE=1，则转换结束会产生中断。通过读ADC_DR，或向EOC写1，来清零EOC。  
ADC_ISR中的EOSMP=1表示ADC采样结束。向EOSMP写1，来清零EOSMP。如果ADC_IER的EOSMPIE=1，则采样结束会产生中断。  
此中断的目的是允许处理与转换同步。通常，模拟多路复用器可以在转换阶段的隐蔽时间访问，以便在下一次采样开始时多路复用器被定位。  
注意：在采样结束和转换结束之间只有很短的时间，强烈建议使用轮询或WFE指令，而不是中断和WFI指令。  
###序列转换结束（EOSEQ）  
EOSEQ=1表示一个完整的序列转换结束。  
序列最后转换的结果存入ADC_DR时，ADC_ISR的EOSEQ置位。如果ADC_IER的EOSEQIE=1，则还会产生中断。通过向EOSEQ写1，将其清零。  
###时序图示例（单次/连续模式，硬件/软件触发）  
![](https://i.imgur.com/eoKC4wi.png)  
######Single conversion sequence code example - Software trigger  

	/* (1) Select HSI14 by writing 00 in CKMODE (reset value) */
	/* (2) Select CHSEL0, CHSEL9, CHSEL10 andCHSEL17 for VRefInt */
	/* (3) Select a sampling mode of 111 i.e. 239.5 ADC clk to be greater than 17.1us */
	/* (4) Wake-up the VREFINT (only for VBAT, Temp sensor and VRefInt) */
	//ADC1->CFGR2 &= ~ADC_CFGR2_CKMODE; /* (1) */
	ADC1->CHSELR = ADC_CHSELR_CHSEL0 | ADC_CHSELR_CHSEL9
				 | ADC_CHSELR_CHSEL10 | ADC_CHSELR_CHSEL17; /* (2) */
	ADC1->SMPR |= ADC_SMPR_SMP_0 | ADC_SMPR_SMP_1 | ADC_SMPR_SMP_2; /* (3) */
	ADC->CCR |= ADC_CCR_VREFEN; /* (4) */
	while (1)
	{
		/* Performs the AD conversion */
		ADC1->CR |= ADC_CR_ADSTART; /* Start the ADC conversion */
		for (i=0; i < 4; i++)
		{
			while ((ADC1->ISR & ADC_ISR_EOC) == 0) /* Wait end of conversion */
			{
				/* For robust implementation, add here time-out management */
			}
			ADC_Result[i] = ADC1->DR; /* Store the ADC conversion result */
		}
		ADC1->CFGR1 ^= ADC_CFGR1_SCANDIR; /* Toggle the scan direction */
	}  
![](https://i.imgur.com/1JvOcQb.png)  
######Continuous conversion sequence code example - Software trigger  

	/* This code example configures the AD conversion in continuous mode and in
	   backward scan. It also enable the interrupts. */
	/* (1) Select HSI14 by writing 00 in CKMODE (reset value) */
	/* (2) Select the continuous mode and scanning direction */
	/* (3) Select CHSEL1, CHSEL9, CHSEL10 and CHSEL17 */
	/* (4) Select a sampling mode of 111 i.e. 239.5 ADC clk to be greater than 17.1us */
	/* (5) Enable interrupts on EOC, EOSEQ and overrrun */
	/* (6) Wake-up the VREFINT (only for VBAT, Temp sensor and VRefInt) */
	//ADC1->CFGR2 &= ~ADC_CFGR2_CKMODE; /* (1) */
	ADC1->CFGR1 |= ADC_CFGR1_CONT | ADC_CFGR1_SCANDIR; /* (2) */
	ADC1->CHSELR = ADC_CHSELR_CHSEL1 | ADC_CHSELR_CHSEL9
				 | ADC_CHSELR_CHSEL10 | ADC_CHSELR_CHSEL17; /* (3) */
	ADC1->SMPR |= ADC_SMPR_SMP_0 | ADC_SMPR_SMP_1 | ADC_SMPR_SMP_2; /* (4) */
	ADC1->IER = ADC_IER_EOCIE | ADC_IER_EOSEQIE | ADC_IER_OVRIE; /* (5) */
	ADC->CCR |= ADC_CCR_VREFEN; /* (6) */
	/* Configure NVIC for ADC */
	/* (7) Enable Interrupt on ADC */
	/* (8) Set priority for ADC */
	NVIC_EnableIRQ(ADC1_COMP_IRQn); /* (7) */
	NVIC_SetPriority(ADC1_COMP_IRQn,0); /* (8) */  
![](https://i.imgur.com/HGICBT6.png)  
######Single conversion sequence code example - Hardware trigger  

	/* Configure the ADC, the ADC and its clock having previously been enabled. */
	/* (1) Select HSI14 by writing 00 in CKMODE (reset value) */
	/* (2) Select the external trigger on rasing edge and external trigger on TIM15_TRGO */
	/* (3) Select CHSEL0, 1, 2 and 3 */
	//ADC1->CFGR2 &= ~ADC_CFGR2_CKMODE; /* (1) */
	ADC1->CFGR1 |= ADC_CFGR1_EXTEN_0 | ADC_CFGR1_EXTSEL_2; /* (2) */
	ADC1->CHSELR = ADC_CHSELR_CHSEL0 | ADC_CHSELR_CHSEL1
	             | ADC_CHSELR_CHSEL2 | ADC_CHSELR_CHSEL3; /* (3) */   
![](https://i.imgur.com/PBsa0X5.png)  
######Continuous conversion sequence code example - Hardware trigger  

	/* (1) Select HSI14 by writing 00 in CKMODE (reset value) */
	/* (2) Select the external trigger on TIM15_TRGO (EXTSEL = 100),falling
	       edge (EXTEN = 10), the continuous mode (CONT = 1)*/
	/* (3) Select CHSEL0/1/2/3 */
	/* (4) Enable interrupts on EOC, EOSEQ and overrrun */
	//ADC1->CFGR2 &= ~ADC_CFGR2_CKMODE; /* (1) */
	ADC1->CFGR1 |= ADC_CFGR1_EXTEN_1 | ADC_CFGR1_EXTSEL_2
	             | ADC_CFGR1_CONT; /* (2) */
	ADC1->CHSELR = ADC_CHSELR_CHSEL0 | ADC_CHSELR_CHSEL1
	             | ADC_CHSELR_CHSEL2 | ADC_CHSELR_CHSEL3; /* (3)*/
	ADC1->IER = ADC_IER_EOCIE | ADC_IER_EOSEQIE | ADC_IER_OVRIE; /* (4) */
	/* Configure NVIC for ADC */
	/* (1) Enable Interrupt on ADC */
	/* (2) Set priority for ADC */
	NVIC_EnableIRQ(ADC1_COMP_IRQn); /* (1) */
	NVIC_SetPriority(ADC1_COMP_IRQn,0); /* (2) */  
##数据管理  
###数据寄存器和数据对齐（ADC_DR,ALIGN）  
当转换结束（EOC=1），转换结果被存放到16位宽的ADC_DR数据寄存器中。  
ADC_DR的格式取决于所配置的数据对齐方式和精度。  
ADC_CFGR1的ALIGN位用于选择数据对齐方式。ALIGN=0数据右对齐，ALIGN=1数据左对齐。  
![](https://i.imgur.com/90gtiM2.png)  
###ADC溢出  
当转换的数据未被CPU或DMA及时读取，而新的转换数据已经有效时，就发生溢出，OVR置位。  
若EOC=1时，新的转换已经完成，则ADC_ISR中的OVR会被置位。如果ADC_IER的OVRIE=1，则产生中断。  
当溢出发生，ADC会继续转换，除非软件设置ADC_CR的ADSTP=1停止转换。  
通过向OVR写1，将其清零。  
当发生溢出时，根据ADC_CFGR1的OVRMOD位的设置，决定ADC_DR中的数据是保持还是被覆盖：  
- OVRMOD=0：ADC_DR中的数据保持旧值，新的转换结果丢弃。如果OVR仍然为1，后续的转换会被执行但是转换结果会被丢弃。  
- OVRMOD=1：ADC_DR会被写入新的转换结果，而前次未被读出的结果丢弃。如果OVR依然为1，后续的转换会继续执行，而ADC_DR总是写入最新得到的转换结果。  
![](https://i.imgur.com/DPdOjuV.png)  
###不使用DMA管理序列转换的结果  
如果ADC转换足够慢，则转换序列可由软件处理。在这种情况下，软件需要利用ECO及其相关的中断去处理每一次的转换结果。每次转换结束ADC_ISR的EOC位置位，可以读取ADC_DR里的转换结果。ADC_CFGR1的OVRMOD可以设置为0，将溢出作为一个错误处理。  
###不使用DMA不考虑溢出管理转换数据  
ADC转换一个或多个通道且不用每次转换完都读取结果；这种情况下，OVRMOD必须配置为1，并且软件要忽略OVR标志。当OVRMOD=1，发生溢出不会阻止ADC继续转换，并且ADC_DR中的值总是最新的转换结果。  
###使用DMA管理转换数据  
因为所有通道的转换结果都要存放到一个数据寄存器中，所以当转换通道超过1个时，使用DMA会更高效。这样可以避免存放到ADC_DR寄存器时丢失转换结果。  
当DMA模式使能（ADC_CFGR1寄存器的DMAEN置1）时，每个通道转换结束会产生DMA请求。这样就允许ADC_DR中的转换结果传送到软件选择的目标位置。  
注意：ADC_CFGR1的DMAEN位必须在ADC校准完成后设置。  
尽管如此，如果因为DMA不能及时处理DMA请求而产生溢出（OVR=1），ADC将停止发出DMA请求，因此新的转换结果也不会再通过DMA传输。这也意味着所有已经传输到RAM的数据可以被认为是有效的(发生溢出后的数据不会再传输)。  
根据OVRMOD的设置，发生溢出时，ADC_DR寄存器中的数据或保持或被覆盖。  
OVR=1时，DMA请求被阻止，直到软件清除OVR。  
有2种不同的DMA模式，取决于ADC_CFGR1寄存器的DMACFG的配置：  
- DMA一次模式（DMACFG=0）：当DMA传输固定数量的数据时，选择此模式。  
- DMA循环模式（DMACFG=1）：当在循环模式或双缓冲模式下使用DMA时，应该选择此模式。  
####DMA一次模式（DMACFG=0）  
在此模式，ADC在每次新的转换数据有效时发出DMA请求，一旦DMA到达最后一次传输（DMA_EOT中断产生），ADC停止发送DMA请求，即使再次启动转换。  
######DMA one shot mode sequence code example  

	/* (1) Enable the peripheral clock on DMA */
	/* (2) Enable DMA transfer on ADC - DMACFG is kept at 0 for one shot mode */
	/* (3) Configure the peripheral data register address */
	/* (4) Configure the memory address */
	/* (5) Configure the number of DMA tranfer to be performs on DMA channel 1 */
	/* (6) Configure increment, size and interrupts */
	/* (7) Enable DMA Channel 1 */
	RCC->AHBENR |= RCC_AHBENR_DMA1EN; /* (1) */
	ADC1->CFGR1 |= ADC_CFGR1_DMAEN; /* (2) */
	DMA1_Channel1->CPAR = (uint32_t) (&(ADC1->DR)); /* (3) */
	DMA1_Channel1->CMAR = (uint32_t)(ADC_array); /* (4) */
	DMA1_Channel1->CNDTR = NUMBER_OF_ADC_CHANNEL; /* (5) */
	DMA1_Channel1->CCR |= DMA_CCR_MINC | DMA_CCR_MSIZE_0 | DMA_CCR_PSIZE_0
	                    | DMA_CCR_TEIE | DMA_CCR_TCIE ; /* (6) */
	DMA1_Channel1->CCR |= DMA_CCR_EN; /* (7) */  
当DMA传输完成（配置在DMA控制器中的所有传输都已经完成）：  
- ADC数据寄存器的内容冻结  
- 任何正在进行的转换被终止且部分结果被丢弃  
- 不会发送新的DMA请求给DMA控制器。如果仍有转换被启动，这种处理可以避免产生溢出错误。  
- ADC停止序列扫描并将其复位  
- DMA停止  
####DMA循环模式（DMACFG=1）  
在这种模式下，每次新的转换结果有效时，ADC会发出DMA请求，即使DMA已达到最后一个DMA传输。处理连续不断的模拟输入数据流时，可以使用此模式。  
######DMA circular mode sequence code example  

	/* (1) Enable the peripheral clock on DMA */
	/* (2) Enable DMA transfer on ADC and circular mode */
	/* (3) Configure the peripheral data register address */
	/* (4) Configure the memory address */
	/* (5) Configure the number of DMA tranfer to be performs on DMA channel 1 */
	/* (6) Configure increment, size, interrupts and circular mode */
	/* (7) Enable DMA Channel 1 */
	RCC->AHBENR |= RCC_AHBENR_DMA1EN; /* (1) */
	ADC1->CFGR1 |= ADC_CFGR1_DMAEN | ADC_CFGR1_DMACFG; /* (2) */
	DMA1_Channel1->CPAR = (uint32_t) (&(ADC1->DR)); /* (3) */
	DMA1_Channel1->CMAR = (uint32_t)(ADC_array); /* (4) */
	DMA1_Channel1->CNDTR = NUMBER_OF_ADC_CHANNEL; /* (5) */
	DMA1_Channel1->CCR |= DMA_CCR_MINC | DMA_CCR_MSIZE_0 | DMA_CCR_PSIZE_0
	                    | DMA_CCR_TEIE | DMA_CCR_CIRC; /* (6) */
	DMA1_Channel1->CCR |= DMA_CCR_EN; /* (7) */  
##低功耗特性  
