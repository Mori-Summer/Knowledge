---
doc_id: computer-systems-dma
title: DMA 技术：地址可达性、缓冲区所有权与可见性同步的统一模型
concept: dma
topic: computer-systems
depth_mode: deep
created_at: '2026-03-23T16:32:00+08:00'
updated_at: '2026-03-23T16:32:00+08:00'
source_basis:
  - linux_dma_api_docs_baseline_2026_03_23
  - linux_dma_attributes_docs_baseline_2026_03_23
  - iommu_translation_and_mapping_model_baseline_2026_03_23
  - pcie_bus_mastering_and_descriptor_ring_pattern_baseline_2026_03_23
  - embedded_dma_controller_pattern_baseline_2026_03_23
time_context: standards_and_engineering_practice_baseline_2026_03_23
applicability: driver_design_buffer_mapping_performance_debugging_device_io_reasoning_and_system_architecture
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/computer-systems/cache-coherence.md
  - docs/computer-systems/virtual-memory-learning-model.md
  - docs/computer-systems/usb-transfer-protocol.md
  - docs/computer-systems/usb-host-controller-data-path.md
open_questions:
  - 在 IOMMU、SVA/PASID 和共享虚拟地址逐渐普及后，通用驱动对“DMA 地址空间”的抽象边界会怎样变化？
  - 面向高吞吐 AI/视频加速器的 DMA 路径里，哪些场景最值得单独拆成“缓存一致性与同步策略”文档？
  - 是否需要再补一篇专门解释 DMA ring、doorbell、completion queue 这套模式如何跨 NVMe、NIC、USB、GPU 复用的文档？
---

# DMA 技术：地址可达性、缓冲区所有权与可见性同步的统一模型

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立的，不是“DMA 就是不经过 CPU 直接搬数据”这句半对半错的定义，而是一套以后能反复调用的内部模型：

- 你能解释为什么 DMA 的关键不是“绕过 CPU”，而是“把一次数据搬运委托给另一个总线主设备或 DMA 引擎”
- 你能区分虚拟地址、物理地址、总线地址、IOVA、设备可见地址这些常被混写的对象
- 你能判断 DMA 正确性为什么总是同时取决于地址映射、缓冲区生命周期、缓存可见性、顺序和完成通知
- 你能分析为什么同一段 DMA 代码在 x86 上看起来“没问题”，到了 non-coherent 平台、IOMMU 环境或另一个设备上就会出错
- 你能把 DMA 模型迁移到 NVMe、网卡、USB 主机控制器、GPU、摄像头采集卡和 MCU 外设上

如果只记一句话：

**DMA 不是“神奇零成本搬运”，而是一份跨 CPU、设备、内存和地址翻译层的契约：设备只有在地址可达、缓冲区稳定、缓存可见、顺序正确且完成信号被消费时，才算真的完成了一次正确的数据移动。**

## 2. 一句话结论 / 问题定义

**DMA 是一种把数据搬运责任交给设备或独立 DMA 引擎的机制：CPU 负责配置描述符、映射地址和同步边界，设备再通过总线主控能力直接读写内存，从而减少逐字节搬运开销并支持高吞吐、低占用和并行 I/O。**

它真正要解决的不是“省一次 `memcpy`”，而是五件事同时成立：

- 设备能访问到正确的内存区域
- 这块内存在 DMA 生命周期内保持稳定可用
- CPU 与设备看到的是同一份数据语义，而不是各自缓存里的不同版本
- 触发和完成之间的顺序不会被重排序破坏
- 错误、完成和回收能被系统正确消费

可以把 DMA 正确性压成一个五元组：

**DMA correctness = 地址可达 + 生命周期稳定 + 缓存可见 + 顺序约束 + 完成闭环**

少任意一项，DMA 就可能“看起来传了，实际上没成”。

## 3. 对象边界与相邻概念

### 3.1 这篇文档直接处理什么

这篇文档直接处理的是：

- 设备或 DMA 引擎如何直接访问内存
- 地址映射、IOMMU/IOVA、buffer pinning、scatter-gather 和 descriptor ring
- CPU 与设备之间的数据所有权切换
- coherent / non-coherent、streaming / consistent DMA 的工程边界

### 3.2 它不等于什么

它不等于：

- **MMIO。** MMIO 是 CPU 访问设备寄存器；DMA 是设备访问内存。
- **cache coherence 本身。** coherence 只覆盖某些参与者和域，DMA 还必须回答设备是否在这个域里、是否需要显式同步。
- **零拷贝。** DMA 常被拿来实现零拷贝，但二者不是同义词。
- **物理连续内存。** 很多现代 DMA 通过 scatter-gather 和 IOMMU 工作，并不要求整个缓冲区天然物理连续。
- **只有高速设备才需要的东西。** MCU 上的 UART/SPI/ADC 也常用 DMA，只是规模和约束不同。

### 3.3 最值得一起看的相邻概念

最值得一起看的相邻概念是：

- [Cache Coherence：同一份数据为什么不会在不同核心缓存里“各说各话”](/Users/maxwell/Knowledge/docs/computer-systems/cache-coherence.md)
- [虚拟内存：地址抽象、访问控制与工作集行为的统一模型](/Users/maxwell/Knowledge/docs/computer-systems/virtual-memory-learning-model.md)
- [USB 传输协议：主机调度、端点契约与传输类型取舍的统一模型](/Users/maxwell/Knowledge/docs/computer-systems/usb-transfer-protocol.md)
- [USB 主机控制器数据路径：从 URB、传输环到 DMA 缓冲区的统一模型](/Users/maxwell/Knowledge/docs/computer-systems/usb-host-controller-data-path.md)
- PCIe bus mastering、NVMe queue、NIC descriptor ring、IOMMU

### 3.4 五组最容易混淆的边界

最常见的混淆有五组：

- **CPU 地址 vs 设备地址。**
  CPU 能拿着虚拟地址访问，不等于设备也能拿这串地址去 DMA。

- **coherent memory vs streaming DMA。**
  两者都叫“DMA buffer”，但生命周期和同步责任完全不同。

- **映射成功 vs 传输正确。**
  地址能映射，只解决“设备找得到”；不解决缓存、顺序和完成语义。

- **设备把数据写完 vs CPU 现在就能安全读。**
  中间可能还差 cache invalidation、memory barrier 或 event completion。

- **高吞吐 offload vs 总体更快。**
  DMA 省 CPU，不等于对所有小消息都更划算；建立描述符、映射和同步也有成本。

### 3.5 本文默认适用边界

本文默认工作边界是：

- 以现代 OS 驱动与系统工程视角为主，同时覆盖 MCU/SoC 上的典型 DMA 直觉
- 关注“设备如何访问内存”这一层，不深入某个具体芯片寄存器手册
- 在“当前推荐实践”部分按 `2026-03-23` 的通用工程基线组织

## 4. 核心结构

### 4.1 最稳的统一模型：六个平面

把 DMA 建模得足够稳，最好的方式是拆成六个平面：

| 平面 | 它解决什么 | 典型对象 |
| --- | --- | --- |
| 请求者平面 | 谁在发起 DMA | PCIe 设备、USB 主机控制器、NIC、NVMe、片上 DMA 引擎 |
| 内存对象平面 | 数据实际落在哪 | payload buffer、descriptor ring、completion queue |
| 地址翻译平面 | 设备看到的地址是什么 | bus address、IOVA、IOMMU、scatter-gather list |
| 可见性平面 | CPU 和设备如何看到同一份数据 | cache coherency、flush/invalidate、DMA sync |
| 顺序与触发平面 | 谁先写描述符、谁后按门铃启动 | doorbell、memory barrier、producer/consumer ownership |
| 完成与回收平面 | 怎么知道这次搬运真的结束 | interrupt、event ring、polling、unmap、recycle |

只要这六个平面还没回答完，“DMA 正常工作”就不应被视为已解释。

### 4.2 八个关键结构件

1. **DMA requestor**
   真正发起内存读写的一方，可能是设备本体，也可能是片上的独立 DMA controller。

2. **源与目的缓冲区**
   一次 DMA 总是有源和目的，只是方向可能是 device-to-memory、memory-to-device 或 memory-to-memory。

3. **地址映射对象**
   设备通常不直接理解 CPU 虚拟地址；系统要把 CPU 侧缓冲区映射成设备可达地址。

4. **描述符与队列**
   高性能 DMA 很少靠单一寄存器描述一次搬运，而是通过 ring、descriptor、SG list 表达工作项。

5. **一致性与同步机制**
   是否硬件 coherent、是否需要软件 sync、是否允许 write combining，都决定数据何时对另一侧真正可见。

6. **启动触发**
   描述符写好后，通常还需要 doorbell、寄存器写入或 enable bit 告诉设备“现在可以开始读了”。

7. **完成通知**
   设备搬完后必须给系统一个 completion 面，否则 CPU 不知道何时安全消费数据。

8. **回收与复用**
   缓冲区、映射和描述符不可能无限增长，DMA 设计必须包含 recycle 策略。

### 4.3 三类最重要的 DMA 形态

| 形态 | 核心特征 | 典型对象 |
| --- | --- | --- |
| Consistent / Coherent DMA | 控制结构长期映射，双方共享同一语义视图 | ring、doorbell 周边控制块 |
| Streaming DMA | payload 在一次或少数几次传输中短期映射 | 网卡包缓冲区、块设备 I/O 缓冲区 |
| Scatter-Gather DMA | 一次工作项可跨多个内存片段 | NIC、NVMe、USB 控制器、GPU |

最危险的误判之一，就是把这三类看成“只是 API 名字不同”。

### 4.4 地址空间不要只记两个，要记四个

理解 DMA 时，最稳的方式不是只分“虚拟地址”和“物理地址”，而是至少分四个：

| 地址空间 | 谁在用 | 为什么重要 |
| --- | --- | --- |
| 用户虚拟地址 | 应用 / 用户态 | 设备通常看不见 |
| 内核虚拟地址 | 驱动 / 内核 | 设备通常也不直接看见 |
| CPU 物理地址 | MMU 后的物理页框 | 不一定等于设备能看到的总线地址 |
| 设备可见地址 | bus address / IOVA | 这是 DMA 真正使用的地址 |

IOMMU 的存在意味着：

**“设备地址 = 物理地址”并不是现代系统里可以默认成立的前提。**

### 4.5 一页纸判断模板

以后遇到 DMA 问题，可以先问这六个问题：

1. 设备实际拿到的是什么地址？
2. 这块缓冲区在 DMA 生命周期内是否稳定、未被释放或迁移？
3. 设备和 CPU 是否处在同一个 coherence domain？
4. doorbell 之前是否确保描述符和 payload 已对设备可见？
5. completion 到来后，CPU 是否完成了必要同步才开始读写？
6. 这次 DMA 是 payload streaming，还是长期共享控制结构？

## 5. 核心机制 / 主链路 / 因果链

### 5.1 通用主链：一次 streaming DMA 如何完成

一次典型 streaming DMA 可以压成下面这条链：

1. **CPU 准备缓冲区**
   分配或借用一块内存，填入要发的数据，或准备接收空间。

2. **CPU 建立设备可达映射**
   通过 OS 的 DMA API 获取设备可见地址，必要时建立 IOMMU 映射，并处理 cache sync。

3. **CPU 写描述符或寄存器**
   把设备地址、长度、方向、flags 写到设备可消费的位置。

4. **CPU 发出启动信号**
   通过 doorbell、寄存器位或队列推进，告诉设备去取工作项。

5. **设备发起总线事务**
   设备读取描述符，再按其中地址对内存发起 DMA read/write。

6. **设备写回完成状态**
   把结果写进 completion queue、status block 或寄存器，并触发 interrupt/polling 可见事件。

7. **CPU 消费完成并同步**
   对 RX 路径可能需要 invalidate/sync for CPU；对 TX 路径可能需要 unmap/recycle buffer。

这条链最重要的认知点是：

**DMA 真正的主链不是“设备自动搬了”，而是“CPU 先交控制权，再由设备按约定的地址和顺序执行，最后再把控制权交回来”。**

### 5.2 方向不同，责任不同

DMA 至少有两条最关键分支：

- **memory-to-device**
  CPU 必须确保 payload 在 doorbell 前对设备可见。

- **device-to-memory**
  CPU 必须确保在设备写完并完成同步前，不提前读取旧缓存内容。

很多 bug 都来自把 TX 和 RX 方向当成“同一段代码换个枚举值”。

### 5.3 为什么 cache coherency 不等于 DMA 正确

即使平台声称 CPU cache coherent，也不能直接推出 DMA 一切正确，因为还缺：

- 映射方向是否正确
- 描述符和 payload 生命周期是否正确
- doorbell 与 completion 前后的内存顺序是否正确
- 设备是否真的在同一 coherence domain

所以更稳的判断是：

**coherency 只帮你减少一部分可见性痛点，不替你回答地址、所有权和顺序问题。**

### 5.4 Descriptor ring 模式的通用主链

在 NVMe、NIC、USB xHCI、很多 GPU 和加速器里，DMA 常表现为 ring 模式：

1. CPU 拥有 producer 位置
2. CPU 写入 descriptor/entry
3. CPU 更新 tail 或写 doorbell
4. 设备按自己的 pace 拉取 descriptor
5. 设备执行 DMA 并写 completion
6. CPU 消费 completion 并回收 slot

这条链的价值是：它几乎可以迁移到大量设备上。  
理解了 ring，你就不再把每个设备都当全新世界。

### 5.5 一个必须记住的统一判断

**DMA 的本质不是数据“会不会被搬”，而是“搬运责任在什么时候从 CPU 移交给设备，又在什么时候被安全拿回来”。**

## 6. 关键 tradeoff 与失败模式

### 6.1 DMA 买到什么，又付出什么

DMA 通常买到的是：

- 降低 CPU 逐字节搬运负担
- 让 I/O 与 CPU 并行
- 支持更高吞吐和更稳定的数据流
- 为 descriptor queue、batching、zero-copy 提供基础

付出的代价是：

- 地址映射复杂度
- 缓冲区生命周期治理
- cache / barrier / ownership 语义
- 调试成本显著上升

### 6.2 七类常见失败模式

1. **把 CPU 虚拟地址直接塞给设备**
   在没有正确 DMA mapping 的环境里，这通常立刻是错的。

2. **过早释放或复用缓冲区**
   设备还在 DMA，CPU 就把 buffer 交给别的逻辑复用了。

3. **漏掉 cache sync**
   非 coherent 平台尤其容易中招；症状常是“偶现旧数据”。

4. **方向写错**
   map/sync/unmap 的 direction 一旦错，调试会非常痛苦。

5. **doorbell 前缺 barrier**
   设备先看到 tail，再看到旧描述符内容，形成“偶现乱包”。

6. **IOMMU fault 或地址位宽不匹配**
   32 位设备、受限 DMA mask、错误 SG 映射都会导致设备根本触不到目标页。

7. **把小消息也机械 DMA 化**
   对极小 payload，映射、同步和完成开销可能压过好处。

### 6.3 三个特别危险的认知错觉

- **“x86 上没问题，所以代码是对的。”**
  这通常只说明 x86 的 coherency 和容错暂时把 bug 藏起来了。

- **“DMA 就是零拷贝。”**
  中间可能仍有 bounce buffer、IOMMU remap 或 cache maintenance。

- **“设备写完中断了，我就能直接读。”**
  completion 事件只是闭环的一部分，不一定自动完成所有 CPU 可见性动作。

## 7. 应用场景

### 7.1 高吞吐存储

NVMe、AHCI、RAID 控制器都重度依赖 DMA ring 和 completion queue。  
没有 DMA，存储 I/O 几乎不可能达到现代吞吐水平。

### 7.2 网络收发

NIC 的 RX/TX 队列是 DMA 模型最典型的现实场景。  
包缓冲区、descriptor ring、doorbell、completion 几乎把本篇模型原样映射出来。

### 7.3 USB 主机控制器与外设采集

USB host controller 会通过 DMA 拉取传输描述符、搬运 payload、写回事件。  
外设采集卡、摄像头、声卡也常在 DMA 和实时流预算之间做平衡。

### 7.4 MCU / SoC 外设搬运

ADC、SPI、I2S、UART 在 MCU 上常配合 DMA 把 CPU 从重复搬运里解放出来。  
虽然没有复杂 IOMMU，但 ownership、circular buffer、completion 语义仍然成立。

## 8. 工业 / 现实世界锚点

### 8.1 NVMe Submission Queue / Completion Queue

NVMe 是 DMA queue 模型最经典的锚点之一。  
主机写 SQ entry、敲 doorbell，控制器 DMA 读命令、DMA 写数据和 CQE，这几乎把现代设备 DMA 主链完整展示出来。

### 8.2 网卡 Descriptor Ring

现代 NIC 的 RX/TX ring 是理解 DMA 生命周期、buffer recycling 和 cache / barrier 问题的最佳现实材料。  
大量“偶发丢包”“包内容旧”“性能突然掉”的问题都能回到这里。

### 8.3 Linux DMA API

Linux DMA API 是工程上最重要的现实锚点之一，因为它把“虚拟地址不能直接给设备”“coherent 与 streaming 不同”“需要 map/sync/unmap”的纪律直接制度化了。

### 8.4 MCU Circular DMA

嵌入式系统里的 circular DMA 是另一个极好锚点。  
它提醒你：即便没有 PCIe、IOMMU 和复杂队列，DMA 依然绕不开 ownership、completion 和 buffer wraparound。

## 9. 当前推荐实践、过时路径与替代

以下“当前实践”判断按 `2026-03-23` 的通用 Linux / Windows / embedded 工程视角组织。

### 9.1 当前更稳的实践

当前更稳的 DMA 工程实践通常是：

- 永远通过 OS / driver framework 提供的 DMA mapping API 获取设备地址，不手搓“看起来像物理地址”的值
- 明确区分长期共享控制结构和一次性 payload buffer，不把 coherent 与 streaming 混用
- 任何高吞吐设备都优先按 ring + descriptor + completion 的统一模型来理解
- 在 non-coherent 或平台差异较大的环境里，先保守处理 cache sync，再做性能优化
- 调试 DMA 时同时看三层：地址映射、所有权切换、completion 闭环，而不是只看设备寄存器

### 9.2 过时或危险的路径

下面这些路径已经不够稳：

- “驱动里拿到的地址就是设备能 DMA 的地址”
- “x86 coherent，所以不需要 DMA API”
- “只要设备中断来了，CPU 读取一定没问题”
- “为了性能，所有 buffer 都长期 pin 住再说”

这些路径的共同问题是：它们把平台、IOMMU、生命周期和缓存边界统统当成不存在。

### 9.3 替代路径与何时不该上 DMA

有些场景不该机械上 DMA：

- payload 很小、频率不高、CPU 空闲充足时，简单 `memcpy` 反而更稳
- 如果真正瓶颈是协议处理、上下文切换或锁争用，DMA 不是第一杠杆点
- 如果设备不在关键吞吐路径上，把系统复杂度换成 DMA 未必划算

更稳的替代是：

- 先确认瓶颈在“数据搬运”
- 再决定是 `memcpy`、PIO、DMA 还是更高层批处理 / queue 化

## 10. 自测题 / 验证入口

1. 为什么“DMA = 不经过 CPU”这个说法不够准确？
2. 为什么设备可见地址不能简单等同于 CPU 虚拟地址？
3. coherent platform 为什么仍然不能自动证明 DMA 路径完全正确？
4. streaming DMA 和长期共享控制结构的同步责任有何不同？
5. 为什么 doorbell 前后的 barrier 可能决定一段 DMA 代码是否偶现损坏？
6. 如果设备中断已到，但 CPU 读到旧数据，最可能该从哪几层查起？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- NIC / NVMe / GPU / USB host controller 的 ring 与 completion 设计
- [USB 主机控制器数据路径：从 URB、传输环到 DMA 缓冲区的统一模型](/Users/maxwell/Knowledge/docs/computer-systems/usb-host-controller-data-path.md)
- [Cache Coherence：同一份数据为什么不会在不同核心缓存里“各说各话”](/Users/maxwell/Knowledge/docs/computer-systems/cache-coherence.md)
- IOMMU、bounce buffer、zero-copy 网络、用户态 I/O 框架
- MCU 上的 circular DMA 与双缓冲设计

最值得迁移的结构不是某个 API，而是：

**地址翻译 -> 所有权移交 -> 设备执行 -> 完成回收**

## 12. 未解问题与继续深挖

- 通用驱动框架在 SVA/PASID、更细粒度共享地址空间下会怎样重写“map/unmap”的抽象？
- heterogeneous memory、CXL 和设备本地内存扩展后，DMA 的“内存可达性”会怎样继续分层？
- 哪些设备最值得单独写一篇“descriptor ring 与 completion queue 模式”的横向对比文档？

## 13. 参考资料

以下资料构成本文的工程模型基础；“当前实践”部分的组织日期为 `2026-03-23`。

- Linux DMA API documentation
- Linux DMA attributes documentation
- IOMMU / DMA mapping model documentation
- PCIe bus mastering and descriptor ring design patterns
- Embedded SoC / MCU DMA controller programming models
