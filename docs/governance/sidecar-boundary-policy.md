---
doc_id: governance-sidecar-boundary-policy
title: Sidecar 与补充材料边界政策：主文档权威、支持资产分类与过期处理
concept: sidecar_boundary_policy
topic: governance
depth_mode: standard
created_at: '2026-05-28T14:34:49+08:00'
updated_at: '2026-05-30T15:53:15+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/4-2-sidecar-boundary-policy.md
  - _bmad-output/implementation-artifacts/4-3-legacy-migration-guide.md
  - docs/index.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/lifecycle-states.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/document-decision-policy.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
time_context: phase_4_epic_4_sidecar_boundary_policy_2026_05_28
applicability: formal_docs_sidecar_supplemental_material_boundary_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/lifecycle-states.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/document-decision-policy.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
open_questions:
  - `docs/governance/legacy-migration-guide.md` 已建立后，是否需要在后续审查中为 old sidecar、legacy notes 和 historical support material 补充更细的 worked examples？
  - Story 5.1 related-doc taxonomy 与 Story 5.2 link maintenance policy 已建立；后续是否需要区分 sidecar link、supporting link、successor link、related_docs 和 owner entry 的维护关系？
  - Epic 6 建立 batch governance runbook 后，是否需要把 sidecar inventory、bulk disconnect 和 batch support-asset cleanup 转入专门 runbook records？
---

# Sidecar 与补充材料边界政策：主文档权威、支持资产分类与过期处理

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 sidecar、report、revision note、example set、review record 和其他 supplemental / support artifact 的 canonical governance asset。它回答的是：正式 Markdown 主文档旁边出现补充材料时，Agent 或 reviewer 应如何分类、记录分类理由、保护主文档权威、检查链接和状态影响，并处理过期、冲突或不再应引用的支持材料。

本文适用于：

- `docs/{topic}/` 下的正式知识资产及其补充材料。
- `docs/methodology/` 下的方法论、模板、质量门禁、fixed prompt、playbook 和方法论支撑资产的补充材料。
- `docs/governance/` 下的治理资产与相邻支持证据。
- `docs/templates/` 下正式模板资产的支持记录。
- 正式 report entry、future runbook evidence、review evidence、completion evidence、story Dev Agent Record、Migration Decision Record、Index Impact Decision Record、Revision / Regeneration Continuity Record 或 future batch/runbook record 中出现的 supplemental material 判断。

本文的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/sidecar-boundary-policy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Epic 4 首版 sidecar boundary policy；`docs/governance/legacy-migration-guide.md` 已细化 legacy migration、old-doc compatibility、legacy notes 和 historical support material 的渐进兼容策略，Story 5.1 已细化 related-doc taxonomy，后续 Story 5.2+ 和 Epic 6 仍会细化 link maintenance、reuse/network governance 和 batch runbook records。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/sidecar-boundary-policy.md
- target title: Sidecar 与补充材料边界政策：主文档权威、支持资产分类与过期处理
- target path: ./governance/sidecar-boundary-policy.md
- target section: docs/index.md ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: Story 4.2 explicitly authorizes the canonical sidecar and supplemental material boundary policy asset
- validation result: target exists and relative link resolves
- unresolved risk: legacy migration is referenced via docs/governance/legacy-migration-guide.md; Epic 5 link taxonomy/link maintenance and Epic 6 batch runbook integrations remain open questions
```

## 职责边界与非目标

本文定义 supplemental artifact 的分类、主文档权威边界、sidecar impact review、过期支持材料处理规则和 Sidecar Boundary Decision Record。它不执行实际抽取、迁移、批量扫描或链接修复。

本文不实现以下范围：

- 创建实际 sidecar assets、report assets、review records、example sets、archive notes 或 completion reports。
- 从主文档中抽取 sidecar，或把旧材料迁移为 sidecar。
- 审查现有 sidecars、生成 sidecar inventory、执行 bulk disconnect、批量迁移、批量 link repair、批量 `related_docs` 更新或批量 status conversion。
- 创建 Epic 5 related-doc taxonomy、link maintenance policy、reusable model entry points、existing-doc reuse procedure 或 Epic 6 batch assets。
- 创建 runtime code、package manifest、source tree、software tests、CLI/API/UI/database/deployment/CI、machine-readable schema、validator、generator、link scanner、sidecar scanner、index generator、completion-report generator、decision generator、migration script 或 executable automation。

本文也不创建新的全局 frontmatter 字段。不得为了表达 sidecar 关系或支持资产状态，在正式资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`sidecar_status`、`support_asset_type`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`quality_gate_version`、`methodology_version`、`decision_status`、`successor`、`canonical_asset`、`sidecar_of`、`supersedes`、`supporting_record` 或类似字段。分类、权威关系、支持角色、过期处理和链接影响应写在正文、review evidence、completion evidence、story Dev Agent Record、Revision / Regeneration Continuity Record、Migration Decision Record、Index Impact Decision Record、future runbook record 或明确命名的 sidecar note 中。

## Supplemental Artifact Taxonomy

任何 supplemental artifact 新增、连接、更新、断开、废弃或归档前，都必须先分类。最低记录必须包括：artifact class、classification reason、owner、main-document relationship、allowed support role、required for current use?、frontmatter impact、links/index impact、quality/lifecycle impact、review/completion/specialized-record impact、freshness/currentness status 和 unresolved risks。

| Artifact class | Use when | Do not use when | Canonical owner | Allowed location / navigation treatment | Main-document obligations | Required records | Stop conditions |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `sidecar` | 围绕一个或少数主文档的伴随补充材料，用于承载过长的 evidence、examples、experiments、counterexamples、supplemental explanation 或 historical material | 需要承载主文档的核心定义、边界、状态、来源、当前结论、唯一例子或唯一失败模式 | 主文档 owner 或当前治理 story；若是正式 `docs/` asset，则还受其 topic/governance owner 约束 | 可为正式 `docs/` asset、受控 report/archive entry、workflow evidence 或明确命名 note；不得默认进入独立 sidecar index section | 主文档必须独立解释核心规则、范围、质量状态、source/time context、open questions 和 current recommendation | Sidecar Boundary Decision Record；必要时 Index Impact、Migration、Continuity、Review/Completion evidence | sidecar 与主文档 current claim、status claim、source claim 或 lifecycle claim 冲突；主文档离开 sidecar 后无法理解 |
| `report` | 一次任务、审查、迁移、规范化或完成汇报的 evidence asset；可作为 source/evidence 或 decision trail | 把 report 直接当成主文档正文、正式质量状态、当前结论或唯一导航入口 | report owner、reviewer、completion owner 或 future runbook owner | 正式 report entry、`_bmad-output/` workflow output、completion evidence 或 review evidence；workflow output 不直接加入 `docs/index.md` | 主文档可以引用 report 的 evidence，但必须在正文中保留当前规则、状态理由和适用范围 | Completion/report evidence；必要时 Review Record、Index Impact、Migration、Batch Readiness | report 结论与目标资产状态冲突；report 试图替代目标 frontmatter/body |
| `revision_note` | 一次修订、重生成、断开、废弃、归档或连续性判断的局部记录 | 需要新增 unauthorized frontmatter fields，或把局部 note 当成新主规范 | 目标资产 owner、revision owner 或 story/review owner | 目标正文、review/completion evidence、story Dev Agent Record、Revision / Regeneration Continuity Record、Migration Decision Record 或明确命名 note | 主文档必须仍说明当前状态、变更原因、source/time impact 和 open questions；note 只补充 evidence | Continuity Record 或 Migration Decision Record；必要时 Sidecar Boundary Decision Record | revision note 分散为 hidden metadata；旧内容处理、identity continuity 或 reference validity 不清 |
| `example_set` | 成组 examples、counterexamples、failure examples、self-test support 或 reviewer calibration material 支持主文档理解和质量门禁 | 主文档缺失必须理解的核心例子、反例或失败模式，只把它们放在 example set | 主文档 owner、quality/review owner 或 methodology owner | 主文档 section、formal support asset、template evidence、review evidence 或受控 sidecar | 主文档必须保留足够的核心例子职责；example set 可扩展深度、覆盖更多场景或保存 calibration evidence | Sidecar Boundary Decision Record；必要时 Review Record、Completion Report、Quality Gate evidence | 删除/断开 example set 后主文档无法识别概念边界、常见误读或验证入口 |
| `review_record` | 审查证据、Hard Fail/blocker、score/equivalent checks、未验证/不适用 register 和 final decision 支撑材料 | 试图自动改变正式资产状态、替代 target asset frontmatter/body，或绕过 document decision policy | reviewer、review workflow、template owner | `docs/templates/review-record-template.md` shape、review evidence、story record 或 future runbook record；不默认作为普通主题文档 | 主文档必须自己承载当前事实、状态和修订结果；review record 只提供 evidence trail | Review Record；Document Decision evidence；必要时 Completion Report、specialized records | review record 与主文档状态/质量声明冲突；decision label 缺少 required evidence |
| `other_support_asset` | 其他被明确授权的 supporting asset，例如 future runbook evidence、source dossier、archive note、owner decision note 或 narrowly scoped support note | 只是因为分类不清而逃避前五类，或需要 future Epic 5/6 资产却提前创建 | 明确命名 owner：Maxwell、future story、runbook owner、source owner 或 target asset owner | 由授权来源决定；必须声明为什么不属于前五类，并说明导航和剩余访问 | 主文档必须说明 support role、trust boundary、是否 required for current use 和断开/过期处理 | Sidecar Boundary Decision Record；必要时 owner decision、Index/Migration/Batch evidence | 不能解释为什么不属于前五类；owner、入口、质量/生命周期影响或 authorized scope 不清 |

分类不能只看文件名。Reviewer 必须比较 artifact 的用途、主文档关系、读者入口、状态影响、来源/时间语境、链接形态和维护 owner。分类不清时，使用 `hold_for_clarification`、`defer_with_reason` 或 `not_authorized`，不得先写入正式状态或索引。

## Main Document Canonical Truth Rule

正式 Markdown 主文档必须是 canonical、可独立理解且不依赖 sidecar 承担 primary truth。Supplemental artifact 可以支持、展开、记录或归档，但不能把主文档变成只剩目录或指针。

主文档必须独立承载：

- 核心定义、适用范围和不适用范围。
- 关键判断、当前推荐实践、阻塞条件和失败模式。
- 来源依据、时间语境、质量状态、生命周期语义和当前性限制。
- open questions、维护触发点、相关链接和必要的相邻治理关系。
- 读者或 Agent 当前使用该文档时必须知道的最低例子、反例、验证入口和升级路径。

Sidecar 或其他补充材料不得成为：

- primary truth。
- hidden source of truth。
- 唯一例子位置或唯一失败模式位置。
- 唯一 review basis。
- 唯一 currentness、source basis、quality status 或 lifecycle evidence。
- 让读者必须离开主文档才能理解核心规则的依赖项。

主文档引用 sidecar 时，必须说明：

- artifact class 和 role。
- scope：它支持哪个 section、claim、evidence 或历史判断。
- trust boundary：哪些内容可作为当前 supporting evidence，哪些只是历史或背景。
- last relevant context：其时间语境或适用阶段。
- required for current use?：是否当前使用主文档必须阅读。
- stale/disconnect handling：过期、冲突、断开或归档时如何处理。

禁止用 “see sidecar” 替代核心解释、质量结论、状态理由、来源限制、open question 或当前推荐实践。如果 sidecar 中出现与主文档冲突的 current claim、status claim、source claim 或 lifecycle claim，必须先按 blocker、`hold_for_clarification`、`defer_with_reason` 或 `not_authorized` 处理，不得让冲突静默共存。

## Sidecar Impact Review

新增、连接、更新、断开、废弃或归档 sidecar / supplemental artifact 时，必须执行 impact review。检查范围应与当前授权相称，但不能假装没有 frontmatter、link、status 或 specialized-record 影响。

最低检查项如下：

| Impact surface | Required review |
| --- | --- |
| Main document frontmatter | `source_basis` 是否需要引用 sidecar/report/review evidence；`related_docs` 是否适合包含 sidecar；`open_questions` 是否需要记录 missing/stale/conflicting sidecar；`quality_status` 是否受支持证据或冲突影响。 |
| Support artifact frontmatter | 若 artifact 是正式 `docs/` asset，必须符合 baseline fields、YAML arrays、stable `doc_id` 和正文 role/scope/navigation 声明；若它是 workflow output，则不得直接加入 `docs/index.md` 或伪装为正式资产。 |
| Body links | main -> sidecar、sidecar -> main、review/completion/story evidence links、fragment anchors 和 discoverable inbound references 是否可解析，是否会误导读者把 sidecar 当 primary entry。 |
| `docs/index.md` | 新增正式 sidecar boundary policy 本身必须索引；实际 sidecar assets 的索引或受控排除由资产类别和导航政策决定，不得默认创建 sidecar index section。 |
| Review/completion evidence | Review Record、Completion Report、Document Decision evidence 是否需要引用或汇总 sidecar evidence；不得用 sidecar 替代这些模板字段权威。 |
| Specialized records | Revision / Regeneration Continuity、Migration、Index Impact、Duplicate/Coexistence、Promotion、Batch Readiness、Source Discipline evidence 是否受影响。 |
| Batch shape | 是否已经变成 sidecar inventory、bulk disconnect、bulk migration、batch link repair、batch `related_docs` update、batch status conversion 或 target set selected by rule。 |

Impact review 的结果必须落到可复查记录中，可以是 Sidecar Boundary Decision Record、target body note、review evidence、completion evidence、story Dev Agent Record、Revision / Regeneration Continuity Record、Migration Decision Record、Index Impact Decision Record 或 future runbook record。

如果缺失、过期或冲突的 sidecar evidence 会影响主文档的 current use、质量状态、来源/时间语境、关键链接、owner decision 或 lifecycle semantics，必须阻塞 acceptance，或使用 `hold_for_clarification` / `defer_with_reason` 记录原因和 owner。Nonblocking impact 也必须有 follow-up owner 和 re-review trigger。

## Outdated Sidecar Handling

Sidecar 或其他 support artifact 只要不再可靠支持当前主文档，就必须被处理。不得让过期支持材料继续以当前 evidence 的形式误导后续 Agent。

过期、失效或冲突触发器包括：

- 主文档已经 revision、structural upgrade、regeneration、replacement、split、merge、deprecation 或 archive。
- source/time context 失效，support artifact 的 checked date、project phase、外部事实或当前性限制不再匹配。
- `quality_status`、lifecycle wording、review decision 或 completion evidence 与主文档冲突。
- link target、fragment anchor、owner entry point 或 index/navigation treatment 断裂或改变。
- sidecar 与主文档 current conclusion、source claim、status claim 或 recommended practice 冲突。
- review record 被 superseded，或 sidecar 只剩历史、审计、migration 或 old decision 价值。
- sidecar 继续被引用会让读者误以为它是 primary entry、current source 或唯一证据。

最终处理动作只有：

| Action | Use when | Required evidence |
| --- | --- | --- |
| `update` | support artifact 仍有当前价值，且当前 story 授权修改它 | reason、changed scope、main doc impact、source/time impact、frontmatter impact、link impact、review/completion evidence、index impact、owner/follow-up |
| `deprecate` | support artifact 保留访问，但不再作为当前 supporting evidence | reason、successor/replacement or missing-successor risk、remaining access、link/index/lifecycle impact、main doc fallback、owner |
| `archive` | support artifact 仅作为历史、审计、migration 或 old decision evidence 保留 | archive location or retrieval expectation、normal-use restriction、successor or no-successor risk、reactivation condition、index visibility、remaining access |
| `disconnect` | 主文档不再引用该 support artifact，或降低它的支持关系 | why disconnect、whether artifact remains、whether main doc must restore core information、links/related_docs/source_basis/open_questions handling、owner/follow-up |

如果当前授权、证据或 owner decision 不足以选择最终处理动作，record 可以先写 `deferred_with_reason`、`hold_for_clarification` 或 `not_authorized`。这些值是停止、延后或未授权状态，不是 stale support artifact 的最终处理动作；后续必须回到 `update`、`deprecate`、`archive` 或 `disconnect` 之一，或明确记录为什么不再适用。

这些动作都不能静默删除正式资产，不能跳过 link/index impact，不能把 lifecycle/status impact 隐藏在 completion note 中，也不能用 sidecar 处理绕过 `docs/governance/revision-regeneration-continuity-policy.md` 或 `docs/governance/rename-migration-policy.md`。

如果需要 rename、move、successor、replacement、split、merge、deprecation、archive 或 remaining access 处理，必须调用 Migration Decision Record。如果主文档 revision/regeneration、旧内容处理、reference validity 或 continuity evidence 受影响，必须调用 Revision / Regeneration Continuity Record。如果影响多个资产或目标集由规则选出，必须先执行 Batch Readiness Record，并在未授权时使用 `not_authorized`。

## Sidecar Boundary Decision Record

`Sidecar Boundary Decision Record` 是人类可读、可复制的支持资产边界记录。它不是 machine-readable schema，也不是新的 frontmatter 字段。

最低字段如下：

```markdown
## Sidecar Boundary Decision Record

| Field | Value |
| --- | --- |
| main asset |  |
| support artifact |  |
| artifact class | sidecar / report / revision_note / example_set / review_record / other_support_asset |
| classification reason |  |
| main-document relationship |  |
| canonical truth location |  |
| what the main document must contain |  |
| allowed support role |  |
| required for current use? | yes / no / limited / unknown |
| frontmatter impact |  |
| links / related_docs impact |  |
| open_questions impact |  |
| quality / lifecycle impact |  |
| review / completion / specialized-record impact |  |
| index / navigation impact | updated / not_applicable / referenced_elsewhere / intentionally_excluded / deferred_with_reason / blocked_index_policy_conflict |
| freshness / currentness status | current / stale / conflicting / historical_only / unknown |
| outdated handling | update / deprecate / archive / disconnect / not_applicable / deferred_with_reason / hold_for_clarification / not_authorized |
| owner / future-story dependency |  |
| validation result |  |
| unresolved risks |  |
```

Record 可以落在：

- target body。
- review evidence。
- completion evidence。
- story Dev Agent Record。
- Revision / Regeneration Continuity Record。
- Migration Decision Record。
- Index Impact Decision Record。
- future runbook record。
- 明确命名的 sidecar note。

不得把 record 字段拆成未经授权的 frontmatter fields。若 record 中任一关键字段未知，必须写 `未验证`、`deferred_with_reason`、`hold_for_clarification`、`not_authorized` 或明确 blocker，而不是留空后宣称完成。

## Relationship to Adjacent Governance Assets

本文调用相邻治理资产，不替代它们的字段权威。

| Asset | 本文如何使用 |
| --- | --- |
| `docs/governance/revision-regeneration-continuity-policy.md` | 调用 update mode、metadata review、old-content handling、reference validity 和 Revision / Regeneration Continuity Record；本文只定义 support artifact 角色边界和 sidecar-specific impact。 |
| `docs/governance/frontmatter-schema.md` | 使用 baseline fields、stable `doc_id`、YAML arrays 和 unauthorized frontmatter field 禁令；sidecar relationship metadata 不新增 schema fields。 |
| `docs/governance/lifecycle-states.md` | 使用 `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` 和 remaining access / reactivation 语义。 |
| `docs/governance/index-synchronization-rules.md` | 使用 index outcomes：`updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason`、`blocked_index_policy_conflict`。 |
| `docs/governance/document-decision-policy.md` | 使用 `accept_promote`、`accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`targeted_revision`、`regenerate`、`reject_duplicate_or_misplaced`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason`、`not_authorized` 和 blocker precedence。 |
| `docs/templates/review-record-template.md` | Review record 汇总 sidecar evidence、blockers、未验证/不适用项和 final decision；本文不替代 review record 字段。 |
| `docs/templates/completion-report-template.md` | Completion report 汇总 changed files、validation evidence、impact scope、risk 和 non-software boundary；本文不创建 actual completion report。 |
| `docs/governance/rename-migration-policy.md` | 处理 sidecar rename/move/replacement/successor/deprecation/archive、old-content handling 和 remaining access。 |
| `docs/governance/candidate-promotion-checklist.md` | 当 support artifact 从 `_bmad-output/`、conversation、temporary note 或 review output 晋升为正式 `docs/` asset 时，必须先通过 promotion gate。 |
| `docs/governance/duplicate-and-coexistence-policy.md` | 当 sidecar、主文档或 supporting asset 之间出现 duplicate、near duplicate、same-topic coexistence 或 canonical entry 风险时，调用 duplicate/coexistence decision。 |
| `docs/governance/batch-readiness-checklist.md` | 判断 sidecar work 是否变成 inventory、bulk disconnect、batch migration、batch link repair、batch status conversion 或 rule-selected target set。 |
| `docs/methodology/document-generation-methodology.md` | 概念文档的新建、升级、审查和仓库集成主执行路径仍由主方法论负责；本文只约束 supplemental material 不得替代主文档职责。 |
| `docs/methodology/intake-and-intent-classification.md` | 判断任务是新建、升级、审查、index-only、governance maintenance、archive/deprecation、batch governance 或 workflow output work。 |
| `docs/methodology/concept-document-quality-gate.md` | 当主文档是概念文档时，sidecar 不得绕过 Hard Fail、评分、质量状态和核心例子/验证职责。 |
| `docs/methodology/source-discipline-and-real-world-anchor-policy.md` | Sidecar 中的 current-practice、source/currentness、historical/deprecated practice 或 external-fact claim 必须服从来源纪律。 |

Future-story boundaries：

- `docs/governance/legacy-migration-guide.md` 负责 legacy migration guide、old-doc compatibility、legacy notes 和 historical support material 的渐进兼容策略。
- Story 5.1 负责 related-doc taxonomy；`docs/governance/link-maintenance-policy.md` 负责 general link maintenance；`docs/governance/reusable-model-entry-points.md` 负责 reusable model entry points；`docs/governance/existing-doc-reuse-procedure.md` 负责 existing-doc reuse procedure；`docs/governance/network-boundary-and-decay-prevention.md` 负责 network boundary assets。
- Epic 6 负责 batch governance runbook、batch change review records、batch completion report 和批量执行记录。

本文可以记录这些 future owner 作为 open question、maintenance trigger 或 `deferred_with_reason`，但不得提前创建它们的资产或执行它们的批量工作。

## Validation Checklist and Maintenance Triggers

使用或维护本文时，至少检查：

1. 任务类型、资产层级、allowed file scope、expected output、validation evidence 和 stop/escalation conditions 已声明。
2. Supplemental artifact 已分类为 `sidecar`、`report`、`revision_note`、`example_set`、`review_record` 或 `other_support_asset`，并记录 classification reason、owner、main-document relationship 和 allowed support role。
3. 主文档仍是 canonical、可独立理解，且不依赖 sidecar 承担 primary truth。
4. 主文档保留核心定义、scope/non-scope、关键判断、source/time context、quality/lifecycle status、open questions、相关链接、必要例子/反例/失败模式和维护触发点。
5. Sidecar 引用说明 role、scope、trust boundary、last relevant context、required for current use? 和 stale/disconnect handling。
6. Sidecar impact review 覆盖 frontmatter、`source_basis`、`related_docs`、`open_questions`、`quality_status`、body links、`docs/index.md`、review/completion evidence 和 specialized records。
7. 过期或冲突 support artifact 已处理为 `update`、`deprecate`、`archive` 或 `disconnect`，并记录 reason、remaining access、successor/replacement or missing-successor risk、link/index/lifecycle impact 和 follow-up owner。
8. Sidecar Boundary Decision Record 字段完整，且没有被拆成 unauthorized frontmatter fields。
9. Index impact 使用 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict`。
10. Decision labels 使用当前 document decision vocabulary，不重新引入旧 canonical labels。
11. Lifecycle vocabulary 使用 `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` 和 reactivation semantics。
12. Changed-file links、`related_docs` 和 `docs/index.md` entry 可解析；缺失或 planned targets 进入 `open_questions` 或 future-story dependency。
13. 没有新增 unauthorized frontmatter fields。
14. 没有创建实际 sidecar assets、执行 sidecar extraction、review existing sidecars、batch link repair、legacy migration、Epic 5 link/network assets、Epic 6 batch assets 或 actual completion reports。
15. 没有创建 runtime code、package manifests、source/test directories、software test framework、schemas、validators、generators、automation、CLI/API/UI/database/deployment/CI、sidecar scanners、link scanners 或 index generators。

维护触发点：

- `docs/governance/legacy-migration-guide.md` 已建立；后续若该指南更新，复核 old sidecar、legacy notes、historical support material、old-doc compatibility 和 support artifact remaining access。
- Story 5.1 related-doc taxonomy、Story 5.2 link maintenance policy、Story 5.3 reusable model entry points 或 Story 5.4 existing-doc reuse procedure 更新后，在授权窄范围内复核 sidecar link、supporting link、successor link、related_docs、owner entry 和 inbound/outbound evidence。
- Epic 6 建立 batch governance runbook、batch change review record 或 batch completion report 后，复核 sidecar inventory、bulk disconnect、batch support-asset cleanup 和 batch completion evidence 的落点。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、sidecar scanner、link scanner、index generator、new global frontmatter fields 或 batch normalization 后，复核本文非软件边界和字段边界。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements](./document-decision-policy.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](../templates/review-record-template.md)
- [完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界](../templates/completion-report-template.md)
- [重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响](./rename-migration-policy.md)
- [候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁](./candidate-promotion-checklist.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
