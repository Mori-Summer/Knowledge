---
doc_id: methodology-operator-guide
title: 方法论文档使用说明：旧版编排说明，已并入主规范
concept: methodology_operator_guide
topic: methodology
created_at: '2026-03-19T18:10:00+08:00'
updated_at: '2026-04-01T20:39:56+08:00'
source_basis:
  - methodology_repository_practice
  - concept_document_generation_workflow_review_2026_03_19
  - document_generation_methodology
  - consolidation_review_2026_04_01
time_context: reference_mode_baseline_2026_04_01
applicability: historical_navigation_reference_for_methodology
prompt_version: not_applicable
template_version: guide_reference_v2
quality_status: maintained_asset
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
open_questions:
  - 后续是否还需要单独保留这份旧版编排说明，还是只保留在主规范中的迁移说明即可？
---

# 方法论文档使用说明：旧版编排说明，已并入主规范

## 1. 当前状态

这份文件**不再是默认执行入口**。  
如果你要新建、升级、审查或集成正式概念文档，默认先看：

- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)

这份 `operator guide` 现在只保留为参考件，主要用于：

- 理解旧版方法论为什么被拆成多份文件
- 解释仓库里旧文档、旧 prompt 或旧说明中对它的引用
- 在维护 `docs/methodology/` 时，回看历史分层与迁移关系

## 2. 现在的默认入口应该怎么走

当前建议顺序已经收束成下面这条：

1. 先看 [document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
2. 直接执行新建、升级、审查或索引更新
3. 只有在主规范不足以支撑任务时，再进入参考件

当前参考件分工是：

- [fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)：快捷触发词
- [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)：完整章节骨架与分型细则
- [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)：完整 Hard Fail 与评分细则
- [learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)：学习目标与理解路径
- [cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)：边界、机制、约束与因果纪律

## 3. 旧版分层当时在解决什么问题

旧版拆分的出发点没有错。  
当时主要是想把几类不同职责分开：

- playbook：负责学习目标和建模纪律
- template：负责输出合同
- quality gate：负责验收
- prompt：负责固定触发方式
- operator guide：负责说明这些部件怎么组合

问题不在于拆分本身，而在于执行入口过多。  
当真正开始批量新建或升级文档时，使用者常常需要同时跳转多份文件，导致：

- 知道规则存在，但不知道默认先读哪一份
- 知道模板和门禁存在，但不知道执行顺序
- 知道有固定 prompt，但不知道它和模板、门禁、集成规范的关系

这也是为什么后来需要收束出一份主规范。

## 4. 旧版栈与主规范的映射关系

现在可以这样理解迁移关系：

- 旧版 `operator guide` 的“编排职责”已经并入 [document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
- 旧版 `prompt` 仍然保留，但现在属于快捷入口，而不是总规范
- 两份 playbook 仍然保留，但现在属于深度参考，而不是每次任务的强制首读
- `template` 与 `quality gate` 仍然保留，但现在主要承担完整细则，而不是并列主入口

一句话说：  
**旧版是“多文件并列协作”，现在是“主规范统筹，参考件补充”。**

## 5. 维护 `docs/methodology/` 时怎么判断写到哪

如果你在继续维护方法论文档，优先按下面规则判断：

- 任何每次新建、升级、审查都必须遵守的稳定规则，写进 [document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
- 任何只是为了快捷调用而存在的短入口，写进 [fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
- 任何需要保留完整章节骨架、完整评分细则、完整学习方法或完整建模方法的内容，写进对应参考件
- 任何只是解释旧版关系和历史分层的内容，才留在这份文件

默认动作应该是：  
**先收束入口，再补参考件，而不是继续新增并列主入口。**

## 6. 这份文件以后还值不值得看

只有在下面这些场景里，这份文件才值得优先打开：

- 你在读旧文档，发现它显式要求先看 `methodology-operator-guide.md`
- 你在清理或重构 `docs/methodology/`，想知道旧版分层是怎么形成的
- 你在判断某条规则应该进主规范还是只该留在参考件

如果你的目标只是“把当前文档任务做完”，这份文件通常不是第一入口。

## 7. 参考资料

- [统一概念文档规范：新建、升级、审查与仓库集成](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)
- [固定概念文档生成 Prompt](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
- [学习新事物的方法手册：从陌生到可理解、可操作、可迁移](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)
- [认知规范与问题建模手册](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
- [统一概念文档模板](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
- [统一概念文档质量门禁](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)
