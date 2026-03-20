---
doc_id: computer-systems-fence
title: Fence：什么时候需要一堵顺序墙，什么时候这堵墙其实并不会替你同步
concept: fence
topic: computer-systems
depth_mode: deep
created_at: '2026-03-16T14:38:05+08:00'
updated_at: '2026-03-20T15:02:04+08:00'
source_basis:
  - cxx_draft_atomics_fences_2026_03_16
  - cxx_draft_atomics_order_2026_03_16
  - cxx_draft_intro_races_2026_03_16
  - gcc_atomic_builtins_docs_2026_03_16
  - llvm_atomics_guide_2026_03_16
  - linux_kernel_memory_barriers_2026_03_16
time_context: current_practice_checked_2026_03_16
applicability: personal_concept_learning_and_low_level_sync_modeling
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/computer-systems/memory-order.md
  - docs/computer-systems/synchronizes-with.md
  - docs/computer-systems/atomic-wait-notify.md
open_questions:
  - 在哪些真实 lock-free 模式里，显式 fence 仍然比把顺序直接绑在原子操作上更值得？
  - 如何把 `atomic_thread_fence` 的三种建边模式压缩成更短的排查口诀？
  - 对现代主流 CPU 与编译器而言，哪些 fence 用法主要是在表达语义，哪些用法同时显著影响机器码成本？
---

# Fence：什么时候需要一堵顺序墙，什么时候这堵墙其实并不会替你同步

## 1. 这份文档要帮你学会什么

这份文档的目标，是把 fence 从“看起来很强的低层黑魔法”压成一个可以稳定调用的模型。

读完后，你应该至少能做到：

- 说清 fence 到底在解决什么问题
- 说清它和 op-specific memory order、`happens-before`、hardware barrier、`atomic_signal_fence` 的边界
- 知道 fence 什么时候真的能参与建立同步，什么时候只是阻止某类重排
- 知道为什么很多工程代码更推荐把顺序绑在原子操作上，而不是到处插 fence
- 避免把 fence 当“凭空同步一切”的万能墙

一句话先给结论：

**fence 是顺序与同步约束的低层原语，但它通常需要借助某个原子对象和特定读值关系才能真正建立跨线程同步；单独一条 fence 并不会自动替你发布整个世界。**

## 2. 一句话结论 / 问题定义

有时候你不想把顺序语义直接绑在某次 load/store/RMW 上，而是想说：

- “这一侧之前的操作不要越过去”
- “另一侧之后的操作不要跑回来”

这就是 fence 想解决的问题。

但 fence 也是并发里最容易被神化的对象之一。很多人会不自觉地脑补：

- 插一条 fence
- 整个程序就“同步了”

当前 C++ 草案不是这么定义它的。

更准确地说：

- fence 首先是在**同一线程内**围出一个顺序点
- 只有在满足标准规定的条件时，它才会**通过某个原子对象**参与建立跨线程 `synchronizes-with`
- 如果没有 carrier atomic object 和相应读值关系，fence 可能仍有本线程约束价值，但不会凭空制造线程间可见性

换句话说，这个概念真正要回答的是：

- fence 到底约束了什么
- 它借什么对象把约束“送到另一个线程”
- 什么情况下这堵墙根本没有连到对面

## 3. 对象边界与相邻概念

### 3.1 fence 管什么

它主要管两件事：

- 某些操作相对 fence 的顺序约束
- 在满足标准条件时，配合原子操作建立同步边

它最适合处理的不是“任何共享状态都怎么同步”，而是这种更低层的问题：

- 我想把 payload 的顺序约束绑在一个程序点上，而不是绑在某条具体原子指令上
- 我想分开“carrier atomic 的读写”和“真正需要被发布 / 获取的非原子 payload”
- 我正在写 runtime、lock-free 基础设施或非常底层的协议代码

### 3.2 它不等于什么

它不等于：

- 原子性本身
- 任何情况下都成立的 full barrier
- 单独就能建立 `happens-before`
- `atomic_signal_fence`
- 锁

如果你脑中把 fence 误画成“只要插上就自动同步全局状态”，后面大概率会写出看起来很强、实际没建边的代码。

### 3.3 和几个相邻概念的边界

**fence vs 给原子操作直接写 order**

- op-specific ordering 把语义绑在某次 load/store/RMW 上
- fence 把语义放在某个程序点上，再借外部原子交互落地

前者通常更直接。后者更灵活，但也更难推理。

**`atomic_thread_fence` vs `atomic_signal_fence`**

当前草案明确写道：

- `atomic_thread_fence` 建立线程间顺序约束，并可能对应到真实硬件/编译器层的更强屏障
- `atomic_signal_fence` 只在同一线程与其 signal handler 之间建立约束，不承担普通线程间同步职责

**fence vs `synchronizes-with`**

`synchronizes-with` 是跨线程边；fence 不是默认就拥有这条边，它必须通过标准规定的建边条件接入。

**fence vs Linux kernel barrier 直觉**

Linux barrier 文档很适合当工程直觉锚点，但它不是 C++ 语言语义本身。真正是否能跨线程同步，仍然要回到 C++ 草案里 fence 的精确建边条件。

### 3.4 本文的默认语境

本文默认主要讨论：

- `std::atomic_thread_fence`
- C++ 内存模型里的 release/acquire/acq_rel/seq_cst fence
- 线程间同步，不是 signal-only 场景

如果你切换到：

- Linux kernel barrier
- 某架构手册里的硬件屏障
- GCC/LLVM 内建原语

你应该保留结构类比，但不要偷换成“完全等价”。

## 4. 核心结构

### 4.1 一条 fence 真正要生效，通常至少要有四个构件

理解 fence，至少要抓住下面四个构件：

| 构件 | 它是什么 | 缺了会怎样 |
| --- | --- | --- |
| fence 本身 | 一个程序点上的顺序墙 | 没有明确顺序切面 |
| payload 操作 | 真正想发布或获取的普通读写/原子操作 | 墙不知道在保护什么 |
| carrier atomic object | 把两边线程接起来的原子对象 | fence 很难跨线程落地 |
| observed value 关系 | 读线程确实看到了写线程那侧相关值 | 没有 `synchronizes-with` 的抓手 |

这四个构件里，最容易被漏掉的是后两个：

- 没有 carrier atomic object
- 没有实际读到对应值

一旦缺这两个，很多 fence 代码只是在本线程里“站了个姿势”，并没有把同步送到另一个线程。

### 4.2 fence 不是一堵墙，而是一对墙加一条通道

更实用的记忆方式是：

- fence 左右两边各自约束本线程中的操作
- 真正把两边接起来的，是某个原子对象上的交互

可以把它压成下面这张图：

```text
Thread A payload
    |
    v
release side fence / operation
    |
    v
carrier atomic write/RMW  ----read-from / release-sequence---->  carrier atomic read/RMW
                                                                |
                                                                v
                                                     acquire side fence / operation
                                                                |
                                                                v
                                                       Thread B payload
```

这张图最重要，因为它告诉你：

- fence 不是跨线程直接连线
- 真正跨线程的是中间那条原子对象交互链

### 4.3 三种主要 fence 建边模式

当前草案里，最值得记住的是三种模式：

| 模式 | 直观说法 | 关键条件 |
| --- | --- | --- |
| release fence -> acquire operation | 发布侧用 fence，读取侧直接用 acquire 操作 | acquire 读到了发布侧相关写出的值 |
| release operation -> acquire fence | 发布侧直接用 release 操作，读取侧把“获取墙”放在读之后 | 读线程先读到该值，再由 acquire fence 把后续 payload 拉住 |
| release fence -> acquire fence | 两边都把顺序放在 fence 上，中间靠一对原子操作接起来 | 仍然要有 carrier object 和读值关系 |

这三种模式共同说明一件事：

**fence 能建立同步，但通常不是单枪匹马；它是在“顺序墙 + carrier atomic + observed value”这个组合里工作。**

### 4.4 两类 fence，不要混

| 名称 | 用途 | 常见误用 |
| --- | --- | --- |
| `atomic_thread_fence` | 线程间顺序与同步建模 | 被误当成“自动同步一切” |
| `atomic_signal_fence` | 同线程与 signal handler 之间的编译器级约束 | 被误拿去做普通线程间同步 |

大部分普通并发问题，只需要先考虑第一类。

### 4.5 order 参数到底代表什么

当前草案定义：

- `relaxed`: 无效果
- `acquire`: acquire fence
- `release`: release fence
- `acq_rel`: 同时是 acquire 和 release fence
- `seq_cst`: sequentially consistent acquire and release fence

这里最值得记住的不是枚举值，而是方向感：

- release 主要保护“墙前的东西不要跑到墙后”
- acquire 主要保护“墙后的东西不要跑到墙前”
- acq_rel 同时两边约束
- seq_cst 是最强的语言级 fence 形式，但仍不意味着你可以跳过 carrier atomic 和读值条件

### 4.6 一页纸判断模板

以后看到 fence 代码，先拿下面这张表过一遍：

| 问题 | 你必须回答什么 |
| --- | --- |
| 这条 fence 想保护什么 payload | 哪些普通读写或原子操作在 fence 两侧 |
| carrier atomic 是谁 | 哪个原子对象负责把两线程连起来 |
| 另一边到底看到了什么 | load/RMW 是否确实读到了相关值 |
| 这是哪种模式 | `fence->op`、`op->fence` 还是 `fence->fence` |
| 其实能不能直接用 op-specific ordering | 如果能，通常更清晰 |

## 5. 核心机制 / 主链路 / 因果链

### 5.1 同线程内的基本作用：先把墙立起来

fence 的第一层作用，始终是同线程内的程序顺序约束。

比如你可以直观地理解：

- release fence 不希望 fence 之前需要“发布”的动作被拖到 fence 之后
- acquire fence 不希望 fence 之后需要“获取后再做”的动作被跑到 fence 之前

但仅有这一步还不够。因为：

- 这只是本线程里的墙
- 还没有跨线程通路

### 5.2 模式一：release fence -> acquire operation

这是最常见也最容易误判的一种。

```cpp
// Thread A
data = 42;
std::atomic_thread_fence(std::memory_order_release);
flag.store(1, std::memory_order_relaxed);

// Thread B
if (flag.load(std::memory_order_acquire) == 1) {
    use(data);
}
```

很多人看到这里会误以为：“因为 store 是 relaxed，所以肯定不成立。”当前草案并不这么简单。

如果满足：

- release fence 在 `flag.store()` 之前
- acquire load 读到了这个 store 写出的值

那么当前草案允许这条 release fence 与该 acquire load 建同步。也就是说，顺序语义可以通过“fence + carrier atomic”落地。

这条模式最值得记住的翻译是：

**发布墙可以立在 carrier store 前面，只要读取侧真的通过 acquire 抓到了这个 carrier。**

### 5.3 模式二：release operation -> acquire fence

这次把发布语义直接绑在写侧原子操作上，把“获取墙”放到读之后。

```cpp
// Thread A
data = 42;
flag.store(1, std::memory_order_release);

// Thread B
if (flag.load(std::memory_order_relaxed) == 1) {
    std::atomic_thread_fence(std::memory_order_acquire);
    use(data);
}
```

这类代码表达的是：

- 读线程先把 carrier 值读出来
- 如果真读到了对应发布值，再用 acquire fence 把后续 payload 拉住

这条模式说明：

**获取墙不一定非要写在 load 上，也可以写在读之后，但前提是这个读确实抓到了发布侧的值。**

### 5.4 模式三：release fence -> acquire fence

这是最容易让人“完全看不懂”的一种，也是最能说明 fence 不是独立魔法的一种。

你可以把它理解成：

- 发布侧把 release 语义放到 fence 上
- 获取侧把 acquire 语义也放到 fence 上
- 中间仍然靠一对原子操作和读值关系接起来

这条模式最值得记住的不是具体语法，而是结论：

**两边都用 fence，不代表更自由；它只是把推理难度再提高了一层。**

### 5.5 为什么单独一条 fence 不会凭空同步

看下面这段：

```cpp
data = 42;
std::atomic_thread_fence(std::memory_order_release);
ready = true;  // 普通非原子写
```

如果另一个线程只是普通读 `ready`，那你不能说“有 release fence，所以数据已经安全发布”。

原因是：

- fence 的建边条件没有被满足
- 缺少 carrier atomic object
- 缺少标准要求的读值关系
- 普通非原子竞争本身就可能让程序掉进 data race 问题

这正是 fence 最容易误用的地方：**墙立了，但根本没有桥。**

### 5.6 `atomic_signal_fence` 为什么不能替你做线程间同步

当前草案明确指出：

- `atomic_signal_fence` 的排序约束只在“线程与同线程 signal handler”之间建立
- 它抑制的是相关编译器重排，不是普通线程间的同步通道

所以把它拿去做普通线程间同步，是对象边界错误，不是“技巧冷门”。

### 5.7 五步诊断链：看到 fence 代码时怎么判它到底成没成

这篇文档最重要的可调用部分，是这条五步诊断链：

1. **先找 payload。**
   这条 fence 前后真正想保护的普通读写是什么？

2. **再找 carrier atomic。**
   哪个原子对象负责把两线程接起来？

3. **再找 observed value。**
   读取侧是否真的看到了发布侧相关写出的值或其 release sequence？

4. **再判模式。**
   这是 `release fence -> acquire op`、`release op -> acquire fence`，还是 `fence -> fence`？

5. **最后反问：能不能直接用 op-specific ordering。**
   如果能，绝大多数时候会更好读、更好证对。

你只要把这五步跑通，大部分“这段 fence 代码到底有没有同步”都能拆开。

### 5.8 真正的因果核心

把这一节压成一句最值得记住的话：

**fence 先在本线程里划出顺序边界，再通过 carrier atomic 和读值关系把这条边界接到另一线程；没有 carrier，没有被观察到的值，fence 就不会自动变成跨线程同步。**

## 6. 关键 tradeoff 与失败模式

### 6.1 四组常见 tradeoff

- **灵活性 vs 可读性。**
  fence 允许把顺序语义从具体原子操作上拆出来；代价是代码读者必须把更多隐藏条件拼回去。

- **局部性能动机 vs 全局证明难度。**
  有时 fence 能让某些 carrier 操作保持 relaxed，从而改变机器码或协议结构；代价是证明正确性明显更难。

- **底层控制感 vs 团队可维护性。**
  写 fence 会让人有“我控制得更细”的感觉；但对大多数团队来说，这种控制感往往以维护成本为代价。

- **语言级精确语义 vs 硬件直觉偷换。**
  工程师容易用“barrier 很强”这类硬件直觉替代语言级建边条件；代价是代码在语言层面不一定成立。

### 6.2 六类高频失败模式

- **把 fence 当凭空同步原语。**
  这是 fence 最大的坑。当前草案里的 fence 建边都要借助具体条件。

- **明明可以直接用 release/acquire，却手搓 fence。**
  这通常会让代码更难读、更难证对。

- **把 `atomic_signal_fence` 当线程间屏障。**
  这不是它的职责范围。

- **把 acquire/release fence 想成 full barrier。**
  Linux barrier 文档本身就提醒过 acquire/release 不是万能双向墙。在 C++ 里更要回到精确语义，而不是口头印象。

- **只画 fence，不画 carrier atomic。**
  这会直接把关键同步条件藏掉。

- **继续依赖 legacy `__sync_synchronize` 或手写 inline asm barrier 直觉。**
  现代主路径应回到标准原语与编译器支持的显式原子接口。

### 6.3 现场症状与优先回查方向

| 现场症状 | 优先回查什么 | 常见真实原因 |
| --- | --- | --- |
| 代码里有 fence，但仍然偶发读到旧 payload | carrier atomic 与读值关系 | fence 存在，但没形成真正 `synchronizes-with` |
| 把 load/store 都写成 relaxed，再靠 fence 补语义 | 是否真满足三种模式之一 | “看起来像”发布-获取，实际没建边 |
| 同步代码极难解释 | 能否改回 op-specific ordering | 设计过度聪明，表达不稳定 |
| 用了 `atomic_signal_fence` 还想解释线程可见性 | 对象边界 | 选错了 fence 类型 |
| 看到 barrier 就默认“全挡住了” | release/acquire 方向性 | 用口头直觉替代语言语义 |
| 低层优化收益不明显，但代码复杂很多 | 是否根本不该用 fence | 本该用 mutex、release/acquire、`atomic_wait` |

### 6.4 为什么很多 fence 代码“看起来像对的”

fence 误用最危险的地方，在于它常常长得很像正确同步代码。

原因有三个：

- 它有“墙”的视觉隐喻，很容易让人脑补为全局屏障
- 它常与 relaxed 原子搭配，表面上看起来像“更高明的优化”
- 编译器、CPU、测试负载和具体平台可能让错误代码在大多数时候“没出事”

这也是为什么 fence 代码必须显式画出：

- payload
- carrier
- observed value path

否则它太容易骗过代码审查。

### 6.5 一个非常实用的判别句式

如果你看到一段 fence 代码：

- **只有 fence，没有原子 carrier**，那它大概率没有线程间同步意义
- **carrier 有了，但读线程不一定读到对应值**，那同步仍然可能不成立
- **能直接写成 release store / acquire load 却没这么写**，那通常值得先怀疑设计复杂度是否过高
- **开发者用“这是 full barrier”解释一切**，那你应该回到精确建边条件而不是接受口头说法

## 7. 应用场景

### 7.1 lock-free 发布协议

当顺序点和具体 carrier 原子操作分离时，fence 可能比直接给每个原子操作加 order 更方便。

典型场景是：

- payload 是一组普通写
- 某个单独原子 flag/sequence 负责“告诉别人这批 payload 准备好了”
- 你想把发布语义放在一个程序点，而不是全写进 carrier store 本身

### 7.2 runtime、内存回收和无锁基础设施

runtime、内存回收、无锁容器里，经常会用 fence 压低某些原子操作成本，或者把协议表达拆成：

- 数据准备
- 顺序墙
- carrier 发布

这类代码里 fence 的出现频率远高于普通业务代码。

### 7.3 与系统 / 硬件边界打交道

当你在更低层场景里处理可见性和重排问题时，fence 往往比普通业务代码里更常出现。

但这里要特别警惕：

- 语言级 fence
- 编译器内建 fence
- 硬件 barrier
- 内核 barrier

这些对象有结构相似性，但不该偷换成完全同义。

### 7.4 signal handler 相关顺序问题

`atomic_signal_fence` 的合理场景，是同一线程与其 signal handler 之间的排序建模，而不是一般线程协作。

这一点的价值在于：

- 让你别把 `atomic_signal_fence` 扔进普通线程同步代码
- 也让你知道它不是“没用”，只是对象边界非常窄

### 7.5 什么时候根本不该用 fence

如果你只是要安全共享状态，那么：

- mutex
- release/acquire 原子操作
- condition variable
- `atomic_wait` / `notify`

通常比手搓 fence 更稳。

这条建议本身就是 fence 最重要的应用边界之一。

## 8. 工业 / 现实世界锚点

### 8.1 C++ 草案把 fence 明确定义成可建边但条件严格的原语

`atomics.fences` 的价值不只是给出 API，而是明确告诉你：

- fence 不是默认同步全局状态
- release/acquire/acq_rel/seq_cst fence 有精确定义
- fence-based synchronization 需要满足明确的对象与读值条件

这构成了本文全部模型的语言级基础。

### 8.2 GCC `__atomic` builtins 与 LLVM atomics guide 说明 fence 不是玄学对象

GCC `__atomic` builtins 与 LLVM atomics guide 都把 fence 作为明确建模的低层原语暴露出来。这说明工业实现确实需要 fence，但不会把它当“模糊的玄学概念”。

更值得注意的是：

- 工具链愿意支持 fence
- 但它们也是把 fence 放进更大原子语义框架里，而不是鼓励“到处插墙”

### 8.3 Linux kernel memory barriers 是很好的工程直觉锚点

Linux barrier 文档持续提醒工程师：

- 屏障不是魔法
- acquire/release 不是 full barrier
- 读值关系和具体场景很重要

这对建立工程直觉非常有帮助。

但要再次强调：

- 它提供的是工程直觉锚点
- 不直接定义 C++ 语言语义

### 8.4 现实世界里 fence 主要出现在“你已经在做低层协议”的地方

从标准、编译器文档和内核 barrier 文档共同能看出的现实结论是：

- fence 不是日常业务并发的首选表达
- fence 更常出现在 runtime、lock-free、低层基础设施和平台边界
- 一旦 fence 出现在普通业务代码里，通常就值得额外审查其必要性

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-16 更推荐的实践

当前更稳的工程实践是：

- 默认优先把顺序语义直接绑在原子操作上，而不是先上 fence
- 只有在你明确知道 fence 为什么比 op-specific ordering 更合适时，才主动使用 `atomic_thread_fence`
- 把 `atomic_signal_fence` 严格局限在 signal handler 相关场景
- 一旦写 fence，就把 carrier atomic 和读值路径一起画出来
- 代码审查时，不接受“这里加了 fence 所以就安全”这种口头说明，必须能说清是哪种模式

### 9.2 过时路径、为什么旧、局限在哪、现在更推荐什么

| 过时路径 | 为什么旧 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| “插 fence 就自动同步了”的旧口头经验 | 这不是当前草案的精确表达 | 容易写出没有 `synchronizes-with` 的伪同步代码 | 回到 `atomics.fences` 的三种建边条件 |
| 明明可以直接写 release/acquire，却手搓 fence | 过度追求灵活性 | 可读性差，证明成本高 | 默认优先用 op-specific ordering |
| 把 `atomic_signal_fence` 当线程间屏障 | 对象边界错误 | 根本不负责普通线程间同步 | 线程间问题用 `atomic_thread_fence` 或更高层原语 |
| 继续把 legacy `__sync_synchronize` 当主路径 | GCC 已把 `__sync` 系列归为 legacy | 表达能力和可移植性不如现代原语 | 用标准原语或 `__atomic` builtins |
| 手写平台特定 inline asm barrier 作为默认路径 | 这是平台细节，不是默认工程接口 | 难以审计，难以迁移 | 非必要时坚持标准原语与正式编译器接口 |
| 在普通状态共享里默认上 fence | 这通常是低层协议过拟合 | 团队可维护性差 | mutex、condition variable、release/acquire、`atomic_wait` / `notify` |

### 9.3 当前更稳的使用流程

如果你怀疑某段代码需要 fence，更推荐按下面这个顺序判断：

1. 先问自己：能不能直接把顺序写在原子操作上？
2. 如果不能，再问：payload 和 carrier 是否天然分离？
3. 再问：这是三种 fence 建边模式中的哪一种？
4. 再问：读取侧是否真的保证读到相关值？
5. 最后问：这件事是不是其实该交给更高层同步原语？

这套流程能显著降低“为了显得聪明而乱用 fence”的概率。

## 10. 自测题 / 验证入口

1. `atomic_thread_fence` 和 `atomic_signal_fence` 的职责边界是什么？
2. 为什么单独一条 fence 不等于自动建立线程间同步？
3. release fence + relaxed store 为什么在满足条件时仍然可能建立同步？
4. 为什么很多时候更推荐把顺序直接绑在原子操作上，而不是分散到 fence？
5. 为什么 acquire/release fence 不能被想成万能 full barrier？
6. 什么时候你应该退回更高层同步原语，而不是继续手搓 fence？
7. 如果代码里有 release fence，但后面只跟了普通非原子写，这段代码最可能缺了什么？
8. 为什么“carrier atomic 存在”还不够，你还必须关心读线程读到了什么值？
9. `release operation -> acquire fence` 和 `release fence -> acquire operation` 的推理入口分别在哪里？
10. 请用一句话解释：fence 和 `synchronizes-with` 的关系到底是什么。

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的核心句

理解了这篇文档后，你应该能把下面这句迁移出去：

**fence 不是独立同步源，它更像一个低层顺序切面，需要借助 carrier atomic 和读值关系才能接入跨线程同步。**

### 11.2 可以迁移到哪些领域

- `synchronizes-with` 的 fence 建边模式
- `memory_order` 的 release/acquire 语义
- Linux kernel barrier 直觉
- runtime / lock-free 基础设施的低层协议
- signal handler 相关的顺序问题
- `atomic_wait` / `notify` 混合设计时的建模

### 11.3 与相邻模型的关系

| 模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| op-specific memory order | 直接把顺序绑在某次原子操作上 | 补“顺序点”和 carrier 分离时该怎么建模 |
| `synchronizes-with` | 定义跨线程边 | 补 fence 如何接入这条边 |
| Linux barrier 直觉 | 提供工程上的方向感 | 补 C++ 语言级精确条件 |
| 锁和条件变量 | 解决大多数共享状态同步 | 补只有在必须做低层协议时 fence 才有价值 |
| `atomic_signal_fence` | 处理 signal-only 顺序 | 补它为什么不该被误用成线程间屏障 |

### 11.4 最值得保留的迁移动作

以后遇到任何 fence 代码，可以先问自己：

- 这条 fence 到底在约束什么 payload？
- 它借助哪个 atomic carrier object 落地？
- 读线程是不是确实抓到了这个 carrier 上的相关值？
- 我是不是在用 fence 解决本该由更高层原语解决的问题？

只要这四问答不清，代码大概率还不够稳。

## 12. 未解问题与继续深挖

- fence 与 `atomic_wait` / `notify` 混用时的最佳建模路径是什么？
- 哪些 lock-free 模式里 fence 真的值得，而不是过度聪明？
- 平台级 barrier 与语言级 fence 之间怎样建立更稳的对应直觉？
- 对现代主流编译器和 CPU 而言，哪些 fence 用法主要是语义表达，哪些会显著改变成本结构？

## 13. 参考资料

以下链接均为本次写作时实际参考的一手资料；涉及“当前状态”的地方，均以 `2026-03-16` 为核对日期。

- C++ draft `atomics.fences`: https://eel.is/c++draft/atomics.fences
- C++ draft `atomics.order`: https://eel.is/c++draft/atomics.order
- C++ draft `intro.races`: https://eel.is/c++draft/intro.races
- GCC `__atomic` builtins: https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html
- LLVM Atomic Instructions and Concurrency Guide: https://llvm.org/docs/Atomics.html
- Linux kernel memory barriers: https://www.kernel.org/doc/html/latest/core-api/wrappers/memory-barriers.html
