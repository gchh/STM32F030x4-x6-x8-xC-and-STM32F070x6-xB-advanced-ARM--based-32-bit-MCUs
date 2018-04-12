#Independent watchdog(IWDG)  
##简介  
此器件集成了一个看门狗外设，具有安全性高、定时准确和使用灵活的优点。看门狗检测和解决由软件错误造成的故障，并在计数器达到给定的超时值时触发系统复位。  
独立看门狗（IWDG）由专用内部低速时钟（LSI）提供时钟，因此即使主时钟失效，仍能保持工作状态。  
IWDG最适用于那些需要一个在主程序之外独立工作的看门狗，而又对时间精度要求较低的应用。  
##IWDG主要特性  
- 自由运行的向下计数器  
- 时钟由独立RC振荡器提供（可在待机和停机模式下运行）  
- 复位条件  
　- 当向下计数器值小于0x000时（如果看门狗已激活）  
　- 在窗口之外重载向下计数器时（如果看门狗已激活）  
##IWDG功能描述  
###IWDG框图  
![](https://i.imgur.com/9sV4Jh5.png)  
通过向键寄存器IWDG_KR写入0x0000 CCCC，启动独立看门狗，计数器从复位值0xFFF开始向下计数。当计数器计数器到终值0x000时，产生一个复位信号（IWDG复位）。  
当向键寄存器IWDG_KR中写入0x0000 AAAA，IWDG_RLR的值会被重载入计数器，以避免产生IWDG复位。  
###窗口选项  
IWDG也可以用作窗口看门狗，需要在IWDG_WINR寄存器中设置合适的窗口。  
IWDG_WINR寄存器的默认值是0x0000 0FFF，如果不更新此默认值，窗口选项是关闭的。  
窗口值一旦改变，向下计数器重载入IWDG_RLR值，并重新计算产生下次重载的周期。  
####配置IWDG，当窗口选项使能时  
1. 写IWDG_KR=0x0000 CCCC，使能IWDG。  
2. 写IWDG_KR=0x0000 5555，使能寄存器操作。  
3. 向IWDG_PR中写入0到7之间的数，配置IWDG预分频器。  
4. 向IWDG_RLR中写入重载值。  
5. 等待寄存器更新（此时，IWDG_SR=0x0000 0000）。  
6. 配置窗口寄存器IWDG_WINR。这会将IWDG_RLR的值自动更新到计数器中。  
注：当IWDG_SR=0x0000 0000时，写入窗口值才会使计数器更新为RLR的值。  
######IWDG configuration with window code example  

	/* (1) Activate IWDG (not needed if done in option bytes) */
	/* (2) Enable write access to IWDG registers */
	/* (3) Set prescaler by 8 */
	/* (4) Set reload value to have a rollover each 100ms */
	/* (5) Check if flags are reset */
	/* (6) Set a 50ms window, this will refresh the IWDG */
	IWDG->KR = IWDG_START; /* (1) */
	IWDG->KR = IWDG_WRITE_ACCESS; /* (2) */
	IWDG->PR = IWDG_PR_PR_0; /* (3) */
	IWDG->RLR = IWDG_RELOAD; /* (4) */
	while (IWDG->SR) /* (5) */
	{
		/* add time out here for a robust application */
	}
	IWDG->WINR = IWDG_RELOAD >> 1; /* (6) */  
####配置IWDG，当窗口选项禁止时  
1. IWDG_KR=0x0000 CCCC，使能IWDG。  
2. IWDG_KR-0x0000 5555，使能寄存器操作。  
3. IWDG_PR=0~7，配置IWDG预分频器。  
4. 配置IWDG_RLR重载值。  
5. 等待寄存器更新，直到IWDG_SR=0x0000 0000。  
6. IWDG_KR=0x0000 AAAA，将计数器的值更新为IWDG_RLR的值。  
######IWDG configuration code example  

	/* (1) Activate IWDG (not needed if done in option bytes) */
	/* (2) Enable write access to IWDG registers */
	/* (3) Set prescaler by 8 */
	/* (4) Set reload value to have a rollover each 100ms */
	/* (5) Check if flags are reset */
	/* (6) Refresh counter */
	IWDG->KR = IWDG_START; /* (1) */
	IWDG->KR = IWDG_WRITE_ACCESS; /* (2) */
	IWDG->PR = IWDG_PR_PR_0; /* (3) */
	IWDG->RLR = IWDG_RELOAD; /* (4) */
	while (IWDG->SR) /* (5) */
	{
		/* add time out here for a robust application */
	}
	IWDG->KR = IWDG_REFRESH; /* (6) */  
###硬件看门狗  
如果通过器件的选项位使能“硬件看门狗”功能，那么在上电时看门狗将自动使能，并且如果在计数器向下计数到0之前没有向IWDG_KR中写入0x0000 AAAA，或是在窗口内没有重载计数器，将产生复位。  
###在待机和停机模式下的行为  
一旦运行，IWDG将不能被停止。  
###寄存器访问保护  
对IWDG_PR,IWDG_RLR和IWDG_WINR寄存器的写访问操作是受保护的。要修改它们，必须首先向IWDG_KR寄存器中写入0x0000 5555。向IWDG_KR中写入其他值，则会破坏该序列，寄存器写保护会再次生效。这意味着重载操作（IWDG_KR写入0x0000 AAAA）也会启动写保护。  
状态寄存器指示预分频值、向下计数重载值或窗口值是否正在更新。  
###调试模式  
当微控制器进入调试模式（内核停止），IWDG根据DBG模块中的DBG_IWDG_STOP配置位选择继续或停止工作。  
##IWDG寄存器  
###键寄存器（IWDG_KR）  
![](https://i.imgur.com/IBJU5zY.png)  
###预分频器寄存器（IWDG_PR）  
![](https://i.imgur.com/NgxzTrd.png)  
###重载寄存器（IWDG_RLR）  
![](https://i.imgur.com/5g90TF8.png)  
###状态寄存器（IWDG_SR）  
![](https://i.imgur.com/K21gEOu.png)  
![](https://i.imgur.com/BkJhsNv.png)  
###窗口寄存器（IWDG_WINR）  
![](https://i.imgur.com/79buFtu.png)  
##IWDG寄存器映射  
![](https://i.imgur.com/zK9YwM1.png)  
不使用IWDG_WINR，则启动IWDG后，在计数器计数到0之前，都可以重载计数器，避免产生IWDG复位。  
而如果使用IWDG_WINR，则启动IWDG后，需要在计数器的值在WINR和0之间，才能重载计数器，而不产生复位；否则，在计数器的值大于WINR时重载计数器，和计数器计数到0时，都会产生复位。  
