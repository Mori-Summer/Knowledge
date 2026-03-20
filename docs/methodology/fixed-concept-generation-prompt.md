---
doc_id: methodology-fixed-concept-generation-prompt
title: 固定概念文档生成 Prompt
concept: fixed_concept_doc_generation_prompt
topic: methodology
created_at: '2026-03-16T10:12:00+08:00'
updated_at: '2026-03-20T14:19:46+08:00'
source_basis:
  - architecture_rules
  - concept_template
  - methodology_operator_guide
  - concept_document_quality_gate
time_context: prompt_baseline_2026_03_19
applicability: future_concept_doc_generation_and_upgrade
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: maintained_asset
related_docs:
  - docs/methodology/concept-document-template.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/concept-document-quality-gate.md
open_questions:
  - 未来是否需要把“生成”“升级”“审查”三种 prompt 完全拆成三个独立文件？
---

# 固定概念文档生成 Prompt

这份文件定义以后你触发概念文档生成与升级时的固定入口。

目标不是“让 AI 解释一个词”，而是让它按你已经定下的知识库规则，生成一篇可直接进入 `docs/` 的模型化文档，并完成必要的仓库集成动作。

建议配套使用顺序是：

1. 先看 [methodology-operator-guide.md](/Users/maxwell/Knowledge/docs/methodology/methodology-operator-guide.md)
2. 再按 [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md) 组织输出
3. 最后用 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md) 做验收

## 1. 最短可用指令

如果你想最快触发，直接发这一句：

```text
按 docs/methodology/methodology-operator-guide.md、docs/methodology/concept-document-template.md、docs/methodology/concept-document-quality-gate.md 和两份 playbook，为概念 {concept} 新建一篇知识库文档。直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，补齐统一 frontmatter，并在完成后更新 docs/index.md。不要写成泛泛解释，必须建立可复用内部模型；如果概念明显跨多层系统、多后端、多时间尺度或多类约束，默认按 deep 模式生成，不要只填满章节。
```

例如：

```text
按 docs/methodology/methodology-operator-guide.md、docs/methodology/concept-document-template.md、docs/methodology/concept-document-quality-gate.md 和两份 playbook，为概念 memory order 新建一篇知识库文档。直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，补齐统一 frontmatter，并在完成后更新 docs/index.md。不要写成泛泛解释，必须建立可复用内部模型。
```

## 2. 推荐标准指令

如果你想让我一次做得更稳，推荐用这个完整版：

```text
为概念 {concept} 新建一篇知识库文档。

要求：
1. 严格遵循：
   - docs/methodology/methodology-operator-guide.md
   - docs/methodology/learning-new-things-playbook.md
   - docs/methodology/cognitive-modeling-playbook.md
   - docs/methodology/concept-document-template.md
   - docs/methodology/concept-document-quality-gate.md
2. 输出目标不是术语解释，而是建立可复用的内部模型。
3. 先判断这篇文档采用 `standard` 还是 `deep` 模式；如果概念明显跨多层系统、多后端、多时间尺度或多类约束，默认按 `deep` 模式生成，并在 frontmatter 中显式写 `depth_mode`。
4. 文档必须直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，并补齐统一 frontmatter。
5. 文档必须至少包含：
   - 这份文档要帮我学会什么
   - 一句话结论 / 问题定义
   - 对象边界与相邻概念
   - 核心结构
   - 核心机制 / 主链路 / 因果链
   - 关键 tradeoff 与失败模式
   - 应用场景
   - 工业 / 现实世界锚点
   - 当前推荐实践、过时路径与替代
   - 自测题 / 验证入口
   - 迁移与关联模型
   - 未解问题与继续深挖
   - 参考资料
6. `deep` 模式下，不能只做章节填空；至少补强：层次结构、完整主链路、多个有差异的应用场景、多个真实工业锚点、明确的旧路径与替代路径判断。
7. 不能泛泛而谈。工业场景必须真实，若提到旧技术或旧做法，必须解释为什么旧、局限在哪、现在更推荐什么。
8. 文档要以“我读完后能分析问题、能迁移模型”为标准，不以“像一篇百科文章”为标准。
9. 如果正文涉及当前实践、现行制度、最新状态、当前产品行为或其他时间敏感内容，必须：
   - 明确写出核对日期
   - 优先使用一手或官方来源
   - 区分事实、推断与结论
10. 新增正式文档后，顺手更新 docs/index.md。
11. 交付前按 docs/methodology/concept-document-quality-gate.md 自查；如果不满足 `upgraded_v1` 条件，就不要伪装成已经达标。

补充上下文：
- 所属主题：{topic}
- 我是在什么场景下遇到它的：{context_where_i_met_it}
- 我现在最不理解的点：{what_confuses_me}
- 我希望后续能拿它分析什么问题：{future_use_case}
- 这个概念是否时间敏感：{is_time_sensitive}
- 我希望看到哪些真实工业锚点：{preferred_real_world_anchors}
```

## 3. 如果你要明确要求 `deep` 模式

如果你已经知道这是个复杂主题，建议直接补这一句：

```text
这不是 standard 模式文档。请按 deep 模式生成：展开到足以支持架构判断、排障和迁移，不要把复杂主题压缩成每节一小段摘要。
```

更完整一点，可以直接这样发：

```text
为概念 {concept} 新建一篇知识库文档，并按 deep 模式生成。

要求：
- 严格遵循 docs/methodology 下的 operator guide、template、quality gate 和两份 playbook
- 在 frontmatter 中显式写 `depth_mode: deep`
- 至少给出一条完整主链路；必要时补一条变体链路
- 至少给出 3 个有差异的应用场景
- 至少给出 2 个真实工业/现实世界锚点，并解释它们为什么重要
- “当前推荐实践、过时路径与替代”必须写清为什么旧、局限在哪、现在更推荐什么、什么条件下选它
- 自测题至少包含预测题、诊断题或选型题，不要只做术语回忆
```

## 4. 一个技术概念示例：`memory order`

你以后如果想生成 `memory order`，直接这样发就够了：

```text
为概念 memory order 新建一篇知识库文档。

要求：
1. 严格遵循：
   - docs/methodology/methodology-operator-guide.md
   - docs/methodology/learning-new-things-playbook.md
   - docs/methodology/cognitive-modeling-playbook.md
   - docs/methodology/concept-document-template.md
   - docs/methodology/concept-document-quality-gate.md
2. 输出目标不是术语解释，而是建立可复用的内部模型。
3. 文档直接写入合适的 docs/{topic}/ 目录，补齐统一 frontmatter，并在完成后更新 docs/index.md。
4. 必须包含：问题定义、对象边界、核心结构、机制/因果链、关键 tradeoff 与失败模式、应用场景、工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题、参考资料。
5. 交付前按 docs/methodology/concept-document-quality-gate.md 自查，不要把未达标稿伪装成正式文档。
6. 重点讲清：
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

## 5. 如果你要“升级旧文档”

如果不是新建，而是让已有文档继续升级，直接用这个：

```text
按 docs/methodology/methodology-operator-guide.md、docs/methodology/concept-document-template.md 和 docs/methodology/concept-document-quality-gate.md 升级现有文档 {path}。

要求：
- 保留原有高价值内容，不要整篇重写
- 补齐当前缺失的标准章节和 frontmatter 字段
- 把文档提升到可保留、可复用、可迁移的状态
- 重点补：工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题
- 如果原主题明显复杂但当前文档过短或过稀，按 `deep` 模式补足展开密度，不要只修补章节名
- 如果正文包含时间敏感内容，补上核对日期和更强来源
- 如果升级完成后达到正式文档标准，顺手更新 docs/index.md（若该文档尚未被索引）
```

## 6. 你以后只需要告诉我的最小信息

最少给我这 4 项，我就能开工：

1. 概念名
2. 主题
3. 你在哪个语境下遇到它
4. 你现在最不理解的点

也就是这种最小格式：

```text
概念：memory order
主题：computer-systems
遇到语境：看 C++ 原子操作和并发资料时遇到
当前困惑：不知道 relaxed/acquire/release/seq_cst 到底差在哪，为什么会影响正确性
```

## 7. 最稳的使用习惯

以后建议你把触发分成三类：

- 新建文档：`为概念 X 新建一篇知识库文档`
- 升级旧文档：`按统一模板升级现有文档 Y`
- 审查达标：`按 concept-document-quality-gate 审查文档 Z`

这样我会自动进入三种不同的工作模式，而不是把“生成”“修订”“验收”混在一起。
