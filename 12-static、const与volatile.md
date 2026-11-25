# 第 12 章 · static / const / volatile

## 本章目标

学完这一章，你要能回答：

- `static` 用在局部变量、全局变量、函数上分别是什么含义？
- `const` 是干什么的？"修饰的到底是什么"？
- `volatile` 为什么在中断、硬件寄存器场景下**必须**用？

---

## 一、概念：三个关键字的三个类比

- **`static` = 公司固定资产**：不随函数结束而消失（数据一直在）
- **`const` = 贴在墙上的规章制度**：只读，不许改
- **`volatile` = 随时可能被外面改的白板**：别让编译器"抄近路"缓存它的值

---

## 二、static 的三种用法

`static` 很"多面"，同一个关键字在不同地方含义不同：

### 1. 修饰局部变量：让变量"活过"整个程序

```c
void counter(void) {
    static int count = 0;   // 只初始化一次，之后值一直保留
    count++;
    printf("called %d times\n", count);
}
```

普通局部变量每次调用都"重生"，`static` 局部变量则"记住上次的值"。

### 2. 修饰全局变量/函数：限制作用域（仅本文件可见）

```c
static int secret = 42;   // 其他 .c 文件看不到它
```

相当于"私有"——防止多个文件之间的名字冲突。

### 3. （嵌入式里 static 局部变量最常用，用于"记住状态"）

---

## 三、const：只读

```c
const int MAX_NUM = 100;   // 之后不能改
MAX_NUM = 200;             // 编译错误！
```

`const` 让编译器帮你拦住"不小心修改常量"的错误。注意 `const` 修饰的是**它右边紧跟的东西**：

```c
const int *p;       // p 指向的内容不能改（*p 只读）
int * const p;      // p 本身不能改（指针不可重新指向）
const int * const p; // 两者都不能改
```

这个细节容易绕晕，但嵌入式里常见的是第一种：`const` 修饰数据，防止函数误改传入的数据。

---

## 四、volatile：别缓存，每次都去读真实值

**这是嵌入式最重要的关键字之一。**

编译器为了优化，可能把变量的值"缓存"在寄存器里，认为"这个变量没人会改它，我读一次就够"。但在嵌入式里，有些变量的值会**被编译器看不见的东西改变**：

- 硬件寄存器：值会随硬件状态自己变
- 中断服务函数里修改的变量：主循环读它时，中断可能随时改它
- 多任务共享的变量

这时必须加 `volatile`，告诉编译器：**"每次都老老实实去内存里读最新值，别抄近路缓存。"**

### 经典错误案例：中断标志

```c
volatile int flag = 0;   // 中断里会置 1，主循环轮询它

/* 主循环 */
while (1) {
    if (flag) {          // 如果不加 volatile，编译器可能优化成"永远读缓存的0"
        flag = 0;
        /* 处理中断 */
    }
}
```

如果不加 `volatile`，编译器可能把 `flag` 优化成"读一次缓存的 0，以后一直以为它是 0"，导致主循环**永远检测不到中断**——这是个极其隐蔽的 bug。

---

## 五、动手练习

```c
#include <stdio.h>

/* static 局部变量：记住调用次数 */
void counter(void) {
    static int count = 0;
    count++;
    printf("called %d times\n", count);
}

/* static 全局变量：仅本文件可见 */
static int secret = 42;

/* const：只读 */
const int MAX_NUM = 100;

int main(void) {
    counter();   /* called 1 times */
    counter();   /* called 2 times */
    counter();   /* called 3 times */

    printf("secret=%d max=%d\n", secret, MAX_NUM);

    /* MAX_NUM = 200;   // 这行会编译报错，注释掉 */

    return 0;
}
```

编译运行，观察 `counter` 每次调用 `count` 都在递增（而不是每次都从 0 开始）。

---

## 六、本章验收

1. `static` 修饰局部变量、全局变量各是什么意思？
2. `const` 有什么用？
3. 为什么中断标志变量必须加 `volatile`？
4. 不加 `volatile` 可能导致什么 bug？
5. 举一个嵌入式里需要 `volatile` 的场景。

---

> 下一章：函数指针与回调——攻克读 SDK 的最大障碍。
