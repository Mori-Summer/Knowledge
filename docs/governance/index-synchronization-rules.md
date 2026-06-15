---
doc_id: governance-index-synchronization-rules
title: docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求
concept: index_synchronization_rules
topic: governance
depth_mode: standard
created_at: '2026-05-26T11:48:58+08:00'
updated_at: '2026-06-15T17:21:39+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/2-4-index-synchronization-rules.md
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/concept-document-contract.md
  - _bmad-output/implementation-artifacts/deferred-work.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_key_draft_review_2026_06_15
applicability: docs_index_synchronization_and_navigation_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/concept-document-contract.md
open_questions:
  - rename、migration、split、merge、deprecation、successor 和 link-impact policy 已建立后，是否需要把本文中的 rename/move/deprecation 处理改为更细的迁移记录入口？
  - Story 2.6 已建立 duplicate/coexistence policy 后，是否还需要为 secondary reference 和 duplicate-entry exception 增加更细决策表？
  - 如果 completion-report template 后续调整 index impact summary 字段，是否需要同步 Index Impact Decision Record 的汇总方式？
  - Story 5.1 related docs taxonomy 与 Story 5.2 link maintenance policy 已建立；后续是否需要把 owner entry、supporting link、related doc、successor link 和 index entry 的维护关系拆分得更细？
  - Epic 6 建立 batch governance runbook 后，是否需要为批量索引 sweep、生成式索引重写和多 topic cleanup 定义单独执行记录？
---

# docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 `docs/index.md` 同步与正式导航入口治理的 canonical governance asset。它回答的是：正式 `docs/` 资产新增、晋升、移动、重命名、改标题、改 topic、废弃、归档或仅正文修订时，`docs/index.md` 应该如何检查、更新、排除、延后或记录不适用证据。

本文适用于：

- `docs/{topic}/` 下的普通 formal knowledge assets。
- `docs/methodology/` 下的方法论、模板、质量门禁、fixed prompt、playbook 和方法论支撑资产。
- `docs/governance/` 下的治理资产。
- `docs/_reports/` 下已经批准保留的正式报告入口。
- 已批准的 `docs/templates/` 资产，以及未来明确批准的 `docs/runbooks/` 资产。
- index-only update、candidate promotion 的 index impact review、正式资产生命周期变更的导航影响检查，以及完成汇报中的索引证据记录。

本文补充而不替代以下资产：

- 主方法论 `docs/methodology/document-generation-methodology.md`：仍负责概念文档新建、升级、审查和仓库集成的主执行路径。
- Story 2.1 frontmatter schema `docs/governance/frontmatter-schema.md`：仍负责 required fields、YAML array、`doc_id` 身份和 frontmatter/body 一致性。
- Story 2.2 topic/path/name policy `docs/governance/topic-path-naming-policy.md`：仍负责 topic、路径、文件名、asset class 和 path ownership。
- Story 2.3 candidate promotion checklist `docs/governance/candidate-promotion-checklist.md`：仍负责候选来源进入正式 `docs/` 前的 promotion gate。
- Story 0.6 navigation policy `docs/governance/governance-asset-navigation-policy.md`：仍负责 governance/template/runbook asset 的 owner entry point 和 navigation treatment 基线；本文细化 `docs/index.md` 同步执行与证据要求。
- 生命周期、批量 readiness、intake 和质量门禁资产：仍分别负责 lifecycle、batch、task routing 和 Hard Fail/等价治理检查。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/index-synchronization-rules.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：index synchronization 触发器、索引条目格式、Index Impact Decision Record、相邻治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

本文只创建 index synchronization governance asset，并为该资产同步 `docs/index.md` 入口。本文不替代 rename/migration policy、duplicate/coexistence policy、link-maintenance policy、review/decision/completion templates 或 Epic 6 batch assets，不执行 index-wide restructuring、批量排序、批量路径修复、生成式索引系统或 executable index tooling。

## `docs/index.md` 的职责边界

`docs/index.md` 是正式仓库导航入口，不是身份、质量、生命周期或来源事实的唯一 source of truth。正式资产的身份由 `doc_id`、文件路径、frontmatter、正文角色说明、来源依据和相关治理记录共同支撑；索引只承担发现、导航和分组信号。

`docs/index.md` 必须反映正式资产的当前可发现入口：

- 普通 formal knowledge assets 应按主题分组列入对应 section。
- 方法论资产应列入 `## methodology`，除非由更具体的正式 entry point 受控引用。
- 治理资产应列入 `## governance`，除非有明确 `referenced_from_governance_entry` 或 `intentionally_excluded` 证据。
- 正式 reports 使用受控 `## reports` 入口和 `docs/_reports/` 路径，不伪装为普通概念 topic。
- 已批准的正式模板资产使用 `## templates` section 和 `docs/templates/` 路径；未来 runbooks 只有在目录和入口被明确批准后，才建立对应 section 或受控引用路径。

`docs/index.md` 不得直接索引以下内容：

- `_bmad-output/` 中的 PRD、Architecture、Epics、Story、readiness report、review output、completion record 或草稿。
- 对话输出、临时 Markdown、未晋升 candidate、工作流中间产物、review finding、story file 或 sprint tracking record。
- 尚未完成 target path、frontmatter、source/time context、quality/equivalent governance gate、owner decision 和 index impact 判断的候选资产。

这些非正式内容可以作为 `source_basis`、candidate evidence、review evidence 或 story evidence，但不会因为被引用、被审查或出现在工作流中就成为 `docs/index.md` 的正式导航入口。

## 同步触发器

任何影响 title、H1、path、topic、lifecycle/status navigation 或 primary discoverability 的目标文件变更，必须在同一变更中检查 `docs/index.md`。检查结果必须进入 completion evidence、review evidence、story Dev Agent Record、Epic 3 completion report 或 future runbook record；不得沉默跳过。

通常需要更新 `docs/index.md` 的触发器如下：

| 触发器 | 必须检查什么 | 默认索引动作 |
| --- | --- | --- |
| new formal asset | 目标是否已是正式 `docs/` 资产，title/path/topic/asset class 是否明确 | 新增正确 section 下的相对链接，或记录受控非索引理由 |
| candidate promotion | promotion gate 是否通过，target path/frontmatter/source/status 是否成立 | 新增正式 target 的入口；不得索引 candidate source |
| rename | 旧路径或旧 filename 是否仍在索引中 | 更新路径；移除同一资产的旧入口 |
| move | section、relative path、topic/asset class 是否改变 | 更新路径和分组；不得留下旧路径入口 |
| retitle / H1/title display change | index display title 是否会误导读者或 Agent | 更新 display title，或记录不改索引的理由 |
| topic/grouping change | frontmatter `topic`、目录和 index section 是否一致 | 移动到正确 section；记录 rejected grouping |
| deprecation | 是否仍应普通导航、标注废弃、转入 successor 或降低可见性 | 按生命周期证据处理；详细 successor/link 由 Story 2.5 |
| archive | 是否保留普通入口、转入 report/archive 入口或受控排除 | 记录 remaining access expectation；详细归档迁移由 Story 2.5 |
| successor/replacement navigation change | primary entry point 是否切换 | 更新导航入口或 `deferred_with_reason`；link/successor 细节由 Story 2.5 |
| formally retained report entry | 报告是否是受控正式报告，而非 workflow output | 保留在 `## reports`；路径使用 `./_reports/...` |

通常需要 index impact review，但不一定需要修改 `docs/index.md` 的触发器如下：

| 触发器 | 允许不改索引的前提 | completion evidence |
| --- | --- | --- |
| body-only revision | title、H1、path、topic、asset class、status/navigation 和 primary discoverability 未变 | `index impact: not_applicable`，说明正文修订不影响导航 |
| non-navigation frontmatter correction | 修正不改变 title/topic/path/status/navigation semantics | `not_applicable` 或 `deferred_with_reason`，说明字段不影响索引 |
| source/time update | 只补来源、核对日期或 time context，不改变入口标题/分组 | `not_applicable`，说明索引不变 |
| quality status note without navigation change | 质量说明未改变使用入口、废弃/归档语义或主要导航定位 | `not_applicable`；若影响可见性则必须处理索引 |
| narrow link/body repair | 只修正文或局部链接，未改目标身份和路径 | `not_applicable`，并记录链接验证结果 |

Index review 与 index edit 是两件事。每次相关变更都必须分类 index impact；只有分类结果为 `updated` 时才实际修改 `docs/index.md`。

## 索引条目格式与分组

`docs/index.md` 条目使用 Markdown 列表链接：

```markdown
- [Human-readable display title](./relative/path-from-index.md)
```

条目必须同时满足：

- display title 对人类可读，且不与目标 H1/frontmatter `title` 表达不同对象。
- link path 是从 `docs/index.md` 出发的相对路径，通常以 `./` 开头。
- link target 指向存在的正式 `docs/` 资产或明确保留的正式 report entry。
- section grouping 与 target path、frontmatter `topic`、asset class 和 reader expectation 一致。
- 同一正式资产默认只有一个 canonical entry。
- display title 不应与其他 index entry 完全相同；如果两个正式资产确实需要相同短标题，必须在 display title 中加入 topic、asset class、限定语或其他人类可读 disambiguator。若无法解释差异，转向 duplicate/coexistence decision，而不是保留两个同名导航入口。

分组规则如下：

| 资产类别 | 默认 section | 默认 path | 分组要求 |
| --- | --- | --- | --- |
| ordinary formal knowledge asset | 对应 topic section | `./{topic}/{slug}.md` | `topic`、目录和 section 必须一致 |
| methodology asset | `## methodology` | `./methodology/{slug}.md` | 方法论、方法论模板/概念文档模板、质量门禁、fixed prompt、playbook 或方法论支撑资产 |
| governance asset | `## governance` | `./governance/{slug}.md` | 治理政策、Agent 约束、生命周期、frontmatter、topic/path、promotion、index sync 等资产 |
| formal report/archive entry | `## reports` | `./_reports/{slug}.md` | `docs/_reports/` 是受控特殊目录，可对应 `topic: reports`，不得重分类为普通 topic |
| formal template asset | `## templates` | `./templates/{slug}.md` | 已批准模板资产按模板 section 索引；候选模板仍需 promotion/index impact evidence |
| future runbook asset | future `## runbooks` 或受控 entry point | `./runbooks/{slug}.md` | 只有后续 story 或 Maxwell 明确批准后使用 |

Index title、target H1、frontmatter `title`、target path、frontmatter `topic` 和 section grouping 不必逐字相同，但不得误导读者或 Agent。允许 display title 为了可读性略作压缩；不允许把一个治理资产放到 ordinary topic section，或把 report/archive entry 当成普通概念文档。

重复入口默认禁止。同一正式资产只有在明确治理理由和 owner entry point 支持时，才允许 secondary reference。Secondary reference 必须说明：

- 为什么 canonical entry 不足以发现或理解该资产。
- secondary reference 的上级入口是什么。
- 是否会制造平行主入口、重复权威或同一资产多处更新风险。
- 后续何时复核或移除该 secondary reference。

## 资产类别边界

Index synchronization 必须先区分资产类别，再决定是否和如何索引。

| 类别 | 是否可直接进入 `docs/index.md` | 处理规则 |
| --- | --- | --- |
| 普通 formal knowledge asset | 可 | 按 topic section 列入，除非有受控排除理由 |
| methodology asset | 可 | 按 `## methodology` 列入或由主方法论受控引用 |
| governance asset | 可 | 默认按 `## governance` 列入；从属细则可受控引用 |
| formal report/archive asset | 可，受控 | 只使用 `## reports` 和 `docs/_reports/` 路径，不重分类为普通 topic |
| formal template asset | 可 | 按 `## templates` 列入；候选模板、review output 或 completion record 不因格式像模板就自动进入索引 |
| future runbook | 未默认开放 | 等后续 story 或 Maxwell 批准目录和入口后再索引 |
| `_bmad-output/` workflow artifact | 不可 | 只能作为来源、候选或 evidence；不得直接索引 |
| candidate / temporary draft | 不可 | promotion gate 通过前不得索引 |
| review output / completion record | 不可 | 作为 evidence 保留，不是正式导航入口 |
| conversation output | 不可 | 可作为用户上下文或候选来源，不是正式资产 |

`referenced_elsewhere`、`intentionally_excluded` 和 `not_applicable` 不是“漏做索引”的同义词：

- `not_applicable` 表示本次变更没有 index impact，或目标不是应纳入正式导航的资产，并且理由已记录。
- `referenced_elsewhere` 表示资产未直接列入总索引，但由明确的正式 methodology/governance/template/runbook entry point 引用；必须能定位引用点。
- `intentionally_excluded` 表示资产被受控排除；必须记录 exclusion reason、owner entry point、remaining access expectation 和 re-review trigger。

如果无法解释为什么不更新索引，不能把缺口伪装成上述状态。应使用 `deferred_with_reason` 或 blocking state，并说明具体 blocker。

## Index Impact Decision Record

每次相关变更必须记录 index impact。记录可以写在正文、review evidence、completion evidence、story Dev Agent Record、`docs/templates/completion-report-template.md` 的 completion report skeleton 或 future runbook record 中；不得新增 `index_policy_version`、`navigation_treatment`、`decision_status` 等全局 frontmatter 字段。

允许的 outcome 值如下：

| outcome | 使用条件 |
| --- | --- |
| `updated` | `docs/index.md` 已新增、修改、移动或删除相关入口，并更新必要 metadata |
| `not_applicable` | 本次变更不影响正式导航；必须说明原因 |
| `referenced_elsewhere` | 未直接列入总索引，但由明确正式入口引用 |
| `intentionally_excluded` | 受控排除；记录排除理由、owner entry point、remaining access expectation 和 re-review trigger |
| `deferred_with_reason` | 当前 story 不授权处理，或需要 future story/owner decision；记录风险和后续入口 |
| `blocked_index_policy_conflict` | 存在 unresolved index-policy conflict，继续写入会制造错误导航或错误权威；必须停止或请求 Maxwell 确认 |

`blocked_pending_story_2_4` 是 Story 2.4 落地前的临时占位状态。本文存在后，新的记录不应再使用 `blocked_pending_story_2_4`。未解决问题必须命名具体 blocker 或 future owner，例如 `blocked_index_policy_conflict`、`deferred_with_reason: pending Story 2.5 successor handling`、`deferred_with_reason: pending duplicate/coexistence decision under Story 2.6 policy` 或 `hold_for_maxwell_confirmation`。

最低记录字段如下：

| 字段 | 必须记录什么 |
| --- | --- |
| affected file | 被新增、修改、移动、重命名、废弃、归档或检查的正式资产 |
| target title | 目标 H1/frontmatter title 或 index display title |
| target path | `docs/index.md` 中应使用的相对路径或不适用理由 |
| target section | `docs/index.md` 的 section，或受控 entry point |
| outcome | 上表允许值之一 |
| action taken or not taken | 实际新增、更新、删除、保留、排除或延后动作 |
| reason | 为什么这样处理 |
| validation result | target exists、relative link resolves、no stale entry、not applicable reason 等 |
| unresolved risk | `none` 或明确命名风险和后续 owner |

本文自身的 index impact decision record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/index-synchronization-rules.md
- target title: docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求
- target path: ./governance/index-synchronization-rules.md
- target section: docs/index.md ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: new canonical governance asset for docs/index.md synchronization
- validation result: target exists and relative link resolves
- unresolved risk: none
```

## Index-only update rules

`index-only update` 只授权处理 `docs/index.md` 的条目和必要 metadata。它不得被用来暗中修改目标资产、晋升候选稿、迁移路径、改变 lifecycle/status，或执行批量重构。

Index-only update 可以做：

- 新增、修改、移动或删除一个明确正式资产的索引条目。
- 更新 `docs/index.md` frontmatter 中与本次索引正文变更一致的 metadata，例如 `updated_at` 和 `time_context`。
- 验证目标文件存在、relative path 可解析、section grouping 与 target topic/asset class 一致。
- 在 completion evidence 中记录 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 blocking state。

Index-only update 不得做：

- 修改 target asset 正文或 target asset frontmatter，除非 active story/workflow 明确授权。
- 修改方法论规则、治理规则、模板、workflow output、sprint status 或 story records，除非当前任务本身授权。
- 把 `_bmad-output/`、conversation output、review output、story file、completion record 或 candidate 直接加入正式索引。
- 通过新增索引入口制造 hidden promotion。
- 执行 broad link migration、bulk sorting、index-wide restructuring、generated index rewrite 或 multi-topic cleanup。

新增索引入口前必须确认：

1. target asset 已存在且位于正式 `docs/` 路径。
2. target asset 不是未晋升 candidate、workflow output 或临时草稿。
3. target title/H1/frontmatter `title` 不与 index display title 表达不同对象。
4. target path 与 frontmatter `topic`、asset class 和 section grouping 不互相误导。
5. link target 存在且 relative path 从 `docs/index.md` 可解析。
6. 不会留下同一正式资产的 stale old path/title entry。
7. 不会制造 duplicate display title；若 display title 与现有 entry 相同，必须先 disambiguate 或转向 duplicate/coexistence decision。

如果当前任务是 body-only revision、narrow metadata correction、source/time update 或 narrow link/body repair，并且不修改索引，完成证据必须写明 `index impact: not_applicable` 以及不适用理由。沉默跳过不是有效证据。

必须 stop/defer 的情况：

- requested index update 实际需要 candidate promotion。
- requested index update 隐含 rename/migration、duplicate/coexistence、successor、deprecation、archive 或 broad link migration。
- target path/title/topic/asset class 不清。
- target link 不存在。
- index update 会变成 batch sweep、generated rewrite、section reorganization、bulk sorting、bulk path repair 或 multi-topic cleanup。
- 需要创建、重写或提前执行相邻治理资产、review/completion templates、Epic 5 reuse/network governance 或 Epic 6 batch assets。

## 验证清单

编辑 `docs/index.md` 或新增正式资产后，至少检查：

1. `docs/index.md` frontmatter 仍存在 required fields，`source_basis`、`related_docs` 和 `open_questions` 仍是 YAML arrays。
2. `docs/index.md` H1 保持稳定，section headings 未被误删、误重命名或批量重排。
3. 新增或修改的 index entry 使用 Markdown list link，display title、relative path 和 section grouping 正确。
4. 每个 changed/new entry target 文件存在。
5. Relative link 从 `docs/index.md` 出发可以解析。
6. 新增正式资产的 target H1/frontmatter title、path、topic、asset class 和 index section 不互相误导。
7. Changed/new entry 没有重复 display title；如需相近标题，已用 topic、asset class 或限定语消歧。
8. Rename、move、retitle 或 topic change 范围内没有 stale old path/title entry。
9. 没有把 `_bmad-output/`、conversation output、temporary draft、story file、review output 或 completion record 加入正式索引。
10. 没有把 `docs/_reports/` 重分类为普通 topic；正式 reports 保留 controlled special directory 语义。
11. 没有未经授权新增 schema fields、index schema、generated index system、lint/scoring tool、software tests、package manifest、`src/`、build config、runtime automation、CLI/API/UI/database/deployment/CI artifact。
12. 如果没有修改索引，completion evidence 说明了 `not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 blocking state。
13. 如果触及多个文件、多个 topics、多个 index entries 或由规则选出目标集，已先满足 `docs/governance/batch-readiness-checklist.md`。

验证结果必须可审查。可接受表述是 `通过`、`未通过`、`未验证`、`不适用`，并附具体证据位置或理由。

## 停止条件与后续归属

遇到以下情况必须停止、延后或请求 Maxwell 确认：

- target file 不存在，或不是正式 `docs/` 资产/受控正式 report entry。
- target 仍是 candidate、conversation output、`_bmad-output/` workflow artifact、review output、story file 或 temporary draft。
- title、H1、path、topic、asset class、owner entry point 或 navigation treatment 互相冲突。
- 当前任务没有授权 promotion、rename/migration、duplicate/coexistence、successor/deprecation/archive、broad link migration 或 batch work。
- Index action 会制造平行主入口、重复 canonical entry、错误 section 或误导性 display title。
- 需要批量索引 sweep、generated index rewrite、section reorganization、bulk sorting、bulk path repair 或 multi-topic cleanup，但没有 batch readiness 和 owner decision。
- 需要新增 executable tooling、lint/scoring automation、machine-readable index schema、software tests、CI 或部署检查。

后续归属边界：

- `docs/governance/rename-migration-policy.md` 负责 rename、migration、split、merge、deprecation、successor、old-content handling 和 link-impact policy。
- `docs/governance/duplicate-and-coexistence-policy.md` 负责 duplicate concept、same-topic coexistence 和 secondary entry exception 的完整决策模型；本文只调用其 outcome，不复制该模型。
- Review record template、document decision policy、rework loop 和 completion report template 负责 review/decision/completion evidence。
- Epic 5 related docs taxonomy、link maintenance、reusable model entry points、existing-doc reuse procedure 和 network boundary / decay prevention governance 负责相邻关系、链接维护、复用入口和网络退化路由。
- Epic 6 负责 batch governance runbook、batch review record 和 batch completion report。

本文可以引用这些相邻 owners，但不创建对应资产、不替代其完整范围，也不把未来策略提前写成当前可执行迁移规则。

## 维护触发点

以下变化要求复核本文：

- Rename/migration/split/merge/deprecation/successor policy 更新。
- Story 2.6 已建立 duplicate/coexistence policy；后续若该政策更新，复核本文的 secondary reference 和 duplicate-entry exception 边界。
- Epic 3 review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更。
- Epic 5 related docs taxonomy、link maintenance policy、reusable model entry points 或后续 reuse/network governance 更新。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report。
- Maxwell 明确授权 generated index pages、machine-readable index schema、executable validation tooling、lint/scoring automation、batch normalization 或 index-wide restructuring。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [Topic、文件命名与路径归属策略：正式 docs 资产的位置、命名与一致性规则](./topic-path-naming-policy.md)
- [候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁](./candidate-promotion-checklist.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](./agent-behavior-constraints.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](../methodology/governance-asset-boundary-policy.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](../methodology/concept-document-contract.md)
