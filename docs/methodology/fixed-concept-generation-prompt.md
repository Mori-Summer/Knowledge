---
doc_id: methodology-fixed-concept-generation-prompt
title: 固定概念文档生成 Prompt
concept: fixed_concept_doc_generation_prompt
topic: methodology
created_at: '2026-03-16T10:12:00+08:00'
updated_at: '2026-03-16T10:12:00+08:00'
source_basis:
  - architecture_rules
  - concept_template
time_context: prompt_baseline_2026_03_16
applicability: future_concept_doc_generation_and_upgrade
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: maintained_asset
related_docs:
  - docs/methodology/concept-document-template.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
open_questions:
  - 后续是否需要再拆出“新建文档 prompt”和“升级旧文档 prompt”两个独立版本？
---

# 固定概念文档生成 Prompt

这份文件定义以后你触发我生成知识文档时的固定入口。

目标不是“让 AI 解释一个词”，而是让它按你已经定下的知识库规则，生成一篇可直接进入 `docs/` 的模型化文档。

## 1. 最短可用指令

如果你想最快触发，直接发这一句：

```text
按 docs/methodology/concept-document-template.md 和两份 playbook，为概念 {concept} 生成一篇新的知识文档，并直接放入合适的 docs/{topic}/ 目录。不要写成泛泛解释，必须包含：问题定义、对象边界、核心结构、机制/因果链、关键 tradeoff 与失败模式、应用场景、工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题、参考资料，并补齐统一 frontmatter。
```

例如：

```text
按 docs/methodology/concept-document-template.md 和两份 playbook，为概念 memory order 生成一篇新的知识文档，并直接放入合适的 docs/{topic}/ 目录。不要写成泛泛解释，必须包含：问题定义、对象边界、核心结构、机制/因果链、关键 tradeoff 与失败模式、应用场景、工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题、参考资料，并补齐统一 frontmatter。
```

## 2. 推荐标准指令

如果你想让我一次做得更稳，推荐用这个完整版：

```text
为概念 {concept} 新建一篇知识库文档。

要求：
1. 严格遵循：
   - docs/methodology/learning-new-things-playbook.md
   - docs/methodology/cognitive-modeling-playbook.md
   - docs/methodology/concept-document-template.md
2. 输出目标不是术语解释，而是建立可复用的内部模型。
3. 文档必须直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，并补齐统一 frontmatter。
4. 文档必须至少包含：
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
5. 不能泛泛而谈。工业场景必须真实，若提到旧技术或旧做法，必须解释为什么旧、局限在哪、现在更推荐什么。
6. 文档要以“我读完后能分析问题、能迁移模型”为标准，不以“像一篇百科文章”为标准。

补充上下文：
- 所属主题：{topic}
- 我是在什么场景下遇到它的：{context_where_i_met_it}
- 我现在最不理解的点：{what_confuses_me}
- 我希望后续能拿它分析什么问题：{future_use_case}
```

## 3. `memory order` 的建议触发方式

你以后如果想生成 `memory order`，直接这样发就够了：

```text
为概念 memory order 新建一篇知识库文档。

要求：
1. 严格遵循：
   - docs/methodology/learning-new-things-playbook.md
   - docs/methodology/cognitive-modeling-playbook.md
   - docs/methodology/concept-document-template.md
2. 输出目标不是术语解释，而是建立可复用的内部模型。
3. 文档直接写入合适的 docs/{topic}/ 目录，补齐统一 frontmatter。
4. 必须包含：问题定义、对象边界、核心结构、机制/因果链、关键 tradeoff 与失败模式、应用场景、工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题、参考资料。
5. 重点讲清：
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

## 4. 如果你要“升级旧文档”

如果不是新建，而是让已有文档继续升级，直接用这个：

```text
按 docs/methodology/concept-document-template.md 升级现有文档 {path}。

要求：
- 保留原有高价值内容，不要整篇重写
- 补齐当前缺失的标准章节和 frontmatter 字段
- 把文档提升到可保留、可复用、可迁移的状态
- 重点补：工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题
```

## 5. 你以后只需要告诉我的最小信息

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

## 6. 最稳的使用习惯

以后建议你把触发分成两类：

- 新建文档：`为概念 X 新建一篇知识库文档`
- 升级旧文档：`按统一模板升级现有文档 Y`

这样我会自动进入两种不同的工作模式，而不是把“生成新文档”和“修旧文档”混在一起。
