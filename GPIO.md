#GPIO  
每个GPIO口都有4个32-bit配置寄存器（GPIOx_MODER,GPIOx_OTYPER,GPIOx_OSPEEDR和GPIOx_PUPDR），2个32-bit数据寄存器（GPIOx_IDR,GPIOx_ODR）和一个32-bit的置位/复位寄存器（GPIOx_BSRR）。端口A和B还有1个32-bit的锁定寄存器（GPIOx_LCKR）和2个32-bit的复用功能选择寄存器（GPIOx_AFRH和GPIOx_AFRL）。  
在STM32F030xB和STM32F030xC上，端口C和D也有2个32-bit的复用功能选择寄存器（GPIOx_AFRH和GPIOx_AFRL）。  
##GPIO主要特性  
- 输出状态：推挽，开漏，上拉或下拉  
- 输出数据来自输出数据寄存器（GPIOx_ODR），或是外设（复用功能输出）  
- 每个IO可以选速度  
- 输入状态：悬空，上拉或下拉，模拟  
- 输入数据来自输入数据寄存器（GPIOx_IDR），或外设（复用功能输入）  
- 置位/复位寄存器（GPIOx_BSRR）提供了对GPIOx_ODR的按位写访问  
- 锁定机制（GPIOx_LCKR）用来冻结端口A和B的配置  
- 模拟功能  
- 复用功能选择寄存器（最多每个IO有16个复用功能）  
- 每2个时钟周期快速切换的能力  
- 高度灵活的引脚复用：可以作为GPIO使用或其他外设功能  
##GPIO功能描述  
根据数据手册列出的每个IO口的硬件特性，每一个IO口可以单独通过软件设置成下面几种模式：  
- 浮空输入  
- 上拉输入  
- 下拉输入  
- 模拟输入/输出  
- 上拉或下拉开漏输出  
- 上拉或下拉推挽输出  
- 上拉/下拉推挽复用功能输出  
- 上拉/下拉开漏复用功能输出  
每个IO口可以自由编程，但是IO口寄存器必须按字，半字或字节访问。GPIOx_BSRR和GPIO_BRR可以按位读/修改GPIOx_ODR。在这种情况下，在读/修改过程中发生IRQ也不会发生危险。  
![](https://i.imgur.com/1yETYYN.png)  
![](https://i.imgur.com/7vPHrlb.png)  
![](https://i.imgur.com/097pYt0.png)  
###GPIO  
在复位期间或刚复位，复用功能还未打开，并且大多数IO口被配置成浮空输入。  
调试引脚复位后被配置成复用AF PU/AF PD：  
- PA14：SWCLK下拉  
- PA13：SWDIO上拉  
当IO配置成输出脚，写入GPIOx_ODR的值将输出到相应引脚上。因此，可以使用推挽或开漏（ODR=0引脚输出低电平，ODR=1时引脚是高阻态）输出驱动外部电路。  
GPIOx_IDR在每个AHB时钟周期捕捉IO引脚上的数据。  
所有IO口带内部弱上拉和下拉，设置GPIOx_PUPDR寄存器使用或关闭它们。  
###IO口的复位功能和映射  
