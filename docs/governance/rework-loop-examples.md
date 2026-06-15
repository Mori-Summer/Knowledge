---
doc_id: governance-rework-loop-examples
title: 失败案例与返工闭环：failure categories、repair instructions、regeneration rationale 与 resubmission checks
concept: rework_loop_examples
topic: governance
depth_mode: standard
created_at: '2026-05-27T11:48:11+08:00'
updated_at: '2026-05-28T10:04:11+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/3-1-review-record-template.md
  - _bmad-output/implementation-artifacts/3-2-document-decision-policy.md
  - _bmad-output/implementation-artifacts/3-3-rework-loop-examples.md
  - _bmad-output/implementation-artifacts/3-4-completion-report-template.md
  - docs/index.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
time_context: phase_4_epic_4_revision_regeneration_continuity_policy_2026_05_28
applicability: formal_docs_rework_loop_review_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
open_questions:
  - 如果 completion-report template 后续调整 rework/prior failure summary 字段，是否需要同步本文的 rework-loop result 表述？
  - Story 4.1 已建立 revision/regeneration continuity policy；后续若 Continuity Record 字段变化，是否需要同步本文 regeneration examples 的 required records？
  - Epic 5 建立 related-doc taxonomy 与 link maintenance policy 后，是否需要细分 link/index/related-doc rework examples？
  - Epic 6 建立 batch governance runbook 与 batch records 后，是否需要增加批量返工闭环示例？
---

# 失败案例与返工闭环：failure categories、repair instructions、regeneration rationale 与 resubmission checks

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 failure classification、rework-loop examples、regeneration rationale examples 和 resubmission checks 的 canonical governance asset。它把 review record、quality gate、source discipline、frontmatter schema、index synchronization、migration、duplicate/coexistence、lifecycle 和 document decision policy 中的失败信号，转成可定位、可执行、可复查的返工闭环。

本文适用于：

- 正式 `docs/` 资产的新建、升级、审查、返工、重新提交和复审。
- 候选资产准备 promotion、replacement、merge、split、deprecation、archive 或 index/navigation 更新时的返工记录。
- `docs/methodology/`、`docs/governance/`、`docs/templates/`、未来 `docs/runbooks/`、`docs/index.md` 和正式 report entry 的等价治理检查。
- review record、decision evidence、story Dev Agent Record、completion evidence 或未来 runbook record 中需要说明失败、修复、重生成和复审结果的场景。

本文补充而不替代以下资产：

| 资产 | 本文如何使用 |
| --- | --- |
| Story 3.1 review-record template | 使用其中的 task classification、Hard Fail/blocker evidence、score/equivalent governance checks、`未验证`/`不适用` register、specialized records 和 final decision 位置。 |
| Story 3.2 document-decision policy | 使用其 final decision labels、blocking/status values、status wording 和 next action 语义。本文不重新定义 decision policy。 |
| Epic 0 foundation | 使用 Agent 行为约束、生命周期、版本治理、质量基线、导航基线和 batch readiness 的边界。 |
| Epic 1 methodology | 使用 concept document contract、quality gate、example catalog、source discipline、main methodology、template 和 fixed prompt 作为失败与修复证据来源。 |
| Epic 2 repository governance | 使用 frontmatter、topic/path、promotion、index synchronization、rename/migration 和 duplicate/coexistence 作为专门记录来源。 |

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/rework-loop-examples.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Story 3.3 首版 rework-loop examples asset；Story 3.4 已建立 completion-report template，Story 4.1 已建立 revision/regeneration continuity policy，Epic 5 和 Epic 6 后续仍会细化 link maintenance 和 batch records。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/rework-loop-examples.md
- target title: 失败案例与返工闭环：failure categories、repair instructions、regeneration rationale 与 resubmission checks
- target path: ./governance/rework-loop-examples.md
- target section: docs/index.md ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: Story 3.3 explicitly authorizes the canonical rework-loop examples governance asset
- validation result: target exists and relative link resolves
- unresolved risk: completion-report integration is present; Story 4.2 sidecar, Story 4.3 legacy migration, Epic 5 link taxonomy and Epic 6 batch integrations remain open questions
```

## 职责边界与非目标

本文提供 failure categories、repair instruction requirements、targeted revision examples、regeneration rationale examples、prior failure records 和 resubmission checks。它不替代 reviewer 的实际审查，不自动生成 review records，也不允许用示例代替目标资产的具体证据。

本文不实现以下范围：

- Story 3.1 review-record template 的字段权威。
- Story 3.2 document-decision policy 的 final decision 语义。
- `docs/templates/completion-report-template.md`。
- Story 4.1 revision/regeneration continuity policy 的 Continuity Record 字段；Story 4.2 sidecar boundary 或 Story 4.3 legacy migration guide。
- Epic 5 related-doc taxonomy、link maintenance、reusable model entry points 或 existing-doc reuse procedure。
- Epic 6 batch governance runbook、batch change review record 或 batch completion report。
- 既有正式资产的实际 review records、批量 review records、generated rework records、completion reports、scoring reports 或 bulk status migrations。
- executable tooling、machine-readable schema、JSON/YAML schema、lint/scoring tool、validator、rework generator、decision generator、review-record generator、CLI/API/UI/database/deployment/CI、package manifest、`src/` 或 `tests/`。

本文不授权在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`decision_status`、`review_record_version`、`rework_record_version`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`quality_gate_version`、`promotion_policy_version`、`migration_status`、`duplicate_status`、`coexistence_status`、`successor`、`canonical_asset` 或类似字段。返工、决策、导航、迁移和共存证据应写在 review evidence、completion evidence、story Dev Agent Record、目标正文中的合规记录、Epic 3 review/completion templates 或 future runbook records 中。

## Failure classification taxonomy

每个失败项或返工需求必须先分类。分类不是标签装饰；它决定 evidence location、required fix、decision label、specialized records 和 resubmission check。

| Failure class | Definition | Common signals | Required evidence locations | Likely decision labels | Repair instruction requirements | Specialized-record references |
| --- | --- | --- | --- | --- | --- | --- |
| `repository_integration` | Path、filename、`doc_id`、frontmatter shape、`related_docs`、links、index entry、owner entry point 或 formal/candidate boundary 破坏仓库纪律。 | 错路径、缺 `doc_id`、YAML array 写成字符串、索引漏项、正文链接断裂、把 `_bmad-output/` 当正式资产。 | target path、YAML frontmatter、`docs/index.md` entry、body link、`related_docs`、Promotion/Index Impact evidence。 | `targeted_revision`、`hold_for_clarification`、`defer_with_reason`、`not_authorized`。 | 指明具体字段、路径、链接或索引条目；说明应补、改、移除或等待 owner decision。 | Promotion Decision Record、Index Impact Decision Record、frontmatter schema evidence。 |
| `structure_completeness` | Required sections、document type structure、review-record fields、governance asset role/scope 或 concept-document information points 缺失。 | H1/title 不清、缺 role/scope、缺 required section、review skeleton 漏字段、概念文档缺边界/机制/验证。 | H1、headings、required sections、review tables、template skeleton、body examples。 | `targeted_revision`、`regenerate`、`hold_for_clarification`。 | 指明缺失 heading 或 table field；说明新增内容的最小范围和不替代关系。 | Review-record template、quality gate、asset boundary policy。 |
| `model_discrimination_quality` | 文档有文字但不能支持分类、诊断、机制理解、相邻概念判别、tradeoff reasoning、failure-mode handling、verification 或 migration。 | 只有定义复述、相邻概念清单无差异、机制链断裂、没有反例/误读纠偏、自测只能背术语。 | boundary section、mechanism/model section、discrimination table、tradeoff/failure-mode section、self-test、transfer section。 | `targeted_revision`、`regenerate`、`accepted_with_nonblocking_follow_up`。 | 定位到具体模型/判别/失败模式/验证 section；说明要增加哪类判断链或例子。 | Quality gate、concept document example catalog。 |
| `evidence_time_context` | Source basis、current-practice claim、historical/deprecated framing、external fact、time context 或 unverifiable claim 无法支撑结论。 | “当前/主流/现行”无来源，历史实践冒充当前推荐，`source_basis` 不能支撑正文 claim，未验证声明支撑通过结论。 | `source_basis`、`time_context`、source claim paragraph、citation/link note、open question、source-discipline evidence。 | `targeted_revision`、`regenerate`、`hold_for_clarification`、`defer_with_reason`。 | 指明具体 claim、来源缺口、核对日期或 open question；要求删除、收窄、标为 inference、补来源或停止澄清。 | Source discipline evidence、quality gate。 |
| `consistency` | Frontmatter、title、H1、body、index、status wording、related docs、source/time claims、review decision 或 story evidence 互相矛盾。 | H1 与 frontmatter title 不同对象，索引标题误导，decision 说 accepted 但 blocker 未解，story status 被写成 `quality_status`。 | frontmatter fields、H1、index entry、conclusion/status paragraph、review record、Dev Agent Record、File List。 | `targeted_revision`、`hold_for_clarification`、`defer_with_reason`。 | 指明冲突双方和要保留的权威来源；说明改哪一处，或暂停等待 owner decision。 | Document decision policy、lifecycle states、index synchronization rules。 |
| `duplicate_risk` | Candidate/target 与 existing asset 重叠、canonical path 错置、same-topic coexistence 不清，或 rejection/merge/link/narrowing decision 缺失。 | 近重复、同主题无法区分、候选路径错置、平行索引入口、旧内容被 silent overwrite。 | target path、existing asset paths、Duplicate / Coexistence Decision Record、topic/path evidence、index entry、related docs。 | `targeted_revision`、`reject_duplicate_or_misplaced`、`regenerate`、`hold_for_clarification`。 | 指明现有 canonical path、拟议 path、差异是否成立、应 merge/link/narrow/reject/coexist。 | Duplicate / Coexistence Decision Record、Migration Decision Record、Index Impact Decision Record。 |
| `lifecycle_issue` | `quality_status`、lifecycle vocabulary、deprecation/archive/successor/reactivation wording 或 BMad story status 混淆或缺证据。 | 把 `ready-for-dev` 写入正式 frontmatter，把 draft 说成 validated，废弃/归档无 successor/index evidence，archive 被当成删除。 | `quality_status`、lifecycle paragraph、document-decision evidence、migration record、index visibility、successor link。 | `targeted_revision`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason`、`not_authorized`。 | 指明状态词位置、证据缺口和允许状态；区分 `deprecated` 与 `archived`。 | Lifecycle states、document decision policy、Migration Decision Record、Index Impact Decision Record。 |

分类必须使用上表 canonical class keys。不得把旧 label 如 `contexted`、`blocked_by_hard_fail`、`needs_revision`、`hold_for_maxwell_confirmation` 或 `defer_to_future_story` 作为 Story 3.3 canonical output；若引用旧来源，必须映射到 Story 3.2 词汇。

## Evidence location and repair instruction rules

返工指令必须能让下一位 reviewer 直接定位失败项。禁止只写“补充来源”“完善结构”“修复链接”“提高质量”。

最低字段如下：

| 字段 | 必填内容 |
| --- | --- |
| failure id | 稳定人工编号，例如 `RF-001`。 |
| class | 七类 failure class 之一。 |
| evidence location | exact file、heading、section、frontmatter field、example、source claim、body link、`related_docs` entry、`docs/index.md` entry 或 specialized record field。 |
| why it blocks / does not block | 为什么阻塞或不阻塞当前 decision、status wording、index visibility 或 reuse scope。 |
| required fix | 必须补什么、改什么、删除什么、迁移什么、保留什么或请求谁确认。 |
| owner | Maxwell、current Agent、reviewer、future story、source owner 或 not_applicable。 |
| blocking status | `blocked`、`nonblocking`、`held`、`deferred_with_reason` 或 `not_authorized`。 |
| decision label | Story 3.2 允许的 final decision label。 |
| resubmission check | 复审时如何证明旧失败已 `resolved`、`still_blocked`、`changed_scope`、`deferred_with_reason`、`not_authorized` 或 `new_failure`。 |

坏例子与可接受例子：

| 不合格返工指令 | 为什么不合格 | 可接受改写 |
| --- | --- | --- |
| “补充来源。” | 没有 claim、字段、来源类型、时间语境或阻塞理由。 | `RF-004 evidence_time_context: docs/topic/example.md ## 当前实践, paragraph containing "现代系统通常..." lacks source and checked date. Required fix: remove universal claim or replace with source-limited claim and add source/time limitation to source_basis/time_context/open_questions as appropriate.` |
| “完善结构。” | 没有指明缺哪个结构位点。 | `RF-002 structure_completeness: docs/methodology/example.md missing "职责边界与非目标" section. Required fix: add section after "资产角色..." stating what this asset does not authorize and which future stories own adjacent scope.` |
| “修一下索引。” | 不知道是路径、标题、分组、重复入口还是不适用。 | `RF-006 repository_integration: docs/index.md ## governance lacks ./governance/example-policy.md entry after new formal governance asset creation. Required fix: add relative link with H1-compatible display title and update index updated_at/time_context.` |
| “质量不够。” | 没有映射到质量门禁或具体可修内容。 | `RF-003 model_discrimination_quality: docs/computer-systems/example.md ## 相邻概念 only lists related terms and gives no distinguishing criteria. Required fix: add discrimination table comparing object boundary, trigger condition, mechanism difference and misuse risk against the three adjacent terms.` |

## Targeted revision examples

以下示例是可复制的 governance examples，不是对当前仓库资产的实际 review records。使用时必须把 path、heading、claim、owner 和 decision 改成真实目标证据。

| Example | Failure class | Evidence location | Why it blocks or not | Required fix | Owner | Decision label | Resubmission check |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `RF-FM-001` missing YAML array | `repository_integration` | `docs/governance/example-policy.md` frontmatter `related_docs: docs/index.md` | Blocks `accept_promote` because schema requires YAML array and `related_docs` cannot be reliably checked. | Change to `related_docs:\n  - docs/index.md`; verify target exists; do not add schema fields. | current Agent | `targeted_revision` | `resolved` only if frontmatter uses YAML array and related target exists; otherwise `still_blocked`. |
| `RF-SEC-001` missing role boundary | `structure_completeness` | `docs/templates/example-template.md` missing `## 职责边界与非目标` | Blocks template use because reviewer cannot know whether it creates records, policy or automation. | Add section naming role, non-goals, future-story boundaries and prohibited outputs. | current Agent | `targeted_revision` | `resolved` only if section names non-goals and does not implement future assets. |
| `RF-MDQ-001` weak discrimination | `model_discrimination_quality` | `docs/programming-languages/example.md ## 相邻概念` | Blocks `validated` wording because the document cannot distinguish adjacent concepts during use. | Add a comparison table with object boundary, mechanism, trigger, failure mode and misuse correction. | current Agent | `targeted_revision` | `resolved` if new table supports at least one concrete diagnosis or transfer check. |
| `RF-SRC-001` unsupported currentness | `evidence_time_context` | `docs/ai-systems/example.md`, paragraph saying “当前所有主流框架...” | Blocks acceptance because current-practice claim lacks source, checked date and scope limitation. | Delete universal claim or replace with source-limited claim; add checked date/source limitation or move uncertainty to `open_questions`. | source owner / current Agent | `targeted_revision` | `resolved` if claim is removed, sourced, narrowed or marked as inference/open question without supporting through-status. |
| `RF-LNK-001` missing index entry | `repository_integration` | `docs/index.md ## governance`; new formal asset not listed | Blocks discoverability for a formal governance asset when index impact is `updated`. | Add `./governance/example-policy.md` under correct section; update index metadata; verify path resolves. | current Agent | `targeted_revision` | `resolved` if relative link target exists and display title matches H1/frontmatter meaning. |
| `RF-CON-001` title/status conflict | `consistency` | frontmatter `title`, H1 and conclusion section use different asset names/status words | Blocks review because reader cannot identify the canonical asset or allowed status wording. | Choose title/H1 from higher-priority source; align index display; replace unsupported `validated` wording with `draft` or decision evidence. | reviewer / current Agent | `targeted_revision` | `resolved` if title/H1/index/status wording are mutually consistent and evidence-backed. |
| `RF-DUP-001` duplicate uncertainty | `duplicate_risk` | proposed `docs/methodology/example.md` overlaps existing `docs/methodology/example-catalog.md` | Blocks promotion because canonical path and reuse boundary are unclear. | Create or reference Duplicate / Coexistence Decision Record; decide `merge`, `adjacent_link`, `narrower_document`, `explicit_coexistence`, `reject`, `defer_with_reason`, or map a required Maxwell decision to review decision `hold_for_clarification`. | Maxwell / reviewer | `hold_for_clarification` or `targeted_revision` | `resolved` only if distinct concept/purpose/reuse context or rejection/merge path is recorded. |
| `RF-LC-001` lifecycle wording conflict | `lifecycle_issue` | `docs/governance/example-policy.md` conclusion says `archived`, index still presents as current primary entry | Blocks lifecycle decision because archive limits normal reuse and needs access/index evidence. | Distinguish `deprecated` versus `archived`; add Migration/Index evidence or hold for owner decision; do not silently delete. | Maxwell / current Agent | `deprecate_or_archive` or `hold_for_clarification` | `resolved` if lifecycle outcome, remaining access, successor/index treatment and status wording align. |

For targeted revision, the original asset identity normally survives. If the repair would change canonical identity, path, old-content handling, duplicate/coexistence outcome or lifecycle visibility, targeted revision must reference the appropriate specialized records before the work is accepted.

## Regenerate examples and rationale requirements

`regenerate` is required only when local repair cannot restore integrity. The rationale must explain why targeted revision is insufficient, and it must decide how high-value content and reviewer notes are preserved.

Use `regenerate` when one or more of these conditions holds:

- Wrong document type: a governance policy is written as an ordinary concept document, or a template is written as a completion report.
- Collapsed structure: the document lacks stable object boundary, mechanism/discrimination frame or required governance fields across most sections.
- Unusable source basis: core conclusions cannot be supported by current `source_basis`, project rules or verifiable external/currentness evidence.
- Pervasive currentness failure: current-practice, historical/deprecated or external fact claims are wrong across the document.
- Identity/path collapse: `doc_id`, title, path, topic and index imply different assets.
- Duplicate/misplacement collapse: the proposed asset should be merge/reject/move/reclassify rather than locally patched.
- Unsafe overwrite: local edits would silently erase old content, reviewer notes, successor evidence or migration history.
- High-value-content preservation risk: useful reviewer notes, examples or source analysis exist, but cannot safely remain in the current document shape.

Regeneration examples:

| Example | Why targeted revision is insufficient | Regeneration scope | Preservation handling | Required records | Decision label | Resubmission check |
| --- | --- | --- | --- | --- | --- | --- |
| `RG-TYPE-001` governance asset written as concept doc | Every section explains the term instead of defining authority, scope, records, allowed outcomes and non-goals. Local inserts would create contradictory document type. | Rewrite as governance asset with role/scope, non-goals, taxonomy, records and validation checklist. | Preserve useful definitions in a short “术语说明” subsection or archive as reviewer note if still relevant. | Index Impact Decision Record; review record evidence. | `regenerate` | `resolved` if regenerated asset has governance frontmatter/body shape and old useful content is preserved, summarized or explicitly rejected. |
| `RG-MODEL-001` concept document lacks stable model | Boundary, mechanism, discrimination, tradeoff and verification sections are absent or mutually inconsistent. | Regenerate using concept document contract and quality gate. | Migrate any accurate examples into the new example/mechanism sections; summarize discarded prose and why it was unusable. | Quality gate evidence; source discipline if examples include external facts. | `regenerate` | `resolved` if new document passes Hard Fail checks for structure and model/discrimination quality. |
| `RG-SRC-001` source basis unusable | The document makes many current-practice claims, but `source_basis` is only a stale internal note and no claim can be isolated cheaply. | Regenerate from valid project rules and verified/narrowed sources; remove unsupported universals. | Preserve reviewer notes as open questions or source-search tasks; do not keep unsupported claims as facts. | Source discipline evidence; review record `未验证` register. | `regenerate` or `hold_for_clarification` | `resolved` if claims are sourced, narrowed, marked as inference/open question, or removed. |
| `RG-ID-001` identity/path collapse | File path, `doc_id`, title, H1 and index entry point to different canonical assets. Local repair might overwrite the wrong asset. | Stop and regenerate only after identity decision; create correct asset or replacement with stable identity. | Preserve old content under Migration Decision Record handling: preserved, moved, summarized, archived or rejected with reason. | Migration Decision Record; Index Impact Decision Record; Duplicate / Coexistence Decision Record if overlap exists. | `regenerate` or `hold_for_clarification` | `resolved` if identity, path, index and old-content handling are explicit. |
| `RG-DUP-001` duplicate/misplacement collapse | Candidate duplicates an existing asset and cannot show distinct concept, purpose or reuse context. Local edits would create parallel authority. | Reject, merge, narrow or regenerate at a new path only after duplicate/coexistence decision. | Preserve unique examples by merging into canonical asset or summarize as rejected candidate evidence. | Duplicate / Coexistence Decision Record; Migration Decision Record if old content moves; Index Impact Decision Record. | `reject_duplicate_or_misplaced` or `regenerate` | `resolved` if canonical target and candidate disposition are recorded. |
| `RG-NOTE-001` high-value reviewer notes at risk | The draft is structurally unusable, but review notes contain hard-won source analysis and failure mapping. Patching would bury notes inside a bad structure. | Regenerate the target asset; extract reviewer notes into review evidence, open questions or an authorized future record. | Preserve notes verbatim only where authorized; otherwise summarize, archive, or explain why not preserved. | Review record; Migration Decision Record if replacing existing formal content. | `regenerate` | `resolved` if high-value content handling is explicit and no silent overwrite occurred. |

Regeneration in this asset is still an Epic 3 review rework example. It does not replace the Story 4.1 revision/regeneration continuity policy. When regeneration affects identity, path, old content, duplicate risk, source basis, lifecycle, reference validity or index/link behavior, the rework loop must reference Revision / Regeneration Continuity Record, Migration Decision Record, Duplicate / Coexistence Decision Record, Index Impact Decision Record or Source Discipline evidence as applicable.

## Reject, hold, defer and not-authorized examples

These examples point back to Story 3.2 decision semantics. They are not replacement policy.

| Example | Use when | Required evidence | Decision label | Specialized policy |
| --- | --- | --- | --- | --- |
| `RJ-DUP-001` duplicate candidate rejected | Candidate has no durable distinction from existing canonical asset and should not enter `docs/`. | existing canonical path, proposed path, duplicate type, rejected alternatives, content disposition. | `reject_duplicate_or_misplaced` | Duplicate/coexistence; index synchronization; migration if old content moves. |
| `HD-ID-001` canonical identity unclear | Reviewer cannot decide whether this is a new asset, a replacement or a revision. | exact conflicting paths/titles/doc_ids, owner question, actions paused until decision. | `hold_for_clarification` | Frontmatter schema; rename/migration; duplicate/coexistence. |
| `DF-E5-001` link taxonomy deferred | Link relationship is real but Epic 5 taxonomy is not yet available and current acceptance is not blocked. | current link impact, why nonblocking, future owner, re-review trigger. | `defer_with_reason` | Index sync now; Epic 5 later. |
| `NA-AUTO-001` rework generator requested | Requested output would create executable automation, validator, schema or scoring tool not authorized by current story. | crossed boundary, forbidden output, safe alternative as human-readable guidance. | `not_authorized` | Agent constraints; batch readiness if future batch automation is requested. |

## Prior failure record and resubmission loop

Re-submitted work must be checked against prior failure records. Reviewer must not accept resubmission by checking only the changed sections when the old failures affected identity, structure, source/time context, lifecycle, duplicate/coexistence or index/navigation.

Copyable prior-failure register:

| failure id | class | evidence location | original decision | required fix | owner | blocking status | resolution evidence | current result |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `RF-001` | `repository_integration` | `docs/index.md ## governance`, missing `./governance/example.md` | `targeted_revision` | Add index entry and update metadata | current Agent | `blocked` | Index entry exists; link target resolves | `resolved` |
| `RF-002` | `evidence_time_context` | `docs/topic/example.md ## 当前实践`, unsupported claim | `targeted_revision` | Add source/date, narrow claim, or move to open questions | source owner | `blocked` | Claim narrowed and limitation added | `resolved` |
| `RF-003` | `duplicate_risk` | candidate path overlaps existing canonical path | `hold_for_clarification` | Record duplicate/coexistence outcome | Maxwell | `held` | Duplicate record chooses `narrower_document` | `changed_scope` |

Allowed resubmission outcomes:

| Result | Meaning | Required reviewer action |
| --- | --- | --- |
| `resolved` | Old failure is fixed in the exact evidence location or its successor location. | Point to resolution evidence and confirm no new failure was introduced. |
| `still_blocked` | Old failure remains or the attempted fix is insufficient. | Preserve or update blocker, required fix and owner. |
| `changed_scope` | The target, path, identity, owner or authorized scope changed so the old fix no longer maps one-to-one. | Explain old-to-new mapping and whether a new failure record is needed. |
| `deferred_with_reason` | The failure is real but authorized deferral exists and does not falsely support acceptance. | Name owner/future story, impact and re-review trigger. |
| `not_authorized` | Fix would cross current story, schema, batch, non-software or owner boundary. | Stop unauthorized work and record allowed scope. |
| `new_failure` | Resubmission introduces a failure not present in the prior record. | Create a new failure id, classify it and apply normal decision policy. |

Resubmission checklist:

- [ ] Every prior failure id is present in the resubmission check.
- [ ] Each prior failure has one result: `resolved`, `still_blocked`, `changed_scope`, `deferred_with_reason`, `not_authorized` or `new_failure`.
- [ ] Evidence locations were rechecked, including successor locations if the section/path moved.
- [ ] Fixes did not create new source/time context failures.
- [ ] Fixes did not create frontmatter, H1/title, `doc_id`, topic/path or status contradictions.
- [ ] Fixes did not create broken body links, `related_docs` targets or stale `docs/index.md` entries.
- [ ] Duplicate/coexistence, migration, lifecycle and index records remain complete when relevant.
- [ ] Specialized records are marked complete, incomplete, blocking, nonblocking, deferred_with_reason, held or not_applicable.
- [ ] Accepted or promoted wording is not used while old blockers remain unresolved.

## Specialized decision record integration

A rework loop may need multiple specialized records. The rework loop must say which records are complete, incomplete, blocking, nonblocking, deferred_with_reason, held or not_applicable.

| Record | Reference when | Rework-loop requirement |
| --- | --- | --- |
| Promotion Decision Record | Candidate source prepares to enter formal `docs/`, replace existing asset or claim promotion. | State whether promotion evidence is complete and whether target path/frontmatter/source/index are blocking. |
| Index Impact Decision Record | New, moved, renamed, retitled, deprecated, archived or navigation-impacting asset. | Record outcome: `updated`, `not_applicable`, `referenced_elsewhere`, `intentionally_excluded`, `deferred_with_reason` or `blocked_index_policy_conflict`. |
| Migration Decision Record | Rename, move, retitle, split, merge, replacement, successor, deprecation, archive, reactivation or old-content handling. | Record identity continuity, old-content handling, successor/replacement, link/index/lifecycle impact and unresolved risks. |
| Revision / Regeneration Continuity Record | Targeted revision, structural upgrade, regeneration, split, merge, deprecation or reference-validity impact. | Record update mode, why changed, what was preserved/removed, old-content disposition, metadata/source/status impact and reference validity. |
| Duplicate / Coexistence Decision Record | Duplicate, near duplicate, overlap, adjacent concept, narrower/deeper asset, same-topic coexistence or rejection. | Record type, chosen outcome, rejected alternatives, distinct concept/purpose/reuse context and topic/path impact. |
| Batch Readiness Record | Target set is selected by rule, multiple assets/topics/status/index entries are affected, or batch-shaped work appears. | Stop or defer until batch readiness names target set, exclusions, recovery, sampling, owner decision and stop conditions. |
| Source Discipline evidence | Current-practice, historical/deprecated, external fact, source/currentness or unverifiable claim failure. | Record claim type, source limitation, checked date or reason to remove/narrow/mark as inference/open question. |

Specialized records keep their own required fields, outcomes and owner decisions. Rework examples must reference them, not collapse their fields into new frontmatter or informal notes.

## Copyable rework loop skeleton

Use this skeleton inside review evidence, decision evidence, story Dev Agent Record, completion evidence or a future authorized template. It is human-readable guidance, not a machine-enforced schema.

```markdown
## Rework Loop

### Scope

- task type:
- asset level:
- target path:
- source/candidate path:
- applicable rules:
- non-goals / unauthorized outputs:

### Failure Register

| failure id | class | evidence location | why it blocks / does not block | required fix | owner | blocking status | decision label | resubmission check |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RF-001 | repository_integration |  |  |  |  | blocked | targeted_revision |  |

### Specialized Records

| record | required? | status | evidence location | blocking impact | owner / next action |
| --- | --- | --- | --- | --- | --- |
| Promotion Decision Record | yes/no | complete/incomplete/deferred_with_reason/held/not_applicable |  | blocked/nonblocking/held/deferred_with_reason/not_authorized |  |
| Index Impact Decision Record | yes/no |  |  |  |  |
| Migration Decision Record | yes/no |  |  |  |  |
| Duplicate / Coexistence Decision Record | yes/no |  |  |  |  |
| Batch Readiness Record | yes/no |  |  |  |  |
| Source Discipline evidence | yes/no |  |  |  |  |

### Regeneration Rationale

- regenerate required: yes/no
- why targeted revision is insufficient:
- high-value content / reviewer notes:
- preservation handling: preserve / migrate / summarize / archive / not preserved with reason
- identity/path/source/duplicate/lifecycle/index impact:
- required specialized records:

### Resubmission Check

| prior failure id | prior evidence location | current evidence location | result | resolution evidence | new failures introduced? |
| --- | --- | --- | --- | --- | --- |
| RF-001 |  |  | resolved/still_blocked/changed_scope/deferred_with_reason/not_authorized/new_failure |  | yes/no |

### Final Rework Result

- final decision label:
- unresolved blockers:
- nonblocking follow-ups:
- deferred items and owner:
- not-authorized items:
- reviewer notes:
```

## Validation checklist and maintenance triggers

Validate a rework loop or this asset by checking:

1. Failure classes use only `repository_integration`, `structure_completeness`, `model_discrimination_quality`, `evidence_time_context`, `consistency`, `duplicate_risk` and `lifecycle_issue`.
2. Every repair instruction identifies an exact file, heading, section, frontmatter field, example, source claim, link, `related_docs` entry or `docs/index.md` entry.
3. Decision labels use Story 3.2 values: `accept_promote`, `accepted_for_current_use`, `accepted_with_nonblocking_follow_up`, `targeted_revision`, `regenerate`, `reject_duplicate_or_misplaced`, `deprecate_or_archive`, `hold_for_clarification`, `defer_with_reason` and `not_authorized`.
4. Blocking/status values use `blocked`, `nonblocking`, `held`, `deferred_with_reason` and `not_authorized`.
5. Resubmission outcomes use `resolved`, `still_blocked`, `changed_scope`, `deferred_with_reason`, `not_authorized` and `new_failure`.
6. Regeneration rationale explains why targeted revision is insufficient.
7. High-value content and reviewer notes are preserved, migrated, summarized, archived or explicitly not preserved with reason.
8. Specialized decision records are referenced when identity, path, old content, duplicate risk, source basis, lifecycle, index or batch scope is affected.
9. No rework/decision metadata is added as unauthorized target frontmatter fields.
10. No executable tooling, schema, generator, validator, package manifest, `src/`, `tests/`, build config, CI/deployment, CLI/API/UI/database asset or batch automation is introduced.

Maintenance triggers:

- `docs/templates/completion-report-template.md` changes rework/prior failure summary fields or completion wording rules.
- Story 4.1 revision/regeneration continuity policy changes Continuity Record fields or update-mode taxonomy.
- Epic 5 creates related-doc taxonomy and link maintenance policy and may refine link/index/related-doc rework examples.
- Epic 6 creates batch governance runbook and batch records and may need batch rework-loop examples.
- Maxwell explicitly authorizes automation, machine-readable schema, validators, lint/scoring tools or generated records.

## 参考资料

- [Knowledge Docs Index](../index.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](../templates/review-record-template.md)
- [文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements](./document-decision-policy.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [概念文档样例目录：合格、不合格与质量门禁证据](../methodology/concept-document-example-catalog.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响](./rename-migration-policy.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
