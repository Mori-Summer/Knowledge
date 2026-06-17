---
doc_id: governance-legacy-migration-guide
title: 旧文档渐进迁移与兼容策略：缺口分类、保留规则、批量风险判定与剩余缺口处理
concept: legacy_migration_guide
topic: governance
depth_mode: standard
created_at: '2026-05-28T15:29:36+08:00'
updated_at: '2026-06-17T10:48:01+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/4-3-legacy-migration-guide.md
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_epic_4_continuity_migration_governance_review_2026_06_17
applicability: formal_docs_legacy_migration_and_compatibility_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
open_questions:
  - related-doc taxonomy、link maintenance、reusable model entry points、existing-doc reuse 或 network boundary policy 更新后，是否需要细分 legacy successor link、supporting link、related_docs、owner entry 和 inbound/outbound evidence？
  - Epic 6 建立 batch governance runbook、batch change review record 或 batch completion report 后，是否需要把 batch legacy migration assessment 转入专门 runbook records？
  - Maxwell 后续明确授权 actual legacy inventory 或 named old-doc migration 后，是否需要为本政策补充真实 migration examples？
---

# 旧文档渐进迁移与兼容策略：缺口分类、保留规则、批量风险判定与剩余缺口处理

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 legacy / old-document migration assessment、old-document compatibility、gap classification、high-value old-content preservation、batch migration risk criteria 和 remaining-gap handling 的 canonical governance asset。它回答的是：当正式资产、候选稿、旧报告、支持材料或历史记录早于当前 prompt、template、frontmatter、quality gate、lifecycle、index/link 或 sidecar governance baseline 时，Agent 或 reviewer 应如何判断是否需要迁移、如何记录缺口、如何保护旧内容价值、何时停止进入 batch readiness，以及如何避免把未迁移资产伪装成已经验证的当前资产。

本文适用于：

- `docs/{topic}/` 下已索引或应被索引的正式知识资产。
- `docs/methodology/`、`docs/governance/` 和 `docs/templates/` 下的方法论、治理和模板资产。
- 旧候选稿、workflow outputs、review evidence、completion evidence、formal reports、archive entries、legacy sidecars、historical support material、old notes 和 supplemental artifacts。
- 需要判断 old-doc compatibility、frontmatter/quality/version drift、source/currentness drift、index/link impact、lifecycle/status impact、sidecar/support-material impact 或 batch migration risk 的场景。

本文的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/legacy-migration-guide.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：legacy asset classes、migration gap taxonomy、old-content preservation、migration/compatibility outcomes、batch-risk criteria、sample-check requirements、Legacy Migration Assessment Record、remaining-gap wording、相邻治理依赖和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/legacy-migration-guide.md
- target title: 旧文档渐进迁移与兼容策略：缺口分类、保留规则、批量风险判定与剩余缺口处理
- target path: ./governance/legacy-migration-guide.md
- target section: docs/index.md ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: Story 4.3 explicitly authorizes the canonical legacy migration and old-document compatibility governance asset
- validation result: target exists and relative link resolves
- unresolved risk: Epic 5 link/reuse/network integrations are referenced as current adjacent assets; Epic 6 batch integrations remain open questions
```

## 职责边界与非目标

本文定义 legacy migration assessment、gap taxonomy、old-content preservation、migration/compatibility outcomes、batch-risk criteria、sample-check requirements、Legacy Migration Assessment Record 和 remaining-gap wording。它不执行任何实际旧文档迁移。

本文不实现以下范围：

- actual old-doc inventory、旧文档升级、旧文档重生成、旧路径清理、旧 report 归档、sidecar extraction、batch migration、batch link/index repair、batch frontmatter normalization、batch status conversion 或 source/currentness sweep。
- candidate promotion、duplicate/coexistence、rename/move/replacement/successor/deprecation/archive、revision/regeneration、sidecar boundary、index synchronization、source discipline、review decision 或 completion reporting 的完整模型；这些由相邻资产负责。
- related-doc taxonomy、general link maintenance、existing-doc reuse、reusable model entry points 或 knowledge-network decay assets。
- Epic 6 的 batch governance runbook、batch change review record、batch completion report template 或 batch execution records。
- runtime code、package manifest、source tree、software tests、CLI/API/UI/database/deployment/CI、machine-readable schema、validator、generator、link scanner、index generator、migration script、completion-report generator、decision generator、lint/scoring tool 或 executable automation。

本文也不创建新的全局 frontmatter 字段。不得为了表达 legacy migration、compatibility、owner、index treatment、schema version、decision status、successor 或 migration batch，在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`migration_status`、`compatibility_status`、`legacy_status`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`quality_gate_version`、`methodology_version`、`decision_status`、`successor`、`migrated_from`、`migrated_to`、`migration_batch` 或类似字段。相关证据应写在正文、review evidence、completion evidence、story Dev Agent Record、Revision / Regeneration Continuity Record、Migration Decision Record、future runbook record 或明确命名的 migration note 中。

新规则存在本身不要求旧文档立即迁移。迁移只在风险、使用场景、review finding、Maxwell 授权、明确 story 或相邻治理资产触发时发生；没有触发时，可以记录兼容结论、缺口或后续 owner，而不是批量改写。

## Legacy Asset Classes

`legacy` 或 `old document` 不是“旧就是坏”。在本项目中，它表示目标资产的生成、结构、metadata、source/time、quality gate、link/index、lifecycle 或 support-material 语境早于当前治理基线，或者使用了当前规则无法直接解释的历史约定。

常见 legacy classes 如下：

| Legacy class | Includes | Migration assessment focus | Compatibility note |
| --- | --- | --- | --- |
| `indexed_formal_doc` | 已在 `docs/index.md` 出现的正式文档 | frontmatter、body structure、quality_status、related_docs、index/link 和 source/time consistency | 可继续作为当前入口，前提是不伪造新规则、已验证状态或当前实践 |
| `pre_schema_or_partial_frontmatter_doc` | 缺少当前 baseline frontmatter、使用旧字段或数组形态不稳定的正式资产 | required fields、YAML arrays、stable `doc_id`、created/updated dates、source_basis、time_context 和 quality evidence | 可先 record gap，不必自动 normalize |
| `old_methodology_or_governance_asset` | 旧方法论、旧治理规则、旧模板、fixed prompt 或 playbook | role/authority、source hierarchy、template/version claim、lifecycle vocabulary 和 adjacent-asset boundary | 不得把历史 wording 当成当前 canonical rule |
| `formal_report_or_archive_entry` | normalization/upgrade reports、archive records、review/completion evidence | evidence role、current truth boundary、remaining access、index visibility 和 lifecycle impact | 可保留为 historical / audit evidence，不能替代当前主文档 |
| `legacy_sidecar_or_support_material` | old sidecars、notes、source dossiers、historical support material、reviewer notes | main-document relationship、trust boundary、source/time、staleness、link impact 和 sidecar/support impact | 按 sidecar boundary policy 判断，不把 support material 当 primary truth |
| `candidate_or_workflow_output` | `_bmad-output/` 产物、候选稿、未晋升草案、story evidence | 是否为正式资产、promotion readiness、source limitations、quality gaps 和 authorization | 不自动进入 `docs/`；晋升另走 candidate promotion |
| `deprecated_or_archived_asset` | 已废弃、归档或只作为历史证据保留的资产 | successor/replacement、remaining access、index treatment、lifecycle/status wording 和 open questions | 不得把 archived / deprecated 资产重新写成 current entry，除非另有 reactivation evidence |

Migration assessment 的目标是判断风险和处理路径，不是证明旧文档“必须升级”。如果旧资产声明的时间语境、状态、来源和入口都诚实，并且当前使用不会误导读者或 Agent，它可以保持 `no_action_currently_compatible` 或 `record_gap_only`。

## Migration Assessment Gap Taxonomy

Reviewer 或 Agent 执行 migration assessment 时，至少检查以下 gap classes。一个目标资产可能同时命中多类缺口；记录缺口不等于已经修复缺口。

### Frontmatter / Metadata Gaps

检查是否存在：

- missing required fields：缺少 `doc_id`、`title`、`concept`、`topic`、`depth_mode`、`created_at`、`updated_at`、`source_basis`、`time_context`、`applicability`、`prompt_version`、`template_version`、`quality_status`、`related_docs` 或 `open_questions`。
- YAML arrays：`source_basis`、`related_docs` 或 `open_questions` 不是 YAML array，或把多值字段压成字符串。
- `doc_id` identity ambiguity：标题、路径、正文对象或索引入口变化后，仍不清楚是否保留同一身份。
- `created_at` / `updated_at` inconsistency：创建时间被重写、更新时间与实质变更不匹配，或观察/评估被伪造成更新。
- `source_basis` gap：正文 claim、治理规则、当前实践或历史判断没有可追溯来源。
- `time_context` gap：当前/历史/阶段性语境缺失，导致旧实践被误读为当前推荐。
- `prompt_version` / `template_version` mismatch：字段声称使用了新规则，但正文或生成证据仍是旧规则。
- `quality_status` evidence gap：`draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated` 或 `archived` 与审查证据不匹配。
- `related_docs` / `open_questions` gap：相邻资产、未决缺口或 future owner work 没有进入相关字段或正文证据。

### Structural Gaps

检查是否缺少：

- 问题定义、scope/non-scope、对象边界、核心结构、机制链、tradeoff、failure modes、真实场景、例子、反例、自测题、验证入口、迁移入口或维护触发点。
- 对治理资产而言，是否缺少 asset role、authority、applicable scope、owner entry point、navigation treatment、non-goals、specialized record、validation checklist 或 adjacent-asset boundaries。
- 对模板或 review/completion evidence 而言，是否缺少 required evidence、blocking semantics、decision owner、unverified/not applicable register 或 future-story boundary。

### Currentness / Outdated-Claim Gaps

检查是否存在：

- 历史实践被写成当前实践，或 legacy/deprecated/superseded/context-limited 材料没有标注时间语境。
- 过时工具、标准、产品行为、组织实践或项目规则缺少 checked date、source date 或 phase context。
- 推断和事实混写，导致读者无法区分已核查事实、基于来源的归纳、作者推断和待核查问题。
- `source_basis` 不支持正文 claim，或 source limitation 没有进入正文/open questions。
- 旧例子仍有解释价值但没有说明为什么旧、当前替代是什么、何时仍可用于 compatibility context。

### Weak Example / Missing Validation Gaps

检查是否存在：

- 例子只装饰正文，不能说明结构、边界、tradeoff、失败模式或迁移判断。
- 没有反例、boundary test、diagnostic question、self-test 或 transfer entry。
- 缺少 downstream reuse / migration usage，导致未来 Agent 不知道何时调用该资产。
- review/completion evidence 只写“已检查”，没有指向具体文件、heading、frontmatter、link、status 或 decision evidence。

### Repository Integration Gaps

检查是否存在：

- path/topic/name mismatch：文件路径、topic、title、concept 或 doc_id 语义不一致。
- index entry 缺失、陈旧、错分组、重复或仍指向旧路径/旧标题。
- body links、fragment anchors、related_docs 或 reference paths 断裂。
- discoverable inbound reference risk：本地可发现的旧路径、旧标题、旧 doc_id 或 old heading reference 会被破坏。
- duplicate/coexistence risk：同主题资产并存但没有区分边界、successor/replacement 或 co-existence rationale。
- successor/replacement ambiguity：旧资产是否仍为入口、是否有替代资产、是否需要 deprecate/archive 不清楚。

### Lifecycle / Quality Gaps

检查是否存在：

- `quality_status` 暗示不存在的审查、迁移、验证或维护证据。
- `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` 混用，或用旧 canonical labels 替代当前治理词汇。
- BMad story status（如 `ready-for-dev`、`in-progress`、`review`、`done`）被误当作正式资产 `quality_status` 或 lifecycle state。
- completion wording 把 assessment、recording、index update 或 limited revision 写成 full migration complete。

### Sidecar / Support-Material Gaps

检查是否存在：

- legacy notes、historical support material、old sidecars、review reports 或 completion records 与主文档 current truth、source/time、links、status 或 lifecycle claim 冲突。
- support material 变成 hidden primary truth，导致主文档离开 sidecar 后无法独立理解。
- stale sidecar 仍被当成 current supporting evidence，或 historical material 缺少 trust boundary。
- old reviewer insight、prior failure record、source caveat 或 open question 被迁移时静默丢失。

## High-Value Old-Content Preservation

旧内容是治理对象，不是模板迁移噪音。Migration、structural upgrade、replacement、deprecation 或 archive 被考虑时，必须先识别高价值旧内容，再决定处置方式。

High-value old content 至少包括：

- accurate explanations、examples、counterexamples、diagrams、tables、diagnostic checks、self-tests 和 transfer entries。
- source notes、source limitations、checked dates、source/currentness caveats 和 historical context。
- reviewer notes、prior review insights、prior failure records、decision rationale、open questions 和 owner decisions。
- links/index evidence、related_docs evidence、legacy assumptions、old path/title/topic evidence 和 compatibility constraints。

允许的 old-content dispositions 是：

| Disposition | Meaning | Required evidence |
| --- | --- | --- |
| `preserved` | 旧内容仍在同一资产中保留，可能经过局部重组 | 说明保留位置和为什么仍有 current/historical value |
| `moved` | 旧内容移动到明确目标资产、section、review evidence 或 support record | 说明目标位置、link/index impact 和 recoverability |
| `summarized` | 旧内容被压缩为摘要，原细节不再完整保留 | 说明压缩理由、丢失风险和是否可恢复 |
| `replaced` | 新内容替换旧内容，旧内容不再作为当前表达 | 说明 replacement reason、source basis、successor/replacement 和旧内容剩余价值 |
| `deleted_with_reason` | 旧内容被删除 | 说明删除原因、risk、recoverability、owner/follow-up 和 link/lifecycle impact |
| `archived` | 旧内容保留为历史、审计、migration 或 legacy context | 说明 remaining access、archive role 和 current-truth boundary |
| `deferred` | 旧内容需要处理但当前 story 不授权或证据不足 | 说明 reason、owner、future-story dependency 和 current risk |

不得为了套新模板、统一风格或缩短篇幅而删除准确解释、旧 reviewer insight、prior failure record、source caveat 或仍有历史/审计价值的材料。删除、压缩、替换或断开旧内容时，至少记录 reason、risk、recoverability、successor/replacement、link/index/lifecycle impact 和 owner/follow-up。

如果旧内容价值不清楚，使用 `hold_for_clarification`、`defer_with_reason` 或 `not_authorized`，不得先执行 overwrite。

## Migration And Compatibility Outcomes

Migration assessment 可以产生 local migration handling outcome，也可以触发 document decision policy 的 final decision label。Local outcome 不是新的全局 frontmatter 字段；它只能写在 body evidence、review evidence、completion evidence、story Dev Agent Record、specialized record 或 migration note 中。

| Local outcome | Use when | Maps to final decision when needed | Required adjacent record |
| --- | --- | --- | --- |
| `no_action_currently_compatible` | 旧资产诚实表达自身时间/来源/状态，当前使用风险低，不需要立即迁移 | `accepted_for_current_use` 或 `accepted_with_nonblocking_follow_up` | none, unless index/link/source issue is discovered |
| `record_gap_only` | 缺口存在但当前任务只授权评估或缺口不阻塞当前使用 | `accepted_with_nonblocking_follow_up`、`defer_with_reason` 或 `hold_for_clarification` | review/completion evidence or open_questions/follow-up |
| `targeted_metadata_backfill` | 只需补齐或修正已授权 metadata，不改变身份、正文主结构或状态语义 | `targeted_revision` | frontmatter-schema evidence; Index Impact if navigation changes |
| `targeted_revision` | 局部正文、metadata、source/time、link/index 或 status wording 修复足够 | `targeted_revision` | Revision / Regeneration Continuity policy check; create the record or record why full record is not applicable |
| `structural_upgrade` | 同一身份内需要补结构、验证入口、迁移入口或治理证据 | `targeted_revision` 或 `accepted_with_nonblocking_follow_up` after evidence | Revision / Regeneration Continuity Record |
| `regeneration_candidate` | 局部修复不足，但当前还未授权实际 regeneration | `regenerate`、`hold_for_clarification` 或 `defer_with_reason` | Continuity Record plus old-content inventory before writing |
| `deprecate_or_archive` | 旧资产不应再作为主要当前入口，或只应作为历史/审计证据保留 | `deprecate_or_archive` | Migration Decision Record and Index Impact Decision Record |
| `hold_for_clarification` | 身份、owner、source、successor、scope 或 old-content value 不清 | `hold_for_clarification` | owner decision evidence |
| `defer_with_reason` | 问题真实存在，但当前 story、授权、batch readiness 或 future owner 不允许处理 | `defer_with_reason` | follow-up, future-story dependency or open_questions |
| `not_authorized` | 请求越过非软件边界、schema boundary、batch readiness、future-story boundary 或 file scope | `not_authorized` | boundary evidence and safe stopping point |

Specialized record routing:

- `targeted_revision`、`structural_upgrade`、`regeneration`、`split`、`merge` 或 `deprecation` 必须调用 `docs/governance/revision-regeneration-continuity-policy.md`。最低要求是执行 continuity policy check；如果不需要完整 Revision / Regeneration Continuity Record，必须记录 `not_applicable` reason。涉及 identity、old-content handling、reference validity 或 update-mode evidence 时，必须创建或更新 Revision / Regeneration Continuity Record。
- rename、move、retitle、topic migration、successor/replacement、deprecation 或 archive 影响 path、identity、old content、remaining access、links/index 或 lifecycle 时，继续调用 `docs/governance/rename-migration-policy.md` 的 Migration Decision Record。
- candidate promotion、duplicate/coexistence、index impact、source discipline、sidecar boundary 和 batch readiness 使用各自资产；本文只提供 legacy migration assessment 入口，不复制完整模型。

## Batch Migration Risk Criteria

单个命名资产的 migration assessment，加上一处直接 `docs/index.md` 更新，不自动构成 batch work。是否 batch-shaped 取决于目标集选择方式、影响面和恢复难度。

以下情况必须先进入 `docs/governance/batch-readiness-checklist.md`，不得直接执行迁移：

- target set selected by rule：例如“所有缺少 `source_basis` 的文档”“所有旧模板版本文档”“所有 governance docs”。
- multiple target assets selected for assessment record collection, even if the immediate output is only a set of assessment records rather than migrated documents.
- multiple files、multiple topics、multiple status/version fields 或 multiple index entries 被同时修改。
- batch frontmatter normalization、batch status conversion、batch prompt/template version normalization 或 batch lifecycle wording conversion。
- batch link/index update、batch deprecation/archive、generated rewrite、old-doc inventory、source/currentness sweep 或 broad inbound-reference repair。
- 需要用脚本、生成器、link scanner、index generator、validator、migration script 或其他 executable automation 才能稳定完成。

Batch migration 前必须记录：

- migration rule：为什么这些文件进入目标集。
- explicit target set：允许修改的文件清单。
- allowed files / excluded files：包括为什么排除。
- expected output / non-output：本轮会产出什么，不会产出什么。
- recovery approach：如何回退、隔离、比较或恢复旧内容。
- sampling/review approach：如何选样、谁复核、样本通过标准是什么。
- conflict matrix：path/topic/index、doc_id、quality_status、source/time、sidecar/support、duplicate/coexistence、successor/replacement、Epic 5/Epic 6 boundary 的冲突。
- owner decision：`proceed`、`proceed_with_exclusions`、`clarify_before_write`、`defer_to_later_story`、`stop_for_maxwell_confirmation` 或 `not_authorized`。
- stop conditions：什么发现会停止批量写入。

Sample check 要求至少验证一个或一组代表性文件的：

- frontmatter shape、YAML arrays、required fields 和 unauthorized field absence。
- path/topic/index relation、body links、fragment anchors、related_docs 和 discoverable inbound reference risk。
- high-value old content inventory 和 old-content disposition feasibility。
- quality/lifecycle impact、source/time context、prompt/template/version claim 和 currentness risk。
- recovery feasibility：能否恢复旧内容、旧路径、旧索引、旧状态或旧支持材料。

Sample pass 是 readiness evidence，不是全量通过。Path/topic/index risks 必须暴露为 readiness evidence，不得在 batch completion wording 中隐藏。

## Legacy Migration Assessment Record

`Legacy Migration Assessment Record` 是 human-readable record，不是 schema、lint rule 或 frontmatter 字段。它可以落在 target body、review evidence、completion evidence、story Dev Agent Record、Revision / Regeneration Continuity Record、Migration Decision Record、future runbook record 或明确命名的 migration note 中。

Copyable skeleton:

```text
Legacy Migration Assessment Record

- target asset:
- target path:
- target doc_id:
- legacy class:
- migration intent:
- current status signals:
- main-document relationship:
- gap classification:
  - frontmatter / metadata:
  - structure:
  - source / time / currentness:
  - examples / validation:
  - identity / path / topic:
  - links / related_docs / index:
  - lifecycle / quality_status:
  - sidecar / support material:
- frontmatter impact:
- structure impact:
- source/time impact:
- examples/validation impact:
- identity/path/topic impact:
- high-value content inventory:
- old-content disposition:
- sidecar/support impact:
- links/related_docs/index impact:
- lifecycle/quality_status impact:
- prompt/template/version impact:
- batch risk classification:
- sample check status:
- specialized records required:
- decision / owner:
- remaining gaps:
- validation result:
- follow-up:
```

Minimum field guidance:

- `target asset` / `target path` / `target doc_id` identify the object being assessed; missing `doc_id` is itself a gap.
- `main-document relationship` is required when the assessed object is sidecar, support material, report, note, candidate, archive entry or review/completion evidence.
- `gap classification` must use the taxonomy in this guide; do not collapse all issues into “needs update”。
- `old-content disposition` must use `preserved`、`moved`、`summarized`、`replaced`、`deleted_with_reason`、`archived` 或 `deferred`。
- `batch risk classification` must state whether the request is single-asset, limited adjacent update or batch-shaped。
- `specialized records required` must name Continuity, Migration, Index Impact, Sidecar Boundary, Review, Completion, Batch Readiness or other adjacent records when applicable。
- `decision / owner` must map to current document decision labels when a final decision is needed。

Do not split record fields into unauthorized frontmatter. If a field deserves durable structured treatment later, it belongs to a future schema/version governance decision, not an ad hoc local field.

## Remaining Gaps And Completion Wording

Remaining gaps must remain visible. They can be recorded in:

- target asset `open_questions` when they affect the asset’s future correctness, source/time boundary, link/index relation, successor relation or owner decision.
- `quality_status` impact explanation in body, review evidence or completion evidence.
- review notes、completion evidence、follow-up list、future-story dependency、owner decision 或 story Dev Agent Record。
- specialized records such as Continuity Record、Migration Decision Record、Index Impact Decision Record、Sidecar Boundary Decision Record 或 future Epic 6 runbook record。

Gap status must be explicit:

| Gap status | Use when | Completion wording allowed | Completion wording forbidden |
| --- | --- | --- | --- |
| `blocked` | Gap blocks current acceptance or safe use | blocked until X is resolved | completed, compatible, accepted |
| `nonblocking` | Gap is real but does not prevent current limited use | accepted with nonblocking follow-up | no risk, fully validated |
| `held` | Owner/source/identity decision is unclear | held for clarification | deferred as if optional |
| `deferred_with_reason` | Current story or future owner boundary prevents handling | deferred with reason, owner and trigger | resolved, migrated, complete |
| `not_authorized` | Requested action crosses current authorization | not authorized; safe stopping point recorded | skipped as convenience, silently excluded |

Do not write:

- “migration complete” when only migration assessment, gap recording, index update or policy creation occurred.
- “gap resolved” when the gap was only recorded, deferred, held or not authorized.
- “validated” or `maintained_asset` when the asset only has draft compatibility or limited review evidence.
- “all old docs migrated” unless an authorized batch process actually migrated the explicit target set and passed its readiness/completion evidence.

Completion evidence must distinguish:

- changed files.
- unchanged-reviewed files.
- skipped files.
- deferred files.
- not-authorized files.

This distinction matters because legacy migration work often includes assessment without mutation. A file can be reviewed and left unchanged; that is not a silent migration.

## Relationship To Adjacent Governance Assets

Use this guide as the legacy assessment entry point, then route specialized decisions to the owning asset.

| Adjacent asset | What it owns | How this guide uses it |
| --- | --- | --- |
| `docs/governance/frontmatter-schema.md` | baseline fields、stable `doc_id`、YAML arrays、unauthorized field prohibition | Identify metadata gaps and prevent ad hoc migration/compatibility fields |
| `docs/governance/lifecycle-states.md` | lifecycle vocabulary and transitions | Keep `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` semantics honest |
| `docs/governance/prompt-template-quality-version-governance.md` | old-doc compatibility、version change record、no automatic version bump、gradual migration | Avoid automatic prompt/template/quality version normalization |
| `docs/governance/revision-regeneration-continuity-policy.md` | update modes、metadata review、old-content handling、reference validity、Continuity Record | Route `targeted_revision`、`structural_upgrade`、`regeneration`、`split`、`merge`、`deprecation` evidence |
| `docs/governance/sidecar-boundary-policy.md` | main-document canonical truth、support artifact taxonomy、stale sidecar handling、Sidecar Boundary Decision Record | Assess legacy sidecars、old notes、historical support material and trust boundary |
| `docs/governance/rename-migration-policy.md` | Migration Decision Record、successor/replacement、remaining access、link/index/lifecycle impact | Route rename、move、retitle、topic migration、successor、replacement、deprecation/archive |
| `docs/governance/index-synchronization-rules.md` | Index Impact outcomes and formal index boundaries | Classify index entry updates, exclusions, blocked conflicts and relative link evidence |
| `docs/governance/batch-readiness-checklist.md` | readiness scope、conflict matrix、sample/review、recovery、owner decision | Stop batch-shaped migration before write actions |
| `docs/governance/document-decision-policy.md` | final decision labels、blocker/status、review/completion evidence | Map local migration outcomes to accepted/current/revision/regenerate/hold/defer/not_authorized decisions |
| `docs/templates/review-record-template.md` | review evidence structure | Place migration assessment findings, blockers, unverified items and specialized records in review evidence |
| `docs/templates/completion-report-template.md` | completion evidence and quality/status impact reporting | Distinguish changed, unchanged-reviewed, skipped, deferred and not-authorized files |
| `docs/methodology/document-generation-methodology.md` | concept document generation/upgrade/review baseline | Evaluate structural gaps and old-doc upgrade needs without bypassing quality gates |
| `docs/methodology/intake-and-intent-classification.md` | task type, path type, depth and missing input handling | Classify whether a request is assessment, upgrade, migration, index-only or governance maintenance |
| `docs/methodology/concept-document-quality-gate.md` | Hard Fail and scoring baseline | Identify weak structure, examples, validation and migration gaps |
| `docs/methodology/source-discipline-and-real-world-anchor-policy.md` | source/currentness, historical/deprecated framing and real-world anchors | Assess outdated claims, source limitations, checked dates and legacy example framing |

Future-owner boundaries:

- `docs/governance/related-docs-taxonomy.md` owns detailed related-doc taxonomy；`docs/governance/link-maintenance-policy.md` owns general link maintenance；`docs/governance/existing-doc-reuse-procedure.md` owns existing-doc reuse；`docs/governance/reusable-model-entry-points.md` owns reusable model entry points；`docs/governance/network-boundary-and-decay-prevention.md` owns knowledge-network decay prevention。本文只记录这些缺口和 maintenance dependency。
- Epic 6 owns batch governance runbook、batch change review record 和 batch completion report。本文只 defines batch migration risk criteria and readiness handoff, not execution.

## Validation Checklist And Maintenance Triggers

Use this checklist when creating or updating this guide, or when applying it to a target asset:

1. H1/title/frontmatter title are consistent enough to identify legacy migration、old-doc compatibility、gap classification、preservation rule、batch risk criteria and remaining-gap handling.
2. Frontmatter contains required baseline fields; `source_basis`、`related_docs` and `open_questions` are YAML arrays.
3. No unauthorized frontmatter fields were added for migration, compatibility, owner, navigation, decision, lifecycle, successor or batch status.
4. Asset role、authority、scope、owner entry point、navigation treatment、quality_status and Index Impact Decision Record are present.
5. Legacy asset classes include formal docs、partial-frontmatter docs、methodology/governance assets、formal reports/archive entries、legacy sidecars/support notes、candidate/workflow outputs and deprecated/archived assets.
6. Gap taxonomy covers frontmatter/metadata、structure、currentness/source-time、examples/validation、repository integration、lifecycle/quality and sidecar/support material.
7. High-value old-content preservation covers explanations、examples、diagrams、source notes、reviewer insights、prior failures、open questions、historical context and links/index evidence.
8. Old-content dispositions use only `preserved`、`moved`、`summarized`、`replaced`、`deleted_with_reason`、`archived` and `deferred`.
9. Migration/compatibility outcomes distinguish local handling from document decision labels.
10. Continuity, Migration, Sidecar, Index, Decision, Review, Completion and Batch Readiness records are routed to owning assets.
11. Batch migration criteria include rule-selected target sets、multiple files/topics/statuses/index entries、batch normalization、batch link/index updates、generated rewrites、old-doc inventories and source/currentness sweeps.
12. Batch readiness evidence includes migration rule、target set、allowed/excluded files、expected output/non-output、recovery approach、sampling/review approach、conflict matrix、owner decision and stop conditions.
13. Sample check requirements include frontmatter shape、path/topic/index relation、link impact、high-value old content、quality/lifecycle impact、source/time context and recovery feasibility.
14. Legacy Migration Assessment Record is copyable, human-readable and explicitly not a schema or frontmatter extension.
15. Remaining gaps and completion wording distinguish `blocked`、`nonblocking`、`held`、`deferred_with_reason` and `not_authorized`.
16. Body links and `related_docs` targets resolve or planned targets are recorded in `open_questions` / future dependencies.
17. `docs/index.md` contains this asset under `## governance` and its relative link resolves.
18. No actual old-doc inventory, old-doc migration, batch frontmatter normalization, batch status conversion, batch link/index repair, actual sidecar extraction, extra Epic 5 link/network work, Epic 6 batch work or software automation was performed under this guide alone.
19. No runtime code、package manifests、source/test directories、software test frameworks、schemas、validators、generators、automation、CLI/API/UI/database/deployment/CI、runbooks、actual completion reports or migration scripts were created.

Maintenance triggers:

- related-doc taxonomy, link maintenance policy, existing-doc reuse procedure, reusable model entry points or network-boundary assets are updated.
- Epic 6 creates batch governance runbook, batch change review record or batch completion report template.
- Maxwell authorizes actual legacy inventory, named old-doc migration or tooling/schema changes.
- A review finds repeated confusion between migration assessment and actual migration, or between draft compatibility and validated/current status.
- Adjacent assets change decision labels, lifecycle vocabulary, frontmatter schema, batch readiness owner decisions or index impact outcomes.

## 参考资料

- [Knowledge Docs Index](../index.md)
- [related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling](./related-docs-taxonomy.md)
- [跨文档链接维护政策：existence/path/topic/meaning checks、inbound/outbound review、one-way reason 与 boundary conflict handling](./link-maintenance-policy.md)
- [可复用模型入口政策：core model、boundary rule、decision frame、failure pattern 与 verification method](./reusable-model-entry-points.md)
- [既有文档复用流程：发现、对齐、复用决策与避免重复生成](./existing-doc-reuse-procedure.md)
- [知识网络主题边界与退化防护政策](./network-boundary-and-decay-prevention.md)
