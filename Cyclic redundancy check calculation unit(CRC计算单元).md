##Cyclic redundancy check calculation unit CRC计算单元  
CRC计算单元根据生成多项式，计算出输入数据的循环冗余校验码。  
CRC校验通常用于检查传输或存储数据的完整性。  
###CRC主要特性  
- 采用CRC-32(以太网标准)多项式：0x4C11DB7  
X<sup>32</sup>+X<sup>26</sup>+X<sup>23</sup>+X<sup>22</sup>+X<sup>16</sup>+X<sup>12</sup>+X<sup>11</sup>+X<sup>10</sup>+X<sup>8</sup>+X<sup>7</sup>+X<sup>5</sup>+X<sup>4</sup>+X<sup>2</sup>+X+1  
- 可以处理8bit,16bit和32bit数据  
- 可编程的CRC初始值  
- 一个32bit数据输入/输出寄存器  
- 输入缓冲避免在计算过程中造成总线阻塞  
- 对于32bit的数据,CRC计算在4个AHB时钟周期（HCLK）内完成  
- 可用于临时存储的通用8bit寄存器  
- 输入输出数据高低位可颠倒  

###CRC功能描述  
![](https://i.imgur.com/d87By7C.png)  
![](https://i.imgur.com/oqx2jgL.png)  

####CRC操作  
CRC计算单元有一个32位的可读写寄存器(CRC_DR)，它用来输入新的数据（写操作），以及输出上次CRC计算结果（读操作）。  
每次写操作，CRC计算单元都会将新写入的值和存储的上次CRC计算结果进行综合计算，得到新的计算结果。CRC计算单元根据写入的数据格式不同来决定是一个32bit整字还是一个字节一个字节的计算。  
CRC_DR寄存器可以按字，右对齐的半字或右对齐的字节访问。而其他寄存器只能按字访问。  
不同宽度的数据需要不同的计算时间：  
- 32-bit的字需要4个AHB时钟周期  
- 16-bit的右对齐半字需要2个AHB时钟周期  
- 8-bit的右对齐字节需要1个AHB时钟周期  
数据缓冲区允许不等待前次CRC计算结束就可以写入数据。  
可以根据给定数据的字节数，动态调整数据长度，以得到最小的写入次数。例如，计算一个5字节的数据，可以先计算一个字，然后再计算一个字节。  
输入数据高地位可以颠倒，以适应不同的大小端体系。对CRC_CR的REV_IN[1:0]这2位进行设置，决定数据颠倒按8-bit，16-bit还是32-bit进行。  
例如，输入数据0x1A2B3C4D：  
- 按字节颠倒得到0x58D43CB2  
- 按半字颠倒得到0xD458B23C  
- 按字颠倒得到0xB23CD458  
输出数据也可以通过设置CRC_CR的REV_OUT位来颠倒，改操是按位进行的：例如，输出数据0x11223344颠倒后就是0x22CC4488。（像是按字颠倒）  
通过CRC_CR的RESET位可以将CRC计算器的初值（默认是0xFFFFFFFF）初始化为程序在CRC_INIT寄存器中设定的值；在初始化时，CRC_INIT的值自动更新到CRC_DR中。  
CRC_IDR寄存器可以用来保存CRC计算的临时结果，它不受CRC_CR的RESET位影响。  

###CRC registers CRC寄存器  
####DATA Register(CRC_DR) 数据寄存器  
![](https://i.imgur.com/kuRvWMi.png)  

####Independent data register(CRC_IDR) 独立数据寄存器  
![](https://i.imgur.com/JLLjFdp.png)  

####Control register(CRC_CR) 控制寄存器  
![](https://i.imgur.com/njMBetR.png)  

####Initial CRC value(CRC_INIT) 初值寄存器  
![](https://i.imgur.com/BLt5aTP.png)  

###CRC register map  
![](https://i.imgur.com/3CgYV5c.png)  

