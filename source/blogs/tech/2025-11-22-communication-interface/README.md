# 芯片通讯接口
## SPI (Serial Peripheral Interface)
SPI 使用的通讯信号线有四条，分别是 ：
- **MOSI（主设备输出/从设备输入）**：主设备发送数据到从设备的线路。
- **MISO（主设备输入/从设备输出）**：从设备发送数据到主设备的线路。
- **SCK（时钟信号）**：由主设备生成的时钟信号，用于同步数据传输。
- **SS（片选信号）**：用于选择特定的从设备，通常是低电平有效信号。

SPI 采用主从模式（Master-Slave），一个 SPI 系统中只能有一个主设备，但可以有多个从设备。主设备通过提供时钟信号和片选信号来控制从设备。数据在主从设备之间是**全双工**传输，即在同一时钟周期内，主设备和从设备可以同时发送和接收数据。SPI 的传输速率通常可以达到几 Mbps，具体取决于设备的支持能力。

一个芯片的 SPI 通讯流程（该芯片为主设备）：
1. 控制 SS 片选信号，选择设备为有效。
2. 设置主设备的时钟频率。
3. 设置主设备 SPI 寄存器。
4. 将发送的数据写入 MOSI 引脚。

以 LPC2103 为例：
```C
#include<LPC21xx.H>
#define false 0
#define true  1

void setIO0DIR(int index, int isSet){
	if (isSet) { IO0DIR = 0b1 << index; } else { IO0DIR = 0b0 << index; }
}
void setGPIOStatu(int index, int isSet){
	if (isSet) { IO0SET = 0b1 << index; } 
	else if (!isSet){ IO0CLR = 0b1 << index; }
}
void Delay(){ for(int i=0; i<1000; i++) for(int i=0; i<4000; i++) ; }

int main(void){
	// 1. set SPI PIN. P0-P3 work as GPIO, P4-P7 work as SPI Pin.
	PINSEL0 = 0b0101010100000000;
	setIO0DIR(3, true);   
	
	// 2. set SPI CLK. A SPI cycle is 100 clock cycles.
	S0SPCCR = 100;
	
	// 3. set SPI register. Set according to LPC21xx Manual.
	S0SPCR = 0b01100000;
	
	// 4. input data. 
	S0SPDR = 0xff;
	
	// 5. set 74HC595 RCK
	setGPIOStatu(3, false);
	Delay();
	setGPIOStatu(3, true);
	
	return 0;
}
```

芯片连接方式如下：
![[SPI-One-chip.png]]
当程序执行到 `4. input data.` 之后，在SCK的不断驱动下，数据被写入 74HC595 的 DS 引脚。

### CPOL 和 CPHA

时钟极性(CPOL)定义了时钟空闲状态电平：
- **CPOL=0**，表示当SCLK=0时处于空闲态，所以有效状态就是SCLK处于高电平时
- **CPOL=1**，表示当SCLK=1时处于空闲态，所以有效状态就是SCLK处于低电平时

时钟相位(CPHA)定义数据的采集时间：
- **CPHA=0**，在时钟的第一个跳变沿（上升沿或下降沿）进行数据采样。，在第2个边沿发送数据
- **CPHA=1**，在时钟的第二个跳变沿（上升沿或下降沿）进行数据采样。，在第1个边沿发送数据

SPI 时序：
![[SPI-timing.jpg]]

两两组合共有四种工作模式：

| SPI模式 | CPOL | CPHA | 空闲时 SCK 时钟 |   采样时刻   |
| :---: | :--: | :--: | :--------: | :------: |
|   0   |  0   |  0   |    低电平     | 第一个边沿（奇） |
|   1   |  0   |  1   |    低电平     | 第二个边沿（偶） |
|   2   |  1   |  0   |    高电平     | 第一个边沿（奇） |
|   3   |  1   |  1   |    高电平     | 第二个边沿（偶） |
