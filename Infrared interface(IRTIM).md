#Infrared interface(IRTIM)  
芯片上有一个可用于遥控的红外接口。它可以和红外发光二极管一起使用，来实现遥控功能。  
它与TIM16内部相连。  
要产生红外遥控信号，IR接口必须使能，并且TIM16的通道1（TIM16_OC1）必须适当配置以产生正确波形。  
红外接收通过基本的输入捕获模式可以很容易实现。  
![](https://i.imgur.com/JvOcplN.png)  
所有标准的IR脉冲调制类型都可以通过编程2个定时器的比较输出通道来获得。  
TIM17用于产生高频率的载波，TIM16用于产生调制包络。  
红外功能在IR_OUT引脚上输出。激活此功能需要使能GPIOx_AFRx寄存器中相应的复用功能位。  
高灌电流LED驱动（仅PB9脚）可以通过设置SYSCFG_CFGR1中的I2C_PB9_FMP位来激活，用来直接驱动红外LED。  
######TIM16 and TIM17 configuration code example  

	/* The following configuration is for RC5 standard */
	/* TIM16 is used for the enveloppe while TIM17 is used for the carrier */
	#define TIM_ENV TIM16
	#define TIM_CAR TIM17
	/* (1) Enable the peripheral clocks of Timer 16 and 17 and SYSCFG */
	/* (2) Enable the peripheral clock of GPIOB */
	/* (3) Select alternate function mode on GPIOB pin 9 */
	/* (4) Select AF0 on PB9 in AFRH for IR_OUT (reset value) */
	/* (5) Enable the high sink driver capability by setting I2C_PB9_FM+ bit in SYSCFG_CFGR1 */
	RCC->APB2ENR |= RCC_APB2ENR_TIM16EN | RCC_APB2ENR_TIM17EN | RCC_APB2ENR_SYSCFGCOMPEN; /* (1) */
	RCC->AHBENR |= RCC_AHBENR_GPIOBEN; /* (2) */
	GPIOB->MODER = (GPIOB->MODER & ~GPIO_MODER_MODER9) | GPIO_MODER_MODER9_1; /* (3) */
	GPIOB->AFR[1] &= ~(0x0F << ((9 - 8) * 4)); /* (4) */
	SYSCFG->CFGR1 |= SYSCFG_CFGR1_I2C_FMP_PB9; /* (5) */
	/* Configure TIM_CAR as carrier signal */
	/* (1) Set prescaler to 1, so APBCLK i.e 48MHz */
	/* (2) Set ARR = 1333, as timer clock is 48MHz the frequency is 36kHz */
	/* (3) Set CCRx = 1333/4, , the signal will be have a 25% duty cycle */
	/* (4) Select PWM mode 1 on OC1 (OC1M = 110),
	       enable preload register on OC1 (OC1PE = 1) */
	/* (5) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1)*/
	/* (6) Enable output (MOE = 1)*/
	TIM_CAR->PSC = 0; /* (1) */
	TIM_CAR->ARR = 1333; /* (2) */
	TIM_CAR->CCR1 = (uint16_t)(1333 / 4); /* (3) */
	TIM_CAR->CCMR1 |= TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1PE; /* (4) */
	TIM_CAR->CCER |= TIM_CCER_CC1E; /* (5) */
	TIM_CAR->BDTR |= TIM_BDTR_MOE; /* (6) */
	/* Configure TIM_ENV is the modulation enveloppe */
	/* (1) Set prescaler to 1, so APBCLK i.e 48MHz */
	/* (2) Set ARR = 42627, as timer clock is 48MHz the period is 888 us */
	/* (3) Select Forced inactive on OC1 (OC1M = 100) */
	/* (4) Select active high polarity on OC1 (CC1P = 0, reset value),
	       enable the output on OC1 (CC1E = 1) */
	/* (5) Enable output (MOE = 1) */
	/* (6) Enable Update interrupt (UIE = 1) */
	TIM_ENV->PSC = 0; /* (1) */
	TIM_ENV->ARR = 42627; /* (2) */
	TIM_ENV->CCMR1 |= TIM_CCMR1_OC1M_2; /* (3) */
	TIM_ENV->CCER |= TIM_CCER_CC1E; /* (4) */
	TIM_ENV->BDTR |= TIM_BDTR_MOE; /* (5) */
	TIM_ENV->DIER |= TIM_DIER_UIE; /* (6) */
	/* Enable and reset TIM_CAR only */
	/* (1) Enable counter (CEN = 1),
	       select edge aligned mode (CMS = 00, reset value),
	       select direction as upcounter (DIR = 0, reset value) */
	/* (2) Force update generation (UG = 1) */
	TIM_CAR->CR1 |= TIM_CR1_CEN; /* (1) */
	TIM_CAR->EGR |= TIM_EGR_UG; /* (2) */
	/* Configure TIM_ENV interrupt */
	/* (1) Enable Interrupt on TIM_ENV */
	/* (2) Set priority for TIM_ENV */
	NVIC_EnableIRQ(TIM_ENV_IRQn); /* (1) */
	NVIC_SetPriority(TIM_ENV_IRQn,0); /* (2) */  
######IRQHandler for IRTIM code example  

	/**
	 * Description: This function handles TIM_16 interrupt request.
	 *              This interrupt subroutine computes the laps between 2
	 *              rising edges on T1IC.
	 *              This laps is stored in the "Counter" variable.
	 */
	void TIM16_IRQHandler(void)
	{
		uint8_t bit_msg = 0;
		if ((SendOperationReady == 1) && (BitsSentCounter < (RC5_GlobalFrameLength * 2)))
		{
			if (BitsSentCounter < 32)
			{
				SendOperationCompleted = 0x00;
				bit_msg = (uint8_t)((ManchesterCodedMsg >> BitsSentCounter)& 1);
				if (bit_msg== 1)
				{
					/* Force active level - OC1REF is forced high */
					TIM_ENV->CCMR1 |= TIM_CCMR1_OC1M_0;
				}
				else
				{
					/* Force inactive level - OC1REF is forced low */
					TIM_ENV->CCMR1 &= (uint16_t)(~TIM_CCMR1_OC1M_0);
				}
			}
			BitsSentCounter++;
		}
		else
		{
			SendOperationCompleted = 0x01;
			SendOperationReady = 0;
			BitsSentCounter = 0;
		}
		/* Clear TIM_ENV update interrupt */
		TIM_ENV->SR &= (uint16_t)(~TIM_SR_UIF);
	}  
