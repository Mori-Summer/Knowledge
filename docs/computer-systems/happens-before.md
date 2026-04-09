---
doc_id: computer-systems-happens-before
title: Happens-Before：把线程内顺序和跨线程同步边拼成可见性总图
concept: happens_before
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:16:47+08:00'
updated_at: '2026-04-09T12:20:00+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
  - llvm_threadsanitizer_docs_2026_03_16
  - linux_kernel_lkmm_explanation_2026_03_16
  - wg21_p3475r2_2025
  - concurrency_term_cluster_rewrite_review_2026_04_09
  - methodology_document_generation_methodology
time_context: current_cxx_memory_model_and_tooling_baseline_checked_2026_03_16
applicability: concurrency_graph_modeling_visibility_reasoning_data_race_judgment_and_sync_debugging
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/computer-systems/synchronizes-with.md
  - docs/computer-systems/modification-order.md
  - docs/computer-systems/release-sequence.md
  - docs/computer-systems/data-race.md
  - docs/computer-systems/memory-order.md
  - docs/computer-systems/fence.md
open_questions:
  - 如何把真实并发 bug 里的 `happens-before` 图进一步压缩成更短的排查步骤？
  - `strongly happens-before` 在当前草案语境下还剩哪些值得单独展开的使用场景？
  - sanitizer 对 `happens-before` 的近似建模与真实语言语义之间，哪些偏差最值得给工程师长期提醒？
---

# Happens-Before：把线程内顺序和跨线程同步边拼成可见性总图

## 1. 这份文档要帮你学会什么

这篇文档的重点，不是继续把 `happens-before` 写成一句定义解释，而是把它收成这组并发术语之上的总图文档。

读完后，你应该至少能做到：

- 说清 `happens-before` 为什么不是“现实时间顺序”，而是并发推理里的总关系图
- 区分 `sequenced-before`、`synchronizes-with`、`modification order`、`memory order` 各自处在图里的哪一层
- 看懂为什么“某个写已经执行过”不等于“另一个线程必须看得见”
- 用一张图来分析 mutex、release/acquire、condition variable、`atomic_wait` 一类代码
- 判断一段代码为什么会 data race，或者为什么虽然并发但仍然处在语言模型允许区域

## 2. 一句话结论 / 问题定义

**`happens-before` 不是墙钟时间里的“先后”，而是语言内存模型里由线程内顺序、跨线程同步边和传递性拼成的偏序总图；它决定哪些写入必须对哪些读取可见，也决定哪些冲突访问没有被合法隔开。**

并发里真正麻烦的不是“谁先执行了”，而是：

- 哪些效果被另一线程**必须**当成已经发生
- 哪些效果只是你根据运行现象或平台直觉猜测已经发生
- 哪些冲突访问根本没有被这张图隔开

所以，这篇文档真正要回答的是：

- 并发里“先发生”在语言层到底是什么意思
- 一段代码里的可见性、合法性和读值推理，到底应该贴着什么图来做

## 3. 对象边界与相邻概念

### 3.1 这篇文档直接处理什么

这篇文档直接处理的是：

- `happens-before` 作为并发总关系图的本体
- 图里的节点、边和传递闭包
- 这张图如何落回可见性与 data race 判定
- 这张图和相邻概念的分层关系

这篇文档不负责替代：

- 单条跨线程同步边的细节枚举
- 单个原子对象自己的修改账本
- `memory_order` 全部枚举值和选型说明

这些内容分别交给：

- [Synchronizes-With：并发图里一条正式成立的跨线程同步边](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md)
- [Modification Order：每个原子对象各有一条修改总序，不是全局时间线](/Users/maxwell/Knowledge/docs/computer-systems/modification-order.md)
- [Memory Order：从原子性到可见性与重排序控制](/Users/maxwell/Knowledge/docs/computer-systems/memory-order.md)

### 3.2 它不等于什么

`happens-before` 不等于：

- **源码里谁写在前面。**
  源码顺序只是可能的原材料，不是总图本身。

- **机器上谁先退休、谁先提交、谁先打印日志。**
  那是现实执行现象，不是标准关系图。

- **某个 atomic object 自己的历史。**
  那更接近 `modification order`，不是整张并发图。

- **单条 `synchronizes-with` 边。**
  后者只是大图里的跨线程边来源之一。

- **cache coherence。**
  coherence 解决的是单地址一致观察，不自动替你拼出跨对象可见性总图。

### 3.3 最容易混淆的相邻对象

| 对象 | 它是什么 | 和 `happens-before` 的关系 |
| --- | --- | --- |
| `sequenced-before` | 单线程内部的局部顺序边 | 是构造总图的重要原材料 |
| `synchronizes-with` | 具体、可命名的跨线程同步边 | 是让总图跨线程闭合的关键桥梁 |
| `modification order` | 单个 atomic object 的修改总序 | 是单对象历史，不是总图本体 |
| `memory order` | 代码里指定的顺序与同步合同 | 它会影响哪些跨线程边最终能成立 |
| `happens-before` | 由局部边、同步边和传递性拼成的总关系图 | 这是本文核心对象 |
| data race 判定 | 检查冲突访问有没有被总图隔开 | 是这张图最重要的输出之一 |

### 3.4 最关键的边界句

**真正重要的不是“谁先跑了”，而是“哪条边让另一个线程被标准要求必须把它当先发生”。**

## 4. 核心结构

### 4.1 把 `happens-before` 看成图，而不是一句话

要把 `happens-before` 变成一个可操作模型，至少要抓住下面六个构件。

| 构件 | 它是什么 | 为什么重要 |
| --- | --- | --- |
| 节点 | 评价、side effect、锁操作、等待动作、原子操作 | 你真正连接的是这些节点，不是“代码行” |
| 线程内边 | `sequenced-before` | 给总图提供局部顺序骨架 |
| 跨线程边 | `synchronizes-with` | 让图从一个线程跨到另一个线程 |
| 传递性 | 图的闭包规则 | 没有它，就拼不出完整可见性链 |
| 可见性落点 | 读取最终能把哪些写当成“已经发生” | 否则图只剩抽象结构 |
| 合法性落点 | 冲突访问是否被图隔开 | 否则不知道这张图为何直接关系到 data race |

### 4.2 边从哪里来

最稳的理解方式，是把 `happens-before` 的来源拆成三层：

| 层 | 典型来源 | 作用 |
| --- | --- | --- |
| 线程内局部边 | `sequenced-before` | 把同一线程里的普通读写、原子操作、锁操作串起来 |
| 跨线程同步边 | `synchronizes-with` | 让状态真正从一个线程合法传到另一个线程 |
| 传递边 | `hb` 的闭包 | 把“先准备数据，再发布；先接收，再消费”变成完整链路 |

这张表很重要，因为它提醒你：

- 你不是直接“得到一张 HB 图”
- 你是先有局部边、同步边，再做传递闭包

### 4.3 三类最常见的跨线程桥

对工程代码来说，最值得优先盯住的跨线程桥基本就是三类：

| 类别 | 例子 | 你该记住什么 |
| --- | --- | --- |
| 锁桥 | `unlock` -> 后续成功 `lock` | mutex 真正值钱的是建边，不只是挡人 |
| 原子发布桥 | `store(release)` -> 读到该值的 `load(acquire)` | 发布-接收是 HB 图里最常见的桥 |
| 等待桥 | condvar / `atomic_wait` 这类等待链 | 等待不是魔法广播，最终仍要落回共享状态和正式同步边 |

### 4.4 总图最容易被偷换的两个维度

很多错误都来自把下面两件事混了：

- **现实顺序**
  机器上谁先执行、谁先打印、谁先调度到

- **模型顺序**
  标准是否要求一个操作在另一个操作之前被观察到

`happens-before` 只关心第二件事。

### 4.5 单对象历史不是总图

`modification order` 给你的是某个 atomic object 自己的账本。  
`happens-before` 给你的是：

- 怎样把多个对象
- 普通内存
- 锁
- 等待
- 原子同步

拼到同一张可见性总图里。

最常见的误判就是：

- “这个 flag 的历史很清楚”
- 于是误以为“和它相关的 data 也就一起清楚了”

并不会。除非你真的把它们连进 HB 图。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 总主链：局部顺序 + 跨线程桥 + 传递闭包

`happens-before` 最值得记住的主链，不是某个单一 API，而是下面这个总过程：

1. 先把每个线程内部的 `sequenced-before` 画出来。
2. 再找真正成立的 `synchronizes-with` 跨线程边。
3. 用传递性把“准备数据 -> 发布 -> 接收 -> 消费”串成完整链。
4. 再用这张图判断：
   - 哪些写对哪些读必须可见
   - 哪些冲突访问已经被合法隔开
   - 哪些地方图断了，所以要么读到旧值，要么直接 data race

这条主链最重要，因为它说明：

**HB 不是单个语法点，而是你把整段并发协议拼成一张可见性图的过程。**

### 5.2 经典链路 1：mutex 把共享状态串起来

看一段最常见的互斥代码时，最稳的分析不是“加了锁所以安全”，而是：

1. 线程 A 在临界区里写共享状态。
2. A `unlock` 同一个 mutex。
3. 线程 B 随后成功 `lock` 同一个 mutex。
4. 这条 `unlock -> lock` 建立 `synchronizes-with`。
5. 再结合两边各自的 `sequenced-before`，A 解锁前的写就可以 `happens-before` B 加锁后的读。

mutex 在这里最值钱的不是“互斥”这两个字，而是：

**它稳定替你提供了一条跨线程桥。**

### 5.3 经典链路 2：release / acquire 不是在同步 flag，而是在同步它前后的历史

看一段发布-接收代码时，真正要画的不是“flag 先变了”，而是：

1. A 在线程内先写普通数据。
2. A 再对 carrier atomic object 做 release。
3. B acquire 并真的读到这次发布值。
4. 这条跨线程桥把 A release 之前那批普通写，传给了 B acquire 之后的读取。

这条链最值得记住的是：

**HB 图必须把普通写也串进去，否则你拿到的只是 flag 的历史，不是 payload 的可见性。**

### 5.4 经典链路 3：condition variable 不是广播“所有历史都过去了”

condvar 最容易被误解成：

- “我 notify 了，对方醒了，所以之前写的都过去了”

更稳的 HB 分析是：

1. 共享 predicate 在 mutex 保护下被改动。
2. 相关线程在同一把 mutex 和 wait 循环中等待条件成立。
3. 真正值钱的仍是 mutex 边和 predicate 所在共享状态的关系。
4. notify 只是让等待者有机会重新竞争并检查条件。

这条链的关键不在“醒没醒”，而在：

**HB 图有没有真的从 predicate 的写，穿过同步边，落到等待线程的后续读。**

### 5.5 经典链路 4：图断了之后会出现两类后果

HB 图一旦断开，通常不是只有一种结果，而是两种不同层次的后果：

- **图断了，但冲突访问还没越过合法边界**
  常见表现是读到旧值、观察次序不符合预期、逻辑协议失效。

- **图断了，而且冲突访问也没被合法隔开**
  这时就不只是“逻辑不对”，而是会直接落到 data race。

所以你在调并发 bug 时，先别一上来问“逻辑为什么错”，而要先问：

- 图是断在了哪里
- 断掉之后落向的是“旧值/错值”还是“直接非法”

## 6. 关键 tradeoff 与失败模式

### 6.1 这张图真正买到什么

HB 图的价值非常明确：

- 你终于有一套不依赖墙钟时间和平台偶然性的并发推理语言
- 你可以把“可见性”“data race”“同步成立没有”压回同一张图
- 代码审查时不必再停留在“感觉这里应该安全”

### 6.2 代价：你必须显式地找边

它的代价同样明确：

- 你不能再靠口头经验推理
- 你要真的标节点、找边、做闭包
- 你得忍住把单对象历史偷换成全局顺序的冲动

### 6.3 五类高频失败模式

**失败模式 1：时间线幻觉**

把“机器上先执行了”误当成“标准上必须先发生”。

**失败模式 2：把单对象账本误当总图**

看到某个 flag 的 `modification order` 很清楚，就误以为和它相关的全部状态都自动清楚。

**失败模式 3：把 API 名字误当成边**

看到 `notify`、`fence`、`atomic` 这些词，就误以为想要的桥已经成立。

**失败模式 4：只看关键字，不看 read-from 关系**

release/acquire 不只是写了关键字，还要看 acquire 到底读到了谁。

**失败模式 5：先谈逻辑，再谈 legality**

很多团队先讨论“业务为什么错”，最后才发现程序连最基本的 HB 图都没站稳。

## 7. 应用场景

### 7.1 代码审查

HB 图最直接的应用，就是把“这里安全吗”改写成：

- 节点有哪些
- 局部边有哪些
- 跨线程桥在哪里
- 传递链有没有真的闭合

### 7.2 并发 bug 排障

当你看到：

- 偶发旧值
- 条件偶尔不成立
- TSan 报 race
- 发布-接收代码偶尔失效

最稳的入口通常都不是日志顺序，而是先重建 HB 图。

### 7.3 原子协议设计

只要协议涉及：

- release/acquire
- RMW 接力
- condvar 或 `atomic_wait`
- 标志位与 payload 分离

就必须把它们贴回 HB 图里验证，而不能只看局部语法。

### 7.4 教学与术语收束

这张图也是理解这组并发术语最短的总入口。  
如果不把它建立起来，`atomicity`、`synchronizes-with`、`modification order`、`data race` 很容易一直像散词。

## 8. 工业 / 现实世界锚点

### 8.1 ISO C++ 内存模型文本

HB 不是社区口号，而是标准文本里真正用于裁决可见性与 data race 的关系框架。  
这是它最硬的锚点。

### 8.2 ThreadSanitizer

TSan 之类工具之所以有价值，本质上就是在近似重建和检查这张图。  
它不是“玄学猜 bug”，而是在近似你是否缺边。

### 8.3 Linux Kernel Memory Model

内核文档长期用更显式的关系图方式讨论并发。  
即使语言层不同，它也能作为“先画边、再看闭包”的现实锚点。

## 9. 当前推荐实践、过时路径与替代

### 9.1 当前更推荐的工作方式

更推荐的工作方式是：

- 先画 HB 图，再谈“结果应该是什么”
- 先把程序拉回有明确 HB 图支撑的区域，再谈弱序优化
- 默认优先使用更容易画清图的同步方式，例如 mutex、清晰的 release/acquire、必要时先用 `seq_cst`

### 9.2 高风险旧路径

高风险路径包括：

- 只按源码顺序或日志顺序推理
- 只看某个 atomic object 的历史，不看整张图
- 只看关键字，不看 read-from 是否真的成立
- 只说“加了 notify / fence / atomic”，却不给出桥的来源

### 9.3 替代与补位关系

HB 图不是替代所有术语，而是给它们定层级：

- 单对象访问单位，回到 [atomicity](/Users/maxwell/Knowledge/docs/computer-systems/atomicity.md)
- 单对象修改账本，回到 [modification-order](/Users/maxwell/Knowledge/docs/computer-systems/modification-order.md)
- 具体跨线程桥，回到 [synchronizes-with](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md)
- 顺序合同与选型，回到 [memory-order](/Users/maxwell/Knowledge/docs/computer-systems/memory-order.md)

## 10. 自测题 / 验证入口

如果你真的理解了这篇文档，至少应该能回答：

1. 为什么 `happens-before` 不是“现实时间里的先后”，而是“总关系图”？
2. 为什么 `synchronizes-with` 更像具体桥，而 `happens-before` 更像拼好的整张图？
3. 为什么某个 atomic object 的历史很清楚，不等于和它相关的普通数据也自动清楚？
4. 为什么并发代码排障时，先画 HB 图通常比先看日志顺序更稳？
5. 为什么 HB 图断开后，有时只是读到旧值，有时会直接落到 data race？

一个最实用的自测动作是：

拿下面三类代码各写一句 HB 判断：

- mutex 临界区
- release / acquire 发布-接收
- condvar 等待 predicate

如果你能稳定说出图里的桥从哪里来、普通数据是怎样被串起来的，这篇文档的核心目标就达到了。

## 11. 迁移与关联模型

理解了 `happens-before` 之后，最值得迁移出去的不是某个语法技巧，而是这组判断：

- 这里讨论的是单对象历史，还是整张可见性图？
- 这里缺的是局部边、跨线程桥，还是传递闭包？
- 这里失败的是“看到旧值”，还是“已经越过合法边界”？

最值得连着看的文档是：

- [Synchronizes-With：并发图里一条正式成立的跨线程同步边](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md)
- [Modification Order：每个原子对象各有一条修改总序，不是全局时间线](/Users/maxwell/Knowledge/docs/computer-systems/modification-order.md)
- [Data Race：冲突访问何时越过语言模型的合法边界](/Users/maxwell/Knowledge/docs/computer-systems/data-race.md)
- [Memory Order：从原子性到可见性与重排序控制](/Users/maxwell/Knowledge/docs/computer-systems/memory-order.md)

最值得保留的迁移句是：

**“并发里到底发生了什么”，最终要靠一张正式的关系图来回答，而不是靠时间直觉。**

## 12. 未解问题与继续深挖

- 如何把真实并发 bug 里的 `happens-before` 图进一步压缩成更短的排查步骤？
- `strongly happens-before` 在当前草案语境下还剩哪些值得单独展开的使用场景？
- sanitizer 对 `happens-before` 的近似建模与真实语言语义之间，哪些偏差最值得给工程师长期提醒？

## 13. 参考资料

- ISO C++ Working Draft, `intro.races`, `atomics.order`, `thread.mutex.requirements`, `thread.condition`
- LLVM ThreadSanitizer Documentation
- Linux Kernel Memory Model Explanations
- WG21 P3475R2
- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
