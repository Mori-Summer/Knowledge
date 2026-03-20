---
doc_id: computer-systems-condition-variable
title: Condition Variable：你等的不是通知，而是条件何时在同步关系下成立
concept: condition_variable
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-20T15:37:08+08:00'
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
  - docs/computer-systems/mutex.md
  - docs/computer-systems/atomic-wait-notify.md
open_questions:
  - `atomic_wait` / `notify` 什么时候已经足够，什么时候仍然必须上 condition variable + mutex？
  - 在高负载线程池和 work stealing 系统里，condition variable 的竞争与唤醒策略还能怎样更系统地建模？
  - 多等待者场景下，`notify_one` / `notify_all` 的最稳决策框架是否值得单独扩写？
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

但 condition variable 最容易被误解成“通知机制”。更准确地说，它真正解决的是：

- 当 predicate 不成立时，如何安全睡眠
- 当 predicate 可能变化后，如何在同步关系下重新检查
- 如何把等待、状态保护和唤醒动作接成一条可推理链

换句话说：

- **你等的不是通知**
- **你等的是条件重新变真**

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
- `atomic_wait` 更适合“只等一个 atomic 值变化”这种更窄的场景

**condition variable vs semaphore**

- semaphore 更适合 permit / 槽位数
- condition variable 更适合复杂 predicate

### 3.4 最关键的边界句

可以把它压成一句话：

**如果等待条件依赖多个共享字段的一致关系，默认先想 condition variable + mutex；如果你只是在等一个 atomic 值变化，先别急着上它。**

## 4. 核心结构

### 4.1 condition variable 的最小模型

理解 condition variable，至少要抓住下面七个构件。

| 构件 | 它是什么 | 缺了会怎样 |
| --- | --- | --- |
| 共享状态 | 真正决定能不能继续的数据 | 你不知道自己在等什么 |
| predicate | 对共享状态的业务条件判断 | wait 会退化成“等通知” |
| mutex | 保护共享状态与 predicate 读取 | 等待链失去可推理基础 |
| wait 的三段原子部分 | 解锁并睡、被唤醒、重新拿锁 | 不理解等待链的关键切面 |
| notify | 宣告“条件可能变化了” | 等待者无法被推进 |
| 单一总序直觉 | wait/notify 在条件对象上的统一历史 | 很难推理谁先睡、谁先唤醒 |
| loop discipline | 醒来后重新检查 predicate | 会被 spurious wakeup 和竞争打穿 |

### 4.2 共享状态才是本体

condition variable 不自己存条件。真正的条件存在于共享状态里。

例如：

- 队列是否为空
- shutdown 是否已经开始
- phase 是否推进
- 某个资源是否准备好

所以你真正等待的不是 condition variable，而是：

- 共享状态进入某个满足 predicate 的配置

### 4.3 predicate 是主角，notify 只是提示

这是 condition variable 最重要的认知转向。

| 误解 | 正确理解 |
| --- | --- |
| “我等的是通知” | “我等的是 predicate 成立” |
| “notify 来了就能继续” | “notify 只说明条件可能变化了，要重新检查” |
| “通知是事件本体” | “事件本体是共享状态变化” |

这就是为什么正确代码总是长成：

- `wait(lock, predicate)`
- 或 `while (!predicate()) wait(lock);`

### 4.4 mutex 是等待模型的地基

共享状态通常必须由 mutex 保护，否则 wait / notify 就会失去可推理基础。

mutex 在这里提供两样东西：

- 共享状态的一致视图
- 修改方与等待方之间的同步边基础

如果 predicate 依赖的状态不是在同一把 mutex 下维护，condition variable 代码就会很快失真。

### 4.5 wait 的三段原子部分

当前草案明确写道，`wait` / `wait_for` / `wait_until` 执行时分三段原子部分：

1. 释放 mutex 并进入等待
2. 被解除阻塞
3. 重新获取 lock

这三段很关键，因为它们解释了：

- 为什么 waiter 能在睡眠时让别人修改状态
- 为什么醒来后不能直接继续，而要重新拿锁
- 为什么 wait/notify 不是“简单睡眠 API”

### 4.6 notify 的原子执行和单一总序

当前草案明确写道：

- `notify_one` 和 `notify_all` 的执行是 atomic 的
- 各次 wait 的三部分与 notify 行为上就像处在一个与 `happens-before` 一致的单一未指定总序里

这并不意味着你能精确预测调度顺序，但它给了你一个稳定推理框架：

- 谁先进入等待
- 谁后发出通知
- waiter 在什么历史上被解除阻塞

### 4.7 一页纸选择模板

如果你要快速判断问题是不是 condition variable，先过下面这张表：

| 问题 | 倾向选择 |
| --- | --- |
| 我在等复杂 predicate | condition variable + mutex |
| 我在等一个 atomic 状态位变化 | `atomic_wait` |
| 我在等 permit / 槽位数 | semaphore |
| 我在保护多字段不变量并且可能要睡眠等待 | mutex + condition variable |

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
3. A 释放锁
4. A 调用 `notify_one()`
5. 线程 B 在 wait 中释放锁、睡眠、被唤醒后重新获取锁
6. B 在重新拿到锁后再次检查 predicate

这套模式值钱的地方在于：

- 不是“notify 让 `ready` magically 变真”
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

### 5.3 为什么共享状态修改必须遵守同一把锁纪律

最容易被低估的问题，是“修改共享状态时到底要不要用同一把锁”。

如果 waiter 是这样等的：

- 在 `m` 下检查 predicate
- `wait(m, predicate)`

那修改方也必须把 predicate 对应的状态在同一把 `m` 下改掉。否则你会失去下面这些能力：

- 稳定说明 predicate 是何时变化的
- 稳定说明 waiter 醒来后看到的是哪一版状态
- 避免“状态变了但等待链没对上”的错位

### 5.4 丢失唤醒链：为什么“先通知再等”会出事

condition variable 最经典的坑之一，就是丢失唤醒。

典型坏链是：

1. 线程 A 还没真正进入等待
2. 线程 B 改了状态并发了 notify
3. 线程 A 随后去 wait
4. 如果它没有通过 predicate 先检查共享状态，就可能错过那次关键变化

这就是为什么：

- predicate 先检查
- wait 必须和共享状态同一套锁纪律绑定

它们不是“最佳实践”，而是 condition variable 成立的结构条件。

### 5.5 `notify_one` 和 `notify_all` 的选择链

很多工程问题不是“要不要 notify”，而是“通知一个还是通知全部”。

可以这样理解：

| 情况 | 倾向选择 |
| --- | --- |
| 一次状态变化通常只够推进一个等待者 | `notify_one` |
| 一次状态变化可能让很多等待者都需要重新评估 | `notify_all` |
| 多等待者共享复杂 predicate，但只有部分会继续 | 先谨慎评估，避免惊群 |

核心不是“哪个更快”，而是：

- 一次状态变化到底能满足多少人
- 唤醒过多线程是否只会制造额外竞争

### 5.6 为什么 `sleep_for` 不是替代品

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
- 不能自然复用 mutex 下的共享状态模型

condition variable 的真正价值，就是把这件事变成受同步语义保护的等待模型。

### 5.7 真正的因果核心

把这一节压成一句最值得记住的话：

**condition variable 的本体不是通知通道，而是“在 mutex 保护的共享状态上，当 predicate 不成立时睡下、当 predicate 可能改变后醒来重查”的等待协议。**

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

### 6.2 六类高频失败模式

**失败模式 1：wait 不带 predicate**

这是最常见的误用之一。你不是在等通知，而是在等条件成立。

**失败模式 2：修改共享状态时不持同一把锁**

这样 wait / notify 的推理基础会被破坏。

**失败模式 3：把 notify 当条件本身**

notify 只是“条件可能变化了”的提示，不是“条件一定已经成立”的证明。

**失败模式 4：用 `sleep_for` 轮询替代 condition variable**

这通常又慢又脆，且不具备同样同步语义。

**失败模式 5：明明只是等一个原子值变化，却还强上 condition variable**

在更窄的“等单个 atomic 值变化”场景里，`atomic_wait` 可能更直接。

**失败模式 6：`notify_all` 用得过重**

如果一次状态变化只能推进一个人，盲目 `notify_all` 往往只会制造竞争和惊群。

### 6.3 现场症状与优先回查方向

| 现场症状 | 优先回查什么 | 常见真实原因 |
| --- | --- | --- |
| 偶发卡住，像是“丢了一次信号” | predicate 与锁纪律 | 把 condition variable 当事件用，没围绕状态建模 |
| 被唤醒了却还是不能继续 | 是否有 loop / predicate | 醒来后没重新检查条件 |
| 系统一通知就一堆线程抢锁 | `notify_all` 选择 | 一次状态变化只能推进少量线程，却唤醒过多 |
| 等待逻辑难以解释 | 共享状态边界 | predicate 依赖的字段没被一致保护 |
| 本该是简单等待却代码很重 | 问题是否只是单 atomic 值变化 | 原本更适合 `atomic_wait` |

### 6.4 为什么“通知”这个词特别容易误导人

condition variable 被叫作“通知变量”很容易把人带偏，因为它让人不自觉地把注意力放在：

- 谁发了通知
- 通知有没有送达

而真正决定正确性的其实是：

- 共享状态是什么
- predicate 是什么
- 状态变化是否在锁纪律下发生
- waiter 醒来后如何重新检查

所以最稳的学习动作是把“通知”降级成辅助动作，把“条件”重新提到中心。

### 6.5 一个很实用的判别句式

如果你看到：

- **等待条件依赖多个字段**，优先想 condition variable + mutex
- **代码在等“某个通知”**，先反问真实 predicate 是什么
- **等待链经常丢事件**，先查是否缺 predicate 和同一把锁纪律
- **问题只是等一个 atomic 值变化**，先反问是不是该换 `atomic_wait`

## 7. 应用场景

### 7.1 生产者-消费者

队列为空时消费者睡眠，有新任务时被唤醒。

这里最重要的不是 `notify_one()`，而是：

- 队列状态由 mutex 保护
- “队列非空”是 predicate

### 7.2 线程池 / 工作队列

没有任务时工作线程睡眠，有任务入队后被通知。

这类场景里，condition variable 的优势在于：

- 可以自然围绕“队列非空或系统关闭”这种复杂 predicate 建模
- 比“轮询任务队列”稳得多

### 7.3 启动 / 关闭协调

某个阶段条件未满足前阻塞，条件成立后统一推进。

典型等待条件通常不是单个标志位，而是：

- 初始化阶段完成
- 资源已准备好
- shutdown 状态已切换

### 7.4 多字段状态机等待

例如：

- `running && !paused && queue_not_empty`
- `closed || ready_count > 0`

这类“多个字段一起决定能不能继续”的场景，本来就是 condition variable 的主场。

## 8. 工业 / 现实世界锚点

### 8.1 标准把 wait 明确定义成三段原子部分

`thread.condition` 对 wait 的三段原子部分定义非常关键，因为它告诉你：

- condition variable 不是普通睡眠接口
- 它的核心是释放锁、等待、重新拿锁这条结构化链路

这为所有生产级 wait / notify 代码提供了语言级骨架。

### 8.2 `notify_one` / `notify_all` 与 wait 历史进入统一总序

标准把 notify 和 wait 的关键阶段放进单一未指定总序的建模里，这给了工程代码一个很重要的好处：

- 你可以从“同一个条件变量对象上的等待历史”去推理行为
- 而不是把每次唤醒想成完全无法建模的随机事件

### 8.3 `atomic_wait` 的出现本身就是一个现实锚点

当前草案 `atomics.wait` 明确指出：原子等待/通知提供了一种“比 polling 更高效地等待 atomic object 值变化”的机制。

这件事对 condition variable 的现实意义是：

- 标准已经承认有一类等待问题比 condition variable 更窄
- 所以当你还在用 condition variable 等单个 atomic 状态位时，就该反思是不是工具过重了

### 8.4 工程里的线程池、工作队列和关闭协调大量依赖这一模型

虽然标准库本身没有线程池，但现实工程里的线程池、任务队列和后台 worker 模式几乎都会围绕：

- mutex
- condition variable
- predicate

来写等待链路。

这不是偶然，而是因为这些场景里的等待条件天然就是复杂 predicate。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 总是把共享状态、mutex 和 predicate 放在第一位
- `wait` 默认写成带 predicate 的形式，或者显式 while-loop
- 修改 predicate 对应的共享状态时，保持锁纪律一致
- 根据一次状态变化能推进多少等待者，谨慎选择 `notify_one` 或 `notify_all`
- 如果只是等待一个 atomic 值变化，认真考虑 `atomic_wait` / `notify`

### 9.2 已经过时、明显不推荐或需要带语境理解的路径

| 路径 | 为什么旧/不稳 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| `sleep_for` / 轮询式等待 | 这不是正式等待模型 | 延迟不稳、CPU 浪费、缺少同步语义 | 复杂共享状态用 condition variable，单 atomic 状态用 `atomic_wait` |
| 把 condition variable 当“通知即事件”的工具 | 会直接诱导你写不带 predicate 的 wait | 容易丢条件、误判唤醒 | 始终以 predicate 为中心建模 |
| 修改共享状态时不走同一把锁 | 破坏等待协议地基 | 难以解释谓词何时成立 | 等待方和修改方统一锁纪律 |
| 默认 `notify_all` | 看起来保险 | 容易制造惊群和竞争 | 根据状态变化影响范围选择通知策略 |
| 明明只是等一个 atomic 值变化，还上 condition variable | 工具过重 | 代码更绕，维护成本更高 | 用 `atomic_wait` / `notify` |

### 9.3 什么时候别用 condition variable

如果你只是等待一个 atomic 值的变化，不涉及复杂共享状态，那么：

- `atomic_wait`
- `atomic_notify_one`
- `atomic_notify_all`

通常更贴近问题本身。

如果你的问题其实是 permit / 槽位数，也应优先考虑 semaphore。

## 10. 自测题 / 验证入口

1. 为什么说 condition variable 等的不是通知，而是条件？
2. 为什么 wait 必须和 predicate / while-loop 绑定？
3. 当前草案把 wait 分成哪三段原子部分，这个设计价值是什么？
4. 为什么修改共享状态时必须维持与 wait 方一致的锁纪律？
5. 在什么情况下你应该考虑 `atomic_wait` 而不是 condition variable？
6. 为什么 `sleep_for` 轮询不是稳定替代方案？
7. 为什么 `notify_one` 和 `notify_all` 的选择本质上是“这次状态变化能推进多少等待者”的问题？
8. 如果一个 bug 表现得像“丢了一次通知”，你最先该检查什么？
9. 为什么 condition variable 不负责存储条件？
10. 请用一句话解释：condition variable 真正保护的是什么结构。

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的核心句

理解了这篇文档后，你应该能把下面这句迁移出去：

**condition variable 的本体不是通知，而是围绕共享状态 predicate 的等待协议。**

### 11.2 可以迁移到哪些领域

- `happens-before` / `synchronizes-with` 的等待图分析
- 线程池、工作队列、关闭协调
- `atomic_wait` 与 condition variable 的选型判断
- semaphore、event、future/promise 等等待抽象的比较
- 多字段状态机等待

### 11.3 与相邻模型的关系

| 模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| mutex | 保护共享状态与不变量 | 补条件不成立时如何睡、条件变化时如何醒 |
| `atomic_wait` | 等单个 atomic 值变化 | 补复杂 predicate 等待 |
| semaphore | 管 permit / 槽位数 | 补多字段业务条件等待 |
| future/promise | 管一次性交付结果 | 补共享状态反复变化下的等待循环 |
| event 直觉 | 让人想到“通知到了” | 补为什么通知本身不是正确性中心 |

### 11.4 最值得保留的迁移动作

以后遇到任何等待逻辑，可以先问自己：

- 我真正等待的 predicate 是什么？
- 这个 predicate 对应的共享状态是否被同一把锁保护？
- 我需要复杂条件等待，还是只需要等一个 atomic 值变化？

只要这三问没答清，等待原语选型通常还不稳。

## 12. 未解问题与继续深挖

- condition variable 与 semaphore / event 的模型对比，是否值得单独拆篇？
- `atomic_wait` 在高并发路径里相对 condition variable 的实际收益，是否需要配套实测方法？
- 多等待者场景下 `notify_one` / `notify_all` 的策略优化，是否值得再补一篇方法文？

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- C++ draft `thread.mutex.requirements.mutex`: https://eel.is/c++draft/thread.mutex.requirements.mutex
- C++ draft `atomics.wait`: https://eel.is/c++draft/atomics.wait
- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
