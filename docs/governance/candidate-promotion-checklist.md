---
doc_id: governance-candidate-promotion-checklist
title: 候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁
concept: candidate_promotion_checklist
topic: governance
depth_mode: standard
created_at: '2026-05-26T10:16:23+08:00'
updated_at: '2026-06-16T09:42:19+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/2-3-candidate-promotion-checklist.md
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_epic_2_governance_review_2026_06_16
applicability: candidate_to_formal_docs_promotion_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
open_questions:
  - '`docs/governance/revision-regeneration-continuity-policy.md` 已建立 Revision / Regeneration Continuity Record；是否需要在后续专门审查中把 promotion replacement evidence 与该 record 进一步对齐？'
  - duplicate/coexistence policy 已定义后，是否还需要为 same-topic candidate coexistence 增加更细判定表？
  - 如果 completion-report template 后续调整 Promotion Decision Record summary 字段，是否需要同步本文的 promotion decision record 汇总方式？
---

# 候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中候选文档晋升为正式 `docs/` 资产的 canonical governance asset。它回答的是：对话输出、`_bmad-output/` 草稿、planning output、review output、completion record、临时 Markdown 或手工候选稿，进入正式知识库前必须检查什么、记录什么、什么时候停止，以及哪些判断必须留给 follow-up owner 或 Maxwell 确认。

本文适用于以下候选来源准备进入 `docs/` 的场景：

- 对话中生成的 Markdown、结构草案、规则片段或候选正文。
- `_bmad-output/` 下的 planning artifacts、story files、review outputs、completion records、readiness reports 和其他 workflow outputs。
- 临时 Markdown 草稿、copied snippets、手工整理的候选文档或从旧记录抽出的候选内容。
- 已有 formal asset 的修订候选、替换候选、继任候选、合并/拆分候选或重复概念候选。

本文补充 `docs/methodology/document-generation-methodology.md`、`docs/methodology/concept-document-contract.md`、`docs/methodology/intake-and-intent-classification.md`、`docs/methodology/concept-document-quality-gate.md`、`docs/methodology/source-discipline-and-real-world-anchor-policy.md`、`docs/governance/frontmatter-schema.md`、`docs/governance/topic-path-naming-policy.md`、`docs/governance/index-synchronization-rules.md`、`docs/governance/rename-migration-policy.md`、`docs/governance/duplicate-and-coexistence-policy.md`、`docs/governance/governance-asset-navigation-policy.md`、`docs/governance/lifecycle-states.md` 和 `docs/governance/batch-readiness-checklist.md`。本文同时依赖 `docs/governance/related-docs-taxonomy.md`、`docs/governance/link-maintenance-policy.md`、`docs/governance/reusable-model-entry-points.md`、`docs/governance/existing-doc-reuse-procedure.md`、`docs/governance/network-boundary-and-decay-prevention.md`、`docs/templates/review-record-template.md`、`docs/templates/completion-report-template.md`、`docs/governance/document-decision-policy.md` 和 `docs/governance/rework-loop-examples.md` 来界定相邻链接、复用、网络退化、review/decision/completion evidence 和返工边界。

本文不替代主方法论、概念文档合同、质量门禁、`docs/governance/frontmatter-schema.md` 或 `docs/governance/topic-path-naming-policy.md`。晋升时必须调用这些资产，而不是在本文中重建第二套 schema、路径政策或质量门禁。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/candidate-promotion-checklist.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：candidate source 边界、promotion gate、replacement/identity continuity、Promotion Decision Record、相邻治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

## 候选来源与非正式边界

候选来源可以提供 `source_basis`、candidate evidence 或 promotion source，但它们不会因为存在、被审查、被故事标记完成或被对话认可，就自动成为正式 `docs/` 资产。

| 来源类别 | 可以证明什么 | 不能证明什么 |
| --- | --- | --- |
| conversation output | 当前用户语境、候选正文、结构草案、局部规则或 operator intent | 不能证明正式发布、质量通过、路径正确或索引可加入 |
| `_bmad-output/` planning artifacts | PRD、Architecture、Epics、readiness、sprint 语境和 story 约束 | 不能直接成为 formal docs，也不能直接加入 `docs/index.md` |
| `_bmad-output/` story files | 实施合同、验收条件、Dev Agent Record、File List 和工作流状态 | 不能替代正式治理资产或完成 promotion evidence |
| review output | 问题证据、修复依据、风险记录和可能的 follow-up | 不能自动改写为正式规则，除非完成晋升检查 |
| completion records | 完成证据、changed files、验证结果和未解决风险 | 不能单独证明目标资产已经 stable、reviewed 或 validated |
| temporary Markdown drafts | 候选正文、待整理结构、局部例子和待迁移内容 | 不能绕过 frontmatter、路径、来源、质量和索引门禁 |
| copied snippets | 可复用片段、术语、例子或 checklist 候选 | 不能证明上下文、来源、适用范围或身份连续性 |
| manually prepared candidates | 人工整理的候选文件或待晋升内容 | 不能因为人工准备而跳过 promotion check |
| existing formal asset | 既有正式资产的当前身份、旧内容、引用关系、生命周期语义和修订/替换基线 | 不能证明候选修订可 silent overwrite、可改变 `doc_id`、可迁移路径或可移除高价值旧内容 |

禁止把 `_bmad-output/` 文件直接加入 `docs/index.md`。`docs/index.md` 只导航正式 `docs/` 资产和明确保留的正式报告入口；workflow output 的 owner entry point 仍是 BMad workflow、story、sprint tracking 或对应记录。

以下信号都不足以单独证明正式晋升：

- 候选文件已经放在某个临时位置。
- story status 是 `ready-for-dev`、`in-progress`、`review` 或 `done`。
- review output 给出同意、建议或无阻塞意见。
- 对话中出现“可以”“确认”“看起来没问题”等局部批准。
- 候选正文已经有标题、章节或看起来完整的 Markdown。
- 候选内容被某个 formal doc 引用为来源。

允许的候选结果必须落入下面的 canonical `operator decision` 值。候选结果、decision record 和后续 review evidence 使用同一组值，避免同一决策在不同记录中出现两套枚举。

| operator decision | 使用条件 |
| --- | --- |
| `promote` | 新资产身份、路径、frontmatter、正文、来源、质量、索引和风险均可审查 |
| `revise_before_promotion` | 可修复缺口明确，但当前版本不能进入正式资产 |
| `reject` | 候选内容不可靠、重复价值不足、来源不可用或不应进入知识库 |
| `hold_for_clarification` | 信息有价值，但 target path、身份、来源、质量、链接、旧内容处理或 owner decision 不足，需要 Maxwell 或 owner 确认后才能继续 |
| `defer_with_reason` | 需要相邻治理资产、review/completion templates、related-doc/link/reuse/network governance 或 Epic 6 的专门规则，并且本轮必须记录 defer reason、impact、owner/follow-up dependency 和 re-review trigger |
| `replace_with_continuity_record` | 替换已有 formal asset，且身份连续、旧内容处理和引用影响已记录 |
| `not_authorized` | 候选需要的文件范围、状态声明、自动化、批量操作或 future-story 资产未被当前授权范围覆盖 |

## Promotion Gate

候选文档准备写入、覆盖、替换或升级正式 `docs/` 资产时，必须按下面顺序检查。任一检查失败，候选内容只能保留为 candidate、进入修订、拒绝、`hold_for_clarification` 或 `defer_with_reason`；不得用索引更新或质量状态字段暗示已经晋升。

### 1. Source classification

先记录候选来源类别、原始位置和当前用途。最低记录包括：

- source location，例如对话、`_bmad-output/...`、临时文件、review output 或手工候选路径。
- source type，例如 conversation output、planning artifact、story file、review output、completion record、temporary draft、snippet、manual candidate 或 existing formal asset。
- source role，例如 source_basis、candidate body、evidence、revision input、replacement proposal 或 rejected input。
- candidate limitations，例如未核查、缺少时间语境、身份不清、只代表局部建议或等待 owner decision。

### 2. Target asset class

晋升前必须判断目标资产类别：

- 普通概念/知识资产：默认进入 `docs/{topic}/{kebab-case-slug}.md`。
- 方法论资产：进入 `docs/methodology/{kebab-case-slug}.md`。
- 治理资产：进入 `docs/governance/{kebab-case-slug}.md`。
- 报告/归档性正式资产：进入受控 `docs/_reports/{kebab-case-slug}.md`，并使用与该路径兼容的 status 和 navigation 语义。
- Template 或 future runbook：正式模板资产进入 `docs/templates/`；runbook 资产只有在后续 story 或 Maxwell 明确批准后进入对应目录。

如果 asset class 不清，必须停止或记录 open question。不得为了让候选稿进入索引而随手选一个目录。

### 3. Target topic、path、filename 与 identity

按 `docs/governance/topic-path-naming-policy.md` 检查：

- target topic 是否由 reuse context、正文对象、neighboring documents、现有 topic directories、`docs/index.md` grouping 和 related docs 支持。
- target path 是否符合资产类别。
- filename 是否是稳定、简短、能表达对象的 `kebab-case` slug。
- frontmatter `concept` 是否是稳定 `snake_case`，且不复制完整标题。
- `doc_id` 是否稳定、唯一、不会随 title、path、topic、H1 或 filename 静默变化。

topic/path/filename/doc_id 任一项无法决定时，停止。候选内容不得进入 `docs/index.md`，也不得声明 promoted、accepted、reviewed、validated 或等价通过状态。

### 4. Frontmatter schema

按 `docs/governance/frontmatter-schema.md` 检查 baseline fields。正式资产至少要有：

- `doc_id`
- `title`
- `concept`
- `topic`
- `depth_mode`
- `created_at`
- `updated_at`
- `source_basis`
- `time_context`
- `applicability`
- `prompt_version`
- `template_version`
- `quality_status`
- `related_docs`
- `open_questions`

`source_basis`、`related_docs` 和 `open_questions` 必须使用 YAML arrays。缺失目标、未来资产、待核查链接或 unresolved risks 应进入 `open_questions` 或 promotion record，而不是伪造为已存在的 related doc。

不得新增未经授权的全局 frontmatter 字段，包括 `schema_version`、`lifecycle_state`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`path_policy_version`、`promotion_policy_version`、`migration_status`、`quality_gate_version`、`review_record_version` 或 `decision_status`。这些信息可以写在正文、review evidence、completion evidence、story Dev Agent Record、review/completion templates 或 future runbook records 中。

### 5. Body structure 与 required information

如果目标是普通概念文档，按 `docs/methodology/document-generation-methodology.md` 和 `docs/methodology/concept-document-contract.md` 检查：

- 文档路径类型：模型型概念文档或纯概念文档。
- 展开密度：`standard` 或 `deep`。
- 必需信息位点：problem/naming、object boundary、non-object scope、adjacent distinction、conditions、failure/misreading、verification、transfer、anchor、self-test 和 open questions。

如果目标是治理、方法论、模板、索引、报告或未来 runbook，应执行等价治理检查：

- role、authority、applicable scope 和不替代关系清楚。
- 与主方法论、相关治理资产和 source hierarchy 的关系清楚。
- owner entry point、navigation treatment 和 index impact 写在正文或 completion evidence 中。
- source/time/status/lifecycle/open questions 可追溯。
- workflow contract、future-story boundary 和 non-software boundary 没有被破坏。

### 6. Quality gate 或等价治理门禁

普通概念文档必须使用 `docs/methodology/concept-document-quality-gate.md` 检查 Hard Fail 和质量证据。治理、方法论、模板、索引、runbook 或报告资产不按普通概念文档六项评分证明通过，但必须执行等价门禁：

- role、authority、scope 和 main methodology relationship 明确。
- frontmatter、path、title、H1、source_basis、time_context、quality_status 和 body 不冲突。
- index/navigation treatment 已判断。
- related docs、正文链接和 changed-file links 可定位，或缺口记录为 open question。
- lifecycle/status 语义保守，不用字段伪造已审查、已验证、已迁移或已维护。
- 未新增 runtime code、package manifest、software tests、lint/scoring automation、CLI/API/UI/database/deployment/CI 或 executable tooling。

### 7. Source discipline 与 time context

候选内容只要包含 current-practice、historical/deprecated practice、外部事实、产品/标准/监管/市场状态或真实世界锚点，就必须调用 `docs/methodology/source-discipline-and-real-world-anchor-policy.md`。

最低要求是：

- 写明核对日期、项目阶段或来源时间语境。
- 区分 verified fact、source-based synthesis、project rule、user context、inference 和 open question。
- 当前性声明有相称来源和范围限制。
- 历史/废弃实践不伪装成当前推荐。
- 不可验证声明被删除、收窄、标为推断、移入 open questions，或停止等待确认。

项目内治理规则可以由 formal docs、当前授权范围、BMad workflow 或 Maxwell 明确指令支撑；它们不能被写成外部行业事实。

### 8. Links、related docs 与 changed-file links

晋升前必须检查：

- frontmatter `related_docs` 目标存在，且是正式资产或明确允许的正式报告入口。
- 正文内链能解析到目标文件。
- changed-file links、source/target path claims 和 reference links 不指向旧路径或候选路径。
- missing/planned targets 被记录为 open questions 或 follow-up dependency。
- inbound references 如果容易发现，应检查是否会被替换、移动或重命名影响。

如果关键链接无法在当前授权范围内修复，停止或记录 blocked/deferred。不得把断链候选晋升为通过状态。

### 9. `docs/index.md` impact

每次 candidate promotion 都必须分类 index impact：

| index impact | 使用条件 |
| --- | --- |
| `updated` | 新增、移动、重命名、改标题、改 topic 或改变导航入口，且当前授权范围允许更新 |
| `not_applicable` | 候选未晋升、资产不属于正式导航、或本次没有导航变化，并说明原因 |
| `referenced_elsewhere` | 未列入 `docs/index.md`，但由明确 methodology、governance、template、runbook 或其他正式 entry point 引用 |
| `intentionally_excluded` | 按 governance navigation policy 受控排除，并记录排除理由、owner entry point、remaining access expectation 和 re-review trigger |
| `deferred_with_reason` | 发现需要同步，但详细规则、目标集或 owner decision 超出当前授权范围 |
| `blocked_index_policy_conflict` | 存在 unresolved index-policy conflict，继续会制造错误导航或错误权威，必须停止或请求 Maxwell 确认 |

不得把 `_bmad-output/`、review output、story file 或 conversation output 直接列入 `docs/index.md`。索引入口只能指向正式 `docs/` 资产或已明确保留的正式报告入口。

`blocked_pending_story_2_4` 是 `docs/governance/index-synchronization-rules.md` 落地前的历史占位状态。该政策落地后的新记录不得继续使用该值；未解决 index impact 应改为 `deferred_with_reason` 并命名后续 owner，或在存在索引政策冲突时使用 `blocked_index_policy_conflict` 并停止处理。

### 10. Operator decision

最后记录 operator decision。没有 decision record 的候选不能算完成晋升。Decision record 可以写在正文、review evidence、completion evidence、story Dev Agent Record、review/completion templates 或 future runbook records 中；即使 review/completion templates 已落地，也不得新增未经授权的全局 frontmatter 字段。

## 替换、覆盖与身份连续性

候选内容会替换、覆盖、升级或继承已有 formal asset 时，必须先判断关系类型：

| 关系类型 | 判断问题 | 默认处理 |
| --- | --- | --- |
| new asset | 仓库是否没有相同身份或同一对象的正式资产？ | 可按新资产 promotion gate 检查 |
| revision | 是否是同一 `doc_id` 的实质修订？ | 保留稳定 `doc_id`，记录变化和质量影响 |
| replacement | 是否用新正文替换旧资产，但身份仍连续？ | 保留 `doc_id`，记录 old-content handling 和 link/index impact |
| split | 一个旧资产是否拆成多个资产？ | defer to `docs/governance/rename-migration-policy.md`，除非已有 Maxwell 明确授权 |
| merge | 多个旧资产是否合并成一个资产？ | defer to `docs/governance/rename-migration-policy.md`，除非已有 Maxwell 明确授权 |
| successor | 新资产是否成为旧资产继任入口？ | defer to `docs/governance/rename-migration-policy.md`，记录 successor ambiguity |
| deprecation/archive | 旧资产是否要降级、废弃或归档？ | defer to `docs/governance/rename-migration-policy.md` 或生命周期专门流程 |
| duplicate/coexistence | 是否存在重复概念或同主题共存风险？ | defer to `docs/governance/duplicate-and-coexistence-policy.md`，除非差异和共存理由已授权 |

简单修订或 replacement 且身份连续时，必须保留旧 `doc_id`。不得因为 title、H1、path、filename、topic 或 body 重写而静默生成新 `doc_id`。

旧内容处理必须明确记录：

- 保留哪些高价值内容。
- 删除、压缩或改写哪些内容，以及理由。
- 是否保留历史说明、successor note、open question 或 review evidence。
- 是否影响 `source_basis`、`time_context`、`related_docs`、body links、index title 或 navigation treatment。
- 是否需要恢复路径或旧版本查找入口。

以下情况不得 silent overwrite：

- identity continuity 不确定。
- split、merge、successor、deprecation、archive 或 link migration 需要专门决策。
- duplicate concept 或 same-topic coexistence 风险未解决。
- stable identity、canonical entry point、lifecycle state、broad navigation 或 existing high-value content 会受影响，且没有当前授权范围或 Maxwell 明确授权。

必须请求 Maxwell 确认的情况包括：

- 替换会改变 stable `doc_id` 或 canonical entry point。
- 替换会把一个 active/maintained/validated 资产降级、废弃或归档。
- 替换会移除高价值旧内容，且不能从当前授权范围或质量门禁明确推出。
- 替换会引发 broad navigation、link migration、topic move、same-topic coexistence 或 successor 判断。

## Promotion Decision Record

Promotion decision record 是 human-readable record/checklist，不是 executable schema、lint rule 或自动化工具。最低字段如下：

| 字段 | 必须记录什么 |
| --- | --- |
| source location | 候选内容的原始位置，例如对话、`_bmad-output/...`、review output、临时文件或手工候选 |
| source type | conversation output、planning artifact、story file、review output、completion record、temporary draft、snippet、manual candidate 或 existing formal asset |
| target path | 准备写入或更新的正式 `docs/` 路径 |
| target asset class | concept document、methodology asset、governance asset、report、template、runbook 或其他已授权类别 |
| target topic | frontmatter `topic` 与目录/索引分组判断 |
| filename | `kebab-case` 文件名和命名理由 |
| `doc_id` | 新身份、保留身份或 identity ambiguity |
| operator decision | promotion 决策值 |
| quality/review decision | quality gate 或等价治理门禁结果 |
| allowed `quality_status` | 当前允许写入的保守质量状态 |
| lifecycle/status note | 生命周期语义如何表达；不得新增未经授权 `lifecycle_state` frontmatter |
| index impact | `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict`；旧记录中的 `blocked_pending_story_2_4` 仅表示 `docs/governance/index-synchronization-rules.md` 落地前的历史占位 |
| related-doc/link impact | `related_docs`、正文链接、changed-file links、入链/出链和缺失目标处理 |
| old-content handling | 适用于 revision、replacement、split、merge、successor、deprecation 或 archive |
| follow-up work | 后续修订、核查、review、migration 或 follow-up dependency |
| unresolved risks | 仍未解决但不阻塞或阻塞的风险 |
| non-software boundary evidence | 确认未创建 runtime code、package manifest、software tests、automation、CLI/API/UI/database/deployment/CI |

允许的 operator decision values 是：

- `promote`
- `revise_before_promotion`
- `reject`
- `hold_for_clarification`
- `defer_with_reason`
- `replace_with_continuity_record`
- `not_authorized`

这些 record 字段属于正文、review evidence、completion evidence、story Dev Agent Record、review/completion templates 或 future runbook records。它们不是新的全局 frontmatter 字段。本文可以定义 required evidence fields，但不替代 review record template、document decision policy、rework loop examples 或 completion report template。

推荐的人工记录形状如下：

```text
Promotion Decision Record

- source location:
- source type:
- target path:
- target asset class:
- target topic:
- filename:
- doc_id:
- operator decision:
- quality/review decision:
- allowed quality_status:
- lifecycle/status note:
- index impact:
- related-doc/link impact:
- old-content handling:
- follow-up work:
- unresolved risks:
- non-software boundary evidence:
```

## 停止条件与后续归属

遇到以下任一情况，必须停止晋升、保留 candidate 状态、记录 open question 或请求 Maxwell 确认：

- canonical topic、target path、title、`doc_id` 或 asset class 无法从当前上下文决定。
- source/time evidence 不足，但候选内容声称 current、validated、accepted、maintained、reviewed 或等价强状态。
- 存在 Hard Fail 或等价治理 Hard Fail，而 promotion 会暗示通过状态。
- `related_docs`、正文链接、changed-file links、index entry 或 source/target path claim 存在关键断裂，且当前范围无法修复。
- promotion 需要 batch work、bulk normalization、index-wide restructuring、broad link migration 或 actual batch execution。
- promotion 需要 executable tooling、lint/scoring automation、CLI/API/UI/database/deployment/CI、software tests 或 runtime automation。
- promotion 会改变 stable identity、canonical entry point、lifecycle/status、broad navigation 或 high-value existing content，但缺少 Maxwell 明确确认。
- promotion 需要创建、重写或提前执行相邻治理资产、review/completion templates、related-doc/link/reuse/network governance 或 Epic 6 batch assets。

后续归属边界：

- `docs/governance/index-synchronization-rules.md` 负责详细 `docs/index.md` synchronization rules。
- `docs/governance/rename-migration-policy.md` 负责 rename、migration、split、merge、deprecation、successor 和 link-impact policy。
- `docs/governance/duplicate-and-coexistence-policy.md` 负责 duplicate concept 和 same-topic coexistence policy。
- `docs/templates/review-record-template.md`、`docs/governance/document-decision-policy.md`、`docs/governance/rework-loop-examples.md` 和 `docs/templates/completion-report-template.md` 负责 review/decision/completion evidence。
- `docs/governance/related-docs-taxonomy.md`、`docs/governance/link-maintenance-policy.md`、`docs/governance/reusable-model-entry-points.md`、`docs/governance/existing-doc-reuse-procedure.md` 和 `docs/governance/network-boundary-and-decay-prevention.md` 负责相邻关系、链接维护、复用入口和网络退化路由。
- Epic 6 负责 batch governance runbook、batch review record 和 batch completion report。

本文可以引用这些相邻归属，但不创建对应资产、不折叠其完整范围，也不执行实际 batch promotion。

## 检查清单

后续 Agent 或 reviewer 在 candidate promotion 前，至少按下面清单逐项确认：

1. 候选来源、source location、source type 和 source limitations 已记录。
2. 候选仍保持 candidate 状态，直到 promotion gate 明确通过。
3. 没有把 `_bmad-output/`、review output、story file 或 conversation output 直接加入 `docs/index.md`。
4. 目标 asset class、topic、path、filename、`concept` 和 `doc_id` 已按 `docs/governance/frontmatter-schema.md` 与 `docs/governance/topic-path-naming-policy.md` 检查。
5. Frontmatter baseline fields 完整，`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
6. 没有新增未经授权的全局 frontmatter 字段。
7. 正文结构覆盖对应资产类型的必需信息位点，或治理/方法论等价检查项。
8. 概念文档已执行质量门禁；非概念资产已执行等价治理门禁。
9. Source/time/currentness/historical/deprecated/real-world anchor 声明已按来源纪律处理。
10. `related_docs`、正文链接、changed-file links、missing/planned targets 和可发现入链/出链已检查。
11. `docs/index.md` impact 已分类为 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict`；旧记录中的 `blocked_pending_story_2_4` 已按 `docs/governance/index-synchronization-rules.md` 视为历史占位。
12. 替换、覆盖、修订、继任、废弃、拆分、合并或重复共存风险已记录 disposition。
13. Maxwell confirmation 条件已检查；需要确认时没有继续写入。
14. Promotion Decision Record 已包含 source、target、decision、quality/status、index/link、old-content、follow-up、risks 和 non-software boundary evidence。
15. 未创建 runtime code、package manifest、source tree、software tests、build config、automation、CLI/API/UI/database/deployment/CI 或未经授权的 future-story assets。

如果任一项不能通过，候选不能晋升。允许的处理是 hold、revise、reject、defer、request confirmation 或 not authorized。

## 维护触发点

以下变化要求复核本文：

- Detailed `docs/index.md` synchronization rules 更新后，复核本文的 index impact vocabulary。
- Rename、migration、split、merge、deprecation、successor 和 link-impact policy 更新。
- Duplicate concept 与 same-topic coexistence policy 更新后，复核本文的重复/共存 disposition。
- Review record template、document decision policy、rework loop examples 或 completion report template 发生实质字段或 vocabulary 变更。
- Revision/regeneration continuity policy 更新后，复核 promotion replacement、old-content handling 和 continuity evidence。
- Sidecar boundary policy 或 legacy migration guide 更新后，复核 candidate promotion 与 sidecar/legacy boundaries。
- Related docs taxonomy、link maintenance、reusable model entry points、existing-doc reuse procedure 或 network boundary / decay prevention policy 更新。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、lint/scoring automation、new global frontmatter fields 或 batch normalization。

本文不负责创建或维护 `docs/governance/index-synchronization-rules.md`、`docs/governance/rename-migration-policy.md`、`docs/governance/duplicate-and-coexistence-policy.md`、review/completion templates 或 Epic 6 batch assets；不改变主方法论、frontmatter schema、topic/path/name policy、quality gate、lifecycle policy、navigation policy 或 batch readiness policy；不执行任何实际 batch promotion。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [Topic、文件命名与路径归属策略：正式 docs 资产的位置、命名与一致性规则](./topic-path-naming-policy.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响](./rename-migration-policy.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling](./related-docs-taxonomy.md)
- [链接维护政策：正文链接、related_docs、入链影响与失效链接处置](./link-maintenance-policy.md)
- [可复用模型入口治理：稳定入口、边界、迁移提示与跨文档复用路径](./reusable-model-entry-points.md)
- [既有文档复用流程：新问题先检索、匹配、复用、再决定是否新建](./existing-doc-reuse-procedure.md)
- [网络边界与退化预防：限制链接噪声、关系漂移与维护失控](./network-boundary-and-decay-prevention.md)
- [Review Record Template](../templates/review-record-template.md)
- [Completion Report Template](../templates/completion-report-template.md)
- [文档决策政策：接受、修订、拒绝与延后记录的治理规则](./document-decision-policy.md)
- [返工循环示例：从审查发现到修订记录的闭环样例](./rework-loop-examples.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](./agent-behavior-constraints.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](../methodology/concept-document-contract.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](../methodology/governance-asset-boundary-policy.md)
