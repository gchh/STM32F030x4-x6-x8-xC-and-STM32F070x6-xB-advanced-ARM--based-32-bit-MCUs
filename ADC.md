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
