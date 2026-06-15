---
doc_id: governance-lifecycle-states
title: 文档生命周期状态：草稿、审查、验证、废弃与归档转换规则
concept: document_lifecycle_states
topic: governance
depth_mode: standard
created_at: '2026-05-21T14:48:13+08:00'
updated_at: '2026-05-28T15:29:36+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/planning-artifacts/implementation-readiness-report-corrected-2026-05-20.md
  - _bmad-output/implementation-artifacts/0-1-agent-behavior-constraints.md
  - _bmad-output/implementation-artifacts/0-2-governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/revision-regeneration-continuity-policy.md
time_context: phase_4_epic_4_revision_regeneration_continuity_policy_2026_05_28
applicability: formal_document_lifecycle_and_transition_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/revision-regeneration-continuity-policy.md
open_questions:
  - Story 0.5 已固化质量状态基线后，是否需要在后续专门审查中调整本资产中的 quality_status 兼容映射？
  - Epic 2 定义正式 frontmatter schema 后，是否需要新增独立 lifecycle_state 字段？
  - Epic 2 Story 2.4 建立详细 index synchronization rules 后，是否需要为 deprecated 与 archived 资产建立单独索引分组？
---

# 文档生命周期状态：草稿、审查、验证、废弃与归档转换规则

## 资产角色、权威与适用范围

这份文档是 `Knowledge` 项目中正式文档生命周期状态与状态转换的治理资产。它定义 `docs/` 下正式知识资产、方法论资产、治理资产、模板、runbook、索引和经过批准的报告资产如何表达成熟度、审查状态、可复用程度、废弃状态和归档状态。

本资产补充 `docs/methodology/document-generation-methodology.md`，不替代概念文档的新建、升级、审查和仓库集成规则。概念文档仍必须服从主方法论和 `docs/methodology/concept-document-quality-gate.md`；本资产规定的是这些质量证据如何影响生命周期状态。

本资产也受 `docs/governance/agent-behavior-constraints.md` 与 `docs/methodology/governance-asset-boundary-policy.md` 约束。Agent 在执行生命周期变更前，必须先声明任务类型、资产层级、允许文件范围、验证证据和停止条件。

`_bmad-output/` 下的 PRD、Architecture、Epics、Story、readiness report、review output 和草稿默认是 workflow output。它们可以作为来源依据或实施上下文，但不自动获得 `docs/` 正式生命周期状态。只有在明确晋升到 `docs/`、补齐 frontmatter、通过适用门禁、处理索引并记录完成证据后，才进入本生命周期模型。

本资产当前 `quality_status: draft`。原因是它是首次建立生命周期规则的 Foundation 资产；Story 0.5 已固化质量状态基线，但本资产尚未经过专门晋升审查，也尚未由 Epic 2 固化独立 `lifecycle_state` frontmatter schema。这个保守状态不削弱本文作为 Story 0.3 输出规则的适用性；它表示后续治理故事仍需复核本资产自身状态。

## 生命周期状态词表

生命周期状态表达的是正式资产在仓库中的成熟度、可依赖程度和使用方式。它不是 BMad story 执行状态，也不等同于 `quality_status` 的全部含义。

| 状态 | 含义 | 允许使用方式 | 典型证据 |
| --- | --- | --- | --- |
| `draft` | 资产已经存在或已进入 `docs/` 候选位置，但结构、证据、链接、边界或质量检查尚未足以支持稳定复用 | 可阅读、可修订、可作为后续工作起点；不得当作已验证规则或最终答案 | frontmatter 基本完整；已记录缺口、来源和未解问题 |
| `reviewed` | 资产经过结构、frontmatter、角色/范围、链接/索引和质量证据审查，已知缺口被记录 | 可作为受控参考；仍需注意未解问题和待验证项 | 审查记录、无明显结构缺失、链接目标存在、已知风险列出 |
| `validated` | 资产没有当前 Hard Fail，仓库集成、来源/时间语境、链接/索引、未解问题和目标复用场景均已检查 | 可作为当前正式参考或复用入口 | 无 Hard Fail；索引和链接有效；source/time context 已复核；open questions 已审视 |
| `maintained_asset` | 稳定的治理、方法论、模板、索引或支持资产处于主动维护状态；在当前项目中作为 active/validated 的既有等价状态使用 | 可作为当前权威入口，前提是维护触发点和更新证据仍然有效 | 资产角色清晰；更新时间、来源、相关文档和维护触发点可追溯 |
| `deprecated` | 资产保留访问，但不应再作为主要参考；已有废弃原因、替代入口或后续处理说明 | 可用于历史比较、迁移依据或兼容说明；新工作应优先使用 successor | 废弃原因、替代/继任资产、受影响链接、索引处理和剩余访问预期 |
| `archived` | 资产仅为历史、证据或审计保留；正常导航和复用预期被限制 | 不作为当前工作依据，除非任务明确要求历史追溯 | 归档原因、检索路径、替代路径、索引可见性和 reactivation 条件 |

`maintained_asset` 是现有仓库中已经使用的 `quality_status` 值。对稳定治理、方法论和索引资产而言，它可以被解释为 active/validated equivalent，但这个解释必须依赖当前维护证据，而不是只依赖字段值本身。

`upgraded_v1` 是既有概念文档质量状态，表示一篇概念文档按当前质量门禁获得较强质量结果。它是质量证据，不自动等于完整生命周期状态。一个 `upgraded_v1` 概念文档通常具备进入 `reviewed` 或 `validated` 的候选资格，但仍需要检查仓库集成、链接、索引、来源/时间语境、未解问题和生命周期记录。

`published` 是旧项目上下文中出现过的历史用语。Story 0.3 后不把 `published` 作为新的必填生命周期状态。旧文档、旧规则或旧 completion report 中的 `published` 应映射到当前 active vocabulary：普通正式概念文档优先映射为 `validated` 候选；稳定治理、方法论、索引资产可映射为 `maintained_asset` 候选。映射前必须检查证据，不能只做文字替换。

## 生命周期、质量状态与 Workflow 状态的边界

`quality_status` 表达质量结果、维护状态或与既有质量门禁相关的状态信号。它可以包含 `draft`、`maintained_asset`、`upgraded_v1` 等既有值。Story 0.5 会进一步固化质量状态基线；在那之前，本资产只建立兼容解释，不静默重写历史字段。

生命周期状态表达仓库中的成熟度和允许使用方式。生命周期判断需要综合 frontmatter 完整性、正文结构、质量门禁、来源/时间语境、链接、索引、相关文档、未解问题和维护触发点。

BMad story 状态只表达执行跟踪，例如 `backlog`、`ready-for-dev`、`in-progress`、`review`、`done`。这些状态不得复制进正式文档 frontmatter，也不得当成正式资产生命周期状态。

质量门禁分数或 Hard Fail 结果是生命周期转换证据之一。生命周期不是分数本身：一个文档可以没有 Hard Fail，但仍因索引缺失、来源过期、open questions 未处理或 successor 未记录而不能晋升。

Epic 2 尚未定义正式 `lifecycle_state` frontmatter 字段。本 story 不向新资产或现有资产添加该字段。生命周期可在正文、审查记录、完成报告、废弃说明或后续 schema 中表达；是否加入 frontmatter 是 Epic 2 的 open question。

## 状态转换规则

状态转换必须由证据驱动。Agent 或维护者不得只因为“文档写完了”“看起来完整”或“已经加入索引”就宣称晋升。

### Workflow Output -> Formal Draft

`_bmad-output/` 中的候选文档、story 输出、review output 或规划片段只有在完成正式晋升动作后，才进入 `docs/` 生命周期。

转换到 `draft` 至少需要：

- 目标目录、topic、文件名和 `doc_id` 明确。
- YAML frontmatter 包含主规范要求的稳定字段。
- 正文说明资产角色、适用范围、来源依据和不适用范围。
- `source_basis`、`time_context`、`related_docs`、`open_questions` 初始可追溯。
- `docs/index.md` 影响已判断；新增正式资产时通常需要添加入口。
- 完成汇报确认该晋升没有引入未授权软件资产或自动化工具。

### `draft` -> `reviewed`

从 `draft` 晋升到 `reviewed` 需要审查证据，而不是只完成初稿。

最低要求是：

- frontmatter 必填字段存在，列表字段使用 YAML array，字段值与正文一致。
- 资产角色、权威、适用范围、与主方法论关系和导航处理清楚。
- Markdown 标题层级稳定，H1 与标题含义一致。
- 相关链接、`related_docs` 和 `docs/index.md` 入口已检查。
- 已知缺口、未解问题、来源限制和时间语境风险已记录。
- 对概念文档，已执行主方法论和质量门禁的 Hard Fail 初查。
- 对治理、方法论、索引、模板或 runbook 资产，已执行等价的角色/范围/权威/集成/workflow-contract 检查。

### `reviewed` -> `validated`

从 `reviewed` 晋升到 `validated` 或等价 active 状态，需要更强证据。

最低要求是：

- 当前没有 Hard Fail 或等价阻塞问题。
- 仓库集成完整：目标路径、文件名、frontmatter、H1、索引和导航处理一致。
- 来源与时间语境已经复核；涉及当前实践或外部事实时记录核对日期或来源限制。
- `related_docs` 和正文内链可解析，缺失目标已作为 open question 或 planned asset 记录。
- `open_questions` 已审视；未解决项不阻塞当前适用范围，或已明确限制使用方式。
- intended reuse 清楚：读者或 Agent 知道什么时候可以引用该资产，什么时候不能引用。
- completion report 记录生命周期前后状态、质量门禁结果、索引/链接影响、未解决风险和非软件边界确认。

`maintained_asset` 可作为治理、方法论、模板、索引等稳定资产的 active equivalent。使用它时还必须记录维护触发点，说明哪些后续 story、质量规则、prompt/template 版本或索引治理变化会要求复核。

### Active State -> `draft` 或 Re-review

已经 `validated` 或 `maintained_asset` 的资产，在出现实质性变化或新风险时，必须重新进入审查，必要时降回 `draft`。

触发条件包括：

- 正文主张、标题、topic、路径、`doc_id` 处理或适用范围发生实质变化。
- `prompt_version`、`template_version`、质量门禁、方法论主规范或 lifecycle policy 发生影响判断结果的变化。
- 来源过期、当前实践变化、时间语境失效或事实核查失败。
- 新增 Hard Fail、未解决风险、链接断裂、索引错误或 related docs 冲突。
- 文档被发现与 Agent 行为约束、方法论资产边界或 BMad workflow contract 冲突。

重新审查时必须记录变更原因、受影响资产、质量证据、索引/链接影响和是否需要 Maxwell 确认。

### Active State -> `deprecated`

正式资产进入 `deprecated` 时，不能只改一个状态词。废弃记录必须说明：

- 废弃原因。
- successor、replacement 或替代入口；如暂无替代，必须说明剩余风险。
- 受影响入链、出链、related docs 和索引入口。
- `docs/index.md` 处理：保留、移动、降级显示、标注废弃或移除导航入口。
- 剩余访问预期：可否用于历史追溯、迁移比较、旧工作复查或审计证据。
- `updated_at`、`source_basis`、`time_context`、`quality_status`、`related_docs` 和 `open_questions` 的处理方式。

本项目禁止静默删除正式资产。删除或移动正式资产必须先经过废弃/迁移规则，并记录替代入口和链接影响。

### `deprecated` -> `archived`

废弃资产进一步进入 `archived` 时，必须确认它不再承担正常导航或复用职责。

最低要求是：

- 已记录归档原因和保留价值，例如历史证据、旧决策复查或迁移审计。
- successor/replacement 可定位，或明确说明没有替代但仍保留历史访问。
- `docs/index.md` 可见性已经处理：保留在普通索引、移动到报告/归档入口、或从普通导航移除并在 successor 中反向说明。
- 入链和出链影响已检查，不留下误导性当前入口。
- remaining access expectations 清楚：谁可以在什么场景引用归档资产。

### `deprecated` / `archived` -> Active State

从 `deprecated` 或 `archived` 重新激活是例外流程，必须获得 Maxwell 明确确认。

重新激活至少需要：

- 说明为什么原废弃或归档原因已经失效。
- 重新执行 frontmatter、Markdown、source/time context、质量门禁、索引、链接和 open questions 检查。
- 记录替代资产或 successor 的影响，避免两个入口产生冲突。
- 更新 completion report，说明 lifecycle before/after、质量证据和 remaining risks。

## 废弃与归档处理

废弃和归档是治理动作，不是清理文件。它们的目标是防止用户或 Agent 误把旧资产当作当前主入口，同时保留必要的历史、证据和迁移路径。

废弃或归档记录必须覆盖：

- `reason`：为什么废弃或归档。
- `replacement` / `successor`：当前应使用哪个资产；如果没有，说明缺口。
- `affected_links`：哪些 inbound/outbound links、related docs、索引入口或方法论引用受影响。
- `index_treatment`：`docs/index.md` 是否保留入口、改标题标注、移到归档入口、或移除普通导航。
- `access_expectation`：读者和 Agent 还能在什么场景访问它。
- `open_questions`：是否还存在待迁移、待补写、待合并或待确认事项。

如果某项暂时无法确认，必须写成未解问题或停止条件，不能假定无影响。

## 元数据一致性要求

生命周期相关变更必须与 frontmatter 和正文一致。不得只改正文状态，也不得只改 frontmatter 字段而不更新证据。

每次生命周期、质量状态、废弃、归档或重新激活变更时，至少检查：

- `updated_at`：是否有实质内容或治理状态变化，需要更新。
- `source_basis`：新增规则、审查证据、替代资产或来源是否需要加入。
- `time_context`：当前日期、实践状态、项目阶段或版本语境是否变化。
- `quality_status`：现有值是否仍能表达质量结果或维护状态。
- `prompt_version`：概念文档或生成规则是否受 prompt 版本影响。
- `template_version`：结构模板或治理资产模板是否变化。
- `related_docs`：successor、replacement、相邻方法论或治理资产是否需要更新。
- `open_questions`：已解决问题是否清理，新风险是否加入。

新增、移动、重命名、废弃、归档、改标题、改 topic 或改变主要导航定位时，必须检查 `docs/index.md`。仅正文微调通常可以不改索引，但完成汇报必须说明不适用理由。

## 验证证据与完成汇报

生命周期治理工作的完成汇报必须具体到证据位置，不能只写“已检查”。

最低完成汇报字段包括：

- 任务类型和资产层级。
- 变更文件，区分新增、修改、删除。
- 生命周期状态 before/after；如果尚无正式 `lifecycle_state` 字段，则说明状态如何在正文或记录中表达。
- `quality_status` before/after 与兼容解释。
- 质量门禁或等价治理门禁结果：Hard Fail、角色/范围/权威、workflow-contract、索引/链接证据。
- `docs/index.md` 影响：已更新、不适用或需后续处理。
- 内部链接和 `related_docs` 检查结果。
- `source_basis`、`time_context`、`prompt_version`、`template_version`、`open_questions` 的处理。
- 未解决风险、后续 story 依赖和 Maxwell 确认项。
- 非软件边界确认：未新增未授权 package manifest、source tree、software tests、build config、runtime automation、CLI/API/UI/database/deployment/CI artifact。

验证结果必须使用 `通过`、`未通过`、`未验证` 或 `不适用`。存在 Hard Fail、关键链接断裂、索引不一致、来源不可验证、未授权范围变更或 workflow contract 破坏时，不得宣称生命周期晋升完成。

## 维护与迁移入口

本资产是 Foundation 生命周期基线，后续治理故事可能要求复核。

已知维护触发点包括：

- Story 0.4 建立 prompt、template 与 quality rule version governance 后，复核 `prompt_version`、`template_version` 与 lifecycle transition 的关系。
- Story 0.5 固化质量门禁 Hard Fail、评分维度与质量状态基线后，复核 `quality_status` 兼容映射和 `validated` 证据要求。
- `docs/governance/governance-asset-navigation-policy.md` 已建立导航入口归属基线；Epic 2 Story 2.4 定义详细 index synchronization rules 后，复核 `deprecated`、`archived` 和 index treatment 规则。
- `docs/governance/batch-readiness-checklist.md` 已定义批量治理 readiness；批量迁移、批量废弃和批量归档必须先按该政策完成 target set、冲突暴露、停止条件和恢复策略判定。
- Epic 2 定义 frontmatter schema、`doc_id`、topic/path/name policy 和 candidate promotion rules 后，决定是否新增正式 `lifecycle_state` 字段。
- Epic 3 review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更时，复核 lifecycle evidence 的报告格式。
- Story 4.1 已建立修订、重生成和版本连续性策略；后续若该政策更新，复核 re-review、reactivation、successor 和 reference validity 记录。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 更新后，复核 lifecycle evidence 与 remaining access 规则。

本资产不授权批量迁移、批量状态改写、批量删除、软件自动化或 schema 大规模重构。此类动作必须先满足 `docs/governance/batch-readiness-checklist.md` 的 readiness 判定，并由后续治理 story 或 Maxwell 明确批准。
