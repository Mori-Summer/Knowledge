---
doc_id: governance-docs-asset-governance
title: 正式 docs 资产治理规范：身份、元数据、路径、生命周期、索引、链接与网络边界
concept: docs_asset_governance
topic: governance
depth_mode: deep
created_at: '2026-06-22T09:47:01+08:00'
updated_at: '2026-06-26T00:00:00+08:00'
source_basis:
  - governance_folder_consolidation_2026_06_22
  - frontmatter_schema_doc_merged_2026_06_22
  - lifecycle_states_doc_merged_2026_06_22
  - prompt_template_quality_version_governance_doc_merged_2026_06_22
  - governance_asset_navigation_policy_doc_merged_2026_06_22
  - index_synchronization_rules_doc_merged_2026_06_22
  - topic_path_naming_policy_doc_merged_2026_06_22
  - related_docs_taxonomy_doc_merged_2026_06_22
  - link_maintenance_policy_doc_merged_2026_06_22
  - reusable_model_entry_points_doc_merged_2026_06_22
  - network_boundary_and_decay_prevention_doc_merged_2026_06_22
  - sidecar_boundary_policy_doc_merged_2026_06_22
time_context: governance_source_basis_rule_alignment_2026_06_26
applicability: formal_docs_asset_identity_metadata_path_lifecycle_index_link_and_network_governance
prompt_version: not_applicable
template_version: governance_asset_v2_consolidated
quality_status: consolidated_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-change-governance.md
  - docs/templates/governance-record-templates.md
  - docs/runbooks/batch-governance-runbook.md
open_questions:
  - '是否需要把 `schema_version`、`lifecycle_state`、`owner_entry_point` 等字段升级为正式 frontmatter 字段？'
  - 是否需要为大型批量治理任务建立机器可读的 link/index/frontmatter 检查脚本？
  - '`related_docs`、正文链接、supporting references 与 sidecar 关系是否需要进一步拆成显式字段？'
---

# 正式 docs 资产治理规范：身份、元数据、路径、生命周期、索引、链接与网络边界

## 1. 规范定位

本文是正式 `docs/` 资产的结构治理入口，回答：

- 一个文件如何成为正式 docs 资产
- frontmatter 至少包含哪些字段
- `doc_id`、`concept`、topic、路径、文件名如何保持稳定
- 生命周期、质量状态、版本字段怎样分工
- `docs/index.md`、`related_docs`、正文链接和 sidecar 如何维护
- 如何防止知识网络退化成重复、过密、过期或 catch-all 结构

变更执行、候选晋升、重复合并、迁移、批量治理、最终决策和返工闭环由 [文档治理执行规范](docs-change-governance.md) 负责。

## 2. 适用资产

本文适用于：

- `docs/{topic}/` 下的正式概念文档
- `docs/methodology/` 下的主规范、质量门禁和来源纪律
- `docs/governance/` 下的治理资产
- `docs/templates/`、`docs/runbooks/` 和经批准保留的报告入口
- 从候选稿、对话输出或 `_bmad-output/` 晋升后的正式资产

正式资产必须可追溯、可链接、可索引、可维护。临时计划、story 输出和工作流中间产物不能自动覆盖正式 docs 资产。

## 3. Frontmatter Baseline

正式 docs 资产必须使用 YAML frontmatter + Markdown body。frontmatter 是治理接口，不是装饰。

最低字段：

```yaml
doc_id: stable_document_identity
title: Human-readable title
concept: stable_snake_case_concept_key
topic: topic_or_asset_group
depth_mode: standard_or_deep_or_equivalent_asset_mode
created_at: 'YYYY-MM-DDTHH:MM:SS+08:00'
updated_at: 'YYYY-MM-DDTHH:MM:SS+08:00'
source_basis:
  - source_or_project_rule
time_context: explicit_time_or_project_phase_context
applicability: where_this_asset_applies
prompt_version: prompt_rule_source_or_not_applicable
template_version: structure_or_asset_template_version
quality_status: draft_or_reviewed_or_validated_or_maintained_asset_or_other_supported_status
related_docs: []
open_questions:
  - unresolved_question_or_future_dependency
```

字段规则：

- `source_basis`、`related_docs`、`open_questions` 必须是 YAML array。
- `concept` 使用稳定 `snake_case`，不跟随标题措辞漂移。
- 文件名和 topic 目录使用 `kebab-case`。
- 日期使用带时区的 ISO 形式，无法精确时也必须显式说明时间语境。
- `prompt_version: not_applicable` 比删除字段更好。
- 不得为单个文档临时发明全局字段。

`source_basis` 允许三类值：

- 可定位仓库路径，例如 `docs/methodology/document-generation-methodology.md`、`docs/governance/docs-change-governance.md`。
- 外部来源或核对来源 token，例如 `rfc9293_tcp_base_spec_checked_2026_03_23`、`linux_mmap_manpage_checked_2026_03_19`。
- 合并、迁移或项目内来源 token，例如 `*_doc_merged_YYYY_MM_DD`、`*_folder_consolidation_YYYY_MM_DD`、`docs_folder_consolidation_progress_YYYY_MM_DD`。

当来源实际指向当前仓库内仍存在的规范、治理、模板或 runbook 资产时，优先写成 `docs/.../*.md` 路径。旧文件、已删除文件、候选稿或 `_bmad-output/` 产物只能作为 provenance，不得冒充当前执行入口。

未经明确治理决策，不新增这些字段：

- `schema_version`
- `lifecycle_state`
- `owner_entry_point`
- `navigation_treatment`
- `index_policy_version`
- `quality_gate_version`
- `methodology_version`
- `migration_status`
- `successor`
- `merged_from`
- `duplicate_of`

这些信息应写入正文、review record、completion report 或 change decision evidence。

## 4. 身份规则

`doc_id` 是文档身份，不是文件名的复述。

必须保持稳定：

- 标题调整不自动改变 `doc_id`。
- 路径移动不自动改变 `doc_id`。
- 正文修订不自动改变 `doc_id`。
- 合并、拆分、替换、废弃和归档时，身份连续性由 change governance 记录。

允许新 `doc_id` 的情况：

- 新资产回答新的 canonical concept。
- 拆分后出现独立长期入口。
- 旧资产只保留历史证据，新资产不再是同一身份延续。

不得出现：

- 一个文件多个 `doc_id`
- 多个正式文件竞争同一 canonical `doc_id`
- 仅因标题更好听而更改 `doc_id`

## 5. Topic、Path 与命名

topic 不是标签堆，而是资产归属。

路径规则：

- 概念文档放在最具体且可长期维护的 `docs/{topic}/`。
- 方法论规则放在 `docs/methodology/`。
- 文档治理规则放在 `docs/governance/`。
- 可复用模板放在 `docs/templates/`。
- 执行顺序和操作流程放在 `docs/runbooks/`。
- 报告和审计记录只有在确实需要长期保留时进入正式报告路径。

文件名规则：

- 使用 `kebab-case`。
- 体现 canonical concept，不追求完整标题复述。
- 不用 `new`、`final`、`v2`、`latest` 作为长期文件名。
- 不把多个无关概念塞进一个 catch-all 文件。

同一 topic 下多文档共存必须能说明：

- concept 不同
- reader question 不同
- reuse context 不同
- 维护边界不同

## 6. 生命周期与质量状态

生命周期、质量状态、review decision、story status 是四类不同信号。

常见 `quality_status` 语义：

| 状态 | 含义 |
|---|---|
| `draft` | 尚未完成正式审查 |
| `reviewed` | 经过适用治理检查，可作为当前入口 |
| `validated` | 已有更强验证证据 |
| `maintained_asset` | 作为长期维护的治理/方法论资产 |
| `consolidated_v1` | 已完成同类旧文档合并 |
| `deprecated` | 不再作为当前入口，但可保留历史 |
| `archived` | 仅保留审计或历史用途 |

规则：

- 不用 `quality_status` 表示 BMad story 的 done/in-progress。
- 不用 lifecycle 词掩盖缺失证据。
- 废弃和归档必须说明 successor、剩余访问方式和 index/link 影响。
- `validated` 必须有明确验证证据，不能只因文字完整就使用。

## 7. Prompt、Template 与质量规则版本

`prompt_version` 表示生成或维护该资产时依据的 prompt / rule source。

`template_version` 表示结构模板或资产模式。

版本变更原则：

- 规则语义变化才需要新版本。
- 普通措辞修订不等于模板版本变化。
- 旧文档可以渐进迁移，不要求一次性批量重写。
- 版本字段必须与正文说明和实际结构一致。

## 8. `docs/index.md` 规则

`docs/index.md` 是正式导航入口，不是全库文件清单。

必须列入 index：

- 当前 canonical 概念入口
- 主方法论、质量门禁、来源纪律
- 当前治理入口、模板入口、runbook 入口
- 需要用户或 AI 直接检索的长期资产

通常不列入 index：

- 被合并的旧文档
- 重复入口
- sidecar / 补充材料
- 中间报告、临时输出、候选稿
- 只作为 source_basis 的历史记录

index 变更必须说明：

- target path
- target section
- action: add / update / remove / no-op
- reason
- validation result
- unresolved risk

## 9. 链接与 Related Docs

链接必须有意义，不只是为了“看起来有关”。

链接类型：

| 类型 | 用途 |
|---|---|
| prerequisite | 读本文前需要的前置模型 |
| successor | 本文的后续或替代入口 |
| adjacent | 相邻但不重复的概念 |
| contrast | 用于区分边界的对照概念 |
| supporting | 支撑材料、模板、runbook 或证据 |
| historical | 历史记录或审计来源 |

规则：

- `related_docs` 只放长期有用的少量关系。
- 正文链接要解释迁移对象或边界，不堆链接。
- 删除或合并文件时必须检查入链和出链。
- 一路互相引用但没有清晰关系类型，说明网络已经退化。

## 10. Sidecar 与补充材料

主文档保存 canonical truth。sidecar 只能保存：

- 大样例
- 证据表
- 批量记录
- 附录材料
- 不适合塞进主文档但需长期保留的支持信息

sidecar 不得成为平行权威。主文档变更后，必须检查 sidecar 是否过期。

## 11. 网络健康

知识网络退化信号：

- 多个文件回答同一 reader question。
- `related_docs` 过密且没有关系类型。
- index 列出过多旧入口。
- 同一 topic 变成 catch-all。
- 支撑材料被误当 canonical 入口。
- 文档引用已经删除或移动的旧路径。

处理优先级：

1. 先确认 canonical entry。
2. 合并或删除重复入口。
3. 保留真正相邻、前置、对照和支撑关系。
4. 更新 index。
5. 再做链接和 related_docs 收敛。

## 12. 验证清单

维护正式 docs 资产后，至少验证：

- frontmatter YAML 可解析。
- `source_basis`、`related_docs`、`open_questions` 是 array。
- Markdown 链接目标存在。
- `docs/index.md` 指向存在文件。
- 删除文件没有活跃旧路径引用。
- 标题、H1、`concept`、文件名和 topic 不互相冲突。
- 新增或保留入口没有与现有文档重复。
- `git diff --check` 无空白错误。
