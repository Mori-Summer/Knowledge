---
doc_id: methodology-document-generation-methodology
title: 统一概念文档规范：AI 生成、升级、审查与仓库集成
concept: unified_concept_document_spec
topic: methodology
depth_mode: standard
created_at: '2026-04-01T20:02:19+08:00'
updated_at: '2026-06-26T00:00:00+08:00'
source_basis:
  - maxwell_principle_model_first_concept_docs_2026_06_26
  - methodology_repository_practice
  - consolidation_review_2026_04_01
  - pure_concept_doc_split_review_2026_04_08
  - methodology_folder_consolidation_2026_06_18
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-asset-governance.md
  - docs/governance/docs-change-governance.md
  - docs/templates/governance-record-templates.md
  - docs/runbooks/batch-governance-runbook.md
time_context: source_basis_navigation_alignment_and_model_first_principle_2026_06_26
applicability: ai_concept_doc_creation_upgrade_review_and_repository_integration
prompt_version: concept_generation_prompt_v5
template_version: unified_spec_v4
quality_status: maintained_asset
related_docs:
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-asset-governance.md
  - docs/governance/docs-change-governance.md
  - docs/templates/governance-record-templates.md
  - docs/runbooks/batch-governance-runbook.md
open_questions:
  - 后续是否需要在一次真实批量文档生成任务中验证合并后的主规范可执行性？
---

# 统一概念文档规范：AI 生成、升级、审查与仓库集成

`docs/methodology/` 是给 AI 执行正式文档任务看的规范目录，不是给人阅读的方法论文章集合。合并后默认只保留三类规范：

| 文件 | 角色 | AI 读取时机 |
| --- | --- | --- |
| `document-generation-methodology.md` | 主执行规范；负责任务摄入、生成合同、正文骨架、固定入口、仓库集成和方法论维护边界 | 新建、升级、审查、索引、方法论维护时默认先读 |
| `concept-document-quality-gate.md` | 质量门禁；负责 Hard Fail、六项评分、审查输出和状态声明限制 | 审查、验收、升级 gap analysis、质量状态变更时读 |
| `source-discipline-and-real-world-anchor-policy.md` | 来源纪律；负责当前实践、历史路径、不可验证声明和真实世界锚点 | 涉及事实、当前性、产品/标准/监管/市场/历史路径时读 |

已合并或删除的旧文件不再是执行入口：

| 旧文件 | 继任位置 |
| --- | --- |
| `intake-and-intent-classification.md` | 本文的任务摄入、缺失输入、停止条件和 `_bmad-output` 边界 |
| `concept-document-contract.md` | 本文的输入合同、输出合同、必需信息位点和候选边界 |
| `concept-document-template.md` | 本文的正文骨架、深度模式和特殊主题规则 |
| `fixed-concept-generation-prompt.md` | 本文的固定触发词 |
| `governance-asset-boundary-policy.md` | 本文的方法论维护边界 |
| `concept-document-example-catalog.md` | `concept-document-quality-gate.md` 的审查校准语义 |
| `methodology-operator-guide.md` | 本文的合并记录；不再保留历史编排说明 |
| `learning-new-things-playbook.md` / `cognitive-modeling-playbook.md` | 本文的最小建模要求；不再作为 AI 执行规范 |

## 1. 适用范围与非目标

本文适用于：

- `docs/{topic}/` 下正式概念文档的新建、升级、审查和仓库集成。
- 现有正式文档的结构补强、来源补强、质量门禁修复和索引同步。
- 方法论目录自身的维护，尤其是避免继续新增平行规范。
- 从对话、草稿或 `_bmad-output/` 候选内容晋升为正式 `docs/` 资产前的判断。

本文不授权：

- 生成代码、CLI、API、UI、数据库、部署、CI、软件测试、自动化检查器或运行时工具。
- 把 `_bmad-output/`、review output、story 文件或对话文本直接加入 `docs/index.md`。
- 无明确目标集地批量移动、删除、重命名、改状态或重写多个 topic。
- 为了保留旧结构继续新增 methodology 支撑文件。

## 2. 执行顺序

AI 处理 Knowledge 文档任务时按固定顺序执行：

1. 读取当前用户指令、目标文件、允许范围和最新上下文。
2. 判断任务类型、资产层级和是否涉及批量范围。
3. 若是正式概念文档，判断文档路径：模型型概念文档或纯概念文档。
4. 判断展开密度：`standard` 或 `deep`。
5. 检查输入是否足够；不足时澄清、记录 open question、显式假设、延期或停止。
6. 写作、升级或审查正文。
7. 按质量门禁、来源纪律、frontmatter、路径、索引和链接完成验证。
8. 汇报变更、验证证据、未解决项和非软件边界。

不要先写正文、先改索引或先改状态字段。

## 3. 任务类型与停止条件

先把请求归入一个主任务类型。混合任务先拆分；拆不开时按更严格的类型处理。

| 任务类型 | 允许范围 | 最低验证证据 |
| --- | --- | --- |
| `new document creation` | 新建一个正式 Markdown 资产，必要时同步索引和相邻链接 | frontmatter、路径/topic、正文合同、来源、质量门禁、索引 |
| `upgrade` | 修改一个既有正式资产和必要的窄同步文件 | 保留 `doc_id`、修复 gap、记录来源/索引/状态影响 |
| `review` | 默认只输出审查结论；除非用户要求修复 | 前置分类、Hard Fail、评分或等价治理检查、最终结论 |
| `index-only update` | 只改 `docs/index.md` 和必要 metadata | 目标存在、标题/路径/topic 一致、无 hidden promotion |
| `methodology maintenance` | 改本文、质量门禁或来源纪律，必要时同步索引/引用 | 角色边界、版本影响、下游影响、删除/合并映射 |
| `planning artifact` | 改 `_bmad-output/` 的 story/report/status | workflow 边界，不自动晋升正式 `docs/` |
| `archive/deprecation` | 目标文件、successor、索引和链接影响 | 废弃理由、剩余访问方式、替代入口 |
| `batch governance` | 多文件、多 topic、多状态或多索引入口 | 先通过 batch readiness，再写目标集 |

以下情况必须停止或请求确认：

- 目标文件、topic、`doc_id`、资产层级或 owner entry point 不清。
- 候选内容被要求直接发布为正式资产，但缺少 promotion 证据。
- 要声明 current、reviewed、validated、maintained、deprecated 或 archived，但来源和状态证据不足。
- 要批量处理，却没有目标集、排除项、恢复策略和 owner decision。
- 需要删除、重命名、迁移或废弃正式资产，但 successor/index/link impact 不清。
- 出现代码、自动化、测试、CI、部署或运行时工具要求，但用户没有明确授权。

## 4. 正式概念文档分型

路径判断先于深度判断。

### 4.0 原理优先与模型优先总则

所有正式概念文档默认服务于**本质理解**，不是服务于“会调用现成工具”。但“本质理解”在不同文档路径下含义不同：

- 模型型、机制型、算法型、系统型、工程型、制度运行型文档，必须原理优先、模型优先。
- 纯概念、纯定义、命名区分、分类边界型文档，不强行要求机制、数学模型、因果链或最小可重建实现；它们的本质是稳定定义、判别轴、边界、例子/反例和误读纠偏。

模型型文档的写作顺序必须是：

```text
本质问题
  -> 抽象对象
  -> 核心变量 / 状态 / 约束 / 不变量
  -> 生成机制 / 因果链 / 数学模型 / 判别模型
  -> 失败边界与 tradeoff
  -> 用法、API、代码、操作步骤或工程接口
```

如果主题涉及库、API、工具、框架、算法实现或产品功能，接口和代码只能作为底层模型的投影或验证入口，不能成为正文主轴。文档要优先回答“为什么这样建模、它靠什么成立、如何从零重建一个最小版本、在哪些条件下会失败”，再回答“怎么调用、参数怎么填、示例怎么写”。

“会造”是概念文档的理解目标。根据主题不同，`会造` 可以表示：

- 能从第一性原理重建一个最小算法、最小系统、最小协议、最小数学模型或最小判别器。
- 能写出核心状态、变量、约束、更新规则、能量函数、数据流、控制流或制度结构。
- 能解释关键设计为什么必要，去掉某个部件会退化成什么。
- 能预测边界条件、失败模式和替代设计的后果。
- 对纯概念文档，能生成稳定定义、判别规则、边界测试和反例，而不只是背一个定义或列正例。

用法、代码、命令、API 表、产品界面和操作步骤应放在原理模型之后，并且只保留能帮助理解模型映射、验证模型或暴露边界的内容。若用户明确要求“只要用法速查”，也必须标明这是受限输出，不得把它伪装成完整概念文档。

纯概念文档的豁免边界：

- 不要求写“核心机制”“主链路”“数学模型”“从零构造路径”。
- 不要求把定义类概念硬解释成因果系统。
- 仍必须写清命名问题、定义边界、相邻概念区别、什么算/不算、例子/反例、常见误读、迁移到哪些更大模型。
- 若纯概念后来被用于算法、系统、工程或制度判断，相关下游文档再按模型型要求展开。

### 4.1 模型型概念文档

主题具有稳定结构、机制、主链路、因果链、失败模式、tradeoff、选型价值、诊断价值或系统行为时，走模型型路径。

必须让读者能回答：

- 它解决什么结构性问题。
- 它的基本变量、状态、约束、不变量或抽象对象是什么。
- 它由哪些部件、层次或分面构成。
- 它如何运转，主链路或因果链是什么。
- 如果从零做一个最小版本，哪些部件必不可少，哪些只是工程包装。
- 哪些条件下适用，哪些条件下失效。
- 关键 tradeoff 和 failure modes 是什么。
- 如何用它解释、预测、调试、取舍或迁移。

### 4.2 纯概念文档

主题主要解决命名、分类、边界、相邻概念区分、例子/反例、误读或上层模型术语稳定性时，走纯概念路径。

必须让读者能回答：

- 这个概念到底在命名什么，为什么需要单独命名。
- 什么算它，什么不算它。
- 它和相邻概念怎么稳定区分。
- 它背后的判别轴、分类规则或定义不变量是什么；若没有机制模型，不要硬造。
- 代表性例子、反例、误读和误用是什么。
- 它后续会进入哪些更大的模型、解释框架或判断流程。

不得把纯概念硬写成伪机制文档，也不得把需要机制判断的对象降成术语说明。

### 4.3 `standard` 与 `deep`

`standard` 适合单主机制、单时间尺度、边界清楚、相邻概念集合较小的主题。

`deep` 适合多层抽象、多后端、多 actor、多执行面、多时间尺度、历史路径重要、failure modes 复杂、常用于架构判断/排障/选型/迁移的主题。命中两条及以上时默认按 `deep`，除非用户明确要求先产出受限 `standard` 候选稿并记录限制。

`standard` 和 `deep` 改变的是证据密度，不改变必需信息位点。

## 5. 输入合同与缺失输入

新建或实质升级概念文档前，至少确认六项输入：

| 输入 | 必须知道什么 |
| --- | --- |
| `concept name` | 要处理的概念、知识点或问题对象 |
| `topic` | 预期归属主题或目录 |
| `user context` | 用户是在什么学习、工作、阅读或问题场景下遇到它 |
| `current confusion` | 用户当前最不理解、最容易混淆或最需要判断的点 |
| `intended downstream use` | 用户后续想拿它分析、判断、排查或迁移到哪里 |
| `time-sensitivity notes` | 是否涉及当前实践、标准、产品、监管、市场、历史路径或废弃做法 |

缺失输入处理：

- 影响路径、topic、来源、时间语境、状态、候选晋升或批量范围时，先澄清。
- 不阻塞候选稿但限制质量判断时，写入 open questions、来源限制或正文假设。
- 只影响表达密度时，可以显式假设后继续。
- 不得静默发明 topic、路径、来源、当前性、下游用途、真实锚点、related docs、状态或 owner decision。

## 6. 输出合同与候选边界

正式概念文档输出必须是 Markdown 知识资产，不是软件产物。

候选稿准备晋升到 `docs/` 时，必须具备：

- YAML frontmatter。
- 唯一稳定的 `doc_id`。
- 与路径一致的 `topic`。
- 明确 `depth_mode`、`source_basis`、`time_context`、`quality_status`、`related_docs` 和 `open_questions`。
- 正文中可定位的角色、边界、验证入口、迁移入口和来源/时间限制。
- `docs/index.md` 入口，或受控的非索引理由。

最低 frontmatter 字段：

```yaml
doc_id: stable-topic-slug
title: Human readable title
concept: stable_concept_identifier
topic: topic-name
depth_mode: standard
created_at: 'YYYY-MM-DDTHH:MM:SS+08:00'
updated_at: 'YYYY-MM-DDTHH:MM:SS+08:00'
source_basis:
  - source_or_related_asset
time_context: explicit_time_context
applicability: where_this_doc_applies
prompt_version: prompt_or_not_applicable
template_version: template_or_asset_version
quality_status: draft
related_docs: []
open_questions:
  - unresolved_question
```

`source_basis`、`related_docs`、`open_questions` 必须是 YAML arrays。

## 7. 必需信息位点

每篇正式概念文档都必须让审查者定位到以下信息：

- problem definition、概念结论，或 naming/classification problem。
- object boundary：什么属于本文对象。
- non-object scope：什么不属于本文对象。
- adjacent concept distinction：最容易混淆的相邻概念和判别差异。
- applicable conditions：判断成立的条件、上下文或使用场景。
- principle/model core：模型型写核心变量、状态、约束、不变量、机制或数学形式；纯概念写判别轴、分类规则、定义不变量或边界测试，不强求机制模型。
- failure/misreading：模型型写失效条件和 failure modes；纯概念写误读、误用和 false equivalence。
- real-world anchor：可定位的真实组织、系统、产品、标准、制度、论文、公开实践或具体场景；确实不适用时说明理由。
- source/time context：当前实践、历史路径、产品、标准、监管或市场相关内容必须有时间语境。
- verification entry：自测题、预测题、诊断题、纠错题或边界判断题。
- transfer entry：可迁移到哪些相邻问题、模型或判断流程。
- open questions：仍待核查、待澄清或后续 story 才会收束的问题。

标题和小节顺序可以调整，信息位点不能静默缺失。

## 8. 推荐正文骨架

### 8.1 模型型概念文档

```markdown
# 标题

## 1. 这份文档要帮你学会什么
## 2. 一句话结论 / 问题定义
## 3. 对象边界与相邻概念
## 4. 核心抽象、变量、状态与不变量
## 5. 核心结构
## 6. 核心机制 / 主链路 / 因果链 / 数学模型
## 7. 最小可重建模型 / 从零构造路径
## 8. 关键 tradeoff 与失败模式
## 9. 用法、接口或工程映射
## 10. 工业 / 现实世界锚点
## 11. 当前推荐实践、过时路径与替代
## 12. 自测题 / 验证入口
## 13. 迁移与关联模型
## 14. 未解问题与继续深挖
## 15. 参考资料
```

### 8.2 纯概念文档

```markdown
# 标题

## 1. 这份文档要帮你学会什么
## 2. 一句话结论 / 概念结论
## 3. 这个概念试图解决什么命名或分类问题
## 4. 概念边界与相邻概念
## 5. 判别轴、分类规则或定义不变量
## 6. 什么算它，什么不算它
## 7. 代表性例子、反例与常见误读
## 8. 它会进入哪些更大的模型或判断框架
## 9. 自测题 / 验证入口
## 10. 迁移与关联模型
## 11. 未解问题与继续深挖
## 12. 参考资料
```

不是每篇都必须逐字照抄目录，但对应路径的信息位点必须可审查。

### 8.3 特殊主题

- 数学推导型：先给建模对象、变量、假设、目标函数或不变量，再给推导步骤、现实问题、工程映射、适用边界、样例/反例和迁移入口。
- API / 库 / 工具 / 框架型：先解释底层抽象、运行模型、数据模型、控制模型或数学模型；接口、参数、代码和命令只能作为模型映射或验证入口，不得主导正文。
- 算法型：先写问题形式化、输入输出、目标函数、核心状态、更新规则、复杂度和失败边界，再写实现接口或示例。
- 工程实践型：先写约束、资源模型、生命周期、风险传播链和设计取舍，再写操作步骤。
- 时间敏感型：写明核对日期、事实/推断区分、来源限制和当前性风险。
- 制度/政策/监管/市场型：给出真实制度或市场对象，说明规则如何落地、旧路径局限和常见失败模式。
- 历史/经典理论型：区分原始语境、后世误读、当前可迁移结构和不可迁移结论。

## 9. 来源纪律与真实世界锚点

涉及当前实践、产品行为、标准、监管、市场状态、历史路径、废弃方案或替代建议时，必须调用 `source-discipline-and-real-world-anchor-policy.md`。

最低要求：

- 写明核对日期或时间语境。
- 区分已核查事实、基于来源的归纳、作者推断、示例化说明和待核查问题。
- 优先使用一手、官方或可定位来源。
- 来源不足时记录限制，不得把推断写成事实。
- 历史路径必须说明为什么旧、局限在哪里、当前替代是什么，以及旧路径何时仍有解释价值。
- 正文、frontmatter `source_basis`、`time_context`、参考资料、open questions 和质量状态语义一致。
- `source_basis` 的仓库路径、外部来源 token、合并/迁移 provenance token 分类由 `docs/governance/docs-asset-governance.md` 负责。

## 10. 质量门禁

审查、验收、升级 gap analysis 和质量状态变更必须调用 `concept-document-quality-gate.md`。

最低审查输出：

- 前置分类：任务类型、资产层级、文档路径、深度模式、来源/时间敏感性。
- Hard Fail 表：问题、证据位置、修复要求。
- 六项评分：问题定义与边界、结构与因果/判别框架、tradeoff/失败模式/误读纠偏、真实锚点/当前实践、验证/迁移、元数据/仓库纪律。
- 必改项。
- 可改进项。
- 最终结论：是否允许进入目标状态、是否允许改 `quality_status`、index/link/version/lifecycle 影响和 unresolved risks。

任意 Hard Fail 存在时，不得宣称 ready、accepted、validated、upgraded、reviewed 或 maintained。

## 11. 仓库集成

正式文档完成前必须检查：

- 路径位于正确 `docs/{topic}/`。
- 文件名使用 `kebab-case`。
- H1、frontmatter `title`、`concept`、`topic`、路径和索引分组不互相误导。
- 新增、重命名、移动、改标题、改 topic、废弃或归档时，同步检查 `docs/index.md`。
- `related_docs` 和正文链接指向存在文件，且关系有意义。
- source/time、quality/status、lifecycle、review decision 和 BMad story status 不混用。

`docs/index.md` 是导航入口，不是质量、身份或生命周期的唯一 source of truth。索引更新不能制造 hidden promotion。

## 12. 固定触发词

### 12.1 新建文档

```text
为概念 {concept} 新建一篇知识库文档。按 docs/methodology/document-generation-methodology.md 执行：先判断任务类型、资产层级、模型型概念文档/纯概念文档路径和 standard/deep；补齐输入合同、正文信息位点、frontmatter、来源/时间语境、质量门禁和 docs/index.md。
默认原理优先、模型优先：模型型、机制型、算法型、系统型和工程型概念先解释本质问题、抽象对象、核心变量/状态/约束/不变量、机制/因果链/数学模型、最小可重建路径和失败边界；再写用法、API、代码或操作步骤。纯概念、纯定义、命名区分或分类边界型文档不强求机制模型，但必须写清定义、判别轴、边界、例子/反例和误读纠偏。除非用户明确要求受限速查，否则不得生成以教程、接口表或代码示例为主的概念文档。
```

### 12.2 升级旧文档

```text
按 docs/methodology/document-generation-methodology.md 升级现有文档 {path}。保留高价值内容，先做 gap analysis，再补路径类型对应的信息位点、来源纪律、验证入口、迁移入口、frontmatter、索引和链接影响；不得为了统一风格整篇推倒重写。
升级时优先检查是否存在“用法/API/例子多，原理/模型/最小可重建路径弱”的结构性缺口；若存在，先补本质模型，再决定是否保留或压缩原用法内容。
```

### 12.3 审查 / 验收

```text
按 docs/methodology/document-generation-methodology.md 和 docs/methodology/concept-document-quality-gate.md 审查文档 {path}。输出前置分类、Hard Fail、六项评分或等价治理检查、必改项、可改进项、最终结论、允许的 quality_status、index/link/version/lifecycle 影响和未解决风险。
```

### 12.4 仅更新索引

```text
检查 docs/index.md 是否正确收录 {path}。如果需要更新，只改导航入口和必要 metadata，保持标题、路径、topic、资产类别和索引分组一致；不得把候选或 workflow output 偷偷晋升为正式资产。
```

## 13. 方法论维护规则

`docs/methodology/` 默认不再新增平行支撑文件。新增规则时先判断归属：

| 规则类型 | 写入位置 |
| --- | --- |
| 任务摄入、生成合同、正文骨架、固定触发词、仓库集成、方法论边界 | 本文 |
| Hard Fail、评分、审查输出、质量状态限制、reviewer calibration | `concept-document-quality-gate.md` |
| 来源、时间语境、当前实践、历史路径、不可验证声明、真实锚点 | `source-discipline-and-real-world-anchor-policy.md` |
| frontmatter schema、`source_basis` 取值分类、路径、索引、链接、资产身份 | `docs/governance/docs-asset-governance.md` |

只有当新规则同时满足以下条件时，才考虑新增 methodology 文件：

- 不能自然并入三份保留规范。
- 不新增平行主入口。
- 有明确 owner entry point、适用范围、非目标、索引处理和下游影响分析。
- 用户明确授权新增，而不是只要求补充当前执行规则。

## 14. 完成证据

完成文档任务时，至少汇报：

- 修改了哪些正式文件。
- 任务类型、资产层级、文档路径和深度判断。
- 质量门禁或等价治理检查结果。
- 来源/时间语境处理。
- `docs/index.md`、frontmatter、链接和 related docs 是否同步。
- 删除、合并、迁移或废弃时的 successor 和引用处理。
- 未解决问题和后续风险。
- 未新增未授权软件产物。
