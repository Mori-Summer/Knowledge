---
doc_id: computer-systems-multithreaded-locks
title: 多线程中的各种锁：不要先问 API 名字，先问所有权、等待方式与读写形状
concept: multithreaded_locks
topic: computer-systems
depth_mode: deep
created_at: '2026-03-20T09:57:44+08:00'
updated_at: '2026-03-20T14:42:35+08:00'
source_basis:
  - posix_pthread_mutex_lock_checked_2026_03_20
  - posix_pthread_mutexattr_gettype_checked_2026_03_20
  - posix_memory_synchronization_checked_2026_03_20
  - posix_pthread_rwlock_rdlock_checked_2026_03_20
  - posix_pthread_rwlock_unlock_checked_2026_03_20
  - linux_pthread_spin_init_manpage_checked_2026_03_20
  - linux_futex_manpage_checked_2026_03_20
  - linux_sem_overview_manpage_checked_2026_03_20
  - linux_kernel_locktypes_docs_checked_2026_03_20
  - linux_kernel_seqlock_docs_checked_2026_03_20
  - linux_kernel_rcu_whatis_docs_checked_2026_03_20
time_context: foundations_plus_current_practice_checked_2026_03_20
applicability: multithreaded_design_lock_selection_and_contention_debugging
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/computer-systems/mutex.md
  - docs/computer-systems/semaphore.md
  - docs/computer-systems/condition-variable.md
  - docs/computer-systems/atomic-wait-notify.md
open_questions:
  - 现代用户态运行时里的 parking mutex、公平队列锁和 NUMA-aware 锁，什么时候值得单独抽出一篇文档比较？
  - 在混合 CPU/GPU、用户态 IO 和协程调度场景里，传统“线程锁选型”模型还需要补哪些维度？
---

# 多线程中的各种锁：不要先问 API 名字，先问所有权、等待方式与读写形状

## 1. 这份文档要帮你学会什么

这篇文档的目标，不是把 `mutex`、`rwlock`、`spinlock`、`seqlock`、`RCU` 的定义各讲一遍，而是把“多线程里各种锁到底该怎么分辨和怎么选”压成一个可复用内部模型。

读完后，你应该至少能做到：

- 不再把所有同步原语都含糊地叫成“锁”
- 先看问题形状，再判断应该上 mutex、rwlock、spinlock、seqlock、RCU，还是干脆不用锁
- 分清“所有权”“等待方式”“读写并发形状”“是否允许重试”“是否需要回收延迟”这几个决定性维度
- 识别常见误用：拿 semaphore 当锁、拿 recursive mutex 掩盖设计问题、在普通用户态程序里乱上 spinlock、把 rwlock 当默认优化
- 把这张选择模型迁移到线程池、缓存、配置热更新、内核式读多写少结构、跨进程共享状态恢复上

## 2. 一句话结论 / 问题定义

**多线程中的“各种锁”本质上不是一串 API 名单，而是一组围绕“谁拥有临界区、等待时是睡还是转、读者能否并发、读者能否重试、更新后何时回收旧对象”展开的设计取舍。默认先选能稳定保护共享不变量的阻塞互斥锁；只有当问题形状明确变化时，才升级到读写锁、自旋锁、序列锁或 RCU。**

它真正要解决的问题不是：

- 这门语言 / 这个平台有哪些锁可以用

而是：

- **你要保护的到底是复杂共享不变量、许可数量、单个状态位，还是读多写少的数据发布**
- **等待线程应该阻塞、忙等，还是乐观读取后重试**
- **错误和异常时，谁负责恢复、谁负责释放、谁负责回收旧对象**

## 3. 对象边界与相邻概念

这篇文档里的“各种锁”，主要指多线程共享状态访问里常见的这些家族：

- `mutex` / 独占阻塞锁
- `rwlock` / 读写锁或共享互斥锁
- `spinlock` / 自旋锁
- `recursive mutex` / 递归锁
- `robust mutex` / 健壮锁
- `seqcount` / `seqlock`
- `RCU`

它不等于：

- `semaphore`
  semaphore 关注 permit 计数和准入，不关心 owner 语义
- `condition variable`
  它处理“条件未满足时怎么睡、条件变化时怎么醒”，不是临界区所有权
- `atomic_wait`
  它围绕单个 atomic 值等待，不是通用共享不变量保护工具
- `lock-free`
  那是另一条设计路线，不是“更快的锁”

最值得先划清的边界是：

- **锁保护的对象是什么**
  是一组共享字段的不变量，还是一个数量门槛，还是一个版本号
- **锁的语义是什么**
  是 owner-based exclusion、shared/exclusive、busy wait，还是 lockless read + retry
- **锁之外还需要什么**
  例如 condition variable、hazard management、grace period、内存回收协议

## 4. 核心结构

理解“各种锁”，最稳的方式不是按 API 分类，而是按**同步问题的形状**分类。

### 4.1 五个决定性维度

先不要问“库里有什么锁”，先问下面五个维度：

| 维度 | 你真正要回答的问题 |
| --- | --- |
| 所有权维度 | 进入临界区后，是否需要明确 owner？ |
| 等待维度 | 等待线程是应该睡眠、忙等，还是根本不该拿锁等？ |
| 读写形状维度 | 是独占写、共享读、快照读，还是读者近似无锁？ |
| 重试维度 | 读者能不能接受“读到一半重来一次”？ |
| 回收维度 | 更新后旧对象能否立刻销毁，还是必须延迟回收？ |

这五个维度比 API 名字更重要，因为它们直接决定：

- 你买到的是互斥、许可、等待，还是发布/回收协议
- 你付出的代价是阻塞、空转、重试，还是回收复杂度
- 出错后问题表现为死锁、饥饿、优先级反转、livelock，还是 use-after-free

### 4.2 三类根本不同的问题

多线程里常被统称为“锁问题”的，实际上至少有三类完全不同的问题：

| 问题类型 | 真正要保护什么 | 更贴题的原语族 |
| --- | --- | --- |
| 共享不变量保护 | 一组字段必须一起保持一致 | `mutex`、`rwlock`、`spinlock` |
| 等待条件成立 | 线程要等某个条件变化，而不是占有临界区 | `condition variable`、`atomic_wait`、`semaphore` |
| 发布与回收 | 读者要低成本看到新版本，旧版本稍后回收 | `seqlock`、`RCU`、hazard-style 方案 |

很多选型错误，根本不是“锁选错了”，而是：

- 你面对的是等待问题，却上了锁
- 你面对的是回收问题，却还在想普通互斥
- 你面对的是共享不变量问题，却拿 permit 或单个 atomic 值硬凑

### 4.3 锁家族在五维空间里的位置

把常见家族放进这五个维度，结构会清楚很多：

| 家族 | 所有权 | 等待方式 | 读写形状 | 读者是否可重试 | 回收要求 |
| --- | --- | --- | --- | --- | --- |
| `mutex` | 强 owner 语义 | 阻塞 / 睡眠 | 独占 | 不适用 | 立刻回收通常没额外困难 |
| `rwlock` | owner 语义 | 阻塞 / 睡眠或平台相关 | 多读单写 | 否 | 回收通常仍靠锁边界保证 |
| `spinlock` | 强 owner 语义 | 忙等 / 自旋 | 独占 | 不适用 | 立刻回收通常没额外困难，但不能长持有 |
| `recursive mutex` | owner + 计数 | 阻塞 / 睡眠 | 独占 | 不适用 | 不改变回收本质，只改变重入语义 |
| `robust mutex` | owner + 崩溃恢复通知 | 阻塞 / 睡眠 | 独占 | 不适用 | 需要恢复共享状态一致性 |
| `seqcount` / `seqlock` | writer 侧需串行化 | 读者 lockless retry，写者持锁 | 读多写少、快照读取 | 是 | 只适合能安全重试的快照数据 |
| `RCU` | 读者近似无锁，更新者需额外串行化 | 读者不阻塞，回收延后 | 读多写少、替换式更新 | 读者通常不重试，但必须容忍旧版本 | 必须显式延迟回收 |

### 4.4 一个实用的总图

把这张表压成一句话就是：

**先问你需要的是“独占所有权”“共享读取”“忙等不睡”“快照重试”还是“发布后延迟回收”，再去选锁。**

真正值钱的结构不是“锁名字表”，而是下面这个排序：

1. 先辨认问题类别
2. 再辨认等待模型
3. 再辨认读写形状
4. 最后才落到 API 家族

## 5. 核心机制 / 主链路 / 因果链

最可复用的选型主链，不是从 API 开始，而是从问题形状开始。

### 5.1 主决策链：从问题形状走到同步原语

1. **你保护的是复杂共享不变量吗？**
   如果是，例如队列 + 状态位 + 容量 + 关闭标志，默认先选 `mutex`。这是 owner-based exclusion 最稳的落点。

2. **你真正的问题其实只是“还有几个名额”或“某个值变了”吗？**
   如果是，那你很可能不该先上锁，而应回到 `semaphore`、`condition variable` 或 `atomic_wait` 这类更贴题原语。

3. **你真的有高价值的并发读吗？**
   如果读临界区足够长、读者确实多且彼此独立，再考虑 `rwlock`；否则很多“读多写少”的直觉优化只是在白白增加复杂度。

4. **等待期间允许睡眠吗？**
   如果允许睡眠，优先阻塞锁；如果绝对不能睡，并且临界区短到足以让忙等值得，才轮到 `spinlock`。

5. **读者能接受读到一半就重来吗？**
   如果可以，而且读对象是不含易失指针的一致快照，那么 `seqcount` / `seqlock` 开始变得合理。

6. **读者需要跟随指针、更新者要替换旧版本并延后回收吗？**
   这已经不是普通锁问题，而是 `RCU` 那类“先 removal，再 reclamation”的模型。

7. **旧代码存在重入、外部回调或崩溃恢复需求吗？**
   这时才考虑 `recursive mutex` 或 `robust mutex` 这类变体，但它们通常是特定约束下的补丁，不是默认答案。

### 5.2 这条主链背后的因果

- 只要问题是“保护复杂共享状态”，`mutex` 的 owner 语义和同步边最值钱
- 一旦目标从“互斥进入”变成“提高读并发”，就会引入 writer latency 和公平性问题
- 一旦目标从“阻塞等待”变成“忙等”或“乐观重试”，你就把 CPU 时间和正确性约束拿来换延迟
- 一旦读者开始 lockless，更新路径几乎总会变复杂，且内存回收不再免费

### 5.3 两条特别容易走错的分支

#### 5.3.1 等待分支：别把“等条件成立”误判成“拿锁保护”

很多代码的问题不是“临界区不安全”，而是“线程需要等某个谓词改变”。  
这里最容易犯的错误是：

- 先上一个锁
- 再希望这个锁顺便解决等待

这往往会把两个不同问题混在一起：

- 共享状态的一致性
- 线程如何在条件未满足时高效等待

所以一个高频判断句是：

**锁负责保护不变量，条件变量 / `atomic_wait` / semaphore 才负责让线程以正确方式等待。**

#### 5.3.2 回收分支：别把“读多写少”直接等同于 `rwlock`

“读多写少”其实有三种完全不同的形状：

| 形状 | 更像什么 |
| --- | --- |
| 读者只是共享读一段状态，写者偶尔独占修改 | `rwlock` |
| 读者只需要一个一致快照，愿意失败后重试 | `seqlock` |
| 读者会跟随指针，更新者替换旧版本，旧对象稍后回收 | `RCU` |

把这三类问题都叫“读多写少”会直接毁掉选型。

### 5.4 一条可用于代码审查的最小判断句

看到一段并发代码时，可以先问：

1. 这里保护的是不变量、等待谓词，还是版本发布？
2. 等待线程能不能睡？
3. 读者是否真的需要并发？
4. 读者能不能重试？
5. 旧对象何时回收？

如果这五个问题答不清，这段同步设计大概率还没真正成型。

## 6. 关键 tradeoff 与失败模式

### 6.1 关键 tradeoff

- **默认 mutex 的 tradeoff**
  你买到的是最稳的推理模型；代价是竞争时要阻塞，热点下会放大切换和排队。

- **rwlock 的 tradeoff**
  你买到的是读者并发；代价是 writer latency、升级困难、实现公平性和平台行为更复杂。

- **spinlock 的 tradeoff**
  你买到的是不睡眠、可能更低的极短路径延迟；代价是空转烧 CPU、优先级反转和无界等待风险。

- **seqlock 的 tradeoff**
  你买到的是便宜的一致快照读取；代价是读者可能反复重试，写侧约束也更严。

- **RCU 的 tradeoff**
  你买到的是读侧几乎无锁、很低干扰；代价是更新串行化、宽限期等待和回收协议复杂。

- **robust mutex 的 tradeoff**
  你买到的是“持有者死了，下一位能知道状态可能脏了”；代价是恢复协议根本没有消失，只是被显式暴露出来。

### 6.2 最常见的失败模式

- **把 semaphore 当成共享不变量保护锁**
  这会把“许可数量”和“状态一致性”混成一件事。

- **把 recursive mutex 当作常规设计手段**
  它常常只是把锁边界不清、调用层次失控、回调重入问题藏起来。

- **看见“读多写少”就直接上 rwlock**
  如果读临界区很短、写者不能饿、升级 / 降级路径复杂，这种优化往往得不偿失。

- **在普通用户态线程上滥用 spinlock**
  一旦持锁线程被抢占，其他线程只会白白消耗 CPU 时间。

- **把 seqlock 用在会跟随指针的数据上**
  kernel docs 已明确警告，这种机制不能用于受保护数据包含指针的情况，因为写者可能使读者正在跟随的指针失效。

- **把 RCU 当成“无锁万能药”**
  RCU 只是在读多写少、替换式更新、延迟回收场景下特别强；更新串行化和内存回收协议仍然必须认真设计。

- **把等待问题硬塞进锁问题**
  线程真正缺的是“何时唤醒”的语义时，只加锁通常只会让临界区更大、等待更糊。

## 7. 应用场景

### 7.1 复杂共享不变量：`mutex` 仍是默认基线

线程池任务队列、连接状态机、缓存元数据、统计与生命周期联合状态，首先都是共享不变量问题。  
这类场景里，默认 `mutex` 的优势不是“简单”，而是它给你最稳的推理边界。

### 7.2 明确读者并发价值：`rwlock` 只在特定负载下有意义

配置快照、路由表、权限表这类读临界区较长且读者确实多的内存结构，才值得认真考虑 `rwlock`。  
这里的关键不是“读多写少”四个字，而是读者并发是否真的值得引入 writer 成本。

### 7.3 不能睡眠的极短临界区：`spinlock`

不能睡眠的内核路径、极短关键区、实时调度下对上下文切换极敏感的局部路径，是 `spinlock` 的典型场景。  
它的使用前提不是“想更快”，而是“确实不能睡，并且锁持有时间极短”。

### 7.4 崩溃后恢复共享状态：`robust mutex`

跨进程共享内存里，持有者崩溃后需要让下一位持有者知道“状态可能不一致”，这时 robust mutex 才真正贴题。  
它解决的是“通知恢复责任”，不是“自动帮你恢复”。

### 7.5 一致快照读取：`seqlock`

类似内核 timekeeping 这种“读者要一致快照、写入很少、读者可以重试”的场景，`seqlock` 很合适。  
这类场景的本质不是读者共享读，而是读者愿意为了轻量读取去重试。

### 7.6 指针结构的读多写少发布：`RCU`

读者遍历指针结构、更新者替换旧版本、旧对象稍后再回收的读多写少结构，是 `RCU` 的典型主场。  
这里最关键的已经不是锁本身，而是发布与回收协议。

## 8. 工业 / 现实世界锚点

### 8.1 POSIX mutex 仍是用户态共享状态保护的基线

Open Group 的 `pthread_mutex_lock()` 文档明确给出 mutex 的 owner 语义：如果 mutex 已被其他线程持有，调用线程会阻塞直到可用；成功返回后，mutex 处于 locked state，且 calling thread 是 owner。  
同一组规范还明确要求 mutex 的 acquire / release 调用执行 memory synchronization。

这意味着在现实用户态程序里，`mutex` 之所以常是默认答案，不只是因为“好写”，而是因为它同时给出：

- 互斥进入
- 明确 owner 语义
- 可推理的同步边

### 8.2 futex 解释了现代阻塞锁为什么“快路径在用户态，慢路径才进内核”

Linux `futex(2)` 手册明确说：futex 通常被用作 shared-memory synchronization 的 blocking construct；大多数同步操作在用户空间完成，只有在线程很可能要阻塞更长时间时才调用内核。  
手册还明确指出，locks 的 uncontended path 可以完全靠用户态原子指令完成，只有竞争时才把 lock flag 当 futex word 交给内核等待 / 唤醒。

这就是现代用户态 mutex / parking lock 的现实结构：

- 无竞争时像原子变量
- 有竞争时才睡眠

### 8.3 Linux kernel 文档把“锁类型”直接分成 sleeping、CPU local、spinning 三大类

`docs.kernel.org/locking/locktypes.html` 明确把 locking primitives 分成 sleeping locks、CPU local locks 和 spinning locks。  
同一份文档还强调：除 semaphores 外，上述 lock types 都有 strict owner semantics；而 semaphore 没有 owner，因此新的 use case 更推荐拆成 serialization 和 waiting 两套机制。

这给“各种锁”最重要的分类启发不是 API 名字，而是：

- 等待时睡不睡
- owner 是否明确
- 是否把“互斥”和“等待”混成一个原语

### 8.4 用户态 spinlock 不是通用优化按钮

`pthread_spin_init(3)` 的 Linux 手册写得很直白：most programs should use mutexes instead of spin locks；spin locks primarily useful with real-time scheduling policies；对 `SCHED_OTHER` 这类 nondeterministic scheduling policy 的使用很可能是 design mistake。  
同一页还明确说：user-space spin locks are not applicable as a general locking solution。

这比“自旋可能更快”更接近现实。  
真正的锚点是：**持锁线程一旦被抢占，别的线程只会空转。**

### 8.5 seqlock 和 RCU 的工业价值都来自内核级读多写少场景

Linux kernel 的 seqlock 文档直接把它定义为 lockless readers + retry loops 的 reader-writer consistency mechanism，并给出典型场景 `system time`。  
同一文档也明确写明：

- 受保护数据不能包含指针
- 写侧临界区绝不能被读侧打断，否则可能 livelock

而 RCU 官方文档则把更新拆成 `removal` 和 `reclamation` 两个阶段：先替换 / 移除引用，再等所有旧读者结束后回收对象。  
这解释了为什么：

- `seqlock` 擅长快照一致性
- `RCU` 擅长指针结构的读多写少发布

## 9. 当前推荐实践、过时路径与替代

以下“当前推荐实践”内容的核对日期为 `2026-03-20`。  
其中有些推荐属于基于 POSIX、Linux man-pages 和 kernel docs 的**工程推断**，我会显式写出。

### 9.1 先用一个同步问题判断表

在真正讨论“该用什么锁”之前，先用下面这张表判断你面对的到底是哪类问题：

| 你的问题形状 | 更推荐的起点 | 为什么 |
| --- | --- | --- |
| 一组字段必须一起保持一致 | `mutex` | 默认最稳，owner 和同步边清晰 |
| 读者很多，但读临界区足够长、写者可接受延迟 | `rwlock` | 只有这时共享读才可能值回复杂度 |
| 线程只是在等某个值或条件变化 | `condition variable` / `atomic_wait` / `semaphore` | 这是等待问题，不是锁家族比较问题 |
| 不能睡眠且临界区极短 | `spinlock` | 前提是确实不能睡，不是因为“看起来更轻” |
| 读者只要一致快照，愿意失败重试 | `seqlock` | 这是快照问题，不是一般共享读问题 |
| 读者跟随指针，更新者替换旧版本并稍后回收 | `RCU` | 这是发布 + 回收问题，不是普通锁问题 |
| owner 可能异常退出，下一位必须知道状态可能脏了 | `robust mutex` | 这是恢复协议入口问题 |

### 9.2 当前更推荐的实践

- 默认先用 `mutex` 保护复杂共享不变量，再拿 profiling 和尾延迟数据证明你真的需要更激进的锁形状。
- 只在读临界区足够长、读者足够多、写者延迟可以接受时再考虑 `rwlock`。这是对 POSIX 读写锁语义和 Linux kernel 公平性差异的工程推断，不是“读多写少就一定更快”的结论。
- 在普通用户态程序里避免把 `spinlock` 当通用优化手段；Linux 手册已经明确说大多数程序应使用 mutex，spin lock 主要适合 real-time scheduling policies。
- 避免把 semaphore 继续当“互斥 + 等待”二合一通用方案；Linux kernel docs 已明确说新的 use case 更推荐把 serialization 和 waiting 分开。对应到用户态，通常就是 `mutex + condition variable` 或 `mutex + atomic_wait` 这类更可推理组合。
- 把 `recursive mutex` 视为旧代码重入兼容层，而不是新设计的默认选项。Open Group 文档明确建议不要把 recursive mutex 和 condition variables 搭配使用。
- 只有在“持有者死亡后必须通知下一位修复共享状态”的场景，才考虑 robust mutex，并且要真正实现 `EOWNERDEAD` 后的恢复逻辑。
- 只有当读者愿意重试快照、或更新路径愿意承担延迟回收时，才进入 `seqlock` / `RCU` 这类高级结构；不要因为“读多写少”四个字就直接跳过去。
- 代码评审时，先要求设计者说明：保护对象是什么、等待线程能不能睡、读者是否可重试、旧对象何时回收；说不清这四点时，不要急着谈 API 名字。

### 9.3 过时路径与替代

- **过时路径：** 把所有同步原语都统称为“锁”。  
  **为什么旧：** 这会把 owner、permit、predicate、single-atomic wait 混成一个模型。  
  **现在更推荐：** 先分清“锁”“等待原语”“准入原语”“lockless 更新协议”。

- **过时路径：** 把 rwlock 当作 mutex 的自然升级版。  
  **为什么旧：** POSIX 已明确说明读写锁的获取和释放会受 writers blocked、priority inversion 和调度策略影响；Linux kernel docs 还说明公平性会因 PREEMPT_RT 配置变化。  
  **现在更推荐：** 把 rwlock 看成特定 workload 下的特化工具，而不是默认加速器。

- **过时路径：** 在一般用户态程序里手写 spinlock 追求“更轻”。  
  **为什么旧：** 现代阻塞锁的快路径已经在用户态，竞争时才通过 futex 进内核；spinlock 反而更容易把抢占和优先级问题放大。  
  **现在更推荐：** 先用 futex-backed 或 runtime-provided blocking mutex；只有明确不能睡眠且临界区极短时才上 spin。

- **过时路径：** 用 recursive mutex 解决锁层级和回调重入。  
  **为什么旧：** 它通常只是在延后暴露锁边界混乱的问题；Open Group 还明确提醒它不适合和 condition variable 搭配。  
  **现在更推荐：** 重画锁边界、拆调用层次、必要时引入显式状态机。

- **过时路径：** 把 seqlock 当成“更快的 rwlock”。  
  **为什么旧：** seqlock 要求读者可重试、写侧不能被打断、数据里不能随意跟随指针。  
  **现在更推荐：** 把它当“快照一致性机制”，不是一般共享结构锁。

- **过时路径：** 看到等待线程就先上锁，再想怎么把它唤醒。  
  **为什么旧：** 这会把不变量保护和等待谓词混成一个问题，经常造成大临界区和糊涂同步。  
  **现在更推荐：** 先把“保护状态”和“等待条件”拆开，再决定是 `mutex + condition variable`、`atomic_wait` 还是别的方案。

## 10. 自测题 / 验证入口

1. 你面对的是“复杂共享不变量”“许可数量”“单个状态位变化”“读多写少快照”，还是“读者跟随指针、更新者稍后回收”？
2. 为什么 `rwlock` 不是“读多写少”四个字一出现就该默认启用的优化？
3. 为什么用户态 spinlock 常常不是比 mutex 更“轻”，而是更脆？
4. `seqlock` 为什么不能直接拿来保护包含指针的数据结构？
5. 在 owner 死亡后要恢复共享状态时，robust mutex 给了你什么，没替你做什么？
6. 什么时候你真正该比较的是 `condition variable`、`atomic_wait` 和 semaphore，而不是各种锁家族？
7. 如果一个读多写少结构里的读者会跟随指针，为什么 `RCU` 和 `rwlock` 才是同一层级的候选，而 `seqlock` 不是？
8. 你在代码评审里看到“为了性能把 mutex 改成 spinlock”，最先该追问哪三个约束？

## 11. 迁移与关联模型

理解了这篇文档后，你可以把模型迁移到：

- **线程池和任务队列**
  先判断是队列不变量保护问题，还是“有无任务可取”的等待问题。

- **缓存和配置热更新**
  先问读者是否真的需要锁，还是可以接受版本切换和延迟回收。

- **内核 / 低延迟系统**
  把“能不能睡眠”“是否会被抢占”作为首要轴，而不是把 API 名字直接照搬到用户态。

- **跨进程共享内存**
  把 robust mutex、futex、shared memory 和恢复协议放进同一张图里。

- **lock-free 设计入口**
  当你发现自己想要“无 owner、少等待、读者几乎无成本”时，可以顺着这篇文档走向 `atomic_wait`、hazard pointers、RCU 或其他 lock-free / wait-free 结构。

最值得保留的迁移句式是：

**先问你保护的是哪类对象，再问等待时睡不睡、读者能不能并发、能不能重试，最后才问 API 叫 mutex、rwlock、spinlock 还是别的名字。**

## 12. 未解问题与继续深挖

- 用户态公平锁、MCS / CLH 队列锁、NUMA-aware 锁在现代服务器负载下的收益边界，是否值得单独做一篇“高阶锁实现”文档？
- `shared_mutex`、`rw_semaphore`、平台自带 SRWLock / parking_lot 这类实现，在公平性和尾延迟上各自最容易踩什么坑？
- 当协程调度器和 OS 线程锁叠加时，“持锁阻塞”的后果会如何被放大？

## 13. 参考资料

以下外部资料中涉及“当前实践”的内容，均于 `2026-03-20` 核对。

- Open Group, `pthread_mutex_lock()`: https://pubs.opengroup.org/onlinepubs/9799919799/functions/pthread_mutex_lock.html
- Open Group, `pthread_mutexattr_gettype()` / `pthread_mutexattr_settype()`: https://pubs.opengroup.org/onlinepubs/9699919799/functions/pthread_mutexattr_gettype.html
- Open Group, Memory Synchronization: https://pubs.opengroup.org/onlinepubs/9799919799/basedefs/V1_chap04.html
- Open Group, `pthread_rwlock_rdlock()`: https://pubs.opengroup.org/onlinepubs/9799919799/functions/pthread_rwlock_rdlock.html
- Open Group, `pthread_rwlock_unlock()`: https://pubs.opengroup.org/onlinepubs/9699919799/functions/pthread_rwlock_unlock.html
- Linux man-pages, `pthread_spin_init(3)`: https://man7.org/linux/man-pages/man3/pthread_spin_init.3.html
- Linux man-pages, `pthread_spin_lock(3)`: https://man7.org/linux/man-pages/man3/pthread_spin_lock.3.html
- Linux man-pages, `futex(2)`: https://www.man7.org/linux/man-pages/man2/futex.2.html
- Linux man-pages, `sem_overview(7)`: https://man7.org/linux/man-pages/man7/sem_overview.7.html
- Linux kernel docs, Lock types and their rules: https://docs.kernel.org/locking/locktypes.html
- Linux kernel docs, Sequence counters and sequential locks: https://docs.kernel.org/locking/seqlock.html
- Linux kernel docs, What is RCU?: https://docs.kernel.org/RCU/whatisRCU.html
