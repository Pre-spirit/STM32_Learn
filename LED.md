# LED硬件需求
### LED基础硬件连接与GPIO配置需求
#### 单一LED：
1、直接连接：LED两端分别连接GPIO和GND，通过GPIO端口来控制LED的亮灭，**LED 必须串联限流电阻，防止过流烧 LED 或 GPIO；GPIO 驱动能力小，所以不能直接驱动大功率 LED。**
2、使用驱动电路：
1）**三极管/MOSFET**：作为电子开关，由GPIO控制其导通与截止，从而控制流过LED的大电流。
2）**继电器**：用于控制交流或大功率直流LED负载，实现电气隔离。GPIO通过驱动三极管来间接控制继电器线圈。

#### 多个LED
1、共阳/共阴极连接：将多个LED的正极或者负极连接到一起，通过GPIO来控制各LED的亮灭。
- 共阴：LED 阴极接 GND，阳极经电阻接 GPIO，**高电平点亮**。
- 共阳：LED 阳极接 VCC，阴极经电阻接 GPIO，**低电平点亮**。
- 多个 LED 共阳/共阴时，**每个 LED 支路最好单独加限流电阻**，不要简单并联共用一个电阻，否则亮度不均、电流分配不可控。

# GPIO端口控制方式
#### 直接电平控制
1、推挽输出：GPIO设置为推挽模式，可以强输出高/低电平，驱动能力强。但仍然不能直接驱动大功率负载。
2、开漏输出：GPIO设置为开漏输出只能强输出低电平，高电平时**输出级关断，需要外部上拉才能得到确定的高电平**。它适合低有效、共阳 LED、I2C、电平转换等场景。
3、输入模式要注意电压范围、上下拉、中断、复用功能等。

## GPIO
GPIO 是控制器上的通用输入/输出引脚，处理的是数字信号。输出为高/低电平，输入是读取外部电平信号。很多还支持复用功能、模拟模式、上下拉、中断、PWM 等。

## 硬件连接
``` text
GPIOA_Pin_1 -> 限流电阻 R -> LED阳极 -> LED阴极 -> GND
高电平点亮，低电平熄灭。
```

限流电阻可以简单算一下：
``` text

R = (VDD - Vf) / I
例如：VDD=3.3V，Vf≈2V，I=5mA
R ≈ (3.3 - 2) / 0.005 = 260Ω，常用 330Ω 或 1kΩ。
```

## 源码
``` C
// main.c
#include "stm32f10x.h"
#include "Delay.h"
#include "LED.h"

int main(void) {
    LED_Init();

    while(1)
    {
        LED_Turn();
        Delay_s(2);
    }
}

```

``` C
// LED.c
#include "stm32f10x.h"                  // Device header
#include "LED.h"

void LED_Init(void)
{
    // 使能GPIOA时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);

    // 创建结构体
    GPIO_InitTypeDef LED_GPIO_Init;
    LED_GPIO_Init.GPIO_Pin = GPIO_Pin_1;
    LED_GPIO_Init.GPIO_Mode = GPIO_Mode_Out_PP;
    LED_GPIO_Init.GPIO_Speed = GPIO_Speed_2MHz;
    
    GPIO_Init(GPIOA, &LED_GPIO_Init);
    LED_Off();
}

void LED_On(void)
{
   GPIO_SetBits(GPIOA, GPIO_Pin_1);
}

void LED_Off(void)
{
   GPIO_ResetBits(GPIOA, GPIO_Pin_1);
}

void LED_Toggle(void)
{
    uint8_t led_status;
    led_status =  GPIO_ReadOutputDataBit(GPIOA, GPIO_Pin_1);
    GPIO_WriteBit(GPIOA, GPIO_Pin_1, (BitAction)(1 - led_status));
}

```

``` C
// LED.h
#ifndef __LED_H
#define __LED_H

void LED_Init(void);
void LED_On(void);
void LED_Off(void);
void LED_Toggle(void);

#endif

```

``` C
// Delay.c
#include "stm32f10x.h"
#include "Delay.h"

/**
  * @brief  微秒级延时
  * @param  xus 延时时长，范围：0~233015
  * @retval 无
  */
void Delay_us(uint32_t xus)
{
    // 注意：当前假设 SystemCoreClock = 72MHz
	SysTick->LOAD = 72 * xus;				//设置定时器重装值
	SysTick->VAL = 0x00;					//清空当前计数值
	SysTick->CTRL = 0x00000005;				//设置时钟源为HCLK，启动定时器
	while(!(SysTick->CTRL & 0x00010000));	//等待计数到0
	SysTick->CTRL = 0x00000004;				//关闭定时器
}

/**
  * @brief  毫秒级延时
  * @param  xms 延时时长，范围：0~4294967295
  * @retval 无
  */
void Delay_ms(uint32_t xms)
{
	while(xms--)
	{
		Delay_us(1000);
	}
}
 
/**
  * @brief  秒级延时
  * @param  xs 延时时长，范围：0~4294967295
  * @retval 无
  */
void Delay_s(uint32_t xs)
{
	while(xs--)
	{
		Delay_ms(1000);
	}
} 

```

``` C
// Delay.h
#ifndef __DELAY_H
#define __DELAY_H

void Delay_us(uint32_t us);
void Delay_ms(uint32_t ms);
void Delay_s(uint32_t s);

#endif

```
