---
doc_id: graphics-systems-opengl-context-resource-lifetime-order
title: OpenGL 上下文与 GL 资源释放顺序：大型项目里的生命周期治理模型
concept: opengl_context_resource_lifetime_order
topic: graphics-systems
depth_mode: deep
created_at: '2026-03-20T11:20:33+08:00'
updated_at: '2026-03-20T14:52:00+08:00'
source_basis:
  - khronos_opengl_context_wiki_checked_2026_03_20
  - khronos_opengl_object_wiki_checked_2026_03_20
  - khronos_glDeleteBuffers_wiki_checked_2026_03_20
  - khronos_glDeleteProgram_wiki_checked_2026_03_20
  - khronos_opengl_synchronization_wiki_checked_2026_03_20
  - qt_qopenglcontext_docs_checked_2026_03_20
  - qt_qopenglcontextgroup_docs_checked_2026_03_20
  - qt_qopenglwidget_docs_checked_2026_03_20
  - glfw_context_guide_checked_2026_03_20
  - glfw_quick_guide_checked_2026_03_20
time_context: foundations_plus_current_practice_checked_2026_03_20
applicability: large_scale_opengl_architecture_resource_lifecycle_shutdown_design_and_multithreaded_cleanup
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/computer-systems/process-memory-layout.md
  - docs/graphics-systems/angle.md
open_questions:
  - 在多 share-group、多插件、上下文可能被宿主提前销毁的场景里，怎样建立跨模块的统一销毁契约？
  - 在不引入全局 `glFinish()` 的前提下，怎样最低成本地证明“旧命令流已不再引用该资源”？
  - 对于历史代码里大量“析构即直删”的路径，怎样做低风险分阶段迁移？
---

# OpenGL 上下文与 GL 资源释放顺序：大型项目里的生命周期治理模型

## 1. 这份文档要帮你学会什么

这篇文档要帮你建立的，不是“`glDelete*` 应该什么时候调”这种 API 记忆，而是一套在大型 OpenGL 项目里可反复调用的生命周期治理模型。

读完后，你应该至少能做到：

- 把一个 GL 资源拆成 `CPU 包装对象生命周期`、`GL 对象名生命周期`、`驱动/GPU 实际使用生命周期`、`context/share-group 生命周期` 四层，而不再把它们混成一个“对象析构”
- 判断某个释放动作到底应该落在 `当前 context`、`某个 share group`，还是根本不应该在析构函数里直接做
- 设计一个不会因窗口关闭、线程切换、插件卸载、热重载而随机崩溃的 GL 关闭链路
- 识别常见伪修复，比如在析构里乱调 `glDelete*`、在 shutdown 时全局 `glFinish()`、或者先销毁 context 再寄希望于 OS 帮你兜底
- 把这套模型迁移到 Vulkan/Metal/D3D12 的资源回收、命令队列排空和延迟回收问题上

如果只记一句话：

**真正要治理的不是“对象什么时候析构”，而是“谁在什么合法执行域里，何时把删除意图兑现成真正的 GL API 调用”。**

## 2. 一句话结论 / 问题定义

**在大型 OpenGL 项目里，正确顺序通常不是“对象析构了就删”，而是“先停止新提交和新创建，再在仍然 current 的正确 context/share-group 上排空延迟销毁，最后才销毁 surface 和 context”；换句话说，GL 资源治理的核心不是析构顺序，而是删除意图、合法执行域、命令流完成和关停秩序之间的耦合。**

这篇文档处理的问题是：

- CPU 侧对象可能在任意线程、任意时间掉引用
- 但 OpenGL 调用只对当前 context 生效，而且一个 context 同时只能 current 在一个线程上
- 一些对象可在 share group 内共享，一些对象则根本不能共享
- `glDelete*` 往往只是发出删除请求或解除 API 可见名字，不等于“此刻 GPU 背后的真实占用已经安全消失”
- 当窗口关闭、`QOpenGLWidget` 重建、context 迁移、插件卸载、宿主异常销毁时，正常删除路径可能突然消失

它真正要回答的是：

- 哪些对象可以在哪个 context 上删
- 哪些删除必须延迟到合法执行域
- shutdown 到底该先停什么、再排什么、最后销毁什么
- 出现异常路径时，什么叫“正常清理失败但系统仍可控”

## 3. 对象边界与相邻概念

### 3.1 这篇文档直接处理什么

这篇文档直接处理的是：

- OpenGL 资源释放与 context 销毁之间的顺序治理
- 多线程、多窗口、多 context、share group 下的合法删除执行域
- shutdown、窗口重建、重父化、插件卸载、热重载时的资源收尾模型
- 资源删除意图、删除队列、同步手段与关停状态机之间的关系

它尤其适用于下面这些场景：

- 长生命周期桌面应用，而不是“跑完即退”的 demo
- Qt 多窗口、多 `QOpenGLWidget`、上下文可能重建的项目
- 渲染线程和资源上传线程分离的系统
- 插件式宿主或编辑器，插件不能直接拥有最终 GL 执行域
- 资源数量大、窗口多、热重载频繁的 CAD、可视化、DCC、HMI 工具

### 3.2 它不等于什么

它不等于：

- **OpenGL 教程。** 这篇不是讲 VAO/FBO/纹理 API 怎么用。
- **“RAII 万能论”。** RAII 只解决 CPU 侧对象所有权，不自动解决“哪个线程、哪个 context 才能调 GL”。
- **Vulkan/D3D12 的显式资源释放。** 它们的 API 形态不同，但可以迁移结构，不能直接照抄步骤。
- **单个 demo 程序的退出逻辑。** 单线程单窗口 demo 常把问题藏住，大型项目恰恰会把它放大。
- **单纯的显存回收问题。** 真正的难点不是“内存什么时候回”，而是“删除动作什么时候合法、什么时候可证明确已安全完成”。

### 3.3 最容易混淆的相邻概念

最相关的相邻概念有：

- current context / thread affinity
- share group / object sharing
- OpenGL 异步执行与 fence/sync
- render thread / command queue / actor 式执行域
- 资源包装器、插件生命周期、窗口/表面生命周期
- `makeCurrent()` / `doneCurrent()` / context detach
- `flush`、`finish`、`fence` 这些等待手段的语义差异

其中最常见的混淆有四个：

- **把 CPU 对象析构和 GL 删除混为一件事。**
- **把 `glDelete*` 返回和“GPU 已不再引用”混为一件事。**
- **把 share group 理解成“所有对象都能跨 context 删除”。**
- **把窗口销毁理解成“剩余 GL 删除自然无害”。**

### 3.4 一条必须先接受的边界

在大型项目里，下面这句必须先接受：

**CPU 逻辑所有权、GL 名字寿命、驱动/GPU 真实占用、context 执行域寿命，这四件事不是同一件事。**

如果你从一开始就不把这四件事拆开，后面所有设计几乎都会退化成“在析构里赌运气”。

## 4. 核心结构

### 4.1 四层生命周期

最稳的理解方式，不是记“先删纹理后删窗口”，而是先拆成四层生命周期。

| 层 | 它是什么 | 你真正该关心什么 | 常见误判 |
| --- | --- | --- | --- |
| CPU 包装对象生命周期 | `Texture`、`Buffer`、`Mesh` 这类 C++/Qt/引擎对象何时构造与析构 | 谁逻辑上拥有它，谁负责发出删除意图 | 误以为包装对象析构就等于底层 GL 资源已安全释放 |
| GL 对象名生命周期 | `GLuint` 名字何时 `glGen*` / `glCreate*`，何时 `glDelete*` 后失去 API 名字 | 删除请求何时发出，在哪个执行域发出 | 误以为 `glDelete*` 返回就等于 GPU 背后资源已立刻回收 |
| 驱动 / GPU 实际使用生命周期 | 命令流、attachment、绑定关系、in-flight work 何时真正不再引用该资源 | 什么时候可以证明“旧工作不再用它” | 误以为 CPU 线程“已经不再用”就等于 GPU 也不再用 |
| context / share-group 生命周期 | 哪个 context current，哪些 context 共享对象，整个 share group 活到何时 | 删除 API 是否还有合法执行域 | 误以为“程序里还有别的窗口”就一定能安全删除所有对象 |

这四层一旦混在一起，就会出现两类典型故障：

- **过早删除。** CPU 对象析构时立刻调 `glDelete*`，但此刻没有合法 current context，或其他 context / GPU 仍可能使用它。
- **过晚删除。** context 已销毁，删除队列还没排空，只能寄希望于进程结束时由驱动/OS 回收。

### 4.2 三个控制平面

除了四层生命周期，还要再看三个控制平面。

| 平面 | 它回答什么 | 典型设计对象 |
| --- | --- | --- |
| 所有权平面 | 谁逻辑上拥有资源包装器，谁负责发出“删除意图” | 资源管理器、场景节点、材质系统、插件模块 |
| 执行平面 | 哪个线程、哪个 current context 可以合法调用 GL | render thread、GL worker、context dispatcher |
| 关停平面 | 谁负责宣布 quiesce、停止创建、排空删除队列、最后销毁 context | engine shutdown coordinator、window/system lifecycle manager |

很多架构失败，不是因为没有资源类，而是因为这三个平面没有明确切开：

- 业务对象误以为自己拥有最终执行权
- GL 执行点散落在任意线程
- 关停没有集中协调者，只剩析构链碰碰运气

### 4.3 三种删除域

不是所有 GL 对象都能在 share group 里随便删。删除域至少要分三种。

| 删除域 | 对象类型 | 例子 | 调度 key |
| --- | --- | --- | --- |
| `share-group domain` | 可共享对象 | texture、buffer、renderbuffer、sampler、部分 GLSL 相关对象、sync objects | `share_group_id` |
| `context-local domain` | 不可共享对象 | framebuffer、vertex array、transform feedback、program pipeline、query | `context_id` |
| `abnormal/process domain` | 正常删除域已消失，只能异常降级 | context 已丢失、宿主已提前销毁窗口系统对象 | `process_exit_or_fault_path` |

这里第三类非常重要。它不是推荐路径，但它提醒你：

- 正常删除并不总能完成
- 你必须区分“正常清理成功”和“系统可接受地降级退出”

### 4.4 五个状态，而不是两个状态

最常见的坏模型只有两态：

- 活着
- 删了

大型项目里至少需要五态：

| 状态 | 含义 | 典型触发 |
| --- | --- | --- |
| `alive` | 资源正常可用 | 创建完成并已注册 |
| `retire_requested` | 业务侧不再需要，已提出删除意图 | 对象析构、插件卸载、场景切换 |
| `pending_delete` | 已投递到合法删除队列，等待执行域处理 | 投递到 render thread / context queue |
| `delete_issued` | `glDelete*` 已在合法 context 上发出 | 队列排空时执行删除 API |
| `physically_reclaimed_or_unknown` | GPU/驱动已回收，或在异常路径下无法再正常证明 | fence 确认、作用域结束、异常销毁 |

这一扩展很关键，因为它把“逻辑不再需要”和“物理上确已安全结束”分开了。

### 4.5 一个更实用的总句

可以把整个结构压成下面这句：

**资源包装器只拥有“提出删除需求”的权力；真正拥有“执行删除”的，是仍然 current 的合法 context；而拥有“宣布整个系统该开始关闭”的，是关停协调器。**

这句话一旦成立，很多设计选择会自动变清晰。

### 4.6 一页纸可调用模板

如果你要在代码架构里直接调用，可以压成下面这张表：

| 维度 | 你必须回答的问题 | 最小交付物 |
| --- | --- | --- |
| 生命周期分层 | 这个对象现在处于哪一层寿命问题 | 四层寿命标注 |
| 删除域 | 它属于 `share-group` 还是 `context-local` | 对象分类表 |
| 执行权 | 谁能在合法 context 上调用 `glDelete*` | 执行域收口点 |
| 完成性 | 是否需要证明旧命令流已不再引用 | fence/wait 策略 |
| 关停秩序 | 关闭系统时先停什么、再排什么、最后销毁什么 | shutdown 状态机 |
| 异常降级 | context 已失效时怎么办 | fault policy |

## 5. 核心机制 / 主链路 / 因果链

### 5.1 正常资源退休链

把大型项目里的正常释放路径压成一条主链，可以这样理解：

1. 某个业务对象不再需要某个 GL 资源，或系统进入 shutdown / window teardown / plugin unload。
2. CPU 包装对象析构时，不默认直接调 `glDelete*`；它首先只产生一个删除意图，里面至少包含：
   - 对象类型
   - 对象名
   - 删除域标识：`context_id` 或 `share_group_id`
   - 必要时的同步信息，例如“最后一次提交在哪个执行队列”
3. 删除意图被投递到合法的 GL 执行域，而不是在任意线程上“顺手删掉”。
4. GL 执行域在一个仍然 current 的、兼容的 context 上排空删除队列：
   - 共享对象在 share group 内的任一兼容 context 上删
   - 不共享对象必须在拥有它的那个 context 上删
5. `glDelete*` 解除 API 级名字与对象状态的关系，但不应被理解成“驱动/GPU 物理占用必然当场释放”。
6. 如果外部正确性要求证明旧命令流不再引用该资源，再使用局部 fence / wait 或阶段性同步，而不是把所有路径都升级成全局 `glFinish()`

这条链真正解决的是：

- 业务对象何时“退休”
- 删除动作谁有权执行
- 删除完成到什么程度才算足够

### 5.2 关停链：正确顺序为什么是反直觉的

进入系统关停时，正确顺序通常不是“先 destroy window/context，再清 wrapper”，而是：

1. 宣布不再接收新 GL 工作
2. 停止新资源创建
3. 让 CPU 侧最后一批包装对象发出删除意图
4. 在合法 context 上排空 `context_local_queue`
5. 在兼容 share group 上排空 `share_group_queue`
6. 仅在外部正确性确实要求时，再做 fence/wait 或局部同步
7. 最后销毁 surface / window / context

可以把它压成下面这个 shutdown 模板：

```text
enter_quiesce()
stop_new_submissions()
stop_new_resource_creation()
release_cpu_refs_and_publish_delete_intents()
make_legal_context_current()
drain_context_local_delete_queues()
drain_share_group_delete_queues()
optionally_wait_for_required_gpu_completion()
destroy_surfaces_and_contexts_last()
```

这个顺序反直觉，是因为很多人默认把 context 当作“普通成员对象”。实际上它更像执行域基础设施：

- 基础设施死了，清理动作就没有合法落点
- 所以 context 必须晚于待清理工作销毁

### 5.3 多线程 / 多 context 链：为什么删除不只是队列问题

在多线程和多 context 场景下，删除还要多过一层检查：

1. 先确定对象是否可共享
2. 再确定谁最近还可能提交引用它的命令
3. 再决定要不要做跨 context 的同步证明
4. 最后才是在哪个 current context 上发删除

这意味着：

- 删除意图需要携带域信息，而不只是 `GLuint`
- 执行点需要知道当前拿到的是哪类对象
- 共享对象即使“能删”，也不等于“现在删就一定不影响其他 context 中尚未完成的工作”

所以，多 context 系统里真正难的不是 `glDelete*` 那一行，而是：

- 拓扑识别
- 最后使用点识别
- 同步与完成性判断
- 统一调度

### 5.4 异常链：当正常删除路径已经消失

最容易被忽视的是异常链。

比如：

- context 已意外丢失
- 宿主先把窗口系统对象回收了
- 插件收到卸载通知时，真正的 GL 执行域已经不存在
- 驱动 reset 导致你不再能走标准清理动作

这时正确做法不是继续假装“还能正常 cleanup”，而是明确进入降级路径：

1. 标记该批资源进入 `abnormal/process domain`
2. 停止再承诺“会正常调用所有 `glDelete*`”
3. 记录异常和泄漏边界
4. 确保宿主或更大系统知道这是一条 fault path，而不是 normal path

这一步的意义在于：

- 正确性承诺要真实
- 不能把“异常退出时驱动/OS 会收尾”误包装成“我们的生命周期架构没问题”

### 5.5 逆向因果链：顺序一旦反过来会发生什么

如果你把顺序做反，一般会出现下面这些链式后果：

- **先销毁 context，再删资源**
  结果：删除 API 失去合法执行域，正常清理链直接断。

- **在析构线程直接 `glDelete*`**
  结果：线程与 current context 不匹配，轻则错误，重则未定义行为或随机崩溃。

- **为了“保险”全局 `glFinish()`**
  结果：把所有问题暂时压住，但代价高、语义过粗，也掩盖了设计上的执行域错位。

- **把 share group 当万能删除域**
  结果：VAO/FBO/query 这类不共享对象会在架构层被错误调度。

- **不区分正常路径与异常路径**
  结果：团队误以为系统“总能正确回收”，实际只是大量问题被拖到更大作用域才暴露。

### 5.6 真正的因果核心

把这一节压成一句最值得记住的话：

**CPU 对象消失，只代表“有人提出了删除需求”；真正的 GL 删除必须在一个仍然活着、且 current 的合法执行域里发生；而 context 的销毁必须晚于这一步。**

## 6. 关键 tradeoff 与失败模式

### 6.1 五组常见 tradeoff

- **RAII 简洁性 vs 执行域合法性**
  把删除写进析构最省心，但它默认析构线程、当前 context 和对象删除域都刚好正确。大型项目里这个假设经常同时失败。

- **确定性等待 vs 停顿成本**
  用 `glFinish()` 或强等待能快速获得“感觉上已经删干净”的确定性，但代价通常是粗暴的全局停顿。

- **中心化删除调度 vs 局部方便**
  把删除统一收口到少数执行点，架构更稳；但业务侧会觉得“不如直接调 API 方便”。

- **尽早回收 vs 正确同步**
  回收越激进，越容易撞上 in-flight work；同步越保守，越容易积压资源和等待成本。

- **正常路径完备性 vs 异常路径现实主义**
  你当然希望每个资源都走完整 cleanup；但现实里必须承认某些 fault path 根本不再具备正常清理条件。

### 6.2 五类高频失败模式

- **析构即直删。**
  典型表现：`~Texture()`、`~Buffer()`、`QObject` 析构里直接 `glDelete*`。

- **context 当普通成员对象。**
  典型表现：窗口析构一到，先把 context/surface 销毁，再希望剩余资源顺带被清掉。

- **共享拓扑误判。**
  典型表现：把 VAO/FBO/query 当成跟 texture 一样可在任意共享 context 上删除。

- **等待策略粗暴。**
  典型表现：shutdown、热重载、窗口关闭全部塞一个全局 `glFinish()`。

- **异常路径伪装成正常路径。**
  典型表现：context 已死还写日志“cleanup success”，让团队误判系统真实可靠性。

### 6.3 现场症状与回查方向

| 现场症状 | 最可能的问题层 | 应优先回查什么 |
| --- | --- | --- |
| 窗口关闭时随机崩溃 | 执行平面 | 析构线程是不是 GL 执行线程，context 是否 current |
| 热重载后显存持续上涨 | 关停平面 / 完成性 | 删除意图有没有真正排空，是否把资源拖到更大作用域 |
| 多窗口项目中只有某些对象释放失败 | 删除域分类 | 是否错误把 context-local 对象投到 share-group 队列 |
| shutdown 很慢 | 等待策略 | 是否把局部同步问题粗暴升级成全局 `glFinish()` |
| 插件卸载后宿主偶发报错 | 所有权平面 / 异常链 | 插件是否越权直删，宿主是否仍掌握合法执行域 |
| cleanup 代码看着很完整，但偶尔泄漏 | 正常/异常路径边界 | 是否把 context 已失效的 fault path 误当正常清理成功 |

### 6.4 为什么一些“修复”其实只是掩盖问题

下面这些做法经常能短期止血，但本质上是掩盖问题：

- **在所有析构前后各加一次 `makeCurrent()`**
  这会把执行域管理从架构问题降成局部手工问题，扩展到多线程、多窗口后很容易失控。

- **把所有删除都转移到主线程**
  如果主线程并不持有正确 context，或者对象属于其他 context-local 域，这只是换个地方犯错。

- **只在进程退出时观察有没有崩**
  这会掩盖长生命周期应用中的真实泄漏和悬挂资源问题。

- **把“驱动最终会回收”当成功标准**
  这对独立进程小程序勉强可接受，但对编辑器、宿主进程、插件系统、Qt 共享上下文根本不够。

### 6.5 一个很实用的判别句式

如果一个问题表现为：

- **对象析构即报错**，优先查执行域是否非法
- **context 已销毁后仍想 cleanup**，优先查关停顺序是否反了
- **共享对象删得掉、VAO/FBO 删不稳**，优先查删除域分类是否错了
- **加了 `glFinish()` 就“好了”**，优先查你是不是在用停顿掩盖同步与生命周期设计缺陷
- **只有异常退出才“成功回收”**，优先查正常清理链是否根本没建成

## 7. 应用场景

### 7.1 游戏引擎 / 编辑器窗口关闭与热重载

材质、纹理、FBO、VAO、shader program 会跨场景与窗口生命周期存在，必须把“编辑器对象释放”与“GL 删除实际执行”分开。

这里最容易踩坑的是：

- 场景对象释放早于渲染线程停止
- 编辑器窗口关了，但 share group 资源仍被别的视图引用
- 热重载触发大量退休请求，却没有统一 drain delete queue

更稳的做法是：

- 业务对象只发删除意图
- 渲染系统统一分类排队
- 窗口与 context 最后才销毁

### 7.2 Qt 多窗口 / 多 `QOpenGLWidget` 应用

一个部件被重父化、顶层窗口变化、上下文重建时，如果没有显式 cleanup 钩子，资源很容易拖到更大范围的共享上下文里继续存活。

这个场景里要特别注意：

- widget 析构不等于 share group 生命周期结束
- `aboutToBeDestroyed()` 不是可有可无的 signal，而是正常 cleanup 的关键插口
- 同一个应用里不同窗口可能共享，也可能不共享，不能拍脑袋决定删除域

### 7.3 离屏上传线程 + 主渲染线程

资源可能在 worker context 创建，在 render context 使用。销毁时如果不区分共享对象和非共享对象，就会在“哪里能删”上出现系统性错误。

这种架构中要额外建模：

- 最后一次使用在哪个队列
- 是否需要用 fence 证明旧命令流不再引用
- 删除时应归到 `share_group_queue` 还是 `context_local_queue`

### 7.4 插件式宿主

插件只知道自己不再需要纹理/程序对象，但宿主通常掌握真正的 GL 执行域和 context 生命周期。此时必须把删除设计成协议，而不是让插件对象自行直删。

推荐理解方式是：

- 插件拥有“退休声明权”
- 宿主拥有“执行删除权”
- 宿主或统一渲染层拥有“关闭系统权”

如果这三权不分，插件系统里的 GL 清理大概率会反复出事故。

### 7.5 大型 CAD / 可视化 / HMI 工具

多窗口、长生命周期、频繁重建设备或渲染表面的系统，最怕“局部窗口关闭”把共享资源拖挂到整应用退出才释放。

这里的典型风险是：

- 局部窗口销毁与全局资源池寿命错位
- 旧模块的资源回收依赖“整进程退出”
- 只在显存曲线严重上涨后才发现删除路径没有真正走通

### 7.6 嵌入式或受宿主控制的 OpenGL 模块

当你不是系统最终宿主，而只是某个大系统里的渲染模块时，问题更难：

- 窗口系统对象可能被宿主先销毁
- 你的 context 可能被迁移、暂停或提前释放
- 你甚至拿不到完整关停时机

这时必须更早承认异常链的存在，并把“正常 cleanup 成功率”与“异常降级策略”分开设计。

## 8. 工业 / 现实世界锚点

### 8.1 Qt 的 `QOpenGLWidget` 官方清理模式，本质上就是“context 先别死，先做显式 cleanup”

Qt 官方文档明确建议：当 `QOpenGLWidget` 对应的 context 即将被销毁时，应该连接 `QOpenGLContext::aboutToBeDestroyed()`，并在 cleanup 里先 `makeCurrent()`，释放 GL 资源后再 `doneCurrent()`；同时析构函数里也要再做一次 cleanup，以覆盖 signal 未触发的情况。

这不是框架小技巧，而是一个非常硬的现实锚点：

- cleanup 发生在 context 真正销毁之前
- cleanup 时要先确保 context current
- cleanup 不能只依赖 widget 析构，因为共享上下文和重父化会让资源作用域比 widget 更大

这几条几乎就是本文“先排删除，再销毁执行域”的直接工业化版本。

### 8.2 Qt 明确把 share group 暴露成一等对象

`QOpenGLContext::shareGroup()` 和 `QOpenGLContextGroup` 的存在，本质上是在告诉你：资源删除和 context 生命周期不能只按“单窗口对象树”来想，而要按共享拓扑来想。

对大型项目来说，这意味着：

- shared texture/buffer 的删除队列应按 share group 归并
- 非共享容器对象仍应按具体 context 管理
- “顶层窗口没了”不一定等于“资源作用域结束了”

很多 Qt 项目真正的 bug，不是 API 不会用，而是一直按 QWidget 对象树思维去想 GL 资源寿命。

### 8.3 GLFW 把线程约束说得非常直接

GLFW 的 context guide 和 quick guide 明确强调：

- context 同一时间只能 current 在一个线程
- 一个线程同一时间只能有一个 current context
- 把 context 移到别的线程前，必须先在旧线程上 detach
- 某些需要 current context 的函数，没有 current context 会直接报错

这给大型项目一个很实用的架构含义：

- GL 删除不应散落在任意业务线程
- 如果确实迁移 context，迁移本身必须被建模成显式状态变更，而不是“线程换了就继续调”
- “主线程统一删”只有在主线程真的是合法 current context 所在执行域时才成立

### 8.4 Khronos 的对象模型直接说明了“删除域”不是统一的

Khronos OpenGL wiki 明确区分：

- 多数对象可以共享
- container objects 与 query objects 不能共享
- 跨 context 的对象状态可见性并非天然立即成立，需要调用方自己同步

这三个事实合在一起，直接推导出本文里的三个工程结论：

- 资源删除必须先做对象分类，而不是只看 `GLuint`
- 共享拓扑本身是删除调度输入，不是附属信息
- “能共享”不等于“删的时候不需要额外同步判断”

### 8.5 `glDeleteProgram` 和 `glDeleteBuffers` 的语义，说明删除不是“立刻物理消失”

结合 Khronos 对 `glDeleteProgram` 与 `glDeleteBuffers` 的说明，可以抓住两个很关键的现实点：

- 某些对象删除可能只是被标记，直到不再属于任一 rendering context 当前状态
- buffer 删除语义也会受到当前状态、映射状态和上下文关系的影响

这提醒你：

- `glDelete*` 不是“对象析构函数的显卡版”
- 删除 API 发出后，后端真实回收和引用解除仍然有延迟与条件

这正是为什么大型项目不能把“调用了删除 API”当成一切问题的终点。

### 8.6 Khronos 的同步模型说明：等待是工具，不是生命周期架构

OpenGL Synchronization 相关材料的最大启发是：

- GPU/驱动执行天然异步
- 如果你确实需要证明某批工作已完成，可以用 fence / sync 等更细粒度机制
- 但等待机制解决的是“完成性证明”，不是“删除域选择”和“关停顺序”问题

这意味着：

- `glFinish()` 不是生命周期模型
- fence 也不是生命周期模型
- 生命周期模型先决定谁能删、何时删、在哪删；等待工具只是其中一段局部证明手段

## 9. 当前推荐实践、过时路径与替代

以下“当前推荐实践”内容核对日期为 `2026-03-20`。其中 API 事实来自 Khronos / Qt / GLFW 文档；“推荐实践”部分是基于这些事实对大型项目做出的工程归纳。

### 9.1 当前更推荐的实践

#### 9.1.1 把 GL 删除设计成两阶段

第一阶段是 CPU 侧“发布删除意图”，第二阶段是 GL 执行域“在合法 current context 上兑现删除”。

这一步几乎是所有后续稳定性的起点。

#### 9.1.2 把删除队列至少分成两类

- `share_group_queue`：可共享对象
- `context_local_queue`：VAO/FBO/query 等非共享对象

如果项目复杂度继续上升，还可以再细分：

- 按模块或插件归属分桶
- 按资源类型分批排空
- 按是否需要完成性证明区分同步策略

#### 9.1.3 让 context/share-group 的生命周期晚于所有待删除工作

不是“资源对象尽量早析构”，而是“真正的 GL 删除必须发生在 context 还活着时”。

这句话听起来简单，但它会直接改变：

- 窗口销毁顺序
- 插件卸载顺序
- 编辑器热重载顺序
- Qt cleanup 钩子的设计方式

#### 9.1.4 把 shutdown 做成显式状态机

至少有下面这几个阶段：

- `running`
- `quiescing`
- `draining deletes`
- `destroying contexts`
- `done`

不要把 shutdown 交给零散析构链去隐式拼出来。

#### 9.1.5 把 GPU 完成等待做成按需策略

只有在确实需要对外证明“该资源不再被 in-flight work 使用”时，才引入 fence / wait；默认不要用全局 `glFinish()` 代替生命周期设计。

一个更稳的选择框架是：

| 需求 | 更推荐 |
| --- | --- |
| 只需要合法发删除，不需要立刻证明 GPU 完成 | 只排删除队列 |
| 需要证明某批旧工作已经完成 | 局部 fence / wait |
| 需要整个系统停机前全局一致 | 最后阶段的受控等待，而不是平时乱用 `glFinish()` |

#### 9.1.6 把“删除能力”收口到渲染系统

业务模块、资源包装器、脚本层、插件层负责发出删除意图；真正 `glDelete*` 的权限收口到少数 GL 执行点。

这会带来两个好处：

- 执行域一致
- 关停秩序可控

### 9.2 过时路径、为什么旧、局限在哪、现在更推荐什么

| 过时路径 | 为什么旧 | 主要局限 | 现在更推荐什么 |
| --- | --- | --- | --- |
| `~Texture()` / `~Buffer()` 里直接 `glDelete*` | 它建立在“析构线程就是 GL 线程、对应 context 仍 current、对象共享边界简单”的早期单窗口假设上 | 大型项目里这些前提经常同时失效 | 析构只发布删除意图，由 GL 执行域统一删除 |
| 先销毁 window/context，再清资源包装器 | 一旦 context 已死，后续很多 GL 删除根本失去合法执行域 | 正常 cleanup 链直接断 | 先 quiesce 和 drain delete queue，context 最后销毁 |
| 所有问题都靠 shutdown 时 `glFinish()` | 它代价高、范围粗，而且无法替代“你必须仍然拥有一个合法 current context”这个前提 | 性能差、语义粗、容易掩盖设计缺陷 | 用显式队列排空 + 局部 fence / sync 证明必要的完成关系 |
| 把共享组理解成“所有 GL 对象都能在任意共享 context 上删” | Khronos 明确区分了可共享对象和 container/query 等不共享对象 | 错误删除域会直接造成错误、未定义行为或泄漏 | 删除时先按对象类别决定 `share_group_id` 还是 `context_id` |
| 依赖进程退出做最终回收 | 它只对“整个进程马上结束”的粗粒度场景勉强成立 | 对宿主进程、长生命周期编辑器、Qt 共享上下文作用域，这个策略会把问题拖大 | 显式 cleanup，把资源寿命绑到真实模块/窗口/系统生命周期上 |
| 把异常路径当正常路径 | 现实里 context 丢失、宿主提前销毁并不少见 | 团队会高估系统可靠性，隐藏真正 fault boundary | 显式区分 normal path 与 fault path，并记录降级承诺 |

### 9.3 当前更稳的落地清单

如果你要开始治理一个现有 OpenGL 项目，当前更推荐至少做到下面这些：

- 建一个对象分类表，明确哪些是 `share-group`，哪些是 `context-local`
- 建一个删除意图结构体，而不是只在各处传 `GLuint`
- 建集中删除执行点，不让任意模块随手 `glDelete*`
- 建 shutdown 状态机，而不是把关闭顺序散在析构函数里
- 为 `QOpenGLWidget`、窗口系统封装、插件卸载建立显式 cleanup 钩子
- 只在必要处引入 fence/sync，避免把等待手段变成默认生命周期策略
- 建 fault policy，明确 context 已失效时什么叫“可接受降级”

## 10. 自测题 / 验证入口

下面这些题，不是为了背 API，而是为了检验你是否真正掌握了这个生命周期模型。

1. 为什么“CPU 对象析构了”不足以推出“现在就可以安全 `glDelete*`”？
2. 如果一个纹理在 share group 内共享，而一个 VAO 只存在于某个具体 context，你的删除队列应该怎样分层？
3. 为什么“我在 shutdown 前调一次 `glFinish()`”不能替代“先排空删除队列，再销毁 context”的设计？
4. `glDeleteProgram` 的“flagged for deletion”语义，说明了哪一层生命周期没有和 CPU 对象析构对齐？
5. 在 Qt `QOpenGLWidget` 被重父化导致 context 重建的场景里，为什么必须在 `aboutToBeDestroyed()` 或显式 cleanup 钩子里处理资源，而不是只等析构函数？
6. 如果 context 已被宿主意外销毁，你此时还能不能把这条路径视为“正常清理成功”？为什么？
7. 一个插件提交了删除意图，但宿主尚未进入 quiesce 状态。你应该允许插件直接 `glDelete*` 吗？为什么？
8. 什么时候你真的需要 fence / wait，什么时候只要把删除请求排进合法队列就够了？
9. 为什么“主线程统一删”不是天然正确答案？它还缺哪个前提？
10. 请用一句话解释：`删除 API 已调用` 和 `资源已经可证明彻底安全结束` 之间差了什么。

## 11. 迁移与关联模型

### 11.1 这篇文档最值得迁移的总结构

这篇文档最值得迁移的，不是 OpenGL 细节，而是下面这个通用结构：

**资源逻辑所有权 != API 名字寿命 != 后端真实使用寿命 != 执行域寿命。**

这个总结构一旦掌握，你会发现它能迁移到很多系统。

### 11.2 可以直接迁移到哪些领域

- **Vulkan / D3D12 / Metal**
  它们更显式，但同样存在“CPU 句柄释放”和“GPU 不再引用”之间的延迟，只是同步与回收责任更直接落到你头上。

- **Actor / command queue 架构**
  “业务线程只发 intent，执行线程统一兑现”本质上和这里的 delete queue 是同一套思想。

- **Hazard pointer / epoch reclamation**
  CPU 并发内存回收里也有“对象逻辑上删了，但还不能立刻物理回收”的两阶段结构，这和 GPU 资源延迟回收高度同构。

- **插件系统 / 脚本运行时**
  插件只知道自己不再需要资源，不应直接拥有底层执行域的删除权限；这和宿主掌握真正销毁序列是同一种治理关系。

- **窗口系统与交换链管理**
  “surface/swapchain/context 是执行域基础设施，应晚于资源删除销毁”这个判断，也能迁移到其他图形 API。

### 11.3 与相邻模型的关系

| 相邻模型 | 它擅长什么 | 这篇文档补了什么 |
| --- | --- | --- |
| RAII / 智能指针 | 管 CPU 侧所有权 | 补执行域合法性与后端完成性 |
| 命令队列 / Actor | 管执行串行化 | 补资源生命周期与关停秩序 |
| 引用计数 | 管逻辑上是否还有人用 | 补 GPU/驱动后端延迟使用 |
| Epoch 回收 | 管延迟回收安全性 | 补图形执行域与上下文拓扑 |
| 显式图形 API 生命周期管理 | 管 fence 和回收时机 | 补 OpenGL 这种隐式执行域下的建模方式 |

### 11.4 最值得保留的迁移句式

以后遇到类似问题，可以先问自己：

**谁在逻辑上不要它了？谁有权执行真正删除？后端是否还可能在用？执行域会不会先于资源死掉？**

只要这四问答不清，删除设计大概率还停留在“碰运气能跑”的阶段。

## 12. 未解问题与继续深挖

- 当一个大型项目同时存在多个 share group、多个插件和多个窗口系统抽象层时，怎样建立不会互相越权的统一销毁协议？
- 哪些对象类型最值得单独建“引用结束证明”机制，而哪些对象只需要删除队列即可？
- 在 context 丢失、驱动 reset、宿主异常销毁窗口句柄的场景下，最现实的降级策略是什么？
- 如果一个项目已经大量依赖“对象析构即直删”的旧模式，怎样渐进迁移到 `delete-intent + drain-queue` 模型，而不一次性重写整个渲染系统？
- 对 Qt、GLFW、自研窗口系统混用的项目，能不能抽象出统一的 cleanup protocol？

## 13. 参考资料

以下外部资料中涉及“当前实践”的内容，均于 `2026-03-20` 核对。

- Khronos OpenGL Wiki, OpenGL Context: https://wikis.khronos.org/opengl/OpenGL_Context
- Khronos OpenGL Wiki, OpenGL Object: https://wikis.khronos.org/opengl/OpenGL_Object
- Khronos OpenGL Wiki, Synchronization: https://wikis.khronos.org/opengl/Synchronization
- Khronos OpenGL Wiki, `glDeleteBuffers`: https://wikis.khronos.org/opengl/GLAPI/glDeleteBuffers
- Khronos OpenGL Wiki, `glDeleteProgram`: https://wikis.khronos.org/opengl/GLAPI/glDeleteProgram
- Qt 6, `QOpenGLContext`: https://doc.qt.io/qt-6/qopenglcontext.html
- Qt 6, `QOpenGLContextGroup`: https://doc.qt.io/qt-6/qopenglcontextgroup.html
- Qt 6, `QOpenGLWidget`: https://doc.qt.io/qt-6/qopenglwidget.html
- GLFW latest context guide: https://www.glfw.org/docs/latest/context_guide.html
- GLFW 3.3 quick guide, current context section: https://www.glfw.org/docs/3.3/quick_guide.html
