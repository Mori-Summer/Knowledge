---
doc_id: computer-systems-atomicity
title: Atomicity：单次访问不可分割，不等于整体同步
concept: atomicity
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-04-09T11:50:00+08:00'
source_basis:
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_atomics_lockfree_2026_03_16
  - cxx_draft_atomics_ref_generic_general_2026_03_16
  - cxx_draft_intro_races_2026_03_16
  - pure_concept_doc_split_review_2026_04_09
  - methodology_document_generation_methodology
time_context: current_cxx_memory_model_boundary_checked_2026_03_16
applicability: concept_boundary_discrimination_for_single_object_indivisibility_atomic_ref_discipline_and_sync_escalation
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/computer-systems/data-race.md
  - docs/computer-systems/memory-order.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/fence.md
open_questions:
  - 在更复杂对象和异构内存上，`atomic_ref` 的对齐与访问纪律还能怎样压缩成更短的判断模板？
  - 如何把“单对象原子性”和“多对象一致发布”之间的鸿沟讲得更不容易被误读成“再加个 flag 就行”？
  - 在设备内存、共享映射和非传统 coherence 环境里，语言级 atomicity 抽象还会遇到哪些额外边界？
---

# Atomicity：单次访问不可分割，不等于整体同步

## 1. 这份文档要帮你学会什么

这篇文档的重点，不是继续把 `atomicity` 写成一篇机制说明，而是先把这个概念本体讲清。

读完后，你应该至少能做到：

- 说清 `atomicity` 为什么首先是在命名单对象访问的“完整单位”，而不是整体并发正确性
- 区分 atomicity、`memory order`、`happens-before`、lock-free、发布语义这几层对象
- 判断某段代码里你真正买到的只是“不可撕裂访问”，还是还需要继续升级到同步层
- 不再把“原子了”误当成“已经同步了”或“整段协议已经对了”
- 知道这篇文档停在概念边界，更大的跨对象关系要去接 `memory order` 和 `happens-before`

## 2. 一句话结论 / 概念结论

**`atomicity` 首先不是“并发已经安全”，而是“对某个对象的一次访问被当成不可分割的完整单位”；它解决的是单对象访问会不会被并发观察成半完成状态，不自动解决跨对象顺序、发布传播或整体协议正确性。**

如果只记一句最稳的话：

**讨论 atomicity 时，先不要问“线程之间是不是已经同步”，而要先问“这个对象的这次访问是不是一个完整单位”。**

## 3. 这个概念试图解决什么命名或分类问题

并发讨论里，最容易混在一起的是下面几件事：

- 一次对象访问是不是不可分割
- 其他对象会不会跟着有序可见
- 线程之间有没有正式同步关系
- 这个原子实现是不是 lock-free

如果没有 `atomicity` 这个概念，下面这些说法会一直漂：

- “它是原子的，所以别的数据也应该一起过去了”
- “我这里用了 `std::atomic`，所以协议应该已经安全了”
- “这操作不是 lock-free，所以大概也不算原子”
- “自然对齐、`volatile`、平台经验差不多也能算原子性”

所以，这个概念首先解决的不是“整个并发协议怎样证明”，而是更基础的一层分类问题：

- 哪些保证只属于单对象访问单位
- 哪些保证必须再上升到顺序和同步层
- 哪些讨论其实是在谈实现属性，而不是语义本体

## 4. 概念边界与相邻概念

### 4.1 这篇文档直接处理什么

这篇文档直接处理的是：

- `atomicity` 作为单对象访问不可分割性的本体
- 原子 load / store
- 原子 RMW 的整体性
- `atomic_ref` 带来的访问纪律边界
- 单对象粒度和相邻概念的区别

这篇文档不负责系统展开：

- 跨对象发布和可见性链
- 更大的 `happens-before` 图
- 性能上的 lock-free / wait-free 进展属性

这些内容分别交给：

- [Memory Order：从原子性到可见性与重排序控制](/Users/maxwell/Knowledge/docs/computer-systems/memory-order.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)

### 4.2 它不等于什么

`atomicity` 不等于：

- **跨对象同步。**
  它只先保证“这个对象的一次访问”是完整单位。

- **发布语义。**
  一个原子 flag 的访问是 atomic，不等于 flag 旁边的普通数据也自动被发布。

- **`happens-before`。**
  atomicity 不是更大的跨线程关系图。

- **lock-free。**
  原子语义和是否依赖隐藏锁、是否可能阻塞，不是同一层问题。

- **`volatile` 或自然对齐经验。**
  这些不能稳定替代语言层正式原子语义。

### 4.3 最容易混淆的相邻对象

| 对象 | 它是什么 | 和 atomicity 的关系 |
| --- | --- | --- |
| 原子 load / store | 对单个原子对象的访问 | 这是 atomicity 最直接的落点 |
| 原子 RMW | 读旧值并写新值的整体动作 | 它体现了更强的“整体不可拆分” |
| `memory order` | 围绕访问周边建立顺序和同步约束 | 在 atomicity 之上再加一层 |
| `happens-before` | 更大的跨线程关系图 | atomicity 不是它的同义词 |
| `atomic_ref` | 把某个对象按原子对象纪律来处理的接口 | 它说明“补上原子语义”仍要遵守完整访问纪律 |
| lock-free | 进展属性 / 实现属性 | 不是 atomicity 本体 |
| `volatile` | 特定场景下的对象修饰语义 | 不是通用原子性替代 |

### 4.4 最关键的边界句

**看到 “atomicity” 时，先追问“我讨论的是这个对象的一次访问是否不可分割”，而不要先把它听成“整段并发协议已经安全”。**

## 5. 什么算它，什么不算它

下面这张表是最实用的判别模板。

| 情形 | 在本文里怎么处理 | 为什么 |
| --- | --- | --- |
| `std::atomic<int>` 上的 load / store | 算 atomicity 的直接例子 | 这是单对象原子访问 |
| 成功 `fetch_add`、成功 CAS 这类 RMW | 算 atomicity 的更强例子 | 整个读改写动作是一个完整单位 |
| `atomic_ref` 生效期间对对象的统一原子访问 | 仍算 atomicity 语义 | 但前提是所有相关访问都遵守同一原子纪律 |
| 原子 flag 与旁边普通 `data` 一起构成的“发布协议” | 不仅仅是 atomicity | 这已经上升到跨对象同步层 |
| 同一底层对象一半按原子访问、一半按普通访问 | 不能说“atomicity 已经解决” | 这种混用会把你拉回 data race / UB 风险 |
| `volatile`、自然对齐或平台习惯 | 不应直接算 atomicity 的正式依据 | 这不是可移植的语言层合同 |

最值得反复提醒的一点是：

**atomicity 先保的是“这个对象的这次访问”，不是“和它相关的全部状态”。**

## 6. 代表性例子、反例与常见误读

### 6.1 代表性例子

最有代表性的例子有三个。

- **原子 load / store。**
  这说明即使 `memory_order_relaxed` 很弱，只要它是原子访问，这次访问也不会合法地被观察成半完成状态。

- **原子 RMW。**
  这说明有些协议不只需要“能原子读”和“能原子写”，而是需要“读和写中间不能被别人插开”。

- **`atomic_ref` 把对象纳入统一原子访问纪律。**
  这说明“补原子语义”不是随便套一层壳，而是要真的把访问纪律一起切换。

### 6.2 反例

最值得记住的反例有下面这些。

- **`data = 42; flag.store(true, relaxed);`**
  这里 `flag` 的 store 可以是 atomic，但 `data` 并没有因此自动被安全发布。

- **同一底层对象同时存在原子路径和普通路径。**
  这不是“更灵活”，而是高风险混用。

- **“它不是 lock-free，所以大概也不算原子。”**
  这是把语义保证和实现属性混成一层。

- **“x86 上通常没撕裂，所以够用了。”**
  这不是正式语言层合同。

### 6.3 常见误读

最常见的误读有下面几类：

- **“原子了，就等于同步了。”**
  错。atomicity 只是更大同步结构的底板之一。

- **“原子 flag 能顺便把旁边普通数据一起搬过去。”**
  错。那是发布语义，需要更高层关系支持。

- **“`volatile` 差不多也能提供原子性。”**
  不稳。现代 C++ 并发代码不应把它当通用替代。

- **“lock-free 才算真原子。”**
  错。lock-free 是进展属性，不是 atomicity 的定义本体。

- **“只要某条路径用了原子接口，这个对象整体就已经安全。”**
  错。真正值钱的是完整一致的原子访问纪律，不是零散地“某处用了 atomic”。

## 7. 它会进入哪些更大的模型或判断框架

`atomicity` 这个概念本体，最重要的下游去向有四个。

### 7.1 data race 判定

一旦访问不是统一原子纪律，或者和普通访问混用，问题就会很快滑向 data race。

### 7.2 `memory order` 与发布语义

atomicity 只是让单对象访问成为完整单位；如果你还要让别的数据跟着过去，就必须再上升到顺序和同步层。

### 7.3 lock-free / wait-free / blocking 实现判断

理解了 atomicity 之后，就更不容易把：

- 语义上是否原子
- 实现上是否 lock-free

这两层混在一起。

### 7.4 更大的并发协议设计

atomicity 是积木，但不是整栋楼。  
任何跨对象协议最终都要明确：哪部分只是单对象原子，哪部分还需要正式同步关系。

## 8. 自测题 / 验证入口

如果你真的理解了这篇文档，至少应该能回答：

1. 为什么 `atomicity` 应该被理解成“单对象访问单位”，而不是“整段协议正确性”？
2. 为什么 `memory_order_relaxed` 仍然可以有 atomicity？
3. 为什么原子 flag 不会自动替旁边普通数据提供发布语义？
4. 为什么 lock-free 和 atomicity 不是同一层问题？
5. 为什么 `atomic_ref` 的关键不只是接口名字，而是统一的访问纪律？

一个最实用的自测动作是：

拿下面四种判断各写一句说明：

- 原子 load / store
- 原子 RMW
- 原子 flag + 普通 data
- lock-free 与否

如果你能稳定分清哪些还只是 atomicity，哪些已经越过 atomicity 进入更高层，这篇文档的核心目标就达到了。

## 9. 迁移与关联模型

理解了 `atomicity` 之后，最值得迁移出去的不是某条 API 口诀，而是这组判断：

- 这是单对象访问单位问题，还是跨对象同步问题？
- 这是语义保证，还是实现属性？
- 这里缺的是 atomicity，还是更高层的顺序 / 发布 / 同步关系？

最值得连着看的文档是：

- [Data Race：冲突访问何时越过语言模型的合法边界](/Users/maxwell/Knowledge/docs/computer-systems/data-race.md)
- [Memory Order：从原子性到可见性与重排序控制](/Users/maxwell/Knowledge/docs/computer-systems/memory-order.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md)

最值得保留的迁移句是：

**“原子”先回答的是‘会不会被撕裂’，不是‘别的一切会不会都随之正确’。**

## 10. 未解问题与继续深挖

- 在更复杂对象和异构内存上，`atomic_ref` 的对齐与访问纪律还能怎样压缩成更短的判断模板？
- 如何把“单对象原子性”和“多对象一致发布”之间的鸿沟讲得更不容易被误读成“再加个 flag 就行”？
- 在设备内存、共享映射和非传统 coherence 环境里，语言级 atomicity 抽象还会遇到哪些额外边界？

## 11. 参考资料

- ISO C++ Working Draft, `atomics.order`, `atomics.ref.generic.general`, `intro.races`
- GCC Atomic Builtins Documentation
- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
