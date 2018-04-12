#System window watchdog(WWDG)  
##简介  
系统窗口看门狗（WWDG）用于监测，由外部干扰或意外的逻辑条件造成的应用程序偏离正常运行序列而导致的软件故障。看门狗电路在到达预设的时间周期时，产生一个MCU复位；除非，在控制寄存器的T6位变0之前，更新向下计数器的值。但是，如果在向下计数器计数到窗口寄存器值之前，更新控制寄存器中的7位向下计数器值，也会产生一个MUC中断。这意味着必须在一个限定的时间窗口内更新计数器的值。  
WWDG时钟APB时钟分频后得到，通过一个可配置的时间窗口来监测应用程序不正常的延迟或提前操作。  
WWDG最适用于要求看门狗在一个精确的时间窗口内做出反应的应用。  
##WWDG主要特性  
- 可编程的自由运行向下计数器  
- 复位条件  
　- 当向下计数器的值小于0x40时（如果看门狗已启动）  
　- 当向下计数器在窗口之外被重载时（如果看门狗已启动）  
- 提前唤醒中断（EWI）：如果看门狗已启动并且EWI被使能，当向下计数器的值等于0x40时，产生提前唤醒中断。  
##WWDG功能描述  
设置WWDG_CR中的WDGA=1，启动WWDG。当7位（T[6:0]）向下计数器的值从0x40变为0x3F（T6位变为0）时，产生复位。如果软件在向下计数器的值大于窗口寄存器中的值时重载计数器，同样会产生复位。  
应用程序必须在正常运行过程中定期地写入WWDG_CR，以防产生MCU复位；但是，只能在计数器的值小于窗口寄存器的值而大于0x3F时执行写入。存储在WWDG_CR中的值必须在0xFF和0xC0之间。  
![](https://i.imgur.com/zsnFERW.png)  
###使能看门狗  
复位后，看门狗总是处于关闭状态。设置WWDG_CR中的WDGA=1,使能看门狗，并且除非复位，不然无法关闭。  
###控制向下计数器  
向下计数器是自由运行的，即使看门狗关闭，依然向下计数器。当使能看门狗时，必须将T6置1，防止立即发生复位。  
T[5:0]位包含了在发生看门狗复位之前的计数值。复位前的延时在最小值和最大值之间变化，只是由于在写WWDG_CR寄存器时，预分频器的状态是未知的（见图191）。配置寄存器WWDG_CFG包含窗口的上限：要避免复位，向下计数器必须在其值小于窗口寄存器值并且大于0x3F时被重载。图191描述了WWDG的过程。  
注：软件设置WDGA=1,T6=0，可以产生复位。  
###看门狗中断高级特性  
如果需要在复位之前执行特定的安全操作或数据记录，可以使用提前唤醒中断EWI。设置WWDG_CFR中的EWI=1，使能EWI中断。当向下计数器的值等于0x40时，产生EWI中断，在器件复位之前，可以用相应的中断服务程序（ISR）来触发特定操作（比如，通信或记录数据）。  
在一些应用中，EWI中断被用来管理软件系统检测和/或系统恢复/功能退化，而不产生WWDG复位。这种情况下，相应的中断服务程序（ISR）必须重载WWDG计数器，以避免产生复位，然后触发所需的操作。  
向WWDG_SR写入EWIF=0，可以清除EWI中断。  
注：例如由于系统锁定在更高优先级的任务中，而无法执行EWI中断时，最终会产生WWDG复位。  
####如何设置看门狗超时  