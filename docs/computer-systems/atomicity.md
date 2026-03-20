---
doc_id: computer-systems-atomicity
title: Atomicity：并发里“不可分割”到底解决了什么，又没有解决什么
concept: atomicity
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-20T15:43:26+08:00'
source_basis:
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_atomics_lockfree_2026_03_16
  - cxx_draft_atomics_ref_generic_general_2026_03_16
  - cxx_draft_intro_races_2026_03_16
  - gcc_atomic_builtins_docs_2026_03_16
  - llvm_atomics_guide_2026_03_16
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_concurrency_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/memory-order.md
  - docs/computer-systems/data-race.md
  - docs/computer-systems/fence.md
open_questions:
  - 在更复杂对象和异构内存上，`atomic_ref` 的对齐与 lock-free 约束还能怎样继续抽象？
  - 如何把“单对象原子性”和“多对象一致提交”之间的鸿沟压成更短的判断模板？
  - 在设备内存、共享映射和非传统 cache coherence 环境下，语言级 atomicity 抽象会遇到哪些额外边界？
---

# Atomicity：并发里“不可分割”到底解决了什么，又没有解决什么

## 1. 这份文档要帮你学会什么

这份文档的目标，不是让你记住“原子操作很安全”这种口号，而是把“atomicity”压成一个你以后分析并发问题时能真正调用的最小模型。

读完后，你应该至少能做到：

- 说清 atomicity 到底在解决什么问题
- 说清它和 `memory order`、`happens-before`、lock-free、可见性之间的边界
- 理解为什么“原子了”不等于“同步了”
- 知道什么时候只需要 atomicity，什么时候必须再加 ordering 或锁
- 避免把 `volatile`、自然对齐经验、硬件直觉误当成可移植的原子性保证

一句话先给结论：

**atomicity 解决的是“对同一个对象的某次访问不能被并发观察成撕裂或半完成状态”；它不自动解决跨对象顺序、线程同步或更高层语义正确性。**

## 2. 一句话结论 / 问题定义

并发里一个最基础的问题是：

- 两个线程同时访问同一个对象
- 某次读会不会读到“半个旧值 + 半个新值”
- 某次更新会不会被观察成中间状态

这就是 atomicity 要解决的问题。

如果没有 atomicity，程序会出现最底层的混乱：

- 撕裂读写
- RMW 不是一个整体
- 同一个对象的访问无法形成稳定可讨论的单位

但 atomicity 只解决“单次访问不可分割”，它不等于：

- 其他对象也跟着有序可见
- 线程之间已经建立同步
- 程序整体逻辑已经正确

换句话说，这个概念真正回答的是：

- 这次访问是不是一个完整单位
- 这个完整单位是否只对“这个对象”成立
- 如果要把它推广到别的对象或更高层协议，还差哪些层

## 3. 对象边界与相邻概念

### 3.1 atomicity 管什么

它主要管：

- 对某个对象的一次访问是否不可分割
- 原子 RMW 是否作为一个整体生效
- 对同一个原子对象的访问能否避免被观察成撕裂状态

### 3.2 它不等于什么

它不等于：

- `memory order`
- `happens-before`
- 可见性传播
- 线程同步
- lock-free
- wait-free

### 3.3 和几个相邻概念的边界

**atomicity vs `memory order`**

- atomicity 回答：“这次访问会不会被并发撕裂？”
- `memory order` 回答：“这次访问会不会顺便给周围读写建立顺序和同步约束？”

一个操作完全可以“原子但不排序”，例如 `memory_order_relaxed`。

**atomicity vs `happens-before`**

- atomicity 是单个对象访问层面的不可分割性
- `happens-before` 是更高层的可见性与合法性关系图

原子性不足以替代 `happens-before`。

**atomicity vs data race**

- atomicity 能把对象访问变成不可撕裂的单位
- data race 关心冲突访问里是否仍存在非原子且无 `happens-before` 的路径

一个程序可以：

- 对某个对象用了原子访问，但别的相关对象仍然 data race
- 没有 data race，但仍然存在更高层逻辑错误

**atomicity vs lock-free**

- 原子性只保证语义上的不可分割
- lock-free 关心实现进展属性：操作是否可能阻塞、是否依赖隐藏锁

当前草案明确说：不 lock-free 的原子操作被视为可能阻塞。也就是说，一个操作可以是 atomic，但并不一定是 lock-free。

### 3.4 最关键的边界句

可以把它压成一句话：

**atomicity 只把“这个对象的一次访问”变成稳定单位，不会自动把“相关的别的对象”“更大的状态关系”一起纳入保护。**

## 4. 核心结构

### 4.1 atomicity 的最小模型

理解 atomicity，至少要抓住下面六个构件。

| 构件 | 它是什么 | 缺了会怎样 |
| --- | --- | --- |
| 原子对象 | 被语言显式建模为 atomic 的对象 | 你不能稳定要求不可分割语义 |
| 不可分割访问 | 单次 load/store/RMW 不能被并发观察成半完成 | 会出现撕裂读写 |
| RMW 整体性 | read-modify-write 是一个单元 | 协议会在“先读再写”之间被打穿 |
| 对象粒度边界 | 保护只对这个对象成立 | 容易把单对象保证误扩展成跨对象保证 |
| `atomic_ref` 独占访问纪律 | 套上原子语义后不能再随便混普通访问 | 会重新掉进 data race / UB |
| 实现属性层 | 对齐、lock-free、可能阻塞 | 把语义保证和实现成本混为一谈 |

### 4.2 原子对象是语言给你的正式落点

当前草案对原子访问的基本承诺，是围绕“atomic object”展开的。如果对象不是原子对象，程序就不能直接要求它享受相同的并发语义。

最值得记住的是：

- 你不是因为“感觉这个对象应该能原子访问”而得到 atomicity
- 你是因为把它显式纳入语言原子模型，才得到 atomicity

### 4.3 不可分割访问是第一层价值

`atomics.order` 当前草案明确写道：

- 即使是 `memory_order::relaxed`，实现也仍然必须保证“对某个特定 atomic object 的任意原子访问，相对于该对象其他原子访问来说是 indivisible 的”

这就是 atomicity 的核心。

最值得记住的翻译是：

**弱顺序不等于可撕裂。即使是 relaxed，只要它是原子访问，它对这个对象仍是完整单位。**

### 4.4 原子 RMW 是第二层价值

像 `fetch_add`、`exchange`、成功的 `compare_exchange` 这类操作，不只是“先读再写”，而是作为一个原子 read-modify-write 整体出现。

这点非常重要，因为很多协议真正依赖的不是“能原子读”和“能原子写”，而是：

- 中间不能插进别人
- 这次读和这次写必须属于同一个不可拆分动作

### 4.5 对象粒度是 atomicity 最大的适用边界

atomicity 默认是“针对这个对象”的，不是“针对一片相关状态”。

这正是为什么一个原子标志位不能自动替别的数据提供完整发布语义。

可以这样记：

- `flag` 原子，不代表 `data` 也自动安全
- 某个计数器原子，不代表与它相关的结构体整体也自动一致

### 4.6 `atomic_ref` 告诉你不能半原子半普通地碰同一对象

当前草案把 `atomic_ref` 定义为：在其生命周期内，被引用对象按原子对象对待；并且在 `atomic_ref` 存在期间，对该对象的访问必须通过这些 `atomic_ref` 实例完成。

这说明 atomicity 也不是“你可以一半按原子访问，一半按普通访问随便混着来”。

这里最值钱的认知是：

- “retrofit 原子语义”不是宽松模式
- 一旦对象被纳入 `atomic_ref` 的原子访问纪律，你就得把它当真的 atomic object 处理

### 4.7 对齐、lock-free 与语义保证不是一层

当前草案对 `atomic_ref` 明确保留了 `required_alignment`，并说明 lock-free 可能依赖对齐。

这提醒你：

- atomicity 的抽象接口由语言给出
- 但底层是否 lock-free、对齐要求多严，是实现相关现实

所以别把下面几件事混成一件：

- 语义上是不是 atomic
- 实现上是不是 lock-free
- 机器上是不是刚好对齐且快

### 4.8 一页纸判断模板

以后看到“这个地方要不要原子”，先过下面这张表：

| 问题 | 你必须回答什么 |
| --- | --- |
| 我保护的是哪个对象 | 单对象还是多对象关系 |
| 我需要的只是不可分割访问吗 | 还是还需要跨对象顺序 |
| 这个对象是否显式建模为原子对象 | `std::atomic` 还是 `atomic_ref` |
| 我有没有和普通访问混用 | 如果混用，风险是否已经失控 |
| 我在讨论语义保证还是实现成本 | 不要把 atomicity 和 lock-free 混在一起 |

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最简单的主链：把“半写入状态”排除掉

先看最小例子：

```cpp
std::atomic<int> x{0};

// Thread A
x.store(42, std::memory_order_relaxed);

// Thread B
int v = x.load(std::memory_order_relaxed);
```

这里最核心的不是“relaxed 很弱”，而是：

1. `x` 是原子对象
2. A 对 `x` 的 store 是原子访问
3. B 对 `x` 的 load 也是原子访问
4. 即使没有额外排序，B 也不会合法地看到一个“半写入的 42”

这就是 atomicity 的底层价值：

**至少让同一个对象的访问单位是完整的。**

### 5.2 跨对象链：为什么 atomicity 在这里立刻失效

```cpp
data = 42;
flag.store(true, std::memory_order_relaxed);
```

这里 `flag` 的 store 可以完全是 atomic。

但这不等于另一个线程看见 `flag == true` 时，就一定也能按预期看见 `data == 42`。

原因很简单：

- atomicity 只保 `flag` 这次访问不可分割
- 它不自动给 `data` 建立发布关系

这条链最值得记住的翻译是：

**atomicity 能把旗子插稳，但不会自动帮你把旗子旁边那堆货一起搬过去。**

### 5.3 RMW 链：为什么某些协议必须依赖“整体动作”

很多协议真正依赖的不是“写进去一个值”，而是：

- 先读当前值
- 再基于它决定写回什么
- 并且不允许别人插入

这就是 RMW 的工作：

1. 读取旧值
2. 在同一个原子动作里完成修改
3. 让别的线程观察到的是“修改前”或“修改后”，而不是被拆开的中间态

这也是为什么：

- 原子 load/store 能解决一类问题
- 原子 RMW 又解决更强一层的问题

### 5.4 为什么 atomicity 也不等于没有 data race

如果你对同一个底层对象：

- 一部分路径走 `std::atomic` / `atomic_ref`
- 另一部分路径仍然走普通非原子访问

那你并没有“更安全”，反而可能直接掉进 data race 或未定义行为。

所以真正的因果不是：

- “我这里有 atomic，所以安全”

而是：

- “所有相关访问是否都在同一个原子访问纪律下”

### 5.5 原子性判断链：看到并发访问时该怎么判断

这篇文档最有价值的可调用部分，是下面这条判断链：

1. **先问对象是谁。**
   你在讨论的是单个对象，还是多个对象的关系？

2. **再问访问是否都显式原子。**
   `std::atomic`、`atomic_ref`、编译器正式原子接口，还是普通访问？

3. **再问需要的只是不可分割，还是还要顺序与同步。**
   如果只是前者，atomicity 可能够；如果是后者，必须继续上升到 memory order / lock。

4. **再问有没有混用路径。**
   一旦同一对象同时被原子和非原子路径碰，风险急剧上升。

5. **最后才问性能与 lock-free。**
   别在语义还没站稳前，先讨论是不是 lock-free。

### 5.6 真正的因果核心

把这一节压成一句最值得记住的话：

**atomicity 先把“这个对象的一次访问”做成完整单位；只有在此之上，再加顺序、同步或锁，程序才有可能跨对象地变正确。**

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

atomicity 的价值非常大，但它只解决最底层那一层问题。

- 好处：把单对象访问变成稳定、不可撕裂的单位
- 代价：如果你把它误当成“并发正确性已经解决”，就会在上层逻辑里出错

### 6.2 六类高频失败模式

**失败模式 1：把“原子”误当“已同步”**

这是最常见错误。原子只说明这个对象的访问不可分割，不说明相关数据已经按预期可见。

**失败模式 2：把 `volatile` 当 atomicity 替代**

当前更稳的可移植路径是 `std::atomic`、`atomic_ref` 或编译器提供的现代原子 builtins。`volatile` 不是跨线程同步或原子性的通用替代。

**失败模式 3：把 lock-free 当 atomicity 本身**

有些人会把“不是 lock-free”误解成“不是原子”。这不对。atomicity 和 progress property 是两层问题。

**失败模式 4：混用 `atomic_ref` 与普通访问**

当前草案已经明确约束：在 `atomic_ref` 存在期间，对该对象的访问应通过这些 `atomic_ref` 完成。混着来，风险极大。

**失败模式 5：依赖“我这台机器上自然对齐就够了”的经验**

这类经验可能在某些平台上凑巧成立，但不是稳定、可移植的知识库结论。当前更稳的路径是回到语言提供的原子抽象。

**失败模式 6：把多对象事务问题压成单对象 atomicity**

这会让协议在表面上“每个点都原子”，实际却没有整体正确性。

### 6.3 现场症状与优先回查方向

| 现场症状 | 优先回查什么 | 常见真实原因 |
| --- | --- | --- |
| 某个标志位稳定，但相关数据还是乱 | 是否缺跨对象同步 | 把 atomicity 当成可见性传播 |
| 代码里用了 atomic_ref 还是偶发出事 | 是否混用了普通访问 | 同一对象存在双重访问纪律 |
| 团队一直在讨论 lock-free 性能 | 语义是否先成立 | 把实现属性先于语义保证讨论 |
| 单对象操作看着都原子，但整体协议错 | 对象粒度边界 | 多对象关系没有被同步建模 |
| 某平台上一直“没问题” | 是否依赖了平台偶然性 | 把自然对齐或硬件直觉当规范 |

### 6.4 为什么“单对象原子”最容易让人高估自己

它之所以容易误导人，是因为它给你一种非常强的感觉：

- 这个值现在很稳
- 所以协议应该也很稳

但真实情况是：

- atomicity 往往只把一个点做稳
- 而协议真正错的，常在点与点之间

所以只要你的脑中出现“这个标志位已经原子了，应该够了吧”，你就该继续追问：

- 别的对象呢？
- 顺序呢？
- `happens-before` 呢？

### 6.5 一个很实用的判别句式

如果你看到：

- **问题只是“同一个对象会不会被撕裂”**，优先想 atomicity
- **问题是“看见这个对象后别的对象是否也应就绪”**，atomicity 不够
- **代码在讨论 lock-free，但语义边界没讲清**，先退回 atomicity 与同步层
- **对象被原子和非原子路径混着碰**，优先怀疑这里已经越界

## 7. 应用场景

### 7.1 计数器与状态字

如果你只需要安全更新一个数值本身，atomicity 往往是第一层必要条件。

典型对象：

- 计数器
- phase number
- 状态字
- generation

### 7.2 标志位与一次性状态翻转

标志位至少要原子，才能避免最基础的并发混乱。

但这里也正好最容易误解：

- 标志位原子，通常只是第一步
- 不是整套发布-订阅协议自动完成

### 7.3 引用计数与对象生命周期管理

引用计数的增减首先要求 atomicity，否则对象生死判断会直接失真。

这类场景非常能说明 atomicity 的底层价值：

- 先保证增减本身是完整动作
- 再看是否还要额外同步生命周期边界

### 7.4 `atomic_wait` / `notify` 的等待对象

`atomic_wait` 之所以成立，本身就依赖那个 atomic object 的值历史可被稳定讨论。

也就是说：

- 没有 atomicity
- 就没有后面基于该对象做等待协议的资格

## 8. 工业 / 现实世界锚点

### 8.1 GCC `__atomic` 与 LLVM atomics 说明工业世界的落点就是“对象 + 操作”

GCC 的 `__atomic` builtins 和 LLVM 的 atomics guide 都把“单个原子对象上的 load/store/RMW”作为一等能力暴露出来。

这说明工业实现对 atomicity 的承载单位，本来就是：

- 原子对象
- 原子操作

而不是“靠运气的自然对齐访问”。

### 8.2 `std::atomic` / `atomic_ref` 给出两条正式路径

当前草案把 `std::atomic` 和 `atomic_ref` 分别提供为：

- “直接定义原子对象”
- “在现有对象上施加原子访问规则”

的两条正式路径。

这说明现实工程里，atomicity 既可能出现在专门设计的状态字上，也可能出现在需要 retrofit 原子语义的已有对象上，但两者都必须走语言显式建模。

### 8.3 状态字、引用计数与一次性标志为什么总反复出现

在运行时、缓存状态机、引用计数和一次性初始化标志里，工程上最常见的原子对象往往就是一个单独的状态字。

这类场景之所以反复出现，恰好说明 atomicity 的真实落点通常是：

- 先把单对象访问变成稳定单位
- 再在其上搭更高层协议

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 如果你需要并发访问某个共享标量，就显式用 `std::atomic`
- 如果你要在现有对象上临时套原子访问，就按 `atomic_ref` 规则来，并尊重其对齐与独占访问要求
- 优先区分“我要定义原子对象”还是“我要 retrofit 一个已有对象”，不要两种路径混着猜
- 如果你只需要该对象本身的原子性、不需要同步别的数据，可以先从 `memory_order_relaxed` 建模
- 一旦需要跨对象发布可见性，就上升到 `memory order`、`happens-before` 或更高层同步原语

### 9.2 已经过时、明显不推荐或必须带语境理解的路径

| 路径 | 为什么旧/不稳 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| 把 `volatile` 当跨线程 atomicity / synchronization 方案 | 这不是当前主流 C++ 并发实践的推荐路径 | 不能稳定替代原子语义与同步 | `std::atomic`、`atomic_ref`、mutex 或其他明确同步原语 |
| legacy `__sync` builtins | GCC 已明确建议新代码优先 `__atomic` | 语义表达更旧、更粗糙 | 标准原语或 `__atomic` builtins |
| 依赖“机器字宽天然原子”经验 | 不适合当知识库基线 | 平台偶然性不可移植 | 回到语言和编译器提供的原子抽象 |
| 把 lock-free 当成是否值得用 atomic 的第一问题 | 讨论顺序错了 | 容易在语义未稳前就优化 | 先确认 atomicity 与同步需求，再讨论进展属性 |
| 用单个原子标志位承诺整批状态安全 | atomicity 只对该对象成立 | 跨对象关系没有被建模 | 继续补 memory order、locks 或更高层协议 |

### 9.3 什么时候别停在 atomicity

如果你真正要解决的是：

- 发布-订阅
- 跨多个字段的一致观察
- 线程间顺序
- data race 合法性

那答案通常不会停在 atomicity，而会继续走向：

- `memory order`
- `happens-before`
- lock / condition variable
- 更高层并发协议

## 10. 自测题 / 验证入口

1. atomicity 到底保证了什么，为什么它不等于同步？
2. 为什么 `memory_order_relaxed` 仍然可以保持 atomicity？
3. 为什么原子标志位不能自动替相关普通数据提供发布语义？
4. 为什么“不是 lock-free”不等于“不是原子”？
5. 为什么 `atomic_ref` 不能和普通访问随便混用？
6. 为什么把 `volatile` 当跨线程 atomicity 方案是危险的？
7. 为什么单对象原子性不能推出多对象一致性？
8. 如果一个协议看起来“每个点都原子”，但整体仍错，最可能漏了哪一层？
9. 为什么讨论 atomicity 时要先分清“语义保证”和“实现属性”？
10. 请用一句话解释：atomicity 的保护边界到底画到哪里。

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的核心句

理解了这篇文档后，你应该能把下面这句迁移出去：

**atomicity 只把一个对象的一次访问做成稳定单位；要让多个对象一起正确，还必须继续往上建模。**

### 11.2 可以迁移到哪些领域

- `memory order` 的“原子但不一定同步”
- `data race` 的“至少有一个不是原子就可能出事”
- `release sequence` 的原子 RMW 链
- 引用计数、状态字、flag 设计
- `atomic_wait` / `notify` 的等待对象建模

### 11.3 与相邻模型的关系

| 模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| `memory_order` | 管周围操作顺序与同步 | 补“单个访问为什么先必须是完整单位” |
| `data_race` | 管语言层并发合法性 | 补为什么“至少这个对象本身不能被撕裂” |
| lock-free / wait-free | 管进展属性 | 补为什么语义正确和进展属性是两层问题 |
| `atomic_ref` | retrofit 原子语义 | 补 retrofit 时为什么不能再混普通访问 |
| `atomic_wait` | 管单对象等待 | 补等待对象为何必须先是稳定 atomic 历史 |

### 11.4 最值得保留的迁移动作

以后看到并发状态访问，可以先问自己：

- 我现在需要的是“单对象不可分割”，还是“跨对象可见性”？
- 这个对象有没有被显式建模为原子对象？
- 我是不是把更高层问题误压成了 atomicity 一层？

只要这三问没答清，后续分析通常还不稳。

## 12. 未解问题与继续深挖

- `atomic_ref` 在复杂对象和特殊对齐场景中的边界，是否值得单独做案例文档？
- 原子性、线性化点和更高层事务语义之间的关系，是否需要专门展开？
- 在异构内存和设备共享对象上，atomicity 抽象还能怎样继续扩展？

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `atomics.lockfree`: https://eel.is/c++draft/atomics.lockfree
- C++ draft `atomics.ref.generic.general`: https://eel.is/c++draft/atomics.ref.generic.general
- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
- LLVM Atomic Instructions and Concurrency Guide: https://llvm.org/docs/Atomics.html
