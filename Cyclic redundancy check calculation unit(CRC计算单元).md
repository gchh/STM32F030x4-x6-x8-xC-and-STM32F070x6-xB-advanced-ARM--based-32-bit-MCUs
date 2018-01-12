##Cyclic redundancy check calculation unit CRC计算单元  
CRC计算单元根据生成多项式，计算出输入数据的循环冗余校验码。  
CRC校验通常用于检查传输或存储数据的完整性。  
###CRC主要特性  
采用CRC-32(以太网标准)多项式：0x4C11DB7  
X<sup>32</sup>+X<sup>26</sup>+X<sup>23</sup>+X<sup>22</sup>+X<sup>16</sup>+X<sup>12</sup>+X<sup>11</sup>+X<sup>10</sup>+X<sup>8</sup>+X<sup>7</sup>+X<sup>5</sup>+X<sup>4</sup>+X<sup>2</sup>+X+1  
可以处理8bit,16bit和32bit数据  
可编程的CRC初始值  
一个32bit数据输入/输出寄存器  
输入缓冲避免在计算过程中造成总线阻塞  
对于32bit的数据,CRC计算在4个AHB时钟周期（HCLK）内完成  
可用于临时存储的通用8bit寄存器  
输入输出数据可反转  

###CRC功能描述  
![](https://i.imgur.com/d87By7C.png)  
![](https://i.imgur.com/oqx2jgL.png)  

####CRC操作  
CRC计算单元有一个32位的可读写寄存器(CRC_DR)，它用来输入新的数据（写操作），以及输出上次CRC计算结果（读操作）。  
每次写操作，CRC计算单元都会将新写入的值和存储的上次CRC计算结果进行综合计算，得到新的计算结果。CRC计算单元根据写入的数据格式不同来决定是一个32bit整字还是一字节一字节的计算。  
