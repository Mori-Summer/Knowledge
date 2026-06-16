---
doc_id: governance-prompt-template-quality-version-governance
title: Prompt、模板与质量规则版本治理：规则演进、字段语义与渐进迁移
concept: prompt_template_quality_version_governance
topic: governance
depth_mode: standard
created_at: '2026-05-21T16:01:10+08:00'
updated_at: '2026-06-16T15:17:56+08:00'
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
  - _bmad-output/implementation-artifacts/0-5-quality-gate-baseline.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/index.md
  - docs/governance/revision-regeneration-continuity-policy.md
time_context: stabilization_foundation_review_2026_06_16
applicability: prompt_template_methodology_quality_rule_version_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/governance/agent-behavior-constraints.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/index.md
  - docs/governance/revision-regeneration-continuity-policy.md
open_questions:
  - frontmatter schema 已建立；后续是否需要授权新增独立 methodology_version、quality_gate_version、version_history 或 migration_status 字段？
  - revision/regeneration continuity policy 已建立；是否需要在后续专门审查中把版本迁移记录与 Continuity Record 进一步对齐？
  - Epic 3 review/completion templates 晋升后，是否需要把 version change record 固定为模板字段？
---

# Prompt、模板与质量规则版本治理：规则演进、字段语义与渐进迁移

## 资产角色、权威与适用范围

这份文档是 `Knowledge` 项目中管理 fixed prompt、template、methodology/main spec、quality gate/rubric 和支撑规则资产版本语义的正式治理资产。它约束的是规则如何演进、如何记录影响、如何在文档 frontmatter 和完成汇报中表达实际使用来源，以及旧文档如何在不被静默改写的前提下渐进迁移。

本资产补充 `docs/methodology/document-generation-methodology.md`，不替代正式概念文档的新建、升级、审查和仓库集成流程。概念文档工作仍以主方法论为默认执行规范；本资产只规定主方法论、模板、质量门禁、fixed prompt 或相邻治理规则发生演进时，版本和迁移证据应该如何记录。

本资产也受 `docs/methodology/governance-asset-boundary-policy.md` 约束。方法论资产职责边界仍由该边界政策控制；本文增加的是版本治理规则，不创建平行方法论流程，也不把 template、quality gate、playbook 或 fixed prompt 提升为主规范。

`docs/governance/lifecycle-states.md` 控制正式资产生命周期和状态转换。版本变化可能触发 re-review、降级、迁移或维护复核，但版本字段本身不是完整生命周期状态，也不能替代质量门禁、索引和链接证据。

`_bmad-output/` 下的 PRD、Architecture、Epics、Story、readiness report、review output 和草稿可以说明为什么需要版本变化，但它们不是正式版本来源。一个规则版本只有在写入正确的 `docs/` 正式资产、补齐 frontmatter、明确权威边界、处理索引和完成验证后，才成为可引用的正式版本。

版本标识是治理证据，不是营销标签、质量背书或自动 freshness claim。`prompt_version`、`template_version` 或完成汇报中的 methodology/quality 来源必须表达实际使用的规则集，而不是仓库里当前存在的最新规则集。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：版本对象边界、`prompt_version` / `template_version` 字段语义、quality gate 版本信号、旧文档兼容、迁移记录、索引入口、相关治理依赖、链接和 frontmatter 一致性已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地，version-related frontmatter 字段扩展也未被授权。

## 版本对象与字段边界

版本治理必须先区分规则对象。不同对象可以互相引用，但不能混用一个字段伪装成同一类版本。

| 版本对象 | 典型正式资产 | 管理对象 | 不得混淆为 |
| --- | --- | --- | --- |
| fixed prompt | `docs/methodology/fixed-concept-generation-prompt.md` | 用户或 Agent 调用生成、升级、审查、索引任务时使用的固定提示入口、任务分类和约束措辞 | 主方法论、完整模板、质量门禁 |
| template | `docs/methodology/concept-document-template.md` | 文档结构、路径类型、深度模式、必需信息位点和推荐正文骨架 | 质量通过证明、执行主流程 |
| methodology/main spec | `docs/methodology/document-generation-methodology.md` | 正式概念文档工作的新建、升级、审查、仓库集成主执行合同 | 某个 prompt 片段或模板细则 |
| quality gate/rubric | `docs/methodology/concept-document-quality-gate.md` | Hard Fail、评分维度、质量证据、审查输出和通过/退回判断 | 模板章节清单、生命周期状态 |
| playbook/supporting reference | `learning-new-things-playbook.md`、`cognitive-modeling-playbook.md`、operator guide 等 | 学习、建模、背景、历史导航或辅助判断纪律 | 默认主入口或强制版本字段 |
| governance asset template | Foundation 治理资产模式 | 治理资产的 frontmatter、角色/权威/范围、导航和完成证据模式 | 概念文档模板 |
| index template | `docs/index.md` 的 `index_v1` 模式 | 正式导航入口的 frontmatter 和主题分组格式 | 文档内容质量或生成规则 |

当前正式文档 frontmatter 已有 `prompt_version` 和 `template_version`。当前 frontmatter schema 未授权 `quality_rule_version`、`methodology_version`、`version_history`、`migration_status` 或类似字段；本文不新增这些字段，也不向既有文档批量补字段。

`prompt_version` 的语义是：该文档在生成、实质升级或审查时实际受哪个 prompt 规则集约束。概念文档通常应保留生成或实质升级时实际使用的 prompt 版本，例如 `concept_generation_prompt_v3`。如果一篇旧文档并未用新 prompt 生成或复核，不得只因为新 prompt 存在而把字段改成新值。

`template_version` 的语义是：该文档或资产实际采用的结构模式。概念文档可能使用 `concept_doc_v2`；质量门禁可以使用 `quality_gate_v2`；治理资产可以使用 `governance_asset_v1`；索引可以使用 `index_v1`。字段值必须反映实际结构合同，不得把 template version 当成质量分数或生命周期状态。

methodology/main spec 与 quality gate/rubric 当前没有独立 frontmatter 字段。完成汇报必须显式列出实际使用的 methodology 和 quality 来源，即使 frontmatter 只能记录 `prompt_version` 和 `template_version`。

治理、方法论、索引和部分规则资产可以使用 `prompt_version: not_applicable`，前提是该资产不是由某个固定生成 prompt 直接治理出来的概念文档。此时 `template_version` 仍应标明实际资产模式，例如 `governance_asset_v1`、`unified_spec_v1`、`quality_gate_v2` 或 `index_v1`。

当前仓库中已经观察到的版本信号包括：

- `concept_generation_prompt_v3`：当前 fixed prompt / 概念生成相关 prompt 信号。
- `concept_generation_prompt_v4`：Story 1.2 后的 fixed prompt 信号；保留四个高频概念文档入口，但新增 intake routing、超出四类时回到 intake asset 的停止/路由语义。
- `concept_doc_v2`：当前概念文档模板信号。
- `unified_spec_v1`：当前主方法论规格信号。
- `unified_spec_v2`：Story 1.2 后的主方法论规格信号；保持主规范身份，但新增正式概念文档入口前的 intake handoff 和审查前置分类/等价治理检查语义。
- `quality_gate_v2`：质量门禁基线固化前的质量门禁/评分规则信号。
- `quality_gate_v3`：Hard Fail、六项评分解释、质量状态和完成汇报语义固化后的当前质量门禁/评分规则信号。
- `template_alignment_v2`：模板资产自身对齐版本信号。
- `governance_asset_v1`：Foundation 治理资产模式信号。
- `not_applicable`：该资产不由固定生成 prompt 直接治理，或 prompt 版本不适用。
- `index_v1`：`docs/index.md` 导航索引结构信号。

这些信号只说明实际来源和结构关系。它们不自动说明文档更高质量、已经迁移、已经重新审查或已经处于最新生命周期状态。

## 何时必须变更版本

版本变化由行为变化触发，不由措辞新旧触发。只要规则变化会改变生成、升级、审查、索引、迁移或完成汇报行为，就应视为版本治理事件。

以下情况必须考虑版本变化，并记录旧值、新值和影响：

| 触发类型 | 说明 | 典型影响 |
| --- | --- | --- |
| prompt 行为变化 | fixed prompt 的任务分类、必填输入、输出约束、停止条件、审查要求或索引动作发生改变 | 后续生成或审查行为可能不同，旧文档不应自动伪装成新 prompt 产物 |
| required frontmatter 变化 | 新增、删除、重命名或改变必填 frontmatter 字段语义 | 需要判断旧文档兼容性和 frontmatter schema 影响 |
| required section 变化 | 模板要求的核心章节、信息位点、文档路径类型或 deep/standard 规则改变 | 影响新文档结构和旧文档升级判断 |
| Hard Fail 或评分规则变化 | 质量门禁的阻塞条件、评分维度、分数解释或通过标准改变 | 既有审查结果可能需要 re-review，不得静默保留旧结论 |
| methodology 执行流变化 | 主方法论的新建、升级、审查、仓库集成、索引同步或质量门禁接入顺序改变 | Agent 执行合同和 completion report 需要同步说明 |
| source/time-context 纪律变化 | 当前实践、历史路径、来源核对、事实/推断区分或时间敏感规则改变 | 旧文档可能成为复核候选，不一定立即迁移 |
| lifecycle/status 处理变化 | `quality_status`、生命周期状态、review/validated/deprecated/archived 证据要求改变 | 影响状态声明和完成汇报，不得批量静默改状态 |
| completion-report 要求变化 | 任务完成必须报告的来源、偏离、迁移、非软件边界或验证证据改变 | 后续 Agent 汇报合同变化，旧报告不自动补写 |
| migration compatibility 变化 | 新规则改变旧文档是否可继续维护、何时迁移或如何记录迁移 | 需要记录兼容策略、迁移触发和批量限制 |

每个 version-changing edit 至少必须记录：

- 受影响规则资产。
- 旧版本值和新版本值，或当前无字段时的规则来源 before/after。
- 变更原因。
- 受影响 docs/assets 范围。
- 预期生成行为影响。
- 预期审查影响。
- 索引/导航影响。
- 迁移影响和旧文档是否仍可在旧版本语境下维护。
- 生命周期或 `quality_status` 影响。
- 已批准偏离和未解决风险。

如果无法判断变化是否改变行为，应先按潜在版本治理事件处理，记录不确定点；不得把不确定的语义变化当成“只是润色”直接合并。

## 何时不得变更版本

不是所有编辑都需要版本变更。版本标识应保持稀疏、可解释和可审查；如果版本被用于每次小修，后续 Agent 会失去判断哪些变化真的影响生成或审查行为的能力。

以下情况通常不得变更版本：

- 只修正错别字、标点、缩进、表格排版或 Markdown 格式。
- 只清理链接文字，且链接目标、语义和导航位置不变。
- 只做不改变执行行为的措辞澄清。
- 只把已有规则写得更清楚，但不改变输入、输出、步骤、门禁、字段、状态或迁移义务。
- 只把新正式资产加入 `docs/index.md`，且不改变 index 模板或导航规则本身。
- 只补充来源说明、维护触发点或 open question，且不改变当前规则行为。
- 只修复明显内部矛盾，让正文重新符合已经存在的版本语义。

不得因为仓库里出现新 prompt、template、methodology 或 quality gate，就自动改旧文档的 `prompt_version` 或 `template_version`。版本字段必须跟随实际生成、实质升级、迁移或审查动作，而不是跟随仓库最新资产。

不得静默重写 `source_basis`、`time_context`、`quality_status`、`prompt_version` 或 `template_version` 来暗示一篇文档已经被复核、迁移、升级或质量通过。如果没有实际执行对应动作，字段必须保留原语义，或在完成汇报中明确说明尚未迁移。

不得为了“统一风格”批量归一化旧版本字段。批量 normalization 会制造假证据，必须先按 `docs/governance/batch-readiness-checklist.md` 完成 readiness 判定，并由具体 story 或 Maxwell 明确批准后才能进入。

## 版本变更记录

每次 prompt、template、methodology、quality gate 或相邻治理规则发生版本治理事件时，必须留下可复制、可审查的记录。记录可以写入 completion report、Change Log、review record、专门迁移记录，或后续明确授权的 schema、report、runbook 位置。

最小记录形状如下：

```yaml
version_change_record:
  changed_asset: docs/path/to/rule-asset.md
  old_value: old_version_or_rule_source
  new_value: new_version_or_rule_source
  change_type: prompt | template | methodology | quality_gate | governance_rule | index_rule | mixed
  reason: why_the_change_is_needed
  affected_docs_or_assets:
    - docs/path/or_scope
  expected_generation_impact: how_future_generation_changes
  expected_review_impact: how_quality_or_review_judgment_changes
  migration_plan: no_action | targeted_review | gradual_migration | batch_readiness_required
  index_navigation_impact: none | index_update_required | navigation_policy_review_required
  lifecycle_quality_status_impact: none | re_review | demotion_candidate | status_policy_review
  approved_deviations:
    - approver: Maxwell
      deviated_rule: rule_name_or_path
      reason: why_deviation_is_approved
      follow_up: required_follow_up_or_none
  unresolved_risks:
    - risk_or_open_question
```

`affected_docs_or_assets` 可以是具体文件，也可以是清晰范围，例如 `docs/{topic}/`、`docs/methodology/` 或 “all documents generated with `concept_generation_prompt_v3`”。范围必须可复查，不能写成“若干文档”“相关文档”。

`migration_plan` 必须具体区分：

- `no_action`：旧文档继续在旧版本语境下维护，当前无误导或质量风险。
- `targeted_review`：只审查命中某个风险、topic、版本或入口的资产。
- `gradual_migration`：后续正常修订时逐步迁移，并记录 before/after。
- `batch_readiness_required`：需要批量迁移或批量 normalization，必须先按 `docs/governance/batch-readiness-checklist.md` 完成 readiness 判定，并由具体 story 或 Maxwell 显式批准。

批准偏离必须写清谁批准、偏离哪条规则、为什么偏离、后续如何处理。没有批准来源的偏离只能记录为 unresolved risk，不能当作已授权例外。

## 旧文档兼容与渐进迁移

旧文档可以继续维护在旧 prompt、template、methodology 或 quality rule 语境下。规则演进不意味着旧文档整体失效，也不授权批量静默改写。

旧文档满足以下条件时，可以暂时保留旧版本：

- 仍可读、可编辑、可定位。
- `docs/index.md`、topic、路径和相关链接没有误导性错误。
- `source_basis`、`time_context`、`prompt_version`、`template_version` 没有暗示不存在的复核或迁移。
- 正文没有因新规则出现而变成明显错误或高风险误导。
- 当前任务没有要求对该文档执行实质升级、重新审查或生命周期转换。

迁移应由风险触发，而不是由新版本存在触发。常见迁移触发包括：

- 新质量门禁发现旧文档存在 Hard Fail 或复用风险。
- `source_basis`、`time_context` 或当前实践声明已经过期或不可验证。
- 生命周期状态、`quality_status` 或完成报告声称与实际证据不一致。
- `docs/index.md`、相关链接、入链/出链或 `related_docs` 已经误导读者或 Agent。
- frontmatter 缺失、字段语义错误或字段值暗示不存在的生成/迁移行为。
- 旧模板结构无法支持当前复用场景、审查任务或后续概念网络治理。

迁移记录必须说明：

- before/after 的 `prompt_version`、`template_version` 和实际 methodology/quality 来源。
- 哪些内容被保留，哪些内容被移除。
- 哪些生成、审查、索引或生命周期行为发生了变化。
- 仍然保留的缺口、未验证项和 open questions。
- 对 `docs/index.md`、`related_docs`、入链、出链和导航入口的影响。

迁移动作分为四类：

| 迁移类型 | 含义 | 本文是否授权 |
| --- | --- | --- |
| targeted revision | 对单篇或小范围资产做具体修订，保留高价值内容并更新实际使用版本证据 | 仅在后续任务明确要求目标文件时授权 |
| structural upgrade | 按新模板或新方法论补齐结构、frontmatter、索引和质量证据 | 本文不执行旧文档升级 |
| regeneration | 基于新 prompt/template 重新生成或大幅重写，并记录旧版本保留方式 | 本文不执行重生成 |
| batch migration | 批量扫描、批量改字段、批量归一化版本或批量重构 | 本文明确不授权 |

批量版本 normalization、批量 frontmatter 改写、批量迁移或批量状态转换必须先按 `docs/governance/batch-readiness-checklist.md` 完成 readiness 判定，并由具体 story 或 Maxwell 在具体任务中明确批准。批准前，Agent 只能记录候选范围和风险，不能自行执行。

## 完成汇报要求

任何生成、升级、审查、prompt/template/quality rule 维护或版本迁移任务完成时，completion report 必须说明实际使用的规则来源。不能只说“按当前规范完成”。

生成、升级或审查正式概念文档时，至少汇报：

- 使用的 fixed prompt 来源和版本，例如 `docs/methodology/fixed-concept-generation-prompt.md` / `concept_generation_prompt_v3`，或说明 `not_applicable`。
- 使用的 template 来源和版本，例如 `docs/methodology/concept-document-template.md` / `concept_doc_v2`。
- 使用的 methodology/main spec 来源，例如 `docs/methodology/document-generation-methodology.md` / `unified_spec_v1`。
- 使用的 quality gate/rubric 来源，例如 `docs/methodology/concept-document-quality-gate.md` / `quality_gate_v3`，或旧文档实际使用的历史质量门禁版本。
- 是否存在 approved deviation；如有，写明批准者、偏离规则、原因和后续处理。
- 是否需要旧文档 no action、targeted review、gradual migration 或 batch-readiness planning。
- `docs/index.md`、`related_docs`、内链、生命周期和 `quality_status` 的影响。
- 非软件边界确认：未新增未授权 package manifest、source tree、software tests、build config、runtime automation、CLI/API/UI/database/deployment/CI artifact。

维护 prompt、template、methodology、quality gate 或治理规则时，至少汇报：

- changed asset。
- old value / new value。
- change type。
- 变更原因。
- 受影响 docs/assets。
- 预期生成行为影响。
- 预期审查影响。
- 迁移计划。
- 索引/导航影响。
- 生命周期或质量状态影响。
- approved deviations。
- unresolved risks。

如果 frontmatter 中只有 `prompt_version` 与 `template_version`，完成汇报仍必须补充 methodology 和 quality 来源。当前没有独立字段不是省略来源的理由。

如果任务只做非版本变化编辑，也必须说明为什么不需要版本 bump，并确认没有把旧文档伪装成新版本产物。

## 导航、维护与后续故事影响

本资产位于 `docs/governance/prompt-template-quality-version-governance.md`，并列入 `docs/index.md` 的 `governance` 分组。加入索引表示它是正式治理资产；这不表示它改变了现有 prompt、template、methodology 或 quality gate 的正文语义，也不表示既有文档已经完成迁移。

本资产自身不要求为了版本治理而反向改写 `docs/methodology/document-generation-methodology.md`、`docs/methodology/concept-document-template.md` 或 `docs/methodology/fixed-concept-generation-prompt.md`。`docs/methodology/concept-document-quality-gate.md` 已提升到 `quality_gate_v3` 并记录为质量规则版本治理事件；后续若再把本文规则写入主规范、模板、质量门禁或 fixed prompt，仍应作为独立版本治理事件处理。

已知维护触发点包括：

- 质量门禁再次改变 Hard Fail、评分维度与质量状态基线时，复核当前 quality gate 版本、`quality_status` 和本文质量规则版本记录是否需要细化。
- `docs/governance/governance-asset-navigation-policy.md` 已建立治理资产导航、索引与入口归属基线；后续版本治理事件涉及导航处理时，应按该政策复核 index/navigation impact。
- `docs/governance/batch-readiness-checklist.md` 已定义批量治理 readiness；batch migration、batch normalization 和批量版本记录必须按该政策暴露 target set、冲突、停止条件、owner decision 和恢复策略。
- `docs/governance/frontmatter-schema.md` 已定义正式 frontmatter schema、`doc_id` 和 baseline required fields；后续若授权新增 `methodology_version`、`quality_gate_version`、`version_history` 或 `migration_status` 等字段，必须回到本文复核版本记录和迁移规则。
- Epic 3 review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更时，复核本文的 version change record 和 completion report 格式。
- `docs/governance/revision-regeneration-continuity-policy.md` 已建立修订、重生成与版本连续性策略；后续若该政策更新，复核 targeted revision、structural upgrade、regeneration 和 gradual migration 的记录深度。
- `docs/governance/sidecar-boundary-policy.md` 或 `docs/governance/legacy-migration-guide.md` 更新后，复核 version migration records 与 sidecar/legacy 边界。

当任何后续 story 改变 prompt/template/methodology/quality gate 的正式行为时，应回到本文检查是否需要版本记录、迁移计划、索引影响说明和旧文档兼容声明。
