# 第 17 章 · GPIO：点亮第一颗灯

## 本章目标

学完这一章，你要能回答：

- GPIO 是什么？
- 一个 GPIO 引脚有哪几种工作模式？
- 点亮一颗 LED 的完整流程是什么？

---

## 一、概念：GPIO = 通用输入输出

**GPIO（General-Purpose Input/Output，通用输入输出）** 是芯片引脚最基本的功能：一个引脚可以被你配置成**输出**（控制外部电平，比如点亮 LED）或**输入**（读取外部电平，比如检测按键）。

一个 GPIO 引脚通常有几个关键属性：

| 属性 | 含义 |
|------|------|
| 方向 | 输入 or 输出 |
| 电平 | 高（1）/ 低（0） |
| 上下拉 | 内部上拉/下拉电阻，用于输入时稳定电平 |
| 复用功能 | 同一个引脚可复用为 UART/SPI/I2C 等（第 09 章的位操作常用来配复用） |

---

## 二、概念：点亮 LED 的完整流程

不管什么芯片，点亮一颗 LED 的套路都差不多，分四步：

1. **使能时钟 / 初始化外设**（有些芯片要先"通电"给 GPIO）
2. **配置引脚为输出**（`uapi_gpio_set_dir(pin, GPIO_DIRECTION_OUTPUT)`）
3. **设置电平为高**（`uapi_gpio_set_val(pin, GPIO_LEVEL_HIGH)`）—— LED 亮
4. **（可选）延时后拉低** —— LED 灭

对应 WS63 的 SDK 接口（提前打个照面，第 23 章上板会用）：

```c
uapi_gpio_init();                                       /* 1. 初始化 GPIO */
uapi_gpio_set_dir(pin, GPIO_DIRECTION_OUTPUT);          /* 2. 设为输出 */
uapi_gpio_set_val(pin, GPIO_LEVEL_HIGH);                /* 3. 拉高 → LED 亮 */
```

---

## 三、概念：为什么"拉高"有时点亮、有时熄灭

这取决于 LED 是怎么接的：

- **高电平点亮**（源极驱动）：LED 阳极接 GPIO，GPIO 输出高 → 灯亮
- **低电平点亮**（漏极驱动）：LED 阴极接 GPIO，GPIO 输出低 → 灯亮

所以拿到一块新板子，先看**原理图**确认 LED 接法，别想当然。WS63 开发板上通常是 GPIO 拉高点亮。

---

## 四、动手：在电脑上模拟 GPIO 的"四步点亮流程"

```c
#include <stdio.h>

/* 模拟 GPIO 方向枚举（和 WS63 SDK 风格一致） */
typedef enum {
    GPIO_DIRECTION_INPUT  = 0,
    GPIO_DIRECTION_OUTPUT = 1,
} gpio_direction_t;

typedef enum {
    GPIO_LEVEL_LOW  = 0,
    GPIO_LEVEL_HIGH = 1,
} gpio_level_t;

typedef enum {
    ERR_OK = 0,
    ERR_PARAM,
} errcode_t;

/* 模拟一个 GPIO 引脚的"硬件状态" */
typedef struct {
    gpio_direction_t dir;
    gpio_level_t     level;
} gpio_pin_t;

static gpio_pin_t pins[32];   /* 模拟 32 个引脚 */

/* 模拟 SDK 函数 */
errcode_t gpio_set_dir(int pin, gpio_direction_t dir) {
    if (pin < 0 || pin > 31) return ERR_PARAM;
    pins[pin].dir = dir;
    printf("  [gpio] pin %d -> %s\n", pin,
           dir == GPIO_DIRECTION_OUTPUT ? "OUTPUT" : "INPUT");
    return ERR_OK;
}

errcode_t gpio_set_val(int pin, gpio_level_t level) {
    if (pin < 0 || pin > 31) return ERR_PARAM;
    if (pins[pin].dir != GPIO_DIRECTION_OUTPUT) {
        printf("  [gpio] pin %d is not output!\n", pin);
        return ERR_PARAM;
    }
    pins[pin].level = level;
    printf("  [gpio] pin %d -> %s (LED %s)\n", pin,
           level == GPIO_LEVEL_HIGH ? "HIGH" : "LOW",
           level == GPIO_LEVEL_HIGH ? "ON" : "OFF");
    return ERR_OK;
}

int main(void) {
    int led_pin = 7;

    printf("=== 点亮 LED 四步流程 ===\n");
    gpio_set_dir(led_pin, GPIO_DIRECTION_OUTPUT);  /* 1. 设输出 */
    gpio_set_val(led_pin, GPIO_LEVEL_HIGH);        /* 2. 拉高 → 亮 */
    gpio_set_val(led_pin, GPIO_LEVEL_LOW);         /* 3. 拉低 → 灭 */

    printf("\n=== 错误演示：没设方向就输出 ===\n");
    gpio_set_val(8, GPIO_LEVEL_HIGH);              /* 会报错，因为 pin8 没设输出 */

    return 0;
}
```

编译运行，观察"四步流程"和"错误演示"的输出差异。

---

## 五、WS63 对应关系

这段模拟代码的 `errcode_t`、`gpio_set_dir`、`gpio_set_val` 命名和参数风格，**就是照着 WS63 SDK 的 `uapi_gpio_*` 系列设计的**。等你第 23 章上板，看到的会是几乎一模一样的调用，只是这些函数由 SDK 真实实现，真的去操作硬件寄存器。

---

## 六、本章验收

1. GPIO 是什么？"输入"和"输出"分别用于什么场景？
2. 点亮一颗 LED 的完整流程分几步？
3. 为什么"拉高"有时点亮有时熄灭？
4. WS63 里设置引脚方向、设置电平分别用什么函数？

---

> 下一章：中断——硬件"叫你"的方式，理解中断服务函数与 volatile。
