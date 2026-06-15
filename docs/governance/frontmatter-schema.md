---
doc_id: governance-frontmatter-schema
title: Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线
concept: frontmatter_schema_and_doc_id_identity_rules
topic: governance
depth_mode: standard
created_at: '2026-05-25T18:00:40+08:00'
updated_at: '2026-06-15T17:21:39+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/2-1-frontmatter-schema-and-doc-id.md
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_key_draft_review_2026_06_15
applicability: formal_docs_frontmatter_schema_and_doc_id_identity_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
open_questions:
  - 后续是否需要授权新增 lifecycle_state、schema_version、quality_gate_version、methodology_version 或 migration_status 等独立 frontmatter 字段？
  - rename/migration/split/merge/successor 规则已建立后，是否需要把 doc_id 例外记录抽成固定决策模板？
  - related docs taxonomy 与 link maintenance policy 已建立；后续是否需要进一步细分 related_docs、source references、owner entry、successor link 与 supporting link 的维护记录？
---

# Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中正式 `docs/` 资产的 canonical frontmatter schema 与 `doc_id` 身份治理资产。它回答的是：正式资产进入、更新、晋升或复核时，frontmatter 至少要包含哪些字段，这些字段如何保持与正文和生命周期一致，以及 `doc_id` 何时必须保持稳定。

本文适用于正式 `docs/` 资产，包括：

- `docs/{topic}/` 下的正式概念文档。
- `docs/methodology/` 下的方法论、模板、质量门禁、fixed prompt 和支撑政策。
- `docs/governance/` 下的治理资产。
- 未来明确批准进入 `docs/templates/`、`docs/runbooks/` 或报告/归档路径的正式资产。

正式概念文档必须完整满足本文的 baseline required fields。治理、方法论、模板、索引、runbook 或报告资产不是普通概念文档，但必须执行等价检查：身份、标题、主题、深度或结构模式、创建/更新时间、来源依据、时间语境、适用范围、版本来源、质量状态、相关文档和未解问题必须可追溯、可审查、与正文一致。

本文补充 `docs/methodology/document-generation-methodology.md`、`docs/methodology/concept-document-contract.md`、`docs/methodology/concept-document-quality-gate.md`、`docs/methodology/source-discipline-and-real-world-anchor-policy.md`、`docs/governance/lifecycle-states.md`、`docs/governance/prompt-template-quality-version-governance.md`、`docs/governance/governance-asset-navigation-policy.md` 和 `docs/governance/batch-readiness-checklist.md`。主方法论仍负责概念文档的新建、升级、审查和仓库集成流程；质量门禁仍负责 Hard Fail、评分和状态声明判断；本文只负责 schema baseline 与身份规则。

本文不替代已落地的 Epic 2、Epic 3、Epic 4、Epic 5 专门资产，也不替代未来 Epic 6 的 batch governance 资产：

- Story 2.2 负责 topic、文件命名与路径归属策略。
- Story 2.3 负责候选文档晋升流程。
- `docs/governance/index-synchronization-rules.md` 负责详细 `docs/index.md` 同步规则。
- Story 2.5 负责 rename、migration、split、merge、deprecation、successor 和 link-impact 决策。
- Story 2.6 负责重复概念与同主题共存政策。
- Epic 3 负责 review record、document decision、rework loop 和 completion report 模板。
- Story 4.1 负责 revision/regeneration continuity、update mode taxonomy 和 Continuity Record。
- Story 4.2 负责 sidecar boundary policy。
- `docs/governance/legacy-migration-guide.md` 负责 legacy migration guide。
- Story 5.1 负责 related docs taxonomy；`docs/governance/link-maintenance-policy.md` 负责 link maintenance；`docs/governance/reusable-model-entry-points.md` 负责 reusable model entry points；`docs/governance/existing-doc-reuse-procedure.md` 负责 existing-doc reuse procedure；`docs/governance/network-boundary-and-decay-prevention.md` 负责 network boundary / decay prevention。
- Epic 6 负责 batch governance runbook、batch review record 和 batch completion report。

本文的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/frontmatter-schema.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：frontmatter baseline、`doc_id` 身份规则、owner/index entry、相关治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

## Frontmatter schema baseline

正式 `docs/` 资产必须使用 YAML frontmatter + Markdown body。Frontmatter 是治理接口，不是装饰性元数据；正文是解释、规则、模型、流程或证据主体。Reviewer 必须能同时检查 frontmatter 和正文，不能只看其中一边。

当前 baseline required fields 是：

```yaml
doc_id: stable_document_identity
title: Human-readable title
concept: stable_snake_case_concept_key
topic: topic_or_asset_group
depth_mode: standard_or_deep_or_equivalent_asset_mode
created_at: 'YYYY-MM-DDTHH:MM:SS+08:00'
updated_at: 'YYYY-MM-DDTHH:MM:SS+08:00'
source_basis:
  - source_or_project_rule
time_context: explicit_time_or_project_phase_context
applicability: where_this_asset_applies
prompt_version: prompt_rule_source_or_not_applicable
template_version: structure_or_asset_template_version
quality_status: draft_or_reviewed_or_validated_or_maintained_asset_or_other_currently_supported_status
related_docs: []
open_questions:
  - unresolved_question_or_future_story_dependency
```

Concept documents must treat all fields above as required. Governance, methodology, template, index, runbook and report assets must use the same baseline unless a later schema decision explicitly defines an asset-specific replacement. If a field is not naturally meaningful for an asset type, the value must still be explicit, for example `prompt_version: not_applicable`, rather than silently deleting the field.

Field naming and value-shape rules:

- Frontmatter keys use `snake_case`.
- `concept` uses stable `snake_case` and must not drift with title wording.
- Topic directories and filenames use `kebab-case`; detailed topic/path ownership belongs to Story 2.2.
- `source_basis`, `related_docs` and `open_questions` use YAML arrays, even when empty or single-item.
- Dates use ISO format with timezone when available.
- Field values must be consistent with body content, path context, index treatment, source/time claims, quality status and lifecycle semantics.

Later schema extensions must be introduced by explicit governance work. Individual documents must not invent one-off global fields to solve local uncertainty. In particular, this story does not authorize `schema_version`, `lifecycle_state`, `owner_entry_point`, `navigation_treatment`, `quality_gate_version`, `methodology_version`, `migration_status` or `index_policy_version` as new frontmatter fields.

## 字段语义与格式要求

| 字段 | 最小语义 | 格式与一致性要求 |
| --- | --- | --- |
| `doc_id` | 稳定文档身份。用于追踪资产连续性，而不是复述文件名。 | 必须稳定、唯一、可长期引用；不得随标题、路径或 topic 变化自动改写。 |
| `title` | 人类可读标题。 | 应与 H1 含义一致；标题变化不自动改变 `doc_id`。 |
| `concept` | 稳定概念或资产键。 | 使用 `snake_case`；表达核心对象，不写整句标题。 |
| `topic` | 当前主题或资产分组。 | 与目录和 `docs/index.md` 分组保持一致；具体归属细则由 Story 2.2 收束。 |
| `depth_mode` | 概念文档的 `standard`/`deep` 深度，或治理资产的等价结构模式。 | 概念文档必须使用当前主方法论允许的深度值；治理资产通常使用 `standard`，除非后续规则定义更细模式。 |
| `created_at` | 资产首次创建为正式文件或正式候选文件的时间。 | 不得因重命名、移动、标题调整或轻量修订而改写成新创建时间。 |
| `updated_at` | 最近一次实质内容、规则、状态或元数据变更时间。 | 只在实质变化时更新；不得暗示未发生的 review、migration 或 regeneration。 |
| `source_basis` | 支撑本文规则、事实、来源纪律、用户决策或项目上下文的依据。 | 必须是 YAML array；可包含项目内正式资产、规划/story 上下文或外部来源；不得凭空发明。 |
| `time_context` | 本文声明的时间基线、项目阶段、核对日期或 currentness 限制。 | 必须支持正文的当前实践、历史路径、来源限制和项目阶段说法。 |
| `applicability` | 资产适用的任务、读者、范围或使用场景。 | 必须与正文的适用/不适用范围一致；不得泛化到未授权资产层级。 |
| `prompt_version` | 生成、升级或审查时实际受哪个 prompt 规则集约束。 | 治理资产可用 `not_applicable`；不得只因新 prompt 存在就改旧资产字段。 |
| `template_version` | 实际采用的结构模板、资产模式或规则集。 | 必须反映实际结构合同，例如 `concept_doc_v2`、`governance_asset_v1` 或 `index_v1`。 |
| `quality_status` | 当前质量、维护或审查状态信号。 | 必须反映证据，不得把 draft/candidate/Hard Fail 资产标成 ready、reviewed、validated、`upgraded_v1`、`maintained_asset` 或等价通过状态。 |
| `related_docs` | 与本文有正式关系的仓库内资产。 | 必须是 YAML array；目标应是存在的正式资产；缺失或计划中的目标写入 `open_questions` 或后续 story 依赖。 |
| `open_questions` | 未解决、待核查、待后续 story 或待 Maxwell 决策的问题。 | 必须是 YAML array；不得为了看起来完整而隐藏 schema、来源、生命周期、身份或晋升风险。 |

`source_basis` 可以引用 `_bmad-output/` 规划或 story 上下文作为实施依据，但这不会把 `_bmad-output/` 文件变成正式 `docs/` 资产。`related_docs` 默认应指向正式 `docs/` 资产；如果相关目标尚未存在，必须作为 open question 或 future-story dependency 暴露。

## `doc_id` 身份规则

`doc_id` 是正式文档身份。它不由标题、文件名、路径、topic、H1、索引标题或翻译标题派生。

当底层资产身份连续时，以下变化不要求、也不得自动触发 `doc_id` 变化：

- 标题措辞变化。
- H1 变得更准确或更短。
- 文件名从旧 slug 改成新 slug。
- 文档从一个 topic 目录移动到另一个 topic 目录。
- `docs/index.md` 分组、排序或导航标题调整。
- 正文局部修订、来源补强或状态复核。

无效身份处理包括：

- 不得为了匹配新文件名而静默修改 `doc_id`。
- 不得为了匹配 retitled H1、翻译标题或营销化标题而修改 `doc_id`。
- 不得在移动 topic 或目录时把 `doc_id` 当成路径缓存一起重写。
- 不得通过新建同内容文件绕过旧 `doc_id` 的历史、链接、质量状态或 open questions。

Split、merge 或 replacement 可能需要新身份，但本资产只定义高层原则。详细 decision record、successor handling、link migration、lifecycle treatment 和 index impact 由 Story 2.5 负责。任何身份例外至少要记录：

- 例外理由。
- 受影响资产和原 `doc_id`。
- 新身份或候选身份。
- successor/replacement/merged-from/split-from 关系，如果已知。
- 尚未解决的问题和后续确认入口。

如果 reviewer 无法判断某次变化是连续修订还是新资产身份，必须先记录 identity ambiguity。该资产不得在身份未决时宣称完成迁移、通过审查或稳定发布。

## 缺失或不完整 frontmatter 处理

缺失 YAML frontmatter、缺失 required fields、数组字段不是 YAML array、字段与正文冲突、`doc_id` 缺失或身份不明，都会阻塞正式 `docs/` 资产晋升。

唯一允许的非阻塞路径是显式标记为 candidate/incomplete，并且不宣称 ready、accepted、reviewed、validated、`upgraded_v1`、`maintained_asset` 或等价通过状态。这个标记可以写在正文、`open_questions`、review evidence、completion evidence 或 workflow record 中，但必须让 reviewer 看出缺口是什么、为什么暂时不能补、后续由谁或哪个 story 处理。

禁止静默发明以下信息：

- `doc_id`、topic、path、title 或 concept。
- `created_at` 或 `updated_at`。
- `source_basis`、外部来源、核对日期或 currentness 依据。
- `quality_status`、review conclusion、lifecycle 语义或已迁移状态。
- `related_docs`、successor、replacement 或 open questions。

对话输出、`_bmad-output/` story output、规划产物、临时 Markdown 草稿和 reviewer 片段都不是正式 `docs/` 资产。它们只有在完成 target path、frontmatter、source/time context、质量门禁、索引和 owner decision 检查后，才可以晋升为正式资产。

## 一致性检查

Reviewer 检查正式资产时，至少执行以下一致性判断：

| 检查面 | 必须成立 |
| --- | --- |
| 日期一致性 | `created_at` 和 `updated_at` 与资产历史一致，不暗示未执行的 review、migration、regeneration 或 validation。 |
| 来源一致性 | `source_basis` 能支撑正文中的项目规则、事实、推断或来源限制；外部 current-practice claim 有相称来源和时间语境。 |
| 时间一致性 | `time_context` 与正文核对日期、项目阶段、历史/当前实践和 open questions 一致。 |
| 状态一致性 | `quality_status` 与实际证据和生命周期语义一致；Hard Fail、blocking open question 或身份未决时不得声明通过状态。 |
| 路径/索引一致性 | `topic`、路径、H1/title 和 `docs/index.md` 入口不互相误导；详细同步规则由 `docs/governance/index-synchronization-rules.md` 负责。 |
| 相关文档一致性 | `related_docs` 目标存在且是正式资产；缺失、计划中或未来 story 拥有的目标必须记录为 open question。 |
| 未解问题一致性 | `open_questions` 暴露 schema、source、lifecycle、related-doc、identity 或 promotion 风险，而不是隐藏风险。 |
| 生命周期一致性 | 与 `docs/governance/lifecycle-states.md` 的状态语义兼容；除非后续明确授权，不新增 `lifecycle_state` frontmatter 字段。 |
| 版本一致性 | `prompt_version` 和 `template_version` 反映实际使用规则；不新增 `quality_gate_version`、`methodology_version`、`version_history` 或 `migration_status` 绕过版本治理。 |
| 非软件边界 | Frontmatter schema 检查是文档治理工作，不要求 runtime code、package manifest、tests、lint/scoring tool、CLI、API、UI、database、deployment 或 CI。 |

发现 unauthorized frontmatter fields 时，reviewer 必须判断它们是局部误加、未来 schema 候选，还是试图绕过治理决策。局部误加应在目标资产中移除或改写为正文/completion evidence；未来候选字段应记录为 open question 或后续 story 依赖。

## 生成、晋升与审查检查表

生成、晋升或审查正式 `docs/` 资产前，至少检查：

1. 任务类型和资产层级已经明确：概念文档、方法论、治理资产、模板、索引、runbook、报告或 workflow output。
2. 目标是否真的是正式 `docs/` 资产；如果来自 `_bmad-output/` 或对话草稿，是否已经完成晋升判断。
3. YAML frontmatter 是否存在，并包含 baseline required fields。
4. `source_basis`、`related_docs`、`open_questions` 是否都是 YAML arrays。
5. `doc_id` 是否稳定、唯一、未被标题/路径/topic/H1 变化牵着走。
6. `title` 与 H1 含义是否一致。
7. `concept`、`topic`、路径和索引分组是否不互相冲突。
8. `created_at`、`updated_at`、`time_context` 是否与正文和资产历史一致。
9. `quality_status` 是否有证据支撑，且没有遮蔽 Hard Fail、blocking open questions 或 candidate/incomplete 状态。
10. `related_docs` 和正文链接是否指向存在的正式资产；不存在时是否写入 open questions。
11. 是否没有新增未授权全局 frontmatter 字段。
12. 是否按 `docs/governance/governance-asset-navigation-policy.md` 处理 owner entry point、navigation treatment 和 `docs/index.md` 影响。
13. 是否没有执行批量 frontmatter normalization、批量迁移、批量状态改写或批量索引重构。
14. 是否没有引入 package manifest、runtime code、software tests、build config、CI/deployment、CLI/API/UI/database 或 executable validation tooling。

如果任一检查失败，资产不能被晋升为正式 ready/accepted/reviewed/validated 状态。可以继续作为 draft/candidate，但缺口和后续处理必须可见。

## 维护触发点

以下变化要求复核本文：

- Topic、文件命名和路径归属策略更新。
- 候选文档晋升流程更新。
- `docs/index.md` 同步规则更新。
- Rename、migration、split、merge、deprecation、successor 和 link-impact 决策规则更新。
- Story 2.6 已建立 duplicate concept 与 same-topic coexistence policy；后续若该政策改变身份/共存边界，复核本文。
- Epic 3 review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更。
- Story 4.1 已建立 revision/regeneration continuity policy；后续若该政策更新，复核 `doc_id` identity、metadata review 和 unauthorized field 边界。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 更新后，复核相关 frontmatter/open question 边界。
- Epic 5 related docs taxonomy、link maintenance policy、reusable model entry points 或后续 reuse/network governance 更新。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、lint/scoring automation 或批量 normalization。

本文当前不执行批量 normalization，不创建 machine-readable validator，不重新定义 Story 2.2-2.6 或 Epic 5 的专门资产，不改变主方法论、模板、quality gate 或 fixed prompt 语义，也不 bump prompt/template/quality 版本。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](../methodology/concept-document-contract.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [概念文档样例目录：合格、不合格与质量门禁证据](../methodology/concept-document-example-catalog.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
- [统一概念文档模板](../methodology/concept-document-template.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [固定概念文档生成 Prompt](../methodology/fixed-concept-generation-prompt.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](../methodology/governance-asset-boundary-policy.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](./agent-behavior-constraints.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [Prompt、模板与质量规则版本治理：规则演进、字段语义与渐进迁移](./prompt-template-quality-version-governance.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
