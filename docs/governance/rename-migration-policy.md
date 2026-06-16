---
doc_id: governance-rename-migration-policy
title: 重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响
concept: rename_migration_policy
topic: governance
depth_mode: standard
created_at: '2026-05-26T15:54:04+08:00'
updated_at: '2026-06-16T09:42:19+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/2-5-rename-migration-policy.md
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_epic_2_governance_review_2026_06_16
applicability: formal_docs_rename_migration_split_merge_deprecation_archive_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/topic-path-naming-policy.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/sidecar-boundary-policy.md
  - docs/governance/legacy-migration-guide.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
open_questions:
  - duplicate/coexistence policy 已建立后，是否还需要把身份歧义与同主题共存的交界条件拆成更细判定表？
  - 如果 completion-report template 后续调整 Migration Decision Record summary 字段，是否需要同步本文的 Migration Decision Record 汇总方式？
  - related docs taxonomy 与 link maintenance policy 已建立；后续是否需要进一步细分 successor link、replacement link、supporting link 和 owner entry 的维护关系？
  - Epic 6 建立 batch governance runbook 后，是否需要为批量迁移、批量链接修复和批量废弃建立独立执行记录？
---

# 重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中 formal `docs/` 资产发生 rename、move/path migration、retitle、topic migration、revision、replacement、split、merge、successor change、deprecation、archive 或 reactivation 时的 canonical governance asset。它回答的是：什么时候必须保留稳定 `doc_id`，什么时候可能产生新身份，旧内容如何处理，继任或替代入口如何记录，链接、`related_docs`、`docs/index.md` 和 lifecycle/status 影响如何检查，以及什么时候必须停止或转入 batch readiness。

本文适用于：

- `docs/{topic}/` 下的普通正式概念/知识资产。
- `docs/methodology/` 下的方法论、模板、质量门禁、fixed prompt、playbook 和方法论支撑资产。
- `docs/governance/` 下的治理资产。
- `docs/_reports/` 下已经批准保留的正式报告/归档入口。
- `docs/index.md` 与未来被明确批准的 root-level 正式导航资产本身；如果根导航资产发生 rename、move、archive、successor 或可见性变化，也必须按本文记录身份、链接和索引影响。
- 已批准的 `docs/templates/` 模板资产，以及未来明确批准的 `docs/runbooks/` 正式资产。
- 正式资产的 replacement、successor、deprecation、archive、split、merge、reactivation 和 migration evidence 判断。

本文补充而不替代以下资产：

- `docs/governance/frontmatter-schema.md`：负责 frontmatter baseline、YAML array、`doc_id` 身份基线和禁止未经授权全局字段。
- `docs/governance/topic-path-naming-policy.md`：负责 topic、path、filename、slug、asset class 和 path ownership。
- `docs/governance/candidate-promotion-checklist.md`：负责候选来源进入正式 `docs/` 前的 promotion gate、replacement continuity 和 Promotion Decision Record。
- `docs/governance/index-synchronization-rules.md`：负责 `docs/index.md` 同步触发器、index impact outcome 和 Index Impact Decision Record。
- `docs/governance/lifecycle-states.md`：负责 draft、reviewed、validated、maintained_asset、deprecated、archived 和 reactivation 语义。
- `docs/governance/batch-readiness-checklist.md`：负责批量治理 readiness、范围、冲突、停止条件和恢复策略。
- `docs/governance/governance-asset-navigation-policy.md`：负责治理资产 owner entry point 与 navigation treatment 基线。
- `docs/methodology/intake-and-intent-classification.md` 与 `docs/methodology/concept-document-quality-gate.md`：负责任务意图、停止条件、Hard Fail 和等价治理门禁。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/rename-migration-policy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

本文自身的 index impact decision record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/rename-migration-policy.md
- target title: 重命名、路径迁移与废弃治理：身份连续性、旧内容处理、继任入口与链接索引影响
- target path: ./governance/rename-migration-policy.md
- target section: docs/index.md ## governance
- outcome: reviewed_existing_entry
- action taken: no index edit in this stabilization pass; existing governance entry verified
- reason: canonical governance asset for rename/migration/split/merge/deprecation/archive handling is already listed
- validation result: existing target exists and relative index link resolves
- unresolved risk: index/link resolution risk none; Epic 6 batch execution records remain future dependencies
```

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：rename/migration decision taxonomy、identity continuity、old-content handling、successor/replacement、link/index/lifecycle impact、相邻治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

## 职责边界与非目标

本文定义决策、证据、停止条件和验证规则；它不执行任何实际文档迁移。

本文授权的输出只有本政策资产和本次直接相关的 `docs/index.md` 导航入口更新。它不授权：

- 实际 rename、move、path migration、retitle、topic migration、split、merge、successor switch、deprecation、archive 或 reactivation。
- 广泛链接修复、入链扫描、`related_docs` 批量更新、index sweep、multi-topic cleanup、duplicate/coexistence cleanup 或平行入口清理。
- 批量 frontmatter normalization、批量 lifecycle/status conversion、批量迁移、批量废弃、批量归档或 generated rewrite。
- Review record template、document decision policy、rework loop examples 或 completion report template。
- Duplicate/coexistence policy。
- Related docs taxonomy、link maintenance policy、reusable model entry points、existing-doc reuse procedure 或 network boundary / decay prevention policy。
- Epic 6 batch governance runbook、batch review record 或 batch completion report。
- runtime code、package manifest、source tree、software tests、CLI/API/UI/database/deployment/CI、lint/scoring automation、link scanner、index generator、migration script 或 executable validation tooling。

本文也不创建新的全局 frontmatter 字段。不得为了表达迁移状态或局部不确定性，在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`path_policy_version`、`promotion_policy_version`、`migration_status`、`quality_gate_version`、`review_record_version`、`decision_status`、`successor`、`merged_from`、`split_from` 或类似字段。相关信息应写在正文、review evidence、completion evidence、story Dev Agent Record、review/completion templates、future runbook records 或显式 migration note 中。

## Decision taxonomy

使用本文时，必须先判定动作类型。不要把不同治理动作混成“整理一下”或“更新一下”。

| decision type | 含义 | 默认身份处理 |
| --- | --- | --- |
| `rename` | 文件名或路径 slug 改变，但资产对象和职责连续 | 保留原 `doc_id` |
| `move` | 资产移动到新目录或路径，通常包括 path migration | 保留原 `doc_id`，除非身份歧义被记录并确认 |
| `retitle` | H1、frontmatter `title` 或 index display title 改变 | 保留原 `doc_id` |
| `topic_migration` | frontmatter `topic`、目录或 `docs/index.md` 分组改变 | 保留原 `doc_id`，除非实际变成新资产 |
| `revision` | 同一资产的正文、来源、结构或证据实质修订 | 保留原 `doc_id` |
| `replacement_with_continuity` | 新正文或候选内容替换旧正文，但底层资产身份连续 | 保留原 `doc_id`，记录 old-content handling |
| `split` | 一个旧资产拆成多个资产，旧身份可能保留、降级或成为 successor 链的一端 | 需要显式 old/new `doc_id` 处理 |
| `merge` | 多个旧资产合并成一个资产，旧身份可能被归并、废弃或转为历史入口 | 需要显式 old/new `doc_id` 处理 |
| `successor_change` | canonical/current entry point 从一个资产切换到另一个资产 | 需要 successor/replacement path 和 link/index/lifecycle evidence |
| `deprecation` | 旧资产保留访问但不再作为主要当前入口 | 保留旧身份，记录原因、successor 或风险 |
| `archive` | 旧资产仅作为历史、审计或迁移证据保留 | 保留旧身份或记录归档身份处理，说明 remaining access |
| `reactivation` | deprecated/archived 资产重新成为 active/current 入口 | 需要 Maxwell 明确确认和完整复核 |
| `deferred_with_reason` | 当前发现迁移/链接/索引需求，但本次不授权或证据不足 | 不宣称完成，记录后续 owner |
| `hold_for_clarification` | 需要 Maxwell 或 owner 处理身份、successor、路径、状态或范围决定 | 停止写入目标变更 |
| `not_authorized` | 请求超出当前授权范围、workflow 或治理授权 | 禁止执行 |

Rename、move、retitle、topic migration、revision 和 replacement with continuity 的共同点是 identity continuity。Split、merge、replacement that creates a new canonical asset、successor change、deprecation、archive 和 reactivation 需要更强证据，因为它们可能改变读者或 Agent 未来应该从哪里进入、引用什么、忽略什么。

Rename 或 move 执行前必须先检查 path collision 和 old-path coexistence：

- 如果 new path 已存在，必须确认它是否是同一 `doc_id` 的受控中间状态。若 new path 属于不同 `doc_id`、不同 title/topic、不同 asset class 或身份不明，必须使用 `hold_for_clarification`，不得覆盖。
- 如果 old path 和 new path 会同时保留，必须记录 old path disposition：删除、归档、保留为 legacy entry、保留为 migration note、deprecation、redirect-like note、deferred 或 hold。不得让同一身份以两个 canonical entries 留在 `docs/index.md` 或相邻正式入口中。
- 如果 rename 只改变大小写，必须按当前文件系统和 git 行为记录处理方式。对于大小写不敏感文件系统，应使用受控两步 rename 或等价证据，避免仓库保留旧大小写。

## 身份连续性规则

`doc_id` 是正式资产身份，不是路径、标题、H1、index title 或 filename 的缓存。当底层资产身份连续时，必须保留原 `doc_id`。

以下变化不自动产生新 `doc_id`：

- title、H1 或 index display title 改得更准确、更短或更符合当前命名规则。
- filename slug 变化。
- path 或 directory 变化。
- frontmatter `topic` 变化。
- `docs/index.md` section 或排序变化。
- 正文修订、来源补强、质量状态复核、链接修正或治理证据补写。

只有在下列情况中，才可能需要新身份：

- true split：一个旧资产实际拆成多个拥有不同职责边界的新资产。
- true merge：多个旧资产合并成一个新的 canonical asset，旧资产不再分别承担原职责。
- replacement creates a new canonical asset：新资产不是旧资产的连续修订，而是替换为不同对象、不同职责或不同复用入口。
- identity ambiguity：当前证据不足以判断是连续资产还是新资产，需要 Maxwell 确认或 follow-up owner 处理。

身份歧义必须先停止或记录。不得在 identity ambiguity 未解决时宣称 migration 完成、review 通过、validation 完成、active/current 状态成立、deprecation 完成或 archive 完成。

`created_at` 不得因为 rename、move、retitle、topic migration 或 identity-continuous replacement 而重置。它表达资产首次创建为正式文件或正式候选文件的时间，不是当前路径的创建时间。

`updated_at` 只在发生实质正文、metadata、lifecycle、link、index 或 governance evidence 更新时改变。只观察到一个文件移动或审查一个未变更资产，不应伪造 `updated_at` 更新。若 path/index/frontmatter/link evidence 本身被写入或更正，则应按实际变更更新时间。

## Migration Decision Record

每次 rename/migration/split/merge/deprecation/archive/successor/replacement work 都必须留下 human-readable Migration Decision Record。它不是 executable schema、lint rule 或自动化工具。

允许的 `decision type` 只有：

- `rename`
- `move`
- `retitle`
- `topic_migration`
- `revision`
- `replacement_with_continuity`
- `split`
- `merge`
- `successor_change`
- `deprecation`
- `archive`
- `reactivation`
- `deferred_with_reason`
- `hold_for_clarification`
- `not_authorized`

最低记录字段如下：

| 字段 | 必须记录什么 |
| --- | --- |
| affected assets | 被新增、修改、移动、替换、废弃、归档、拆分、合并或检查的资产 |
| decision type | 上述允许值之一 |
| reason | 为什么需要该动作，或者为什么延后/持有/拒绝 |
| old path/title/topic | 旧路径、旧标题、旧 topic；不适用时说明原因 |
| new path/title/topic | 新路径、新标题、新 topic；未决定时说明 blocker |
| old/new `doc_id` treatment | 保留、创建新身份、废弃旧身份、归并、拆分、持有或未授权 |
| old-content handling | preserved、moved、summarized、replaced、deleted with reason、archived 或 deferred |
| successor/replacement path(s) | 当前应使用的继任/替代入口；split/merge 可记录多条 old anchor -> new path 映射；没有时说明风险和 access expectation |
| link impact | changed-file links、body links、fragment anchors、`related_docs`、可发现 inbound references、搜索证据和 unresolved links |
| index impact | `docs/governance/index-synchronization-rules.md` outcome：`updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 `blocked_index_policy_conflict` |
| lifecycle/status impact | draft/reviewed/validated/maintained_asset/deprecated/archived/reactivation 语义如何受影响 |
| validation result | Markdown、frontmatter、links、index、lifecycle 和 workflow-contract evidence |
| unresolved risks | 仍未解决的 identity、successor、duplicate/coexistence、link/index、source/time、status 或 batch 风险 |
| owner/follow-up dependency | Maxwell、duplicate/coexistence policy、review/completion templates、related-doc/link/reuse/network governance、Epic 6 或其他后续 owner |

Migration Decision Record 可以写在：

- 目标资产正文中的迁移说明或废弃说明。
- review evidence。
- completion evidence。
- story Dev Agent Record。
- Review/completion templates。
- future runbook records。
- 明确命名的 migration note。

这些字段不得作为新的全局 frontmatter 字段散落到正式资产中。

本文的 Migration Decision Record 与 Promotion Decision Record、Index Impact Decision Record 是互补关系：

- Promotion Decision Record 负责 candidate 是否可以进入 formal `docs/`、目标路径/身份/质量/索引是否成立。
- Index Impact Decision Record 负责 `docs/index.md` 是否更新、排除、延后或阻塞。
- Migration Decision Record 负责连续身份、old-content handling、successor/replacement、link/index/lifecycle impact、split/merge/deprecation/archive 和 stop/defer reason。

一次变更可以同时需要三种记录。不得用 Migration Decision Record 替代 promotion gate 或 index impact classification。

推荐的人工记录形状如下：

```text
Migration Decision Record

- affected assets:
- decision type:
- reason:
- old path/title/topic:
- new path/title/topic:
- old/new doc_id treatment:
- old-content handling:
- successor/replacement path(s):
- link impact:
- index impact:
- lifecycle/status impact:
- validation result:
- unresolved risks:
- owner/follow-up dependency:
```

## 旧内容处理

旧内容是治理对象，不是迁移噪音。任何 replacement、split、merge、successor change、deprecation 或 archive 都必须显式处理高价值旧内容。

允许的 old-content disposition 是：

| disposition | 使用条件 |
| --- | --- |
| `preserved` | 旧内容仍在同一资产中保留，可能经过局部重组 |
| `moved` | 旧内容移动到明确目标资产或 section，并保留路径/链接证据 |
| `summarized` | 旧内容被压缩为摘要，原细节不再完整保留 |
| `replaced` | 新内容替换旧内容，且说明为什么旧内容不再作为当前表达 |
| `deleted_with_reason` | 旧内容被删除，并记录原因、风险和是否可恢复 |
| `archived` | 旧内容保留为历史、审计、迁移或 legacy context |
| `deferred` | 当前不能决定，必须记录 blocker 和 owner |

不得 silent overwrite。以下情况必须停止、记录风险或请求 Maxwell 确认：

- identity continuity 不清。
- old-content value 不清。
- successor/replacement path 不清。
- lifecycle/status impact 不清。
- link/index impact 不清。
- 删除或压缩旧内容会影响已验证、维护中、高复用或历史决策证据。

删除、压缩或替换高价值旧内容时，必须说明是否保留历史说明、successor note、open question、review evidence 或 migration note。只要旧内容仍可能被读者用于历史比较、迁移审计、legacy context 或旧工作复查，就不能让它静默消失。

## Successor、replacement 与生命周期处理

Deprecation、archive、replacement、split、merge 或 canonical entry point change 都必须处理 successor/replacement。

最低要求是：

- 记录 successor/replacement path。
- 如果没有 successor/replacement，记录 remaining risk 和 remaining access expectation。
- 说明旧资产是否仍可用于 current reference、historical comparison、migration audit、legacy context 或 intentionally excluded。
- 检查 affected inbound/outbound links、body links、`related_docs` 和 `docs/index.md`；link review evidence 使用 `docs/governance/link-maintenance-policy.md`。
- 按 `docs/governance/lifecycle-states.md` 记录 deprecation/archive/reactivation 证据。

`deprecation` 不是删除。废弃资产必须说明废弃原因、当前推荐入口、剩余访问预期、受影响链接和索引处理。没有 successor 时，不得伪装成无风险；必须把缺口写成 unresolved risk 或 open question。

`archive` 不是清理文件。归档资产必须说明归档原因、保留价值、检索路径、替代路径或缺口、索引可见性和 reactivation 条件。归档后若仍在普通索引中可见，必须有 remaining access expectation；若从普通索引移除，必须有可定位的 successor、report/archive entry 或受控排除理由。

`reactivation` 是例外流程，必须有 Maxwell 明确确认。重新激活至少要重新检查 frontmatter、Markdown、source/time context、quality gate 或等价治理门禁、links、related docs、`docs/index.md`、successor/replacement 冲突和 open questions。

## 链接、related docs、索引与 lifecycle impact 检查

每次 rename、move、retitle、topic migration、replacement、split、merge、successor change、deprecation、archive 或 reactivation 都必须检查链接和索引影响。检查范围应与当前授权相称，但不能假装不存在。

最低检查项：

- changed-file links：当前变更文件中的 Markdown links 是否仍可解析。
- body links：目标资产正文中指向旧路径、新路径、successor、replacement 或相邻治理资产的链接是否准确；如果链接包含 `#fragment`，必须确认目标 heading/anchor 仍存在或记录 blocker。
- frontmatter `related_docs`：目标是否存在，是否需要新增、移除、替换或记录缺口。
- inbound references if discoverable：容易通过本地搜索发现的旧路径、旧标题、旧 `doc_id`、旧入口引用或旧 heading fragment 是否会被破坏；完成证据应写明搜索词、范围、结果摘要或为什么本次未授权继续搜索。
- `docs/index.md`：是否需要新增、更新、移动、删除、标注、排除或延后。
- lifecycle/status evidence：是否需要在正文、review evidence、completion evidence 或 future record 中表达 deprecated、archived、reactivated、replacement 或 successor 语义。

`docs/index.md` 处理必须使用 `docs/governance/index-synchronization-rules.md` vocabulary：

| outcome | 使用条件 |
| --- | --- |
| `updated` | 同一变更已更新 `docs/index.md` 并更新必要 metadata |
| `not_applicable` | 本次没有导航影响，并说明原因 |
| `referenced_elsewhere` | 未直接列入总索引，但由明确正式入口引用 |
| `intentionally_excluded` | 受控排除，并记录理由、owner entry point、remaining access expectation 和 re-review trigger |
| `deferred_with_reason` | 当前授权范围不覆盖或需要 future owner decision |
| `blocked_index_policy_conflict` | 继续会制造错误导航或错误权威，必须停止或请求 Maxwell 确认 |

Frontmatter 只能在有实际证据时更新。允许因迁移相关变更而更新的既有字段包括 `title`、`topic`、path-related claims in body、`updated_at`、`source_basis`、`time_context`、`quality_status`、`related_docs`、`open_questions`、`prompt_version` 或 `template_version`。不得新增 `lifecycle_state`、`migration_status`、`successor`、`merged_from`、`split_from` 或其他未经授权字段来解决局部不确定性。

如果关键链接、successor、replacement、index entry 或 lifecycle evidence 无法在当前授权范围内处理，必须使用 `deferred_with_reason`、`hold_for_clarification`、`blocked_index_policy_conflict`、open question 或 explicit unresolved risk。沉默跳过不是有效证据。

## Batch 与 stop conditions

一个明确目标资产的 rename/move，加上一处直接相关的 `docs/index.md` 更新，不自动构成 batch work。但仍必须记录 identity、path/title/topic、old-content、link/index 和 lifecycle impact。

以下情况必须先执行 `docs/governance/batch-readiness-checklist.md` 或停止：

- broad migration。
- index sweep、index-wide restructuring、bulk sorting 或 generated index rewrite。
- multi-topic cleanup。
- generated rewrite。
- batch link repair。
- batch `related_docs` update。
- batch status conversion。
- batch deprecation/archive。
- batch frontmatter normalization。
- target set selected by a rule，而不是一个明确命名的目标资产。
- 任何一次操作会影响多个资产、多个 topics、多个 lifecycle statuses、多个 index entries 或多个 workflow/skill contracts。

以下情况必须停止、延后或请求 Maxwell 确认：

- identity continuity、canonical entry point、successor/replacement 或 old-content value 不清。
- requested change 会改变 stable `doc_id`，但没有明确授权。
- split/merge 影响多个旧身份或多个新身份。
- duplicate concept 或 same-topic coexistence 判断进入 `docs/governance/duplicate-and-coexistence-policy.md` 范围。
- broad link migration、batch index update 或 generated rewrite 被隐含在单个请求中。
- lifecycle/status 会被降级、废弃、归档、重新激活或伪装成 validated/maintained，但证据不足。
- 需要新增全局 frontmatter schema field。
- 需要创建或修改 `.agents/skills/`、planning artifacts、source methodology rules、quality gate semantics、prompt/template versions 或 sprint workflow contracts，而当前授权范围未覆盖。
- 需要创建 runtime code、package manifest、software tests、CLI/API/UI/database/deployment/CI、lint/scoring automation、link scanner、index generator、migration script 或 executable tooling。

相邻归属边界：

- `docs/governance/duplicate-and-coexistence-policy.md` 负责 duplicate concept 和 same-topic coexistence policy。本文调用该政策，但不复制重复/共存完整模型。
- `docs/templates/review-record-template.md`、`docs/governance/document-decision-policy.md`、`docs/governance/rework-loop-examples.md` 和 `docs/templates/completion-report-template.md` 负责 review record、document decision、rework loop 和 completion report templates。本文只定义 Migration Decision Record 的字段需求。
- `docs/governance/revision-regeneration-continuity-policy.md` 负责 targeted revision、structural upgrade、regeneration、continuity record、reference validity 和 high-value content preservation；本文保持 rename/migration/split/merge/deprecation/archive 的 Migration Decision Record 字段权威。
- `docs/governance/related-docs-taxonomy.md` 负责 related docs taxonomy；`docs/governance/link-maintenance-policy.md` 负责 general link maintenance；`docs/governance/reusable-model-entry-points.md` 负责 reusable model entry points；`docs/governance/existing-doc-reuse-procedure.md` 负责 existing-doc reuse procedure；`docs/governance/network-boundary-and-decay-prevention.md` 负责 network boundary / decay prevention。本文只规定迁移相关链接检查。
- Epic 6 负责 batch governance runbook、batch review records 和 batch completion reports。本文只规定何时必须进入 batch readiness。

## 验证清单与维护触发点

执行 rename/migration/split/merge/deprecation/archive/replacement work 前后，至少检查：

1. 任务类型、资产层级、allowed file scope、expected output、validation evidence 和 stop/escalation conditions 已声明。
2. decision type 使用本文允许值。
3. `doc_id` identity continuity 已判断；连续身份保留旧 `doc_id`。
4. 任何新身份、split、merge 或 replacement creating new canonical asset 都有 reason、old/new `doc_id` treatment 和 Maxwell/follow-up owner evidence。
5. `created_at` 未因 rename/move/retitle/topic migration 被重置。
6. `updated_at` 只在实际 body、metadata、lifecycle、link、index 或 governance evidence 更新时改变。
7. Migration Decision Record 覆盖 affected assets、old/new path/title/topic、path collision、old-path disposition、reason、old-content handling、successor/replacement path(s)、link/index/lifecycle impact、validation result 和 unresolved risks。
8. High-value old content 有 explicit disposition：preserved、moved、summarized、replaced、deleted with reason、archived 或 deferred。
9. Deprecation/archive 有 reason、remaining access expectation、successor/replacement 或 unresolved risk。
10. Changed-file links、body links、fragment anchors、`related_docs`、可发现 inbound references 和 `docs/index.md` 已检查，或缺口已记录。
11. `docs/index.md` impact 使用 `docs/governance/index-synchronization-rules.md` vocabulary。
12. Lifecycle/status evidence 与 `docs/governance/lifecycle-states.md` 兼容。
13. 没有新增未经授权的 frontmatter fields。
14. 没有把本文当成实际批量迁移、批量链接修复、批量 frontmatter normalization、duplicate/coexistence cleanup 或可执行工具授权。
15. 如果影响多个资产、topics、statuses 或 index entries，已先执行 batch readiness 或停止。
16. Duplicate/coexistence policy、review/completion templates、related-doc/link/reuse/network governance 和 Epic 6 范围只作为相邻或 future owner 记录，没有在本文中提前实现。

本文的维护触发点包括：

- Duplicate/coexistence policy 更新后，复核 identity ambiguity、split/merge 和 same-topic coexistence 的交界。
- Review record template、document decision policy、rework loop examples 或 completion report template 发生实质字段或 vocabulary 变更时，复核 Migration Decision Record 的落点和字段名。
- Revision/regeneration continuity policy 更新后，复核 revision、replacement、created_at/updated_at、continuity record 和 reference validity 边界。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 更新后，复核 sidecar note、legacy migration 和 old-version compatibility 规则。
- Related docs taxonomy、link maintenance policy、reusable model entry points、existing-doc reuse procedure 或 network boundary / decay prevention policy 更新后，复核 successor/replacement link、related docs 和 inbound/outbound evidence。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report 后，复核 batch readiness 与 batch execution 的边界。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、link scanner、index generator、migration script、new global frontmatter fields 或 batch normalization 后，复核本文的非软件和字段边界。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [Topic、文件命名与路径归属策略：正式 docs 资产的位置、命名与一致性规则](./topic-path-naming-policy.md)
- [候选文档晋升 Checklist：从工作流输出到正式 docs 资产的治理门禁](./candidate-promotion-checklist.md)
- [docs/index.md 同步与导航治理规则：正式导航入口的更新、排除与证据要求](./index-synchronization-rules.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
- [Sidecar 边界政策：正式文档、临时记录、草稿与旁注的归属规则](./sidecar-boundary-policy.md)
- [Legacy Migration Guide：旧文档迁移、保留、归档与兼容处理](./legacy-migration-guide.md)
- [related docs 与相邻概念关系分类：关系类型、边界区分、meaningful-link evidence 与 unresolved target handling](./related-docs-taxonomy.md)
- [链接维护政策：正文链接、related_docs、入链影响与失效链接处置](./link-maintenance-policy.md)
- [可复用模型入口治理：稳定入口、边界、迁移提示与跨文档复用路径](./reusable-model-entry-points.md)
- [既有文档复用流程：新问题先检索、匹配、复用、再决定是否新建](./existing-doc-reuse-procedure.md)
- [网络边界与退化预防：限制链接噪声、关系漂移与维护失控](./network-boundary-and-decay-prevention.md)
- [Review Record Template](../templates/review-record-template.md)
- [Completion Report Template](../templates/completion-report-template.md)
- [文档决策政策：接受、修订、拒绝与延后记录的治理规则](./document-decision-policy.md)
- [返工循环示例：从审查发现到修订记录的闭环样例](./rework-loop-examples.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](./agent-behavior-constraints.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
