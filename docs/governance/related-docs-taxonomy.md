---
doc_id: governance-related-docs-taxonomy
title: related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling
concept: related_docs_taxonomy
topic: governance
depth_mode: standard
created_at: '2026-05-29T10:14:38+08:00'
updated_at: '2026-06-15T15:46:07+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/5-1-related-docs-taxonomy.md
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/intake-and-intent-classification.md
time_context: phase_5_epic_5_related_docs_taxonomy_2026_05_29
applicability: formal_docs_related_docs_taxonomy_and_meaningful_link_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/intake-and-intent-classification.md
open_questions:
  - reusable model entry points policy 已建立后，是否需要在后续 review/completion templates 中进一步对齐 Relationship Record、Link Maintenance Record 和 Reusable Model Entry Point Record？
  - network boundary and decay prevention policy 已建立后，是否需要继续把 dense/noisy links、weak links 和 stale links 映射到更专门的 Epic 6 batch network review records？
  - Epic 6 建立 batch governance runbook and records 后，是否需要为 batch related_docs update / link normalization 提供专门 evidence record？
---

# related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 formal `docs/` 资产使用 `related_docs`、正文内部链接、相邻概念链接和可解释跨文档关系时的 canonical relationship taxonomy。它回答的是：一个链接为什么有意义，目标资产是什么状态，关系属于哪一类，哪些结构、边界、模型或判断可以迁移，以及 missing 或 planned target 应如何暴露。

本文适用于：

- 正式 `docs/` 资产的 frontmatter `related_docs` 列表。
- 正文内部链接中的 meaningful relationship evidence。
- 治理资产、方法论资产、模板资产和正式概念文档之间的相邻概念、对比、前置、深入阅读、共享机制、示例/应用和继任/替代关系。
- review evidence、completion evidence、story Dev Agent Record、Link Maintenance Record 或 future reuse procedure record 中的人类可读关系说明。

本文不适用于：

- 批量建立或 normalize 全库 `related_docs`。
- 执行 inbound/outbound link scan、link repair、stale link cleanup、network health review 或 index-wide cleanup。
- 替代 reusable model entry points、existing-doc reuse procedure、link maintenance policy 或 [network boundary and decay prevention policy](./network-boundary-and-decay-prevention.md)。
- 新增 frontmatter 字段、machine-readable schema、validator、generator、link scanner、index generator、CLI/API/UI/database/deployment/CI 或 software tests。

本文补充 `docs/methodology/document-generation-methodology.md`、`docs/governance/governance-asset-navigation-policy.md`、`docs/governance/frontmatter-schema.md`、`docs/governance/index-synchronization-rules.md`、`docs/governance/duplicate-and-coexistence-policy.md`、`docs/governance/rename-migration-policy.md`、`docs/governance/revision-regeneration-continuity-policy.md`、`docs/governance/sidecar-boundary-policy.md` 和 `docs/governance/legacy-migration-guide.md`。主方法论仍负责正式文档的新建、升级、审查和仓库集成；frontmatter schema 仍负责 baseline fields；index synchronization rules 仍负责正式导航入口；本文只负责 related-doc relationship type、target status 和 meaningful-link evidence。

本文的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/related-docs-taxonomy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Epic 5 首版 relationship taxonomy；link maintenance policy、reusable model entry points policy、existing-doc reuse procedure 和 network boundary / decay prevention policy 已落地，Epic 6 后续仍会细化 batch link governance。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record
- source asset: docs/governance/related-docs-taxonomy.md
- trigger: Story 5.1 explicitly authorizes a canonical related docs taxonomy governance asset
- index file: docs/index.md
- section: ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: the asset is a formal governance policy with listed_in_docs_index navigation treatment
- validation result: target exists and relative link resolves
- unresolved risk: link maintenance policy, reusable model entry points policy, existing-doc reuse procedure and network boundary / decay prevention policy are now direct adjacent authorities; Epic 6 integrations remain future dependencies, recorded in open_questions
```

## Relationship Target Status

每个 `related_docs` 或正文关系必须先判断 target status。Target status 决定它能否进入 `related_docs`，以及是否必须写入 `open_questions`、review notes 或 follow-up evidence。

| Target status | Meaning | Allowed treatment | Hard stop / fail signal |
| --- | --- | --- | --- |
| `existing` | 目标是已经存在的正式 `docs/` 资产、正式模板资产、正式索引资产或受控正式报告入口。 | 可以进入 `related_docs`；正文可链接；记录 relation type 和 transfer object。 | 路径不可解析、目标不是正式资产、标题/路径/状态暗示错误。 |
| `intentionally_planned` | 目标尚不存在，但 planning artifact、story、owner decision 或明确 future story 已授权。 | 不作为普通 existing target 描述；必须在正文或 `open_questions` 说明 planned status、owner/future story 和当前保留关系的理由。 | 把 planned target 写得像当前可访问资产，或让 index/completion wording 暗示已经存在。 |
| `unresolved` | 目标缺失、归属未定、owner 未定、路径不清或关系无法验证。 | 不得静默进入普通 `related_docs`；记录到 `open_questions`、review notes、completion evidence、follow-up list 或 Dev Agent Record。 | 缺失目标没有 open question；用 decorative link 掩盖 unresolved target。 |

`related_docs` 默认优先指向 `existing` 的正式 `docs/` 资产。`intentionally_planned` 目标只有在已由 planning artifacts、story 或 owner decision 明确授权时才允许保留为 future dependency；它不等于当前存在资产。`unresolved` 目标不得被写成普通可点击关系，也不得让索引、正文或完成汇报暗示它已经可访问。

区分以下对象：

- `related_docs`：frontmatter 中表达正式跨文档关系的 YAML array。
- source references：正文或 frontmatter `source_basis` 中记录的依据来源，不自动构成 related relationship。
- supporting sidecar：由 `docs/governance/sidecar-boundary-policy.md` 管理的补充材料、review notes、examples 或 supporting records，不替代 main document canonical truth。
- successor/replacement：由 `docs/governance/rename-migration-policy.md` 和 `docs/governance/revision-regeneration-continuity-policy.md` 管理的继任、替代、remaining access 或 lifecycle/index 关系。
- index entry：由 `docs/governance/index-synchronization-rules.md` 管理的导航入口，不等同于 related relationship。
- owner entry：由 `docs/governance/governance-asset-navigation-policy.md` 管理的发现入口，不等同于 `related_docs`。

关系原因写在正文、review evidence、completion evidence、Relationship Record 或明确命名的 relationship note 中。不得为了表达 relation type、target status、link role、owner entry 或 successor relation 而新增未经授权的 frontmatter fields。

## Relationship Type Taxonomy

每个 meaningful relation 必须使用以下关系类型之一，或在严格条件下使用 `approved_equivalent`。

| Relationship type | Required meaning | Required evidence |
| --- | --- | --- |
| `prerequisite` | Target 是读者需要先理解的前置概念、规则、模型或治理资产。 | 说明哪个概念、规则、模型或治理资产是前置条件，以及缺少它会导致什么误用。 |
| `adjacent_concept` | Source 与 target 概念不同但边界相邻，可迁移部分结构、边界或判断。 | 说明各自回答的问题、对象边界、适用/不适用条件、boundary distinction 和 reuse role。 |
| `contrast_concept` | Target 用于防止混淆，重点是差异、反例、选择边界或误用风险。 | 说明 contrast axis、混淆后果、selection boundary 和 reviewer 应检查的问题。 |
| `deeper_dive` | Source 只建立入口或摘要，target 提供更深机制、推导、案例、规则或治理细节。 | 说明哪个 detail 被有意委托给 target，以及 source 为什么不在本处展开。 |
| `shared_mechanism` | Source 和 target 共享机制、结构、失败模式、验证方式、source/time discipline 或治理判断。 | 说明 shared object 是什么，哪些部分可迁移，迁移后必须重新验证什么。 |
| `example_application` | Target 是 source 模型的示例、应用场景、实践落点、反例或测试用例。 | 说明 target 应用或检验了 source 的哪个模型、边界、failure mode 或 judgment。 |
| `successor_replacement` | Target 是继任、替代、迁移后入口、deprecated/archive 的剩余访问入口或 replacement path。 | 说明 successor/replacement reason、old/new boundary、remaining access、index/lifecycle implication，并在需要时调用 rename/migration 或 continuity policy。 |
| `approved_equivalent` | 现有类型无法准确表达，且当前关系确实需要被记录。 | 正文记录为什么现有类型失败、临时类型边界、owner/reviewer、future taxonomy review trigger 和撤销/合并条件。 |

`approved_equivalent` 不是扩展 taxonomy 的快捷方式。只有当七个标准类型无法准确表达关系，并且不新增 frontmatter schema、不提前实现后续 story、可被 reviewer 复核时，才允许使用。

Supporting evidence、owner entry、sidecar support、source reference、index entry、review record 或 completion record 可以作为边界说明出现，但不能被本文替代 link maintenance policy、reusable model entry points policy、existing-doc reuse procedure 或 network boundary / decay prevention policy，也不能提前实现 Epic 6 的资产、流程、schema 或自动化。

## Meaningful-Link Evidence And Transfer Object

一个关系只有在记录了 transfer object 时才算 meaningful。Transfer object 是读者或 reviewer 可以从 source 迁移到 target、从 target 回到 source、或用来判断两者边界的对象。

允许的 transfer object 包括：

| Transfer object | Meaning | Evidence question |
| --- | --- | --- |
| `structure` | 分解、顺序、层级、状态模型、record shape、分类框架或文档结构。 | 哪个结构可以复用？复用后在哪些对象上要重新命名或重划边界？ |
| `boundary` | 纳入/排除规则、相邻概念区分、duplicate/coexistence boundary 或 scope limit。 | 这个链接帮助区分什么？如果不链接会混淆什么？ |
| `model` | 可复用 mental model、因果链、机制、决策模型或治理模型。 | 哪个模型可以迁移？迁移到 target 时哪些前提会变化？ |
| `judgment` | 决策规则、reviewer question、acceptance check、escalation condition 或 owner decision。 | 读者要借用什么判断？判断失败时应转向哪个治理资产？ |
| `failure_mode` | 误分类、stale-link risk、duplicate risk、weak-link pattern、misuse 或 silent promotion。 | 这个链接帮助避免哪类错误？错误出现时如何识别？ |
| `verification_method` | 检查项、证据位置、review record、completion evidence、self-test 或 changed-file link check。 | 如何验证关系真实存在，且不是装饰性引用？ |
| `source_time_context` | 来源、时间、currentness、历史/当前实践、核对日期或 source discipline。 | 哪些 source/time discipline 可以复用，哪些必须重新核查？ |
| `lifecycle_index_impact` | successor、replacement、deprecation、archive、index entry、owner entry 或 navigation implication。 | 关系是否改变入口、可见性、剩余访问或 lifecycle/index 说明？ |

链接文字、关系说明、上下文句或 Relationship Record 必须让 reviewer 判断“为什么链接到这里”。如果 reviewer 只能看到名词相似、路径相邻或“参见”，但看不到 transfer object，该关系应被退回、删去或记录为 unresolved/follow-up。

### Positive examples

| Relation | Good evidence pattern |
| --- | --- |
| 前置规则 | “本文使用 baseline frontmatter 字段；字段完整性和 YAML array 要求以 [frontmatter schema](./frontmatter-schema.md) 为 prerequisite。” Transfer object 是 `structure` 和 `judgment`。 |
| 相邻概念边界 | “相邻链接表示两篇资产保留独立 concept；若疑似重复或合并，必须转向 [duplicate/coexistence policy](./duplicate-and-coexistence-policy.md)。” Transfer object 是 `boundary` 和 `judgment`。 |
| 对比概念 | “owner entry 是发现入口，related_docs 是关系证据；混淆二者会制造 parallel canonical entry。” Transfer object 是 `boundary` 和 `failure_mode`。 |
| 防误用 deeper dive | “本文只定义关系类型；入链/出链维护由 [link maintenance policy](./link-maintenance-policy.md) 处理。” Transfer object 是 `judgment` 和 `verification_method`。 |
| 共享机制 | “batch related_docs update 与 batch index update 都是 rule-selected target set 风险。” Transfer object 是 `structure` 和 `failure_mode`。 |
| 例子/应用 | “Review/completion templates 可以承载 relationship evidence，但不把 record 字段变成 frontmatter。” Transfer object 是 `judgment` 和 `verification_method`。 |
| 继任/替代 | “旧入口如果被 deprecated，只能通过 successor/replacement evidence 保留 remaining access。” Transfer object 是 `lifecycle_index_impact`。 |

### Weak-link examples to reject or defer

- 只因为两个文档都提到 “link” 就互相加入 `related_docs`。
- `related_docs` 指向 missing file，却没有 planned status、owner、future story 或 open question。
- 正文只写“另见某文档”，但不说明读者要复用什么结构、边界、模型或判断。
- 用 `adjacent_concept` 掩盖疑似 duplicate/coexistence decision。
- 用 `successor_replacement` 暗示迁移完成，但没有 migration/index/lifecycle evidence。
- 因为多个文件看起来模式类似，就批量添加关系，而未先执行 batch readiness。

正文链接和 `related_docs` 影响必须保持窄范围、可解释、可验证；入链/出链复核、one-way reason 和 rename/migration lifecycle link impact 使用 [link maintenance policy](./link-maintenance-policy.md)。本文不授权 full-repo link scan、broad inbound/outbound repair 或 batch relationship assignment。

## Adjacent Concept Boundary And Reuse Role Rules

`adjacent_concept` 是最容易被误用的关系类型。它必须证明“相邻但不同”，不能只是“有点像”。

使用 `adjacent_concept` 时，至少说明：

- Source 回答什么问题，target 回答什么问题。
- Source 的对象边界是什么，target 的对象边界是什么。
- Source 适用条件和不适用条件。
- Target 适用条件和不适用条件。
- 二者为什么不是重复文档、上下位文档、前置文档、对比文档、示例文档或继任/替代文档。
- 读者从 source 到 target 或从 target 到 source 时，实际复用什么。

允许的 reuse role 包括：

| Reuse role | Meaning |
| --- | --- |
| 借用定义 | 借用一个术语的稳定工作定义，但不借用整个文档职责。 |
| 借用边界 | 借用 inclusion/exclusion、topic boundary、duplicate/coexistence boundary 或 scope limit。 |
| 借用机制 | 借用机制、因果链、状态转换或治理流程形状。 |
| 借用判断框架 | 借用 reviewer question、decision rule、acceptance check 或 stop condition。 |
| 借用失败模式 | 借用误用、混淆、stale link、weak link、duplicate 或 silent promotion 风险。 |
| 借用验证方式 | 借用 changed-file link check、frontmatter consistency check、completion evidence 或 review evidence。 |
| 借用例子 | 借用一个示例或反例，用来检验 source 的模型或边界。 |

相邻关系与其他关系的边界：

| Candidate relation | Use when | Do not collapse into `adjacent_concept` when |
| --- | --- | --- |
| duplicate | 两篇资产无法证明独立 concept、purpose、reuse context 或 reader question。 | 疑似重复、merge、split、narrow 或 explicit coexistence 尚未决策；转向 duplicate/coexistence policy。 |
| broader/narrower | Target 是上位框架或下位细分入口。 | 下位资产只是在举例；可能更适合 `example_application` 或 `deeper_dive`。 |
| prerequisite | Target 是安全理解或使用 source 的前置规则。 | Source 只是顺手提到 target；需要说明缺失前置会造成什么误用。 |
| contrast | Target 主要用于防止混淆或说明“不是什么”。 | 关系重点是差异、反例或选择边界，而不是可迁移 reuse。 |
| example/application | Target 是 source 的案例、实践落点或反例。 | Target 没有独立相邻概念职责，只是应用 source 模型。 |
| successor/replacement | Target 是旧入口的继任、替代或 remaining access。 | 关系改变 identity、path、index、lifecycle 或 deprecation/archive 语义；转向 rename/migration 和 continuity policy。 |

相邻概念链接不得替代正式 duplicate/coexistence decision、rename/migration decision、revision/regeneration continuity 或 legacy migration assessment。疑似重复、合并、拆分、废弃、继任、旧内容处理或旧路径剩余访问问题，必须路由到对应治理资产，而不是用 `adjacent_concept` 临时盖住。

## Related Docs Relationship Record

`Related Docs Relationship Record` 是 copyable human-readable record。它不是 schema、不是 lint rule、不是 generator 输入，也不是 frontmatter extension。它可以落在 target body、review evidence、completion evidence、story Dev Agent Record、Link Maintenance Record、Existing Doc Reuse Scan Record 或明确命名的 relationship note 中。

```markdown
## Related Docs Relationship Record

- source asset:
- target asset:
- target status: existing | intentionally_planned | unresolved
- relation type:
- direction: source_to_target | target_to_source | bidirectional | one_way_with_reason
- link location: frontmatter related_docs | body link | index entry | review evidence | completion evidence | other
- transfer object: structure | boundary | model | judgment | failure_mode | verification_method | source_time_context | lifecycle_index_impact | other
- boundary distinction:
- reuse role:
- related_docs impact:
- index impact:
- source/time impact:
- lifecycle or successor impact:
- sidecar/support impact:
- duplicate/coexistence impact:
- unresolved risk:
- owner/future story:
- validation result: pass | fail | not_applicable | unverified
```

Record 字段不能被拆进未经授权的 frontmatter fields。若未来确实需要 schema 化 relation type、target status、link role、owner、successor、inbound/outbound 或 network-health 字段，必须由 frontmatter schema、version governance、后续 Epic 5/Epic 6 story 或 Maxwell 明确授权。

## Adjacent Governance Asset Boundaries

本文调用相邻治理资产，但不复制或替代它们的完整职责：

| Asset | What this taxonomy reuses | Boundary kept here |
| --- | --- | --- |
| [document generation methodology](../methodology/document-generation-methodology.md) | formal docs task classification、repository integration、quality/equivalent governance check、completion evidence baseline | 本文不创建平行主方法论。 |
| [governance asset navigation policy](./governance-asset-navigation-policy.md) | owner entry point、navigation treatment、index treatment、related-doc / owner-entry boundary | 本文不新增 owner/navigation frontmatter fields。 |
| [frontmatter schema](./frontmatter-schema.md) | baseline fields、stable `doc_id`、YAML arrays、unauthorized field 禁令 | 本文不扩展 schema；relation details 写在人类可读 evidence 中。 |
| [index synchronization rules](./index-synchronization-rules.md) | Index Impact outcomes、导航入口边界、planned/unresolved target 风险 | 本文只记录 relationship/index impact，不执行 index-wide cleanup。 |
| [batch readiness checklist](./batch-readiness-checklist.md) | batch link/index update、batch `related_docs` update、rule-selected target set、stop criteria、recovery evidence | 多文件 relationship assignment 是 batch-shaped，需另行 readiness。 |
| [duplicate/coexistence policy](./duplicate-and-coexistence-policy.md) | duplicate、coexistence、`adjacent_link` decision boundary | `adjacent_concept` 不替代 duplicate/coexistence decision。 |
| [rename/migration policy](./rename-migration-policy.md) | successor/replacement、remaining access、link-index impact、Migration Decision Record | 本文不执行 rename、move、merge、split、deprecation 或 archive。 |
| [revision/regeneration continuity policy](./revision-regeneration-continuity-policy.md) | reference validity、old-content handling、Continuity Record | 本文不重新定义 revision/regeneration modes。 |
| [sidecar boundary policy](./sidecar-boundary-policy.md) | sidecar support、main-document canonical truth、supporting asset boundary | supporting link 不等于 `related_docs`，sidecar 不替代 main asset。 |
| [legacy migration guide](./legacy-migration-guide.md) | old-doc compatibility、remaining gap、batch-shaped link/index risk | 本文不执行 legacy migration 或 old-doc inventory。 |
| [reusable model entry points policy](./reusable-model-entry-points.md) | `core_model`、`boundary_rule`、`decision_frame`、`failure_pattern`、`verification_method` 和 Reusable Model Entry Point Record。 | 本文仍只定义 relation type、target status 和 transfer object；callable anchor 由 reusable model entry points policy 负责。 |
| [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) | new-problem scan surfaces、scan order、scan result taxonomy 和 Existing Doc Reuse Scan Record。 | 本文仍只定义 relationship taxonomy；scan result 不能替代 relation type 或 transfer object。 |
| [review record template](../templates/review-record-template.md) | review evidence、unverified items、future-story handling | 本文不创建 actual review record。 |
| [document decision policy](./document-decision-policy.md) | hold/defer/not_authorized、status impact、decision evidence | 本文不改变 final decision vocabulary。 |
| [completion report template](../templates/completion-report-template.md) | completion evidence、unresolved risk、non-software boundary reporting | 本文不创建 actual completion report。 |
| [concept document quality gate](../methodology/concept-document-quality-gate.md) | Hard Fail、related_docs/link/status consistency、migration/reuse checks | 本文不替代质量门禁评分。 |
| [source discipline policy](../methodology/source-discipline-and-real-world-anchor-policy.md) | source/time/currentness discipline | `source_time_context` transfer object 必须重新核查当前性。 |
| [intake and intent classification](../methodology/intake-and-intent-classification.md) | task type、asset level、allowed file scope、future story routing | 本文不扩大当前 story 授权范围。 |

Future story boundaries：

- [link maintenance policy](./link-maintenance-policy.md) owns inbound/outbound link maintenance、link repair、one-way link reason、rename/move/merge/split/deprecation link review。
- [reusable model entry points policy](./reusable-model-entry-points.md) owns reusable model entry point 分类和 callable model anchors。
- [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) owns existing-doc scan procedure、exact/adjacent/prereq/contrast/no-asset results。
- [network boundary and decay prevention policy](./network-boundary-and-decay-prevention.md) owns identification and routing of dense/noisy/weak/stale relationship signals、topic health、repeated overlap 和 navigation relevance evidence；actual cleanup routes to link maintenance、batch readiness 或 Epic 6。
- Epic 6 owns batch link normalization、batch related_docs updates、batch review/completion records。

## Narrow Update And Completion Rules

使用本文时只能做窄范围关系维护；需要入链/出链、one-way reason 或 lifecycle link impact evidence 时，调用 [link maintenance policy](./link-maintenance-policy.md)：

1. 只处理当前 story、当前目标文件或明确授权的相邻文件。
2. 只为已存在、可解析、正式 `docs/` 资产添加 `related_docs` 或正文关系，除非 planned target 已由 story/planning/owner 明确授权。
3. 不批量 normalize `related_docs` lists、open questions、time_context、source_basis、index ordering 或 unrelated references。
4. 不执行全库 link scan、inbound/outbound repair、network health review 或 stale link cleanup。
5. 不把 relationship evidence 写成新 frontmatter fields。
6. 不让 planned target、future story 或 unresolved target 在正文、索引或 completion wording 中看起来已经存在。

Stop / escalation conditions：

- 需要新增全局 frontmatter fields 才能表达 relation type、link role、owner entry、successor、inbound/outbound、network health 或 schema version。
- 关系目标缺失，且没有 planning artifact、story 或 owner decision 授权为 intentionally planned。
- 相邻关系实际需要 merge、split、deprecation、archive、rename/migration、legacy migration 或 duplicate/coexistence decision。
- 需要 broad existing-doc scan、batch relationship assignment、index-wide link cleanup、link scanner、validator、generator、schema 或 automation。
- 关系会改变 canonical entry point、path ownership、frontmatter schema、identity continuity、index semantics、lifecycle/status 或 non-software boundary。

## Validation Checklist

使用或维护本文时，至少检查：

1. Target file 位于 `docs/governance/related-docs-taxonomy.md`。
2. Frontmatter 包含 baseline fields；`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
3. 未新增 `relation_type`、`related_doc_type`、`link_role`、`owner_entry_point`、`successor`、`predecessor`、`inbound_links`、`outbound_links`、`link_status`、`reuse_entry_point`、`network_health`、`schema_version`、`taxonomy_version` 或类似 unauthorized fields。
4. H1、frontmatter `title` 和 `docs/index.md` 入口标题一致。
5. Asset role、authority、scope、owner entry point、navigation treatment 和 relationship to main methodology 已声明。
6. Relationship type taxonomy 覆盖 `prerequisite`、`adjacent_concept`、`contrast_concept`、`deeper_dive`、`shared_mechanism`、`example_application`、`successor_replacement` 和 `approved_equivalent`。
7. Target handling 区分 `existing`、`intentionally_planned` 和 `unresolved`。
8. Adjacent concept rules 要求 boundary distinction 和 reuse role。
9. Meaningful-link rules 要求 transfer object evidence。
10. Positive examples 和 weak-link/decorative-link 反例存在。
11. Related Docs Relationship Record 存在，并明确不是 schema、lint rule 或 frontmatter extension。
12. Link maintenance policy、reusable model entry points policy、existing-doc reuse procedure、network boundary and decay prevention policy 和 Epic 6 boundaries 明确。
13. Body links、frontmatter `related_docs` 和 `docs/index.md` entry 可解析，planned/missing targets 已记录为 `open_questions` 或 future dependency。
14. 没有实际执行 full-repo link scan、inbound/outbound repair、batch related_docs update、network health review、stale-link cleanup、legacy migration、sidecar extraction、existing-doc reuse scan、actual review record 或 actual completion report。
15. 没有新增 runtime code、package manifests、source/test directories、software test frameworks、schemas、validators、generators、automation、CLI/API/UI/database/deployment/CI、runbooks 或 batch assets。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响](./rename-migration-policy.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
- [Sidecar 与补充材料边界政策：主文档权威、支持资产分类与过期处理](./sidecar-boundary-policy.md)
- [旧文档渐进迁移与兼容策略：缺口分类、保留规则、批量风险判定与剩余缺口处理](./legacy-migration-guide.md)
- [跨文档链接维护政策：existence/path/topic/meaning checks、inbound/outbound review、one-way reason 与 boundary conflict handling](./link-maintenance-policy.md)
- [可复用模型入口政策：core model、boundary rule、decision frame、failure pattern 与 verification method](./reusable-model-entry-points.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](../templates/review-record-template.md)
- [完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界](../templates/completion-report-template.md)
- [文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements](./document-decision-policy.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
