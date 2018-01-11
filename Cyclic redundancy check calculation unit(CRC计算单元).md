##Cyclic redundancy check calculation unit CRC计算单元  
CRC计算单元根据生成多项式，计算出输入数据的循环冗余校验码。  
CRC校验通常用于检查传输或存储数据的完整性。  
###CRC主要特性  
采用CRC-32(以太网标准)多项式：0x4C11DB7  
X<sup>32</sup>+X<sup>26</sup>+X<sup>23</sup>+X<sup>22</sup>+X<sup>16</sup>+X<sup>12</sup>+X<sup>11</sup>+X<sup>10</sup>+X<sup>8</sup>+X<sup>7</sup>+X<sup>5</sup>+X<sup>4</sup>+X<sup>2</sup>+X+1  
可以处理8bit,16bit和32bit数据  
可编程的CRC初始值  
单向32bit数据输入/输出寄存器  
输入缓冲避免在计算过程中造成总线阻塞  
对于32bit的数据,CRC计算在4个AHB时钟周期（HCLK）内完成  
可用于临时存储的通用8bit寄存器  
输入输出数据可反转  

###CRC功能描述  
![](https://i.imgur.com/d87By7C.png)  
![](https://i.imgur.com/oqx2jgL.png)  

