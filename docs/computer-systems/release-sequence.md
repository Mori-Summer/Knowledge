---
doc_id: computer-systems-release-sequence
title: Release Sequence：同一原子对象上延续发布语义的特殊修改链
concept: release_sequence
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:16:47+08:00'
updated_at: '2026-04-09T11:20:00+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_diff_cpp17_2026_03_16
  - wg21_p3475r2_2025
  - pure_concept_doc_split_review_2026_04_09
  - methodology_document_generation_methodology
time_context: current_cxx_memory_model_boundary_checked_2026_03_16
applicability: concept_boundary_discrimination_for_release_sequence_atomic_state_words_and_sync_chain_review
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/computer-systems/modification-order.md
  - docs/computer-systems/synchronizes-with.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/memory-order.md
open_questions:
  - 如何把 `release sequence` 与 fence、`atomic_wait` / `notify`、mixed-order litmus tests 压缩成统一判断模板？
  - 在真实 lock-free 协议里，哪些状态字最容易被误写成会断链的普通 store？
  - 如果未来标准继续收束 `consume` 相关历史包袱，`release sequence` 在教学和工具提示里应如何被更早暴露？
---

# Release Sequence：同一原子对象上延续发布语义的特殊修改链

## 1. 这份文档要帮你学会什么

这篇文档的重点，不是继续把 `release sequence` 写成一篇机制推导演练，而是先把这个概念本体讲清。

读完后，你应该至少能做到：

- 说清 `release sequence` 为什么首先是一段“特殊修改链”，而不是整个同步图
- 区分 release 头部、carrier atomic object、后继 RMW、最终 acquire 着陆这几个层次
- 判断一段状态字推进代码里，到底有没有形成 `release sequence`
- 不再把“同一个原子对象上的后继写”误当成都会自动续上这条链
- 知道这篇文档停在概念边界，最终跨线程同步边要去接 `synchronizes-with`

## 2. 一句话结论 / 概念结论

**`release sequence` 首先不是一条同步边，而是同一原子对象的 `modification order` 上，从某个 release 头部出发、沿连续成功 RMW 延续下去的一段特殊修改链；如果后面的 acquire 读到了这条链上的值，它就可能同步回头部。**

如果只记一句最稳的话：

**讨论 `release sequence` 时，先不要问“最终读到了哪个状态”，而要先问“这个值是不是还落在同一对象、同一账本、同一条连续 RMW 链上”。**

## 3. 这个概念试图解决什么命名或分类问题

在 lock-free 状态机、CAS 状态推进、引用计数和阶段字协议里，最容易混在一起的是下面几件事：

- 头部 release
- 后续状态推进
- acquire 最终读到的值
- 真正跨线程成立的同步边

如果没有 `release sequence` 这个概念，下面这些说法会一直漂：

- “最终读到的不是头部 release 写的值，所以应该同步不到头部”
- “同一个状态字后来又被别人改了，所以原先发布的历史已经断了”
- “后面只要还有写这个对象，就都算发布语义继续往前走”
- “release/acquire 边直接连就行，不用单独关心中间链条”

所以，这个概念首先解决的不是“怎样完整证明协议”，而是更细的一层命名问题：

- 头部 release 和尾部 acquire 之间，那段特殊的中间链条到底叫什么
- 哪些后继修改还在链上，哪些已经把链剪断
- 最终 acquire 究竟是在同步回头部，还是只是在读某个后来的状态

## 4. 概念边界与相邻概念

### 4.1 这篇文档直接处理什么

这篇文档直接处理的是：

- `release sequence` 作为特殊修改链的本体
- 头部 release
- 同一 carrier atomic object
- 连续成功 RMW 的延续条件
- acquire 最终如何“着陆”到链上

这篇文档不负责系统展开：

- 最终的跨线程正式边如何命名
- 整张 `happens-before` 图如何闭合
- 多对象协议的全部设计

这些内容分别交给：

- [Synchronizes-With：并发图里一条正式成立的跨线程同步边](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)

### 4.2 它不等于什么

`release sequence` 不等于：

- **最终那条 `synchronizes-with` 边。**
  它是某些同步边成立时的中间桥梁，不是最后那条边本身。

- **整个 `modification order`。**
  它只是单对象账本里的一段特殊子链，不是整本账。

- **同一对象上的任意后继写。**
  普通 store 不会自动续上这条链。

- **任意 acquire / release 关键字的并列出现。**
  关键不只在关键字，还在链条是否真的成立、最终 acquire 是否读到了链上值。

- **跨多个 atomic object 的同步传播。**
  一旦换对象，这条链就结束。

### 4.3 最容易混淆的相邻对象

| 对象 | 它是什么 | 和 `release sequence` 的关系 |
| --- | --- | --- |
| 头部 release | 链的起点 | 没有头部 release，就没有这条链 |
| `modification order` | 单个 atomic object 的全部修改总序 | `release sequence` 是其中一段特殊连续区间 |
| 成功 RMW | 会在链上继续接力的后继修改 | 它们是延续条件的重要组成 |
| 普通 atomic store | 也是修改，但不是链上自动延续者 | 它常常是断链点 |
| acquire 读到链上值 | 最终的着陆动作 | 它让同步得以回到头部 |
| `synchronizes-with` | 最终正式成立的跨线程边 | `release sequence` 帮它搭桥，但不等于它 |

### 4.4 最关键的边界句

**看到 “release sequence” 时，先追问“它是不是同一原子对象的连续成功 RMW 链”，而不要先把“后续任何写状态字”都想成链还在继续。**

## 5. 什么算它，什么不算它

下面这张表是最实用的判别模板。

| 情形 | 在本文里怎么处理 | 为什么 |
| --- | --- | --- |
| `store(memory_order_release)` 作为头部 | 算链头 | 它提供发布起点 |
| 头部之后，对同一 atomic object 的连续成功 RMW | 继续留在链上 | 这是本文采用的当前工作基线 |
| 头部之后的普通 atomic store | 不自动留在链上 | 它会把这条特殊链剪断 |
| 失败的 CAS | 不算链上的新修改点 | 失败 CAS 没有写 side effect |
| 换到另一个 atomic object 做状态推进 | 不算 | 这条链从头到尾只沿一个对象前进 |
| acquire 最终读到链上某个值 | 能借此同步回头部 | 这是这条链真正有价值的着陆方式 |
| acquire 没读到链上值 | 不能借这条链自证同步 | 着陆条件没满足 |

最值得反复提醒的一点是：

**能否延续 `release sequence`，关键看“是不是同一对象上的连续成功 RMW”，而不是看代码看起来是否仍在推进同一个状态概念。**

## 6. 代表性例子、反例与常见误读

### 6.1 代表性例子

最有代表性的例子有三个。

- **头部 release 后，别人用 `fetch_add` 把状态从 `1` 推到 `2`，最后 acquire 读到 `2`。**
  这就是最典型的“尾部读值仍能同步回头部”的场景。

- **头部 release 后，别人用成功 CAS 把 `ready` 推成 `claimed`。**
  这说明链条可以由成功 RMW 接力，不要求最终 acquire 直接看见头部原值。

- **状态字长期被不同线程接力推进，但始终没换 carrier object。**
  这说明这个概念首先服务于“同一状态字的接力式发布”。

### 6.2 反例

最值得记住的反例有下面这些。

- **头部 release 后，后续只是普通 store。**
  这会断链，最终 acquire 不能再仅凭 `release sequence` 同步回头部。

- **CAS 执行了，但失败了。**
  失败 CAS 没有写 side effect，不是链上的修改点。

- **后面推进的是另一个 atomic object。**
  这不叫“同一条链传过去了”，而是对象已经换了。

- **最终 acquire 没读到链上的值。**
  就算链在那儿，也没有被这个 acquire 接住。

### 6.3 常见误读

最常见的误读有下面几类：

- **“只要后面还在写同一个状态概念，链就还在。”**
  错。链看的是同一 atomic object 与特定修改类型，不看你口头上的“状态概念”。

- **“后继普通 store 也能帮忙把发布语义往前带。”**
  错。普通 store 正是最容易被忽略的断链点之一。

- **“尾部 RMW 必须自己也写成 release，才能留在链上。”**
  不稳。留在链上的最低条件不是“尾部也带 release”，而是“它仍是链上连续成功 RMW”；但工程上尾部是否要更强语义，还要看别的职责。

- **“最终 acquire 读到的是后人写的值，所以最多只和后人有关。”**
  不稳。若那个值仍在链上，它可以同步回头部 release。

- **“`release sequence` 就等于同步已经证明完了。”**
  错。它只是中间桥梁，最后还要落到正式同步边和更大的 `happens-before` 图。

## 7. 它会进入哪些更大的模型或判断框架

`release sequence` 这个概念本体，最重要的下游去向有四个。

### 7.1 `synchronizes-with` 边

这是它最直接的下游。  
`release sequence` 本身不是边，但它会帮助某些 acquire 合法同步回头部 release。

### 7.2 lock-free 状态字与 CAS 协议

只要协议依赖“头部发布一次，后面由别人继续接力推进状态字”，就很容易进入 `release sequence` 的判断框架。

### 7.3 `modification order` 账本思维

理解了 `release sequence` 之后，就更容易看清：

- 它为什么只沿一个对象前进
- 它为什么要贴着单对象账本判断
- 它为什么不能被误读成全局传播链

### 7.4 fence、`atomic_wait` / `notify` 等更复杂边界

这些主题更难的地方，不是 API 数量，而是它们经常会和 `release sequence` 混在一起。  
先把这个概念本体分清，后面才不容易漂。

## 8. 自测题 / 验证入口

如果你真的理解了这篇文档，至少应该能回答：

1. 为什么 `release sequence` 不应该被理解成“整条同步边”？
2. 为什么它必须始终贴着同一个 atomic object 的 `modification order` 判断？
3. 为什么普通 store 和失败 CAS 都是高风险断链点？
4. 为什么最终 acquire 读到链尾值时，仍可能同步回头部 release？
5. 为什么“后继 RMW 不必自己也带 release 才能留在链上”和“工程上尾部可一律 relaxed”不是同一结论？

一个最实用的自测动作是：

拿下面四种后继修改各写一句判断：

- 成功 `fetch_add`
- 成功 CAS
- 普通 store
- 失败 CAS

如果你能稳定分清哪些还在链上、哪些已经断链，这篇文档的核心目标就达到了。

## 9. 迁移与关联模型

理解了 `release sequence` 之后，最值得迁移出去的不是某条 litmus test，而是这组判断：

- 这是链头、链身，还是链尾着陆？
- 这是同一对象上的连续成功 RMW，还是已经断链？
- 这里需要的是“中间桥梁”判断，还是最终同步边判断？

最值得连着看的文档是：

- [Modification Order：每个原子对象各有一条修改总序，不是全局时间线](/Users/maxwell/Knowledge/docs/computer-systems/modification-order.md)
- [Synchronizes-With：并发图里一条正式成立的跨线程同步边](/Users/maxwell/Knowledge/docs/computer-systems/synchronizes-with.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)

最值得保留的迁移句是：

**“release 之后还有人继续推进状态”这件事，本身不自动成立为同步；你最终总要回到‘链有没有断、值是不是落在链上’来判断。**

## 10. 未解问题与继续深挖

- 如何把 `release sequence` 与 fence、`atomic_wait` / `notify`、mixed-order litmus tests 压缩成统一判断模板？
- 在真实 lock-free 协议里，哪些状态字最容易被误写成会断链的普通 store？
- 如果未来标准继续收束 `consume` 相关历史包袱，`release sequence` 在教学和工具提示里应如何被更早暴露？

## 11. 参考资料

- ISO C++ Working Draft, `atomics.order`, `intro.races`
- WG21 P3475R2
- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
