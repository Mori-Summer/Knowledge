---
doc_id: governance-reusable-model-entry-points
title: 可复用模型入口政策：core model、boundary rule、decision frame、failure pattern 与 verification method
concept: reusable_model_entry_points
topic: governance
depth_mode: standard
created_at: '2026-05-29T17:37:26+08:00'
updated_at: '2026-06-17T11:45:28+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/5-3-reusable-model-entry-points.md
  - _bmad-output/implementation-artifacts/5-2-link-maintenance-policy.md
  - _bmad-output/implementation-artifacts/5-1-related-docs-taxonomy.md
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/concept-document-template.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_epic_5_reusable_model_entry_points_review_2026_06_17
applicability: formal_docs_reusable_model_entry_point_and_callable_anchor_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/concept-document-template.md
open_questions:
  - network boundary and decay prevention policy 更新后，是否需要继续把 weak/noisy model anchors 映射到更专门的 Epic 6 batch network review records？
  - Epic 6 建立 batch governance records 后，是否需要为批量 entry point gap review 提供专门记录？
---

# 可复用模型入口政策：core model、boundary rule、decision frame、failure pattern 与 verification method

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 formal `docs/` 资产暴露可复用模型入口和 callable anchor 的 canonical governance asset。它回答的是：已有文档如何让后续问题回到可调用的结构、边界、机制、取舍、失败模式和验证方法，而不是重新从零生成、执行全库扫描、制造噪声链接，或把缺失入口伪装成已经可复用。

本文适用于：

- 正式概念文档、方法论资产、治理资产和模板资产中的 reusable model entry point 判断。
- review evidence、completion evidence、story Dev Agent Record、target body、Existing Doc Reuse Scan Record 或明确命名的 model-entry note 中的人类可读入口记录。
- 已知候选文档或已被直接引用的既有文档，是否能为新问题提供 `core_model`、`boundary_rule`、`decision_frame`、`failure_pattern` 或 `verification_method`。
- examples、self-tests 和 open questions 是否支撑 recognition、boundary judgment、scenario application、counterexample analysis 或 transfer judgment。
- 缺失可复用入口时，如何记录为质量、修订、审查、模型或迁移缺口。

本文不适用于：

- 执行 existing-doc reuse scan、topic directory scan、`docs/index.md` reuse scan、related docs scan、neighboring concepts scan、duplicate scan 或 full-repo model-entry inventory。
- 批量分配 entry points、批量更新 `related_docs`、批量 link normalization、network health review、stale relationship cleanup、index-wide cleanup 或 batch governance。
- 新增 `reusable_entry_points`、`model_entry_points`、`callable_anchors`、`core_model`、`boundary_rule`、`decision_frame`、`failure_pattern`、`verification_method`、`anchor_status`、`reuse_status`、`network_health`、`schema_version` 或类似 frontmatter fields。
- 创建 runtime code、package manifest、`src/`、`tests/`、software tests、CLI/API/UI/database/deployment/CI、machine-readable schema、validator、generator、model scanner、link scanner、index generator 或 automation。

本文补充而不替代以下资产：

- [related docs 与相邻概念关系分类](./related-docs-taxonomy.md)：负责 relation type、target status、transfer object、meaningful-link evidence 和 Related Docs Relationship Record。本文只把 transfer object 进一步接到 callable model entry types。
- [跨文档链接维护政策](./link-maintenance-policy.md)：负责 changed-file link checks、one-way reason、fragment/anchor validation、bounded inbound/outbound review 和 Link Maintenance Record。本文定义 callable anchor 的含义，但不执行 link maintenance scan。
- [既有文档复用流程](./existing-doc-reuse-procedure.md)：负责 new-problem scan surfaces、scan order、scan result taxonomy、reuse evidence 和 Existing Doc Reuse Scan Record。本文只在候选文档已识别后证明是否存在 callable anchor。
- [Frontmatter schema 与 doc_id 身份规则](./frontmatter-schema.md)：负责 baseline fields、YAML arrays、stable `doc_id` 和 unauthorized field 禁令。本文所有 entry point 证据都留在正文或人类可读记录中。
- [docs/index.md 同步与导航治理规则](./index-synchronization-rules.md)：负责 Index Impact outcomes、relative path、formal navigation entry 和 index-wide cleanup 边界。
- [审查记录模板](../templates/review-record-template.md)、[完成汇报模板](../templates/completion-report-template.md) 和 [文档决策政策](./document-decision-policy.md)：负责 review/completion/decision evidence、unverified/future-story handling、quality/revision gaps、hold/defer/not_authorized 和 status impact。
- [统一概念文档规范](../methodology/document-generation-methodology.md)、[统一概念文档质量门禁](../methodology/concept-document-quality-gate.md) 和 [统一概念文档模板](../methodology/concept-document-template.md)：负责模型、边界、例子、自测、未解问题和迁移入口的质量要求。

本文的 owner entry point 是 [docs/index.md](../index.md) 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/reusable-model-entry-points.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：reusable entry point types、callable anchor boundary、new-problem mapping、examples/self-tests/open questions 复用证据、missing entry point gap handling、Reusable Model Entry Point Record、相邻治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record
- source asset: docs/governance/reusable-model-entry-points.md
- trigger: Story 5.3 explicitly authorizes a canonical reusable model entry points governance asset
- index file: docs/index.md
- section: ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: the asset is a formal governance policy with listed_in_docs_index navigation treatment
- validation result: target exists and relative link resolves
- unresolved risk: existing-doc reuse procedure and network boundary / decay prevention policy are direct adjacent authorities; Epic 6 integrations remain future dependencies, recorded in open_questions
```

## Callable Anchor Boundary

Callable anchor 是正文中的人类可读入口、章节、表格、record、说明句或验证提示。它让 reviewer 或 future Agent 能说清：这里可以调用什么模型、边界、判断、失败模式或验证方法，适用于什么新问题，以及何时必须停止迁移。

Callable anchor 不是：

- 自动生成的 anchor registry；
- frontmatter schema；
- lint target；
- database row；
- generated inventory；
- tool input；
- link scanner 或 model scanner 的配置。

一个 callable anchor 至少要让人类 reviewer 看见：

1. 可调用对象是什么。
2. 它支持哪类新问题。
3. 它迁移的是 structure、boundary、mechanism、tradeoff、failure mode 还是 verification method。
4. 它的适用边界和停止条件是什么。
5. 哪个例子、自测、open question 或验证方式能检查这次迁移。

如果正文只有标题相似、关键词相同、路径相邻或泛泛“参见”，不能算 callable anchor。它最多是弱链接候选，必须转向 relationship evidence、link maintenance evidence、quality gap 或 [existing-doc reuse procedure](./existing-doc-reuse-procedure.md)。

## Reusable Entry Point Types

Formal docs 可以暴露以下五类 reusable entry point。每类入口都可以落在正文段落、表格、record、示例说明、自测说明或验证清单中；不得拆成未经授权的 frontmatter fields。

| Entry point type | Required meaning | Must explain |
| --- | --- | --- |
| `core_model` | 文档可被新问题调用的核心结构、机制、因果链、状态模型或 mental model。 | 哪个模型可调用，它期待什么输入/问题形状，以及它支持什么输出、解释或判断。 |
| `boundary_rule` | 判断什么算入/不算入、相邻概念如何区分、适用/不适用条件如何判定。 | 什么算、什么不算，哪个相邻概念容易混淆，哪些条件会停止复用。 |
| `decision_frame` | 用于在具体场景中做选择、拒绝、升级、修订、比较、持有或转向相邻治理资产的判断框架。 | 它支持什么 decision，需要什么证据，正在权衡什么 tradeoff。 |
| `failure_pattern` | 常见误用、弱模型、错误迁移、反例、stale assumption、退化路径或混淆信号。 | 如何识别失败，为什么重要，哪个边界或验证检查能捕捉它。 |
| `verification_method` | 检查、self-test、evidence pattern 或 scenario exercise，用来验证理解、关系、场景应用、反例分析或迁移判断。 | 它验证什么，证据在哪里，什么结果会阻塞复用。 |

正文中可以使用以下辅助 anchor，但它们只是 body-level concepts：

| Supporting anchor | Use | Boundary |
| --- | --- | --- |
| `example_anchor` | 用正例、场景、反例或真实对象帮助 recognition、scenario application、counterexample analysis 或 transfer judgment。 | 不能变成 frontmatter field、schema extension、generated inventory 或 automation input。 |
| `self_test_anchor` | 用问题、练习或判断题测试 recognition、boundary judgment、scenario application、counterexample analysis 或 transfer。 | 不得只问术语记忆；必须检查理解、边界、迁移或反例。 |
| `open_question_anchor` | 暴露未验证边界、待核查条件、迁移风险、source/time uncertainty 或 future model-building direction。 | 不能把 unresolved risk 写成 resolved、reviewed、validated 或 reusable。 |

Entry point type 不替代 [related docs taxonomy](./related-docs-taxonomy.md) 的 relation type。Relation type 说明两个资产之间是什么关系；entry point type 说明某个资产内部哪个模型入口可被调用。Transfer object 可以是 `structure`、`boundary`、`model`、`judgment`、`failure_mode`、`verification_method`、`source_time_context` 或 `lifecycle_index_impact`；entry point type 在这些 transfer object 上进一步说明调用方式。

## New Problem To Existing Model Mapping

当新问题已经有候选 existing document，或用户/Agent 直接指向某份已有文档时，必须把新问题映射到可迁移对象，而不是只靠标题、关键词或路径相邻。

Reusable entry point 必须帮助新问题挂到以下至少一种对象上：

- `structure`：结构、层级、状态模型、分类框架或 record shape。
- `boundary`：纳入/排除规则、相邻概念区分、scope limit 或 duplicate/coexistence boundary。
- `mechanism`：机制、因果链、主链路、状态转换或治理流程形状。
- `tradeoff`：取舍、选择条件、升级条件、拒绝条件、持有条件或相邻资产转向条件。
- `failure_mode`：误用、弱模型、错误迁移、stale assumption、退化路径或反例。
- `verification_method`：自测、证据位置、scenario exercise、counterexample analysis 或 transfer check。

映射不能只写“主题相关”“都提到同一个词”“路径都在同一目录”或“应该能复用”。必须说明可迁移的结构、边界、机制或 tradeoff，以及复用后必须重新验证什么。

推荐使用以下人类可读记录：

```markdown
## New Problem To Existing Model Mapping

- source question:
- candidate existing doc:
- entry point type:
- entry point location:
- callable object:
- mapped object:
- mapped structure / boundary / mechanism / tradeoff:
- fit reason:
- boundary limit:
- transfer risk:
- failure or false-transfer risk:
- verification method:
- next verification:
- result: reusable | reusable_with_limit | revision_needed | no_valid_entry | unverified
```

这个 mapping 不是 [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) 的 scan procedure。它只定义在候选既有文档已经被识别或直接引用之后，什么证据能证明“这份文档有可调用入口”。如何搜索 topic directory、`docs/index.md`、related docs、neighboring concepts、duplicates，以及如何分类 exact/adjacent/prereq/contrast/no-asset results，使用 existing-doc reuse procedure。

如果判断需要 topic directory scan、`docs/index.md` reuse scan、related docs scan、neighboring concepts scan 或 duplicate scan，使用 [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) 的窄范围流程；如果需要 full-repo model inventory、all-doc reuse scan 或 batch-shaped work，记录为 `deferred_with_reason`、batch readiness 或 not authorized，而不是偷偷完成一半并宣称复用已验证。

## Examples, Self-Tests And Open Questions

Examples、self-tests 和 open questions 只有在支撑 reusable entry point 时才算复用证据。它们不能只是模板填充内容。

| Evidence type | Reuse responsibility | Reviewer check |
| --- | --- | --- |
| examples | 支持 recognition、scenario application、counterexample analysis 和 transfer judgment。 | 示例必须说明它验证了哪个 model、boundary、mechanism、failure pattern 或 tradeoff。 |
| self-tests | 支持 recognition、boundary judgment、scenario application、counterexample analysis 和 transfer judgment。 | 自测不得只问术语记忆；必须能检查解释、边界、机制、反例或迁移。 |
| open questions | 暴露未验证边界、待核查条件、迁移风险、source/time uncertainty 或 future model-building direction。 | 未解问题必须说明它限制哪个入口、哪类迁移或哪个验证结论。 |

Examples/self-tests/open_questions 与 reusable entry point 的关系必须可被 reviewer 检查。合格表述示例：

- “这个 example_anchor 证明 `boundary_rule` 能区分 owner entry 与 related_docs；若读者仍把 index entry 当 related relationship，则复用失败。”
- “这个 self_test_anchor 检查 `decision_frame` 是否能判断 link maintenance 与 existing-doc scan 的边界；答不出停止条件时，不能迁移该判断。”
- “这个 open_question_anchor 暴露本轮没有执行 existing-doc reuse scan，因此当前只能判断候选文档的 callable anchor，不能宣称完成 reuse scan。”

缺少 examples/self-tests/open_questions 支撑时，不能宣称某个入口已经支持对应的 recognition、boundary judgment、scenario application、counterexample analysis 或 transfer judgment。可行处理是记录 `quality_gap`、`revision_gap`、`review_gap`、`model_gap` 或 `transfer_gap`，并说明是否阻塞当前用途。

## Missing Entry Point Gap Handling

缺失 reusable entry point 不等于文档必须废弃。它意味着当前文档在某个复用场景下缺少可调用入口，需要按缺口是否阻塞当前用途路由到 targeted revision、quality gate issue、open question、future story、hold for Maxwell 或 not authorized。

缺失入口使用人类可读状态记录，例如：

- `quality_gap`：质量门禁或等价治理检查缺少模型、边界、验证或迁移证据。
- `revision_gap`：目标文档需要定向修订才能暴露入口。
- `review_gap`：review evidence 未检查或未记录入口。
- `model_gap`：核心结构、机制、decision frame 或 failure pattern 尚未建立。
- `transfer_gap`：无法把新问题安全映射到已有结构、边界、机制或 tradeoff。

缺口记录至少包含：

| Field | Required evidence |
| --- | --- |
| affected document | 哪份正式资产或候选资产受影响。 |
| missing entry type | 缺 `core_model`、`boundary_rule`、`decision_frame`、`failure_pattern`、`verification_method` 或 supporting anchor。 |
| blocked reuse scenario | 哪个新问题、迁移、review 或 completion claim 被阻塞。 |
| evidence location | 缺口出现在 frontmatter、正文章节、例子、自测、open questions、review evidence、completion evidence 或 story evidence 的哪里。 |
| proposed fix | 应补模型、边界、示例、自测、open question、link evidence、review evidence 或 completion evidence。 |
| owner/future story | 当前 Agent、reviewer、Maxwell、existing-doc reuse procedure、network boundary / decay prevention policy、Epic 6 或其他 owner。 |
| decision impact | targeted revision、quality gate issue、open question、future story、hold for Maxwell、defer_with_reason 或 not_authorized。 |
| validation status | pass、fail、not_applicable、unverified、planned 或 unresolved。 |

[审查记录模板](../templates/review-record-template.md) 可以承载 quality/revision gaps、unverified items、future-story handling 和 reviewer-visible blockers。[完成汇报模板](../templates/completion-report-template.md) 可以汇总 unresolved risks、not authorized scope、changed-file evidence 和 non-software boundary。[文档决策政策](./document-decision-policy.md) 负责 targeted_revision、hold_for_clarification、defer_with_reason、not_authorized 等 decision wording。

Unresolved gap 不得被 completion wording 描述为 resolved、reviewed、validated、maintained、fully reusable 或 safely transferable。若缺口不阻塞当前 narrow use，可以写成 nonblocking follow-up；若它阻塞当前 reuse claim，必须进入 targeted revision、hold/defer 或 not authorized。

## Reusable Model Entry Point Record

`Reusable Model Entry Point Record` 是 copyable human-readable record。它不是 schema、frontmatter extension、lint rule、generator input、database row 或 automation contract。它可以落在 target body、review evidence、completion evidence、story Dev Agent Record、Existing Doc Reuse Scan Record 或明确命名的 model-entry note 中。

```markdown
## Reusable Model Entry Point Record

- source asset:
- entry point location:
- entry point type: core_model | boundary_rule | decision_frame | failure_pattern | verification_method | other_with_reason
- anchor wording:
- callable object:
- target reuse scenario:
- mapped structure / boundary / mechanism / tradeoff:
- fit reason:
- boundary limit:
- failure pattern:
- verification method:
- example support:
- self-test support:
- open-question support:
- related docs impact:
- link maintenance impact:
- quality or revision gap:
- owner/future story:
- validation result: pass | fail | not_applicable | unverified
```

Record 字段不得拆进未经授权的 frontmatter fields。若未来确实需要 schema 化 reusable entry point、anchor status、reuse status、network health 或 batch model-entry evidence，必须由 frontmatter schema、version governance、network boundary / decay prevention policy、Epic 6 或 Maxwell 明确授权。

## Adjacent Governance Asset Boundaries

本文调用相邻治理资产，但不复制或替代它们的完整职责：

| Asset | This policy uses it for | Boundary kept here |
| --- | --- | --- |
| [related docs taxonomy](./related-docs-taxonomy.md) | relation type、target status、transfer object、meaningful-link evidence 和 Relationship Record。 | 本文定义 callable model entry types，不改写 taxonomy。 |
| [link maintenance policy](./link-maintenance-policy.md) | changed-file link checks、one-way reason、fragment/anchor validation、bounded inbound/outbound review 和 Link Maintenance Record。 | 本文不执行 link maintenance scan、stale-link cleanup 或 inbound/outbound repair。 |
| [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) | new-problem scan surfaces、scan order、scan result taxonomy、reuse evidence 和 Existing Doc Reuse Scan Record。 | 本文只证明候选 existing doc 的 callable anchor，不执行候选发现或 scan result classification。 |
| [frontmatter schema](./frontmatter-schema.md) | baseline fields、YAML arrays、stable `doc_id` 和 unauthorized field 禁令。 | 本文不新增 entry point fields。 |
| [index synchronization rules](./index-synchronization-rules.md) | Index Impact outcomes、relative path、formal navigation entry 和 index-wide cleanup boundary。 | 本文只记录本资产的 index impact，不执行 index-wide cleanup。 |
| [document decision policy](./document-decision-policy.md) | targeted revision、hold/defer/not_authorized、status impact 和 decision evidence。 | 本文不发明新 decision labels。 |
| [review record template](../templates/review-record-template.md) | review classification、Hard Fail、unverified items、quality/revision gap evidence。 | 本文不创建 actual review record。 |
| [completion report template](../templates/completion-report-template.md) | completion evidence、unresolved risk、changed files、non-software boundary。 | 本文不创建 actual completion report。 |
| [document generation methodology](../methodology/document-generation-methodology.md) | 模型型/纯概念路径、结构、边界、验证和迁移入口要求。 | 本文不创建平行主方法论。 |
| [concept document quality gate](../methodology/concept-document-quality-gate.md) | Hard Fail、验证/迁移、元数据/仓库纪律和等价治理检查。 | 本文不替代质量门禁评分或 final decision。 |
| [concept document template](../methodology/concept-document-template.md) | examples、self-tests、open questions、迁移入口和可复用结构信息位点。 | 本文不改写普通概念文档模板。 |

Adjacent and future boundaries：

- [existing-doc reuse procedure](./existing-doc-reuse-procedure.md) owns topic/index/related-doc/neighboring/duplicate scan sequence and exact/adjacent/prereq/contrast/no-asset scan results。
- [network boundary and decay prevention policy](./network-boundary-and-decay-prevention.md) owns identification and routing of dense/noisy/weak/stale relationship signals、topic health、repeated overlap、network-decay risk and navigation relevance evidence; actual cleanup remains bounded link maintenance, batch readiness or Epic 6 work。
- Epic 6 owns batch entry-point inventory、batch review/completion records、runbooks and batch governance。

If applying this policy reveals weak/noisy/stale relationship signals、topic-boundary drift、or navigation relevance gaps, route classification to [network boundary and decay prevention policy](./network-boundary-and-decay-prevention.md). If the work requires full-repo model-entry scan、all-doc reuse scan、batch entry-point assignment、network health review、stale-link cleanup、batch related_docs update 或 index-wide cleanup, stop and route execution to Epic 6 or batch readiness as applicable.

## Validation Checklist

Validate this policy or future applications of it with the following checklist:

1. Target file exists at `docs/governance/reusable-model-entry-points.md`.
2. Target frontmatter contains all required baseline fields.
3. `source_basis`, `related_docs` and `open_questions` are YAML arrays.
4. H1, frontmatter `title` and `docs/index.md` display title are consistent.
5. Entry point types cover `core_model`, `boundary_rule`, `decision_frame`, `failure_pattern` and `verification_method`.
6. Callable anchor is defined as body-level human-readable evidence, not registry、schema、lint target、database、tool input or automation.
7. New-problem mapping requires structure、boundary、mechanism、tradeoff、failure mode or verification method evidence, without implementing existing-doc reuse scan procedure.
8. Examples、self-tests and open questions support recognition、boundary judgment、scenario application、counterexample analysis or transfer judgment.
9. Missing entry points are recorded as quality/revision/review/model/transfer gaps with affected document、missing type、blocked scenario、evidence location、owner/future story、decision impact and validation status.
10. Reusable Model Entry Point Record exists and is explicitly non-schema、non-frontmatter and non-automation.
11. Related docs taxonomy、link maintenance policy、frontmatter schema、index synchronization rules、review/completion templates、document decision policy、methodology、quality gate and template boundaries are explicit.
12. Existing-doc reuse procedure、network boundary and decay prevention policy and Epic 6 boundaries are explicit.
13. Body links, frontmatter `related_docs` and `docs/index.md` entry are path-resolvable for existing files.
14. Planned/missing future targets are recorded in `open_questions` or future dependency notes.
15. No unauthorized frontmatter fields are introduced.
16. No existing-doc reuse scan、full-repo model-entry scan、topic directory scan、`docs/index.md` reuse scan、related docs scan、neighboring concepts scan、duplicate scan、batch entry-point assignment、network health review、stale-link cleanup、batch related_docs update or index-wide cleanup is performed.
17. No runtime code、package manifests、source/test directories、software test framework、schemas、validators、generators、automation、link scanners、model scanners、index generators、CLI/API/UI/database/deployment/CI、runbooks、actual review records or actual completion reports are created.

Maintenance triggers:

- related docs taxonomy、link maintenance policy、existing-doc reuse procedure 或 network boundary and decay prevention policy 更新后，复核 callable anchor、transfer object、scan boundary 和 weak/noisy relationship routing。
- revision/regeneration continuity、legacy migration 或 sidecar boundary policy 更新后，复核 model-entry note、support artifact 和 old-content continuity 边界。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report 后，复核 batch entry-point inventory、batch gap review 和 runbook record 落点。
- Maxwell 明确授权 schema、scanner、generator、automation、new global frontmatter fields 或 batch model-entry work 后，复核本文非软件边界和字段边界。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling](./related-docs-taxonomy.md)
- [跨文档链接维护政策：existence/path/topic/meaning checks、inbound/outbound review、one-way reason 与 boundary conflict handling](./link-maintenance-policy.md)
- [既有文档复用流程：发现、对齐、复用决策与避免重复生成](./existing-doc-reuse-procedure.md)
- [知识网络主题边界与退化防护政策](./network-boundary-and-decay-prevention.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
- [旧文档渐进迁移与兼容策略：缺口分类、保留规则、批量风险判定与剩余缺口处理](./legacy-migration-guide.md)
- [Sidecar 与补充材料边界政策：主文档权威、支持资产分类与过期处理](./sidecar-boundary-policy.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](../templates/review-record-template.md)
- [完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界](../templates/completion-report-template.md)
- [文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements](./document-decision-policy.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [统一概念文档模板](../methodology/concept-document-template.md)
