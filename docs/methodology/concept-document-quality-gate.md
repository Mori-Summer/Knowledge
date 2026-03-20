---
doc_id: methodology-concept-document-quality-gate
title: 统一概念文档质量门禁
concept: concept_document_quality_gate
topic: methodology
created_at: '2026-03-19T18:10:00+08:00'
updated_at: '2026-03-20T14:19:46+08:00'
source_basis:
  - concept_document_template
  - learning_new_things_playbook
  - cognitive_modeling_playbook
  - repository_concept_doc_review_2026_03_19
time_context: quality_gate_baseline_2026_03_19
applicability: concept_doc_acceptance_review_and_upgrade
prompt_version: concept_generation_prompt_v1
template_version: quality_gate_v1
quality_status: maintained_asset
related_docs:
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/fixed-concept-generation-prompt.md
open_questions:
  - 后续是否需要把这套门禁转成可自动执行的 lint 规则与评分脚本？
---

# 统一概念文档质量门禁

## 1. 目的

这份门禁的作用，不是把文档审得“更像标准件”，而是避免知识库里出现看起来完整、实际上不可调用的概念文档。

它主要解决三个问题：

- 文档章节齐了，但内部模型仍然是空的
- 文档章节齐了，但展开密度太稀，支撑不了判断和迁移
- 文档写了工业案例，但案例不真实或不可验证
- 文档写了“当前实践”，却没有时效性和来源纪律

## 2. 适用范围

这份门禁适用于：

- 新建正式概念文档
- 升级旧概念文档
- 审查是否配标 `upgraded_v1`

它不要求每篇文档一样长，但要求每篇文档都能稳定回答模板规定的核心问题。

## 3. Hard Fail 条件

只要命中任意一条，就不应标记为合格正式文档，更不能标 `upgraded_v1`。

### 3.1 仓库集成层 Hard Fail

- 没有写入合适的 `docs/{topic}/` 目录
- 文件名不是 `kebab-case`
- 缺失模板规定的必需 frontmatter 字段
- 新增正式文档后没有更新 `docs/index.md`

### 3.2 结构完整性 Hard Fail

- 缺少模板要求的任一核心章节
- 没有明确的一句话结论 / 问题定义
- 没有对象边界与相邻概念
- 没有核心机制 / 主链路 / 因果链
- 没有关键 tradeoff 与失败模式
- 没有自测题 / 验证入口
- 没有迁移与关联模型
- 没有未解问题与继续深挖
- 对明显复杂对象，各核心章节只用极短摘要草草带过，无法支撑判断、选型或迁移

### 3.3 模型质量 Hard Fail

- 文档主要是在复述定义，而不是建立可复用内部模型
- 文档没有说明“为什么这样”，只有“它是什么”
- 文档没有把机制压成可调用判断框架
- 文档没有指出边界、适用条件或失效条件
- 文档面对高复杂度主题时，没有给出与复杂度匹配的展开密度

### 3.4 证据与时效 Hard Fail

- 工业 / 现实世界锚点泛泛而谈，无法定位到真实组织、系统、产品、市场、标准或制度
- 提到“旧做法 / 旧技术 / 旧路径”却没有解释为什么旧、局限在哪、现在更推荐什么
- 涉及当前实践、现行制度、当前产品行为、最新状态，却没有显式核对日期
- 时间敏感结论没有相称的一手或官方来源支撑

### 3.5 一致性 Hard Fail

- frontmatter 的 `time_context`、`source_basis` 与正文内容明显不一致
- 参考资料存在，但无法支撑正文最关键的结论
- 正文把事实、推断和价值判断混写到无法区分

## 4. 评分规则

在没有 Hard Fail 的前提下，再做评分。  
每项按 `0-2` 分计：

- `0` 分：基本缺失或明显不合格
- `1` 分：有，但较弱，仍需补强
- `2` 分：清晰、扎实、可复用

### 4.1 问题定义与边界

检查：

- 问题是否被明确写清
- 边界是否清楚
- 是否区分了相邻概念和非对象范围

### 4.2 结构与因果模型

检查：

- 是否给出关键结构件
- 是否有主链路 / 因果链
- 结构是否能支持后续解释、预测和判断
- 展开密度是否与主题复杂度匹配；对明显复杂对象，是否至少达到 `deep` 模式应有的展开度

### 4.3 Tradeoff 与失败模式

检查：

- 是否指出核心取舍
- 是否说明失效条件
- 失败模式是否具体，而不是空泛提醒

### 4.4 工业锚点与当前实践

检查：

- 案例是否真实、具体、可定位
- 当前实践是否写明核对日期
- 旧路径与替代路径是否讲清原因

### 4.5 验证与迁移

检查：

- 是否存在有效自测题或验证入口
- 是否能把模型迁移到相邻问题
- 是否能帮助后续分析而不只是帮助记忆

### 4.6 元数据与仓库纪律

检查：

- frontmatter 是否完整且语义一致
- topic、命名、索引是否正确
- 参考资料与 `source_basis` 是否能反映证据基础

## 5. 评分结论解释

- `10-12` 分，且无 Hard Fail：可标 `upgraded_v1`
- `7-9` 分，且无 Hard Fail：可保留，但不建议标 `upgraded_v1`
- `0-6` 分，或出现任一 Hard Fail：应退回重做或补强

## 6. 审查流程

建议按下面顺序审查，而不是边看边凭直觉打分：

1. 先判断主题复杂度，以及它应按 `standard` 还是 `deep` 审视。
2. 再查仓库集成与 frontmatter。
3. 再查结构完整性。
4. 再看正文是否真的提供了内部模型。
5. 最后再看工业锚点、当前实践、来源和时间敏感性。

这样做的原因是：

- 前三步能快速排除明显不合格稿
- 后两步才真正区分“能看”和“能用”

## 7. 推荐审查输出格式

每次审查建议固定输出以下内容：

### 7.1 Hard Fail

- 是否命中
- 命中哪几条

### 7.2 复杂度判定

- 该主题更适合按 `standard` 还是 `deep` 审视
- 判定依据是什么

### 7.3 评分表

- 问题定义与边界：`0-2`
- 结构与因果模型：`0-2`
- Tradeoff 与失败模式：`0-2`
- 工业锚点与当前实践：`0-2`
- 验证与迁移：`0-2`
- 元数据与仓库纪律：`0-2`

### 7.4 必改项

- 不改就不能过门禁的项

### 7.5 可改进项

- 不影响过门禁，但能显著提高复用性的项

## 8. 对时间敏感文档的额外规则

凡是正文里出现下面这类表达，默认视为时间敏感内容：

- 当前推荐实践
- 最新 / 现行 / 目前
- 截至某日
- 现行制度 / 当前标准 / 当前产品行为

对此至少要求：

- 正文显式写核对日期
- 参考资料里能看见一手或官方来源
- 如果存在推断，应明确写成推断，而不是伪装成事实

## 9. 对 `deep` 模式和高复杂度主题的额外规则

凡是命中下面两条及以上的主题，默认按 `deep` 模式审视：

- 多层抽象并存
- 多后端、多 actor 或多执行面并存
- 多时间尺度或多阶段生命周期并存
- 当前推荐实践与历史路径都重要
- 主要用途是架构判断、排障、选型，而不是只做概念记忆

对此至少额外要求：

- `核心结构` 应给出可调用的层次表、分面框架或部件图式描述
- `核心机制 / 主链路 / 因果链` 至少有一条完整链路，必要时补充变体或分支
- `应用场景` 至少 `3` 类，且场景之间有真实差异
- `工业 / 现实世界锚点` 至少 `2` 个真实对象，并解释为何重要
- `当前推荐实践、过时路径与替代` 必须写清 `为什么旧 -> 局限 -> 替代 -> 何时选`
- `自测题 / 验证入口` 至少包含一种预测题、诊断题或选型题

## 10. 不能被门禁放过的伪优秀文档

下面这些文档最危险，因为它们看起来很像好文档：

- 结构齐全，但核心机制空心
- 结构齐全，但明显复杂的主题却每节只有一小段摘要
- 案例很多，但都是泛泛行业印象
- 语言流畅，但没有判断框架
- 有“当前实践”，但没有日期和来源纪律
- 有“迁移”小节，但只是列出相邻名词，没有说明可迁移的结构

这类文档不应因“读起来顺”而放行。

## 11. 未解问题

- 是否需要按概念类型进一步拆出“技术概念门禁”“制度概念门禁”“数学概念门禁”三个子版本？
- 是否需要给 `source_basis`、`quality_status`、`time_context` 建立受控词表？

## 12. 参考资料

- [方法论文档使用说明：怎么读、怎么用、怎么验收](/Users/maxwell/Knowledge/docs/methodology/methodology-operator-guide.md)
- [统一概念文档模板](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
- [学习新事物的方法手册：从陌生到可理解、可操作、可迁移](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)
- [认知规范与问题建模手册](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
- [固定概念文档生成 Prompt](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
