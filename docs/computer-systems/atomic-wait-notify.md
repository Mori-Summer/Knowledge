---
doc_id: computer-systems-atomic-wait-notify
title: Atomic Wait / Notify：当你只是在等一个原子值变化时，不必先上 condition variable
concept: atomic_wait_notify
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:56:22+08:00'
updated_at: '2026-04-08T15:51:00+08:00'
source_basis:
  - cxx_draft_atomics_wait_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_intro_races_2026_03_16
  - linux_futex_man_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
  - methodology_document_generation_methodology
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_wait_notify_modeling
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/condition-variable.md
  - docs/computer-systems/modification-order.md
  - docs/computer-systems/fence.md
open_questions:
  - 在真实线程池和 runtime 里，`atomic_wait` 相对 condition variable 的收益边界如何量化？
  - `atomic_wait` 与 ABA 问题结合时，哪些等待协议最值得整理成专门案例？
  - 多等待者场景下，`notify_one` / `notify_all` 的策略与惊群成本是否值得再做专篇？
---

# Atomic Wait / Notify：当你只是在等一个原子值变化时，不必先上 condition variable

## 1. 这份文档要帮你学会什么

这份文档的目标，是把 `atomic_wait` / `notify` 从“C++20 新 API”压成一个你以后可以稳定选型的等待模型。

读完后，你应该至少能做到：

- 说清 `atomic_wait` / `notify` 到底在解决什么问题
- 说清它和 polling、condition variable、semaphore 的边界
- 理解为什么它适合“等待单个 atomic 值变化”，而不是复杂共享条件
- 知道 wait / notify 的可唤醒条件、spurious wakeup 和 ABA 风险
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

换句话说，它真正解决的是：

- “我只关心一个 atomic object 的值历史”
- “只要它不再是 old，我就能重新推进”

## 3. 对象边界与相邻概念

### 3.1 `atomic_wait` / `notify` 管什么

它主要管：

- 围绕单个 atomic object 的值变化进行等待与唤醒
- 比轮询更高效地等待值变化
- 把等待协议直接绑在该 atomic object 的 modification order 上

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

**`atomic_wait` vs 普通事件通道**

- notify 只是唤醒动作
- 真正被等待和检查的是 atomic object 的值

### 3.4 本文的时间边界

这篇文档的大部分内容是在建立一个可复用的等待模型，因此主体判断是偏 evergreen 的。

但下面这些内容带有明确时间边界：

- 当前标准草案中的措辞与章节位置
- “当前推荐实践、过时路径与替代”里的结论
- 现实世界锚点里对标准库与系统接口的定位

这些时间敏感内容在本篇文档里统一以 `2026-03-16` 为核对日期。  
本次升级主要是按统一规范补强结构、时间纪律和表达方式，不把它伪装成一次更晚日期的外部资料重验。

### 3.5 最关键的边界句

可以把它压成一句话：

**如果你真正等待的是“单个原子值不再等于 old”，优先考虑 `atomic_wait`；如果你真正等待的是“多个字段组成的 predicate”，不要硬塞给它。**

## 4. 核心结构

### 4.1 `atomic_wait` 的最小模型

理解 `atomic_wait` / `notify`，至少要抓住下面七个构件。

| 构件 | 它是什么 | 缺了会怎样 |
| --- | --- | --- |
| atomic object `M` | 整个等待协议围绕的唯一对象 | 等待目标失焦 |
| old 值 | waiter 当前不想看到的值 | wait 语义失去锚点 |
| modification order | 判断值历史是否前进的时间轴 | 无法严格说明什么叫“变了” |
| eligible-to-be-unblocked 条件 | 哪些 waiter 有资格被某次 notify 解除阻塞 | 会把 notify 误当成广播魔法 |
| notify 动作 | 唤醒等待者 | 值变了但等待者可能继续沉睡 |
| spurious wakeup | 唤醒不等于条件已满足 | 醒来后不重查会出错 |
| ABA 风险 | 值可能短暂变化后又回到 old | “发生过变化”不等于“你一定看见过变化” |

### 4.2 wait 的语义不是“等某个未来值”，而是“如果还是 old 就睡”

这是最值得记住的地方。

wait 的直观语义不是：

- “等它变成 1”

而是：

- “如果它现在还是 old，就阻塞”
- “醒来后再看它还是不是 old”

所以它天然更适合：

- phase number
- ready flag
- state enum
- generation counter

### 4.3 modification order 是它的真正历史轴

当前草案要求：

- waiter 在观察到 side effect `X` 后阻塞
- 某个 side effect `Y` 在 modification order 中晚于 `X`
- 且 `Y` happens before notify 调用

这说明 `atomic_wait` / `notify` 不是纯粹的操作系统睡眠接口，而是明确建在 atomic 对象历史上的。

这比 condition variable 更窄，但也更直接：

- 你不用单独再引入 predicate 对象
- 你直接围绕这个 atomic 的值历史建模

### 4.4 notify 不存状态

notify 是唤醒动作，不是状态本身。

真正的状态仍然存在 atomic object 里。

这意味着：

- 先更新值，再 notify，才是通常可推理的主路径
- 如果你只看 notify，不看值历史，就会把这套机制误想成事件通道

### 4.5 spurious wakeup 和 ABA 不是边角料

wait 也可能被 spuriously unblocked，所以工程上依然要按“醒来重新检查值”建模。

而标准 note 还直接提醒：

- 程序不保证观察到 transient atomic values
- 这就是 ABA 问题
- 条件只短暂满足，wait 仍可能继续阻塞

这两点合起来意味着：

- `atomic_wait` 不是“看见所有中间状态”的工具
- 它只是围绕“现在是否还等于 old”建立等待协议

### 4.6 一页纸选择模板

如果你要快速判断问题是不是 `atomic_wait`，先过下面这张表：

| 问题 | 倾向选择 |
| --- | --- |
| 我只是在等一个 atomic 状态位或序号变化 | `atomic_wait` |
| 我在等复杂 predicate | condition variable + mutex |
| 我在等 permit / 槽位数 | semaphore |
| 我只想避免轮询等待一个单对象状态 | `atomic_wait` |

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
- 不需要额外 mutex 去保护一个更大的 predicate

### 5.2 “先改值，再 notify”为什么几乎总是主路径

如果你只盯着 `notify_one()`，而不先问：

- 它通知之前，对象值是否真的已经变了
- 这个变化是否 happens before notify

那你就会把 wait / notify 误当成“纯事件通道”。

当前草案不是这么定义它的。wait 能否被 notify 合法解除阻塞，明确依赖：

- side effect 在 modification order 上是否前进
- 该 side effect 是否 happens before notify

这就是为什么更稳的直觉始终是：

- **先更新状态**
- **再 notify**

### 5.3 eligible-to-be-unblocked 链：notify 为什么不是“叫谁都醒”

waiter 不是因为“收到了某次 notify”就自动合理返回。它必须满足标准规定的 eligible 条件。

你可以把这条链压成这样：

1. waiter 先观察到某个旧 side effect `X`
2. waiter 因为值仍等于 old 而睡下
3. notifier 让该对象出现更晚的 side effect `Y`
4. `Y` happens before `notify_*`
5. waiter 因而有资格被解除阻塞

这条链说明：

- 真正的逻辑中心是 atomic 对象的值历史
- notify 只是“推动 waiter 重新检查”的手段

### 5.4 为什么 `atomic_wait` 不是复杂 predicate 的替代品

如果你真正的条件是：

- 队列不为空
- 并且 shutdown 标志为假
- 并且还有容量

那你已经超出了“等一个 atomic 值变化”的对象边界。此时 condition variable 往往更贴题。

这点非常重要，因为 `atomic_wait` 最容易被误用成：

- “我就拿一个 atomic 把所有条件都编码进去”

但一旦真实业务条件依赖多个字段，这种压缩常常会让协议更脆、更难维护。

### 5.5 ABA 链：为什么“值发生过变化”不等于 waiter 一定看见

ABA 的经典坏链是：

1. waiter 在等 `state != A`
2. notifier 把值改成 `B`
3. 很快又改回 `A`
4. waiter 最终只看见“它还是 A”

这说明：

- `atomic_wait` 能让你高效等“当前值不再等于 old”
- 但不能保证你看见所有中间态

如果你的协议依赖“状态一旦离开 A 就必须被观察到”，你通常需要：

- generation / version counter
- 更强的状态编码
- 或更高层协议

### 5.6 真正的因果核心

把这一节压成一句最值得记住的话：

**`atomic_wait` / `notify` 把等待协议直接绑在单个 atomic object 的值历史上：waiter 盯住 old，notifier 推进 modification order 并唤醒等待者，waiter 醒来后再确认自己是否终于脱离 old。**

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

`atomic_wait` / `notify` 的好处是：

- 模型窄
- API 贴题
- 不用先搭 mutex + predicate
- 对“等一个值变化”这类问题比 condition variable 更直接

代价是：

- 只适合单对象状态变化
- 更容易忽视 ABA、spurious wakeup 和状态协议设计
- 一旦你把复杂条件硬塞进去，协议会迅速变脆

### 6.2 六类高频失败模式

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

**失败模式 6：更新值和 notify 顺序混乱**

如果先 notify、后改值，往往会把协议解释力打散，甚至制造很隐蔽的等待问题。

### 6.3 现场症状与优先回查方向

| 现场症状 | 优先回查什么 | 常见真实原因 |
| --- | --- | --- |
| 线程明明被 notify 了还没推进 | 是否真的脱离 old | notify 不是状态，值可能没正确前进 |
| 协议偶发“看不到某次变化” | ABA / transient value | 值变化过快又回到 old |
| 等待逻辑越来越复杂 | 问题是不是已经不是单对象等待 | 该升级到 condition variable 了 |
| 系统仍在轮询某个 atomic | 是否该换 `atomic_wait` | 没用上更贴题的等待机制 |
| 醒来后逻辑仍出错 | 是否重查值 | 忽略了 spurious wakeup 或竞争变化 |
| 多等待者唤醒后竞争严重 | notify 策略 | `notify_all` 或状态编码过粗 |

### 6.4 为什么 `atomic_wait` 容易让人过度自信

它看起来很漂亮，因为：

- API 很窄
- 好像不用 mutex 了
- 好像比 condition variable 更新潮

但这会诱发一种误判：

- 以为只要有一个 atomic，就能把任何等待问题都压进去

实际并不是。`atomic_wait` 的价值恰恰来自它窄，所以一旦问题不再窄，它就不再贴题。

### 6.5 一个很实用的判别句式

如果你看到：

- **等待条件本质是“这个 atomic 不要再等于 old”**，优先想 `atomic_wait`
- **等待条件依赖多个字段**，优先退回 condition variable
- **值变化可能 ABA**，优先补 version / generation，而不是假设 wait 会看见所有中间态
- **notify 看起来像事件总线**，先反问真正被等待的值到底是什么

## 7. 应用场景

### 7.1 阶段位 / phase flag

一个线程等待 phase number 前进，特别适合。

例如：

- 初始化阶段完成
- 某个 epoch 前进
- 某个 generation 更新

### 7.2 单对象状态机

例如等待：

- `not_ready -> ready`
- `queued -> published`
- `stopping -> stopped`

只要条件本体真的是单对象状态切换，`atomic_wait` 通常很贴题。

### 7.3 runtime 停车 / 唤醒

轻量等待单个原子状态时，比 condition variable 更直接。

这也是它和 futex 风格直觉最接近的地方：围绕一个小状态字做等待/唤醒。

### 7.4 替代轮询等待

如果你现在的代码只是：

- 反复 load 一个 atomic
- 不断 `sleep_for`
- 或忙等直到它变化

那通常就该认真考虑 `atomic_wait`。

## 8. 工业 / 现实世界锚点

### 8.1 Linux futex 是最强的现实世界直觉锚点

Linux `futex(2)` 文档把 futex 描述为：

- waiting until a certain condition becomes true
- shared-memory synchronization 的 blocking construct
- user space fast path + kernel only when likely to block

这和 `atomic_wait` / `notify` 的现实工程直觉非常接近：

- 都围绕共享内存中的一个小状态字做等待/唤醒
- 都强调用户态快路径
- 都在“只等单对象状态变化”这类问题上特别贴题

如果你把这个锚点误读成“任何等待都能压成一个状态字”，就会高估 `atomic_wait` 的适用边界，并把复杂 predicate 等待错误地往单对象协议里压。

### 8.2 C++ 把 atomic waiting / notifying 正式纳入标准库

当前草案把 atomic waiting / notifying 正式纳入 `<atomic>`，说明这类等待已经不是平台私货，而是主流并发模型的一部分。

它带来的现实含义是：

- “等待单个状态位变化”已经被认为是足够常见、足够独立的一类问题
- 这类问题不必默认都回退到 condition variable

理解错这一点，常见后果是继续沿用 polling，或者用更重的 condition variable 包装一个本来只需要围绕单个 atomic 值建模的问题。

### 8.3 `atomics.wait` 明确把它定位成“比 polling 更高效”的路径

标准本身就把它描述成：

- 一种比 polling 更高效地等待 atomic object 值变化的机制

这给工程选型一个非常直接的锚点：

- 如果你现在还在为单 atomic 状态写轮询，说明模型大概率还没升级到当前更稳路径

这个锚点的价值不在于“某个平台内部可能怎么实现”，而在于它把默认选型基线直接往“阻塞式等待单对象值变化”推了一步。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 的事实、推断与建议

先把这部分拆开看。

**事实**

- C++ 标准草案已经把 atomic waiting / notifying 纳入 `<atomic>` 的正式模型。
- `atomics.wait` 明确把它定位成一种比 polling 更高效的等待 atomic object 值变化的机制。
- Linux `futex(2)` 仍然是理解这类“围绕共享状态字阻塞/唤醒”问题的强现实锚点。

**推断**

- 当等待目标确实可以压成“单个 atomic 值不再等于 old”时，语言级等待原语已经足够独立，不该再默认把这类问题视为 condition variable 的变体。
- 当协议正确性依赖“观察到所有中间态”时，单纯的 `atomic_wait` 模型并不够，因为 ABA 与 transient value 不可见的问题仍然存在。

**建议**

- 只要等待目标真的是“单个 atomic 值变化”，优先考虑 `atomic_wait` / `notify`。
- 先更新状态，再 notify。
- wait 醒来后继续按“重新检查值”的方式建模。
- 一旦条件变成复杂 predicate，就升级到 condition variable + mutex。
- 对可能 ABA 的协议，优先使用 generation / sequence 方案，而不是依赖“中间态一定可见”。

### 9.2 已经过时、明显不推荐或需要带语境理解的路径

| 路径 | 为什么旧/不稳 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| `sleep_for` / 轮询式等待 | 如果问题本身只是等一个 atomic 值变化，这通常已不是更稳路径 | 延迟不稳、CPU 浪费 | `atomic_wait` / `notify` |
| 用 condition variable 包一层单对象 atomic 等待 | 这不一定错，但常常比问题本身更重 | 代码更绕，边界更重 | 直接用 `atomic_wait` / `notify` |
| 把 notify 当事件本体 | notify 不是状态 | 容易丢掉对值历史的关注 | 始终围绕 atomic object 的值建模 |
| 忽略 ABA | 标准明确提醒 transient value 可能不可见 | 协议会偶发失效 | 用 version / generation / 更稳状态编码 |
| 醒来不重查值 | wait 可能 spurious wakeup，状态也可能再变 | 逻辑不稳 | 醒来总是重新检查 |

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
7. 如果一个协议表现得像“明明变过一次却没看见”，你首先应该怀疑什么？
8. 为什么“先改值，再 notify”比反过来更自然、更稳？
9. `atomic_wait` 和 futex 风格等待的共性是什么？
10. 请用一句话解释：它到底是在等“通知”，还是在等“对象值历史前进”？

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的核心句

理解了这篇文档后，你应该能把下面这句迁移出去：

**`atomic_wait` / `notify` 的本体不是事件广播，而是围绕单个 atomic object 的值历史建立等待协议。**

### 11.2 可以迁移到哪些领域

- futex 风格等待
- 单对象 phase/flag 协议
- `modification order` 与可唤醒条件
- `condition_variable` 与 `atomic_wait` 的选型判断
- runtime parking / unparking 设计

### 11.3 与相邻模型的关系

| 模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| condition variable | 复杂 predicate 等待 | 补单对象状态变化等待 |
| semaphore | permit / 令牌控制 | 补不是 permit，而是值历史变化的等待 |
| polling | 最直接但低效的等待 | 补如何把单对象等待升级成阻塞协议 |
| futex 直觉 | 小状态字等待/唤醒 | 补 C++ 语言级建模与可见性边界 |
| modification order | 定义原子对象历史 | 补等待协议如何依赖这条历史 |

### 11.4 最值得保留的迁移动作

以后遇到任何等待逻辑，可以先问自己：

- 我真的只是等一个 atomic 值变化吗？
- 这个变化是否已经通过对象历史和 happens-before 落地？
- 我是不是把更复杂的问题硬塞给了 `atomic_wait`？

只要这三问没答清，选型通常还不稳。

## 12. 未解问题与继续深挖

- ABA 与 wait / notify 的更系统协议设计，是否值得拆成专门案例文档？
- `atomic_wait` 与 futex、park/unpark 模型的更细对应关系，是否需要更底层实现篇？
- 多等待者场景下 notify 策略与惊群成本，是否值得补系统化分析？

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- [C++ draft `atomics.wait`](https://eel.is/c++draft/atomics.wait)
- [C++ draft `atomics.order`](https://eel.is/c++draft/atomics.order)
- [C++ draft `intro.races`](https://eel.is/c++draft/intro.races)
- [C++ draft `thread.condition`](https://eel.is/c++draft/thread.condition)
- [`futex(2)` Linux manual page](https://man7.org/linux/man-pages/man2/futex.2.html)
