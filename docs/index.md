---
doc_id: docs-index
title: Knowledge Docs Index
concept: knowledge_docs_index
topic: root
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-03-16T18:05:42+08:00'
source_basis:
  - derived_repository_index
time_context: snapshot_2026_03_16
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

- [固定概念文档生成 Prompt](./methodology/fixed-concept-generation-prompt.md)
- [统一概念文档模板](./methodology/concept-document-template.md)
- [认知规范与问题建模手册](./methodology/cognitive-modeling-playbook.md)
- [学习新事物的方法手册：从陌生到可理解、可操作、可迁移](./methodology/learning-new-things-playbook.md)

## ai-systems

- [AI应用中的 Agent、MCP、Skill 与 OpenClaw：概念、机制与边界](./ai-systems/agent-mcp-skill-openclaw-concepts.md)

## programming-languages

- [协程认知手册：以 C++20 为例，从概念到可操作模型](./programming-languages/cpp20-coroutine-playbook.md)
- [PImpl：当你真正想隔离的是 ABI、编译依赖与实现细节](./programming-languages/pimpl.md)
- [回调函数：显式 continuation、异步边界与变量生命周期管理](./programming-languages/callback-lifetime-management.md)

## computer-systems

- [Atomic Wait / Notify：当你只是在等一个原子值变化时，不必先上 condition variable](./computer-systems/atomic-wait-notify.md)
- [Atomicity：并发里“不可分割”到底解决了什么，又没有解决什么](./computer-systems/atomicity.md)
- [Cache Coherence：同一份数据为什么不会在不同核心缓存里“各说各话”](./computer-systems/cache-coherence.md)
- [Condition Variable：你等的不是通知，而是条件何时在同步关系下成立](./computer-systems/condition-variable.md)
- [Data Race：什么时候程序不是“有点危险”，而是直接掉进未定义行为](./computer-systems/data-race.md)
- [Fence：什么时候需要一堵顺序墙，什么时候这堵墙其实并不会替你同步](./computer-systems/fence.md)
- [False Sharing：明明线程没抢同一个变量，为什么缓存行还在疯狂打架](./computer-systems/false-sharing.md)
- [Happens-Before：并发里真正决定“可见”与“成不成 race”的关系](./computer-systems/happens-before.md)
- [Memory Order：从原子性到可见性与重排序控制](./computer-systems/memory-order.md)
- [Modification Order：为什么每个原子对象都有自己的修改总序，但整个世界并没有一个总时间线](./computer-systems/modification-order.md)
- [Mutex：你真正买到的不是“挡别人一下”，而是互斥区间和同步边](./computer-systems/mutex.md)
- [Release Sequence：为什么 release 之后的一串 RMW 还能继续“带着发布语义往前走”](./computer-systems/release-sequence.md)
- [Semaphore：当问题是“还有没有令牌”，而不是“谁拥有锁”](./computer-systems/semaphore.md)
- [Synchronizes-With：并发关系图里真正跨线程连边的那一下](./computer-systems/synchronizes-with.md)
- [虚拟内存：按“学习建模”方式理解的完整说明](./computer-systems/virtual-memory-learning-model.md)

## image-processing

- [导向滤波的数学原理、公式推导与 C++ 对照](./image-processing/guided-filter-derivation.md)
- [快速导向滤波的数学原理、公式推导与 C++ 对照](./image-processing/fast-guided-filter-derivation.md)

## reports

- [文档规范化迁移报告 2026-03-16](./_reports/normalization-2026-03-16.md)
- [文档升级报告 2026-03-16](./_reports/upgrade-2026-03-16.md)
