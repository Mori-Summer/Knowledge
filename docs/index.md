---
doc_id: docs-index
title: Knowledge Docs Index
concept: knowledge_docs_index
topic: root
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-04-09T12:20:00+08:00'
source_basis:
  - derived_repository_index
time_context: snapshot_2026_04_09
applicability: repository_navigation
prompt_version: not_applicable
template_version: index_v1
quality_status: maintained_asset
related_docs: []
open_questions:
  - 后续是否需要把 index 进一步拆成按主题自动生成的导航页？
---

# Knowledge Docs Index

当前按主题归档的知识文档如下。

## methodology

- [统一概念文档规范：新建、升级、审查与仓库集成](./methodology/document-generation-methodology.md)
- [固定概念文档生成 Prompt](./methodology/fixed-concept-generation-prompt.md)
- [统一概念文档模板](./methodology/concept-document-template.md)
- [统一概念文档质量门禁](./methodology/concept-document-quality-gate.md)
- [认知规范与问题建模手册](./methodology/cognitive-modeling-playbook.md)
- [学习新事物的方法手册：从陌生到可理解、可操作、可迁移](./methodology/learning-new-things-playbook.md)
- [方法论文档使用说明：旧版编排说明，已并入主规范](./methodology/methodology-operator-guide.md)

## ai-systems

- [AI 代理栈分层：Agent、MCP、Skill、Prompt 与 OpenClaw 的概念边界](./ai-systems/agent-mcp-skill-openclaw-concepts.md)
- [Agent：目标驱动执行闭环，不是会聊天的模型](./ai-systems/agent.md)
- [Prompt：从一次性提问到系统指令栈的输入控制面](./ai-systems/prompt.md)
- [Skill：把 SOP、约束与操作策略沉淀成可复用行为资产](./ai-systems/skill.md)
- [MCP：把工具、资源与提示接成标准能力面的协议层](./ai-systems/mcp.md)

## programming-languages

- [回调函数中的死锁与强引用泄漏：为什么会“自己等自己”，又为什么会“永远不释放”](./programming-languages/callback-deadlock-and-ownership-cycles.md)
- [协程：可挂起控制流、运行时恢复与结构化并发的统一模型](./programming-languages/coroutine.md)
- [C++20 协程：可挂起控制流、语言机制与运行时边界](./programming-languages/cpp20-coroutine-playbook.md)
- [PImpl：当你真正想隔离的是 ABI、编译依赖与实现细节](./programming-languages/pimpl.md)
- [回调函数：显式 continuation、异步边界与变量生命周期管理](./programming-languages/callback-lifetime-management.md)

## computer-systems

- [AoS / SoA 选型：把访问模式、SIMD、GPU、列式执行与混合布局接到同一判断框架](./computer-systems/aos-soa-layout-selection.md)
- [Array of Structs：按记录聚合、按对象边界取数的数据布局](./computer-systems/array-of-structs.md)
- [Struct of Arrays：按字段拆流、按批量同构操作取数的数据布局](./computer-systems/struct-of-arrays.md)
- [Atomic Wait / Notify：当你只是在等一个原子值变化时，不必先上 condition variable](./computer-systems/atomic-wait-notify.md)
- [Atomicity：单次访问不可分割，不等于整体同步](./computer-systems/atomicity.md)
- [Cache Coherence：同一份数据为什么不会在不同核心缓存里“各说各话”](./computer-systems/cache-coherence.md)
- [Condition Variable：你等的不是通知，而是条件何时在同步关系下成立](./computer-systems/condition-variable.md)
- [Data Race：冲突访问何时越过语言模型的合法边界](./computer-systems/data-race.md)
- [DLL 注入：把代码带进别的进程时，真正起作用的是装载链、执行触发与权限边界](./computer-systems/dll-injection.md)
- [DMA 技术：地址可达性、缓冲区所有权与可见性同步的统一模型](./computer-systems/dma.md)
- [从 DMA 看程序内存：虚拟地址、物理页与设备可达窗口的统一模型](./computer-systems/dma-view-of-program-memory.md)
- [DMA 缓冲区生命周期与地址稳定性：pin、映射、同步、完成与回收的统一模型](./computer-systems/dma-buffer-lifetime-and-address-stability.md)
- [Fence：什么时候需要一堵顺序墙，什么时候这堵墙其实并不会替你同步](./computer-systems/fence.md)
- [False Sharing：明明线程没抢同一个变量，为什么缓存行还在疯狂打架](./computer-systems/false-sharing.md)
- [Happens-Before：把线程内顺序和跨线程同步边拼成可见性总图](./computer-systems/happens-before.md)
- [malloc 的底层原理：从用户请求到 size class、arena、mmap 与碎片治理](./computer-systems/malloc-internals.md)
- [内存地址构建与转换：命名空间、锚点、偏移与授权链的统一模型](./computer-systems/memory-address-construction-and-translation.md)
- [内存池设计：把 free list、arena、slab 放回生命周期、局部性与回收边界的统一模型](./computer-systems/memory-pool-design.md)
- [Memory Order：从原子性到可见性与重排序控制](./computer-systems/memory-order.md)
- [Modification Order：每个原子对象各有一条修改总序，不是全局时间线](./computer-systems/modification-order.md)
- [多线程中的各种锁：不要先问 API 名字，先问所有权、等待方式与读写形状](./computer-systems/multithreaded-locks.md)
- [Mutex：你真正买到的不是“挡别人一下”，而是互斥区间和同步边](./computer-systems/mutex.md)
- [进程：执行实例、资源容器与隔离边界的统一模型](./computer-systems/process.md)
- [Release Sequence：同一原子对象上延续发布语义的特殊修改链](./computer-systems/release-sequence.md)
- [Semaphore：当问题是“还有没有令牌”，而不是“谁拥有锁”](./computer-systems/semaphore.md)
- [Synchronizes-With：并发图里一条正式成立的跨线程同步边](./computer-systems/synchronizes-with.md)
- [线程：共享地址空间里的调度执行流，不是更轻量的进程，也不是协程](./computer-systems/thread.md)
- [USB 主机控制器数据路径：从 URB、传输环到 DMA 缓冲区的统一模型](./computer-systems/usb-host-controller-data-path.md)
- [USB 传输协议：主机调度、端点契约与传输类型取舍的统一模型](./computer-systems/usb-transfer-protocol.md)
- [进程的内存布局：从 ELF 装载到堆、mmap 区、栈与共享库的统一模型](./computer-systems/process-memory-layout.md)
- [虚拟内存：地址抽象、访问控制与工作集行为的统一模型](./computer-systems/virtual-memory-learning-model.md)

## image-processing

- [导向滤波的数学原理、公式推导与 C++ 对照](./image-processing/guided-filter-derivation.md)
- [快速导向滤波的数学原理、公式推导与 C++ 对照](./image-processing/fast-guided-filter-derivation.md)

## mathematics

- [复数：把实轴与虚轴接成一个完整可运算的数系](./mathematics/complex-number.md)
- [虚数：把“负数开方无解”扩展成可运算的新轴](./mathematics/imaginary-number.md)
- [虚数的作用：为什么振荡、旋转、频域与线性系统都喜欢复数表示](./mathematics/imaginary-number-uses.md)
- [四元数：一种带非交换乘法的四维实代数对象](./mathematics/quaternion.md)
- [四元数在 3D 旋转中的应用：姿态状态、组合、插值与时间推进的统一模型](./mathematics/quaternion-in-3d-rotation.md)
- [四元数与欧拉旋转：局部坐标、全局状态与工程边界的区别](./mathematics/quaternion-vs-euler-rotation.md)

## economics

- [看得见的手与看不见的手：市场、组织与国家如何分工协调资源](./economics/visible-hand-and-invisible-hand.md)

## governance

- [法利权情理：把复杂冲突拆成规则、利益、权力、情感与法理的五维模型](./governance/fa-li-quan-qing-li.md)

## graphics-systems

- [ANGLE：把 OpenGL ES/EGL 固定为稳定前端的跨后端图形兼容层](./graphics-systems/angle.md)
- [OpenGL 上下文与 GL 资源释放顺序：大型项目里的生命周期治理模型](./graphics-systems/opengl-context-resource-lifetime-order.md)

## networking

- [DNS：名字解析不是电话簿，而是分层委派、缓存驱动的全球名字控制面](./networking/dns.md)
- [TCP：可靠字节流、连接状态机与反馈控制闭环的统一模型](./networking/tcp.md)
- [UDP：最小报文服务、应用自带控制面与路径现实的统一模型](./networking/udp.md)

## security

- [未授权程序观测逻辑：把 DMA、进程内存与抓包放回同一状态获取模型](./security/program-observation-surfaces.md)
- [DMA 作为越权观测面：带外内存可见性、权限边界与防御判断](./security/dma-memory-observation-boundaries.md)
- [进程内存观测：从地址空间投影到状态重建的统一模型](./security/process-memory-observation-and-state-reconstruction.md)
- [抓包与状态推断：网络投影能看到什么、看不到什么](./security/packet-capture-and-state-inference.md)

## systems

- [去中心化：不是“没有中心”，而是拆掉单点控制、单点信任与单点故障](./systems/decentralization.md)

## reports

- [文档规范化迁移报告 2026-03-16](./_reports/normalization-2026-03-16.md)
- [文档升级报告 2026-03-16](./_reports/upgrade-2026-03-16.md)
