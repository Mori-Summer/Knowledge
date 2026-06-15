---
doc_id: template-completion-report-template
title: 完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界
concept: completion_report_template
topic: templates
depth_mode: standard
created_at: '2026-05-27T14:52:11+08:00'
updated_at: '2026-05-28T15:29:36+08:00'
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
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/lifecycle-states.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
time_context: phase_4_epic_4_revision_regeneration_continuity_policy_2026_05_28
applicability: formal_docs_completion_quality_status_and_decision_reporting
prompt_version: not_applicable
template_version: template_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/templates/review-record-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/lifecycle-states.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
open_questions:
  - Story 4.1 已建立 revision/regeneration continuity policy；后续若 Continuity Record 字段变化，是否需要同步本文 specialized records summary？
  - Epic 5 related-doc taxonomy 与 link maintenance policy 已建立；后续是否需要细分 link impact、related_docs impact、owner entry 和 successor link 字段？
  - Epic 6 建立 batch completion report template 后，是否需要把本文的单项 completion report 与批量 completion report 字段对齐？
---

# 完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中正式 completion report 的 canonical template asset。它规定文档治理任务完成、停止、退回、持有、重生成、废弃或不授权时，完成汇报必须如何记录 task type、asset level、changed files、methodology/template、lifecycle status、quality gate result、validation evidence、impact scope、risk、unresolved questions、`quality_status` impact 和 final decision evidence。

本文适用于：

- 正式 `docs/` 资产的新建、升级、审查、候选晋升、模板维护、治理政策维护、索引影响、迁移/共存判断和 report entry 维护。
- `_bmad-output/` story、review output、completion evidence 或临时草稿准备进入正式 `docs/` 前的完成证据汇总。
- `docs/methodology/`、`docs/governance/`、`docs/templates/`、未来 `docs/runbooks/`、`docs/index.md` 和正式 report entry 的等价治理完成汇报。
- 需要把 review record、document decision、rework loop、quality gate、index impact、frontmatter evidence、source discipline 或 batch readiness 结果汇总给 reviewer / owner / future Agent 的场景。

本文补充而不替代主方法论、质量门禁、审查记录模板、文档决策政策、返工闭环、frontmatter schema、index synchronization、candidate promotion、rename/migration、duplicate/coexistence 和 batch readiness 规则。完成汇报只能汇总已有证据、说明缺口和标明下一步；它不能凭空完成 review、不能替代 decision policy、不能把未验证项包装成通过结论。

本文自身的 owner entry point 是 `docs/index.md` 的 `templates` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## templates` 下列出 `docs/templates/completion-report-template.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守模板资产状态。原因是本文是 Story 3.4 首版 completion-report template；Story 4.1 已建立 revision/regeneration continuity policy，`docs/governance/sidecar-boundary-policy.md` 和 `docs/governance/legacy-migration-guide.md` 已提供 sidecar / legacy migration 的相邻 evidence 入口，Epic 5 和 Epic 6 后续仍会细化 link/network governance 和 batch completion reporting。

本文与相邻资产的关系如下：

| 资产 | 本文如何使用 |
| --- | --- |
| Epic 0 foundation | 调用 Agent 行为约束、生命周期、版本治理、质量基线、导航基线和 batch readiness 的完成证据要求。 |
| Epic 1 methodology | 汇总主方法论、概念文档质量门禁、来源纪律和模板来源，不重建概念文档生成流程。 |
| Epic 2 repository governance | 汇总 frontmatter、topic/path、candidate promotion、index impact、migration 和 duplicate/coexistence 的专门记录，不替代其字段权威。 |
| Story 3.1 review-record template | 引用 review classification、Hard Fail、score/equivalent check、`未验证`/`不适用` register 和 final decision evidence。 |
| Story 3.2 document-decision policy | 使用其 final decision labels、blocking/status values、allowed wording 和 `quality_status` impact 规则。 |
| Story 3.3 rework-loop examples | 汇总 failure class、repair instruction、regeneration rationale、prior failure record 和 resubmission outcome。 |
| Story 4.1 / Story 4.2 / Story 4.3 / Epic 5 / Epic 6 | 汇总 revision/regeneration continuity、sidecar boundary 和 legacy migration assessment evidence；仅记录 link taxonomy、runbook 或 batch completion template 的 future dependency 和 maintenance trigger。 |

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record

- affected file: docs/templates/completion-report-template.md
- target title: 完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界
- target path: ./templates/completion-report-template.md
- target section: docs/index.md ## templates
- outcome: updated
- action taken: add template entry and update index metadata
- reason: Story 3.4 explicitly authorizes the canonical completion-report template asset
- validation result: target exists and relative link resolves
- unresolved risk: none for this index entry; Story 4.1 continuity, Story 4.2 sidecar and Story 4.3 legacy migration integrations are referenced, while Epic 5 link taxonomy and Epic 6 batch integrations remain open questions
```

## 职责边界与非目标

本文定义 completion report 的记录形状和最低证据要求。它不自动审查目标资产，不自动做出 accept/promote decision，也不允许用完整表格替代真实证据。

本文不实现以下范围：

- 既有资产的实际 completion reports、batch completion reports、batch review records、generated decision reports、scoring reports 或 bulk status migration reports。
- Story 3.1 review-record template 的记录字段权威。
- Story 3.2 document-decision policy 的 final decision 语义。
- Story 3.3 rework-loop examples 的 failure classes、repair instructions、regeneration examples 或 resubmission checks。
- Story 4.1 revision/regeneration continuity policy 的 Continuity Record 字段；`docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 的相邻 evidence 规则。
- Epic 5 related-doc taxonomy、link maintenance、reusable model entry points 或 existing-doc reuse procedure。
- Epic 6 batch governance runbook、batch change review record 或 batch completion report template。
- executable tooling、machine-readable schema、JSON/YAML schema、validator、lint/scoring tool、completion-report generator、decision generator、CLI/API/UI/database/deployment/CI、package manifest、`src/` 或 `tests/`。

本文不授权在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`decision_status`、`review_record_version`、`completion_report_version`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`quality_gate_version`、`promotion_policy_version`、`migration_status`、`duplicate_status`、`coexistence_status`、`successor`、`canonical_asset` 或类似字段。Completion evidence 应写在 completion report 正文、review evidence、story Dev Agent Record、目标正文中的合规说明或 future authorized records 中。

## Completion scope classification

每份 completion report 必须先完成 scope classification。分类缺失时，changed-file evidence、quality gate result、`quality_status` impact 和 final decision 都不能算完整。

| 字段 | 必填内容 | 示例或允许值 |
| --- | --- | --- |
| task type | 本次完成或停止的任务类型 | new document、upgrade、review、index-only update、candidate promotion、rename/migration、duplicate/coexistence decision、lifecycle change、methodology/governance/template maintenance、report review、batch-readiness review |
| asset level | 目标资产层级 | formal knowledge asset、methodology asset、governance asset、template asset、runbook asset、`docs/index.md`、formal report、candidate/workflow output、BMad workflow/skill asset |
| target path | 本次目标路径 | `docs/...`、candidate path、workflow output path、planned target、not_applicable |
| source/candidate path | 候选或来源位置 | conversation、`_bmad-output/...`、temporary draft、existing formal asset、not_applicable |
| document type | 文档类型或等价资产类型 | model-oriented concept document、pure-concept document、methodology asset、governance asset、template asset、runbook asset、index asset、report、not_applicable |
| depth mode | 展开密度或不适用原因 | `standard`、`deep`、`not_applicable`、`未验证` |
| lifecycle/status signals | 生命周期、质量状态和 workflow 状态 | `quality_status`、正文 lifecycle wording、BMad story status 必须分开记录 |
| methodology/template/governance sources | 实际使用的规则来源 | main methodology、quality gate、review-record template、document-decision policy、rework-loop examples、frontmatter schema、index sync、source discipline、promotion/migration/duplicate/batch readiness |
| owner instruction | Maxwell、workflow 或 story 的明确授权/限制 | 例如 `not_authorized: no executable tooling` |
| report owner | 谁负责当前完成汇报 | current Agent、reviewer、Maxwell、future story、not_applicable |

`quality_status`、lifecycle vocabulary、review decision label 和 BMad story status 是四类不同信号。不得把 `ready-for-dev`、`in-progress`、`review` 或 `done` 写成正式资产 `quality_status`，也不得因为 story 完成就宣称目标资产 reviewed、validated、promoted 或 maintained。

## Changed files evidence

Completion report 必须列出实际改变、仅检查、排除、延后或未授权的文件。不能只列“已完成”。

| file path | change type | reason | related AC/rule | validation evidence | index/link/frontmatter/source/status impact | owner/follow-up |
| --- | --- | --- | --- | --- | --- | --- |
|  | `created` / `modified` / `deleted` / `inspected-only` / `not_applicable` / `deferred` / `not_authorized` |  |  |  |  |  |

要求：

- `created`、`modified`、`deleted` 必须说明为什么变更、满足哪个 AC/rule、如何验证。
- `inspected-only` 用于合理相关但未修改的文件；必须说明检查目的和未修改理由。
- `not_applicable` 用于确实不属于当前任务或资产类别的文件。
- `deferred` 必须写明 `deferred_with_reason`、owner、风险和 re-review trigger。
- `not_authorized` 必须写明越过了哪个 boundary，以及本次没有执行的动作。
- 如果某些文件很容易被误认为在范围内，但本次没有处理，必须在 excluded/out-of-scope 行中列出。

Changed-file evidence 还必须说明是否存在 index impact、body link impact、frontmatter impact、`related_docs` impact、source/time impact、`quality_status` impact、lifecycle impact、workflow/sprint impact 或 specialized record impact。

## Methodology / template / source context

完成汇报必须列出实际使用的规则来源，不得只说“按当前规范完成”。

| source or rule | required? | result | evidence / notes |
| --- | --- | --- | --- |
| main methodology | yes/no | `通过` / `未通过` / `未验证` / `不适用` |  |
| concept quality gate or equivalent governance check | yes/no |  |  |
| review-record template | yes/no |  |  |
| document-decision policy | yes/no |  |  |
| rework-loop examples / prior failure record | yes/no |  |  |
| frontmatter schema | yes/no |  |  |
| index synchronization rules | yes/no |  |  |
| lifecycle states | yes/no |  |  |
| source discipline | yes/no |  |  |
| candidate promotion / migration / duplicate / batch readiness | yes/no |  |  |

缺失的 context 不能静默当作通过。相关但未查的内容写 `未验证`；确实不属于当前任务的内容写 `不适用`；当前 story 不授权但未来需要处理的内容写 `deferred_with_reason`；需要 owner 决策的内容写 `held`；越界内容写 `not_authorized`。

## Quality gate or equivalent governance check

Completion report 必须给出 quality gate 或等价治理检查结果。允许结果是 `通过`、`未通过`、`未验证`、`不适用`。

| 检查项 | result | evidence location | impact | owner / next action |
| --- | --- | --- | --- | --- |
| Hard Fail / equivalent blocker |  |  | `blocked` / `nonblocking` / `held` / `deferred_with_reason` / `not_authorized` |  |
| score or equivalent check evidence |  |  |  |  |
| source/time evidence |  |  |  |  |
| index/link evidence |  |  |  |  |
| frontmatter evidence |  |  |  |  |
| lifecycle/status evidence |  |  |  |  |
| workflow-contract evidence |  |  |  |  |
| non-software boundary evidence |  |  |  |  |

普通概念文档可以引用六项评分，但任意 Hard Fail 存在时，分数不能抵消 blocker。治理、方法论、模板、索引、runbook 或 report 资产必须使用等价治理检查：role/authority/scope、frontmatter baseline、source/time context、index/navigation、links/related docs、lifecycle/status、version impact、workflow contract 和 non-software boundary。

## `quality_status` and lifecycle impact

`quality_status` impact 必须反映实际 review result、quality gate 或 equivalent governance check。不得仅因结构完整、checkbox 完成、文件创建、索引存在或 BMad story status 进入 `review`/`done` 就晋升。

| 字段 | 必填内容 |
| --- | --- |
| current / before `quality_status` | 目标资产当前 frontmatter 值，或 `not_applicable` / `未验证` |
| proposed / after `quality_status` | 本次是否保留、降低、提升或保持不变；只能写当前治理允许的质量状态值，不能写 blocking/decision 状态 |
| evidence location | 支撑该影响的 review record、quality gate、equivalent check、body section、frontmatter、story evidence 或 owner decision |
| allowed wording | 当前可用的完成措辞，例如 `draft maintained with evidence gap recorded` |
| prohibited wording | 当前不能使用的 ready、accepted、validated、promoted、maintained 或等价词 |
| update applied? | yes/no/deferred_with_reason/held/not_authorized |
| blocking/status disposition | `blocked`、`nonblocking`、`held`、`deferred_with_reason`、`not_authorized` 或 `not_applicable` |
| lifecycle impact | draft/reviewed/validated/maintained_asset/deprecated/archived/reactivation 等语义影响，或 `not_applicable` |

禁止规则：

- 结构完整不能自动提升 `quality_status`。
- checklist 完成不能自动提升 `quality_status`。
- 文件创建或索引存在不能自动提升 `quality_status`。
- BMad story status 不能复制为正式资产 `quality_status`。
- unresolved Hard Fail、blocking `未验证`、critical link/index failure、source/currentness blocker、identity ambiguity、owner decision gap 或 unauthorized scope 存在时，不得使用无保留通过措辞。

## Final decision and follow-up

Completion report 必须使用 `docs/governance/document-decision-policy.md` 的 final decision labels，不得发明平行 canonical labels。

| user-facing outcome | canonical decision label options | 使用说明 |
| --- | --- | --- |
| accepted | `accept_promote` / `accepted_for_current_use` / `accepted_with_nonblocking_follow_up` | 只有无 blocker 或限制已明确为 nonblocking 时使用。 |
| returned | `targeted_revision` | 局部可修复缺口、Hard Fail 或等价 blocker 需要返工。 |
| regenerated | `regenerate` | 局部修复不足以恢复结构、身份、来源或复用完整性。 |
| rejected | `reject_duplicate_or_misplaced` | 候选或入口重复、错置或不应进入正式资产时使用。 |
| deprecated | `deprecate_or_archive` | 正式资产不再作为主要当前入口或只作历史/审计/迁移证据。 |
| held | `hold_for_clarification` | 需要 Maxwell/owner 澄清才能继续写入或判断时使用。 |
| deferred | `defer_with_reason` | 当前 story、授权、时间或 future Epic owner 不允许本轮处理时使用。 |
| not authorized | `not_authorized` | 请求跨过明确授权边界或禁止范围时使用。 |

Terminology note: `defer_with_reason` 是 canonical decision label；`deferred_with_reason` 是 blocking/status、impact 或 specialized-record status。`not_authorized` 在 decision label、blocking/status 和 changed-file evidence 中均使用 underscore 拼写。

每个 final decision 必须记录：

| 字段 | 必填内容 |
| --- | --- |
| decision label | 上表 canonical label 之一 |
| decision evidence | 支撑 decision 的 review record、quality gate/equivalent check、specialized record、owner instruction 或 blocker evidence |
| evidence location | 具体文件、frontmatter 字段、heading、section、link、index entry、story/workflow evidence |
| quality/status impact | `quality_status` 和 allowed wording 是否改变 |
| lifecycle impact | draft/reviewed/validated/maintained_asset/deprecated/archived/reactivation 等影响 |
| index/link impact | index outcome、body link、`related_docs`、successor/replacement 或 missing target 影响 |
| unresolved risks | blockers、nonblocking risks、open questions、future-story dependencies |
| required follow-up | 必须补、改、迁移、复审、确认或停止的动作 |
| owner | Maxwell、current Agent、reviewer、future story、source owner、not_applicable |
| blocking status | `none`、`blocked`、`nonblocking`、`held`、`deferred_with_reason`、`not_authorized`、`not_applicable` |
| re-review trigger | 哪个事件触发重新审查或完成汇报更新 |

## Specialized records summary

Completion report 可以汇总专门记录，但不得替代它们的 required fields。

| record | required when | completion report must summarize | boundary |
| --- | --- | --- | --- |
| Promotion Decision Record | 候选来源进入正式 `docs/`、替换或修订正式资产 | operator decision、quality/review decision、allowed status、index/link/old-content evidence | 字段权威仍属 candidate promotion checklist |
| Index Impact Decision Record | 新增、移动、重命名、改标题、改 topic、废弃、归档或导航影响判断 | outcome、action、reason、validation result、unresolved risk | outcome 权威仍属 index synchronization rules |
| Migration Decision Record | rename、move、retitle、split、merge、replacement、successor、deprecation、archive、reactivation | identity continuity、old-content handling、successor/replacement、link/index/lifecycle impact | 字段权威仍属 rename/migration policy |
| Revision / Regeneration Continuity Record | targeted revision、structural upgrade、regeneration、split、merge、deprecation 或 reference-validity impact | update mode、changed what/why、preserved/removed、old-content disposition、source/time/status impact、reference validity、unresolved risks | 字段权威仍属 revision/regeneration continuity policy |
| Duplicate / Coexistence Decision Record | duplicate、near duplicate、overlap、same-topic coexistence、reject、merge、link、narrow | chosen outcome、rejected alternatives、distinct concept/purpose/reuse context | taxonomy 和 outcome 权威仍属 duplicate/coexistence policy |
| Batch Readiness Record | 目标集由规则选出，影响多个 assets/topics/status/index entries，或需要批量治理 | target set、exclusions、stop conditions、owner decision、recovery/sampling | 批量执行和 batch completion template 属 Epic 6 |
| Source Discipline evidence | 涉及 current-practice、historical/deprecated、external fact、real-world anchor 或 unverifiable claim | claim type、source limitation、checked date、inference/open question handling | 来源纪律和质量门禁保持权威 |
| Rework Loop / Prior Failure evidence | 有 prior failure、repair instruction、regeneration rationale 或 resubmission check | failure id、class、resolution evidence、resubmission outcome、new failure risk | 字段权威仍属 rework-loop examples |

## Hard Fail / blocker wording control

未解决 Hard Fail 或等价 blocker 存在时，completion wording 必须先记录 blocker、required fix、owner 和下一步。不得用通过措辞覆盖阻塞事实。

| prohibited wording while blocker remains | replacement wording |
| --- | --- |
| ready | blocked pending required fix / held pending owner decision |
| publication-ready | not ready for publication; blocker recorded |
| published | index or lifecycle impact unresolved; publication wording not authorized |
| accepted | returned for targeted revision / held / deferred_with_reason / not_authorized |
| promoted | promotion blocked; required evidence missing |
| validated | validation incomplete or failed; evidence gap recorded |
| maintained | maintained wording not supported by current evidence |
| complete without caveat | completed scope is limited; blocker/nonblocking risk/defer/hold stated |

Unresolved blocker fields:

| 字段 | 必填内容 |
| --- | --- |
| condition | Hard Fail、equivalent blocker、blocking `未验证`、critical link/index failure、source/currentness blocker、owner decision gap、identity ambiguity、unauthorized scope |
| evidence location | 文件、frontmatter 字段、heading、section、link、index entry、source claim、story/workflow evidence |
| why it blocks | 为什么阻止 ready、accepted、validated、promoted、maintained 或等价通过措辞 |
| required fix | 必须补、改、删除、迁移、确认或停止的动作 |
| owner | Maxwell、current Agent、reviewer、future story、source owner、not_applicable |
| blocking status | `blocked`、`held`、`deferred_with_reason`、`not_authorized` |
| decision label | `targeted_revision`、`regenerate`、`hold_for_clarification`、`defer_with_reason`、`not_authorized` 等 |
| next action | 返工、重生成、澄清、延后、停止或重新审查 |
| work state | stopped、returned、held、deferred、not_authorized 或 current limited scope complete |

Nonblocking risks 必须和 blockers 分开。每项 nonblocking risk 也要有 owner、future story 或 re-review trigger。

## Impact scope

Completion report 必须说明本次任务的影响范围。每项 impact 应使用 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason`、`held`、`not_authorized`、`未验证` 或当前治理资产允许的等价状态。

| impact area | status | evidence / notes |
| --- | --- | --- |
| frontmatter |  |  |
| `docs/index.md` |  |  |
| body links |  |  |
| `related_docs` |  |  |
| source/time context |  |  |
| lifecycle/status |  |  |
| prompt/template/quality version |  |  |
| workflow/sprint state |  |  |
| specialized records |  |  |
| non-software boundary |  |  |

## Rework / prior failure summary

如果 Story 3.3 rework loop 或 prior failure record 存在，completion report 必须汇总其结果；如果不适用，也要说明原因。

| prior failure id | class | original evidence location | required fix | current result | resolution evidence | new failure introduced? |
| --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  | `resolved` / `still_blocked` / `changed_scope` / `deferred_with_reason` / `not_authorized` / `new_failure` / `不适用` |  | yes/no |

Regeneration、old-content handling、high-value reviewer notes、successor/replacement、migration、duplicate/coexistence 和 index/link impact 只在适用时汇总。本文不替代 Story 4.1 revision/regeneration continuity policy。

## Risks, unresolved questions and follow-up

Risks 和 open questions 必须按阻塞程度分开。

| item type | description | evidence location | impact | owner | required next action | re-review trigger |
| --- | --- | --- | --- | --- | --- | --- |
| blocker |  |  | `blocked` |  |  |  |
| nonblocking risk |  |  | `nonblocking` |  |  |  |
| open question |  |  | `held` / `deferred_with_reason` / `nonblocking` |  |  |  |
| future-story dependency |  |  | `deferred_with_reason` |  |  |  |
| not_authorized item |  |  | `not_authorized` |  |  |  |

不得只写“后续优化”。Follow-up 必须有 owner、下一步和 re-review trigger。

## Copyable completion report skeleton

以下 skeleton 是人工可复制模板，不是 machine-enforced schema。

```markdown
## Completion Report

### Scope

| Field | Value | Evidence / notes |
| --- | --- | --- |
| task type |  |  |
| asset level |  |  |
| target path |  |  |
| source/candidate path |  | `不适用` if there is no source/candidate asset |
| document type |  |  |
| depth mode |  |  |
| lifecycle/status signals |  | keep `quality_status`, lifecycle wording and BMad story status separate |
| methodology/template/governance sources |  | use `未验证` if related source was not checked |
| owner instruction |  |  |
| report owner |  |  |

### Changed Files

| file path | change type | reason | related AC/rule | validation evidence | index/link/frontmatter/source/status impact | owner/follow-up |
| --- | --- | --- | --- | --- | --- | --- |
| docs/example.md | modified |  |  |  |  |  |
| docs/unrelated.md | inspected-only | checked for direct contradiction |  | no change needed | not_applicable |  |
| docs/future-batch.md | deferred | Epic 6 owns batch completion template |  |  | deferred_with_reason | Epic 6 / re-review after batch template exists |
| docs/automation/schema.yaml | not_authorized | executable/schema output not authorized |  |  | not_authorized | Maxwell authorization required |

### Source / Methodology / Template Used

| source or rule | required? | result | evidence / notes |
| --- | --- | --- | --- |
| main methodology | yes/no |  |  |
| quality gate or equivalent governance check | yes/no |  |  |
| review-record template | yes/no |  |  |
| document-decision policy | yes/no |  |  |
| rework-loop examples | yes/no | `不适用` / `未验证` / result |  |
| frontmatter / index / lifecycle / source discipline | yes/no |  |  |

### Quality Gate or Equivalent Check

| check | result | evidence location | impact | owner / next action |
| --- | --- | --- | --- | --- |
| Hard Fail / equivalent blocker |  |  | blocked/nonblocking/held/deferred_with_reason/not_authorized |  |
| score or equivalent check evidence |  |  |  |  |
| source/time evidence |  |  |  |  |
| index/link evidence |  |  |  |  |
| frontmatter evidence |  |  |  |  |
| lifecycle/status evidence |  |  |  |  |
| workflow-contract evidence |  |  |  |  |
| non-software boundary evidence |  |  |  |  |

### `quality_status` Impact

| Field | Value | Evidence / notes |
| --- | --- | --- |
| current / before `quality_status` |  |  |
| proposed / after `quality_status` | unchanged / draft / reviewed / validated / maintained_asset / upgraded_v1 / other currently supported quality_status value | do not write lifecycle or blocking states here |
| update applied? | yes/no/deferred_with_reason/held/not_authorized |  |
| allowed wording |  |  |
| prohibited wording |  |  |
| blocking/status disposition | blocked/nonblocking/held/deferred_with_reason/not_authorized/not_applicable |  |
| lifecycle impact |  |  |

### Lifecycle / Index / Link / Source Impact

| impact area | status | evidence / notes |
| --- | --- | --- |
| frontmatter | updated/not_applicable/deferred_with_reason/held/not_authorized/未验证 |  |
| `docs/index.md` | updated/not_applicable/referenced_elsewhere/intentionally_excluded/deferred_with_reason/blocked_index_policy_conflict |  |
| body links |  |  |
| `related_docs` |  |  |
| source/time context |  |  |
| lifecycle/status |  |  |
| prompt/template/quality version |  |  |
| workflow/sprint state |  |  |
| specialized records |  |  |
| non-software boundary |  |  |

### Final Decision

| Field | Value | Evidence / notes |
| --- | --- | --- |
| user-facing outcome | accepted / returned / regenerated / rejected / deprecated / held / deferred / not authorized |  |
| canonical decision label | accept_promote / accepted_for_current_use / accepted_with_nonblocking_follow_up / targeted_revision / regenerate / reject_duplicate_or_misplaced / deprecate_or_archive / hold_for_clarification / defer_with_reason / not_authorized |  |
| decision evidence |  |  |
| evidence location |  |  |
| quality/status impact |  |  |
| lifecycle impact |  |  |
| index/link impact |  |  |
| unresolved risks |  |  |
| required follow-up |  |  |
| owner |  |  |
| blocking status | none/blocked/nonblocking/held/deferred_with_reason/not_authorized/not_applicable |  |
| re-review trigger |  |  |

### Specialized Records Summary

| record | required? | status | key evidence | owner/follow-up |
| --- | --- | --- | --- | --- |
| Revision / Regeneration Continuity Record | yes/no/not_applicable | complete/incomplete/blocking/nonblocking/held/deferred_with_reason/not_applicable | update mode; identity treatment; what changed; what was preserved/removed; old-content disposition; reference validity; source/time/status impact |  |
| Promotion / Index / Migration / Duplicate-Coexistence / Batch Readiness / Source Discipline | yes/no/not_applicable | complete/incomplete/blocking/nonblocking/held/deferred_with_reason/not_applicable | summarize applicable specialized record outcomes without replacing their required fields |  |

### Rework / Prior Failure Summary

| prior failure id | class | current result | resolution evidence | next action |
| --- | --- | --- | --- | --- |
| not_applicable |  | `不适用` | no prior failure record for this task |  |

### Risks / Open Questions / Follow-up

| item type | description | evidence location | impact | owner | next action | re-review trigger |
| --- | --- | --- | --- | --- | --- | --- |
| blocker |  |  | blocked |  |  |  |
| nonblocking risk |  |  | nonblocking |  |  |  |
| open question |  |  | held/deferred_with_reason/nonblocking |  |  |  |

### Validation Evidence

| validation item | result | evidence |
| --- | --- | --- |
| Markdown H1/title/heading/table formatting |  |  |
| frontmatter fields and YAML arrays |  |  |
| AC-required fields present |  |  |
| Hard Fail/blocker wording control |  |  |
| `docs/index.md` relative link |  |  |
| changed-file links and related targets |  |  |
| lifecycle/quality-status vocabulary |  |  |
| unauthorized outputs absent |  |  |

### Non-software Boundary

- runtime code created: no
- package manifest created: no
- `src/` or `tests/` tree created: no
- executable automation / validator / schema / generator created: no
- CLI/API/UI/database/deployment/CI artifact created: no
- boundary notes:

### Completion Wording

Use only wording supported by the blocker status and decision evidence.
Do not write ready, publication-ready, accepted, promoted, validated, maintained or complete without caveat while unresolved Hard Fail or equivalent blockers remain.
```

## Validation checklist and maintenance triggers

使用或维护本文时，至少检查：

1. Frontmatter baseline fields 存在，`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
2. 未新增 unauthorized global frontmatter fields。
3. H1、frontmatter `title`、`doc_id`、`concept`、`topic`、path 和 index entry 不互相误导。
4. 正文说明 asset role、authority、applicable scope、owner entry point、navigation treatment、relationship to main methodology、relationship to Story 3.1/3.2/3.3、relationship to Epic 0/1/2 和 future-story boundaries。
5. Scope classification 覆盖 task type、asset level、target path、source/candidate path、document type、depth mode、lifecycle/status signals、methodology/template/governance sources 和 report owner。
6. Changed-file evidence 区分 created、modified、deleted、inspected-only、not_applicable、deferred 和 not_authorized files。
7. Quality gate 或 equivalent governance check 使用 `通过`、`未通过`、`未验证` 或 `不适用`，并包含 Hard Fail/blocker、score/equivalent check、source/time、index/link、frontmatter、lifecycle、workflow-contract 和 non-software boundary evidence。
8. `quality_status` impact 反映实际 review result / quality gate / equivalent governance check，不因结构完整、checkbox、文件创建、index entry 或 BMad story status 自动晋升。
9. Final decision 使用 Story 3.2 labels：`accept_promote`、`accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`targeted_revision`、`regenerate`、`reject_duplicate_or_misplaced`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason`、`not_authorized`。
10. User-facing accepted、returned、regenerated、rejected、deprecated、held、deferred 和 not authorized outcomes 已映射到 canonical labels，且没有新增平行 canonical vocabulary。
11. Promotion、Index Impact、Migration、Revision / Regeneration Continuity、Duplicate/Coexistence、Batch Readiness、Source Discipline 和 Rework Loop records 被汇总但未被替代。
12. Unresolved Hard Fail / blocker wording control 明确禁止 ready、publication-ready、published、accepted、promoted、validated、maintained 和 complete-without-caveat wording。
13. Copyable skeleton 覆盖 Scope、Changed Files、Source / Methodology / Template Used、Quality Gate or Equivalent Check、`quality_status` Impact、Lifecycle / Index / Link / Source Impact、Final Decision、Rework / Prior Failure Summary、Risks / Open Questions、Follow-up、Validation Evidence、Non-software Boundary 和 Completion Wording。
14. `docs/index.md` 有 `## templates` entry，relative link 从 `docs/index.md` 可解析。
15. Changed-file links、body links 和 `related_docs` targets 存在；planned targets 只出现在 `open_questions` 或 future-story dependency。
16. Lifecycle/quality-status vocabulary 与当前治理规则兼容，不伪造 review、validation、migration、promotion、lifecycle 或 batch evidence。
17. 未创建 runtime code、package manifest、source tree、software tests、build config、automation、CLI/API/UI/database/deployment/CI、completion-report generator、decision generator、validation script、batch completion template、Story 4.2/4.3 asset、Epic 5 asset、Epic 6 asset 或 actual completion report。

以下变化要求复核本文：

- Story 3.1 review-record template、Story 3.2 document-decision policy 或 Story 3.3 rework-loop examples 的字段、label、blocking/status values 或 resubmission vocabulary 发生实质变更。
- Story 4.1 revision/regeneration continuity policy 的 Continuity Record 字段变化。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 的相邻 evidence 规则发生实质变更。
- Epic 5 related-doc taxonomy、link maintenance policy、reusable model entry point、existing-doc reuse procedure 或后续 network governance 更新。
- Epic 6 建立 batch governance runbook、batch change review record 或 batch completion report template。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、completion-report generator、decision generator、review-record generator、lint/scoring automation、batch completion reports 或批量状态迁移。
- `quality_status`、lifecycle、frontmatter schema、promotion、index、migration、duplicate/coexistence 或 batch readiness vocabulary 发生实质治理变更。

维护触发不等于自动执行批量更新。任何批量改写、批量索引、批量状态调整、批量 link repair、自动化工具、schema 或 generator 都必须先满足相应 story、batch readiness 和 Maxwell 授权。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](./review-record-template.md)
- [文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements](../governance/document-decision-policy.md)
- [失败案例与返工闭环：failure categories、repair instructions、regeneration rationale 与 resubmission checks](../governance/rework-loop-examples.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](../governance/lifecycle-states.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [治理资产导航、索引与入口归属政策](../governance/governance-asset-navigation-policy.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](../governance/index-synchronization-rules.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](../governance/frontmatter-schema.md)
- [候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁](../governance/candidate-promotion-checklist.md)
- [重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响](../governance/rename-migration-policy.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](../governance/duplicate-and-coexistence-policy.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](../governance/revision-regeneration-continuity-policy.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](../governance/batch-readiness-checklist.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
