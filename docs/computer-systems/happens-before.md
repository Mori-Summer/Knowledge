---
doc_id: computer-systems-happens-before
title: Happens-Before：并发里真正决定“可见”与“成不成 race”的关系
concept: happens_before
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:16:47+08:00'
updated_at: '2026-03-20T15:58:02+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
  - llvm_threadsanitizer_docs_2026_03_16
  - linux_kernel_lkmm_explanation_2026_03_16
  - wg21_p3475r2_2025
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_concurrency_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/memory-order.md
  - docs/computer-systems/release-sequence.md
  - docs/computer-systems/data-race.md
  - docs/computer-systems/fence.md
open_questions:
  - 如何把真实并发 bug 里的 `happens-before` 图压缩成更短的排查步骤？
  - 当前草案里 `strongly happens-before` 还剩下哪些最值得单独拆解的使用场景？
  - sanitizer 对 `happens-before` 的近似建模与真实语言语义之间，最值得显式提醒的偏差有哪些？
---

# Happens-Before：并发里真正决定“可见”与“成不成 race”的关系

## 1. 这份文档要帮你学会什么

这份文档的目标，不是让你背一个定义，而是让你拿到一张以后可以反复用于并发分析的关系图。

读完后，你应该至少能做到：

- 说清 `happens-before` 到底在裁决什么问题
- 说清它和源码顺序、真实时间顺序、`synchronizes-with`、`modification order` 的边界
- 看懂为什么“一个写已经执行过了”并不等于“另一个线程必须看得见”
- 用 `happens-before` 分析 mutex、condition variable、atomic 发布-获取链路
- 判断一段代码为什么会变成 data race，或者为什么虽然并发但仍然是合法的

一句话先给结论：

**`happens-before` 不是“现实时间里的先后”，而是语言内存模型里一条可传递的偏序关系；它决定哪些写入必须对哪些读取可见，也决定哪些冲突访问会构成 data race。**

## 2. 一句话结论 / 问题定义

并发程序最麻烦的地方不是“代码看起来先写后读”，而是：

- 编译器和硬件都可能改变观察顺序
- 不同线程不共享一个天然可靠的“时间线”
- 某个写入做过了，不等于另一个线程已经被标准强制要求看见

这时就需要一个**可裁决**的关系，而不是直觉。

`happens-before` 要解决的问题就是：

- 哪些操作在模型里被视为“确定发生在另一些操作之前”
- 哪些 side effect 对哪些读取可见
- 哪些冲突访问没有被这条关系隔开，从而构成 data race

换句话说，它真正回答的是：

- “先发生”在语言层到底是什么意思
- 哪些边只是直觉，哪些边才是标准承认的边

## 3. 对象边界与相邻概念

### 3.1 `happens-before` 管什么

它主要管三件事：

- 并发执行中哪些评价与 side effect 建立了先后关系
- 某次读取能把哪些写入当成“已经发生”
- 两个冲突访问之间有没有足够的同步隔离

### 3.2 它不等于什么

它不等于：

- 源码里写在前面的语句
- 机器上更早退休或更早提交的指令
- 日志里更早打印出来的事件
- 某个原子对象自己的 `modification order`
- cache coherence

### 3.3 和几个相邻概念的边界

**`sequenced-before` vs `happens-before`**

- `sequenced-before` 是单线程内部的求值顺序关系
- `happens-before` 可以跨线程，并且是更高层的可传递关系

单线程里的 `sequenced-before` 往往是构造 `happens-before` 的原材料之一，但两者不是同一个概念。

**`synchronizes-with` vs `happens-before`**

- `synchronizes-with` 是一条具体的跨线程同步边
- `happens-before` 是把 `sequenced-before`、`synchronizes-with` 等关系做传递闭包之后得到的更大关系图

换句话说，`synchronizes-with` 更像边，`happens-before` 更像整个图。

**`modification order` vs `happens-before`**

- `modification order` 只针对某一个原子对象，给出该对象修改的单独总序
- `happens-before` 可以把不同对象、普通内存、锁操作、条件变量等待等关系连起来

**`happens-before` vs cache coherence**

coherence 主要回答“同一个地址的写入如何被一致观察”；`happens-before` 回答的是“哪些效果必须先于另一些效果被看见”。

coherence 不自动替你把“标志位之外的其他数据”一起发布出去。

### 3.4 最关键的边界句

可以把它压成一句话：

**真正重要的不是‘谁先执行了’，而是‘哪条边让另一个线程被标准要求必须把它当先发生’。**

## 4. 核心结构

### 4.1 把 `happens-before` 看成图，而不是一句定义

要把 `happens-before` 变成一个可操作模型，至少要抓住下面六个构件。

| 构件 | 它是什么 | 缺了会怎样 |
| --- | --- | --- |
| 评价与 side effect | 真正被关系连接的基本单位 | 你会误把“代码行”当分析对象 |
| `sequenced-before` | 单线程内部的局部顺序边 | 图没有原材料 |
| `synchronizes-with` | 跨线程同步边入口 | 图跨不过线程边界 |
| 传递性 | 把局部边拼成大图 | 看不到真正的可见性链 |
| 可见 side effect | 读取到底能合法看见哪些写 | 无法把图落回观察结果 |
| data race 判定 | 图缺失时程序何时直接非法 | 不知道这张图为什么重要 |

### 4.2 边从哪里来

可以把边源分成三类：

| 边类型 | 典型来源 | 作用 |
| --- | --- | --- |
| 单线程边 | `sequenced-before` | 把线程内普通写、普通读、原子操作串起来 |
| 跨线程同步边 | `synchronizes-with` | 让状态真正从一个线程合法交给另一个线程 |
| 传递边 | `hb` 的闭包 | 让“先写数据，再发布；先接收，再读数据”变成完整链 |

这张表最重要，因为它告诉你：

- 你不是直接“得到一张 HB 图”
- 你是先拿到边，再做闭包

### 4.3 三类最常见的跨线程边

对大多数工程代码，最重要的 `synchronizes-with` 来源基本就三类：

| 类别 | 例子 | 你该记住什么 |
| --- | --- | --- |
| 锁边 | `unlock` -> 后续成功 `lock` | mutex 真正值钱的是建边，不只是挡人 |
| 原子发布边 | `store(release)` -> 读到该值的 `load(acquire)` | 发布-接收是 HB 图最常见桥梁 |
| 等待边 | condition variable / wait-notify 机制 | 等待链本质上仍靠共享状态与同步边成立 |

### 4.4 两个不要混的顺序

很多错误都来自把下面两件事混了：

- **现实顺序**
  机器上谁先执行、日志谁先打印、谁先调度到

- **模型顺序**
  标准是否要求一个操作在另一个操作之前被观察到

`happens-before` 只关心第二件事。

### 4.5 单一对象的历史，不等于整个程序的历史

`modification order` 给你的是某个 atomic object 自己的历史线。

`happens-before` 给你的是：

- 如何把多个对象、普通内存、锁、等待、原子同步拼到一起

所以最常见的误判就是：

- “这个 flag 的历史很清楚”
- 于是误以为“和它相关的 data 也就一起清楚了”

并不会。除非你真正把它们连进 HB 图。

### 4.6 一页纸判断模板

以后看到并发读写代码，先过这张表：

| 问题 | 你必须回答什么 |
| --- | --- |
| 图里有哪些节点 | 具体是哪些读、写、锁、解锁、wait、notify、原子操作 |
| 单线程边在哪里 | 哪些操作在本线程内 `sequenced-before` |
| 跨线程边在哪里 | 哪个 unlock/lock、哪个 release/acquire、哪个等待链建立了 `synchronizes-with` |
| 传递链是否完整 | 普通写有没有真的被串到读取方 |
| 如果边断了会怎样 | 是读到旧值，还是直接 data race |

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最常见的一条链：锁保护共享状态

```cpp
// Thread A
std::lock_guard<std::mutex> g(m);
data = 42;
ready = true;

// Thread B
std::lock_guard<std::mutex> g(m);
if (ready) {
    use(data);
}
```

这里真正值钱的不是“加了锁就安全”这句口号，而是：

1. 线程 A 在临界区里写 `data` 和 `ready`
2. A 解锁 `m`
3. 后续线程 B 成功锁住同一个 `m`
4. 当前草案规定：该 `unlock()` 与该 `lock()` `synchronize-with`
5. 于是 A 解锁前的写入，通过 `happens-before` 关系对 B 加锁后的读取变得可见

你应该看到的不是“mutex 很神奇”，而是：

**mutex 在替你建立 `happens-before`。**

### 5.2 第二条主链：release / acquire 发布与接收

```cpp
// Thread A
data = 42;
flag.store(1, std::memory_order_release);

// Thread B
if (flag.load(std::memory_order_acquire) == 1) {
    use(data);
}
```

这里发生的是：

1. A 先写普通数据 `data`
2. 再用 release store 发布 `flag`
3. B 用 acquire load 读到这个发布值
4. 该发布与接收建立 `synchronizes-with`
5. A 在 release 之前的写入，经由 `happens-before` 对 B 在 acquire 之后的读取可见

这条链最值得记住的不是“release/acquire 很强”，而是：

**HB 图必须把普通写也串进来，否则你拿到的只是 flag 的历史，不是 data 的可见性。**

### 5.3 第三条主链：condition variable 不是魔法广播

条件变量也很容易被误解。

真正稳的模式仍然是：

- 用同一个 mutex 保护共享状态
- 用 predicate 判断状态是否成立
- 把 `wait()` 放在循环里

为什么？因为这条链其实仍然是在建 HB：

1. 修改方持同一把锁更新共享状态
2. 修改方解锁并通知
3. waiter 在 wait 中释放锁、睡眠、被唤醒、重新拿锁
4. waiter 在重新拿到锁后检查 predicate

条件变量不是替代 `happens-before`；它只是另一种建立和利用 `happens-before` 的机制。

### 5.4 第四条主链：从 HB 图回到 data race 判定

把前面的链条收束起来，你会发现：

- 如果一条写入能通过 `happens-before` 走到某个读取，那么这个读取在正确条件下就应当能看见相应效果
- 如果两个冲突访问之间没有被 `happens-before` 隔开，那么程序就可能直接非法

所以它既不是“性能提示”，也不是“文档上的理论关系”，而是：

- **可见性的裁判线**
- **data race 的生死线**

### 5.5 五步排障链：为什么这里读到了旧值或直接成 race

这篇文档最重要的可调用部分，是下面这条排障链：

1. **先列节点。**
   具体是哪个写、哪个读、哪个锁、哪个原子操作？

2. **再列单线程边。**
   哪些写本来在本线程里先后发生？

3. **再找跨线程边。**
   哪个 unlock/lock、release/acquire、wait/notify 真的把图跨过去了？

4. **再看传递链是否闭合。**
   你关心的普通写有没有被串到目标读之前？

5. **最后分类结果。**
   如果边不够，是“可能读旧值”，还是“已经构成 data race”？

### 5.6 真正的因果核心

把这一节压成一句最值得记住的话：

**`happens-before` 不是时间线，而是由单线程顺序边和跨线程同步边拼起来的一张图；并发正确性的关键不是‘谁先跑了’，而是‘这张图是否真的把你关心的写入送到了那个读取之前’。**

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

`happens-before` 的优点是精确，代价是抽象。

- 好处：它给了你一套跨编译器、跨硬件、跨库抽象的统一判断语言
- 代价：它不迎合直觉，很多“看起来先发生”的东西在模型里并不成立

### 6.2 六类高频失败模式

**失败模式 1：把墙钟时间当 `happens-before`**

日志里先打印，不等于标准里先发生。某条指令更早执行过，也不等于别的线程必须已经能看见它。

**失败模式 2：把“变量是 atomic”误当“已经建立 `happens-before`”**

原子性只解决该对象本身的竞态与撕裂问题，不自动给相关普通数据建立可见性关系。

**失败模式 3：把 coherence 当 `happens-before`**

coherence 只给同一地址提供一致观察规则；它不自动保证“先看见标志，再看见对应数据”。

**失败模式 4：条件变量不用 predicate**

如果你只盯着 `notify_one()` / `wait()`，却没有在共享状态上建立正确条件和锁保护，最后仍然会出错。

**失败模式 5：忘记 `happens-before` 是图，不是一条边**

很多 bug 就出在这里：程序员只盯着最后一个 acquire 或最后一个 lock，却忘了前面那段普通写入到底有没有被真正串到这条链里。

**失败模式 6：先讨论业务逻辑，不先画 HB 图**

这会让你在“为什么读到旧值”上空转很久，因为你还没先回答“它有没有资格读到新值”。

### 6.3 现场症状与优先回查方向

| 现场症状 | 优先回查什么 | 常见真实原因 |
| --- | --- | --- |
| 标志位看见了，数据还是旧的 | HB 图是否只覆盖了 flag | 普通写没被串入发布链 |
| condition variable 代码偶发失效 | predicate 与锁纪律 | 没建立完整等待链 |
| 原子代码看起来没 race，但结果仍随机 | HB 是否根本没建立 | 原子了，但没有同步边 |
| sanitizer 报 race，而人眼觉得“明明先执行了” | 现实顺序 vs 模型顺序 | 用时间直觉代替了 HB 图 |
| 代码里有很多 acquire/release 仍然读到旧值 | 图是否断在中间 | 只连了边缘节点，没连 payload |

### 6.4 为什么 HB 图最容易在“中间一段”断掉

最常见的错误不是完全没有边，而是：

- 两端看起来都有同步动作
- 中间真正关心的普通写没被连进去

例如：

- 先写 payload
- 再写 flag
- 读线程先看 flag
- 但 payload 写没有被正确放在 release 之前或 acquire 之后

这类 bug 之所以难，是因为：

- 头尾都有“同步味”
- 但中间那段图实际上断了

### 6.5 一个很实用的判别句式

如果你看到：

- **我明明先写了**，先问它有没有 HB 到对面
- **这个变量是 atomic**，先问它有没有替相关数据建边
- **这里已经 notify 了**，先问 predicate 是否真的在锁纪律下成立
- **这段代码看着很像对的**，先把边画出来，不要先相信直觉

## 7. 应用场景

### 7.1 mutex 保护的共享状态

这是最典型、也最稳的 `happens-before` 使用场景。

### 7.2 发布-订阅

一个线程准备数据，另一个线程在条件成立后读取数据，这本质上就是在搭一条 `happens-before` 链。

### 7.3 condition variable 等待条件成立

等待线程不是在“等通知”，而是在等“某个共享状态通过同步关系变得可安全观察”。

### 7.4 并发 bug 定位与 race 检测

一旦你要解释“为什么这次读到了旧值”或“为什么这里构成 data race”，最终都会落回 `happens-before` 图。

## 8. 工业 / 现实世界锚点

### 8.1 C++ 标准库同步原语本质上都在替你建边

现实工程里，大多数 `happens-before` 并不是程序员手画出来的，而是通过：

- mutex
- condition variable
- release/acquire 原子操作

这些库接口被稳定建立出来的。

这说明 HB 不只是理论词，它就是标准库同步原语真正提供的现实价值之一。

### 8.2 ThreadSanitizer 之类工具之所以有意义，就是因为 HB 图可近似

LLVM 官方文档把 ThreadSanitizer 定位为 data race 检测工具。这类工具之所以有意义，正是因为工程上需要把“哪些访问被同步隔开、哪些没有”落成可检查模型。

这本身也是一个现实锚点：

- 如果 HB 根本不可操作，race detector 也无从工作

### 8.3 Linux 内核内存模型也把 HB 当成核心解释框架

Linux LKMM 文档把 `happens-before` 放在核心解释框架里，这说明它不是“C++ 教材里的抽象词”，而是系统工程在分析屏障、锁和并发访问时真正在用的模型。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

对大多数工程代码，更稳的路径是：

- 先用高层同步原语建立 `happens-before`，而不是先猜硬件会不会“恰好按你想的顺序”
- 先画出边：谁 `sequenced-before` 谁，谁 `synchronizes-with` 谁
- 对条件变量，总是把共享状态和 predicate 放在第一位，而不是把通知本身当成核心
- 对原子发布链路，优先用显式的 release/acquire 建模
- 遇到并发 bug 时，先判 HB 图是否存在，再谈业务结果为何错

### 9.2 需要带历史语境理解的旧路径

| 路径 | 为什么旧/需带语境 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| consume 时代的 `dependency-ordered before / simply happens-before` 包袱 | 当前工作草案主路径已重新收束到更直接的 HB 主模型 | 普通工程代码更难稳定依赖 | 用明确的 acquire/release 或更高层同步原语 |
| “按真实时间或 x86 经验推理”的旧习惯 | 这不是标准意义上的可移植分析方法 | 平台偶然性太强 | 回到语言模型里明确列出同步边 |
| 只盯最后一个同步动作 | 会漏掉普通写入是否真正串上图 | 图常断在中间 | 画完整链，不只看末端 |

### 9.3 什么时候别直接手算

如果你只是要安全共享复杂状态，而不是做 lock-free 基础设施，那么：

- mutex
- condition variable
- `std::jthread` / join 语义
- 更高层并发库

通常比手工拼一堆细碎原子关系更稳。

HB 图是分析工具，但不是要求所有业务代码都手写底层边。

## 10. 自测题 / 验证入口

1. 为什么日志里更早打印的事件，不等于它 `happens-before` 另一个线程里的读取？
2. `synchronizes-with` 和 `happens-before` 的区别是什么？
3. 为什么“一个变量是 atomic”不等于“相关普通数据已经被正确发布”？
4. 为什么条件变量等待不能只靠 `notify_one()`，还要靠 predicate 和锁？
5. 两个冲突访问在什么条件下会构成 data race？
6. 如果你读到旧材料里的 `simply happens-before` 或 `memory_order_consume`，当前更稳的工程替代路径是什么？
7. 为什么“边从哪里来”比“代码看起来谁先写”更重要？
8. HB 图里最容易断在哪一段？为什么？
9. 为什么 mutex 的真正价值之一可以被翻译成“替你建 HB”？
10. 请用一句话解释：HB 图到底在裁决什么。

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的核心句

理解了这篇文档后，你应该能把下面这句迁移出去：

**并发里真正重要的不是‘发生过’，而是‘有没有被标准承认为先发生’。**

### 11.2 可以迁移到哪些领域

- `memory order` 的发布-获取链路
- `release sequence` 的跨线程 RMW 链
- `mutex` / `condition_variable` 的同步语义
- Linux kernel barrier 与 LKMM 分析
- Java 内存模型里的 visibility 关系
- Rust 并发原语的安全发布与 race 分析

### 11.3 与相邻模型的关系

| 模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| `sequenced-before` | 给出单线程局部顺序 | 补如何跨线程形成更大图 |
| `synchronizes-with` | 给出具体跨线程同步边 | 补这些边如何变成完整 HB 图 |
| `memory_order` | 定义原子操作顺序语义 | 补这些语义最终如何落到可见性和 race 判定 |
| `data_race` | 判语言层合法性 | 补判定时真正看的关系图 |
| race detector | 近似抓缺失的同步边 | 补其理论基础为何是 HB 图 |

### 11.4 最值得保留的迁移动作

迁移时最关键的问题始终是：

- 边从哪里来？
- 这条边是单线程顺序、同步边，还是它们的传递结果？
- 我依赖的“可见”到底有没有被标准模型真正建立出来？

## 12. 未解问题与继续深挖

- `strongly happens-before` 在当前草案里还剩哪些最实用的区分价值？
- race detector 在真实工业代码里如何近似实现 `happens-before` 图？
- `atomic_wait` / `notify`、fence、mixed-order 场景该怎样以更短方式纳入同一模型？

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `thread.mutex.requirements.mutex`: https://eel.is/c++draft/thread.mutex.requirements.mutex
- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- WG21 P3475R2, *Defang and deprecate memory_order::consume*: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3475r2.pdf
- Clang ThreadSanitizer: https://clang.llvm.org/docs/ThreadSanitizer.html
- Linux kernel memory model explanation: https://docs.kernel.org/dev-tools/lkmm/docs/explanation.html
