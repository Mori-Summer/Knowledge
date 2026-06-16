---
doc_id: governance-revision-regeneration-continuity-policy
title: 修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性
concept: revision_regeneration_continuity_policy
topic: governance
depth_mode: standard
created_at: '2026-05-28T10:04:11+08:00'
updated_at: '2026-05-28T15:29:36+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/4-1-revision-regeneration-continuity-policy.md
  - _bmad-output/implementation-artifacts/4-2-sidecar-boundary-policy.md
  - _bmad-output/implementation-artifacts/4-3-legacy-migration-guide.md
  - docs/index.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/templates/completion-report-template.md
  - docs/templates/review-record-template.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
time_context: phase_4_epic_4_revision_regeneration_continuity_policy_2026_05_28
applicability: formal_docs_revision_regeneration_identity_and_continuity_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/templates/completion-report-template.md
  - docs/templates/review-record-template.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
open_questions:
  - "`docs/governance/legacy-migration-guide.md` 已建立后，是否需要在后续审查中把 structural upgrade 与 legacy migration 的交界条件补充为更细的 worked examples？"
  - Story 5.1 related-doc taxonomy 与 Story 5.2 link maintenance policy 已建立；后续是否需要细分 reference validity、successor link、supporting link 和 related_docs impact 的维护记录？
  - Epic 6 建立 batch governance runbook 后，是否需要把 batch regeneration / batch migration continuity records 转入专门 runbook records？
---

# 修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 revision、structural upgrade、regeneration、split、merge、deprecation 和相关 continuity record 的 canonical governance asset。它回答的是：正式资产需要更新时，Agent 或 reviewer 应如何选择更新模式、保留或改变 `doc_id` 身份、处理旧内容、检查 metadata、判断 links/index/reference validity，并在完成或停止时留下可审查的连续性证据。

本文适用于：

- `docs/{topic}/` 下的正式知识资产。
- `docs/methodology/` 下的方法论、模板、质量门禁、fixed prompt、playbook 和方法论支撑资产。
- `docs/governance/` 下的治理资产。
- `docs/templates/` 下的正式模板资产。
- `docs/index.md`、正式 report entry，以及未来被 Maxwell 或后续 story 明确批准的 `docs/runbooks/` 资产。
- review evidence、completion evidence、story Dev Agent Record、Migration Decision Record、Index Impact Decision Record 或 future runbook record 中需要汇总 continuity evidence 的场景。

本文的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/revision-regeneration-continuity-policy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Epic 4 首版 revision/regeneration continuity policy；`docs/governance/sidecar-boundary-policy.md` 已细化 sidecar boundary，`docs/governance/legacy-migration-guide.md` 已细化 old-doc compatibility 与 legacy migration assessment，Story 5.1 已细化 related-doc taxonomy，后续 Story 5.2+ 和 Epic 6 仍会细化 link/network governance 和 batch runbook records。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/revision-regeneration-continuity-policy.md
- target title: 修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性
- target path: ./governance/revision-regeneration-continuity-policy.md
- target section: docs/index.md ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: Story 4.1 explicitly authorizes the canonical revision/regeneration continuity policy asset
- validation result: target exists and relative link resolves
- unresolved risk: sidecar boundary is referenced via docs/governance/sidecar-boundary-policy.md; legacy migration is referenced via docs/governance/legacy-migration-guide.md; Epic 5 link taxonomy and Epic 6 batch runbook integrations remain open questions
```

## 职责边界与非目标

本文定义 decision/evidence/record requirements。它不执行任何实际 revision、regeneration、split、merge、deprecation、archive、legacy migration、sidecar extraction、batch migration 或 batch link repair。

本文不实现以下范围：

- `docs/governance/sidecar-boundary-policy.md`：sidecar、supplemental material、revision notes、example sets、review records 和 supporting notes 的角色边界。
- `docs/governance/legacy-migration-guide.md`：旧文档渐进迁移、兼容策略、legacy migration assessment 和 old-doc compatibility。
- Epic 5 related-doc taxonomy、link maintenance、reusable model entry points、existing-doc reuse procedure 或 network boundary assets。
- Epic 6 batch governance runbook、batch change review record、batch completion report template 或 batch execution records。
- 既有正式资产的实际重生成、批量迁移、批量状态改写、批量 frontmatter normalization、旧文档升级或 generated continuity records。
- runtime code、package manifest、source tree、software tests、CLI/API/UI/database/deployment/CI、machine-readable schema、validator、generator、link scanner、index generator、migration script、lint/scoring tool 或 executable automation。

本文不授权在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`quality_gate_version`、`methodology_version`、`migration_status`、`decision_status`、`successor`、`merged_from`、`split_from`、`regeneration_status` 或类似字段。Continuity evidence 应写在目标正文、review evidence、completion evidence、story Dev Agent Record、Migration Decision Record、future runbook record 或明确命名的 revision note 中。

## Update mode taxonomy

更新模式必须先判定，再写入。不得把 revision、structural upgrade、regeneration、split、merge 和 deprecation 混成“更新一下”。

| Mode | Use when | Do not use when | Identity handling | Old-content handling | Required evidence and records | Allowed decision labels | Stop conditions |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `targeted_revision` | 同一资产身份连续，缺口可定位到具体 field、section、link、source claim、index entry 或 status wording | 文档类型错误、结构坍塌、source basis 不可用、identity/path collapse，或局部修复会 silent overwrite | 默认保留原 `doc_id`；不得因 title/path/topic/H1 改动自动换身份 | 默认保留高价值内容，只改必要位置；删除或压缩必须说明理由 | metadata review、affected section/link evidence、Index Impact Decision Record if navigation changes、Migration Decision Record if old-content or path/identity is affected | `targeted_revision`、`accepted_with_nonblocking_follow_up`、`accepted_for_current_use` | 修复范围不可定位；修复会改变 identity；critical source/link/index/status risk 无法处理 |
| `structural_upgrade` | 同一资产身份连续，但结构、模板、章节、quality gate coverage 或 governance evidence 覆盖面有实质增强 | 目标变成不同资产、需要新 canonical path，或旧内容价值/身份连续性不清 | 通常保留原 `doc_id`；`created_at` 不重置；`updated_at` 只因实质变更更新 | 保留准确解释、examples、review notes、source notes、open questions 和历史 context；移动或摘要必须记录 | metadata review、before/after structure rationale、version impact review、Index Impact Decision Record if title/navigation changes、Migration Decision Record if old-content disposition changes | `targeted_revision`、`accepted_with_nonblocking_follow_up`、`accept_promote` only after evidence supports it | structural change 会伪造 prompt/template/quality version；需要 batch migration；old-content preservation 不清 |
| `regeneration` | 局部修复不足以恢复完整性，需要重建正文或核心结构；必须说明为什么 targeted revision 不足 | 局部修复可恢复完整性；当前 story 不授权重写；高价值旧内容无法处理 | 先判断旧身份是否连续；连续则保留 `doc_id`，非连续则需要 owner/Migration evidence | 不得静默覆盖；必须分类处理 accurate explanations、examples、diagrams、source notes、reviewer notes、prior failures、open questions、links、related_docs、index evidence 和 historical context | pre-regeneration check、why targeted revision insufficient、high-value content inventory、Revision / Regeneration Continuity Record、Migration Decision Record if replacement/successor/split/merge/deprecation/archive affected、Index/Duplicate/Source evidence as applicable | `regenerate`、`hold_for_clarification`、`defer_with_reason`、`not_authorized` | identity continuity、old-content value、successor/replacement、link/index impact 或 lifecycle/status impact 不清 |
| `split` | 一个旧资产拆成多个职责边界清楚的新资产或 successor path | 只是局部章节重排，或旧资产仍承担同一职责 | 必须记录 old/new `doc_id` treatment；可保留旧身份、创建新身份、废弃旧入口或建立 successor mapping | 每个旧 section 的 disposition 必须是 `preserved`、`moved`、`summarized`、`replaced`、`deleted_with_reason`、`archived` 或 `deferred` | Migration Decision Record、Duplicate / Coexistence Decision Record if overlap exists、Index Impact Decision Record、Continuity Record、reference validity check | `accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`accept_promote`、`targeted_revision`、`regenerate`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason` | old/new identity、path ownership、successor mapping、duplicate/coexistence 或 batch scope 不清 |
| `merge` | 多个旧资产合并为一个 canonical asset 或共享入口 | 资产应保持显式共存，或只是新增 cross-reference | 必须记录多个旧 `doc_id` 的保留、归并、废弃、归档或 successor treatment | 逐项说明旧内容 preserved/moved/summarized/replaced/deleted/archived/deferred；不得丢失高价值旧内容或 prior failures | Migration Decision Record、Duplicate / Coexistence Decision Record、Index Impact Decision Record、Continuity Record、reference validity check | `accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`accept_promote`、`reject_duplicate_or_misplaced`、`targeted_revision`、`regenerate`、`deprecate_or_archive`、`hold_for_clarification` | canonical entry、merged identity、old-content disposition、link/index impact 或 remaining access expectation 不清 |
| `deprecation` | 旧资产保留访问但不再作为主要当前入口，或旧资产需进入 archive treatment | 只是轻量修订，或应直接修复为 current entry | 保留旧 `doc_id`；archive 时记录旧身份如何保留、从哪里可访问、是否有 successor | 旧内容保留为历史、legacy context、migration audit、archive evidence 或旧决策证据；必须说明 remaining access | deprecation/archive reason、successor/replacement or missing-successor risk、Migration Decision Record、Index Impact Decision Record、lifecycle/status impact、reference validity check | `deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason`、`not_authorized` | successor/replacement、remaining access、archive location、index visibility、links 或 lifecycle/status evidence 不清 |

`structural_upgrade` 是同一身份内的实质结构增强，不是新 lifecycle/frontmatter 字段，也不是自动 quality promotion。它必须复核 prompt/template/version impact，但不得伪造旧文档已迁移到新 prompt、template 或 quality gate。

`regeneration` 是强动作。它必须先证明 targeted revision 不足，并且必须保护旧内容价值、reviewer notes、prior failure records 和 reference validity。不能用“重新生成更快”替代证据。

`archived` 不是独立 update mode；它是 `deprecation` / `deprecate_or_archive` 下的 stronger lifecycle treatment。Archive 决策必须记录 remaining access、archive location 或 retrieval expectation，并说明旧引用、index visibility 和 successor/replacement 是否仍然有效。

## Substantive revision metadata review

任何具有实质内容、状态、来源或结构影响的 revision / regeneration / split / merge / deprecation，都必须复核 frontmatter 与正文证据。只观察、阅读或确认无变化，不应伪造 metadata 更新。

最低检查项如下：

| Metadata / evidence | Required review |
| --- | --- |
| `updated_at` | 只有实质正文、metadata、source/time、quality/status、link/index、identity 或 governance evidence 改变时才更新；不得因观察、未变更审查或 story checkbox 完成而伪造更新时间。 |
| `source_basis` | 检查是否需要加入新规则、review evidence、source evidence、old-content rationale、owner decision 或 story evidence；不得加入未实际使用的来源。 |
| `time_context` | 检查当前实践、历史/淘汰说法、项目阶段、source/currentness 限制和正文日期是否仍匹配。 |
| `quality_status` | 必须由 review/quality/equivalent governance evidence 支撑；不得因结构完整、文件创建、索引存在或 BMad story status 自动晋升。 |
| `open_questions` | 检查是否需新增、解决、保留或转为 follow-up；不得隐藏 identity、source、link、index、status、successor 或 old-content 风险。 |
| `prompt_version` | 检查修订是否实际受 fixed prompt 变化影响；不得只因新 prompt 存在就 bump。 |
| `template_version` | 检查结构模式是否实际改变；不得把 structural upgrade 伪装成旧资产已完整迁移。 |
| `related_docs` | 检查 related targets、successor/replacement、supporting governance assets 和 planned/future targets；不存在的目标写入 open questions 或 future dependency。 |
| body links and fragments | 检查 changed-file links、fragment anchors、successor/replacement links 和相邻治理资产链接。 |
| `docs/index.md` | 使用 index synchronization vocabulary 判断 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict`。 |
| lifecycle wording | 与 `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` 和 reactivation 语义兼容。 |
| specialized records | 判断是否需要 Promotion、Index Impact、Migration、Duplicate/Coexistence、Batch Readiness、Source Discipline、Review/Rework/Completion evidence。 |

Metadata 更新必须是 evidence-based。不得用 cosmetic edit、story status、agent assertion 或 “看起来完整” 支撑 `updated_at`、`quality_status`、`prompt_version`、`template_version` 或 lifecycle wording。

## Regeneration overwrite prevention

Regeneration 不得静默覆盖已有正式资产。不得通过创建新文件绕过旧 `doc_id`、旧 path、旧 review notes、旧 failures、旧 open questions、旧 links 或旧 index evidence。

Regeneration 前必须记录：

- target identity：current path、title、topic、`doc_id`、H1 和 index entry。
- candidate source：新内容来自 user request、story、review finding、旧资产、候选稿、source evidence 或 owner decision。
- why targeted revision is insufficient：说明局部修复为什么不能恢复结构、身份、来源、复用或安全性。
- old-content value：哪些旧内容仍准确、有解释价值、有历史价值、有 review value 或有 source value。
- stop/hold conditions：哪些 identity、source、link、index、old-content、successor 或 lifecycle gap 会停止 regeneration。

High-value content 必须显式处理，至少包括：

- accurate explanations。
- examples、diagrams、tables 或 diagnostic checks。
- source notes、source limitations 和 checked dates。
- reviewer notes、prior failure records 和 resubmission evidence。
- open questions、unverified items、deferred items 和 owner decisions。
- body links、fragment anchors、`related_docs`、index evidence 和 discoverable inbound references。
- migration evidence、duplicate/coexistence evidence、historical context 和 legacy assumptions。

允许的 old-content disposition 只有：

| Disposition | Meaning |
| --- | --- |
| `preserved` | 旧内容保留在同一资产中，可能经过局部重组。 |
| `moved` | 旧内容移动到明确目标资产、section 或 authorized record。 |
| `summarized` | 旧内容被压缩为摘要，原细节不完整保留。 |
| `replaced` | 新内容替换旧内容，并说明旧内容为什么不再作为当前表达。 |
| `deleted_with_reason` | 删除旧内容并记录原因、风险和是否可恢复。 |
| `archived` | 旧内容作为历史、审计、legacy context 或 migration evidence 保留。 |
| `deferred` | 当前不能决定，记录 blocker、owner 和 re-review trigger。 |

当 identity continuity、old-content value、successor/replacement、link/index impact、duplicate/coexistence 或 lifecycle/status impact 不清时，必须使用 `hold_for_clarification`、`defer_with_reason`、`not_authorized` 或明确 blocker。不得继续覆盖。

Regeneration 影响 replacement、successor、split、merge、deprecation、archive 或 old-content handling 时，必须引用 Migration Decision Record。影响 index/navigation 时引用 Index Impact Decision Record。影响 duplicate、near duplicate、same-topic coexistence 或 canonical path 时引用 Duplicate / Coexistence Decision Record。涉及 source/currentness 时引用 Source Discipline evidence。目标集变成批量时先执行 Batch Readiness Record。

## Revision / Regeneration Continuity Record

`Revision / Regeneration Continuity Record` 是人类可复制的连续性记录。它不是 machine-readable schema，也不是新的 frontmatter 字段。

最低字段如下：

```markdown
## Revision / Regeneration Continuity Record

| Field | Value |
| --- | --- |
| affected assets |  |
| update mode | targeted_revision / structural_upgrade / regeneration / split / merge / deprecation |
| reason |  |
| old path / title / topic / doc_id |  |
| new path / title / topic / doc_id |  |
| identity treatment | preserve_doc_id / new_doc_id / successor_mapping / merged_identity / split_identity / deprecated_identity / held / deferred_with_reason / not_authorized |
| what changed |  |
| why changed |  |
| what was preserved |  |
| what was removed |  |
| old-content handling | per-section or per-entry list: section_or_item, disposition, target_or_reason, validation |
| source/time impact |  |
| quality/lifecycle impact |  |
| links/related_docs impact |  |
| index impact | updated / not_applicable / referenced_elsewhere / intentionally_excluded / deferred_with_reason / blocked_index_policy_conflict |
| reference validity |  |
| required specialized records | Promotion / Index Impact / Migration / Duplicate-Coexistence / Batch Readiness / Source Discipline / Review-Rework-Completion / not_applicable |
| validation result |  |
| unresolved risks |  |
| owner/future-story dependency |  |
```

`reference validity` 必须明确说明以下引用是否仍有效：

- existing body links。
- fragment anchors。
- `related_docs` entries。
- `docs/index.md` entries。
- discoverable inbound references。
- successor/replacement references。
- review/completion/story evidence links if they are part of the decision.

如果某项没有验证，必须写 `未验证`、`deferred_with_reason` 或 blocker。不得把未知引用有效性写成 “none”。

Continuity Record 可以落在：

- 目标正文中的 revision note、regeneration note、deprecation note 或 continuity section。
- review evidence。
- completion evidence。
- story Dev Agent Record。
- Migration Decision Record。
- future runbook record。
- 明确命名的 revision note。

Continuity Record 不得作为多个新 frontmatter fields 散落到正式资产中。

## Relationship to lifecycle, migration, decision and completion assets

本文调用相邻治理资产，不替代它们的字段权威。

| Asset | 本文如何使用 |
| --- | --- |
| `docs/governance/lifecycle-states.md` | 使用 `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` 和 reactivation 语义；revision/regeneration 不自动晋升生命周期状态。 |
| `docs/governance/frontmatter-schema.md` | 使用 required fields、YAML array、stable `doc_id` identity 和 unauthorized frontmatter field 禁令；continuity evidence 不新增 schema fields。 |
| `docs/governance/rename-migration-policy.md` | 使用 Migration Decision Record、old-content handling、successor/replacement、split/merge/deprecation/archive evidence；本文补充 revision/regeneration 连续性问题。 |
| `docs/governance/index-synchronization-rules.md` | 使用 index impact outcomes：`updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason`、`blocked_index_policy_conflict`。 |
| `docs/governance/document-decision-policy.md` | 使用 decision labels：`accept_promote`、`accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`targeted_revision`、`regenerate`、`reject_duplicate_or_misplaced`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason`、`not_authorized`。 |
| `docs/governance/rework-loop-examples.md` | 使用 regeneration rationale、prior failure preservation 和 resubmission logic；本文不复制完整示例库。 |
| `docs/templates/completion-report-template.md` | Completion report 汇总 continuity evidence、quality/status impact、specialized record status 和 unresolved risks；不替代 review/decision/migration/continuity records。 |
| `docs/templates/review-record-template.md` | Review record 提供 task classification、Hard Fail/blocker evidence、未验证/不适用登记和 final decision 位置。 |
| `docs/governance/batch-readiness-checklist.md` | 判断 revision/regeneration 是否已变成 batch migration、batch link repair、batch status conversion、batch frontmatter normalization 或 batch generated rewrite。 |
| `docs/governance/prompt-template-quality-version-governance.md` | 检查 structural upgrade、template/prompt/quality rule changes、gradual migration 和 fake version bump 风险。 |
| `docs/governance/candidate-promotion-checklist.md` | 当候选内容进入正式资产或 replacement with continuity 时，检查 promotion / replacement continuity evidence。 |
| `docs/governance/duplicate-and-coexistence-policy.md` | 当 split、merge、duplicate、near duplicate、same-topic coexistence 或 canonical path 风险出现时，使用其 record 和 outcome。 |
| `docs/governance/sidecar-boundary-policy.md` | 分类 sidecar、report、revision note、example set、review record 和 other support asset，并检查主文档 canonical truth、support artifact impact 与 stale sidecar handling。 |

一次变更可以同时需要多个 records。不得用 Continuity Record 覆盖 Migration Decision Record、Index Impact Decision Record、Duplicate / Coexistence Decision Record 或 Completion Report 的 required fields。

## Reference validity and index/link impact

Revision、regeneration、split、merge 和 deprecation 完成或停止时，必须说明 existing references 是否仍然 valid。检查范围应与当前授权相称，但不能假装没有影响。

最低检查项：

- changed-file body links：当前被修改文件中的 Markdown links 是否可解析。
- fragment anchors：带 `#fragment` 的链接是否仍指向存在 heading 或 anchor；未知时记录 `未验证`。
- `related_docs`：目标是否存在，是否需要新增、移除、替换或延后。
- `docs/index.md`：使用 index synchronization outcomes 判断更新、排除、引用 elsewhere、延后或阻塞。
- discoverable inbound references：通过本地搜索容易发现的旧 path、title、`doc_id`、slug、heading 或 index display title 是否受影响。
- successor/replacement references：读者或 Agent 是否能从旧入口找到当前入口；没有 successor 时必须说明 remaining access risk。
- review/completion/story evidence：如果它们支撑本次 decision，链接或路径是否仍可定位。

如果检查超出当前 story 授权，必须写 `deferred_with_reason` 并说明 owner/future story。发现 batch-shaped link/index work 时，停止并引用 `docs/governance/batch-readiness-checklist.md`，不得把批量修复塞进单个 revision。

## Validation checklist and maintenance triggers

执行本文相关工作或验证本文自身时，至少检查：

1. 任务类型、资产层级、allowed file scope、expected output、validation evidence 和 stop/escalation conditions 已声明。
2. Update mode 明确为 `targeted_revision`、`structural_upgrade`、`regeneration`、`split`、`merge` 或 `deprecation`。
3. 实质修订已复核 `updated_at`、`source_basis`、`time_context`、`quality_status`、`open_questions`、`prompt_version`、`template_version`、`related_docs`、body links、`docs/index.md`、lifecycle wording 和 specialized records。
4. Regeneration 已解释 why targeted revision is insufficient。
5. Regeneration 未 silent overwrite 旧资产、旧 `doc_id`、旧 path、review notes、prior failures、open questions、links 或 index evidence。
6. High-value content 已处理为 `preserved`、`moved`、`summarized`、`replaced`、`deleted_with_reason`、`archived` 或 `deferred`。
7. Revision / Regeneration Continuity Record 解释 changed what、why、what was preserved、what was removed、old-content handling 和 reference validity。
8. Identity continuity、old/new path/title/topic/doc_id、successor/replacement、link/index impact 和 lifecycle/status impact 不清时，使用 hold/defer/not_authorized/blocker，而不是继续写入。
9. Index impact 使用 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict`。
10. Decision labels 使用 `accept_promote`、`accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`targeted_revision`、`regenerate`、`reject_duplicate_or_misplaced`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason` 或 `not_authorized`。
11. Lifecycle vocabulary 使用 `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` 和 reactivation semantics。
12. 没有新增 unauthorized frontmatter fields。
13. 没有执行未授权 regeneration、split、merge、deprecation、batch migration、batch link update、bulk status conversion、old document upgrade 或 generated continuity records。
14. 没有创建 runtime code、package manifest、source/test directories、software test framework、schemas、validators、generators、automation、CLI/API/UI/database/deployment/CI、Epic 5 link/network assets 或 Epic 6 batch assets。

维护触发点：

- `docs/governance/sidecar-boundary-policy.md` 已建立 sidecar boundary；后续若该政策更新，复核 supplemental notes、revision notes、sidecar references 与 continuity record 的边界。
- `docs/governance/legacy-migration-guide.md` 已建立；后续若该指南更新，复核 structural upgrade、legacy migration、old-version compatibility 和 migration guide references。
- Story 5.1 related-doc taxonomy、Story 5.2 link maintenance、Story 5.3 reusable model entry points 或 Story 5.4 existing-doc reuse procedure 更新后，复核 reference validity、successor link、supporting link 和 related_docs impact。
- Epic 6 建立 batch governance runbook、batch change review record 或 batch completion report template 后，复核 batch regeneration / batch migration continuity records 的落点。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、link scanner、index generator、migration script、new global frontmatter fields 或 batch normalization 后，复核本文非软件边界和字段边界。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响](./rename-migration-policy.md)
- [Prompt、模板与质量规则版本治理：规则演进、字段语义与渐进迁移](./prompt-template-quality-version-governance.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements](./document-decision-policy.md)
- [失败案例与返工闭环：failure categories、repair instructions、regeneration rationale 与 resubmission checks](./rework-loop-examples.md)
- [完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界](../templates/completion-report-template.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](../templates/review-record-template.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁](./candidate-promotion-checklist.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [Sidecar 与补充材料边界政策：主文档权威、支持资产分类与过期处理](./sidecar-boundary-policy.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
