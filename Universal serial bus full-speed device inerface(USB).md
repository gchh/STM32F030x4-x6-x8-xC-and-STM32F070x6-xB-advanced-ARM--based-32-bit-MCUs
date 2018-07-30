# Universal serial bus full-speed device interface(USB)  
该部分仅针对STM32F070x6和STM32F070xB。  
## 简介  
USB外设在全速USB2.0总线和APB总线之间提供一个接口。  
支持USB暂停/恢复，以停止设备时钟减少功耗。  
## USB主要特征  
- 遵循USB 2.0全速设备规范  
- 可配置1到8个USB端点(endpoint)  
- 1024字节的专用数据包缓冲SRAM  
- CRC产生/检查，反转不归0（NRZI）编码/解码，位填充（bit-stuffing）  
- 支持同步传输  
- 支持批量/同步端点的双缓冲  
- USB暂停/恢复操作  
- 帧锁定时钟脉冲产生  
- 支持USB 2.0链路层电源管理  
- 支持电池充电规范版本1.2  
- 控制USB连接/断开（USB_DP线上内置可控上拉电阻）  
## USB实现  
![](https://i.imgur.com/WK8JfOT.png)  
## USB功能描述  
![](https://i.imgur.com/BKszalT.png)  
USB外设在PC主机和MCU实现的功能之间提供了一个符合USB规范的连接。PC主机和MCU之间的数据传输，通过USB外设可直接访问的一个专用的数据包缓冲存储区进行。这个专用的存储区大小有1024个字节，有多达16个单向或8个双向端点可供使用。USB外设按照USB标准要求：连接USB主机，检测令牌包，处理数据收发，处理握手包。事务（Transaction）格式化由硬件执行，包括CRC产生和校验。  
每个端点都有一个缓冲区描述块，描述该端点所用的内存区地址、大小和传输的字节数。