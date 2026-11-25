# 第 25 章 · RTOS 多任务（LiteOS）

## 本章目标

学完这一章，你要能回答：

- 任务（task）是什么？
- 信号量、队列、互斥锁各解决什么问题？
- 为什么 WS63 开发离不开 RTOS？

---

## 一、概念：任务 = 一段"看似独立运行"的代码

第 15 章提过 RTOS。现在深入一点。

在 LiteOS 里，一个**任务（task/thread）** 就是一段 `while(1)` 循环，由操作系统**调度**，让多个任务"看起来同时运行"：

```c
void task_a(void) { while(1) { /* 干 A 的事 */ osal_msleep(10); } }
void task_b(void) { while(1) { /* 干 B 的事 */ osal_msleep(20); } }
```

操作系统在任务之间快速切换，每个任务分到一小片 CPU 时间。因为切换极快，宏观上就像"同时"在跑。

---

## 二、核心概念：任务间的通信与同步

多个任务同时跑，就会遇到两个问题：

1. **同步**：任务 A 要等任务 B 做完某事才能继续
2. **通信**：任务 A 要传数据给任务 B
3. **互斥**：两个任务不能同时改同一个共享资源

LiteOS 提供了几个工具：

| 工具 | 作用 | 类比 |
|------|------|------|
| **信号量（semaphore）** | 同步：A 等 B 的信号 | 红绿灯 / 举牌 |
| **队列（queue）** | 通信：A 传数据给 B | 传送带 |
| **互斥锁（mutex）** | 互斥：同一时间只准一个任务访问 | 厕所门锁 |
| **事件（event）** | 一个任务等多个事件 | 多盏信号灯 |

---

## 三、概念：信号量（最常用）

信号量像一个"计数器"：

- 任务 B 做完事，`give`（发出）信号量，计数 +1
- 任务 A 想继续，`take`（获取）信号量；如果没有（计数为 0），A 就**阻塞等待**

```c
/* 任务A：等待信号 */
void task_a(void) {
    while (1) {
        osal_sem_take(sem, WAIT_FOREVER);  /* 等信号，没信号就睡 */
        /* 拿到信号，继续干活 */
        do_work();
    }
}

/* 任务B：做完事发信号 */
void task_b(void) {
    while (1) {
        prepare_data();
        osal_sem_give(sem);   /* 发信号：数据好了！ */
    }
}
```

这解决了"任务 A 空转等任务 B"的浪费——A 拿不到信号时直接睡，CPU 让给别人。

---

## 四、概念：队列（传数据）

信号量只能"通知"，不能"传数据"。要传数据用**队列**：

```c
/* 生产者：往队列塞数据 */
osal_queue_write(queue, &data, sizeof(data), 0);

/* 消费者：从队列取数据 */
osal_queue_read(queue, &data, sizeof(data), WAIT_FOREVER);
```

队列是"先进先出"的传送带，天然适合"串口收到数据 → 塞队列 → 处理任务取队列"这类场景。

---

## 五、动手：在电脑上模拟信号量

电脑上没有 LiteOS，但可以用变量模拟信号量的"计数 + 阻塞"逻辑：

```c
#include <stdio.h>

/* 模拟信号量：一个计数器 */
int sem_count = 0;

/* give：信号量 +1 */
void sem_give(void) {
    sem_count++;
    printf("  [sem] give, count=%d\n", sem_count);
}

/* take：信号量 -1；如果为 0 就"阻塞"（这里用返回值表示失败） */
int sem_take(void) {
    if (sem_count > 0) {
        sem_count--;
        printf("  [sem] take ok, count=%d\n", sem_count);
        return 1;   /* 拿到 */
    }
    printf("  [sem] take fail (blocked)\n");
    return 0;   /* 没拿到，阻塞 */
}

int main(void) {
    printf("=== 任务A尝试获取信号（还没有） ===\n");
    sem_take();   /* 失败，阻塞 */

    printf("=== 任务B发出信号 ===\n");
    sem_give();   /* count 变 1 */

    printf("=== 任务A再获取信号（成功） ===\n");
    sem_take();   /* 成功 */
    return 0;
}
```

编译运行，理解信号量的"计数 + 阻塞"语义。真实 LiteOS 里 `sem_take` 拿不到时会**真的让任务睡下**，等 `give` 来了再唤醒。

---

## 六、WS63 对应关系

在 WS63 / 星闪开发里，RTOS 无处不在：

- 你的应用是一个任务，星闪协议栈、Wi-Fi 栈各自跑在别的任务
- 星闪"连接成功""数据到达"通过回调 + 信号量/队列通知你的任务
- `osal_msleep`、`osal_sem_*`、`osal_queue_*` 这些 `osal_*` 函数就是 LiteOS 的封装接口

**你不需要精通 RTOS 内部，但要会用这几个工具：任务、延时、信号量、队列。** 够你写绝大多数星闪应用了。

---

## 七、本章验收

1. 什么是任务？操作系统怎么让多任务"并行"？
2. 信号量、队列、互斥锁分别解决什么问题？
3. `take` 拿不到信号量时会发生什么？
4. 队列和信号量最大的区别是什么？
5. WS63 里 `osal_*` 系列函数是什么的封装？

---

> 下一章：第一个综合小项目——串口命令控制台（串口发命令控制 LED）。
