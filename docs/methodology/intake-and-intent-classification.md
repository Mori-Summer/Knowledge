---
doc_id: methodology-intake-and-intent-classification
title: 输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理
concept: intake_and_intent_classification
topic: methodology
depth_mode: standard
created_at: '2026-05-25T14:20:37+08:00'
updated_at: '2026-06-17T09:15:48+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/1-1-concept-document-generation-contract.md
  - _bmad-output/implementation-artifacts/1-2-intake-and-intent-classification.md
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_core_methodology_review_2026_06_17
applicability: intake_intent_classification_for_knowledge_documentation_tasks
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
open_questions:
  - frontmatter schema 已建立后，是否需要把部分 intake 输出字段 schema 化？
  - review/completion templates 后续若调整 scope classification 字段，是否需要把 intake decision record 抽成固定报告项？
  - Epic 6 batch runbook 落地后，是否需要把 batch-governance routing 与 batch execution steps 进一步拆分？
---

# 输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目的正式方法论支撑资产，定义 Agent 在产出内容或写入文件前如何摄入输入、判断任务意图、选择治理路径、识别缺失输入、确定验证证据和停止条件。

本文适用于 Knowledge 文档治理请求的前置判定，包括正式概念文档、方法论资产、治理资产、索引、BMad workflow output、skill/workflow 维护、规划产物、废弃归档和批量治理。它回答的是“这次请求到底是什么类型、应该写到哪里、应该按什么规则验证、缺什么必须问清楚，以及什么时候不能继续写”。

本文补充 `docs/methodology/document-generation-methodology.md`、`docs/methodology/concept-document-contract.md`、`docs/methodology/concept-document-template.md`、`docs/methodology/concept-document-quality-gate.md`、`docs/methodology/fixed-concept-generation-prompt.md`、`docs/governance/agent-behavior-constraints.md`、`docs/governance/governance-asset-navigation-policy.md`、`docs/governance/lifecycle-states.md`、`docs/governance/prompt-template-quality-version-governance.md`、`docs/governance/revision-regeneration-continuity-policy.md` 和 `docs/governance/batch-readiness-checklist.md`。

主执行入口仍按任务类型选择：

- 正式概念文档的新建、升级、审查和仓库集成，仍以 `docs/methodology/document-generation-methodology.md` 为主规范。
- 概念文档候选稿的输入、输出、必需信息位点和出界边界，仍以 `docs/methodology/concept-document-contract.md` 为合同。
- 质量门禁和审查证据，仍以 `docs/methodology/concept-document-quality-gate.md` 为准。
- 修订、结构升级、重生成、split、merge、deprecation 或 reference-validity impact，必须在适用时调用 `docs/governance/revision-regeneration-continuity-policy.md` 的 update-mode taxonomy 和 Continuity Record；迁移、生命周期和索引字段权威仍归各自治理资产。
- 批量治理写入前的 readiness，仍以 `docs/governance/batch-readiness-checklist.md` 为准。

本文不替代主方法论、概念文档生成合同、模板、质量门禁、固定 Prompt、生命周期政策、版本治理政策、导航政策、frontmatter/index schema、已落地 review/completion assets、document decision policy 或批量 readiness 政策；也不提前定义 Epic 6 的 batch governance runbook、batch review record 或 batch completion report。

本文定义的是人工可执行的文档治理判断流程，不是可执行分类器、CLI、API、UI、数据库、测试框架、lint/scoring 脚本、自动化工具或运行时工作流。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：任务类型分类、概念文档分型、缺失输入处理、`_bmad-output` 边界、路由输出、状态词映射、owner/index entry、相关治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

本文的 owner entry point 是 `docs/index.md` 的 `methodology` 分组。Navigation treatment 为 `listed_in_docs_index`；index treatment 是在 `docs/index.md` 的 `## methodology` 下列出 `docs/methodology/intake-and-intent-classification.md`。这个索引入口表示本文是可发现的 intake 支撑资产，不表示它成为平行主规范。

## Intake 顺序

每次 Knowledge 文档治理请求开始时，先按下面顺序判定。不要先写正文、先改索引或先补 frontmatter。

1. 读取当前用户指令和适用 BMad workflow/story。
2. 判断输入来源：用户直接请求、正式 `docs/` 资产、`_bmad-output/` planning artifact、review output、Agent discussion output、候选稿或已存在的 formal asset。
3. 判断资产层级：普通概念文档、方法论资产、治理资产、模板、索引、runbook、workflow output、报告、BMad skill/workflow 或 mixed scope。
4. 判断顶层任务类型：九类任务之一，见下一节。
5. 如果属于正式概念文档工作，先判定文档路径类型：模型型概念文档或纯概念文档。
6. 在路径类型确定后，再判定展开密度：`standard` 或 `deep`。
7. 检查必需输入和治理关键输入是否足够。
8. 决定输出路由、允许文件范围、验证证据和停止条件。
9. 只有上述判断足够清楚，才进入写入、审查或完成汇报。

Intake 的输出不需要新增 frontmatter 字段。它可以写在完成汇报、story Dev Agent Record、review record、readiness record 或正文中，前提是审查者能看出任务类型、资产层级、规则来源、允许范围、验证证据和未解决风险。

## 任务类型分类表

请求必须先落到一个主任务类型。若请求混合多个任务类型，先拆分；无法拆分时按更严格的类型处理，或停止请求 Maxwell 确认。

| 任务类型 | 典型触发 | 目标资产层级与路由 | 允许范围 | 输出与验证 | 停止条件 |
| --- | --- | --- | --- | --- | --- |
| `new document creation` | 新建概念文档、新增正式方法论/治理资产、把候选稿晋升为正式资产 | 正式 `docs/` 资产；概念文档走主方法论、合同、模板、质量门禁 | 明确目标文件、必要相邻引用、`docs/index.md` | 新 Markdown 资产、frontmatter、role/scope、路径/topic/name、索引、链接、质量证据 | topic/path/asset level 不清；候选稿被要求直接发布；必需输入缺失 |
| `upgrade` | 升级旧文档、补结构、补来源、修 Hard Fail、提高复用性 | 现有 formal asset；概念文档先走质量门禁 gap analysis；涉及 targeted revision、structural upgrade、regeneration、split、merge、deprecation 或 reference-validity impact 时调用 Story 4.1 continuity policy | 目标文件和必要索引/链接同步 | 保留稳定 `doc_id` 和高价值内容；记录质量、来源、索引、版本影响；适用时记录 update mode、Continuity Record 和 Migration/Index impact | 要求默认整篇重写；需要改 `doc_id`、topic、status 但无证据 |
| `review` | 审查、验收、code-review、quality gate、readiness check | review output 或 story review；概念文档走质量门禁，非概念资产走等价治理检查 | 默认只产出审查结论，除非任务明确要求修复 | 前置分类、Hard Fail、修复指导、结论、风险和允许状态 | 审查中暗自修改目标；缺证据却声称通过 |
| `index-only update` | 只补 `docs/index.md`、修导航、检查是否收录 | `docs/index.md` 和已存在 formal asset | index 条目与必要 metadata；不改正文内容 | 标题、路径、topic、索引分组一致；无重复入口 | 目标资产尚未正式晋升；index 更新会制造 hidden promotion |
| `methodology maintenance` | 修改主规范、模板、质量门禁、fixed prompt、playbook、方法论支撑资产 | `docs/methodology/` | 命名目标文件和必要窄同步 | role/authority/scope、主规范关系、generation behavior、quality gate、index/navigation、Agent behavior、版本治理、`quality_status`、`related_docs`、`open_questions`、migration obligation 和下游影响 | 支撑资产将变成平行主规范；改变 prompt/template/quality 行为但未记录影响 |
| `BMad skill/workflow maintenance` | 改 `.agents/skills/`、workflow step、trigger、wait、state field、on-complete | `.agents/skills/` 或 BMad workflow contract | 明确 skill/workflow 文件和状态字段 | trigger、输入、输出、等待点、状态、完成条件和回退风险 | workflow contract impact 不确定；用户未授权改 skill/workflow |
| `planning artifact` | PRD、Architecture、Epics、sprint planning、story、readiness report、review output | `_bmad-output/` workflow output | story/status/report 文件；不自动进 `docs/` | planning context、candidate source、evidence record 或 story status | 把 planning output 直接当正式 `docs/` 规则或加入索引 |
| `archive/deprecation` | 废弃、归档、替代、降低可见性、保留历史入口 | formal asset lifecycle | 目标文件、successor、索引、入链/出链 | reason、successor、remaining access、index/link impact、生命周期证据 | 没有 successor/索引/链接证据；批量废弃无 readiness |
| `batch governance` | 批量生成、迁移、重构、状态改写、链接/索引更新、批量审查 | 多文件、多 topic、多状态或多 workflow target | 先写 readiness record；未通过前不写目标集 | target set、allowed/excluded files、conflicts、recovery、owner decision | target set 模糊；试图无 readiness 执行 batch write |

如果一个请求看似只是“顺手改一下”，但会影响多个正式资产、多个 topic、多个状态字段、多个 index 条目或多个 workflow contract，必须升级为 `batch governance` 或停止确认。

## 正式概念文档分型

正式概念文档工作必须按固定顺序判定：

1. 先判断任务类型：新建、升级、审查或仅更新索引。
2. 再判断文档路径类型：模型型概念文档或纯概念文档。
3. 最后判断展开密度：`standard` 或 `deep`。

路径判断先于深度判断。`standard` 和 `deep` 只解决展开密度和证据负担，不解决一篇文档到底应该写机制链还是判别框架。

### 模型型概念文档

如果主题具有稳定结构、机制、主链路、因果链、失败模式、tradeoff、选型价值、诊断价值或系统行为，应走模型型路径。

模型型路径必须让读者能回答：

- 它解决什么结构性问题。
- 它由哪些部件、层次或分面构成。
- 它如何运转，主链路或因果链是什么。
- 哪些条件下适用，哪些条件下失效。
- 关键 tradeoff 和 failure modes 是什么。
- 如何用它解释、预测、调试、取舍或迁移。

### 纯概念文档

如果主题主要解决命名、分类、边界、相邻概念区分、例子/反例、false equivalence、misuse 或上层模型术语稳定性，应走纯概念路径。

纯概念路径必须让读者能回答：

- 这个概念到底在命名什么，为什么需要单独命名。
- 什么算它，什么不算它。
- 它和相邻概念怎么稳定区分。
- 代表性例子、反例、误读和误用是什么。
- 它后续会进入哪些更大的模型、解释框架或判断流程。

不得为了套模板把纯概念硬写成伪机制文档，也不得把需要机制判断的对象降成术语说明。

### `standard` 与 `deep`

`standard` 更适合：

- 单主机制或单主判别问题。
- 单时间尺度。
- 边界较清楚。
- 相邻概念集合较小。
- 目标是建立稳定定义、边界和最小复用判断。

`deep` 更适合：

- 多层抽象、多后端、多 actor、多执行面或多时间尺度同时存在。
- 当前实践与历史路径都重要。
- 主题会被用于架构判断、排障、选型或复杂迁移。
- failure modes 和 tradeoffs 本身构成理解核心。
- 相邻概念网络复杂，常见误读路径很多。

命中 `deep` 条件两条及以上时，默认按 `deep` 处理，除非用户明确要求先产出受限的 `standard` 候选稿并记录范围限制。

路径类型当前不新增 frontmatter 字段。`depth_mode` 仍只写 `standard` 或 `deep`。路径判断和依据写在正文、完成汇报或审查输出中。

`standard` 与 `deep` 改变的是证据密度、例子数量、来源纪律、链路深度和迁移要求，不改变 `docs/methodology/concept-document-contract.md` 中的核心合同面。边界、相邻概念、验证入口、迁移入口和 open questions 不能因为是 `standard` 就缺失。

## 输入充分性与缺失输入处理

概念文档生成至少需要以下六个输入：

| 必填输入 | 必须知道什么 |
| --- | --- |
| concept name | 要处理的概念、知识点或问题对象 |
| topic | 预期归属主题或目录 |
| user context | 用户是在什么学习、工作、阅读或问题场景下遇到它 |
| current confusion | 用户现在最不理解、最容易混淆或最需要判断的点 |
| intended downstream use | 用户后续想拿它分析、判断、排查或迁移到哪里 |
| time-sensitivity notes | 是否涉及当前实践、标准、产品、监管、市场、历史路径或废弃做法 |

其他任务也必须满足治理关键输入。缺失项不一定都阻塞写作，但必须明确处理。

| 缺失输入类别 | 典型例子 | 默认处理 |
| --- | --- | --- |
| 任务动作不清 | 不知道是新建、升级、审查、索引、维护、规划、废弃还是批量治理 | `clarify_before_write` |
| 目标文件或路径不清 | 只说“改这个文档”但没有路径；新资产 topic 不明确 | `clarify_before_write` |
| 资产层级不清 | 不知道目标是 formal docs、methodology、governance、workflow output 还是 skill | `clarify_before_write` |
| 来源或时间语境不清 | 涉及当前实践、产品、标准、监管、历史路径但没有来源或日期 | `record_open_question`；若要声明 current/validated，则 `stop_for_maxwell_confirmation` |
| 下游用途不清 | 不知道用户要拿结果分析什么，导致深度或边界无法判断 | 概念文档默认 `clarify_before_write`；非阻塞场景可 `record_open_question` |
| 质量或状态声明不清 | 想标 reviewed、validated、maintained、upgraded、deprecated、archived 但无证据 | `stop_for_maxwell_confirmation` 或退回质量门禁 |
| canonical vs candidate 不清 | 候选稿、对话文本或 `_bmad-output` 被要求直接进入正式资产 | `clarify_before_write` |
| index 或 promotion 预期不清 | 是否加入 `docs/index.md`、是否晋升为 formal docs 没说明 | `clarify_before_write` |
| batch target set 不清 | “批量处理相关文档”但未列目标集、排除项和恢复策略 | `stop_for_maxwell_confirmation`，先走 batch readiness |
| 废弃/归档 successor 不清 | 要废弃文档但无替代、剩余访问和索引处理 | `clarify_before_write` |
| BMad workflow/story target 不清 | 不知道要改哪个 story、skill、status 或 workflow field | `clarify_before_write` |
| 允许文件范围不清 | 相邻文件是否可改、是否可改 `.agents/skills/` 或规划产物不明 | `clarify_before_write` |

处理动作含义：

- `clarify_before_write`：先问 Maxwell 或从明确上下文中补足证据，未澄清前不写目标文件。
- `record_open_question`：可以写候选或 draft，但必须在正文、frontmatter `open_questions`、completion report 或 Dev Agent Record 中记录限制。
- `proceed_with_explicit_assumption`：只适用于非治理关键缺口；假设必须写明，且不得支持 validated/current/canonical 状态声明。
- `defer_to_later_story`：缺口属于明确未来 story 或授权外后续工作，例如 schema extension、template field change、link/network batch records 或 Epic 6 runbook。
- `stop_for_maxwell_confirmation`：缺口会影响 canonical path、身份、状态、批量目标、workflow contract、自动化授权或正式发布。

这些处理动作是 intake 或 Batch Readiness owner-decision 语境下的本地动作值，不是 review/completion final decision labels。写入 review record、completion report 或 story final decision 时，必须映射到当前 document decision vocabulary：需要确认的情况写 `hold_for_clarification`，后续 story 或授权外延期写 `defer_with_reason`，明确越界写 `not_authorized`；blocking/status 字段可使用 `held`、`deferred_with_reason` 或 `not_authorized`。

Agent 不得静默发明 topic、target path、source basis、currentness、downstream use、user confusion、formal status、lifecycle status、index treatment、batch scope、related docs、successor 或 owner decision。

## `_bmad-output` 与对话输出边界

`_bmad-output/` 是 BMad workflow output 生命周期，不是正式 `docs/` 生命周期。Agent discussion output 也不是正式文档本身。

| 来源状态 | 可以怎么用 | 不得怎么用 |
| --- | --- | --- |
| planning artifact | PRD、Architecture、Epics、sprint plan、readiness report 可作为上下文和约束来源 | 不得直接加入 `docs/index.md` 或覆盖 formal docs |
| story file | 可作为实施合同、任务边界、验收条件和 Dev Agent Record | 不得当成正式方法论资产发布 |
| review output | 可作为问题证据、修复依据和 completion evidence | 不得自动改写成正式规则，除非用户确认且完成 promotion |
| candidate source | 可作为候选正文、结构或草稿 | 不得跳过 frontmatter、path、index、quality evidence 直接发布 |
| user-approved rule | 可作为 formalization 依据 | 不等于已经成为 `docs/` 资产 |
| formal `docs/` asset | 已位于正确 `docs/` 路径并完成集成验证 | 不由 `_bmad-output/` 文件自行产生 |

从 `_bmad-output/` 或对话输出晋升到 `docs/` 的最低检查是：

1. 判定 task type 和 asset level。
2. 确认 target path、topic、文件名和稳定身份。
3. 补齐正式 YAML frontmatter，且不新增未经授权字段。
4. 声明 role、authority、applicable scope、owner entry point、main methodology relationship。
5. 说明 source basis、time context、quality status、lifecycle impact 和 open questions。
6. 选择 navigation treatment，并处理 `docs/index.md` 或记录受控排除。
7. 验证 related docs、正文链接、heading hierarchy、frontmatter array fields 和正文主张一致。
8. 在 completion report 或 story Dev Agent Record 中记录验证结果、未解决风险和非软件边界。

禁止把 `_bmad-output/` 文件直接加入 `docs/index.md`。如果一个 `_bmad-output/` 文件只是 story、review output 或 completion record，它的 owner entry point 仍是 BMad workflow 或 sprint tracking。

## 路由输出、验证证据与停止条件

Intake 完成后，输出应能让后续执行者直接知道写什么、改哪里、怎么验证、何时停止。

| 任务类型 | 预期输出 | 最低验证证据 |
| --- | --- | --- |
| `new document creation` | 新 formal Markdown asset 或明确 candidate | frontmatter、role/scope、path/topic/name、合同信息位点、source/time、index/link、quality gate |
| `upgrade` | 修改后的 existing asset 和 completion evidence | 保留身份、gap 修复、source/time、quality gate、index/link、version/status impact；适用时包含 Story 4.1 update-mode classification、Continuity Record、reference-validity evidence 和 Migration/Index records |
| `review` | review report 或 story review section | 前置分类、Hard Fail/等价治理检查、证据位置、修复指导、最终结论 |
| `index-only update` | `docs/index.md` 变更或不适用说明 | target exists、title/path/topic 一致、无重复、metadata 更新 |
| `methodology maintenance` | 方法论/支撑资产变更和影响分析 | role/authority、main-spec relationship、version governance、index、downstream impact |
| `BMad skill/workflow maintenance` | workflow/skill 变更或审查记录 | trigger、input、output、steps、waiting point、state fields、on-complete |
| `planning artifact` | `_bmad-output/` story/report/status 更新 | workflow contract、status transition、file list、non-promotion boundary |
| `archive/deprecation` | deprecation/archive note 或 asset update | reason、successor、remaining access、index/link/lifecycle impact |
| `batch governance` | readiness record and owner decision | target set、allowed/excluded files、conflicts、recovery、sampling/review、stop conditions |

通用验证证据包括：

- Markdown structure：H1/title 一致，heading hierarchy 稳定，表格和列表无破损。
- Frontmatter：必填字段存在，数组字段是 YAML arrays，字段值与正文、路径、来源、时间、状态一致。
- Role/authority/scope：资产职责、权威边界、适用范围和不替代关系明确。
- Repository placement：路径、topic、文件名、`doc_id` 和索引处理一致。
- Source/time context：当前实践、历史路径、产品、标准、监管或市场相关内容有来源限制和时间语境。
- Index/link impact：`docs/index.md`、`related_docs`、正文链接和 changed-file links 已检查。
- Lifecycle/status distinction：`quality_status`、生命周期状态、review decision、candidate/canonical status 和 BMad story status 不混用。
- Version governance：prompt/template/methodology/quality 行为变化已记录；无依据不 bump 版本。
- Workflow contract：story checkbox、Dev Agent Record、File List、Change Log、sprint status 和 on-complete 行为一致。
- Non-software boundary：未新增未授权 code、CLI、API、UI、database、deployment、CI、software test framework、runtime automation 或 executable tooling。

以下情况必须停止或请求 Maxwell 确认：

- canonical path、topic、`doc_id`、资产层级或 owner entry point 不确定。
- 任务类型混合且无法拆分。
- source/time context 不足但输出要声明 current、validated、accepted 或 maintained。
- index/promotion/candidate boundary 不清。
- lifecycle/status/quality claim 无证据。
- batch target set、excluded files、recovery 或 owner decision 不清。
- workflow/skill contract impact 不确定。
- 需要新增 schema 字段、批量改状态、批量迁移、删除、重命名、废弃或归档。
- 出现 executable tooling、automation、software test、CI、deployment 或 runtime scaffold 压力，但 Maxwell 未明确授权。

## 完成与维护触发点

后续 Agent 验证本文时，至少检查：

- 九类任务类型是否全部可定位。
- 正式概念文档路径和深度判定是否写清，且路径先于深度。
- 缺失输入是否有澄清、open question、明确假设、defer 或 stop 处理。
- `_bmad-output` 和 Agent discussion output 是否被明确排除在自动 canonical docs 之外。
- 路由输出、验证证据和停止条件是否足以阻止 hidden promotion、fake status、batch overreach 和 unauthorized tooling。
- 本文是否仍列入 `docs/index.md` 的 `methodology` 分组。
- 本文是否仍未创建 unauthorized frontmatter fields 或平行主规范。

已知维护触发点包括：

- frontmatter schema、topic/path/naming、candidate promotion 和 index synchronization rules 发生实质字段或 vocabulary 变更时，复核本文是否需要更正式的 intake decision record。
- review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更时，复核本文的验证证据是否应同步到固定模板。
- Story 4.1 已建立 revision/regeneration continuity policy；后续若该政策更新，复核 revision/regeneration/deprecation routing。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 更新后，复核 sidecar/legacy routing。
- related docs taxonomy、link maintenance policy、existing-doc reuse 或 network boundary policy 更新后，复核 related docs、successor 和 owner entry 的边界。
- Epic 6 batch runbook、batch review record 和 batch completion report 落地后，复核 batch-governance routing 与 batch execution steps 的分离。

本文不授权批量目录重组、批量 frontmatter normalization、软件自动化检查或第二套索引同步系统。此类动作必须先满足 `docs/governance/batch-readiness-checklist.md`，并由对应 story 或 Maxwell 明确批准。

## 参考资料

- [统一概念文档规范：新建、升级、审查与仓库集成](./document-generation-methodology.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](./concept-document-contract.md)
- [统一概念文档模板](./concept-document-template.md)
- [统一概念文档质量门禁](./concept-document-quality-gate.md)
- [固定概念文档生成 Prompt](./fixed-concept-generation-prompt.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](./governance-asset-boundary-policy.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](../governance/agent-behavior-constraints.md)
- [治理资产导航、索引与入口归属政策](../governance/governance-asset-navigation-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](../governance/lifecycle-states.md)
- [Prompt、模板与质量规则版本治理：规则演进、字段语义与渐进迁移](../governance/prompt-template-quality-version-governance.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](../governance/batch-readiness-checklist.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](../governance/revision-regeneration-continuity-policy.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](../governance/frontmatter-schema.md)
- [docs/index.md 同步与导航治理规则](../governance/index-synchronization-rules.md)
- [候选文档晋升 Checklist：canonical 入库、证据门禁与停止条件](../governance/candidate-promotion-checklist.md)
- [文档决策政策：accept、revise、regenerate、defer、reject 与 lifecycle 结果](../governance/document-decision-policy.md)
- [返工闭环示例：失败类型、修复路径、重生成边界与复审入口](../governance/rework-loop-examples.md)
- [related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling](../governance/related-docs-taxonomy.md)
- [跨文档链接维护政策：existence/path/topic/meaning checks、inbound/outbound review、one-way reason 与 boundary conflict handling](../governance/link-maintenance-policy.md)
- [既有文档复用流程：发现、对齐、复用决策与避免重复生成](../governance/existing-doc-reuse-procedure.md)
- [知识网络主题边界与退化防护政策](../governance/network-boundary-and-decay-prevention.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](../templates/review-record-template.md)
- [完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界](../templates/completion-report-template.md)
