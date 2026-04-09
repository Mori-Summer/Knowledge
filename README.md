# Knowledge

一个用于沉淀“可学习、可区分、可建模、可迁移”知识文档的 Markdown 仓库。

这个仓库的目标，不是收集术语解释，也不是写百科式摘要，而是把一个概念沉淀成以后还能反复调用的理解资产。  
多数文档会走模型型路径，帮助你分析问题、判断边界、解释机制、迁移到相邻主题；少数基础概念会走纯概念路径，重点解决命名、分类、边界和相邻概念区分问题，而不是为了建模而建模。

## 这个项目怎么用

这个仓库主要有四种用法：

1. 直接阅读现有知识文档。
2. 按方法论新建、升级、审查知识文档。
3. 通过 BMAD 技能和代理命令，让 AI 按仓库规则协助你工作。
4. 单独维护 `docs/methodology/` 下的方法论文档。

如果你只是想开始读：

- 先看 [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)
- 再看 [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)

如果你是想继续生产或维护文档：

- 先判断任务属于 `新建`、`升级`、`审查 / 验收`，还是 `仅更新索引`
- 再判断目标文档更适合走“模型型概念文档”还是“纯概念文档”路径
- 默认先看 [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)

## 快速开始

### 用法 1：只读仓库

最稳的阅读顺序：

1. [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)
2. [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
3. 任选一个你关心的主题目录进入，例如 `docs/computer-systems/`

### 用法 2：直接用固定 Prompt 处理文档

当前已经固化成四段入口：

1. 新建文档
2. 升级旧文档
3. 审查 / 验收
4. 仅更新索引

统一规范见 [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)。  
固定触发词版本见 [docs/methodology/fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)。  
如果只想先记住最短触发方式，可以直接这样用：

```text
为概念 {concept} 新建一篇知识库文档。先判断它应走模型型概念文档还是纯概念文档路径。
```

```text
按统一规范升级现有文档 {path}。
```

```text
按统一规范审查文档 {path}。
```

```text
检查 docs/index.md 是否正确收录 {path}。
```

### 用法 3：通过 BMAD 技能 / 代理命令使用仓库

如果你在 Codex / BMAD 环境里工作，推荐这样用。

第一步：先问路。

```text
$bmad-help 我应该从哪里开始处理一个新概念文档？
```

第二步：加载技术写作代理。

```text
$bmad-tech-writer
```

进入 `tech-writer` 之后，最常用的是这些命令：

- `WD`：写文档或升级文档正文
- `VD`：按标准审查文档
- `US`：把新确认的稳定规则写入写作标准
- `EC`：解释复杂概念
- `MG`：生成 Mermaid 图
- `MH`：重新显示菜单
- `DA`：退出代理

常见用法示例：

```text
WD：为概念 memory order 新建一篇知识库文档，并按 deep 模式生成。
```

```text
WD：按统一规范升级现有文档 docs/networking/dns.md。
```

```text
VD：按统一规范审查 docs/computer-systems/memory-order.md。
```

```text
US：把这次确认下来的规则写进 _bmad/_memory/tech-writer-sidecar/documentation-standards.md。
```

### 用法 4：只维护方法论文档

如果你改的是 `docs/methodology/` 下的内容，不要把它当普通概念文档处理。  
最稳的路径是：

1. 先看 [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
2. 先判断你要改的是“主规范”还是“参考件”
3. 只有在确实需要更深背景、模板细节或评分细则时，再进入对应参考文件

## 仓库现在长什么样

当前仓库已经形成了比较明确的两层结构：

- `docs/{topic}/`：正式知识文档，按主题分目录归档
- `docs/methodology/`：文档规范层，负责“怎么新建、怎么升级、怎么审查、怎么集成”

也就是说，这里不只是放文档，还放了一套生成、升级、审查和集成文档的工作规程。

当前目录结构：

```text
docs/
  methodology/                方法论文档与执行入口
  ai-systems/                 AI 系统相关概念
  computer-systems/           计算机系统相关概念
  economics/                  经济学与制度类概念
  image-processing/           图像处理相关概念
  programming-languages/      编程语言相关概念
  systems/                    系统设计与一般系统概念
  _reports/                   规范化/升级报告
  index.md                    文档导航
```

入口文档：

- [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)：当前知识文档导航
- [CONTRIBUTING.md](/Users/maxwell/Knowledge/CONTRIBUTING.md)：贡献与维护规则

## 方法论规范

`docs/methodology/` 现在按“一份主规范 + 多份参考件”组织。

- [document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
  作用：主规范，统一定义新建、升级、审查与仓库集成的执行合同

- [fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
  作用：快捷入口，适合直接复制固定触发词

- [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
  作用：参考模板，补充完整章节骨架和分型细节

- [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)
  作用：参考门禁，补充完整 Hard Fail 与评分细则

- [learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)
  作用：学习参考，补充学习目标与理解路径

- [cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
  作用：建模参考，补充边界、机制、约束与因果纪律

- [methodology-operator-guide.md](/Users/maxwell/Knowledge/docs/methodology/methodology-operator-guide.md)
  作用：旧版编排说明，保留作参考

如果你第一次进入这个仓库，默认顺序改成：

1. [document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
2. [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)
3. 只有在主规范不足以支撑任务时，再进入对应参考件

## 现在支持的 4 类主要工作流

当前默认工作流不是三类，而是四类：

1. 新建文档
2. 升级旧文档
3. 审查 / 验收
4. 仅更新索引

此外还有一个长期存在的旁路任务：`方法论文档维护`。

也就是说，`deep` 不是第 5 种工作流。  
真正的判断顺序是：

1. 先判断你在做哪类工作流
2. 再判断文档走“模型型概念文档”还是“纯概念文档”路径
3. 最后才判断 `standard` 还是 `deep`

## 写文档前先判断：先分路径，再定展开密度

### 模型型概念文档

更适合下面这类主题：

- 本体就是机制、协议、结构、系统对象或方法
- 读者后续主要拿它做解释、预测、调试、选型或排障
- 如果不写主链路，就无法真正理解它

### 纯概念文档

更适合下面这类主题：

- 本体更像基础概念、分类边界、命名区分或对比概念
- 主要困难不在“它如何运转”，而在“它到底在说什么、和什么不同”
- 强行补机制链、tradeoff 和工业锚点，只会把文档拆得四分五裂

### 再判断：`standard` 还是 `deep`

- `standard`：更适合单主机制、单时间尺度、边界比较清晰的概念，或相邻概念集合较小、判别负担较低的纯概念主题
- `deep`：更适合多层系统、多后端、多 actor、多时间尺度、多约束同时作用的概念，或者相邻概念网络本身就很复杂的纯概念主题

如果一个主题命中下面两条及以上，默认按 `deep` 模式处理：

- 需要同时解释多层抽象，例如接口层、实现层、运行时层、治理层
- 存在多个后端、多个执行面、多个参与方或多个时间尺度
- 当前推荐实践与历史路径都重要，不能只写静态原理
- 读者后续主要拿它做架构判断、选型、排障，而不是只记定义
- 失败模式和 tradeoff 本身就比“定义”更重要

`deep` 模式不是单纯把篇幅拉长，而是要求文档具备更强判断支撑力。至少要补强：

- 层次化的核心结构，而不是只列名词
- 至少一条完整主链路；必要时补一条变体链路
- 多个有差异的应用场景
- 多个真实工业 / 现实世界锚点
- 明确的旧路径、局限和替代路径判断

## 概念文档的最小合同

所有正式概念文档默认都应包含：

1. 这份文档要帮你学会什么
2. 一句话结论 / 问题定义
3. 边界与相邻概念
4. 自测题 / 验证入口
5. 迁移与关联模型
6. 未解问题与继续深挖
7. 参考资料

如果是模型型概念文档，默认再补：

1. 核心结构
2. 核心机制 / 主链路 / 因果链
3. 关键 tradeoff 与失败模式
4. 应用场景
5. 工业 / 现实世界锚点
6. 当前推荐实践、过时路径与替代

如果是纯概念文档，默认再补：

1. 这个概念试图解决的命名、分类或区分问题
2. 什么算它，什么不算它
3. 代表性例子、反例与常见误用
4. 它最容易和哪些相邻概念混淆
5. 它会进入哪些更大的模型、解释或判断框架

frontmatter 最少应包含：

- `doc_id`
- `title`
- `concept`
- `topic`
- `depth_mode`
- `created_at`
- `updated_at`
- `source_basis`
- `time_context`
- `applicability`
- `prompt_version`
- `template_version`
- `quality_status`
- `related_docs`
- `open_questions`

其中：

- `concept` 统一使用稳定的 `snake_case` 标识
- `topic` 对应 `docs/{topic}/` 目录
- `depth_mode` 建议在新建或实质升级时显式写成 `standard` 或 `deep`
- 时间敏感内容必须有日期语境与相称来源

## 当前质量线

一篇合格文档不应只是“写得顺”，还至少要满足：

- 问题定义和边界清楚
- 模型型主题有结构、有机制、有失效条件；纯概念主题有边界、有例子 / 反例、有误读纠偏
- 展开密度和主题复杂度匹配；明显复杂的主题不能只写成低密度标准档
- 如果正文需要工业 / 现实世界锚点，这些锚点必须真实、可定位
- 如果正文包含当前实践部分，要写清核对日期、旧路径局限和当前替代
- 有验证入口和迁移入口
- frontmatter 与正文语义一致

正式验收时，以 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md) 为准，不以主观阅读感受为准。

## 常用入口回顾

- 看导航：读 [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)
- 看方法：读 [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
- 看固定入口：读 [docs/methodology/fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
- 看统一执行流程：读 [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
- 用 BMAD 帮你判断下一步：输入 `$bmad-help`
- 用技术写作代理执行任务：输入 `$bmad-tech-writer`

## 贡献与维护

如果你要继续往仓库里增加、升级或审查文档，直接看 [CONTRIBUTING.md](/Users/maxwell/Knowledge/CONTRIBUTING.md)。

## 一句话总结

这是一个把概念沉淀成“可调用理解资产”的知识库。  
其中大多数文档是模型型概念文档，少数是纯概念文档；仓库同时提供生成、升级、审查、索引维护和命令化使用这些文档的方法论。
