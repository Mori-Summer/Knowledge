---
doc_id: computer-systems-thread
title: 线程：共享地址空间里的调度执行流，不是更轻量的进程，也不是协程
concept: thread
topic: computer-systems
depth_mode: deep
created_at: '2026-03-23T10:31:57+08:00'
updated_at: '2026-03-23T10:31:57+08:00'
source_basis:
  - linux_pthreads_manpage_checked_2026_03_23
  - linux_pthread_create_manpage_checked_2026_03_23
  - linux_pthread_join_manpage_checked_2026_03_23
  - linux_pthread_detach_manpage_checked_2026_03_23
  - linux_clone_manpage_checked_2026_03_23
  - linux_sched_manpage_checked_2026_03_23
time_context: current_practice_checked_2026_03_23
applicability: concurrency_modeling_parallel_execution_thread_lifecycle_and_synchronization_design
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/computer-systems/process.md
  - docs/computer-systems/multithreaded-locks.md
  - docs/computer-systems/mutex.md
  - docs/computer-systems/condition-variable.md
  - docs/computer-systems/semaphore.md
  - docs/programming-languages/cpp20-coroutine-playbook.md
open_questions:
  - 是否需要补一篇专门解释 Linux NPTL、`clone` 标志组合与用户态线程库之间关系的实现文档？
  - 线程数、CPU 亲和性、NUMA、本地缓存与线程池 sizing 之间，是否值得再写一篇性能导向文档？
  - 线程与协程混合系统里的取消、阻塞 API 隔离与执行器边界，是否需要单独总结为工程模式？
---

# 线程：共享地址空间里的调度执行流，不是更轻量的进程，也不是协程

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立的，不是“线程比进程轻”这种粗粒度印象，而是一套以后能反复调用的内部模型：

- 你能解释线程为什么首先是进程内的执行流与调度对象，而不是“缩小版进程”
- 你能区分线程共享什么、独享什么，以及这会怎样改变并发设计
- 你能说清 `pthread_create -> run/block -> terminate -> join/detach` 这条主链路
- 你能把线程和进程、协程、CPU 核心、任务对象这些相邻概念稳定地区分开
- 你能把这个模型迁移到线程池、同步设计、join/detach 纪律和“该不该上线程”的判断里

如果只记一句话：

**线程买到的是独立执行流和并行机会，但它不提供隔离；它共享进程的大多数资源，因此把通信便利换成了同步与生命周期复杂度。**

## 2. 一句话结论 / 问题定义

**线程是进程内部的调度执行流：它共享进程的大多数资源与地址空间，但保留自己的栈、寄存器上下文、线程 ID、信号屏蔽字和调度属性，因此成为“同一资源容器中的多个并发执行成员”。**

线程真正要解决的是：

- 如何在同一进程内同时推进多条执行路径
- 如何在共享地址空间下做更低成本的数据交换
- 如何把 CPU 并行、阻塞等待隔离、响应性提升这些需求放进同一个资源容器里

它同时也引入新的问题：

- 共享状态带来的 race、死锁、生命周期耦合
- “执行流更多”不等于“性能一定更高”
- 线程退出、join、detach、取消和 process-wide 终止之间的关系

## 3. 对象边界与相邻概念

### 3.1 这篇文档直接处理什么

这篇文档直接处理的是：

- 线程作为进程内 schedulable execution flow 的概念
- 线程共享和独享的资源划分
- 线程创建、运行、阻塞、终止、join / detach 的生命周期
- 调度、优先级、亲和性、信号与同步原语的线程语义

### 3.2 它不等于什么

它不等于：

- **进程。** 进程是资源容器和生命周期边界；线程是容器内执行流。
- **CPU 核心。** 线程是调度对象，不等于物理核心数量；线程数远大于核心数并不罕见。
- **协程。** 协程是可挂起控制流结构，不是内核调度对象；协程文章已经在 [cpp20-coroutine-playbook.md](/Users/maxwell/Knowledge/docs/programming-languages/cpp20-coroutine-playbook.md) 单独展开。
- **任务 / future / actor。** 这些是更高层的并发抽象，不自动等价于一个 OS thread。
- **单纯“更轻量的进程”。** 这句类比能帮你入门，但会遮住线程最重要的共享语义和同步代价。

### 3.3 最值得一起看的相邻概念

最值得一起看的相邻概念是：

- [进程：执行实例、资源容器与隔离边界的统一模型](/Users/maxwell/Knowledge/docs/computer-systems/process.md)
- [多线程中的各种锁：不要先问 API 名字，先问所有权、等待方式与读写形状](/Users/maxwell/Knowledge/docs/computer-systems/multithreaded-locks.md)
- [Mutex：你真正买到的不是“挡别人一下”，而是互斥区间和同步边](/Users/maxwell/Knowledge/docs/computer-systems/mutex.md)
- [Condition Variable：你等的不是通知，而是条件何时在同步关系下成立](/Users/maxwell/Knowledge/docs/computer-systems/condition-variable.md)
- [Semaphore：当问题是“还有没有令牌”，而不是“谁拥有锁”](/Users/maxwell/Knowledge/docs/computer-systems/semaphore.md)
- [C++20 协程：可挂起控制流、语言机制与运行时边界](/Users/maxwell/Knowledge/docs/programming-languages/cpp20-coroutine-playbook.md)

### 3.4 本文的默认适用边界

本文默认工作边界是：

- 以现代 Linux / POSIX 线程模型为主
- 以 pthread API 和 Linux thread group 现实实现为主
- 重点处理真实工程里“共享什么、独享什么、怎么退出、怎么同步、怎么和协程区分”

涉及 pthread、调度、`clone` / thread group 语义的当前事实核对日期均为 `2026-03-23`。

### 3.5 五组最容易混淆的边界

最常见的混淆有五组：

- **线程 vs 进程。**
  线程共享进程大部分资源；进程是资源和生命周期容器。

- **线程 vs 核心。**
  核心是硬件执行资源；线程是被调度的抽象成员。

- **线程 vs 协程。**
  线程由 OS 调度，可并行；协程由语言/runtime 管理挂起恢复，不天然等于并行。

- **`pthread_t` vs TID / PID。**
  `pthread_t` 是 pthread 层句柄；Linux 还有内核 TID 和 thread group 共享的 TGID / PID。

- **线程终止 vs 线程资源释放。**
  线程 return / `pthread_exit` 不等于其所有系统资源都已被 join 或 detach 处理。

## 4. 核心结构

### 4.1 最稳的模型：线程 = 共享容器中的独立执行成员

理解线程，最稳的方式不是记“它有自己的栈”，而是先记住下面这个模型：

| 维度 | 线程层事实 | 为什么重要 |
| --- | --- | --- |
| 容器归属 | 线程活在某个进程里 | 决定它共享什么、不共享什么 |
| 执行上下文 | 有自己的寄存器状态、指令位置、栈 | 决定它能独立暂停、恢复、阻塞 |
| 身份 | 有线程 ID / `pthread_t` / TID | 决定 join、signal、affinity、调试 |
| 调度属性 | 调度策略、优先级、CPU 亲和性是 per-thread | 决定实际运行行为 |
| 生命周期 | create / run / block / terminate / join / detach | 决定资源回收与设计纪律 |

### 4.2 共享什么，独享什么

官方 `pthreads(7)` 给出的第一性事实非常关键：

- 同一进程内的线程共享 global memory，也就是 data / heap segments
- 每个线程有自己的 stack
- 进程 ID、父进程 ID、进程组 / session、open file descriptors、signal dispositions 等是 process-wide
- `pthread_t`、signal mask、`errno`、alternate signal stack、real-time scheduling policy and priority 等是 per-thread

把这组事实压成最小判断表：

| 类型 | 常见对象 |
| --- | --- |
| 共享 | 地址空间、heap、FD、信号处置、PID/TGID |
| 独享 | 栈、寄存器上下文、`pthread_t`、TID、signal mask、调度属性 |

这张表解释了线程最核心的 tradeoff：

- 共享让通信更便宜
- 共享也让同步更难、故障传播更快

### 4.3 Linux 现实模型：thread group，而不是“很多小进程”

Linux `clone(2)` 关于 `CLONE_THREAD` 的说明非常关键：

- 设置 `CLONE_THREAD` 后，新线程进入与调用者相同的 thread group
- 组内线程共享一个 TGID；`getpid()` 返回的是这个 thread group identifier
- 每个线程仍有自己唯一的 TID
- 线程终止时不会像普通 child process 那样给创建者发 `SIGCHLD`，也不能用 `wait(2)` 取得状态

这意味着现代 Linux 上：

- “进程”更像 thread group + process-wide resources 的组合
- “线程”是组内的具体执行成员

### 4.4 线程不是天然“更轻”，而是“共享更多”

很多入门资料把线程讲成“轻量进程”。  
这不是完全错，但不够稳。

更稳的表述是：

- 线程之所以常显得更轻，是因为它**不复制出新的资源容器**
- 它直接活在已有进程里，共享地址空间、FD、PID/TGID 等大块资源
- 所以创建/通信成本通常更低
- 代价是同步、生命周期和调试复杂度显著上升

### 4.5 一页纸判断模板

看到线程问题时，可以先过这张表：

| 你在问什么 | 优先从哪层判断 |
| --- | --- |
| 为什么多个执行流能直接看同一份对象 | 共享地址空间 |
| 为什么一个线程崩坏可能影响整个进程 | 共享资源容器 |
| 为什么加线程后没有变快 | 调度、核心数、锁争用、共享缓存 |
| 为什么线程结束了还要 join / detach | 生命周期与资源释放 |
| 为什么协程不能直接替代线程 | 调度层级和并行能力不同 |

## 5. 核心机制 / 主链路 / 因果链

### 5.1 一条完整主链路：`pthread_create -> run/block -> terminate -> join/detach`

这是线程最核心的主链路。

1. 某线程在当前进程中调用 `pthread_create()`。
2. 官方 `pthread_create(3)` 明确指出：新线程在**调用者所属进程内**启动，执行 `start_routine(arg)`。
3. 新线程继承调用线程的一些上下文，例如 signal mask 和 floating-point environment；其 pending signal 集合为空，CPU-time clock 从 0 开始。
4. 调度器按照 per-thread scheduling policy、priority 和可运行状态决定何时让它上 CPU。官方 `sched(7)` 直接说明：调度器选择的是**下一个 runnable thread**。
5. 运行中，线程可能因为锁、条件变量、IO、睡眠或调度让出而 block / yield。
6. 线程可以通过三种最常见方式结束：从 `start_routine` return、调用 `pthread_exit()`、被取消；另外，若进程中任一线程调用 `exit(3)` 或主线程从 `main()` 返回，则整个进程中的所有线程都会终止。
7. 结束后的线程若是 joinable，需要其他线程 `pthread_join()`；若已 detached，则终止后系统会自动回收其线程级资源。

这条链真正解释的是：

- 线程不是“扔出去自己跑就完了”
- 它有明确的创建、运行、终止、回收协议

### 5.2 `pthread_join` / `pthread_detach`：线程资源为什么不会自己总是干净落地

官方 `pthread_join(3)` 和 `pthread_detach(3)` 给出的关键事实是：

- `pthread_join()` 等待指定线程终止，并在成功返回后保证目标线程已终止
- 若 joinable 线程既不 join 也不 detach，会形成 zombie thread，继续消耗系统资源
- `pthread_detach()` 会让线程在终止后自动把其资源释放回系统
- 每个应用创建的线程，最终都应该走向 **join 或 detach** 之一

这一点非常像进程里的“退出”和“回收”要分开理解：

- 线程执行结束
- 不等于其线程级资源已被正确处理

### 5.3 共享地址空间变体链：为什么线程通信容易、同步困难

因为线程共享 data / heap segments，所以：

1. 一个线程写入堆对象
2. 另一个线程可以直接通过同一地址读到
3. 这带来很低的共享成本
4. 同时也带来 race、锁、条件变量、原子操作、缓存争用和生命周期耦合问题

所以线程的第一性 tradeoff 是：

- 你用“无需显式 IPC”的便利
- 交换了“必须显式同步共享状态”的义务

### 5.4 调度变体链：线程是调度单位，不是抽象任务

官方 `sched(7)` 的关键表述是：

- 调度器决定下一个上 CPU 的 runnable thread
- scheduling policy、priority、CPU affinity 等是 per-thread 维度

这条链解释了三个常见现实：

- 开更多线程不等于开更多核心
- 一个进程内不同线程可以有不同调度属性
- “线程多就一定快”是错误直觉，真正结果要看 runnable 竞争、锁、核心数和缓存局部性

### 5.5 线程组与 `execve` / `wait` 边界

Linux `clone(2)` 和 `execve(2)` 合起来给出两个关键事实：

- 线程组中的普通线程不能像子进程那样被 `wait(2)` 取得退出状态
- 若线程组中任一线程执行 `execve()`，则除 thread-group leader 外的其他线程会被销毁，新程序在 leader 中运行

这说明线程生命周期并不是独立世界，它仍被更大的进程生命周期包裹。

## 6. 关键 tradeoff 与失败模式

### 6.1 四个核心 tradeoff

| tradeoff | 你买到什么 | 你付出什么 |
| --- | --- | --- |
| 共享地址空间 vs 隔离 | 更便宜的数据交换 | 更高的同步和内存安全风险 |
| 更多线程 vs 更高并行度 | 更可能利用多核、隔离阻塞 | 过度切换、锁争用、oversubscription |
| joinable 生命周期 vs detached 方便性 | 更清晰的结果收集与清理点 | 需要显式管理，不可随意忘记 |
| 线程级调度控制 vs 系统复杂度 | 可做优先级、affinity、RT policy 调整 | 更容易产生调度反直觉与性能陷阱 |

### 6.2 七类高频失败模式

1. **把线程当“更轻量的进程”。**
   这样你会低估共享状态和同步代价。

2. **把线程当 CPU 核心。**
   这样你会误判吞吐和调度。

3. **创建线程后既不 join，也不 detach。**
   官方 `pthread_join(3)` 明确说明这会导致 zombie thread。

4. **混淆 `pthread_t`、TID 和 PID/TGID。**
   这会导致日志、调试、信号和线程管理语义错位。

5. **想当然地认为“加线程就能消掉阻塞”。**
   它能隔离阻塞，但不能消掉阻塞成本；它只是把等待放到别的执行流里。

6. **用线程解决本质上是控制流表达的问题。**
   如果问题主要是等待链表达和挂起恢复，而不是并行执行，协程或 event loop 语义通常更贴切。

7. **低估 process-wide 终止。**
   `exit(3)` 或主线程从 `main()` 返回会终止整个进程的全部线程。

## 7. 应用场景

### 7.1 CPU 并行与 worker pool

当问题的本质是：

- 多块任务可同时跑
- 需要用到多个核心
- 共享部分数据但不想走 IPC

线程就是直接的执行抽象。

### 7.2 把阻塞 API 隔离到后台执行流

如果底层 API 是阻塞式的，你又不想卡住主流程：

- 后台 worker thread
- 专用 IO thread
- 专用 decode / parse / logging 线程

都属于线程的典型使用场景。

### 7.3 UI / 主循环响应性

当你需要：

- 主线程保持响应
- 后台线程处理耗时任务
- 再把结果交回主线程

线程模型就会比“所有事都堵在一个执行流里”更可控。

### 7.4 共享状态并发系统

线程的高价值场景不只是“跑更多事”，还包括：

- 一个共享缓存
- 一个共享连接池
- 一个共享对象图

多个执行流围绕同一套 process-wide 资源协作。

## 8. 工业 / 现实世界锚点

### 8.1 POSIX pthread API：最现实的跨平台线程语义入口

`pthreads(7)`、`pthread_create(3)`、`pthread_join(3)`、`pthread_detach(3)` 是线程概念最直接的现实锚点。

它的重要性在于：

- 它把共享 / 独享属性、创建、等待、分离、终止语义都明确成了系统接口合同
- 它不是教程约定，而是大量 C/C++/系统库的真实基线

### 8.2 Linux `clone` thread group：线程在内核里的现实落点

Linux 通过 `clone(2)` 的 thread-group 语义说明：

- 线程不是语言层幻觉
- 它在内核里有真实的 TGID / TID、signal、wait 和 `execve` 交互约束

这能把“pthread 只是库接口”进一步落到真实内核模型上。

### 8.3 `sched(7)`：线程才是 CPU 调度的直接对象

这条锚点的重要性在于，它纠正了一个最常见误判：

- 不是“进程整体被调度”
- 而是 runnable threads 被调度

一旦理解这点，你就能更稳地分析线程数、核心数、优先级、affinity 和实时策略。

## 9. 当前推荐实践、过时路径与替代

本节涉及 pthread、`clone`、`sched` 等当前 Linux / POSIX 语义，核对日期均为 `2026-03-23`。  
下面“更推荐什么”的部分，除 man page 直接给出的事实外，其余是基于这些语义的工程推断。

### 9.1 三条过时路径

#### 过时路径 1：把线程讲成“更轻量的进程”

局限：

- 会弱化共享地址空间的本质
- 会低估同步、生命周期和故障耦合

更稳替代：

- 先把线程讲成“共享进程容器中的执行成员”

#### 过时路径 2：把线程讲成“天然更快”

局限：

- 线程多可能只是引入更多 runnable 竞争、锁争用、缓存抖动
- 并行收益取决于工作负载、核心数、同步成本和阻塞形状

更稳替代：

- 先问问题本质是并行计算、阻塞隔离，还是等待链表达

#### 过时路径 3：把线程结束视为自动收尾

局限：

- joinable 线程不 join / detach 会留下 zombie thread
- 生命周期管理会悄悄变成资源泄漏

更稳替代：

- 为每个线程预先定义 join 或 detach 策略，而不是事后想起再补

### 9.2 截至 2026-03-23 的更稳基线

更稳的基线通常是：

1. **需要真并行或阻塞隔离时才上线程。**
   线程最值钱的是执行流和调度权，不是“写起来高级”。

2. **先建模共享 / 独享边界，再建模锁。**
   不先分清哪些状态是 process-wide 共享，锁策略就会飘。

3. **线程生命周期要么 join，要么 detach。**
   这是官方语义直接要求你面对的纪律。

4. **把调度、优先级、CPU affinity 当成 per-thread 属性看。**
   不要把它们粗暴地当进程整体属性。

### 9.3 何时优先线程，何时更像是在找协程

这是工程推断，但通常足够稳：

- **优先线程：**
  当你需要真正并行、需要把阻塞 API 挪到别的执行流、需要线程级调度和亲和性控制。

- **更像是在找协程：**
  当你真正痛的是等待链表达、回调破碎、挂起恢复、而不是 CPU 并行。

协程文章已经在仓库里有了，所以这里最关键的边界句是：

- **线程解决“谁在 CPU 上跑”**
- **协程解决“流程怎样挂起后再回来”**

### 9.4 join / detach 的更稳纪律

更推荐的纪律是：

- 默认 joinable，只有明确不需要结果且生命周期合同清楚时才 detach
- 不做“裸 fire-and-forget 线程”
- 不把“进程结束时都会清掉”当正常资源管理策略

## 10. 自测题 / 验证入口

1. 为什么“线程比进程轻”不足以支撑真实并发设计？
2. 同一进程内线程共享什么、独享什么？这为什么既带来优势，也带来同步代价？
3. 为什么一个 joinable 线程结束后仍需要 `pthread_join()` 或 `pthread_detach()`？
4. 为什么线程不是 CPU 核心的同义词？调度器真正选择的是什么？
5. 当问题主要是等待链表达而不是并行执行时，为什么线程往往不是最贴切抽象？
6. 为什么说在线程模型里，“共享地址空间”既是通信优势，也是正确性风险源头？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- 线程池 sizing 与 worker 生命周期设计
- mutex / condition variable / semaphore 的选择
- CPU 绑定、优先级和实时线程调度判断
- 线程与进程边界选择：共享内存 vs 隔离故障域
- 线程与协程混合系统：谁负责并行，谁负责控制流结构

## 12. 未解问题与继续深挖

1. Linux NPTL、`clone` 标志、glibc pthread API 之间，最小可调用实现模型怎样再压缩一层？
2. 在线程池、NUMA、cache locality、false sharing 叠加时，线程数量和绑定策略怎样系统建模？
3. 协程、线程池、阻塞 IO 隔离线程这三层混合时，取消、背压和生命周期合同怎样最不易错？

## 13. 参考资料

以下“当前实践”相关内容的核对日期均为 `2026-03-23`。

- Linux `pthreads(7)`: https://man7.org/linux/man-pages/man7/pthreads.7.html
- Linux `pthread_create(3)`: https://man7.org/linux/man-pages/man3/pthread_create.3.html
- Linux `pthread_join(3)`: https://man7.org/linux/man-pages/man3/pthread_join.3.html
- Linux `pthread_detach(3)`: https://man7.org/linux/man-pages/man3/pthread_detach.3.html
- Linux `clone(2)`: https://man7.org/linux/man-pages/man2/clone.2.html
- Linux `sched(7)`: https://man7.org/linux/man-pages/man7/sched.7.html
