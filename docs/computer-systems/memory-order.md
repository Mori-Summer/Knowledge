---
doc_id: computer-systems-memory-order
title: Memory Order：从原子性到可见性与重排序控制
concept: memory_order
topic: computer-systems
created_at: '2026-03-16T10:40:00+08:00'
updated_at: '2026-03-19T21:20:00+08:00'
source_basis:
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_depr_atomics_order_2026_03_16
  - cxx_draft_intro_races_2026_03_16
  - wg21_p3475r2_2025
  - wg21_p0668r3_2018
  - gcc_atomic_builtins_docs_2026_03_16
  - llvm_atomics_guide_2026_03_16
  - linux_kernel_memory_barriers_2026_03_16
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_concurrency_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/virtual-memory-learning-model.md
  - docs/programming-languages/cpp20-coroutine-playbook.md
open_questions:
  - 如何把 release sequence、fence 和 mixed-order litmus tests 再压缩成更短的判断模板？
  - 如果后续单独写 `happens-before` 文档，应该如何与这篇文档拆边界？
---

# Memory Order：从原子性到可见性与重排序控制

## 1. 这份文档要帮你学会什么

这份文档不是为了让你记住几个枚举值，而是为了让你拿到一个以后可以反复调用的并发模型。

读完后，你应该至少能做到：

- 说清 `memory order` 到底在解决什么问题
- 说清它和 `atomic`、`happens-before`、可见性、重排序之间的边界
- 理解 `relaxed`、`acquire`、`release`、`acq_rel`、`seq_cst` 的本质差异
- 判断某个并发代码为什么“原子了但还是错”
- 在 lock-free 结构、发布-订阅、计数器、标志位等场景中大致知道该从哪种 order 起步

一句话先给结论：

**`memory order` 不是“让操作变原子”的开关，而是“告诉编译器和硬件：这个原子操作周围的读写，哪些必须被别的线程以什么顺序看见”的契约。**

## 2. 一句话结论 / 问题定义

并发程序真正麻烦的地方，不只是“两个线程同时写同一个变量”，而是：

- 编译器会重排
- CPU 会乱序执行
- 不同核心观察到写入生效的时间不一致
- 一个线程以为“我已经先写了数据再写标志”，另一个线程却可能先看见标志，再看见旧数据

`memory order` 就是为了解决这个问题。

它给原子操作附上一个顺序与同步语义，让程序不仅知道“某次访问是不可分割的”，还知道：

- 这次访问是否建立同步关系
- 这次访问之前/之后的普通内存操作能否被移动
- 另一个线程在什么条件下能看见这些效果

## 3. 对象边界与相邻概念

### 3.1 `memory order` 管什么

它主要管三件事：

- 原子操作是否建立线程间同步
- 一个原子操作附近的普通读写能否被重排
- 另一个线程何时能把这些效果当成“已经发生”

### 3.2 它不等于什么

它不等于：

- 原子性本身
- 锁本身
- cache coherence 本身
- “代码执行顺序看起来像源码顺序”

更具体地说：

- `atomic` 解决的是“单次访问不可撕裂、不会数据竞争”这一层
- `memory order` 解决的是“别的内存操作如何跟着这次原子访问一起建立顺序和可见性”
- `happens-before` 是更上层的关系，用来描述哪些写入对哪些读取可见
- 编译器/CPU 重排序是底层现象，`memory order` 是约束这类现象的编程接口

### 3.3 和几个相邻概念的边界

**`atomic` vs `memory order`**

- `atomic` 回答：“这次访问会不会被并发撕裂？”
- `memory order` 回答：“这次访问会不会顺便给别的读写建立顺序约束？”

所以，“变量是原子的”不代表“周围数据就已经按你想的顺序发布了”。

**`memory order` vs `happens-before`**

- `memory order` 是你写代码时指定的语义接口
- `happens-before` 是程序执行后成立的关系图

例如一个 `store(memory_order_release)` 与一个读取到该值的 `load(memory_order_acquire)`，会建立同步边，从而让前者之前的写入对后者之后的读取变得可见。

**`memory order` vs cache coherence**

cache coherence 保证的是“同一个地址的写入有某种一致观察规则”；它不自动给“这个地址之外的别的数据”建立顺序保证。

这正是很多初学者踩坑的地方：  
“标志位已经是原子的，为什么数据还是旧的？”  
因为 coherence 管的是那个标志位本身，不自动替你发布整批相关数据。

## 4. 核心结构

如果要把 `memory order` 模型化，至少要抓住下面六个构件。

### 4.1 原子对象

所有顺序语义都附着在原子对象或原子 fence 上。  
没有原子访问，谈这些顺序规则基本没有意义，因为程序先掉进 data race 或未定义行为了。

### 4.2 程序顺序

源码里“先写 A 再写 B”只是当前线程视角下的程序顺序。  
这不等于别的线程一定会按同样顺序观察到结果。

### 4.3 modification order

C++ 草案要求：对每个原子对象，都存在一个独立的修改总序。  
这保证你不会得到“同一个原子对象的写入完全无序”的荒唐结果，但这仍然只是**每个原子对象各自**的序，不是所有对象的全局序。

### 4.4 同步边

`release` 和 `acquire` 的本质，不是“一个强、一个弱”，而是它们能在合适的读写配对上建立同步边。

如果没有这个同步边，很多普通内存写入只是“看起来写过了”，不等于另一个线程已经必须看见。

### 4.5 `happens-before`

当同步边、程序顺序等关系拼起来时，就形成了 `happens-before`。  
你要的不是“我觉得它应该先发生”，而是“标准模型里它确实先发生”。

### 4.6 五种当前主 order

截至 `2026-03-16`，当前 C++ 工作草案在 `atomics.order` 主条文里列出的常规 `std::memory_order` 枚举值是：

- `relaxed`
- `acquire`
- `release`
- `acq_rel`
- `seq_cst`

与此同时，兼容性附录 `depr.atomics.order` 仍保留一个**附加的、已弃用的** `memory_order::consume = 1`，并规定它在允许出现的地方与 `memory_order::acquire` 具有相同语义。  
换句话说，更准确的表述不是“`consume` 从世界上消失了”，而是“它已经退出主路径，退化为兼容性保留项”。  
WG21 2025 papers 页面显示 `P3475R2 Defang and deprecate memory_order::consume` 已于 `2025-02` 标记为 adopted，目标为 `C++26`。这是一个需要显式带日期理解的“当前状态”。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最经典的发布-获取链路

这是理解 `memory order` 最值钱的一条主链路：

```cpp
// Thread A
data = 42;                                  // 普通写
ready.store(true, std::memory_order_release);

// Thread B
if (ready.load(std::memory_order_acquire)) {
    use(data);                              // 普通读
}
```

这里真正发生的事是：

1. 线程 A 先写 `data`
2. 再用 `release` store 发布 `ready = true`
3. 线程 B 用 `acquire` load 读到这个 `true`
4. 一旦读到的是这个发布出来的值，A 在 `release` 之前的写入，就必须对 B 在 `acquire` 之后的读取可见

这就是为什么大家常说：

- `release` 像“发布”
- `acquire` 像“接收”

### 5.2 `relaxed` 到底少了什么

`relaxed` 不是“没意义”。  
它仍然保证对**该原子对象本身**的原子性和修改顺序。

但它不建立一般性的同步关系。

所以这类代码通常是错的：

```cpp
// Thread A
data = 42;
ready.store(true, std::memory_order_relaxed);

// Thread B
if (ready.load(std::memory_order_relaxed)) {
    use(data);  // 这里不安全
}
```

为什么？

- `ready` 自己是原子的
- 但 `data` 的发布没有被这次 store/load 建立同步边
- 于是 B 可以看到 `ready == true`，却仍然看到旧的 `data`

### 5.3 `acq_rel` 的位置

`acq_rel` 主要给 read-modify-write 操作用，比如 `fetch_add`、`compare_exchange` 这类既读又写的原子操作。

它的含义是：

- 作为读的一面，它像 acquire
- 作为写的一面，它像 release

这很适合用于“我一边接过别人发布的状态，一边又发布新的状态”的原子更新点。

### 5.4 `seq_cst` 多出来的东西

`seq_cst` 不是“比 acquire/release 多几个栅栏”这么简单。  
它的核心是：所有 `seq_cst` 操作要进入一个单一全序。

直觉上你可以先把它理解成：

- 它在跨线程 reasoning 上通常最像人脑里那种“大家都沿着一个统一时间线看这些原子操作”
- 但这个“像”并不代表程序里所有普通读写都 magically 线性化了

当前草案还特别提醒：`seq_cst` 与更弱顺序混用时，可能出现让人意外的结果。  
所以 `seq_cst` 不是“只要我某些地方写了最强顺序，其他地方弱一点也自动安全”的保险按钮。

### 5.5 一个重要但容易忽略的点

Linux kernel barrier 文档明确提醒：`ACQUIRE` 后接 `RELEASE`，**不能**默认当成 full memory barrier。  
这是一个很好的现实世界提醒：  
不要把 acquire/release 想成“万能双向墙”，它们本质上是定向约束。

但要注意：这里引用 Linux kernel barrier 文档，是为了给你一个系统工程里的直觉锚点，不是说 Linux 文档本身在定义 C++ 语言语义。真正的语言层判断，仍然应回到 C++ 内存模型与标准文本。

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

`memory order` 的 tradeoff 不是“强一点更好”。

真正的 tradeoff 是：

- 更弱：性能和实现自由度更大，但 reasoning 成本高，错误更隐蔽
- 更强：更容易推理、更稳，但在弱内存序架构上可能更贵，也更容易把并发优化空间锁死

### 6.2 常见失败模式

**失败模式 1：把“原子”误当“已同步”**

这是最常见错误。  
一个原子标志位只说明标志位本身安全，不说明它保护的数据已经被正确发布。

**失败模式 2：默认硬件和编译器会按源码顺序来**

x86 上有些代码“似乎能跑”，会让人形成危险错觉。  
但可移植并发代码必须按语言内存模型推理，而不是按“我这台机器目前没出事”推理。

**失败模式 3：把 acquire/release 当 full barrier**

这在 lock-free 结构、ring buffer、复杂状态机里尤其容易出错。

**失败模式 4：混用 `seq_cst` 与较弱顺序，却以为全局顺序直觉还成立**

WG21 的修订论文明确指出，这一类混用在 Power、ARM 等架构上曾暴露出很麻烦的可实现性和推理问题。

**失败模式 5：随手用 `relaxed` 做“优化”**

在没有明确证明正确性之前，`relaxed` 通常不是“聪明的性能优化”，而是“延迟爆炸的 bug 注入器”。

## 7. 应用场景

### 7.1 计数器与统计量

如果一个原子值只是独立计数，不负责发布别的数据，那么 `relaxed` 常常够用。

例如：

- QPS 计数
- 命中次数统计
- debug metrics

关键前提是：  
没人依赖“看见计数变化”这件事去推断“其他数据现在也该可见了”。

### 7.2 发布-订阅 / 准备好标志

如果一个线程先准备数据，再告诉另一个线程“现在可以读了”，这是典型的 `release`/`acquire` 场景。

### 7.3 lock-free 数据结构

CAS、`fetch_add`、版本号推进、指针发布这些地方，常用 `acq_rel` 或更复杂组合。  
这类场景里最容易出问题的，不是“原子 API 不会用”，而是：

- 哪一步是在发布
- 哪一步是在接收
- 哪些普通内存访问必须被带过去

### 7.4 默认先求稳的跨线程状态协调

如果你没有性能证据，也没有严谨证明，`seq_cst` 通常是更稳的起点。  
先写对，再局部变弱，通常比一开始追最弱顺序更靠谱。

## 8. 工业 / 现实世界锚点

### 8.1 C++ 运行时、无锁队列与高性能并发库

工业里 `memory order` 最真实的应用，不是在面试题里，而是在这些地方：

- lock-free queue / stack
- 线程池与调度器
- 引用计数和生命周期管理
- 一次发布、多次读取的数据结构

在这些系统里，问题从来不是“有没有 atomic”，而是：

- 这个读是只想看值，还是想拿到它所发布的整批状态？
- 这次 RMW 是在读旧状态、更新新状态，还是在做同步点？
- 你优化掉的到底是多余顺序，还是正确性本身？

### 8.2 Linux 内核屏障实践

Linux kernel 官方 barrier 文档是很好的现实锚点，因为它反复强调：

- 屏障不是魔法
- acquire/release 不是 full barrier
- 编译器和 CPU 都会做你不想看到的移动

这类文档的价值在于，它把 `memory order` 从“语言课本概念”压到了“系统工程里你真的会因为它踩坑”的层次。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

对于大多数工程代码，比较稳的起手式是：

- 默认先用 `seq_cst`，除非你有明确性能压力和正确性证明
- 发布数据时用 `release`，接收数据时用 `acquire`
- 对只保护自身数值、不发布其他状态的独立计数器，才认真考虑 `relaxed`
- 对 CAS / `fetch_add` / 状态推进这种既读又写的同步点，再考虑 `acq_rel`

换句话说，推荐顺序通常不是“先想怎么最弱”，而是“先想我在建立什么同步边”。

### 9.2 已经过时、明显不推荐或需要带日期理解的路径

**`memory_order::consume`**

这是目前最需要带时间语境理解的点。

- 当前 WG21 2025 论文 `P3475R2` 目标是 `C++26`
- 2025 papers 页面显示它已在 `2025-02` 标记为 adopted
- 当前工作草案在 `atomics.order` 主条文中已不把它列为常规枚举值，而是在 `depr.atomics.order` 中以兼容性附加枚举值保留，且语义与 `acquire` 相同

这里要特别避免一个常见误解：  
“当前工作草案已经这样”不等于“你今天手上的所有编译器、所有已发布标准模式、所有旧代码都已经完全长成这样”。  
更准确的理解是：

- 就当前工作草案走向而言，`consume` 已经退出主推荐路径
- 就主流实现实践而言，编译器长期也没有把它当成一个独立、可靠、可依赖的用户态建模工具
- 但在旧标准文本、旧代码、特定工具链文档里，你仍然可能看到它的名字继续存在

但现实编译器生态仍有遗留：

- GCC 文档仍列出 `__ATOMIC_CONSUME`
- 并明确说明它目前按更强的 `__ATOMIC_ACQUIRE` 实现
- LLVM 文档也直接写明：前端应把 C/C++ `memory_order_consume` 按 `Acquire` 处理

所以当前更稳的结论不是“consume 完全没历史价值”，而是：

- 不要把它当现代 C++ 用户态代码里的推荐起点
- 如果看到历史代码、RCU 论文或内核风格依赖链路，要知道它在概念上想表达什么
- 但在今天的主流用户态 C++ 实践里，应优先按 acquire/release 或更高层同步原语建模

**legacy `__sync` builtins**

GCC 官方文档明确说，新代码应优先用 `__atomic` builtins 替代 legacy `__sync` builtins。  
原因很简单：`__atomic` 才显式携带内存顺序参数，更接近当前语言内存模型。

### 9.3 什么时候别自己手搓

如果你只是要安全地共享复杂状态，而不是写 lock-free 基础设施，那么：

- mutex
- condition variable
- channel / queue
- 更高层并发库

往往比手工调 `memory order` 更稳。  
`memory order` 是底层能力，不代表所有场景都该直接裸用。

## 10. 自测题 / 验证入口

1. 为什么“某个变量是原子的”不等于“它保护的数据已经被正确发布”？
2. `relaxed` 保留了什么，放弃了什么？
3. 为什么 `store(release)` + `load(acquire)` 能把普通写入一起带过去？
4. 为什么 `acquire` 后面接 `release` 不能默认当 full barrier？
5. 在什么情况下你应该从 `seq_cst` 起步，而不是一开始就追求 `relaxed`？
6. 截至 `2026-03-16`，为什么说 `memory_order::consume` 需要带时间语境理解？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把同一套模型迁移到：

- `mutex` 的 lock/unlock acquire-release 语义
- `condition_variable` 的等待与通知配合
- Linux kernel barrier / RCU / ring buffer 设计
- Rust `Ordering`
- Java `volatile` / 原子类
- lock-free bug 分析里的“为什么看见了标志却没看见数据”

迁移时最关键的问题始终是：

- 哪一步是在发布状态？
- 哪一步是在接收状态？
- 我依赖的是“这个原子值本身”，还是“它所代表的整批其他内存效果”？

## 12. 未解问题与继续深挖

后续值得单独成文或继续深挖的点包括：

- `happens-before` 该怎样单独讲，才不会和这篇文档重复又不会割裂
- release sequence、fence、mixed-order litmus tests 如何压缩成更短的判断套路
- 在 RCU、hazard pointer、epoch reclamation 这类内存回收机制里，`memory order` 应该如何继续细分

## 13. 参考资料

以下链接均为本次写作时实际参考的官方或一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `depr.atomics.order`: https://eel.is/c++draft/depr.atomics.order
- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- WG21 2025 papers 页面，`P3475R2` 条目与 adopted 状态: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/
- WG21 P3475R2, *Defang and deprecate memory_order::consume*: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3475r2.pdf
- WG21 P0668R3, *Revising the C++ memory model*: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0668r3.html
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
- LLVM Atomic Instructions and Concurrency Guide: https://llvm.org/docs/Atomics.html
- Linux kernel memory barriers: https://www.kernel.org/doc/html/latest/core-api/wrappers/memory-barriers.html
