---
doc_id: computer-systems-semaphore
title: Semaphore：当问题是“还有没有令牌”，而不是“谁拥有锁”
concept: semaphore
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:56:22+08:00'
updated_at: '2026-03-20T15:20:34+08:00'
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
  - docs/computer-systems/atomic-wait-notify.md
open_questions:
  - 在高负载限流、连接池和 GPU/IO 提交队列里，semaphore 相对 condition variable 的收益边界如何量化？
  - binary_semaphore 与 event/parking token 之间，哪些工程模式最值得单独整理？
  - permit 泄漏、取消语义与 shutdown/drain 流程之间，是否值得再补一篇专门文档？
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

**semaphore 管的是“有多少个许可可以被拿走”，而不是“谁拥有一段临界区”；它特别适合 admission control、槽位计数和 handoff，不适合直接保护复杂共享状态不变量。**

## 2. 一句话结论 / 问题定义

很多并发问题的核心不是“谁能独占进来”，而是：

- 现在最多允许多少个线程继续
- 还有多少个资源槽位可以被占用
- 某个阶段是否已经有“可继续的许可”被生产出来

这正是 semaphore 要解决的问题。

当前工作草案 `thread.sema.general` 直接把 semaphore 定位为：

- lightweight synchronization primitive
- used to constrain concurrent access to a shared resource

换句话说，它真正回答的是：

- 你还有没有 permit
- 如果没有，要不要阻塞
- 如果有人 release 了 permit，谁可以被放行

它不直接回答：

- 谁拥有临界区
- 一组共享字段现在是否满足复杂 predicate
- 某个对象不变量是否被完整保护

## 3. 对象边界与相邻概念

### 3.1 semaphore 管什么

它主要管：

- permit / resource count
- acquire 令牌时是否需要阻塞
- release 令牌时是否能唤醒等待者
- “最多同时允许多少个执行者继续”这种 admission control 问题

### 3.2 它不等于什么

它不等于：

- mutex ownership
- 复杂 predicate 等待
- condition variable
- 一个普通计数器本身
- 复杂共享状态保护

最容易犯的错，是把“还有没有令牌”误当成“共享状态已经安全一致”。

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

**semaphore vs queue length / ordinary counter**

- 普通计数器只能表达数值
- semaphore 还能把“数值不足时阻塞，数值增加时唤醒”一起包进去

### 3.4 一条最关键的边界

可以把它压成一句话：

**如果你真正要保护的是“一个对象的一组不变量”，默认先想 mutex；如果你真正要控制的是“最多允许多少个执行者穿过这道门”，再想 semaphore。**

## 4. 核心结构

### 4.1 semaphore 的最小模型

理解 semaphore，至少要抓住下面六个构件。

| 构件 | 它是什么 | 缺了会怎样 |
| --- | --- | --- |
| internal counter | 内部 permit 计数器 | 不知道还剩多少名额 |
| acquire | 试图拿走一个 permit | 无法表达“进入前先占位” |
| block | permit 不足时等待 | 只能忙等或手工轮询 |
| release | 归还或生产 permit | 系统会逐渐耗尽容量 |
| wake-up relation | release 后可能解除等待 | 令牌增加却没人能前进 |
| synchronization meaning | release / acquire 之间不仅是计数，还有可见性关系 | 容易把它误看成普通计数器 |

### 4.2 internal counter 是核心，但不是全部

当前草案 `thread.sema.cnt` 明确写道：

- `counting_semaphore` 维护一个内部计数器
- acquire 时递减
- release 时递增

但如果只记“它有个计数器”，你会漏掉两件很重要的事：

- permit 不足时它会阻塞，而不是只返回错误
- release / acquire 之间还承载同步语义，而不是纯数值变化

### 4.3 acquire / block / release 的三态流程

每次与 semaphore 交互，通常处在下面三种状态之一：

| 状态 | 发生了什么 | 你该怎样理解 |
| --- | --- | --- |
| permit 充足 | acquire 立即成功 | 你拿走了一个名额 |
| permit 不足 | acquire 阻塞 | 问题是“等名额”，不是“等复杂条件” |
| release 到来 | 计数器增加并可能唤醒 | 有新的名额被归还或生产出来 |

这说明 semaphore 的本体不是“保护区间”，而是“通行额度”。

### 4.4 `counting_semaphore` 与 `binary_semaphore`

当前草案还明确给出：

- `binary_semaphore` 只有两种状态
- 当适用时，它应比“单位计数的 counting_semaphore 默认实现”更高效

可以这样理解：

| 类型 | 适合什么 | 不适合什么 |
| --- | --- | --- |
| `counting_semaphore<N>` | 多槽位池、并发度限制、空槽/满槽计数 | 复杂 predicate 同步 |
| `binary_semaphore` | 简单 handoff、0/1 permit、单次放行 | 用它硬凑复杂状态机 |

### 4.5 synchronization relation 不能被忽略

当前草案对 `release()` 还给出了非常重要的语义：

- `release()` 的效果 strongly happens before 观察到这些效果的 `try_acquire()` 调用

这说明 semaphore 不只是计数器，它也参与同步关系。

最值得记住的翻译是：

**你 release 的不只是一个数字，也是“之前完成的那批动作可以被后来的 acquire 侧按约定观察到”的一部分基础。**

### 4.6 `try_acquire()` 的 spurious failure

当前草案允许 `try_acquire()` spuriously fail。

所以：

- “没拿到 permit”不总等于“计数器一定为 0”
- `try_acquire()` 失败不是绝对状态证明

这一点和 mutex 的 `try_lock()` 边界很像：失败的非阻塞尝试，不适合被神化为强观察点。

### 4.7 一页纸选择模板

如果你要快速判断某个问题是不是 semaphore，先过下面这张表：

| 问题 | 倾向选择 |
| --- | --- |
| 我要控制同时最多 N 个执行者 | semaphore |
| 我要表达“有几个槽位空着 / 几个任务就绪了” | semaphore |
| 我要保护复杂共享状态不变量 | mutex |
| 我要等待复杂 predicate 成立 | mutex + condition variable |
| 我只是等一个原子值变化 | `atomic_wait` |

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
- 它不记录“谁是 owner”，只记录“名额还剩多少”

### 5.2 资源池链：为什么它天然贴合池化问题

连接池、线程槽位池、GPU 提交槽位控制，本质上都可以压成下面这条链：

1. 系统初始化时放入 `N` 个 permit
2. 每个消费者在真正占用资源前先 `acquire`
3. 占用结束后 `release`
4. 任意时刻正在运行的人数不超过 `N`

这条链的好处是：

- 约束模型直接
- 不必额外手写一个“资源剩余数 + wait + notify”协议

### 5.3 生产者-消费者中的双计数链

bounded queue 是 semaphore 最经典的真实模型之一。

你通常会看到两种 permit：

- `empty_slots`
- `filled_slots`

然后再配合一个真正保护队列结构的 mutex。

这条链最值得记住：

1. producer 先 `empty_slots.acquire()`，确保确实有空位
2. producer 在 mutex 保护下写入队列
3. producer `filled_slots.release()`，宣布“又多了一个可消费元素”
4. consumer 先 `filled_slots.acquire()`，确保确实有元素
5. consumer 在 mutex 保护下取出元素
6. consumer `empty_slots.release()`，归还空槽位

这条链说明了 semaphore 的对象边界：

- 它非常适合表达“空位 / 满位的数量”
- 但真正的队列不变量，仍要靠 mutex 保护

### 5.4 handoff 链：binary semaphore 作为“你现在可以继续”的令牌

`binary_semaphore` 很适合表达简单 handoff：

- 一侧完成初始化
- 另一侧等一个“现在可以继续了”的令牌

这种场景下，问题不是复杂条件，而是单次放行或 0/1 状态切换。用 `binary_semaphore` 往往比把 condition variable 硬塞进来更直接。

### 5.5 为什么 semaphore 不适合直接保护复杂共享状态

如果你有一组复杂不变量，例如：

- 队列元素
- shutdown 标志
- 容量上限
- 当前模式位

semaphore 只能帮你表达“现在还有几个名额”。它不会自动替你保护那组状态本身。

这会带来一个关键因果：

- semaphore 可以告诉你“允许继续的人数”
- 但不能保证“多个字段此刻已经一致”

所以一旦你的问题从“名额”升级成“状态关系”，semaphore 就开始失焦。

### 5.6 shutdown / drain 链：permit 模型最容易漏的地方

semaphore 最容易在 shutdown 场景里出问题，因为你往往同时有两类事情：

- 不再接受新工作
- 清空或回收已有 permit

典型错误链是：

1. 系统准备关闭
2. 线程还在尝试 acquire
3. 某些持有 permit 的路径提前返回、取消或异常
4. `release` 没有对称发生
5. 于是你看到的不是“立刻炸掉”，而是系统吞吐越来越低、最后像死锁一样卡住

这也是为什么 semaphore 的 bug 往往比 mutex 更阴：它经常先表现成容量慢慢蒸发。

### 5.7 真正的因果核心

把这一节压成一句最值得记住的话：

**semaphore 的核心不是“锁住别人”，而是把“可继续的名额”显式计数，并把 acquire / release 变成系统吞吐与资源占用之间的门闩。**

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

semaphore 的好处是：

- 模型直接
- 天然适合限流、池化、并发度控制
- 比“计数器 + 自己管阻塞唤醒”更自然

代价是：

- 它不追踪 owner
- 它不天然保护复杂共享不变量
- permit 泄漏时问题会很隐蔽
- 一旦被拿来表达复杂 predicate，模型会快速失真

### 6.2 五类高频失败模式

**失败模式 1：把 semaphore 当 mutex 替代**

如果你的真实问题是“保护一段复杂状态”，semaphore 往往不贴题。

**失败模式 2：忘记 release 对称性**

permit 没有归还，系统容量会被悄悄吃掉。

**失败模式 3：把 `try_acquire()` 失败当成绝对状态证明**

当前草案允许 spurious failure。

**失败模式 4：用 semaphore 表达复杂 predicate**

这会让模型越来越扭曲，通常该回到 condition variable 或 mutex。

**失败模式 5：只建 permit，不建状态保护**

比如 bounded queue 里只放 semaphore，却忘了真正的队列结构仍需 mutex。

### 6.3 现场症状与优先回查方向

| 现场症状 | 优先回查什么 | 常见真实原因 |
| --- | --- | --- |
| 吞吐越来越低，像慢性死锁 | permit 对称性 | 某些路径 acquire 后没 release |
| 队列逻辑还是乱了 | 状态保护边界 | 只用了 semaphore，没有保护结构不变量 |
| `try_acquire()` 偶发失败导致错误分支 | 是否把失败当成绝对事实 | 忽略了 spurious failure 边界 |
| 系统能限制并发，但 shutdown 很难收尾 | drain / cancel 设计 | permit 生命周期没和关闭协议对齐 |
| 单次 handoff 写得很绕 | 是否该用 `binary_semaphore` | 把简单 0/1 场景写成了复杂 condition protocol |

### 6.4 为什么 permit 泄漏特别隐蔽

mutex 泄漏往往更快表现成“谁都进不去”。semaphore 泄漏更阴，因为：

- 它可能只是把容量从 8 偷偷吃到 7、6、5
- 系统还能跑，只是越来越差
- 很多团队会先怀疑性能、调度、IO，而不是 permit 对称性

所以 permit 模型最关键的纪律之一就是：

**每一个 acquire 都要能画出对应 release 的闭环。**

### 6.5 一个很实用的判别句式

如果你看到：

- **问题在问“还剩几个名额”**，优先想 semaphore
- **问题在问“谁能安全改这组状态”**，优先想 mutex
- **问题在问“某个复杂条件什么时候成立”**，优先想 condition variable
- **系统并发度越来越小却没明显死锁**，优先查 permit 泄漏

## 7. 应用场景

### 7.1 连接池 / 资源池

控制同时可用资源数量，是 semaphore 最自然的场景。

典型例子：

- 数据库连接池
- 固定数量的 GPU command slot
- 有限 worker slot

### 7.2 并发度限制

限制同时运行的 worker、任务、请求数量，是 permit 模型的标准形态。

这种场景里，你真正关心的是：

- 最多允许多少个任务在飞
- 达到上限时，新来的任务是阻塞、排队还是失败

### 7.3 生产者-消费者中的槽位控制

当问题是“还有几个空槽位 / 可消费槽位”时，semaphore 很贴题。

尤其在 bounded queue 里，它几乎就是“空位数”和“满位数”的自然编码。

### 7.4 handoff / start gate

`binary_semaphore` 很适合做：

- 一个线程准备好之后放行另一个线程
- 单次启动闸门
- 简单的“现在可以继续”信号

### 7.5 backpressure 与 admission control

很多系统设计问题表面上是“线程同步”，实质上是 backpressure：

- 不能无限制放任务进系统
- 到上限就必须阻塞、排队或拒绝

semaphore 在这里比 mutex 更贴题，因为它表达的是系统容量，不是临界区所有权。

## 8. 工业 / 现实世界锚点

### 8.1 POSIX semaphores 说明它不是教学玩具

Linux `sem_overview(7)` 明确说明：

- POSIX semaphores 可用于 process 和 thread 同步
- 有 named / unnamed 两种形式
- System V semaphores 是更老的接口，POSIX semaphores 更简单、设计更好

这说明 semaphore 不是教学玩具，而是操作系统层长期存在的主力同步模型之一。

### 8.2 C++20 `<semaphore>` 把 permit 模型正式拉进标准库

当前草案把 `counting_semaphore` / `binary_semaphore` 正式纳入标准库，说明 permit 模型已经是主流 C++ 并发工具箱的一部分。

这背后的现实含义是：

- admission control 不是边缘需求
- 标准库承认“名额模型”和“ownership 模型”是不同问题

### 8.3 POSIX 对 named / unnamed、process / thread 的区分，提醒你它的作用域不止线程内

`sem_overview(7)` 还给出一个很现实的工业边界：

- semaphore 可以是线程内同步工具
- 也可以扩展到进程间同步

这和 `std::counting_semaphore` 的典型进程内用法不同，但它提醒你 semaphore 的核心模型比“某个 C++ 类模板”更广。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 当问题本质是 permit / admission control 时，优先考虑 `std::counting_semaphore`
- 当只是 0/1 许可或简单 handoff，考虑 `binary_semaphore`
- 当问题变成复杂共享状态或 predicate，退回 mutex + condition variable
- 把 acquire / release 对称性当成一等设计约束，而不是事后补丁
- 对 shutdown、取消、超时路径单独检查 permit 是否会泄漏

### 9.2 已经过时、明显不推荐或必须带语境理解的路径

| 路径 | 为什么旧/不稳 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| System V semaphores 作为默认首选 | `sem_overview(7)` 已明确 POSIX semaphores 更简单、设计更好 | 接口更老、默认选择价值低 | 进程内优先 `std::counting_semaphore`，需要 OS 级语义时优先 POSIX semaphore |
| 用 condition variable / 轮询硬拼 permit 模型 | 问题本体明明就是“名额数” | 代码绕，状态机容易扭曲 | 直接用 semaphore 表达 permit |
| 用 semaphore 直接保护复杂共享状态 | 它不具备 owner 和不变量保护语义 | 容易留下状态竞态 | 退回 mutex 或 mutex + condition variable |
| 只在 happy path 上 release | 忽略异常、取消、早返回 | permit 泄漏隐蔽 | 为所有 acquire 建 release 闭环 |
| 把 `try_acquire()` 失败当“绝对没有名额” | 标准允许 spurious failure | 推理不稳 | 只把失败当“这次没拿到”，不要过度外推 |

### 9.3 什么时候别用 semaphore

如果你的问题是：

- 保护复杂共享对象不变量
- 需要 owner 语义
- 需要复杂 predicate 等待

那就该考虑：

- mutex
- condition variable
- `atomic_wait`

如果你的问题是“多个字段必须一起看”，semaphore 往往不是正确入口。

## 10. 自测题 / 验证入口

1. semaphore 和 mutex 的本质差别是什么？
2. 为什么 semaphore 更像 permit 模型，而不是 owner 模型？
3. 为什么 permit 泄漏会让 bug 很隐蔽？
4. 为什么 semaphore 不适合直接保护复杂共享状态？
5. `binary_semaphore` 和 `counting_semaphore` 各适合什么问题？
6. 为什么说 System V semaphores 在现代默认路径里不是首选？
7. 为什么 bounded queue 往往既需要 semaphore 又需要 mutex？
8. `try_acquire()` 失败时，哪些结论不能草率推出？
9. shutdown / cancel 路径为什么特别容易把 semaphore 用坏？
10. 请用一句话解释：什么时候问题本质是“名额”，什么时候本质是“不变量”？

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的核心句

理解了这篇文档后，你应该能把下面这句迁移出去：

**semaphore 最适合表达“系统还剩多少通行额度”，而不是“这组共享状态已经一致”。**

### 11.2 可以迁移到哪些领域

- admission control
- resource pool
- bounded queue
- `mutex` / `condition_variable` / `atomic_wait` 选型判断
- backpressure 与容量治理
- simple handoff / start gate

### 11.3 与相邻模型的关系

| 模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| mutex | 保护复杂共享状态不变量 | 补 permit / 名额控制视角 |
| condition variable | 围绕 predicate 等待和唤醒 | 补“条件本体其实只是数量”时的更直接表达 |
| `atomic_wait` | 等单个原子值变化 | 补“等待本体是令牌数”时的更自然编码 |
| backpressure 模型 | 管系统容量与节流 | 补线程级 acquire/release 语义 |
| queue / pool 设计 | 管资源生命周期 | 补槽位计数与 admission control 原语 |

### 11.4 最值得保留的迁移动作

以后遇到任何同步问题，可以先问自己：

- 我真正控制的是“名额数量”，还是“共享状态一致性”？
- 如果拿不到继续资格，我是该阻塞等待，还是该检查复杂 predicate？
- 每一个 acquire 能不能清楚地配回一个 release？

只要这三问没答清，选型通常还不稳。

## 12. 未解问题与继续深挖

- `binary_semaphore` 与 event / park token 模型的对照，哪些工程模式最值得单独拆开？
- semaphore 与 backpressure 设计之间的系统关系，是否值得扩成独立文档？
- permit 泄漏、公平性、取消语义和 shutdown/drain 流程，是否需要更系统的排查模板？

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `thread.sema`: https://eel.is/c++draft/thread.sema
- C++ draft `thread.sema.cnt`: https://eel.is/c++draft/thread.sema.cnt
- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- `sem_overview(7)` Linux manual page: https://man7.org/linux/man-pages/man7/sem_overview.7.html
