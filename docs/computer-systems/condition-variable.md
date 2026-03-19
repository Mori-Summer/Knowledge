---
doc_id: computer-systems-condition-variable
title: Condition Variable：你等的不是通知，而是条件何时在同步关系下成立
concept: condition_variable
topic: computer-systems
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-19T21:20:00+08:00'
source_basis:
  - cxx_draft_thread_condition_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_atomics_wait_2026_03_16
  - cxx_draft_intro_races_2026_03_16
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_wait_notify_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/synchronizes-with.md
open_questions:
  - `atomic_wait` / `notify` 什么时候已经足够，什么时候仍然必须上 condition variable + mutex？
  - 在高负载线程池和 work stealing 系统里，condition variable 的竞争与唤醒策略还能怎样更系统地建模？
---

# Condition Variable：你等的不是通知，而是条件何时在同步关系下成立

## 1. 这份文档要帮你学会什么

这份文档的目标，不是教你背 `wait` / `notify_one` 的 API，而是让你真正理解 condition variable 背后的等待模型。

读完后，你应该至少能做到：

- 说清 condition variable 到底在解决什么问题
- 说清它和 mutex、predicate、semaphore、`atomic_wait` 的边界
- 理解为什么 wait 不是“等通知”，而是“等条件在同步关系下成立”
- 知道为什么必须用 predicate / loop，而不是裸等一次
- 知道什么时候 condition variable 是合适工具，什么时候 `atomic_wait` 更直接

一句话先给结论：

**condition variable 是让线程在某个条件未满足时高效阻塞、在条件可能变化后被重新调度的同步原语；真正受保护和被等待的是 predicate 对应的共享状态，而不是通知动作本身。**

## 2. 一句话结论 / 问题定义

并发里经常会遇到这种需求：

- 线程 A 暂时不能继续，因为某个条件还不成立
- 它不想忙等浪费 CPU
- 条件一旦被别的线程改变，它希望被唤醒再检查

这就是 condition variable 要解决的问题。

如果没有它，程序往往只能在两个坏选项之间摇摆：

- 自旋 / 轮询，浪费 CPU
- `sleep_for` 式猜时间，既不可靠也不高效

## 3. 对象边界与相邻概念

### 3.1 condition variable 管什么

它主要管：

- 线程在条件未满足时阻塞
- 条件可能改变时被通知并重新竞争执行
- 与 mutex 配合，围绕共享状态建立安全等待模型

### 3.2 它不等于什么

它不等于：

- 条件本身
- 状态存储本身
- mutex 本身
- “通知到了就一定能继续”
- semaphore
- `atomic_wait`

### 3.3 和几个相邻概念的边界

**condition variable vs mutex**

- mutex 保护共享状态和临界区
- condition variable 管“条件不成立时怎么睡，条件可能变化时怎么醒”

它们通常配套出现，但职责不同。

**condition variable vs predicate**

- condition variable 不是条件
- predicate 才是你真正等待的业务条件

这就是为什么 wait 必须围绕 predicate 写，而不是只写“收到通知就继续”。

**condition variable vs `atomic_wait`**

- condition variable 适合“有一组共享状态 + 一个 mutex + 条件判断”这种等待
- `atomic_wait` 更适合“只等一个原子值变化”这种更窄的场景

## 4. 核心结构

理解 condition variable，至少要抓住下面七个构件。

### 4.1 共享状态

condition variable 不自己存条件。  
真正的条件存在于共享状态里。

### 4.2 predicate

你真正等待的是 predicate 为真，而不是通知本身。

### 4.3 mutex

共享状态通常必须由 mutex 保护，否则 wait / notify 就会失去可推理基础。

### 4.4 wait 的三段原子部分

当前草案明确写道，`wait` / `wait_for` / `wait_until` 执行时分三段原子部分：

1. 释放 mutex 并进入等待
2. 被解除阻塞
3. 重新获取 lock

### 4.5 notify 的原子执行

当前草案明确写道：`notify_one` 和 `notify_all` 的执行是 atomic 的。

### 4.6 单一总序

当前草案还规定：  
所有 `notify_one`、`notify_all` 以及各次 wait 的三部分执行，行为上就像处在一个与 `happens-before` 一致的单一未指定总序里。

### 4.7 spurious wakeup

等待线程被唤醒，不等于条件一定已经成立。  
这就是为什么 predicate 和循环是设计核心，而不是“API 使用小技巧”。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 标准用法的主链

```cpp
std::mutex m;
std::condition_variable cv;
bool ready = false;

// Thread A
{
    std::lock_guard<std::mutex> g(m);
    ready = true;
}
cv.notify_one();

// Thread B
std::unique_lock<std::mutex> lk(m);
cv.wait(lk, [] { return ready; });
```

这里真正发生的事是：

1. 共享状态 `ready` 由 mutex 保护
2. 线程 A 持锁修改 `ready`
3. A 解锁后通知 `cv`
4. 线程 B 在 wait 中释放锁、睡眠、被唤醒后重新获取锁
5. B 在重新拿到锁后再次检查 predicate

这套模式值钱的地方在于：

- 不是“notify 让 ready magically 变真”
- 而是“共享状态在锁保护下被改变，waiter 被唤醒后在同步关系下重新检查它”

### 5.2 为什么必须围绕 predicate 写

如果你写成：

```cpp
cv.wait(lk);
// 然后默认条件已成立
```

那你会遇到两个问题：

- spurious wakeup
- 你醒来时，条件可能仍然不成立，或者已经被别的线程再次改变

所以真正稳的模型不是“等通知”，而是：

- 等待期间睡眠
- 醒来后重新检查条件
- 条件不成立就继续等

### 5.3 为什么 `sleep_for` 不是替代品

很多新手最早写的是：

```cpp
while (!ready) {
    std::this_thread::sleep_for(10ms);
}
```

这看起来简单，但它的问题是：

- 没有正式等待协议
- 不是精确唤醒
- 延迟和 CPU 浪费都不可控

condition variable 的真正价值，就是把这件事变成受同步语义保护的等待模型。

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

condition variable 的优点是：

- 不忙等
- 和 mutex 协作稳定
- 适合复杂条件

代价是：

- 必须严格维护 predicate / mutex 纪律
- 容易被误用成“通知即事件”
- 多等待者场景会有惊群和竞争成本

### 6.2 常见失败模式

**失败模式 1：wait 不带 predicate**

这是最常见的误用之一。  
你不是在等通知，而是在等条件成立。

**失败模式 2：修改共享状态时不持同一把锁**

这样 wait / notify 的推理基础会被破坏。

**失败模式 3：把 notify 当条件本身**

notify 只是“条件可能变化了”的提示，不是“条件一定已经成立”的证明。

**失败模式 4：用 `sleep_for` 轮询替代 condition variable**

这通常又慢又脆，且不具备同样同步语义。

**失败模式 5：明明只是等一个原子值变化，却还强上 condition variable**

在更窄的“等单个 atomic 值变化”场景里，`atomic_wait` 可能更直接。

## 7. 应用场景

### 7.1 生产者-消费者

队列为空时消费者睡眠，有新任务时被唤醒。

### 7.2 线程池 / 工作队列

没有任务时工作线程睡眠，有任务入队后被通知。

### 7.3 启动 / 关闭协调

某个阶段条件未满足前阻塞，条件成立后统一推进。

## 8. 工业 / 现实世界锚点

### 8.1 标准库线程池类实现模式

虽然标准库本身没提供线程池，但工业里线程池、任务队列、后台 worker 模式几乎都会围绕：

- mutex
- condition variable
- predicate

来写等待链路。

### 8.2 `atomic_wait` 的出现

当前草案 `atomics.wait` 明确指出：原子等待/通知提供了一种“比 polling 更高效地等待 atomic object 值变化”的机制。  
这说明现实工程已经把“只等单个 atomic 值变化”的场景，从 condition variable 里进一步剥离出来了。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 总是把共享状态、mutex 和 predicate 放在第一位
- `wait` 默认写成带 predicate 的形式，或者显式 while-loop
- 修改 predicate 对应的共享状态时，保持锁纪律一致
- 如果只是等待一个 atomic 值变化，认真考虑 `atomic_wait` / `notify`

### 9.2 已经过时、明显不推荐或需要带语境理解的路径

**`sleep_for` / 轮询式等待**

这不是当前推荐的正式等待模型。  
当前更稳的替代是：

- 复杂共享状态 -> condition variable + mutex + predicate
- 单 atomic 状态 -> `atomic_wait` / `notify`

**把 condition variable 当“通知即事件”的工具**

这条路会直接诱导你写出不带 predicate 的 wait。  
当前更稳的替代是：始终以条件本身为中心建模，而不是以通知动作为中心。

### 9.3 什么时候别用 condition variable

如果你只是等待一个 atomic 值的变化，不涉及复杂共享状态，那么：

- `atomic_wait`
- `atomic_notify_one`
- `atomic_notify_all`

通常更贴近问题本身。

## 10. 自测题 / 验证入口

1. 为什么说 condition variable 等的不是通知，而是条件？
2. 为什么 wait 必须和 predicate / while-loop 绑定？
3. 当前草案把 wait 分成哪三段原子部分，这个设计价值是什么？
4. 为什么修改共享状态时必须维持与 wait 方一致的锁纪律？
5. 在什么情况下你应该考虑 `atomic_wait` 而不是 condition variable？
6. 为什么 `sleep_for` 轮询不是稳定替代方案？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- `happens-before` / `synchronizes-with` 的等待图分析
- 线程池、工作队列、关闭协调
- `atomic_wait` 与 condition variable 的选型判断
- semaphore、event、future/promise 等等待抽象的比较

迁移时最关键的问题始终是：

- 我真正等待的条件是什么？
- 这个条件对应的共享状态是否被正确保护？
- 我是需要复杂条件等待，还是只需要等一个 atomic 值变化？

## 12. 未解问题与继续深挖

后续值得继续单独拆的点包括：

- condition variable 与 semaphore / event 的模型对比
- `atomic_wait` 在高并发路径里相对 condition variable 的实际收益
- 多等待者场景下 `notify_one` / `notify_all` 的策略优化

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- C++ draft `thread.mutex.requirements.mutex`: https://eel.is/c++draft/thread.mutex.requirements.mutex
- C++ draft `atomics.wait`: https://eel.is/c++draft/atomics.wait
- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
