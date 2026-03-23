---
doc_id: computer-systems-process
title: 进程：执行实例、资源容器与隔离边界的统一模型
concept: process
topic: computer-systems
depth_mode: deep
created_at: '2026-03-23T10:31:57+08:00'
updated_at: '2026-03-23T10:31:57+08:00'
source_basis:
  - linux_fork_manpage_checked_2026_03_23
  - linux_execve_manpage_checked_2026_03_23
  - linux_waitpid_manpage_checked_2026_03_23
  - linux_pidfd_open_manpage_checked_2026_03_23
  - linux_pid_namespaces_manpage_checked_2026_03_23
  - linux_posix_spawn_manpage_checked_2026_03_23
  - linux_sched_manpage_checked_2026_03_23
time_context: current_practice_checked_2026_03_23
applicability: operating_system_modeling_process_lifecycle_reasoning_service_supervision_and_container_isolation
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/computer-systems/thread.md
  - docs/computer-systems/process-memory-layout.md
  - docs/computer-systems/virtual-memory-learning-model.md
open_questions:
  - 是否需要再单独补一篇“fork/exec/spawn 的工程选型”文档，把 `fork`、`vfork`、`posix_spawn`、`clone` 的使用边界专门展开？
  - cgroup、namespace、capability 与进程故障域之间，是否值得再补一篇“容器里进程边界”的专门文档？
  - pidfd、subreaper、现代服务监督树在大型系统里的最佳组合，是否需要再写一篇排障导向文档？
---

# 进程：执行实例、资源容器与隔离边界的统一模型

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立的，不是“进程就是运行中的程序”这句入门定义，而是一套以后能反复调用的内部模型：

- 你能解释进程为什么首先是一个资源容器和生命周期对象，而不只是“会跑的代码”
- 你能说清 `fork -> exec -> wait/reap` 这条主链路分别在改变什么、不改变什么
- 你能区分进程、线程、程序、容器、cgroup 这些常被混在一起的对象
- 你能分析为什么多进程常用于隔离与监督，而不是只因为“历史上一直这么做”
- 你能把这个模型迁移到 shell、服务管理、容器 PID 1、子进程回收和故障域设计上

如果只记一句话：

**进程是操作系统给一次执行实例建立的资源容器、身份边界和生命周期对象；线程是容器里的执行流，而不是进程的同义词。**

## 2. 一句话结论 / 问题定义

**进程是 OS 用来承载一个程序执行实例的“资源 + 身份 + 生命周期”对象：它拥有或关联地址空间、文件描述符集合、PID、父子关系、信号与退出语义，并通过创建、替换映像、等待与回收形成完整的执行生命周期。**

它真正要解决的，不只是“让代码跑起来”，而是同时让下面几件事成立：

- 给一次执行实例一个稳定的身份与可管理的生命周期
- 给资源、权限和失败结果一个清晰归属边界
- 允许父进程创建子进程、等待其退出、收集结果并继续监督
- 允许执行实例在保留身份和部分资源关系的前提下，用 `execve()` 换掉程序映像
- 为隔离、故障遏制、权限切分、容器化和服务监督提供基础边界

在现代 Linux 上，再多加一条特别关键的现实约束：

- 调度器选择下一个 CPU 上运行的是**线程**，不是抽象意义上的“整个进程”；所以进程更接近“容器与生命周期边界”，线程更接近“被调度的执行成员”

## 3. 对象边界与相邻概念

### 3.1 这篇文档直接处理什么

这篇文档直接处理的是：

- 程序执行实例如何被创建、替换映像、终止和回收
- 进程作为资源容器，承载哪些资源与关系
- 父子关系、PID、等待语义、监督语义与隔离边界
- 进程和线程、命名空间、地址空间之间的关系

### 3.2 它不等于什么

它不等于：

- **程序文件。** 程序是磁盘上的代码与数据映像；进程是一次正在发生的执行实例。
- **线程。** 线程是进程内部的执行流；进程可以含有多个线程。
- **容器。** 容器常由 namespace、cgroup、capability、mount 等机制组合实现，进程只是其内部和外部都必须面对的基本对象。
- **cgroup。** cgroup 是资源控制和统计边界，不是进程身份边界。
- **服务。** 一个服务可能对应一个进程、多个进程，甚至跨机器和容器。

### 3.3 最值得一起看的相邻概念

最值得一起看的相邻概念是：

- [线程：共享地址空间里的调度执行流，不是更轻量的进程，也不是协程](/Users/maxwell/Knowledge/docs/computer-systems/thread.md)
- [进程的内存布局：从 ELF 装载到堆、mmap 区、栈与共享库的统一模型](/Users/maxwell/Knowledge/docs/computer-systems/process-memory-layout.md)
- [虚拟内存：地址抽象、访问控制与工作集行为的统一模型](/Users/maxwell/Knowledge/docs/computer-systems/virtual-memory-learning-model.md)
- `fork(2)` / `execve(2)` / `waitpid(2)` / `pidfd_open(2)` / `posix_spawn(3)`
- PID namespace / `/proc`

### 3.4 本文的默认适用边界

本文默认工作边界是：

- 以现代 Linux / POSIX 用户态为主
- 以 shell、服务、子进程、容器语境下的通用进程模型为主
- 既看抽象概念，也看 Linux 的 thread group、pidfd、PID namespace 等现实实现

涉及当前 Linux 行为、`pidfd_open()`、`posix_spawn()`、PID namespace 等内容的核对日期均为 `2026-03-23`。

### 3.5 五组最容易混淆的边界

最常见的混淆有五组：

- **程序 vs 进程。**
  程序是静态映像，进程是执行实例。

- **进程 vs 线程。**
  进程是资源与生命周期边界；线程是容器内的调度执行流。

- **`fork` vs `execve`。**
  `fork()` 复制出一个新的进程实例；`execve()` 不是“再开一个进程”，而是用新程序替换当前进程映像。

- **退出 vs 回收。**
  子进程结束执行不等于父进程已经收尸；没被 wait 的已退出子进程仍可能以 zombie 形态保留状态。

- **PID vs 稳定句柄。**
  PID 是数字标识，不是天然无 race 的长期句柄；现代 Linux 里 pidfd 才是更稳的监督句柄。

## 4. 核心结构

### 4.1 最稳的模型：把进程看成五层容器，而不是一个 PID

理解进程，最稳的方式不是记“它有 PID”，而是把它看成五层叠起来的对象：

| 层 | 它解决什么 | 典型对象 |
| --- | --- | --- |
| 身份层 | 让这次执行实例在系统里可定位 | PID、PPID、process group、session |
| 资源层 | 让资源有归属边界 | 地址空间、文件描述符、cwd、环境、凭证 |
| 执行层 | 承载真正跑代码的成员 | 一个或多个线程 |
| 生命周期层 | 让创建、替换映像、退出、回收可被监督 | `fork`、`execve`、`exit`、`waitpid` |
| 可见性 / 隔离层 | 决定别人如何看到它、能否操作它 | PID namespace、权限、`/proc`、pidfd |

如果你只记 PID，就会看不见资源归属、监督边界和隔离语义。

### 4.2 进程最值得记住的五个结构件

1. **地址空间与映像**
   进程持有自己的虚拟地址空间。`fork()` 之后，父子处在不同 memory space；`execve()` 之后，当前进程的程序映像被新程序替换。

2. **文件描述符与其他 process-wide 资源**
   文件描述符集合、工作目录、很多信号与凭证语义都以进程为边界存在或共享。

3. **父子关系与监督树**
   进程不是孤立点，而在监督树里有 parent / child 关系；这直接决定谁 wait、谁 reap、谁继承 orphan。

4. **线程组与执行成员**
   现代 Linux 上，一个“进程”往往对应一个 thread group；调度真正作用于组内各线程。

5. **退出状态与回收语义**
   进程不是“死了就没了”；退出状态要被父进程或其他 reaper 消费，否则就会留下 zombie。

### 4.3 进程真正拥有的，不只是内存

很多人把进程先想成“内存空间”。这不够。

更稳的做法是同时记三类归属：

| 归属 | 例子 | 为什么重要 |
| --- | --- | --- |
| 地址空间归属 | text/data/heap/mmap/stack | 决定隔离、COW、映像替换 |
| 资源归属 | 文件描述符、cwd、credentials | 决定权限、IO 共享、exec 行为 |
| 生命周期归属 | child status、zombie、wait 责任 | 决定服务监督与回收是否正确 |

### 4.4 三条最重要的关系边

从建模角度，进程最值得记住的不是属性清单，而是三条关系边：

1. **父子边**
   负责创建、等待、回收和 orphan 处理。

2. **映像替换边**
   负责把“还是同一个进程”与“代码/内存映像已经换了”这两个事实同时成立。

3. **可见性边**
   决定谁能看到它、发信号给它、以哪个 PID 看到它、能否进入其 namespace 语境。

### 4.5 一页纸判断模板

看到进程问题时，可以先过这张表：

| 你在问什么 | 优先从哪层判断 |
| --- | --- |
| 这是同一个进程，还是新建了一个 | 生命周期层：`fork` / `execve` / `clone` |
| 为什么资源还在 / 不在 | 资源层：FD、CLOEXEC、cwd、凭证、映像保留规则 |
| 为什么父进程还看得到它 | 生命周期层：waitable、zombie、reap |
| 为什么容器里看到的 PID 不一样 | 可见性层：PID namespace |
| 为什么一个“进程”里会有多条执行流 | 执行层：thread group / threads |

## 5. 核心机制 / 主链路 / 因果链

### 5.1 一条完整主链路：`fork -> execve -> wait/reap`

这条链是进程概念最核心的主链路。

1. 父进程调用 `fork()`。
2. 内核创建子进程。官方 `fork(2)` 明确说明：子进程和父进程处于**不同的 memory spaces**；在 Linux 上主要通过 copy-on-write 复制页表与任务结构，避免立即复制全部物理页。
3. 子进程继承父进程的大量资源关系，包括打开的文件描述符副本；这些副本引用同一 open file description，因此会共享 offset 和文件状态。
4. 如果子进程要跑不同程序，通常紧接着调用 `execve()`。
5. `execve()` 用新的程序映像替换当前进程。官方 `execve(2)` 明确说明：大部分 process attributes 被保留，但内存映射、捕获型 signal disposition、很多 pthread 对象不会被保留；若原进程是多线程，**除调用 `execve()` 的线程外，其余线程都会被销毁**。
6. 子进程退出后，会先进入 waitable 终止态；父进程通过 `wait()` / `waitpid()` / `waitid()` 获取退出状态并完成回收。
7. 如果没人回收，退出状态仍占据系统资源，形成 zombie。

这条链真正解释了三件事：

- 为什么“创建新程序”通常不是一个 syscall，而是一条两阶段链
- 为什么 `execve()` 不新建 PID，却会让程序映像完全变掉
- 为什么退出和回收必须分开理解

### 5.2 `fork()` 到底复制了什么，没复制什么

官方 `fork(2)` 给出的最关键事实是：

- 子进程与父进程最开始拥有相同内容的地址空间，但在不同 memory spaces 中运行
- 写入、`mmap()`、`munmap()` 等后续修改互不影响
- 子进程只带着调用 `fork()` 的那个线程过去；如果原进程是多线程，child 里不会复制出所有线程
- 多线程程序在 `fork()` 之后、`execve()` 之前，child 只能安全调用 async-signal-safe 函数

所以 `fork()` 的第一性结论不是“复制了一个完整世界”，而是：

- 复制出一个新的进程实例
- 继承大量资源关系
- 但在多线程程序里，child 的执行上下文被强烈收缩

### 5.3 `execve()` 的本质：换映像，不换身份容器

`execve()` 最容易被误解成“启动另一个进程”。  
更稳的理解是：

- `execve()` 不创建新 PID
- 它是在原有进程身份容器里，把程序映像和若干相关属性重置为新程序的状态

官方 `execve(2)` 明确指出：

- memory mappings 不保留
- file descriptor table 会解除 `CLONE_FILES` 共享
- 默认情况下 file descriptors 会跨 `execve()` 保留，除非设置了 `FD_CLOEXEC`
- 多线程进程在 `execve()` 时，其余线程被销毁

所以 `fork + execve` 这条链其实在做两件不同的事：

- `fork` 负责建立新的生命周期对象
- `execve` 负责把这个对象装入另一套程序映像

### 5.4 `waitpid()` / `pidfd`：为什么“结束”不是终点

官方 `waitpid(2)` 明确指出：

- `wait()` / `waitpid()` 挂起的是**调用线程**
- 它们等待的是 child state change，默认最常见的是等待 child terminate
- 若子进程已改变状态，调用可立即返回；否则阻塞到有 child 变成 waitable

这说明进程生命周期不是“运行 -> 结束”两态，而至少有：

- 运行中
- 已终止但未回收
- 已回收

在现代 Linux 上，还多了一条非常重要的监督路径：

- `pidfd_open()` 创建的是一个指向 task 的 file descriptor
- 官方文档明确说，对于已经存在的进程，`pidfd_open()` 是获得 PID file descriptor 的**preferred way**
- 它可被 `poll` / `epoll` 监视，可被 `waitid()` 使用，也避免了单纯持有数字 PID 时的复用 race

### 5.5 进程可见性变体链：PID namespace 与“容器里的 PID 1”

PID namespace 是进程概念必须补上的现代变体链。

官方 `pid_namespaces(7)` 给出的几个关键事实是：

- 不同 PID namespace 可以出现相同的 PID
- 新 namespace 里的第一个进程是该 namespace 的 PID 1
- 如果 namespace 的 init 进程终止，内核会向该 namespace 中的所有进程发送 `SIGKILL`
- 进程的 PID namespace membership 在创建时决定，之后不能改变

这条链解释了容器语境下两个高频现象：

- 为什么“容器里的 PID 1”不是普通编号，而是监督/孤儿收养语义中心
- 为什么进程在不同观察者眼里能有不同 PID

## 6. 关键 tradeoff 与失败模式

### 6.1 进程边界买到什么，付出什么

| tradeoff | 你买到什么 | 你付出什么 |
| --- | --- | --- |
| 进程隔离 vs 进程内共享 | 更强故障隔离、权限边界、回收与监督清晰 | IPC、地址空间切分、数据复制或序列化成本 |
| `fork/exec` 灵活性 vs 生命周期复杂度 | 强大的启动、替换映像和子进程树模型 | 回收、CLOEXEC、atfork、spawn 细节变复杂 |
| 多进程设计 vs 多线程设计 | 更稳的故障域与资源边界 | 跨进程共享状态更贵 |
| 数字 PID 简单性 vs 稳定监督 | 传统 API 普遍可用 | 需要面对 PID reuse；现代上常需 pidfd |

### 6.2 六类高频失败模式

1. **把进程等同于“运行中的程序文本”。**
   这样你就看不见它真正提供的资源归属和生命周期语义。

2. **把 `execve()` 当成“再开一个进程”。**
   这会让你看不懂为什么 PID 没变、FD 可能仍在、但映像全换了。

3. **在多线程程序里 `fork()` 后做太多事。**
   官方 `fork(2)` 明确限制 child 在 `execve()` 前只能安全调用 async-signal-safe 函数。

4. **只结束子进程，不回收子进程。**
   这会制造 zombie，并逐步消耗 PID / 内核表项等系统资源。

5. **把 PID 当稳定句柄长期保存。**
   当进程退出、PID 被复用后，监督、发信号、诊断都可能打到错误对象。

6. **把容器里的 PID 1 当普通进程。**
   在 PID namespace 里，PID 1 对 orphan 和 namespace 生命周期有特殊地位。

## 7. 应用场景

### 7.1 shell、任务启动器与命令执行

这是最经典也最现实的场景：

- 父进程负责创建 child
- child 配置重定向、环境、工作目录
- child `execve()` 成目标程序
- 父进程 `waitpid()` 收集退出状态

理解这条链后，shell、job runner、构建系统和命令执行器就不会再只是“黑箱跑命令”。

### 7.2 服务监督与子进程管理

服务管理器、守护进程、worker supervisor 的核心问题不是“怎么起个进程”，而是：

- 谁监督退出
- 谁收尸
- 谁处理 orphan
- 谁判断这个 PID 还是不是原来那个任务

这里 `waitpid()`、`pidfd_open()`、subreaper、SIGCHLD 都落在进程模型之内。

### 7.3 容器与 PID namespace

容器不是“把进程替换掉”，而是让进程在新的可见性与隔离语境里运行。

理解进程后，你才能看懂：

- 为什么容器里的 PID 1 特殊
- 为什么宿主机和容器内看到的 PID 不同
- 为什么 namespace membership 不是运行中随便改的属性

### 7.4 故障隔离、权限切分与插件沙箱

当你真正关心的是：

- 崩了不要带倒主进程
- 权限要切开
- 内存破坏不要直接污染主地址空间

那默认想的往往是进程边界，而不是线程边界。

## 8. 工业 / 现实世界锚点

### 8.1 `fork` / `execve` / `waitpid`：所有 shell 与命令执行器的现实基线

这不是教材里的抽象套路，而是 Unix/Linux 用户态最真实的执行器基线。

它的重要性在于：

- 它定义了“启动子程序”的经典路径
- 它解释了 FD 继承、退出码收集、COW 和 zombie 的全部现实行为
- 任何命令执行器、守护进程管理器、构建系统和很多语言 runtime 都绕不开这条链

### 8.2 `pidfd_open()`：现代 Linux 的稳定监督句柄

这条锚点重要，不是因为它取代 PID，而是因为它把“监督进程”从数字标识提升成了 file descriptor 语义：

- 可 `poll` / `epoll`
- 可被 `waitid()` 使用
- 避免仅持有 PID 带来的复用 race

这对服务监督和大型系统中的 child management 非常关键。

### 8.3 PID namespace：容器语境下进程模型的现代扩展

PID namespace 之所以是强锚点，是因为它说明：

- 进程身份不是只有一个全局视角
- “谁是 PID 1”会带来真实的监督与生死后果
- 容器本质上没有绕开进程，而是在更复杂的进程可见性框架里使用它

## 9. 当前推荐实践、过时路径与替代

本节涉及 `pidfd_open()`、`posix_spawn()`、PID namespace 等当前 Linux 行为的内容，核对日期均为 `2026-03-23`。  
其中“更推荐什么”的部分，除明确来自官方文档的条目外，其余是基于这些接口语义做的工程推断。

### 9.1 三条过时路径

#### 过时路径 1：把进程教成“运行中的程序”

局限：

- 解释不了资源归属
- 解释不了 `fork/execve`
- 解释不了 supervision、zombie、container PID 1

更稳替代：

- 把进程理解成“执行实例 + 资源容器 + 生命周期对象”

#### 过时路径 2：把 `execve()` 教成“创建新进程”

局限：

- 会把生命周期和映像替换混在一起
- 会误判 PID、FD、线程和地址空间行为

更稳替代：

- 明确区分：`fork()` 建立新实例，`execve()` 替换当前实例映像

#### 过时路径 3：把 PID 当监督句柄

局限：

- 仅靠数字 PID 无法规避 PID reuse race
- 在复杂 supervision 和异步监控里不够稳

更稳替代：

- 在现代 Linux 里，需要稳定监督既有进程时优先考虑 pidfd

### 9.2 截至 2026-03-23 的更稳基线

更稳的工程基线通常是：

1. **把 `fork` 和 `execve` 当成两阶段模型。**
   创建与装载不同，不要混成一个“spawn 黑箱”。

2. **在多线程程序里谨慎对待 `fork()`。**
   官方 man page 已明确 child 的安全调用窗口极窄。

3. **需要“启动并立刻换映像”的场景，可优先考虑 `posix_spawn()` 这类更收敛的 spawn 接口。**
   这是工程推断，不是 POSIX 对所有场景的强制唯一推荐；它的价值主要在于缩小手写 `fork + pre-exec housekeeping + exec` 的危险窗口。

4. **需要稳定监督既有子进程时，在 Linux 上优先理解 pidfd 路线。**
   这里是官方文档有明确表述支撑的。

5. **容器里必须显式建模 PID 1。**
   不要把它当普通工作进程。

### 9.3 何时优先用进程边界，而不是线程边界

这是工程推断，但通常更稳：

- 当你更在意故障隔离、权限切分、独立重启、监督树和资源归属时，优先进程
- 当你更在意共享内存下的超低通信成本与细粒度并发时，再考虑线程

别把“线程更轻量”误写成“线程在所有场景都更先进”。

## 10. 自测题 / 验证入口

1. 为什么“进程就是运行中的程序”不足以解释 zombie、wait 和 PID namespace？
2. `fork()` 和 `execve()` 分别改变了什么？什么是创建了新实例，什么是替换了旧映像？
3. 为什么 child 已退出，系统里仍可能保留它的一部分状态？这部分状态由谁消费？
4. 在多线程程序里，为什么 `fork()` 后 child 不能随便调用普通库函数？
5. 为什么在容器语境下，“PID 1”不是普通编号，而是必须单独建模的对象？
6. 只保存一个 PID 数字来监督外部进程，为什么会有 race？pidfd 解决了哪一层问题？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- shell / job runner / build system 的命令执行模型
- 守护进程、supervisor、subreaper 与 child lifecycle 管理
- 容器里的 PID 1、namespace 视角差异与 `/proc` 观察
- 多进程隔离、权限分层与沙箱设计
- 线程与协程的边界判断：谁负责容器，谁负责执行流，谁负责控制流结构

## 12. 未解问题与继续深挖

1. pidfd、subreaper、signal、namespace init 这几套监督语义，怎样形成一套更不易误判的统一心智模型？
2. 多进程服务中的 crash-only design、热重启与文件描述符移交，是否值得单独写成一篇更偏工程实践的文档？
3. 进程边界与 cgroup / namespace / seccomp / capability 叠加后的“真实隔离强度”，是否需要拆成专门章节？

## 13. 参考资料

以下“当前实践”相关内容的核对日期均为 `2026-03-23`。

- Linux `fork(2)`: https://man7.org/linux/man-pages/man2/fork.2.html
- Linux `execve(2)`: https://man7.org/linux/man-pages/man2/execve.2.html
- Linux `wait(2)` / `waitpid(2)`: https://man7.org/linux/man-pages/man2/waitpid.2.html
- Linux `pidfd_open(2)`: https://man7.org/linux/man-pages/man2/pidfd_open.2.html
- Linux `pid_namespaces(7)`: https://man7.org/linux/man-pages/man7/pid_namespaces.7.html
- Linux `sched(7)`: https://man7.org/linux/man-pages/man7/sched.7.html
- Linux `posix_spawn(3)`: https://man7.org/linux/man-pages/man3/posix_spawn.3.html
