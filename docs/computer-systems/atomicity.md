---
doc_id: computer-systems-atomicity
title: Atomicity：并发里“不可分割”到底解决了什么，又没有解决什么
concept: atomicity
topic: computer-systems
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-16T14:38:05+08:00'
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
open_questions:
  - 在更复杂对象和异构内存上，`atomic_ref` 的对齐与 lock-free 约束还能怎样继续抽象？
  - 如何把“单对象原子性”和“多对象一致提交”之间的鸿沟压成更短的判断模板？
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

**atomicity vs lock-free**

- 原子性只保证语义上的不可分割
- lock-free 关心实现进展属性：操作是否可能阻塞、是否依赖隐藏锁

当前草案明确说：不 lock-free 的原子操作被视为可能阻塞。  
也就是说，一个操作可以是 atomic，但并不一定是 lock-free。

## 4. 核心结构

理解 atomicity，至少要抓住下面六个构件。

### 4.1 原子对象

当前草案对原子访问的基本承诺，是围绕“atomic object”展开的。  
如果对象不是原子对象，程序就不能直接要求它享受相同的并发语义。

### 4.2 不可分割访问

`atomics.order` 当前草案明确写道：  
即使是 `memory_order::relaxed`，实现也仍然必须保证“对某个特定 atomic object 的任意原子访问，相对于该对象其他原子访问来说是 indivisible 的”。

这就是 atomicity 的核心。

### 4.3 原子 RMW

像 `fetch_add`、`exchange`、成功的 `compare_exchange` 这类操作，不只是“先读再写”，而是作为一个原子 read-modify-write 整体出现。

### 4.4 对象粒度

atomicity 默认是“针对这个对象”的，不是“针对一片相关状态”。  
这正是为什么一个原子标志位不能自动替别的数据提供完整发布语义。

### 4.5 `atomic_ref`

当前草案把 `atomic_ref` 定义为：在其生命周期内，被引用对象按原子对象对待；并且在 `atomic_ref` 存在期间，对该对象的访问必须通过这些 `atomic_ref` 实例完成。

这说明 atomicity 也不是“你可以一半按原子访问，一半按普通访问随便混着来”。

### 4.6 对齐与 lock-free 是额外约束

当前草案对 `atomic_ref` 明确保留了 `required_alignment`，并说明 lock-free 可能依赖对齐。  
这提醒你：

- atomicity 的抽象接口由语言给出
- 但底层是否 lock-free、对齐要求多严，是实现相关现实

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

### 5.2 但 atomicity 只保这个对象，不保整批状态

```cpp
data = 42;
flag.store(true, std::memory_order_relaxed);
```

这里 `flag` 的 store 可以完全是 atomic。  
但这不等于另一个线程看见 `flag == true` 时，就一定也能按预期看见 `data == 42`。

原因很简单：

- atomicity 只保 `flag` 这次访问不可分割
- 它不自动给 `data` 建立发布关系

### 5.3 原子性也不等于一定没有 race

如果你对同一个底层对象：

- 一部分路径走 `std::atomic` / `atomic_ref`
- 另一部分路径仍然走普通非原子访问

那你并没有“更安全”，反而可能直接掉进 data race 或未定义行为。

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

atomicity 的价值非常大，但它只解决最底层那一层问题。

- 好处：把单对象访问变成稳定、不可撕裂的单位
- 代价：如果你把它误当成“并发正确性已经解决”，就会在上层逻辑里出错

### 6.2 常见失败模式

**失败模式 1：把“原子”误当“已同步”**

这是最常见错误。  
原子只说明这个对象的访问不可分割，不说明相关数据已经按预期可见。

**失败模式 2：把 `volatile` 当 atomicity 替代**

当前更稳的可移植路径是 `std::atomic`、`atomic_ref` 或编译器提供的现代原子 builtins。  
`volatile` 不是跨线程同步或原子性的通用替代。

**失败模式 3：把 lock-free 当 atomicity 本身**

有些人会把“不是 lock-free”误解成“不是原子”。  
这不对。atomicity 和 progress property 是两层问题。

**失败模式 4：混用 `atomic_ref` 与普通访问**

当前草案已经明确约束：在 `atomic_ref` 存在期间，对该对象的访问应通过这些 `atomic_ref` 完成。  
混着来，风险极大。

**失败模式 5：依赖“我这台机器上自然对齐就够了”的经验**

这类经验可能在某些平台上凑巧成立，但不是稳定、可移植的知识库结论。  
当前更稳的路径是回到语言提供的原子抽象。

## 7. 应用场景

### 7.1 计数器与状态字

如果你只需要安全更新一个数值本身，atomicity 往往是第一层必要条件。

### 7.2 标志位与一次性状态翻转

标志位至少要原子，才能避免最基础的并发混乱。

### 7.3 引用计数与对象生命周期管理

引用计数的增减首先要求 atomicity，否则对象生死判断会直接失真。

## 8. 工业 / 现实世界锚点

### 8.1 GCC `__atomic` 与 LLVM atomics

GCC 的 `__atomic` builtins 和 LLVM 的 atomics guide 都把“单个原子对象上的 load/store/RMW”作为一等能力暴露出来。  
这说明工业实现对 atomicity 的承载单位，本来就是“原子对象 + 原子操作”，而不是“靠运气的自然对齐访问”。

### 8.2 `atomic_ref`

当前草案把 `atomic_ref` 引进来，就是为了让你能在现有对象上按明确规则获得原子访问，而不是靠平台猜测。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 如果你需要并发访问某个共享标量，就显式用 `std::atomic`
- 如果你要在现有对象上临时套原子访问，就按 `atomic_ref` 规则来，并尊重其对齐与独占访问要求
- 如果你只需要该对象本身的原子性、不需要同步别的数据，可以先从 `memory_order_relaxed` 建模
- 一旦需要跨对象发布可见性，就上升到 `memory order`、`happens-before` 或更高层同步原语

### 9.2 已经过时、明显不推荐或必须带语境理解的路径

**把 `volatile` 当跨线程 atomicity / synchronization 方案**

这条路不是当前主流 C++ 并发实践的推荐路径。  
当前更稳的替代是：`std::atomic`、`atomic_ref`、mutex 或其他明确同步原语。

**legacy `__sync` builtins**

GCC 官方文档明确说：新代码优先用 `__atomic` builtins，而不是 legacy `__sync` builtins。  
原因是后者更旧、更粗糙，前者才显式表达现代原子与内存顺序语义。

**依赖“机器字宽天然原子”经验**

这类经验不适合当知识库基线。  
当前更稳的替代是：回到语言和编译器提供的原子抽象，而不是把平台偶然性写成规范结论。

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

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- `memory order` 的“原子但不一定同步”
- `data race` 的“至少有一个不是原子就可能出事”
- `release sequence` 的原子 RMW 链
- 引用计数、状态字、flag 设计
- `atomic_wait` / `notify` 的等待对象建模

迁移时最关键的问题始终是：

- 我现在需要的是“单对象不可分割”，还是“跨对象可见性”？
- 这个对象有没有被显式建模为原子对象？
- 我是不是把更高层问题误压成了 atomicity 一层？

## 12. 未解问题与继续深挖

后续值得继续单独拆的点包括：

- `atomic_ref` 在复杂对象和特殊对齐场景中的边界
- 原子性、线性化点和更高层事务语义之间的关系
- 在异构内存和设备共享对象上，atomicity 抽象还能怎样继续扩展

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `atomics.lockfree`: https://eel.is/c++draft/atomics.lockfree
- C++ draft `atomics.ref.generic.general`: https://eel.is/c++draft/atomics.ref.generic.general
- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
- LLVM Atomic Instructions and Concurrency Guide: https://llvm.org/docs/Atomics.html
