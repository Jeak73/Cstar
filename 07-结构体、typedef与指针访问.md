# 第 07 章 · 结构体、typedef 与指针访问

## 本章目标

学完这一章，你要能回答：

- 结构体是干什么的？
- `typedef` 是干什么的？
- `.` 和 `->` 运算符有什么区别？什么时候用哪个？

---

## 一、概念：结构体 = 一张登记表

结构体（struct）把一组**相关的、不同类型**的数据打包成"一张登记表"。比如一个 LED，它有引脚号、亮度、闪烁周期，这三个信息属于同一个东西，就该装在一起：

```c
struct led {
    int pin;       /* 引脚号 */
    int level;     /* 当前亮度/电平 */
    int blink_ms;  /* 闪烁周期（毫秒） */
};
```

---

## 二、概念：typedef = 给类型起个短名字

`struct led` 写起来啰嗦，用 `typedef` 给它起个短名 `led_t`：

```c
typedef struct led {
    int pin;
    int level;
    int blink_ms;
} led_t;   /* 现在可以直接写 led_t，不用写 struct led */

led_t my_led;      /* 简洁 */
my_led.pin = 7;    /* 用 . 访问成员 */
```

> 嵌入式 SDK 里到处是 `_t` 结尾的类型名（`errcode_t`、`pin_t`、`task_t`），就是 `typedef` 起的别名。

---

## 三、概念：`.` 和 `->` 的区别

- **`.`**：用一个**结构体变量**（实物本身）访问成员
- **`->`**：用一个**结构体指针**（门牌号）访问成员，等价于 `(*p).member`

```c
led_t led;
led.pin = 7;        /* 变量用 . */

led_t *p = &led;
p->level = 1;       /* 指针用 ->，等价于 (*p).level = 1 */
```

`->` 就是"按门牌号找到表，然后改某一栏"。

---

## 四、动手：写一个 led_toggle

```c
#include <stdio.h>

typedef struct led {
    int pin;
    int level;
    int blink_ms;
} led_t;

/* 翻转 level：必须传指针，否则改不到外面 */
void led_toggle(led_t *led) {
    led->level ^= 1;    /* ^= 是"翻转"，第 09 章细讲 */
    printf("pin %d -> level %d\n", led->pin, led->level);
}

int main(void) {
    led_t my_led = {7, 0, 500};   /* 初始化：pin=7, level=0, blink=500 */

    led_toggle(&my_led);   /* 期望 level 变 1 */
    led_toggle(&my_led);   /* 期望 level 变 0 */
    return 0;
}
```

编译运行，观察 level 在 1 和 0 之间翻转。

### 动手实验：改成值传递，看它"改不动"

把函数签名改成 `void led_toggle(led_t led)`（去掉 `*`），把 `led->level` 改成 `led.level`，重新编译运行。你会发现 level **再也翻转不了**——因为改的是复印件。

做完这个实验，你对"为什么要传指针"会有肌肉记忆。

---

## 五、概念：回调里的 void * 强转（提前打个照面）

以后读 WS63 SDK，你会频繁看到这样的写法：

```c
void my_callback(void *arg) {
    led_t *led = (led_t *)arg;   // 把通用指针"还原"成 led_t 指针
    led->level ^= 1;             // 再 -> 取成员
}
```

`void *` 是"通用门牌号"，谁的门牌号都能存，但用之前必须**强制类型转换**回具体类型。这个"强转 + 取成员"的组合，是理解 SDK 回调代码的关键，第 13 章会再深入。

---

## 六、本章验收

1. 结构体解决什么问题？
2. `typedef` 起什么作用？`_t` 后缀代表什么？
3. `.` 和 `->` 分别在什么情况下用？
4. 为什么 `led_toggle(led_t *led)` 能改到外面的变量，`led_toggle(led_t led)` 不能？
5. `void *` 是什么？用它之前要先做什么？

---

> 下一章：宏定义与条件编译——#define 和 #ifdef。
