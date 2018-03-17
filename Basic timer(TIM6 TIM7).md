#Basic timer(TIM6/TIM7)  
本节适用于STM32F030x8，STM32F070xB和STM32F030xC设备。TIM7只在STM32F070xB和STM32F030xC中有。  
##TIM6/TIM7简介  
基本定时器TIM6包含一个有可编程预分频器驱动的16位自动重载计数器。TIM7也同样。  
##TIM6/TIM7主要特性  
- 16位自动重装载向上计数器  
- 16位可编程预分频器（可实时修改），可以按1到65535之间任意数值分频时钟频率  
- 发生更新事件（计数器溢出）时产生中断/DMA请求  
![](https://i.imgur.com/LDR9DNZ.png)  
##TIM6/TIM7功能描述  
###时基单元  
定时器的主要模式是一个带自动重装载的16位向上计数器。计数器时钟由预分频器提供。  
计数器，自动重载寄存器和预分频器可以由软件进行读写，甚至在它们运行中。  
时基单元包括：  
- 计数器寄存器TIMx_CNT  
- 预分频器寄存器TIMx_PSC  
- 自动重载寄存器TIMx_ARR  
自动重载寄存器是预装载的。每次读写自动重载寄存器实际是访问预装载寄存器。预装载寄存器的值是立即写入影子寄存器，还是在发生更新事件UEV时才写入，取决于TIMx_CR1中的自动重载预装载使能位ARPE。如果TIMx_CR1中的UDIS=0，则当计数器计数达到溢出值时，就会发生更新事件。更新事件也可以由软件设置产生。  
计数器的时钟是预分频器输出的CK_CNT，只有当TIMx_CR1中的计数器使能位CEN=1时，时钟才被使能。  
注意，实际的计数器使能信号CNT_EN在CEN被置1一个时钟周期后才被置1。  
####预分频器描述  
预分频器可以将时钟频率在1到65535之间任意分频，然后提供给计数器。它是基于16位计数器通过16位寄存器（在TIMx_PSC中）控制的。它可以被实时修改，因为它带有缓冲。修改后的新分频值在发生更新事件时写入影子寄存器，开始使用。  
图135和136显示了当实时改变分频率时计数器的行为。  
![](https://i.imgur.com/1Erp0fw.png)  
![](https://i.imgur.com/l4U5jrO.png)  
###计数模式  
计数器从0计数到自动重载值（TIMx_ARR的内容），然后重新从0计数，并产生一个计数器溢出事件。  
更新事件的产生可以通过每一次的计数器溢出或向TIMx_EGR中写入UG=1（通过软件或使用从模式控制器）。  
UEV事件可以被禁止，通过向TIMx_CR1中写入UDIS=1。这避免了当向预装载寄存器写入新值时，发生更新事件，导致影子寄存器更新到错误值。此时，在UDIS=0之前，不会再有更新事件产生。但是，计数器和预分频器计数器还会如正常一样重新从0计数（但是分频率不变）。另外，如果TIMx_CR1中的URS=1，UG置1会产生更新事件UEV，但是不会置位UIF（所以也不会有中断或DMA请求产生）。  
当一个更新事件发生，所有寄存器被更新，并且TIMx_SR中的UIF置1（取决于URS的设置）：  
- 预分频器的预装载值写入其缓冲器。  
- 自动重载影子寄存器被更新为预装载值（TIMx_ARR）。  
下面的图显示了当TIMx_ARR=0x36时，不同时钟频率下的计数器行为。  
![](https://i.imgur.com/zReKfUB.png)  
![](https://i.imgur.com/OJJMry6.png)  
![](https://i.imgur.com/2HipOCo.png)  
![](https://i.imgur.com/aKpoQSl.png)  
![](https://i.imgur.com/SiYBJfF.png)  
![](https://i.imgur.com/zrjJxXO.png)  
###时钟源  
计数器的时钟由内部时钟源CK_INT提供。  
TIMx_CR1中的CEN位和TIMx_EGR中的UG位是实际控制位，并且只能通过软件修改它们（除了，UG位可以被自动清除）。一旦CEN=1，内部时钟向预分频器提供时钟。  
图143显示了控制电路和向上计数器在正常模式下的行为，不分频。  
![](https://i.imgur.com/oPdcr8S.png)  
###调试模式  
当微控制器进入调试模式（Cortex-M0内核停止），由DBG模块的DBG_TIMx_STOP配置位决定，TIMx计数器继续计数或停止。  
##TIM6/TIM7寄存器  
可以按半字（16位）或字（32位）来访问这些外设寄存器。  
###TIM6/TIM7控制寄存器（TIMx_CR1）  
![](https://i.imgur.com/vGv4VbQ.png)  
![](https://i.imgur.com/1yTqHvC.png)  
###TIM6/TIM7 DMA/中断使能寄存器（TIMx_DIER）  
![](https://i.imgur.com/ldjpO9X.png)  
###TIM6/TIM7状态寄存器（TIMx_SR）  
![](https://i.imgur.com/8f8QOhH.png)  
###TIM6/TIM7事件生成寄存器（TIMx_EGR）  
![](https://i.imgur.com/HhoV43l.png)  
###TIM6/TIM7计数器（TIMx_CNT）  
![](https://i.imgur.com/y0AIqhq.png)  
###TIM6/TIM7预分频器（TIMx_PSC）  
![](https://i.imgur.com/PqmAo9y.png)  
###TIM6/TIM7自动重载寄存器（TIMx_ARR）  
![](https://i.imgur.com/ZeBJsjr.png)  
##TIM6/TIM7寄存器映射  
![](https://i.imgur.com/hyUkKNX.png)  
