---
doc_id: governance-governance-asset-navigation-policy
title: 治理资产导航、索引与入口归属政策
concept: governance_asset_navigation_policy
topic: governance
depth_mode: standard
created_at: '2026-05-22T14:16:47+08:00'
updated_at: '2026-05-30T15:53:15+08:00'
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
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/batch-readiness-checklist.md
  - docs/index.md
  - docs/governance/revision-regeneration-continuity-policy.md
time_context: phase_4_epic_4_revision_regeneration_continuity_policy_2026_05_28
applicability: governance_asset_navigation_index_and_owner_entry_decisions
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/revision-regeneration-continuity-policy.md
open_questions:
  - Epic 2 Story 2.4 定义详细 index synchronization rules 后，是否需要把本文的导航处理值升级为更正式的 schema？
  - 如果 completion-report template 后续调整 index/navigation impact 字段，是否需要同步本文的 index treatment 汇报要求？
  - Story 5.1 related docs taxonomy 与 Story 5.2 link maintenance policy 已建立；后续是否需要进一步区分 owner entry、supporting link、related doc 和 successor link 的维护证据？
---

# 治理资产导航、索引与入口归属政策

## 资产角色、权威与适用范围

这份文档是 `Knowledge` 项目的正式 Foundation 治理资产，负责规定治理资产、模板资产和 runbook 资产进入正式 `docs/` 体系时如何声明角色、权威、适用范围、owner entry point、导航处理和索引处理。

本资产补充 `docs/methodology/document-generation-methodology.md`、`docs/governance/agent-behavior-constraints.md` 和 `docs/methodology/governance-asset-boundary-policy.md`。它不替代主方法论、概念文档质量门禁、生命周期政策、prompt/template/quality 版本治理，也不提前定义 Epic 2 Story 2.4 的详细 index synchronization rules。

本资产适用于：

- `docs/governance/` 下的新建、实质更新、废弃或归档治理政策。
- `docs/templates/` 下的正式模板资产。
- 未来 `docs/runbooks/` 下的正式操作流程资产。
- 把 `_bmad-output/` 中的规划产物、草稿、story、review output 或 completion record 晋升为正式 `docs/` 资产前的导航与入口判断。
- 完成汇报中对 index/navigation impact、owner entry point 和 intentionally excluded 的证据说明。

本资产不授权批量重写 `docs/index.md`、批量迁移旧文档、创建平行主方法论、创建第二套索引同步细则、引入 executable validation tooling，或新增 `owner_entry_point`、`navigation_treatment`、`index_policy_version`、`lifecycle_state`、`schema_version` 等 frontmatter 字段。相关 schema 扩展由 Epic 2 负责。

当前 `quality_status: draft` 是保守治理状态。原因是本文是新的 Foundation 导航基线；Epic 2 和 Epic 3 已细化 schema、index、review/decision/rework/completion report 形态，Story 5.1 已细化 related docs taxonomy，Story 5.2 已细化 link maintenance，Story 5.3 已细化 reusable model entry points，Story 5.4 已细化 existing-doc reuse procedure，Story 5.5 已细化 network boundary / decay prevention，后续 Epic 6 仍会细化 batch governance。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组；navigation treatment 是 `listed_in_docs_index`。本文的权威范围限于 governance/template/runbook asset 的导航、索引与入口归属判断，不覆盖概念文档生成主流程、质量门禁评分、生命周期状态转换或 prompt/template/quality 版本 bump 决策。

## 必须声明的资产归属字段

正式 governance/template/runbook asset 必须在正文或 completion report 中声明以下归属信息。除非后续 Epic 2 明确扩展 schema，这些要求是正文与报告要求，不是新的全局 frontmatter 字段。

| 声明项 | 必须说明什么 | 不合格信号 |
| --- | --- | --- |
| `role` | 资产在仓库中的职责，例如治理政策、模板、runbook、索引支持资产或方法论支撑资产 | 只写主题，不说明承担什么治理职责 |
| `authority` | 它约束谁、在哪些任务中可作为规则依据、是否只是支撑说明 | 暗示自己替代主方法论或质量门禁 |
| `applicable scope` | 适用文件、任务、资产层级和使用场景 | 泛化到所有文档或所有 Agent 行为 |
| `owner entry point` | 人或 Agent 应从哪个入口发现、理解和应用该资产 | 同一规则同时宣称多个平行主入口 |
| `relationship to main methodology` | 它如何补充 `docs/methodology/document-generation-methodology.md`，以及不替代哪些主规范 | 没有说明与主方法论的边界 |
| `navigation treatment` | 使用四种允许处理之一：`listed_in_docs_index`、`referenced_from_methodology_entry`、`referenced_from_governance_entry`、`intentionally_excluded` | 新资产进入 `docs/` 后没有导航说明 |
| `index treatment` | 是否更新 `docs/index.md`，不更新时为什么不更新 | 缺入口且没有排除理由 |
| `source/lifecycle/quality evidence` | 来源依据、时间语境、质量状态、生命周期影响、维护触发点 | 用 `_bmad-output/` 直接充当正式规则 |
| `completion-report evidence` | 完成汇报中如何证明已处理 frontmatter、links、index、workflow contract 和非软件边界 | 只说“已完成”或“已检查” |

资产正文可以使用自然语言、小节、表格或 checklist 表达这些声明，但必须让审查者能直接判断通过或不通过。

## Owner Entry Point

`owner entry point` 是读者或 Agent 试图发现、理解或应用某个资产时，应该优先进入的唯一主入口。它解决的问题不是“还有哪些文件相关”，而是“从哪里开始才不会误读权威关系”。

允许的 owner entry point 包括：

- `docs/index.md`：正式资产的默认可发现导航入口，适合有广泛复用价值的正式 governance/template/runbook asset。
- `docs/methodology/document-generation-methodology.md`：仅当资产直接改变、支撑或解释主方法论执行路径时，才可作为 owner entry point 或主入口引用点。
- 某个正式治理资产：当资产是另一份治理政策的从属细则，且放入普通索引会制造噪声或重复入口时使用。
- 未来 template/runbook index：当 `docs/templates/` 或 `docs/runbooks/` 形成专门入口后，模板或 runbook 可以声明这些入口为 owner entry point。
- 明确命名的 workflow、skill 或 story：仅适用于尚未晋升为正式 `docs/` 资产的 workflow output，不得伪装成 `docs/` 正式导航。

owner entry point 必须足够唯一，避免两个资产同时声称对同一规则拥有平行主权威。如果两个资产都像主入口，Agent 必须记录冲突并按主规范、治理资产边界和当前 story 权限处理；影响 canonical entry point 时必须请求 Maxwell 确认。

Story 3.1 后，`docs/templates/` 已有首个正式模板入口；`docs/runbooks/` 当前仍未开放为正式目录。创建这些目录中的正式资产时，资产正文仍必须声明 owner entry point：在专门索引存在前，默认从 `docs/index.md` 或明确上级 governance/methodology entry point 进入；专门索引存在后，再由对应索引承担默认入口。

## Navigation Treatment Decision Model

每个新建或实质更新的正式 governance/template/runbook asset 必须选择且记录一种导航处理。不得把“暂时没处理索引”当成隐含状态。

| navigation treatment | 适用条件 | 必须提供的证据 |
| --- | --- | --- |
| `listed_in_docs_index` | 资产是正式 `docs/` 资产，具有广泛复用价值，读者或 Agent 应能从仓库总索引发现 | `docs/index.md` 已新增或更新入口；标题、路径、topic 和正文/frontmatter 一致 |
| `referenced_from_methodology_entry` | 资产直接支撑、改变或解释主方法论执行路径；读者从主方法论进入比从普通索引更能避免误用 | 说明引用点、为什么属于 methodology execution path、是否需要 `docs/index.md` 同步 |
| `referenced_from_governance_entry` | 资产是另一份治理政策的从属细则，直接列入总索引会制造重复入口、噪声或错误权威层级 | 说明上级治理入口、从属关系、剩余发现路径和为什么不直接列入总索引 |
| `intentionally_excluded` | 资产不适合进入正式导航，例如一次性记录、内部审查证据、临时工作产物、已明确保留但不应被复用的资产 | 写明排除理由、owner entry point、剩余访问预期、re-review trigger，并在 completion report 中提供证据 |

正式 governance asset 如果具有跨任务、跨 Agent 或长期复用价值，默认处理是 `listed_in_docs_index`。偏离默认处理时，必须说明为什么 `docs/index.md` 不是合适入口。

`referenced_from_methodology_entry` 只能用于真正影响主方法论执行路径的资产。不能因为资产重要，就把它强行塞进 `docs/methodology/document-generation-methodology.md`；主方法论仍是概念文档执行合同，不是所有治理规则的目录页。

`referenced_from_governance_entry` 适合从属政策、窄范围规则或避免重复入口的场景。上级治理入口必须已经存在，或在同一任务中被明确创建并完成验证。

`intentionally_excluded` 是受控例外，不是偷懒路径。它必须包含：

- exclusion reason：为什么不列入普通导航。
- owner entry point：从哪里仍可定位或理解它。
- remaining access expectation：谁可以在什么场景访问它。
- re-review trigger：什么变化会要求重新考虑索引或导航。
- completion-report evidence：完成汇报中明确列出该排除判断。

Story 0.6 只建立 Foundation 导航基线。详细的新增、移动、重命名、标题变化、topic 变化、状态变化、废弃和归档索引同步规则，仍由 Epic 2 Story 2.4 负责。

## `_bmad-output/` Promotion Boundary

`_bmad-output/` 是 workflow output 生命周期，不是正式 `docs/` 生命周期。PRD、Architecture、Epics、Story、readiness report、review output、completion record 和草稿可以作为上下文、证据或候选来源，但不会自动成为正式知识资产或治理规则。

禁止把 `_bmad-output/` 文件直接加入 `docs/index.md`，也禁止把它们复制为正式规则后跳过 frontmatter、角色声明、导航处理和验证证据。

从 `_bmad-output/` 晋升到 `docs/` 的最低检查是：

1. 判断 task/asset type：新建正式资产、升级正式资产、审查记录、一次性报告、模板、runbook 或治理政策。
2. 确认 target path：必须位于正确的 `docs/{topic}/`、`docs/methodology/`、`docs/governance/`、`docs/templates/` 或未来 `docs/runbooks/`。
3. 补齐正式 YAML frontmatter：使用现有 schema，不新增未经授权字段。
4. 声明 role、authority、applicable scope、owner entry point、relationship to main methodology。
5. 说明 source basis、time context、quality status、lifecycle impact 和 open questions。
6. 选择 navigation treatment，并处理 `docs/index.md` 或记录受控排除。
7. 验证 changed-file links、`related_docs`、heading hierarchy、frontmatter array fields 和正文主张一致。
8. 在 completion report 中记录验证结果、未解决风险和非软件边界。

如果某个 `_bmad-output/` 文件只是 story、review output 或 completion record，它的 owner entry point 仍是 BMad workflow 或 sprint tracking，不是 `docs/index.md`。只有被明确 promotion 后，才按正式资产处理。

## Completion Report Requirements

涉及 governance/template/runbook asset 的完成汇报必须明确说明导航处理，不能只列文件名。

`docs/index.md` impact 必须是以下之一：

- `updated`：已更新 `docs/index.md`，并列出新增、移动、删除或标题变更。
- `not_applicable`：资产不是正式 `docs/` 资产，或本次没有导航影响，并说明原因。
- `referenced_elsewhere`：未列入总索引，但由明确 methodology/governance/template/runbook entry point 引用。
- `intentionally_excluded`：按本文例外要求写明排除理由、owner entry point、remaining access expectation 和 re-review trigger。
- `deferred_with_reason`：发现需要后续处理，但当前 story 不授权；必须说明风险和后续入口。

验证结果必须使用 `通过`、`未通过`、`未验证` 或 `不适用`。最低验证项包括：

| 验证项 | 期望证据 |
| --- | --- |
| Markdown | H1/title 一致，heading hierarchy 稳定，表格和列表无明显破损 |
| frontmatter | 必填字段存在；数组字段为 YAML arrays；`doc_id`、`title`、`concept`、`topic`、文件路径、`source_basis`、`time_context`、`quality_status` 与正文一致；文件路径是验证证据，不是新增 frontmatter 字段 |
| role/authority/scope | 正文明确角色、权威、适用范围和不替代关系 |
| owner entry point | 入口唯一且不会制造平行主规范 |
| navigation treatment | 四种允许值之一，并有索引或例外证据 |
| links | `related_docs` 和正文内链可定位；未存在目标必须进入 open questions 或 planned asset |
| lifecycle/quality status | 与 `docs/governance/lifecycle-states.md` 和质量门禁基线兼容，不伪造晋升状态 |
| version governance | 没有无依据 bump prompt/template/quality 版本；如改变规则行为，记录影响 |
| workflow contract | story tasks、Dev Agent Record、File List、Change Log、sprint status 按 workflow 更新 |
| non-software boundary | 未新增 package manifest、`src/`、`tests/`、build config、runtime automation、CLI/API/UI/database/deployment/CI artifact |

如果某项未验证，完成汇报必须说明原因。存在阻塞性未验证项、断链、索引不一致、未经授权的 schema 字段或非软件边界破坏时，不得把资产标记为完成。

## 维护触发点

本资产是 Foundation 导航基线，后续治理故事可能要求复核。

已知维护触发点包括：

- `docs/governance/batch-readiness-checklist.md` 已定义批量治理 readiness 与范围判定；批量新增、批量移动、批量排除和批量索引更新必须按该政策暴露 target set、excluded files、冲突、停止条件和恢复策略。
- Epic 2 frontmatter、`doc_id`、topic、path、naming、candidate promotion 和 index synchronization rules 落地后，复核本文是否需要 schema 化 navigation treatment 或 owner entry point。
- Epic 3 review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更时，复核本文完成汇报字段是否应同步。
- Story 4.1 已建立 revision/regeneration continuity policy；后续若该政策更新，复核 deprecated/archived asset 的导航、successor 入口和 reference validity 要求。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 更新后，复核 supplemental/legacy assets 的导航边界。
- Story 5.1 related docs taxonomy、Story 5.2 link maintenance policy 更新后，复核 owner entry、supporting link、related doc、successor link 和 remaining access expectation 的边界。

本资产不授权批量目录重组、批量 frontmatter normalization、软件自动化检查或第二套索引同步系统。此类动作必须先满足 `docs/governance/batch-readiness-checklist.md` 的 readiness 判定，并由后续 story 或 Maxwell 明确批准。
