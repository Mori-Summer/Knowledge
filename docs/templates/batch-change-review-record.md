---
doc_id: template-batch-change-review-record
title: 批量变更审查记录模板：目标、样本、冲突、决策与治理契约证据
concept: batch_change_review_record
topic: templates
depth_mode: standard
created_at: '2026-06-18T09:31:06+08:00'
updated_at: '2026-06-18T09:31:06+08:00'
source_basis:
  - _bmad-output/implementation-artifacts/epic-6-context.md
  - _bmad-output/implementation-artifacts/spec-6-1-batch-governance-runbook.md
  - _bmad-output/implementation-artifacts/spec-6-2-batch-change-review-record.md
  - docs/index.md
  - docs/runbooks/batch-governance-runbook.md
  - docs/governance/batch-readiness-checklist.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/document-decision-policy.md
time_context: epic_6_story_6_2_batch_change_review_record_2026_06_18
applicability: batch_change_review_conflict_and_operator_decision_evidence
prompt_version: not_applicable
template_version: template_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/runbooks/batch-governance-runbook.md
  - docs/governance/batch-readiness-checklist.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/document-decision-policy.md
open_questions:
  - Story 6.3 建立 batch completion report template 后，是否需要把本文的 final status 与 completion summary 字段进一步对齐？
  - 后续如 Maxwell 授权实际 batch records 进入正式 docs 路径，是否需要补充单独的 record storage 与 index treatment 政策？
---

# 批量变更审查记录模板：目标、样本、冲突、决策与治理契约证据

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目的正式 template asset，用于记录一次命名批量生成、批量重构、批量规范化或相邻批量治理动作在样本审查、冲突处理和继续/停止决策中的证据。它回答的是：目标文件是什么、实际改了什么、跳过了什么、哪些文件只审查未改、使用了什么规则、样本是否通过、冲突如何定级、operator 如何决策、剩余目标是否可以继续。

本文补充 `docs/governance/batch-readiness-checklist.md` 和 `docs/runbooks/batch-governance-runbook.md`。Readiness checklist 负责写入前范围、冲突、停止条件、恢复策略和 owner decision；runbook 负责 readiness 后的执行顺序、样本 gate、失败暂停和恢复证据；本文负责把批量变更审查、冲突处理、样本失败后的规则复核和 operator decision 固化为可复查记录。

本文也补充 `docs/templates/review-record-template.md` 和 `docs/templates/completion-report-template.md`。通用 review record 仍负责普通审查分类、Hard Fail、评分或等价治理检查和 final decision；completion report 仍负责完成汇报、未解决风险和 follow-up 汇总。未来 Story 6.3 的 batch completion report template 落地前，本文不替代最终批量完成汇报，只提供 batch change review evidence。

本文不授权实际批量写入，不替代 readiness approval，不创建 actual batch records、actual review records、actual completion reports、Story 6.3 template、frontmatter schema fields、automation、validators、scanners、generators、CLI/API/UI/database/deployment/CI、package manifest、`src/` 或 `tests/`。使用本文生成具体 review record 时，必须由当前 story、Maxwell 或已批准 workflow 明确授权记录存放位置、artifact class、index treatment 和目标范围。

本文自身的 owner entry point 是 `docs/index.md` 的 `templates` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## templates` 下列出 `docs/templates/batch-change-review-record.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

`docs/index.md` 在本文 `source_basis` 中只作为 owner entry point、navigation treatment 和 index placement 的依据；本文字段权威来自当前 Story 6.2、batch readiness checklist、Story 6.1 runbook、review/completion templates 和相邻治理资产，不来自索引条目本身。

当前 `quality_status: reviewed` 表示本文在 Story 6.2 范围内按治理资产标准进行人工审查并保守入库：role、authority、target/change inventory、transformation rules、sample checks、quality/index/link/frontmatter/status checks、conflict handling、operator decision、failed-sample rule review、governance contract-point review、artifact separation、owner/index entry、相邻治理依赖和非软件边界已检查。本文不声明 `validated`，因为 Story 6.3 batch completion report template 仍未落地，且本文尚未在实际命名批量任务中验证。

## 与相邻资产的关系

| 资产 | 本文如何使用 | 不替代什么 |
| --- | --- | --- |
| `docs/governance/batch-readiness-checklist.md` | 引用 operation type、target set、allowed/excluded files、conflict summary、owner decision、stop conditions 和 recovery approach | 不重新授权批量写入，不扩大 target set |
| `docs/runbooks/batch-governance-runbook.md` | 检查批量执行是否遵守 sample gate、failure pause、remaining-target handling、artifact separation 和 non-software boundary | 不重写 runbook 执行路径 |
| `docs/templates/review-record-template.md` | 复用 review classification、Hard Fail、等价治理检查、`未验证`/`不适用` register 和 final decision discipline | 不替代普通 review record |
| `docs/templates/completion-report-template.md` | 为 completion report 提供 batch review evidence、conflict outcome 和 unresolved risk 输入 | 不替代 completion report |
| future Story 6.3 batch completion report template | 当前只记录 future dependency 和 maintenance trigger | 不提前创建 Story 6.3 模板或 actual completion report |

## 使用时机与记录状态

使用本文前，必须先能定位 readiness evidence 和 runbook execution context。若 readiness record 不存在、owner decision 不允许写入、target set 未冻结、sample plan 不明确或外部授权缺失，不能用本文绕过前置 gate。

| 使用时机 | 必须记录 |
| --- | --- |
| sample review 完成后 | sample set、checks performed、pass/fail、findings、rule changed、remaining-target decision |
| sample 失败时 | failed sample、failure category、rule review、修正规则或缩小范围、是否重跑 sample、停止状态 |
| 批量变更完成一个阶段后 | target files、changed files、skipped files、unchanged-reviewed files、quality/index/link/frontmatter/status impact |
| governance-layer 目标受影响时 | triggers、inputs、outputs、menus、waiting points、state fields、completion conditions 的 contract-point review |
| 冲突需要 operator 决策时 | severity、proposed resolution、operator decision、status、owner/follow-up |

记录状态应使用正文证据表达，不能新增全局 frontmatter 字段。建议状态包括：`draft_review`、`sample_failed`、`rule_revised`、`sample_rerun_required`、`ready_for_remaining_targets`、`stopped`、`deferred_with_reason`、`not_authorized`、`closed_for_current_scope`。

## Batch Review Classification

每份 batch change review record 必须先完成分类。分类缺失时，后续样本结论、冲突状态和 operator decision 都不能算完整。

| Field | Required evidence |
| --- | --- |
| batch operation name | 本次命名批量动作的可识别名称 |
| operation type | `batch_generation`、`batch_refactor`、`batch_frontmatter_normalization`、`batch_link_index_update`、`batch_review` 或 mixed with split notes |
| initiating source | user request、story、readiness record、review finding、runbook execution 或其他命名来源 |
| readiness evidence | readiness record 位置、owner decision、allowed files、excluded files、stop conditions |
| runbook evidence | 使用的 runbook step、sample strategy、failure/stop handling |
| record storage authorization | actual record 的批准存放位置、artifact class、index treatment；没有授权时写 `not_authorized` |
| review owner | current Agent、reviewer、Maxwell、future story 或 not_applicable |
| target set basis | explicit file list、index-derived set、topic-derived set、manually approved list 或 other reviewed basis |
| current record status | 上节允许的状态之一，并附 evidence location |

若 operation type 是 pure `batch_review` 且没有写入动作，Changed Files、Skipped Files 或 remaining-target sections 可以写 `not_applicable`，但必须明确“review-only / no write authorization”。Pure review 不能从本文推导批量写入授权。

## Target And Change Inventory

Target inventory 必须先于 changed-file 结论。不能只列“已修改”。

### Target Files

| target file | asset class | intended action | inclusion evidence | allowed by readiness? | notes |
| --- | --- | --- | --- | --- | --- |
|  | formal knowledge / methodology / governance / template / runbook / index / report / workflow evidence | create / modify / inspect / skip / defer |  | yes / no / held / not_applicable |  |

### Changed Files

| changed file | change type | changed sections or fields | reason | rule applied | quality_status changed? | review evidence status | authorization status | index/link/frontmatter impact | validation evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  | created / modified / deleted |  |  |  | yes/no + before/after or unchanged | reviewed / held / not_applicable | authorized / held / not_authorized |  |  |

### Skipped Files

| skipped file | reason | evidence | risk if skipped | owner/follow-up | status |
| --- | --- | --- | --- | --- | --- |
|  | excluded / conflict / needs Maxwell / requires future story / not applicable / failed sample category / not_authorized |  |  |  | skipped / deferred_with_reason / stopped |

### Unchanged-Reviewed Files

Use this section when a file was in or near scope and was reviewed without change. Omit only when no file was reviewed without change.

| reviewed file | why reviewed | checks performed | finding | reason no change was made | status |
| --- | --- | --- | --- | --- | --- |
|  | target member / adjacent dependency / index owner / governance dependency / sample control |  |  | no impact / already compliant / out of scope / deferred | inspected-only |

## Transformation Or Generation Rules

每条规则必须让 reviewer 能人工复现意图。不能用“统一规范化”“批量清理”代替具体规则。

| rule id | rule statement | applies to | does not change | before/after sample or expected output | source rule | stop condition |
| --- | --- | --- | --- | --- | --- | --- |
|  |  | files / sections / fields | semantics / status / source claims / links / schema fields / owner decisions |  | readiness / runbook / story / user instruction | sample fail / conflict / scope expansion / owner needed |

规则变化必须记录：

- why the rule changed;
- which sample failed or new evidence triggered the change;
- whether the target set, exclusions or remaining work changed;
- whether sample review was rerun after the change;
- whether previous sample pass is still valid or invalidated.

## Sample Checks

Sample review 是继续处理剩余目标的 gate。样本必须能代表 target set 的主要风险，尤其是 governance/template/runbook/index 目标、不同 asset class、边界文件和高风险 transformation branch。

| sampled file or candidate | why representative | checks performed | finding | result | rule changed? | remaining-target decision |
| --- | --- | --- | --- | --- | --- | --- |
|  | typical / boundary / high-risk / governance-layer / mixed-asset | frontmatter / index / links / quality / source-time / workflow contract / non-software |  | pass / fail / held / not_applicable | yes / no | proceed / rerun sample / revise rule / shrink scope / stop |

Minimum sample checks:

- target path、title/H1、frontmatter `doc_id`、`concept`、`topic` 和 index section 不互相冲突；
- `source_basis`、`related_docs` 和 `open_questions` 仍是 YAML arrays；
- `quality_status` 没有因批量处理、样本通过或文件创建自动晋升；
- changed-file Markdown links、`related_docs` targets 和 `docs/index.md` 入口可解析；changed-file 断链或缺失 `related_docs` target 是 blocker，除非 owner/story 明确授权 deferred handling，单独记录 unresolved risk 不能算通过；
- transformation/generation rule 没有改变语义、伪造来源、扩大 scope 或制造 unauthorized schema/status/version impact；
- non-software boundary 未被突破。

## Failed-Sample Rule Review

样本失败时，必须先复核 transformation/generation rule，再决定是否处理剩余目标。不能把失败样本标成后续 cleanup 后继续全量执行。

| failed sample | failure category | rule defect or target issue | required rule review | action before continuing | rerun sample evidence | status |
| --- | --- | --- | --- | --- | --- | --- |
|  | frontmatter / link / index / source-time / quality-status / semantic drift / workflow contract / non-software / scope conflict |  | revise rule / shrink target / add exclusion / ask Maxwell / stop |  | pass / fail / not_rerun | stopped / rule_revised / sample_rerun_required / ready_for_remaining_targets |

Remaining targets may continue only when one of these is true:

- the rule was revised and the relevant sample was rerun successfully;
- the failing branch was removed from the target set, exclusions were recorded, and evidence shows the remaining branch is unaffected; if the shared rule changed, representative sample review must be rerun;
- Maxwell or the authorized story explicitly approved a narrowed scope;
- the record is closed as stopped/deferred and no remaining target work continues.

## Quality, Index, Link, Frontmatter And Status Checks

Batch change review must record each check as `通过`、`未通过`、`未验证` or `不适用` with evidence. A blank row is not evidence.

| check area | required check | result | evidence / unresolved risk |
| --- | --- | --- | --- |
| quality checks | Hard Fail/equivalent blocker、review evidence、status wording、no false `validated`/`reviewed` claim |  |  |
| frontmatter checks | YAML parse、required fields、array fields、no unauthorized fields、title/concept/topic/path consistency |  |  |
| status checks | `quality_status`、lifecycle wording、review decision、BMad story status separated |  |  |
| index checks | `docs/index.md` entry updated/not_applicable/deferred with reason; no stale entry; target link resolves |  |  |
| link checks | changed-file links, `related_docs`, inbound/outbound scope when applicable |  |  |
| source/time checks | source_basis and time_context support changed claims; currentness limitations recorded |  |  |
| artifact separation | formal docs changes separated from `_bmad-output/`, candidate, review evidence and completion evidence |  |  |
| non-software boundary | no runtime code, scripts, automation, validators, scanners, package manifests, `src/`, `tests` or CI artifacts |  |  |

## Conflict Handling And Operator Decision

每个冲突都必须记录 severity、proposed resolution、operator decision 和 status。`no conflicts found` 只有在说明检查了哪些类别后才有效。

Severity levels:

| Severity | Meaning | Default handling |
| --- | --- | --- |
| critical | 继续会破坏身份、路径、质量状态、来源、workflow contract 或授权边界 | stop or ask Maxwell |
| high | 继续可能制造错误索引、断链、语义漂移、批量误应用或无法恢复的 reviewer burden | revise rule, shrink scope or hold |
| medium | 可以局部修复或延后，但必须记录 owner 和 follow-up | proceed with explicit risk only for non-governance low-impact cases; otherwise defer, hold or ask owner |
| low | 不阻塞当前范围，但需要记录 evidence 或 maintenance trigger | proceed with note |

| conflict id | category | affected files | evidence location | severity | proposed resolution | operator decision | authorization basis | status | owner/follow-up |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  | naming / path / topic / duplicate / doc_id / frontmatter / quality-status / lifecycle / source-time / links / index / workflow-contract / non-software / owner-decision |  |  | critical / high / medium / low | revise rule / shrink scope / add exclusion / repair link / update index / ask Maxwell / defer / stop | proceed / proceed_with_exclusions / revise_rule / rerun_sample / defer_with_reason / hold_for_clarification / stop / not_authorized | readiness owner decision + allowed target set, or explicit owner/story confirmation | open / resolved / blocked / deferred / not_applicable |  |

Operator decision must match the evidence and cannot expand readiness owner decision, target set, allowed files, excluded files or current-story authorization. If the decision is `proceed` or `proceed_with_exclusions`, it must cite the existing owner decision and allowed target set. If the decision is `proceed_with_exclusions`, the excluded files or branches must appear in the Skipped Files table. If the decision is `rerun_sample`, the rerun evidence must appear in Sample Checks. If any conflict remains `open` or `blocked`, final record status cannot be `closed_for_current_scope`; it must be `stopped`, `deferred_with_reason`, `held` or `not_authorized`. If the decision is `stop` or `not_authorized`, no remaining target work can continue under this record.

Any medium-or-higher conflict affecting governance-layer assets, index entries, `quality_status`, `source_basis`, `related_docs`, frontmatter identity, workflow contract or non-software boundary requires owner/story confirmation before any proceed decision.

## Governance-Layer Contract-Point Review

This section is required when target files include governance, methodology, template, runbook, quality, prompt, BMad workflow/skill, project context or index assets. It is optional only when the target set is clearly ordinary concept documents and no adjacent governance contract is affected. If any index、status、frontmatter、source_basis、related_docs、link 或 workflow-contract impact is recorded elsewhere in the review, this section is required.

Allowed result values are `pass`、`fail`、`held`、`stop_for_maxwell_confirmation` and `not_applicable_with_reason`. `not_applicable_with_reason` must prove there is no contract-point impact; a blank or generic `not_applicable` is not evidence.

| contract point | must check | result | evidence / notes |
| --- | --- | --- | --- |
| triggers | 是否改变 skill/workflow trigger、methodology entry condition、batch readiness trigger 或 index sync trigger |  |  |
| inputs | 是否改变 required user input、readiness fields、template fields、source requirements 或 owner decision inputs |  |  |
| outputs | 是否改变 allowed output artifacts、formal docs path、candidate/review/completion evidence 或 non-output promises |  |  |
| menus | 是否改变用户选择菜单、allowed values、decision labels 或 status vocabulary |  |  |
| waiting points | 是否新增、移除或绕过 ask/HALT/confirmation/checkpoint |  |  |
| state fields | 是否影响 story status、checkbox、sprint status、frontmatter status、quality_status 或 review follow-up state |  |  |
| completion conditions | 是否改变 done/accepted/blocked/deferred/not_authorized 条件、validation evidence 或 follow-up obligations |  |  |

Any uncertain governance-layer contract impact is a stop condition. Record `held` or `stop_for_maxwell_confirmation`; do not treat uncertainty as a nonblocking note.

## Artifact Separation

Batch review evidence must keep artifact classes separate:

| Artifact class | Allowed role in this record | Not allowed |
| --- | --- | --- |
| formal `docs/` asset | target, changed file, skipped file, unchanged-reviewed file or referenced rule | silent overwrite, unreviewed status promotion |
| `_bmad-output/` workflow artifact | planning context, story evidence, candidate source or temporary evidence location | direct `docs/index.md` entry or automatic promotion |
| candidate output | sample or proposed formal asset before promotion | treated as formal asset without promotion evidence |
| batch change review record | review/conflict/operator decision evidence when explicitly authorized | completion report, readiness approval, execution authorization or record storage authorization |
| future Story 6.3 batch completion report | future completion summary owner | created or assumed available by Story 6.2 |

Review record closure only closes the current review evidence scope. It is not final batch completion evidence, does not replace a completion report, and must not imply all batch work is complete unless a separate authorized completion evidence location says so.

## Copyable Batch Change Review Record Skeleton

以下 skeleton 是人工可复制模板，不是 machine-enforced schema。

```markdown
# Batch Change Review Record: <batch operation name>

## 1. Classification

| Field | Value | Evidence |
| --- | --- | --- |
| batch operation name |  |  |
| operation type |  |  |
| initiating source |  |  |
| readiness evidence |  |  |
| runbook evidence |  |  |
| record storage authorization |  | approved path/artifact class/index treatment or not_authorized |
| review owner |  |  |
| target set basis |  |  |
| current record status |  |  |

## 2. Target files

| target file | asset class | intended action | inclusion evidence | allowed by readiness? | notes |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

## 3. Changed files

| changed file | change type | changed sections or fields | reason | rule applied | quality_status changed? | review evidence status | authorization status | index/link/frontmatter impact | validation evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | yes/no + before/after or unchanged | reviewed / held / not_applicable | authorized / held / not_authorized |  |  |

## 4. Skipped files

| skipped file | reason | evidence | risk if skipped | owner/follow-up | status |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

## 5. Unchanged-reviewed files

| reviewed file | why reviewed | checks performed | finding | reason no change was made | status |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

## 6. Transformation or generation rules

| rule id | rule statement | applies to | does not change | before/after sample or expected output | source rule | stop condition |
| --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |  |

## 7. Sample checks

| sampled file or candidate | why representative | checks performed | finding | result | rule changed? | remaining-target decision |
| --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |  |

## 8. Failed-sample rule review

| failed sample | failure category | rule defect or target issue | required rule review | action before continuing | rerun sample evidence | status |
| --- | --- | --- | --- | --- | --- | --- |
| not_applicable | not_applicable or specific failure category | must state why no failed sample exists, or describe defect |  |  | must state rerun evidence or unaffected-branch evidence |  |

## 9. Quality / index / link / frontmatter / status checks

| check area | result | evidence / unresolved risk |
| --- | --- | --- |
| quality checks |  |  |
| frontmatter checks |  |  |
| status checks |  |  |
| index checks |  |  |
| link checks |  |  |
| source/time checks |  |  |
| artifact separation |  |  |
| non-software boundary |  |  |

## 10. Conflicts and operator decision

| conflict id | category | affected files | evidence location | severity | proposed resolution | operator decision | authorization basis | status | owner/follow-up |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| none | checked categories: naming/path/topic/doc_id/frontmatter/status/source/link/index/workflow/non-software/owner-decision | not_applicable | evidence for no conflicts | not_applicable | not_applicable | not_applicable | current readiness/story authorization checked | not_applicable | not_applicable |

## 11. Governance-layer contract-point review

| contract point | result | evidence / notes |
| --- | --- | --- |
| triggers | pass / fail / held / stop_for_maxwell_confirmation / not_applicable_with_reason |  |
| inputs | pass / fail / held / stop_for_maxwell_confirmation / not_applicable_with_reason |  |
| outputs | pass / fail / held / stop_for_maxwell_confirmation / not_applicable_with_reason |  |
| menus | pass / fail / held / stop_for_maxwell_confirmation / not_applicable_with_reason |  |
| waiting points | pass / fail / held / stop_for_maxwell_confirmation / not_applicable_with_reason |  |
| state fields | pass / fail / held / stop_for_maxwell_confirmation / not_applicable_with_reason |  |
| completion conditions | pass / fail / held / stop_for_maxwell_confirmation / not_applicable_with_reason |  |

## 12. Review record closure and follow-up

This section closes only this review record. It is not a batch completion report.

| Field | Value | Evidence / notes |
| --- | --- | --- |
| final record status |  | closed_for_current_scope / stopped / deferred_with_reason / not_authorized; cannot close with open/blocked conflicts |
| remaining targets |  | proceed / stopped / deferred / not_applicable; must cite owner decision and allowed target set if proceeding |
| unresolved risks |  |  |
| required follow-up |  |  |
| Story 6.3 completion report dependency | future dependency / not_applicable | `not_applicable` only when no completion summary is required for this scope |
```

## 本文自身的 Index Impact Decision Record

```text
Index Impact Decision Record

- affected file: docs/templates/batch-change-review-record.md
- target title: 批量变更审查记录模板：目标、样本、冲突、决策与治理契约证据
- target path: ./templates/batch-change-review-record.md
- target section: docs/index.md ## templates
- outcome: updated
- action taken: add template entry and update index metadata
- reason: Story 6.2 explicitly authorizes the formal batch change review record template asset
- validation result: target exists and relative link resolves
- unresolved risk: Story 6.3 batch completion report template remains a future dependency
```

## Validation Checklist And Maintenance Triggers

使用或维护本文时，至少检查：

1. Frontmatter baseline fields 存在，`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
2. `quality_status` 是 `reviewed`，没有声明 `validated`。
3. H1、frontmatter `title`、`doc_id`、`concept`、`topic`、path 和 index entry 不互相误导。
4. 正文说明 asset role、authority、scope、owner entry point、navigation treatment、index treatment、relationship to Story 6.1 runbook、readiness checklist、review template、completion template 和 future Story 6.3。
5. Target files、changed files、skipped files 和 relevant unchanged-reviewed files 都有记录位置。
6. Transformation/generation rules、sample checks、failed-sample rule review 和 remaining-target decision 字段存在。
7. Quality、index、link、frontmatter、status、source/time、artifact separation 和 non-software checks 字段存在。
8. Conflict table 覆盖 severity、proposed resolution、operator decision、authorization basis 和 status；open/blocked conflicts 不得与 closed final status 共存。
9. Governance-layer contract-point review 覆盖 triggers、inputs、outputs、menus、waiting points、state fields 和 completion conditions，并在不适用时给出 `not_applicable_with_reason`。
10. Artifact separation 明确 formal docs、`_bmad-output/`、candidate、review evidence 和 future Story 6.3 completion evidence 的边界。
11. `docs/index.md` 有 `## templates` entry，relative link 从 `docs/index.md` 可解析。
12. 未创建 actual batch record、actual review record、actual completion report、Story 6.3 template、runtime code、package manifest、source tree、software tests、build config、automation、validator、scanner、generator、CLI/API/UI/database/deployment/CI artifact；除 `docs/templates/batch-change-review-record.md` 外，未创建额外 Story 6.2 asset。

以下变化要求复核本文：

- `docs/runbooks/batch-governance-runbook.md` 的 sample review、failure/stop、artifact separation 或 recovery evidence 规则发生实质变更。
- `docs/governance/batch-readiness-checklist.md` 的 owner decision、conflict matrix、target set、stop condition 或 recovery approach 发生实质变更。
- `docs/templates/review-record-template.md` 或 `docs/templates/completion-report-template.md` 的 review/decision/completion evidence 字段发生实质变更。
- Story 6.3 batch completion report template 建立后，复核本文 final status、remaining targets、unresolved risks 和 completion dependency 字段。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、batch record generator、scanner、lint/scoring automation 或 batch completion reports。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [批量治理 Runbook：生成、重构、抽样审查与停止条件](../runbooks/batch-governance-runbook.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](../governance/batch-readiness-checklist.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](./review-record-template.md)
- [完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界](./completion-report-template.md)
- [docs/index.md 同步与导航治理规则](../governance/index-synchronization-rules.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](../governance/frontmatter-schema.md)
- [跨文档链接维护政策](../governance/link-maintenance-policy.md)
- [文档决策政策：accept、revise、regenerate、defer、reject 与 lifecycle 结果](../governance/document-decision-policy.md)
