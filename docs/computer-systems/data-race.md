---
doc_id: computer-systems-data-race
title: Data Race：冲突访问何时越过语言模型的合法边界
concept: data_race
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-04-09T11:50:00+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - clang_threadsanitizer_docs_2026_03_16
  - pure_concept_doc_split_review_2026_04_09
  - methodology_document_generation_methodology
time_context: current_cxx_memory_model_boundary_checked_2026_03_19
applicability: concept_boundary_discrimination_for_concurrent_legality_happens_before_review_and_drf_entry
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/computer-systems/atomicity.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/mutex.md
  - docs/computer-systems/memory-order.md
open_questions:
  - race detector 在真实大规模项目里如何近似 `happens-before`，以及哪些场景最容易漏报或误报？
  - DRF 和更高层逻辑正确性之间，能否再压缩出一条更短的代码审查链？
  - 在异构系统、设备共享内存和弱一致平台上，data race 边界还会遇到哪些额外工程问题？
---

# Data Race：冲突访问何时越过语言模型的合法边界

## 1. 这份文档要帮你学会什么

这篇文档的重点，不是继续把 data race 写成一篇并发排障教程，而是先把这个概念本体讲清。

读完后，你应该至少能做到：

- 说清 data race 为什么首先是“语言层合法性边界”，而不是泛泛的“竞态”坏词
- 区分 data race、race condition、死锁、性能竞争这几类不同问题
- 判断某段代码里的共享访问到底还在语言合同内，还是已经越界
- 不再把“用了点 atomic”“机器上暂时没复现”“这是个逻辑 bug”误当成 data race 判定
- 知道这篇文档停在概念边界，更大的同步证明要去接 `happens-before`

## 2. 一句话结论 / 概念结论

**data race 首先不是“结果不稳定”这么简单，而是语言内存模型划出的硬边界：两个潜在并发的冲突访问里，只要至少有一个是非原子访问，并且它们之间没有 `happens-before`，程序就已经越过语言合同，落入未定义行为。**

如果只记一句最稳的话：

**讨论 data race 时，先不要问“这次到底读到 0 还是 1”，而要先问“这两次冲突访问之间到底有没有合法边把它们隔开”。**

## 3. 这个概念试图解决什么命名或分类问题

并发讨论里，最容易混在一起的是下面几件事：

- 逻辑结果受调度影响
- 共享访问在语言层是否合法
- 程序会不会死锁
- 程序是不是只是性能差

如果没有 data race 这个概念，下面这些说法会一直混不清：

- “这是个 race，所以可能就是 data race”
- “程序只是偶尔错一下，不一定算那么严重”
- “我这里加了一个 atomic flag，所以整段共享状态应该没 race 了”
- “机器上没出事，所以这里大概不是 data race”

所以，这个概念首先解决的不是“怎样把并发逻辑全部证明完”，而是更基础的一层合法性分类：

- 哪些共享访问还在语言模型允许的区域
- 哪些共享访问已经越过语言模型的硬边界
- 哪些问题只是逻辑竞态，不属于 data race

## 4. 概念边界与相邻概念

### 4.1 这篇文档直接处理什么

这篇文档直接处理的是：

- data race 作为语言层非法条件的本体
- 冲突访问、潜在并发、非原子访问、`happens-before` 缺失这几个判定部件
- data race 与相邻问题的边界
- data-race-free 作为后续分析入口

这篇文档不负责系统展开：

- 整张 `happens-before` 图怎样构造
- 更高层逻辑竞态如何排障
- sanitizer 的全部实现细节

这些内容分别交给：

- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)
- [Atomicity：单次访问不可分割，不等于整体同步](/Users/maxwell/Knowledge/docs/computer-systems/atomicity.md)

### 4.2 它不等于什么

data race 不等于：

- **一般意义上的 race condition。**
  后者是更广的逻辑概念，前者是语言层非法条件。

- **死锁或活锁。**
  那是进展问题，不是共享访问合法性问题。

- **性能竞争。**
  争锁、抢缓存、调度抖动都不自动构成 data race。

- **“两个线程碰了同一个变量”。**
  还要看是否冲突、是否原子、是否有 `happens-before`。

- **“偶尔才出错”的小 bug。**
  一旦真是 data race，就不是“偶尔不稳定”那么轻。

### 4.3 最容易混淆的相邻对象

| 对象 | 它是什么 | 和 data race 的关系 |
| --- | --- | --- |
| race condition | 调度不同会导致结果不同的更广义逻辑问题 | 可能没有 data race，也可能同时存在 |
| data race | 语言层非法的共享访问形态 | 这是本文核心对象 |
| 冲突访问 | 同一位置、至少一方写 | 是 data race 的必要组成之一 |
| 原子访问 | 被原子模型显式建模的访问 | 会改变 data race 的判定路径 |
| `happens-before` | 用来隔开冲突访问的正式关系图 | 缺了它，冲突访问可能直接越界 |
| DRF | data-race-free 的更高层约束状态 | 是后续逻辑分析更稳的起点 |

### 4.4 最关键的边界句

**看到 “data race” 时，先追问“这是不是潜在并发的冲突访问里至少有一个非原子，且它们之间没有 `happens-before`”，而不要先把它听成“并发下结果不稳定”的模糊坏词。**

## 5. 什么算它，什么不算它

下面这张表是最实用的判别模板。

| 情形 | 在本文里怎么处理 | 为什么 |
| --- | --- | --- |
| 同一位置上潜在并发的普通写 / 普通读，且无 HB | 算 data race | 满足核心定义 |
| 同一位置上潜在并发的普通写 / 普通写，且无 HB | 算 data race | 冲突更直接 |
| 两次访问都为原子访问 | 不按同一路径判成 data race | 是否逻辑正确另说，但定义路径不同 |
| 普通访问被同一个 mutex 正式隔开 | 不算 data race | 冲突访问已被合法同步边隔开 |
| 原子 flag 配普通 `data`，而 `data` 的访问无 HB | `data` 那部分仍可能构成 data race | 只修 flag 不等于整段状态都合法 |
| 结果依赖调度，但共享访问都被正式同步隔开 | 可能是逻辑 race condition，不一定是 data race | 逻辑问题和语言层非法要分开 |

最值得反复提醒的一点是：

**data race 判的是“访问是否合法”，不是“结果看起来合不合理”。**

## 6. 代表性例子、反例与常见误读

### 6.1 代表性例子

最有代表性的例子有三个。

- **普通变量上的并发写 / 读，无锁、无原子、无 HB。**
  这是最标准的 data race 例子。

- **普通变量上的并发写 / 写。**
  它提醒你：不只是读写冲突，写写冲突同样可能越界。

- **只把 flag 做成 atomic，旁边共享数据仍是普通访问。**
  这说明“看起来用了原子”并不等于程序整体已经 data-race-free。

### 6.2 反例

最值得记住的反例有下面这些。

- **同一共享状态在同一个 mutex 下访问。**
  这属于正式同步后的普通访问，不是 data race。

- **两个访问都走原子路径。**
  这不按同一路径落入 data race 定义；是否逻辑正确要另看。

- **结果受调度影响，但访问关系都合法。**
  这可能是 race condition，但不一定是 data race。

- **性能抖动、假共享、锁争用。**
  这些都可能很糟，但不是 data race 的定义本体。

### 6.3 常见误读

最常见的误读有下面几类：

- **“data race 就是任何 race。”**
  错。data race 是更窄、更硬的语言层非法条件。

- **“机器上没出事，所以这里应该不算 data race。”**
  错。判定看语言关系，不看某次平台现象。

- **“只要给一个标志位加 atomic，整段共享状态都安全了。”**
  错。这是并发代码里最常见的错觉之一。

- **“data race 只是偶现小 bug。”**
  错。它会把程序带到未定义行为区域。

- **“没有 data race，就等于程序逻辑一定对。”**
  错。你只是先回到了语言层合法区域，逻辑仍可能错。

## 7. 它会进入哪些更大的模型或判断框架

data race 这个概念本体，最重要的下游去向有四个。

### 7.1 DRF 纪律

很多并发工程实践首先追求的，不是“直接把逻辑最优”，而是先把程序拉回 data-race-free 区域。

### 7.2 `happens-before` 图

data race 判定最终离不开：

- 冲突访问是谁
- 它们之间有没有足够的 `happens-before`

### 7.3 atomicity 与更高层同步

理解了 data race 之后，就更容易看清：

- 为什么只有单对象 atomicity 还不够
- 为什么跨对象关系最终总要落到正式同步图

### 7.4 工具与代码审查

TSan 一类工具、锁纪律检查和代码审查流程，最终都在围绕同一件事运转：  
你的程序是不是先越过了语言层合法边界。

## 8. 自测题 / 验证入口

如果你真的理解了这篇文档，至少应该能回答：

1. 为什么 data race 应该被理解成“语言层越界”，而不是泛泛的“并发下可能出错”？
2. 为什么 race condition 和 data race 不是同一层问题？
3. 为什么“用了 atomic flag”不自动说明整段共享状态没有 data race？
4. 为什么判定 data race 不能靠“这台机器上没复现”？
5. 为什么“没有 data race”与“逻辑一定正确”之间还隔着一层？

一个最实用的自测动作是：

拿下面四种情形各写一句判断：

- 普通写 / 普通读，无 HB
- 普通访问在同一 mutex 下
- 原子 flag + 普通 data
- 调度相关但访问都合法的逻辑竞态

如果你能稳定分清哪些是 data race，哪些只是别的并发问题，这篇文档的核心目标就达到了。

## 9. 迁移与关联模型

理解了 data race 之后，最值得迁移出去的不是某个工具名，而是这组判断：

- 这是语言层合法性问题，还是逻辑层正确性问题？
- 这里缺的是原子访问纪律，还是缺正式同步边？
- 这里已经越界了，还是只是还没达到业务正确？

最值得连着看的文档是：

- [Atomicity：单次访问不可分割，不等于整体同步](/Users/maxwell/Knowledge/docs/computer-systems/atomicity.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)
- [Mutex：你真正买到的不是“挡别人一下”，而是互斥区间和同步边](/Users/maxwell/Knowledge/docs/computer-systems/mutex.md)

最值得保留的迁移句是：

**“并发结果不对”还不一定是 data race；但一旦真有 data race，很多更高层讨论都已经失去可靠地板。**

## 10. 未解问题与继续深挖

- race detector 在真实大规模项目里如何近似 `happens-before`，以及哪些场景最容易漏报或误报？
- DRF 和更高层逻辑正确性之间，能否再压缩出一条更短的代码审查链？
- 在异构系统、设备共享内存和弱一致平台上，data race 边界还会遇到哪些额外工程问题？

## 11. 参考资料

- ISO C++ Working Draft, `intro.races`, `thread.mutex.requirements`, `atomics.order`
- Clang ThreadSanitizer Documentation
- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
