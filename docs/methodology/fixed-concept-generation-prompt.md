---
doc_id: methodology-fixed-concept-generation-prompt
title: 固定概念文档生成 Prompt
concept: fixed_concept_doc_generation_prompt
topic: methodology
created_at: '2026-03-16T10:12:00+08:00'
updated_at: '2026-04-08T16:09:09+08:00'
source_basis:
  - architecture_rules
  - concept_template
  - methodology_operator_guide
  - concept_document_quality_gate
  - document_generation_methodology
  - pure_concept_doc_split_review_2026_04_08
time_context: prompt_updated_2026_04_08
applicability: concept_doc_generation_upgrade_review_and_index_maintenance
prompt_version: concept_generation_prompt_v3
template_version: concept_doc_v1
quality_status: maintained_asset
related_docs:
  - docs/methodology/concept-document-template.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/document-generation-methodology.md
open_questions:
  - 未来是否需要把这四段固定入口进一步拆成四个独立文件，并补上 machine-readable 版本？
---

# 固定概念文档生成 Prompt

这份文件把概念文档的高频动作固定成四段入口：

- 新建文档
- 升级旧文档
- 审查 / 验收
- 仅更新索引

目标不是“让 AI 解释一个词”，而是让它按你已经定下的知识库规则，产出、修订、审查和集成正式概念文档。

从这次版本开始，固定入口默认要求先判断：

- 这篇文档是模型型概念文档，还是纯概念文档
- 在对应路径下，再判断 `standard` 还是 `deep`

## 1. 使用前先判断你要做什么

先把需求归到下面四类，再选对应入口：

- 仓库里还没有对应正式文档：走“新建文档”
- 仓库里已有文档，但质量、结构或时效不足：走“升级旧文档”
- 你要判断某篇文档能不能算正式达标：走“审查 / 验收”
- 文档内容不一定要改，只需要补仓库导航：走“仅更新索引”

建议配套使用顺序仍然是：

1. 先看 [document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
2. 新建或升级时，先判断走模型型概念文档还是纯概念文档路径
3. 需要更深背景时，再用两份 playbook 补建模或补边界纪律
4. 再按 [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md) 组织输出
5. 最后用 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md) 做验收

## 2. 固定入口一：新建文档

```text
为概念 {concept} 新建一篇知识库文档。

要求：
1. 严格遵循：
   - docs/methodology/document-generation-methodology.md
   - docs/methodology/learning-new-things-playbook.md
   - docs/methodology/cognitive-modeling-playbook.md
   - docs/methodology/concept-document-template.md
   - docs/methodology/concept-document-quality-gate.md
2. 先判断这篇文档应走模型型概念文档还是纯概念文档路径，再判断采用 `standard` 还是 `deep` 模式，并在 frontmatter 中显式写 `depth_mode`。
3. 如果是模型型概念文档，输出目标不是术语解释，而是建立可复用的内部模型。
4. 如果是纯概念文档，输出目标是建立稳定的概念辨识资产：写清命名问题、边界、相邻概念、例子、反例和常见误读；不要为了套模板而硬造机制链。
5. 文档直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，补齐统一 frontmatter。
6. 如果是模型型概念文档，至少覆盖：问题定义、对象边界、核心结构、核心机制 / 主链路 / 因果链、关键 tradeoff 与失败模式、应用场景、工业 / 现实世界锚点、当前推荐实践与过时路径、自测题 / 验证入口、迁移与关联模型、未解问题、参考资料。
7. 如果是纯概念文档，至少覆盖：命名 / 分类问题、概念边界与相邻概念、什么算它与什么不算它、代表性例子与反例、常见误读、下游进入哪些更大模型、自测题 / 验证入口、迁移与关联模型、未解问题、参考资料。
8. 如果正文涉及当前实践、现行制度、当前产品行为或其他时间敏感内容，必须写明核对日期，优先使用一手或官方来源，并区分事实、推断与结论。
9. 新增正式文档后，更新 docs/index.md。
10. 交付前按 docs/methodology/concept-document-quality-gate.md 自查；如果不满足 `upgraded_v1` 条件，不要伪装成正式达标稿。

补充上下文：
- 所属主题：{topic}
- 我是在什么场景下遇到它的：{context_where_i_met_it}
- 我现在最不理解的点：{what_confuses_me}
- 我希望后续能拿它分析什么问题：{future_use_case}
- 这个概念是否时间敏感：{is_time_sensitive}
- 我希望看到哪些真实工业锚点：{preferred_real_world_anchors}
```

## 3. 固定入口二：升级旧文档

```text
按 docs/methodology/document-generation-methodology.md、docs/methodology/concept-document-template.md、docs/methodology/concept-document-quality-gate.md 升级现有文档 {path}。

要求：
- 保留原有高价值内容，不要整篇重写
- 先按质量门禁找出 Hard Fail 和低分项，再判断这篇文档应走模型型概念文档还是纯概念文档路径
- 补齐缺失的标准章节、frontmatter 字段和 `depth_mode`
- 重点补：对应路径下真正需要的结构、自测题、迁移入口、未解问题
- 如果主题明显复杂但当前展开过短或过稀，按 `deep` 模式补足展开密度，不要只修补章节名
- 如果现有文档只是为了套模板而硬写了伪机制、伪 tradeoff 或伪工业锚点，删掉这些假结构，改成更贴题的纯概念写法
- 如果正文包含时间敏感内容，补上核对日期、`time_context`、`source_basis` 和更强来源
- 实质升级后更新 `updated_at`；若路径、标题或 topic 变化，顺手更新 docs/index.md
```

## 4. 固定入口三：审查 / 验收

```text
按 docs/methodology/concept-document-quality-gate.md 审查文档 {path}。

要求：
- 先判断该文档属于模型型概念文档还是纯概念文档，并说明依据
- 再判断该主题应按 `standard` 还是 `deep` 审视，并说明依据
- 输出 Hard Fail：是否命中、命中哪几条
- 输出六项评分：问题定义与边界、结构与因果 / 判别框架、tradeoff / 失败模式 / 误读纠偏、工业锚点与当前实践、验证与迁移、元数据与仓库纪律
- 输出必改项：不改就不能过门禁的项
- 输出可改进项：不影响过门禁但能显著提高复用性的项
- 最后给出结论：是否可标 `upgraded_v1`
```

## 5. 固定入口四：仅更新索引

```text
检查 docs/index.md 是否已经正确收录 {path}。

要求：
- 如果该文档未被收录，按当前 topic 分组风格补入正确条目
- 如果索引中已有条目但标题、路径或 topic 不一致，顺手修正
- 不要引入重复条目
- 若只是正文轻量修改且路径、标题、topic 都未变，说明无需更新索引
```

## 6. 常用补充句

### 6.1 强制纯概念路径

```text
这个主题的主要难点是概念边界、命名区分和相邻概念辨析，不是独立机制本身。请按纯概念文档路径写，不要为了满足模板硬造核心机制、tradeoff 或工业锚点。
```

### 6.2 强制 `deep` 模式

```text
这不是 standard 模式文档。请按 deep 模式生成：展开到足以支持架构判断、排障和迁移，不要把复杂主题压缩成每节一小段摘要。
```

### 6.3 强制时间敏感纪律

```text
正文涉及当前实践、现行制度、当前产品行为或其他时间敏感内容。请显式写出核对日期，优先使用一手或官方来源，并区分事实、推断与结论。
```

### 6.4 只做正式达标，不做草率完成

```text
如果文档未达到 docs/methodology/concept-document-quality-gate.md 规定的正式标准，不要把它伪装成已经达标；请明确指出仍缺什么。
```

## 7. 你最少只需要提供什么

### 7.1 新建文档最小输入

1. 概念名
2. 主题
3. 遇到语境
4. 当前困惑

### 7.2 升级 / 审查 / 索引最小输入

1. 文档路径
2. 你要做的动作
3. 若有额外关注点，再补一句

## 8. 推荐使用习惯

以后建议把触发稳定分成四类：

- 新建文档：`为概念 X 新建一篇知识库文档`
- 升级旧文档：`按统一模板升级现有文档 Y`
- 审查达标：`按 concept-document-quality-gate 审查文档 Z`
- 仅更新索引：`检查 docs/index.md 是否正确收录文档 W`

这样“生成”“修订”“验收”“仓库集成”就不会再混在一起。

## 9. 一个新建示例：`memory order`

```text
为概念 memory order 新建一篇知识库文档。

要求：
1. 严格遵循：
   - docs/methodology/document-generation-methodology.md
   - docs/methodology/learning-new-things-playbook.md
   - docs/methodology/cognitive-modeling-playbook.md
   - docs/methodology/concept-document-template.md
   - docs/methodology/concept-document-quality-gate.md
2. 先判断它应走模型型概念文档还是纯概念文档路径；对 `memory order` 这类主题，应该走模型型概念文档路径。
3. 输出目标不是术语解释，而是建立可复用的内部模型。
4. 文档直接写入合适的 docs/{topic}/ 目录，补齐统一 frontmatter，并在完成后更新 docs/index.md。
5. 必须包含：问题定义、对象边界、核心结构、机制/因果链、关键 tradeoff 与失败模式、应用场景、工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题、参考资料。
6. 交付前按 docs/methodology/concept-document-quality-gate.md 自查，不要把未达标稿伪装成正式文档。
7. 重点讲清：
   - memory order 到底在解决什么问题
   - 它和 atomic、synchronization、visibility、reordering、happens-before 的边界
   - acquire / release / seq_cst / relaxed 的核心差异
   - 常见误区和真实工程中的使用判断

补充上下文：
- 所属主题：computer-systems
- 我是在并发/原子操作语境下遇到它的
- 我现在最不理解的是：为什么不同 memory order 会影响程序正确性，以及什么时候该选哪一种
- 我希望后续能拿它分析并发 bug、无锁数据结构和性能 tradeoff
```
