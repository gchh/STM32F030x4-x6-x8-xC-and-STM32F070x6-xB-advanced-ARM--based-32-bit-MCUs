##Option bytes
一个32bit的字分割成如下的格式写入选项字节：  
![](https://i.imgur.com/1ImRPeZ.png)  

选项字节的组织结构：  
![](https://i.imgur.com/b6zxAMJ.png)  

可以从上表列出的内存地址或FLASH_OBR选项字节寄存器和FLASH_WRPR来获知选项字节的内容。  
The new programmed option byte (user, read/write protection) are not loaded after a system reset. To reload them, either a POR or setting to '1' the OBL_LAUNCH bit is necessary.  
在系统复位后，新编写的选项字节(user,read/write protection)将不会装载到FLASH_OBR和FLASH_WRPR寄存器中，需要上电复位或OBL_LAUNCH置位，将新编写的选项字节载入寄存器中。  
每次上电复位，选项字节装载器（OBL）都会读信息区的数据将其存入FLASH_OBR和FLASH_WRPR寄存器中。在载入选项字节期间，会验证选项字节和它相应的补码选项字节的互补关系，如果失败，FLASH_OBR的OPTERR置1，而且这个选项字节会被认为是0xFF（强制将其值变为0xFF）。如果，选项字节和它的补码选项字节都是0xFF（电擦除状态），则不会产生选项字节错误（OPTERR不会置1）。  

###选项字节描述  
####User and read protection option byte用户及读保护选项字节  
![](https://i.imgur.com/1jeQcUD.png)  
![](https://i.imgur.com/mVqwT3T.png)  
![](https://i.imgur.com/T0U82Mc.png)  

####User data option byte用户数据选项字节
![](https://i.imgur.com/bRA2jfL.png)  

####Write protection option byte写保护选项字节
清WRPx相应的位（同时，对应的nWRPx位置位）将对相应的扇区写保护。  
对于STM32F030x4,STM32F030x6,STM32F030x8,STM32F070x6和STM32F070xB，WRP的0到31bit每一bit保护一个4KB的扇区（共32个扇区，256KB）。  
对于STM32F030xC，WRP的0到30bit每一bit保护一个4KB的扇区（共124KB），31bit保护剩余的132KB。  
![](https://i.imgur.com/pywfYv6.png)  

###Option byte map
![](https://i.imgur.com/nC4KnFk.png)  

