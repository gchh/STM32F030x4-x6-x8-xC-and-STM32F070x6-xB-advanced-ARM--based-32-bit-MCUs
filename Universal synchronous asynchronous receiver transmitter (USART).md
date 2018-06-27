# Universal synchronous asynchronous receiver transmitter (USART)  
## 简介  
通用同步异步收发器（USART）提供了一种灵活的方式，与要求工业标准NRZ异步串行数据格式的外部设备进行全双工数据交换。USART使用可编程波特率发生器，提供很宽范围的波特率。  
它支持同步单向通信和半双工单线通信，以及多处理器通信。它也支持Modem调制解调器流控操作（CTS/RTS）。  
使用DMA（直接存储器存取）配置多个缓冲区，可以实现高速数据通信。  
##USART主要特性  
- 全双工异步通信  
- NRZ标准格式（mark标记/space空格）  
- 配置16倍或8倍过采样，提供速度和时钟