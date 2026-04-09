---
doc_id: computer-systems-modification-order
title: Modification Order：每个原子对象各有一条修改总序，不是全局时间线
concept: modification_order
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:56:22+08:00'
updated_at: '2026-04-09T11:20:00+08:00'
source_basis:
  - cxx_draft_basic_exec_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_intro_races_2026_03_16
  - pure_concept_doc_split_review_2026_04_09
  - methodology_document_generation_methodology
time_context: current_cxx_memory_model_boundary_checked_2026_03_16
applicability: concept_boundary_discrimination_for_single_atomic_histories_read_from_judgment_and_release_sequence_entry
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/computer-systems/release-sequence.md
  - docs/computer-systems/synchronizes-with.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/atomic-wait-notify.md
open_questions:
  - 如何把 `modification order`、读值候选和 coherence 约束压成更短的推理模板？
  - 在 mixed-order litmus tests 里，哪些最反直觉的现象最值得单独整理成案例库？
  - 如何把 per-object 修改总序与硬件 coherence 的“相像但不等价”关系讲得更不容易误导？
---

# Modification Order：每个原子对象各有一条修改总序，不是全局时间线

## 1. 这份文档要帮你学会什么

这篇文档的重点，不是继续把 `modification order` 写成一篇机制推导，而是先把这个概念本体讲清。

读完后，你应该至少能做到：

- 说清 `modification order` 为什么首先是一份“单对象修改账本”，而不是全局时间线
- 区分“哪些修改点会进账本”“哪些访问根本不在账本里”
- 不再把多个 atomic object 的历史偷偷拼成一条共同时间线
- 看懂为什么 `release sequence`、读值来源、`atomic_wait` 都要先站在这份账本之上
- 知道这篇文档停在概念边界，跨对象联系要去接 `synchronizes-with` 和 `happens-before`

## 2. 一句话结论 / 概念结论

**`modification order` 首先不是程序的全局时间线，而是某一个 atomic object 上全部修改组成的单独总序；它给这个对象一份一致可讨论的写入历史，但不会自动把别的对象并进来。**

如果只记一句最稳的话：

**讨论 `modification order` 时，先不要说“程序里谁更晚”，而要先说“这是哪个 atomic object 自己的账本”。**

## 3. 这个概念试图解决什么命名或分类问题

并发原子推理里，最容易混在一起的是下面几件事：

- 某个 atomic object 自己的修改历史
- load 最终读到了哪一次写
- 多个 atomic object 之间的关系
- 更大的 `happens-before` 图

如果没有 `modification order` 这个概念，下面这些说法会一直混不清：

- “原子变量都有顺序，所以整体应该也有顺序”
- “这个 load 肯定读最近那次写”
- “既然 `y` 已经看到新值，`x` 也应该早就更新完了”
- “release sequence 只是看源码上谁写在后面”

所以，这个概念首先解决的不是“怎样完整分析所有并发协议”，而是更基础的一层对象分类：

- 哪些历史只属于单个 atomic object
- 哪些历史是以后才会被别的关系图连起来的
- 读值分析到底是先贴账本，还是先谈同步

## 4. 概念边界与相邻概念

### 4.1 这篇文档直接处理什么

这篇文档直接处理的是：

- `modification order` 作为单对象修改总序的本体
- 哪些修改会出现在这条总序里
- 哪些访问不属于这条总序
- 单对象账本和相邻概念的边界

这篇文档不负责系统展开：

- 多对象同步如何闭合成整张图
- 最终跨线程正式边如何成立
- `seq_cst` 的更强全局排序网

这些内容分别交给：

- [Synchronizes-With：并发图里一条正式成立的跨线程同步边](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)

### 4.2 它不等于什么

`modification order` 不等于：

- **全程序统一时间线。**
  它只属于某一个 atomic object。

- **`happens-before`。**
  前者是单对象写历史，后者是更大的关系图。

- **`synchronizes-with`。**
  前者不是跨线程边，只是某些边判断时的账本背景。

- **`seq_cst` 的全局顺序网。**
  `modification order` 是 per-object，`seq_cst` 讨论的是更强的跨对象排序约束。

- **全部访问日志。**
  这份账本记录的是修改，不是所有 load / store / fence 事件。

### 4.3 最容易混淆的相邻对象

| 对象 | 它是什么 | 和 `modification order` 的关系 |
| --- | --- | --- |
| atomic store | 会写出新的 side effect | 会在账本里留下新的修改点 |
| 成功 RMW | 既读旧值又写新值的修改 | 也会在账本里留下新的修改点 |
| load | 读取已有历史 | 不会作为新修改点进入账本 |
| 失败 CAS | 只走读路径，没有新 side effect | 不会在账本里新增修改点 |
| `release sequence` | 从某个 release 头部开始的一段特殊链 | 它贴着这份单对象账本往前切片 |
| `happens-before` | 更大的可见性关系图 | 它会约束读值候选，但不等于账本本体 |
| 另一个 atomic object 的历史 | 另一份账本 | 不会自动和这份账本并成一条线 |

### 4.4 最关键的边界句

**看到 “modification order” 时，先追问“这是哪个 atomic object 自己的修改账本”，而不要立刻把它听成“全程序已经有了一条统一时间线”。**

## 5. 什么算它，什么不算它

下面这张表是最实用的判别模板。

| 情形 | 在本文里怎么处理 | 为什么 |
| --- | --- | --- |
| 初始值写入 | 算账本里的起点 | 单对象历史总要有起点 |
| atomic store | 算新的修改点 | 它产生新的 side effect |
| 成功 `fetch_add` / 成功 CAS | 算新的修改点 | 成功 RMW 也会写出新的 side effect |
| atomic load | 不算修改点 | 它只读，不写 |
| 失败 CAS | 不算新的修改点 | 失败时没有新的写 side effect |
| 另一个 atomic object 上的修改 | 不算这本账的一部分 | 每个对象各有自己的账本 |
| 普通非原子对象的写 | 不算这本账的一部分 | 本文讨论的是 atomic object 的修改总序 |

最值得反复提醒的一点是：

**`modification order` 是“写账本”，不是“所有事件日志”；load 会参考它，但不进入它。**

## 6. 代表性例子、反例与常见误读

### 6.1 代表性例子

最有代表性的例子有三个。

- **同一个 `x` 上的两次 atomic store。**
  不管它们来自哪个线程，都必须能进入 `x` 自己那条总序。

- **同一个 `x` 上的一次成功 RMW。**
  它说明账本里会出现“既读取旧历史、又写出新修改点”的节点。

- **`release sequence` 贴着同一对象往后切一段连续链。**
  这说明很多更高层机制都站在单对象账本之上。

### 6.2 反例

最值得记住的反例有下面这些。

- **把 `x` 和 `y` 的修改排成一条共同线。**
  这正是最常见的全局时钟幻觉。

- **把 load 当成账本里的新修改点。**
  load 会参考账本，但不会新增账本条目。

- **把失败 CAS 当成“也参与写历史”。**
  失败 CAS 没有写 side effect，不该被记进这本修改账。

- **看见 `y` 读到新值，就推断 `x` 也应该已经是新值。**
  这是把两本账偷偷并成一本的典型错误。

### 6.3 常见误读

最常见的误读有下面几类：

- **“原子变量都有顺序，所以整个程序就有全局顺序。”**
  错。每个对象各有一条总序，不等于世界只有一条线。

- **“load 也属于 `modification order`。”**
  错。它参考账本，不新增账本。

- **“读到了一个较晚值，别的对象上的写也就跟着较晚了。”**
  错。那是把单对象账本偷换成跨对象时间线。

- **“失败 CAS 也算在账本里，因为它确实执行过。”**
  错。执行过不等于产生了新的写 side effect。

- **“`modification order` 和硬件 coherence 就是一回事。”**
  不稳。它们相像，但抽象边界和语言层约束并不完全相同。

## 7. 它会进入哪些更大的模型或判断框架

`modification order` 这个概念本体，最重要的下游去向有四个。

### 7.1 读值来源判断

load 到底能读到哪次写，首先就要回到这份单对象账本上谈候选来源。

### 7.2 `release sequence`

这是最直接的下游之一。  
`release sequence` 本质上就是在这份账本里切一段特殊连续链。

### 7.3 `atomic_wait` / `notify`

等待者到底是不是看到了“更晚的一笔写”，本质上也要回到同一对象账本有没有向后翻页。

### 7.4 coherence 约束与更高层同步图

单对象账本给出“候选历史轴”，更高层关系再去裁剪不合法读值、连接跨对象历史。  
这两层不分开，后面很容易漂。

## 8. 自测题 / 验证入口

如果你真的理解了这篇文档，至少应该能回答：

1. 为什么 `modification order` 应该被理解成“单对象账本”，而不是“全局时间线”？
2. 为什么 load 会参考这份账本，却不作为新修改点进入账本？
3. 为什么失败 CAS 不该被记成账本里的新条目？
4. 为什么 `x` 的账本和 `y` 的账本不能自动拼成一条共同历史？
5. 为什么很多更高层概念都必须先站在这份账本上？

一个最实用的自测动作是：

拿下面五种事件各写一句判断：

- `x.store(...)`
- `x.fetch_add(...)`
- `x.load(...)`
- 失败 CAS on `x`
- `y.store(...)`

如果你能稳定分清哪些进入 `x` 的账本、哪些只是参考或属于别的账本，这篇文档的核心目标就达到了。

## 9. 迁移与关联模型

理解了 `modification order` 之后，最值得迁移出去的不是某条 litmus test，而是这组判断：

- 这是哪一个 atomic object 的账本？
- 这是账本里的修改点，还是账本之外的读取动作？
- 这里需要的是“单对象历史”判断，还是“跨对象同步”判断？

最值得连着看的文档是：

- [Release Sequence：同一原子对象上延续发布语义的特殊修改链](/Users/maxwell/Knowledge/docs/computer-systems/release-sequence.md)
- [Synchronizes-With：并发图里一条正式成立的跨线程同步边](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)

最值得保留的迁移句是：

**“有顺序”从来都要先问“是哪一个对象的顺序”；不先守住对象粒度，后面的并发推理几乎一定会偷换。**

## 10. 未解问题与继续深挖

- 如何把 `modification order`、读值候选和 coherence 约束压成更短的推理模板？
- 在 mixed-order litmus tests 里，哪些最反直觉的现象最值得单独整理成案例库？
- 如何把 per-object 修改总序与硬件 coherence 的“相像但不等价”关系讲得更不容易误导？

## 11. 参考资料

- ISO C++ Working Draft, `basic.exec`, `atomics.order`, `intro.races`
- LLVM Atomics Guide
- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
