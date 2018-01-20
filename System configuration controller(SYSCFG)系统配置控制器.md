#系统控制器（SYSCFG）  
器件有一组配置寄存器。系统配置控制器的主要目的是：  
- 在一些IO口启用或禁用I<sup>2</sup>C超快模式（Fast Mode Plus）  
- 重映射一些DMA触发源到不同的DMA通道  
- 管理连接到GPIO的外部中断  
- 管理系统可靠性  
##SYSCFG寄存器  
####SYSCFG配置寄存器1（SYSCFG_CFGR1）  
该寄存器用于内存和DMA请求重映射的具体配置，以及控制特殊IO的特性。  
有2位用来配置地址0x0000 0000的内存类型。  
这些位用于由软件择物理映射，因此绕过了硬件启动选择。  
复位后，这些位的值由实际启动模式配置。  
![](https://i.imgur.com/7UvLrLp.png)  
![](https://i.imgur.com/YJT8ZD8.png)  
![](https://i.imgur.com/vFOVGAL.png)  
![](https://i.imgur.com/R0uMNTG.png)  
