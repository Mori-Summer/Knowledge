---
doc_id: methodology-concept-document-contract
title: 概念文档生成合同：输入、输出、边界与必需信息位点
concept: concept_document_generation_contract
topic: methodology
depth_mode: standard
created_at: '2026-05-25T10:52:56+08:00'
updated_at: '2026-05-27T10:29:56+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/1-1-concept-document-generation-contract.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
time_context: phase_4_epic_1_source_discipline_2026_05_25
applicability: concept_document_candidate_generation_contract
prompt_version: not_applicable
template_version: concept_document_contract_v1
quality_status: draft
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
open_questions:
  - Epic 2 frontmatter schema 落地后，是否需要把 intake decision record 或输入缺失处理进一步 schema 化？
  - Epic 3 建立 review record 和 decision policy 后，是否需要把合同中的来源/时间缺失处理映射到正式审查记录字段？
---

# 概念文档生成合同：输入、输出、边界与必需信息位点

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目的正式方法论支撑资产，用来定义概念文档候选稿生成时的输入、输出、边界、必需信息位点和验证证据。它回答的是“一个生成任务至少要收什么、产出什么、不能产出什么，以及生成结果必须让审查者看到哪些判断能力”。

本文补充 `docs/methodology/document-generation-methodology.md`、`docs/methodology/intake-and-intent-classification.md`、`docs/methodology/concept-document-template.md`、`docs/methodology/concept-document-quality-gate.md` 和 `docs/methodology/fixed-concept-generation-prompt.md`。主执行入口仍是 `docs/methodology/document-generation-methodology.md`；完整 intake 与任务意图判定入口是 `docs/methodology/intake-and-intent-classification.md`。本文不替代主规范、intake 流程、模板、质量门禁、固定 Prompt、生命周期政策、版本治理政策、导航政策，也不提前定义 Epic 2、Epic 3 或 Epic 6 的未来 schema、审查记录、completion report 或 batch runbook。

`docs/methodology/concept-document-example-catalog.md` 提供本文合同的 passing/failing calibration examples。它让 reviewer 看到必需信息位点如何映射到质量门禁证据，但不改变本文的输入/输出合同，也不创建第二套质量门禁。

本文适用于单篇概念文档候选稿的生成合同。候选稿可以来自对话、手工草稿或 BMad workflow，但只有在完成正式晋升检查后，才可以成为 `docs/` 下的正式知识资产。

本文的 owner entry point 是 `docs/index.md` 的 `methodology` 分组。Navigation treatment 为 `listed_in_docs_index`；index treatment 是在 `docs/index.md` 的 `## methodology` 下列出 `docs/methodology/concept-document-contract.md`。这个索引入口表示本文是可发现的支撑合同，不表示它成为平行主规范。

## 输入合同

生成概念文档候选稿前，accepted input 至少包含六个字段。字段名可以用自然语言表达，但信息不得缺失。

| 必填输入 | 生成前必须知道什么 | 为什么需要 |
| --- | --- | --- |
| concept name | 要生成的概念、知识点或问题对象名称 | 固定目标对象，避免把多个对象混成一篇文档 |
| topic | 预期归属主题，例如 `computer-systems`、`ai-systems`、`methodology` | 支持路径、索引和相邻文档判断 |
| user context | 用户是在什么学习、工作、阅读或问题场景下遇到它 | 决定解释角度、例子密度和下游复用位置 |
| current confusion | 用户现在最不理解、最容易混淆或最需要判断的点 | 防止生成结果退化成百科式定义 |
| intended downstream use | 用户后续想拿这份理解分析什么、判断什么、排查什么或迁移到哪里 | 决定验证入口、迁移入口和边界深度 |
| time-sensitivity notes | 主题是否涉及当前实践、标准、产品、监管、市场、历史路径或已废弃做法 | 决定来源纪律、核对日期和 open questions |

缺失输入必须保守处理：

- 如果缺失项会影响 topic、路径、来源、时间语境、候选晋升、质量状态或下游使用边界，必须先请求澄清。
- 如果缺失项不阻塞候选稿生成，但会限制质量判断，必须在候选文档的 open questions、来源限制或正文假设中记录。
- 不得静默发明 topic、时间语境、来源依据、下游用途、用户困惑或真实世界锚点。
- 只有在缺失项非治理关键、且候选文档明确标注假设时，才允许继续生成。

可选支持输入包括 preferred depth、known adjacent concepts、source constraints、preferred real-world anchor、existing related docs 和 desired example style。这些输入可以提高候选稿质量，但不得遮蔽六个必填输入。

本文只定义 accepted input 的最小合同。完整 intake decision workflow、task type、model-oriented vs pure-concept path、`standard` vs `deep` 的摄入流程，由 `docs/methodology/intake-and-intent-classification.md` 负责。该流程是人工文档治理流程，不是自动分类规则或可执行工具。

## 输出对象

required output 是一篇 Markdown concept document candidate。输出必须是可阅读、可审查、可继续修订的 Markdown 文档候选稿，而不是代码产物、CLI 命令、Web UI、API、数据库、部署配置、自动化工具、测试框架、lint/scoring 脚本、CI 流程或任何可执行软件产物。

如果候选稿准备晋升为正式 `docs/` 资产，它必须使用 YAML frontmatter 加 Markdown body，并满足主规范、模板、质量门禁、路径、索引和来源纪律要求。如果候选稿尚未准备晋升，它也必须显式标注 candidate status、未验证项、open questions 或后续检查入口，不能假装已经是最终正式资产。

对话中生成的文本、`_bmad-output/` 草稿、story 输出或临时候选文档都不是自动发布的正式知识资产。它们必须经过 frontmatter、path、topic、index、quality gate、source/time context、related docs、open questions 和 operator decision 检查后，才可被正式纳入 `docs/`。

## 必需信息位点

概念文档候选稿的标题、顺序和小节名称可以调整，但以下信息位点不得静默缺失。

### 通用必需信息

所有候选稿都必须覆盖：

- problem definition、概念结论，或 naming/classification problem。
- object boundary：什么属于本文对象。
- non-object scope：什么不属于本文对象。
- adjacent concept distinction：至少说明最容易混淆的相邻概念，以及判别差异。
- applicable conditions：在什么条件、上下文或使用场景下本文判断成立。
- verification entry：读者如何检查自己是否真正理解。
- transfer entry：这份理解可以迁移到哪些相邻问题、模型或判断中。
- real-world anchor：至少给出可定位的真实组织、系统、产品、标准、制度、论文、公开实践或具体场景；若主题确实不依赖真实锚点，必须说明不适用理由。
- self-test questions：用于检验解释、边界判断、误读纠偏、诊断、预测或迁移能力，不得只考术语记忆。
- open questions：记录候选稿仍未解决、待核查、待澄清或后续 story 才会收束的问题。

### 模型型概念文档的必需信息

如果主题具有稳定结构、机制、主链路、失败模式、选型价值或调试价值，候选稿必须走模型型概念文档路径，并覆盖：

- 问题定义：它试图解决什么结构性问题。
- 核心结构、部件、层次或分面框架。
- mechanism、main path、causal chain 或可调用判断链。
- tradeoff、applicable conditions、失效条件和 failure modes。
- 应用场景，以及现实里为什么有人关心它。
- 当前实践、历史路径或替代路径；仅在主题涉及这些内容时展开，但不得把历史做法写成当前推荐。

### 纯概念文档的必需信息

如果主题主要解决命名、分类、边界、相邻概念区分或上层模型术语稳定性，候选稿必须走纯概念文档路径，并覆盖：

- naming/classification problem：为什么需要单独命名或区分这个概念。
- 什么算它、什么不算它。
- discrimination frame 或 classification frame：读者如何判断某个对象是否属于它。
- adjacent concept distinction：相邻概念之间的判别差异，而不是名词清单。
- 代表性例子、反例、常见误读、false equivalences 或 misuse paths。
- 它会进入哪些更大的模型、解释框架、判断流程或后续文档。

## 来源与时间语境要求

候选稿只要涉及当前实践、标准、产品行为、监管、市场状态、历史实践、旧方案、已废弃路径或替代建议，就必须显式处理 source/time-context expectations：

- 写明核对日期或时间语境。
- 区分已核查事实、基于来源的归纳、作者推断、示例化说明和待核查问题。
- 优先使用一手、官方或可定位来源；来源不足时记录限制，而不是把推断写成事实。
- 说明历史路径为什么旧、局限在哪里、当前替代路径是什么，以及何时旧路径仍可能有解释价值。
- 确保正文、frontmatter `source_basis`、`time_context`、参考资料、open questions 和质量状态语义一致。

更细的 source discipline 与 real-world anchor policy 由 `docs/methodology/source-discipline-and-real-world-anchor-policy.md` 承担。本文只定义生成合同中的最低来源和时间语境期望；遇到 current-practice claim、historical/deprecated practice、不可验证声明或锚点 adequacy 判断时，应回到该专门政策细查。

## Mandatory vs Adaptable

Mandatory 是候选稿必须可审查的信息位点和治理证据；Adaptable 是在不丢失信息位点的前提下可以随主题调整的表达形式。

| 层级 | 不得缺失 | 可以调整 |
| --- | --- | --- |
| 信息位点 | problem/naming definition、boundary、non-object scope、adjacent distinction、frame、conditions、failure/misreading、verification、transfer、anchor、self-test、open questions | 小节标题、排序、段落密度和是否合并相邻小节 |
| 文档路径 | 模型型与纯概念路径必须先分清；路径差异决定 frame 类型 | 复杂主题可以在正文中解释混合边界，但不能逃避路径判断 |
| 深度模式 | `standard` 与 `deep` 都必须保留核心合同面 | `deep` 增加证据密度、来源约束、例子数量和判断链深度；不新增另一套合同 |
| 表达形式 | required information 必须在正文中可定位 | 可以使用列表、表格、图示说明、例子组、反例组或 discipline-specific explanation style |
| 不适用处理 | 信息位点不适用时必须写明理由 | 理由可以放在对应小节、open questions 或来源限制中 |

标题可以变化，信息位点不能消失。候选稿可以把“对象边界”和“相邻概念区分”合并，也可以把“验证入口”和“自测题”放在同一节；但审查者必须能直接定位到每个必需信息位点。

`standard` 与 `deep` 改变的是密度和证据负担，不改变合同表面。`standard` 也必须有边界、验证、迁移和 open questions；`deep` 则需要更强的分面结构、来源纪律、真实锚点、失败模式、误读网络和迁移判断。

模型型路径和纯概念路径的差异必须保留。模型型主题不能缺少结构或机制链；纯概念主题不能被强行写成伪机制文档，也不能缺少分类/判别框架、例子/反例和误读纠偏。

## 出界输出与下游边界

本文不授权生成、修改或要求任何软件产物。以下输出明确出界：

- code artifacts、package manifests、source trees、software tests、build configs、CI/CD、deployment assets。
- CLI commands、Web UI、API、database assets、runtime workflows、automation tools、lint/scoring scripts 或 executable validation。
- Python、`uv`、Typer、Pydantic、pytest、JavaScript、frontend framework 或任何运行时工程脚手架。

本文也不创建下游治理资产：

- 不替代 `docs/methodology/intake-and-intent-classification.md`；完整任务类型、路径、深度和缺失输入流程由该资产负责。
- 不把 `docs/methodology/concept-document-example-catalog.md` 的示例片段当成 standalone formal concept documents；该资产只提供 reviewer calibration examples。
- 不替代 `docs/methodology/source-discipline-and-real-world-anchor-policy.md`；来源纪律、时间语境、历史/废弃实践和真实世界锚点 adequacy 的细则由该资产负责。
- 不定义 Epic 2 的 frontmatter schema、`doc_id`、topic、path、naming、candidate promotion 或 index synchronization rules。
- 不定义 Epic 3 的 review record、document decision policy、rework loop 或 completion report template。
- 不定义 Epic 6 的 batch governance runbook、batch review record 或 batch completion report。

本文不授权 batch generation、batch migration、bulk status changes、批量索引重构或自动化验证。批量工作必须先满足 `docs/governance/batch-readiness-checklist.md` 和后续 Epic 6 规则。

## 完成与验证证据

未来生成或审查概念文档候选稿时，至少按以下问题验证本文合同是否被满足：

| 验证项 | 通过证据 |
| --- | --- |
| 输入合同 | 六个 required input 已提供，或缺失项按澄清、open questions、明确假设处理 |
| 输出对象 | 产物是一篇 Markdown concept document candidate，不是软件、工具、命令或自动化产物 |
| 候选边界 | 候选稿未伪装成已发布正式资产；晋升前检查项清楚 |
| 必需信息位点 | problem/naming、boundary、adjacent distinction、frame、conditions、failure/misreading、verification、transfer、anchor、self-test、open questions 可定位 |
| 路径差异 | 模型型与纯概念路径的 frame 和缺口处理不同，且未硬造伪结构 |
| Mandatory vs adaptable | 可调的是标题、顺序、密度和表达方式；必需信息位点没有静默缺失 |
| 来源与时间 | 当前实践、历史路径、标准、产品、监管或市场相关内容有时间语境、来源限制和事实/推断区分 |
| 出界边界 | 未生成代码、CLI、API、UI、数据库、部署、CI、软件测试、自动化或未来 story 拥有的治理资产 |

对准备晋升到 `docs/` 的候选稿，还必须继续执行主规范、模板、质量门禁、frontmatter、路径、索引、related docs、open questions、生命周期和版本治理检查。本文本身不替代这些门禁。

## 参考资料

- [统一概念文档规范：新建、升级、审查与仓库集成](./document-generation-methodology.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](./intake-and-intent-classification.md)
- [概念文档样例目录：合格、不合格与质量门禁证据](./concept-document-example-catalog.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](./source-discipline-and-real-world-anchor-policy.md)
- [统一概念文档模板](./concept-document-template.md)
- [统一概念文档质量门禁](./concept-document-quality-gate.md)
- [固定概念文档生成 Prompt](./fixed-concept-generation-prompt.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](./governance-asset-boundary-policy.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](../governance/agent-behavior-constraints.md)
- [治理资产导航、索引与入口归属政策](../governance/governance-asset-navigation-policy.md)
