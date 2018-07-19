# Serial peripheral interface串行外设接口(SPI)  
## 简介  
SPI接口可以使用SPI协议与外部设备进行通信。SPI模式由软件选择。在器件复位后默认选择SPI Motorola模式。  
串行外设接口（SPI）协议支持与外部器件的半双工、全双工和单工同步串行通信。接口可以配置为主模式，向外部从器件提供通信时钟（SCK）。该接口也可在多主机配置下工作。  
## SPI主要特性  
- 主/从操作  
- 三线全双工同步传输  
- 两线半双工同步传输（使用双向数据线）  
- 两线单工同步传输（使用单向数据线）  
- 4到16位数据大小，可选择  
- 多主模式能力  
- 8个主模式波特率预分频器，高达f<sub>PCLK</sub>/2  
- 从模式频率高达f<sub>PCLK</sub>/2  
- 主或从模式下，NSS可由硬件或软件管理：主/从操作的动态变化  
- 可编程的时钟极性和相位  
- 可编程的数据移位顺序：MSB在前或LSB在前  
- 专门的发送和接收标志，可以触发中断  
- SPI总线忙状态标志  
- 支持SPI Motorola模式  
- 确保可靠通信的硬件CRC功能：  
　- 在Tx模式，最后一个字节发送CRC值  
　- 接收方，对最后一个接收的字节，自动进行CRC错误校验  
- 主模式故障、溢出标志，可以触发中断  
- CRC错误标志  
- 2个支持DMA的内置32位Rx和Tx FIFO  
- 支持SPI TI模式  
## SPI实现  
![](https://i.imgur.com/npTsjwl.png)  
## SPI功能描述  
### 一般说明  
SPI支持MCU和外设之间的同步串行通信。应用软件可以通过查询状态标志或专门的SPI中断，来管理通信。下图显示了SPI主要的部件和它们之间的相互作用。  
![](https://i.imgur.com/xB99se9.png)  
4个引脚用于于外设进行SPI通信。  
- MISO：主输入/从输出数据。通常，该引脚用于在从模式下发送数据，而在主模式下接收数据。  
- MOSI：主输出/从输入数据。通常，该引脚用于在主模式下发送数据，而在从模式下接收数据。  
- SCK：SPI主器件的串行时钟输出，SPI从器件的串行时钟输入。  
- NSS：从器件选择引脚。根据SPI和NSS的设置，该引脚用于：  
　- 选中一个从器件来通信  
　- 使数据帧同步  
　- 检测多个主器件间的冲突  
SPI支持一个主器件和多个从器件之间通信。总线至少要2条线：一个时钟信号线，一个同步传输数据线。其它信号根据SPI节点之间的数据交换和它们的从器件选择信号的管理来添加。  
### 一主一从之间通信  
SPI支持MCU根据不同的目标器件和应用需求，使用不同的配置进行通信。这些配置使用2或3条线（软件NSS管理），也可以使用3或4条线（硬件NSS管理）。通信总是由主器件发起。  
#### 全双工通信  
默认情况下，SPI配置为全双工通信。主器件和从器件的移位寄存器通过各自MOSI引脚之间和MISO引脚之间的2条单向线连接在一起。在SPI通信过程中，数据在主器件提供的时钟SCK的边沿同步移位。主器件通过MOSI线向从器件发送数据，通过MISO线接收从器件发来的数据。当数据帧传输完成（所有位都已移出），主器件和从器件之间完成信息交换。  
![](https://i.imgur.com/j2imCTC.png)  
1. NSS引脚可以用于在主器件和从器件之间提供硬件控制流。另外，也可以不使用NSS引脚，此时，需要在主器件和从器件内部处理控制流。  
#### 半双工通信  
通过将SPIx_CR1寄存器中的BIDIMODE位置1，可以将SPI配置为半双工通信。通过一条交叉线将主器件和从器件的移位寄存器连接起来。在通信过程中，数据在SCK时钟的边沿同步移位，传输方向由各自的SPIx_CR1寄存器中的BDIOE位设置。此模式下，主器件的MISO引脚和从器件的MOSI引脚没有使用，可以作为GPIO使用。  
![](https://i.imgur.com/u5XjHTs.png)  
1. NSS引脚可以用于在主器件和从器件之间提供硬件控制流。另外，也可以不使用NSS引脚，此时，需要在主器件和从器件内部处理控制流。  
2. 主器件的MISO引脚和从器件的MOSI引脚，可以用作GPIO。  
3. 当2个节点之间的通信方向不同步改变时，会发生严重情况。新的发送器访问数据线，而此时旧的发送器还在线上保持相反的值（该值取决于SPI配置和通信数据）。当2个节点在同一线路上输出相反的电平，会造成冲突，直到一个节点改变自己的方向（设置为反方向）。建议在主器件的MOSI和从器件的MISO引脚之间串接一个电阻，以保护输出和限制它们之间的电流。  
#### 单工通信  
通过SPIx_CR2寄存器中的RXONLY位选择只发送或只接收，将SPI配置为单工通信。通过一条线将主器件和从器件的移位寄存器连接起来。通信不使用的MISO和MOSI引脚，可以用作GPIO。  
- 只发送（RXONLY=0）：配置设置和全双工一样。忽略未用引脚上的输入，该引脚可以配置为GPIO使用。  
- 只接收（RXONLY=1）：禁止SPI输出。从模式下，MISO输出被禁止，该引脚可以用作GPIO。当从器件选择信号有效时，从器件从MOSI引脚接收数据。接收数据事件根据数据缓冲区的配置产生。在主模式下，MOSI输出禁止，该引脚可以配置为GPIO。时钟信号只要SPI使能，就会一直产生。要停止时钟输出，只有将RXONLY位或SPE位清零，并等待MISO引脚上的输入完成，根据数据缓冲区的配置，将其填满。  
![](https://i.imgur.com/cIzcG1Q.png)  
1. NSS引脚可以用于在主器件和从器件之间提供硬件控制流。另外，也可以不使用NSS引脚，此时，需要在主器件和从器件内部处理控制流。  
2. 输入移位寄存器中偶尔捕获的数据，在只发送模式下，必须被丢弃（例如，OVF标志）。  
3. 主器件和从器件的MISO引脚可以当作GPIO使用。  
注：单工通信可以用方向固定的半双工通信替代（BDIO位保持不变）。  
### 标准的多从器件通信  
在有2个或更多独立的从器件时，主器件使用多个GPIO引脚作为每个从器件的选择引脚。主器件将和从器件NSS输入相连的GPIO拉低，来选择该从器件。此时，一个标准的主器件和从器件的通信就建立起来了。  
![](https://i.imgur.com/r9my9rj.png)  
1. 主器件的NSS引脚不使用。它需要在内部管理（SSM=1，SSI=1）以避免MODF错误。  
2. 因为所有从器件的MISO连接在一起，它们的MISO引脚的GPIO配置都要配置为复用功能开漏。  
### 多主器件通信  
除非SPI总线不支持多主器件，否则，用户可以使用内部功能来检测2个节点同时试图掌控总线造成的冲突。为此，NSS引脚要配置为输入。  
这种模式下，连接的SPI节点不能超过2个，因为某一时刻只能有一个节点在一个公共数据线上输出数据。  
当2个节点都未激活时，默认处于从模式。一旦一个节点想要控制总线，它就切换到主模式，并通过GPIO引脚向另一个节点的从模式选择输入引脚，输出有效电平。通信完成后，从器件选择信号被释放，处于主模式的节点恢复到从模式，等待下一次通信。  
如果2个节点同时要求控制总线，就会产生冲突（参见模式故障MODF事件）。然后，用户可以应用一些简单仲裁程序（例如，通过在两个节点上应用预定义的不同超时来推迟下一次尝试）。  
![](https://i.imgur.com/Z0Bf61S.png)  
1. 2个节点的NSS引脚都要配置为输入。NSS上的有效电平会使能MISO线的输出控制，当被动节点配置为从器件时。  
### 从器件选择（NSS）引脚管理  
在从模式下，NSS作为片选输入，使从器件和主器件通信。在主模式下，NSS可以用作输出或输入。作为输入，它可以防止多主模式总线冲突；作为输出，它可以驱动单个从器件的从器件选择信号。  
使用硬件或软件管理从器件选择，由SPIx_CR1中的SSM位选择：  
- 软件NSS管理（SSM=1）：由SPIx_CR1中的SSI位的值来选择从器件。NSS引脚可以作其他用。  
- 硬件NSS管理（SSM=0）：有2种可能的配置：取决于NSS输出配置（SPIx_CR1中的SSOE位）。  
　- NSS输出使能（SSM=0，SSOE=1）：主器件才能使用此配置。NSS引脚由硬件管理。主器件使能SPI（SPE=1），NSS就输出低电平，并保持低电平直到关闭SPI（SPE=0）。如果NSS脉冲模式打开（NSSP=1），在连续通信期间会产生脉冲。多主器件不能用此配置。  
　- NSS输出禁止（SSM=0，SSOE=0）：多主器件通信可以使用此配置。拉低NSS，SPI将进入主模式故障状态并自动重新配置为从模式。从模式下，NSS作为片选输入，当其为低电平时，选中该从器件。  
![](https://i.imgur.com/XAw8IXK.png)  
### 通信格式  
在SPI通信中，接收和发送同时执行。串行时钟（SCK）使数据线上的移位和采样同步。通信格式取决于时钟相位、时钟极性和数据帧格式。为了能够通信，主器件和从器件必须遵循相同的通信格式。  
#### 时钟相位和极性控制  
使用SPIx_CR1中的CPOL和CPHA位，可以选择4种可能的时序关系。CPOL（时钟极性）控制当无数据传输时，空闲状态的时钟值。该位对主/从模式都有用。如果CPOL=0，SCK引脚在空闲状态输出低电平；CPOL=1，SCK引脚在空闲状态输出高电平。  
如果CPHA=1，则在时钟的第二边沿捕获传输的第一个数据位（如果CPOL=0，该边沿为下降沿；CPOL=1，为上升沿）。每次出现该时钟边沿时，数据被锁存。如果CPHA=0，SCK引脚上的第一个边沿捕获传输的第一个数据位（CPOL=0，该边沿为上升沿；CPOL=1，为下降沿）。每次出现该时钟边沿时，数据被锁存。  
CPOL（时钟极性）和CPHA（时钟相位）共同决定数据捕获的时钟边沿。  
图252显示了在SPI全双工传输时，CPHA和CPOL的4种组合。  
注：在改变CPOL/CPHA位之前，要先将SPE位清零来关闭SPI。  
SCK的空闲状态必须符合SPIx_CR1中的极性选择（CPOL=1，拉高SCK；CPOL=0，拉低SCK）。  
![](https://i.imgur.com/hqRwEoi.png)  
#### 数据帧格式  
LSBFIRST位的值，决定了SPI移位寄存器移出数据MSB在前或LSB在前。数据帧的长度由DS位选择，可以设置为4到16位长，对接收和发送都有效。无论数据帧长度如何，对FIFO的读访问必须与FRXTH水平对齐。当访问SPIx_DR寄存器时，数据帧总是按字节（数据不超过一个字节）或半字右对齐。通信工程中，只有数据帧内的位才会提供时钟和传输。  
![](https://i.imgur.com/2bFuxWs.png)  
注：数据长度最小为4。如果选择的数据长度小于4，会强制数据帧长度为8位。  
### 配置SPI  
主模式和从模式的配置流程几乎相同。标准通信的初始化，执行下列步骤：  
1. 配置相关的GPIO寄存器：将相应的GPIO配置为MOSI、MISO和SCK引脚。  
2. 配置SPI_CR1寄存器：  
　a) 通过BR[2:0]位配置串行时钟的波特率（注：4）。  
　b) 配置CPOL和CPHA位（在NSSP模式下，CPHA必须清零）（注：2-除了TI模式下CRC使能的情况）。  
　c) 配置RXONLY位选择单工模式；配置BIDIMODE和BIDIOE位选择半双工模式（RXONLY和BIDIMODE不能同时置1）。  
　d) 配置LSBFIRST位，定义帧格式（注：2）。  
　e) 如果需要CRC，配置CRCL和CRCEN位（当SCK时钟信号在空闲状态时）。  
　f) 配置SSM和SSI（注：2和3）。  
　g) 配置MSTR位（在多主机NSS配置中，如果主控器配置为防止MODF错误，则避免NSS上的冲突状态）。  
3. 配置SPI_CR2寄存器：  
　a) 配置DS[3:0]位，选择传输的数据帧长度。  
　b) 配置SSOE位（注：1、2和3）。  
　c) 如果使用TI模式，将FRF位置1（保持NSSP位为0）。  
　d) 如果2个数据单元之间需要NSS脉冲，将NSSP位置1（在NSSP模式，保持CHPA和TI位为0）。  
　e) 配置FRXTH位。RXFIFO阈值必须和SPIx_DR寄存器的读访问大小相同。  
　f) 如果在封装模式下使用DMA，初始化LDMA_TX和LDMA_RX位。  
4. 配置SPI_CRCPR寄存器：配置CRC多项式，如果需要。  
5. 配置相应的DMA寄存器：配置SPI Tx和Rx专用的DMA数据流，如果使用DMA数据流。  
注：（1）从器件不需要配置该步骤。  
　　（2）TI模式不需要配置该步骤。  
　　（3）NSSP模式不需要配置该步骤。  
　　（4）除了从器件工作于TI模式外，从器件不需要配置该步骤。  
###### SPI master configuration code example  

    /* (1) Master selection, BR: Fpclk/256 (due to C27 on the board, SPI_CLK is
           set to the minimum) CPOL and CPHA at zero (rising first edge) */
    /* (2) Slave select output enabled, RXNE IT, 8-bit Rx fifo */
    /* (3) Enable SPI1 */
    SPI1->CR1 = SPI_CR1_MSTR | SPI_CR1_BR; /* (1) */
    SPI1->CR2 = SPI_CR2_SSOE | SPI_CR2_RXNEIE | SPI_CR2_FRXTH
              | SPI_CR2_DS_2 | SPI_CR2_DS_1 | SPI_CR2_DS_0; /* (2) */
    SPI1->CR1 |= SPI_CR1_SPE; /* (3) */    
###### SPI slave configuration code example  

    /* nSS hard, slave, CPOL and CPHA at zero (rising first edge) */
    /* (1) RXNE IT, 8-bit Rx fifo */
    /* (2) Enable SPI2 */
    SPI2->CR2 = SPI_CR2_RXNEIE | SPI_CR2_FRXTH 
              | SPI_CR2_DS_2 | SPI_CR2_DS_1 | SPI_CR2_DS_0; /* (1) */
    SPI2->CR1 |= SPI_CR1_SPE; /* (2) */  
### SPI使能步骤  
建议在主器件发送时钟前，使能SPI从器件。否则，可能发生不希望的数据传输。在和主器件开始通信前（在通信时钟的第一个边沿之前，或，如果时钟信号连续，则在正进行的通信结束前），从器件要发送的数据必须已经存入了数据寄存器中。在SPI从器件使能之前，SCK信号必须保持稳定在所选的空闲状态电平。  
当SPI使能，且TXFIFO不为空或向TXFIFO写入下一个数据时，配置为全双工或只发模式的主器件开始通信。  
配置为只收模式（RXONLY=1，或BIDIMODE=1并且BIDIOE=0）的主器件，在SPI使能后，马上开始通信，同时时钟开始运行。  
### 数据发送和接收过程  
#### RXFIFO和TXFIFO  
所有的SPI数据交换都是通过内置的FIFO进行的。这使得SPI可以连续工作，并防止数据帧较短时发生溢出。每个方向都有自己的FIFO被称为TXFIFO和RXFIFO。这些FIFO用于除使能CRC计算的只收模式（主/从器件）外的所有SPI模式。  
FIFO的处理取决于数据交换的模式（双工、单工），数据帧格式（一帧的位数），FIFO数据寄存器的访问大小（8位或16位），以及是否使用数据包访问FIFO。  
读SPIx_DR寄存器，会返回RXFIFO中还未读取的最早的值。写入SPIx_DR寄存器的新值，会放在TXFIFO中发送队列的尾部。读访问必须和SPIx_CR2中的FRXTH位配置的RXFIFO阈值相一致。FTLVL[1:0]和FRLVL[1:0]会指出2个FIFO的占用水平。  
对SPIx_DR寄存器的读取必须由RXNE事件管理。当有数据存入RXFIFO并且达到了FRXTH位定义的阈值时，该事件触发。当RXNE被清除，RXFIFO被认为空。类似的，待发送数据帧的写访问由TXE事件管理。当TXFIFO中的数据不超过其容量的一半时，该事件触发。否则，TXE被清除，TXFIFO被认为满。当数据帧不超过8位时，RXFIFO最多可以存储4个数据帧，而TXFIFO最多只能存储3个数据帧。这种差异可以防止TXFIFO中已经有3个8位数据帧存在时，软件试图在16位模式下向其中写入更多数据，从而造成已有数据损坏的情况。TXE和RXNE事件可以通过轮询和中断来处理。  
另一种管理数据交换的方式是使用DMA。  
当RXFIFO满时，收到数据，会导致发送溢出事件。溢出事件可以通过查询或中断处理。  
BSY=1表示正在进行数据传输。当时钟信号连续运行时，在主器件中，数据帧之间BSY标志依然保持1；而在从器件中，数据帧之间BSY标志会变为0并保持一个SPI时钟周期的时间。  
#### 序列处理  
可以通过单个序列传递几个数据帧来完成消息。当发送使能后，只要主器件的TXFIFO中存在数据，序列就开始并持续。主器件持续提供时钟直到TXFIFO变为空，然后时钟停止等待另外的数据。  
在只收模式，半双工（BIDIMODE=1，BIDIOE=0）或单工（BIDIMODE=0，RXONLY=1），只要SPI和只收模式被使能，主器件立即开始接收序列。主器件提供时钟直到SPI或只读模式被关闭。在此之前，主器件会连续接收数据帧。  
主器件在连续模式下工作，必须要考虑到从器件处理数据的能力。必要时，主器件必须降低通信速度，提供更慢的时钟，或是在帧或数据段之间加入足够的延时。注意，在SPI模式下没有主器件或从器件的下溢错误信号，来自从器件的数据总是由主器件发起接收并处理，即使从器件不能正确及时将数据准备好。所以，从器件最好使用DMA，尤其当数据帧较短且总线速率较高时。  
在多从器件系统中，每个序列都要由NSS脉冲控制，以只选中其中一个从器件来进行通信。在单个从器件系统中，不需要NSS来选择从器件；但是，提供NSS脉冲更好，可以将从器件和每个数据序列的开始进行同步。NSS可以有软件或硬件进行管理。  
当BSY=1时，表示有数据帧传输正在进行。当一帧传输完成，RXNE标志被置1。最后一位采样后，完整的数据帧会被存入RXFIFO中。  
#### 关闭SPI的步骤  
要关闭SPI，必须按照本段中介绍的步骤进行操作。当外设时钟停止，系统进入低功耗模式之前，按步操作很重要；否则，正在进行的传输可能被破环。在某些模式下，按照步骤关闭SPI是停止连续通信的唯一方法。  
处于全双工或只发模式的主器件可以停止发送数据，以结束传输。这种情况下，时钟在最后一个数据传输后停止。必须特别注意在封装模式下，当奇数个数据帧被传输以防止一些虚拟字节交换时。在这些模式下关闭SPI之前，用户必须遵守标准的关闭步骤。当关闭SPI时，主器件正发送数据或下一个数据帧已经存入TXFIFO，将无法保证SPI的行为。  
当主器件处于只收模式时，停止连续时钟的唯一方法是SPE=0关闭SPI。必须在传输的最后一帧数据的第一位采样与最后一位传输开始之前的时间段内，关闭SPI（为了接收完整的有效字节，并避免接收有效字节后接收到任何空数据）。这种模式下，必须执行特定的步骤来关闭SPI。  
关闭SPI后，已接收但还未读取的数据会依然保留在RXFIFO中，下次打开SPI，开始新的序列之前，要处理掉这些数据。要确保没有未读取的数据，可以执行正确的关闭操作来保证关闭SPI时RXFIFO为空，或者通过控制外设复位的特定寄存器用软件复位的方式初始化所有SPI寄存器（参见RCC_APBiRSTR寄存器中的SPIiRST位）。  
标准的关闭步骤是基于检查BSY状态和FTLVL[1:0]来判断传输是否完全结束。当需要识别正在进行的传输结束时，也可以进行这种检查，例如：  
- 当NSS信号由软件管理，主器件需要向从器件提供正确的NSS脉冲结束时，或  
- 当来自DMA或FIFO的传输流完成，而最后一个数据帧或CRC帧还在总线上传输时。  
正确的关闭步骤是（除使用只收模式外）：  
1. 等待FTLVL[1:0]=00（没有数据要发送）  
2. 等待BSY=0（最后一帧数据已经处理完）  
3. 关闭SPI（SPE=0）  
4. 读取数据，直到FRLVL[1:0]=00（读取所有已接收的数据）  
某些只收模式的正确关闭步骤：  
1. 在接收最后一个数据帧的特定时间窗口内关闭SPI（SPE=0）  
2. 等待BSY=0（最后一帧数据已经处理完）  
3. 读取数据，直到FRLVL[1:0]=00（读取所有接收到的数据）  
注：如果使用封装模式，并且要接收奇数个帧大小≤8bits的数据帧，FRXTH必须在FRLVL[1:0]=01时设为1，以产生RXNE事件从而读取最后的奇数帧并保持FIFO指针对齐。  
#### 数据封装  
当数据帧大小适合一个字节（≤8bits）时，对SPIx_DR寄存器的16-bit读写，会自动将数据打包起来。这种情况，将并行处理2个数据帧。SPI先对低8位进行操作，然后是高8位。图254给出了数据打包模式下数据处理的例子。对发送器的SPIx_DR一次16-bit写访问，将发送2个数据帧。如果接收器的RXFIFO阈值设为16位（FRXTH=0），这2个数据帧只产生一个RXNE事件。作为对这次RXNE事件的响应，接收器必须按16-bit读取SPIx_DR来一次读取这2个数据帧。在接收器端，RXFIFO阈值的设置必须和后续SPIx_DR的读取位数保持一致，否则，会丢失数据。  
如果要处理奇数个此类“不超过一个字节”的数据帧，会出现特定的问题。在发送端，只需用8-bit的访问方式将最后一帧写入SPIx_DR。而，接收端需要改变RXFIFO阈值，以在最后一帧接收后产生RXNE事件。  
![](https://i.imgur.com/Qx0WVCY.png)  
#### 使用DMA进行通信  
为了以最大速度工作，并提高读写速度以避免溢出，SPI提供了DMA功能，该功能采用简单的请求/应答协议。  
当SPIx_CR2寄存器中的TXE或RXNE位置1时，发出DMA请求。向TX和RX缓冲区发送的请求是独立的。  
- 发送时，每次TXE=1发出DMA请求。然后DMA向SPIx_DR寄存器写入数据。  
- 接收时，每次RXNE=1时发出DMA请求。然后DMA从SPIx_DR寄存器中读取数据。  
见图255到图258。  
当SPI只用来发送数据时，可以只使能SPI TX DMA通道。这种情况下，OVR标志被置1，因为不会读取接收数据。当SPI只用来接收数据时，可以只使能SPI RX DMA通道。  
在发送模式下，当DMA已将所有要发送的数据写入SPIx_DR（DMA_ISR中的TCIF标志置1），可以监视BSY标志以确保SPI通信完成。在关闭SPI或进入停机模式之前，要求避免破环最后的传输。软件首先等待FTLVL[1:0]=00然后等待BSY=0。  
当使用DMA进行通信时，要防止DMA通道管理引起错误事件，必须遵循下面的步骤：  
1. 如果使用DMA RX，将SPI_CR2中的RXDMAEN位置1，使能DMA RX缓冲区。  
2. 如果使用数据流，在DMA寄存器中使能TX和RX的DMA数据流。  
3. 如果使用DMA TX，将SPI_CR2中的TXDMAEN位置1，使能DMA TX缓冲区。  
4. SPE位置1，使能SPI。  
###### SPI master configuration with DMA code example  

    /* (1) Master selection, BR: Fpclk/256 
           (due to C27 on the board, SPI_CLK is set to the minimum)
           CPOL and CPHA at zero (rising first edge) */
    /* (2) TX and RX with DMA,
           enable slave select output,
           enable RXNE interrupt,
           select 8-bit Rx fifo */
    /* (3) Enable SPI1 */
    SPI1->CR1 = SPI_CR1_MSTR | SPI_CR1_BR; /* (1) */
    SPI1->CR2 = SPI_CR2_TXDMAEN | SPI_CR2_RXDMAEN | SPI_CR2_SSOE
              | SPI_CR2_RXNEIE | SPI_CR2_FRXTH
              | SPI_CR2_DS_2 | SPI_CR2_DS_1 | SPI_CR2_DS_0; /* (2) */
    SPI1->CR1 |= SPI_CR1_SPE; /* (3) */  
###### SPI slave configuration with DMA code example  

    /* nSS hard, slave, CPOL and CPHA at zero (rising first edge) */
    /* (1) Select TX and RX with DMA,
           enable RXNE interrupt,
           select 8-bit Rx fifo */
    /* (2) Enable SPI2 */
    SPI2->CR2 = SPI_CR2_TXDMAEN | SPI_CR2_RXDMAEN
              | SPI_CR2_RXNEIE | SPI_CR2_FRXTH
              | SPI_CR2_DS_2 | SPI_CR2_DS_1 | SPI_CR2_DS_0; /* (1) */
    SPI2->CR1 |= SPI_CR1_SPE; /* (2) */  
要关闭通信，必须按顺序执行下面的步骤：  
1. 如果使用数据流，在DMA寄存器中关闭TX和RX的DMA数据流。  
2. 按照关闭SPI的步骤，关闭SPI。  
3. 如果使用DMA TX和或DMA RX，将SPI_CR2中的TXDMAEN和或RXDMAEN位清零，关闭DMA TX和或DMA RX缓冲区。  
#### 使用DMA时的数据封装  
如果传输由DMA管理（SPIx_CR2中的TXDMAEN=1和RXDMAEN=1），根据为SPI TX和SPI RX DMA通道配置的PSIZE值，自动打开/关闭封装模式。如果DMA通道的PSIZE值＝16-bit，而SPI数据帧长度≤8-bit，那么封装模式使能。DMA自动管理对SPIx_DR寄存器的写操作。  
如果使用数据封装，但传输的数据量不是偶数，那么LDMA_TX/LDMA_RX位必须置1。SPI就认为最后一次DMA传输只需发送或接收一个数据。  
#### 通信图表  
本部分介绍一些典型的时序图。无论SPI事件通过轮询、中断或DMA来处理，这些时序图都是有效的。为了简单起见，这里均假设LSBFIRST=0、CPOL=0和CPHA=1。不提供完整的DMA流配置。  
下面带编号的注释适用于图255到图258。  
1. 当NSS被激活且SPI被使能后，从器件开始控制MISO线；当其中一条不成立时，从器件就断开与MISO线的连接。必须为从器件提供足够的时间，以在开始传输前准备好发给主器件的数据。  
对于主器件，只要SPI被使能，SPI外设就开始控制MOSI和SCK信号（偶尔也控制NSS信号）。一旦SPI被关闭，SPI外设就与GPIO逻辑断开联系，这些引脚上的电平只取决于GPIO的设置。  
2. 对于主器件，如果通信（时钟信号）连续，在帧与帧之间BSY依然保持置1。对于从器件，在帧与帧之间BSY至少要保持1个时钟周期的低电平。  
3. 只有TXFIFO满，才能清除TXE。  
4. DMA仲裁程序在TXDMAEN位置1后开始。TXE中断在TXEIE=1时才会产生。当TXE=1时，开始向TXFIFO写入数据，直到TXFIFO变满或DMA传输完成。  
5. 如果要发送的所有数据可以一次放入TXFIFO，那么在SPI总线通信开始前DMA TX TCIF标志就置1了。该标志在SPI发送完成前会一直保持为1。  
6. 一个数据包的CRC值是在SPIx_TXCRCR和SPIx_RXCRCR中逐帧连续计算的。CRC信息在整个数据包完成后进行处理，由DMA自动处理（TX通道必须设置为要处理的数据帧个数），或由软件处理（用户必须在处理最后一帧数据期间处理CRCNEXT位）。  
当SPIx_TXCRCR中计算的CRC值由发送器发出，接收的CRC信息会被载入RXFIFO中，然后与SPIx_RXCRCR中的内容进行比较（如果不同，CRC错误标志会置1）。因此用户必须注意刷新FIFO中的内容，通过软件读出RXFIFO中存储的所有数据，或者，当RX通道已设置适当的数据帧个数（数据帧个数+CRC帧个数），也可通过DMA刷新（参见假设示例的设置）。  
7. 在数据封装模式，TXE和RXNE事件成对出现，每次对FIFO的读写都是16-bit宽，直到数据帧的个数为偶数。如果TXFIFO是3/4满，FTLVL将保持FIFO满时的状态。因此在TXFIFO变为1/2满之前，最后的奇数帧不能被存储。当LDMA_TX=1时，该帧可以由软件或DMA按8-bit宽写入TXFIFO。  
8. 要在封装模式下接收最后的奇数帧，当在处理最后一个数据帧时，必须把RX阈值改为8-bit，通过软件设置FRXTH=1，或当LDMA_RX=1时由DMA内部信号自动将其改变。  
![](https://i.imgur.com/eZGniPD.png)  
主器件全双工通信示例假设：  
- 数据长度＞8bit  
如果使用DMA：  
- DMA传输的TX帧个数设为3  
- DMA传输的RX帧个数设为3  
通用假设和注释，参见上面的内容。  
![](https://i.imgur.com/yx0MzBu.png)  
从器件全双工通信示例假设：  
- 数据长度＞8bit  
如果使用DMA：  
- DMA传输的TX帧个数设为3  
- DMA传输的RX帧个数设为3  
通用假设和注释，参见上面的内容。  
###### SPI full duplex communication code example  

    if ((SPI1->SR & SPI_SR_TXE) == SPI_SR_TXE) /* Test Tx empty */
    {
        /* Will inititiate 8-bit transmission if TXE */
        *(uint8_t *)&(SPI1->DR) = SPI1_DATA;
    }  
![](https://i.imgur.com/pjjtUsf.png)  
主器件全双工带CRC通信示例假设：  
- 数据长度＞8bit  
- CRC使能  
如果使用DMA：  
- DMA传输的TX帧个数设为2  
- DMA传输的RX帧个数设为3  
通用假设和注释，参见上面的内容。  
![](https://i.imgur.com/dFHVOLa.png)  
主器件在封装模式下全双工通信示例假设：  
- 数据长度＝5bit  
- 读写FIFO主要按16-bit宽进行  
- FRXTH=0  
如果使用DMA：  
- DMA传输的TX帧个数设为3  
- DMA传输的RX帧个数设为3  
- TX和RX的DMA通道的PSIZE都设为16-bit  
- LDMA_TX=1和LDMA_RX=1  
通用假设和注释，参见上面的内容。  
### SPI状态标志  
应用程序可以通过3个状态标志来监视SPI总线的全部状态。  
#### TX缓冲器空标志（TXE）  
当发送TXFIFO有足够空间来存储要发送的数据时，TXE标志置1。TXE标志与TXFIFO占用率相关。该标志变为1并保持，直到TXFIFO的存储水平≤FIFO深度的1/2。如果SPIx_CR2中的TXEIE=1，将产生中断。当TXFIFO的存储水平＞FIFO深度的1/2时，TXE标志自动清零。  
#### RX缓冲器非空（RXNE）  
RXNE标志根据SPIx_CR2中的FRXTH位的值进行设置：  
- 如果FRXTH=1，当RXFIFO存储水平≥FIFO深度的1/4（8bit）时，RXNE标志置1。  
- 如果FRXTH=0，当RXFIFO存储水平≥FIFO深度的1/2（16bit）时，RXNE标志置1。  
如果SPIx_CR2中的位RXNEIE=1，将产生中断。  
当RXFIFO的存储水平不满足条件时，RXNE被硬件自动清零。  
#### 忙标志（BSY）  
BSY标志由硬件置位和清零（对此标志进行写操作，无效）。  
BSY=1，表示SPI正在传输数据（SPI总线忙）。  
某些模式下可以通过BSY标志查看传输是否结束，以便在进入低功耗模式前，软件关闭SPI或它的外设时钟，而不破环最后的传输。  
BSY标志也可以用于在多主器件模式中防止写冲突。  
下列条件满足任一个，BSY标志将被清零：  
- 正确关闭SPI  
- 在主模式下检测到故障（MODF=1）  
- 主模式下，当完成数据传输并且没有新的数据准备发送时  
- 从模式下，在每次数据传输之间BSY标志清零至少一个SPI时钟周期  
注：当主器件可以立即处理下一次传输时（例如，主器件只接收，或者它的TXFIFO非空），通信将是连续的，并且BSY标志在传输间一直保持为1。虽然从器件不是这样，但是建议总是用TXE和RXNE（而不是BSY）标志来处理发送或接收操作。  
### SPI错误标志  
当下面任一个错误标志置1并且ERRIE位也置1时，将产生中断。  
#### 溢出标志（OVR）  
当主器件或从器件接收到数据而RXFIFO没有足够的空间存放该接收的数据时，发生溢出。如果软件或DMA没有足够的时间去读出先前接收到数据（存储在RXFIFO中），就会出现这种情况。当数据存储空间受到限制时，也可能发生这种情况；例如只接收模式下使能了CRC导致RXFIFO不能使用，此时接收缓冲区被限制只有一个数据帧大小。  
发送溢出时，新接收到的值不会把RXFIFO中存储的先前值覆盖。新接收到值以及它之后的所有数据，将丢失。先读SPI_DR寄存器，然后读SPI_SR寄存器，将清除OVR标志。  
#### 模式故障（MODF）  
当主器件内部的NSS信号（NSS硬件模式下是NSS引脚，NSS软件模式下是SSI位）被拉低时，发生模式故障。自动将MODF位置1。主器件模式故障影响SPI以下几个方面：  
- MODF=1，如果ERRIE=1，将产生中断  
- 将SPE位清零，阻止器件的所有输出，并关闭SPI接口  
- 将MSTR位清零，强制器件进入从模式  
按照以下步骤将MODF位清零：  
1. MODF=1时，对SPIx_SR寄存器进行读或写  
2. 然后，写SPIx_CR1寄存器  
为了避免多从器件发生冲突，必须在MODF位清零期间拉高NSS引脚。MODF清零后，SPE和MSTR位恢复原始状态。安全起见，在MODF=1时，硬件不允许将SPE和MSTR位置1。在从器件中，MODF位不可能被置1，除非是先前多主冲突造成并遗留下来的。  
#### CRC错误（CRCERR）  
当SPIx_CR1中的CRCEN位置1时，该标志用来检查接收数据的准确性。如果接收的CRC和SPIx_RXCRCR寄存器中计算的CRC值不同，SPIx_SR寄存器中的CRCERR标志就将置1。该标志由软件清零。  
#### TI模式帧格式错误（FRE）  
如果SPI配置为从模式，并且要符合TI模式协议，那么在通信进行期间，出现NSS脉冲，就会造成TI模式帧格式错误。错误产生时，SPIx_SR中的FRE标志置1。该错误发生，不会关闭SPI，但会忽略该NSS脉冲，然后SPI等待下一个NSS脉冲再开始新的传输。自检测到错误开始可能会丢失2个字节数据，造成数据损坏。  
读SPIx_SR寄存器，FRE标志会被清零。如果ERRIE=1，在检测到NSS错误时，会产生中断。这种情况下，应该关闭SPI，因为数据的完整性无法保证，并在从器件重新使能SPI后，主器件重新开启通信。  
### NSS脉冲模式  
将SPIx_CR2中的NSSP位置1，可以启用该模式。但是该模式只有在SPI接口配置为Motorola SPI主器件（FRF=0）并在时钟第一个边沿捕获（CPHA=0）时，才起作用。此模式下，在2次连续数据帧传输之间会产生一个NSS脉冲，并保持NSS高电平至少1个时钟周期。该模式允许从器件锁存数据。NSSP脉冲模式设计用于单个主从器件的应用。  
![](https://i.imgur.com/T2w6BF2.png)  
注：当CPOL=0时情况类似：采样发生在SCK的上升沿，NSS的采样也发生在这些边沿。  
### TI模式  
#### 主模式下的TI协议  
SPI接口兼容TI协议。SPIx_CR2中的FRF位置1，配置SPI使用TI协议。  
无论SPIx_CR1中的设置如何，时钟极性和相位被强制符合TI协议。这种情况下，NSS管理也要符合TI协议，设置SPIx_CR1和SPIx_CR2寄存器（SSM,SSI,SSOE）不能配置NSS。  
在从模式下，SPI波特率预分频器用于控制当传输完成时MISO引脚状态切换为高阻态的时刻。可以使用任何波特率，可以很灵活地确定该时刻。但是，波特率通常设置为外部主器件的时钟波特率。MISO变为高阻态的延时（t<sub>release</sub>）取决于内部重新同步和SPIx_CR1中BR[2:0]位设置的波特率。公式如下：  
t<sub>baud_rate</sub>/2 + 4 × t<sub>pclk</sub> ＜ t<sub>release</sub> ＜ t<sub>baud_rate</sub>/2 + 6 × t<sub>pclk</sub>  
如果从器件在数据帧传输期间检测到错位的NSS脉冲，FRE标志会被置1。  
如果数据长度等于4-bit或5-bit，工作在全双工或只发送模式下的主器件使用在LSB后面增加一个空数据位的协议。将在这个空位而不是LSB的时钟周期内，产生NSS脉冲。  
此特性不适用于Motorola SPI通信（FRF=0）。  
![](https://i.imgur.com/CryR2lR.png)  
### CRC计算  
2个独立的CRC计算器分别用于检查发送数据和接收数据的可靠性。SPI提供CRC8或CRC16计算，与帧的数据宽度设置为8-bit或16-bit无关；但是对于其他帧宽度，CRC不适用。  
#### CRC原理  
在SPI使能（SPE=1）之前，将SPIx_CR1寄存器中的CRCEN位置1，使能CRC计算。CRC值由一个值为奇数的可编程多项式对每一位进行计算得到。在SPIx_CR1中的CPHA和CPOL位定义的采样时钟边沿进行计算。计算得到的CRC值在由CPU或DMA管理传输的数据块末尾自动被检查。当根据接收数据计算出的CRC值和发送来的CRC不匹配时，CRCERR标志置1，指出数据损坏。CRC计算的正确步骤取决于SPI的配置和所选的传输管理。  
注：多项式的值只能是奇数，不支持偶数。  
#### CPU管理CRC传输  
通信开始和持续如常，直到最后的数据帧被发送或接收。CRCNEXT必须在最后一帧数据传输完成之前置1，以便在最后一帧数据传输完后发送或接收CRC帧。在CRC传输期间，CRC计算被冻结。  
接收的CRC以字节或字的形式存放在RXFIFO中。这就是在CRC模式下，接收缓冲区被视为一个16-bit缓冲区，一次只能接收一个数据帧的原因。  
通常在数据序列的末尾再传输一帧CRC帧。但是，当数据帧长度设为8-bit而做16-bit CRC校验时，则需要2帧才能完整发送CRC。  
当最后的CRC数据被接收后，自动与SPIx_RXCRC寄存器中的值进行比较。软件需要检查SPIx_SR中的CRCERR标志以确定接收的数据是否损坏。向CRCERR标志写0，可以将其清除。  
接收CRC后，CRC存入RXFIFO，必须读取SPIx_DR寄存器，以清除RXNE标志。  
#### DMA管理CRC传输  
当带CRC和DMA模式的SPI通信被使能，在通信的结尾自动发送和接收CRC（在只收模式下CRC的接收除外）。CRCNEXT位不必再由软件处理。SPI发送的DMA通道计数器设置的要发送数据帧的个数不能包括CRC帧。在接收端，接收的CRC值在传输结束时由DMA自动处理，但是用户必须注意刷除RXFIFO中的CRC，因为CRC总是被载入RXFIFO中。在全双工模式中，接收的DMA通道的计数器设置的要接收的数据帧个数必须包括CRC帧，这意味着，在数据帧长度为8-bit而CRC为16-bit时：  
DMA_RX=Numb_of_data + 2   
在只收模式，DMA接收通道计数器应该只包含传输数据的数量，而不包括CRC。在DMA传输完成后，软件从FIFO中读取CRC值，因为在此模式下FIFO作为单个缓冲区。  
如果在传输过程中出现损坏，数据和CRC传输完后，SPIx_SR寄存器中的CRCERR标志被置1。  
如果使用封装模式，当要传输的数据个数是奇数时，需要用LDMA_RX位来管理。  
#### 重置SPIx_TXCRC和SPIx_RXCRC的值  
SPIx_TXCRC和SPIx_RXCRC的值在CRC阶段后对新数据采样时，会被自动清零。这就允许使用DMA循环模式（只收模式除外）不间断的传输数据（一些数据块会被中间进行的CRC校验阶段覆盖）。  
要在通信过程中关闭SPI，必须遵循以下步骤：  
1. 关闭SPI  
2. CRCEN位清零  
3. CRCEN位置1  
4. 使能SPI  
注：当SPI处于从模式时，只要CRCEN位被置1，且SCK引脚上有输入时钟，CRC计数器就会工作，而不关心SPE位的值如何。为了避免任何错误的CRC计算，软件必须等时钟稳定后再使能CRC计算。  
当SPI接口被配置为从模式，一旦CRCNEXT信号被释放，在CRC传输阶段NSS内部信号要保持低电平。这就是CRC计算在从模式NSS脉冲模式下不能使用的原因。  
在TI模式，尽管时钟相位和极性的设置是固定的且和SPIx_CR1寄存器无关，但是如果要使用CRC，必须在SPIx_CR1寄存器中设置CPOL=0和CPHA=1。另外，在主器件和从器件关闭SPI重新使能CRCEN期间，CRC计算要重置，否则CRC计算会损坏。  
## SPI中断  
在SPI通信期间，下列事件会导致中断产生：  
- TXFIFO准备好装入数据，TXE=1  
- RXFIFO接收到数据，RXNE=1  
- 主器件模式故障，MODF=1  
- 溢出错误，OVR=1  
- TI帧格式错误，FRE=1  
- CRC错误，CRCERR=1  
中断可以单独使能和禁止。  
![](https://i.imgur.com/zqTJBWC.png)  
###### SPI interrupt code example  

	if ((SPI1->SR & SPI_SR_RXNE) == SPI_SR_RXNE)
	{
		SPI1_Data = (uint8_t)SPI1->DR; /* receive data, clear flag */
		/* Process */
	}  
## SPI寄存器  
### SPI控制寄存器1（SPIx_CR1）  
![](https://i.imgur.com/FvuJZ6y.png)  
![](https://i.imgur.com/367JB5t.png)  
![](https://i.imgur.com/834FQdn.png)  
![](https://i.imgur.com/lVGi6yB.png)  
![](https://i.imgur.com/R8HYZpi.png)  
### SPI控制寄存器2（SPIx_CR2）  
![](https://i.imgur.com/ZYXfomP.png)  
![](https://i.imgur.com/BR6O6jt.png)  
![](https://i.imgur.com/w7pgs1F.png)  
![](https://i.imgur.com/4OjRSHV.png)  
![](https://i.imgur.com/bDJ6Xyu.png)  
### SPI状态寄存器（SPIx_SR）  
![](https://i.imgur.com/g22wwco.png)  
![](https://i.imgur.com/qRUqFad.png)  
![](https://i.imgur.com/hnYUKHy.png)  
### SPI数据寄存器（SPIx_DR）  
![](https://i.imgur.com/Von9Nvc.png)  
### SPI CRC多项式寄存器（SPIx_CRCPR）  
![](https://i.imgur.com/7L2ecpl.png)  
### SPI Rx CRC寄存器（SPIx_RXCRCR）  
![](https://i.imgur.com/LaNskSZ.png)  
### SPI Tx CRC寄存器（SPIx_TXCRCR）  
![](https://i.imgur.com/kLdWksX.png)  
## SPI寄存器映射  
![](https://i.imgur.com/qF62ASZ.png)  
