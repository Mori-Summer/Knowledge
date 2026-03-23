---
doc_id: computer-systems-dll-injection
title: DLL 注入：把代码带进别的进程时，真正起作用的是装载链、执行触发与权限边界
concept: dll_injection
topic: computer-systems
depth_mode: deep
created_at: '2026-03-23T11:47:09+08:00'
updated_at: '2026-03-23T11:47:09+08:00'
source_basis:
  - microsoft_process_security_access_rights_docs_checked_2026_03_23
  - microsoft_virtualallocex_docs_checked_2026_03_23
  - microsoft_writeprocessmemory_docs_checked_2026_03_23
  - microsoft_createremotethread_docs_checked_2026_03_23
  - microsoft_loadlibrary_docs_checked_2026_03_23
  - microsoft_dynamic_link_library_search_order_docs_checked_2026_03_23
  - microsoft_setwindowshookex_docs_checked_2026_03_23
  - microsoft_appinit_dlls_secure_boot_docs_checked_2026_03_23
  - microsoft_dynamic_link_library_best_practices_checked_2026_03_23
  - microsoft_process_mitigation_dynamic_code_policy_checked_2026_03_23
  - microsoft_defender_exploit_protection_reference_checked_2026_03_23
  - mitre_attack_t1055_001_checked_2026_03_23
  - mitre_attack_det0389_checked_2026_03_23
time_context: current_practice_checked_2026_03_23
applicability: windows_process_model_reasoning_loader_security_detection_and_tooling_design
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/computer-systems/process.md
  - docs/computer-systems/thread.md
  - docs/computer-systems/process-memory-layout.md
  - docs/computer-systems/virtual-memory-learning-model.md
open_questions:
  - Windows 上受保护进程、Code Integrity Guard、CFG 与 EDR 用户态 hook 之间，哪些组合最值得单独拆成“注入失败定位矩阵”？
  - 是否需要再补一篇专门解释 reflective DLL loading、manual mapping 与普通 loader 路径差异的文档？
  - 是否需要再补一篇把 DLL 注入、APC、线程劫持、进程空洞化放进同一“代码进入他进程的执行面模型”的文档？
---

# DLL 注入：把代码带进别的进程时，真正起作用的是装载链、执行触发与权限边界

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立的，不是“`WriteProcessMemory + CreateRemoteThread`”这一条配方式记忆，而是一套可以反复调用的内部模型：

- 看到一个 DLL 注入案例时，你能先判断它到底走的是哪类装载路径
- 你能区分“拿到目标进程句柄”“让目标进程看到 DLL”“让目标线程真正跑到代码里”这三件不同的事
- 你能解释为什么同样叫 DLL 注入，有的走系统 loader，有的在本质上已经变成自带 loader 的 PE 注入
- 你能预测为什么注入常常死在 bitness、session、desktop、保护级别、DllMain、依赖解析或系统缓解策略上
- 你能把这个模型迁移到检测、排障、工具设计、扩展点设计和安全评估上

这篇文档不会写成“实操教程”。  
重点是建立判断框架，而不是教你如何在目标机上执行一步一步的操作。

如果只记一句话：

**DLL 注入不是“把一个 DLL 塞进别的进程”这么简单，而是让目标进程在自身地址空间里满足一条完整链：能接触到载荷、能由某个执行面触发 loader 或替代 loader、且这一切没有被权限和防护边界拦下。**

## 2. 一句话结论 / 问题定义

**DLL 注入是一个技术族：它通过组合目标进程访问权、载荷承载方式、装载路径和执行触发点，让 DLL 代码在另一个进程的用户态地址空间内被装入并执行。**

它真正要解决的问题不是“跨进程写一点内存”，而是让下面四件事同时成立：

- 目标进程能接触到要执行的代码或能解析到 DLL 文件
- 某个线程最终会进入这段代码，或进入能装载这段代码的入口
- 依赖、重定位、TLS、`DllMain` 等装载语义不会把进程带崩
- 目标进程的权限模型、保护级别和 exploit mitigations 没有阻断这条路径

所以，DLL 注入的最稳理解方式不是“某几个 API 名字”，而是：

- **谁在提供代码**
- **谁在负责装载**
- **谁在触发第一次执行**
- **谁在观察或阻断这条链**

## 3. 对象边界与相邻概念

### 3.1 这篇文档直接处理什么

这篇文档直接处理的是 Windows 用户态里“让 DLL 代码进入他进程并执行”的几类典型路径，尤其是：

- 由系统 loader 接管的路径，例如远程 `LoadLibrary` 或 hook 驱动的 DLL 装入
- 借用 GUI / desktop 机制的路径，例如 `SetWindowsHookEx`
- 借用历史扩展点的路径，例如 `AppInit_DLLs`
- 与 DLL 注入紧邻、但会把 loader 责任改写掉的路径，例如 reflective loading / manual mapping

### 3.2 它不等于什么

它不等于：

- **泛化的进程注入。** DLL 注入只是更大“代码进入别的进程执行”家族中的一个分支。
- **普通插件机制。** 插件系统是目标程序主动暴露扩展点；注入是外部实体把代码带入目标。
- **DLL 搜索顺序劫持。** 那是利用装载时的名字解析错误，让目标自己加载错误 DLL；不是典型的跨进程内存操作链。
- **内核态驱动注入。** 本文聚焦用户态进程与 Win32 / NT 用户态接口。
- **纯 PE 注入。** 如果你绕开系统 loader 自己做映射、重定位和导入解析，本质上已经更接近 custom loader / PE injection。

### 3.3 最值得一起看的相邻概念

最值得一起看的相邻概念是：

- 进程对象、线程对象和访问令牌
- 进程虚拟地址空间与远程内存操作
- PE / DLL 装载器、导入表、重定位、TLS 回调、`DllMain`
- GUI thread、desktop、hook 链
- 完整性级别、保护进程、签名策略与 exploit mitigations
- 进程内扩展点设计与 out-of-process 替代架构

### 3.4 三组最容易混淆的边界

最常见的混淆有三组：

- **“写入 DLL 字节” vs “让 DLL 被装起来”。**  
  把内容写进目标地址空间，不等于系统已经按 DLL 语义处理了导入、重定位、TLS 和入口点。

- **“拿到目标进程句柄” vs “拿到可用执行面”。**  
  拿到 `PROCESS_VM_WRITE` 或 `PROCESS_VM_OPERATION` 只解决了“能不能碰它的地址空间”，没解决“谁来跑第一段代码”。

- **“注入成功” vs “代码稳定驻留并长期工作”。**  
  一次入口触发成功，不代表依赖路径、消息泵、线程生命周期、loader lock 和防护策略允许它稳定存在。

### 3.5 本文的适用边界

本文默认工作边界是：

- 以现代 Windows 桌面用户态进程为主
- 重点解释 loader 参与路径、线程触发路径和系统防护面
- 在“当前推荐实践”中显式给出核对日期 `2026-03-23`

它不试图详细展开：

- 内核驱动、PatchGuard、内核回调和 ring 0 机制
- 每一种 injection 变体的操作细节或代码模板
- 某一家 EDR 的私有检测规则实现

## 4. 核心结构

### 4.1 最稳的统一模型：五层而不是五个 API

把 DLL 注入建模得足够稳，最好的方式不是背 API，而是把它拆成五层：

| 层 | 你要问什么 | 典型对象 |
| --- | --- | --- |
| 目标进程层 | 目标进程是什么形态，允许什么，不允许什么 | bitness、session、desktop、完整性级别、保护级别、GUI / 非 GUI |
| 载荷层 | 你究竟要带进去什么 | 磁盘上的 DLL、内存中的映像、导入依赖、导出函数、TLS、`DllMain` |
| 装载层 | 谁来完成映射、重定位、依赖解析和入口回调 | Windows loader、`LoadLibrary` 路径、自定义 reflective / manual loader |
| 执行触发层 | 第一段代码由谁、何时、在哪个线程上跑起来 | 远程线程、现有线程消息分发、hook、进程启动扩展点 |
| 策略与观测层 | 什么会拦它、什么会看见它 | 访问掩码、签名策略、CIG、动态代码限制、extension point 缓解、遥测关联 |

只要你先问这五层，大多数 DLL 注入现象都能落回同一张图里。

### 4.2 六个核心结构件

更具体地说，DLL 注入最少由六个结构件组成：

1. **目标进程外壳。**  
   这是注入能否成立的第一层边界。你面对的不是抽象“另一个进程”，而是带有 bitness、session、desktop、token、完整性级别和保护属性的具体进程。

2. **DLL 载荷。**  
   这里真正重要的不是“有个 DLL 文件”，而是它是否带依赖、入口点会做什么、是否需要 TLS 回调、是否要求特定搜索路径或签名状态。

3. **进入目标地址空间的承载方式。**  
   可能是一个完整路径字符串、一个节映射、一个共享对象、一个 hook 指向的模块，或者一段已经自带映射逻辑的 loader stub。

4. **装载责任的归属。**  
   这是区分路径本质的关键。  
   如果由 Windows loader 负责，那是典型 DLL 装载语义；如果由注入方自带 loader 负责，那已经把“DLL 注入”改写成了“自己实现装载器”。

5. **第一次执行的触发器。**  
   没有执行触发器，载荷只是静态存在。这个触发器可能是新线程、消息钩子、现有线程上下文、启动期扩展点，或某种回调。

6. **观测与阻断面。**  
   安全产品和系统缓解通常不只看一个 API，而是看“句柄权限 + 远程内存操作 + 执行触发 + 模块装载”的关联。

### 4.3 四个必须同时记住的属性

分析任一 DLL 注入路径时，至少同时回答四个问题：

| 属性 | 你要回答什么 |
| --- | --- |
| 模块来源 | 代码来自磁盘、共享位置，还是已在内存中的映像 |
| 装载责任 | 由 OS loader 完成，还是由自定义 loader 完成 |
| 首次执行面 | 新线程、现有线程、消息回调、启动期自动装载，还是别的入口 |
| 可见性 | 它会不会出现在正常模块列表、ImageLoad 遥测、API 相关性检测里 |

很多误判都来自只看其中一个属性。

### 4.4 三个坐标系：文件坐标、地址空间坐标、策略坐标

理解 DLL 注入时，至少要分清三个坐标系：

| 坐标系 | 你看到什么 |
| --- | --- |
| 文件坐标系 | DLL 路径、签名状态、搜索顺序、依赖文件 |
| 地址空间坐标系 | 目标进程是否有承载区、有没有新映射、新线程、模块是否进入正常 loader 视图 |
| 策略坐标系 | 访问权限是否足够、CIG / ACG / Disable extension points / protected process 是否拦截 |

如果你把这三个坐标系混成一个问题，就会得到“明明写进去了却没跑”“明明线程建出来了却没看到模块”“明明句柄有了却还是失败”这类表面矛盾。

### 4.5 一页纸可调用模板

以后遇到 DLL 注入相关问题，可以先用下面这个模板：

| 问题 | 最小判断动作 |
| --- | --- |
| 这是系统 loader 路径还是自定义 loader 路径 | 看谁负责导入解析、重定位、`DllMain` 调用 |
| 为什么目标进程句柄不够用 | 回查是否缺执行触发面、缺额外访问权、或被保护属性拦截 |
| 为什么 hook 只对部分进程生效 | 回查 same desktop、bitness、消息泵和应用模型约束 |
| 为什么注入后进程不稳定 | 回查 `DllMain`、loader lock、依赖解析、线程副作用和时序变化 |
| 为什么检测系统能很快发现 | 看是否暴露出句柄打开、远程内存写入、远程线程创建和可疑 ImageLoad 的关联 |

## 5. 核心机制 / 主链路 / 因果链

### 5.1 经典主链：由系统 loader 接管的远程装载链

从概念上看，最经典的 DLL 注入主链可以压成下面这条：

1. 选择目标进程，并拿到足够的进程访问权。  
   这一步解决的是“能不能碰它”。官方文档明确把 `PROCESS_CREATE_THREAD`、`PROCESS_VM_OPERATION`、`PROCESS_VM_WRITE`、`PROCESS_VM_READ`、`PROCESS_QUERY_INFORMATION` 这类权限和远程线程 / 远程内存操作绑定在一起。

2. 让目标进程能够“看见”要装入的 DLL。  
   这里的关键不是复制字节，而是让目标进程最终能定位到可装载的模块来源。对于 loader 管理路径，完整路径通常比裸文件名稳定，因为后者会落回 DLL 搜索顺序问题。

3. 给目标地址空间准备承载区。  
   这一层只是让目标进程能接收到必要参数或中间对象，并不等于代码已经真正装入。

4. 安排一次执行触发。  
   常见心智模型是“让目标进程中的某个线程最终调用到 loader 入口或相关回调”。没有这一步，前面都只是静态布置。

5. 由系统 loader 完成真正的 DLL 语义。  
   一旦走到 `LoadLibrary` 类路径，接下来的核心工作就不再是“注入方在写代码”，而是 Windows loader 在做映射、依赖解析、重定位、TLS 和入口调用。

6. 模块进入目标进程的运行时结构。  
   如果这是正常 loader 路径，它通常会留下较标准的模块可见性和加载痕迹；这既带来兼容性，也带来可观测性。

最重要的是理解：

**远程装载链的本质不是“跨进程拷贝 DLL”，而是“跨进程安排一次本地装载”。**

### 5.2 分支链：当装载责任从 Windows loader 转移到自定义 loader

很多实践里会把 reflective DLL loading 或 manual mapping 也和 DLL 注入放在一起讲。  
它们和经典路径的最大不同，不在“更隐蔽”，而在：

- 载荷不一定以普通文件路径交给系统 loader
- 映射、重定位、导入解析、TLS、入口触发，部分或全部由注入方自带代码完成
- 可见性可能不同于正常 `LoadLibrary` 路径

这意味着它们的本质变化是：

**“谁负责装载”被改写了。**

一旦装载责任转移，兼容性和维护成本会迅速上升：

- 你要自己处理更多 PE / DLL 语义
- 正常 loader 帮你兜底的部分少了
- 一些看似“静悄悄”的路径，实际换来了更高的脆弱性

所以，把 manual mapping 理解成“只是更 stealth 的 `LoadLibrary`”是错误的。  
更准确的理解是：它已经跨到了相邻概念。

### 5.3 GUI / desktop 分支：`SetWindowsHookEx` 为什么像注入，又不等于通用注入

`SetWindowsHookEx` 是理解 DLL 注入边界特别好的例子。

它说明了两件事：

- 有些 DLL 装入不是“我自己在目标里造一个线程”，而是借用 GUI / message / desktop 基础设施让系统在合适时机把 DLL 带进去
- 这类路径从来不是无边界的，它强依赖同桌面、消息泵和 bitness 匹配

官方文档明确指出：

- 如果 hook 面向别的进程或全局线程，hook 过程必须在 DLL 中
- 32 位 DLL 不能注入 64 位进程，64 位 DLL 也不能注入 32 位进程
- 全局 hook 影响的是同一 desktop 内的应用，并且属于共享资源
- 这类全局 hook 应限制在特殊用途应用或调试辅助场景

因此，这条链最适合被建模为：

**不是“我控制了任意进程”，而是“我借用了 Win32 输入 / 消息扩展点，把 DLL 装入同 desktop 的合适进程上下文”。**

### 5.4 历史扩展点分支：`AppInit_DLLs` 为什么越来越不像可依赖路径

`AppInit_DLLs` 说明了另一类路径：在目标进程启动期，通过系统扩展点自动把 DLL 带进去。

但它也恰好说明为什么一些旧路径今天已经不适合作为一般性模型：

- 它面向的是交互式应用的全局 API hook / 扩展
- 从 Windows 7 起需要数字签名
- 从 Windows 8 起，若开启 Secure Boot，该基础设施会被禁用
- Defender Exploit Protection 里的 `Disable extension points` 还能进一步禁掉这类扩展点

所以，`AppInit_DLLs` 今天更适合作为**历史路径和系统防护演化锚点**，而不是“当前通用可依赖方法”。

### 5.5 时序与稳定性链：为什么“能跑起来”不等于“跑得对”

DLL 注入最容易被低估的一层，是时序和 loader 约束。

官方文档给出的几个信号非常关键：

- `CreateRemoteThread` 本身会改变目标进程的线程形态、时序和内存布局
- 它即使在起始地址无效时也可能先成功返回，真正失败在远程线程开始执行之后才暴露
- 在 DLL 初始化或进程启动阶段，线程创建和 DLL 初始化是串行约束的
- `DllMain` 运行在 loader lock 下，重初始化、跨线程等待或再做复杂装载都可能导致死锁或崩溃

所以，真正稳定的模型必须把下面这条链写进去：

**执行触发越侵入，副作用越强；`DllMain` 越复杂，进程级故障半径越大。**

## 6. 关键 tradeoff 与失败模式

### 6.1 四个核心 tradeoff

DLL 注入最常见的取舍可以压成四组：

| 取舍 | 你买到什么 | 你付出什么 |
| --- | --- | --- |
| 系统 loader 路径 vs 自定义 loader 路径 | 前者兼容性高，后者可调整可见性 | 前者更容易被正常模块视图和检测看见，后者实现复杂且脆弱 |
| 新线程触发 vs 借现有机制触发 | 新线程直观，现有机制更贴近目标执行流 | 新线程会明显改变线程形态和时序；借现有机制则受 message、hook、上下文约束 |
| 磁盘可见载荷 vs 内存内载荷 | 磁盘载荷更容易让系统 loader 完整接管 | 磁盘路径、搜索顺序、签名与 image load 可见性更强 |
| 一般性注入能力 vs 稳定生产工程 | 前者覆盖面广 | 后者通常需要显式插件模型、签名策略和严格约束 `DllMain` |

### 6.2 最常见的失败模式

1. **权限不够。**  
   `PROCESS_VM_WRITE` 不等于你已经拥有完整远程执行所需的访问面；有些路径还受 token、DACL 和 `SeDebugPrivilege` 影响。

2. **bitness 不匹配。**  
   32 位和 64 位之间不是简单“地址更大或更小”的差别，而是模块格式、调用环境和 hook 行为都变了。

3. **session / desktop / GUI 约束被忽略。**  
   `CreateRemoteThread` 在 Windows 8 之前跨 session 会失败；hook 路径则强依赖 same desktop 和消息泵。

4. **路径与依赖解析出错。**  
   模块主文件能找到，不代表其依赖能找到；用裸文件名还会把问题变成 DLL 搜索顺序问题。

5. **`DllMain` 过重。**  
   在 loader lock 下做复杂初始化、等待别的线程、再次装载模块，都是把单个 DLL 问题升级成整个目标进程问题。

6. **系统缓解策略阻断。**  
   CIG、动态代码限制、Block remote images、Disable extension points、protected process 都可能让链路在不同位置被截断。

7. **远程线程副作用导致排障错位。**  
   `CreateRemoteThread` 文档明确提醒它会改变目标的线程与内存布局，还可能因锁竞争造成死锁；所以“注入后出问题”未必是 DLL 业务逻辑本身。

8. **观测面过强。**  
   MITRE 的检测策略已经把“远程分配 + 远程写入 + 远程线程 + 可疑 DLL 加载”作为明确可关联的行为模式。

## 7. 应用场景

### 7.1 需要同地址空间能力的合法工具

某些工具必须进入目标进程才能拿到足够强的上下文，例如：

- 调试辅助
- 性能剖析与 API 拦截
- 无障碍与输入法相关扩展
- 兼容性 shim 或 UI 自动化辅助

但这里的正确问题不是“能不能注入”，而是：

**有没有更明确、更可维护的官方扩展点，或者能不能改成 out-of-process 设计。**

### 7.2 安全攻防与恶意软件分析

在攻防场景里，DLL 注入的重要性不在于它神秘，而在于它是“代码进入高价值进程上下文”的一条常见路由。

因此，安全分析真正关心的是：

- 代码是怎样进来的
- 谁替它完成装载
- 首次执行发生在什么线程或回调里
- 目标进程为什么没有挡住它
- 检测系统为什么能或不能看到它

### 7.3 GUI 子系统与跨进程事件扩展

hook 类路径说明：有些所谓“注入”其实是 GUI 子系统的扩展副作用。  
这类场景高度依赖 Win32 消息模型，因此：

- 不是所有目标进程都适合
- 不是所有 bitness 组合都有效
- 也不是所有现代应用模型都允许 in-process hook

### 7.4 遗留企业软件与历史扩展点

`AppInit_DLLs`、老式 IME、全局 hook 等路径在老系统和老软件里长期存在。  
它们对今天仍然有价值，但价值主要是：

- 帮你解释历史系统为什么会这样
- 帮你理解现代防护为何专门去封这些洞

而不是作为新系统设计的默认方案。

## 8. 工业 / 现实世界锚点

### 8.1 Win32 远程内存与线程 API：官方暴露出来的最小能力面

微软官方文档把 `VirtualAllocEx`、`WriteProcessMemory`、`CreateRemoteThread`、process access rights 写得很清楚。  
这很重要，因为它们不是地下技巧，而是 OS 显式暴露出来的能力面。

它们作为锚点的重要性在于：

- 它们告诉你 DLL 注入首先是**进程对象权限问题**
- 它们把“远程地址空间操作”和“远程执行触发”分成了不同能力
- 它们也明确暴露了副作用和失败边界，例如访问权不足、无效起始地址、跨 session 历史约束

### 8.2 `SetWindowsHookEx`：最能说明 desktop / bitness 约束的官方扩展点

`SetWindowsHookEx` 是一个特别好的现实锚点，因为它把很多抽象边界一次性讲明白了：

- 同 desktop 才有意义
- 跨进程 / 全局 hook 时，过程必须在 DLL 中
- 32/64 位必须匹配
- 全局 hook 是共享资源，官方建议限于特殊用途或调试辅助

它证明 DLL 注入不是“一把万能锤”，而是深受子系统边界约束。

### 8.3 Exploit Protection：现代系统如何正面收缩注入面

微软 Defender Exploit Protection 的参考文档直接把多类注入相关缓解写成了系统策略：

- Block non-Microsoft signed binaries
- Arbitrary Code Guard / 动态代码限制
- Block remote images
- Disable extension points

这些策略重要的地方在于：

- 它们不只是“查杀恶意样本”，而是在映射、装载或扩展点层面直接收缩可用路径
- 它们把 DLL 注入问题从“API 能不能调”提升到了“内存管理器和策略层允不允许映像落地”

### 8.4 MITRE ATT&CK：为什么检测越来越看行为链而不是单点 API

MITRE ATT&CK 已把 DLL 注入单列为 `T1055.001`，并且给出 `DET0389` 这类行为检测策略。  
其核心不是盯某个单独 API，而是相关联：

- 远程内存分配
- 对远程进程写入
- 远程线程创建
- 由 `LoadLibrary` 或 reflective loading 触发的可疑模块装载

这正好证明：

**现代检测面对的对象，已经不是“用了哪个 API”，而是“整条装载与执行链是否异常”。**

## 9. 当前推荐实践、过时路径与替代

以下“当前推荐实践”相关判断的核对日期为 `2026-03-23`。

### 9.1 对合法工程实现，更推荐先问“为什么必须 in-process”

截至 `2026-03-23`，更稳的工程实践通常是：

- 优先选择官方插件接口、浏览器 / 应用扩展模型、COM、脚本扩展点、ETW、调试接口或 IPC，而不是先上通用 DLL 注入
- 如果必须走 in-process 路径，优先选择**系统 loader 可理解**、依赖和搜索路径可控、签名状态可管理的方案
- 对 DLL 路径显式使用完整路径和受控搜索目录，而不是把装载成败交给默认搜索顺序
- 把 `DllMain` 视为最小化入口，只做最少初始化，把复杂逻辑延后

这背后的原因不是“注入绝对不能用”，而是：

**DLL 注入天然把稳定性、兼容性和可维护性问题一起带进来了。**

### 9.2 对检测与防护，更推荐按行为链和策略面建模

截至 `2026-03-23`，更稳的防护与检测思路通常是：

- 不单独盯一个 API，而是关联“目标句柄访问权 + 远程地址空间操作 + 执行触发 + ImageLoad”
- 对高价值进程按兼容性启用收缩注入面的 exploit mitigations，例如 Block non-Microsoft signed binaries、Block remote images、Disable extension points、动态代码限制
- 把“注入失败”也当成重要信号，因为许多缓解会在映射、装载或扩展点层面提前中止链路

### 9.3 过时路径与更稳替代

下面这些思路今天已经明显不够稳：

- **把 `AppInit_DLLs` 当通用路径。**  
  它是历史设施，不适合作为新系统默认方案；Secure Boot 与 exploit mitigation 已显著压缩它的可用性。

- **把全局 hook 当一般工程架构。**  
  hook 适合特殊用途和调试辅助，但受 same desktop、bitness、消息泵和应用模型约束，不适合拿来代替正式扩展系统。

- **把“只要能调 `CreateRemoteThread` 就够了”当完整模型。**  
  这忽略了模块定位、依赖装载、`DllMain`、保护策略和检测面。

- **把 manual mapping 当成普通 DLL 装载的平替。**  
  它的真正替代不是“普通路径”，而是“自己承担 loader 责任”。如果没有充分理由，不应轻易走这条路。

### 9.4 更推荐的替代方向

如果你的目标是合法的产品或平台能力，更稳的替代方向通常是：

- 显式插件系统
- 进程外代理 + IPC
- 官方可审计扩展点
- 只读观测优先，少做 in-process 修改
- 对必须 in-process 的代码做签名、路径控制和最小初始化设计

## 10. 自测题 / 验证入口

1. 为什么“拿到目标进程句柄”不等于“已经具备注入能力”？
2. 为什么说 DLL 注入至少要区分“载荷进入”“装载发生”“首次执行”三件事？
3. 一个 32 位 hook DLL 在 64 位 Windows 上为什么可能只对部分进程生效，还要求安装方持续泵消息？
4. 为什么 `CreateRemoteThread` 成功返回后，目标进程仍然可能什么都没真正跑起来？
5. 为什么说 manual mapping 不是“更隐蔽的 `LoadLibrary`”，而是“自定义 loader 路径”？
6. 为什么 `DllMain` 的设计质量会直接决定“注入成功后目标是否稳定”？
7. 为什么现代检测更关心 API 相关性和模块装载链，而不是单独拦某个函数调用？
8. 如果一个高价值进程启用了 Block non-Microsoft signed binaries 或 Block remote images，你会优先把故障定位到哪一层？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- **PE / shellcode / manual mapping 注入。**  
  核心迁移点是：把“装载责任归谁”作为第一判断维度。

- **DLL 搜索顺序劫持。**  
  这里不是远程内存写入问题，而是“模块来源选择”被攻击者控制。

- **浏览器沙箱与高价值进程防护。**  
  这里最关键的是策略层如何收缩 extension points、动态代码和 image load 面。

- **插件系统设计。**  
  如果你的业务真的需要 in-process 代码，最好把它做成显式、可治理的扩展合同，而不是隐式注入。

- **排障与事件分析。**  
  当你看到远程内存写入、可疑 ImageLoad、异常线程、hook 装载失败时，可以直接按五层模型定位。

## 12. 未解问题与继续深挖

- Windows 上 CIG、CFG、动态代码限制、protected process、UIAccess 与 EDR 用户态监控的组合，哪些最值得做成统一兼容矩阵？
- reflective loading / manual mapping 在现代 Windows 11 环境下，与标准 loader 路径相比，哪些失败模式最常见、最容易误判？
- 合法安全软件和 exploit mitigations 之间的兼容性边界，是否值得单独拆成“API interception 的系统代价模型”？

## 13. 参考资料

以下时间敏感资料的核对日期均为 `2026-03-23`。

- Microsoft Learn, Process Security and Access Rights: https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights
- Microsoft Learn, VirtualAllocEx: https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex
- Microsoft Learn, WriteProcessMemory: https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory
- Microsoft Learn, CreateRemoteThread: https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread
- Microsoft Learn, LoadLibraryA: https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya
- Microsoft Learn, Dynamic-link library search order: https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order
- Microsoft Learn, SetWindowsHookExA: https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowshookexa
- Microsoft Learn, AppInit DLLs and Secure Boot: https://learn.microsoft.com/en-us/windows/win32/dlls/secure-boot-and-appinit-dlls
- Microsoft Learn, Dynamic-Link Library Best Practices: https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-best-practices
- Microsoft Learn, PROCESS_MITIGATION_DYNAMIC_CODE_POLICY: https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-process_mitigation_dynamic_code_policy
- Microsoft Learn, Exploit protection reference: https://learn.microsoft.com/en-us/defender-endpoint/exploit-protection-reference
- MITRE ATT&CK, Dynamic-link Library Injection `T1055.001`: https://attack.mitre.org/techniques/T1055/001/
- MITRE ATT&CK, Behavioral Detection of DLL Injection via Windows API `DET0389`: https://attack.mitre.org/detectionstrategies/DET0389/
