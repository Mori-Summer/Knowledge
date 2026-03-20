---
doc_id: programming-languages-cpp20-coroutine
title: C++20 协程：可挂起控制流、语言机制与运行时边界
concept: cpp20_coroutine
topic: programming-languages
depth_mode: deep
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-03-20T17:09:33+08:00'
source_basis:
  - cpp_draft_coroutines_checked_2026_03_19
  - cpp_draft_coroutine_support_library_checked_2026_03_19
  - boost_asio_cpp20_coroutines_docs_checked_2026_03_19
  - lewissbaker_cppcoro_repository_checked_2026_03_19
time_context: current_practice_checked_2026_03_19
applicability: async_control_flow_modeling_runtime_design_and_cpp_engineering_judgment
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/programming-languages/callback-lifetime-management.md
open_questions:
  - 是否需要补充 coroutine 与 executor、io_uring 或网络运行时结合的工业案例？
  - 是否需要单列 cancellation、lifetime 与 allocator 的失败模式？
---

# C++20 协程：可挂起控制流、语言机制与运行时边界

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立一套“协程到底在解决什么，语言负责什么，运行时又负责什么”的内部模型，而不是把 `co_await` 记成一个更时髦的异步语法。

读完后，你应该至少能做到：

- 解释协程为什么本质上是在管理“可挂起、可恢复的控制流”
- 区分协程、线程、回调、Future、调度器和异步 IO 之间的边界
- 说清 C++20 里 `promise_type`、coroutine frame、`coroutine_handle` 和 awaiter 各自承担什么角色
- 判断一个工程问题该不该上协程，以及协程真正依赖什么样的 runtime / executor
- 识别生命周期、恢复线程、取消、异常传播和阻塞 API 误用这几类高频失败模式

## 2. 一句话结论 / 问题定义

**协程是把“中途需要等待、以后还要从原地继续”的流程压缩回一个可组合的程序结构；在 C++20 里，这通过编译器把包含 `co_await` / `co_yield` / `co_return` 的函数转换成状态机，并把状态保存在 coroutine frame 中实现。**

它真正解决的问题不是“怎么并行计算”，而是：

- 如何在不阻塞线程的前提下表达等待链
- 如何把回调和手写状态机里的“暂停/恢复协议”收束回可维护的代码结构
- 如何让语言层、返回对象、awaiter 协议和运行时分工清楚

## 3. 对象边界与相邻概念

协程不是：

- 线程：线程是执行资源，协程是控制流结构
- 调度器：协程能挂起和恢复，但谁来恢复通常由 event loop / executor / runtime 决定
- 异步 IO 本身：如果底层还是阻塞 API，协程语法不会替你消掉阻塞
- 性能银弹：它经常改善吞吐和可维护性，但会引入 frame、间接恢复和生命周期复杂度

最该放在一起比较的相邻概念是：

- 回调与 continuation
- Future / Promise 链
- 生成器 / 迭代器
- runtime / executor / event loop

## 4. 核心结构

把 C++20 协程压成最小结构，可以只记 6 个部件：

1. 协程函数入口
2. 保存状态的 coroutine frame
3. 暴露协议的 `promise_type`
4. 可恢复 / 销毁的 `std::coroutine_handle`
5. 决定挂起与恢复行为的 awaiter / awaitable
6. 选择何时、何地继续执行的 runtime / executor

它们缺一不可：只有语言机制没有运行时，协程只是“可暂停”；只有运行时没有语言机制，程序员就得继续手搓状态机和回调链。

## 5. 核心机制 / 主链路 / 因果链

一条最值得记住的主链路是：

1. 协程函数被调用后，编译器生成的状态机先创建 coroutine frame
2. 返回对象和 `promise_type` 建立起外部观察与内部状态之间的通道
3. 执行遇到 `co_await` 时，awaiter 决定是继续、挂起，还是把恢复动作交给外部运行时
4. 运行时在某个时刻、某个线程或某个执行上下文中恢复 `coroutine_handle`
5. 协程走到 `co_return`、异常路径或 `final_suspend`，最后由拥有 handle 的一方负责销毁

最关键的因果点有两个：

- 协程把“分散在回调和闭包里的未来执行路径”收束成一段看起来接近顺序的代码
- 协程的正确性不只取决于语言语义，还取决于 runtime 是否明确了恢复线程、取消协议、生命周期和异常通道

## 6. 关键 tradeoff 与失败模式

协程的收益是控制流重新线性化、等待链更易读、状态保存从业务逻辑中剥离；代价是 frame、间接恢复、调试复杂度和更多生命周期边界。

最常见的失败模式是：

- 把协程误当成异步 IO，自以为“写成 `co_await` 就不阻塞了”
- 不清楚恢复线程，导致 UI 线程、线程亲和对象或锁语境出错
- handle、引用或对象所有权不清楚，造成悬空、泄漏或重复恢复
- 只有挂起语法，没有取消、超时和异常传播协议
- 对所有函数一股脑协程化，结果引入不必要复杂度

## 7. 应用场景

协程最适合的不是所有代码，而是“等待主导”的流程：

- 高并发网络服务的请求链
- UI / 游戏主循环中的跨事件恢复
- 生成器式数据流
- 需要把 async pipeline 重新写回接近顺序逻辑的系统

## 8. 工业 / 现实世界锚点

### 8.1 C++ 标准协程语义

C++ 草案明确把 coroutine 作为语言机制与支持库协议来定义：函数何时被视为 coroutine、`promise_type` 怎么被发现、`std::coroutine_handle` 如何接入，都是标准层的真实约束，不是某个框架自创语法。

### 8.2 Boost.Asio 的 `awaitable` / `co_spawn`

Boost.Asio 的 C++20 coroutine 支持把“协程不等于调度器”这件事落成了真实工程接口：

- `awaitable` 作为返回对象承接挂起流程
- `co_spawn` 把协程挂到 executor / I/O context 上运行
- per-operation cancellation 等能力说明，真正工业可用的协程从来不只靠语言关键字

### 8.3 `cppcoro` 与工程实现路径

`cppcoro` 不是标准，但它是现代 C++ coroutine 生态里非常真实的实现锚点：它把 task、generator、async sequence 等模式沉淀成可复用库形态，帮助理解“语言提供的是协议，工程上还需要返回对象与 runtime 设计”。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-19 更推荐的实践

当前更稳的工程做法通常是：

- 只在真正等待主导的流程上使用协程
- 把协程放在明确的 runtime / executor / event loop 之上
- 显式设计取消、超时、异常传播和资源释放协议
- 把“恢复线程 / 恢复上下文”当成接口合同，而不是默认假设

### 9.2 过时路径与替代

下面这些路径并非完全失效，但通常已不是更稳的主路径：

- 深层回调链：问题不在能不能工作，而在控制流碎裂、错误传播困难
- 大量手写状态机：可控但维护成本高
- “把阻塞 API 包进协程函数就等于异步”：这是最常见误判

更稳的替代是：

- 用 executor 驱动的 awaitable / task 模型代替裸回调链
- 用明确 runtime 契约取代“协程自己会调度自己”的幻想
- 在不需要挂起的地方继续使用普通函数、回调或同步代码

## 10. 自测题 / 验证入口

1. 为什么协程解决的核心问题是“等待链表达”，而不是“并行计算”？
2. `promise_type`、coroutine frame、`coroutine_handle` 和 awaiter 分别负责什么？
3. 为什么 `co_await` 一个阻塞 API 不能自动让它变成非阻塞？
4. 为什么说协程的正确性很大程度上取决于恢复线程和生命周期合同？
5. 什么情况下你应该坚持回调、普通函数或线程，而不是把问题强行改写成协程？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- Rust `async/await`
- Python `asyncio`
- C# `async/await`
- generator / iterator 风格控制流
- 任何“把暂停/恢复协议压缩回顺序表达”的异步框架

## 12. 未解问题与继续深挖

- C++ coroutine 与 executor 标准化、`io_uring`、网络运行时之间的边界还会怎样继续收敛？
- cancellation、allocator、lifetime 三者叠加后，哪些模式最适合沉淀成团队级规范？
- `final_suspend`、销毁责任和跨线程恢复语境怎样做成更不易误用的库接口？

## 13. 参考资料

以下“当前实践”相关内容的核对日期均为 `2026-03-19`。

- C++ draft, coroutine function definition: https://eel.is/c++draft/dcl.fct.def.coroutine
- C++ draft, coroutine support library: https://eel.is/c++draft/support.coroutine
- Boost.Asio, C++20 Coroutines overview: https://www.boost.org/doc/libs/1_88_0/doc/html/boost_asio/overview/composition/cpp20_coroutines.html
- Lewis Baker, `cppcoro` repository: https://github.com/lewissbaker/cppcoro

## 14. 详细展开与原有分析笔记

### 1. 文档目的

很多人第一次接触协程时，会把它理解成下面几种东西之一：

- 更轻量的线程
- 一种更好看的异步写法
- `Future/Promise` 的语法糖
- 某个网络库或运行时的专属机制

这些说法都不完全错，但都不够准。  
它们抓到的是协程的某个外部表现，而不是它的核心结构。

这份文档不打算把协程写成百科词条，也不打算上来就堆 `promise_type`、`co_await`、`coroutine_handle` 的 API。  
它想回答的是更底层的几个问题：

- 协程到底试图解决什么问题
- 协程到底是什么，不是什么
- 为什么说协程本质上是在管理“可中断的控制流”
- C++20 协程是如何把这种抽象落到语言机制上的
- 在真实工程里，协程的边界、取舍和失败模式是什么

一句话概括：  
**协程不是为了让代码看起来高级，而是为了把“会被暂停和继续的流程”变成一种可表达、可管理、可组合的程序结构。**

---

### 2. 协程要解决的真实问题

#### 2.1 问题不在“计算”，而在“等待”

很多函数真正麻烦的地方，不是算不出来，而是中间需要等待：

- 等网络响应
- 等磁盘 IO
- 等定时器触发
- 等锁
- 等某个外部事件完成

如果把这类流程写成普通同步函数，最直接的写法通常是：

```cpp
Response handle_request() {
    auto user = read_user_from_db();
    auto profile = call_profile_service(user.id);
    return build_response(user, profile);
}
```

这个写法的优点是顺序清楚。  
但如果 `read_user_from_db()` 和 `call_profile_service()` 都会阻塞当前线程，那么线程在等待期间其实什么都没做。

问题于是出现了：

- 逻辑上你只是在等
- 资源上你却占着一个线程

当并发量上来时，这会很贵。

#### 2.2 传统方案一：线程

一种直觉解法是“多开线程”。  
这样每个请求都可以用同步代码写，等待期间让别的线程去做事。

但线程方案有明显代价：

- 线程不是免费的，栈空间和调度开销都要付钱
- 线程数太多会带来上下文切换成本
- 线程同步本身会引入新的复杂度
- 很多时候你真正想表达的是“流程暂停”，不是“开一个新的并行执行单元”

所以，线程解决的是“并行执行单元”的问题，不是“如何优雅表达中途等待”的问题。

#### 2.3 传统方案二：回调

另一个常见方案是回调：

```cpp
void handle_request(Request req, Callback<Response> done) {
    read_user_from_db(req.user_id, [done](User user) {
        call_profile_service(user.id, [done, user](Profile profile) {
            done(build_response(user, profile));
        });
    });
}
```

这解决了阻塞线程的问题，但引入了另一类成本：

- 原本顺序的业务逻辑被拆碎
- 局部变量的生命周期变复杂
- 错误处理会分散到多个 continuation 里
- 代码结构不再跟真实业务顺序一致

本质上，你把“暂停后继续执行”的控制流，手工编码成了“把下一步包装成函数对象交给别人以后再叫我”。

#### 2.4 传统方案三：手写状态机

如果再往底层看，回调背后其实是在做一件事：

- 保存当前状态
- 记录下一步该从哪里继续
- 等外部事件完成后恢复执行

这其实就是一个状态机。  
你当然可以自己写：

```cpp
switch (state) {
case 0:
    start_db_read();
    state = 1;
    return;
case 1:
    user = db_result;
    start_profile_call();
    state = 2;
    return;
case 2:
    profile = rpc_result;
    finish(build_response(user, profile));
    return;
}
```

它能工作，但可读性和维护成本都很差。  
因为你把“业务流程”降级成了“手工管理程序计数器和局部状态”。

#### 2.5 协程给出的核心承诺

协程想解决的不是“如何并行计算”，而是下面这个更具体的问题：

> 当一个函数中间需要等待时，能不能在不阻塞线程的前提下，仍然把代码写成接近顺序逻辑的样子？

协程给出的答案是：

- 可以暂停函数
- 可以保留函数现场
- 可以在未来某个时刻从暂停点继续执行
- 这些状态保存与恢复工作，尽量交给语言和编译器，而不是交给程序员手工拼回调和状态机

所以，协程的核心价值不是“更快”，而是：

- 更自然地表达会中断的流程
- 把状态管理从业务逻辑里剥出去
- 让异步控制流重新接近顺序思维

---

### 3. 先给一句话定义

协程可以先粗略定义为：

> **协程是一种可以在执行过程中挂起、稍后再恢复的函数或控制流抽象。**

如果换成 C++20 语境，可以再精确一点：

> **C++20 协程是编译器把包含 `co_await`、`co_yield`、`co_return` 的函数转换成状态机，并把其局部状态保存在 coroutine frame 中的一套语言机制。**

这个定义里有四个关键点：

- 它首先是“函数/控制流”层面的抽象
- 它支持“挂起”和“恢复”
- 它依赖“状态保存”
- 它在 C++20 里是“语言机制 + 库协议”的组合，不只是一个库技巧

---

### 4. 协程不是什么

理解协程，最容易出错的地方不是“没记住定义”，而是把它和邻近概念混掉。

#### 4.1 协程不是线程

线程是操作系统或运行时调度的执行单元。  
协程是程序内部的一段可暂停、可恢复的控制流。

线程关心的是：

- 谁在执行
- 在哪个 CPU 时间片上执行
- 如何与别的线程并行

协程关心的是：

- 当前流程是否要暂停
- 暂停时要保留什么状态
- 将来从哪里继续

一个线程里可以跑很多协程。  
一个协程也可能在不同时间点被不同线程恢复。  
所以两者不在同一抽象层。

#### 4.2 协程不等于异步

“异步”描述的是调用双方的时序关系：发起和完成不在同一时刻。  
“协程”描述的是控制流的表达与恢复机制。

你可以：

- 用协程写异步代码
- 用协程写生成器
- 在协程里做同步阻塞操作

最后一种虽然通常不推荐，但它说明一件事：  
**协程本身不自动让阻塞消失。**

如果你在协程体里直接调用阻塞式 IO，那么当前线程还是会被阻塞。  
协程只是给你一种“可挂起”的表达能力，不会凭空把同步 API 变成非阻塞 API。

#### 4.3 协程不是调度器

协程可以暂停和恢复，但“什么时候恢复、由谁恢复、在哪个线程恢复”通常不由协程语言机制本身决定。

这通常由外部系统决定，例如：

- 事件循环
- 线程池
- IO 完成通知
- 定时器系统
- 自定义执行器

因此，协程的边界要看清：

- 语言机制负责保存和恢复控制流
- 运行时系统负责决定何时推进它

#### 4.4 协程不是性能银弹

协程常常能改善吞吐、降低线程数、让代码更易维护。  
但这不代表它天然更快。

协程也有成本：

- 需要 coroutine frame
- 可能发生动态分配
- 控制流变成间接恢复
- 调试栈可能不如普通函数直观
- 生命周期和所有权处理更微妙

所以协程优化的是“控制流组织方式”，不是无条件优化所有性能指标。

---

### 5. 用第一性原理拆解协程

如果把协程外面的语法糖剥开，底层其实只剩几件事。

#### 5.1 底层事实

一个可中断流程要成立，至少要满足以下事实：

- 它不能每次都从头开始
- 它暂停时必须保存当前位置
- 它暂停时必须保存以后还要用到的状态
- 外部必须有办法重新拿到它并恢复它
- 恢复后必须能继续产出结果、错误或下一个值

换句话说，协程的核心并不是 `co_await` 这个关键字，  
而是：

- 程序计数器要保存
- 局部状态要保存
- continuation 要能被重新触达

#### 5.2 如果没有协程，这些工作谁来做

如果语言不提供协程，你还是得做同样的工作，只是换成更原始的形式：

- 回调用闭包保存一部分状态
- 手写状态机保存步骤号
- Future/Promise 链把 continuation 包起来
- 事件注册系统负责以后再调用你

因此，协程不是凭空创造了一种全新能力。  
它做的是：

> **把本来分散在回调、闭包、状态机、调度器之间的“暂停/恢复协议”收束成一套统一表达。**

#### 5.3 协程的必要条件

如果站在机制层看，一个协程系统至少要有下面几个部分：

- 一个保存状态的存储体
- 一个表示“这段流程以后还能继续”的句柄
- 一套描述“现在能不能继续、不能的话如何挂起、恢复后返回什么”的协议
- 一个把结果或异常交给外部的通道

在 C++20 里，这几部分大致对应：

- coroutine frame
- `std::coroutine_handle`
- awaiter 协议
- `promise_type`

#### 5.4 协程的本质压缩

协程最核心的压缩，可以写成一句话：

> **把“分散在多个回调函数和状态变量里的未来执行路径”，压缩回一个看起来像普通函数的结构里。**

这也是为什么协程最大的收益，往往不是局部性能，而是：

- 控制流重新线性化
- 错误处理回到接近同步代码的结构
- 局部变量不必被手工拆散到闭包里
- 业务逻辑和状态保存职责重新分离

---

### 6. 用系统思维看协程

如果只记一句“协程是可暂停函数”，很容易还是学不稳。  
因为你仍然不知道它的边界、构成和失败模式。

下面用九个维度把协程拆开。

#### 6.1 目的

协程的目的不是并行，而是：

- 表达会等待的流程
- 避免阻塞式写法浪费线程
- 避免回调和手写状态机打散业务逻辑

#### 6.2 边界

协程通常负责：

- 暂停当前函数
- 保存函数状态
- 恢复执行
- 产出结果、异常或序列值

协程通常不负责：

- 让阻塞 API 自动变非阻塞
- 决定调度策略
- 决定线程绑定
- 决定网络库、磁盘库如何完成 IO

#### 6.3 构成

一个典型的 C++20 协程系统中，常见的关键对象包括：

- 协程函数本身
- coroutine frame
- `promise_type`
- `std::coroutine_handle<>`
- awaitable / awaiter
- 调用方
- 恢复协程的外部事件源或执行器

#### 6.4 关系

它们之间的关系可以先记成下面这张图：

```text
调用方
  |
  v
协程返回对象 <--> coroutine_handle <--> coroutine frame <--> promise_type
                                              |
                                              v
                                           awaiter
                                              |
                                              v
                                    外部事件源 / 执行器 / 调度器
```

其中：

- `handle` 是操作 frame 的把手
- `promise_type` 是协程和外部交互的定制点
- awaiter 决定这次等待是否挂起、挂起时怎么登记 continuation、恢复时返回什么

#### 6.5 机制

协程的机制主链路可以概括为：

1. 创建协程并分配 frame  
2. 根据 `initial_suspend` 决定是否立即开始执行  
3. 执行到 `co_await` / `co_yield` / `co_return`  
4. 在挂起点保存状态  
5. 把恢复权交给外部  
6. 以后某个时刻通过 `handle.resume()` 继续  
7. 执行完毕后进入 `final_suspend`  
8. 最终由拥有者销毁 frame

#### 6.6 约束

协程成立有几个典型约束：

- 需要精确处理生命周期
- 恢复时机必须正确
- 挂起点前后的对象有效性必须可证明
- 异常传播路径必须明确
- 如果恢复发生在别的线程，线程安全要自己保证

#### 6.7 取舍

协程换来的不是纯收益，而是结构性取舍：

- 用更自然的业务表达，换更复杂的底层协议
- 用更少的线程阻塞，换更细致的生命周期管理
- 用更线性的代码，换更不直观的底层控制流跳转

#### 6.8 演化

为什么现代语言和库越来越重视协程？  
因为很多真实系统越来越依赖大量“等待型工作”：

- 网络服务
- GUI 事件处理
- 游戏脚本与帧逻辑
- 流式数据处理
- 异步资源加载

这些场景的核心问题不是 CPU 算不过来，而是中间有大量等待与恢复。  
协程正好击中这个结构。

#### 6.9 失败模式

协程最常见的失败模式包括：

- 把协程误当线程，错误推断并发行为
- 在协程里做阻塞操作，结果线程照样卡住
- 误判对象生命周期，导致悬空引用
- handle 泄漏，没有 `destroy()`
- 在已经完成或已销毁的协程上继续恢复
- 不清楚恢复线程是谁，导致线程安全问题

---

### 7. 协程与邻近概念的区别

理解边界最快的方法之一，就是做对比。

#### 7.1 协程 vs 线程

- 线程是执行资源，协程是控制流结构
- 线程由 OS 或运行时调度，协程通常由用户态逻辑恢复
- 线程天然并发，协程天然只表示“可以中断并继续”
- 线程通常有独立栈，协程通常有自己的 frame

最常见误判是：

> “我用了协程，所以它会自动并行。”

不对。  
协程是否并行，取决于你把它恢复到哪里、由谁恢复、是否有多线程执行器参与。

#### 7.2 协程 vs 回调

- 回调把未来步骤显式写成函数对象
- 协程把未来步骤隐式保存在状态机和 frame 中
- 回调暴露 continuation 结构
- 协程尽量把 continuation 隐藏在顺序代码背后

所以可以把协程理解成：

> 回调风格的一种更强表达形式，但底层仍然离不开 continuation 思维。

#### 7.3 协程 vs Future

`Future` 关注的是“未来会有一个结果”。  
协程关注的是“这段流程如何暂停并恢复”。

两者经常配合，但不是一回事：

- Future 更像结果占位符
- 协程更像控制流容器

很多库会让 `Future` 变成一个 awaitable，于是你可以 `co_await future`。  
这时：

- Future 提供“结果何时就绪”的语义
- 协程提供“等待期间如何挂起、就绪后如何恢复”的语义

#### 7.4 协程 vs 生成器

生成器通常是协程的一种特化用法。  
它的重点不是“等某个异步结果”，而是“每次产出一个值，然后暂停，等下次再继续”。

所以：

- `Task<T>` 型协程偏向“一次性完成，最后给一个结果”
- `Generator<T>` 型协程偏向“多次挂起，多次给出中间值”

---

### 8. C++20 协程的语言模型

到了 C++20，协程不再只是库技巧，而是语言级支持。  
但它不是“只要写个关键字就行”的黑盒，背后有一组明确角色。

#### 8.1 什么函数会变成协程

在 C++20 里，一个函数只要在函数体内使用了下面任意一个关键字：

- `co_await`
- `co_yield`
- `co_return`

它就会被视为协程函数。

注意：

- 普通 `return` 不是协程关键点
- 一个函数签名长得像任务函数，不代表它就是协程
- 是否成为协程，取决于函数体里是否用了协程关键字

#### 8.2 三个关键字分别在做什么

#### `co_await`

等待一个 awaitable 对象。  
如果对象还没准备好，当前协程可以挂起；准备好后恢复，并拿到结果。

#### `co_yield`

向外产出一个中间值，然后挂起。  
它常见于生成器场景。

从概念上看，`co_yield value` 可以近似理解为：

```cpp
co_await promise.yield_value(value);
```

#### `co_return`

结束协程，并把最终结果通过 `promise_type` 交给外部。

#### 8.3 几个关键角色

#### 1. coroutine frame

可以把它理解成“这次协程调用的持久化现场”。  
里面通常放着：

- 参数副本
- 仍然活着的局部变量
- 当前执行到哪一步的状态
- `promise_type`
- 某些 awaiter 临时对象

普通函数退出后栈帧就没了。  
协程之所以能暂停后继续，是因为这个现场没有随着一次函数返回而消失。

#### 2. `promise_type`

这是协程返回类型对应的定制点。  
标准层面上，编译器通过 `std::coroutine_traits<R, Args...>::promise_type` 找到它；工程里最常见的写法是把它作为返回类型里的嵌套类型。

它不是 `std::promise`。  
名字像，但职责不同。

它通常负责：

- 提供返回对象 `get_return_object()`
- 决定协程开始时是否先挂起 `initial_suspend()`
- 决定结束时是否挂起 `final_suspend()`
- 接收 `co_return` 结果 `return_value()` / `return_void()`
- 接收未处理异常 `unhandled_exception()`

可以把它看成：

> 协程与外部世界之间的协议中枢。

#### 3. `std::coroutine_handle<>`

它是指向 coroutine frame 的轻量句柄。  
常见能力包括：

- `resume()`
- `destroy()`
- `done()`
- `promise()`

如果说 frame 是“现场本体”，那么 handle 就是“可操作这个现场的把手”。

#### 4. awaitable / awaiter

`co_await expr` 并不是“随便等一个对象”。  
这个对象必须提供一套等待协议。

概念上，最终会落到一个 awaiter，上面最关键的方法是：

- `await_ready()`
- `await_suspend(handle)`
- `await_resume()`

它们分别回答：

- 现在结果是不是已经好了
- 如果没好，当前协程该怎么挂起
- 恢复之后，把什么值交回给协程

需要补一句准确性说明：  
标准语义里，`await_suspend` 不一定非得返回 `void`，也可以返回 `bool` 或另一个 `coroutine_handle`。  
但如果你现在的目标是先建立核心模型，那么先把它理解成“挂起时把 continuation 交出去的钩子”就够了。

---

### 9. C++20 协程是如何运行的

这一节最重要。  
如果这部分模型不稳，后面所有代码都会显得像魔法。

#### 9.1 一个协程调用的大致生命周期

先看主链路：

```text
调用协程函数
  ->
创建 coroutine frame
  ->
构造 promise_type
  ->
get_return_object()
  ->
initial_suspend
  ->
执行协程体
  ->
co_await / co_yield / co_return
  ->
final_suspend
  ->
destroy
```

下面逐步解释。

#### 9.2 第一步：调用协程函数

当你写：

```cpp
Task t = answer();
```

这看起来像一次普通函数调用。  
但如果 `answer()` 是协程，编译器不会把它当作普通函数栈帧来处理，而是进入协程构造流程。

#### 9.3 第二步：创建 coroutine frame

编译器会为这次协程调用准备一块持久化状态区。  
概念上它包含：

- 当前状态编号
- 局部变量
- 参数
- promise
- 暂停点所需临时对象

这块状态区通常不在普通调用栈上长期存活。  
因此协程可以“函数先返回，状态还留着”。

#### 9.4 第三步：生成返回对象

`promise_type::get_return_object()` 会生成外部拿到的返回对象。  
这个返回对象常见形式有：

- `Task<T>`
- `Generator<T>`
- 某个库自定义的 async type

它通常内部会持有 `coroutine_handle<promise_type>`。

#### 9.5 第四步：看 `initial_suspend`

`initial_suspend()` 决定协程在刚创建完时是否立刻进入函数体。

常见两种策略：

- `std::suspend_never`：立即开始执行，称为 eager
- `std::suspend_always`：先挂起，等外部显式 `resume()`，称为 lazy

这点非常重要。  
因为它直接决定“调用协程函数之后，函数体到底有没有马上跑起来”。

#### 9.6 第五步：执行到挂起点

当协程体执行到 `co_await` 时，事情大致分三步：

1. 看看结果是不是已经准备好  
2. 如果没准备好，保存状态并挂起  
3. 以后恢复时再继续往下执行

这三步对应 awaiter 协议。

#### `await_ready()`

如果返回 `true`，说明不需要挂起，直接拿结果继续。  
如果返回 `false`，说明需要走挂起流程。

#### `await_suspend(handle)`

它在真的要挂起时被调用。  
最重要的事情通常是：

- 拿到当前协程的 handle
- 把这个 handle 注册到外部事件源
- 让外部在适当时机恢复它

从概念上看，这一步就是“把未来继续执行的权力交出去”。

#### `await_resume()`

当协程以后恢复时，会调用它，把等待结果交回协程体。

#### 9.7 第六步：执行到 `co_return`

当协程执行到 `co_return expr` 时，通常会：

- 调用 `promise.return_value(expr)` 或 `promise.return_void()`
- 记录最终结果
- 然后进入 `final_suspend`

注意：  
进入 `final_suspend` 不等于 frame 已经被销毁。  
很多实现会让协程在结束时再挂起一次，等拥有者决定何时 `destroy()`。

#### 9.8 第七步：异常处理

如果协程体里抛出异常且没有在内部处理，通常会走到：

- `promise.unhandled_exception()`

然后再进入结束流程。  
这意味着：

- 协程异常不是自动“消失”
- 你需要在返回对象里定义好如何把异常传播给外部

#### 9.9 第八步：销毁

协程最终必须被销毁。  
否则 frame 会泄漏。

常见做法是：

- 返回对象析构时调用 `handle.destroy()`
- 或者由框架在确定协程不再需要时释放

如果你只记一个工程事实，那就是：

> **协程不是普通函数返回后就自动万事大吉，它背后通常还有一块需要正确回收的状态。**

---

### 10. 编译器到底帮你做了什么

协程之所以常被说成“编译器帮你写状态机”，是因为从概念上，它确实接近下面这种转换。

假设原始代码是：

```cpp
Task<int> foo() {
    int a = co_await read_a();
    int b = co_await read_b();
    co_return a + b;
}
```

可以把它想成被“近似”改写成下面的伪代码：

```cpp
struct FooFrame {
    int state = 0;
    int a = 0;
    int b = 0;
    promise_type promise;
    Awaiter awaiter_a;
    Awaiter awaiter_b;
};

void resume(FooFrame& frame) {
    switch (frame.state) {
    case 0:
        frame.awaiter_a = make_awaiter(read_a());
        if (!frame.awaiter_a.await_ready()) {
            frame.state = 1;
            frame.awaiter_a.await_suspend(handle_of(frame));
            return;
        }
        frame.a = frame.awaiter_a.await_resume();

    case 1:
        frame.awaiter_b = make_awaiter(read_b());
        if (!frame.awaiter_b.await_ready()) {
            frame.state = 2;
            frame.awaiter_b.await_suspend(handle_of(frame));
            return;
        }
        frame.b = frame.awaiter_b.await_resume();

    case 2:
        frame.promise.return_value(frame.a + frame.b);
        goto final_suspend;
    }

final_suspend:
    // 等待外部销毁 frame
    return;
}
```

这段伪代码不是标准精确展开，但它能帮助你抓住三个核心事实：

- 协程真的很像状态机
- 局部变量会进入 frame，而不是只活在瞬时栈上
- `co_await` 的关键不是“神秘等待”，而是“决定是否挂起，并登记以后如何恢复”

---

### 11. 最小示例一：一个只返回一次结果的 `Task`

先看一个最小但完整的例子。  
它的目的不是教你做生产级任务库，而是把几个关键角色串起来。

这里的 `Task`，以及后面出现的 `Generator`，都是为了教学手写的最小类型。  
标准库提供的是协程底层原语，不直接提供一个通用现成的 `Task` 高层封装。

```cpp
#include <coroutine>
#include <exception>
#include <optional>
#include <utility>

class Task {
public:
    struct promise_type;
    using handle_type = std::coroutine_handle<promise_type>;

    explicit Task(handle_type h) : handle(h) {}

    Task(Task&& other) noexcept
        : handle(std::exchange(other.handle, {})) {}

    Task(const Task&) = delete;
    Task& operator=(const Task&) = delete;

    ~Task() {
        if (handle) {
            handle.destroy();
        }
    }

    bool resume() {
        if (!handle || handle.done()) {
            return false;
        }
        handle.resume();
        return !handle.done();
    }

    int result() {
        if (handle.promise().error) {
            std::rethrow_exception(handle.promise().error);
        }
        return *handle.promise().value;
    }

private:
    handle_type handle;

public:
    struct promise_type {
        std::optional<int> value;
        std::exception_ptr error;

        Task get_return_object() {
            return Task{handle_type::from_promise(*this)};
        }

        std::suspend_always initial_suspend() noexcept {
            return {};
        }

        std::suspend_always final_suspend() noexcept {
            return {};
        }

        void return_value(int v) noexcept {
            value = v;
        }

        void unhandled_exception() noexcept {
            error = std::current_exception();
        }
    };
};

Task answer() {
    co_return 42;
}
```

#### 11.1 这个例子里发生了什么

当执行：

```cpp
Task t = answer();
```

会发生的关键事情是：

1. 协程 frame 被创建  
2. `promise_type` 被构造  
3. `get_return_object()` 生成 `Task`  
4. 因为 `initial_suspend()` 返回 `std::suspend_always`，所以函数体此时还没真正跑  

也就是说，`answer()` 这个调用完成后，你拿到的是一个“尚未开始执行的协程对象”。

接着如果执行：

```cpp
t.resume();
```

协程才开始真正跑进函数体，执行到 `co_return 42`：

- `promise.return_value(42)` 被调用
- 协程进入 `final_suspend`
- `handle.done()` 这时会变成 `true`

最后：

```cpp
int v = t.result();
```

你从 `promise` 中取回最终结果。

#### 11.2 这个例子揭示了什么机制

这个最小例子暴露了协程的几个关键事实：

- 调用协程函数，得到的不一定是“立刻算出来的结果”，而可能是“控制这段流程的对象”
- `initial_suspend` 决定协程是 eager 还是 lazy
- `co_return` 不等于普通 `return`
- `promise_type` 是结果和异常的存放位置之一
- `Task` 往往只是对 `coroutine_handle` 的 RAII 封装

#### 11.3 为什么这里 `final_suspend` 也是 `suspend_always`

因为我们希望协程在结束后，frame 仍然保留一下，  
让外部还能安全读取结果，然后再由 `Task` 析构时统一 `destroy()`。

这正好说明：

- 协程“逻辑完成”
- frame “物理销毁”

这两件事不是同一个时刻。

---

### 12. 最小示例二：`co_await` 一个自定义 awaiter

上一个例子说明了 `co_return` 和生命周期。  
下面这个例子说明“挂起和恢复到底是谁决定的”。

```cpp
#include <coroutine>
#include <exception>
#include <optional>
#include <utility>

class Task {
public:
    struct promise_type;
    using handle_type = std::coroutine_handle<promise_type>;

    explicit Task(handle_type h) : handle(h) {}

    Task(Task&& other) noexcept
        : handle(std::exchange(other.handle, {})) {}

    Task(const Task&) = delete;
    Task& operator=(const Task&) = delete;

    ~Task() {
        if (handle) {
            handle.destroy();
        }
    }

    bool resume() {
        if (!handle || handle.done()) {
            return false;
        }
        handle.resume();
        return !handle.done();
    }

    int result() {
        if (handle.promise().error) {
            std::rethrow_exception(handle.promise().error);
        }
        return *handle.promise().value;
    }

private:
    handle_type handle;

public:
    struct promise_type {
        std::optional<int> value;
        std::exception_ptr error;

        Task get_return_object() {
            return Task{handle_type::from_promise(*this)};
        }

        std::suspend_always initial_suspend() noexcept {
            return {};
        }

        std::suspend_always final_suspend() noexcept {
            return {};
        }

        void return_value(int v) noexcept {
            value = v;
        }

        void unhandled_exception() noexcept {
            error = std::current_exception();
        }
    };
};

struct ManualEvent {
    bool signaled = false;
    std::coroutine_handle<> waiter{};

    bool await_ready() const noexcept {
        return signaled;
    }

    void await_suspend(std::coroutine_handle<> h) noexcept {
        waiter = h;
    }

    void await_resume() const noexcept {}

    void set() noexcept {
        signaled = true;
        if (waiter) {
            auto h = waiter;
            waiter = {};
            h.resume();
        }
    }
};

Task wait_then_return(ManualEvent& ev) {
    co_await ev;
    co_return 7;
}
```

#### 12.1 这个例子的时间线

假设这样使用：

```cpp
ManualEvent ev;
Task t = wait_then_return(ev);
```

此时因为 `initial_suspend = suspend_always`，协程体还没开始。

第一次：

```cpp
t.resume();
```

协程开始执行，遇到：

```cpp
co_await ev;
```

然后发生以下事情：

1. `await_ready()` 被调用  
2. 因为 `signaled == false`，返回 `false`  
3. 于是进入 `await_suspend(current_handle)`  
4. `ManualEvent` 把当前协程的 handle 记下来  
5. 当前协程挂起，控制权回到外部

以后某个时刻：

```cpp
ev.set();
```

这时：

1. `signaled = true`  
2. `set()` 内部拿出之前保存的 handle  
3. 调用 `h.resume()`  
4. 协程从 `co_await ev` 之后继续执行  
5. 进入 `co_return 7`

#### 12.2 这个例子揭示了什么机制

这个例子把协程最重要的边界讲清楚了：

- `co_await` 不是“神奇地等待”
- awaiter 负责定义等待协议
- `await_suspend` 里常常会把 continuation 交给外部系统
- 真正让协程继续跑起来的，往往是外部事件源

也就是说：

> 协程负责“我可以停在这里，以后再继续”，  
> awaiter 和外部系统负责“什么时候再叫你继续”。

#### 12.3 这也解释了线程问题

如果 `ev.set()` 在另一个线程里被调用，那么协程就会在那个线程里被恢复。  
这说明：

- 协程恢复在哪个线程，不是语法自动保证的
- 线程亲和性和同步策略，需要运行时或库自己定义

---

### 13. `co_yield` 与生成器：协程不一定只返回一次

很多人把协程只理解成 async/await。  
这不完整。  
协程还很适合做生成器。

看一个最小的 `Generator<int>`：

```cpp
#include <coroutine>
#include <exception>
#include <utility>

template <typename T>
class Generator {
public:
    struct promise_type;
    using handle_type = std::coroutine_handle<promise_type>;

    explicit Generator(handle_type h) : handle(h) {}

    Generator(Generator&& other) noexcept
        : handle(std::exchange(other.handle, {})) {}

    Generator(const Generator&) = delete;
    Generator& operator=(const Generator&) = delete;

    ~Generator() {
        if (handle) {
            handle.destroy();
        }
    }

    bool next() {
        if (!handle || handle.done()) {
            return false;
        }
        handle.resume();
        return !handle.done();
    }

    const T& value() const {
        return handle.promise().current_value;
    }

private:
    handle_type handle;

public:
    struct promise_type {
        T current_value{};

        Generator get_return_object() {
            return Generator{handle_type::from_promise(*this)};
        }

        std::suspend_always initial_suspend() noexcept {
            return {};
        }

        std::suspend_always final_suspend() noexcept {
            return {};
        }

        std::suspend_always yield_value(T value) noexcept {
            current_value = std::move(value);
            return {};
        }

        void return_void() noexcept {}

        void unhandled_exception() {
            std::terminate();
        }
    };
};

Generator<int> numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}
```

如果这样用：

```cpp
auto g = numbers();
while (g.next()) {
    auto v = g.value();
    // v 会依次是 10, 20, 30
}
```

其工作方式是：

- 每个 `co_yield` 把一个值写进 `promise.current_value`
- 然后协程挂起
- 调用方下次再 `resume()`，协程继续往后跑

这说明协程并不天然等于“最后给一个值”。  
它还可以表达：

- 多次产出
- 每次产出后暂停
- 下次需要时再继续

所以生成器本质上也是协程，只是目标函数不同：

- async task 关注“最终完成”
- generator 关注“逐步产出”

---

### 14. 最容易误解的几个点

#### 14.1 “用了协程就不会阻塞线程”

错。  
协程只是在语言层面允许你挂起。  
如果你调用的是阻塞函数，它照样会阻塞当前线程。

#### 14.2 “协程就是更轻量的线程”

不准确。  
协程没有自动并发语义。  
它首先是控制流抽象，不是调度资源。

#### 14.3 “`promise_type` 就是 `std::promise`”

不是。  
`promise_type` 是协程返回类型的定制协议对象。  
`std::promise` 是并发库里和 `std::future` 配套的结果通道。

#### 14.4 “调用协程函数就等于执行了整个函数体”

不一定。  
要看 `initial_suspend()` 返回什么。

如果是：

- `suspend_never`：通常会立刻开始执行
- `suspend_always`：通常先停住，等以后再恢复

#### 14.5 “协程结束了，资源自然就都没了”

不一定。  
协程逻辑完成后，frame 可能还活着，直到 `destroy()`。

---

### 15. C++20 协程的常见坑

#### 15.1 悬空引用

这是最危险也最常见的问题之一。

例如，协程里如果保存了对某个局部对象或外部对象的引用，而协程挂起后这个对象已经失效，那么恢复时就可能访问悬空对象。

要特别警惕：

- 引用捕获
- `string_view` / span 之类非拥有型视图
- 指向调用栈对象的裸指针

#### 15.2 handle 泄漏

只要 frame 没有被销毁，就可能泄漏。  
所以必须明确：

- 谁拥有 handle
- 谁负责在何时 `destroy()`

#### 15.3 对已完成协程重复恢复

如果协程已经处于完成状态，再错误地恢复它，行为就不安全。  
通常要在 `resume()` 前检查 `done()`，并明确所有权协议。

#### 15.4 异常传播路径不清楚

如果 `unhandled_exception()` 只是吞掉异常，外部可能完全不知道失败发生了。  
所以返回对象通常需要定义：

- 异常如何存储
- 外部在什么 API 上重新抛出

#### 15.5 恢复线程不符合预期

如果 awaiter 在某个线程池线程里恢复协程，那么协程后半段就会在那个线程执行。  
这在 UI 线程、游戏主线程、actor 模型等场景里尤其敏感。

#### 15.6 误把协程当零成本抽象

协程常常很值，但不意味着没有成本。  
常见成本包括：

- frame 分配
- 更复杂的生命周期分析
- 栈回溯和调试复杂度提升
- 优化器可见性下降

工程上要做的不是“迷信协程”，而是判断它是否值得。

---

### 16. 什么场景适合协程，什么场景不一定适合

#### 16.1 适合

下面这些场景通常很适合协程：

- 有大量等待点的网络服务
- 需要把异步流程写成顺序逻辑的业务代码
- 资源加载、定时器、事件驱动流程
- 需要生成器式逐步产出的逻辑

这些场景的共性是：

- 控制流会多次中断
- 中断期间需要保留状态
- 顺序表达对可维护性很重要

#### 16.2 不一定适合

下面这些场景不一定值得上协程：

- 完全同步、没有等待点的短小纯计算函数
- 生命周期和所有权本来就极其复杂，协程反而会增加理解难度
- 团队对协程协议不熟，维护成本会明显超过收益

如果一个函数没有“中途暂停、以后恢复”的需求，协程通常不是杠杆点。

---

### 17. 如何判断自己是否真的理解了协程

如果你想避免“我好像懂了”的错觉，可以用下面这些问题自测。

#### 17.1 解释测试

你能否不用“语法糖”这四个字，独立解释：

- 协程到底在解决什么问题
- 协程和线程的差别是什么
- 为什么说协程本质上接近状态机

#### 17.2 机制测试

你能否明确说出：

- `promise_type` 负责什么
- `coroutine_handle` 负责什么
- `await_ready / await_suspend / await_resume` 各负责什么
- `initial_suspend` 和 `final_suspend` 分别影响什么

#### 17.3 预测测试

看到下面这些情况时，你能否预测行为：

- `initial_suspend = suspend_always`，调用协程函数后函数体会不会立刻执行
- `await_ready() == true` 时会不会真的挂起
- 一个事件在别的线程里恢复协程，后半段代码在哪个线程跑
- 协程结束后如果没人 `destroy()` 会发生什么

#### 17.4 边界测试

你能否指出：

- 协程本身负责什么
- 调度器或事件循环负责什么
- 为什么协程不能自动把阻塞 API 变成非阻塞

如果这些问题你都能清楚回答，说明你的模型基本已经成型。

---

### 18. 一页式总结

最后把整篇压缩成一页。

#### 18.1 一句话

**协程是一种把“可暂停、可恢复的流程”当成一等结构来表达的机制。**

#### 18.2 它解决什么

- 避免等待型流程把代码拆成回调地狱
- 避免手工维护状态机
- 让异步或分段执行逻辑重新接近顺序表达

#### 18.3 它不解决什么

- 不自动提供并行
- 不自动提供调度器
- 不自动把阻塞 API 变成非阻塞

#### 18.4 在 C++20 里它靠什么实现

- 编译器把协程函数改写成状态机
- coroutine frame 保存局部状态
- `promise_type` 负责外部协议
- `coroutine_handle` 负责恢复和销毁
- awaiter 协议负责等待、挂起、恢复

#### 18.5 最重要的工程认知

协程最核心的收益是**控制流表达能力**，不是魔法性能。  
它最核心的风险是**生命周期和恢复语义比表面代码看起来更复杂**。

如果你记住这两句话，后面学任何具体协程库都会稳很多。

---

### 19. 工业 / 现实世界锚点

#### 19.1 高并发网络服务

协程最典型的工业场景，是高并发网络服务里的“等待链”。

例如一个 RPC 或网关请求往往会经历：

- 读取 socket
- 查缓存
- 等数据库或远程服务
- 做少量业务逻辑
- 写回响应

这里真正贵的不是算，而是中间的大量等待与恢复。协程的价值在于把这条等待链重新写回接近顺序的控制流，同时保留异步运行时对资源的调度能力。

但要注意：只有当底层 IO、定时器、网络库或等待对象本身真的是异步可挂起的，这个收益才成立。协程不是给阻塞 API 抹粉。

#### 19.2 UI / 游戏主线程与事件驱动系统

另一个很现实的锚点，是 UI 主线程、游戏主循环或 actor 风格的事件驱动系统。

这些系统经常需要表达：

- 等待某个事件
- 分段恢复流程
- 在某个线程或调度上下文里继续执行

协程在这里的价值，不是并行，而是把“跨帧、跨事件、跨阶段恢复”的控制流写清楚。

### 20. 当前推荐实践、过时路径与替代

#### 20.1 当前更推荐的实践

当前更稳的工程做法通常是：

- 把协程放在明确的 runtime / executor / event loop 之上使用
- 明确恢复线程和恢复上下文，而不是默认“恢复后应该还在原线程”
- 给取消、超时、错误传播和生命周期管理留出明确协议
- 只在真正的等待型流程上用协程，而不是把所有函数都改成协程

#### 20.2 较旧或明显不推荐的路径

下面这些路径不是完全消失了，但在大型工程里通常更不稳：

- 深层回调链：可工作，但控制流碎裂、错误传播难、组合困难
- 手写状态机：可控但维护成本高，尤其在状态越来越多时
- “把阻塞 API 包进协程函数就等于非阻塞”：这是最常见误用之一

替代关系要看清楚：

- 回调和手写状态机并没有在逻辑上失效，它们是更底层的实现手段
- 协程的价值是把这些低层机制重新压缩成更可维护的表达层
- 真正替代阻塞调用的，不是协程语法本身，而是异步 IO、事件循环、线程池或调度运行时

### 21. 迁移入口

如果你已经建立了这篇文档里的模型，那么你应该能把它迁移到：

- Rust `async/await`
- Python `asyncio`
- C# `async/await`
- generator / iterator 风格控制流
- 任何“把分段恢复流程压缩成顺序表达”的异步框架

迁移时最关键的不是 API 名字，而是重新问：

- 谁保存状态
- 谁决定何时恢复
- 恢复后在哪个执行上下文继续
- 资源谁拥有，何时销毁

### 22. 未解问题与继续深挖

当前还值得单独深挖的点包括：

- coroutine 与 executor、io_uring 或网络运行时结合时，真正的边界责任如何划分？
- cancellation、allocator、lifetime 三者叠加后，哪些失败模式最容易在工程里爆炸？
