---
doc_id: template-review-record-template
title: 审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录
concept: review_record_template
topic: templates
depth_mode: standard
created_at: '2026-05-27T09:29:12+08:00'
updated_at: '2026-05-27T14:52:11+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/3-1-review-record-template.md
  - _bmad-output/implementation-artifacts/3-2-document-decision-policy.md
  - _bmad-output/implementation-artifacts/3-3-rework-loop-examples.md
  - _bmad-output/implementation-artifacts/3-4-completion-report-template.md
  - docs/index.md
  - docs/templates/completion-report-template.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
time_context: phase_4_epic_3_completion_report_template_2026_05_27
applicability: formal_docs_review_record_evidence_template
prompt_version: not_applicable
template_version: template_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/templates/completion-report-template.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
open_questions:
  - 如果 completion-report template 后续调整 final decision、follow-up 或 blocker 汇总字段，是否需要同步本文的 final decision section？
  - Epic 5 related docs taxonomy 与 link maintenance policy 已建立；后续是否需要细分 related-doc、supporting link、successor link 和 owner entry 的证据字段？
  - Epic 6 建立 batch governance runbook 和 batch review record 后，是否需要扩展本文的 batch-readiness evidence 部分？
---

# 审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中正式审查记录的 canonical template asset。它定义 reviewer 在审查正式 `docs/` 资产、候选资产、方法论资产、治理资产、模板资产、索引更新、正式报告、迁移/共存决策或受控 workflow evidence 时，必须留下哪些可复查证据。

本文适用于：

- 正式 `docs/` 资产的新建、升级、审查、修订、迁移、废弃、归档、重复/共存判断和索引影响判断。
- `_bmad-output/`、对话输出、review output、completion record 或临时草稿准备晋升为正式资产前的候选审查。
- `docs/methodology/`、`docs/governance/`、`docs/templates/`、未来 `docs/runbooks/`、`docs/index.md` 和正式 report entry 的等价治理检查。
- 需要记录 Hard Fail、评分证据、未验证项、不适用项、专门 decision record 引用和最终 decision 的人工审查场景。

本文补充而不替代主方法论、质量门禁、来源纪律、生命周期政策、frontmatter schema、导航/index 政策、候选晋升、迁移、重复/共存和批量 readiness 规则。主方法论仍负责概念文档执行入口；质量门禁仍负责概念文档 Hard Fail 与六项评分；Epic 2 治理资产仍负责各自的专门 decision record；本文只提供统一记录形状和证据要求。

本文自身的 owner entry point 是 `docs/index.md` 的 `templates` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## templates` 下列出 `docs/templates/review-record-template.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守模板资产状态。原因是本文是 Epic 3 首个模板资产；Story 3.2 已建立 document decision policy，Story 3.3 已建立 rework-loop examples，Story 3.4 已建立 completion-report template，Epic 5 和 Epic 6 后续仍会细化链接分类和批量记录。

本文与各阶段资产的关系如下：

| 来源阶段 | 本文如何使用 |
| --- | --- |
| Epic 0 foundation | 调用 Agent 行为约束、生命周期、导航、质量基线和 batch readiness 的证据要求。 |
| Epic 1 methodology | 调用主方法论、质量门禁和来源纪律，不重建概念文档生成流程。 |
| Epic 2 repository governance | 调用 frontmatter、topic/path、candidate promotion、index sync、migration 和 duplicate/coexistence 记录，不替代其专门字段。 |
| Epic 3 companion assets | Final decision 语义由 Story 3.2 document decision policy 定义；failure categories、repair paths、regeneration rationale 和 resubmission checks 由 Story 3.3 rework-loop examples 定义；completion 汇报由 Story 3.4 completion-report template 汇总；本文仍只提供 review record 记录形状。 |

本文自身的 index impact decision record 是：

```text
Index Impact Decision Record

- affected file: docs/templates/review-record-template.md
- target title: 审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录
- target path: ./templates/review-record-template.md
- target section: docs/index.md ## templates
- outcome: updated
- action taken: add template entry and update index metadata
- reason: Story 3.1 explicitly authorizes the first formal template asset under docs/templates/
- validation result: target exists and relative link resolves
- unresolved risk: none for this index entry; future template index ownership remains an open question
```

## 职责边界与非目标

本文定义 review record 的记录形状和证据要求。它不自动做出审查决定，也不允许 reviewer 用空表格代替实际证据。

本文不实现以下范围：

- Story 3.2 document decision policy：通过、定向修订、重生成、拒绝、废弃、持有等完整决策语义已由 `docs/governance/document-decision-policy.md` 定义。
- Story 3.3 rework-loop examples：失败类别、返工路径和示例库已由 `docs/governance/rework-loop-examples.md` 定义。
- `docs/templates/completion-report-template.md`：最终完成汇报模板和项目级完成摘要。
- Epic 5 related-doc taxonomy、link maintenance 或 reusable model entry point。
- Epic 6 batch runbook、batch review record 或 batch completion report。
- 对既有文档生成实际 review records、批量审查记录、评分报告或自动化输出。
- executable tooling、machine-readable schema、JSON/YAML schema、lint/scoring tool、review-record generator、CLI/API/UI/database/deployment/CI、package manifest、`src/` 或 `tests/`。

本文不授权在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`review_record_version`、`decision_status`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`quality_gate_version`、`promotion_policy_version`、`migration_status`、`duplicate_status`、`coexistence_status`、`successor`、`canonical_asset` 或类似字段。审查、决策、导航、迁移和共存证据应写在 review evidence、completion evidence、正文记录、story Dev Agent Record、Epic 3 review/completion templates 或 future runbook records 中。

## 使用前分类

正式审查记录必须先完成分类。分类缺失时，后续 Hard Fail、评分、等价治理检查和 final decision 都不能算完整。

| 字段 | 必填内容 | 允许值或示例 |
| --- | --- | --- |
| task type | 本次审查任务类型 | new document、upgrade、review、index-only update、candidate promotion、rename/migration、duplicate/coexistence decision、lifecycle change、methodology/governance/template maintenance、report review、batch-readiness review |
| asset level | 目标资产层级 | formal knowledge asset、methodology asset、governance asset、template asset、runbook asset、`docs/index.md`、formal report、candidate/workflow output、BMad workflow/skill asset |
| target path | 被审查或拟写入的目标路径 | `docs/...`、candidate path、workflow output path 或 planned target |
| candidate/source path if any | 候选或来源位置 | conversation、`_bmad-output/...`、temporary draft、existing formal asset、not_applicable |
| target status | 目标当前状态 | formal `docs/`、candidate、workflow output、planned asset、unknown/unverified |
| document type | 文档类型 | model-oriented concept document、pure-concept document、methodology asset、governance asset、template asset、runbook asset、index asset、report、not_applicable |
| depth mode | 展开密度或不适用原因 | `standard`、`deep`、`not_applicable`、`unverified`，并说明 reason |
| lifecycle/status | 当前质量、生命周期和 workflow 状态 | 当前 `quality_status`；正文或证据中的 lifecycle vocabulary；相关 BMad story status；三者必须分开 |
| applicable methodology source | 使用的规则来源 | main methodology、quality gate、source discipline、frontmatter schema、topic/path policy、promotion checklist、index sync、rename/migration、duplicate/coexistence、lifecycle、navigation、batch readiness、workflow/skill source |
| decision | 本记录的最终 decision 字段 | 使用 `docs/governance/document-decision-policy.md` 中允许的 final decision label，或在专门 records 中使用其各自允许值 |

`quality_status`、lifecycle vocabulary 和 BMad story status 是三类不同信号。不得把 `ready-for-dev`、`in-progress`、`review` 或 `done` 写成正式资产 frontmatter 状态，也不得因为 story 完成就宣称目标资产 reviewed、validated、maintained 或 promoted。

## Review scope and source context

审查记录必须说明实际看过什么、用过什么规则、排除了什么，以及排除是否会影响结论。

| 字段 | 必填内容 |
| --- | --- |
| files reviewed | 被审查的目标文件、候选来源、索引文件、相邻治理/方法论资产 |
| changed files | 本次新增、修改、删除或检查的文件 |
| source_basis checked | frontmatter `source_basis`、story context、planning artifacts、正式 docs 或外部来源是否足以支撑正文 |
| methodology/governance rules used | 本次调用的主方法论、质量门禁、frontmatter、index、promotion、migration、duplicate、lifecycle、batch 等规则 |
| user instruction / owner decision | Maxwell 或当前 workflow 的明确指令、约束或授权边界 |
| excluded files | 合理可能被误认为在范围内、但本次未审查的文件 |
| exclusion reason | `not_applicable`、`not_authorized`、`deferred_with_reason`、`unverified` 或其他明确理由 |
| impact of exclusion | 排除是否阻塞 decision、只形成后续风险，或完全不影响当前结论 |

如果某项 context 缺失，不得静默视为通过。必须写成 `未验证`、`不适用`、`deferred_with_reason`、`hold_for_clarification` 或其他当前政策允许的明确状态，并说明 impact、owner 和下一步。

## Hard Fail / blocker evidence

Hard Fail 或等价阻塞项必须以表格记录。每一项都要能让后续 reviewer 找到证据位置、理解为什么阻塞，并知道如何修复。

| 字段 | 必填内容 |
| --- | --- |
| result | `通过`、`未通过`、`未验证`、`不适用`、`blocked`、`nonblocking` 或等价中文标签 |
| condition | 命中的 Hard Fail、等价治理失败或阻塞条件 |
| evidence location | 具体路径、frontmatter 字段、heading、section、link、index entry、source claim、example、story/workflow evidence |
| reason / why it blocks | 为什么阻塞 ready、accepted、validated、promoted、maintained 或等价通过措辞 |
| required fix | 必须补什么、改什么、删除什么、迁移什么或请求谁确认 |
| blocking status | `blocked`、`nonblocking`、`held`、`deferred_with_reason`、`not_authorized` |
| owner | Maxwell、当前 Agent、future story、reviewer、source owner 或 not_applicable |
| related AC/rule | 对应验收标准、质量门禁、治理规则、story task 或 workflow contract |

Hard Fail 类别示例包括：仓库集成、required frontmatter、文档结构、边界清晰度、验证/迁移、来源/时间语境、一致性、link/index、lifecycle/status、duplicate/coexistence、workflow-contract failure 和 non-software boundary failure。

任何 Hard Fail 或等价 blocker 存在时，记录不得使用 ready、accepted、validated、promoted、`upgraded_v1`、`maintained_asset` 或等价通过措辞，除非该 blocker 已解决或被明确持有且不影响当前适用范围。分数不能抵消 Hard Fail。

## Quality score evidence

普通概念文档可以使用质量门禁六项评分。评分只能在 Hard Fail 检查之后进行；如果存在 Hard Fail，评分只能作为返工参考，不能作为通过依据。

| 维度 | 分数 | supporting evidence | source category | notes |
| --- | --- | --- | --- | --- |
| 问题定义与边界 | `0` / `1` / `2` / `未验证` / `不适用` | body heading、boundary section、frontmatter、examples、links 或 explicit gap | body / frontmatter / links / examples / source basis / index-navigation evidence |  |
| 结构与因果 / 判别框架 | `0` / `1` / `2` / `未验证` / `不适用` | 机制链、分面框架、判别表、section evidence 或 explicit gap | body / examples / source basis |  |
| Tradeoff / 失败模式 / 误读纠偏 | `0` / `1` / `2` / `未验证` / `不适用` | tradeoff、failure mode、counterexample、misreading correction | body / examples / source basis |  |
| 真实世界锚点 / 当前实践 | `0` / `1` / `2` / `未验证` / `不适用` | concrete anchor、currentness source、核对日期、source limitation | source basis / body / external source note |  |
| 验证 / 迁移 | `0` / `1` / `2` / `未验证` / `不适用` | self-test、diagnostic question、transfer entry、downstream model | body / examples / links |  |
| 元数据 / 仓库纪律 | `0` / `1` / `2` / `未验证` / `不适用` | frontmatter、path、index、related_docs、links、status evidence | frontmatter / index-navigation evidence / links |  |

每个分数都必须有 supporting evidence。证据可以来自正文、frontmatter、链接、例子、source basis、index/navigation evidence，或显式 `未验证`。不得只写分数。

## Equivalent governance check

方法论、治理、模板、索引、runbook、report、workflow 或 skill 资产不是普通概念文档，不应被强行套入六项概念文档评分。它们需要等价治理检查。

| 检查项 | 期望证据 | result | evidence / notes |
| --- | --- | --- | --- |
| role / authority / scope | 资产角色、权威范围、适用对象和不替代关系清楚 |  |  |
| frontmatter baseline | baseline fields 存在，数组字段为 YAML arrays，没有 unauthorized fields |  |  |
| source / time context | `source_basis`、`time_context` 和正文主张一致；current-practice claim 可核查或标明限制 |  |  |
| index / navigation | owner entry point、navigation treatment、index impact 和相对链接证据明确 |  |  |
| links / related_docs | `related_docs` 和正文链接目标存在；缺失目标进入 open questions 或 future dependency |  |  |
| lifecycle / status | `quality_status`、lifecycle vocabulary 和 workflow status 分开，不伪造 reviewed/validated/maintained |  |  |
| version impact | prompt/template/quality/version 影响明确；无未授权版本 bump |  |  |
| workflow contract | story tasks、Dev Agent Record、File List、Change Log、sprint status 或 skill contract 未被破坏 |  |  |
| non-software boundary | 未新增 runtime code、package manifest、software tests、automation、CLI/API/UI/database/deployment/CI |  |  |
| future-story boundary | 未提前实现未授权或尚未到期的 Epic 5、Epic 6 或其他未来资产；对已落地 Story 3.2/3.3/3.4 只检查是否越过其资产边界 |  |  |

非概念资产的 decision 必须基于这些等价证据，而不是用概念文档分数证明通过。

## `未验证` / `不适用` register

`未验证` 和 `不适用` 必须分开记录。

- `不适用` 表示该项确实不属于当前任务、资产类别、文档类型或授权范围；必须说明为什么 out of scope。
- `未验证` 表示该项理论上相关，但当前证据缺失、范围未授权、目标不存在、来源不可核查或 owner decision 未定；必须说明缺什么证据、是否阻塞、谁处理。

| item | expected evidence | current result | reason | impact | blocking status | owner / future story | required next action |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  | `未验证` / `不适用` |  |  | `blocked` / `nonblocking` / `held` / `deferred_with_reason` / `not_authorized` |  |  |

必须进入本 register 的常见情况包括：

- source/current-practice claim 无法验证，需要回到来源纪律和质量门禁处理。
- missing links、missing target assets、planned related docs 或 fragment anchor 不存在。
- 缺少 methodology context、governance rule、quality gate evidence 或 workflow source。
- lifecycle/status 不明，或 `quality_status` 是否可升级不明。
- depth mode、document type、asset level、owner decision 或 index impact 不清。
- 当前 story 不授权处理，但后续 story、Maxwell 或 batch readiness 可能需要接手。

## Specialized decision records referenced

本文可以引用专门 decision records，但不得替代它们的 required fields。

| record | 何时引用 | 边界 |
| --- | --- | --- |
| Promotion Decision Record | 候选来源准备进入正式 `docs/`，或替换/修订正式资产时 | 由 candidate promotion checklist 保持字段权威 |
| Index Impact Decision Record | 新增、移动、重命名、改标题、改 topic、废弃、归档或导航影响判断时 | 由 index synchronization rules 保持 outcome 和字段权威 |
| Migration Decision Record | rename、move、retitle、split、merge、replacement、successor、deprecation、archive、reactivation 时 | 由 rename/migration policy 保持字段权威 |
| Duplicate / Coexistence Decision Record | duplicate、near duplicate、overlap、adjacent、narrower、same-topic coexistence 或 reject 判断时 | 由 duplicate/coexistence policy 保持 taxonomy 和 outcome 权威 |
| Batch Readiness Record | 目标集由规则选出、影响多个资产/topics/status/index entries，或需要批量治理时 | 由 batch readiness checklist 和未来 Epic 6 资产保持权威 |
| Source discipline evidence | 涉及当前实践、历史路径、外部事实、不可验证声明或真实世界锚点时 | 由 source discipline policy 和 quality gate 保持权威 |

一次 review record 可以引用多种专门记录。引用时必须说明：记录是否已存在、是否完整、是否仍有 blocker，以及本 review decision 是否依赖该记录。

## Final decision and follow-up

本节提供 final decision 的记录位置。完整 accept/promote、targeted revision、regenerate、reject、deprecate/archive、hold、defer 和 not authorized 语义由 `docs/governance/document-decision-policy.md` 定义。Reviewer 必须使用该政策允许的 final decision label；Promotion、Index Impact、Migration、Duplicate/Coexistence 和 Batch Readiness 等专门 records 仍使用各自资产定义的 allowed values。

| 字段 | 必填内容 |
| --- | --- |
| review decision label | 使用 `docs/governance/document-decision-policy.md` 允许的 final decision label |
| decision rationale | 依据哪些 Hard Fail、score、governance check、specialized record 和 unresolved risk |
| allowed status wording | 当前允许或禁止使用的 ready、accepted、reviewed、validated、promoted、maintained、deprecated 等措辞 |
| `quality_status` impact | 是否允许保留、降低、提升或不修改 `quality_status`；不得伪造证据 |
| lifecycle impact | draft/reviewed/validated/maintained/deprecated/archived/reactivation 等语义影响，若无正式字段则写 evidence location |
| index impact | `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason`、`blocked_index_policy_conflict` |
| link / related-doc impact | 正文链接、`related_docs`、changed-file links、planned targets 和 unresolved link risks |
| version impact | prompt/template/quality/version 影响、是否触发版本治理或 future migration |
| approved deviations | 已批准偏离；必须说明批准来源和适用范围 |
| unresolved risks | nonblocking risks、blocking risks、open questions、future-story dependencies |
| follow-up owner | Maxwell、当前 Agent、reviewer、Story 3.2/3.3/3.4、Epic 5、Epic 6 或其他 owner |

不得把 review/decision metadata 作为 unauthorized frontmatter fields 写入目标资产。需要持久化的 review evidence 应留在 review record、completion evidence、目标正文的合规说明、story Dev Agent Record、Epic 3 review/completion evidence 或 future Epic 6 records 中。

## Copyable review record skeleton

以下 skeleton 是人工可复制模板，不是 machine-enforced schema。

```markdown
# Review Record: <target title or path>

## 1. Review classification

| Field | Value | Evidence / reason |
| --- | --- | --- |
| task type |  |  |
| asset level |  |  |
| target path |  |  |
| candidate/source path if any |  |  |
| target status |  | formal docs / candidate / workflow output / planned / unverified |
| document type |  | model-oriented concept / pure concept / methodology / governance / template / runbook / index / report / not_applicable |
| depth mode |  | standard / deep / not_applicable / unverified |
| lifecycle/status |  | quality_status / lifecycle evidence / BMad story status separated |
| applicable methodology source |  |  |
| decision |  | placeholder until final section |

## 2. Review scope and source context

| Field | Evidence |
| --- | --- |
| files reviewed |  |
| changed files |  |
| source_basis checked |  |
| methodology/governance rules used |  |
| user instruction / owner decision |  |
| excluded files |  |
| exclusion reason and impact |  |

## 3. Hard Fail / blocker evidence

| result | condition | evidence location | reason / why it blocks | required fix | blocking status | owner | related AC/rule |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |  |  |

## 4. Quality score evidence for concept documents

| Dimension | Score | Supporting evidence | Source category | Notes |
| --- | --- | --- | --- | --- |
| 问题定义与边界 |  |  |  |  |
| 结构与因果 / 判别框架 |  |  |  |  |
| Tradeoff / 失败模式 / 误读纠偏 |  |  |  |  |
| 真实世界锚点 / 当前实践 |  |  |  |  |
| 验证 / 迁移 |  |  |  |  |
| 元数据 / 仓库纪律 |  |  |  |  |

## 5. Equivalent governance check for non-concept assets

| Check | Result | Evidence / notes |
| --- | --- | --- |
| role / authority / scope |  |  |
| frontmatter baseline |  |  |
| source / time context |  |  |
| index / navigation |  |  |
| links / related_docs |  |  |
| lifecycle / status |  |  |
| version impact |  |  |
| workflow contract |  |  |
| non-software boundary |  |  |
| future-story boundary |  |  |

## 6. `未验证` / `不适用` register

| item | expected evidence | current result | reason | impact | blocking status | owner / future story | required next action |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |  |  |

## 7. Specialized decision records referenced

| Record | Exists / needed | Evidence location | Completeness | Impact on decision |
| --- | --- | --- | --- | --- |
| Promotion Decision Record |  |  |  |  |
| Index Impact Decision Record |  |  |  |  |
| Migration Decision Record |  |  |  |  |
| Duplicate / Coexistence Decision Record |  |  |  |  |
| Batch Readiness Record |  |  |  |  |
| Source discipline evidence |  |  |  |  |

## 8. Final decision and follow-up

| Field | Value | Evidence / notes |
| --- | --- | --- |
| review decision label |  |  |
| decision rationale |  |  |
| allowed status wording |  |  |
| quality_status impact |  |  |
| lifecycle impact |  |  |
| index impact |  |  |
| link / related-doc impact |  |  |
| version impact |  |  |
| approved deviations |  |  |
| unresolved risks |  |  |
| follow-up owner |  |  |
```

## Validation checklist and maintenance triggers

使用或维护本模板时，至少检查：

1. Frontmatter baseline fields 存在，`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
2. 未新增 unauthorized global frontmatter fields。
3. H1、frontmatter `title`、`doc_id`、`concept`、`topic`、path 和 index entry 不互相误导。
4. 正文说明 asset role、authority、applicable scope、owner entry point、navigation treatment、relationship to main methodology 和 relationship to Epic 0/1/2/3。
5. 使用前分类覆盖 task type、asset level、target path、document type、depth mode、lifecycle/status、methodology source 和 decision。
6. Hard Fail 表覆盖 condition、evidence location、reason、required fix、blocking status、owner 和 related rule。
7. 概念文档评分要求每个分数都有 body、frontmatter、links、examples、source basis 或 index/navigation evidence。
8. 非概念资产使用等价治理检查，不被强行套入六项概念文档分数。
9. `未验证` 和 `不适用` register 区分 out-of-scope 与 missing evidence，并说明 impact、owner 和 next action。
10. Final decision 字段存在，并引用 `docs/governance/document-decision-policy.md` 的允许 label 和状态措辞。
11. Promotion、Index Impact、Migration、Duplicate/Coexistence、Batch Readiness 和 Source Discipline records 可被引用，但没有被本文替代。
12. `docs/index.md` 有 `## templates` entry，relative link 从 `docs/index.md` 可解析。
13. Changed-file links、body links 和 `related_docs` targets 存在；planned targets 只出现在 `open_questions` 或 future-story dependency。
14. Lifecycle/quality-status vocabulary 与当前治理规则兼容，不伪造 review、validation、migration、promotion 或 lifecycle evidence。
15. 未创建 runtime code、package manifest、source tree、software tests、build config、automation、CLI/API/UI/database/deployment/CI、review-record generator、validation script、Epic 5 asset、Epic 6 asset 或额外 Epic 3 asset。

以下变化要求复核本文：

- Story 3.2 document decision policy 更新 final decision label、allowed status wording 或 specialized-record integration。
- `docs/governance/rework-loop-examples.md` 的 failure classes、repair instruction、regeneration rationale 或 resubmission vocabulary 发生实质变更。
- `docs/templates/completion-report-template.md` 的 completion summary、decision、follow-up 或 blocker 字段发生实质变更。
- Epic 5 related docs taxonomy、link maintenance policy、reusable model entry point 或后续 reuse/network governance 更新。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、review-record generator、lint/scoring automation 或批量审查记录。

维护触发不等于自动执行批量更新。任何批量改写、批量索引、批量状态调整或自动化工具都必须先满足相应 story、batch readiness 和 Maxwell 授权。
