---
doc_id: runbook-batch-governance-runbook
title: 批量治理 Runbook：生成、重构、抽样审查与停止条件
concept: batch_governance_runbook
topic: runbooks
depth_mode: standard
created_at: '2026-06-17T16:20:05+08:00'
updated_at: '2026-06-23T00:00:00+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/0-7-batch-readiness.md
  - _bmad-output/implementation-artifacts/epic-6-context.md
  - _bmad-output/implementation-artifacts/spec-6-1-batch-governance-runbook.md
  - docs_folder_consolidation_progress_2026_06_23
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-change-governance.md
  - docs/governance/docs-asset-governance.md
  - docs/templates/governance-record-templates.md
time_context: docs_folder_consolidation_progress_2026_06_23
applicability: batch_generation_and_batch_refactor_execution_after_readiness
prompt_version: not_applicable
template_version: runbook_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-change-governance.md
  - docs/governance/docs-asset-governance.md
  - docs/templates/governance-record-templates.md
open_questions:
  - 后续实际批量任务使用 `docs/templates/governance-record-templates.md` 后，是否需要进一步收紧 sample review、conflict evidence 和 completion evidence 的必填字段？
  - 后续如 Maxwell 授权 executable validation tooling，是否需要为本 runbook 补充独立 tooling-readiness 与 architecture/ADR 前置条件？
---

# 批量治理 Runbook：生成、重构、抽样审查与停止条件

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目的正式 runbook 资产，规定在 `docs/governance/docs-change-governance.md` 已经完成、owner decision 已经由当前任务或 Maxwell 明确授权之后，Agent 如何约束一次命名的批量生成或批量重构任务。

本文补充 `docs/governance/docs-change-governance.md`、`docs/governance/docs-asset-governance.md`、`docs/methodology/document-generation-methodology.md`、`docs/methodology/concept-document-quality-gate.md` 和 `docs/templates/governance-record-templates.md`。Readiness checklist 仍负责写入前范围、冲突、排除、停止条件和 owner decision；本文只负责外部授权之后的批量执行顺序、检查点、样本审查、失败暂停、恢复证据和完成证据位置。

本文不替代主方法论、质量门禁、资产治理、变更治理或记录模板；本文只规定批量任务获得授权后的执行顺序和停止条件。

本文不授权实际批量工作本身，也不把 `owner_decision` 从禁止状态改成允许状态。一次批量生成或批量重构仍必须有独立 readiness record、明确 target set、allowed files、excluded files、owner decision、当前任务或 owner 授权和 recovery approach。本文不创建 actual batch records、actual review records、actual completion reports、额外 batch review templates、额外 batch completion templates、schema fields、automation、validators、scanners、generators、CLI/API/UI/database/deployment/CI、package manifest、`src/` 或 `tests/`。

本文的 owner entry point 是 `docs/index.md` 的 `runbooks` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## runbooks` 下列出 `docs/runbooks/batch-governance-runbook.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: reviewed` 表示本文已按治理资产标准进行人工审查并保守入库：runbook role、readiness handoff、batch generation path、batch refactor path、sample review、failure/stop handling、recovery evidence、artifact separation、owner/index entry、相邻治理依赖、链接/索引边界和非软件边界已检查。2026-06-23 consolidation 只同步直接引用和相邻依赖表述。本文不声明 `validated`，因为本文尚未在实际命名批量任务中验证。

## 前置条件

执行本文前，必须先满足下列条件。任一条件不满足，不能进入批量写入。

| 前置条件 | 必须看到的证据 | 不满足时 |
| --- | --- | --- |
| readiness record exists | 有命名的 Batch Readiness Record，包含 operation type、target set、allowed/excluded files、expected output、non-output、recovery approach、sample/review plan、conflict summary 和 owner decision | 回到 `docs/governance/docs-change-governance.md` |
| external authorization permits writes | readiness `owner_decision` 是 `proceed` 或 `proceed_with_exclusions`，且当前任务或 Maxwell 明确允许本次写入 | `clarify_before_write`、`defer_to_later_story`、`stop_for_maxwell_confirmation`、`not_authorized` 或缺少当前写入授权时都必须停止 |
| operation type fits this runbook | operation type 是 `batch_generation`、`batch_refactor` 或先拆分成这两类的 mixed task；mixed task 必须为每条路径分别冻结 target、sample 和 recovery evidence | 其他批量类型先回 readiness 或等待对应 owner/task 明确授权；不得从 review evidence 或 completion evidence 推导执行授权 |
| target set concrete | explicit file list、index-derived set 或 topic-derived set 已展开为可复查文件清单，或 Maxwell manually approved list 可复查 | 目标集未冻结或只能用 glob-like rule 推导时停止 |
| excluded files explicit | `_bmad-output/`、`.agents/skills/`、`docs/methodology/`、`docs/governance/`、`docs/templates/`、`docs/runbooks/` 等相邻路径是否排除或允许已写明；frozen target set 与 excluded files 交集必须为空 | 排除边界不清或交集非空时停止 |
| recovery approach ready | 写入前知道如何恢复、如何保留旧状态、如何区分 changed/skipped/unchanged-reviewed files | 没有恢复路径时停止 |
| non-software boundary intact | 不需要可执行工具、脚本、CI、软件测试或自动化 | 需要 tooling 时停止并请求 Maxwell 确认与新架构决策 |

Readiness approval 加当前任务/owner 授权，只能覆盖 readiness record 中命名的那一次批量动作。不得把一次批准扩展成后续无关批量工作，也不得把本文当作长期批量写入授权。

如果 frozen target set 为空，任务必须停止并记录 `not_applicable` 或 `deferred_with_reason`；不得写出暗示已经执行批量工作的完成证据。

## 执行总览

一次批量任务按下列顺序执行。不能跳过 checkpoint，也不能在 sample review 失败后继续全量写入。

| Step | 动作 | 产出 |
| --- | --- | --- |
| 1 | 读取 readiness record 和 source hierarchy | 明确 operation、target set、allowed/excluded files、stop conditions |
| 2 | 冻结 target set | 写入前文件列表、sample set、excluded files 和 recovery notes |
| 3 | 选择执行路径 | `batch_generation` 或 `batch_refactor` |
| 4 | 执行小样本 | 少量候选输出或少量目标文件变更 |
| 5 | sample review | 检查 frontmatter、path、topic、index、links、quality status、source/time context 和 non-software boundary |
| 6 | 处理失败或冲突 | stop、revise rule、shrink scope、defer 或 ask Maxwell |
| 7 | 处理剩余目标 | 只在样本通过、stop conditions 未触发、且外部授权覆盖剩余目标后继续 |
| 8 | 完成验证 | changed/skipped/unchanged-reviewed files、index/link impact、status impact、unresolved risks |
| 9 | 汇报或交给后续记录 | 使用 `docs/templates/governance-record-templates.md`；实际 report storage 仍需当前任务或 owner 授权 |

如果任务会影响 governance、methodology、template、runbook、quality gate、prompt、BMad workflow、skill 或 project context，必须在 Step 2 前额外确认 downstream impact analysis 已完成。影响不确定时停止，不得在批量执行中顺手改规则。

## 批量生成路径

Batch generation 指一次生成多篇候选文档、正式文档、模板化草稿或治理资产。它的风险是把缺失输入、错误 topic、弱来源、错误质量状态或候选输出批量写入正式 `docs/`。

执行前必须确认 common input rules：

| 输入项 | 最低要求 |
| --- | --- |
| concept/topic source | 每个目标对象的 concept、topic、path expectation 和 user context 可定位 |
| generation contract | 使用 `docs/methodology/document-generation-methodology.md` 的输入/输出合同 |
| intake route | 使用 `docs/methodology/document-generation-methodology.md` 区分任务类型、文档路径类型和 depth mode |
| prompt/template version | 写明使用的 prompt/template/source rule，不得伪造版本 bump |
| source/time context | 每个 candidate 都有 source basis、time context、verified/inference/open question 边界 |
| promotion criteria | 候选进入 `docs/` 前必须满足 candidate promotion、frontmatter、quality gate、index 和 link checks |

Batch generation 顺序：

1. 先生成 candidate outputs，不直接覆盖正式 `docs/` 资产。
2. 对 sample set 执行完整候选审查，至少覆盖一个典型、一个边界或高风险样本；如果 target set 很小，也可以全量审查。
3. 每个 candidate 必须检查 title/H1、frontmatter baseline、topic/path、source basis、time context、related docs、open questions、quality status 和 index impact。
4. Candidate 只有在 promotion criteria 满足、且当前 story/owner 授权正式写入后，才可以写入正式 `docs/` 路径。
5. 被外部授权正式写入后，必须同步 `docs/index.md` 或记录不适用/延后/排除理由。
6. 任何 sample 命中 Hard Fail、topic/path conflict、source/time overclaim、duplicate ambiguity 或 status overclaim 时，暂停剩余生成，先修正 generation rule 或缩小 target set。

Batch generation 不得做：

- 把 `_bmad-output/` 候选直接复制为正式资产并跳过 promotion。
- 批量声明 `quality_status: reviewed` 或 `validated`，除非每个目标都有对应审查证据。
- 为了批量方便新增 schema fields、index schema、生成器配置或自动化工具。
- 在同一次任务中顺手生成实际 batch records、actual batch completion reports 或未经授权的额外治理/模板资产。

## 批量重构路径

Batch refactor 指用同一转换规则修改多篇已有资产。它的风险是批量破坏身份、标题、路径、frontmatter、链接、质量状态、来源语境或读者可理解性。

执行前必须冻结 transformation rule：

| 字段 | 必须说明 |
| --- | --- |
| transformation rule | 具体改什么，不改什么；必须可用人工审查复现 |
| target file list | 逐个文件或可复查选择规则 |
| exclusions | 哪些文件、字段、小节、状态、链接、topic 或 asset class 不改 |
| before/after evidence | 至少一个样本的改前/改后期望 |
| semantic preservation | 如何确认意思没变、状态没被伪晋升、来源没被伪造 |
| index/link impact | 哪些路径、标题、索引或链接可能受影响 |
| stop conditions | 样本失败、冲突、范围扩张或需要新规则时如何停 |

Batch refactor 顺序：

1. 先对 sample set 应用 transformation rule，不直接改完整 target set。
2. Sample review 必须检查 Markdown structure、frontmatter parse、title/H1、`doc_id` stability、topic/path、source/time context、quality status、links 和 index impact。
3. 如果 sample review 发现转换规则改变语义、破坏证据、制造断链、扩大 scope 或触发 Hard Fail，停止并修订规则；不能继续全量套用。
4. Sample 通过后，再按 frozen target list 和外部授权范围处理剩余文件。
5. 每个 changed file 必须能说明 changed section、reason、quality/status impact 和 unresolved risk。
6. Skipped files 必须记录 reason，例如 excluded、conflict、needs Maxwell、requires future story、not applicable 或 failed sample category。

Batch refactor 不得做：

- 批量 frontmatter normalization 后声称质量状态提高。
- 批量改 `quality_status`、lifecycle、prompt/template version、`doc_id` 或 path ownership，除非 readiness 和 owner decision 明确授权且每个目标证据充分。
- 把 “style cleanup” 扩展成 topic 迁移、重复合并、路径重命名、index-wide restructuring 或 link sweep。
- 用脚本、生成器、lint/scoring automation 或 CI 作为默认执行方式。

## Sample Review Set

Sample review 是继续执行的 gate，不是装饰性抽样。Readiness record 必须说明 sample set 如何选择；runbook 执行时必须检查样本是否仍代表 target set 风险。

最低 sample 策略：

| Target set size / risk | Sample expectation |
| --- | --- |
| 2-5 files, low risk | 至少 1 个典型样本；若包含不同 asset class，按 asset class 各取 1 个 |
| 6-20 files | 至少 2 个样本：一个典型、一个边界或高风险 |
| 20+ files | 至少 3 个样本，并覆盖主要 topic、asset class 或 transformation branch |
| governance/methodology/template/runbook targets | 至少 1 个 governance-layer sample，并检查 downstream impact |
| mixed target set | 按 asset class 拆分 sample，不得用一个普通文档样本代表全部 |

Sample review 必须记录：

- sampled files or candidate outputs;
- why these samples represent risk;
- checks performed;
- findings and local disposition;
- whether transformation/generation rule changed after sample;
- whether remaining target set can proceed.

如果样本失败，默认停止。只有失败被明确分类、规则被修正、样本重新通过，或 owner 明确批准缩小范围后，才可以继续。任何 rule、target set、exclusion、prompt/template selection 或 transformation branch 在 sample pass 后发生变化，都必须重新运行对应 sample review，不能把旧 sample pass 复用于新规则。

## Index、Frontmatter 与 Link Impact

Batch execution 不能把 index、frontmatter 和 links 留到最后猜测。每个批量任务必须在执行中持续记录影响。

Frontmatter checks:

- required fields exist and remain YAML-compatible;
- `doc_id` remains stable unless explicitly authorized;
- `title`、`concept`、`topic`、path 和 H1 不互相冲突;
- `source_basis`、`related_docs`、`open_questions` remain arrays;
- `quality_status` 不因格式调整、生成完成或 sample pass 自动晋升;
- no unauthorized fields such as `schema_version`、`lifecycle_state`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`decision_status` or batch metadata fields are added.

Index checks:

- new formal assets get `docs/index.md` entries or controlled non-index evidence;
- rename/move/retitle/topic changes update or defer index impact explicitly;
- batch index changes do not become index-wide restructuring unless authorized;
- no `_bmad-output/` workflow output, candidate, temporary draft, review output or completion record is added to formal index.

Link checks:

- changed-file Markdown links resolve or are recorded as unresolved with owner/future action;
- `related_docs` targets exist unless intentionally planned and recorded in open questions;
- inbound/outbound review scope follows `docs/governance/docs-asset-governance.md` when applicable;
- successor/replacement, duplicate/coexistence or migration implications route to their owning policies.

## Failure、Stop 与 Recovery

Unexpected conflicts or failures pause work. The Agent must not continue batch execution while treating failures as later cleanup.

Failure evidence minimum:

| Field | Meaning |
| --- | --- |
| failure type | naming、path、topic、duplicate、frontmatter、quality status、lifecycle、source/time、links、index、workflow contract、non-software boundary、sample failure or owner decision |
| affected files | concrete files or candidate outputs |
| evidence location | section, frontmatter field, link, index entry or sample output |
| risk | what would break if execution continues |
| proposed next step | revise rule、shrink target set、defer、ask Maxwell、regenerate sample、stop |
| current status | stopped、deferred、waiting owner、rule revised、sample rerun or resolved |

Mandatory stop conditions are inherited from `docs/governance/docs-change-governance.md`. In this runbook, also stop when:

- sample review fails and the rule has not been revised;
- generated candidates do not meet promotion criteria;
- refactor rule changes meaning or quality/status claims;
- target set expands beyond readiness approval;
- required batch review or completion evidence is needed before evidence can be trusted, but the current task has not authorized that record/report;
- completion wording would need to claim `validated`;
- execution requires code, scripts, automation, scanners, generated reports, package manifests or CI.

Recovery evidence must separate:

- changed files;
- skipped files and reasons;
- unchanged-reviewed files;
- candidate outputs not promoted;
- formal docs changes;
- workflow/planning artifacts used as evidence;
- restore or manual rollback notes.

## Formal Docs 与 Workflow Artifacts 分离

Formal `docs/` assets and `_bmad-output/` workflow artifacts have different lifecycle rules.

| Artifact class | Allowed use in batch execution | Not allowed |
| --- | --- | --- |
| formal `docs/` asset | canonical content, governed frontmatter/body/index/link changes | silent overwrite, unreviewed status promotion |
| `_bmad-output/` story/spec/context | planning context, execution evidence, checklist state | direct index entry, automatic promotion to `docs/` |
| candidate output | sample or draft for review | treated as formal asset before promotion |
| review evidence | supports decision and failure handling | changes target state without decision policy |
| completion evidence | summarizes work and risk | substitutes for missing validation |
| batch change review record | structured review, conflict and operator-decision evidence when explicitly authorized | created as an actual record without current-task authorization, or treated as execution authorization |
| batch completion report | structured completion summary fields for named batch tasks | created as an actual completion report without current-task authorization, or treated as execution authorization |

If a batch task produces candidate documents, reports, review notes or completion summaries, they remain workflow/planning artifacts until a separate promotion or formal asset story approves their target path, frontmatter, index treatment and quality status.

## Completion Evidence For This Runbook

Batch review, conflict evidence and named batch completion evidence should use `docs/templates/governance-record-templates.md` fields when the current task or owner authorizes an actual report location. Without report storage authorization, final completion evidence remains a summary inside the current response, review evidence, existing completion report or authorized workflow note. It must not create a standalone batch completion report or extra template without authorization.

Minimum batch completion summary:

| Evidence category | Required statement |
| --- | --- |
| operation | generation, refactor or mixed; readiness record reference |
| scope | target set, allowed files, excluded files, changed/skipped/unchanged-reviewed files |
| sample review | sampled files, checks, findings, pass/fail and rule changes |
| quality/status impact | status changes made or not made, with evidence |
| index/link impact | updated, not applicable, deferred or blocked |
| source/time impact | source basis and time context changes or non-impact |
| failure/stop events | failures, affected files, decisions and follow-up |
| recovery | restore notes and preserved old state |
| formal/workflow separation | what entered `docs/`, what stayed in `_bmad-output` or candidate state |
| non-software boundary | no unauthorized executable tooling or software assets introduced |

The completion evidence must use `通过`、`未通过`、`未验证` or `不适用` where a validation conclusion is needed.

## 本文自身的 Index Impact Decision Record

```text
Index Impact Decision Record

- affected file: docs/runbooks/batch-governance-runbook.md
- target title: 批量治理 Runbook：生成、重构、抽样审查与停止条件
- target path: ./runbooks/batch-governance-runbook.md
- target section: docs/index.md ## runbooks
- outcome: updated
- action taken: add runbooks section and runbook entry
- reason: formal runbook asset under docs/runbooks/
- validation result: expected target exists and relative link resolves
- unresolved risk: none for this index entry; actual batch completion report storage remains subject to current story or owner authorization
```

## 验证清单

验证本文自身时，至少检查：

1. Frontmatter required fields exist; `source_basis`、`related_docs` and `open_questions` are YAML arrays.
2. H1 matches the title meaning and the path `docs/runbooks/batch-governance-runbook.md`.
3. Body states role, authority, applicable scope, owner entry point, navigation treatment and index treatment.
4. Body explains readiness handoff and does not replace `docs/governance/docs-change-governance.md`.
5. Batch generation path includes common input rules, prompt/template version selection, topic assignment, candidate review and promotion criteria.
6. Batch refactor path includes transformation rule, target file list, exclusions, sample review set, frontmatter/index/link impact and stop conditions.
7. Failure handling records failure type, affected files, risk and proposed next step.
8. Recovery separates changed, skipped, unchanged-reviewed, candidate and formal docs changes.
9. `docs/templates/governance-record-templates.md` is referenced as the structured review and completion evidence template.
10. `docs/index.md` has a valid `## runbooks` entry.
11. No unauthorized schema field, actual batch record, actual review record, actual completion report, additional batch review template, additional batch completion template, runtime code, package manifest, `src/`, `tests/`, automation, validator, scanner, generator, CLI/API/UI/database/deployment/CI or software test framework was introduced.

## 维护触发点

本文需要在以下情况下复核：

- `docs/templates/governance-record-templates.md` 更新 batch review、failure record、conflict evidence 或 completion evidence 字段。
- 实际批量任务暴露 completion evidence 与 changed/skipped/unchanged-reviewed 汇报字段不足。
- `docs/governance/docs-change-governance.md` 修改 owner decision、conflict matrix、stop condition 或 recovery approach。
- `docs/governance/docs-asset-governance.md` 或 `docs/templates/governance-record-templates.md` 改变相关证据要求。
- Maxwell 明确授权 executable validation tooling、machine-readable schema、generated reports、batch scanner、lint/scoring automation 或 CI；此时必须先做新的 architecture/ADR 和 scope decision，不能从本文直接推导软件实现。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [统一概念文档规范：AI 生成、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
- [正式 docs 资产治理规范：身份、元数据、路径、生命周期、索引、链接与网络边界](../governance/docs-asset-governance.md)
- [文档治理执行规范：复用扫描、晋升、合并、迁移、修订、批量、决策与返工闭环](../governance/docs-change-governance.md)
- [文档治理记录模板：审查、批量审查、完成汇报与批量完成汇报](../templates/governance-record-templates.md)
