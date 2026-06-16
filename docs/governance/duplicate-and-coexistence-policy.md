---
doc_id: governance-duplicate-and-coexistence-policy
title: 重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策
concept: duplicate_and_coexistence_policy
topic: governance
depth_mode: standard
created_at: '2026-05-26T19:55:32+08:00'
updated_at: '2026-06-16T09:42:19+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/2-6-duplicate-and-coexistence-policy.md
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/governance-asset-boundary-policy.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_epic_2_governance_review_2026_06_16
applicability: formal_docs_duplicate_concept_same_topic_coexistence_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/governance-asset-boundary-policy.md
open_questions:
  - 如果 completion-report template 后续调整 Duplicate / Coexistence Decision Record summary 字段，是否需要同步本文的 Duplicate / Coexistence Decision Record 汇总方式？
  - related docs taxonomy 与 link maintenance policy 已建立；后续是否需要把 adjacent_link、supporting link、secondary reference 和 reuse path 的维护关系拆得更细？
  - Epic 6 建立 batch governance runbook 后，是否需要为批量重复扫描、批量 topic cleanup 和批量 link/index cleanup 建立独立执行记录？
---

# 重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 duplicate concept、near duplicate、overlapping concept、adjacent concept、narrower/deeper document、same-topic coexistence 和 topic-boundary protection 的 canonical governance asset。它回答的是：正式 `docs/` 资产或候选资产在创建、晋升、替换、修订、索引或保留时，如何判断是重复、相邻、窄化、可共存、应移动、应延后、应持有还是应拒绝。

本文适用于：

- `docs/{topic}/` 下的普通正式知识资产。
- `docs/methodology/` 下的方法论、模板、质量门禁、fixed prompt、playbook 和方法论支撑资产。
- `docs/governance/` 下的治理资产。
- `docs/_reports/` 下已经批准保留的正式报告入口。
- 准备从 `_bmad-output/`、对话输出、临时草稿、review output 或手工候选稿晋升为正式 `docs/` 资产的候选内容。
- 正式资产的 replacement、revision、merge、split、successor、deprecation、archive、index entry、secondary reference 和 topic/path 决策中出现的重复或共存风险。

本文补充而不替代以下资产：

- 主方法论 `docs/methodology/document-generation-methodology.md`：仍负责正式概念文档的新建、升级、审查和仓库集成主流程；本文只负责重复/共存风险的治理判断。
- `docs/methodology/concept-document-contract.md` 与来源纪律资产：仍负责概念文档合同、必需信息位点、来源/时间语境和真实世界锚点；本文不重建第二套内容质量规则。
- `docs/governance/frontmatter-schema.md`：负责 frontmatter baseline、YAML array、`doc_id` 身份和禁止未经授权全局字段。
- `docs/governance/topic-path-naming-policy.md`：负责 topic、path、filename、slug、asset class 和 path ownership。
- `docs/governance/candidate-promotion-checklist.md`：负责候选晋升门禁、Promotion Decision Record 和 silent overwrite 风险。
- `docs/governance/index-synchronization-rules.md`：负责 `docs/index.md` 同步触发器、index impact outcome 和 Index Impact Decision Record。
- `docs/governance/rename-migration-policy.md`：负责 rename、move、split、merge、replacement、successor、deprecation、archive、old-content handling 和 Migration Decision Record。
- `docs/governance/lifecycle-states.md`：负责 draft、reviewed、validated、maintained_asset、deprecated、archived 和 reactivation 语义。
- `docs/governance/batch-readiness-checklist.md`：负责批量治理 readiness、目标集、冲突暴露、停止条件和恢复策略。
- `docs/governance/governance-asset-navigation-policy.md`：负责治理资产 owner entry point 与 navigation treatment 基线。
- `docs/methodology/intake-and-intent-classification.md` 与 `docs/methodology/concept-document-quality-gate.md`：负责任务分类、停止条件、Hard Fail 和等价治理门禁。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/duplicate-and-coexistence-policy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

本文自身的 index impact decision record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/duplicate-and-coexistence-policy.md
- target title: 重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策
- target path: ./governance/duplicate-and-coexistence-policy.md
- target section: docs/index.md ## governance
- outcome: reviewed_existing_entry
- action taken: no index edit in this stabilization pass; existing governance entry verified
- reason: canonical governance asset for duplicate concept and same-topic coexistence handling is already listed
- validation result: existing target exists and relative index link resolves
- unresolved risk: index/link resolution risk none; Epic 6 batch execution records remain future dependencies
```

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：duplicate/coexistence taxonomy、allowed outcomes、Decision Record、same-topic coexistence、topic-boundary protection、相邻治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

## 职责边界与非目标

本文定义决策、证据、停止条件和验证规则；它不执行任何实际重复清理、合并、移动、重命名、废弃、归档、候选晋升、索引扫全库、topic normalization 或链接大修。

本文授权的输出只有本政策资产和本次直接相关的 `docs/index.md` 导航入口更新。它不授权：

- 实际 duplicate cleanup、batch duplicate scan、batch merge、batch rejection、topic-directory cleanup、multi-topic normalization、generated rewrite 或 broad link repair。
- 实际 candidate promotion、formal asset replacement、old-content deletion、successor switch、deprecation、archive 或 reactivation。
- 创建或重写 `docs/governance/related-docs-taxonomy.md`、`docs/governance/link-maintenance-policy.md`、review/decision/completion templates、related-doc/link/reuse/network governance assets 或 Epic 6 batch assets。
- 修改 `.agents/skills/`、planning artifacts、主方法论规则、quality gate semantics、prompt/template versions 或 sprint workflow contracts。
- runtime code、package manifest、source tree、software tests、CLI/API/UI/database/deployment/CI、lint/scoring automation、duplicate scanner、link scanner、index generator、migration script、topic normalizer 或 executable validation tooling。

本文也不创建新的全局 frontmatter 字段。不得为了表达重复、共存、决策、继任、合并来源、拆分来源或 canonical/secondary 状态，在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`path_policy_version`、`promotion_policy_version`、`duplicate_status`、`coexistence_status`、`decision_status`、`successor`、`merged_from`、`split_from`、`canonical_asset`、`secondary_reference`、`duplicate_of` 或类似字段。相关信息应写在正文、review evidence、completion evidence、story Dev Agent Record、review/completion templates、future runbook records 或显式 decision note 中。

## Decision taxonomy

使用本文时，必须先判定 duplicate/coexistence type。不要把不同关系混成“有点像”“同一主题”或“整理一下”。

| type | 含义 | 默认风险 |
| --- | --- | --- |
| `exact_duplicate` | 两个资产或候选基本回答同一 concept、同一 reader question、同一 reuse context，正文差异不能形成独立资产价值 | 平行 canonical authority、索引重复、维护分叉 |
| `near_duplicate` | 核心 concept 和用途高度重叠，但例子、标题、深度或局部角度不同 | 误以为是不同资产，实际应 merge、narrow 或 reject |
| `overlapping_concept` | 两个资产覆盖区间交叉，各自有部分独立范围 | 边界不清、读者不知道从哪里进入 |
| `adjacent_concept` | 两个资产相邻、互相解释或互为前置/后续，但不承担同一 canonical concept | 被误合并，或缺少合理 link/reuse path |
| `broader_asset` | 一个资产承担更宽的概念、框架或上位模型 | 下位内容被吸成 catch-all，导致广泛但不稳 |
| `narrower_or_deeper_asset` | 一个资产专门回答更窄、更深、更工程化或更场景化的问题 | 被误判为重复而丢失可复用深度 |
| `same_topic_valid_coexistence` | 同一 `topic` 下多篇资产长期共存，且 concept、purpose、reuse context 和 reader question 可区分 | 需要维护边界，防止未来 collapse |
| `misplaced_topic_overlap` | 资产看似与当前 topic 相邻，但真实 topic、asset class 或 reuse context 不属于拟议目录 | topic 目录变成 catch-all bucket |
| `index_duplicate_entry` | 同一资产或同一 concept 在 `docs/index.md` 出现多个 canonical 入口 | 平行入口、过期路径、重复权威 |
| `candidate_with_insufficient_distinction` | 候选声称是新资产，但无法证明与现有资产的 concept、purpose 或 reuse context 差异 | 不得晋升、索引或替换 |

同一 `topic` 不是重复证据。不同 `topic` 也不是不存在重复的证据。Reviewer 必须比较 concept、reader question、purpose、reuse context、maintenance boundary、`doc_id` 处理、path/index impact 和实际正文职责。

## Allowed outcomes

Duplicate/coexistence 风险必须明确落入允许 outcome。不得沉默创建、沉默晋升、沉默替换、沉默索引或用模糊文字掩盖未决状态。

主要 outcome 如下：

| outcome | 使用条件 | 最低证据 |
| --- | --- | --- |
| `merge` | 多个资产或候选应合并到一个 canonical target，或候选内容只适合作为既有资产的补充 | canonical target、source/old content disposition、`doc_id` treatment、index/link impact、Migration Decision Record evidence |
| `adjacent_link` | 资产概念不同但相邻，适合通过正文链接、`related_docs` 或 related-doc relationship taxonomy 连接 | relationship reason、link direction、expected reuse path、related-doc/link evidence if detailed taxonomy is needed |
| `narrower_document` | 候选或资产可成立，但必须缩窄为下位、更深或更具体文档，避免重复 canonical authority | narrower scope、parent/adjacent relationship、specific title/path/concept、authority boundary |
| `explicit_coexistence` | 同 topic 或相邻 topic 下多篇资产长期共存，且差异可维护 | distinct concept、purpose、reuse context、reader question、maintenance boundary、index/navigation treatment、future review trigger |
| `reject` | true duplicate、区分不足、topic 错置、来源/质量失败、没有 durable reuse context 或范围未授权 | rejection reason、affected assets、future owner if any |

支持 outcome 用于边界情况：

| outcome | 使用条件 |
| --- | --- |
| `move_or_reclassify_candidate` | 候选有价值，但真实 topic、asset class、report/template/runbook class 或 reuse context 不属于拟议位置 |
| `defer_with_reason` | 当前授权范围不覆盖、证据不足、需要 review/completion templates、related-doc/link/reuse/network governance、Epic 6 资产，或目标集构成 batch work |
| `hold_for_maxwell_confirmation` | canonical entry、身份、topic、path、old-content、index visibility 或共存理由需要 Maxwell 决策 |
| `not_authorized` | 请求会越过当前授权范围、workflow、治理政策或非软件边界 |

当 duplicate/coexistence 判断会导致 split、merge、successor、replacement、deprecation、archive、reactivation 或 old-content handling 时，必须调用 `docs/governance/rename-migration-policy.md`，不得把这些动作静默折叠进本文。本文决定“重复/共存关系如何判”，rename/migration policy 决定“身份、迁移、旧内容、继任和生命周期如何处理”。

当判断影响 `docs/index.md` secondary reference、duplicate entry、canonical entry 或 index visibility 时，必须调用 `docs/governance/index-synchronization-rules.md` outcome：`updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict`。不得用 secondary reference 制造平行 canonical entry。

## Duplicate / Coexistence Decision Record

每次正式资产或候选资产存在 duplicate/coexistence 风险时，必须留下 human-readable Duplicate / Coexistence Decision Record。它不是 executable schema、lint rule 或 frontmatter extension。

允许的 decision outcome 只有：

- `merge`
- `adjacent_link`
- `narrower_document`
- `explicit_coexistence`
- `reject`
- `move_or_reclassify_candidate`
- `defer_with_reason`
- `hold_for_maxwell_confirmation`
- `not_authorized`

最低记录字段如下：

| 字段 | 必须记录什么 |
| --- | --- |
| affected assets/candidates | 受影响的正式资产、候选来源、旧路径、新路径或 index entries |
| existing canonical path if any | 现有 canonical asset 路径；没有时说明未发现或不适用 |
| proposed path if any | 候选或新资产拟议路径；未决定时说明 blocker |
| duplicate/coexistence type | 本文 taxonomy 中的 type |
| chosen outcome | 上述允许 outcome 之一 |
| reason | 为什么选择该 outcome |
| rejected alternatives | 为什么不 merge、不 link、不 narrow、不 coexist 或不 reject |
| distinct concept | 若保留或共存，必须说明独立 concept；若不存在，说明为 reject/merge 原因 |
| distinct purpose | 资产服务的目的、判断任务或读者用途 |
| distinct reuse context | 后续何种问题会回到该资产，而不是回到相邻资产 |
| `doc_id` treatment | 保留、创建新身份、归并、拒绝、持有或交给 `docs/governance/rename-migration-policy.md` |
| topic/path impact | topic、目录、filename、asset class 和 catch-all 风险 |
| index impact | `docs/governance/index-synchronization-rules.md` outcome 和 index action |
| related-doc/link impact | changed-file links、body links、`related_docs`、discoverable inbound references 和 unresolved links |
| old-content handling | merge/replacement 涉及时的旧内容处理；不适用时说明原因 |
| unresolved risks | 仍未解决的身份、topic、index、link、source、quality、lifecycle 或 batch 风险 |
| validation result | Markdown/frontmatter/link/index/governance validation 结果 |
| owner/follow-up dependency | Maxwell、rename/migration policy、review/completion templates、related-doc/link/reuse/network governance、Epic 6 或其他后续 owner |

记录可以写在：

- 目标资产正文。
- review evidence。
- completion evidence。
- story Dev Agent Record。
- Review/completion templates。
- future runbook records。
- 明确命名的 decision note。

这些字段不得作为新的全局 frontmatter 字段散落到正式资产中。

本文的 Decision Record 与 Promotion Decision Record、Index Impact Decision Record、Migration Decision Record 是互补关系：

- Promotion Decision Record 负责 candidate 是否可以进入 formal `docs/`、目标路径、身份、质量、索引和风险是否成立。
- Index Impact Decision Record 负责 `docs/index.md` 是否更新、排除、引用、延后或阻塞。
- Migration Decision Record 负责 identity continuity、old-content handling、successor/replacement、link/index/lifecycle impact、split/merge/deprecation/archive 和 stop/defer reason。
- Duplicate / Coexistence Decision Record 负责重复、相邻、窄化、共存、topic-boundary 和 reject 判断。

一次变更可以同时需要多种记录。不得用本文记录替代 promotion gate、index impact classification 或 migration evidence。

推荐的人工记录形状如下：

```text
Duplicate / Coexistence Decision Record

- affected assets/candidates:
- existing canonical path if any:
- proposed path if any:
- duplicate/coexistence type:
- chosen outcome:
- reason:
- rejected alternatives:
- distinct concept:
- distinct purpose:
- distinct reuse context:
- doc_id treatment:
- topic/path impact:
- index impact:
- related-doc/link impact:
- old-content handling:
- unresolved risks:
- validation result:
- owner/follow-up dependency:
```

如果 distinction、canonical path、topic ownership、old-content handling、index visibility 或 owner 决策未决，允许结果只有 `defer_with_reason`、`hold_for_maxwell_confirmation` 或 `not_authorized`。未决时不得创建、晋升、替换、插入索引、提高 `quality_status`、宣称 reviewed/validated，或把候选当作长期正式入口。

## Same-topic coexistence rules

同一 `topic` 下多篇资产可以长期共存，但共存必须被证明，而不是默认接受。

有效共存必须同时证明：

| 要素 | 必须回答的问题 |
| --- | --- |
| distinct `concept` | 这篇资产的核心对象是什么，为什么不是相邻资产的别名或展开段落？ |
| distinct purpose | 读者使用它是为了做什么判断、解释、迁移或复用？ |
| distinct reuse context | 后续遇到什么问题会回到它，而不是回到已有资产？ |
| expected reader question | 它回答的读者问题是否不同于同 topic 下其他资产？ |
| maintenance boundary | 未来修订时哪些内容归它，哪些内容应留给相邻资产？ |

同 topic 与 same concept 是两件事。`docs/computer-systems/memory-order.md` 与其他并发资产可以属于同一 topic，但仍需按 concept、purpose 和 reader question 区分。不同 topic 也不能自动消除重复风险；如果一个治理资产和一个方法论资产实际都承担同一规则入口，仍可能构成重复权威。

如果未来新增资产、重命名、扩写、候选晋升或 link/index 调整让上述差异坍塌，必须触发 re-review。Re-review 结果只能是维持 `explicit_coexistence`、改为 `merge`、改为 `narrower_document`、改为 `adjacent_link`、`move_or_reclassify_candidate`、`reject`、`defer_with_reason` 或 `hold_for_maxwell_confirmation`。

## Topic boundary and catch-all prevention

Topic directory 是导航和复用边界，不是“没有更好地方就放这里”的桶。Reviewer 必须阻止 topic、index section 或 governance/methodology special path 吸收相邻但无明确归属的资产。

当候选或资产出现以下信号时，必须检查 topic boundary：

- 标题很宽，但正文只覆盖一个窄场景。
- 正文混合多个 topic，只因为某个段落相关就放入拟议目录。
- 候选属于 methodology、governance、report、template、runbook 或 workflow output，却被放成普通 topic 文档。
- 候选只是解释一个已有资产中的 subsection，却声称是新 canonical asset。
- 同一 topic 下已有资产能承担主要 reader question，候选只提供重复说明。
- 不同 topic 下已有资产实际承担同一 concept 或 same reuse context。
- Index section 因为找不到更准位置而被用作 catch-all。

处理规则如下：

| 情况 | 应用 outcome |
| --- | --- |
| 资产有独立价值，但拟议 topic/path 错误 | `move_or_reclassify_candidate` |
| 资产太宽，容易吞掉相邻概念 | `narrower_document` 或 `defer_with_reason` |
| 资产只是相邻概念支撑材料 | `adjacent_link` |
| 资产与现有 canonical concept 不可区分 | `merge` 或 `reject` |
| 资产 topic ownership 不清 | `defer_with_reason` 或 `hold_for_maxwell_confirmation` |
| 资产没有 durable reuse context | `reject` |
| 当前请求会变成批量 topic cleanup | 按 batch readiness 停止 |

不允许用“暂时放这里”“以后再整理”“目录还空着”“同主题都可以放一起”作为正式 path/topic 决策理由。无法确认归属时，应把问题记录为 open question、defer、hold 或 rejection，而不是创建一个会长期误导索引的入口。

## Merge, link, narrow, keep and reject handling

### `merge`

`merge` 适用于 true duplicate、near duplicate、同一 canonical concept 的多入口、候选只能补充既有资产、或多个资产需要归并为一个明确 target 的场景。

选择 `merge` 时必须记录：

- canonical target。
- 被合并来源和 source/old content disposition。
- 原 `doc_id` 和目标 `doc_id` 的处理。
- 旧路径、旧标题、旧 topic、旧 index entry 和旧 link 的影响。
- `related_docs`、正文链接、changed-file links 和 discoverable inbound references。
- Migration Decision Record，尤其是 old-content handling、successor/replacement path、index impact 和 lifecycle/status impact。

未完成 rename/migration evidence 时，不得实际执行正式资产 merge、删除旧内容或宣称 migration 完成。

### `adjacent_link`

`adjacent_link` 适用于概念不同但有明确相邻关系的资产。它不是重复处理的折中词，而是表示“保留独立资产，并建立可解释的复用路径”。

选择 `adjacent_link` 时必须记录：

- relationship reason：为什么相邻而非重复。
- link direction：从哪篇资产链接到哪篇资产，是否双向。
- expected reuse path：读者在什么问题中从一篇跳到另一篇。
- 是否只需要当前正文链接，是否应按 `docs/governance/related-docs-taxonomy.md` 记录 relation type / transfer object，或是否需要按 `docs/governance/link-maintenance-policy.md` 记录 inbound/outbound scope、one-way reason 和 link impact evidence。

即使 link maintenance policy 已落地，也不得借 `adjacent_link` 大规模建立全库关系网或批量维护 related docs。只允许当前变更范围内必要、可验证、目标存在的链接；关系类型和 meaningful-link evidence 使用 `docs/governance/related-docs-taxonomy.md`，link impact evidence 使用 `docs/governance/link-maintenance-policy.md`。

### `narrower_document`

`narrower_document` 适用于候选或资产有独立价值，但必须收窄到下位、深度、场景化、工程化或特定判断问题，才能避免重复已有 canonical authority。

选择 `narrower_document` 时必须记录：

- narrower scope。
- parent 或 adjacent asset relationship。
- title、path、filename 和 `concept` 如何表达窄化后的对象。
- 哪些内容不能再覆盖，以免重复上位资产。
- index/navigation treatment。
- 未来何时复核边界是否重新变宽。

窄化后仍不能证明 distinct concept、purpose 和 reuse context 的，应改为 `merge`、`reject`、`defer_with_reason` 或 `hold_for_maxwell_confirmation`。

### `explicit_coexistence`

`explicit_coexistence` 适用于同 topic 或相邻 topic 下多篇资产长期保留，且每篇资产的 concept、purpose、reuse context、reader question 和 maintenance boundary 可区分。

选择 `explicit_coexistence` 时必须记录：

- 每篇资产的 distinct concept。
- 每篇资产的 distinct purpose。
- 每篇资产的 distinct reuse context。
- 各自 index/navigation treatment。
- 是否存在 secondary reference 风险。
- future review trigger：什么变化会要求重新判断共存。

`explicit_coexistence` 不是“都留着再说”。如果不能写清 why coexistence is valid，必须使用其他 outcome。

### `reject`

`reject` 适用于候选或拟议入口不应进入正式资产体系的场景。允许原因包括：

- true duplicate，且没有值得合并的新增内容。
- insufficient distinction，无法证明独立 concept、purpose 或 reuse context。
- misplaced topic，且当前没有可授权的 reclassification。
- source、quality、frontmatter、identity、link 或 index evidence 失败。
- 没有 durable reuse context。
- 请求越过当前授权范围、workflow、future-story boundary 或非软件边界。

Reject 必须记录原因、受影响资产、是否有后续 owner、是否可作为候选素材保留，以及是否需要 `defer_with_reason` 或 `hold_for_maxwell_confirmation` 替代直接拒绝。

## Link, related-docs, index and lifecycle impact

Duplicate/coexistence 决策只要影响正式资产，就必须检查链接、related docs、索引和生命周期。

最低检查项如下：

- Changed-file links：本次修改文件中的相对链接是否仍能解析。
- Body links：正文内链是否指向正确 canonical、adjacent 或 successor target。
- `related_docs`：frontmatter list 是否只包含存在且被允许引用的正式资产或受控报告入口。
- Discoverable inbound references：容易通过文本搜索发现的入链是否会因 merge、narrow、move、reject 或 coexistence 改变含义。
- `docs/index.md`：使用 `docs/governance/index-synchronization-rules.md` outcome 分类为 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict`。
- Lifecycle/status：如果导致 rejection、deprecation、archive、replacement、lowered visibility 或 reactivation，按 `docs/governance/rename-migration-policy.md` 和 lifecycle policy 记录证据。

Frontmatter 更新只能反映实际证据。允许根据实际变更更新 title、topic、path-related claims、`updated_at`、`source_basis`、`time_context`、`quality_status`、`related_docs`、`open_questions`、`prompt_version` 或 `template_version`。不得用未经授权的 duplicate/coexistence metadata 字段解决本地不确定性。

如果关键 link、related-doc、index 或 lifecycle evidence 无法在当前授权范围内完成，必须记录 `defer_with_reason`、`hold_for_maxwell_confirmation`、`blocked_index_policy_conflict` 或 `not_authorized`，不得宣称处理完成。

## Batch and follow-up boundaries

一个明确 target asset 或 candidate 的一个明确 duplicate/coexistence decision，不自动构成 batch work。只要目标、风险、链接、索引和 owner 都局部明确，可以按本文记录并执行。

以下情况必须先调用 `docs/governance/batch-readiness-checklist.md`，不得直接执行：

- broad duplicate scanning。
- topic-directory cleanup。
- index-wide duplicate-entry cleanup。
- generated rewrite。
- multi-topic normalization。
- broad link repair。
- batch rejection。
- batch merge。
- 由规则自动选出的 target set。
- 任何需要扫描、重写、移动、合并或拒绝多个资产的操作。

相邻归属边界如下：

- `docs/templates/review-record-template.md`、`docs/governance/document-decision-policy.md`、`docs/governance/rework-loop-examples.md` 和 `docs/templates/completion-report-template.md` 负责 review record、document decision、rework loop 和 completion report templates。
- `docs/governance/related-docs-taxonomy.md` 负责 detailed related-doc relationship taxonomy；`docs/governance/link-maintenance-policy.md` 负责 general link maintenance policy；`docs/governance/reusable-model-entry-points.md` 负责 reusable model entry points；`docs/governance/existing-doc-reuse-procedure.md` 负责 new-problem reuse scan 和 suspected duplicate/coexistence routing；`docs/governance/network-boundary-and-decay-prevention.md` 负责 network boundary assets。
- Epic 6 负责 batch governance runbook、batch review record 和 batch completion report。

本文可以引用这些 owner 作为 deferred dependency，但不实现它们的资产、模板、runbook 或自动化。

## Stop and escalation conditions

出现以下情况时，必须停止、延后或请求 Maxwell 确认：

- 无法判断 exact duplicate、near duplicate、overlap、adjacent、narrower/deeper 或 valid coexistence。
- canonical entry point 不清。
- 同一 `doc_id`、同一 concept 或同一 index entry 出现平行权威风险。
- same-topic coexistence 不能证明 distinct concept、purpose、reuse context、reader question 和 maintenance boundary。
- topic directory、index section 或 governance/methodology special path 正在变成 catch-all bucket。
- merge、split、successor、replacement、deprecation、archive 或 old-content handling 需要 rename/migration evidence，但当前未授权。
- secondary reference 或 duplicate index entry 会制造 parallel canonical entry。
- 请求需要 broad duplicate scan、batch cleanup、batch link repair、topic normalization 或 executable tooling。
- 需要新增未经授权 frontmatter 字段才能表达本地状态。
- quality/status/lifecycle 结论会暗示未完成的 review、validation、migration、coexistence 或 promotion。
- 当前授权范围、workflow、Maxwell 指令或非软件边界未授权该动作。

允许的停止结果是 `defer_with_reason`、`hold_for_maxwell_confirmation`、`not_authorized`、`blocked_index_policy_conflict` 或按 rename/migration、lifecycle、batch policy 进入对应处理。不得用模糊 completion note 代替明确停止理由。

## Validation checklist and maintenance triggers

使用或维护本文时，至少检查：

1. Task type、asset level、applicable rules、allowed file scope、expected output artifact、validation evidence 和 stop conditions 已声明。
2. Target frontmatter 使用 baseline fields，`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
3. 未新增 unauthorized global frontmatter fields。
4. H1、frontmatter `title`、`concept`、`topic`、path、`doc_id` 和正文角色一致。
5. 正文说明本文 role、authority、scope、owner entry point、navigation treatment、relationship to main methodology 和 relationship to adjacent Epic 2 governance assets。
6. Taxonomy 覆盖 exact duplicate、near duplicate、overlap、adjacent、broader、narrower/deeper、same-topic valid coexistence、misplaced-topic overlap、index duplicate entry 和 insufficient distinction。
7. Allowed outcomes 覆盖 `merge`、`adjacent_link`、`narrower_document`、`explicit_coexistence`、`reject`、`move_or_reclassify_candidate`、`defer_with_reason`、`hold_for_maxwell_confirmation` 和 `not_authorized`。
8. Duplicate / Coexistence Decision Record 字段完整，并与 Promotion、Index Impact 和 Migration records 的边界清楚。
9. Same-topic coexistence、topic-boundary、catch-all prevention、merge/link/narrow/keep/reject、link/index/lifecycle impact、batch boundary 和 stop conditions 都有可人工执行的操作规则。
10. `docs/index.md` treatment 已更新或有明确 outcome。
11. Changed-file links、body links 和 `related_docs` targets 存在，或缺口记录为 open questions / defer / hold。
12. Lifecycle/quality-status vocabulary 与当前治理资产一致，不伪造 review、validation、migration、promotion 或 lifecycle evidence。
13. 未创建 runtime code、software tests、CLI/API/UI/database/deployment/CI、scanner、generator、script、automation、review/completion templates、related-doc/link/reuse/network governance assets 或 Epic 6 batch assets。

本文需要在以下事件后复核：

- Rename/migration policy、duplicate/coexistence policy 或相邻 Epic 2 资产调整了 identity、migration、index 或 topic/path rules。
- Review/decision/completion record templates 发生实质字段或 vocabulary 变更。
- Related docs taxonomy、link maintenance policy、reusable model entry points、existing-doc reuse procedure 或 network boundary / decay prevention policy 更新。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、batch duplicate cleanup 或 broad link/index automation。
- 正式资产中反复出现 duplicate/coexistence、secondary reference、topic catch-all 或 index duplicate entry 风险。

本文不授权批量执行上述维护触发。触发点只说明何时应重新审查本政策或创建后续治理任务。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [Topic、文件命名与路径归属策略：正式 docs 资产的位置、命名与一致性规则](./topic-path-naming-policy.md)
- [候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁](./candidate-promotion-checklist.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响](./rename-migration-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling](./related-docs-taxonomy.md)
- [链接维护政策：正文链接、related_docs、入链影响与失效链接处置](./link-maintenance-policy.md)
- [可复用模型入口治理：稳定入口、边界、迁移提示与跨文档复用路径](./reusable-model-entry-points.md)
- [既有文档复用流程：新问题先检索、匹配、复用、再决定是否新建](./existing-doc-reuse-procedure.md)
- [网络边界与退化预防：限制链接噪声、关系漂移与维护失控](./network-boundary-and-decay-prevention.md)
- [Review Record Template](../templates/review-record-template.md)
- [Completion Report Template](../templates/completion-report-template.md)
- [文档决策政策：接受、修订、拒绝与延后记录的治理规则](./document-decision-policy.md)
- [返工循环示例：从审查发现到修订记录的闭环样例](./rework-loop-examples.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
- [Sidecar 边界政策：正式文档、临时记录、草稿与旁注的归属规则](./sidecar-boundary-policy.md)
- [Legacy Migration Guide：旧文档迁移、保留、归档与兼容处理](./legacy-migration-guide.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](./agent-behavior-constraints.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](../methodology/concept-document-contract.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](../methodology/governance-asset-boundary-policy.md)
