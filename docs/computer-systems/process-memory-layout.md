---
doc_id: computer-systems-process-memory-layout
title: 进程的内存布局：从 ELF 装载到堆、mmap 区、栈与共享库的统一模型
concept: process_memory_layout
topic: computer-systems
created_at: '2026-03-19T19:41:43+08:00'
updated_at: '2026-03-19T20:50:00+08:00'
source_basis:
  - linux_execve_manpage_checked_2026_03_19
  - linux_proc_pid_maps_manpage_checked_2026_03_19
  - linux_proc_pid_smaps_manpage_checked_2026_03_19
  - linux_mmap_manpage_checked_2026_03_19
  - linux_brk_manpage_checked_2026_03_19
  - linux_vdso_manpage_checked_2026_03_19
  - glibc_malloc_manpage_checked_2026_03_19
  - linux_dynamic_linker_manpage_checked_2026_03_19
  - elf_gabi_program_header_spec_checked_2026_03_19
  - linux_kernel_randomize_va_space_docs_checked_2026_03_19
time_context: current_practice_checked_2026_03_19
applicability: operating_system_reasoning_loader_analysis_memory_debugging_and_process_observability
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/computer-systems/virtual-memory-learning-model.md
open_questions:
  - 是否需要补一篇专门解释 PIE、ASLR、RELRO 与地址稳定性预期的配套文档？
  - 是否需要进一步比较 glibc、jemalloc、tcmalloc、JVM 在地址空间占用形态上的差异？
---

# 进程的内存布局：从 ELF 装载到堆、mmap 区、栈与共享库的统一模型

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立一个能反复调用的内部模型：看到一个进程的地址空间时，你能把它还原成“是谁把哪些对象映射到哪里，为什么会长成这样，以及接下来还会怎样变化”。

读完后，你应该至少能做到：

- 不再把“代码段 / 数据段 / 堆 / 栈”那张教学图当成完整现实，而是把进程布局理解成一组持续变化的映射区
- 解释 `execve()`、ELF 程序头、动态链接器、`brk()`、`mmap()`、线程和 ASLR 如何共同塑形一个进程的地址空间
- 看 `/proc/<pid>/maps` 或 `/proc/<pid>/smaps` 时，判断某块地址大概率属于主程序映像、共享库、堆、匿名映射、线程栈还是内核注入对象
- 分析“RSS 涨了但 heap 指标没怎么变”“同一程序每次地址都不同”“线程一多内存图就很碎”这类常见现象
- 把这个模型迁移到调试段错误、解释 allocator 行为、理解动态装载和分析容器内内存问题上

## 2. 一句话结论 / 问题定义

**进程的内存布局不是一张固定示意图，而是进程虚拟地址空间在 `execve()` 之后，由 ELF 装载规则、动态链接器、内核虚拟内存子系统、运行时分配器、线程机制和地址随机化共同形成的一组映射结果。**

它真正要解决的问题不是“把内存分成几块”，而是让下面这些事情同时成立：

- 代码、只读数据、可写数据、未初始化数据能按权限和装载规则进入地址空间
- 程序在运行中还能继续申请新内存、装入共享库、建立文件映射和启动新线程
- 地址空间既能支持共享和懒加载，又能支持隔离、随机化和调试观测
- 用户看到的是统一地址空间，底层却允许不同来源的映射以不同生命周期共存

## 3. 对象边界与相邻概念

这个概念直接处理的是：

- 一个进程的用户态虚拟地址空间如何被切分、映射、增长和观测
- 每块映射区的地址范围、权限、是否私有 / 共享、是否有文件后备、来自哪个装载阶段
- 进程从启动到运行期，哪些新对象会继续进入地址空间

它不等于：

- `虚拟内存`：虚拟内存是更底层的地址抽象、页表、驻留和缺页机制；进程内存布局是这个机制在单个进程上的组织结果
- `malloc/new`：分配器只是布局演化的参与者之一，不是布局本体
- `物理内存使用量`：地址空间很大，不代表实际驻留的 RAM 很大
- `文件系统`：文件可以映射进地址空间，但文件系统不等于进程布局

最值得一起看的相邻概念是：

- ELF 可执行文件与 program header，而不是只看 section 名字
- `mmap(2)`、`brk(2)` 与用户态分配器
- 动态链接器 `ld-linux.so`
- 线程栈、TLS 与 guard page
- ASLR / PIE / vDSO
- `/proc/<pid>/maps`、`/proc/<pid>/smaps`、`readelf -l`

## 4. 核心结构

最稳的建模方式不是记“5 个大块”，而是把布局看成一组带属性的映射区。

在现代 64-bit Linux ELF 用户态进程里，常见形态大致像这样：

```text
high addresses
[main thread stack | argv | envp | auxv]
[shared libraries | dynamic linker | special mappings such as vdso]
[anonymous/file-backed mmap regions | allocator arenas | mapped files]
[heap via program break]
[writable data | bss]
[text | rodata | other executable image mappings]
low addresses
```

这张图只能当方向感，不能当固定地址真相。实际位置会受到架构、ABI、是否 PIE、是否启用 ASLR、线程数、分配器和运行时行为影响。

真正要记的结构件有 8 个：

1. **主程序映像**
   `execve()` 之后，内核根据 ELF program header 的 `PT_LOAD` 等条目建立初始映射。课本上的 `.text`、`.rodata`、`.data`、`.bss` 是链接视角的 section；运行时真正驱动装载的是 segment。
2. **零填充尾部与 `bss`**
   ELF gABI 规定：若某个 `PT_LOAD` 段的内存尺寸大于文件尺寸，超出的那部分在内存中补零。这就是为什么未初始化数据能在运行时占地址空间，却不需要完整存进可执行文件。
3. **`brk` 管理的 heap**
   程序 break 标记数据段末端之后的位置，向上推进 break 就扩展 heap。它仍是现实布局中的一部分，但已不再等于“所有动态分配内存”。
4. **`mmap` 区**
   文件映射、匿名映射、大块分配、共享内存、JIT 或特殊运行时对象通常落在这里。现代进程里，这一带往往比传统 heap 更能解释地址空间的复杂性。
5. **共享库与动态链接器**
   如果 ELF 带有 `PT_INTERP`，内核会按其中记录的解释器路径启动动态链接器；后者再把依赖的共享对象映射进来、做重定位并准备运行环境。
6. **主线程栈**
   初始栈不只是函数调用栈，还承载启动时的 `argv`、`envp` 和辅助向量。它通常位于较高地址，并伴随向下增长的语义。
7. **线程相关对象**
   新线程会带来额外线程栈、TLS 相关运行时对象和 guard page。线程数上升时，布局会变得更碎，匿名映射与栈相关区间显著增加。
8. **特殊内核注入映射**
   现代 Linux 会自动把 vDSO 映射进每个用户进程，让某些高频系统调用路径可以退化成用户态函数调用加少量内存访问，而不是每次都陷入内核。

## 5. 核心机制 / 主链路 / 因果链

把进程布局压成一条主链路，可以这样理解：

1. `execve()` 替换旧进程映像，重新初始化新的栈、heap 和数据段。
2. 内核读取 ELF program header，而不是只看 section 名字；`PT_LOAD` 决定哪些文件字节要映射进内存、映射到什么虚拟地址、带什么权限。
3. 如果某个 load segment 满足 `p_memsz > p_filesz`，文件里不存在的尾部在内存里按零填充，于是得到运行时可见的 `bss`。
4. 如果 ELF 含 `PT_INTERP`，内核把控制权交给该条目指定的动态链接器；动态链接器继续映射 `libc` 等共享库，建立重定位和符号解析所需状态，然后才真正开始执行程序。
5. 内核把初始参数、环境变量和辅助向量放到主线程栈上；vDSO 的基址也通过辅助向量传给进程。
6. 进入运行期后，分配器开始塑形布局。以 glibc 为现实锚点，常规分配通常从 heap 方向增长；超过 `MMAP_THRESHOLD` 的较大块默认改用私有匿名 `mmap()`；多线程竞争下还会建立额外 arena，而 arena 自身可能来自 `brk()` 或 `mmap()`。
7. 程序调用 `mmap()` 时，内核会在当前地址空间里选择不与已有映射冲突的区间；如果强行用 `MAP_FIXED` 指定地址，则可能直接覆盖原映射，因此手工固定地址是高风险操作。
8. 每创建一个新线程，地址空间通常再加入该线程的栈和相关运行时对象；因此“线程数”本身就是布局形态的自变量。
9. 若系统启用 ASLR，`mmap` 基址、栈、vDSO、共享库地址都会随机化；对 PIE 可执行文件，主程序代码起始位置也会随机化；在 full randomization 模式下，heap 也会随机化。

这条链最关键的因果不是“先有一块 heap，再有一块 stack”，而是：

**二进制装载规则决定初始骨架，动态链接器补齐共享对象，运行时分配与线程活动让布局持续演化，ASLR 让同一骨架在不同运行间以不同地址实例化。**

## 6. 关键 tradeoff 与失败模式

- `brk` 型 heap 把一部分动态分配集中在连续区域里，便于形成“传统堆”的直觉；但一旦程序大量依赖 `mmap`、多 arena 或自定义分配器，这个直觉就会迅速失真。
- `mmap` 让大块内存和文件映射拥有独立生命周期，更容易按映射粒度回收；代价是映射数增加、布局更碎，并可能碰到 `max_map_count` 这类系统上限。
- 共享库、PIE 和 ASLR 提升了共享性与安全性；代价是地址不再稳定，任何假设“这个库永远在那个地址”的代码、脚本和调试习惯都会变脆。
- 线程提供并发，但每个线程都不是“只有几 KB 元数据”；线程栈、TLS 和分配器 arena 都会侵入地址空间，并抬高观测复杂度。
- 文件映射和共享对象把启动和拷贝成本后移到首次访问与缺页时刻；这提高了效率，也让“启动快但第一次访问慢”变成常见现象。

最常见的失败模式是：

- 把“进程内存布局”仍然画成五块固定长方形，完全看不到共享库、匿名映射、线程栈和 vDSO
- 把“heap”误当成“所有动态内存”，从而在 RSS 增长时错过匿名 `mmap` 区、额外 arena 或线程栈
- 只看 VSS 或一个总内存数字，不去区分映射大小、实际驻留、共享份额和 swap
- 认为同一程序每次运行地址应该差不多，于是把 ASLR/PIE 造成的变化误判成异常
- 用老文章里对 `/proc/<pid>/maps` 的经验直接套当前系统，例如还期待看到 `[stack:tid]` 标签，却不知道这个字段自 Linux 4.5 起已从该文件移除
- 在调试栈溢出、深递归或 `alloca()` 问题时，仍按“只要还有 heap 就没事”的错误模型判断风险

## 7. 应用场景

这套模型最常用于：

- **段错误和非法访问定位**：拿 fault address 对照 `maps`，先判断它落在代码段、共享库、stack 附近，还是压根落在 unmapped hole
- **RSS 异常增长分析**：区分增长来自 `[heap]`、匿名 `mmap`、文件映射、线程栈还是共享库装载
- **解释启动与装载行为**：为什么一个 ELF 程序一启动就带上 `ld-linux.so`、`libc.so` 和 `vdso`
- **理解线程成本**：为什么线程变多时，地址空间和匿名映射数上升得比业务对象更快
- **设计文件映射与零拷贝路径**：判断什么时候该用 `mmap` 映射文件，而不是把一切都塞进用户态 heap
- **安全与可复现实验**：理解 ASLR / PIE 打开后，为什么地址相关 exploit、回溯地址记录和实验复现方式都要调整

## 8. 工业 / 现实世界锚点

### 8.1 `/proc/<pid>/maps` + `readelf -l` 是生产环境里的真实观测面

Linux 官方手册明确把 `/proc/<pid>/maps` 定义为“当前映射区及其权限”的视图，并指出对 ELF 文件可以把 `maps` 中的 `offset` 与 `readelf -l` 里的 program header offset 对起来看。  
这意味着生产排障时，真正有效的做法不是背抽象图，而是把“文件里的段”和“运行时的映射”一一对齐。

### 8.2 `/proc/<pid>/smaps` 解决的是“哪块地址真正吃了 RAM”

`smaps` 会按映射区给出 `Rss`、`Pss`、`Anonymous`、`Swap`、`KernelPageSize`、`MMUPageSize` 等指标。  
所以当线上 C/C++、Python、Java 服务出现“总内存高但 heap 仪表盘解释不完”的情况，真正的切入口通常是 mapping 级别，而不是先争论“是不是 heap leak”。

### 8.3 glibc 分配器会把多线程服务的布局变复杂

`malloc(3)` 的当前手册明确写到：glibc 默认通常从 heap 分配，但较大块会走匿名 `mmap()`，而在多线程争用下还会创建额外 arena，每个 arena 可来自 `brk()` 或 `mmap()`。  
这就是为什么线程较多的 Linux 服务常出现“`[heap]` 变化不大，但匿名映射和 RSS 一直涨”的现象。

### 8.4 现代 Linux 上地址随机化是现实基线，不是安全课件里的装饰

内核文档对 `randomize_va_space` 明确区分了 `0 / 1 / 2` 三种级别：`1` 会随机化 `mmap` 基址、栈、vDSO 和共享库，并让 PIE 二进制的代码起始地址随机；`2` 进一步开启 heap 随机化。  
这意味着当你在真实系统里看地址空间时，应该先假定“同一程序不同运行的地址不稳定”，除非你明确处在关闭随机化或特殊兼容环境中。

## 9. 当前推荐实践、过时路径与替代

以下“当前推荐实践”内容的核对日期为 `2026-03-19`。

### 9.1 当前更推荐的实践

- 先用三层视图同时建模：`readelf -l` 看静态装载意图，`/proc/<pid>/maps` 看实际映射结果，`/proc/<pid>/smaps` 看真实 RAM 占用和共享份额。
- 分析动态内存时，把“内存增长”拆成至少四类来源：`brk` heap、匿名 `mmap`、文件映射、线程栈 / TLS；不要只盯 `[heap]`。
- 一般用途 Linux 系统默认应把 ASLR / PIE 当现实基线；除非处在受控调试或遗留兼容场景，不要把关闭随机化当常态排障手段。这里的推荐，是基于内核文档对 full randomization 与 `CONFIG_COMPAT_BRK` 的兼容说明做出的工程推断。
- 应用代码不要直接依赖 `brk()` / `sbrk()` 做分配接口。`brk(2)` 已明确把它们标成应避免直接使用的老接口；现代应用应通过分配器或显式 `mmap()` 表达意图。
- 线程容量规划时，把线程栈、TLS 和 allocator arena 视为显式成本，而不是“系统自己会处理的背景噪声”。

### 9.2 过时路径与替代

- **过时路径：** 把“text / data / bss / heap / stack”当成完整现实。  
  **为什么旧：** 这只抓住了静态教学骨架，看不到共享库、匿名映射、线程栈、vDSO 和随机化。  
  **现在更推荐：** 用“映射区集合 + 装载链路 + 运行时演化”的模型。

- **过时路径：** 认为“heap 就是所有动态内存”。  
  **为什么旧：** glibc 等现实分配器会把较大分配和额外 arena 放进匿名 `mmap`。  
  **现在更推荐：** 把动态分配分成 `brk` heap 与 `mmap` 区两条路径，再结合线程和运行时判断。

- **过时路径：** 调试内存问题时只看 VSS 或单一“内存占用”数字。  
  **为什么旧：** 大地址空间不等于大驻留，文件映射与共享库还会扭曲直觉。  
  **现在更推荐：** 用 `smaps` 中的 `Rss`、`Pss`、`Anonymous`、`Swap` 等字段做 mapping 级归因。

- **过时路径：** 直接按旧博客去读 `/proc/<pid>/maps`，期待每个线程都显示成 `[stack:tid]`。  
  **为什么旧：** 该标签只存在于 Linux 3.4 到 4.4，4.5 起已移除，因为对大量线程进程代价过高。  
  **现在更推荐：** 结合 `/proc/<pid>/task/<tid>/...` 路径、调试器或 profiler 观测线程栈。

- **过时路径：** 为了“方便复现”长期关闭 ASLR，甚至让工具链和脚本依赖固定地址。  
  **为什么旧：** 这会让你的调试环境与真实生产基线脱节，也掩盖了现代系统默认存在的安全与布局约束。  
  **现在更推荐：** 默认保留随机化，只在受控实验环境中临时收紧变量。

## 10. 自测题 / 验证入口

1. 为什么 `readelf -S` 看到的 section 名字，不足以解释运行时真实布局？为什么还需要看 program header 和动态链接器链路？
2. 一个进程的 RSS 一直涨，但 `/proc/<pid>/maps` 中 `[heap]` 区间几乎没变。至少列出三种仍然可能导致 RSS 上升的布局机制。
3. 为什么同一个 PIE 二进制在两次运行中，主程序代码地址、共享库地址和 vDSO 地址都可能不同？heap 地址是否也一定不同？
4. 如果你在 `maps` 里看到一个 `rw-p`、无 pathname 的大区间，它更像来自什么来源？如果再结合 `smaps` 里的 `Anonymous` 和 `Swap` 字段，你能进一步验证什么？
5. 为什么“线程数翻倍但业务对象没变”仍可能显著改变地址空间形态和内存观测结果？

## 11. 迁移与关联模型

学会这个模型后，可以向外迁移到这些问题：

- **虚拟内存**：把“布局是什么样”继续下钻到“这些页如何被映射、驻留、回收和缺页处理”
- **分配器模型**：进一步分析 glibc / jemalloc / tcmalloc / JVM 为什么会在地址空间里留下不同足迹
- **动态装载与插件系统**：理解 `dlopen`、共享库版本、重定位和地址冲突
- **线程模型与 TLS**：从“线程有自己的栈”继续走向“线程局部数据如何实例化、为什么线程多时开销明显”
- **安全硬化**：把 ASLR / PIE / NX / RELRO 放回真实地址空间，而不是只把它们当编译器选项名
- **文件映射与 page cache**：理解为什么数据库、浏览器和分析引擎常把大对象留在 `mmap` 路径而不是传统 heap

最值得保留的迁移句式是：

**先问一个对象是静态装载来的、运行时映射来的，还是线程 / 运行时附加来的；再问它是文件后备、匿名私有，还是共享映射；最后再问它到底占了多少真实 RAM。**

## 12. 未解问题与继续深挖

- glibc 之外的 allocator 和运行时，究竟会把“同样的业务对象”投射成怎样不同的地址空间形态？
- 在开启 huge pages、JIT、`userfaultfd` 或语言运行时自管内存后，传统 `maps/smaps` 模型还要补哪些层？
- ASLR、容器检查点恢复、可重复调试和性能剖析之间，最稳的工程折中是什么？

## 13. 参考资料

以下外部资料中涉及“当前实践”的内容，均于 `2026-03-19` 核对。

- Linux `execve(2)`: https://man7.org/linux/man-pages/man2/execve.2.html
- Linux `proc_pid_maps(5)`: https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html
- Linux `proc_pid_smaps(5)`: https://man7.org/linux/man-pages/man5/proc_pid_smaps.5.html
- Linux `mmap(2)`: https://man7.org/linux/man-pages/man2/mmap.2.html
- Linux `brk(2)`: https://man7.org/linux/man-pages/man2/brk.2.html
- Linux `vdso(7)`: https://man7.org/linux/man-pages/man7/vdso.7.html
- Linux `ld.so(8)`: https://man7.org/linux/man-pages/man8/ld-linux.so.8.html
- Linux `malloc(3)`: https://man7.org/linux/man-pages/man3/malloc.3.html
- ELF gABI, Program Header: https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.pheader.html
- Linux kernel docs, `randomize_va_space`: https://docs.kernel.org/admin-guide/sysctl/kernel.html#randomize-va-space
