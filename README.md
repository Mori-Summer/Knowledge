# Knowledge

一个用于沉淀“可学习、可建模、可迁移”知识文档的 Markdown 仓库。

这个仓库的目标，不是收集术语解释，也不是写百科式摘要，而是把一个概念压缩成以后还能反复调用的内部模型。  
理想结果是：读完一篇文档后，你不只是“知道这个词”，而是能用它分析问题、判断边界、解释机制、迁移到相邻主题。

## 这个仓库现在长什么样

当前仓库已经形成了比较明确的两层结构：

- `docs/{topic}/`：正式概念文档，按主题分目录归档
- `docs/methodology/`：方法论栈，负责“怎么学、怎么建模、怎么输出、怎么验收、怎么触发”

也就是说，这里不只是放文档，还放了一套生成、升级、审查概念文档的工作规程。

## 仓库目标

这个仓库要避免两类常见失败：

- 只有定义，没有机制，读完还是不会分析
- 只有碎片笔记，没有统一结构，过段时间无法复用

因此，这里的概念文档默认都应该回答：

1. 它试图解决什么问题
2. 它的边界在哪里，不是什么
3. 它由哪些关键结构构成
4. 它如何运转，核心因果链是什么
5. 什么时候有效，什么时候失效
6. 现实里它怎样被使用，工业上为什么关心它
7. 旧做法为什么不够好，现在更推荐什么
8. 怎样验证自己是否真的理解了
9. 怎样把模型迁移到相邻问题
10. 还有哪些未解问题值得继续深挖

## 当前目录结构

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

- [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)：当前概念文档导航
- [CONTRIBUTING.md](/Users/maxwell/Knowledge/CONTRIBUTING.md)：贡献与维护规则

## 方法论栈

当前方法论已经分成 6 份角色明确的文档：

- [methodology-operator-guide.md](/Users/maxwell/Knowledge/docs/methodology/methodology-operator-guide.md)
  作用：说明整套方法论文档怎么读、怎么组合、怎么用于新建 / 升级 / 验收

- [learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)
  作用：定义“怎么把一个陌生对象学到可理解、可操作、可迁移”

- [cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
  作用：约束建模纪律，强制区分边界、机制、变量、约束、因果链、事实与推断

- [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
  作用：规定正式概念文档的输出合同、标准章节和 frontmatter 最小集合

- [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)
  作用：规定 Hard Fail、评分规则和 `upgraded_v1` 验收标准

- [fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
  作用：提供稳定的生成 / 升级触发指令，并明确 `standard` / `deep` 两档展开模式的触发方式

如果你第一次进入这个仓库，最稳的阅读顺序是：

1. [methodology-operator-guide.md](/Users/maxwell/Knowledge/docs/methodology/methodology-operator-guide.md)
2. [learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)
3. [cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
4. [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
5. [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)
6. [fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)

## 现在支持的 3 类主要工作流

这套方法论现在有两层判断：

- 第一层：你是在 `新建`、`升级` 还是 `审计 / 审查`
- 第二层：这篇文档应按 `standard` 还是 `deep` 模式展开

也就是说，`deep` 不是第 4 种工作流，而是新建 / 升级时都要先判断的一档展开密度。

## 写文档前先判断：`standard` 还是 `deep`

- `standard`：更适合单主机制、单时间尺度、边界比较清晰的概念
- `deep`：更适合多层系统、多后端、多 actor、多时间尺度、多约束同时作用的概念

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

### 1. 新建概念文档

适用场景：

- 仓库里还没有对应概念
- 想把一个新对象直接写成正式知识文档

最短入口参考：

```text
按 docs/methodology/methodology-operator-guide.md、docs/methodology/concept-document-template.md、docs/methodology/concept-document-quality-gate.md 和两份 playbook，为概念 {concept} 新建一篇知识库文档。直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，补齐统一 frontmatter，并在完成后更新 docs/index.md。不要写成泛泛解释，必须建立可复用内部模型；如果概念明显跨多层系统、多后端、多时间尺度或多类约束，默认按 deep 模式生成，不要只填满章节。
```

### 2. 升级已有文档

适用场景：

- 仓库里已有旧文档，但结构、证据或当前实践部分不够稳
- 需要补齐标准章节、frontmatter、工业锚点、参考资料或时间语境

最短入口参考：

```text
按 docs/methodology/methodology-operator-guide.md、docs/methodology/concept-document-template.md 和 docs/methodology/concept-document-quality-gate.md 升级现有文档 {path}；如果原主题明显复杂但当前文档过短或过稀，按 deep 模式补足展开密度，不要只修补章节名。
```

### 3. 审计 / 审查现有文档

适用场景：

- 想系统性检查现有概念库还有哪些 Hard Fail 或高价值改进点
- 想按统一标准做 findings-first review

推荐基线：

- 以 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md) 为审计标准
- 先查 frontmatter 和结构完整性，再看模型质量、工业锚点、时间敏感性和来源纪律

## 概念文档的最小合同

正式概念文档默认应包含以下信息位点：

1. 这份文档要帮你学会什么
2. 一句话结论 / 问题定义
3. 对象边界与相邻概念
4. 核心结构
5. 核心机制 / 主链路 / 因果链
6. 关键 tradeoff 与失败模式
7. 应用场景
8. 工业 / 现实世界锚点
9. 当前推荐实践、过时路径与替代
10. 自测题 / 验证入口
11. 迁移与关联模型
12. 未解问题与继续深挖
13. 参考资料

frontmatter 最少应包含：

- `doc_id`
- `title`
- `concept`
- `topic`
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

- `concept` 现在统一使用稳定的 `snake_case` 标识
- `topic` 对应 `docs/{topic}/` 目录
- `depth_mode` 建议在新建或实质升级时显式写成 `standard` 或 `deep`，未填写时默认按 `standard` 审视
- 时间敏感内容必须有日期语境与相称来源

## 当前质量线

一篇合格文档不应只是“写得顺”，还至少要满足：

- 问题定义和边界清楚
- 有结构、有机制、有失效条件
- 展开密度和主题复杂度匹配；明显复杂的主题不能只写成低密度标准档
- 有真实、可定位的工业 / 现实世界锚点
- 当前实践部分写清核对日期、旧路径局限和当前替代
- 有验证入口和迁移入口
- frontmatter 与正文语义一致

正式验收时，以 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md) 为准，不以主观阅读感受为准。

## 快速入口

- 看导航：读 [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)
- 看方法：读 [docs/methodology/methodology-operator-guide.md](/Users/maxwell/Knowledge/docs/methodology/methodology-operator-guide.md)
- 看模板：读 [docs/methodology/concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
- 看验收：读 [docs/methodology/concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)
- 看触发命令：读 [docs/methodology/fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
- 看现有例子：从 [docs/index.md](/Users/maxwell/Knowledge/docs/index.md) 中任选主题进入

## 贡献与维护

如果你要继续往仓库里增加、升级或审查文档，直接看 [CONTRIBUTING.md](/Users/maxwell/Knowledge/CONTRIBUTING.md)。

## 一句话总结

这是一个“把概念写成可调用模型”的知识库。  
现在它不只包含概念文档，也包含一整套生成、升级、审查这些文档的方法论与验收规则。
