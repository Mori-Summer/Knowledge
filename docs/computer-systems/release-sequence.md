---
doc_id: computer-systems-release-sequence
title: Release Sequence：为什么 release 之后的一串 RMW 还能继续“带着发布语义往前走”
concept: release_sequence
topic: computer-systems
created_at: '2026-03-16T14:16:47+08:00'
updated_at: '2026-03-19T21:20:00+08:00'
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
  - docs/computer-systems/happens-before.md
open_questions:
  - 如何把 release sequence 与 fence、`atomic_wait`/`notify`、mixed-order litmus tests 一起压缩成统一判断模板？
  - 在真实 lock-free 内存回收机制里，哪些 release sequence 最容易被误判成“只是普通原子更新”？
---

# Release Sequence：为什么 release 之后的一串 RMW 还能继续“带着发布语义往前走”

## 1. 这份文档要帮你学会什么

这份文档要解决的，不是“记住一个标准术语”，而是把一个经常在 lock-free 代码里决定成败的细节压成可调用模型。

读完后，你应该至少能做到：

- 说清 `release sequence` 到底在解决什么问题
- 说清它和 `release`、`acquire`、`modification order`、`happens-before` 的边界
- 知道为什么 acquire 读到的不是最初那个 release 写入，也可能仍然同步到那个 release 头部
- 知道当前草案里什么会延续 release sequence，什么不会
- 在 CAS / `fetch_add` / 状态字推进这类代码里知道该重点盯哪几个点

一句话先给结论：

**`release sequence` 是一条沿着“同一原子对象的修改顺序”继续传播发布语义的链；它让一个 acquire 读取到后续 RMW 的结果时，仍然可以同步回最初的 release 头部。**

## 2. 一句话结论 / 问题定义

假设有这样一条并发链路：

1. 线程 A 先写普通数据
2. A 对某个原子状态做一次 `store(release)`
3. 线程 B 对同一个原子状态继续做一次或多次 RMW，比如 `fetch_add` 或成功的 `compare_exchange`
4. 线程 C 最终用 `load(acquire)` 读到了 B 写出的值

问题来了：

- C 读到的明明不是 A 当初 release store 写下的那个值
- 那 C 还能不能“接到”A 之前发布出去的普通数据？

`release sequence` 就是为了解决这个问题。

没有它，很多跨线程状态推进、引用计数、lock-free 状态机都会变得更难表达，甚至需要更重的同步。

## 3. 对象边界与相邻概念

### 3.1 `release sequence` 管什么

它主要管一件事：

- 同一个原子对象上，一次 release 写入后面的某些后继原子修改，是否仍然算这次发布链的一部分

### 3.2 它不等于什么

它不等于：

- 整个 `modification order`
- 任意“后来发生”的写入
- 任意普通 store
- `happens-before` 本身
- release fence 本身

### 3.3 和几个相邻概念的边界

**`release sequence` vs `modification order`**

- `modification order` 是该原子对象全部修改的总序
- `release sequence` 只是这条总序里，从某个 release 头部出发的一段特殊连续子序列

**`release sequence` vs `happens-before`**

- `release sequence` 不是最终关系图
- 它只是用来帮助 release 与 acquire 建立 `synchronizes-with` 的中间机制

一旦 acquire 读取到 release sequence 中某个 side effect 的值，才可能进一步导出 `happens-before`。

**`release sequence` vs “后来同线程又写了一次”**

这是最容易踩坑的点之一。  
在当前工作草案里，release sequence 不是“同线程以后随便再写同一个原子都还能续上去”的概念。

## 4. 核心结构

理解 `release sequence`，至少要抓住下面六个构件。

### 4.1 头部：一个 release 操作 A

release sequence 必须有头，而且头必须是 release 操作。

### 4.2 对象：同一个原子对象

release sequence 只沿着**同一个原子对象**的修改顺序前进。  
换对象，这条链就断了。

### 4.3 基底：该对象的 modification order

release sequence 不是按线程局部时间排，也不是按真实时间排，而是按该原子对象的 modification order 去切一段连续子序列。

### 4.4 当前草案中的延续条件

截至 `2026-03-16`，当前 C++ 工作草案把 release sequence 定义为：

- 以 release 操作 A 为头部
- 在该对象的 modification order 中，从 A 开始向后取**最长的连续子序列**
- 其中 A 之后的每个操作都必须是**原子 read-modify-write**

这意味着一个关键事实：

- 后继 RMW 不一定自己也要是 release
- 但它必须真的是 RMW，且仍然处在这条连续链里

### 4.5 读取条件

当前草案规定：如果一个 acquire 操作 B 读到了“某个由 release sequence 中 side effect 写出的值”，那么头部 release A 与 B 之间可以建立 `synchronizes-with`。

### 4.6 结果：把头部之前的写入一起带过去

一旦同步建立，A 在 release 之前的普通写入，就能通过 `happens-before` 对 B 之后的读取可见。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最值钱的主链：release 头部 + 后续 RMW + 最终 acquire

```cpp
// Thread A
data = 42;
state.store(1, std::memory_order_release);   // 头部 A

// Thread B
state.fetch_add(1, std::memory_order_relaxed); // 把 1 改成 2

// Thread C
if (state.load(std::memory_order_acquire) == 2) {
    use(data);
}
```

这段代码里最容易被忽略的点是：

- 线程 C 读到的是 B 写出的 `2`
- 它并没有直接读到 A 写出的 `1`

但如果 B 这次 `fetch_add` 属于以 A 为头部的 release sequence，那么：

1. A 是头部 release
2. B 对同一个原子对象做了后继 RMW
3. C 的 acquire load 读到了该 RMW 产生的值
4. 当前草案允许 C 的 acquire 通过这条 release sequence 同步回 A
5. 因而 A 在 release 前写入的 `data`，可以对 C 可见

这正是 release sequence 的价值：  
**它让“发布语义”沿着一个原子状态字的 RMW 链继续传下去。**

### 5.2 一个重要细节：尾部 RMW 自己不必是 release

这点很反直觉，但很关键。

很多人会误以为：

- 头是 release
- 后面每一步也都必须继续写 `release` / `acq_rel`

当前草案并不是这么定义的。  
release sequence 的延续条件看的是：**它是不是连续的原子 RMW 链**。

当然，工程上并不意味着你应该随便把尾部都降成 `relaxed`。  
它只说明“release sequence 能否延续”和“尾部本身还承担哪些额外同步职责”是两件事。

### 5.3 什么会把这条链切断

看下面这段：

```cpp
// Thread A
state.store(1, std::memory_order_release);
state.store(2, std::memory_order_relaxed);
```

如果另一个线程 acquire-load 读到了 `2`，很多人会本能地以为它还能同步回第一次 release store。  
但按当前草案，这样推理是不成立的。

原因是：

- 第二次操作只是普通原子 store
- 它不是 RMW
- 所以它不属于由第一次 release 头部延续出来的 release sequence

### 5.4 为什么这东西会决定 lock-free 代码的生死

很多 lock-free 结构不是“一次 release，另一个线程直接 acquire 读到原值”这么简单。  
更常见的是：

- 一个状态字被多个线程继续用 RMW 推进
- 最终观察者读到的是链条中后面的值

如果你不理解 release sequence，就很容易在这里误判：

- 以为同步还在，其实已经断了
- 以为读到后续值就一定能回溯到最初发布，其实不一定

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

release sequence 的价值是：它允许更细粒度地表达 lock-free 状态推进。  
代价是：推理门槛明显更高。

- 好处：减少不必要的重同步，允许状态在多个线程的 RMW 间继续传递发布语义
- 代价：一旦看错“谁在链里、谁不在链里”，bug 会非常隐蔽

### 6.2 常见失败模式

**失败模式 1：把整个 modification order 都当成 release sequence**

不是每个后继修改都在链里。  
当前草案下，关键条件是“连续的 RMW”。

**失败模式 2：以为同线程后续普通 store 还会自动续上去**

这正是很多老材料最容易误导人的地方。  
当前工作草案已经不是这种规则了。

**失败模式 3：忘记 acquire 到底读到了哪个值**

release sequence 不是“只要后来读同一个原子就行”。  
关键是 acquire 操作读到的值，是否来自这条 release sequence 中的某个 side effect。

**失败模式 4：把失败的 CAS 也算进链条**

失败的 `compare_exchange` 没有写 side effect，就不能像成功 RMW 那样去延续 release sequence。

**失败模式 5：把“能延续 release sequence”误当“所有同步责任都自动满足”**

一个后继 RMW 即使留在链里，也不代表它在程序其他位置承担的同步职责就一定够。  
release sequence 只是整个同步设计的一部分，不是万能保险。

## 7. 应用场景

### 7.1 状态字推进

多个线程在同一个原子状态字上做 `fetch_add`、成功 CAS、状态迁移时，release sequence 经常决定最终观察者能否接到最初发布的数据。

### 7.2 引用计数与生命周期管理

一些高性能运行时会把“对象已发布”“引用计数推进”“最终观察者接手”压在同一原子变量的 RMW 链上。

### 7.3 lock-free 队列、栈和一次性发布状态机

如果一个对象的状态不是只被一个发布者改一次，而是被多线程继续原子推进，release sequence 就很容易进入核心路径。

## 8. 工业 / 现实世界锚点

### 8.1 GCC / LLVM 原子内建与 IR 语义

GCC `__atomic` builtins 和 LLVM atomics guide 都把原子 RMW 作为一等公民暴露出来。  
这说明现实里的编译器和运行时实现，确实把“同一原子对象上的 RMW 链”视为需要精确建模的工程对象。

### 8.2 lock-free 基础设施

真实工业代码里，release sequence 不常出现在普通业务逻辑里，但会反复出现在：

- runtime
- 并发容器
- 无锁队列
- 引用计数/状态机

这些“状态不是只写一次”的地方。

这里最危险的不是 API 不会用，而是把标准里这条链的边界看错。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 只有在你真的需要跨线程 RMW 链延续发布语义时，才主动依赖 release sequence
- 一旦依赖它，就显式画清：头部是谁，链条里的 RMW 是谁，最终 acquire 读到的是谁写的值
- 如果你的设计不需要这层微妙机制，优先退回更直接的 release/acquire 或 mutex

### 9.2 已经过时或必须带版本语境理解的路径

**C++20 之前那种“同线程普通原子写也能继续留在 release sequence 里”的旧理解**

当前工作草案的 `diff.cpp17` 明确说明：  
从 C++20 起，release sequence 的定义发生变化，头部后面的“写同一原子对象的操作”不再像旧规则那样自动延续 release sequence。

当前更稳的替代理解是：

- 把 release sequence 收窄理解为“头部 release + 连续 RMW 链”
- 不再把同线程后续普通 store 当成默认仍在链里

**拿 `memory_order_consume` 之类依赖顺序魔法来补 release sequence 理解**

这条路在当前主路径里也不再推荐。  
当前工作草案和 `P3475R2` 的方向都在收束到更直接、可实现、可分析的 acquire/release 主模型。

### 9.3 什么时候别手搓这玩意

如果你只是需要一个线程安全的共享状态，而不是在做 lock-free 基础设施，那么：

- 直接 release/acquire
- mutex
- condition variable
- 更高层并发封装

通常比把正确性压在 release sequence 细节上更稳。

## 10. 自测题 / 验证入口

1. 为什么 acquire 读到的不是最初 release 写下的值，也可能仍然同步回那个 release 头部？
2. release sequence 和整个 modification order 的区别是什么？
3. 当前草案下，为什么后继普通 store 不能自动留在 release sequence 里？
4. 为什么失败的 `compare_exchange` 不能像成功 RMW 一样延续 release sequence？
5. 在什么情况下你应该主动依赖 release sequence；在什么情况下你应该退回更直接的同步方案？
6. 为什么说“尾部 RMW 不一定自己是 release”与“工程上随便把它写成 relaxed”不是同一回事？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- `memory order` 的 release/acquire 传播链
- `happens-before` 的同步边构造
- CAS 循环、引用计数、状态机推进
- Linux kernel / runtime 里的原子状态字设计
- 其他语言里“通过同一原子状态推进 ownership 或 phase”的实现

迁移时最关键的问题始终是：

- 头部 release 是谁？
- 中间哪些修改真的是连续 RMW？
- 最终 acquire 到底读到了谁写出的那个值？

## 12. 未解问题与继续深挖

后续值得继续单独拆的点包括：

- release sequence 与 fence 的组合何时值得直接建模
- `atomic_wait` / `notify` 是否能在某些场景下把 release sequence 推理再压缩一层
- 在 hazard pointer、epoch reclamation 里，哪些 release sequence 最容易被误读

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `diff.cpp17`: https://eel.is/c++draft/diff.cpp17
- WG21 P3475R2, *Defang and deprecate memory_order::consume*: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3475r2.pdf
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
- LLVM Atomic Instructions and Concurrency Guide: https://llvm.org/docs/Atomics.html
