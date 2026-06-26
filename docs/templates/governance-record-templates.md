---
doc_id: template-governance-record-templates
title: 文档治理记录模板：审查、批量审查、完成汇报与批量完成汇报
concept: governance_record_templates
topic: templates
depth_mode: deep
created_at: '2026-06-22T09:58:00+08:00'
updated_at: '2026-06-22T09:58:00+08:00'
source_basis:
  - templates_folder_consolidation_2026_06_22
  - review_record_template_merged_2026_06_22
  - batch_change_review_record_merged_2026_06_22
  - completion_report_template_merged_2026_06_22
  - batch_completion_report_template_merged_2026_06_22
  - docs/governance/docs-asset-governance.md
  - docs/governance/docs-change-governance.md
time_context: templates_consolidation_2026_06_22
applicability: review_record_batch_review_completion_report_and_batch_completion_report_templates
prompt_version: not_applicable
template_version: template_asset_v2_consolidated
quality_status: consolidated_v1
related_docs:
  - docs/governance/docs-asset-governance.md
  - docs/governance/docs-change-governance.md
  - docs/runbooks/batch-governance-runbook.md
open_questions:
  - 是否需要把这些 Markdown skeleton 拆成机器可填写的 YAML/JSON schema？
  - 实际批量治理任务使用后，是否需要增加更严格的 sample gate 和 recovery evidence 字段？
---

# 文档治理记录模板：审查、批量审查、完成汇报与批量完成汇报

## 1. 规范定位

本文是 `docs/templates/` 的唯一当前入口，提供四类人工可复制记录模板：

- Review Record：单个资产审查记录
- Batch Change Review Record：批量变更审查记录
- Completion Report：单个任务完成汇报
- Batch Completion Report：批量任务完成汇报

具体治理规则由 [正式 docs 资产治理规范](../governance/docs-asset-governance.md) 和 [文档治理执行规范](../governance/docs-change-governance.md) 定义。本文只提供记录形状，不授权写入、晋升、删除、批量修改或状态提升。

## 2. 使用选择

| 场景 | 使用模板 |
|---|---|
| 单个文档审查、候选晋升审查、索引影响审查 | Review Record |
| 多文件生成、重构、删除、合并、链接修复后的抽样审查 | Batch Change Review Record |
| 单个任务结束、停止、退回、持有、完成汇报 | Completion Report |
| 批量任务结束、恢复、停止或批量完成汇总 | Batch Completion Report |

如果任务同时包含单项和批量部分，优先用 Batch Change Review Record / Batch Completion Report 汇总批量范围，再为高风险单项补 Review Record。

## 3. Review Record Skeleton

```markdown
# Review Record: <target title or path>

## 1. Review classification

| Field | Value | Evidence / reason |
|---|---|---|
| task type |  | new document / upgrade / review / index-only update / candidate promotion / rename-migration / duplicate-coexistence / lifecycle change / methodology-governance-template maintenance / report review / batch-readiness review |
| asset level |  | concept / methodology / governance / template / runbook / index / report / candidate |
| target path |  |  |
| candidate or source path |  |  |
| document type |  | concept / methodology / governance / template / runbook / index / report / not_applicable |
| current status |  | quality_status, lifecycle evidence, workflow/story status kept separate |
| applicable rules |  | methodology, quality gate, source discipline, asset governance, change governance |
| provisional decision |  | leave blank until final section |

## 2. Scope and source context

| Field | Evidence |
|---|---|
| files reviewed |  |
| changed files |  |
| source_basis checked |  |
| user instruction / owner decision |  |
| excluded files |  |
| exclusion reason and impact |  |

## 3. Hard Fail / blocker evidence

| result | condition | evidence location | why it blocks | required fix | blocking status | owner |
|---|---|---|---|---|---|---|
|  |  |  |  |  | blocked / nonblocking / held / deferred / not_authorized |  |

## 4. Quality or equivalent governance checks

| Check | Result | Evidence |
|---|---|---|
| problem definition / boundary |  |  |
| structure / causality / decision frame |  |  |
| tradeoff / failure modes |  |  |
| source / time context |  |  |
| validation / transfer |  |  |
| metadata / repository discipline |  |  |
| non-concept equivalent checks |  | role, authority, frontmatter, index, links, lifecycle, workflow, non-software boundary |

## 5. Unverified / not applicable register

| item | expected evidence | result | reason | impact | owner / next action |
|---|---|---|---|---|---|
|  |  | unverified / not_applicable |  | blocking / nonblocking |  |

## 6. Specialized records referenced

| Record | Needed? | Evidence location | Completeness | Impact |
|---|---|---|---|---|
| Promotion Decision Record |  |  |  |  |
| Index Impact Decision Record |  |  |  |  |
| Migration Decision Record |  |  |  |  |
| Duplicate / Coexistence Decision Record |  |  |  |  |
| Batch Readiness Record |  |  |  |  |

## 7. Final decision and follow-up

| Field | Value | Evidence |
|---|---|---|
| decision label |  | use docs-change-governance allowed labels |
| rationale |  |  |
| quality_status impact |  |  |
| lifecycle impact |  |  |
| index impact |  |  |
| link / related_docs impact |  |  |
| unresolved risks |  |  |
| follow-up owner |  |  |
```

## 4. Batch Change Review Record Skeleton

```markdown
# Batch Change Review Record: <batch operation name>

## 1. Classification

| Field | Value |
|---|---|
| operation type | batch_generation / batch_refactor / batch_delete_merge / batch_link_index_update |
| target set source | explicit list / index-derived / topic-derived / owner-approved list |
| owner decision | proceed / proceed_with_exclusions / clarify_before_write / defer / stop |
| readiness evidence |  |
| excluded files |  |
| recovery approach |  |

## 2. Target files

| path | expected action | reason |
|---|---|---|
|  |  |  |

## 3. Changed files

| path | action | summary |
|---|---|---|
|  | added / modified / deleted / moved |  |

## 4. Skipped or unchanged-reviewed files

| path | result | reason |
|---|---|---|
|  | skipped / unchanged-reviewed |  |

## 5. Transformation or generation rules

- Rule 1:
- Rule 2:
- Explicit exclusions:

## 6. Sample checks

| sample path | check performed | result | issue / evidence |
|---|---|---|---|
|  |  | pass / fail / not_checked |  |

## 7. Failed-sample rule review

| failure | affected scope | rule change needed? | action |
|---|---|---|---|
|  |  | yes / no |  |

## 8. Quality / index / link / frontmatter / status checks

| check | command or evidence | result |
|---|---|---|
| frontmatter YAML |  |  |
| Markdown links |  |  |
| old-path residue |  |  |
| docs/index.md |  |  |
| git diff --check |  |  |

## 9. Conflicts and operator decision

| conflict | decision | owner | follow-up |
|---|---|---|---|
|  |  |  |  |

## 10. Closure

- Final decision:
- Remaining risk:
- Next action:
```

## 5. Completion Report Skeleton

```markdown
# Completion Report: <task name>

## 1. Scope

| Field | Value |
|---|---|
| task type |  |
| asset level |  |
| target paths |  |
| source paths |  |
| user instruction |  |

## 2. Changed files

| path | action | summary |
|---|---|---|
|  | added / modified / deleted / moved |  |

## 3. Validation performed

| validation | command / evidence | result |
|---|---|---|
| frontmatter YAML |  |  |
| Markdown links |  |  |
| old-path residue |  |  |
| docs/index.md |  |  |
| git diff --check |  |  |

## 4. Quality and status impact

| asset | quality_status impact | lifecycle / decision evidence |
|---|---|---|
|  |  |  |

## 5. Index and link impact

- Index changes:
- Removed links:
- Updated links:
- Historical references intentionally left:

## 6. Final decision and follow-up

| Field | Value |
|---|---|
| decision label |  |
| unresolved risks |  |
| follow-up owner |  |
| non-software boundary | no runtime/code/dependency changes unless explicitly stated |
```

## 6. Batch Completion Report Skeleton

```markdown
# Batch Completion Report: <batch operation name>

## 1. Classification

| Field | Value |
|---|---|
| operation type |  |
| target set |  |
| owner decision |  |
| readiness evidence |  |

## 2. Outcome inventory

| category | count | paths or summary |
|---|---:|---|
| added |  |  |
| modified |  |  |
| deleted |  |  |
| moved/renamed |  |  |
| skipped |  |  |
| unchanged-reviewed |  |  |

## 3. Validation performed

| validation | command / evidence | result |
|---|---|---|
| frontmatter YAML |  |  |
| Markdown links |  |  |
| old-path residue |  |  |
| docs/index.md |  |  |
| git diff --check |  |  |

## 4. Index and link impact

- Canonical entries kept:
- Entries removed:
- Entries updated:
- Old paths intentionally retained only as historical records:

## 5. Unresolved items and follow-up

| item | impact | owner | next action |
|---|---|---|---|
|  |  |  |  |

## 6. Recovery and closure

- Recovery approach used:
- Files requiring re-review:
- Final decision:
- Non-software boundary:
```

## 7. Maintenance

Update this template only when:

- `docs/governance/docs-asset-governance.md` changes required asset evidence.
- `docs/governance/docs-change-governance.md` changes decision labels, readiness rules, rework rules, or batch stop conditions.
- The batch runbook adds new mandatory evidence.
- A real task exposes a missing field that affects review or completion auditability.
