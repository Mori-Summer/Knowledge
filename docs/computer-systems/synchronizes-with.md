---
doc_id: computer-systems-synchronizes-with
title: Synchronizes-With：并发关系图里真正跨线程连边的那一下
concept: synchronizes_with
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-20T16:16:02+08:00'
source_basis:
  - cxx_draft_intro_races_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_atomics_fences_2026_03_16
  - cxx_draft_thread_mutex_requirements_2026_03_16
  - cxx_draft_thread_condition_2026_03_16
  - cxx_draft_support_signal_2026_03_16
  - llvm_threadsanitizer_docs_2026_03_16
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
  - docs/computer-systems/mutex.md
  - docs/computer-systems/condition-variable.md
  - docs/computer-systems/atomic-wait-notify.md
open_questions:
  - 如何把 `synchronizes-with` 的各种来源统一压缩成一张最短检查表？
  - 在 `atomic_wait` / `notify`、fence 和库同步原语混用时，哪些边最容易被人遗漏？
  - 如果未来工具链把更多同步边直接可视化，最值得首先暴露给工程师的是哪些非直觉边？
---

# Synchronizes-With：并发关系图里真正跨线程连边的那一下

## 1. 这份文档要帮你学会什么

这份文档的目标，不是让你记住一个术语，而是让你以后分析并发 bug 时，能先问一句最有价值的话：

**“我声称这里安全，那条跨线程边到底从哪里来？”**

读完后，你应该至少能做到：

- 说清 `synchronizes-with` 为什么是并发证明里最关键的跨线程边
- 把它和 `sequenced-before`、`happens-before`、`modification order` 分层区分
- 看懂 release/acquire、release sequence、mutex、fence 各自如何落成同步边
- 避免把“大家都在访问同一个原子”“先 notify 后 wait”“我加了 fence”这类口头经验误当正式同步
- 在代码审查时把同步问题压成“边表”，而不是漂在直觉上

一句话先给结论：

**`synchronizes-with` 是标准并发模型里具体、可命名、可检查的跨线程同步边；很多你最终依赖的 `happens-before`，都要先靠它落地。**

## 2. 一句话结论 / 问题定义

并发里真正难的不是“线程 A 先做了什么”，而是：

- 线程 B 凭什么能把 A 的那次动作当成“已经发生”
- 这种“已经发生”到底靠哪条正式边支撑

如果没有明确的跨线程边，你只能说：

- “感觉它应该先写完了”
- “它后面读到的是那个值，所以大概同步了”

这两种说法都不够。  
`synchronizes-with` 解决的是：

- 哪些特定条件下，一次操作会与另一线程中的一次操作形成直接同步边
- 这条边如何再与线程内顺序一起长成 `happens-before`

## 3. 对象边界与相邻概念

### 3.1 它真正管什么

它主要管：

- 哪些跨线程操作之间，标准承认存在直接同步边
- 这条边在什么条件下成立
- 这条边能把哪些先前动作通过传递性带过去

### 3.2 它不等于什么

它不等于：

- `happens-before` 整张图
- 线程内源码顺序
- 真正墙钟时间先后
- 同一个 atomic object 上的任意访问
- 任意“先 notify 后 wait”的经验判断

### 3.3 和几个相邻概念的边界

**`sequenced-before` vs `synchronizes-with`**

- `sequenced-before` 是线程内部的先后
- `synchronizes-with` 是跨线程的一条直接边

**`synchronizes-with` vs `happens-before`**

- `synchronizes-with` 是一条具体边
- `happens-before` 是用这条边再加线程内顺序与传递性拼出来的更大关系图

最稳的理解方式是：

- `synchronizes-with` 是砖块
- `happens-before` 是砌好的墙

**`synchronizes-with` vs `modification order`**

- `modification order` 是单对象修改历史
- `synchronizes-with` 是跨线程同步边

前者回答“读到了谁”，后者回答“凭什么把先前动作一起带过去”。

**`synchronizes-with` vs `release sequence`**

- `release sequence` 是某类原子同步边成立时的桥梁
- `synchronizes-with` 才是最终真正成立的那条边

### 3.4 最危险的误判

最常见的错法是：

- 看见 `release` 和 `acquire` 关键字，就默认边已经成立
- 看见同一个 mutex / atomic / condition variable，就默认边已经存在
- 把 “我通知了” 和 “对方同步到了我之前的写” 混成一件事

这三类错法的根问题一样：  
**没有把同步证明压成“边是否成立”的检查。**

## 4. 核心结构

理解 `synchronizes-with`，最稳的方式是把它拆成五个固定槽位。

### 4.1 边的起点：一个具有发布意义的 source event

source event 常见来源包括：

- release store
- release / acq_rel 型成功 RMW
- `mutex::unlock()`
- release fence 与其后续原子写的组合

起点的价值在于：  
它把“本线程之前的动作”准备好，等待被另一线程接住。

### 4.2 边的载体：标准认可的匹配条件

不是任何 source event 都能随便连到另一边。  
还必须满足具体匹配条件，例如：

- acquire 读到了这个 release 或其 release sequence 写出的值
- later successful `lock()` 对应同一个 mutex 的 prior `unlock()`
- fence 组合里存在中介原子对象和正确的读值关系

载体条件没满足，就没有边。

### 4.3 边的终点：一个具有获取意义的 destination event

destination event 常见来源包括：

- acquire load
- acquire / acq_rel 型成功 RMW
- 成功 `lock()` / `try_lock()`
- acquire fence 与前面的原子读值条件组合

终点的作用是“接住”source event 之前准备好的那批动作。

### 4.4 边的结果：不是只同步一个值，而是带来一段历史

一条 `synchronizes-with` 成立后，真正值钱的不是“这次读到了同一个值”。  
更重要的是：

- source event 之前经线程内顺序排在它前面的动作
- 可以通过 `happens-before` 被带到 destination event 之后

所以边的价值在于“带历史”，不是“对个暗号”。

### 4.5 非边长相：很多看起来像同步，其实不是

下面这些都很像“好像同步了”，但不应被默认当边：

- relaxed load / store
- acquire 没读到对应 release 链上的值
- 普通 `notify` 但没有正确 predicate + mutex 保护
- 单独一条 fence 却没有 carrier atomic object
- plain `bool` 轮询或 `volatile` 标志

### 4.6 一张最短边表

把 `synchronizes-with` 当成边表来记，通常最稳：

- 原子边：release -> acquire，前提是 acquire 读到了对应 release / release sequence 的值
- mutex 边：prior `unlock()` -> later successful `lock()` / `try_lock()`，前提是同一个 mutex
- fence 边：release fence / release op 与 acquire op / acquire fence 的组合，前提是经由 carrier atomic object 满足标准条件

condvar、`atomic_wait` / `notify` 之类更高层原语，也最终要落回这些正式边或与之兼容的时序约束，不应只凭“API 名字像同步”就直接下结论。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 主链路：release store -> acquire load

```cpp
std::atomic<int> flag{0};
int payload = 0;

// Thread A
payload = 42;
flag.store(1, std::memory_order_release);

// Thread B
if (flag.load(std::memory_order_acquire) == 1) {
    use(payload);
}
```

这段代码真正成立的因果链是：

1. A 先写普通数据 `payload`
2. A 再对 `flag` 执行 release store
3. B 对 `flag` 执行 acquire load
4. B 读到的值来自 A 那次 release store
5. 因此 A `synchronizes-with` B
6. 再结合线程内顺序，A 在 release 之前的写可以 `happens-before` B 之后的读

注意关键点不是“一个 release、一个 acquire 写在源码里”，而是：  
**acquire 确实读到了对应的 side effect。**

### 5.2 变体链路：release sequence 让边能跨过中继 RMW

```cpp
std::atomic<int> state{0};
int payload = 0;

// Thread A
payload = 42;
state.store(1, std::memory_order_release);       // 头部 release

// Thread B
state.fetch_add(1, std::memory_order_relaxed);   // 1 -> 2，成功 RMW

// Thread C
if (state.load(std::memory_order_acquire) == 2) {
    use(payload);
}
```

这里同步边并不是：

- B 写了 `2`，所以 C 只和 B 同步

更准确的说法是：

1. A 的 release 是头部
2. B 的成功 RMW 处在这条 release sequence 上
3. C 的 acquire 读到了 `2`
4. 因此这条 acquire 可以同步回 A 的头部 release

这说明 `synchronizes-with` 的建立，有时要借助 release sequence 作为桥梁。

### 5.3 第二条主链：unlock -> lock

```cpp
std::mutex mu;
int shared = 0;

// Thread A
{
    std::lock_guard<std::mutex> lk(mu);
    shared = 42;
} // unlock

// Thread B
{
    std::lock_guard<std::mutex> lk(mu); // successful lock
    use(shared);
}
```

这里真正值钱的不是“B 被阻塞过”，而是：

1. A 在持锁区写 `shared`
2. A 对同一 mutex 执行 `unlock()`
3. B 随后成功 `lock()` 同一 mutex
4. 标准规定 prior `unlock()` synchronize with later successful `lock()`
5. 所以 `shared` 的更新通过这条边与线程内顺序被安全带到 B

mutex 的本质，不是“把别人拦住”，而是**稳定地产生这条同步边**。

### 5.4 第三条主链：fence 不能凭空立法，它要借 carrier object 落地

很多人以为：

- 只要放一条 release fence
- 另一边放一条 acquire fence
- 就自然同步了

这不对。  
标准里的 fence 建边通常仍要求：

- fence 之后或之前有具体原子操作
- 两边通过某个 atomic object 的读值关系被连起来

也就是说，fence 不是无中生有的同步魔法，仍然要“借对象落地”。

### 5.5 非例子 1：同一个 atomic object，不代表自动有边

```cpp
std::atomic<int> flag{0};
int payload = 0;

// Thread A
payload = 42;
flag.store(1, std::memory_order_relaxed);

// Thread B
if (flag.load(std::memory_order_relaxed) == 1) {
    use(payload); // 不能仅凭这段推出安全
}
```

虽然双方都碰了同一个 atomic object，但这里没有你想要的 `synchronizes-with`。  
原因是：

- relaxed 不会自动给出那条边
- 这段最多给你原子性，不自动给跨线程历史传递

### 5.6 非例子 2：notify 不是“同步完成通知”

不论是 `condition_variable::notify_one()` 还是 `atomic_notify_one()`，都不该被简单理解成：

- “我通知了，所以对方一定同步到了我之前所有写”

更稳的理解是：

- `notify` 负责让等待者有机会醒来
- 真正守住共享状态可见性的，仍然是 mutex 边、原子边或相应的读值条件

这也是为什么 condvar 正确用法始终强调 predicate + mutex，而不是只强调 notify。

### 5.7 一套边检查流程

以后分析并发代码，先按下面顺序走：

1. 先标出你声称存在同步的 source event。
2. 再标出 destination event。
3. 再问标准承认的匹配条件是什么。
4. 检查这些条件是否真的满足，而不是“看起来差不多”。
5. 只有边成立后，才继续画 `happens-before` 传递图。

这个顺序能把很多“凭感觉的同步证明”当场打断。

## 6. 关键 tradeoff 与失败模式

### 6.1 这套概念真正带来的好处

`synchronizes-with` 的价值是精确：

- 边从哪里来，能说清
- 为什么安全，能落到正式条件
- 为什么不安全，也能指出缺哪条边

这对代码审查、工具建模、并发 bug 复盘都非常关键。

### 6.2 代价：它很苛刻，不会替你脑补

代价同样明确：

- 条件没满足就真的没有边
- 语义不像 mutex 那样总是显眼
- acquire / release 关键字存在，不代表边一定成立

所以它适合拿来做严谨推理，不适合拿来做心理安慰。

### 6.3 常见失败模式

**失败模式 1：把 `synchronizes-with` 和 `happens-before` 混成一个词**

这样你会失去最重要的信息：  
到底是哪一条直接边在支撑结论。

**失败模式 2：看见 release / acquire 就自动判边成立**

关键还要看 acquire 是否真的读到了对应 release 或其 release sequence 的 side effect。

**失败模式 3：把 relaxed 原子访问也当成“反正是 atomic，所以同步了”**

这类错误在工程上极常见，而且很隐蔽。

**失败模式 4：把 fence 当万能屏障**

单独一条 fence 通常不会凭空同步一切。  
缺 carrier atomic object 或缺读值条件，就没有那条边。

**失败模式 5：把 notify 当作完整同步语义**

notify 负责唤醒机会，不等于完整状态同步证明。

**失败模式 6：沿用 `consume` 时代的依赖顺序直觉**

当前主路径已经明显转向更清晰、可实现、可审查的 acquire / release 模型。

**失败模式 7：只会画“阻塞关系”，不会画“同步边”**

一个线程被阻塞过，不等于它一定同步到了对方的先前写入。  
真正值钱的是标准承认的边，而不是“看起来有等待”。

## 7. 应用场景

### 7.1 mutex 保护的共享状态

最典型的场景。  
mutex 的工程价值，很大一部分就在于它把同步边做成了默认路径。

### 7.2 单原子发布-获取协议

像 ready flag、状态字 hand-off、一次性发布这类轻量同步，核心都在于 release/acquire 是否真的落成边。

### 7.3 lock-free 状态推进

在 CAS 或 RMW 接力里，release sequence 往往是原子边成立的桥梁。

### 7.4 fence 驱动的低层协议

runtime、内存回收、底层容器这类代码里，工程师会主动手搓 fence 组合。  
这里最需要 `synchronizes-with` 视角来防止“觉得有屏障就够了”。

### 7.5 等待 / 唤醒类库原语的正确使用

condition variable、`atomic_wait` / `notify`、平台等待原语的正确性，最终都要落回“有没有正式同步边”。

## 8. 工业 / 现实世界锚点

### 8.1 ThreadSanitizer 的 happens-before 图

ThreadSanitizer 这类工具在做 race 诊断时，本质上就在近似维护一张 happens-before 图，而这张图最关键的输入之一就是“边从哪里来”。  
换句话说，工具层虽然不一定把每条规则都直接以 `synchronizes-with` 名义展示，但它在追的正是这类边。

### 8.2 GCC / LLVM 原子语义接口

GCC `__atomic` builtins 与 LLVM 原子 IR 指令，把 release / acquire / fence / RMW 明确做成接口层语义。  
这说明工业工具链不接受“靠约定俗成认为这里同步了”，而要求同步边能落到正式语义对象上。

### 8.3 `std::mutex`、`std::condition_variable` 与 `atomic::wait`

libstdc++、libc++、MSVC STL 这些库实现虽然底层可能落到 futex、SRWLock、WaitOnAddress 等 OS 原语，但对用户暴露的正确性承诺仍然必须符合标准同步边。  
这也是为什么你最终仍应从 `synchronizes-with` 视角理解这些库，而不是停在“它会阻塞/唤醒”。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的实践是：

- 先找边，再谈图：先问 `synchronizes-with` 是否成立，再去拼 `happens-before`
- 优先选边来源直接、审查成本低的原语：mutex、清晰的 release/acquire、`atomic_wait` + 明确 carrier
- 在注释或设计文档里显式写“source / carrier / destination”
- 遇到 fence 协议时，强制补上“通过哪个 atomic object 落地”的说明

### 9.2 当前最推荐的决策顺序

1. 我需要的是阻塞、唤醒，还是可见性，还是两者都要？
2. 哪个原语能最直接地产生我要的同步边？
3. 如果边要靠读值条件成立，我是否能明确证明 read-from？
4. 如果边要靠 fence 成立，我是否真的需要这么低层？

这套顺序比“先写代码，再凭感觉补充内存序”稳得多。

### 9.3 已经过时、明显不推荐或必须带语境理解的路径

**把 `memory_order_consume` 或依赖顺序魔法当现代主路径**

当前 WG21 方向已经明显不鼓励把这条路当主模型。  
今天更稳的替代是显式 acquire / release 或更高层同步原语。

**plain bool / `volatile` 标志 + 睡眠轮询**

这类做法最大的问题不是“不优雅”，而是根本没有可靠同步边。

**“我全用 `seq_cst` 就不用管边了”**

更强的 order 不会替你补上缺失的匹配条件。  
没有正确的 source / destination / read-from 关系，`seq_cst` 也不能无中生有造边。

**把 notify 当完整同步协议**

现在更推荐的做法是：

- condvar 用 predicate + mutex
- `atomic_wait` 用同一个 atomic object 做 carrier
- 不再把“通知发生过”误当成“状态已同步”

### 9.4 替代方案

如果协议很复杂，但团队不想长期维护低层边证明，当前更稳的替代通常是：

- 直接 mutex
- condition variable
- channel / queue / task system 等更高层并发抽象

它们不是更“先进”，但往往更可维护。

## 10. 自测题 / 验证入口

1. `synchronizes-with` 和 `happens-before` 的区别是什么？为什么不能混用？
2. 为什么“同一个 atomic object 上各访问了一次”不等于自动同步？
3. 一个 acquire load 明明写在源码里，为什么仍可能接不到那条 release 边？
4. mutex 的真正价值为什么不仅是互斥，还包括稳定产生同步边？
5. 为什么单独一条 fence 往往不够，你还必须说明 carrier atomic object？
6. 为什么 notify 不能被当成完整同步证明？
7. 如果 reviewer 只会说“这里应该已经同步了”，但说不出边从哪来，最大的风险是什么？
8. 在什么情况下，你应该放弃手搓 fence / 原子边，而改用更直接的库原语？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把这套模型迁移到：

- [happens-before.md](/Users/maxwell/Knowledge/docs/computer-systems/happens-before.md) 里的整图推理
- [release-sequence.md](/Users/maxwell/Knowledge/docs/computer-systems/release-sequence.md) 里的原子边桥接机制
- [fence.md](/Users/maxwell/Knowledge/docs/computer-systems/fence.md) 里的三类 fence 建边模式
- [mutex.md](/Users/maxwell/Knowledge/docs/computer-systems/mutex.md) 与 [condition-variable.md](/Users/maxwell/Knowledge/docs/computer-systems/condition-variable.md) 的库同步理解
- [atomic-wait-notify.md](/Users/maxwell/Knowledge/docs/computer-systems/atomic-wait-notify.md) 的“值变化 + 唤醒机会”模型

迁移时最关键的动作只有一个：

- 先画具体边，再画整张图

## 12. 未解问题与继续深挖

后续值得继续深挖的点包括：

- `atomic_wait` / `notify` 与传统 condvar 模型在同步边结构上的系统对比
- fence、release sequence、mutex 这三类边在实际审查里的最短统一检查模板
- ThreadSanitizer 等工具如何把标准边近似映射成诊断图，以及它们最常见的误报 / 漏报边界
- 如果未来标准继续简化历史包袱，哪些同步边最值得被教学材料放到更靠前的位置

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `atomics.fences`: https://eel.is/c++draft/atomics.fences
- C++ draft `thread.mutex.requirements.mutex`: https://eel.is/c++draft/thread.mutex.requirements.mutex
- C++ draft `thread.condition`: https://eel.is/c++draft/thread.condition
- C++ draft `support.signal`: https://eel.is/c++draft/support.signal
- WG21 P3475R2, *Defang and deprecate memory_order::consume*: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3475r2.pdf
- LLVM ThreadSanitizer C++ Manual: https://github.com/google/sanitizers/wiki/threadsanitizercppmanual
