---
doc_id: governance-batch-readiness-checklist
title: 批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略
concept: batch_readiness_checklist
topic: governance
depth_mode: standard
created_at: '2026-05-25T09:34:08+08:00'
updated_at: '2026-06-15T17:21:39+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/planning-artifacts/sprint-change-proposal-2026-05-20.md
  - _bmad-output/planning-artifacts/implementation-readiness-report-corrected-2026-05-20.md
  - _bmad-output/implementation-artifacts/0-1-agent-behavior-constraints.md
  - _bmad-output/implementation-artifacts/0-2-governance-asset-boundary-policy.md
  - _bmad-output/implementation-artifacts/0-3-lifecycle-states.md
  - _bmad-output/implementation-artifacts/0-4-prompt-template-quality-version-governance.md
  - _bmad-output/implementation-artifacts/0-5-quality-gate-baseline.md
  - _bmad-output/implementation-artifacts/0-6-governance-asset-navigation-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/methodology/document-generation-methodology.md
  - docs/index.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_key_draft_review_2026_06_15
applicability: batch_governance_readiness_scope_conflict_and_stop_criteria
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/methodology/document-generation-methodology.md
  - docs/index.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
open_questions:
  - index synchronization rules 已建立后，是否需要细化批量索引更新的 sample/review 规则？
  - 如果 completion-report template 后续调整 Batch Readiness Record summary 字段，是否需要同步本文的 readiness record 汇总方式？
  - Epic 6 建立 batch governance runbook 后，是否需要把本文中的 readiness decision 与 runbook 执行步骤拆分得更严格？
---

# 批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略

## 资产角色、权威与适用范围

这份文档是 `Knowledge` 项目的 Foundation 批量治理 readiness 政策。它定义任何批量生成、迁移、重构、规范化、审查或相邻批量治理动作开始前，Agent 必须如何判定范围、暴露冲突、记录停止条件、说明恢复策略，并取得 owner decision。

本资产补充以下正式规则：

- `docs/methodology/document-generation-methodology.md`
- `docs/governance/agent-behavior-constraints.md`
- `docs/governance/lifecycle-states.md`
- `docs/governance/prompt-template-quality-version-governance.md`
- `docs/governance/revision-regeneration-continuity-policy.md`
- `docs/methodology/concept-document-quality-gate.md`
- `docs/governance/governance-asset-navigation-policy.md`

本资产不替代主方法论、质量门禁、生命周期政策、版本治理、导航政策、`docs/governance/index-synchronization-rules.md`，也不替代未来 Epic 6 batch runbook、batch review template 或 batch completion template。它只规定批量动作写入前的 readiness 判定基线。

本资产不授权实际批量生成、批量迁移、批量 normalization、批量状态重写、批量重命名、批量删除、批量废弃、批量归档、index-wide restructuring 或 executable validation tooling。完成 readiness 只表示该次命名批量动作具备进入下一步确认或执行的依据；它不是后续无关批量工作的长期授权。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组；navigation treatment 是 `listed_in_docs_index`。本文的权威范围限于 batch readiness、scope judgment、conflict exposure、stop criteria、downstream impact analysis 和 recovery evidence，不覆盖具体批量执行步骤。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：batch readiness 范围判定、冲突暴露、停止条件、恢复策略、owner decision、相关治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

## 什么算批量治理

批量治理指一次任务会按同一规则影响多个文件、多个概念、多个 topic、多个状态字段、多个链接入口、多个索引项或多个 workflow/skill contract 的动作。只要目标集不是单一明确文件，或需要先用规则选出一组目标，就必须先执行 readiness。

以下动作默认属于批量治理：

| 操作类型 | 定义 | readiness 重点 |
| --- | --- | --- |
| batch generation | 一次生成多篇正式文档、候选文档、模板化草稿或治理资产 | target set、命名、topic、source basis、index impact |
| batch refactor | 用同一结构或措辞规则重构多篇文档或多个治理资产 | 保留原意、路径/链接影响、质量状态影响 |
| batch migration | 把一组旧文档迁移到新 schema、路径、topic、模板、版本或生命周期规则 | 恢复策略、before/after、旧版本保留、迁移证据 |
| batch frontmatter normalization | 批量补齐、重排、改写或统一 frontmatter 字段 | 字段语义、假证据风险、Epic 2 schema 边界 |
| batch status conversion | 批量改写 `quality_status`、废弃/归档标记或等价状态表达 | 生命周期证据、质量门禁、禁止静默晋升 |
| batch link/index update | 批量更新正文链接、`related_docs`、入链、出链或 `docs/index.md` 条目 | link evidence、index policy、navigation treatment |
| batch deprecation/archive | 一次废弃、归档、降级或替换多篇资产 | successor、remaining access、索引可见性 |
| batch review | 用同一标准审查一组文件、候选文档或治理资产 | 抽样、冲突记录、completion evidence |

以下动作不自动构成批量治理，但仍要按对应任务类型完成验证：

- 修改一个明确目标文件。
- 为一个新正式资产同步一个直接相关的 `docs/index.md` 入口。
- 在 story 明确授权范围内，对一个目标文件和一处紧邻引用做小范围同步。
- 只更新当前 story 的 Dev Agent Record、File List、Change Log、checkbox 或 sprint status。
- 只记录候选范围、风险或 open question，不对目标文件集写入。

如果任务混合正式 `docs/` 资产、`_bmad-output/` 候选来源、方法论资产、治理资产、workflow/skill 文件或 future templates/runbooks，必须按资产层级拆分 target set，并采用更严格的范围控制。

## Readiness Record

每次批量治理写入前，必须留下 readiness record。记录可以出现在 story、completion report、review record、runbook execution note 或用户确认记录中；即使 Epic 3 固定模板已落地，也不得新增未经授权的全局 frontmatter 字段。

最小记录形状如下：

```yaml
batch_readiness_record:
  operation_type: batch_generation | batch_refactor | batch_migration | batch_frontmatter_normalization | batch_status_conversion | batch_link_index_update | batch_deprecation_archive | batch_review | mixed
  purpose: why_this_batch_action_is_needed
  initiating_source: user_request | story | review_finding | governance_policy | runbook | other_named_source
  target_document_set:
    evidence_type: explicit_file_list | glob_like_description_with_samples | index_derived_set | topic_derived_set | manually_approved_list
    files_or_rule:
      - docs/path/or_selection_rule
    sampled_files:
      - docs/path/to/sample.md
  selection_rule: exact_rule_for_inclusion
  affected_topics:
    - topic_name
  allowed_files:
    - docs/path/or_scope
  excluded_files:
    - docs/path/or_scope
  expected_output:
    - changed_file_or_output_type
  non_output:
    - explicitly_not_created_or_not_changed
  recovery_approach:
    preserve_old_version: yes_or_no_with_method
    overwrite_rule: no_overwrite_until_review | explicit_overwrite_authorized | append_only | manual_patch
    rollback_or_restore_notes: how_to_restore_or_revert_manually
  sampling_review_approach: sample_size_rule_and_review_scope
  conflict_summary:
    naming: proceed | stop | clarify | defer
    path: proceed | stop | clarify | defer
    topic: proceed | stop | clarify | defer
    duplicate_concepts: proceed | stop | clarify | defer
    frontmatter: proceed | stop | clarify | defer
    quality_status: proceed | stop | clarify | defer
    links: proceed | stop | clarify | defer
    docs_index: proceed | stop | clarify | defer
    workflow_contract: proceed | stop | clarify | defer
  owner_decision: proceed | proceed_with_exclusions | clarify_before_write | defer_to_later_story | stop_for_maxwell_confirmation | not_authorized
  stop_conditions:
    - condition_that_stops_work
```

Required fields are: operation type, purpose, initiating source, target document set, selection rule, affected topics, allowed files, excluded files, expected output, non-output, recovery approach, sampling/review approach, owner decision, and stop conditions.

Target set evidence must be concrete. Acceptable evidence is:

- an explicit file list;
- a glob-like description plus sampled files;
- an index-derived set from `docs/index.md`;
- a topic-derived set with listed topic paths;
- a manually approved list from Maxwell or an authorized story.

Excluded files must be explicit when a path could reasonably be mistaken as in scope. Common exclusion candidates include `_bmad-output/`, `.agents/skills/`, `docs/methodology/`, `docs/governance/`, `docs/_reports/`, future `docs/templates/`, and future `docs/runbooks/`.

Readiness approval permits only the named batch action over the named target set. It does not authorize later batch work with a different target set, operation type, status field, schema, topic, path rule, owner decision or output artifact.

## 范围判定与排除规则

Scope must be proven before any write action. The Agent must be able to answer: which files are in scope, why they are in scope, which nearby files are not in scope, what output is expected, and how to recover if the batch action is wrong.

Minimum scope checks:

| 检查项 | 必须说明什么 | Stop signal |
| --- | --- | --- |
| operation type | 本次属于哪一种 batch operation；若 mixed，拆出子类型 | 只说“批量处理”而没有动作类型 |
| target document set | 目标集如何得出，是否可复查 | 目标集来自模糊描述，如“相关文档” |
| affected topics | 影响哪些 topic、目录、资产层级或 workflow group | topic 归属不清或跨层混合 |
| allowed files | 哪些文件允许写入，是否包含 index/story/status | allowed scope 比 story 授权更大 |
| excluded files | 哪些容易误判的路径明确排除 | 没有排除 `_bmad-output/`、methodology/governance、skills 等相邻路径 |
| expected output | 预期新增、修改、删除或生成什么 | 输出包括未授权模板、runbook、脚本、测试或工具 |
| non-output | 明确不会创建或修改什么 | readiness 被误读成实际批量执行 |
| recovery approach | 写入前如何保留旧状态和恢复路径 | 覆盖旧文件但没有恢复说明 |
| sampling/review | 如何检查样本、冲突和完成证据 | 没有 review plan 却准备批量写入 |

`_bmad-output/` 是 workflow output 边界。它可以作为 planning context、story evidence、candidate source 或 review record 来源，但不能自动晋升为 `docs/` 正式资产。任何从 `_bmad-output/` 到 `docs/` 的 promotion 都必须重新判定 target path、frontmatter、source basis、navigation treatment、index treatment 和 quality evidence。

Formal docs、methodology assets、governance assets、reports、future templates/runbooks、skills/workflows 必须按 asset class 分开处理。混合批量动作不能用一个总判断覆盖所有资产层级；如果无法拆分，必须 `clarify_before_write` 或 `stop_for_maxwell_confirmation`。

## 冲突暴露矩阵

Readiness 必须暴露冲突，而不是只在结论中写“无冲突”。每个 checked conflict category 都必须记录 local disposition：`proceed`、`stop`、`clarify` 或 `defer`。local disposition 是单项判断，和 overall owner decision 分开。

| 冲突类别 | Evidence location | Severity basis | Required record | Mandatory stop when |
| --- | --- | --- | --- | --- |
| naming conflict | 文件名、标题、H1、`doc_id`、index title | 是否制造重复身份或误导入口 | affected files、证据位置、proposed handling、owner decision needed、local disposition | canonical title/file/doc_id 冲突无法判定 |
| path conflict | 目标路径、目录层级、topic 路径、promotion path | 是否进入错误资产层级 | affected paths、expected path、conflict reason、local disposition | canonical path 或 asset class 不确定 |
| topic ambiguity | frontmatter `topic`、目录、index section、正文 scope | 是否无法判断知识归属 | topic candidates、why ambiguous、owner decision needed | topic 归属会影响索引或 source hierarchy |
| duplicate concepts | `concept`、相邻文档、同义主题、候选文档 | 是否出现重复概念或平行入口 | possible duplicates、difference claim、merge/split/defer proposal | duplicate concept ambiguity 无法解决 |
| `doc_id` identity | `doc_id`、旧文档、successor/replacement | 是否破坏稳定身份 | old/new identity evidence、handling | 需要改稳定身份但无授权 |
| frontmatter gaps | required fields、YAML arrays、版本/状态字段 | 是否导致 schema 或假证据风险 | missing/conflicting fields、proposed handling | 需要新增 unauthorized schema 或批量伪造状态 |
| quality status impact | `quality_status`、review decision、lifecycle state | 是否暗示不存在的审查/迁移/验证 | before/after claim、quality evidence | Hard Fail、状态晋升证据不足或批量状态 rewrite 未授权 |
| lifecycle impact | draft/reviewed/validated/deprecated/archived 等状态 | 是否改变正式资产允许使用方式 | lifecycle before/after、successor/index impact | lifecycle/status ambiguity 无法消除 |
| prompt/template versions | `prompt_version`、`template_version`、quality gate version | 是否触发版本治理事件或迁移计划 | no action / targeted review / gradual migration / batch readiness required | 需要批量 normalization 或 version bump 但无 readiness/授权 |
| source/time context | `source_basis`、`time_context`、正文日期、来源限制 | 是否制造当前性或来源假证据 | source checked、time context、unknowns | 来源不可验证但输出声称 validated/current |
| related docs | `related_docs`、successor、supporting docs | 是否断开相邻规则或重复入口 | related targets、missing targets、open questions | 关键 related docs 不存在且未记录 |
| links | 正文链接、入链、出链、相对路径、absolute path | 是否留下断链或旧路径 | changed links、checked links、unresolved links | broken links cannot be repaired in scope |
| `docs/index.md` impact | index section、title、path、topic、navigation treatment | 是否需要新增、移动、删除或排除入口 | index action、reason、local disposition | index policy conflict 或 index-wide restructuring 未授权 |
| `_bmad-output/` promotion boundary | candidate source、story output、review output | 是否把 workflow output 伪装成 formal docs | source/target separation、promotion evidence | 自动 promotion 或无 frontmatter/index/quality evidence |
| workflow contract | BMad story、skill、review follow-up、sprint status | 是否破坏 step order、state fields 或 completion rules | affected workflow fields、required updates | workflow/skill contract impact 不确定 |

Every conflict record must include affected files, evidence location, severity, proposed handling, owner decision needed, and whether work must stop.

`no conflicts found` is valid only if the record says what was checked. A blank conflict section is not evidence.

## 治理层下游影响分析

Batch work that touches governance, methodology, template, runbook, quality, prompt, project context, BMad workflow or skill assets requires extra downstream impact analysis before writes.

This heightened analysis applies when targets include:

- `docs/methodology/`
- `docs/governance/`
- future `docs/templates/`
- future `docs/runbooks/`
- `.agents/skills/`
- `_bmad-output/project-context.md`
- BMad workflow assets, story files, sprint status, skill instructions or project context outputs

Minimum downstream impact checks:

| Impact area | Must check |
| --- | --- |
| main methodology | 是否改变 `docs/methodology/document-generation-methodology.md` 的默认执行入口、任务分类、质量门禁接入或仓库集成规则 |
| template | 是否改变 required section、depth mode、document path type、frontmatter expectation 或 future template obligations |
| quality gate | 是否改变 Hard Fail、评分、通过结论、质量状态或 completion evidence |
| fixed prompt | 是否改变触发词、输入、输出、任务分类、等待点或停止条件 |
| version governance | 分类为 `no_action`、`targeted_review`、`gradual_migration` 或 `batch_readiness_required` |
| lifecycle | 是否改变 draft/reviewed/validated/maintained/deprecated/archived 判断或状态转换证据 |
| navigation/index | 使用 `listed_in_docs_index`、`referenced_from_methodology_entry`、`referenced_from_governance_entry` 或 `intentionally_excluded` 分类 |
| related docs | 是否需要新增、移除、重排或记录 missing related docs/open questions |
| review/decision records | 是否需要 Epic 3 review/completion evidence 或 owner decision 记录 |
| completion reports | 是否改变完成汇报字段、证据格式或后续追踪义务 |
| skill/workflow triggers | 是否改变 skill 触发条件、workflow activation、story discovery 或 review continuation |
| menus/inputs/outputs | 是否改变用户选择菜单、必填输入、输出位置或文档语言 |
| waiting points | 是否新增、移除或绕过 ask/HALT/confirmation step |
| state fields | 是否影响 story Status、checkbox、sprint status、frontmatter status 或 review follow-up 状态 |
| on-complete behavior | 是否影响 workflow 终止指令、completion hook 或后续建议 |
| project context | 是否需要更新 project-level persistent facts 或 Agent default boundary |

Governance-layer batch work with uncertain contract impact must stop for Maxwell confirmation before writes.

## 停止、继续与确认条件

Readiness 的 overall owner decision 只能使用以下值：

| Decision | Meaning | Write permission |
| --- | --- | --- |
| `proceed` | 范围、冲突、恢复和验证路径明确 | 只允许执行记录中命名的 batch action |
| `proceed_with_exclusions` | 允许执行，但排除项必须写明 | 不得触碰 excluded files/scope |
| `clarify_before_write` | 可以继续分析，但写入前必须澄清 | 不允许写入目标集 |
| `defer_to_later_story` | 当前 story 不授权，应转后续 story | 当前不写入 |
| `stop_for_maxwell_confirmation` | 需要 Maxwell 明确确认 | 等确认前不写入 |
| `not_authorized` | 不在当前授权范围内 | 禁止执行 |

Mandatory stop conditions:

- target set uncertain or only described as “related files” / “all relevant docs”;
- conflicting canonical path、topic、title、`doc_id` 或 owner entry point;
- duplicate concept ambiguity that cannot be resolved locally;
- broken links that cannot be repaired within the named scope;
- `docs/index.md` policy conflict, index-wide restructuring, or second index synchronization system;
- lifecycle/status ambiguity, including status conversion without evidence;
- quality gate Hard Fail or equivalent governance Hard Fail;
- unauthorized frontmatter schema change or new global field;
- unauthorized batch status rewrite, batch version normalization, batch rename, batch deletion, batch archive or batch deprecation;
- any need for executable tooling, CLI script, software test, lint/scoring automation, CI, runtime job, package scaffold or source tree;
- governance/workflow/skill contract impact is uncertain;
- Maxwell confirmation is required by an applicable workflow, story, quality gate or current user instruction.

If any mandatory stop condition is present, the Agent must stop before writes, summarize evidence, and request or record the required owner decision.

## 恢复策略与完成证据

Recovery approach must be defined before write actions. A valid recovery approach includes:

- preserve old version or explain how previous file state can be restored;
- avoid overwrite until review unless explicit overwrite authorization exists;
- list affected files before or during completion;
- separate changed files, skipped files and unchanged-reviewed files;
- keep candidate outputs separate from formal `docs/` assets until promotion is approved;
- record rollback/manual restore notes in completion evidence;
- avoid destructive operations unless explicitly authorized.

Batch completion evidence must state results as `通过`、`未通过`、`未验证` or `不适用`. Minimum evidence categories:

| Evidence category | Required conclusion |
| --- | --- |
| Markdown | H1/title, heading hierarchy, table/list formatting and checklist readability |
| frontmatter | required fields, YAML arrays, `doc_id`/title/concept/topic/path/source/time/status consistency |
| index | `docs/index.md` updated, excluded, deferred, or not applicable with reason |
| links | changed-file links, `related_docs`, inbound/outbound or unresolved targets |
| lifecycle/status | lifecycle vocabulary, `quality_status`, review decision and BMad story status remain distinct |
| version governance | no unauthorized prompt/template/quality bump; impact record if semantics changed |
| workflow contract | story checkbox, Dev Agent Record, File List, Change Log, sprint status and review follow-up handling |
| readiness scope | operation type, target set, allowed/excluded files, expected output and non-output recorded |
| conflict exposure | every required conflict category checked with local disposition |
| downstream impact | methodology/governance/template/runbook/prompt/quality/workflow/skill/project-context impact checked when applicable |
| recovery approach | restore/rollback notes and changed/skipped/unchanged-reviewed separation present |
| non-software boundary | no package manifest, `src/`, `tests/`, build config, runtime automation, CLI/API/UI/database/deployment/CI, lint/scoring automation or executable validation tooling introduced |

Story 0.7 itself authorizes creation of this readiness policy only. It does not authorize performing the batch action that a future readiness record may approve.

## 维护触发点

This policy is a Foundation baseline. It must be revisited when later governance stories add stronger schema, records, runbooks or migration rules.

Known maintenance triggers:

- Frontmatter schema、`doc_id`、topic/path/naming、candidate promotion 和 index synchronization rules 更新后，复核 readiness record 是否需要 schema 化或更细的 index sample/review rules。
- Review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更时，复核 readiness record、owner decision 和 conflict evidence 是否应同步。
- Story 4.1 已建立 revision/regeneration continuity policy；后续若该政策更新，复核 recovery approach、old version preservation、batch regeneration 和 Continuity Record stop conditions。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 更新后，复核 sidecar/legacy migration stop conditions。
- Related docs taxonomy、link maintenance、reusable model entry points、existing-doc reuse procedure 或 network boundary / decay prevention policy 更新后，复核 related docs、入链/出链、successor 和 navigation evidence。
- Epic 6 batch governance runbook、batch change review record 和 batch completion report template 落地后，复核本文中的 readiness decision 与具体 batch execution steps 的边界。

This policy should not be maintained by silently expanding scope during unrelated batch work. If future work needs a stronger record format, automation, schema field or runbook step, it must be authorized by the relevant story or Maxwell confirmation.
