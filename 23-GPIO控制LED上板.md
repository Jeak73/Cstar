# 第 23 章 · GPIO 控制 LED（上板版）

## 本章目标

学完这一章，你要能回答：

- 如何在 WS63 上点亮/闪烁 LED？
- `uapi_gpio_set_dir`、`uapi_gpio_set_val` 怎么用？
- `osal_msleep` 和普通延时有什么区别？

---

## 一、回顾：点亮 LED 的四步

第 17 章学过，点亮 LED 分四步。现在上真板子，把模拟代码换成真实 SDK 调用：

```
1. uapi_gpio_init()                             初始化 GPIO
2. uapi_gpio_set_dir(pin, GPIO_DIRECTION_OUTPUT)  设为输出
3. uapi_gpio_set_val(pin, GPIO_LEVEL_HIGH)        拉高 → LED 亮
4. 延时后拉低 → LED 灭（闪烁）
```

---

## 二、关键 API

```c
/* 初始化 GPIO（使用前调用一次） */
errcode_t uapi_gpio_init(void);

/* 设置引脚方向 */
errcode_t uapi_gpio_set_dir(pin_t pin, gpio_direction_t dir);
/* dir: GPIO_DIRECTION_OUTPUT / GPIO_DIRECTION_INPUT */

/* 设置引脚电平 */
errcode_t uapi_gpio_set_val(pin_t pin, gpio_level_t level);
/* level: GPIO_LEVEL_HIGH / GPIO_LEVEL_LOW */
```

记住第 04 章的纪律：**这些函数都返回 `errcode_t`，成功是 `ERR_OK`（0），要接住判断**。

---

## 三、LED 在哪个引脚？

WS63 开发板上 LED 的引脚，**以你的板子原理图为准**。常见做法是在 Kconfig 里通过「Choose BLINKY pin」选项配置，或从原理图查。不同版本板子 LED 引脚可能不同（比如有的在某个特定 IO），**一定要先确认再写代码**。

---

## 四、上板代码骨架（闪烁 LED）

```c
#include "app_init.h"
#include "gpio.h"       /* uapi_gpio_* 接口 */
#include "osal.h"       /* osal_msleep 延时 */

#define LED_PIN  7      /* 改成你板子实际的 LED 引脚号！ */

static void *led_task(const char *arg) {
    /* 1. 初始化 GPIO */
    errcode_t ret = uapi_gpio_init();
    if (ret != ERR_OK) {
        printf("gpio init failed: %d\n", ret);
        return NULL;
    }

    /* 2. 设为输出 */
    ret = uapi_gpio_set_dir(LED_PIN, GPIO_DIRECTION_OUTPUT);
    if (ret != ERR_OK) {
        printf("set dir failed: %d\n", ret);
        return NULL;
    }

    /* 3. 循环闪烁 */
    while (1) {
        uapi_gpio_set_val(LED_PIN, GPIO_LEVEL_HIGH);   /* 亮 */
        osal_msleep(500);                               /* 等 500ms */
        uapi_gpio_set_val(LED_PIN, GPIO_LEVEL_LOW);    /* 灭 */
        osal_msleep(500);                               /* 等 500ms */
    }
    return NULL;
}

app_run(led_task);   /* 注册入口，交给 SDK 调度 */
```

> 注意：具体头文件名、API 名、`app_run` 的写法，**务必以你 SDK 版本的实际例程为准**。SDK 会随版本略有差异，最稳的办法是：**先跑通官方 blinky 例程，再在它基础上改**。

---

## 五、关键区别：osal_msleep vs 空转延时

| | `osal_msleep(500)` | `for` 空转延时 |
|---|---|---|
| 本质 | RTOS 延时，让出 CPU 给别的任务 | 死等，霸占 CPU |
| 影响 | 其他任务照常运行 | 阻塞整个系统 |

在 RTOS 上，**永远用 `osal_msleep` 这类系统延时，不要写空转延时**。这是从"电脑上写 C"到"板子上写任务"的一个重要转变。

---

## 六、进阶：呼吸灯（PWM 调光）

掌握闪烁后，可以试 PWM 呼吸灯（第 19 章学的 PWM）。SDK 里通常有 `uapi_pwm_*` 系列接口，思路是：

1. 初始化 PWM 通道
2. 设置周期（频率）
3. 循环调整占空比，让 LED 亮度渐变

具体 API 以你 SDK 文档为准，先跑官方 PWM 例程再改造。

---

## 七、上板验收清单

- [ ] LED 能稳定闪烁
- [ ] 闪烁频率和代码里的 `osal_msleep(500)` 一致
- [ ] 把 `LED_PIN` 改错，观察返回值判断是否生效（能打印出 `set dir failed`）
- [ ] 把延时改成 100ms / 1000ms，观察闪烁变快变慢

---

## 八、本章验收

1. 点亮 LED 的四步，在 WS63 上分别对应什么函数？
2. 为什么这些 API 的返回值要接住判断？
3. `osal_msleep` 和空转延时有什么区别？为什么 RTOS 上要用前者？
4. 拿到一块新板子，第一步应该确认什么？

---

> 下一章：串口收发（上板版）——板子与电脑双向对话。
