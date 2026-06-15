---
doc_id: governance-network-boundary-and-decay-prevention
title: 知识网络主题边界与退化防护政策：topic health、weak links、overlap handling 与 navigation relevance evidence
concept: network_boundary_and_decay_prevention
topic: governance
depth_mode: standard
created_at: '2026-06-15T14:34:30+08:00'
updated_at: '2026-06-15T15:46:07+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/5-5-network-boundary-and-decay-prevention.md
  - _bmad-output/implementation-artifacts/5-4-existing-doc-reuse-procedure.md
  - _bmad-output/implementation-artifacts/5-3-reusable-model-entry-points.md
  - _bmad-output/implementation-artifacts/5-2-link-maintenance-policy.md
  - _bmad-output/implementation-artifacts/5-1-related-docs-taxonomy.md
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
time_context: phase_5_epic_5_network_boundary_decay_prevention_2026_06_01
applicability: formal_docs_topic_boundary_network_decay_prevention_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
open_questions:
  - Epic 6 建立 batch governance records 后，是否需要为 batch network review、stale-link cleanup 和 batch relationship updates 提供专门 runbook 或 batch review record？
  - 后续是否需要把 Network Boundary / Decay Review Record 与 review/completion templates 的 specialized records summary 进一步对齐？
---

# 知识网络主题边界与退化防护政策：topic health、weak links、overlap handling 与 navigation relevance evidence

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 formal `docs/` 知识网络扩展后维护 topic boundary、cross-document link quality、repeated overlap routing 和 next-document navigation relevance 的 canonical governance asset。它回答的是：当一个 topic 变得过宽、链接开始变密但不可解释、多个文档反复回答同一问题，或正文导航只写“参见”却没有迁移理由时，Agent / reviewer 应如何识别退化信号、记录证据、选择治理路由，并保持动作仍在当前授权范围内。

本文适用于：

- 正式 `docs/` 资产的 topic health review、topic boundary clarification、subtopic split signal 和 misplaced document signal。
- 正文链接、frontmatter `related_docs`、curated next-doc sections、review evidence、completion evidence、story Dev Agent Record 或明确命名 network-boundary note 中的 weak/dense/noisy/stale relationship signal。
- 多个正式资产出现 repeated overlap、parallel canonical entry、same reader question、shared concept authority 或 stale replacement overlap 时的治理路由。
- 需要说明 next document 为什么相关、迁移什么 reasoning、不能迁移什么边界、何时应停止或转向其他治理资产的导航证据。

本文不适用于：

- 执行 topic-wide cleanup、full-repo network scan、all-doc inbound/outbound repair、stale-link cleanup、dense-link cleanup、batch relationship assignment、batch `related_docs` update、index-wide cleanup 或 generated network health report。
- 实际执行 merge、split、deprecate、archive、move、rename、successor switch、candidate promotion、document regeneration、quality status promotion 或 old-content deletion。
- 新增 `network_health`、`topic_health`、`link_density`、`decay_status`、`weak_links`、`stale_links`、`boundary_status`、`overlap_status`、`navigation_relevance`、`schema_version` 或类似 frontmatter fields。
- 创建 runtime code、package manifest、`src/`、`tests/`、software tests、CLI/API/UI/database/deployment/CI、machine-readable schema、validator、generator、network scanner、link scanner、model scanner、index generator 或 automation。

本文补充而不替代以下资产：

| Adjacent asset | 本文调用它做什么 | 本文保留的边界 |
| --- | --- | --- |
| [related docs 与相邻概念关系分类](./related-docs-taxonomy.md) | 调用 target status、relation type、transfer object、meaningful-link evidence 和 Relationship Record。 | 本文不重写 relationship taxonomy；只识别 weak/noisy/stale relation risk。 |
| [跨文档链接维护政策](./link-maintenance-policy.md) | 调用 changed-file link checks、one-way reason、fragment/anchor validation、bounded inbound/outbound review 和 Link Maintenance Record。 | 本文不执行 broad link repair、all-doc scan 或 stale cleanup。 |
| [可复用模型入口政策](./reusable-model-entry-points.md) | 调用 callable anchors、model-entry gap handling 和 reusable entry point types。 | 本文可识别 weak/noisy model anchor signal，但不做 model-entry inventory。 |
| [既有文档复用流程](./existing-doc-reuse-procedure.md) | 调用 reuse scan surfaces、result taxonomy 和 existing-doc reuse evidence。 | 本文不执行 new-problem reuse scan 或 candidate promotion。 |
| [重复概念与同主题共存治理](./duplicate-and-coexistence-policy.md) | 调用 duplicate/coexistence taxonomy、allowed outcomes 和 Decision Record。 | 本文只路由 repeated overlap，不执行 cleanup。 |
| [重命名、路径迁移与废弃治理](./rename-migration-policy.md)、[修订、重生成与版本连续性策略](./revision-regeneration-continuity-policy.md) 和 [文档生命周期状态](./lifecycle-states.md) | 调用 split、merge、successor、replacement、deprecation、archive、identity continuity、old-content handling 和 remaining access 规则。 | 本文不执行身份、生命周期或迁移动作。 |
| [Frontmatter schema 与 doc_id 身份规则](./frontmatter-schema.md) | 调用 baseline fields、YAML arrays、stable `doc_id` 和 unauthorized field 禁令。 | 本文所有 review/result/record 字段都留在正文或人类可读证据中。 |
| [docs/index.md 同步与导航治理规则](./index-synchronization-rules.md) | 调用 Index Impact outcomes、relative path、formal navigation entry 和 index-wide cleanup 边界。 | 本文不改变全局 index 格式，也不做 index-wide cleanup。 |
| [批量治理 Readiness Checklist](./batch-readiness-checklist.md) | 判断 topic-wide cleanup、network scan、batch relationship assignment、batch link/index normalization 是否越界。 | 本文不授权 batch work；batch network review 留给 Epic 6。 |
| [审查记录模板](../templates/review-record-template.md)、[完成汇报模板](../templates/completion-report-template.md) 和 [文档决策政策](./document-decision-policy.md) | 承载 review/completion/decision evidence、unverified/future-story handling 和 status impact。 | 本文不创建 actual review record 或 actual completion report。 |

本文的 owner entry point 是 [docs/index.md](../index.md) 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/network-boundary-and-decay-prevention.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Epic 5 首版 network boundary and decay prevention policy；Epic 6 后续仍会细化 batch network review、batch stale-link cleanup、batch relationship updates 和 batch governance records。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record
- source asset: docs/governance/network-boundary-and-decay-prevention.md
- trigger: Story 5.5 explicitly authorizes a canonical network boundary and decay prevention governance asset
- index file: docs/index.md
- section: ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: the asset is a formal governance policy with listed_in_docs_index navigation treatment
- validation result: target exists and relative link resolves
- unresolved risk: Epic 6 batch governance records remain future dependencies, recorded in open_questions
```

## Topic Health Review

Topic health review 是有界的人类可读治理检查，不是 generated score、network map、lint rule 或自动扫描。开始前必须记录 review input；不得先做全库扫描再回填结论。

最低输入包括：

- source topic / candidate topic；
- candidate asset；
- review trigger；
- allowed review scope；
- representative docs；
- neighboring topics/docs；
- known index entries；
- current boundary statement；
- owner/future story when scope cannot be resolved in the current task。

### Topic Health Signals

| Signal | Required meaning | Evidence to record | Allowed route |
| --- | --- | --- | --- |
| `topic_boundary_coherence` | Topic 内资产围绕可解释 reader question、concept family、asset class 或 governance purpose，而不是 catch-all bucket。 | Boundary statement、representative docs、excluded neighboring topics、why current grouping is coherent。 | `pass` 或继续保持当前 topic。 |
| `subtopic_split_candidate` | Topic 已承载稳定下位问题、读者路径、模型家族或维护边界，普通 linking 不再足够。 | Candidate subtopic、affected docs、split reason、why simple links are insufficient。 | Route to topic/path policy、rename/migration、batch readiness or future story。 |
| `misplaced_doc_candidate` | 文档的 concept、reader question、asset class、reuse context 或 index grouping 与当前 topic 不匹配。 | Current path/topic、expected path/topic、mismatch reason、routing owner。 | Route to topic/path policy and rename/migration; do not move silently。 |
| `catch_all_topic_drift` | Topic 正在吸收过多松散资产，reader question 和边界不再清楚。 | Repeated weak boundary signals、unrelated reader questions、noisy links、broad index grouping。 | Boundary clarification, subtopic split candidate, or batch readiness。 |
| `canonical_entry_ambiguity` | 多个资产看起来都像同一 reader question 的 primary entry。 | Affected assets、suspected canonical ambiguity、index / duplicate / coexistence impact。 | Duplicate/coexistence route and index synchronization review。 |
| `boundary_clarification_needed` | Topic 可保留，但需要明确 inclusion/exclusion wording。 | Missing boundary statement、affected docs、proposed clarification location。 | Targeted revision if authorized; otherwise defer with owner。 |
| `unverified_boundary` | 当前 review scope 不足以支持 topic health conclusion。 | Unchecked scope、blocker、owner/future story、completion wording limit。 | `hold_for_maxwell`、`defer_with_reason` 或 batch readiness。 |

Topic health review 可以记录问题和路由，但本 policy 本身不授权移动、拆分、合并、归档、废弃、批量 topic cleanup 或 full-repo topic sweep。若需要按规则选择多个文件、扫描全 topic、修复多个关系或重排索引，必须停止并进入 [批量治理 Readiness Checklist](./batch-readiness-checklist.md) 或 Epic 6。

## Weak, Dense, Noisy And Stale Link Signals

Weak/noisy/stale links 是 network-decay signals，不是 frontmatter 字段。每个 signal 至少必须记录 source asset、target asset、link location、target status、relation type、transfer object、why weak/noisy/stale、affected reader path 和 validation result。

| Signal | Meaning | Default handling |
| --- | --- | --- |
| `decorative_link` | Link 只是装饰、泛关联或“顺手参考”，没有 transfer evidence。 | `remove_link` if in scope; otherwise `annotate_boundary_or_transfer` or `defer_with_reason`。 |
| `keyword_only_relation` | Link 只因 shared term、标题相似或关键词相同而存在。 | Reclassify with valid relation/transfer object, or remove/defer。 |
| `same_topic_only_relation` | Link 只因位于同一 topic，没有说明 relationship meaning。 | Add boundary/reuse reason or remove。 |
| `related_docs_as_navigation_dump` | `related_docs` 堆积多个资产，但没有 target status、relation type 或 transfer object。 | Prune/reclassify in current scope; route batch-shaped cleanup to Epic 6。 |
| `bidirectional_without_reason` | 双向链接存在，但没有说明为什么两边都需要导航。 | Add one-way/bidirectional reason or remove one side within authorized scope。 |
| `misclassified_relation_type` | relation type 与实际迁移对象不匹配。 | Route to related docs taxonomy and reclassify if current file scope allows。 |
| `unverified_planned_target` | planned/missing target 被写得像当前可用资产。 | Convert to open question/future dependency or remove misleading link。 |
| `missing_transfer_object` | Link 没有说明迁移 structure、boundary、model、judgment、failure mode 或 verification method。 | Annotate transfer object or reject the link。 |
| `stale_successor_or_lifecycle_link` | Link 指向过时 successor、deprecated current target 或 lifecycle-confusing target。 | Route to link maintenance, rename/migration and lifecycle policies。 |
| `topic_path_mismatch` | Link target 的 topic/path 与关系说明冲突。 | Route to topic/path policy, link maintenance and index synchronization。 |
| `over_dense_hub_linking` | 某资产成为低价值链接 hub，reader path 被噪声淹没。 | Require relation/transfer evidence for each edge; batch cleanup requires batch readiness。 |

Allowed handling labels:

- `remove_link`
- `reclassify_relation`
- `annotate_boundary_or_transfer`
- `convert_to_open_question`
- `route_to_link_maintenance`
- `route_to_related_docs_taxonomy`
- `route_to_duplicate_coexistence`
- `route_to_rename_migration`
- `route_to_batch_readiness`
- `hold_for_maxwell`
- `defer_with_reason`
- `not_authorized`

`remove_link`、`reclassify_relation` 和 `annotate_boundary_or_transfer` 只能发生在当前 story 授权的 changed-file scope 内。超过 named files、需要 generated inventory、all-doc inbound scan、topic-wide stale cleanup、index-wide cleanup 或 batch relationship assignment 时，必须停止并路由到 batch readiness / Epic 6。实施本 story 时不得顺手清理现有网络中的 dense/noisy links。

## Repeated Overlap Routing

Repeated overlap 是治理路由信号，不是直接执行 merge/split/deprecate/archive 的授权。它说明多个资产之间的边界、入口或复用角色正在变得难以判断，需要把问题交给 duplicate/coexistence、rename/migration、revision continuity、lifecycle 或 index synchronization。

| Overlap signal | Meaning | Route |
| --- | --- | --- |
| `same_reader_question_repeated` | 多个资产反复回答同一 reader question。 | `merge_candidate`、`boundary_clarification` 或 explicit coexistence review。 |
| `shared_canonical_concept` | 多个资产都在声明同一 concept authority。 | duplicate/coexistence plus frontmatter/doc_id review。 |
| `repeated_boundary_conflict` | 同一 inclusion/exclusion conflict 在多处出现。 | targeted revision、boundary clarification、hold for Maxwell。 |
| `parallel_index_authority` | Index 或导航暗示多个 primary entry。 | index synchronization and duplicate/coexistence route。 |
| `narrower_depth_hidden_as_duplicate` | 深入/窄化资产被误当 duplicate，或 duplicate 被误当 deeper dive。 | deeper-dive document or narrower-document decision。 |
| `stale_replacement_overlap` | replacement、merge、split 后旧/新资产仍持续重叠。 | rename/migration and lifecycle route。 |
| `copied_related_docs_reason` | 多处复制同一 related-doc reason，但没有独立 transfer evidence。 | relationship reclassification or link cleanup route。 |
| `distinct_reuse_context_unproven` | Candidate docs 一直无法证明不同复用场景。 | duplicate/coexistence, reject, hold, or defer。 |

Allowed outcomes:

- `merge_candidate`
- `split_candidate`
- `deprecate_or_archive_candidate`
- `boundary_clarification`
- `deeper_dive_document`
- `explicit_coexistence`
- `hold_for_maxwell`
- `defer_with_reason`
- `not_authorized`

Repeated overlap decisions must call [重复概念与同主题共存治理](./duplicate-and-coexistence-policy.md) for duplicate / adjacent / narrower / coexistence / reject. Split、merge、successor、replacement、deprecation、archive、identity continuity、old-content handling 和 remaining access must call [重命名、路径迁移与废弃治理](./rename-migration-policy.md), [修订、重生成与版本连续性策略](./revision-regeneration-continuity-policy.md) and [文档生命周期状态](./lifecycle-states.md). This policy only routes; it does not execute those changes.

## Navigation Relevance Evidence

`docs/index.md` 是 formal discovery/navigation signal，不承担每个条目的完整 relationship explanation。Index entry 能说明“目标可发现”，不能单独证明“为什么这个目标相关、迁移什么 reasoning、边界在哪里”。当正文、治理资产、review evidence 或 completion evidence 提供 curated next-doc navigation 时，必须给出 navigation relevance evidence。

Minimum evidence:

- next document / target asset；
- target status；
- relation type from [related docs taxonomy](./related-docs-taxonomy.md)；
- transfer object；
- why the next document is relevant；
- reasoning transfer：迁移 structure、boundary、model、judgment、failure pattern、source/time discipline 或 verification method 中的哪一类；
- boundary limit：什么不能迁移；
- one-way or bidirectional reason；
- next verification or route if uncertain。

`see also`、标题相似、同 topic、路径相邻、关键词相同、index 邻近或“相关文档”字样都不能单独证明 navigation relevance。缺少 relation type 或 transfer object 时，应改写为有证据的导航句、转成 open question，或删除/延后该链接。

### Adjacent Governance Navigation

| Next document | Why relevant | Reasoning transfer | Boundary kept here |
| --- | --- | --- | --- |
| [related docs taxonomy](./related-docs-taxonomy.md) | 判断 link/relationship 是否有意义。 | target status、relation type、transfer object、meaningful-link evidence。 | 本文只识别 network-decay signals，不改 taxonomy。 |
| [link maintenance policy](./link-maintenance-policy.md) | 处理 changed-file links、bounded inbound/outbound review 和 link conflict。 | existence/path/topic/meaning checks、one-way reason、Link Maintenance Record。 | 本文不执行 broad link scan 或 repair。 |
| [reusable model entry points](./reusable-model-entry-points.md) | 判断已有文档是否有可调用模型入口。 | callable anchors、model-entry gap、verification method。 | 本文不做 model-entry inventory。 |
| [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) | 新问题进入前优先找 existing docs。 | scan surfaces、result taxonomy、reuse evidence。 | 本文不执行 new-problem scan。 |
| [duplicate and coexistence policy](./duplicate-and-coexistence-policy.md) | repeated overlap 需要 duplicate/coexistence 判断。 | merge/split/coexist/reject routing evidence。 | 本文不执行 cleanup。 |
| [rename/migration policy](./rename-migration-policy.md) | split/merge/deprecation/archive 会影响身份和路径。 | successor/replacement、identity continuity、remaining access。 | 本文不改身份或生命周期。 |
| [index synchronization rules](./index-synchronization-rules.md) | primary entry 或 index grouping 受影响时要处理导航。 | Index Impact outcome、relative path、formal navigation entry。 | 本文不改变全局 index 格式。 |
| [batch readiness checklist](./batch-readiness-checklist.md) | network cleanup 一旦变成多文件规则选择就必须先做 readiness。 | scope、target set、conflicts、stop criteria、recovery evidence。 | 本文不授权 batch execution。 |

## Network Boundary / Decay Review Record

`Network Boundary / Decay Review Record` 是 copyable human-readable record，不是 schema、frontmatter extension、lint rule、generator input、database row、network scanner output 或 automation contract。它可以落在 target body、review evidence、completion evidence、story Dev Agent Record、future runbook record 或明确命名的 network-boundary note 中。

```markdown
## Network Boundary / Decay Review Record

- source topic / asset:
- review trigger:
- allowed review scope:
- topic boundary statement:
- representative docs checked:
- neighboring topics/docs:
- topic health result: topic_boundary_coherence | subtopic_split_candidate | misplaced_doc_candidate | catch_all_topic_drift | canonical_entry_ambiguity | boundary_clarification_needed | unverified_boundary
- subtopic split signal:
- misplaced doc signal:
- weak/dense/noisy/stale link signals:
- link handling action: remove_link | reclassify_relation | annotate_boundary_or_transfer | convert_to_open_question | route_to_link_maintenance | route_to_related_docs_taxonomy | route_to_duplicate_coexistence | route_to_rename_migration | route_to_batch_readiness | hold_for_maxwell | defer_with_reason | not_authorized
- repeated overlap signal:
- overlap routing decision: merge_candidate | split_candidate | deprecate_or_archive_candidate | boundary_clarification | deeper_dive_document | explicit_coexistence | hold_for_maxwell | defer_with_reason | not_authorized
- navigation relevance evidence:
- relation type:
- transfer object:
- reasoning transfer:
- boundary limit:
- index impact:
- related_docs impact:
- source/time impact:
- lifecycle impact:
- batch-shaped risk:
- owner/future story:
- validation result: pass | fail | not_applicable | unverified
```

Record 字段不得拆成未经授权的 frontmatter fields。若未来确实需要 schema 化 topic health、network health、link density、overlap status、navigation relevance 或 batch network review，必须由 frontmatter schema、version governance、Epic 6 或 Maxwell 明确授权。

## Batch Boundary And Stop Conditions

当前 task 只能处理授权文件范围内的 policy definition、index entry 和必要 direct-consistency wording。出现以下任一情况时，必须停止并路由到 [批量治理 Readiness Checklist](./batch-readiness-checklist.md) 或 Epic 6：

- 需要按规则选出多个 target files。
- 需要 full-repo network scan、topic-wide sweep 或 all-doc inbound/outbound review。
- 需要批量删除、重分类、补注 `related_docs` 或正文链接。
- 需要 generated inventory、network map、link density report、stale link cleanup note 或 network health report。
- 需要批量 merge/split/deprecate/archive/move/rename。
- 需要修改全局 frontmatter schema、index format、automation 或 tooling。

Stop condition 的 completion wording 必须具体说明：触发原因、受影响文件范围、为什么当前 story 不授权、建议 owner/future story，以及是否阻塞当前目标。

## Validation Checklist

实施或审查本文时，至少验证：

1. Target file path is `docs/governance/network-boundary-and-decay-prevention.md`.
2. Frontmatter required fields exist; `source_basis`、`related_docs`、`open_questions` use YAML arrays.
3. H1 matches frontmatter `title`.
4. Asset role、authority、scope、non-goals and adjacent asset boundaries are explicit.
5. Topic health review covers coherent boundary、subtopic split、misplaced docs、canonical ambiguity、catch-all drift、boundary clarification and unverified boundary.
6. Weak/dense/noisy/stale link signals include required evidence and allowed handling labels.
7. Repeated overlap routing includes merge/split/deprecate/boundary/deeper-dive/coexist/hold/defer/not-authorized outcomes and routes to duplicate/coexistence plus rename/migration/lifecycle policies.
8. Navigation relevance evidence explains target status、relation type、transfer object、why relevant、reasoning transfer、boundary limit and one-way/bidirectional reason.
9. `Network Boundary / Decay Review Record` remains human-readable body/review/completion/story evidence, not frontmatter.
10. `docs/index.md` contains the formal governance entry and the relative link resolves.
11. No actual topic health review、dense/noisy/stale cleanup、full-repo network scan、all-doc inbound/outbound repair、topic-wide cleanup、batch relationship assignment、batch `related_docs` update、index-wide cleanup、candidate promotion、document regeneration、merge/split/deprecate/archive/move/rename or generated report was performed.
12. No runtime code、package manifest、source/test directory、software test framework、schema、validator、generator、automation、scanner、index generator、CLI/API/UI/database/deployment/CI、runbook、actual review record or actual completion report was created.
