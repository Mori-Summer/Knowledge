---
doc_id: computer-systems-data-race
title: Data Race：什么时候程序不是“有点危险”，而是直接掉进未定义行为
concept: data race
topic: computer-systems
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-16T14:38:05+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - clang_threadsanitizer_docs_2026_03_16
  - gcc_atomic_builtins_docs_2026_03_16
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_concurrency_bug_analysis
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/atomicity.md
  - docs/computer-systems/happens-before.md
open_questions:
  - race detector 在真实大规模项目里如何近似 `happens-before`，以及哪些场景最容易漏报？
  - 数据竞争自由（DRF）和更高层逻辑正确性之间，还能怎样建立更短的检查链？
---

# Data Race：什么时候程序不是“有点危险”，而是直接掉进未定义行为

## 1. 这份文档要帮你学会什么

这份文档的目标，不是让你把“竞态”当成一个模糊坏词，而是让你精确地区分：  
什么时候程序只是逻辑上可能竞争，什么时候它已经触碰到语言模型里的 data race。

读完后，你应该至少能做到：

- 说清 data race 的标准定义
- 说清它和一般意义上的 race condition、死锁、时序 bug 的边界
- 理解为什么 data race 在 C++ 里是未定义行为，不是“偶尔错一下”
- 用 `happens-before`、mutex、atomic 去判断某段代码会不会 data race
- 理解为什么“加了 atomic”不一定解决逻辑 bug，但通常能改变 data race 判定

一句话先给结论：

**data race 是语言内存模型对并发程序划出的硬边界：如果两个潜在并发的冲突访问里至少有一个不是原子，而且它们之间没有 `happens-before`，程序就已经掉进未定义行为。**

## 2. 一句话结论 / 问题定义

并发里并不是所有“看起来会抢”的情况都一样严重。

有些问题是：

- 顺序不符合业务预期
- 线程先后让结果不同
- 需要更高层协议才能正确

但 data race 更底层，也更致命。

当前草案对它的核心定义可以压缩成一句：

- 两个潜在并发的冲突动作
- 至少一个不是原子
- 二者之间 neither happens before the other

满足这三个条件，程序就不是“需要再优化”，而是已经不再受语言模型保护。

## 3. 对象边界与相邻概念

### 3.1 data race 管什么

它主要管：

- 共享内存访问在语言层面是否合法
- 非原子冲突访问是否被同步关系隔开
- 编译器和运行时还能不能继续假设程序 obey 了并发规则

### 3.2 它不等于什么

它不等于：

- 一般意义上的 race condition
- 逻辑 bug
- 死锁
- 活锁
- 性能竞争
- “两个线程都碰了同一个变量”

### 3.3 和几个相邻概念的边界

**data race vs race condition**

- race condition 是更广义的逻辑概念：不同调度下结果不同，可能出错
- data race 是语言内存模型里的具体非法条件

一个程序可以：

- 没有 data race，但仍然有逻辑 race condition
- 有 data race，此时逻辑分析往往都失去意义，因为程序已经 UB

**data race vs atomicity**

- atomicity 解决“这个对象的访问不可分割”
- data race 关心“冲突访问里是否至少有一个非原子，且没有被 `happens-before` 隔开”

**data race vs `happens-before`**

`happens-before` 是判定 data race 的核心工具。  
没有这张图，data race 只能靠猜。

## 4. 核心结构

理解 data race，至少要抓住下面六个构件。

### 4.1 potential concurrency

当前草案先看的是：两个动作是不是 potentially concurrent。  
不同线程里的动作，通常先满足这个前提。

### 4.2 conflicting actions

要构成 data race，它们还必须是冲突访问。  
典型就是：

- 同一内存位置
- 至少一方写

### 4.3 at least one non-atomic

这是 data race 定义里的硬条件。  
如果两个访问都是原子访问，它们未必逻辑正确，但不会按同样方式落入 data race 定义。

### 4.4 no `happens-before`

如果两者之间存在 `happens-before`，那么它们被模型隔开了。  
如果 neither happens before the other，危险才成立。

### 4.5 undefined behavior

当前草案写得非常直接：  
any such data race results in undefined behavior。

### 4.6 DRF 直觉

当前草案还给出一个很重要的工程直觉：  
正确用 mutex 与 `memory_order::seq_cst` 防止所有 data race 的程序，会表现得像一种简单交错执行。  
这就是并发里常说的 DRF（data-race-free）直觉价值。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最小坏例子：两个线程并发碰同一普通变量

```cpp
int x = 0;

// Thread A
x = 1;

// Thread B
int y = x;
```

如果这两次访问：

- 可能并发发生
- 没有锁
- 不是原子
- 没有其他同步建立 `happens-before`

那么这就是 data race。

关键不是“B 读到了 0 还是 1”，而是：  
**程序已经越过语言允许的边界。**

### 5.2 解决这件事的两条主路

通常只有两类稳定路径：

1. 用锁把冲突访问隔开
2. 把相关访问改成原子，并在需要时补上顺序语义

第一条是高层同步。  
第二条是原子建模。

### 5.3 “没有 data race”不等于“逻辑就对”

看这个例子：

```cpp
std::atomic<bool> ready{false};
int data = 0;

// Thread A
data = 42;
ready.store(true, std::memory_order_relaxed);

// Thread B
if (ready.load(std::memory_order_relaxed)) {
    use(data);
}
```

这里最核心的判断是：

- `ready` 的访问是原子的
- 但 `data` 仍是普通对象
- 一旦 B 在没有足够同步关系下读 `data`，程序仍然可能直接 data race

这提醒你：

- 用了 atomic，不代表整个程序已经 DRF
- 你要看的是所有共享对象的访问关系，而不是只看一个标志位

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

避免 data race 的成本，通常是：

- 更强的同步
- 更严格的共享状态设计
- 更多建模工作

但不避免它，代价不是“偶尔不稳定”，而是“你的程序从语言层面已经不受保护”。

### 6.2 常见失败模式

**失败模式 1：把 data race 当“偶现小 bug”**

不是。  
当前草案明确把它归为 undefined behavior。

**失败模式 2：把 race condition 和 data race 混为一谈**

这会让分析完全失焦。  
前者是更广的逻辑问题，后者是语言层非法。

**失败模式 3：给一个标志位加 atomic，就以为整段共享状态都安全了**

这在发布-订阅、工作队列、启动信号里都极常见。

**失败模式 4：按“我这台 x86 上没出事”推理**

没有 data race 的判定不看“这台机器今天跑出来什么”，而看语言模型下是否建立了足够的关系。

**失败模式 5：把 `volatile` 当 data race 解决方案**

这不是现代 C++ 并发代码的推荐路径。  
`volatile` 不是通用跨线程同步原语。

## 7. 应用场景

### 7.1 并发 bug 排查

很多并发问题的第一步不是问“为什么结果不对”，而是先问“这段代码是不是已经 data race”。

### 7.2 代码审查

看到共享普通对象被多线程读写时，首先要问：有没有同步图？有没有原子语义？

### 7.3 sanitizer 驱动的回归检查

如果项目进入并发阶段，data race 检测几乎是最值得建立的自动化入口之一。

## 8. 工业 / 现实世界锚点

### 8.1 ThreadSanitizer

Clang 官方文档把 ThreadSanitizer 定位为 data race 检测工具。  
这说明在真实工程里，“是否 data-race-free”不是学术附加题，而是值得工具化监控的核心质量线。

### 8.2 标准库同步原语

mutex、condition variable、atomic ordering 在工业里的最基本价值之一，就是帮你把程序维持在 data-race-free 区间内。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 默认先追求 data-race-free，再谈性能微调
- 共享普通对象只要跨线程读写，就优先问同步图有没有建立
- 默认先用 mutex、condition variable、显式原子序来表达同步
- 用 ThreadSanitizer 这类工具做真实代码的 data race 检测

### 9.2 已经过时、明显不推荐或需要带语境理解的路径

**把 `volatile` 当跨线程通信方案**

这条路不是当前主流 C++ 并发实践的推荐路径。  
当前更稳的替代是：`std::atomic`、mutex、condition variable 或更高层同步原语。

**“只要在 x86 上没问题就不算 race”**

这不是语言层判断标准。  
当前更稳的替代是：按 `happens-before` 和原子/锁语义来判定。

**legacy `__sync` builtins 乱入新代码**

GCC 官方文档已明确建议优先使用 `__atomic` builtins。  
这不是因为 `__sync` 永远不能用，而是因为当前主路径已经转向更明确的现代原子语义。

### 9.3 什么时候别直接手工判断

如果代码足够复杂，别只靠肉眼。  
工具、测试和更高层同步设计往往比“我觉得这里没事”更可靠。

## 10. 自测题 / 验证入口

1. data race 和一般意义上的 race condition 有什么本质差别？
2. 构成 data race 的三个关键条件是什么？
3. 为什么 data race 在 C++ 里不是“结果未指定”，而是未定义行为？
4. 为什么“给标志位加 atomic”不等于“整个共享状态就 data-race-free”？
5. 为什么没有 data race 仍然不等于逻辑一定正确？
6. 工程上为什么值得引入 ThreadSanitizer？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- `happens-before` 图分析
- `atomicity` 与 `memory order` 的职责划分
- 发布-订阅代码的合法性检查
- sanitizer 驱动的并发回归
- 锁、条件变量与更高层并发库的正确使用

迁移时最关键的问题始终是：

- 这两个冲突访问是不是 potentially concurrent？
- 其中是否至少有一个非原子？
- 它们之间到底有没有 `happens-before`？

## 12. 未解问题与继续深挖

后续值得继续单独拆的点包括：

- sanitizer 的 `happens-before` 近似模型与局限
- DRF-SC 直觉在真实大型工程里如何更系统地落地
- 在异构系统和设备共享内存下，data race 边界还能怎样继续扩展

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `thread.mutex.requirements.mutex`: https://eel.is/c++draft/thread.mutex.requirements.mutex
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- Clang ThreadSanitizer: https://clang.llvm.org/docs/ThreadSanitizer.html
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
