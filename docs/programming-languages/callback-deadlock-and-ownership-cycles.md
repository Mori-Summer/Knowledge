---
doc_id: programming-languages-callback-deadlock-and-ownership-cycles
title: 回调函数中的死锁与强引用泄漏：为什么会“自己等自己”，又为什么会“永远不释放”
concept: callback deadlock and ownership cycles
topic: programming-languages
created_at: '2026-03-18T17:07:38+08:00'
updated_at: '2026-03-18T17:07:38+08:00'
source_basis:
  - apple_dispatch_queues_2026_03_18
  - qt_threads_qobject_2026_03_18
  - qt_signals_slots_qobject_2026_03_18
  - cpp_core_guidelines_cp22_f53_f54_2026_03_18
  - cppreference_weak_ptr_2026_03_18
  - apple_working_with_blocks_2026_03_18
  - apple_practical_memory_management_2026_03_18
time_context: current_practice_checked_2026_03_18
applicability: callback_api_design_async_programming_and_lifetime_safety
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/programming-languages/callback-lifetime-management.md
  - docs/computer-systems/mutex.md
  - docs/computer-systems/condition-variable.md
open_questions:
  - 如何把 callback 的等待图与所有权图自动化成静态分析规则，提前发现“自己等自己”和引用环？
  - 在多语言混合系统里，callback 生命周期策略如何用统一契约描述，而不是靠每个框架各自的约定？
---

# 回调函数中的死锁与强引用泄漏：为什么会“自己等自己”，又为什么会“永远不释放”

## 1. 这份文档要帮你学会什么

这篇文档不是解释“什么是回调”，而是帮你建立一个能复用的排障模型，专门处理回调里最常见、也最容易被误判成“偶发玄学”的两类问题：

- 为什么回调一加同步等待、锁或跨线程阻塞，就会出现“自己等自己”式死锁
- 为什么为了避免悬空，把 `self`、`shared_ptr` 或其他强引用一把抓进去，最后对象永远不释放

读完后，你应该至少能做到：

- 用“等待图”判断一个 callback 设计会不会形成自等待、锁内重入或事件循环卡死
- 用“所有权图”判断一个 callback 捕获会不会形成强引用环
- 区分“需要暂时保活”与“形成长期引用环”这两件事，不再把强引用和泄漏画等号
- 在 Qt、Apple GCD / block、C++ `shared_ptr` 这类真实框架里识别相同结构，而不是只记各家 API 口诀

一句话先给结论：

**回调问题真正要同时看的不是一张图，而是两张图：等待图决定你会不会卡死，所有权图决定你会不会放不掉。**

## 2. 一句话结论 / 问题定义

回调把“现在继续执行”改成了“以后由别的执行上下文再调用回来”。  
一旦这样做，程序就多了两组关系：

- 谁在等谁
- 谁在持有谁

于是两个经典故障随之出现：

1. **死锁**：回调所在的线程、队列、事件循环或锁，反过来又在等待这个回调自己推动的那条路径继续前进，于是形成闭环等待。
2. **内存泄漏**：持有 callback 的对象，反过来又被 callback 强持有，形成引用计数环或长期保活链，导致释放条件永远到不了。

所以，这个概念真正解决的问题不是“回调能不能写”，而是：

> **当 continuation 被交给未来时，如何同时控制等待关系和所有权关系，不让它既卡住又放不掉。**

## 3. 对象边界与相邻概念

### 3.1 这里说的“回调中的死锁”是什么

这里关心的不是所有死锁，而是**由 callback 触发或放大的等待闭环**。  
典型形态包括：

- 回调在串行队列里 `sync` 回同一队列
- 回调持锁调用外部代码，外部代码又回进来抢同一把锁
- GUI / 事件线程里的回调阻塞等待“稍后仍需该事件线程处理”的工作
- “阻塞式跨线程回调”里，发送方和接收方其实是同一线程或同一执行资源

### 3.2 这里说的“强指针导致内存泄漏”是什么

这里关心的不是所有泄漏，而是**callback 捕获带来的强持有环**。  
典型形态包括：

- `self` 强持有 closure / block 属性，closure 又强持有 `self`
- 对象 A 持有订阅句柄或定时器，句柄/定时器持有 callback，callback 又强持有 A
- `std::shared_ptr<A>` 被存进 callback，而该 callback 又被 A 自己拥有或被 A 拥有的组件长期保存

### 3.3 它不等于什么

**不等于“所有阻塞都是死锁”**

如果回调等待的是另一条独立执行路径，而且那条路径不需要当前锁/队列/线程继续推进，就只是阻塞，不一定是死锁。  
死锁的关键不是“等了”，而是“形成闭环等待”。

**不等于“所有强捕获都是泄漏”**

如果 callback 是 one-shot、不会被 owner 长期保存，并且完成后会释放，强捕获通常只是“刻意保活”，不是泄漏。  
泄漏的关键不是“强持有”，而是“强持有环没有可达的断边”。

**不等于“用了 weak 就一定安全”**

`weak` / `weak_ptr` 只是在所有权图上断边，不会自动修复等待图；而且它也可能把“泄漏”换成“工作默默不执行”。

### 3.4 和相邻概念的边界

**回调死锁 vs 普通 mutex 死锁**

- 普通 mutex 死锁关注锁顺序
- 回调死锁更常见的是“执行器 + 锁 + 回调重入”一起形成的混合闭环

**强引用泄漏 vs 悬空引用**

- 悬空引用是对象活得太短
- 强引用泄漏是对象活得太长

它们是同一生命周期问题的两个方向，不是两个毫无关系的问题。

**回调 vs 协程 / promise**

协程和 promise 不是没有 continuation，而是把 continuation 的保存方式换掉了。  
如果你在协程里持锁跨 suspend，或者把 `Task` / future 又反向挂回 owner，同样会重现等待图和所有权图的问题。

## 4. 核心结构

要稳定分析这类问题，最少要抓住六个构件。

### 4.1 callback owner

谁在持有 callback：

- 对象属性
- 订阅句柄
- 定时器
- 事件源
- 队列 / 线程池 / 运行时

### 4.2 callback body

回调里真正执行的逻辑。  
关键不是“代码多复杂”，而是它会不会：

- 同步等待
- 再入 owner
- 申请锁
- 强持有 owner 或同级对象

### 4.3 execution resource

回调跑在哪个执行资源上：

- 串行队列
- 主线程 / UI 线程
- 某个事件循环
- 工作线程
- 某把锁保护的临界区

这是等待图的基础节点。

### 4.4 wait edge

谁在等谁。  
常见边包括：

- `dispatch_sync`
- `BlockingQueuedConnection`
- `future.get()`
- `condition_variable::wait`
- `join()`
- 同步 RPC / 同步消息发送

### 4.5 ownership edge

谁在持有谁。  
常见边包括：

- `strong` / ARC strong reference
- `std::shared_ptr`
- 成员属性
- 框架内部对 callback 的 retain / copy

### 4.6 cancel / disconnect edge

有没有可靠的断边路径：

- `disconnect`
- `cancel`
- `unsubscribe`
- one-shot 自动解除
- completion 后自动释放

没有这条边，等待图和所有权图都更容易闭环。

## 5. 核心机制 / 主链路 / 因果链

### 5.1 先用两张图看 callback

分析时先不要急着看语法，先画两张图：

```text
等待图:
线程/队列/锁 A -> callback -> 同步等待 B -> ... -> 需要 A 才能继续

所有权图:
owner A -> callback/handle -> captured object -> ... -> A
```

只要等待图成环，你就会卡住。  
只要所有权图成强环，你就会泄漏。

### 5.2 死锁主链路 1：串行执行资源上的“自己等自己”

这是最经典也最容易被忽略的一类。

```text
串行队列 Q 正在执行 callback C
callback C 同步派发/阻塞等待任务 T 在 Q 上完成
但 Q 只有 callback C 结束后才能开始 T
=> C 等 T, T 等 C 退出, 成环
```

Apple 在 `Dispatch Queues` 文档里明确要求：不要在正在该队列执行的任务里再对同一队列调用 `dispatch_sync`，因为会死锁。  
这就是最纯粹的“自己等自己”。

这个模型还可以迁移到：

- 主线程回调里同步等主线程任务
- actor / serial executor 上同步回投同一执行器
- 事件循环线程里同步等待一个仍需该事件循环处理的事件

### 5.3 死锁主链路 2：锁内调用未知 callback，回调再入同一对象

第二类不是“队列自己等自己”，而是“持锁时把控制权交给别人，别人又回来要这把锁”。

```text
owner 持有 mutex M
owner 在锁内调用 callback / virtual method / 外部 hook
callback 再次调用 owner，或访问同一受 M 保护的状态
=> 再次申请 M，形成自锁死
```

C++ Core Guidelines 的 CP.22 专门把这条规则写得很直白：  
**持锁时不要调用未知代码，例如 callback。**

这里真正危险的不是 callback 语法，而是你在**对象不变量尚未安全对外暴露时**，把控制权交了出去。

### 5.4 死锁主链路 3：阻塞式跨线程回调，但两端其实共用同一事件资源

Qt 的 `BlockingQueuedConnection` 是一个很好的现实锚点。  
Qt 官方文档明确说：如果把这种连接类型用于同一线程中的对象，会死锁。

因果链是：

```text
发送方发出信号后阻塞等待 slot 返回
slot 需要接收方线程的事件循环来调度
但发送方与接收方其实在同一线程/同一事件循环
=> 线程为了等 slot 而阻塞, slot 又需要该线程继续跑事件循环
=> 成环
```

这说明“跨线程回调”这个名字本身不重要，重要的是你是否真的拥有独立推进的执行资源。

### 5.5 强引用泄漏主链路：owner 保存 callback，callback 又反向强持有 owner

这是 callback 泄漏的标准形状：

```text
owner A 强持有 callback
callback 强捕获 A
=> A -> callback -> A
```

Apple 的内存管理文档明确指出：

- block 会强引用其捕获的对象
- 如果 `self` 也强持有这个 block，便会形成 strong reference cycle

在 C++ 里，结构完全一样，只是语法换成：

```text
A (shared_ptr-managed)
-> std::function / timer / subscription
-> lambda capture(shared_ptr<A>)
-> A
```

`std::weak_ptr` 的标准用法正是用来打断这种 `shared_ptr` 环。

### 5.6 为什么“为防悬空一律强捕获”会把问题从崩溃改成泄漏

很多工程事故不是因为不知道风险，而是因为只修了一半：

1. 一开始用裸 `this` / 引用捕获，结果 callback 到达时对象已析构，发生悬空
2. 于是改成强持有 `self` / `shared_ptr`
3. 崩溃消失了，但 callback 又被 owner 或 owner 的子对象长期保存
4. 于是形成引用环，对象再也不释放

也就是说：

- 借用策略失败时，症状通常是 UAF / crash
- 共享拥有策略失败时，症状通常是 leak / 永不析构

这不是两类独立 bug，而是同一生命周期问题沿两个方向失控。

### 5.7 为什么“全部 weak 化”也不是答案

如果你把所有 escaping callback 都机械改成 `weak self` / `weak_ptr`，会得到另一个问题：

- owner 在回调前已经消失
- callback 静默跳过
- 状态更新、取消收尾、资源释放或最终通知没有发生

所以正确问题不是“强还是弱”，而是：

> **这段工作在语义上是必须把 owner 保活到完成，还是 owner 消失后就应该自然放弃？**

只有回答了这个问题，捕获策略才有依据。

## 6. 关键 tradeoff 与失败模式

### 6.1 核心 tradeoff 1：同步确认感 vs 进度独立性

同步等待的好处是调用方推理简单：返回时结果已经到手。  
代价是你把进度耦合到了当前线程、当前队列或当前锁。

一旦回调路径还需要同一执行资源继续推进，就会形成“自己堵住自己”。

### 6.2 核心 tradeoff 2：保活能力 vs 可释放性

强引用的好处是回调执行时对象还在。  
代价是 owner 和 callback 更容易互相保活。

弱引用的好处是不会轻易形成环。  
代价是工作可能在运行前失效，需要显式处理“对象已不存在”的业务语义。

### 6.3 核心 tradeoff 3：自动化安全 vs 语义精度

框架给你的 auto-disconnect、context object、one-shot handle，能减少很多事故。  
但它们只能替你断一部分边：

- 不一定能阻止已经入队的事件继续投递
- 不一定能理解你的业务必须完成还是可以放弃
- 不一定能修复锁内重入和同步等待结构

### 6.4 常见失败模式

**失败模式 1：串行队列回调里同步回投同一队列**

这是 Apple GCD 文档直接点名的 guaranteed deadlock 场景。  
常见于“我只是想回主线程拿个值再继续”这种思路。

**失败模式 2：持锁调用 callback / virtual method / observer**

这是 CP.22 的典型反例。  
你以为在“通知别人”，其实是在把未知代码塞进临界区。

**失败模式 3：把 `BlockingQueuedConnection` 当成“方便的同步 RPC”**

它确实给你同步语义，但前提是真有另一条独立线程/事件循环在推进。  
同线程或伪跨线程情况下，它直接变成死锁制造器。

**失败模式 4：`self` 持有 closure 属性，closure 又捕获 `self`**

这在 Apple block / closure 生态里是最经典的 retain cycle。  
如果 closure 还被 timer、observer 或 subscription 长期持有，泄漏链会更长。

**失败模式 5：C++ 里拿 `shared_from_this()` 到处塞进 callback**

它能把对象保活，但如果 callback 被 owner 自己拥有的组件保存成长期监听器，就会形成 `shared_ptr` 环。  
这条路不是“现代 C++ 默认安全写法”，只是特定场景下的保活工具。

**失败模式 6：把 `recursive_mutex` 当通用补丁**

它只能缓解“同线程再次拿同一把锁”的局部问题。  
它不能修复：

- 同一串行队列的自等待
- 多把锁组成的等待环
- 锁内调用外部代码导致的不变量泄漏

而且它经常掩盖了“你把外部回调放在错误边界里调用”的设计问题。

**失败模式 7：把所有 callback 都弱捕获化**

这会把“泄漏”换成“回调静默不执行”。  
如果这段回调负责 ack、状态提交、句柄回收或关闭流程，逻辑上同样会出事故。

## 7. 应用场景

### 7.1 GUI 主线程回调

按钮事件、菜单命令、网络完成回主线程更新 UI，是最容易出现“主线程回调里再同步等主线程工作”的场景。  
这里死锁通常表现为界面冻结，而不是明显的锁栈。

### 7.2 定时器、订阅和长期监听器

只要 callback 不是 one-shot，而是长期注册：

- timer
- observer
- signal-slot
- message subscription

所有权图就必须单独画，因为这类 callback 往往比调用者本身活得更久。

### 7.3 线程池 / 执行器完成回调

后台任务完成后，常会回调 owner 更新状态。  
如果 owner 用强引用把自己钉进 callback，且回调句柄又由 owner 保存，就很容易形成“任务没完成前正常保活，任务结束后却忘了断开”的隐性泄漏。

### 7.4 C++ 网络库、消息总线和异步状态机

这类系统最容易同时踩两种坑：

- 为了同步确认，在 handler 里 `wait()` / `get()`
- 为了防止对象提前析构，在 handler 里捕获 `shared_ptr<this>`

于是既可能卡死，也可能永不析构。

## 8. 工业 / 现实世界锚点

### 8.1 Apple GCD：`dispatch_sync` 同队列死锁

Apple 官方 `Dispatch Queues` 文档明确说：

- 不要在正在该队列执行的任务里再对同一队列调用 `dispatch_sync`
- 如果要派发到当前队列，应使用 `dispatch_async`

这不是风格建议，而是队列模型直接推出来的死锁条件。  
在 iOS / macOS 应用里，最常见变体就是主线程 / 主队列上的自等待。

### 8.2 Qt：`BlockingQueuedConnection` 同线程死锁

Qt 官方 `Threads and QObjects` 文档明确说明：  
`BlockingQueuedConnection` 如果用于同一线程中的对象，会导致死锁。

这说明现实框架并不把“阻塞式回调”当成普适安全机制；它有明确的适用边界。

### 8.3 Qt：lambda 连接如果不给 context，对象生命周期仍需你自己负责

Qt 官方 `QObject::connect` / `Signals & Slots` 文档给出的当前实践非常具体：

- sender / receiver 销毁时，普通连接会自动移除
- 对 lambda 连接，最好提供 `context` 对象
- 如果不用 `context`，lambda 里用到的对象是否还活着，仍要自己保证

这正对应了所有权图模型：  
框架能替你管理连接边，但未必能替你管理 lambda 捕获边。

### 8.4 Apple block / closure：`self` 与 block 形成 retain cycle

Apple 官方内存管理文档明确说明：

- block 会强引用其捕获的变量
- 如果 block 使用 `self`，而 `self` 又强持有该 block，便会出现 strong reference cycle

这就是 iOS/macOS 工程里最常见的“回调导致对象永不释放”锚点。  
它不是 Swift/Objective-C 特例，而是引用计数系统下 callback 持有模型的直接结果。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-18 更推荐的实践

**先画等待图，再决定能不能同步等**

在 callback 里做任何同步等待前，先问三件事：

1. 这段等待需要哪条线程、哪条队列、哪把锁继续推进？
2. 当前 callback 是否正占着其中某个资源？
3. 回调链里是否可能重入当前对象？

只要答案出现闭环，就不要同步等。

**锁内只做状态变更，不做外部回调**

当前更稳的主路径是：

1. 持锁更新受保护状态
2. 如有需要，先把要通知的数据做快照
3. 退出临界区
4. 再调用 callback / observer / virtual hook

这比“锁内直接通知”更稳，因为它避免了把未知代码塞进不变量尚未封闭的区间。

**对 escaping callback，明确写出生命周期策略**

至少明确它属于哪一类：

- 借用：只允许严格局部、严格同步使用
- 强保活：保证 owner 在任务完成前必须活着
- 弱观察：owner 消失则任务应放弃
- 外部上下文托管：由 framework context / connection handle 管理

不写清楚这一点，代码审查时很难判断强弱捕获是否合理。

**长期监听优先使用 context / token / connection handle**

Qt 的 `context` 连接重载就是这类设计的现实范例。  
任何支持：

- context object
- subscription token
- scoped connection
- auto-disconnect

的框架，都应该优先使用这些显式断边机制，而不是把释放寄托在析构碰巧发生。

**C++ 中把 `weak_ptr::lock()` 当成“临时提升所有权”，而不是长期默认强持有**

对于 repeating callback / listener，更稳的写法通常是：

1. 外部只保存 `weak_ptr`
2. callback 开头 `lock()`
3. 提升成功则在本次执行期间形成局部 `shared_ptr`
4. 回调结束后局部强引用释放

这样可以把“保活窗口”缩到一次执行体内部，而不是把整个订阅周期都变成强保活。

**把同步回调链改成异步切边或显式完成通知**

如果你只是想表达“等它做完再继续”，当前更稳的替代通常是：

- completion callback
- promise / future
- coroutine / async-await
- 显式状态机

这些替代不会自动消灭生命周期问题，但它们更容易把等待边写清楚，也更容易把 one-shot completion 和长期订阅区分开。

### 9.2 已经过时、明显不推荐或必须带语境理解的路径

**过时路径 1：为了拿到同步语义，在回调里直接阻塞等结果**

这条路旧，不是因为“同步已经过时”，而是因为在事件驱动系统里，它太容易把等待边闭成环。  
它的局限是：只要结果生成仍依赖当前执行资源，这种写法就会把进度源堵死。  
当前更推荐的替代是显式 completion、future 或异步状态机。

**过时路径 2：为了防崩溃，默认把所有 owner 都强捕获**

这条路旧，是因为它把“对象活得不够长”的问题粗暴改成“对象永远死不掉”。  
它的局限是：对长期订阅、定时器、属性保存的 closure 来说，强捕获非常容易形成闭环。  
当前更推荐的替代是根据语义选择：

- 必须完成的 one-shot 工作，用受控强保活
- 可放弃的长期监听，用 `weak` / `weak_ptr` + 明确的失效分支
- 能托管给框架的，优先用 context / token

**过时路径 3：用 `recursive_mutex` 掩盖锁内回调设计问题**

它旧，不是因为再也不能用，而是因为它常被误当成“锁内重入的通用修复”。  
它的局限是：只能处理同线程对同一把锁的再次加锁，无法修复队列自等待、跨锁环和锁边界设计错误。  
当前更推荐的是缩小临界区、拆出无锁通知阶段、让 callback 在锁外发生。

**过时路径 4：不区分 one-shot completion 与 repeating subscription**

旧做法常把“完成回调”和“长期监听器”都叫 callback，然后套同一套捕获规则。  
局限在于：

- one-shot completion 更适合短期保活
- repeating subscription 更容易形成长期引用环

当前更推荐先分型，再选所有权策略。

### 9.3 一个够用的判断表

如果你的真实场景是：

- **主线程 / 串行队列回调**：默认先假设不能同步等待同一执行器
- **锁保护状态的对象回调**：默认先假设 callback 必须在锁外调用
- **长期 observer / timer / subscription**：默认先怀疑存在所有权环
- **one-shot completion**：先判断 provider 是否会在完成后立刻释放 callback，再决定是否允许短期强保活

## 10. 自测题 / 验证入口

1. 为什么“回调里阻塞了”不自动等于“死锁了”？请用等待图回答。
2. 为什么“callback 强捕获了 owner”不自动等于“泄漏了”？请用所有权图回答。
3. `dispatch_sync` 到当前串行队列，为什么本质上是“自己等自己”？
4. 为什么 CP.22 会把“持锁调用 callback”视为高风险，而不是普通代码风格问题？
5. 为什么 `weak_ptr` / `weak self` 只能打断所有权环，不能修复等待环？
6. 为什么 `recursive_mutex` 不能解决主线程回调里同步等待主线程任务的问题？
7. 给你一个定时器回调：`self.timer` 强持有 timer，timer 强持有 block，block 强持有 `self`。请你画出所有权图，并给出至少两种断边方案。
8. 给你一个 Qt 信号链：A 发信号给 B，使用 `BlockingQueuedConnection`，但 A 和 B 在同一线程。请画出等待图，说明卡死点在哪里。

## 11. 迁移与关联模型

理解这篇文档后，你应该能把模型迁移到这些相邻问题：

- `mutex` / `condition_variable`
  - 迁移重点：等待图里到底是谁持有推进条件
- observer / event bus / signal-slot
  - 迁移重点：长期订阅几乎总要额外画所有权图
- future / promise / coroutine
  - 迁移重点：continuation 虽然不再手写 callback，但等待边和生命周期边并没有消失
- actor / serial executor / UI event loop
  - 迁移重点：任何单线程串行执行模型都天然怕“自己等自己”

一个实用的迁移心法是：

> **看到 callback，就同时问“它跑在哪个资源上”和“它被谁留住”。**

前者是等待图，后者是所有权图。

## 12. 未解问题与继续深挖

后续值得继续单独拆的方向包括：

- 静态分析如何识别“锁内回调”“同执行器同步等待”和“owner <-> callback”引用环
- structured concurrency 能替 callback 消掉多少生命周期负担，哪些问题只是换皮未消失
- UI 框架、网络库、消息总线之间，是否能提炼出统一的 callback ownership contract
- 对必须完成但 owner 又可能先消失的任务，怎样设计比“全强捕获”更稳的 completion ownership 模型

## 13. 参考资料

以下资料为本次写作时实际参考的主要资料；涉及“当前推荐实践”的判断，核对日期为 `2026-03-18`。

- 仓库内方法论：
  - `docs/methodology/learning-new-things-playbook.md`
  - `docs/methodology/cognitive-modeling-playbook.md`
  - `docs/methodology/concept-document-template.md`
- 仓库内相关文档：
  - `docs/programming-languages/callback-lifetime-management.md`
  - `docs/computer-systems/mutex.md`
  - `docs/computer-systems/condition-variable.md`
- Apple, *Dispatch Queues*: https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html
- Qt, *Threads and QObjects*: https://doc.qt.io/qt-6.9/threads-qobject.html
- Qt, *QObject*: https://doc.qt.io/qt-6.8/qobject.html
- Qt, *Signals & Slots*: https://doc.qt.io/qt-6/signalsandslots.html
- C++ Core Guidelines, CP.22 / F.53 / F.54: https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
- cppreference, `std::weak_ptr`: https://en.cppreference.com/w/cpp/memory/weak_ptr.html
- Apple, *Working with Blocks*: https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html
- Apple, *Practical Memory Management*: https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html
