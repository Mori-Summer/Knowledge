---
doc_id: methodology-concept-document-quality-gate
title: 统一概念文档质量门禁
concept: concept_document_quality_gate
topic: methodology
depth_mode: standard
created_at: '2026-03-19T18:10:00+08:00'
updated_at: '2026-06-23T00:00:00+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/planning-artifacts/implementation-readiness-report-corrected-2026-05-20.md
  - _bmad-output/implementation-artifacts/0-1-agent-behavior-constraints.md
  - _bmad-output/implementation-artifacts/0-2-governance-asset-boundary-policy.md
  - _bmad-output/implementation-artifacts/0-3-lifecycle-states.md
  - _bmad-output/implementation-artifacts/0-4-prompt-template-quality-version-governance.md
  - _bmad-output/implementation-artifacts/0-5-quality-gate-baseline.md
  - methodology_folder_consolidation_2026_06_18
  - docs_folder_consolidation_progress_2026_06_23
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-change-governance.md
  - docs/governance/docs-asset-governance.md
  - docs/templates/governance-record-templates.md
time_context: docs_folder_consolidation_progress_2026_06_23
applicability: concept_doc_acceptance_review_upgrade_and_quality_status_decisions
prompt_version: concept_generation_prompt_v4
template_version: quality_gate_v4
quality_status: maintained_asset
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/governance/docs-change-governance.md
  - docs/governance/docs-asset-governance.md
  - docs/templates/governance-record-templates.md
open_questions:
  - 如果 `docs/governance/docs-asset-governance.md` 定义新的 frontmatter schema 字段，是否需要新增独立 quality_gate_version 字段？
  - 如果 `docs/templates/governance-record-templates.md` 后续调整 Hard Fail evidence 字段，是否需要同步本门禁的审查输出格式？
  - Maxwell 另行明确授权 executable tooling 后，是否需要评估只读 lint/scoring 辅助检查？当前 batch readiness policy 不授权自动化工具。
---

# 统一概念文档质量门禁

## 资产角色、权威与适用范围

这份文档是 `Knowledge` 项目中概念文档验收、审查、升级和质量状态判断的正式质量门禁。它定义 Hard Fail、六项评分、审查输出、质量状态词汇和完成汇报证据，目标是避免知识库出现章节齐全但不可调用、不可验证、不可迁移的文档。

本门禁支持 `docs/methodology/document-generation-methodology.md`，但不替代主规范。概念文档的新建、升级、审查和仓库集成仍以主规范为默认执行入口；本文件负责回答“这篇文档能否通过质量判断，以及失败时应该修哪里”。

本门禁适用于：

- 新建正式概念文档进入 `docs/{topic}/` 前的质量判断
- 旧概念文档升级、复核或质量状态调整
- 审查是否允许声明 `reviewed`、`validated`、`upgraded_v1` 或等价通过结论
- 质量状态、审查结论、完成汇报和修复指导的统一解释

治理、方法论、模板、索引、runbook 或 BMad workflow output 不是普通概念文档。它们需要等价检查角色、权威、适用范围、frontmatter、来源/时间语境、索引/链接、版本记录和 workflow contract fit；不得假装套用概念文档六项分数来证明自身通过。

本质量门禁资产自身属于方法论治理资产。验证它时应检查 `doc_id`、路径、frontmatter、角色/权威、与主规范关系、来源/时间语境、版本治理记录、链接和维护触发点，而不是把它当成一篇普通概念文档打分。

`prompt_version: concept_generation_prompt_v4` 与 `template_version: quality_gate_v4` 表示 2026-06-18 的 methodology folder consolidation 已把旧样例目录的 reviewer calibration 语义并入本门禁，并删除并列支撑入口。

## 审查前置分类

正式审查必须先分类，再进入 Hard Fail 和评分。分类不是建议步骤，而是审查输出的一部分。

概念文档质量门禁的审查顺序固定为：

1. 判定概念文档质量语境下的任务类型：`新建`、`升级`、`审查`、`仅更新索引`、`迁移/废弃` 或 `规范维护`。若请求实际属于 candidate promotion、duplicate/coexistence、lifecycle change、batch readiness 或 non-concept governance/template review，必须同时调用对应治理资产和 document decision vocabulary，不能只靠本门禁分类。
2. 判定资产层级：普通概念文档、方法论资产、治理资产、模板、索引、runbook、workflow output 或报告。
3. 对概念文档判定文档路径：模型型概念文档或纯概念文档。
4. 判定展开密度：`standard` 或 `deep`。
5. 执行 Hard Fail 检查。
6. 仅在无 Hard Fail 时执行六项评分。
7. 输出最终审查结论和允许使用的质量状态。

模型型概念文档适合具有稳定结构、机制、主链路、失败模式、选型价值或调试价值的对象。纯概念文档适合主要解决命名、分类、边界、相邻概念区分、例子/反例和误读纠偏的对象。不得为了套模板把纯概念硬写成伪机制，也不得把需要机制判断的对象降成术语说明。

`standard` 与 `deep` 只决定展开密度和证据要求，不改变文档路径类型。`deep` 要求更高的信息密度、判断支撑、来源纪律和迁移能力，不是更长的文本。

审查输出必须写明分类结果和依据。缺少前置分类本身就是审查流程 Hard Fail，后续分数无效。

## Hard Fail 条件

只要命中任意 Hard Fail，就不得宣称文档 ready、accepted、validated、`upgraded_v1`、`maintained_asset` 或等价通过状态。审查可以继续列出修复建议，但最终结论必须是阻塞或需修订。

每个 Hard Fail 输出都必须包含：

| 字段 | 要求 |
| --- | --- |
| failure condition | 命中的具体失败条件 |
| evidence location | 证据位置，尽量指向 frontmatter 字段、章节标题、链接、索引项或具体段落 |
| why it blocks | 为什么该问题阻塞通过状态或质量声明 |
| repair guidance | 修复指导，说明应补什么、改什么、删除什么或请求谁确认 |

### 仓库集成 Hard Fail

- 目标文件没有位于正确的 `docs/{topic}/`、`docs/methodology/`、`docs/governance/`、`docs/templates/`、`docs/runbooks/` 或其他已批准正式路径。
- 概念文档文件名不是 `kebab-case`，或命名无法稳定表达主题对象。
- 缺失或变更了稳定 `doc_id`，导致身份无法追踪。
- 新增、移动、重命名、改标题、改 topic、废弃或归档正式资产后，没有检查 `docs/index.md` 影响。
- 需要更新索引时未更新，或更新后的索引标题、路径、topic 与正文/frontmatter 不一致。
- changed-file links、正文内链、`related_docs` 或索引链接指向不存在的文件，且未记录为 open question 或 planned asset。
- 把 `_bmad-output/` workflow output 当成正式 `docs/` 资产直接引用或发布，没有经过晋升、frontmatter、索引和质量证据检查。

### 必需 Frontmatter Hard Fail

- 正式概念文档缺少主规范要求的必需字段：`doc_id`、`title`、`concept`、`topic`、`depth_mode`、`created_at`、`updated_at`、`source_basis`、`time_context`、`applicability`、`prompt_version`、`template_version`、`quality_status`、`related_docs`、`open_questions`。
- 需要 YAML array 的字段使用了字符串、逗号拼接文本或其他不可稳定解析的格式，例如 `source_basis`、`related_docs`、`open_questions`。
- frontmatter 的 `doc_id`、`title`、`concept`、`topic`、`depth_mode`、`source_basis`、`time_context`、`prompt_version`、`template_version` 或 `quality_status` 与正文、来源、索引或审查结论矛盾。
- 正式概念文档缺少 `depth_mode`，或正文声称按 `deep`/`standard` 审查但 frontmatter 没有相应语境。
- `quality_status`、版本字段或正文结论暗示已审查、已迁移、已验证或已升级，但没有对应证据。
- 新增全局 frontmatter 字段、迁移状态字段或质量版本字段来规避资产治理决策，例如未经授权添加 `quality_gate_version`、`methodology_version`、`version_history` 或 `migration_status`。

### 文档类型结构 Hard Fail

模型型概念文档缺少以下任一信息位点时阻塞通过：

- 清晰的问题定义或一句话结论
- 对象边界和非对象范围
- 核心结构、部件、层次或分面框架
- 核心机制、主链路、因果链或可调用判断链
- 关键 tradeoff、失败模式、失效条件或误用边界
- 自测题、诊断题、预测题或其他验证入口
- 迁移入口、关联模型或下游判断使用方式
- 真实世界锚点、工业对象、当前实践或历史路径纪律，前提是主题本身需要这些语境才能判断

纯概念文档缺少以下任一信息位点时阻塞通过：

- 命名、分类或区分问题
- 这个概念在命名什么，以及为什么需要单独命名
- 什么算它，什么不算它
- 相邻概念区分
- 代表性例子、反例、误用或常见误读
- 自测题、辨析题或其他验证入口
- 迁移入口、下游模型位置或后续判断使用方式

所有概念文档还必须有未解问题、继续深挖或模型边界说明。若某类信息确实不适用，必须在正文中说明不适用理由；不能静默缺失。

### 边界清晰度 Hard Fail

- 对象边界不清，读者无法判断什么属于本文对象、什么不属于。
- 没有相邻概念区分，或把相邻概念列成名词清单但不说明判别差异。
- 把相关概念做成错误等价，例如把实现机制、使用场景、历史做法和当前推荐实践混为一谈。
- 文档主张无法用于区分、预测、诊断、选型、迁移或纠错，只能复述定义。
- 关键限定条件、适用条件、失效条件或误用边界缺失，导致结论容易被过度泛化。

### 验证与迁移 Hard Fail

- 没有自测题、验证入口、诊断入口、预测入口、辨析入口或可执行的理解检查。
- 验证入口只要求回忆术语，无法检验解释、边界判断、机制理解、误读纠偏或迁移能力。
- 没有迁移入口、关联模型、相邻问题使用方式或下游判断位置。
- 迁移小节只是列出相邻名词，没有说明可迁移的结构、边界、判断链或失败模式。

### 来源依据与真实世界锚点 Hard Fail

本节的 Hard Fail 语义仍由本文承担；`docs/methodology/source-discipline-and-real-world-anchor-policy.md` 只补充锚点 adequacy、声明类型、current-practice、historical/deprecated practice 和不可验证声明处理细则，不创建第二套质量门禁。

- 工业、现实世界、组织、系统、产品、标准、制度或市场锚点泛泛而谈，无法定位。
- 真实世界锚点不能支撑正文关键结论，或只作为装饰性例子出现。
- 涉及当前实践、现行制度、当前产品行为、现行标准或最新状态，却没有来源依据、核对日期或来源限制说明。
- 把历史路径、旧方案、已淘汰实践或旧版本行为写成当前推荐实践。
- 时间敏感结论缺少相称的一手、官方或可核查来源；如果只能做推断，必须标明是推断。

### 时间语境与一致性 Hard Fail

- `updated_at`、`time_context`、正文日期、来源日期和审查结论互相矛盾。
- `source_basis` 声称依赖某来源，但正文关键结论无法从这些来源或明确推断中得到支撑。
- 正文把事实、推断、建议和价值判断混写到无法区分。
- 正文、frontmatter、索引、`related_docs`、参考资料或审查输出对文档状态、版本、路径、title、topic 或适用范围的说法不一致。
- 存在阻塞性的未解决风险、断链、来源缺口或迁移缺口，却在结论中宣称 ready、accepted、validated、`upgraded_v1`、`maintained_asset` 或等价通过状态。版本记录中的未来治理风险、后续模板工作或待授权自动化工作，若不影响当前资产角色、权威、来源、链接和维护触发点，可以作为 nonblocking future risk 记录。

## 评分规则

正式评分只能在 Hard Fail 检查全部通过后执行。分数不能抵消 Hard Fail；任意 Hard Fail 存在时，不输出正式通过分数。如需辅助返工，可以单独给出“repair guidance score”，但必须明确标注为修复参考，不得作为通过依据。

六项维度各按 `0-2` 分计：

- `0`：基本缺失、明显不合格，或只能给出标签式内容。
- `1`：有对应内容，但支撑弱、边界不稳、证据不足或迁移价值有限。
- `2`：清晰、扎实、可复用，能支持解释、区分、判断、验证或迁移。

### 问题定义与边界

| 分数 | 操作含义 |
| --- | --- |
| `0` | 没有明确问题定义、命名问题或一句话结论；对象边界不清；相邻概念无法区分。 |
| `1` | 说明了对象大意，但边界、非对象范围、适用条件或相邻概念区分仍需要读者自行补全。 |
| `2` | 问题定义、对象边界、非对象范围、适用条件和相邻概念差异清楚，能直接用于判别。 |

### 结构与因果 / 判别框架

| 分数 | 操作含义 |
| --- | --- |
| `0` | 模型型文档缺少结构/机制链，或纯概念文档缺少分类/判别框架；内容主要是定义复述。 |
| `1` | 有结构件、机制链或判别框架，但层次浅、链路断、例子不足，难以支撑复杂判断。 |
| `2` | 模型型文档给出可调用结构和主链路；纯概念文档给出稳定分类边界和判别框架。 |

### Tradeoff / 失败模式 / 误读纠偏

| 分数 | 操作含义 |
| --- | --- |
| `0` | 没有 tradeoff、失败模式、失效条件、误读纠偏或反例。 |
| `1` | 提到取舍、失败或误读，但比较抽象，缺少触发条件、后果或纠偏判断。 |
| `2` | 模型型文档能说明核心取舍、失效条件和误用边界；纯概念文档能纠正常见误读、错误类比和过度泛化。 |

### 真实世界锚点 / 当前实践

| 分数 | 操作含义 |
| --- | --- |
| `0` | 没有真实锚点，或锚点不可定位；当前实践、历史路径和来源纪律混乱。 |
| `1` | 有真实对象或实践说明，但定位、来源、日期、历史/当前区分或解释作用不足。 |
| `2` | 锚点真实可定位，能支撑正文判断；当前实践、旧路径、替代方案、核对日期和来源限制清楚。 |

### 验证 / 迁移

| 分数 | 操作含义 |
| --- | --- |
| `0` | 没有验证入口或迁移入口，或只提供术语回忆题。 |
| `1` | 有自测、验证或迁移内容，但主要检查记忆，不能充分支撑解释、边界判断或相邻问题迁移。 |
| `2` | 验证入口能检查解释、判别、诊断、预测或选型；迁移入口说明可复用结构和下游模型位置。 |

### 元数据 / 仓库纪律

| 分数 | 操作含义 |
| --- | --- |
| `0` | frontmatter、路径、命名、索引、链接、来源或状态存在明显缺口或冲突。 |
| `1` | 基本可用，但部分字段、链接、来源、时间语境、索引影响或状态解释需要补强。 |
| `2` | `doc_id`、title、topic、path、frontmatter、links、index、source/time context、version/status 语义一致且可追溯。 |

### `standard` 与 `deep` 的评分差异

`standard` 要求文档达到稳定理解和最小复用。它可以较短，但仍必须通过 Hard Fail，并在六项评分中证明问题定义、结构/判别、验证/迁移和元数据足够可靠。

`deep` 要求更强的判断密度和证据支撑。命中以下条件两条及以上时，默认按 `deep` 审视：

- 多层抽象并存
- 多 actor、多后端、多执行面或多生命周期阶段并存
- 当前推荐实践与历史路径都重要
- 相邻概念多，误读路径多，边界网络复杂
- 文档会用于架构判断、排障、选型、迁移或治理决策
- 失败模式和 tradeoff 本身构成理解核心

`deep` 文档如果只是在每节增加篇幅，但没有增强判断链、来源证据、失败模式、锚点和迁移能力，不应获得高分。

### 分数阈值

阈值只在无 Hard Fail 时有效：

| 总分 | 允许结论 |
| --- | --- |
| `10-12` | 可声明通过当前质量门禁，并可考虑 `upgraded_v1` 或相应通过结论。 |
| `7-9` | 可保留或进入受控修订，但不建议声明 `upgraded_v1`、`validated` 或强通过状态。 |
| `0-6` | 应退回补强、重做或进入 targeted revision。 |

任意 Hard Fail 存在时，不管总分多高，最终结论都必须是阻塞或需修订。

## 质量状态与审查结论

`quality_status`、生命周期状态、review decision label 和 BMad story 执行状态是四类不同信号。

`quality_status` 是文档 frontmatter 中表达质量结果、维护状态或既有质量门禁兼容信号的字段。生命周期状态表达正式资产在仓库中的成熟度、可依赖程度和使用方式。BMad story 状态只表达 workflow 执行进度，例如 `ready-for-dev`、`in-progress`、`review`、`done`，不得复制进正式文档 frontmatter。

当前兼容的 `quality_status` 值解释如下：

| 值 | 质量门禁语境中的含义 |
| --- | --- |
| `draft` | 已存在或已进入候选位置，但尚未完成足以支持稳定复用的质量审查。 |
| `reviewed` | 已经过结构、frontmatter、链接/索引和质量证据审查；已知缺口已记录。 |
| `validated` | 当前没有 Hard Fail，来源/时间语境、链接/索引、未解问题和目标复用场景已检查。 |
| `maintained_asset` | 适用于 active methodology/governance/template/index/support asset；只有在角色、权威、版本、来源/时间语境、链接、维护触发点仍当前时才成立。 |
| `upgraded_v1` | 既有概念文档质量信号，表示按当时质量门禁完成过较强升级；它不是完整生命周期状态，也不自动等于当前 `validated`。 |
| `deprecated` | 资产保留访问但不再作为主要当前入口；需要废弃原因和替代入口。 |
| `archived` | 资产主要为历史、证据或审计保留；正常复用预期被限制。 |

审查结论应与 frontmatter status 分开表达，并使用当前 document decision vocabulary。质量门禁常见结论包括：

- `targeted_revision`：存在可定位、可修复的 Hard Fail 或质量缺口，不得宣称通过。
- `regenerate`：局部修复不足以恢复结构、来源、身份或模型完整性，需要重生成并处理旧内容。
- `accepted_for_current_use`：满足当前使用范围，可作为正式参考或保留资产。
- `accepted_with_nonblocking_follow_up`：无阻塞 Hard Fail，但仍有明确 nonblocking follow-up、owner 和 re-review trigger。
- `accept_promote`：候选或目标已具备完整质量、来源、索引/链接和必要 specialized-record evidence，可支持晋升或强通过表达。
- `hold_for_clarification`：涉及路径、身份、状态、schema、批量迁移、自动化或资产边界冲突，需要 Maxwell/owner 确认。
- `defer_with_reason`：真实问题存在，但当前 story、授权、时间或 future owner 不允许本轮处理，且必须记录 reason、impact、owner 和 re-review trigger。

任何 Hard Fail 存在时，审查输出不得使用 ready、accepted、validated、`upgraded_v1`、`maintained_asset` 或等价通过措辞。必须列出具体失败项、证据位置、阻塞理由和修复指导。

本门禁不授权批量归一化既有 `quality_status`。既有文档不得被静默改写成 `validated`、`upgraded_v1`、`maintained_asset` 或任何新值。状态调整必须来自明确目标文档的审查、升级、迁移或 Maxwell 授权的批量 readiness 流程。

## 审查输出格式

正式审查输出必须可复查，不能只写“整体不错”“建议更完整”。

推荐格式如下：

### 1. 前置分类

| 项 | 结论 | 依据 |
| --- | --- | --- |
| 任务类型 |  |  |
| 资产层级 |  |  |
| 文档路径类型 | 模型型概念文档 / 纯概念文档 / 不适用 |  |
| 展开密度 | `standard` / `deep` / 不适用 |  |

### 2. Hard Fail 表

| 失败条件 | 证据位置 | 为什么阻塞 | 修复指导 |
| --- | --- | --- | --- |
|  |  |  |  |

若无 Hard Fail，也必须写明“未发现 Hard Fail”，并说明检查覆盖了仓库集成、frontmatter、文档类型结构、边界、验证、迁移、来源、时间语境和一致性。

### 3. 六项评分

| 维度 | 分数 | 依据 |
| --- | --- | --- |
| 问题定义与边界 | `0-2` |  |
| 结构与因果 / 判别框架 | `0-2` |  |
| Tradeoff / 失败模式 / 误读纠偏 | `0-2` |  |
| 真实世界锚点 / 当前实践 | `0-2` |  |
| 验证 / 迁移 | `0-2` |  |
| 元数据 / 仓库纪律 | `0-2` |  |

### 4. 必改项

列出不改就不能通过的项目。每一项都要能对应 Hard Fail 或低分维度。

### 5. 可改进项

列出不阻塞当前结论，但能提高复用性、迁移性、来源可靠性或仓库治理质量的项目。

### 6. 最终结论与允许状态

必须说明：

- 审查决策
- 是否允许通过状态
- 是否允许或不允许改 `quality_status`
- 是否需要 index/link/version/lifecycle 后续动作
- 是否存在 approved deviations 或 unresolved risks

## 版本、迁移与完成汇报

Hard Fail、评分维度、分数解释、质量状态、审查结论、完成汇报义务或迁移规则发生语义变化时，必须按 `docs/governance/docs-asset-governance.md` 记录版本治理事件。

历史版本治理事件曾将本门禁从 `quality_gate_v2` 提升为 `quality_gate_v3`。最小版本变更记录如下：

```yaml
version_change_record:
  changed_asset: docs/methodology/concept-document-quality-gate.md
  old_value: quality_gate_v2
  new_value: quality_gate_v3
  change_type: quality_gate
  reason: 固化 Hard Fail、评分维度、质量状态与完成汇报语义
  affected_docs_or_assets:
    - docs/methodology/concept-document-quality-gate.md
    - docs/methodology/document-generation-methodology.md
    - existing concept documents using quality_status: upgraded_v1
  expected_generation_impact: no_direct_generation_change; future generation self-check must classify before Hard Fail and scoring
  expected_review_impact: 后续审查必须先分类、再 Hard Fail、再评分；Hard Fail 阻塞通过状态
  migration_plan: targeted_review
  index_navigation_impact: none_expected_unless_title_path_topic_changes
  lifecycle_quality_status_impact: status_policy_review_without_bulk_rewrite
  approved_deviations: []
  unresolved_risks:
    - "historical risk at the time: 独立 quality_gate_version/frontmatter schema 尚未定义"
    - "historical context only: review/completion assets 当时尚未建立；当前维护应使用 docs/templates/governance-record-templates.md"
    - "nonblocking future risk: batch readiness policy 已落地，但 executable lint/scoring/CI/tooling 仍需 Maxwell 另行授权和新的范围/架构判断"
```

完成概念文档生成、升级、审查或质量门禁维护时，汇报至少覆盖：

- 使用的 fixed prompt、template、methodology 和 quality gate 来源及版本；不适用时写 `not_applicable`。
- 任务类型、资产层级、允许文件范围和实际变更文件。
- Hard Fail 检查结果、六项评分结果或等价治理检查结果。
- `quality_status`、生命周期状态、BMad story 状态的边界和影响。
- `docs/index.md`、`related_docs`、正文链接、路径、标题、topic 和 `doc_id` 影响。
- 涉及 targeted revision、structural upgrade、regeneration、old-content handling 或 reference-validity impact 时，记录 update mode、continuity evidence、reference validity，并在适用时引用 migration/index records。
- 版本治理事件、迁移计划、approved deviations 和 unresolved risks。
- 非软件边界确认：未新增未授权 package manifest、source tree、software tests、build config、runtime automation、CLI/API/UI/database/deployment/CI artifact。

本门禁不授权 executable lint、评分脚本、CI 检查或其他自动化工具。`docs/governance/docs-change-governance.md` 只提供手动文档治理 readiness 判定；任何自动化仍只能在 Maxwell 明确授权 executable tooling 后，经过新的范围、架构和文件影响判断。

## 维护触发点

以下变化要求复核本门禁：

- 主方法论、来源纪律或 reviewer calibration 语义发生变化时。
- `docs/governance/docs-asset-governance.md` 的 frontmatter、`doc_id`、topic、路径、命名、index 或 lifecycle 规则发生实质字段或 vocabulary 变更时。
- `docs/governance/docs-change-governance.md` 的 candidate promotion、document decision、rework loop、revision/regeneration 或 batch 规则发生实质字段或 vocabulary 变更时。
- `docs/templates/governance-record-templates.md` 的 review/completion evidence 字段发生实质变化时。
- `docs/governance/docs-asset-governance.md` 或 `docs/governance/docs-change-governance.md` 更新时。
- `docs/runbooks/batch-governance-runbook.md` 的 batch evidence 要求发生实质变化时。
- `docs/governance/docs-change-governance.md` 已定义批量治理 readiness；未来如 Maxwell 明确授权批量状态迁移或自动化辅助检查，仍需重新复核本门禁。

维护时必须重新确认：本门禁仍是 `docs/methodology/concept-document-quality-gate.md` 的 canonical asset；`docs/methodology/document-generation-methodology.md` 仍是主执行规范；没有创建平行质量门禁；没有批量改写既有概念文档状态。

## 不能被门禁放过的伪优秀文档

下面这些文档看起来完整，但不得因语言流畅或章节齐全而放行：

- 结构齐全，但核心机制空心。
- 结构齐全，但纯概念被硬拆成伪机制文档。
- 结构齐全，但明显复杂的主题却每节只有一小段摘要。
- 案例很多，但都是泛泛行业印象，无法定位真实对象。
- 语言流畅，但没有判断框架、边界或迁移入口。
- 有“当前实践”，但没有日期、来源或历史/当前区分。
- 有“迁移”小节，但只是列出相邻名词，没有说明可迁移的结构。
- frontmatter 看起来完整，但状态、版本、来源或时间语境暗示了不存在的审查。

这类文档应退回修复，而不是通过门禁。

## Reviewer action calibration

Reviewer 可以用下面的紧凑表判断候选稿该补写、重生成还是接受。正式结论仍必须使用本门禁的 Hard Fail、六项评分、必改项、可改进项和最终结论格式。

| action signal | 适用信号 | 不适用信号 |
| --- | --- | --- |
| `targeted_revision` | 高价值结构已经存在；缺失证据位置具体；identity、path、source status 没有根本错误；补写不会改变核心判断链 | 对象边界、来源、状态和主链路都不可信 |
| `regeneration_candidate` | path/type 错误；结构是模板填空；多个核心信息点命中 Hard Fail；source/time claims 广泛 unsupported；内容太泛，保留会延续错误判断 | 只有少数可定位缺口，且修复不会改变主结构 |
| `acceptance_candidate` | 无 Hard Fail；frontmatter/path/index discipline sound；必需信息点可定位；未决风险 nonblocking；状态声明保守 | 存在阻塞来源缺口、断链、状态过度声明或不可定位锚点 |

来源和时间语境证据至少按以下标签理解：

| label | 含义 |
| --- | --- |
| `verified fact` | 已从本地正式资产或合适来源核对的事实，并有检查日期或时间语境 |
| `project rule` | 来自本项目正式方法论、治理资产、story 或 Maxwell 指令的规则 |
| `user context` | 来自 Maxwell 当前请求、story、intake 或对话语境的使用背景 |
| `inference` | Agent 对项目规则的操作化整理、归纳或推导，不是新增规则 |
| `open question` | 当前任务不解决、尚待后续核查或归属后续治理工作的事项 |

## 参考资料

- [统一概念文档规范：AI 生成、升级、审查与仓库集成](./document-generation-methodology.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](./source-discipline-and-real-world-anchor-policy.md)
- [正式 docs 资产治理规范：身份、元数据、路径、生命周期、索引、链接与网络边界](../governance/docs-asset-governance.md)
- [文档治理执行规范：复用扫描、晋升、合并、迁移、修订、批量、决策与返工闭环](../governance/docs-change-governance.md)
- [文档治理记录模板：审查、批量审查、完成汇报与批量完成汇报](../templates/governance-record-templates.md)
