---
doc_id: governance-docs-change-governance
title: 文档治理执行规范：复用扫描、晋升、合并、迁移、修订、批量、决策与返工闭环
concept: docs_change_governance
topic: governance
depth_mode: deep
created_at: '2026-06-22T09:47:01+08:00'
updated_at: '2026-06-26T00:00:00+08:00'
source_basis:
  - governance_folder_consolidation_2026_06_22
  - agent_behavior_constraints_doc_merged_2026_06_22
  - batch_readiness_checklist_doc_merged_2026_06_22
  - candidate_promotion_checklist_doc_merged_2026_06_22
  - document_decision_policy_doc_merged_2026_06_22
  - duplicate_and_coexistence_policy_doc_merged_2026_06_22
  - existing_doc_reuse_procedure_doc_merged_2026_06_22
  - legacy_migration_guide_doc_merged_2026_06_22
  - rename_migration_policy_doc_merged_2026_06_22
  - revision_regeneration_continuity_policy_doc_merged_2026_06_22
  - rework_loop_examples_doc_merged_2026_06_22
time_context: governance_control_plane_link_alignment_2026_06_26
applicability: docs_change_execution_reuse_scan_promotion_merge_migration_revision_batch_decision_and_rework_governance
prompt_version: not_applicable
template_version: governance_asset_v2_consolidated
quality_status: consolidated_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-asset-governance.md
  - docs/templates/governance-record-templates.md
  - docs/runbooks/batch-governance-runbook.md
open_questions:
  - 批量治理是否需要独立机器可执行 readiness/checkpoint schema？
  - 重生成与定向修订的边界是否需要按资产类型分别细化？
  - 是否需要为“旧内容删除而非重定向保留”建立更严格的审计记录格式？
---

# 文档治理执行规范：复用扫描、晋升、合并、迁移、修订、批量、决策与返工闭环

## 1. 规范定位

本文是正式 docs 变更执行入口，回答：

- AI 在文档治理任务开始前必须判断什么
- 新文档或候选稿怎样先复用现有文档，避免重复生成
- 什么时候晋升、合并、拆分、迁移、废弃、归档或拒绝
- 定向修订与重生成怎样分界
- 批量治理何时可以继续、何时必须停
- final decision 和返工闭环需要哪些证据

资产结构、frontmatter、路径、生命周期、index、links 和网络健康由 [正式 docs 资产治理规范](docs-asset-governance.md) 负责。

## 2. Agent 执行约束

任何文档治理任务开始前，先声明：

- 目标路径或目标文件夹
- 操作类型：新增、升级、合并、删除、迁移、索引更新、链接修复、批量治理
- 资产层级：概念文档、方法论、治理、模板、runbook、报告、候选稿
- 允许修改范围
- 明确排除范围
- 验证方式

禁止：

- 未做复用扫描就新建同类文档
- 未说明 successor / old content handling 就删除正式资产
- 未检查 index/link impact 就移动或删除文件
- 用 review decision 代替实际修订
- 修改 `.agents/skills/`、工作流资产或软件代码来绕过文档治理问题
- 为单次变更发明新的全局 frontmatter 字段

## 3. Existing Doc Reuse Scan

新建或晋升前必须先扫描现有 docs。

扫描结果类型：

| 结果 | 处理 |
|---|---|
| `exact_match` | 复用现有文档，不新建 |
| `near_duplicate` | 合并、窄化或拒绝 |
| `adjacent` | 添加有意义链接，不合并 |
| `prerequisite` | 引用为前置模型 |
| `contrast` | 引用为边界对照 |
| `gap` | 可以新建或补充，但要说明缺口 |

复用证据至少写明：

- 搜索范围
- 命中的现有文档
- 是否存在 canonical entry
- 新内容为何不能并入既有文档
- index 和 related_docs 影响

## 4. Duplicate / Coexistence Decision

重复和共存判断不能只看标题相似度。

必须比较：

- concept
- reader question
- reuse context
- scope
- canonical authority
- maintenance boundary
- index role

允许 outcome：

| outcome | 使用条件 |
|---|---|
| `merge` | 多个文档回答同一核心问题，独立保留会形成平行权威 |
| `narrower_document` | 内容可保留，但必须缩窄为下位或更具体问题 |
| `adjacent_link` | 概念相邻但不重复 |
| `explicit_coexistence` | 同 topic 下长期共存且边界清晰 |
| `reject_duplicate_or_misplaced` | 候选没有独立价值或路径归属错误 |
| `defer_with_reason` | 当前授权不足或需后续 owner decision |

合并时优先保留最稳定、最常被引用、最能承载 canonical concept 的入口。

## 5. Candidate Promotion

候选稿晋升为正式 docs 资产前必须满足：

- 有明确 target path。
- 有稳定 `doc_id`、`concept`、title、topic。
- 通过适用 frontmatter 和质量检查。
- 有来源依据和时间语境。
- 不与现有文档重复。
- `docs/index.md` 处理清楚。
- old content、successor、related docs、inbound/outbound links 不悬空。

不允许 silent promotion：

- 不能直接把 `_bmad-output/` 当正式 docs。
- 不能把对话输出直接塞入正式目录。
- 不能覆盖正式资产而不记录身份和迁移影响。

## 6. 变更类型与身份连续性

| 变更类型 | 规则 |
|---|---|
| `targeted_revision` | 原身份保持，修订局部结构、证据、链接或措辞 |
| `regeneration` | 内容整体重写，但需说明是否继承 `doc_id` |
| `rename_or_move` | 路径变化不自动改变 `doc_id` |
| `merge` | 选择 canonical successor，处理旧内容和链接 |
| `split` | 说明哪些内容留在原文，哪些成为新身份 |
| `deprecate` | 不再作为当前入口，但可保留历史或 successor |
| `archive` | 只保留审计或历史用途，通常不在 index 主入口 |
| `delete` | 仅在内容已被吸收、无保留价值或用户明确要求时执行 |

删除旧文档时必须确认：

- 关键内容已进入 successor。
- index 不再指向旧路径。
- 活跃 Markdown 链接已更新。
- 保留历史记录只存在于报告或 source_basis，不作为可点击正式入口。

## 7. Revision vs Regeneration

选择定向修订，当：

- 文档身份正确
- 结构基本可用
- 缺口可定位
- 来源或链接问题局部可修

选择重生成，当：

- 文档类型错了
- 重复和平行权威严重
- 结构被旧模板污染
- 来源纪律大面积不可恢复
- 局部修补会比重写更难维护

重生成不得无差别覆盖高价值旧内容。需要先列出保留内容、删除内容和 successor 归属。

## 8. Batch Readiness

批量治理必须先判断 readiness。

批量治理包括：

- 多文件合并 / 删除
- 多 topic 重命名
- 大范围 index 更新
- 大范围 link 修复
- 同类 frontmatter 迁移
- 模板或治理规则批量替换

Readiness record 最少包含：

- operation type
- target set
- excluded files
- expected output
- non-output
- conflict summary
- owner decision
- stop conditions
- recovery approach
- validation plan

Readiness record 通过后，批量执行顺序、样本审查、失败暂停、恢复证据和完成证据使用 `docs/runbooks/batch-governance-runbook.md`。记录形状使用 `docs/templates/governance-record-templates.md`，除非当前任务或 owner 没有授权写入报告文件。

必须停止的情况：

- target set 不清
- 发现用户未授权的高风险删除
- 多个 canonical successor 竞争
- link/index impact 无法收敛
- 需要改 `.agents/skills/` 或工作流规则但未授权
- 校验失败且无法在当前范围内修复

## 9. Final Decision Labels

最终决策必须使用明确 label，不写“差不多可以”。

| label | 含义 |
|---|---|
| `accept_promote` | 可正式晋升或作为当前正式入口 |
| `accepted_for_current_use` | 当前范围可用，但不声明更强状态 |
| `accepted_with_nonblocking_follow_up` | 可用且有非阻塞后续项 |
| `targeted_revision` | 需要可定位修订 |
| `regenerate` | 需要重生成或整体重写 |
| `reject_duplicate_or_misplaced` | 重复、错置或不可晋升 |
| `deprecate_or_archive` | 进入废弃或归档处理 |
| `hold_for_clarification` | 需要 owner / Maxwell 确认 |
| `defer_with_reason` | 当前范围不处理，但记录原因 |
| `not_authorized` | 越过授权边界，必须停止 |

决策证据必须说明：

- target path
- applicable rules
- blocker / non-blocker
- link/index impact
- status impact
- next action

## 10. Rework Loop

返工不是“再改改”，必须明确失败类型和修复路径。

失败类型：

- schema/frontmatter failure
- source/currentness failure
- duplicate/coexistence failure
- path/topic failure
- index/link failure
- lifecycle/status failure
- structure/template failure
- authorization boundary failure

返工记录至少包含：

- failure category
- evidence location
- repair instruction
- regenerate rationale, if needed
- owner
- resubmission checks

同一失败反复出现时，不继续盲改；应升级为 hold、defer 或 not_authorized。

## 11. Legacy Migration

旧文档处理原则：

- 有价值内容并入 successor。
- 旧路径不默认保留 redirect stub。
- 历史报告可以保留旧路径记录，但不能作为当前入口。
- 旧文档若只造成重复、错链和索引噪音，应删除。
- 如果旧文档仍有审计价值，标记为 report/archive，而不是保留在正式 topic 入口。

批量旧文档迁移优先顺序：

1. 找 canonical successor。
2. 合并独有内容。
3. 更新 index。
4. 更新活跃链接。
5. 删除旧文件。
6. 验证没有活跃旧路径引用。

## 12. 验证清单

每次治理变更后至少运行：

- 旧路径残留搜索。
- Markdown 链接存在性检查。
- frontmatter YAML 和 array 字段检查。
- `docs/index.md` 目标存在性检查。
- `git diff --check`。

完成汇报必须说明：

- 合并了哪些文件
- 删除了哪些文件
- 保留入口是什么
- 更新了哪些 index/link
- 哪些旧路径只作为历史记录保留
- 哪些风险或后续项仍未解决
