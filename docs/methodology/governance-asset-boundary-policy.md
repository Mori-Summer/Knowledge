---
doc_id: methodology-governance-asset-boundary-policy
title: 方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工
concept: governance_asset_boundary_policy
topic: methodology
depth_mode: standard
created_at: '2026-05-21T11:39:02+08:00'
updated_at: '2026-05-25T09:34:08+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/planning-artifacts/implementation-readiness-report-corrected-2026-05-20.md
  - _bmad-output/implementation-artifacts/0-1-agent-behavior-constraints.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/methodology-operator-guide.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
time_context: phase_4_foundation_2026_05_25
applicability: methodology_asset_boundary_and_change_impact_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/methodology-operator-guide.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
open_questions:
  - Story 0.3 定义正式生命周期状态后，是否需要调整本资产的 quality_status？
  - Story 0.4 定义 prompt/template/quality version governance 后，是否需要补充更细的版本变更记录格式？
  - Story 0.5 固化质量门禁基线后，是否需要把本资产的影响分析清单对齐到新的质量证据格式？
---

# 方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工

## 资产角色、权威与适用范围

这份文档是 `Knowledge` 项目的正式方法论治理资产。它的职责是固定 `docs/methodology/` 中各类方法论资产的使用边界、权威关系、冲突处理方式和变更影响分析要求。

它约束的是方法论资产如何被读取、引用、修改和调和，不是新的概念文档生成流程，也不替代 `docs/methodology/document-generation-methodology.md`。

本资产适用于：

- 判断正式概念文档任务应该先读取哪一份方法论资产
- 判断 template、quality gate、fixed prompt、playbook、operator guide 是否越界成为平行主规范
- 修改 prompt、template、方法论、质量门禁、playbook 或 operator guide 前做影响分析
- 发现方法论资产互相冲突时记录冲突、应用优先级并决定是否停止确认
- 汇报方法论或治理资产变更时提供可追溯的验证证据

本资产不适用于：

- 直接生成某一篇具体概念文档的正文
- 定义最终生命周期状态词表
- 定义最终 prompt/template/quality version governance 规则
- 定义最终质量状态词表或 Hard Fail 基线
- 替代 `docs/governance/batch-readiness-checklist.md` 中的批量治理 readiness 策略

这些内容分别由对应 Foundation 治理资产承担。本资产只能说明方法论资产边界和影响分析，不能替代生命周期、版本、质量、导航或 batch readiness 的 canonical policy。

## 方法论资产职责地图

`docs/methodology/document-generation-methodology.md` 是正式概念文档工作的主规范和默认执行入口。只要任务目标是新建、升级、审查或仓库集成正式概念文档，Agent 默认先读取并服从这份主规范。

| 资产 | 角色 | 权威边界 | 不得做什么 |
| --- | --- | --- | --- |
| `docs/methodology/document-generation-methodology.md` | 主规范、默认执行入口 | 定义正式概念文档的新建、升级、审查、索引集成和质量门禁接入方式 | 不应被支撑资产绕过或平行替代 |
| `docs/methodology/concept-document-template.md` | 结构参考与章节骨架支持 | 提供文档类型、深度模式和推荐结构细则 | 不得单独成为执行流程或覆盖主规范 |
| `docs/methodology/concept-document-quality-gate.md` | 验收与审查门禁支持 | 提供 Hard Fail、评分维度、证据要求和审查输出标准 | 不得替代主规范的任务分类、仓库集成和入口顺序 |
| `docs/methodology/fixed-concept-generation-prompt.md` | 可复用调用入口和触发词资产 | 固定高频新建、升级、审查、索引任务的 prompt 形态 | 不得扩展成更广的方法论来源或覆盖主规范 |
| `docs/methodology/learning-new-things-playbook.md` | 学习目标、理解路径和迁移意识的背景参考 | 帮助补齐“为什么学、要掌握什么、如何验证理解”的输入纪律 | 不得成为每次任务的强制首读主入口 |
| `docs/methodology/cognitive-modeling-playbook.md` | 建模纪律、边界、变量、因果和证据层的背景参考 | 帮助复杂主题建立更强判断结构 | 不得替代概念文档模板或质量门禁 |
| `docs/methodology/methodology-operator-guide.md` | 旧版编排说明和历史导航参考 | 解释旧版多文件分层为何收束为主规范统筹 | 不得恢复为默认执行入口 |
| `docs/governance/agent-behavior-constraints.md` | 相邻 Agent 行为约束 | 约束任务开始声明、资产层级、非软件边界、workflow contract 和完成证据 | 不得改写具体方法论资产的正文合同 |

一句话规则是：主规范负责执行合同，支撑资产负责结构、门禁、触发、背景或历史说明。支撑资产可以补细节，但不能通过暗示、旧链接或局部措辞变成平行主规范。

## 执行入口与读取顺序

正式概念文档任务的默认读取顺序是：

1. 读取当前用户指令、适用 BMad workflow 或 story step。
2. 读取 `_bmad-output/project-context.md` 和相关 Agent 行为约束，确认项目边界和允许修改范围。
3. 读取 `docs/methodology/document-generation-methodology.md`，完成任务类型、文档路径类型和深度模式判断。
4. 仅在需要对应细则时读取支撑资产：
   - 需要章节结构或分型细节时读取 `concept-document-template.md`。
   - 需要 Hard Fail、评分或验收证据时读取 `concept-document-quality-gate.md`。
   - 需要复用固定触发词时读取 `fixed-concept-generation-prompt.md`。
   - 需要学习目标或理解路径时读取 `learning-new-things-playbook.md`。
   - 需要更强建模纪律时读取 `cognitive-modeling-playbook.md`。
   - 需要理解旧版分层时读取 `methodology-operator-guide.md`。

如果主规范已经足以完成任务，不必强制回读所有支撑资产。反过来，如果某个支撑资产包含更细的检查项，Agent 可以用它补足验证证据，但必须把它标记为支撑来源，而不是更高一级的主规范。

## 冲突记录与处理规则

当方法论资产之间出现冲突时，Agent 不得静默选择最方便的一边继续写。必须先记录冲突，再判断是否可以在当前授权范围内修复。

冲突记录至少包含：

| 字段 | 说明 |
| --- | --- |
| 冲突资产 | 发生冲突的文件路径或 workflow/skill 来源 |
| 受影响文件 | 本次将被修改、引用、索引或验证的文件 |
| 冲突主张 | 每个来源分别要求什么，冲突点是什么 |
| 冲突类型 | 主入口冲突、质量门禁冲突、模板结构冲突、prompt 行为冲突、索引/导航冲突、Agent 行为冲突、生命周期/状态冲突等 |
| 优先级规则 | 本次采用哪条规则解决，以及为什么适用 |
| 拟议处理 | 修正文案、保留差异、暂停确认、拆分后续 story、或记录为未决问题 |
| Maxwell 确认条件 | 是否需要停止并请求 Maxwell 明确确认 |

方法论资产冲突的优先级为：

1. 当前 Maxwell 明确指令和适用 BMad workflow/story step。
2. `_bmad-output/project-context.md`、Agent 行为约束和项目级执行边界。
3. `docs/methodology/document-generation-methodology.md` 主规范。
4. `concept-document-quality-gate.md`、`concept-document-template.md`、`fixed-concept-generation-prompt.md`、两份 playbook 和 `methodology-operator-guide.md` 等支撑资产。
5. 单篇旧文档、旧 prompt、历史说明或临时 workflow output 中的局部习惯。

只要冲突会改变以下任一事项，Agent 必须停止并请求 Maxwell 确认：

- 主规范是否仍是默认执行入口
- 支撑资产是否被提升为平行主规范
- quality gate 的 Hard Fail 或评分标准是否改变
- fixed prompt 的触发行为或任务分类是否改变
- `docs/index.md` 或方法论导航入口是否改变
- Agent 行为约束、workflow contract 或等待点是否改变
- 生命周期状态、版本治理、质量状态词表或批量 readiness 是否被提前定义

如果冲突只是本次新资产内部需要说明边界，并且不修改已有主规范或支撑资产，可以在本资产中记录处理规则后继续。

## 变更影响分析

修改 prompt、template、methodology、quality gate、playbook 或 operator guide 前，必须先说明影响范围。完成后，汇报必须覆盖实际影响和未影响项。

影响分析至少检查：

- 生成文档行为：是否改变新建、升级、审查、索引任务的默认步骤或输出结构
- 审查决策：是否改变 Hard Fail、评分维度、质量证据或通过/退回判断
- 索引集成：是否需要更新 `docs/index.md`、方法论导航、相关文档链接或入口说明
- Agent 行为：是否改变任务开始声明、允许修改范围、停止条件、workflow contract 或完成证据
- `prompt_version` 与 `template_version`：是否需要更新已有或未来文档的版本字段含义
- `quality_status`：是否改变文档状态词表、晋升条件或保守标记策略
- `related_docs`：是否需要新增、移除或重排相关文档关系
- `open_questions`：是否新增未决治理问题或清理已经解决的问题
- 迁移义务：是否需要批量扫描旧文档、重写旧链接、更新旧 prompt、或创建后续 story

影响分析不能只写“无影响”。如果判定无影响，必须说明检查了哪些范围，为什么不需要变更。

Story 0.2 的边界是建立职责边界和影响分析合同，不定义最终 lifecycle vocabulary、最终 version governance、最终 quality-status vocabulary 或 batch-readiness policy。批量治理 readiness 现在由 `docs/governance/batch-readiness-checklist.md` 承担；本资产只在方法论资产变更影响分析中引用其停止条件和范围判定要求。

## 导航与索引处理

本资产位于 `docs/methodology/governance-asset-boundary-policy.md`，并应列入 `docs/index.md` 的 `methodology` 分组。

把本资产加入索引表示它是正式方法论治理资产；这不表示它替代主规范，也不表示所有支撑资产都必须反向链接它。

本次新增本资产不自动要求修改 `docs/methodology/document-generation-methodology.md`。只有当 Maxwell 后续授权新增主规范交叉引用、方法论入口重组或导航策略调整时，才应修改主规范正文。

## 完成汇报与验证证据

涉及方法论资产边界的实施完成汇报必须包含：

- 任务类型和资产层级
- 变更文件和明确未变更文件
- 采用的主规范、支撑资产和 workflow/story 来源
- 是否存在冲突；如存在，冲突记录和处理结论
- `docs/index.md` 的导航处理
- frontmatter、标题层级、路径、文件名、链接、索引、质量门禁和 workflow-contract 验证证据
- 非软件边界确认：没有新增 package manifest、source tree、test framework、CLI/API/UI/database/deployment/CI 或 runtime automation
- 未解决问题、后续维护触发点和需要 Maxwell 未来确认的事项

验证证据必须指向具体文件或具体字段，不能只写“已检查”。如果某项不适用，必须说明原因。

## 维护与迁移入口

本资产的维护触发点包括：

- Story 0.3 定义正式生命周期状态和状态转换规则后，复核 `quality_status` 与状态词表是否对齐。
- Story 0.4 建立 prompt、template 与 quality version governance 后，补充版本字段变更记录格式。
- Story 0.5 固化质量门禁 Hard Fail、评分维度与质量状态基线后，复核本资产的验证证据清单。
- `docs/governance/governance-asset-navigation-policy.md` 已建立治理资产导航、索引与入口归属基线；后续如改变主规范交叉引用或导航策略，应按该政策复核。
- `docs/governance/batch-readiness-checklist.md` 已定义批量治理 readiness；批量变更触及方法论资产时，必须按该政策补充 target set、excluded files、冲突暴露、停止条件和 downstream impact analysis。

如果未来要把本资产从 `draft` 晋升为更高状态，至少应确认：

- 后续 Foundation story 没有与本资产冲突的最终规则
- `docs/index.md` 中的导航位置仍正确
- `source_basis`、`time_context`、`related_docs` 和 `open_questions` 与当前治理状态一致
- 本资产没有被误用为概念文档生成主流程
