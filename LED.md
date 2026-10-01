# LED硬件需求
### LED基础硬件连接与GPIO配置需求
#### 单一LED：
1、直接连接：LED两端分别连接GPIO和GND，通过GPIO端口来控制LED的亮灭，但是由于GPIO的驱动电流较小，所以要添加一个限流电阻来保护LED和GPIO引脚。
2、使用驱动电路：
1）**三极管/MOSFET**：作为电子开关，由GPIO控制其导通与截止，从而控制流过LED的大电流。
2）**继电器**：用于控制交流或大功率直流LED负载，实现电气隔离。GPIO通过驱动三极管来间接控制继电器线圈。

#### 多个LED
1、共阳/共阴极连接：将多个LED的正极或者负极连接到一起，通过GPIO来控制各LED的亮灭。

# GPIO端口控制方式
#### 直接电平控制
1、推挽输出：GPIO设置为推挽模式，可以强输出高/低电平，驱动能力强。
2、开漏输出：GPIO设置为开漏输出只能强输出低电平，高电平时为高阻态，所以外部需要配置上拉电阻。适合共阳LED或需要电平转换场景。

## GPIO
GPIO 是控制器上的通用输入/输出引脚，处理的是数字信号。输出为高/低电平，输入是读取外部电平信号。



``` C
// main.c
#include "stm32f10x.h"
#include "Delay.h"
#include "LED.h"


uint8_t key = 0;

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
// led.c
#include "stm32f10x.h"                  // Device header

void LED_Init(void)
{
    // 使能GPIOA时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);

    // 创建结构体
    GPIO_InitTypeDef LED_GPIO_Init;
    LED_GPIO_Init.GPIO_Pin = GPIO_Pin_1;
    LED_GPIO_Init.GPIO_Mode = GPIO_Mode_Out_PP;
    LED_GPIO_Init.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOA, &LED_GPIO_Init);
    GPIO_SetBits(GPIOA, GPIO_Pin_1);
}

void LED_On(void)
{
   GPIO_SetBits(GPIOA, GPIO_Pin_1);
}

void LED_Off(void)
{
   GPIO_ResetBits(GPIOA, GPIO_Pin_1);
}

void LED_Turn(void)
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
void LED_Turn(void);

#endif

```


