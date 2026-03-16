---
doc_id: computer-systems-semaphore
title: Semaphore：当问题是“还有没有令牌”，而不是“谁拥有锁”
concept: semaphore
topic: computer-systems
created_at: '2026-03-16T14:56:22+08:00'
updated_at: '2026-03-16T14:56:22+08:00'
source_basis:
  - cxx_draft_thread_sema_2026_03_16
  - cxx_draft_thread_sema_cnt_2026_03_16
  - linux_sem_overview_man_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_admission_control_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/mutex.md
  - docs/computer-systems/condition-variable.md
open_questions:
  - 在高负载限流、连接池和 GPU/IO 提交队列里，semaphore 相对 condition variable 的收益边界如何量化？
  - binary_semaphore 与 event/parking token 之间，哪些工程模式最值得单独整理？
---

# Semaphore：当问题是“还有没有令牌”，而不是“谁拥有锁”

## 1. 这份文档要帮你学会什么

这份文档的目标，是把 semaphore 从“古老同步原语”压成一个你以后能稳定选型的令牌模型。

读完后，你应该至少能做到：

- 说清 semaphore 到底在解决什么问题
- 说清它和 mutex、condition variable、`atomic_wait` 的边界
- 理解为什么 semaphore 关心的是 permit / counter，不是 ownership
- 知道 `counting_semaphore` 和 `binary_semaphore` 分别适合什么
- 避免把 semaphore 误当成复杂共享不变量的通用保护工具

一句话先给结论：

**semaphore 管的是“有多少个许可可以被拿走”，而不是“谁拥有一段临界区”；它特别适合 admission control 和资源计数，不适合直接保护复杂共享状态不变量。**

## 2. 一句话结论 / 问题定义

很多并发问题的核心不是“谁能独占进来”，而是：

- 现在最多允许多少个线程继续
- 还有多少个资源槽位可以被占用

这正是 semaphore 要解决的问题。

当前工作草案 `thread.sema.general` 直接把 semaphore 定位为：

- lightweight synchronization primitive
- used to constrain concurrent access to a shared resource

## 3. 对象边界与相邻概念

### 3.1 semaphore 管什么

它主要管：

- permit / resource count
- acquire 令牌时是否需要阻塞
- release 令牌时是否能唤醒等待者

### 3.2 它不等于什么

它不等于：

- mutex ownership
- 复杂 predicate 等待
- 条件变量
- 一个普通计数器本身

### 3.3 和几个相邻概念的边界

**semaphore vs mutex**

- mutex 更关心“临界区的唯一拥有者”
- semaphore 更关心“还剩多少个 permit”

**semaphore vs condition variable**

- condition variable 更适合围绕共享状态和 predicate 等待
- semaphore 更适合“数量门槛 / 许可数量”

**semaphore vs `atomic_wait`**

- `atomic_wait` 等一个 atomic 值变化
- semaphore 则把“可继续”的条件直接压缩成内部计数器

## 4. 核心结构

理解 semaphore，至少要抓住下面六个构件。

### 4.1 internal counter

当前草案 `thread.sema.cnt` 明确写道：

- `counting_semaphore` 维护一个内部计数器
- acquire 时递减
- release 时递增

### 4.2 acquire / block

如果线程尝试 acquire 时计数器为 0，它会阻塞，直到别的线程 release。

### 4.3 release / unblock

release 会原子地增加计数器，并解除等待线程的阻塞。

### 4.4 `binary_semaphore`

当前草案还明确给出：

- binary semaphore 只有两种状态
- 当适用时，它应比“单位计数的 counting_semaphore 默认实现”更高效

### 4.5 synchronization relation

当前草案对 `release` 还给出了非常重要的语义：

- `release()` 的效果 strongly happens before 观察到这些效果的 `try_acquire()` 调用

这说明 semaphore 不只是计数器，它也参与同步关系。

### 4.6 spurious failure

当前草案允许 `try_acquire()` spuriously fail。  
所以“没拿到 permit”不总等于“计数器一定为 0”。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最基本的 permit 链

```cpp
std::counting_semaphore<8> slots(8);

// worker
slots.acquire();
use_resource();
slots.release();
```

这里真正发生的事是：

1. `slots` 表示剩余 permit 数
2. `acquire()` 试图拿走一个 permit
3. 如果没有 permit，就阻塞
4. `release()` 归还 permit，并可能唤醒等待者

这个模型和 mutex 的关键区别在于：

- 允许多个线程同时继续，只要 permit 还够

### 5.2 为什么它不适合直接保护复杂共享状态

如果你有一组复杂不变量，例如：

- 队列元素
- shutdown 标志
- 容量上限

semaphore 只能帮你表达“现在还有几个名额”。  
它不会自动替你保护那组状态本身。

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

semaphore 的好处是：

- 模型直接
- 天然适合限流、池化、并发度控制

代价是：

- 它不追踪 owner
- 它不天然保护复杂共享不变量
- permit 泄漏时问题会很隐蔽

### 6.2 常见失败模式

**失败模式 1：把 semaphore 当 mutex 替代**

如果你的真实问题是“保护一段复杂状态”，semaphore 往往不贴题。

**失败模式 2：忘记 release 对称性**

permit 没有归还，系统容量会被悄悄吃掉。

**失败模式 3：把 `try_acquire()` 失败当成绝对状态证明**

当前草案允许 spurious failure。

**失败模式 4：用 semaphore 表达复杂 predicate**

这会让模型越来越扭曲，通常该回到 condition variable 或 mutex。

## 7. 应用场景

### 7.1 连接池 / 资源池

控制同时可用资源数量，是 semaphore 最自然的场景。

### 7.2 并发度限制

限制同时运行的 worker、任务、请求数量。

### 7.3 生产者-消费者中的槽位控制

当问题是“还有几个空槽位/可消费槽位”时，semaphore 很贴题。

## 8. 工业 / 现实世界锚点

### 8.1 POSIX semaphores

Linux `sem_overview(7)` 明确说明：

- POSIX semaphores 可用于 process 和 thread 同步
- 有 named / unnamed 两种形式
- System V semaphores 是更老的接口，POSIX semaphores 更简单、设计更好

这说明 semaphore 不是教学玩具，而是操作系统层长期存在的主力同步模型之一。

### 8.2 C++20 `<semaphore>`

当前草案把 `counting_semaphore` / `binary_semaphore` 正式纳入标准库，说明 permit 模型已经是主流 C++ 并发工具箱的一部分。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 当问题本质是 permit / admission control 时，优先考虑 `std::counting_semaphore`
- 当只是 0/1 许可或简单 handoff，考虑 `binary_semaphore`
- 当问题变成复杂共享状态或 predicate，退回 mutex + condition variable

### 9.2 已经过时、明显不推荐或必须带语境理解的路径

**System V semaphores 作为默认首选**

Linux `sem_overview(7)` 已明确指出：

- System V semaphores 是 older API
- POSIX semaphores 提供了更简单、更好设计的接口

对现代进程内 C++ 代码而言，当前更直接的替代通常是：

- `std::counting_semaphore`
- 或在需要进程共享/OS 级语义时使用 POSIX semaphore

**用 condition variable / 轮询硬拼 permit 模型**

如果问题本体就是“令牌数”，当前更贴题的替代通常就是 semaphore。

### 9.3 什么时候别用 semaphore

如果你的问题是：

- 保护复杂共享对象不变量
- 需要 owner 语义
- 需要复杂 predicate 等待

那就该考虑：

- mutex
- condition variable
- `atomic_wait`

## 10. 自测题 / 验证入口

1. semaphore 和 mutex 的本质差别是什么？
2. 为什么 semaphore 更像 permit 模型，而不是 owner 模型？
3. 为什么 permit 泄漏会让 bug 很隐蔽？
4. 为什么 semaphore 不适合直接保护复杂共享状态？
5. `binary_semaphore` 和 `counting_semaphore` 各适合什么问题？
6. 为什么说 System V semaphores 在现代默认路径里不是首选？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- admission control
- resource pool
- bounded queue
- `mutex` / `condition_variable` / `atomic_wait` 选型判断

迁移时最关键的问题始终是：

- 我真正控制的是“人数/名额”，还是“临界区拥有权”？
- 我需要 permit 计数，还是复杂状态保护？
- 我是不是把更复杂的不变量错误压成了“一个计数器”？

## 12. 未解问题与继续深挖

后续值得继续单独拆的点包括：

- `binary_semaphore` 与 event / park token 模型的对照
- semaphore 与 backpressure 设计之间的系统关系
- permit 泄漏和公平性问题的更系统排查模板

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `thread.sema`: https://eel.is/c++draft/thread.sema
- C++ draft `thread.sema.cnt`: https://eel.is/c++draft/thread.sema.cnt
- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- `sem_overview(7)` Linux manual page: https://man7.org/linux/man-pages/man7/sem_overview.7.html
