---
doc_id: governance-topic-path-naming-policy
title: Topic、文件命名与路径归属策略：正式 docs 资产的位置、命名与一致性规则
concept: topic_path_naming_policy
topic: governance
depth_mode: standard
created_at: '2026-05-26T09:28:34+08:00'
updated_at: '2026-05-28T10:04:11+08:00'
source_basis:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/2-2-topic-path-naming-policy.md
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
time_context: phase_4_epic_4_revision_regeneration_continuity_policy_2026_05_28
applicability: formal_docs_topic_path_naming_and_repository_placement_governance
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/index.md
  - docs/governance/frontmatter-schema.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/duplicate-and-coexistence-policy.md
  - docs/governance/revision-regeneration-continuity-policy.md
open_questions:
  - Story 2.3 建立 candidate promotion workflow 后，是否需要把 promotion evidence 中的 topic/path/name 判断抽成固定记录项？
  - Story 2.4 建立详细 index synchronization rules 后，是否需要把本文中的 index grouping 检查拆成更细的 add/move/rename/title/topic/status 同步规则？
  - Story 2.5 建立 rename/migration policy 后，是否需要把 legacy filename、path exception 和 link-impact 处理迁出本文并改为引用迁移政策？
  - Story 2.6 已建立 duplicate/coexistence policy 后，是否还需要为同 topic 近义文档、重叠概念和共存条件新增更细判定表？
---

# Topic、文件命名与路径归属策略：正式 docs 资产的位置、命名与一致性规则

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目中正式 `docs/` 资产的 canonical topic、path 和 naming governance asset。它回答的是：一篇正式知识资产、方法论资产或治理资产准备创建、晋升、移动、重命名或复核时，应放在哪个正式路径，topic 如何判定，文件名和 frontmatter `concept` 应采用什么形状，以及 title、topic、concept、filename、path 和 `docs/index.md` grouping 如何保持一致。

本文适用于正式 `docs/` 资产的仓库放置和命名判断，包括：

- `docs/{topic}/` 下的普通正式概念/知识资产。
- `docs/methodology/` 下的主方法论、模板、质量门禁、fixed prompt、playbook 和方法论支撑资产。
- `docs/governance/` 下的治理政策和规则资产。
- `docs/_reports/` 下已经批准保留的报告资产。
- 未来明确批准的 `docs/templates/` 和 `docs/runbooks/` 资产。
- 从 `_bmad-output/`、对话输出或候选稿进入正式 `docs/` 前的 topic/path/name 判断。

本文补充 `docs/methodology/document-generation-methodology.md`、`docs/methodology/concept-document-contract.md`、`docs/methodology/intake-and-intent-classification.md`、`docs/methodology/concept-document-quality-gate.md`、`docs/governance/frontmatter-schema.md`、`docs/governance/governance-asset-navigation-policy.md`、`docs/governance/lifecycle-states.md` 和 `docs/governance/batch-readiness-checklist.md`。主方法论仍负责正式概念文档的新建、升级、审查和仓库集成流程；Story 2.1 frontmatter schema 仍负责 required fields、YAML array、`doc_id` 身份和 frontmatter/body 一致性。本文只负责 topic、正式路径、文件名、slug、path ownership 和跨信号一致性。

本文不替代后续 Epic 2、Epic 3、Epic 4、Epic 5 或 Epic 6 的专门资产：

- Story 2.3 负责 candidate promotion workflow 和 promotion evidence。
- Story 2.4 负责详细 `docs/index.md` synchronization rules。
- Story 2.5 负责 rename、migration、split、merge、deprecation、successor 和 link-impact 决策。
- Story 2.6 负责 duplicate concept 和 same-topic coexistence policy。
- Epic 3 负责 review record、document decision、rework loop 和 completion report 模板。
- Story 4.1 负责 revision/regeneration continuity、update mode taxonomy、old-content handling 和 reference validity。
- Story 4.2 负责 sidecar boundary policy。
- Story 4.3 负责 legacy migration guide。
- Epic 5 负责 related docs taxonomy、link maintenance 和 reusable model entry points。
- Epic 6 负责 batch governance runbook、batch review record 和 batch completion report。

本文自身的 owner entry point 是 `docs/index.md` 的 `governance` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## governance` 下列出 `docs/governance/topic-path-naming-policy.md`。这些归属信息写在正文中，不是新的全局 frontmatter 字段。

当前 `quality_status: draft` 是保守治理状态。原因是本文是 Epic 2 的首版 topic/path/name policy；Story 2.3-2.6 已细化 promotion、index synchronization、migration 和 duplicate/coexistence，后续 Epic 3、Epic 5 和 Epic 6 仍会细化 review evidence、related docs 和 batch execution。

## 正式路径与资产类别

普通正式概念/知识资产默认路径是：

```text
docs/{topic}/{kebab-case-slug}.md
```

其中 `{topic}` 是正式 topic directory，使用 `kebab-case`；`{kebab-case-slug}` 是文件导航 slug，也使用 `kebab-case`。例如当前仓库中的 `docs/computer-systems/memory-order.md`、`docs/programming-languages/coroutine.md` 和 `docs/ai-systems/agent.md`。

不是所有正式资产都属于 `docs/{topic}/` 普通概念路径。资产类别必须先判定，再决定路径：

| 资产类别 | 默认路径 | 归属规则 |
| --- | --- | --- |
| 普通正式概念/知识资产 | `docs/{topic}/{kebab-case-slug}.md` | 用于解释、建模、区分或复用某个知识对象；topic 是仓库主题分组。 |
| 主方法论和方法论支撑资产 | `docs/methodology/{kebab-case-slug}.md` | 用于定义概念文档生成、升级、审查、模板、质量门禁、fixed prompt、playbook 或方法论支撑规则。 |
| 治理资产 | `docs/governance/{kebab-case-slug}.md` | 用于约束 Agent 行为、frontmatter、生命周期、导航、版本、批量 readiness、topic/path/name 或其他仓库治理规则。 |
| 报告/归档性正式资产 | `docs/_reports/{kebab-case-slug}.md` | 用于保留已批准的报告、迁移记录或历史审计入口；`docs/_reports/` 是受控特殊目录，可对应 frontmatter `topic: reports`；不得伪装成普通概念文档。 |
| 未来模板资产 | `docs/templates/{kebab-case-slug}.md` | 只有在后续 story 或 Maxwell 明确批准该目录和入口后使用。 |
| 未来 runbook 资产 | `docs/runbooks/{kebab-case-slug}.md` | 只有在后续 story 或 Maxwell 明确批准该目录和入口后使用。 |
| BMad workflow/planning output | `_bmad-output/...` | 可作为 source context、story evidence、planning record 或 candidate source；不是正式 `docs/` 资产。 |

`_bmad-output/` 下的 PRD、Architecture、Epics、Story、readiness report、review output、completion record 和草稿不能被当成正式 `docs/` 资产直接索引或发布。它们可以出现在 `source_basis` 中作为实施依据，但这不会改变它们的 workflow output 生命周期。进入正式 `docs/` 前，必须按 Story 2.3 的 candidate promotion workflow 完成 promotion 判断，并显式检查 target path、frontmatter、topic、filename、source/time context、quality status、related docs、open questions 和 index impact。

正式非概念资产是否进入 `docs/governance/` 或 `docs/methodology/`，按职责判断：

- 如果资产定义仓库治理规则、Agent 约束、生命周期、导航、frontmatter、topic/path/name、batch readiness、版本治理或 future review/completion policy，应进入 `docs/governance/`。
- 如果资产定义概念文档生成、升级、审查、模板、质量门禁、fixed prompt、学习/建模方法或方法论执行支撑，应进入 `docs/methodology/`。
- 如果一个文件既像方法论又像治理规则，先看 owner entry point 和主要约束对象；影响主概念文档执行合同的，通常归 `docs/methodology/`；约束跨资产仓库治理的，通常归 `docs/governance/`。无法可靠判断时，记录 open question 或请求 Maxwell 确认。

本文只定义路径归属规则，不执行 candidate promotion procedure、详细 index synchronization、rename/migration、duplicate/coexistence、batch normalization 或批量路径修正。

## Topic 选择规则

`topic` 是仓库分组和 frontmatter 分类，不是随意增加的标签云。正式资产的 `topic` 必须帮助读者和 Agent 判断它属于哪个知识区域、应该从哪个 index section 发现、与哪些相邻文档比较，以及是否处在正确路径下。

Topic 判定必须至少检查以下上下文：

1. 用户意图和 reuse context：用户后续要拿文档解释、判断、排查、迁移或治理什么。
2. 候选文档或目标资产的对象边界：正文到底在处理哪个知识对象或治理对象。
3. Neighboring documents：候选 topic 目录下已有文档是否提供相邻概念、命名模式和边界校准。
4. Existing topic directories：当前仓库已有 topic 目录是否已经覆盖该对象域。
5. `docs/index.md` grouping：总索引是否已有对应 section，以及同 section 条目的命名和主题边界。
6. `related_docs` 和正文链接：相关资产是否指向某个更合理的 topic 或资产类别。
7. 方法论/治理边界：该资产是否其实属于 `docs/methodology/`、`docs/governance/`、`docs/_reports/`、future `docs/templates/` 或 future `docs/runbooks/`。

Topic 选择时，先优先复用现有 topic。不要因为一个文档有新的表达角度就创建新 topic；只有当现有 topic 都会误导读者、相邻文档无法承载、`docs/index.md` grouping 无合适入口，且该主题有稳定扩展空间时，才考虑新 topic。新 topic 会影响导航和未来网络边界，通常需要在 completion evidence 或 open question 中说明理由。

Topic 歧义必须显性处理。可接受结果只有以下几类：

| 情况 | 允许处理 |
| --- | --- |
| 有足够证据选择一个 topic | 选择该 topic，并在 completion evidence、review evidence 或正文中记录简短理由。 |
| 两个 topic 都合理但一个更贴合 reuse context | 选择更贴合的 topic，记录 rejected alternative 和理由。 |
| 主题实际属于 governance/methodology/report/template/runbook | 放入对应特殊路径，不强行进入普通 `docs/{topic}/`。 |
| 信息不足以可靠判断 | 记录 open question 或请求 Maxwell 确认；不得静默选择。 |
| 可能是 duplicate concept、same-topic coexistence、split/merge 或 successor 问题 | 记录 future-story dependency，交给 Story 2.5 或 Story 2.6；不得在本文范围内擅自合并或迁移。 |

未解决 topic ambiguity 会阻塞 strong ready、validated、maintained 或等价通过声明。它必须出现在 `open_questions`、review evidence、completion evidence、story Dev Agent Record 或未来 promotion record 中。不得一边把 topic 当成不确定，一边更新索引和质量状态来暗示已完成归属判断。

## 文件名、slug 与 `concept` 命名

Topic directory 和 Markdown filename 必须使用 `kebab-case`。Frontmatter `concept` 必须使用稳定 `snake_case`。Frontmatter key 也使用 `snake_case`，但本文重点约束 `concept` 与路径/文件名的关系。

文件名 slug 的选择规则：

- 使用简短、稳定、能表达核心对象的英文或技术通用词。
- 优先选择对象名或概念名，而不是整句标题。
- 使用小写字母、数字和连字符；避免空格、下划线、大小写混排和标点。
- 避免营销化、临时化、任务化或评价性词语，例如 `best`、`new`、`final`、`improved`。
- 不把日期写入普通概念文档 slug，除非资产类别是报告、归档、事件记录或后续政策明确要求日期。
- 不把 depth、quality status、review status、prompt version 或 story number 写入普通概念文档 slug。
- 不为了匹配新 H1 或标题措辞而静默重命名文件。

`concept` 的选择规则：

- 使用稳定 `snake_case`，表达核心概念或资产键。
- 不复制完整 H1、长标题、索引标题或营销式描述。
- 不把 `concept` 当成文件名缓存；文件移动或 slug 调整不自动改变 `concept`。
- 不把 `concept` 当成 `doc_id`；`doc_id` 是稳定文档身份，由 `docs/governance/frontmatter-schema.md` 管理。
- 不为了让 `concept` 看起来贴近展示标题而静默改写。

`doc_id`、title、path、filename、H1、index title 和 `concept` 是不同信号：

| 信号 | 作用 | 不得混用为 |
| --- | --- | --- |
| `doc_id` | 稳定文档身份，追踪资产连续性 | 文件名、路径、标题、topic 或 `concept` 的自动派生值 |
| `title` / H1 | 人类可读标题和正文入口 | 身份字段或 slug 生成器 |
| `concept` | 稳定概念/资产键 | 整句标题、`doc_id`、路径缓存或 index title |
| filename slug | 导航 handle 和文件系统定位 | 稳定身份或质量状态 |
| path | 仓库归属和资产类别 | 概念身份或标题证据 |
| `docs/index.md` title/grouping | 导航入口和主题发现 | frontmatter 替代品或 promotion 证明 |

无效命名行为包括：

- 静默把文件重命名为新 H1/title 的直译。
- 静默把 `concept` 改成展示标题的 `snake_case` 版本。
- 根据 story 名、用户临时措辞或任务名生成正式文件名。
- topic/path 上下文缺失时发明 topic、path 或 slug。
- 因为文件名变化而修改 `doc_id`。
- 用批量规则统一旧文件名、目录、topic、frontmatter 或索引项；批量 normalization 必须先满足 `docs/governance/batch-readiness-checklist.md` 并获得明确授权。

## 一致性与例外处理

正式资产进入或留在 `docs/` 体系时，title、H1、topic、concept、filename、path 和 index grouping 必须互相支持，而不是互相解释不了。

最低一致性要求：

| 检查面 | 必须成立 |
| --- | --- |
| title 与 H1 | 含义一致，可以有可读性差异，但不得表达不同对象。 |
| topic 与目录 | Frontmatter `topic` 与所在目录或受控资产类别映射一致；治理/方法论资产不强行伪装成普通知识 topic，`docs/_reports/` 可以使用 `topic: reports`。 |
| path 与资产类别 | 普通概念资产在 `docs/{topic}/`；方法论、治理、报告、future templates/runbooks 分别使用对应正式路径。 |
| `concept` 与对象边界 | `concept` 指向稳定对象或资产键，不复制完整标题，不与相邻概念混淆。 |
| filename 与导航 | filename slug 足够描述核心对象，能稳定导航，但不承担身份职责。 |
| index grouping | `docs/index.md` section 与 path/topic 不误导读者或 Agent。 |
| `related_docs` 与正文链接 | 链接目标存在且关系合理；缺失目标写入 `open_questions` 或 future-story dependency。 |

发现不一致时，必须记录：

- affected file(s)。
- 冲突信号，例如 title/path/topic/index grouping 不一致。
- 选择的 disposition：修正、保留为受控例外、记录 open question、请求 Maxwell 确认、或 defer to future story。
- 处理理由。
- unresolved follow-up owner 或 story，如果当前 story 不授权修复。

Legacy exception 不是静默通过。旧文件、历史 slug、旧 index title 或旧 frontmatter 可以暂时保留，但必须不误导当前使用；如果它影响 identity、path ownership、topic boundary、index discovery、quality status 或 lifecycle 语义，应记录为 open question、review finding 或 future migration candidate。

移动、重命名、拆分、合并、废弃、归档和 successor 处理不由本文直接执行。本文只要求在发现相关信号时不要静默处理；详细决策和 link impact 由 Story 2.5 负责。重复概念、同 topic 共存和相邻主题重叠的细则由 Story 2.6 负责。

## 检查清单

新建、晋升、移动、重命名或审查正式 `docs/` 资产前，至少检查：

1. 任务类型已经判定：新建、升级、审查、index-only、methodology maintenance、governance maintenance、planning artifact、archive/deprecation 或 batch governance。
2. 资产类别已经判定：普通概念/知识资产、methodology asset、governance asset、report、future template、future runbook 或 `_bmad-output/` workflow output。
3. 目标 path 是否符合资产类别：普通概念资产走 `docs/{topic}/{kebab-case-slug}.md`，特殊资产走对应正式目录。
4. `_bmad-output/`、对话输出或候选稿是否仍是 candidate；如果要进入 `docs/`，是否记录 promotion 检查或 Story 2.3 dependency。
5. Topic 是否通过 reuse context、neighboring documents、existing topic directories、`docs/index.md` grouping 和 related docs 检查。
6. Topic 歧义是否已经解决并记录理由，或作为 open question/future-story dependency 暴露。
7. Topic directory 和 Markdown filename 是否使用 `kebab-case`。
8. Frontmatter `concept` 是否使用稳定 `snake_case`，且没有复制完整标题或路径。
9. `doc_id` 是否保持稳定，并且没有根据 filename、title、path、topic、H1 或 index title 自动改写。
10. Title、H1、topic、concept、filename、path 和 index grouping 是否互相一致。
11. `docs/index.md` 是否需要新增或更新；如果不适用，是否有明确理由。
12. `related_docs` 和正文链接目标是否存在；不存在或计划中目标是否写入 `open_questions`。
13. 是否没有执行批量 rename、path migration、topic correction、frontmatter normalization、index-wide restructuring 或 duplicate/coexistence cleanup。
14. 是否没有新增 runtime code、package manifest、source tree、software tests、build config、automation、CLI/API/UI/database/deployment/CI 或 executable validation tooling。

任一检查失败时，不能宣称 strong ready、validated、maintained、accepted、promoted 或等价通过状态。可以继续作为 draft/candidate，但缺口和后续处理必须可见。

## 维护触发点

以下变化要求复核本文：

- Story 2.3 建立 candidate promotion workflow 后，复核本文中的 promotion placeholder 和最低 promotion 检查。
- Story 2.4 建立 detailed `docs/index.md` synchronization rules 后，复核本文中的 index grouping、一致性检查和 navigation treatment 表述。
- Story 2.5 建立 rename/migration/split/merge/deprecation/successor/link-impact policy 后，复核 filename/path exception 和 legacy handling。
- Story 2.6 已建立 duplicate concept 与 same-topic coexistence policy；后续若该政策改变 topic ambiguity 或 overlap handling，复核本文。
- Epic 3 建立 review record、document decision policy、rework loop 或 completion report template 后，复核本文要求的 rationale、open question 和 evidence 记录位置。
- Story 4.1 已建立 revision/regeneration continuity policy；后续若该政策更新，复核 path、identity、old-content handling 和 reference validity 规则。
- Story 4.2 或 Story 4.3 建立 sidecar boundary 或 legacy migration guide 后，复核 sidecar path 和 legacy exception 规则。
- Epic 5 建立 related docs taxonomy、link maintenance 或 reusable model entry points 后，复核 related docs、neighboring documents 和 reuse context 的判断顺序。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report 后，复核本文的 batch exclusion 和 non-output 边界。
- Maxwell 明确授权新 topic taxonomy、machine-readable schema、executable validation tooling、lint/scoring automation、批量 normalization 或 index restructuring。

本文当前不执行批量 normalization，不创建 machine-readable validator，不创建或修改 candidate promotion、index synchronization、rename/migration、duplicate/coexistence 等专门资产，不改变主方法论、frontmatter schema、quality gate、fixed prompt、lifecycle policy、batch readiness policy 或 `docs/index.md` 同步细则。

## 参考资料

- [Knowledge Docs Index](../index.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](./frontmatter-schema.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](../methodology/concept-document-contract.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](../methodology/intake-and-intent-classification.md)
- [统一概念文档质量门禁](../methodology/concept-document-quality-gate.md)
- [治理资产导航、索引与入口归属政策](./governance-asset-navigation-policy.md)
- [重复概念与同主题共存治理：合并、相邻链接、窄化、保留与拒绝决策](./duplicate-and-coexistence-policy.md)
- [修订、重生成与版本连续性策略：更新模式、旧内容处理、身份连续性与引用有效性](./revision-regeneration-continuity-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](./lifecycle-states.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](./batch-readiness-checklist.md)
