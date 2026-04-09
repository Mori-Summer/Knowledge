---
doc_id: computer-systems-synchronizes-with
title: Synchronizes-With：并发图里一条正式成立的跨线程同步边
concept: synchronizes_with
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-04-09T11:20:00+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_atomics_fences_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
  - pure_concept_doc_split_review_2026_04_09
  - methodology_document_generation_methodology
time_context: current_cxx_memory_model_boundary_checked_2026_03_16
applicability: concept_boundary_discrimination_for_cross_thread_sync_edges_happens_before_graphing_and_code_review
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/release-sequence.md
  - docs/computer-systems/mutex.md
  - docs/computer-systems/fence.md
  - docs/computer-systems/atomic-wait-notify.md
open_questions:
  - 如何把 `synchronizes-with` 的常见来源压缩成一张更短的代码审查清单？
  - 在 `atomic_wait` / `notify`、fence 与库同步原语混用时，哪些边最容易被误判为“已经成立”？
  - 如果未来工具链更直接展示同步图，哪些非直觉边最值得优先可视化？
---

# Synchronizes-With：并发图里一条正式成立的跨线程同步边

## 1. 这份文档要帮你学会什么

这篇文档的重点，不是继续把 `synchronizes-with` 写成一篇机制总览，而是先把这个概念本体讲清。

读完后，你应该至少能做到：

- 说清 `synchronizes-with` 为什么首先是一条“正式成立的跨线程边”，而不是整张并发关系图
- 区分 `sequenced-before`、`synchronizes-with`、`happens-before`、`modification order` 这几层对象
- 判断某段代码里到底有没有那条被标准承认的同步边
- 不再把“大家都碰了同一个 atomic”“写了 release/acquire 关键字”“我发了 notify”误当成边已经成立
- 知道这篇文档停在概念边界，完整推理图要去接更大的 `happens-before` 模型

## 2. 一句话结论 / 概念结论

**`synchronizes-with` 首先不是“同步已经完成”的口头描述，而是并发模型里一条可命名、可检查、可跨线程成立的正式同步边；很多你最终依赖的 `happens-before`，都要先靠它落地。**

如果只记一句最稳的话：

**讨论并发安全时，先不要说“这里大概同步了”，而要先问“这条跨线程边到底算不算 `synchronizes-with`”。**

## 3. 这个概念试图解决什么命名或分类问题

并发讨论里最常见的漂移之一，就是把下面几件事混成一团：

- 线程内语句先后
- 跨线程的一条正式同步边
- 由很多边拼出来的整张 `happens-before` 图
- “我感觉它应该已经同步”的经验说法

如果没有 `synchronizes-with` 这个概念，下面这些说法会一直混不清：

- “这里有 release/acquire，所以安全”
- “它们都访问了同一个原子变量，所以已经连上了”
- “我先 notify 了，对方醒了，所以之前写的东西一定都过去了”
- “我加了一条 fence，所以边应该自己长出来了”

所以，这个概念首先解决的不是“怎样证明整个协议”，而是更基础的一层分类问题：

- 哪些对象只是线程内顺序
- 哪些对象才是标准承认的跨线程同步边
- 哪些对象只是以后可能被用来构造边的上下文

## 4. 概念边界与相邻概念

### 4.1 这篇文档直接处理什么

这篇文档直接处理的是：

- `synchronizes-with` 作为跨线程正式同步边的本体
- 最常见的原子边与 mutex 边
- 这条边和相邻概念的边界
- 何时该说“边成立了”，何时只能说“看起来像同步”

这篇文档不负责系统展开：

- `happens-before` 如何把多条边拼成整张关系图
- `release sequence` 如何作为中间桥梁接力
- fence、condvar、`atomic_wait` 的全部细则

这些内容分别交给：

- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)
- [Release Sequence：同一原子对象上延续发布语义的特殊修改链](/Users/maxwell/Knowledge/docs/computer-systems/release-sequence.md)
- [Fence：什么时候需要一堵顺序墙，什么时候这堵墙其实并不会替你同步](/Users/maxwell/Knowledge/docs/computer-systems/fence.md)

### 4.2 它不等于什么

`synchronizes-with` 不等于：

- **`happens-before` 整张图。**
  它更像其中一条具体边，不是最后的大图结果。

- **线程内源码顺序。**
  线程内先后更接近 `sequenced-before`，不是跨线程边。

- **任意“都访问了同一个 atomic”的情况。**
  同对象访问不自动等于边成立。

- **任何“通知了别人”的动作。**
  `notify`、wake、signal 之类动作不自动等于你想要的那条历史传递边。

- **墙钟时间先后。**
  现实里谁先跑完，不等于标准上谁被另一线程必须当作“已经发生”。

### 4.3 最容易混淆的相邻对象

| 对象 | 它是什么 | 和 `synchronizes-with` 的关系 |
| --- | --- | --- |
| `sequenced-before` | 单线程内部的求值先后关系 | 常是构造大图的原材料，但不是跨线程边 |
| `synchronizes-with` | 一条正式成立的跨线程同步边 | 这是本文核心对象 |
| `happens-before` | 由线程内顺序、同步边和传递性拼成的更大关系图 | `synchronizes-with` 是它的重要边来源 |
| `modification order` | 单个 atomic object 的修改总序 | 它不是跨线程边，但常是边成立的载体背景 |
| `release sequence` | 某条 release 头部往后的特殊修改链 | 它会帮助某些同步边成立，但本身不是边 |
| “同一个 atomic” | 同一对象被多线程访问 | 这只是上下文，不自动给出同步边 |

### 4.4 最关键的边界句

**看到 “synchronizes-with” 时，先追问“它是不是一条已经满足标准匹配条件的具体跨线程边”，而不是先把它想成“差不多同步了”的模糊判断。**

## 5. 什么算它，什么不算它

下面这张表是最实用的判别模板。

| 情形 | 在本文里怎么处理 | 为什么 |
| --- | --- | --- |
| `store(release)` 与读到该值的 `load(acquire)` | 算 `synchronizes-with` | acquire 真的接住了对应 side effect |
| 头部 release 经由 release sequence，最终被 acquire 读到链上值 | 算 `synchronizes-with` | acquire 可以同步回头部 release |
| 对同一 mutex 的 prior `unlock()` 与后续成功 `lock()` | 算 `synchronizes-with` | 这是标准承认的典型 mutex 边 |
| 同一个 atomic 上的 relaxed store / relaxed load | 不算 | 有原子性，不自动有你想要的同步边 |
| acquire load 没读到对应 release 链上的值 | 不算 | 关键条件没满足 |
| 单独一条 fence，没有 carrier atomic object 把两边连起来 | 不算 | fence 不能无中生有地立边 |
| “我先 notify，对方后面醒了” | 不能直接算 | 醒来不等于那条历史传递边已经成立 |

最值得反复提醒的一点是：

**判断 `synchronizes-with` 时，不能只看 API 名字或 memory order 关键字，还要看匹配条件是否真的成立。**

## 6. 代表性例子、反例与常见误读

### 6.1 代表性例子

最有代表性的例子有三个。

- **release store -> acquire load，且 acquire 读到了对应值。**
  这是最标准的原子同步边。

- **头部 release -> 中继 RMW -> acquire 读到链尾值。**
  这说明边有时要借 [release sequence](/Users/maxwell/Knowledge/docs/computer-systems/release-sequence.md) 才落地。

- **prior `unlock()` -> later successful `lock()`。**
  这说明 mutex 最值钱的不只是互斥，而是它稳定地产生正式同步边。

### 6.2 反例

最值得记住的反例有下面这些。

- **双方都用了同一个 atomic，但都是 relaxed。**
  这不自动构成 `synchronizes-with`。

- **写了 release，读也写成 acquire，但 acquire 读到的是别的值。**
  关键匹配条件没满足，边不能硬说成立。

- **只写了 fence。**
  fence 仍需要 carrier atomic object 和读值关系配合，不能独自立边。

- **“对方醒了”。**
  醒来只是执行机会发生了变化，不自动说明之前的共享状态历史已经安全传过去。

### 6.3 常见误读

最常见的误读有下面几类：

- **“release + acquire 关键字一出现，边就自动有了。”**
  错。还要看 acquire 到底读到了谁。

- **“同一个 atomic object 就天然同步。”**
  错。原子性和同步边是两层。

- **“notify 就是同步完成通知。”**
  不稳。更稳的是把 notify 看成唤醒机会，而不是全部可见性证明。

- **“`synchronizes-with` 就是 `happens-before`。”**
  错。前者是具体边，后者是更大的关系图。

- **“fence 比对象语义更高，所以它能直接替你把边立出来。”**
  错。fence 仍要借具体对象和匹配条件落地。

## 7. 它会进入哪些更大的模型或判断框架

`synchronizes-with` 这个概念本体，最重要的下游去向有四个。

### 7.1 `happens-before` 图

`synchronizes-with` 最重要的下游，就是进入 `happens-before` 大图。  
没有这条边，很多跨线程可见性链根本拼不起来。

### 7.2 data race 判定

很多“有没有 data race”的判断，最后都要回到：

- 有没有足够的跨线程边
- 这些边能不能通过传递性把冲突访问隔开

### 7.3 release/acquire、release sequence 与 fence 推理

这些机制之所以难，不是因为术语多，而是因为它们都在争夺同一件事：  
怎样让一条正式同步边成立。

### 7.4 wait/notify 类原语的正确理解

理解了 `synchronizes-with` 之后，就更不容易把：

- 唤醒
- 获得执行机会
- 共享状态历史真正被传过去

这三层混在一起。

## 8. 自测题 / 验证入口

如果你真的理解了这篇文档，至少应该能回答：

1. 为什么 `synchronizes-with` 应该被理解成“一条边”，而不是“整张图”？
2. 为什么同一个 atomic object 上的两次访问不自动说明边成立？
3. 为什么 release/acquire 的关键不只是关键字，而是 acquire 是否读到了对应 side effect？
4. 为什么 mutex 的核心价值之一是“建边”，而不只是“挡别人一下”？
5. 为什么 notify 或 fence 不能靠名字就直接被当成同步证明？

一个最实用的自测动作是：

拿下面四种情形各写一句判断：

- release store / acquire load 读到同一值
- relaxed store / relaxed load
- prior unlock / later successful lock
- notify 之后 waiter 醒来

如果你能稳定分清哪些真的构成正式同步边，哪些只是看起来像同步，这篇文档的核心目标就达到了。

## 9. 迁移与关联模型

理解了 `synchronizes-with` 之后，最值得迁移出去的不是某条语法规则，而是这组判断：

- 这是线程内顺序，还是跨线程正式边？
- 这是边本体，还是边成立所依赖的中间结构？
- 这里真正被证明的是“历史传递”，还是只是“执行机会变化”？

最值得连着看的文档是：

- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)
- [Release Sequence：同一原子对象上延续发布语义的特殊修改链](/Users/maxwell/Knowledge/docs/computer-systems/release-sequence.md)
- [Modification Order：每个原子对象各有一条修改总序，不是全局时间线](/Users/maxwell/Knowledge/docs/computer-systems/modification-order.md)

最值得保留的迁移句是：

**“同步”不是口头感觉；在标准模型里，你最终总要落回某条具体、可检查、可命名的跨线程边。**

## 10. 未解问题与继续深挖

- 如何把 `synchronizes-with` 的常见来源压缩成一张更短的代码审查清单？
- 在 `atomic_wait` / `notify`、fence 与库同步原语混用时，哪些边最容易被误判为“已经成立”？
- 如果未来工具链更直接展示同步图，哪些非直觉边最值得优先可视化？

## 11. 参考资料

- ISO C++ Working Draft, `intro.races`, `atomics.order`, `atomics.fences`, `thread.mutex.requirements`
- WG21 P3475R2
- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
