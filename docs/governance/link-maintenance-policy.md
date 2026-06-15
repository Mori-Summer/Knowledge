---
doc_id: governance-link-maintenance-policy
title: 跨文档链接维护政策：existence/path/topic/meaning checks、inbound/outbound review、one-way reason 与 boundary conflict handling
concept: link_maintenance_policy
topic: governance
depth_mode: standard
created_at: '2026-05-29T14:00:16+08:00'
updated_at: '2026-06-15T16:56:49+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/5-2-link-maintenance-policy.md
  - _bmad-output/implementation-artifacts/5-1-related-docs-taxonomy.md
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/intake-and-intent-classification.md
  - _bmad-output/implementation-artifacts/deferred-work.md
time_context: stabilization_deferred_triage_2026_06_15
applicability: formal_docs_cross_document_link_maintenance_and_inbound_outbound_review_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/intake-and-intent-classification.md
open_questions:
  - network boundary and decay prevention policy 已建立后，是否需要继续把 dense/noisy links、stale links 和 weak links 映射到更专门的 Epic 6 batch network review records？
  - Epic 6 建立 batch governance runbook and records 后，是否需要为 batch link normalization / batch related_docs update 提供专门 execution record？
---

# 跨文档链接维护政策：existence/path/topic/meaning checks、inbound/outbound review、one-way reason 与 boundary conflict handling

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 formal `docs/` 资产维护正文内部链接、frontmatter `related_docs`、入链复核、出链复核、单向链接理由和 definition/boundary conflict evidence 的 canonical governance asset。它回答的是：当前变更触及链接时，Agent 或 reviewer 必须检查哪些目标、如何判断链接是否有意义、何时需要复核入链/出链、如何记录冲突，以及什么时候必须停止进入 batch readiness 或 future story。

本文适用于：

- 正式 `docs/` 资产的 changed-file body links、frontmatter `related_docs`、source/support links、successor/replacement references 和 index links。
- 新建、升级、修订、重命名、移动、合并、拆分、废弃、归档、retitle/topic change、canonical entry change 或 successor/replacement switch 时的 link impact evidence。
- review evidence、completion evidence、story Dev Agent Record、target body、明确命名的 link maintenance note 或 future runbook record 中的人类可读链接维护记录。
- 单文件或窄范围变更中的 inbound/outbound link review scope 说明。

本文不适用于：

- 执行 full-repo link scan、all-doc inbound scan、all-doc outbound scan、stale-link cleanup、network health review 或 index-wide cleanup。
- 批量修复正文链接、批量更新 `related_docs`、批量 relationship assignment、batch link normalization 或 batch generated reports。
- 定义 relationship taxonomy、frontmatter schema、rename/migration identity rules、duplicate/coexistence decisions、existing-doc reuse procedure、reusable model entry points 或 network decay prevention；这些由相邻治理资产负责。
- 新增 `link_status`、`inbound_links`、`outbound_links`、`one_way_reason`、`relationship_status`、`link_policy_version`、`relation_type`、`owner_entry_point`、`successor`、`predecessor`、`schema_version`、`network_health` 或类似 frontmatter fields。
- 创建 runtime code、package manifest、`src/`、`tests/`、software tests、CLI/API/UI/database/deployment/CI、machine-readable schema、validator、generator、link scanner、index generator 或 automation。

本文补充而不替代以下资产：

- [related docs 与相邻概念关系分类](./related-docs-taxonomy.md)：负责 relation type、target status、transfer object、meaningful-link evidence 和 Related Docs Relationship Record。
- [可复用模型入口政策](./reusable-model-entry-points.md)：负责 `core_model`、`boundary_rule`、`decision_frame`、`failure_pattern`、`verification_method`、callable anchors 和 Reusable Model Entry Point Record。
- [既有文档复用流程](./existing-doc-reuse-procedure.md)：负责 new-problem scan surfaces、scan order、scan result taxonomy、reuse evidence 和 Existing Doc Reuse Scan Record。
- [知识网络主题边界与退化防护政策](./network-boundary-and-decay-prevention.md)：负责 topic health、dense/noisy/weak/stale link signals、repeated overlap routing 和 navigation relevance evidence。
- [docs/index.md 同步与导航治理规则](./index-synchronization-rules.md)：负责 Index Impact outcomes、relative path、formal navigation entry 和 index-wide cleanup 边界。
- [Frontmatter schema 与 doc_id 身份规则](./frontmatter-schema.md)：负责 baseline fields、YAML arrays、stable `doc_id` 和 unauthorized field 禁令。
- [重命名、路径迁移与废弃治理](./rename-migration-policy.md)：负责 rename/move/merge/split/deprecation/archive、successor/replacement、remaining access、identity continuity 和 Migration Decision Record。
- [重复概念与同主题共存治理](./duplicate-and-coexistence-policy.md)：负责 adjacent_link、duplicate/coexistence、narrower document、same-topic coexistence 和 conflict routing。
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)：负责 owner entry point、navigation treatment 和 governance asset navigation boundaries。
- [批量治理 Readiness Checklist](./batch-readiness-checklist.md)：负责 rule-selected target set、batch link/index update、batch `related_docs` update、stop criteria 和 recovery evidence。
- [修订、重生成与版本连续性策略](./revision-regeneration-continuity-policy.md)、[Sidecar 与补充材料边界政策](./sidecar-boundary-policy.md) 和 [旧文档渐进迁移与兼容策略](./legacy-migration-guide.md)：负责 reference validity、supporting asset、legacy/successor 和 remaining-gap boundaries。
- [审查记录模板](../templates/review-record-template.md)、[完成汇报模板](../templates/completion-report-template.md) 和 [文档决策政策](./document-decision-policy.md)：负责 review/completion/decision evidence 的承载位置和状态词。
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)、[来源纪律与真实世界锚点政策](../methodology/source-discipline-and-real-world-anchor-policy.md) 和 [输入摄入与任务意图判定](../methodology/intake-and-intent-classification.md)：负责质量证据、source/time discipline、任务分类和 defer-to-future-story handling。

本文的 owner entry point 是 [docs/index.md](../index.md) 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/link-maintenance-policy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Epic 5 首版 link maintenance policy；reusable model entry points policy、existing-doc reuse procedure 和 network boundary / decay prevention policy 已落地，Epic 6 后续仍会细化 batch link governance。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record
- source asset: docs/governance/link-maintenance-policy.md
- trigger: Story 5.2 explicitly authorizes a canonical link maintenance policy governance asset
- index file: docs/index.md
- section: ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: the asset is a formal governance policy with listed_in_docs_index navigation treatment
- validation result: target exists and relative link resolves
- unresolved risk: reusable model entry points policy, existing-doc reuse procedure and network boundary / decay prevention policy are now direct adjacent authorities; Epic 6 integrations remain future dependencies, recorded in open_questions
```

## Link Target And Meaning Checks

每个 changed-file internal link 和 frontmatter `related_docs` entry 至少必须检查 target existence、relative path、target asset class、topic/path fit 和 relationship meaning。若检查结果无法成立，不能把链接静默包装成普通可用关系。

| Check | Required meaning | Required evidence |
| --- | --- | --- |
| `existence` | Target exists, or missing/planned/unresolved status is visible. | Existing target path, planned owner/future story, or unresolved open question. |
| `path` | Relative link path resolves from the source location. Path spelling must match repository path casing exactly, even on case-insensitive local filesystems. | Source path, target path, relative path expectation, exact filename/directory casing, fragment/anchor handling if relevant. |
| `target_asset_class` | Target is an appropriate formal docs asset, template, index, report, sidecar/support asset, or intentionally planned future dependency. | Asset class, path group, owner entry, formal/candidate/support distinction. |
| `topic_fit` | Link target belongs to an appropriate topic/asset class for the relationship. | Topic/path/name/index relationship and catch-all risk. |
| `relationship_meaning` | Link has relation type and transfer object from [related docs taxonomy](./related-docs-taxonomy.md). | Relation type, transfer object, boundary/reuse meaning, or why unresolved. |
| `definition_boundary_consistency` | Source and target do not contradict definitions, boundaries, or scope without record. | Affected docs and proposed resolution. |
| `direction` | Link direction is one-way or bidirectional with reason. | One-way reason or bidirectional evidence. |

Path validation is case-sensitive as a governance rule. A link that works only because the local filesystem ignores case is not considered validated; record it as `path_case_mismatch` and patch the link spelling or defer with an owner. Directory names, file basenames and extensions must match the repository entry exactly. A casing mismatch makes `path result: fail`; `path casing result` is the diagnostic field that names the casing-specific cause.

Fragment or anchor links are conservative. Use a `#fragment` only when the target heading or explicit anchor is visible in the existing target file and the reviewer can name the exact target section. If the heading text is duplicated, mixed-language slug generation is unclear, punctuation-heavy, renderer-dependent or likely to change during retitle/rewrite, prefer a file-level link with section wording in the sentence, or record `fragment_anchor_ambiguity` with a proposed resolution. Fragment validation is a human-readable Markdown check, not a generated anchor scan.

`docs/governance/related-docs-taxonomy.md` remains the authority for relation type, target status, transfer object and meaningful-link evidence. This policy calls that taxonomy; it does not redefine it. Relationship types such as `prerequisite`, `adjacent_concept`, `contrast_concept`, `deeper_dive`, `shared_mechanism`, `example_application`, `successor_replacement` and tightly justified `approved_equivalent` must keep their taxonomy meanings.

Target status handling here is an application of [related docs taxonomy](./related-docs-taxonomy.md), not a replacement definition. Use the taxonomy as the authority; this policy only states how those statuses affect link maintenance evidence:

- `existing`: target exists and can be checked for path, topic fit, asset class and relationship meaning.
- `intentionally_planned`: target does not yet exist, but planning artifact, story or owner decision explicitly authorizes it as a future dependency. It may be recorded in `open_questions`, body evidence or follow-up notes, but wording must not imply the asset is currently available.
- `unresolved`: target is missing, path/owner is unclear, or relationship cannot be verified. It must go to `open_questions`, review notes, completion evidence, Dev Agent Record or follow-up list. It must not be silently added as ordinary `related_docs` or as a body link presented as usable.
- `deprecated_or_archived`: target exists but is not a normal current target. It may be linked only as successor/replacement, historical context, remaining access, migration audit or explicit legacy support, with lifecycle/migration evidence visible.

Different link locations carry different meanings:

| Link location | What it means | What this policy checks |
| --- | --- | --- |
| `related_docs` | Formal frontmatter relationship signal. | Existing/planned/unresolved target status, relation type, transfer object, path/topic fit and relationship meaning. |
| body link | Contextual reader path, rule reference, example, evidence, successor note or explanation. | Relative path, link text, target fit, local meaning and whether it should also affect `related_docs`. |
| source reference | Source or basis for a claim. | Whether it belongs in `source_basis`/body rather than `related_docs`; source/time discipline remains separate. |
| supporting sidecar | Supplemental evidence or support material. | Whether target is support rather than canonical relationship; sidecar authority remains separate. |
| owner entry | Discovery or navigation entry point. | Whether link is navigation, not relationship; owner entry evidence remains separate. |
| index entry | Formal navigation link. | Relative path and display title consistency; index outcome remains owned by index synchronization rules. |
| successor/replacement link | Lifecycle, remaining access or replacement path. | Link impact evidence; identity/lifecycle decisions remain owned by rename/migration and continuity policies. |
| review/completion evidence | Human-readable record of checks, decisions and unresolved risks. | Whether unresolved/planned/conflict states are visible and not overclaimed as resolved. |

Planned targets are allowed only when a planning artifact, story or owner decision authorizes them. A planned target may be named as future dependency, but body text, index entries and completion reports must not say or imply that the target exists, has been reviewed, or is ready for normal navigation.

## Outbound Link Review

Outbound link review examines links from the changed file or changed asset outward to targets it references. It is required whenever the current change adds, removes, changes, retitles, reclassifies or relies on:

- body links;
- frontmatter `related_docs`;
- successor/replacement references;
- support/supporting links;
- source references that may be confused with relationship links;
- owner-entry or index links;
- completion/review evidence links.

For each outbound link in the changed-file scope, check:

1. Source location: file path, section, frontmatter field or record location.
2. Target path: relative path from source and whether fragment/anchor is used.
3. Target status: existing, intentionally planned, unresolved or deprecated/archived.
4. Target asset class: formal docs asset, governance asset, template, report, support/sidecar, source reference, owner entry or future dependency.
5. Topic/path fit: whether target topic and path match the relationship.
6. Relationship meaning: relation type and transfer object from related docs taxonomy.
7. Direction: one-way or bidirectional with reason.
8. Related-doc impact: whether a body link should, should not or cannot become `related_docs`.
9. Index/lifecycle impact: whether navigation, successor, replacement, deprecation or archive semantics are involved.
10. Conflict state: whether definition, boundary, source/time, quality/status, duplicate/coexistence or index/navigation conflict exists.

Outbound review is normally narrow. It covers current changed files and directly affected links. It does not authorize a whole-topic sweep, all-doc link inventory, all-doc `related_docs` normalization, stale-link cleanup or generated report.

## Inbound Link Review

Inbound link review examines discoverable references from other assets into the target affected by the change. It is required when a target undergoes any of these triggers:

In normal story work, inbound review is bounded to the authorized narrow scope: current changed files, affected `docs/index.md` or owner-entry lines, and adjacent files explicitly named by the story, reviewer or implementation notes. Searching beyond those named files is not part of narrow review. If the only way to gain confidence is an all-doc search, generated inventory, rule-selected target set or topic sweep, record `deferred_with_reason` or route to [batch readiness](./batch-readiness-checklist.md) before making completion claims.

| Trigger | Inbound review requirement | Boundary owner |
| --- | --- | --- |
| rename | In the authorized narrow scope, check references to old path, old title, old slug, old H1 or old `doc_id` and record scope. | Identity/path migration remains under [rename/migration policy](./rename-migration-policy.md). |
| move | Check old path, new path, index entry and directly discoverable references. | Path ownership and index treatment remain under topic/path and index rules. |
| merge | Check references to merged assets and record successor/replacement or remaining access expectation. | Merge identity and old-content treatment remain under rename/migration and continuity policies. |
| split | Check references to old sections, new targets and any ambiguous old link. | Split identity and old-content disposition remain under rename/migration and continuity policies. |
| deprecation | Check references that still present deprecated target as current primary entry. | Lifecycle and remaining access remain under lifecycle and rename/migration policies. |
| archive | Check references that would hide or misrepresent remaining access. | Archive visibility and retrieval expectation remain under lifecycle and rename/migration policies. |
| successor/replacement switch | Check references that point to old successor, missing successor or conflicting replacement. | Successor/replacement semantics remain under rename/migration and continuity policies. |
| retitle/topic change | Check references whose link text, index display or relationship type becomes misleading. | Topic/path and index semantics remain under their policies. |
| canonical entry change | Check owner/index/review references that might still route users to the wrong entry. | Owner entry and index semantics remain under navigation/index policies. |

Inbound review scope must be explicit. A valid narrow scope can be: “checked current changed files plus direct references found by target path/title in the named adjacent files.” If a reviewer cannot reasonably discover references without all-doc search, generated inventories or rule-selected target sets, record `deferred_with_reason` or route to [batch readiness](./batch-readiness-checklist.md).

Inbound review does not mean every possible inbound reference in the repository was checked. Completion evidence must not say “all inbound links are fixed” unless that full scope was separately authorized and actually performed.

## Rename, Migration And Lifecycle Link Impact

Rename, move, merge, split, deprecation and archive decisions affect links, but this policy does not decide identity, old-content handling, successor/replacement, lifecycle state or index treatment. Those remain owned by [rename/migration policy](./rename-migration-policy.md), [revision/regeneration continuity policy](./revision-regeneration-continuity-policy.md), [lifecycle states](./lifecycle-states.md) and [index synchronization rules](./index-synchronization-rules.md).

This policy requires link review evidence around those decisions:

- Outbound links from changed assets must be checked for path, topic fit, relationship meaning and direction.
- Discoverable inbound references must be checked within a named scope when the target identity/path/title/topic/lifecycle/entry point changes.
- Successor/replacement links must state whether the relationship is current, historical, remaining access, replacement, migration audit or unresolved.
- Deprecated or archived targets must not be presented as current normal targets unless the responsible lifecycle/migration evidence allows that treatment.
- Old paths, old titles, old headings and old slugs that remain discoverable must either resolve, redirect through human-readable successor evidence, or be recorded as unresolved/deferred/not authorized.

If link impact reveals duplicate/coexistence ambiguity, same-topic overlap or adjacent-link uncertainty, route that decision to [duplicate/coexistence policy](./duplicate-and-coexistence-policy.md). Link maintenance can record the impact and proposed resolution; it must not silently decide merge/split/deprecate/archive or same-topic coexistence.

## Narrow Review Versus Batch-Shaped Review

Single-file or narrow review is allowed when:

- the changed files are named;
- target links are directly in those changed files or adjacent files explicitly named by the story;
- the review can be completed by checking visible body links, `related_docs`, target paths and directly discoverable references;
- no rule-selected target set is needed;
- no all-topic sweep, index-wide cleanup or all-doc inbound scan is required.

Batch-shaped review starts when any of these are true:

- target files are selected by rule instead of being explicitly named;
- multiple topics or many unrelated assets are affected;
- the work requires all-doc inbound scan, full topic sweep, index-wide cleanup, batch `related_docs` normalization or stale-link cleanup;
- the work needs a generated inventory, link scanner, index generator, validation script or automation;
- recovery, sampling, rollback, owner decision or batch completion evidence is required.

When review becomes batch-shaped, stop the single-story link maintenance work and route to [batch readiness](./batch-readiness-checklist.md) or future Epic 6. Do not complete a narrow story by partially doing batch work and describing it as complete.

## Definition And Boundary Conflict Handling

If a source and target conflict in definition, boundary, topic fit, concept identity, scope, successor/replacement, quality/status or currentness, the conflict must be recorded with affected documents and proposed resolution. A conflict record is human-readable evidence. It is not a schema, lint rule, generated report or frontmatter extension.

Conflict categories:

| Category | Meaning | Evidence to record |
| --- | --- | --- |
| `definition_mismatch` | Source and target define the same term or concept differently. | Affected docs, conflicting definitions, preferred authority or hold condition. |
| `boundary_mismatch` | Inclusion/exclusion rules or scope boundaries disagree. | Boundary statements, affected use case, proposed clarification. |
| `topic_path_mismatch` | Link target path/topic does not fit the relationship. | Source path, target path, expected topic, proposed path or removal. |
| `relationship_type_mismatch` | Relation type or transfer object is wrong or unsupported. | Current relation, expected relation, taxonomy reference. |
| `stale_successor_replacement` | Link points to old, missing or conflicting successor/replacement target. | Old target, proposed current target, migration/lifecycle evidence needed. |
| `duplicate_coexistence_ambiguity` | Link may be hiding duplicate, merge, split, adjacent or narrower-document decision. | Candidate overlap, suspected decision type, duplicate/coexistence owner. |
| `source_time_conflict` | Source/time/currentness claims disagree or are unverifiable. | Source basis, time context, affected claims, source discipline action. |
| `quality_status_mismatch` | Target status or quality is overclaimed. | Target `quality_status`, body wording, completion/review claim. |
| `missing_target` | Target path or planned asset does not exist or is not authorized. | Missing path, owner/future story if any, unresolved state. |
| `path_case_mismatch` | Link target exists only by case-insensitive filesystem behavior or differs from repository path casing. | Source link, exact repository path, corrected spelling or deferral owner. |
| `fragment_anchor_ambiguity` | Link fragment, heading or anchor is missing, unstable or ambiguous. | Source link, target heading, proposed link text or anchor handling. |
| `index_navigation_mismatch` | Index/owner entry/link text routes to the wrong asset or wrong status. | Index entry, owner entry, target title/path/status, index impact action. |

Each conflict record must include a proposed resolution. Allowed resolution types include:

- `targeted_revision`;
- `relationship_note`;
- `related_docs_removal`;
- `link_text_clarification`;
- `open_question`;
- `duplicate_coexistence_decision`;
- `rename_migration_decision`;
- `hold_for_maxwell`;
- `defer_to_future_story`;
- `batch_readiness`;
- `not_authorized`.

Unresolved conflicts must not be described as resolved in completion wording. Use `未验证`, `deferred_with_reason`, `hold_for_clarification`, `not_authorized` or an equivalent current policy status that makes the unresolved state visible.

## One-Way Link Rules

Bidirectional links are not automatic. A bidirectional relationship is allowed only when both sides have meaningful relationship evidence, both targets exist, path/topic fit is clear, and the reciprocal link will not create noisy network edges.

One-way links are allowed when the reason is clear. Common valid cases:

- Source needs target, but target does not need source.
- Target is only an example, application, counterexample or test case of the source model.
- Successor/remaining access relation only needs one direction to preserve lifecycle clarity.
- Supporting sidecar or source reference is not a symmetric conceptual relationship.
- Index/owner entry is navigation, not a relationship.
- Source cites a governance rule as prerequisite, but the governance rule should not list every consumer.

One-way link reason must state at least:

1. direction: source to target, target to source, or one-way-with-reason;
2. why reciprocal link is not needed;
3. transfer object from related docs taxonomy;
4. whether frontmatter `related_docs` should change;
5. whether body link wording should change;
6. whether index, successor/replacement or lifecycle impact exists;
7. whether the non-reciprocal relation creates unresolved review risk.

One-way reason must not be expressed through new frontmatter fields. Put it in body text, Related Docs Relationship Record, Link Maintenance Record, review evidence, completion evidence, story Dev Agent Record or a clearly named link maintenance note.

## Link Maintenance Record

`Link Maintenance Record` is a copyable human-readable record for link review evidence. It is not a schema, not a lint rule, not a frontmatter field set, not a generator input and not an automation contract.

```markdown
## Link Maintenance Record

- source asset:
- changed file / trigger:
- target asset:
- link location: body link | frontmatter related_docs | index entry | successor/replacement note | support/sidecar link | review evidence | completion evidence | other
- target status: existing | intentionally_planned | unresolved | deprecated_or_archived
- relation type:
- direction: source_to_target | target_to_source | bidirectional | one_way_with_reason
- one-way reason:
- transfer object:
- existence result: pass | fail | planned | unresolved | not_applicable | unverified
- path result: pass | fail | not_applicable | unverified
- path casing result: pass | fail | not_applicable | unverified
- topic fit result: pass | fail | not_applicable | unverified
- definition/boundary consistency:
- related_docs impact:
- body link impact:
- index impact:
- lifecycle or successor impact:
- duplicate/coexistence impact:
- source/time impact:
- sidecar/support impact:
- inbound review scope:
- outbound review scope:
- conflict found:
- proposed resolution:
- owner/future story:
- validation result: pass | fail | not_applicable | unverified
```

The record can live in target body, review evidence, completion evidence, story Dev Agent Record, future runbook record or a clearly named link maintenance note. Record fields must not be split into unauthorized frontmatter fields.

Use this record when a change needs reviewer-visible evidence but does not justify creating a separate report. Do not use it to imply full-repo review unless full-repo scope was authorized and completed.

## Adjacent Governance Asset Boundaries

This policy relies on adjacent governance assets instead of replacing them:

| Adjacent asset | This policy uses it for | Boundary |
| --- | --- | --- |
| [related docs taxonomy](./related-docs-taxonomy.md) | relation type, target status, transfer object, meaningful-link evidence and Relationship Record. | This policy does not redefine taxonomy. |
| [reusable model entry points policy](./reusable-model-entry-points.md) | `core_model`, `boundary_rule`, `decision_frame`, `failure_pattern`, `verification_method`, callable anchors and Reusable Model Entry Point Record. | This policy checks links to anchors but does not define reusable entry point semantics. |
| [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) | new-problem scan surfaces, scan result taxonomy, reused-context evidence and Existing Doc Reuse Scan Record. | This policy checks changed-file links that reuse scan depends on, but does not perform reuse scans. |
| [index synchronization rules](./index-synchronization-rules.md) | Index Impact outcomes, relative path, formal navigation entry and index-wide cleanup boundary. | This policy records link/index impact evidence; index outcomes remain owned there. |
| [rename/migration policy](./rename-migration-policy.md) | rename/move/merge/split/deprecation/archive, successor/replacement, remaining access and Migration Decision Record. | This policy records link review evidence only. |
| [duplicate/coexistence policy](./duplicate-and-coexistence-policy.md) | adjacent_link, duplicate/coexistence, narrower document, same-topic coexistence and conflict routing. | This policy records link impact after the duplicate/coexistence outcome is known. |
| [frontmatter schema](./frontmatter-schema.md) | baseline fields, YAML arrays, stable `doc_id` and unauthorized field prohibition. | This policy does not add fields. |
| [batch readiness checklist](./batch-readiness-checklist.md) | batch link/index update, rule-selected target set, batch `related_docs` update, stop criteria and recovery evidence. | This policy stops when work becomes batch-shaped. |
| [review record template](../templates/review-record-template.md), [completion report template](../templates/completion-report-template.md), [document decision policy](./document-decision-policy.md) | review/completion/decision evidence. | This policy does not create actual review or completion reports. |
| [revision/regeneration continuity](./revision-regeneration-continuity-policy.md), [sidecar boundary](./sidecar-boundary-policy.md), [legacy migration guide](./legacy-migration-guide.md) | reference validity, supporting asset boundaries, legacy/successor and remaining-gap boundaries. | This policy records link impact without performing migration, sidecar extraction or legacy cleanup. |

Future story boundaries:

- [reusable model entry points policy](./reusable-model-entry-points.md) owns reusable model entry point classification and callable model anchors.
- [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) owns existing-doc scan procedure and exact/adjacent/prereq/contrast/no-asset results.
- [network boundary and decay prevention policy](./network-boundary-and-decay-prevention.md) owns identification and routing of dense/noisy/weak/stale link signals, topic health, repeated overlap and navigation relevance evidence; actual cleanup remains bounded link maintenance, batch readiness or Epic 6 work.
- Epic 6 owns batch link normalization, batch `related_docs` updates, batch review/completion records and batch execution runbooks.

## Narrow Update And Completion Rules

When applying this policy in a normal story:

1. Name the changed files and link triggers.
2. Check body links, `related_docs`, successor/replacement references, support links and index links only within authorized scope.
3. Check exact path casing for directories, basenames and extensions.
4. For missing/planned targets, record planned owner/future story or unresolved state.
5. For definition/boundary conflict, record affected documents and proposed resolution.
6. For one-way links, record direction and reason.
7. For inbound review, state the discoverable reference scope actually checked.
8. If the work becomes batch-shaped, stop and route to batch readiness.
9. Do not create new frontmatter fields, schema, validator, generator, scanner or automation.
10. Do not describe unresolved, unverified, deferred or not-authorized links as resolved.

Completion evidence must distinguish:

- `pass`: checked and satisfied inside authorized scope;
- `fail`: checked and found invalid;
- `planned`: target intentionally planned by story/planning/owner;
- `unresolved`: missing or ambiguous without resolution;
- `not_applicable`: link dimension does not apply;
- `unverified`: not checked and not counted as complete.

## Validation Checklist

Validate this policy or any future application of it with the following checklist:

1. Target file exists at `docs/governance/link-maintenance-policy.md`.
2. Target frontmatter contains all required baseline fields.
3. `source_basis`, `related_docs` and `open_questions` are YAML arrays.
4. H1, frontmatter title and `docs/index.md` title are consistent.
5. Link target checks cover existence, path, exact path casing, target asset class, topic fit, relationship meaning, definition/boundary consistency and direction.
6. Inbound and outbound review triggers are explicit.
7. Rename/move/merge/split/deprecation/archive link review calls rename/migration and continuity policies for identity/lifecycle decisions.
8. Definition/boundary conflict handling records affected documents and proposed resolution.
9. One-way links require explicit direction, reason, transfer object and reciprocal-link decision.
10. Link Maintenance Record exists and is explicitly non-schema, non-frontmatter and non-automation.
11. Reusable model entry points policy, existing-doc reuse procedure, network boundary and decay prevention policy and Epic 6 boundaries are explicit.
12. Body links, frontmatter `related_docs` and `docs/index.md` entry are path-resolvable for existing files.
13. Planned/missing targets are recorded in `open_questions`, future dependency notes, review evidence, completion evidence or Dev Agent Record.
14. No unauthorized frontmatter fields are introduced.
15. No full-repo link scan, inbound/outbound repair, batch `related_docs` update, index-wide cleanup, legacy migration, sidecar extraction, existing-doc reuse scan, network health review or stale-link cleanup is performed.
16. No runtime code, package manifest, source/test directories, software test framework, schemas, validators, generators, automation, link scanners, index generators, CLI/API/UI/database/deployment/CI, runbooks, actual review records or actual completion reports are created.
