#Direct memory access controller(DMA控制器)  
DMA用于在外设和内存、内存和内存之间，提供高速数据传输。Data可以通过DMA快速传输，而不需要CPU干预；为其他操作节省了CPU资源。  
DMA控制器有5个通道，每个通道专门用来管理来自一个或多个外设的内存访问请求。还有一个仲裁器处理DMA请求的优先级。  
##DMA的主要特性  
- 5个独立可配置的DMA通道（请求）  
- 每个通道连接专门的硬件DMA请求或软件触发，由软件进行配置。  
- DMA请求的优先级可由软件设置（4级：很高，高，中，低），在优先级相等时，由硬件决定（请求1优先级高于请求2，以此类推）。  
- 源和目标的传输数据宽度（字节，半字，字）是独立，如果不同，DMA会把数据打包或拆开，以满足目标要求的数据宽度。源/目标地址必须按传输数据宽度对齐。  
- 支持循环缓冲区管理  
- 每个通道有3个事件标志（DMA半传输，DMA传输完成和DMA传输错误），它们按逻辑或成一个中断请求。  
- 内存间传输  
- 外设和内存间传输，外设间传输  
- 外设可以作为源或目标访问   
- 可编程的数据传输个数：最大65535  
##DMA功能描述  
![](https://i.imgur.com/a4zzoQc.png)  
DMA控制器通过和ARM Cortex-M0内核共享系统总线，执行直接内存传输。当CPU和DMA访问同样的目标（内存或外设）时，DMA请求可能会在几个总线周期内暂停CPU访问系统总线。总线矩阵实现循环调度，以确保CPU至少有一半系统总线宽度。  
####DMA处理  
发生一个事件后，外设向DMA控制器发送请求。DMA控制器根据通道优先级处理请求。当DMA控制器访问发送请求的外设时，DMA控制器会发送一个应答信号给这个外设。当这个外设收到DMA控制器的应答后，立马释放请求。一旦外设释放请求，DMA控制器也会撤销应答。如果有多个请求，那么外设就可以启动下个处理。  
总的来说，每个DMA传输包含3组操作：  
- 从外设数据寄存器或当前外设/内存地址寄存器所指内存地址，取数据。在DMA_CPARx或DMA_CMARx寄存中编写的外设/内存地址，作为第一次传输的起始地址。  
- 存储，从外设数据寄存器或当前外设/内存地址寄存器所指内存地址，取得的数据。在DMA_CPARx或DMA_CMARx寄存中编写的外设/内存地址，作为第一次传输的起始地址。  
- DMA_CNDTRx寄存器存放需要执行的处理个数，完成一次处理，其值减1。  
###仲裁器  
仲裁器根据通道优先级管理通道请求，并启动外设/内存存取步骤。  
优先级管理有两个阶段：  
- 软件：在DMA_CCRx寄存器配置每个通道的优先级。有四个等级：  
　　- 最高级  
　　- 高级  
　　- 中级  
　　- 低级  
- 硬件：如果2个请求有相同的软件优先级，则请求的通道编号越低优先级越高。比如，通道2的优先级就高于通道4。  
###DMA通道  
每个通道都可以处理外设寄存器和内存间的DMA传输，传输的数据量是可编程的，最大65535。存储传输数据量的寄存器，在每次传输完成后自减1，直到0。  
####可编程的数据传输量  
外设和内存的传输数据量可以通过DMA_CCRx寄存器的PSIZE和MSIZE位设置。  
####增量指针  
通过设置DMA_CCRx寄存器的PINC和MINC位，外设和内存指针在每次传输完后可以选择自动增加。当设置为增量模式时，下次传输的地址是在上次传输的地址上加1,2或4（取决于所选的传输数据宽度）。第一次传输的地址是编写在DMA_CPARx/DMA_CMARx寄存器中的。在传输过程中，这些寄存器的值保持不变。当前传输的地址（在内部外设/内存地址寄存器中的当前值）不能被软件改变。  
如果通道被配置成非循环传输模式，传输结束（当传输数据计数减到0）后将不再产生DMA请求。必须先关闭DMA通道，才能在传输数据计数寄存器DMA_CNDTRx中写入新值。  
注意：DMA通道关闭，DMA寄存器并不会复位。DMA通道寄存器（DMA_CCRx,DMA_CPARx和DMA_CMARx）在通道配置阶段会保持初始值。  
在循环传输模式，在最后一次传输结束时（DMA_CNDTRx=0），DMA_CNDTRx会自动转载编程的初始值。当前内部地址寄存器自动装载DMA_CPARx/DMA_CMARx寄存器的值。  
####通道配置流程  
下面是配置DMA通道x(x代表通道号)的步骤：  
- 在DMA_CPARx寄存器中设置外设寄存器地址。发生外设数据传输时，将从这个地址开始传送数据到内存，或结束内存传来的数据。  
- 在DMA_CMARx寄存器设置内存地址。当发生外设数据传输时，将从这个地址开始写入或读取数据。  
- 在DMA_CNDTRx寄存器设置传输数据的总数。每次传输后，该寄存器的值自动递减。  
- 在DMA_CCRx寄存器的PL[1:0]位配置通道优先级。  
- 在DMA_CCRx寄存器设置传输方向，循环模式，外设和内存增量模式，外设和内存数据宽度，传输一半和/或传输完成产生中断。  
- 在DMA_CCRx寄存器的ENABLE位，打开该通道。  
######DMA Channel Configuration sequence code example  

	/* The following example is given for the ADC. It can be easily ported on
	any peripheral supporting DMA transfer taking of the associated channel
	to the peripheral, this must check in the datasheet. */
	/* (1) Enable the peripheral clock on DMA */
	/* (2) Enable DMA transfer on ADC */
	/* (3) Configure the peripheral data register address */
	/* (4) Configure the memory address */
	/* (5) Configure the number of DMA tranfer to be performs on channel 1 */
	/* (6) Configure increment, size and interrupts */
	/* (7) Enable DMA Channel 1 */
	RCC->AHBENR |= RCC_AHBENR_DMA1EN; /* (1) */
	ADC1->CFGR1 |= ADC_CFGR1_DMAEN; /* (2) */
	DMA1_Channel1->CPAR = (uint32_t) (&(ADC1->DR)); /* (3) */
	DMA1_Channel1->CMAR = (uint32_t)(ADC_array); /* (4) */
	DMA1_Channel1->CNDTR = 3; /* (5) */
	DMA1_Channel1->CCR |= DMA_CCR_MINC | DMA_CCR_MSIZE_0 | DMA_CCR_PSIZE_0
						| DMA_CCR_TEIE | DMA_CCR_TCIE ; /* (6) */
	DMA1_Channel1->CCR |= DMA_CCR_EN; /* (7) */
	/* Configure NVIC for DMA */
	/* (1) Enable Interrupt on DMA Channel 1 */
	/* (2) Set priority for DMA Channel 1 */
	NVIC_EnableIRQ(DMA1_Channel1_IRQn); /* (1) */
	NVIC_SetPriority(DMA1_Channel1_IRQn,0); /* (2) */  
一旦通道打开，它就能响应任何来自连接到该通道上的外设的DMA请求。  
但数据传输一半时，HTIF半传标志置位；并且，如果HTIE半传中断使能位置位，将产生中断。当传输完成时，TCIF传输完成标志置位；如果TCIE传输完成中断使能位置位，将产生中断。  
####循环模式  
循环模式可以用来处理循环缓冲区和连续数据流（例如，ADC扫描模式）。DMA_CCRx寄存器的CIRC位用于开启或关闭该功能。当开启循环模式，当一组数据传输完成，DMA_CNDTRx寄存器将自动装载在通道配置阶段编程的传输数据量初始值，并将持续响应DMA请求。  
####内存到内存模式  
DMA通道同样可以在没有外设请求的情况下工作，这种模式被称为内存到内存模式。  
如果DMA_CCRx寄存器的MEM2MEM位置位，一旦DMA_CCRx的EN位置位，通道马上启动传输。一旦DMA_CNDTRx寄存器的值减到0，传输停止。内存到内存模式不能和循环模式同时使用。  
###可编程的数据宽度，数据对齐方式和大小端  
