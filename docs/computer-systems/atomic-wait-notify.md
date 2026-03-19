---
doc_id: computer-systems-atomic-wait-notify
title: Atomic Wait / Notify：当你只是在等一个原子值变化时，不必先上 condition variable
concept: atomic_wait_notify
topic: computer-systems
created_at: '2026-03-16T14:56:22+08:00'
updated_at: '2026-03-19T21:20:00+08:00'
source_basis:
  - cxx_draft_atomics_wait_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_intro_races_2026_03_16
  - linux_futex_man_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_wait_notify_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/condition-variable.md
  - docs/computer-systems/modification-order.md
open_questions:
  - 在真实线程池和 runtime 里，`atomic_wait` 相对 condition variable 的收益边界如何量化？
  - `atomic_wait` 与 ABA 问题结合时，哪些等待协议最值得整理成专门案例？
---

# Atomic Wait / Notify：当你只是在等一个原子值变化时，不必先上 condition variable

## 1. 这份文档要帮你学会什么

这份文档的目标，是把 `atomic_wait` / `notify` 从“C++20 新 API”压成一个你以后可以稳定选型的等待模型。

读完后，你应该至少能做到：

- 说清 `atomic_wait` / `notify` 到底在解决什么问题
- 说清它和 polling、condition variable、semaphore 的边界
- 理解为什么它适合“等待单个 atomic 值变化”，而不是复杂共享条件
- 知道 wait / notify 的可唤醒条件和 ABA 风险
- 知道什么时候它比 condition variable 更直接

一句话先给结论：

**`atomic_wait` / `notify` 是围绕“单个 atomic object 的值变化”建立的等待机制；它比轮询高效，比 condition variable 更窄更直接，但也只适合那类更窄的问题。**

## 2. 一句话结论 / 问题定义

很多并发代码其实不是在等“复杂条件”，而是在等：

- 一个状态位变了
- 一个 phase number 前进了
- 一个标志从旧值变成新值

如果你用：

- `sleep_for` 轮询
- 自旋忙等
- 或为此专门搭一套 mutex + condition variable

都有可能不够贴题。

当前草案 `atomics.wait` 直接给出了更窄的机制：

- wait 原子值是否还等于旧值
- 如果是，就阻塞
- notify 在值更新后唤醒等待者

## 3. 对象边界与相邻概念

### 3.1 `atomic_wait` / `notify` 管什么

它主要管：

- 围绕单个 atomic object 的值变化进行等待与唤醒
- 比轮询更高效地等待值变化

### 3.2 它不等于什么

它不等于：

- condition variable
- 复杂 predicate 等待
- semaphore 令牌模型
- 事务式状态机
- 自动避免 ABA

### 3.3 和几个相邻概念的边界

**`atomic_wait` vs polling**

当前草案 `atomics.wait` 明确说：

- atomic waiting / notifying 提供了一种比 polling 更高效的机制，去等待 atomic object 的值变化

所以它首先是在替代“傻等 + 一直读”。

**`atomic_wait` vs condition variable**

- `atomic_wait` 适合单个 atomic 对象的值变化
- condition variable 适合“共享状态 + predicate + mutex”这种更复杂等待

**`atomic_wait` vs semaphore**

- `atomic_wait` 等的是“某个值是否还等于 old”
- semaphore 等的是“令牌计数是否大于 0”

## 4. 核心结构

理解 `atomic_wait` / `notify`，至少要抓住下面七个构件。

### 4.1 等待对象：某个 atomic object `M`

整个机制都围绕同一个 atomic object 展开。

### 4.2 old 值

wait 的语义不是“等到某个未来值”，而是：

- 先 load
- 如果当前值和 `old` 相等，就阻塞
- 不等就直接返回

### 4.3 eligible to be unblocked

当前草案定义了一组精确条件：  
一次 wait 只有在满足这些条件时，才“有资格”被某次 notify 解除阻塞。

### 4.4 modification order 参与判断

当前草案要求：

- waiter 在观察到 side effect `X` 后阻塞
- 某个 side effect `Y` 在 modification order 中晚于 `X`
- 且 `Y` happens before notify 调用

这说明 wait / notify 不是纯粹的操作系统睡眠接口，而是明确建在 atomic 对象历史上的。

### 4.5 spurious wakeup

wait 也可能被 spuriously unblocked。  
所以工程上依然要按“醒来重新检查值”建模。

### 4.6 ABA 风险

当前草案 note 直接提醒：

- 程序不保证观察到 transient atomic values
- 这就是 ABA 问题
- 条件只短暂满足，wait 仍可能继续阻塞

### 4.7 notify 不存状态

notify 是唤醒动作，不是状态本身。  
真正的状态仍然存在 atomic object 里。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最基本的等待链

```cpp
std::atomic<int> state{0};

// waiter
state.wait(0, std::memory_order_acquire);
int v = state.load(std::memory_order_acquire);

// notifier
state.store(1, std::memory_order_release);
state.notify_one();
```

这里真正发生的事是：

1. waiter 先看 `state` 是否还等于 `0`
2. 如果是，就阻塞
3. notifier 把 `state` 改成 `1`
4. notifier 调用 `notify_one()`
5. waiter 被唤醒后重新检查值，发现已经不是 `0`，返回

这套模型值钱的地方在于：

- 等待和状态绑在同一个 atomic 对象上
- 不需要额外 mutex 保护一个更大的 predicate

### 5.2 为什么 notify 必须和状态更新配套想

如果你只盯着 `notify_one()`，而不先问：

- 它通知之前，对象值是否真的已经变了
- 这个变化是否 happens before notify

那你就会把 wait / notify 误当成“纯事件通道”。

当前草案不是这么定义它的。  
wait 能否被 notify 合法解除阻塞，明确依赖：

- side effect 在 modification order 上是否前进
- 该 side effect 是否 happens before notify

### 5.3 `atomic_wait` 不是复杂 predicate 的替代品

如果你真正的条件是：

- 队列不为空
- 并且 shutdown 标志为假
- 并且还有容量

那你已经超出了“等一个 atomic 值变化”的对象边界。  
此时 condition variable 往往更贴题。

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

`atomic_wait` / `notify` 的好处是：

- 模型窄
- API 贴题
- 不用先搭 mutex + predicate

代价是：

- 只适合单对象状态变化
- 更容易忽视 ABA、spurious wakeup 和状态协议设计

### 6.2 常见失败模式

**失败模式 1：把 notify 当状态本身**

真正受等待的是 atomic object 的值，不是 notify 调用。

**失败模式 2：继续写 `sleep_for` 轮询**

如果你只是等待单个 atomic 值变化，这通常已经不是当前更稳的主路径。

**失败模式 3：明明是复杂 predicate，还硬用 `atomic_wait`**

这样常常会把逻辑拆碎，最后比 condition variable 更难维护。

**失败模式 4：忘记 ABA**

值如果短暂离开 old 又回到 old，wait 可能看不到那次瞬时满足。

**失败模式 5：醒来后不重新检查**

wait 可能 spurious wakeup，也可能被唤醒时状态已再次变化。

## 7. 应用场景

### 7.1 阶段位 / phase flag

一个线程等待 phase number 前进，特别适合。

### 7.2 单对象状态机

例如等待 “not ready -> ready” 或 “queued -> published” 这种单对象切换。

### 7.3 runtime 停车 / 唤醒

轻量等待单个原子状态时，比 condition variable 更直接。

## 8. 工业 / 现实世界锚点

### 8.1 Linux futex

Linux `futex(2)` 文档把 futex 描述为：

- waiting until a certain condition becomes true
- shared-memory synchronization 的 blocking construct
- user space fast path + kernel only when likely to block

这和 `atomic_wait` / `notify` 的现实工程直觉非常接近：  
都是围绕共享内存中的一个小状态字做等待/唤醒。

### 8.2 C++ runtime 和标准库

当前草案把 atomic waiting / notifying 正式纳入 `<atomic>`，说明这类等待已经不是平台私货，而是主流并发模型的一部分。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 只要等待目标真的是“单个 atomic 值变化”，优先考虑 `atomic_wait` / `notify`
- 先更新状态，再 notify
- wait 醒来后继续按“重新检查值”的方式建模
- 一旦条件变成复杂 predicate，就升级到 condition variable + mutex

### 9.2 已经过时、明显不推荐或需要带语境理解的路径

**`sleep_for` / 轮询式等待**

如果问题本身只是等一个 atomic 值变化，这通常不是当前更稳的路径。  
当前更推荐的替代是 `atomic_wait` / `notify`。

**用 condition variable 包一层单对象 atomic 等待**

这不一定错，但常常比问题本身更重。  
当前更直接的替代是 `atomic_wait` / `notify`。

### 9.3 什么时候别用 `atomic_wait`

如果你的等待条件依赖：

- 多个共享字段
- 复杂谓词
- 配套互斥不变量

那就该回到：

- condition variable
- mutex
- semaphore

这类更贴题的原语。

## 10. 自测题 / 验证入口

1. `atomic_wait` / `notify` 适合解决哪一类等待问题？
2. 为什么 notify 不是状态本身，而只是唤醒动作？
3. wait 合法被 notify 解除阻塞，为什么要依赖 modification order 和 happens-before？
4. 为什么 `atomic_wait` 不能自动解决 ABA？
5. 在什么情况下你应该退回 condition variable，而不是继续用 `atomic_wait`？
6. 为什么“醒来重新检查值”仍然是必要动作？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- futex 风格等待
- 单对象 phase/flag 协议
- `modification order` 与可唤醒条件
- `condition_variable` 与 `atomic_wait` 的选型判断

迁移时最关键的问题始终是：

- 我真的只是等一个 atomic 值变化吗？
- 这个变化是否已经通过对象历史和 happens-before 落地？
- 我是不是把更复杂的问题硬塞给了 `atomic_wait`？

## 12. 未解问题与继续深挖

后续值得继续单独拆的点包括：

- ABA 与 wait / notify 的更系统协议设计
- `atomic_wait` 与 futex、park/unpark 模型的更细对应关系
- 多等待者场景下 notify 策略与惊群成本的系统化分析

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `atomics.wait`: https://eel.is/c++draft/atomics.wait
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- `futex(2)` Linux manual page: https://man7.org/linux/man-pages/man2/futex.2.html
