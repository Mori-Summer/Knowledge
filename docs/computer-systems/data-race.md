---
doc_id: computer-systems-data-race
title: Data Race：什么时候程序不是“有点危险”，而是直接掉进未定义行为
concept: data_race
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-20T15:43:26+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - clang_threadsanitizer_docs_2026_03_16
  - clang_thread_safety_analysis_docs_2026_03_19
  - gcc_atomic_builtins_docs_2026_03_16
time_context: current_practice_checked_2026_03_19
applicability: personal_concept_learning_and_concurrency_bug_analysis
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/atomicity.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/mutex.md
open_questions:
  - race detector 在真实大规模项目里如何近似 `happens-before`，以及哪些场景最容易漏报？
  - 数据竞争自由（DRF）和更高层逻辑正确性之间，还能怎样建立更短的检查链？
  - 在异构系统、设备共享内存和弱一致平台上，data race 边界还会遇到哪些额外工程问题？
---

# Data Race：什么时候程序不是“有点危险”，而是直接掉进未定义行为

## 1. 这份文档要帮你学会什么

这份文档的目标，不是让你把“竞态”当成一个模糊坏词，而是让你精确地区分：什么时候程序只是逻辑上可能竞争，什么时候它已经触碰到语言模型里的 data race。

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

换句话说，它真正回答的是：

- 这段代码在语言层面是否还合法
- 编译器和运行时是否还必须按你以为的方式对待它

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

`happens-before` 是判定 data race 的核心工具。没有这张图，data race 只能靠猜。

### 3.4 最关键的边界句

可以把它压成一句话：

**data race 不是“表现不好”的代码，而是“语言不再承诺任何结果”的代码。**

## 4. 核心结构

### 4.1 data race 的最小判定模型

理解 data race，至少要抓住下面六个构件。

| 构件 | 它是什么 | 缺了会怎样 |
| --- | --- | --- |
| potential concurrency | 两个动作可能并发发生 | 如果根本不并发，就谈不上 data race |
| conflicting actions | 同一位置、至少一方写 | 不冲突就不构成 data race |
| at least one non-atomic | 至少有一方不是原子访问 | 如果都原子，判定路径不同 |
| no `happens-before` | 两者之间没有同步顺序边 | 有 HB 就被合法隔开 |
| undefined behavior | 满足定义即掉进 UB | 不是“结果未指定”，而是语言不再保护 |
| DRF 视角 | 把程序维持在 data-race-free 区域 | 才能谈更高层逻辑稳定性 |

### 4.2 三步判定法

看到疑似并发访问时，最实用的不是先争论“会不会刚好出错”，而是跑这三步：

1. **先判冲突。**
   是不是同一内存位置，且至少一方写？

2. **再判原子性。**
   是否至少一方是非原子访问？

3. **最后判 `happens-before`。**
   两者之间有没有锁、原子同步或其他正式边把它们隔开？

这三步里，真正值钱的是顺序：

- 不先判冲突，容易误把无关访问当 race
- 不先判原子性，容易把“逻辑有问题”和“语言层非法”混在一起
- 不判 HB，就只能凭直觉猜

### 4.3 `happens-before` 是真正的生死线

很多人会把 data race 想成“是否用了锁”。

这不够准确。更本质的说法是：

- 有没有足够的 `happens-before`

因为：

- mutex 可以建 HB
- 合法的原子同步也可以建 HB
- 某些更高层同步原语也在底层落回 HB 图

data race 真正问的是：

- 这两个冲突访问有没有被 HB 隔开

### 4.4 原子访问会改变判定路径，但不会自动修好协议

如果两个访问都是原子访问，它们未必逻辑正确，但不会按同样方式落入 data race 定义。

这点非常重要，因为它解释了一个常见误解：

- “没有 data race”不等于“程序逻辑就对”

你可能已经从“语言层非法”退回到了“逻辑仍可能错，但至少可讨论”的状态。

### 4.5 UB 不是“偶尔会错”，而是分析地板被抽走

当前草案写得非常直接：

- any such data race results in undefined behavior

这句话的真正含义不是“结果有点随机”，而是：

- 编译器可以做更激进假设
- 你的“我在机器上看到的现象”不再具有稳定解释力
- 很多上层逻辑讨论会失焦

### 4.6 DRF 直觉为什么重要

当前草案还给出一个很重要的工程直觉：

- 正确用 mutex 与 `memory_order::seq_cst` 防止所有 data race 的程序，会表现得像一种简单交错执行

这就是并发里常说的 DRF（data-race-free）直觉价值。

它最重要的工程意义是：

- 先把程序推进到 DRF 区间
- 再谈性能、弱序和更高层协议优化

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

### 5.2 两条主路：把程序拉回合法区

通常只有两类稳定路径：

1. 用锁把冲突访问隔开
2. 把相关访问改成原子，并在需要时补上顺序语义

第一条是高层同步。第二条是原子建模。

但无论走哪条，你真正做的都是：

- 把“无 HB 的冲突访问”变成“被合法同步隔开的访问”

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

### 5.4 数据竞争排障链：先判 legality，再判 logic

这篇文档最重要的可调用部分，是下面这条排障链：

1. **先找共享对象。**
   真正跨线程共享的对象有哪些？

2. **再找冲突访问。**
   哪些读写落在同一位置，且至少一方写？

3. **再找原子边界。**
   哪些访问是普通访问，哪些是原子访问？

4. **再画 HB 图。**
   锁、条件变量、原子同步、线程创建/加入边在哪里？

5. **最后才讨论逻辑正确性。**
   如果程序先天就 data race，先别急着谈高层业务对不对。

这条链很重要，因为很多团队会反过来：

- 先讨论业务结果
- 最后才发现代码根本没过 data race 合法性这一关

### 5.5 真正的因果核心

把这一节压成一句最值得记住的话：

**data race 不是“两个线程碰上了”这么简单，而是“两个潜在并发的冲突访问里至少有一个非原子，并且它们之间没有 `happens-before`”；一旦满足，程序就从“逻辑可能错”直接跌到“语言层不再保护”。**

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

避免 data race 的成本，通常是：

- 更强的同步
- 更严格的共享状态设计
- 更多建模工作

但不避免它，代价不是“偶尔不稳定”，而是“你的程序从语言层面已经不受保护”。

### 6.2 六类高频失败模式

**失败模式 1：把 data race 当“偶现小 bug”**

不是。当前草案明确把它归为 undefined behavior。

**失败模式 2：把 race condition 和 data race 混为一谈**

这会让分析完全失焦。前者是更广的逻辑问题，后者是语言层非法。

**失败模式 3：给一个标志位加 atomic，就以为整段共享状态都安全了**

这在发布-订阅、工作队列、启动信号里都极常见。

**失败模式 4：按“我这台 x86 上没出事”推理**

没有 data race 的判定不看“这台机器今天跑出来什么”，而看语言模型下是否建立了足够的关系。

**失败模式 5：把 `volatile` 当 data race 解决方案**

这不是现代 C++ 并发代码的推荐路径。`volatile` 不是通用跨线程同步原语。

**失败模式 6：只谈性能，不先谈 DRF**

还没先把程序拉回 data-race-free 区间，就开始做弱序和 lock-free 优化，通常是顺序反了。

### 6.3 现场症状与优先回查方向

| 现场症状 | 优先回查什么 | 常见真实原因 |
| --- | --- | --- |
| 并发 bug 表现非常随机 | 先判是不是 data race | 程序已掉进 UB，现象本身不稳定 |
| 标志位看着是 atomic，相关数据却乱 | 是否仍有普通共享对象无 HB | 只修了标志位，没修整段共享状态 |
| 代码审查中大家争论“应该读到哪个值” | 先画 HB 图 | 可能根本还没过合法性判定 |
| sanitizer 偶发报 race，手工复现却不稳定 | 不要先否认工具 | data race 本来就不承诺稳定复现 |
| 团队一直以平台表现为依据 | 判定标准错位 | 把硬件偶然性当语言合同 |

### 6.4 为什么 data race 会让上层讨论失真

data race 最麻烦的地方不只是“程序可能错”，而是它会污染上层推理。

一旦程序已是 UB：

- “这次为什么读到了 7”这类问题未必还有稳定答案
- “我本地一直正常”不再是强证据
- 很多优化、重构、工具分析结果都会被干扰

所以 data race 分析的正确顺序永远是：

- 先判 legality
- 再判 logic

### 6.5 一个很实用的判别句式

如果你看到：

- **两个线程都在碰同一普通对象**，先问有没有 HB
- **只有一个标志位是 atomic，其他数据是普通对象**，先别急着说“已经安全”
- **讨论一直围绕“平台上应该没事”**，先把分析拉回语言模型
- **结果随机到像幽灵 bug**，优先怀疑 data race，而不是先怀疑业务逻辑

## 7. 应用场景

### 7.1 并发 bug 排查

很多并发问题的第一步不是问“为什么结果不对”，而是先问“这段代码是不是已经 data race”。

### 7.2 代码审查

看到共享普通对象被多线程读写时，首先要问：

- 有没有同步图？
- 有没有原子语义？
- 这些访问之间有没有 HB？

### 7.3 发布-订阅和启动信号设计

这是 data race 最容易被误判的场景之一。

因为开发者很容易：

- 只给 flag 上 atomic
- 却忘了与其关联的数据和顺序关系

### 7.4 sanitizer 驱动的回归检查

如果项目进入并发阶段，data race 检测几乎是最值得建立的自动化入口之一。

它的价值不只是抓一个 bug，而是把程序长期维持在 DRF 区间内。

## 8. 工业 / 现实世界锚点

### 8.1 ThreadSanitizer 说明 data race 值得进入工程质量线

Clang 官方文档把 ThreadSanitizer 明确定位为 data race 检测工具，并直接给出 `-fsanitize=thread` 的使用方式。

这说明在真实工程里，“程序是否 data-race-free”不是事后猜测，而是值得进入测试、回归和 CI 的工具化质量线。

### 8.2 Clang Thread Safety Analysis 说明工业界也会静态建模锁纪律

Clang 的 Thread Safety Analysis 文档把“受 mutex 保护的数据”“线程持有 capability 集合”做成了静态分析模型，并明确写到它已经在 Google 内部代码库中被广泛使用。

这说明工业界处理 data race，不只是靠运行后抓问题，也会在代码层显式标注哪些状态必须在什么锁语境下访问。

### 8.3 标准库同步原语的现实价值之一，就是把程序维持在 DRF 区间

mutex、condition variable、atomic ordering 在工业里的最基本价值之一，就是帮你把程序维持在 data-race-free 区间内。

现实工作流里更稳的做法通常不是“人工代码审查单独负责”，而是把：

- 同步原语设计
- sanitizer 检测
- 静态分析
- 回归用例

一起接成质量闭环。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-19 更推荐的实践

当前更稳的工程实践是：

- 默认先追求 data-race-free，再谈性能微调
- 共享普通对象只要跨线程读写，就优先问同步图有没有建立
- 默认先用 mutex、condition variable、显式原子序来表达同步
- 用 ThreadSanitizer 这类工具做真实代码的 data race 检测
- 如果代码库规模足够大或锁纪律复杂，考虑把 capability / guarded-by 这类静态分析约束也纳入工程规则

### 9.2 已经过时、明显不推荐或需要带语境理解的路径

| 路径 | 为什么旧/不稳 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| 把 `volatile` 当跨线程通信方案 | 这不是当前主流 C++ 并发实践的推荐路径 | 不能稳定建立合法同步 | `std::atomic`、mutex、condition variable 或更高层同步原语 |
| “只要在 x86 上没问题就不算 race” | 这不是语言层判断标准 | 把平台偶然性当语言合同 | 按 `happens-before` 和原子/锁语义判定 |
| legacy `__sync` builtins 乱入新代码 | GCC 已明确建议优先使用 `__atomic` | 现代原子语义表达不够明确 | `__atomic` builtins 或标准原语 |
| 只给 flag 上 atomic 就宣布共享状态安全 | 修复粒度过窄 | 相关普通对象仍可能 data race | 画完整同步图，覆盖所有共享对象 |
| 先做弱序和 lock-free 优化，再补 DRF | 顺序错了 | 容易在 UB 上继续优化 | 先把程序拉回 data-race-free 区间 |

### 9.3 什么时候别只靠肉眼判断

如果代码足够复杂，别只靠肉眼。工具、测试和更高层同步设计往往比“我觉得这里没事”更可靠。

尤其在下面这些场景：

- 大量模板 / 回调 / 线程池代码
- 锁纪律复杂
- 发布-订阅和 shutdown 协议交织
- 历史代码有原子与普通访问混用痕迹

## 10. 自测题 / 验证入口

1. data race 和一般意义上的 race condition 有什么本质差别？
2. 构成 data race 的三个关键条件是什么？
3. 为什么 data race 在 C++ 里不是“结果未指定”，而是未定义行为？
4. 为什么“给标志位加 atomic”不等于“整个共享状态就 data-race-free”？
5. 为什么没有 data race 仍然不等于逻辑一定正确？
6. 工程上为什么值得引入 ThreadSanitizer？
7. 为什么 data race 分析的正确顺序是“先判 legality，再判 logic”？
8. 如果一个 bug 在不同机器和不同编译选项下表现很飘，你为什么要优先怀疑 data race？
9. `happens-before` 在 data race 判定里到底扮演什么角色？
10. 请用一句话解释：data race 到底是“逻辑竞争”，还是“语言层越界”？

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的核心句

理解了这篇文档后，你应该能把下面这句迁移出去：

**data race 不是“代码可能竞争”，而是“代码已经越过语言允许的并发边界”。**

### 11.2 可以迁移到哪些领域

- `happens-before` 图分析
- `atomicity` 与 `memory_order` 的职责划分
- 发布-订阅代码的合法性检查
- sanitizer 驱动的并发回归
- 锁、条件变量与更高层并发库的正确使用
- DRF-first 的工程流程设计

### 11.3 与相邻模型的关系

| 模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| atomicity | 解释单对象访问是否不可撕裂 | 补什么时候程序整体仍会非法 |
| `memory_order` | 解释顺序和同步边 | 补为什么这些边在 legality 判定里是生死线 |
| mutex / condition variable | 提供同步原语 | 补这些原语最基本的现实价值之一是避免 data race |
| ThreadSanitizer | 运行时近似抓 race | 补其在整个 DRF 工作流中的位置 |
| race condition 分析 | 处理逻辑时序问题 | 补为什么 legality 比 logic 更底层 |

### 11.4 最值得保留的迁移动作

以后看到任何并发共享访问，可以先问自己：

- 这两个冲突访问是不是 potentially concurrent？
- 其中是否至少有一个非原子？
- 它们之间到底有没有 `happens-before`？

只要这三问里任何一问答不清，data race 风险就还没被排除。

## 12. 未解问题与继续深挖

- sanitizer 的 `happens-before` 近似模型与局限，是否值得单独拆篇？
- DRF-SC 直觉在真实大型工程里如何更系统地落地？
- 在异构系统和设备共享内存下，data race 边界还能怎样继续扩展？

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，核对日期为 `2026-03-19`。

- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `thread.mutex.requirements.mutex`: https://eel.is/c++draft/thread.mutex.requirements.mutex
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- Clang ThreadSanitizer: https://clang.llvm.org/docs/ThreadSanitizer.html
- Clang Thread Safety Analysis: https://clang.llvm.org/docs/ThreadSafetyAnalysis.html
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
