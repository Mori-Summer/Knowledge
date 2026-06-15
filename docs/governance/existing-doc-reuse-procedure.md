---
doc_id: governance-existing-doc-reuse-procedure
title: 既有文档复用流程：new-problem scan、exact/adjacent/prereq/contrast results 与 reuse evidence
concept: existing_doc_reuse_procedure
topic: governance
depth_mode: standard
created_at: '2026-05-30T15:53:15+08:00'
updated_at: '2026-06-15T15:46:07+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/5-4-existing-doc-reuse-procedure.md
  - _bmad-output/implementation-artifacts/5-3-reusable-model-entry-points.md
  - _bmad-output/implementation-artifacts/5-2-link-maintenance-policy.md
  - _bmad-output/implementation-artifacts/5-1-related-docs-taxonomy.md
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/candidate-promotion-checklist.md
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
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
time_context: phase_5_epic_5_existing_doc_reuse_procedure_2026_05_30
applicability: formal_docs_new_problem_existing_doc_reuse_scan_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/candidate-promotion-checklist.md
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
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
open_questions:
  - network boundary and decay prevention policy 已建立后，是否需要继续把 weak/noisy reuse results 映射到更专门的 Epic 6 batch network review records？
  - Epic 6 建立 batch governance records 后，是否需要为批量 existing-doc reuse scan 提供专门 runbook 或 batch review record？
  - 后续是否需要把 Existing Doc Reuse Scan Record 与 review/completion templates 的 specialized records summary 进一步对齐？
---

# 既有文档复用流程：new-problem scan、exact/adjacent/prereq/contrast results 与 reuse evidence

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 new problem 进入正式知识库前，如何优先复用 existing docs 的 canonical governance asset。它回答的是：面对一个新问题、新文档候选、修订候选或审查任务时，Agent / reviewer 应该按什么顺序检查 `docs/index.md`、topic directories、candidate docs、related docs、neighboring concepts、duplicate/coexistence candidates 和 reusable model entry points；如何把扫描结果区分为 exact match、adjacent concept、prerequisite、contrast、deeper dive、reusable model candidate、duplicate/coexistence candidate、revision opportunity 或 no useful existing asset；以及如何记录实际复用的 model、boundary、example、source/time context、open question、failure pattern、verification method、decision frame 或 relationship evidence。

本文适用于：

- 新问题进入 `docs/` 前的 existing-doc reuse scan。
- 新建正式文档、升级正式文档、审查正式文档、候选晋升、duplicate/coexistence decision、review/completion evidence 或 story Dev Agent Record 中的复用证据记录。
- 已经有候选 topic、candidate concept、known adjacent docs 或可能 existing answer 时的窄范围扫描。
- 需要证明“复用了哪份已有文档的哪个模型/边界/例子/来源/open question”的人类可读记录。

本文不适用于：

- 执行 full-repo reuse scan、topic-wide cleanup、batch duplicate scan、broad link/index repair、network health review、existing-doc inventory、candidate promotion、document regeneration、batch relationship assignment、stale-link cleanup、batch `related_docs` update 或 index-wide cleanup。
- 执行 merge、split、reject、move、deprecate、archive、successor/replacement switch、candidate promotion 或 quality status change。
- 新增 `reuse_status`、`scan_status`、`scan_result`、`candidate_docs`、`reuse_entry_points`、`exact_match`、`adjacent_docs`、`duplicate_candidates`、`network_health`、`schema_version` 或类似 frontmatter fields。
- 创建 runtime code、package manifest、`src/`、`tests/`、software tests、CLI/API/UI/database/deployment/CI、machine-readable schema、validator、generator、search tool、link scanner、model scanner、duplicate scanner、index generator 或 automation。

本文补充而不替代以下资产：

| Adjacent asset | 本文调用它做什么 | 本文保留的边界 |
| --- | --- | --- |
| [输入摄入与任务意图判定](../methodology/intake-and-intent-classification.md) | 判定 task type、asset level、candidate concept、allowed file scope 和缺失输入处理。 | 本文不重写 intake taxonomy。 |
| [统一概念文档规范](../methodology/document-generation-methodology.md) | 约束正式文档的新建、升级、审查和仓库集成主流程。 | reuse scan 不能替代正式生成/升级流程。 |
| [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md) | 判断模型、边界、例子、来源、迁移和质量证据是否足够。 | 本文不伪造 quality gate pass。 |
| [related docs taxonomy](./related-docs-taxonomy.md) | target status、relation type、transfer object、meaningful-link evidence 和 Relationship Record。 | 本文不改写 relation type 语义。 |
| [link maintenance policy](./link-maintenance-policy.md) | changed-file link checks、one-way reason、fragment/anchor validation 和 bounded inbound/outbound review。 | 本文不执行 broad link maintenance。 |
| [reusable model entry points policy](./reusable-model-entry-points.md) | `core_model`、`boundary_rule`、`decision_frame`、`failure_pattern`、`verification_method`、New Problem To Existing Model Mapping 和 Reusable Model Entry Point Record。 | 本文负责找候选和分类结果；callable anchor 证据仍由该政策负责。 |
| [duplicate/coexistence policy](./duplicate-and-coexistence-policy.md) | exact/near duplicate、overlap、same-topic coexistence、canonical ambiguity、allowed outcomes 和 Decision Record。 | 本文只记录 suspected duplicate/coexistence result，不执行 cleanup。 |
| [candidate promotion checklist](./candidate-promotion-checklist.md) | 候选进入正式 `docs/` 的 promotion gate。 | reuse scan 结果不能自动晋升候选或发布新文档。 |
| [frontmatter schema](./frontmatter-schema.md) | baseline fields、YAML arrays、stable `doc_id` 和 unauthorized field 禁令。 | 本文所有 scan/result/record 字段都留在正文或人类可读证据中。 |
| [index synchronization rules](./index-synchronization-rules.md) | Index Impact outcomes、relative path、formal navigation entry 和 index-wide cleanup 边界。 | 本文只同步本资产 index entry，不做 index-wide cleanup。 |
| [review record template](../templates/review-record-template.md), [completion report template](../templates/completion-report-template.md), [document decision policy](./document-decision-policy.md) | review/completion/decision evidence 的承载位置、状态词和未验证/阻塞项处理。 | 本文不创建 actual review record 或 actual completion report。 |
| [batch readiness checklist](./batch-readiness-checklist.md) | 当扫描或修复变成 rule-selected target set、批量关系赋值或 broad cleanup 时停止。 | 本文只允许窄范围 new-problem reuse scan。 |

本文的 owner entry point 是 [docs/index.md](../index.md) 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/existing-doc-reuse-procedure.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Epic 5 首版 existing-doc reuse procedure；network boundary / decay prevention policy 已落地，Epic 6 后续仍会细化 batch reuse scans 和 batch governance records。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record
- source asset: docs/governance/existing-doc-reuse-procedure.md
- trigger: Story 5.4 explicitly authorizes a canonical existing-doc reuse procedure governance asset
- index file: docs/index.md
- section: ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: the asset is a formal governance policy with listed_in_docs_index navigation treatment
- validation result: target exists and relative link resolves
- unresolved risk: network boundary / decay prevention policy is now a direct adjacent authority; Epic 6 batch reuse/batch governance integrations remain future dependencies, recorded in open_questions
```

## 输入与前置分类

开始扫描前，必须先记录输入，不得先搜一圈再回填结论。最低输入包括：

- source question / new task：用户问题、候选正文、审查请求、story requirement 或修订触发。
- task type：new document、upgrade、review、index-only update、candidate promotion、duplicate/coexistence decision、planning artifact、batch governance 或其他已说明类型。
- asset level：formal knowledge asset、methodology asset、governance asset、template asset、`docs/index.md`、candidate/workflow output 或 BMad workflow/skill asset。
- candidate concept：候选概念键、reader question、对象边界或待建模问题。
- candidate topic：拟议 topic、相邻 topic、已知 topic directory 或 unknown。
- known context：用户给出的已有文档、关键词、source/time context、相邻资产、review finding 或 open question。
- intended downstream use：新建、修订、审查、复用模型、避免重复、补充链接、完成证据或持有等待。
- time/source sensitivity：是否涉及当前实践、现行制度、外部事实、版本状态、历史路径或 source discipline。
- allowed file scope：本轮允许查看和修改的文件范围。

本 procedure 的目标是找到可复用 existing docs 或确认缺口。它不是生成新文档、晋升候选、替换旧资产、批量清理关系、改变 `quality_status` 或更新全库网络。

使用 [输入摄入与任务意图判定](../methodology/intake-and-intent-classification.md) 先判断当前任务是否属于 new document、upgrade、review、index-only、candidate promotion、duplicate/coexistence decision、planning artifact 或 batch governance。如果输入不足以判断 topic、candidate concept、reuse question 或 target asset class，记录为 `hold_for_clarification` 或 open question；不得编造 topic、candidate、scan conclusion 或 reuse evidence。

## 扫描面与执行顺序

Reuse scan 必须是有界的、可复核的、人类可读的。它不是 search tooling，也不是全库 inventory。推荐顺序如下：

1. intake classification。
2. `docs/index.md` scan。
3. candidate topic directory scan。
4. candidate doc frontmatter/body scan。
5. related docs scan。
6. neighboring concepts scan。
7. duplicate/coexistence check。
8. reusable entry point mapping。
9. result classification。

### `docs/index.md` scan

使用 [docs/index.md](../index.md) 查找 display title、topic section、known canonical entry、methodology/governance/templates/reports section 和 nearby index entries。Index entry 是导航证据，不是 relationship evidence 或 reuse evidence 本身。

记录时至少说明：

- 查看的 section 或 nearby entries。
- 找到的 candidate canonical entry。
- 为什么该 entry 是候选。
- 为什么 index 位置不能单独证明 exact/adjacent/prerequisite/contrast/deeper-dive。

### Topic directory scan

只检查候选 topic、相邻 topic 和明确命名的目录。可按 slug、title、concept、frontmatter、H1/body heading 查找候选。不得扩大为 full-repo topic sweep，除非 [batch readiness checklist](./batch-readiness-checklist.md) 授权了 rule-selected target set。

如果扫描需要“所有 topic 都看看”“全库找相似标题”“把整个目录整理一遍”，停止并记录为 batch-shaped work，而不是在普通 story 中继续。

### Candidate doc scan

对已识别 candidate docs 检查 frontmatter、H1、body structure、examples、source/time context、open questions、body links、`related_docs` 和 reusable entry point evidence。不得因为打开候选文档就顺手重写正文、改状态或修复所有链接。

候选文档扫描要回答：

- 它是否直接回答 new question。
- 它是否只提供前置、对比、深入、相邻或可迁移模型。
- 它有哪些可复用对象：model、boundary、example、source_basis、time_context、open_question、failure_pattern、verification_method、decision_frame 或 relationship evidence。
- 哪些证据缺失，是否阻塞当前复用。

### Related docs scan

只在已识别 candidate docs 的 `related_docs`、正文关系说明和 Relationship Record 中查找 next candidates。必须调用 [related docs taxonomy](./related-docs-taxonomy.md) 的 target status、relation type、transfer object 和 meaningful-link evidence。

`related_docs` 中的目标默认只是候选链路。复用结论必须再检查实际 source asset、evidence location、transfer object、boundary limit 和 next verification。

### Neighboring concepts scan

检查同 topic 邻近资产、同一 reader question、shared mechanism、prerequisite/contrast/deeper-dive candidates 和 known adjacent assets。必须说明为什么候选是 neighboring，而不是重复、上位、下位、继任、source reference 或 decorative link。

相邻判断至少说明：

- source question 与候选文档各自回答什么问题。
- 共享或可迁移的 model / boundary / mechanism / failure pattern / verification method 是什么。
- 哪条边界阻止它成为 exact match 或 duplicate。
- 迁移后必须重新验证什么。

### Duplicate/coexistence scan

当出现 exact/near duplicate、overlap、same-topic coexistence、canonical-entry ambiguity 或 candidate distinction 不足时，调用 [duplicate/coexistence policy](./duplicate-and-coexistence-policy.md)。本 procedure 只记录 suspected duplicate/coexistence result 和 routing，不执行 cleanup、merge、split、reject、move、deprecate、archive 或 broad duplicate scan。

如果复用扫描的真实结果是“这可能是重复文档或共存候选”，不得用 `adjacent_concept` 掩盖。必须记录 `duplicate_or_coexistence_candidate` 并转向 duplicate/coexistence policy。

### Reusable entry point check

对候选 existing docs 查找 `core_model`、`boundary_rule`、`decision_frame`、`failure_pattern`、`verification_method`、example/self-test/open-question support 和 Reusable Model Entry Point Record。必须调用 [reusable model entry points policy](./reusable-model-entry-points.md) 的 New Problem To Existing Model Mapping 和 Reusable Model Entry Point Record。

本步骤只检查候选文档是否有可调用入口。它不创建 model-entry inventory，不批量分配 anchors，不把 entry point type 写进 frontmatter。

## Scan Result Taxonomy

Result label 是 human-readable scan result，不是 frontmatter 字段、schema、数据库状态、automation output 或 final document decision。允许结果如下：

| Result | Meaning | Required evidence |
| --- | --- | --- |
| `exact_match` | 已有正式资产直接回答新问题或 candidate concept，且有足够 reusable entry point evidence。 | Source asset、evidence location、exact fit reason、reused object、boundary limit、source/time check 和 next verification。 |
| `adjacent_concept` | 已有资产不直接回答新问题，但可迁移边界、模型、机制、失败模式或验证方式。 | relation type、transfer object、boundary distinction、reuse role、transfer risk 和为什么不是 duplicate/exact。 |
| `prerequisite` | 已有资产是理解或安全使用新问题模型的前置规则、概念、方法论或治理资产。 | 前置对象、缺失它会导致的误用、当前任务如何引用它。 |
| `contrast` | 已有资产主要用于防止混淆、说明反例、边界差异或错误迁移风险。 | contrast axis、confusion risk、selection boundary、verification question。 |
| `deeper_dive` | 已有资产提供更深机制、推导、案例、规则、治理细节或 reusable anchor，而当前问题只需要入口或摘要。 | delegated detail、为什么当前任务不重复展开、可回跳位置和 boundary limit。 |
| `reusable_model_candidate` | 存在可调用模型入口，但 scan 结果不足以证明 exact/adjacent/prerequisite/contrast/deeper-dive。 | entry point type、mapped object、missing evidence、review/targeted revision/next verification。 |
| `duplicate_or_coexistence_candidate` | 发现 exact/near duplicate、overlap、same-topic coexistence 或 canonical ambiguity。 | suspected type、affected assets、routing to duplicate/coexistence policy、not-cleaned-up statement。 |
| `revision_opportunity` | 已有资产有价值但缺模型、边界、例子、来源、open question、link evidence 或 reusable entry point。 | affected doc、missing evidence、blocked reuse scenario、proposed fix、owner/future story。 |
| `no_useful_existing_asset` | 授权范围内没有可用 existing doc、candidate target 不存在，或 existing docs 不能安全支持当前新问题。 | scan scope checked、rejected candidates、为什么不能复用、是否需要 new document candidate / hold / defer。 |

结果可组合，但必须有 primary result。例如一个扫描可 primary 为 `adjacent_concept`，并附带 `revision_opportunity`；也可以 primary 为 `duplicate_or_coexistence_candidate`，并说明 exact match 暂不能确认。不得用多个标签堆叠来掩盖缺失证据。

如果候选 source asset 存在 blocking `quality_status`、deprecated / archived lifecycle state、unresolved successor / replacement ambiguity、failed quality gate、source/time blocker 或 other governance blocker，不得记录为 primary `exact_match`。这类结果必须降级为 `revision_opportunity`、`duplicate_or_coexistence_candidate`、`hold_for_clarification`、`deferred_with_reason` 或 `no_useful_existing_asset`，并记录 decision impact、blocked reuse scenario 和 next verification。

## Reused Context Evidence

每个复用结论必须指出 reused object，至少包含以下一种：

- `model`
- `boundary`
- `example`
- `source_basis`
- `time_context`
- `open_question`
- `failure_pattern`
- `verification_method`
- `decision_frame`
- `link/relationship evidence`

每条复用证据至少记录：

- source existing doc。
- specific location：section、paragraph/table、frontmatter field、record、example、open question 或 link sentence。
- entry point type：若适用，使用 reusable model entry points policy 的 `core_model`、`boundary_rule`、`decision_frame`、`failure_pattern` 或 `verification_method`。
- relation type：若涉及跨文档关系，使用 related docs taxonomy。
- transfer object：structure、boundary、model、judgment、failure_mode、verification_method、source_time_context、lifecycle_index_impact 或紧密说明的其他对象。
- fit reason：为什么这份 existing doc 能支持当前问题。
- boundary limit：复用到哪里必须停止。
- transfer risk：错误迁移、过期事实、同名异义、source/time mismatch、weak link、duplicate ambiguity 或 quality gap。
- next verification：下一步如何确认复用安全。

Source/time discipline 是强制检查。复用旧文档的 `source_basis`、`time_context`、外部事实、当前实践或历史判断时，必须检查该来源是否仍能支持当前问题；不得把旧文档的 source/time context 自动当作当前事实复用。涉及当前实践或外部事实时，调用 [来源纪律与真实世界锚点政策](../methodology/source-discipline-and-real-world-anchor-policy.md)。

复用 body link 或 `related_docs` 时，必须调用 [link maintenance policy](./link-maintenance-policy.md) 的 existence/path/topic/meaning/direction checks。该检查只覆盖 changed-file 或授权窄范围；不得执行 broad inbound/outbound repair。

复用 reusable entry point 时，必须调用 [reusable model entry points policy](./reusable-model-entry-points.md) 的 New Problem To Existing Model Mapping 和 Reusable Model Entry Point Record。

以下信号只能产生候选，不能单独证明复用：

- 标题相似。
- 关键词相同。
- 同 topic。
- 路径相邻。
- index 邻近。
- `related_docs` 中出现目标但无 relation/transfer evidence。
- 正文出现“参见”但未说明 transfer object。

## Insufficient Existing Docs And Gap Handling

Insufficient existing doc 不等于自动创建新正式资产。它表示当前 existing docs 在某个复用场景下缺少证据，需要按缺口性质进入 targeted revision、new document candidate、candidate promotion、duplicate/coexistence decision、hold for Maxwell、route to network boundary / decay prevention policy、defer to Epic 6 或 not authorized。

可使用的人类可读 gap status 包括：

- `revision_opportunity`
- `open_question`
- `quality_gap`
- `model_gap`
- `source_gap`
- `link_gap`
- `boundary_gap`
- `duplicate_coexistence_gap`
- `no_valid_entry`
- `deferred_with_reason`
- `hold_for_clarification`
- `not_authorized`

每个 gap 至少记录：

| Field | Required evidence |
| --- | --- |
| affected doc | 哪份 existing doc、candidate doc 或 planned target 受影响。 |
| missing evidence | 缺 model、boundary、example、source/time、open question、link evidence、entry point 或 duplicate/coexistence decision。 |
| blocked reuse scenario | 哪类 new question 或 downstream use 被阻塞。 |
| why it blocks or does not block | 说明缺口是否阻塞当前任务，不能只写“待补”。 |
| proposed fix | targeted revision、review check、source/time check、link check、mapping check、duplicate/coexistence decision 或 new document candidate。 |
| owner/future story | Maxwell、当前 Agent、reviewer、network boundary / decay prevention policy、Epic 6 或其他 owner。 |
| decision impact | 是否影响 new document、upgrade、review、promotion、index、quality status 或 completion wording。 |
| validation status | pass、fail、not_applicable、unverified、deferred_with_reason 或 not_authorized。 |

未解决 gap 不得在 completion wording 中描述为 resolved、reviewed、validated、reused、ready 或 safe。若要新建或晋升文档，必须回到 intake、candidate promotion、quality gate、index impact 和 source/time discipline。

## Existing Doc Reuse Scan Record

`Existing Doc Reuse Scan Record` 是 copyable human-readable record。它不是 schema、frontmatter extension、lint rule、generator input、database row 或 automation contract。它可以落在 target body、review evidence、completion evidence、story Dev Agent Record、future reuse procedure note 或明确命名的 scan note 中。

```markdown
## Existing Doc Reuse Scan Record

- source question:
- task type:
- candidate concept:
- candidate topic:
- allowed scan scope:
- scan surfaces checked:
- candidate existing docs:
- scan result: exact_match | adjacent_concept | prerequisite | contrast | deeper_dive | reusable_model_candidate | duplicate_or_coexistence_candidate | revision_opportunity | no_useful_existing_asset | unverified
- unverified scope/blocker: required when scan result is `unverified`; list unchecked surfaces, blocker reason, whether reuse/completion is blocked, and next owner.
- source asset:
- evidence location:
- reused object: model | boundary | example | source_basis | time_context | open_question | failure_pattern | verification_method | decision_frame | link_relationship_evidence | other_with_reason
- entry point type:
- relation type:
- transfer object:
- fit reason:
- boundary limit:
- transfer risk:
- source/time check:
- link/related_docs impact:
- duplicate/coexistence impact:
- revision opportunity or open question:
- decision impact:
- owner/future story:
- validation result: pass | fail | not_applicable | unverified
```

Record 字段不得拆成未经授权的 frontmatter fields。若未来确实需要 schema 化 scan result、candidate docs、reuse status、network health 或 batch reuse evidence，必须由 frontmatter schema、version governance、network boundary / decay prevention policy、Epic 6 或 Maxwell 明确授权。

## 与相邻治理资产和 Future Stories 的边界

本 procedure 的核心边界是：找候选、分类结果、记录复用证据；不替代相邻治理资产的决策权。

| Boundary | Rule |
| --- | --- |
| Relationship taxonomy | 调用 [related docs taxonomy](./related-docs-taxonomy.md) 的 target status、relation type、transfer object、meaningful-link evidence 和 Relationship Record；本文不改写 taxonomy。 |
| Link maintenance | 调用 [link maintenance policy](./link-maintenance-policy.md) 的 changed-file link checks、one-way reason、fragment/anchor validation 和 bounded inbound/outbound review；本文不执行 broad link maintenance。 |
| Reusable model anchors | 调用 [reusable model entry points policy](./reusable-model-entry-points.md) 的 entry point types、callable anchors、New Problem To Existing Model Mapping 和 Reusable Model Entry Point Record。 |
| Duplicate/coexistence | 调用 [duplicate/coexistence policy](./duplicate-and-coexistence-policy.md) 的 duplicate/coexistence taxonomy、allowed outcomes 和 Decision Record；本文不执行 cleanup、merge/split/reject/move 或 broad duplicate scan。 |
| Frontmatter schema | 调用 [frontmatter schema](./frontmatter-schema.md) 的 baseline fields、YAML arrays、stable `doc_id` 和 unauthorized field 禁令；scan result 不进入 frontmatter。 |
| Index synchronization | 调用 [index synchronization rules](./index-synchronization-rules.md) 的 Index Impact outcomes、relative path、formal navigation entry 和 index-wide cleanup 边界。 |
| Candidate promotion | 调用 [candidate promotion checklist](./candidate-promotion-checklist.md) 的 promotion gate；reuse scan 结果不能自动晋升候选或发布新文档。 |
| Review/completion/decision evidence | 调用 [review record template](../templates/review-record-template.md)、[completion report template](../templates/completion-report-template.md) 和 [document decision policy](./document-decision-policy.md)；本文不创建 actual review/completion records。 |
| Network boundary | Dense/noisy/weak/stale relationship signals、topic health、network decay、repeated overlap 和 navigation relevance evidence 的识别与路由调用 [network boundary and decay prevention policy](./network-boundary-and-decay-prevention.md)；actual cleanup 或 batch-shaped execution 必须转向 link maintenance、batch readiness 或 Epic 6。 |
| Batch governance | Batch reuse scans、batch relationship assignment、batch index/link normalization、batch review/completion records 和 batch governance 留给 Epic 6。 |

Stop / escalation conditions：

- 需要新增全局 frontmatter fields 才能表达 scan result、reuse status、candidate docs、relation status、model status 或 network health。
- 需要 full-repo scans、all-topic sweeps、all-doc inventories、batch duplicate scans、batch relationship assignment、index-wide cleanup 或 broad link normalization。
- proposed scan target 缺失，且没有 story、planning artifact 或 owner decision 说明其 planned status。
- suspected duplicate/coexistence result 需要 actual merge、split、reject、move、deprecation、archive、successor/replacement 或 old-content handling。
- reuse result 需要创建或晋升新正式文档，但尚未运行 candidate promotion、quality gate、index impact 和 source/time discipline。
- 工作会需要 code、automation、validators、scanners、schemas、generators、software tests 或 search tooling。
- 无法区分本 procedure 与 reusable model entry point semantics、network boundary / decay prevention policy 或 Epic 6 batch governance。

## Narrow Completion And Validation Checklist

应用本文完成一次普通 new-problem reuse scan 时，completion evidence 必须区分已做和未做。不得说“全库已查”“所有相关文档已验证”“所有链接已修复”，除非该 full scope 被授权并实际完成。

验证本文或本文的一次应用时，至少检查：

1. Target file 位于 `docs/governance/existing-doc-reuse-procedure.md`。
2. Frontmatter 包含 baseline fields；`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
3. H1、frontmatter `title` 和 `docs/index.md` display title 一致。
4. 扫描面覆盖 `docs/index.md`、topic directories、candidate docs、related docs、neighboring concepts、duplicate/coexistence candidates 和 reusable entry point checks。
5. 扫描顺序明确，并从 intake/task classification 开始。
6. Result taxonomy 覆盖 `exact_match`、`adjacent_concept`、`prerequisite`、`contrast`、`deeper_dive`、`reusable_model_candidate`、`duplicate_or_coexistence_candidate`、`revision_opportunity` 和 `no_useful_existing_asset`。
7. Reused context evidence 要求指出 model、boundary、example、source/time、open question、failure pattern、verification method、decision frame 或 relationship evidence。
8. Insufficient existing docs 被记录为 revision opportunities、open questions、quality/model/source/link/boundary gaps、no valid entry 或 deferred/not-authorized outcomes。
9. Existing Doc Reuse Scan Record 存在，并明确 non-schema、non-frontmatter、non-automation。
10. Related docs taxonomy、link maintenance policy、reusable model entry points policy、duplicate/coexistence policy、candidate promotion、network boundary / decay prevention policy 和 Epic 6 boundaries 明确。
11. Body links、frontmatter `related_docs` 和 `docs/index.md` entry 可解析。
12. Planned/missing future targets 已进入 `open_questions` 或 future dependency notes。
13. 没有新增 unauthorized frontmatter fields。
14. 没有实际执行 full-repo reuse scan、topic-wide cleanup、batch duplicate scan、broad link/index repair、network health review、existing-doc inventory、candidate promotion、document regeneration、batch relationship assignment、stale-link cleanup、batch `related_docs` update 或 index-wide cleanup。
15. 没有新增 runtime code、software tests、automation、search tool、schema、validator、generator、model scanner、duplicate scanner、link scanner、index generator、CLI/API/UI/database/deployment/CI、package manifest、runbook、actual review record 或 actual completion report。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
- [related docs 与相邻概念关系分类](./related-docs-taxonomy.md)
- [跨文档链接维护政策](./link-maintenance-policy.md)
- [可复用模型入口政策](./reusable-model-entry-points.md)
- [重复概念与同主题共存治理](./duplicate-and-coexistence-policy.md)
- [候选文档晋升 Checklist](./candidate-promotion-checklist.md)
- [Frontmatter schema 与 doc_id 身份规则](./frontmatter-schema.md)
- [docs/index.md 同步与导航治理规则](./index-synchronization-rules.md)
- [批量治理 Readiness Checklist](./batch-readiness-checklist.md)
- [审查记录模板](../templates/review-record-template.md)
- [完成汇报模板](../templates/completion-report-template.md)
- [文档决策政策](./document-decision-policy.md)
