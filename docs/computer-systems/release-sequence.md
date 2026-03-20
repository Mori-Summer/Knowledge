---
doc_id: computer-systems-release-sequence
title: Release Sequence：为什么 release 之后的一串 RMW 还能继续“带着发布语义往前走”
concept: release_sequence
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:16:47+08:00'
updated_at: '2026-03-20T16:16:02+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_diff_cpp17_2026_03_16
  - gcc_atomic_builtins_docs_2026_03_16
  - llvm_atomics_guide_2026_03_16
  - wg21_p3475r2_2025
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_lock_free_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/memory-order.md
  - docs/computer-systems/modification-order.md
  - docs/computer-systems/synchronizes-with.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/fence.md
open_questions:
  - 如何把 release sequence 与 fence、`atomic_wait`/`notify`、mixed-order litmus tests 一起压缩成统一判断模板？
  - 在真实 lock-free 内存回收机制里，哪些 release sequence 最容易被误判成“只是普通原子更新”？
  - 如果未来标准继续收束 `consume` 相关历史包袱，release sequence 在教学和工具提示里应如何被更早暴露出来？
---

# Release Sequence：为什么 release 之后的一串 RMW 还能继续“带着发布语义往前走”

## 1. 这份文档要帮你学会什么

这份文档的目标，不是让你记住一句“release sequence 是 release 后面的一段东西”，而是让你真正拿到一套能用于代码审查和协议验证的判断流程。

读完后，你应该至少能做到：

- 说清 `release sequence` 试图补的到底是哪一类同步缺口
- 把它和 `release`、`acquire`、`modification order`、`synchronizes-with`、`happens-before` 分层区分
- 看懂“最终 acquire 没读到头部写下的值，为什么仍可能同步回头部”
- 在 CAS、`fetch_add`、状态字推进、引用计数这类代码里识别它到底有没有成立
- 在重构并发代码时识别“把一个 RMW 改成普通 store”是否会偷偷把同步链剪断

一句话先给结论：

**`release sequence` 是沿着“同一原子对象的 modification order”传播发布语义的一段连续链；只要 acquire 读到了这条链上某个 side effect 写出的值，它就可能同步回链头那次 release。**

## 2. 一句话结论 / 问题定义

它要解决的核心问题是：

- 发布者 A 已经把普通数据准备好了
- A 用一次 `store(release)` 或 release 型 RMW 把“状态可见”发布出去
- 之后这个状态字又被别的线程继续用 RMW 往前推进
- 最终观察者 C 读到的已经不是头部 release 写下的原值

这时 C 还能不能合法看到 A 在 release 之前写的那些普通数据？

如果没有 `release sequence`，很多现实里的 lock-free 状态机都只能退回更重的同步手段。  
有了它，语言模型就允许“发布语义”沿着同一原子对象的连续 RMW 链继续往后传。

## 3. 对象边界与相邻概念

### 3.1 它真正管的对象

`release sequence` 只管一件事：

- **同一个原子对象**上
- 从某个 release 头部开始
- 往后连续延伸的一段特殊修改链

它不管多个原子对象之间的全局顺序，也不管任意后来的普通写。

### 3.2 它不等于什么

它不等于：

- 整个 `modification order`
- 整个 `happens-before`
- 任意“后来的写入”
- 任意 acquire / release 关键字组合
- “我感觉这个状态已经经过很多线程了，所以应该算同步了”

### 3.3 和相邻概念的边界

**`release sequence` vs `modification order`**

- `modification order` 是某个 atomic object 的全部修改总序
- `release sequence` 是从某个 release 头部出发、在这条总序上切出来的一段连续特殊子序列

**`release sequence` vs `synchronizes-with`**

- `release sequence` 不是最终那条跨线程同步边
- 它只是 release / acquire 建边时的中间桥梁
- 真正的跨线程边叫 `synchronizes-with`

**`release sequence` vs `happens-before`**

- `happens-before` 是关系图结果
- `release sequence` 是关系图里某类边成立所依赖的中间结构

**`release sequence` vs fence**

- fence 也能参与 release/acquire 风格的同步
- 但 fence 不是 release sequence
- fence 往往还需要借某个 carrier atomic object 落地

### 3.4 最容易搞错的边界

最危险的误解有三个：

- 误以为“同线程后续普通 store 也还能自动留在 release sequence 里”
- 误以为“只要最后读到的是同一个 atomic，就一定能同步回头部”
- 误以为“所有后继 RMW 自己都必须写成 `release` 或 `acq_rel` 才能继续这条链”

这三条都不对，至少都需要更精确的条件。

## 4. 核心结构

理解 `release sequence`，最稳的方式不是背标准句子，而是把它拆成六个固定构件。

### 4.1 头部：一次 release 操作

这条链必须有头，头必须是 release 操作：

- `store(memory_order_release)`
- 成功的 RMW，且该 RMW 具备 release 语义，例如 `memory_order_acq_rel`

没有 release 头部，就没有后面的 release sequence。

### 4.2 载体：同一个 atomic object

这条链只沿着**同一个原子对象**前进。  
换对象，链就断了。

这意味着如果你的协议语义分散在两个 atomic 上，那么 release sequence 本身并不会替你跨对象传递同步。

### 4.3 地板：该对象的 modification order

release sequence 不是按墙钟时间排，也不是按“哪个线程先执行完”排。  
它是沿着该对象的 `modification order` 切出来的一段连续区间。

所以判断它成不成立，第一步不是看源码位置，而是：

- 先找 carrier atomic object
- 再沿这个对象的 modification order 看后继修改

### 4.4 延续条件：连续的成功 RMW 链

按当前工作草案语义，头部之后继续留在 release sequence 里的，关键是：

- 必须仍是对同一 atomic object 的修改
- 必须是连续的原子 RMW
- 失败的 CAS 不算，因为它没有写 side effect

这点极关键：  
**release sequence 看的不是“后面是不是还在写这个对象”，而是“后面是不是一段连续成功 RMW”。**

### 4.5 着陆条件：最终 acquire 读到了链上的值

链本身还不够。  
最终还要有一个 acquire 操作去“接住”它：

- 这个 acquire 必须读到 release sequence 中某个 side effect 写出的值
- 如果没读到链上的值，这条链对它就没有同步意义

### 4.6 推导结果：同步边回到头部，而不是停在尾部

一旦 acquire 读到了这条链上的某个值：

- 它建立的不是“只和最后那个尾部 RMW 有点关系”
- 而是允许同步回**头部 release**

然后再通过线程内顺序与传递性，把头部之前发布出去的普通写一起带到 acquire 之后。

### 4.7 一张最短判断图

判断某段代码是不是在依赖 release sequence，可以固定问这七个问题：

1. 头部 release 是谁？
2. 载体 atomic object 是哪个？
3. 头部之后有哪些后继修改？
4. 这些后继里哪些是真正成功的 RMW？
5. 其中是否出现普通 store、失败 CAS、换对象等断链点？
6. 最终 acquire 到底读到了哪个值？
7. 这个值是不是来自这条连续链上的某个 side effect？

七问走完，结论通常就很难再漂。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 主链路：release 头部 -> RMW 接力 -> acquire 着陆

```cpp
std::atomic<int> state{0};
int payload = 0;

// Thread A
payload = 42;
state.store(1, std::memory_order_release);   // 头部

// Thread B
state.fetch_add(1, std::memory_order_relaxed); // 1 -> 2，成功 RMW

// Thread C
if (state.load(std::memory_order_acquire) == 2) {
    use(payload);
}
```

这段代码里真正值钱的因果链是：

1. A 先写普通数据 `payload = 42`
2. A 再把 `state` 写成 `1`，并带 release 语义
3. B 在同一个 atomic object `state` 上做一次成功 RMW，把值推进到 `2`
4. 这次成功 RMW 处在以 A 为头部的 release sequence 里
5. C 的 acquire load 读到了值 `2`
6. 因为 `2` 来自 release sequence 上的 side effect，C 可以同步回 A 的头部 release
7. A 在头部 release 之前写入的 `payload` 对 C 可见

这就是 release sequence 最典型的价值：  
**观察者不必直接读到“最初发布值”，仍可能合法继承那次发布。**

### 5.2 变体链路：中间人做的是 CAS 状态迁移

```cpp
enum : int { kInit = 0, kReady = 1, kClaimed = 2 };

std::atomic<int> state{kInit};
Job* job = nullptr;

// Thread A
job = prepare_job();
state.store(kReady, std::memory_order_release);   // 头部

// Thread B
int expected = kReady;
state.compare_exchange_strong(
    expected,
    kClaimed,
    std::memory_order_acq_rel,
    std::memory_order_relaxed);                   // 成功 CAS，属于 RMW

// Thread C
if (state.load(std::memory_order_acquire) == kClaimed) {
    run(job);
}
```

这里最容易被忽视的是：

- C 并没有看到 `kReady`
- 它看到的是 B CAS 成功后写出的 `kClaimed`

但只要这次 CAS 成功，并且它正好是头部后续的连续 RMW，C 仍可能同步回 A 的头部 release。  
这就是很多“阶段推进型状态字”能成立的关键。

### 5.3 断链案例 1：后继普通 store 不自动续上去

```cpp
// Thread A
payload = 42;
state.store(1, std::memory_order_release);   // 头部

// Thread B
state.store(2, std::memory_order_relaxed);   // 普通 store，不是 RMW

// Thread C
if (state.load(std::memory_order_acquire) == 2) {
    use(payload); // 不能仅凭这段推出安全
}
```

为什么这段不能靠 release sequence 自证？

- 因为 B 的这次写只是普通 atomic store
- 它不是成功 RMW
- 所以它不属于从 A 头部延续出来的那段 release sequence

换句话说，C 虽然读到了同一个 atomic 的后继值，但这个值不是从 release sequence 链上掉下来的。

### 5.4 断链案例 2：失败的 CAS 不写 side effect

```cpp
int expected = 1;
state.compare_exchange_strong(
    expected,
    2,
    std::memory_order_acq_rel,
    std::memory_order_relaxed); // 如果失败，没有写 side effect
```

失败 CAS 的危险在于它“看起来参与了竞争”，但并没有形成修改 side effect。  
对 release sequence 来说：

- 成功 CAS 是链上的一个写点
- 失败 CAS 只是一次读路径，不是链上的写点

所以代码审查时不能把“执行过 CAS”与“向前延续了 release sequence”混为一谈。

### 5.5 断链案例 3：换 carrier object，链立刻失效

如果协议是这样：

- A 在 `flag1` 上 release
- B 在 `flag2` 上做一串 RMW
- C 在 `flag2` 上 acquire

那 release sequence 根本不适用。  
因为它从头到尾都只沿着一个 atomic object 的 modification order 前进。

### 5.6 为什么尾部 RMW 不必自己也是 release

这是最反直觉、但最值得单独记住的一条：

- release sequence 的延续条件，核心看“是不是连续成功 RMW”
- 不是看“尾部 RMW 自己有没有 release 语义”

但这不等于工程上可以把尾部一律写成 `relaxed`。  
原因是尾部 RMW 可能还承担别的职责：

- 它自己可能既要消费上游状态，又要发布下游状态
- 它可能还要约束本线程前后的普通读写
- 它可能还要给其他观察者提供更直接的同步语义

所以“能延续 release sequence”只是最低条件，不是完整选型依据。

### 5.7 一套可执行的审查流程

以后看到类似代码，不要直接问“这里是不是有 release sequence”。  
按下面顺序走更稳：

1. 找出真正承载状态推进的 carrier atomic object。
2. 标出头部 release。
3. 沿该对象的 modification order 排出后继修改。
4. 划掉所有普通 store、失败 CAS、不同对象上的操作。
5. 检查剩下的是不是一段连续成功 RMW。
6. 再看最终 acquire 的确切 read-from 来源。
7. 如果 acquire 没读到这条链上的 side effect，就不要再借 release sequence 给自己加同步边。

## 6. 关键 tradeoff 与失败模式

### 6.1 这套机制真正买到的东西

release sequence 让你买到的是：

- 在一个共享状态字被多个线程持续 RMW 推进时
- 不必要求最终观察者直接读到“最初发布值”
- 仍能把最初发布出去的普通数据合法带过来

这对 lock-free 状态机是非常值钱的表达能力。

### 6.2 代价：推理难度显著上升

代价也同样明确：

- 正确性严重依赖“最终读到谁写的值”
- 一个看似无害的重构就可能断链
- 协议正确性不再从单行代码可读性里直接显现

所以它适合底层协议，不适合作为普通业务代码的默认同步手段。

### 6.3 常见失败模式

**失败模式 1：把整个 modification order 都当成 release sequence**

不是所有后继修改都在链里。  
当前主路径下，关键是连续成功 RMW。

**失败模式 2：把“同线程后续普通 store”沿用成老时代直觉**

这是最常见的历史包袱误判。  
今天更稳的理解是：普通 store 不该被默认视为仍在这条链里。

**失败模式 3：没搞清 acquire 实际读到了谁**

release sequence 的成立不是“有个 acquire 就够了”。  
必须追到 read-from 来源。

**失败模式 4：把失败 CAS 也记成状态推进**

失败 CAS 很可能让你在脑中画出一条并不存在的链。

**失败模式 5：把 release sequence 当万能同步保险**

它只解决“单 carrier atomic 上的发布语义延续”。  
跨对象顺序、睡眠唤醒语义、复杂生命周期约束，仍要单独设计。

**失败模式 6：为了省一点开销，把 RMW 悄悄改成普通 store**

这类“优化”极危险。  
它看起来只改了一条机器指令，却可能把整个同步证明基础拆掉。

## 7. 应用场景

### 7.1 多阶段状态字

典型模式是：

- `init -> ready -> claimed -> done`
- 每个阶段都写在同一个原子状态字里
- 不同线程通过成功 CAS 或 `fetch_or` / `fetch_add` 往前推进

如果最终观察者读到的是后继阶段值，但仍需要看到最初发布的 payload，release sequence 往往就是关键一环。

### 7.2 引用计数与控制块生命周期

在引用计数或控制块设计里，多个线程可能都在对同一个计数做 RMW。  
虽然每个实现的内存序策略不完全相同，但代码审查常常要追问：

- 最终看到“最后一个引用”这一状态的线程
- 是否还能合法看见对象发布时写下的元数据

这种分析经常要落到同一原子计数的连续 RMW 链上。

### 7.3 lock-free 队列、栈和任务调度状态机

在真正的 lock-free 结构里，状态通常不是“写一次就结束”，而是：

- 被多个线程竞争性推进
- 最终消费者看到的是中间人更新过的状态

此时 release sequence 就不再是边缘知识，而是协议成败点。

### 7.4 轻量级 hand-off 协议

如果系统想避免 mutex，希望用一个状态字表达：

- 数据已准备
- 某线程已接手
- 工作已完成或回收

那这个状态字经常会成为 release sequence 的 carrier。

## 8. 工业 / 现实世界锚点

### 8.1 LLVM IR 与 GCC `__atomic` builtins

LLVM IR 里有专门的 `atomicrmw`、`cmpxchg`；GCC 也提供 `__atomic_fetch_add`、`__atomic_compare_exchange_n` 这类一等内建。  
这不是语法装饰，而是现实世界里确实存在大量“同一原子对象上的 RMW 接力”协议。

release sequence 之所以重要，正是因为这些 RMW 不是孤立存在，而是经常承担发布链中继角色。

### 8.2 `std::shared_ptr` 控制块与工业级引用计数实现

libstdc++、libc++、MSVC STL 的共享所有权控制块，以及 Chromium / Folly / Abseil 体系里常见的 intrusive refcount 设计，都会围绕同一原子计数反复做 RMW。  
它们的细节实现不完全相同，但审查这些代码时经常会遇到同一类问题：

- 哪个线程真正接住了“最后一个状态”
- 这个线程是否还能安全看到对象先前发布的数据

这正是 release sequence 擅长解决的地带。

### 8.3 Boost.Lockfree、Folly 等无锁状态字设计

工业级 lock-free 容器和调度组件里，状态不是“单写单读”，而是被多线程 CAS 接力推进。  
这些项目里不一定处处写出“release sequence”这个词，但它们依赖的正是这套语义。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的实践不是“尽量多用 release sequence”，而是：

- 只有在协议确实存在“头部发布 + 中间 RMW 接力 + 尾部 acquire 着陆”时才主动依赖它
- 一旦依赖，就在设计说明或代码注释里明确写出 `head / carrier / relay / landing read`
- 对每次把 RMW 改成 store 的优化，都重新证明同步链
- 如果某个中继 RMW 还承担本地顺序职责，就按职责选 `acq_rel`、`acquire`、`release` 或 `relaxed`，不要只盯“能不能留在链里”

### 9.2 当前最稳的选型分界

更适合主动依赖 release sequence 的情况：

- 单 carrier atomic 已经是协议中心
- 中间确实存在多个成功 RMW 接力
- 最终观察者大概率读到的是中继值而不是头部原值

更适合退回直接同步原语的情况：

- 只是一次性发布，没有多跳状态推进
- 协议跨多个 atomic object
- 维护者未必熟悉 read-from / release sequence 推理
- 正确性比极限性能更重要

### 9.3 已经过时或必须带历史语境理解的路径

**把“同线程后续普通 store 还能继续 release sequence”当今天的默认规则**

这条理解必须带 C++20 之前的历史语境。  
在今天的主路径里，更稳的做法是把 release sequence 收窄为：

- 头部 release
- 后续连续成功 RMW

不要再把普通 store 默认纳入。

**拿 `memory_order_consume` 或依赖顺序魔法去替代清晰的 acquire/release 推理**

这条路径在当前 WG21 方向里已经明显退居历史边缘。  
更推荐的是显式 acquire/release 或更高层同步原语。

**用“全写成 `seq_cst` 就不用想了”替代正确建模**

更强的 order 不会替你修复“读到的不是这条链上的值”这种根本问题。  
它可能更重，但不自动更对。

### 9.4 替代方案

如果你不想让维护者背这套细粒度模型，当前更稳的替代通常是：

- 直接的 `release` / `acquire` flag
- `mutex`
- `condition_variable`
- `atomic_wait` / `notify`
- 更高层并发抽象

这些路径的同步边更显式，审查成本更低。

## 10. 自测题 / 验证入口

1. 某线程对 `state.store(1, release)` 之后，另一个线程执行 `state.fetch_add(1, relaxed)`，第三个线程 `load(acquire)` 读到 `2`。为什么这里可能仍能看到头部 release 之前的普通写？
2. 如果中间那步不是 `fetch_add`，而是 `state.store(2, relaxed)`，结论为什么变了？
3. 为什么失败的 `compare_exchange` 不能被算进 release sequence？
4. 一个尾部 RMW 明明可以写成 `relaxed` 也留在链里，为什么工程上仍可能应该写成 `acq_rel`？
5. 协议里如果中间换了另一个 atomic object，为什么 release sequence 立刻失效？
6. 如果 reviewer 只看到了“这里有 release、那里有 acquire”，却没追 read-from 来源，最可能漏掉什么 bug？
7. 你能不能把某段状态机代码里的 `head / carrier / relay / landing read` 四个角色各自标出来？
8. 在什么情况下，宁可放弃 release sequence，也要退回 mutex 或更直接的 release/acquire？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- [memory-order.md](/Users/maxwell/Knowledge/docs/computer-systems/memory-order.md) 里的 release/acquire 传播判断
- [modification-order.md](/Users/maxwell/Knowledge/docs/computer-systems/modification-order.md) 里的单对象写历史分析
- [synchronizes-with.md](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md) 里的跨线程建边判断
- [happens-before.md](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md) 里的传递性分析
- CAS 循环、状态字推进、引用计数、无锁 hand-off 协议

迁移时最关键的不是重复背术语，而是重复执行同一套检查：

- 哪个 atomic 在当 carrier？
- 哪个写是头部 release？
- 中间是不是连续成功 RMW？
- 结尾 acquire 到底读到了谁？

## 12. 未解问题与继续深挖

后续值得继续深挖的点包括：

- release sequence 与 `atomic_thread_fence` 三种建边模式混用时的最短判断模板
- `atomic_wait` / `notify` 背后的“值变化 + 唤醒”模型如何和 release sequence 拼在一起
- 引用计数、hazard pointer、epoch reclamation 中，哪些协议真的依赖 release sequence，哪些只是看起来像
- sanitizer 和代码审查工具是否值得直接把“release sequence 断链点”做成专门提示

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `diff.cpp17`: https://eel.is/c++draft/diff.cpp17
- WG21 P3475R2, *Defang and deprecate memory_order::consume*: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3475r2.pdf
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
- LLVM Atomic Instructions and Concurrency Guide: https://llvm.org/docs/Atomics.html
