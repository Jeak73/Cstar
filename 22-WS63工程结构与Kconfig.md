# 第 22 章 · WS63 工程结构与 Kconfig

## 本章目标

学完这一章，你要能回答：

- FBB 框架是什么？为什么用它？
- 一个 WS63 工程有哪些关键目录？
- Kconfig 是怎么把"图形配置"变成"代码开关"的？

---

## 一、概念：FBB 框架

**FBB（Family Big Box，统一开发框架、统一 API）** 是海思星闪方案的开发框架。核心价值：

- **统一 API**：不同星闪芯片（WS63、BS21 等）用同一套接口，代码好移植
- **降低门槛**：屏蔽底层差异，开发者专注业务逻辑
- 你在 WS63 上写的应用，将来容易迁移到其他星闪解决方案

这就是为什么 SDK 里 API 都带 `uapi_` 前缀——**u**nified **API**，统一接口。

---

## 二、工程的关键目录

一个 WS63 SDK / 工程大致长这样：

```
fbb_ws63/
  ├── docs/        ← 软件资料、用户指南、IO复用关系表
  ├── src/         ← SDK 源码（新建工程时要选到这一层）
  ├── tools/       ← 开发工具、环境搭建指南
  └── vendor/      ← 合作厂商的板子硬件/软件资料、案例代码
```

你自己的应用工程里，则常见：

```
你的工程/
  ├── src/         ← 你的应用源码（.c）
  ├── include/     ← 你的头文件（.h）
  ├── CMakeLists.txt  ← 构建配置（"把哪些源文件装订进来"）
  └── 配置文件       ← Kconfig 生成的东西
```

---

## 三、概念：CMakeLists.txt 决定"谁参与编译"

第 14 章学过：多文件工程要"把源文件列进编译"，否则报 `undefined reference`。

在 WS63 工程里，这个"清单"写在 **`CMakeLists.txt`** 里。你新增了一个 `.c` 文件，就要在 `CMakeLists.txt` 里登记它。**忘了登记 → 链接错误**，这和第 14 章的原理一模一样。

---

## 四、概念：Kconfig 图形化配置

**Kconfig** 是一种"菜单式配置"系统（ESP32、Linux 内核都用它）。你在图形界面里勾勾选选，最终会：

1. 生成一个配置文件
2. 配置文件里的选项变成 `CONFIG_XXX` 宏
3. 代码里用 `#ifdef CONFIG_XXX` 决定编译哪段（第 08 章学过）

**完整链路**（这条链要刻进脑子）：

```
Kconfig 图形界面勾选
      ↓
生成 CONFIG_ 宏
      ↓
#ifdef 决定编译哪些代码
      ↓
CMake 决定编译哪些文件
      ↓
最终固件
```

举例：你在 Kconfig 里勾选「Support BLINKY Sample」，系统会定义 `CONFIG_BLINKY` 之类的宏，工程里相应例程的代码就被编译进来。

---

## 五、概念：一个典型应用的结构（blinky 例程）

WS63 的 LED 闪烁例程，应用代码大致是"注册入口回调"的模式（第 13 章的回调）：

```c
#include "app_init.h"

/* 应用入口：由 SDK 调度器调用 */
static void *blinky_entry(const char *arg) {
    /* 初始化 GPIO、循环闪灯 */
    while (1) {
        uapi_gpio_set_val(led_pin, GPIO_LEVEL_HIGH);
        osal_msleep(500);
        uapi_gpio_set_val(led_pin, GPIO_LEVEL_LOW);
        osal_msleep(500);
    }
    return NULL;
}

/* 注册入口：把函数地址交给 SDK */
app_run(blinky_entry);
```

这里能看到前面学的一切：
- `uapi_gpio_*` 统一 API（第 17 章）
- `while(1)` 超级循环（第 15 章）
- `app_run(blinky_entry)` 把函数地址交给 SDK = 回调注册（第 13 章）
- `osal_msleep` 是 RTOS 的延时，不阻塞其他任务

---

## 六、动手：读例程

即使还没完全上板，也可以先**打开 SDK 里的 blinky 例程代码读一遍**（在 HiSpark Studio 里），对照上面第 22 章的知识，标注出：

- 哪里是"入口回调注册"
- 哪里是"超级循环"
- 哪里调用了 `uapi_gpio_*` API
- 哪里用到了宏/配置

读不懂没关系，标注出来，带着问题继续往下学，第 23 章会逐行讲。

---

## 七、本章验收

1. FBB 框架的核心价值是什么？`uapi_` 前缀代表什么？
2. 新增一个 `.c` 文件后，还要在哪个文件里登记？忘了会怎样？
3. 画出"Kconfig → 固件"的完整链路。
4. `app_run(blinky_entry)` 本质是什么（提示：和回调的关系）？

---

> 下一章：GPIO 控制 LED（上板版）——在真板子上点亮/闪烁 LED。
