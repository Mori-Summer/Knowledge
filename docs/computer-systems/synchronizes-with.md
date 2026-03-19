---
doc_id: computer-systems-synchronizes-with
title: Synchronizes-With：并发关系图里真正跨线程连边的那一下
concept: synchronizes_with
topic: computer-systems
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-19T21:20:00+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_atomics_fences_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
  - cxx_draft_support_signal_2026_03_16
  - wg21_p3475r2_2025
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_concurrency_graph_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/happens-before.md
  - docs/computer-systems/release-sequence.md
  - docs/computer-systems/fence.md
open_questions:
  - 如何把 `synchronizes-with` 的各种来源统一压缩成一张最短检查表？
  - 在 `atomic_wait` / `notify`、fence 和库同步原语混用时，哪些边最容易被人遗漏？
---

# Synchronizes-With：并发关系图里真正跨线程连边的那一下

## 1. 这份文档要帮你学会什么

这份文档的目标，是把并发分析里最容易“听过但用不出来”的一个术语，压成一张你以后可以直接拿来画图的边。

读完后，你应该至少能做到：

- 说清 `synchronizes-with` 到底在解决什么问题
- 说清它和 `sequenced-before`、`happens-before`、`modification order` 的边界
- 知道哪些操作会真正建立 `synchronizes-with`
- 避免把“同一个原子对象上的任意访问”误当成自动同步
- 在 mutex、release/acquire、fence 场景里知道边是怎么来的

一句话先给结论：

**`synchronizes-with` 是并发关系图里的具体跨线程同步边；`happens-before` 往往就是靠这条边，再加上线程内顺序和传递性，真正长出来的。**

## 2. 一句话结论 / 问题定义

并发分析里一个根本问题是：

- 线程 A 里的某个动作，凭什么会对线程 B 的某个动作“算作先发生”？

如果没有一条明确的跨线程连边，你就只能靠直觉说：

- “我先写了”
- “它后面读到的应该就是这个”

但这种直觉不够。

`synchronizes-with` 解决的正是这个问题：

- 在哪些明确条件下，一个线程里的动作会和另一个线程里的动作建立同步边
- 这条边一旦建立，如何进一步长成 `happens-before`

## 3. 对象边界与相邻概念

### 3.1 `synchronizes-with` 管什么

它主要管：

- 哪些跨线程操作在标准模型里形成直接同步边
- 这条边如何作为 `happens-before` 的原材料

### 3.2 它不等于什么

它不等于：

- `happens-before` 整张图
- 源码顺序
- 真实时间顺序
- 同一个原子对象上的任意访问
- 任意“先 notify 后 wait”直觉

### 3.3 和几个相邻概念的边界

**`sequenced-before` vs `synchronizes-with`**

- `sequenced-before` 是线程内部顺序
- `synchronizes-with` 是跨线程边

**`synchronizes-with` vs `happens-before`**

- `synchronizes-with` 是边
- `happens-before` 是用边和线程内顺序拼出的更大关系图

所以你可以先把它理解成：

- `synchronizes-with` 是构件
- `happens-before` 是构造后的结果

**`synchronizes-with` vs release sequence**

- release sequence 是某些 release / acquire 建边时要用到的中间机制
- `synchronizes-with` 才是最终跨线程那条边本身

## 4. 核心结构

理解 `synchronizes-with`，至少要抓住下面六个构件。

### 4.1 两端都是具体评价

这条边连接的不是“两个概念块”，而是具体操作：  
某次 unlock、某次 acquire load、某次 fence、某次 signal handler 安装/触发语义等。

### 4.2 需要满足标准列出的特定条件

不是任何“读到同一个变量”都能建边。  
必须满足标准明确给出的建边条件。

### 4.3 典型来源 1：release / acquire

当前草案 `atomics.order` 直接规定：

- release 操作 A
- acquire 操作 B
- B 读取 A 或其 release sequence 中 side effect 写出的值

则 A `synchronizes-with` B。

### 4.4 典型来源 2：mutex unlock / lock

当前草案 `thread.mutex.requirements.mutex` 明确规定：

- prior `unlock()` operations on the same mutex synchronize with the later successful `lock()` or successful `try_lock()`

### 4.5 典型来源 3：fence 组合

当前草案 `atomics.fences` 还允许：

- release fence 与 acquire 操作
- release 操作与 acquire fence
- release fence 与 acquire fence

在满足特定中介原子读写条件时建立 `synchronizes-with`。

### 4.6 结果：变成 `happens-before`

当前草案 `intro.races` 给出的 `happens-before` 定义里，`A synchronizes with B` 本身就是直接构成 `A happens before B` 的一个入口。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 最直接的一条边：release store -> acquire load

```cpp
// Thread A
data = 42;
flag.store(1, std::memory_order_release);

// Thread B
if (flag.load(std::memory_order_acquire) == 1) {
    use(data);
}
```

这里真正值钱的点是：

1. A 的 release store 不是“只是写了个 1”
2. B 的 acquire load 也不是“只是读了个 1”
3. 一旦 B 读到的是 A 发布出来的值，就建立 `synchronizes-with`
4. 然后通过线程内顺序和传递性，A 之前的普通写入可以 `happens-before` B 之后的读取

### 5.2 第二条边：unlock -> lock

mutex 的真正本质，不是“阻塞别人”，而是：

1. A 在持锁区里写共享状态
2. A `unlock()`
3. B 成功 `lock()` 同一 mutex
4. 标准规定 prior `unlock()` operations synchronize with this `lock()`
5. 于是共享状态通过这条边和线程内顺序变得可安全观察

### 5.3 第三条边：fence 不是单独立法，它要借原子对象落地

很多人会误以为 fence 是“凭空建边”。  
当前草案不是这么定义的。

以 release fence -> acquire load 为例，标准要求还要存在：

- fence 之后的某个原子写 X
- acquire 读到 X 或其假想 release sequence 里的值

也就是说，fence 本身通常仍然需要借助某个 atomic carrier object 才能真正形成可用同步边。

### 5.4 条件变量里的一个现实提醒

当前草案对 condition variable 定义了：

- `wait` 分三段原子部分执行
- `notify_one` / `notify_all` 和这些部分存在一个与 `happens-before` 一致的单一总序

这说明 condition variable 当然参与整个同步故事。  
但工程上更稳的理解仍然是：

- 真正守住共享状态的是 predicate + mutex
- 不是“有个 notify 就自动万事大吉”

## 6. 关键 tradeoff 与失败模式

### 6.1 tradeoff 的本质

`synchronizes-with` 的优点是精确，缺点是苛刻。

- 好处：你能明确知道边从哪里来
- 代价：不满足条件就真的没有边，不能靠想当然补

### 6.2 常见失败模式

**失败模式 1：把同一原子对象上的任意访问都当自动同步**

不是。  
比如 relaxed 原子访问通常不会自动建立你想要的 `synchronizes-with`。

**失败模式 2：把 `synchronizes-with` 和 `happens-before` 混成一个词**

这样会丢掉最关键的信息：边是从哪里来的。

**失败模式 3：看见 acquire/release 就自动认定一定建边**

还不够。  
关键还要看 acquire 是否真的读到了对应 release 或其 release sequence 的 side effect。

**失败模式 4：把 fence 当万能屏障**

fence 很强，但标准里的 fence 建边条件并不等于“插一条 fence 就凭空同步一切”。

**失败模式 5：沿用 consume 时代的历史直觉**

如果你还把 `memory_order_consume` 相关依赖顺序魔法当现代主路径，很容易把边画错。

## 7. 应用场景

### 7.1 mutex 保护的共享状态

最常见的 `synchronizes-with` 来源之一。

### 7.2 原子发布-获取链

这是 lock-free 和轻量同步里最关键的建边方式之一。

### 7.3 fence 驱动的低层协议

在 runtime、底层容器、系统代码里，会需要显式利用 fence 组合建立同步边。

## 8. 工业 / 现实世界锚点

### 8.1 编译器和运行时原子接口

GCC `__atomic` builtins 与 LLVM atomics guide 都建立在明确同步语义之上。  
现实世界不会接受“大家靠直觉认为这里同步了”，而是需要能落到正式边条件上的接口。

### 8.2 race detector 和并发代码审查

无论是工具还是人工审查，真正值钱的问题通常都是：

- 边从哪里来？
- 这条边有没有成立？

这本质上就是在追 `synchronizes-with`。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 先找边，再谈图：先问有没有 `synchronizes-with`，再去拼 `happens-before`
- 默认优先用 mutex、release/acquire 这种边来源直接、语义稳定的路径
- 只有在底层协议真的需要时，再依赖 fence 组合建边
- 遇到复杂并发 bug 时，把“它读到了谁写的值”当成第一优先级问题

### 9.2 已经过时、明显不推荐或需要带日期理解的路径

**把 `memory_order_consume` 相关依赖顺序当现代主路径**

当前 WG21 `P3475R2` 已把 `memory_order::consume` 从主路径里移走。  
今天更稳的替代是：显式 acquire/release 或更高层同步原语。

**“同一个原子对象就天然同步了”的口头经验**

这不是当前模型的准确表达。  
当前更稳的替代是：回到标准列出的建边条件，一条一条核对。

### 9.3 什么时候别自己手搓边

如果你不是在写底层 lock-free 基础设施，通常更稳的路径是直接用：

- mutex
- condition variable
- `atomic_wait` / `notify`
- 更高层并发库

而不是手工拼复杂 fence 图。

## 10. 自测题 / 验证入口

1. `synchronizes-with` 和 `happens-before` 的区别是什么？
2. 为什么同一原子对象上的任意访问不等于自动同步？
3. release/acquire 想要建立 `synchronizes-with`，除了“看起来一边 release 一边 acquire”还缺什么？
4. 为什么说 fence 通常需要借某个 atomic carrier object 才能落成真正的同步边？
5. 为什么 mutex 的真正价值之一，是替你稳定地产生 `synchronizes-with`？
6. 为什么 consume 时代的历史直觉今天已经不适合作为主路径？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- `happens-before` 图分析
- `release sequence` 的同步回溯
- `fence` 的三种建边模式
- mutex / atomic / signal handler 的同步理解
- `atomic_wait` / `notify` 与更高层同步库的判断

迁移时最关键的问题始终是：

- 边从哪里来？
- 条件真的满足了吗？
- 我画的是“具体边”，还是已经偷换成了整张图？

## 12. 未解问题与继续深挖

后续值得继续单独拆的点包括：

- `atomic_wait` / `notify` 与更传统 `notify`/`wait` 模型的边结构对比
- fence 与 release sequence 混用时最容易遗漏的读值条件
- 库同步原语与 `synchronizes-with` 关系图的更系统分类

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `atomics.fences`: https://eel.is/c++draft/atomics.fences
- C++ draft `thread.mutex.requirements.mutex`: https://eel.is/c++draft/thread.mutex.requirements.mutex
- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- C++ draft `support.signal`: https://eel.is/c++draft/support.signal
- WG21 P3475R2, *Defang and deprecate memory_order::consume*: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3475r2.pdf
