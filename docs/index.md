---
doc_id: docs-index
title: Knowledge Docs Index
concept: knowledge_docs_index
topic: root
depth_mode: standard
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-06-26T00:00:00+08:00'
source_basis:
  - derived_repository_index
time_context: docs_navigation_alignment_2026_06_26
applicability: repository_navigation
prompt_version: not_applicable
template_version: index_v1
quality_status: maintained_asset
related_docs: []
open_questions: []
---

# Knowledge Docs Index

当前按主题归档的知识文档如下。

## methodology

- [统一概念文档规范：AI 生成、升级、审查与仓库集成](./methodology/document-generation-methodology.md)
- [统一概念文档质量门禁](./methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](./methodology/source-discipline-and-real-world-anchor-policy.md)

## governance

- [正式 docs 资产治理规范：身份、元数据、路径、生命周期、索引、链接与网络边界](./governance/docs-asset-governance.md)
- [文档治理执行规范：复用扫描、晋升、合并、迁移、修订、批量、决策与返工闭环](./governance/docs-change-governance.md)

## templates

- [文档治理记录模板：审查、批量审查、完成汇报与批量完成汇报](./templates/governance-record-templates.md)

## runbooks

- [批量治理 Runbook：生成、重构、抽样审查与停止条件](./runbooks/batch-governance-runbook.md)

## ai-systems

- [AI 代理栈分层：Agent、Prompt、Skill 与 MCP 的概念边界](./ai-systems/agent-prompt-skill-mcp.md)

## computer-systems

- [AoS / SoA 选型：把访问模式、SIMD、GPU、列式执行与混合布局接到同一判断框架](./computer-systems/aos-soa-layout-selection.md)
- [Cache Coherence：同一份数据为什么不会在不同核心缓存里“各说各话”](./computer-systems/cache-coherence.md)
- [DLL 注入：把代码带进别的进程时，真正起作用的是装载链、执行触发与权限边界](./computer-systems/dll-injection.md)
- [DMA 技术：地址可达性、缓冲区所有权与可见性同步的统一模型](./computer-systems/dma.md)
- [False Sharing：明明线程没抢同一个变量，为什么缓存行还在疯狂打架](./computer-systems/false-sharing.md)
- [Happens-Before：把线程内顺序和跨线程同步边拼成可见性总图](./computer-systems/happens-before.md)
- [malloc 的底层原理：从用户请求到 size class、arena、mmap 与碎片治理](./computer-systems/malloc-internals.md)
- [内存地址构建与转换：命名空间、锚点、偏移与授权链的统一模型](./computer-systems/memory-address-construction-and-translation.md)
- [内存池设计：把 free list、arena、slab 放回生命周期、局部性与回收边界的统一模型](./computer-systems/memory-pool-design.md)
- [Memory Order：从原子性到可见性与重排序控制](./computer-systems/memory-order.md)
- [多线程中的各种锁：不要先问 API 名字，先问所有权、等待方式与读写形状](./computer-systems/multithreaded-locks.md)
- [进程：执行实例、资源容器与隔离边界的统一模型](./computer-systems/process.md)
- [线程：共享地址空间里的调度执行流，不是更轻量的进程，也不是协程](./computer-systems/thread.md)
- [USB 传输协议：主机调度、端点契约与传输类型取舍的统一模型](./computer-systems/usb-transfer-protocol.md)
- [进程的内存布局：从 ELF 装载到堆、mmap 区、栈与共享库的统一模型](./computer-systems/process-memory-layout.md)
- [虚拟内存：地址抽象、访问控制与工作集行为的统一模型](./computer-systems/virtual-memory.md)

## economics

- [看得见的手与看不见的手：市场、组织与国家如何分工协调资源](./economics/visible-hand-and-invisible-hand.md)

## graphics-systems

- [ANGLE：把 OpenGL ES/EGL 固定为稳定前端的跨后端图形兼容层](./graphics-systems/angle.md)
- [OpenGL 上下文与 GL 资源释放顺序：大型项目里的生命周期治理模型](./graphics-systems/opengl-context-resource-lifetime-order.md)

## image-processing

- [导向滤波与快速导向滤波：局部线性模型、低分辨率系数近似与工程选型](./image-processing/guided-filter-derivation.md)

## mathematics

- [复数、虚数与复表示：新轴入口、完整数系与工程用途的统一模型](./mathematics/complex-number.md)
- [四元数与 3D 旋转：代数本体、姿态状态、欧拉角边界与工程选型](./mathematics/quaternion.md)

## networking

- [DNS：名字解析不是电话簿，而是分层委派、缓存驱动的全球名字控制面](./networking/dns.md)
- [TCP：可靠字节流、连接状态机与反馈控制闭环的统一模型](./networking/tcp.md)
- [UDP：最小报文服务、应用自带控制面与路径现实的统一模型](./networking/udp.md)

## programming-languages

- [回调函数：显式 continuation、异步边界与变量生命周期管理](./programming-languages/callback-lifetime-management.md)
- [协程：可挂起控制流、运行时恢复与结构化并发的统一模型](./programming-languages/coroutine.md)
- [PImpl：当你真正想隔离的是 ABI、编译依赖与实现细节](./programming-languages/pimpl.md)

## security

- [未授权程序观测逻辑：把 DMA、进程内存与抓包放回同一状态获取模型](./security/program-observation-surfaces.md)

## social-systems

- [法利权情理：把复杂冲突拆成规则、利益、权力、情感与法理的五维模型](./social-systems/fa-li-quan-qing-li.md)

## systems

- [去中心化：不是“没有中心”，而是拆掉单点控制、单点信任与单点故障](./systems/decentralization.md)
