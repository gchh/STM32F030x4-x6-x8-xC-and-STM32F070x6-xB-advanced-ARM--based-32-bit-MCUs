##Flash memory operations FLASH操作  
Flash memory包括两块：main flash memory和information block；  
information block包括system memory和option byte.  
编程和擦除Flash memory必须打开内部高速RC振荡器(HSI)。  
###unlocking the flash memory
复位后，flash memory默认受保护，避免意外的写或擦除，此时FLASH_CR(flash控制寄存器)不可写。需要向FLASH_KEYR(flash关键字寄存器)写关键字KEY1=0x45670123和KEY2=0xCDEF89AB，解锁对flash memory的操作；如果写入的关键字不对，将产生总线错误并引发硬件错误中断。  
可以向FLASH_CR寄存器的LOCK位写1，重新锁定FLASH_CR，恢复对Flah memory的保护。  
######Flash memory unlocking sequence code

	/* (1) Wait till no operation is on going */
	/* (2) Check that the Flash is unlocked */
	/* (3) Perform unlock sequence */
	while ((FLASH->SR & FLASH_SR_BSY) != 0) /* (1) */
	{
	/* For robust implementation, add here time-out management */
	}
	if ((FLASH->CR & FLASH_CR_LOCK) != 0) /* (2) */
	{
	FLASH->KEYR = FLASH_FKEY1; /* (3) */
	FLASH->KEYR = FLASH_FKEY2;
	}  

###main flash memory programming  
主flash编程必须每次写入16bits，否则会发生总线错误产生硬件错误中断。  
步骤：  
1. 检查FLASH_SR的BSY，待BSY=0才能进行下一步  
2. 检查FLASH_CR的LOCK是否是1，LOCK=1，需要首先解锁  
3. 将FLASH_CR的PG（编程开始位）置1，开始编程  
4. 向main flash写半字（16bits）  
5. 读FLASH_SR(flash状态寄存器)中的BSY是否为1，BSY=1表示编程正在进行中，需要等待BSY=0编程结束，才能进行下一步操作  
6. 当编程操作结束，FLASH_SR的EOP位置1，需要软件写1清除  
7. 将PG位清零，LOCK置1  
8. 通过读被编程地址的内容检查编程是否成功  
在编程前，FLASH接口会先读取待编程flash的内容是否已经擦除，如果没有擦除将跳过编程并置位FLASH_SR的PGERR位警告编程发生错误；除非要编程的内容是0，则不管是否擦除都会继续编程而不置位PGERR。  
如果待编程的FLASH被FLASH_WRPR(FLASH写保护寄存器)写保护，则会跳过编程并置位FLASH_SR的WRPRTERR位警告发生写保护错误。  
######Main Flash programming sequence code example

	/* (1) Set the PG bit in the FLASH_CR register to enable programming */
	/* (2) Perform the data write (half-word) at the desired address */
	/* (3) Wait until the BSY bit is reset in the FLASH_SR register */
	/* (4) Check the EOP flag in the FLASH_SR register */
	/* (5) clear it by software by writing it at 1 */
	/* (6) Reset the PG Bit to disable programming */
	FLASH->CR |= FLASH_CR_PG; /* (1) */
	*(__IO uint16_t*)(flash_addr) = data; /* (2) */
	while ((FLASH->SR & FLASH_SR_BSY) != 0) /* (3) */
	{ /* For robust implementation, add here time-out management */
	}
	if ((FLASH->SR & FLASH_SR_EOP) != 0) /* (4) */
	{
	FLASH->SR = FLASH_SR_EOP; /* (5) */
	}
	else
	{
	/* Manage the error cases */
	}
	FLASH->CR &= ~FLASH_CR_PG; /* (6) */

###Flash memory erase  
Flash擦除有两种方法页擦除和整片擦除。  
####Page Erase 页擦除  
步骤：  
1. 通过FLASH_SR的BSY位检查是否有Flash操作正在进行，待其结束，BSY=0  
2. 通过FLASH_CR的LOCK位检查Flash是否锁定，若LOCK=1，要首先解锁  
3. 置位FLASH_CR的PER位，使能页擦除  
4. 将要擦除的页地址写入FLASH_AR（FLASH地址寄存器） 
5. 置位FLASH_CR的STRT位，开始擦除
6. 等待BSY=0，擦除结束  
7. 检查EOP是否为1，擦除成功，并写1清零EOP  
8. 擦除完成，PER位清0，STRT位清0  
9. 通过读擦除页的内容检查该页是否已被擦除  
######Page erase sequence code example

	/* (1) Set the PER bit in the FLASH_CR register to enable page erasing */
	/* (2) Program the FLASH_AR register to select a page to erase */
	/* (3) Set the STRT bit in the FLASH_CR register to start the erasing */
	/* (4) Wait until the BSY bit is reset in the FLASH_SR register */
	/* (5) Check the EOP flag in the FLASH_SR register */
	/* (6) Clear EOP flag by software by writing EOP at 1 */
	/* (7) Reset the PER Bit to disable the page erase */
	FLASH->CR |= FLASH_CR_PER; /* (1) */
	FLASH->AR = page_addr; /* (2) */
	FLASH->CR |= FLASH_CR_STRT; /* (3) */
	while ((FLASH->SR & FLASH_SR_BSY) != 0) /* (4) */
	{
	/* For robust implementation, add here time-out management */
	}
	if ((FLASH->SR & FLASH_SR_EOP) != 0) /* (5) */
	{
	FLASH->SR = FLASH_SR_EOP; /* (6)*/
	}
	else
	{
	/* Manage the error cases */
	}
	FLASH->CR &= ~FLASH_CR_PER; /* (7) */

####Mass Erase 整片擦除  
步骤：  
1. 通过FLASH_SR的BSY位检查是否有Flash操作正在进行，待其结束，BSY=0  
2. 通过FLASH_CR的LOCK位检查Flash是否锁定，若LOCK=1，要首先解锁  
3. 置位FLASH_CR的MER位，使能整片擦除   
4. 置位FLASH_CR的STRT位，开始擦除
5. 等待BSY=0，擦除结束  
6. 检查EOP是否为1，擦除成功，并软件写1清0  
7. 擦除完成，MER位清0，STRT位清0  
8. 通过读Flash的内容检查是否已被擦除  
######Mass erase sequence code example

	/* (1) Set the MER bit in the FLASH_CR register to enable mass erasing */
	/* (2) Set the STRT bit in the FLASH_CR register to start the erasing */
	/* (3) Wait until the BSY bit is reset in the FLASH_SR register */
	/* (4) Check the EOP flag in the FLASH_SR register */
	/* (5) Clear EOP flag by software by writing EOP at 1 */
	/* (6) Reset the PER Bit to disable the mass erase */
	FLASH->CR |= FLASH_CR_MER; /* (1) */
	FLASH->CR |= FLASH_CR_STRT; /* (2) */
	while ((FLASH->SR & FLASH_SR_BSY) != 0) /* (3) */
	{
	/* For robust implementation, add here time-out management */
	}
	if ((FLASH->SR & FLASH_SR_EOP) != 0) /* (4)*/
	{
	FLASH->SR = FLASH_SR_EOP; /* (5) */
	}
	else
	{
	/* Manage the error cases */
	}
	FLASH->CR &= ~FLASH_CR_MER; /* (6) */

###Option byte programming 选项字节编程  
选项字节有16个字节，每2个字节组成正反对，所以使用者只需设置8个字节就行，另外8个字节系统自动填充反码。  
RDP   字节0：  读保护字节，存储对主存储块的读保护设置。 
USER  字节2：  用户字节，配置看门狗、停机、待机。  
Data0 字节4：  数据字节0，由芯片使用者自由使用。  
Data1 字节6：  数据字节1，由芯片使用者自由使用。  
WRP0  字节8：  写保护字节0，存储对主存储块的写保护设置。  
WRP1  字节10： 写保护字节1，存储对主存储块的写保护设置。  
WRP2  字节12： 写保护字节2，存储对主存储块的写保护设置。  
WRP3  字节14： 写保护字节3，存储对主存储块的写保护设置。  

在对选项字节操作（编程/擦除）前，需要将FLASH_CR中的OPTWRE=1；该位不能软件置1，需要通过向FLASH_OPTKEYR(FLASH选项字节关键字寄存器)依次写入KEY1和KEY2，硬件自动置1该位，此后，才能进行选项字节操作。OPTWRE位由软件清零。  

步骤：  
1. 检查FLASH_SR的BSY位，是否有flash操作在进行，等待其结束  
2. 向FLASH_OPTKEYR(FLASH选项字节关键字寄存器)依次写入KEY1和KEY2，解锁FLASH_CR的OPTWRE位，使能选项字节操作  
3. 将FLASH_CR的OPTPG位置1，开始选项字节编程  
4. 写入要编程的半字到指定的地址（使用半字的低字节，并自动计算出低字节的反码作为高字节写入指定地址的下个地址）  
5. 等待BSY=0  
6. 从指定的地址读取内容检查是否正确  
7. 检查EOP位是否为1，并软件写1清0  
8. 将OPTPG清0，OPTWRE清0  
和主FLASH编程一样，待编程的选项字节会先读取它的值，检查它是否已经擦除，如果没有会跳过编程操作并置位FLASH_SR的WRPRTERR位产生一个警告。  
当设置读保护选项字节，将读保护等级改为无保护时，在重新编程读保护选项字节前会先执行整片擦除将整个主FLASH擦除。  
######Option byte programming sequence code example

	/* (1) Set the PG bit in the FLASH_CR register to enable programming */
	/* (2) Perform the data write */
	/* (3) Wait until the BSY bit is reset in the FLASH_SR register */
	/* (4) Check the EOP flag in the FLASH_SR register */
	/* (5) Clear the EOP flag by software by writing it at 1 */
	/* (6) Reset the PG Bit to disable programming */
	FLASH->CR |= FLASH_CR_OPTPG; /* (1) */
	*opt_addr = data; /* (2) */
	while ((FLASH->SR & FLASH_SR_BSY) != 0) /* (3) */
	{
	/* For robust implementation, add here time-out management */
	}
	if ((FLASH->SR & FLASH_SR_EOP) != 0) /* (4) */
	{
	FLASH->SR = FLASH_SR_EOP; /* (5) */
	}
	else
	{
	/* Manage the error cases */
	}
	FLASH->CR &= ~FLASH_CR_OPTPG; /* (6) */  

###Option byte Erase 选项字节擦除  
步骤：  
1. 首先检查FLASH_SR的BSY位，确认是否正在进行FLASH操作，等待其结束  
2. 向FLASH_OPTKEYR(FLASH选项字节关键字寄存器)依次写入KEY1和KEY2，解锁FLASH_CR的OPTWRE位，使能选项字节操作  
3. 将FLASH_CR的OPTER位置位，使能擦除操作  
4. 将FLASH_CR的STRT位置位，开始擦除  
5. 等待BSY=0，擦除结束  
6. 检查EOP是否为1，表示擦除成功  
7. 读取选项字节，检查是否擦除成功  
8. 清OPTER和STRT位，OPTWRE清0  
######Option byte erasing sequence code example

	/* (1) Set the OPTER bit in the FLASH_CR register to enable option byte erasing */
	/* (2) Set the STRT bit in the FLASH_CR register to start the erasing */
	/* (3) Wait until the BSY bit is reset in the FLASH_SR register */
	/* (4) Check the EOP flag in the FLASH_SR register */
	/* (5) Clear EOP flag by software by writing EOP at 1 */
	/* (6) Reset the PER Bit to disable the page erase */
	FLASH->CR |= FLASH_CR_OPTER; /* (1) */
	FLASH->CR |= FLASH_CR_STRT; /* (2) */
	while ((FLASH->SR & FLASH_SR_BSY) != 0) /* (3) */
	{
	/* For robust implementation, add here time-out management */
	}
	if ((FLASH->SR & FLASH_SR_EOP) != 0) /* (4) */
	{
	FLASH->SR = FLASH_SR_EOP; /* (5)*/
	}
	else
	{
	/* Manage the error cases */
	}
	FLASH->CR &= ~FLASH_CR_OPTER; /* (6) */

##Memory protection 存储保护  
读保护防止数据被非法读出，写保护防止数据被非法改写。写保护以扇区（4页）为单位。  
###Read protection 读保护  
通过RDP读保护选择字节设置好所需的保护等级，然后系统复位，重装载RDP选项字节，读保护就生效了。  
![](https://i.imgur.com/cHytKHJ.png)  

- level 0无保护：对主FLASH和选项字节的读，编程和擦除都可以操作   
- level 1读保护：这是RDP默认的保护等级。在用户模式下执行的代码可以对主FLASH和选项字节进行全部操作；在DEBUG时或程序从RAM/boot loader启动，将无法操作主FLASH，读主FLASH将产生总线错误发生硬件错误中断，编程/擦除会使FLASH_SR的PGERR置位  
- level 2无debug：包含level 1的保护功能，此外，CortexMO的debug功能被禁止，因此，SWD调试接口，从RAM或boot loader启动的功能也不能用了。在用户模式下，依然可以对主FLASH进行各种操作；而对选项字节只能进行读和编程操作，而不能擦除。此外，RDP选项字节不能编程，因此，level 2将无法被改变；试图编程RDP，会造成FLASH_SR的保护错误标志WRPRTERR置位并产生中断  
![](https://i.imgur.com/iALr8HM.png)  
可以简单的通过改变RDP的值，使保护等级从level 0变为level 1或level 2；然而，从level 1变为level 0，当RDP被编程为0xAA会立马擦除整片主FLASH。  
为了使改变后的保护等级生效，选项字节必须通过FLASH_CR的OBL_LAUNCH位置1产生一个系统复位重新加载，以使其生效。  

###Write protection 写保护  
写保护以扇区进行保护，通过配置WRPx写保护选项字节，然后置位FLASH_CR的OBL_LAUNCH位使其重载，使写保护生效。  
如果对受保护的扇区编程或擦除，将使FLASH_SR的WRPRTERR位置1。  
写保护解除：  
- 将FLASH_CR的OPTER位置1  
- 清除WRPx，并将FLASH_CR的OBL_LAUNCH置1,产生系统复位，重新载入WRPx，解除写保护  

###option byte write protection 选项字节写保护  
选项字节默认可读，受写保护。需要向FLASH_OPTKEYR依次写入KEY1和KEY2，正确的操作会使FLASH_CR的OPTWRE置位，表示解除写保护。同样，可以通过软件将OPTWRE清零，重新对选项字节进行写保护。  

###Flash interrupts 
![](https://i.imgur.com/Hn5uyUG.png)  

###Flash register description flash寄存器  
flash寄存器必须按32bit（words）访问。  

####Flash access control register(FLASH_ACR)  
![](https://i.imgur.com/tRs4zij.png)  

#####Flash key register(FLASH_KEYR)  
![](https://i.imgur.com/793ZRx9.png)  

#####Flash option key register(FLASH_OPTKEYR)  
![](https://i.imgur.com/ioCb1HP.png)  

#####Flash status register(FLASH_SR)
![](https://i.imgur.com/wA82GjC.png)  

#####Flash control register(FLASH_CR)
![](https://i.imgur.com/JliRiqS.png)
![](https://i.imgur.com/baIYpKM.png)  

#####Flash address register(FLASH_AR)  
![](https://i.imgur.com/2Qz6SlN.png)  

#####Flash Option byte register(FLASH_OBR)
![](https://i.imgur.com/lnrppG5.png)
![](https://i.imgur.com/3AuRUsJ.png)

#####Write protection register(FLASH_WRPR)
![](https://i.imgur.com/WQeUs7v.png)  


#####Flash register map
![](https://i.imgur.com/nkCwv1L.png)