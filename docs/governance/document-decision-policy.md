---
doc_id: governance-document-decision-policy
title: 文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements
concept: document_decision_policy
topic: governance
depth_mode: standard
created_at: '2026-05-27T10:45:17+08:00'
updated_at: '2026-06-17T10:21:48+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/planning-artifacts/implementation-readiness-report-corrected-2026-05-20.md
  - _bmad-output/implementation-artifacts/3-2-document-decision-policy.md
  - _bmad-output/implementation-artifacts/3-3-rework-loop-examples.md
  - _bmad-output/implementation-artifacts/3-4-completion-report-template.md
  - docs/index.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_epic_3_evidence_asset_review_2026_06_17
applicability: formal_docs_review_decision_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/index.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/rename-migration-policy.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/reusable-model-entry-points.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
open_questions:
  - 如果 completion-report template 后续调整 accepted/returned/regenerated/deprecated/held outcome mapping，是否需要同步本文 copyable decision summary skeleton？
  - related-doc taxonomy、link maintenance、reusable model entry point、existing-doc reuse 或 network boundary policy 更新后，是否需要细分 index/link/related-doc impact 的证据分类？
  - Epic 6 建立 batch governance runbook 与 batch records 后，是否需要为批量 review decision 汇总增加专门引用方式？
---

# 文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中正式文档 review final decision 的 canonical governance policy。它定义 reviewer 在完成 review record 分类、Hard Fail、评分或等价治理检查、未验证/不适用登记以及必要的专门 decision records 后，如何选择最终 decision label、记录 required evidence、表达 allowed status wording，并说明对 `quality_status`、生命周期、索引和后续动作的影响。

本文适用于：

- 正式 `docs/` 资产的新建、升级、审查、修订、候选晋升、废弃、归档、迁移、重复/共存判断和索引影响判断。
- `docs/methodology/`、`docs/governance/`、`docs/templates/`、未来 `docs/runbooks/`、`docs/index.md` 和正式 report entry 的等价治理审查。
- `_bmad-output/`、对话输出、review output、completion evidence 或临时草稿准备晋升为正式资产前的 candidate decision。
- 涉及 source/current-practice claim、lifecycle change、candidate promotion、index impact、rename/migration、duplicate/coexistence 或 batch readiness 的 final decision 汇总。

本文补充而不替代以下资产：

- Story 3.1 `docs/templates/review-record-template.md`：负责 review record 的记录形状、分类字段、Hard Fail/score/evidence 表和 final decision 位置。
- Story 3.3 `docs/governance/rework-loop-examples.md`：负责 failure classes、repair instructions、regeneration rationale examples、prior failure records 和 resubmission checks。
- `docs/methodology/concept-document-quality-gate.md`：负责概念文档 Hard Fail、六项评分、质量状态词汇和概念文档审查结论。
- `docs/governance/lifecycle-states.md`：负责 `draft`、`reviewed`、`validated`、`maintained_asset`、`deprecated`、`archived` 和 reactivation 语义。
- Epic 0 foundation：Agent 行为约束、方法论资产边界、生命周期、版本治理、质量基线、导航基线和 batch readiness。
- Epic 1 methodology：概念文档生成合同、输入摄入、样例、来源纪律、主方法论、模板、质量门禁和 fixed prompt。
- Epic 2 repository governance：frontmatter schema、topic/path/name、candidate promotion、index synchronization、rename/migration、duplicate/coexistence。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/document-decision-policy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：decision prerequisites、allowed final decision labels、status vocabulary separation、blocker precedence、weak evidence handling、regenerate/reject/hold/defer/not_authorized boundaries、owner/index entry、相关治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

本文自身的 Index Impact Decision Record 是：

```text
Index Impact Decision Record

- affected file: docs/governance/document-decision-policy.md
- target title: 文档决策政策：accept/promote、revision、regenerate、reject、deprecate/archive、hold/defer、status impact 与 evidence requirements
- target path: ./governance/document-decision-policy.md
- target section: docs/index.md ## governance
- outcome: updated
- action taken: add governance entry and update index metadata
- reason: Story 3.2 explicitly authorizes the canonical document-decision policy asset
- validation result: target exists and relative link resolves
- unresolved risk: none for this index entry; completion-report and Epic 5 link/network integrations are referenced; Epic 6 batch integrations remain open questions
```

## 职责边界与非目标

本文定义 final decision 的语义、证据要求、状态影响和升级路径。它不自动创建 review records，不替代 reviewer 对目标资产的实际检查，也不允许用一个总分或一句通过结论覆盖证据缺口。

本文不实现以下范围：

- Story 3.3 rework-loop examples：失败类别、返工路径和示例库已由 `docs/governance/rework-loop-examples.md` 定义。
- `docs/templates/completion-report-template.md`：最终完成汇报模板和项目级完成摘要。
- related-doc taxonomy、link maintenance、reusable model entry point、existing-doc reuse procedure 或 network boundary policy。
- Epic 6 batch governance runbook、batch review record 或 batch completion report。
- 既有正式资产的实际 review records、批量审查记录、评分报告、完成报告或状态迁移。
- executable tooling、machine-readable schema、JSON/YAML schema、lint/scoring tool、decision generator、review-record generator、CLI/API/UI/database/deployment/CI、package manifest、`src/` 或 `tests/`。

本文不授权在目标资产 frontmatter 中新增 `schema_version`、`lifecycle_state`、`decision_status`、`review_record_version`、`owner_entry_point`、`navigation_treatment`、`index_policy_version`、`quality_gate_version`、`promotion_policy_version`、`migration_status`、`duplicate_status`、`coexistence_status`、`successor`、`canonical_asset` 或类似字段。Decision labels 应记录在 review record、completion evidence、story Dev Agent Record、目标正文中的合规证据或未来 Epic 3/6 records 中；它们不是新的全局 frontmatter 字段。

## Decision prerequisites

Reviewer 做 final decision 前，必须先完成 Story 3.1 review classification。最低前置证据包括：

| 证据 | 必须说明什么 |
| --- | --- |
| task type | new document、upgrade、review、index-only update、candidate promotion、rename/migration、duplicate/coexistence decision、lifecycle change、methodology/governance/template maintenance、report review 或 batch-readiness review |
| asset level | formal knowledge asset、methodology asset、governance asset、template asset、runbook asset、`docs/index.md`、formal report、candidate/workflow output 或 BMad workflow/skill asset |
| target path and source path | 被审查目标、候选来源、planned target 或不适用原因 |
| document type and depth mode | 概念文档路径类型、`standard`/`deep` 或非概念资产的等价治理检查理由 |
| current status signals | 当前 `quality_status`、生命周期语义和 BMad story status；三者必须分开 |
| applicable rule sources | 主方法论、质量门禁、frontmatter、topic/path、promotion、index、migration、duplicate、lifecycle、source discipline、batch readiness 或 workflow/skill source |
| Hard Fail / blocker evidence | 命中的失败项、证据位置、阻塞理由、required fix 和 owner |
| score or equivalent checks | 概念文档六项评分，或治理/模板/索引/方法论资产的等价角色、权威、frontmatter、source、link、index、workflow-contract 检查 |
| `未验证` / `不适用` register | 相关但未验证的证据、确实不适用的证据、impact、owner 和 next action |
| specialized records | Promotion、Index Impact、Migration、Duplicate/Coexistence、Batch Readiness 或 Source Discipline evidence 是否需要、是否完整、是否阻塞 |

缺少前置证据时，final decision 不能是 `accept_promote`、`accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`validated`、`maintained`、`ready` 或任何等价通过措辞。允许路径是 `hold_for_clarification`、`defer_with_reason`、`targeted_revision`、`regenerate`、`reject_duplicate_or_misplaced`、`deprecate_or_archive` 或 `not_authorized`，取决于缺口性质和授权范围。

## Allowed final decision labels

以下 labels 是 review final decision 的允许值。它们统一 review record、completion evidence 和 story Dev Agent Record 中的 final decision 语义；它们不替代 Promotion Decision Record、Index Impact Decision Record、Migration Decision Record、Duplicate / Coexistence Decision Record 或 Batch Readiness owner decision 的专门枚举。

| Decision label | Use when | Forbidden when | Required evidence | Allowed status wording | `quality_status` impact | Lifecycle impact | Index/link impact | Required next action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `accept_promote` | 候选或目标已通过适用 Hard Fail/等价治理检查，专门 records 完整，owner decision 允许晋升、推广或 stronger status 表达 | 任意 Hard Fail、blocking `未验证`、owner gap、identity ambiguity、source/currentness blocker、index/link blocker、duplicate/migration blocker 或授权边界未决 | Review classification、无 blocker 证据、score 或等价检查、必要 specialized records、index/link validation、allowed status rationale | 可写 accepted、promoted、validated candidate、ready for current formal use；具体措辞必须说明范围 | 可保留或提升到当前规则允许的值，例如 `reviewed`、`validated`、`maintained_asset` 或概念文档适用的 `upgraded_v1`；不得越过证据 | 可支持 draft -> reviewed/validated 或 candidate -> formal；治理资产可支持 maintained equivalent | 必须有 Index Impact outcome；新增/移动/晋升通常是 `updated` | 更新目标证据、索引和 completion/review record；若提升 status，记录依据 |
| `accepted_for_current_use` | 当前使用范围足够可靠，无 Hard Fail，存在非阻塞限制或无需 promotion/status 提升 | 需要正式 promotion 但 promotion record 不完整；存在 blocker；使用范围无法限定 | 可用范围、限制条件、无 blocker 证据、score/等价检查、unresolved nonblocking risks | 可写 accepted for current use、usable within stated scope；不得写 fully validated、promoted、unqualified ready | 通常保留当前值或保守设为 `reviewed`；不自动提升到 `validated`/`maintained_asset` | 支持受控参考，不等于全局稳定入口 | Index/link 可为 `updated`、`not_applicable` 或其他已记录 outcome；风险必须可定位 | 记录适用范围、限制、follow-up owner 和 re-review trigger |
| `accepted_with_nonblocking_follow_up` | 无 Hard Fail，弱项不阻塞当前用途，但需要明确后续修订、source check、link cleanup、index clarification 或 future-story work | follow-up 实际阻塞身份、来源、schema、critical link/index、owner decision 或当前使用结论 | 非阻塞风险清单、每项 evidence location、owner、due story 或 trigger、为什么不阻塞当前结论 | 可写 accepted with nonblocking follow-up；不得写 no risk、fully complete、validated without caveat | 通常不提升；若提升，必须说明 follow-up 不影响该状态 | 生命周期可保持现状或进入 reviewed；不得因有 follow-up 自动 validated | Index/link impact 必须分类；未处理链接不能被隐藏 | 创建具体 follow-up，不允许只写“后续优化” |
| `targeted_revision` | 结构、frontmatter、source、link、index、score 或局部证据存在可定位、可修复缺口；保留当前资产形状比重生成更合适 | 缺口已经造成整体文档类型错误、身份坍塌、来源不可用、结构不可修复或授权外重写 | 每项修复的 section/frontmatter/link/source/index evidence location、required fix、owner、blocking status | 可写 needs targeted revision、blocked pending revision；不得写 accepted、validated、ready、maintained | 保留或降低到 `draft`/candidate；不得提升 | 生命周期通常保持 draft/re-review；若 active 资产出现 blocker，应记录降级或 re-review | 需要 index/link impact；关键缺口未修复时不得更新为通过入口 | 执行或排期具体修订；修订后重新 review |
| `regenerate` | 文档结构坍塌、文档类型错误、source basis 不可用、currentness 大面积失败、重复/错置导致局部修补不足，或需要保留高价值旧内容后重写 | 局部修复足以恢复完整性；重生成会覆盖高价值旧内容且无处置证据；当前 story 不授权重写 | 为什么 targeted revision 不足、可保留内容、不可保留原因、source/time risk、old-content handling、owner/future story | 可写 regenerate required、rewrite required before acceptance；不得写 revised enough、accepted | 目标保持或降为 `draft`/candidate；不允许提升 | 可能触发 replacement、successor、migration 或 deprecation evidence | 若路径/身份/旧内容/索引受影响，必须引用 Migration/Index/Duplicate records | 指定重生成范围、保留内容、owner 和后续 review；示例留给 Story 3.3 |
| `reject_duplicate_or_misplaced` | 候选或拟议入口重复、同主题不可区分、topic/path 错置、无 durable reuse context、来源/质量不足或不应进入正式资产 | duplicate/coexistence evidence 不完整；更适合 merge、adjacent link、narrower document、hold 或 defer | Duplicate / Coexistence Decision Record 或等价引用、rejected alternatives、canonical path、topic/path reason、index impact | 可写 rejected as duplicate/misplaced candidate；不得写 accepted、promoted、coexisting without record | 不写入或不提升目标 `quality_status`；已有资产需要另行 lifecycle/migration 证据 | 候选可退出；正式资产若受影响，可能进入 deprecate/archive 或 migration 流程 | 通常 `not_applicable`、`intentionally_excluded`、`referenced_elsewhere` 或 `updated`；不得制造平行 canonical entry | 记录拒绝原因、是否保留候选素材、是否需要 link/adjacent reference |
| `deprecate_or_archive` | 正式资产不再应作为主要当前入口，或仅保留为历史、审计、迁移或旧决策证据 | successor/replacement、remaining access、link/index/lifecycle impact 未决且需要 owner decision；当前 story 不授权 lifecycle change | target lifecycle outcome (`deprecated` or `archived`)、deprecation/archive reason、successor/replacement 或 missing-successor risk、remaining access expectation、Migration Decision Record、Index Impact outcome | 可写 deprecated、archived、not current; 不得写 active/current/validated without caveat | 可设为 `deprecated` 或 `archived`，前提是 frontmatter schema 允许且证据完整；否则在正文/review evidence 记录 | 必须明确目标生命周期 outcome；`deprecated` 与 `archived` 的证据不能混用，archive 不等于删除 | 必须处理 index visibility、successor links、related docs、inbound/outbound impact | 更新资产说明和索引/link evidence，或 hold/defer 等待 owner |
| `hold_for_clarification` | 缺 Maxwell/owner decision、identity/canonical path 不清、schema 冲突、lifecycle/status ambiguity、高风险 source uncertainty 或需要确认后才能写 | 问题只是可执行的局部修复；或当前授权明确禁止继续，应使用 `not_authorized` | 要澄清的问题、为什么阻塞、候选选项、需要谁确认、暂停的写入范围 | 可写 held pending clarification、hold for Maxwell/owner decision；不得写 accepted/ready | 不提升；通常保持当前状态或保守 `draft` | 生命周期不变，除非已有证据要求降级并被授权 | 关键 index/link 变更暂停；可记录 deferred/blocked impact | 请求 owner decision；确认前不做会改变权威/状态的写入 |
| `defer_with_reason` | 真实问题存在，但当前 story、授权、时间、目标集或 future Epic owner 不允许本轮处理；当前结论可继续或必须暂停要明确 | 问题实际阻塞当前 acceptance 却被伪装成 deferred；或无具体 owner/future story | defer reason、impact blocking/nonblocking、owner/future story、current work allowed or stopped、re-review trigger | 可写 deferred with reason；不得写 resolved、complete、accepted if blocking | 不因 defer 提升；非阻塞 defer 可保持当前值 | 生命周期保持或进入 re-review；阻塞 defer 不允许 active/validated wording | Index/link impact 使用 `deferred_with_reason` 或相应 blocking state | 创建 future-story dependency 或 owner follow-up，并说明当前是否可继续 |
| `not_authorized` | 请求跨过非软件边界、future-story boundary、schema boundary、batch readiness boundary、workflow contract、文件范围或 Maxwell 授权 | 当前 story 明确授权且证据足够；问题只是需要 clarification 而非禁止 | crossed boundary、source instruction、affected forbidden scope、what was not done、safe stopping point | 可写 not authorized、stopped by boundary；不得写 deferred as if optional、accepted | 不改变目标 status；不得新增字段或状态声明 | 生命周期不变，除非另有授权处理 | 不执行未授权 index/link/status/batch 变更 | 停止越界动作；记录允许范围和需要的授权路径 |

## Status vocabulary separation

`quality_status`、lifecycle vocabulary、review decision label 和 BMad story status 是四类不同信号，不能互相替代。

| 信号 | 表达什么 | 禁止替代 |
| --- | --- | --- |
| review decision label | 本次 review 对目标或候选的 final decision | 不能写入目标 frontmatter 作为 `decision_status` |
| `quality_status` | frontmatter 中的质量、维护或历史兼容状态 | 不能用 story `review`/`done` 代替 |
| lifecycle vocabulary | 资产在仓库中的成熟度、使用方式、废弃或归档语义 | 不能只靠一个 decision label 自动成立 |
| BMad story status | workflow 执行跟踪，例如 `ready-for-dev`、`in-progress`、`review`、`done` | 不能作为正式资产状态或质量证据 |

Decision labels 可以出现在 review record、completion evidence、story Dev Agent Record、目标正文的合规说明或 future templates 中。除非后续 schema 明确授权，它们不得成为新的全局 frontmatter fields。

## Blocker precedence

任意阻塞项存在时，accept/promote 和 through-status wording 必须停止。阻塞项包括但不限于：

- 任意 Hard Fail。
- 等价 governance blocker：frontmatter、path/topic、identity、template、index、lifecycle、batch readiness 或 workflow contract 失败。
- blocking `未验证` 项，尤其是相关来源、currentness、link、index、identity、owner decision 或 status evidence。
- 身份歧义、canonical path ambiguity、duplicate/coexistence ambiguity 或 topic ownership 不清。
- 缺 Maxwell/owner decision，且该 decision 会影响路径、身份、状态、source、批量范围、schema 或生命周期。
- critical link/index evidence 断裂，或 `docs/index.md` 入口会误导读者/Agent。
- source/current-practice blocker：时间敏感、外部事实、历史/现行实践或真实世界锚点缺少相称来源。
- unauthorized scope：请求跨越当前 story、非软件边界、future-story boundary、schema boundary、batch boundary 或 active workflow contract。

Scores cannot override blockers。概念文档总分再高，也不能覆盖 Hard Fail。非概念治理检查不能被概念文档六项分数替代；治理、方法论、模板、索引、runbook 或 workflow evidence 必须使用角色、权威、frontmatter、source/time、link/index、workflow contract 和专门 records 的等价检查。

每个 blocker decision 必须包含：

| 字段 | 必须记录什么 |
| --- | --- |
| evidence location | 文件、frontmatter 字段、heading、section、link、index entry、source claim、review table 或 story/workflow evidence |
| reason | 为什么阻塞 accept/promote、validated、maintained、ready 或等价通过措辞 |
| required fix or decision owner | 必须补什么、改什么、删除什么、迁移什么、请求谁确认或转给哪个 future story |
| blocking status | `blocked`、`nonblocking`、`held`、`deferred_with_reason` 或 `not_authorized` |
| next action | `targeted_revision`、`regenerate`、`reject_duplicate_or_misplaced`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason` 或 `not_authorized` |

## Weak score, partial evidence and follow-up handling

弱评分、局部缺口或非阻塞风险不能被总分掩盖。每个弱项必须转成具体 follow-up，至少说明 section/frontmatter/link/source/index evidence location。

概念文档评分处理规则：

- `10-12` 且无 blocker：可以考虑 `accept_promote`、`accepted_for_current_use` 或 `accepted_with_nonblocking_follow_up`，取决于专门 records 和未解决风险。
- `7-9` 且无 blocker：通常使用 `accepted_for_current_use`、`accepted_with_nonblocking_follow_up` 或 `targeted_revision`；不建议宣称 `validated`、`upgraded_v1` 或无条件 through-status。
- `0-6`：通常使用 `targeted_revision` 或 `regenerate`，除非证据表明应 reject、hold、defer 或 not_authorized。
- 任意 Hard Fail：只能进入阻塞/非通过决策，不得使用 accepted/promoted/validated/ready/maintained wording。

弱项转换为修订任务时，必须写清：

| 弱项类型 | Follow-up 必须定位到 |
| --- | --- |
| 问题定义或边界弱 | H1、introduction、scope/non-scope、相邻概念区分或目标读者/任务 |
| 结构/因果/判别框架弱 | 核心结构、机制链、分类表、判断链或模型边界 |
| tradeoff/失败模式弱 | tradeoff section、failure modes、误读纠偏、适用/失效条件 |
| source/currentness 弱 | `source_basis`、time_context、外部来源、核对日期或历史/当前实践 framing |
| 验证/迁移弱 | 自测题、诊断入口、迁移入口、关联模型或下游使用方式 |
| metadata/repository discipline 弱 | frontmatter 字段、`related_docs`、body links、`docs/index.md`、topic/path、quality_status 或 open questions |

`未验证` 和 `不适用` 必须区分：

- `未验证` 表示相关证据应该存在但尚未检查或无法确认。若影响 identity、source/currentness、critical link/index、owner decision、schema 或 status claim，必须阻塞 accept/promote。
- `不适用` 表示该证据确实不属于当前目标、资产类型或授权范围。必须说明为什么 out of scope，以及这是否影响 final decision。

每个 unresolved nonblocking risk 都必须有 follow-up owner、future-story dependency 或 re-review trigger。不得只写“后续优化”“可进一步完善”。

## Regenerate versus targeted revision

`targeted_revision` 适用于资产骨架、身份、来源和目标类型仍成立，问题可被局部修复的情况。`regenerate` 只在局部修复无法恢复完整性时使用。

优先使用 `targeted_revision` 的情况：

- 缺失或薄弱的 section 可明确定位。
- frontmatter、link、index 或 source 缺口可在当前授权内修复。
- 概念文档的一个或几个评分维度弱，但文档类型、身份、路径和来源仍成立。
- 非概念资产的角色、权威或记录字段局部缺失，但不需要重建资产。

使用 `regenerate` 的情况：

- 文档类型错误，例如把治理政策写成普通概念文档，或把模板写成执行报告。
- 结构坍塌，reader/Agent 无法恢复对象边界、决策链或证据位置。
- source basis 不可用，关键结论不能由现有来源或合理推断支撑。
- current-practice、historical/deprecated 或外部事实大面积失真。
- duplicate/misplacement 问题导致当前目标路径或 canonical identity 不成立。
- 局部修补会制造 silent overwrite、移除高价值旧内容或掩盖旧决策。

Regenerate decision 必须处理高价值内容：保留、迁移、总结、归档或说明为什么不能保留。若 replacement、successor、split、merge、deprecation、archive 或 old-content handling 受影响，必须引用 Migration Decision Record 和必要的 Index/Duplicate records；若 regeneration 影响旧内容、身份连续性、reference validity 或 revision/regeneration evidence，必须引用 `docs/governance/revision-regeneration-continuity-policy.md` 的 Continuity Record。具体失败示例和返工循环留给 Story 3.3，不在本文实现。

## Reject, deprecate/archive, hold, defer and not authorized

### `reject_duplicate_or_misplaced`

当候选或拟议入口不应进入正式资产体系时使用。常见原因包括 true duplicate、同主题不可区分、topic/path 错置、缺少 durable reuse context、来源不可用、质量严重不足或范围未授权。

涉及 duplicate/coexistence 风险时，必须尊重 `docs/governance/duplicate-and-coexistence-policy.md`。Final decision 可以引用 Duplicate / Coexistence Decision Record，但不得替代其 required fields。若正确处理应是 `merge`、`adjacent_link`、`narrower_document`、`explicit_coexistence` 或 `move_or_reclassify_candidate`，final decision 不得用 rejection 直接覆盖。

### `deprecate_or_archive`

当正式资产不再应作为主要当前入口，或只应作为历史、审计、迁移或旧决策证据保留时使用。必须尊重 `docs/governance/lifecycle-states.md` 和 `docs/governance/rename-migration-policy.md`。

Deprecate/archive 必须记录：

- target lifecycle outcome：`deprecated` 或 `archived`。
- reason。
- successor/replacement，或没有 successor 时的 missing-successor risk。
- remaining access expectation。
- index visibility。
- link、`related_docs`、inbound/outbound impact。
- `quality_status`、生命周期语义和正文状态说明。

`deprecated` 和 `archived` 的证据要求必须分开判断。`deprecated` 需要说明为什么资产不再作为主要当前入口、successor 或 missing-successor risk、剩余访问和索引可见性；`archived` 还必须说明正常复用预期被限制、历史/审计访问路径、替代路径和 reactivation 条件。

Archive 不是删除。不得静默删除正式资产，也不得只改一个状态词就宣称归档完成。

### `hold_for_clarification`

Hold 用于需要 Maxwell 或 owner decision 才能继续的情况，例如 identity/canonical path ambiguity、schema conflict、lifecycle/status ambiguity、batch scope uncertainty、高风险 source uncertainty、required owner decision gap 或当前授权含糊。

Hold 必须说明：

- 需要确认的问题。
- 为什么该问题阻塞当前 final decision 或写入动作。
- 可选路径及其影响。
- 谁是 decision owner。
- 确认前哪些动作不得执行。

### `defer_with_reason`

Defer 用于真实问题存在，但本轮不授权、不具备目标集、属于 future story、需要 batch readiness 或不影响当前有限结论的情况。

Defer 必须说明：

- defer reason。
- impact 是 blocking 还是 nonblocking。
- 当前工作是否可以继续。
- owner 或 future story。
- re-review trigger。

Blocking defer 不能被包装成 accepted。Nonblocking defer 也必须进入 follow-up，不得消失在 completion note 中。

### `not_authorized`

Not authorized 用于请求或必要动作越过当前授权边界，例如：

- 非软件边界：要求 runtime code、package manifest、software tests、automation、CLI/API/UI/database/deployment/CI。
- Future-story boundary：提前实现未授权或尚未到期的 Epic 6 或其他未来资产，或在已落地 Story 3.3/3.4 和 Epic 5 资产边界外扩展 rework-loop、completion-report 或 link/network policy。
- Schema boundary：新增未经授权 frontmatter 字段、machine-readable schema、validators 或 status fields。
- Batch readiness boundary：未完成 batch readiness 却要求批量状态转换、批量索引、批量迁移、批量 link repair 或批量 review。
- Active workflow contract：修改 story/sprint 状态、BMad skill、Agent 行为或 workflow hook 超出当前任务授权。

Not authorized decision 必须记录 crossed boundary、source instruction、affected forbidden scope、未执行的动作和需要的授权路径。

## Specialized records integration

Final decision 必须引用所有适用的 specialized records，并说明它们是 complete、incomplete、blocking、nonblocking、deferred_with_reason 还是 not_applicable。一次 review 可以需要多个 records。

| Specialized record or evidence | 何时必须引用 | Final decision 如何使用 | 边界 |
| --- | --- | --- | --- |
| Promotion Decision Record | candidate/workflow output 准备进入正式 `docs/`，或候选修订/替换正式资产 | 说明 promotion operator decision、quality/review decision、allowed status、index/link/old-content evidence 是否支持 accept/promote | 字段和 operator decision 仍由 candidate promotion checklist 保持权威 |
| Index Impact Decision Record | 新增、移动、重命名、改标题、改 topic、废弃、归档、candidate promotion 或导航可见性变化 | 说明 outcome 是否 `updated`、`not_applicable`、`referenced_elsewhere`、`intentionally_excluded`、`deferred_with_reason` 或 blocking | outcome 和最低字段仍由 index synchronization rules 保持权威 |
| Migration Decision Record | rename、move、retitle、topic migration、revision、replacement、split、merge、successor、deprecation、archive、reactivation | 说明 identity continuity、old-content handling、successor/replacement、link/index/lifecycle impact 是否支持 final decision | decision type 和字段仍由 rename/migration policy 保持权威 |
| Revision / Regeneration Continuity Record | targeted revision、structural upgrade、regeneration、split、merge、deprecation 或 old-content/reference-validity evidence 需要解释 changed what、why、preserved/removed 和 references validity | 说明 update mode、metadata review、old-content disposition、reference validity、quality/lifecycle impact 和 unresolved risks 是否支持 final decision | update mode taxonomy 和 continuity 字段仍由 revision/regeneration continuity policy 保持权威 |
| Duplicate / Coexistence Decision Record | duplicate、near duplicate、overlap、same-topic coexistence、topic boundary、reject 或 move/reclassify 判断 | 说明 chosen outcome 是否支持 reject、coexist、merge/link/narrow、defer、hold 或 not_authorized | taxonomy/outcome 和字段仍由 duplicate/coexistence policy 保持权威 |
| Batch Readiness Record | 目标集由规则选出、影响多个 assets/topics/status/index entries，或需要批量治理 | 说明 owner decision 是否允许当前写入，还是 `proceed`、`proceed_with_exclusions`、`clarify_before_write`、`defer_to_later_story`、`stop_for_maxwell_confirmation` 或 `not_authorized` | owner decision 和 mandatory stop conditions 仍由 batch readiness checklist 保持权威；未来 Epic 6 细化记录 |
| Source Discipline evidence | 涉及 current-practice、historical/deprecated practice、产品/标准/监管/市场状态、真实世界锚点或不可验证声明 | 说明 source/currentness 是否 blocker、nonblocking risk、not applicable 或 defer | 来源纪律和质量门禁保持 Hard Fail/声明类型权威 |

Final decision 不得把专门 record metadata 添加为目标资产的 unauthorized frontmatter fields。需要持久化的证据应写在 review record、completion evidence、目标正文的合规说明、story Dev Agent Record、future completion report 或 future runbook records 中。

## Copyable decision summary skeleton

以下 skeleton 是人工可复制的 review final decision 摘要，不是 machine-enforced schema。

```markdown
## Final decision

| Field | Value | Evidence / notes |
| --- | --- | --- |
| review decision label | <accept_promote / accepted_for_current_use / accepted_with_nonblocking_follow_up / targeted_revision / regenerate / reject_duplicate_or_misplaced / deprecate_or_archive / hold_for_clarification / defer_with_reason / not_authorized> |  |
| decision rationale |  |  |
| blocker status | <none / blocked / nonblocking / held / deferred_with_reason / not_authorized> |  |
| Hard Fail / governance blocker evidence |  |  |
| score or equivalent checks |  |  |
| `未验证` / `不适用` impact |  |  |
| required specialized records | <Promotion / Index Impact / Migration / Revision-Regeneration Continuity / Duplicate-Coexistence / Batch Readiness / Source Discipline / not_applicable> |  |
| specialized record status | <complete / incomplete / blocking / nonblocking / held / deferred_with_reason / not_applicable> |  |
| allowed status wording |  |  |
| prohibited status wording |  |  |
| `quality_status` impact |  |  |
| lifecycle impact |  |  |
| target lifecycle outcome | <not_applicable / unchanged / reviewed / validated / maintained_asset / deprecated / archived> | required when using `deprecate_or_archive` |
| index impact |  |  |
| link / related-doc impact |  |  |
| follow-up owner |  |  |
| required next action |  |  |
| re-review trigger |  |  |
| non-software / authorization boundary |  |  |
```

If the decision is not `accept_promote`, the summary must make clear what prevents accept/promote and what must happen next. If the decision is accepted with limitations, the summary must name the limitation and why it is nonblocking.

## Validation checklist and maintenance triggers

使用或维护本文时，至少检查：

1. Frontmatter baseline fields 存在，`source_basis`、`related_docs` 和 `open_questions` 是 YAML arrays。
2. 未新增 unauthorized global frontmatter fields。
3. H1、frontmatter `title`、`doc_id`、`concept`、`topic`、path 和 index entry 不互相误导。
4. 正文说明 asset role、authority、applicable scope、owner entry point、navigation treatment、relationship to main methodology、relationship to Story 3.1、relationship to Epic 0/1/2 和 future-story boundaries。
5. Decision prerequisites 覆盖 review classification、Hard Fail/blocker evidence、score/equivalent checks、`未验证`/`不适用` register 和 specialized records。
6. Allowed labels 覆盖 `accept_promote`、`accepted_for_current_use`、`accepted_with_nonblocking_follow_up`、`targeted_revision`、`regenerate`、`reject_duplicate_or_misplaced`、`deprecate_or_archive`、`hold_for_clarification`、`defer_with_reason` 和 `not_authorized`。
7. 每个 label 都说明 use when、forbidden when、required evidence、allowed status wording、`quality_status` impact、lifecycle impact、index/link impact 和 next action。
8. Blocker precedence 明确：Hard Fail、等价 governance blocker、blocking `未验证`、identity ambiguity、owner decision gap、critical link/index failure、source/current-practice blocker、unauthorized scope 都阻止 accept/promote。
9. 弱评分、局部证据和非阻塞 follow-up 不被总分掩盖，并能转成具体 section/frontmatter/link/source/index 修复项。
10. Regenerate、reject、deprecate/archive、hold、defer 和 not_authorized 的边界清楚，且不替代 `docs/governance/rework-loop-examples.md`、`docs/templates/completion-report-template.md`、Epic 5 或 Epic 6。
11. Promotion、Index Impact、Migration、Duplicate/Coexistence、Batch Readiness 和 Source Discipline records 被引用但未被替代。
12. `docs/index.md` 有 `## governance` entry，relative link 从 `docs/index.md` 可解析。
13. Changed-file links、body links 和 `related_docs` targets 存在；planned targets 只出现在 `open_questions` 或 future-story dependency。
14. Lifecycle/quality-status vocabulary 与当前治理规则兼容，不伪造 review、validation、migration、promotion、lifecycle 或 batch evidence。
15. 未创建 runtime code、package manifest、source tree、software tests、build config、automation、CLI/API/UI/database/deployment/CI、decision generator、validation script、额外 completion-report asset、额外 Epic 5 asset、Epic 6 asset 或 actual review record。

以下变化要求复核本文：

- `docs/governance/rework-loop-examples.md` 的 failure classes、repair instruction、regeneration rationale 或 resubmission vocabulary 发生实质变更。
- `docs/templates/completion-report-template.md` 的 outcome mapping、completion summary 或 blocker/follow-up 字段发生实质变更。
- related docs taxonomy、link maintenance、reusable model entry point、existing-doc reuse procedure 或 network boundary policy 发生实质变更。
- Epic 6 建立 batch governance runbook、batch change review record 或 batch completion report。
- Maxwell 明确授权 machine-readable schema、executable validation tooling、decision generator、review-record generator、lint/scoring automation 或批量审查记录。
- `quality_status`、lifecycle、frontmatter schema、promotion、index、migration、duplicate/coexistence 或 batch readiness vocabulary 发生实质治理变更。

维护触发不等于自动执行批量更新。任何批量改写、批量索引、批量状态调整、批量 link repair 或自动化工具都必须先满足相应 story、batch readiness 和 Maxwell 授权。
