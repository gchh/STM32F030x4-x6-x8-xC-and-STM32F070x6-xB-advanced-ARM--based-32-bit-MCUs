# Universal synchronous asynchronous receiver transmitter (USART)  
## 简介  
通用同步异步收发器（USART）提供了一种灵活的方式，与要求工业标准NRZ异步串行数据格式的外部设备进行全双工数据交换。USART使用可编程波特率发生器，提供很宽范围的波特率。  
它支持同步单向通信和半双工单线通信，以及多处理器通信。它也支持Modem调制解调器流控操作（CTS/RTS）。  
使用DMA（直接存储器存取）配置多个缓冲区，可以实现高速数据通信。  
## USART主要特性  
- 全双工异步通信  
- NRZ标准格式（mark标记/space空格）  
- 配置16倍或8倍过采样，提供速度和时钟公差之间灵活的选择  
- 当时钟频率时48MHz，且采用8倍过采样率时，通用可编程收发波特率可以达到6Mbit/s  
- 方便的波特率编程  
- 自动波特率检测  
- 可编程的数据字长（8位或9位）  
- 可编程的数据顺序，MSB最高位或LSB最低位在前  
- 可配置的停止位（1或2个停止位）  
- 用于同步通信的同步模式和时钟输出  
- 单线半双工通信  
- 使用DMA实现连续通信  
- 使用DMA将收到/发送的字节缓存到预留的SRAM区域中  
- 分开的发送和接收使能位  
- 分开的发送和接收信号极性控制  
- 可交换的Tx/Rx引脚配置  
- 用于Modem调制解调器和RS-485收发器的硬件流控制  
- 通信控制/错误检测标志  
- 奇偶校验控制：  
　- 发送奇偶校验位  
　- 检查接收的数据字节的奇偶性  
- 十四个具有标志位的中断源  
- 多处理器通信  
- 如果地址不匹配，USART进入静默模式  
- 从静默模式唤醒（检测到线路空闲或地址匹配标记）  
## USART配备  
![](https://i.imgur.com/rwosfMn.png)  
![](https://i.imgur.com/9fcdvIG.png)  
## USART功能描述  
任何USART双向通信都需要至少2个引脚：RX接收数据和TX发送数据：  
- RX：接收数据输入引脚。过采样技术用于数据恢复，通过鉴别有效的输入数据和噪声。  
- TX：发送数据输出引脚。当发送器关闭时，该输出引脚回到其I/O口配置状态。当发送器启动，但未发送数据时，TX引脚是高电平。在单线模式，该I/O口用作发送和接收数据。  
正常USART模式下，串行数据通过这些引脚发送和接收。数据帧由以下组成：  
- 发送或接收前保持线路空闲  
- 一个起始位  
- 一个数据字（8位或9位），最低有效位在前  
- 1个或2个停止位，指示帧结束  
- USART接口使用波特率发生器  
- 一个状态寄存器USART_ISR  
- 接收和发送数据寄存器USART_RDR和USART_TDR  
- 一个波特率寄存器USART_BRR  
下面的引脚在同步模式下需要：  
- CK：时钟输出。该引脚为同步发送，输出发送数据时钟，类似于SPI主模式（起始位和停止位没有时钟脉冲，最后一位数据软件可选是否发送时钟脉冲）。同时，RX可以同步接收数据。这可以用于控制带移位寄存器的外设。时钟相位和极性可由软件编程。  
下列引脚用于RS232硬件流控制模式：  
- CTS：当高电平时，在当前发送结束后阻止数据发送  
- RTS：低电平，指示USART准备好接收数据  
下面的引脚用于RS485硬件控制模式：  
- DE：驱动器使能，用于激活外部收发器的发送模式  
注：DE和RTS共享同一个引脚。  
![](https://i.imgur.com/y7GnxBD.png)  
### USART字符说明  
通过USART_CR1寄存器中的M位（第12bit：M0），可以编程选择8位或9位字长。  
- 8位字符长度：M0=0  
- 9位字符长度：M0=1  
默认情况下，TX和RX信号在起始位时为低电平，而在停止位时为高电平。  
通过极性配置控制，可以分别对TX或RX的这些值取反。  
空闲字符可以理解为整个帧每一位都为1。  
断开字符可以理解为每一位都为0的帧。在断开帧的结尾，发送器会插入2个停止位。  
发送器和接收器有共同的波特率发生器驱动，发送和接收各自的使能位置1后，将为其产生时钟。  
![](https://i.imgur.com/LCrh5yZ.png)  
![](https://i.imgur.com/2ZZr5n5.png)  
注：这两幅图有差异，感觉第2幅图才是正确的，符合断开字符的描述。  
### USART发送器  
发送器可以发送8位或9位的数据字，取决于M位的状态。发送使能位TE必须置1，以激活发送器功能。发送移位寄存器中的数据输出到TX引脚上，同时相应的时钟脉冲输出在CK引脚上。  
#### 字符发送  
USART发送时，默认配置下，数据的最低有效位首先移出到TX引脚上。在此模式下，USART_TDR寄存器充当了一个内部总线和发送移位寄存器之间的缓冲器（TDR）（见图226）。  
每个字符前面都有一个起始位，它是持续一个位周期的逻辑低电平。字符以配置的1个或2个停止位结束。  
注：在向USART_TDR寄存器写入要发送的数据之前必须先将TE位置1。  
TE位在发送数据期间不能复位。否则，会冻结波特率计数器，导致TX引脚上的数据损坏，当前正在发送的数据就会丢失。  
TE位置1后，将发送一个空闲帧。  
#### 可配置的停止位  
跟随每个字符发送的停止位个数可以在控制寄存器2的第12和13位编程。  
- 1个停止位：停止位个数的默认值。  
- 2个停止位：正常的USART，单线模式和调制解调器模式，支持。  
发送的空闲帧包括停止位。  
发送的断开帧包括10个低电平位（M0=0）或11个低电平位（M0=1），其后紧跟2个停止位（见图228）。无法发送更长的断开帧（长度大于10或11位）。  
![](https://i.imgur.com/PzlGimn.png)  
#### 字符发送配置  
1. 配置USART_CR1中的M位确定字长。  
2. 配置USART_BRR寄存器确定波特率。  
3. 在USART_CR2中配置停止位个数。  
4. 将USART_CR1寄存器中的UE位置1，使能USART。  
5. 如果采用多缓冲器通信，在USART_CR3中选择DMA使能（DMAT）。按照多缓冲器通信的说明配置DMA寄存器。  
6. 将USART_CR1中的TE位置1，发送一个空闲帧作为第一次发送。  
7. 向USART_TDR寄存器中写入要发送的数据（这将清零TXE位）。在单缓冲器的情况下，对每个要发送的数据重复该步骤。  
8. 将最后一个数据写入USART_TDR寄存器后，等待TC=1。它表示最后一帧数据发送完成。在禁止USART或进入停机模式之前，可以用它来确认发送是否完成，以免破环最后一次发送。  
###### USART transmitter configuration code example  

	/* (1) Oversampling by 16, 9600 baud */
	/* (2) 8 data bit, 1 start bit, 1 stop bit, no parity */
	USART1->BRR = 480000 / 96; /* (1) */
	USART1->CR1 = USART_CR1_TE | USART_CR1_UE; /* (2) */  
#### 单字节通信  
清零TXE位总是通过向发送数据寄存器写数据来执行的。  
TXE位由硬件置位，它表示：  
- 数据已经从USART_TDR寄存器转移到的移位寄存器中，且数据发送已经开始。  
- USART_TDR寄存器是空的。  
- 下一个数据可以写入USART_TDR寄存器中，而不会覆盖前一个数据。  
###### USART transmit byte code example  

	/* Start USART transmission */
	USART1->TDR = stringtosend[send++]; /* Will inititiate TC if TXE is set*/  
如果TXEIE位置1，TXE=1会产生中断。  
在发送中，对USART_TDR寄存器的写操作将数据存入TDR寄存器；此后，在当前正在进行的发送结束后，该数据复制进移位寄存器。  
未发送时，对USART_TDR寄存器的写操作会将数据直接放入移位寄存器，数据发送开始时，TXE位会被置1。  
如果一帧发送完毕（停止位发送后），此时TXE位还是1，那么，TC位将被置1。如果USART_CR1寄存器中的TCIE位置1，此时还会产生一个中断。  
向USART_TDR寄存器写入最后一个要发送的数据后，在禁止USART或设置微控制器进入低功耗模式之前，必须等待TC=1。  
![](https://i.imgur.com/pPeiG8A.png)  
###### USART transfer complete code example  

	if ((USART1->ISR & USART_ISR_TC) == USART_ISR_TC)
	{
		if (send == sizeof(stringtosend))
		{
			send=0;
			USART1->ICR |= USART_ICR_TCCF; /* Clear transfer complete flag */
		}
		else
		{
			/* clear transfer complete flag and fill TDR with a new char */
			USART1->TDR = stringtosend[send++];
		}
	}  
#### 断开字符  
将SBKRQ位置1，发送一个断开字符。断开帧的长度取决于M位（见图227）。  
如果SBKRQ位被置1，将在完成当前字符发送后，在TX上发送断开字符。SBKF位会在SBKRQ=1时置1，并在断开字符发送完成时由硬件清零（在断开字符后的停止位发送期间）。USART在断开字符后插入2个停止位，以确保下一帧起始位的识别。  
在应用需要在之前插入的所有数据（包括尚未发送的数据）后发送断开字符的情况下，在SBKRQ位置1之前需要确定TXE=1。  
#### 空闲字符  
将TE位置1，USART会在第一个数据帧之前首先发送一个空闲字符。  
### USART接收器  
USART接收8位或9位的数据字，取决于USART_CR1寄存器中M位的状态。   
#### 起始位侦测  
起始位侦测顺序是一样的，不管是8倍过采样还是16倍过采样。  
在USART中，当辨识出一个特定的采样序列，就认为侦测到一个起始位。该序列是：1 1 1 0 X 0 X 0 X 0 X 0 X 0 X 0。  
![](https://i.imgur.com/DFgCZeH.png)  
注：如果该序列不完整，则，起始位检测终止，接收器返回空闲状态（无标志位置1）等待下降沿。  
如果3个采样位都是0（在第3、5、7位的第一次采样3位都是0，在第8、9、10位的第二次采样3位也都是0），则可以确认收到起始位（RXNE标志置1，如果RXNEIE=1会产生中断）。  
下列情况也可以认为收到起始位（RXNE标志置1，如果RXNEIE=1会产生中断），但是噪声标志NF会置1：  
a) 2次采样，3个采样位中有2位是0  
b) 2次采样，其中一次采样的3个采样位中有2位是0，另一次采样3个采样位都是0  
如果a和b条件都不满足，则中止起始位检测，接收器回到空闲状态（无标志位置1）。  
#### 字符接收  
在USART接收期间，数据的最低有效位（默认配置）首先通过RX引脚移入。这种情况下，USART_RDR寄存器可以被看作是内部总线和接收移位寄存器之间的缓冲器（RDR）。  
##### 字符接收配置步骤  
1. 配置USART_CR1中的M位，确定字长。  
2. 配置USART_BRR，确定波特率。  
3. 在USART_CR2中配置停止位个数。  
4. 将USART_CR1中的UE位置1,使能USART。  
5. 如果采用多缓冲器通信，配置USART_CR3中的DMA使能位（DMAR）。根据多缓冲器通信的说明配置DMA寄存器。  
6. 将USART_CR1中的RE位置1，使能接收器，开始侦测起始位。  
###### USART receiver configuration code example  

    /* (1) oversampling by 16, 9600 baud */
    /* (2) 8 data bit, 1 start bit, 1 stop bit, no parity, reception mode */
    USART1->BRR = 480000 / 96; /* (1) */
    USART1->CR1 = USART_CR1_RXNEIE | USART_CR1_RE | USART_CR1_UE; /* (2) */  
当接收到一个字符时：  
- RXNE位置1，表示移位寄存器的内容已经传送到RDR。也就是，数据已经被接收，并可以读取（以及相关的错误标志）。  
- 如果RXNEIE位为1，还会产生中断。  
- 如果接收期间检测到帧错误、噪声或溢出错误，相应的错误标志位会置1。PE标志也会和RXNE一起被置1。  
- 在多缓冲器通信时，每接收一个字节后RXNE都会被置1，并由DMA读取接收数据寄存器而被清零。  
- 在单缓冲器模式下，RXNE位由软件读USART_RDR寄存器而被清零。RXNE标志也可以通过将USART_RQR中的RXFRQ位置1,而被清零。RXNE位必须在下一个字符接收结束之前被清零，以免发生溢出错误。  
###### USART receive byte code example  

    if ((USART1->ISR & USART_ISR_RXNE) == USART_ISR_RXNE)
    {
        chartoreceive = (uint8_t)(USART1->RDR); /* Receive data, clear flag */
    }  
#### 断开字符  
当收到断开字符后，USART会像帧错误一样处理它。  
#### 空闲字符  
检测到空闲帧时，会和接收到数据一样处理它；另外，如果IDLEIE位置1，还将产生中断。  
#### 溢出错误  
当RXNE=1时，又接收到一个字符，会导致溢出错误。新接收到的字符不会从移位寄存器传送到RDR寄存器中，直到RXNE位清零。  
每接收一个字节后，RXNE标志都会置1。如果在下一个数据已经被接收到或上一个DMA请求还未处理时RXNE标志被置1，则会产生溢出错误。当一个溢出错误发生时：  
- ORE位置1。  
- RDR中的内容不会丢失。读USART_RDR依然能得到前次数据。  
- 移位寄存器的内容会被覆盖；溢出发生后，接收到的数据都将丢失。  
- RXNEIE位和EIE位任何一个置1，都将产生中断。  
- 将USART_ICR中的ORECF位置1，会清除ORE位。  
注：当ORE位置1时，表明至少有1个数据已经丢失。有2种可能： 
- 如果RXNE=1，RDR接收寄存器中还有最后一个有效数据，可以被读取。  
- 如果RXNE=0，最后一个有效数据已经读取，RDR中已没有数据可读取。收到新数据（会丢失）的同时读取最后一个有效数据，会发生此种情况。  
#### 选择时钟源和适当的过采样方法  
时钟源的选择通过时钟控制系统完成（参见复位和时钟控制（RCC）章节）。在使能USART（UE位置1）之前，必须选择好时钟源。  
必须根据2个标准来选择时钟源：  
- USART可能会在低功耗模式下使用  
- 通信速度  
时钟源频率是f<sub>CK</sub>。  
时钟源决定了通信速度范围（尤其是最大通信速度）。  
接收器采用不同的用户可配置的过采样技术，将有效输入和噪声区分开来，从而实现数据恢复。这实现了最大通信速度和抗噪声/时钟误差之间的平衡。  
通过USART_CR1中的OVER8位选择过采样是波特率的8倍或16倍（图231和232）。  
根据应用：  
- 选择8倍过采样（OVER8=1）以获得较高的速度（高达f<sub>CK</sub>/8）。这种情况下，接收器对时钟偏差的最大容忍度会降低。  
- 选择16倍过采样（OVER8=1）会提高接收器对时钟偏差的容忍度。这种情况下，最大速度限制在f<sub>CK</sub>/16，其中f<sub>CK</sub>为时钟源频率。  
配置USART_CR3中的ONEBIT位，选择用于评估逻辑电平的方法。有2种选择：  
- 根据接收位中央的3次采样进行多数票决。这种情况下，当3才采样结果不等时，NF位置1。  
- 接收位的中央进行单次采样。  
根据不同的应用：  
- 在噪声环境下工作时，选择3次采样多数票决的方法（ONEBIT=0）；当检测到噪声时，抛弃该数据，因为它表明在采样过程中有干扰。  
- 在线路无噪声时，选择单次采样的方法（ONEBIT=1），以提高接收器对时钟偏差的容忍度。在这种情况下，NF位永远不会被置位。  
当在帧中检测到噪声：  
- NF位在RXNE位的上升沿被置1。  
- 无效的数据仍然会从移位寄存器转移到USART_RDR寄存器中。  
- 在单字节通信的情况下不会产生中断。但是和NF位同时置位的RXNE位可以产生中断。在多缓冲器通信的情况，如果USART_CR3中的EIE位置1，则会产生中断。  
将USART_ICR中的NFCF位置1，清除NF位。  
![](https://i.imgur.com/usaJ56U.png)  
![](https://i.imgur.com/1pF6npK.png)  
![](https://i.imgur.com/FsZPyyz.png)  
#### 帧错误  
失去同步、大量噪声、或是在预期的时间没有辨认出停止位，都会导致帧错误发生。  
检测到帧错误时：  
- FE位由硬件置1  
- 无效的数据仍然会从移位寄存器转移到USART_RDR中  
- 在单字节通信的情况下不会产生中断。但是和FE位同时置位的RXNE位可以产生中断。在多缓冲器通信的情况，如果USART_CR3中的EIE位置1，则会产生中断。  
将USART_ICR中的FECR位置1，清除FE位。  
#### 配置接收期间的停止位  
通过USART_CR2配置接收的停止位个数；正常模式下，可以是1位或2位。  
- 1个停止位：在第8、9和10个采样点对1个停止位进行采样。  
- 2个停止位：在第8、9和10个采样点对第1个停止位进行采样，如果发现帧错误，FE标志位会被置1。第2个停止位不检查帧错误。在第1个停止位结束时，RXNE标志会被置1。  
### USART波特率产生  
接收器和发送器（RX和TX）的波特率都在USART_BRR中设置为相同的值。  
#### 公式1：标准USART（包括SPI模式）（OVER8=0或1）的波特率  
OVER8=0时，Tx/Rx波特率=f<sub>CK</sub>/USARTDIV  
OVER8=1时，Tx/Rx波特率=(2×f<sub>CK</sub>)/USARTDIV  
USARTDIV是一个存放在USART_BRR寄存器中的无符号定点数。  
- 当OVER8=0时，BRR=USARTDIV。  
- 当OVER8=1时，  
　- BRR[2:0]=USARTDIV[3:0]>>1  
　- BRR[3]必须保持为0  
　- BRR[15:4]=USARTDIV[15:4]  
注：写入USART_BRR中的波特率新值会立即更新波特率计数器。因此，在通信过程中，不应该改变波特率寄存器的值。  
在8倍或16倍过采样的情况下，USARTDIV必须大于等于16（推测，是因为这样采样频率才不会超过f<sub>CK</sub>，不知是否如此？）。  
#### 如何从USART_BRR寄存器的值算出USARTDIV  
##### 例1  
f<sub>CK</sub>=8MHz，波特率9600：  
- OVER8=0，16倍过采样：USARTDIV=8000000/9600，BRR=USARTDIV=833d=0341h  
- OVER8=1，8倍过采样：USARTDIV=(2*8000000)/9600=1667d=683h，BRR[3:0]=3h>>1=1h，BRR=0x681  
##### 例2  
f<sub>CK</sub>=48MHz，波特率921.6K：  
- OVER8=0，16倍过采样：USARTDIV=48000000/921600，BRR=USARTDIV=52d=34h  
- OVER8=1，8倍过采样：USARTDIV=(2*48000000)/921600=104d=68h，BRR[3:0]=USARTDIV[3:0]>>1=8h>>1=4h，BRR=0x64  
![](https://i.imgur.com/weJyRTY.png)  
![](https://i.imgur.com/I7tiKHQ.png)  
### USART接收器对时钟偏差的容忍度  
仅当全部时钟系统的偏差小于USART接收器的公差时，USART的异步接收器才能正确工作。影响总偏差的因素有：  
- DTRA：发送器误差引起的偏差（包括发送器的振荡器偏差）  
- DQUANT：接收器波特率取整数引起的误差  
- DREC：接收器的振荡器的偏差  
- DTCL：传输线路造成的偏差（通常由于收发器从低电平变为高电平的转换时间，和从低电平变为高电平的转换时间不一致造成）  
DTRA + DQUANT + DREC + DTCL < USART接收器的容忍度  
USART接收器在表88和表89中指定的最大容许偏差下，可以正确接收数据。表88和表89中指定的最大容许偏差的大小取决于下面的选择：  
- USART_CR1中的M0位选择8或9位字符长度  
- USART_CR1中的OVER8位选择8倍或16倍过采样  
- USART_BRR寄存器中的BRR[3:0]位等不等于0000  
- USART_CR3寄存器中的ONEBIT位选择对数据进行1位或3位采样  
![](https://i.imgur.com/HDgPQv4.png)  
注：当接收到的帧包含一些空闲帧（M=0时10位，M=1时11位）时，表88和表89中的数据可能会有些许偏差。  
### USART波特率自动检测  
USART可以根据接收到的一个字符来检测并自动设置USART_BRR寄存器的值。在2种情况下，波特率自动检测很有用：  
- 事先不知道系统的通信速度。  
- 系统使用的时钟源精度比较低。自动检测波特率可以在不测量时钟偏差的情况下得到正确的波特率。  
时钟源频率必须与期望的通信速度匹配（16倍过采样时，波特率必须在f<sub>CK</sub>/65535和f<sub>CK</sub>/16之间）。  
在打开波特率自动检测前，要先选择波特率自动检测模式。根据不同的字符模式，有各种检测模式。  
可以通过USART_CR2中的ABRMOD[1:0]位选择这些模式。在这些自动波特率模式下，波特率在同步数据接收期间被多次测量，并且每次测量结果都会和前一次比较。  
这些模式是：  
- 模式0：以1开头的任何字符。这种情况下，USART测量起始位的持续时间（下降沿到上升沿）。  
- 模式1：以10xx开头的任何字符。这种情况下，USART测量起始位和第一个数据位的持续时间。测量下降沿到下降沿的时间，可以确保在信号斜率较小时得到更好的精度。  
同时，对RX线上的每次中间转换执行其它检查。如果RX上的转换未与接收器（接收器的波特率是由bit0计算得到的）充分同步，将产生错误。  
在打开波特率自动检测之前，需要初始化USART_BRR为非0值。  
将USART_CR2中的ABREN位置1，打开波特率自动检测。之后USART等待RX线上的第一个字符。USART_ISR寄存器中的ABRF标志置1，表示自动波特率操作完成。如果线路充满噪声，则无法保证检测到正确波特率。这种情况下，BRR的值可能是错误的，并且ABRE错误标志将置1。如果通信速度和自动波特率检测范围不兼容（16倍过采样时，1位持续时间不在16到65536个时钟周期内；8倍过采样时，位持续时间不在8到65536个时钟周期内），也会发送这种情况。  
RXNE的中断将指示操作结束。  
任何时候，都可以将ABRF清零，重启波特率自动检测。  
注：如果在波特率自动检测操作期间，关闭USART（UE=0），则可能损坏BRR的值。  
### 多处理器使用USART通信  
在多处理通信中，下列的位要清零：  
- USART_CR2中的LINEN位，  
- USART_CR3中的HDSEL、IREN和SCEN位。  
通过USART进行多机通信是可行的（多个USART连接在一个网络中）。例如，其中一个USART作为主机，它的TX输出连接其它USART的RX输入。其它的USART就作为从机，它们各自的TX输出经过逻辑与连接主机的RX输入。  
在多机通信配置中，通常希望只激活期望的接收器来接收完整信息，从而减少未被寻址的接收器带来的多余的USART服务开销。  
可以通过静默功能将未被寻址的器件置于静默模式中。要使用静默模式，USART_CR1中的MME位必须置1。  
在静默模式中：  
- 没有任何接收状态位会被置1。  
- 所有接收中断被阻止。  
- USART_ISR中的RWU位置1。RWU可以由硬件自动控制，或在某些情况下，通过USART_RQR中的MMRQ位，由软件控制。  
USART有2种方法进入或退出静默模式，使用哪一种取决于USART_CR1中的WAKE位：  
- 如果WAKE=0，则进行空闲线路检测，  
- 如果WAKE=1，则进行地址标记检测。  
#### 空闲线路检测（WAKE=0）  
当MMRQ位被置1，RWU被自动置1，USART进入静默模式。  
检测到一个空闲帧时，会被唤醒。RWU位由硬件清零，但是USAR_ISR中的IDLE位不会被置1。如图233。  
![](https://i.imgur.com/9fKeXZt.png)  
注：如果在空闲字符已经过去后才将MMRQ置1，则不会进入静默模式（RWU没被置1）。  
如果在线路空闲时激活USART，在经过一个空闲帧的持续时间后会检测到空闲状态（而不仅仅只在接收到一个字符帧之后）。  
#### 4位或7位地址标记检测（WAKE=1）  
在该模式中，如果字节的MSB位是1，则被认为是地址；否则就是数据。在地址字节中，目标接收器的地址放在低4位或低7位中（由ADDM7位选择）。4位或7位的地址和，在USART_CR2中ADD位中设置的接收器地址，进行比较。  
注：在7位或9位数据模式中，地址检测分别按6位或8位地址（ADD[5:0]或ADD[7:0]）进行。  
当接收器接收到的地址和自身地址不匹配时，该USART进入静默模式。这种情况下，RWU位由硬件置1。当USART进入静默模式，RXNE不会针对该地址字节置1，也不会产生中断或DMA请求。  
当接收的地址和自身的地址匹配时，USART退出静默模式。然后，RWU位被清零，随后的字节正常接收。一旦RWU位被清零，RXNE位会针对接收的地址字节被置1。  
![](https://i.imgur.com/EeyOD8a.png)  
### USART奇偶校验控制  
将USART_CR1中的PCE位置1，使能奇偶校验控制（发送时生成奇偶校验位，接收时进行奇偶校验检查）。根据M位定义的帧长度，USART可能的帧格式在表90中列出。  
![](https://i.imgur.com/ygy9OPG.png)  
#### 偶校验  
一帧的低7位或8位（取决于M位）中1的个数是偶数还是奇数；如果是偶数，校验位就是0，否则为1；这样**<font color=green>整个帧中1的个数总是偶数</font>**。  
例如，数据00110101有4个1，如果选择偶校验（USART_CR1中的PS=0），则校验位为0。  
#### 奇校验  
一帧的低7位或8位（取决于M位）中1的个数是偶数还是奇数；如果是偶数，校验位就是1，否则为0；这样**<font color=green>整个帧中1的个数总是奇数</font>**。  
例如，数据00110101有4个1，如果选择奇校验（USART_CR1中的PS=1），则校验位为1。  
#### 接收进行奇偶校验检查  
如果奇偶校验检查失败，USART_ISR中的PE标志置1，并且如果USART_CR1中的PEIE位置1，还会产生中断。PE位由软件将USART_ICR中的PECF位置1，而清除。  
#### 发送生成奇偶校验位  
如果USART_CR1中的PCE位被置1，则数据寄存器中的数据的MSB位被奇偶校验位替换后发送（PS=0选择偶校验，要有偶数个1；PS=1选择奇校验，有奇数个1）。  
### USART同步模式  
将USART_CR2中CLKEN位置1，选择同步模式。在同步模式下，下列位必须保持为0：  
- USART_CR3中的SCEN、HDSEL和IREN位。  
在此模式下，主模式下的USART可以控制双向同步串行通信。CK引脚是USART发送器的时钟输出。在起始位和停止位期间，CK引脚不会有时钟脉冲。USART_CR2中的LBCL位，决定在最后一位有效数据位（地址标记）期间，是否输出时钟脉冲。USART_CR2中的CPOL位，选择时钟的极性。USART_CR2中的CPHA位，选择外部时钟的相位（参加图235，图236和237）。  
空闲状态期间、序文和发送断开字符时，外部CK时钟被禁止。  
在同步模式下USART发送器的工作方式和在异步模式下完全相同。但由于CK与TX同步（通过CPOL和CPHA），所以TX上的数据是同步的。  
同步模式下USART接收器的工作方式和异步模式下不同。如果RE=1，数据在CK脉冲时被采样（上升沿还是下降沿，取决于CPOL和CPHA），而不是进行任何过采样。但必须确保建立和保持时间（取决于波特率：1/16位持续时间）。  
注：CK引脚是同TX引脚结合工作的。因此，只有当发送器被使能（TE=1），并且正在发送数据时（已写入USART_TDR），才会输出时钟。也就是说，在没有发送数据的情况下，不可能接收同步数据。  
LBCL、CPOL和CPHA位必须在USART未使能（UE=0）时设置，以确保时钟脉冲功能正确。  
###### USART synchronous mode code example  

	/* (1) Oversampling by 16, 9600 baud */
	/* (2) Synchronous mode
		   CPOL and CPHA = 0 => rising first edge
	       Last bit clock pulse
	       Most significant bit first in transmit/receive */
	/* (3) 8 data bit, 1 start bit, 1 stop bit, no parity
	       Transmission enabled, reception enabled */
	USART1->BRR = 480000 / 96; /* (1) */
	USART1->CR2 = USART_CR2_MSBFIRST | USART_CR2_CLKEN | USART_CR2_LBCL; /* (2) */
	USART1->CR1 = USART_CR1_TE | USART_CR1_RXNEIE | USART_CR1_RE | USART_CR1_UE; /* (3) */
	/* Polling idle frame Transmission w/o clock */
	while ((USART1->ISR & USART_ISR_TC) != USART_ISR_TC)
	{
		/* add time out here for a robust application */
	}
	USART1->ICR |= USART_ICR_TCCF; /* Clear TC flag */
	USART1->CR1 |= USART_CR1_TCIE; /* Enable TC interrupt */  
![](https://i.imgur.com/TUU2olx.png)  
![](https://i.imgur.com/z8DHA5j.png)  
![](https://i.imgur.com/qCo7YUn.png)  
![](https://i.imgur.com/XJKfHO6.png)  
### USART单线半双工通信  
USART_CR3中的HDSEL位置1，选择单线半双工模式。在该模式下，下列位必须保持为0：  
- USART_CR2中的CLKEN位，  
- USART_CR3中的SCEN和IREN位。  
在单线半双工模式下，USART的TX和RX在内部是连接在一起的。  
一旦HDSEL被置1：  
- TX和RX在内部连接在一起  
- RX引脚不再使用  
- 当无数据发送时，TX引脚被释放。因此，在空闲或接收时，TX引脚可以作为标准I/O。这意味着，必须把该I/O配置成复用功能TX，开漏并外接上拉。  
除此之外，通信协议与正常USART模式相同。线路上的任何冲突必须通过软件进行管理（例如，使用中央仲裁器）。尤其要注意，当TE位被置1后，发送永远不会被硬件阻止，只要数据写入数据寄存器，就持续发送。  
### USART使用DMA进行连续通信  
USART可以使用DMA进行连续通信。Rx缓冲区和Tx缓冲区的DMA请求时独立产生的。  
###### USART DMA code example  

    /* (1) Oversampling by 16, 9600 baud */
    /* (2) Enable DMA in reception and transmission */
    /* (3) 8 data bit, 1 start bit, 1 stop bit, no parity, reception and
    transmission enabled */
    USART1->BRR = 480000 / 96; /* (1) */
    USART1->CR3 = USART_CR3_DMAT | USART_CR3_DMAR; /* (2) */
    USART1->CR1 = USART_CR1_TE | USART_CR1_RE | USART_CR1_UE; /* (3) */
    /* Polling idle frame Transmission */
    while ((USART1->ISR & USART_ISR_TC) != USART_ISR_TC)
    {
        /* add time out here for a robust application */
    }
    USART1->ICR |= USART_ICR_TCCF; /* Clear TC flag */
    USART1->CR1 |= USART_CR1_TCIE; /* Enable TC interrupt */  
#### 使用DMA进行发送  
将USART_CR3中的DMAT位置1，使能利用DMA发送。每当TXE位置1时，数据从DMA外设配置的SRAM区域载入到USART_TDR寄存器中。按以下步骤，配置一个DMA通道用于USART发送（x表示通道号）：  
1. 将USART_TDR寄存器的地址写入DMA控制寄存器，作为DMA传输的目标地址。每个TXE事件后，数据从内存中移入此地址中。  
2. 将内存地址写入DMA控制寄存器，作为DMA传输的源地址。每次TXE事件后，数据从此内存区写入USART_TDR寄存器中。  
3. 在DMA控制寄存器中，配置传输总字节数。  
4. 配置DMA通道优先级。  
5. 根据需求，配置在传输完成一半/全部时，产生DMA中断。  
6. 将USART_ICR中的TCCF位置1，清除USART_ISR中的TC标志。  
7. 激活该DMA通道。  
当达到DMA控制器设置的传输数据数量时，DMA控制器在该DMA通道中断向量上产生一个中断。  
在发送模式下，一旦DMA发送完所有数据（DMA_ISR中的TCIF标志置1），可以监测TC标志，以确保USART通信完成。这是必要的，以避免破坏关闭USART或进入停机模式之前的最后一个传输，必须等待TC=1。TC标志在所有数据发送期间保持为0，并在最后一帧发送结束后，由硬件置1。  
![](https://i.imgur.com/NcNPaKm.png)  
#### 使用DMA进行接收  
将USART_CR3中的DMAR位置1，使能利用DMA接收。每收到一个字节，数据就从USART_RDR寄存器中转移到DMA外设配置的SRAM区域。按以下步骤，为USART接收，配置一个DMA通道：  
1. 将USART_RDR寄存器的地址写入DMA控制寄存器，作为DMA传输的源地址。每次RXNE事件后，数据从此地址转移到内存中。  
2. 将内存地址写入DMA控制寄存器，作为DMA传输的目标地址。每次RXNE事件后，数据从USART_RDR写入该内存区域。  
3. 在DMA控制寄存器中，配置传输总字节数。  
4. 在DMA控制寄存器中，配置通道优先级。  
5. 根据需求，配置传输一半/全部时，产生DMA中断。  
6. 激活该DMA通道。  
当达到DMA控制器设置的传输数据总数时，DMA控制器在该DMA通道中断向量上产生一个中断。  
![](https://i.imgur.com/64qGSZm.png)  
#### 多缓冲器通信中的错误标志和中断产生  
在多缓冲器通信中，在传输期间发生任何错误，都会在当前字节传输完成后，将错误标志置1。如果相应的中断使能位为1，就会产生中断。在一个字节接收过程中，发生的帧错误、溢出错误和噪声标志会和RXNE一同置1，这些错误有一个单独的错误中断使能位（USART_CR3中的EIE位）；如果该中断使能位置1，发生这些错误中的任何一个，都会产生中断。  
### USART实现RS232硬件流控制和RS485驱动器使能  
利用CTS输入和RTS输出，可以控制2个器件之间的串行数据流。  
![](https://i.imgur.com/FQPG7kI.png)  
将USART_CR3中的RTSE和CTSE位置1，分别使能RS232的RTS和CTS流控制。  
#### RS232 RTS流控制  
如果RTS流控制被使能（RTSE=1），一旦USART接收器准备好接收新数据，RTS就生效，变为低电平。当接收寄存器满时，RTS失效变为高电平，指示希望当前帧结束后停止发送。  
![](https://i.imgur.com/TIdt66D.png)  
#### RS232 CTS流控制  
如果CTS流控制被使能（CTSE=1），发送器在发送下一帧之前会检查CTS输入。如果CTS是有效低电平，则发送下一个数据（假设数据已准备好发送，换句话说TXE=0）；否则不会发送。如果发送期间CTS变为无效的高电平，当前帧发送完后，发送器则停止发送。  
当CTSE=1时，CTSIF状态位跟随CTS输入变化，由硬件自动设置；它指示接收器是否准备好接收数据。如果USART_CR3中的CTSIE位置1，则产生中断。  
![](https://i.imgur.com/8kCmeLx.png)  
注：为了正确运行，CTS必须在当前帧结束前至少3个USART时钟源周期之前变为有效低电平。另外要注意，短于2个PCLK周期的脉冲可能无法置位CTSCF标志。  
###### USART hardware flow control code example  

    /* (1) oversampling by 16, 9600 baud */
    /* (2) RTS and CTS enabled */
    /* (3) 8 data bit, 1 start bit, 1 stop bit, no parity, reception and transmission enabled */
    USART1->BRR = 480000 / 96; /* (1) */
    USART1->CR3 = USART_CR3_RTSE | USART_CR3_CTSE; /* (2) */
    USART1->CR1 = USART_CR1_TE | USART_CR1_RXNEIE | USART_CR1_RE | USART_CR1_UE; /* (3) */
    /* Polling idle frame Transmission */
    while ((USART1->ISR & USART_ISR_TC) != USART_ISR_TC)
    {
        /* add time out here for a robust application */
    }
    USART1->ICR |= USART_ICR_TCCF; /* Clear TC flag */
    USART1->CR1 |= USART_CR1_TCIE; /* Enable TC interrupt */  
#### RS485驱动使能  
将USART_CR3中的DEM位置1，使能驱动器使能功能。这样用户可以使用DE（驱动使能）信号激活外部收发器控制。使能时间是从DE信号有效到起始位之间的时间，它通过USART_CR1中的DEAT[4:0]位设置。禁止时间是从发送消息的最后一个停止位结束到DE信号无效之间的时间，它通过USART_CR1中的DEDT[4:0]位设置。DE信号的极性可以通过USART_CR3中的DEP位配置。  
在USART中，DEAT和DEDT以采样时间单位表示（根据过采样率，单位为1/8或1/16位持续时间）。  
## USART低功耗模式  
![](https://i.imgur.com/2S9W2C8.png)  
## USART中断  
![](https://i.imgur.com/9NLZImd.png)  
USART中断事件都连接到同一个中断向量（见图244）。  
- 发送期间：发送完成、CTS、发送数据寄存器空中断。  
- 接收期间：空闲检测、溢出错误、接收数据寄存器非空、校验错误、噪声标志、帧错误、字符匹配等中断。  
只有相应的中断使能位置1，这些事件才会产生中断。  
![](https://i.imgur.com/2Ipc7rG.png)  
## USART寄存器  
### 控制寄存器1（USART_CR1）  
![](https://i.imgur.com/ewVOn8M.png)  
![](https://i.imgur.com/naWIVDD.png)  
![](https://i.imgur.com/rXp7V3Q.png)  
![](https://i.imgur.com/zXVAJRf.png)  
![](https://i.imgur.com/L5K3zID.png)   
![](https://i.imgur.com/yenQFw8.png)  
