---
doc_id: programming-languages-coroutine
title: 协程：可挂起控制流、运行时恢复与结构化并发的统一模型
concept: coroutine
topic: programming-languages
depth_mode: deep
created_at: '2026-03-23T10:47:29+08:00'
updated_at: '2026-03-23T10:47:29+08:00'
source_basis:
  - python_asyncio_coroutines_tasks_docs_checked_2026_03_23
  - python_asyncio_overview_docs_checked_2026_03_23
  - kotlin_coroutines_basics_docs_checked_2026_03_23
  - kotlin_coroutine_context_dispatchers_docs_checked_2026_03_23
  - cpp_draft_coroutine_definition_checked_2026_03_23
  - cpp_draft_coroutine_support_library_checked_2026_03_23
time_context: current_practice_checked_2026_03_23
applicability: async_control_flow_modeling_runtime_boundary_analysis_and_language_runtime_judgment
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/programming-languages/cpp20-coroutine-playbook.md
  - docs/programming-languages/callback-lifetime-management.md
  - docs/computer-systems/thread.md
  - docs/computer-systems/process.md
open_questions:
  - 是否需要再单独补一篇“结构化并发”文档，把 scope、task group、Job 树与取消传播统一成独立概念？
  - C++ coroutine 与 executor 标准化、Python asyncio、Kotlin coroutine scope 这三条生态未来会继续收敛到哪些共同约束？
  - 协程、线程池、阻塞 API 隔离线程三层混合时，最稳的工程边界是否值得再沉淀成专门 playbook？
---

# 协程：可挂起控制流、运行时恢复与结构化并发的统一模型

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立的，不是“协程是更轻量的线程”这种常见印象，而是一套以后能反复调用的内部模型：

- 你能解释协程真正解决的是“可挂起控制流”问题，而不是直接解决“并行执行”问题
- 你能区分协程、线程、task/job、event loop、dispatcher、future 这些相邻对象
- 你能说清一次协程从启动、挂起、注册 continuation、恢复到结束的主链路
- 你能看懂为什么协程常常需要和 runtime / executor / event loop 一起建模，而不是只看语言语法
- 你能把这个模型迁移到 Python `asyncio`、Kotlin coroutines、C++20 coroutines 以及更广义的异步系统设计上

如果只记一句话：

**协程不是“替代线程”的硬件执行资源，而是把“流程可以暂停、以后还能从原地继续”这件事变成一种可组合的程序结构。**

## 2. 一句话结论 / 问题定义

**协程是可挂起、可恢复的控制流单元：它把等待、分步推进和后续恢复从回调链或手写状态机里抽离出来，让程序在不必为每条等待链都独占一个 OS 线程的前提下，仍能用接近顺序代码的方式表达并发流程。**

它真正要解决的不是：

- “怎样让代码并行跑起来”
- “怎样绕过线程”
- “怎样自动把阻塞 API 变成非阻塞”

它真正要解决的是：

- 当流程中途需要等待时，怎样暂停当前控制流
- 等待条件满足后，怎样把流程恢复回来
- 谁负责保存状态、谁负责恢复、恢复到哪个执行上下文
- 怎样把大量等待主导的流程收束回可维护、可取消、可组合的结构

## 3. 对象边界与相邻概念

### 3.1 这篇文档直接处理什么

这篇文档直接处理的是：

- 协程作为“可挂起控制流”的抽象本体
- 协程的状态保存、挂起点、恢复点、continuation、调度边界
- 协程与 task/job、dispatcher / executor / event loop 的关系
- 协程在不同语言生态中的共同结构

### 3.2 它不等于什么

它不等于：

- **线程。** 线程是 OS 调度的执行资源；协程是用户态控制流结构。
- **scheduler / event loop / dispatcher。** 协程可以挂起和恢复，但谁来安排它何时恢复、在哪恢复，通常是 runtime 的责任。
- **异步 IO 本身。** 协程能把等待写得更像顺序代码，但不会把阻塞系统调用神奇变成非阻塞。
- **task / job / future。** 这些往往是 runtime 用来承载协程执行、结果、取消和父子关系的对象，不是协程本体。
- **生成器。** 生成器与协程都能暂停和恢复，但生成器更偏“逐步产出值”，协程更偏“等待并继续”。

### 3.3 最值得一起看的相邻概念

最值得一起看的相邻概念是：

- [C++20 协程：可挂起控制流、语言机制与运行时边界](/Users/maxwell/Knowledge/docs/programming-languages/cpp20-coroutine-playbook.md)
- [回调函数：显式 continuation、异步边界与变量生命周期管理](/Users/maxwell/Knowledge/docs/programming-languages/callback-lifetime-management.md)
- [线程：共享地址空间里的调度执行流，不是更轻量的进程，也不是协程](/Users/maxwell/Knowledge/docs/computer-systems/thread.md)
- future / promise / task / job
- event loop / dispatcher / executor
- 结构化并发与取消传播

### 3.4 本文的默认适用边界

本文默认工作边界是：

- 以现代主流语言中的协程模型为主
- 重点看 coroutine 作为通用抽象，而不是局限在某一种语言语法
- 用 Python `asyncio`、Kotlin coroutines 和 C++20 coroutines 做现实锚点

涉及 Python 3.14 文档、Kotlin 当前 coroutine 文档和 C++ draft 的内容，核对日期均为 `2026-03-23`。

### 3.5 六组最容易混淆的边界

最常见的混淆有六组：

- **协程 vs 线程。**
  协程解决控制流挂起恢复；线程解决 OS 调度和并行执行。

- **协程 vs task/job。**
  协程是可挂起的流程；task/job 往往是 runtime 包裹它的执行管理对象。

- **协程 vs event loop / dispatcher。**
  协程自己不决定何时恢复、在哪个线程恢复。

- **协程 vs 非阻塞 IO。**
  协程只改善表达；底层若还是阻塞调用，线程仍会被卡住。

- **协程 vs future/promise。**
  future/promise 更偏结果通道；协程更偏流程结构。

- **协程 vs 生成器。**
  两者都能暂停，但生成器主要解决“下一步产出什么”，协程主要解决“什么时候能继续做后续工作”。

## 4. 核心结构

### 4.1 最稳的模型：协程是“流程状态 + 挂起协议 + 恢复协议”

理解协程，最稳的方式不是先记某种语法，而是先记下面这张结构表：

| 结构件 | 它解决什么 | 典型形态 |
| --- | --- | --- |
| coroutine body | 真正的业务流程本体 | `async def`、`suspend fun`、含 `co_await` 的函数 |
| coroutine state / frame | 挂起后状态放哪里 | coroutine object、frame、captured locals |
| suspension point | 什么时刻能停下来 | `await` / `co_await` / suspend call |
| continuation / handle | 以后从哪里继续 | Task、Job、`coroutine_handle`、continuation |
| runtime scheduler | 谁决定何时恢复、在哪恢复 | event loop、dispatcher、executor |
| scope / parent-child relation | 生命周期、取消和收尾如何组织 | TaskGroup、CoroutineScope / Job tree |
| underlying execution resource | 最后到底在哪执行 | 一个或多个 OS threads |

如果你缺了后两层，就会把协程误认为“语言自己会调度自己”的魔法。

### 4.2 五个最关键的问题

分析任何 coroutine 系统时，最值得先问的是五个问题：

1. 挂起时，状态保存在什么对象里
2. 什么事件或 awaitable 会触发恢复
3. 恢复由谁触发，是 event loop、dispatcher 还是显式 resumer
4. 恢复到哪个线程 / 执行上下文，有没有线程亲和性保证
5. 协程结束、取消、异常时，谁负责收尾和父子传播

很多“协程很神秘”的感觉，本质上就是这五个问题没被拆开。

### 4.3 三层分工：语言层、运行时层、执行资源层

协程主题最容易讲乱，是因为它横跨三层：

| 层 | 负责什么 | 不负责什么 |
| --- | --- | --- |
| 语言层 | 提供 suspend/await/coroutine 语义、状态机或协议 | 不自动提供完整调度器 |
| 运行时层 | 管 task/job、scope、取消、调度、恢复 | 不替你改变底层阻塞 API 本质 |
| 执行资源层 | 真正提供 CPU、线程、事件循环驱动 | 不自动提供好用的上层表达 |

这三层在不同生态里的切法不同：

- Python `asyncio`：语言给 `async`/`await`，`asyncio` 给 Task 和 event loop
- Kotlin：语言有 `suspend`，`kotlinx.coroutines` 给 scope、dispatcher、Job
- C++20：语言和标准库给 coroutine 变换与 `<coroutine>` 支持，runtime / executor 常由库或工程自行补足

### 4.4 四个最该记住的判断变量

1. 问题是否等待主导，而不是 CPU 主导
2. 协程是否依赖特定调度上下文
3. 是否需要结构化取消与父子生命周期
4. 是否会跨 suspend point 携带锁、线程局部状态或线程亲和对象

如果这四个变量没先定清，协程设计极容易失控。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 一条完整主链路：从进入协程到挂起、恢复、结束

一次典型协程执行可以压成下面这条链：

1. 调用方创建或进入一个 coroutine。
2. 运行时 / 语言层为它建立状态对象，保存局部变量、继续点和必要上下文。
3. 协程开始执行，直到遇到某个 suspension point。
4. suspension point 会把“后续从哪里继续”注册到某个 awaitable / event source / continuation 上。
5. 当前执行资源返回给 runtime；如果设计正确，这一步意味着线程可以去做别的工作，而不是被阻塞。
6. 当等待条件满足，runtime 决定何时恢复该协程。
7. 恢复可能发生在原线程，也可能在别的线程或 executor 上，取决于具体 runtime 语义。
8. 协程继续执行，直到再次挂起、抛异常、被取消或正常结束。
9. 结束时，结果、异常、取消状态和资源清理会沿 task/job/scope 边界传播。

这条链真正解释的是：

- 协程不是“后台偷偷跑着”
- 它是一条被 runtime 驱动、可反复挂起恢复的控制流

### 5.2 Python `asyncio` 变体链：coroutine object -> Task -> event loop

Python 官方 `asyncio` 文档给出的关键事实非常直接：

- `async def` 声明的 coroutine 是推荐写法
- 仅仅调用一个 coroutine function 只会产生 coroutine object，并不会自动运行
- 要么 `await` 它，要么把它包装成 `Task` 才会被调度
- event loop 使用 cooperative scheduling，一次只运行一个 Task；当 Task await 某个 Future 时，loop 才有机会去跑别的 Task
- `TaskGroup` 是比裸 `create_task()` 更现代的做法，文档明确写它提供了更强的 safety guarantees

这条链说明：

- 协程不是线程
- 协程对象也不是已经在运行的任务
- 真正跑起来的是“被 event loop 驱动的 task 化协程”

### 5.3 Kotlin 变体链：suspend function -> dispatcher -> Job tree

Kotlin 官方文档给出的关键事实也很清楚：

- coroutine 是 suspendable computation
- coroutine 可以并发运行，甚至在某些条件下并行，但底层仍运行在 OS 管理的 threads 上
- coroutine 可以 suspend its execution instead of blocking a thread
- coroutine 不绑定单一线程，可以在一个线程挂起、在另一个线程恢复
- coroutine context 里的 dispatcher 决定它使用哪个线程或线程池
- parent coroutine always waits for the completion of all its children

这条链说明 Kotlin 的协程模型里，scope / Job 树和 dispatcher 不是附属品，而是协程语义的一部分。

### 5.4 C++20 变体链：coroutine transform -> frame -> handle

C++ draft 和 `<coroutine>` 支持库给出的关键事实是：

- 语言会把 coroutine function 按规则变换成带 `initial_suspend` / `final_suspend`、promise 和状态存储的 replacement body
- 实现可能为 coroutine state 分配额外存储
- `<coroutine>` 里的 `coroutine_handle` 可以引用 suspended or executing coroutine
- resuming a coroutine on a different execution agent 可能带来实现定义行为或线程身份风险；草案明确提醒，不应跨 suspend point 假定一致线程身份，例如 holding a mutex object across a suspend point

所以 C++20 的关键边界是：

- 语言负责“怎样变成 coroutine”
- 但“由谁恢复、在哪恢复、怎样和 executor 集成”仍经常是运行时 / 库设计问题

### 5.5 为什么协程能省线程，但不能省掉设计

协程常被看成“线程省钱工具”，这只说对了一半。

更完整的因果链是：

1. 如果工作负载主要是等待主导，就会有大量“线程只是在等”的时间。
2. 协程把等待中的流程状态搬进用户态状态对象，而不是长期占住一个内核线程栈。
3. runtime 能在同一批线程上轮转更多等待主导任务。
4. 于是线程数量、上下文切换和阻塞浪费下降。
5. 但这要求你显式设计 suspend points、恢复上下文、取消传播和 scope。

所以：

- 协程买到的是更高效的等待链表达和线程利用率
- 不是“省掉并发建模”本身

## 6. 关键 tradeoff 与失败模式

### 6.1 四个核心 tradeoff

| tradeoff | 你买到什么 | 你付出什么 |
| --- | --- | --- |
| 顺序式表达 vs runtime 复杂度 | 等待链更线性、更易读 | 必须理解 task/job/dispatcher/event loop |
| 少线程占用 vs 更强上下文约束 | 大量等待主导流程可复用线程 | 恢复线程、线程亲和性、thread-local 更复杂 |
| 结构化并发 vs 生命周期约束 | 更稳的取消、收尾与父子关系 | 需要显式 scope，不适合随意 fire-and-forget |
| 更便宜的等待 vs 不适合 CPU 密集阻塞 | IO/等待场景收益大 | CPU 重任务和阻塞调用仍需线程或专用执行资源 |

### 6.2 七类高频失败模式

1. **把协程当线程。**
   这样会误判并行能力、调度粒度和隔离边界。

2. **以为写成 `async` / `await` 就自动非阻塞。**
   如果底层还是阻塞 API，线程照样会被卡住。

3. **忽略恢复上下文。**
   Kotlin 文档和 C++ draft 都清楚表明，恢复线程不一定等于挂起线程。

4. **跨 suspend point 持有锁、线程亲和对象或 thread-local 假设。**
   这会把“流程恢复”问题变成真实同步 bug。

5. **把 coroutine object 当已运行任务。**
   Python 官方文档明确表明，单纯调用 coroutine function 不会自动执行。

6. **无 scope 地 fire-and-forget。**
   这会让取消、异常传播、结束等待和清理点失控。

7. **用协程解决本质上是 CPU 并行的问题。**
   协程很擅长等待主导流程，不天然擅长吞掉 CPU 重任务。

## 7. 应用场景

### 7.1 IO 主导的网络与服务端流程

这是协程最典型的高价值场景：

- 网络请求链
- 数据库访问链
- RPC fan-out / fan-in
- 大量短等待、少量真实计算的服务端逻辑

这里协程最值钱的是：

- 把等待链写回接近顺序结构
- 让大量等待任务复用少量线程

### 7.2 UI、游戏脚本与多帧工作流

当流程天然需要：

- 等下一帧
- 等用户动作
- 等动画完成
- 等某个异步状态返回后继续

协程能把“暂停后回来”写得比回调和手搓状态机清楚得多。

### 7.3 结构化子任务编排

当一个上层任务需要：

- 派生多个子任务
- 等它们一起结束
- 出错时成组取消
- 生命周期跟着父级 scope 走

协程 + structured concurrency 就比裸 future / callback 更稳。

### 7.4 生成器、流式处理与分阶段推进

尽管生成器与协程不完全等价，但在很多“分阶段推进、阶段间可暂停”的问题里，两者共享一部分建模方法。  
理解协程后，很多流式 pipeline、lazy sequence 和 incremental workflow 也会更容易看懂。

## 8. 工业 / 现实世界锚点

### 8.1 Python `asyncio`：协程、Task 与 event loop 的最经典教材级现实对象

它重要在于：

- 官方文档把 coroutine object、Task、Future、TaskGroup 和 event loop 的边界讲得非常清楚
- 明确展示了协程不是“调用就跑”，而是要被 await 或被 task 化后由 loop 驱动
- 明确给出了 structured concurrency 的现代入口 `TaskGroup`

### 8.2 Kotlin coroutines：把 suspend、dispatcher、scope、Job 树做成统一工程模型

它重要在于：

- 它把 coroutine 从“语法技巧”推进成“语言 + runtime + lifecycle”的完整系统
- 官方文档明确写出 coroutine 可以 suspend on one thread and resume on another
- parent-child 关系、dispatcher、context 不再是附属约定，而是正面 API

### 8.3 C++20 coroutines：把语言层协议和运行时层边界分得最清楚

它重要在于：

- 标准明确规定了 frame / promise / handle / suspend 点的语言级结构
- 同时也暴露了“语言支持 != 完整 runtime”的事实
- 这让它特别适合作为理解 coroutine 本体与运行时边界的锚点

仓库里已有的 [cpp20-coroutine-playbook.md](/Users/maxwell/Knowledge/docs/programming-languages/cpp20-coroutine-playbook.md) 就是这个特化方向的展开版。

## 9. 当前推荐实践、过时路径与替代

本节涉及 Python 3.14 `asyncio`、Kotlin 官方 coroutine 文档和当前 C++ draft 的内容，核对日期均为 `2026-03-23`。  
下面“更推荐什么”的部分，除直接来自官方文档的事实外，其余是基于这些语义做的工程推断。

### 9.1 三条过时路径

#### 过时路径 1：把协程当“更轻量的线程”

局限：

- 它会掩盖 coroutine 和 thread 的层级差异
- 会把 scheduler / event loop / dispatcher 的责任看丢
- 会错误地把并行能力归因给 coroutine 本体

更稳替代：

- 把协程理解成“可挂起控制流 + runtime 恢复协议”

#### 过时路径 2：深层回调链和手写状态机作为默认异步表达

局限：

- 控制流碎裂
- 生命周期、错误传播和取消更难统一
- continuation 藏在闭包和局部状态里，不利于全局推理

更稳替代：

- 在等待主导流程里优先使用协程或等价的 async/await 结构

#### 过时路径 3：无 scope 的裸 fire-and-forget 协程

局限：

- 异常传播不清
- 取消不清
- 结束等待不清
- 父子关系和清理点不清

更稳替代：

- 优先 structured concurrency
- Python 里优先理解 `TaskGroup`
- Kotlin 里优先理解 scope / Job 树

### 9.2 截至 2026-03-23 的更稳基线

更稳的工程基线通常是：

1. **把协程用于等待主导流程，而不是拿来硬套 CPU 密集任务。**
2. **把协程和运行时一起建模。**
   没有 task/job/dispatcher/event loop 的 coroutine 文档通常都不够稳。
3. **优先结构化并发。**
   Python 官方文档已把 `TaskGroup` 明确写成更现代的替代；Kotlin 官方文档明确强调 parent waits for children。
4. **把恢复上下文当接口合同。**
   不要假设挂起前后一定在同一线程。
5. **把取消和清理设计成一等语义。**
   Python 官方文档明确提醒 cancellation 与 structured concurrency 组件强相关，不应随意吞掉取消异常。

### 9.3 何时更像是在找协程，何时其实该用线程

这是工程推断，但通常足够稳：

- **更像是在找协程：**
  当你主要痛的是等待链表达、回调碎裂、需要大量挂起恢复，而不是纯粹的 CPU 并行。

- **其实该用线程：**
  当你要处理阻塞 API 隔离、线程亲和资源、CPU 并行计算，或者底层 runtime 根本没有合适 coroutine 调度器。

最稳的边界句是：

- **线程解决“谁在 CPU 上跑”**
- **协程解决“流程如何暂停并继续”**

### 9.4 C++20 的额外纪律

截至 `2026-03-23`，基于 C++ draft 更稳的纪律是：

- 不把 C++20 coroutine 当完整 async runtime
- 显式设计恢复线程 / executor 合同
- 不跨 suspend point 默认假设一致线程身份
- 不把锁、thread-affine 对象和“永远同线程”的假设偷偷塞进 coroutine 代码

## 10. 自测题 / 验证入口

1. 为什么说协程真正解决的是“可挂起控制流”，而不是“并行执行”？
2. 协程、线程、task/job、event loop / dispatcher 分别承担什么职责？
3. 为什么“把函数写成 `async` / `await`”并不能自动把阻塞 IO 变成非阻塞？
4. 为什么 Python 里仅仅调用 coroutine function 并不会让它开始运行？
5. 为什么结构化并发会比裸 fire-and-forget 更稳？它究竟解决的是哪一层问题？
6. 为什么跨 suspend point 继续假设同一线程、继续持有锁，往往会出事？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- Python `asyncio`、Kotlin coroutines、C++20 coroutines 的共性与差异
- 回调、future/promise、task/job、generator 之间的边界判断
- 线程池、event loop、dispatcher 的分工设计
- 结构化并发、取消传播、超时控制和资源清理
- UI、网络服务、脚本流程、流式计算中的暂停/恢复建模

## 12. 未解问题与继续深挖

1. structured concurrency 是否值得在仓库里升级成独立概念，而不是继续分散在 coroutine、task、scope 文档里？
2. 协程恢复上下文、线程亲和和 thread-local 数据传递，是否需要单独做一篇“协程恢复语义与陷阱”文档？
3. 在 mixed model 系统里，怎样更系统地决定一段逻辑该落在线程、协程，还是回调 / 状态机上？

## 13. 参考资料

以下“当前实践”相关内容的核对日期均为 `2026-03-23`。

- Python `asyncio` overview: https://docs.python.org/3/library/asyncio.html
- Python coroutines and tasks: https://docs.python.org/3.14/library/asyncio-task.html
- Kotlin coroutines basics: https://kotlinlang.org/docs/coroutines-basics.html
- Kotlin coroutine context and dispatchers: https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html
- C++ draft, coroutine definitions: https://eel.is/c++draft/dcl.fct.def.coroutine
- C++ draft, `<coroutine>` support library: https://eel.is/c++draft/support.coroutine
