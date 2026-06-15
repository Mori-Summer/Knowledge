---
doc_id: methodology-source-discipline-and-real-world-anchor-policy
title: 来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理
concept: source_discipline_and_real_world_anchor_policy
topic: methodology
depth_mode: standard
created_at: '2026-05-25T16:43:09+08:00'
updated_at: '2026-05-27T14:52:11+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/1-1-concept-document-generation-contract.md
  - _bmad-output/implementation-artifacts/1-2-intake-and-intent-classification.md
  - _bmad-output/implementation-artifacts/1-3-concept-document-example-catalog.md
  - _bmad-output/implementation-artifacts/1-4-source-discipline-and-real-world-anchor-policy.md
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
time_context: phase_4_epic_1_source_discipline_2026_05_25
applicability: concept_document_source_discipline_and_real_world_anchor_policy
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/concept-document-example-catalog.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
open_questions:
  - Epic 2 frontmatter schema 落地后，是否需要把来源声明类型或核对日期进一步 schema 化？
  - 如果 Epic 3 review/completion templates 后续调整 unverifiable claim evidence 字段，是否需要同步本文的不可验证声明处理记录方式？
  - Epic 5 建立 related docs taxonomy 后，是否需要区分 source reference、supporting link、related doc 和 successor link？
---

# 来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目的正式方法论支撑资产，用来定义概念文档生成和审查中的来源纪律、时间语境、真实世界锚点、历史/废弃实践和不可验证声明处理规则。

本文补充 `docs/methodology/document-generation-methodology.md`、`docs/methodology/concept-document-contract.md`、`docs/methodology/intake-and-intent-classification.md`、`docs/methodology/concept-document-example-catalog.md`、`docs/methodology/concept-document-template.md`、`docs/methodology/concept-document-quality-gate.md`、`docs/methodology/fixed-concept-generation-prompt.md` 和 Foundation governance assets。它不替代主方法论、合同、intake 流程、模板、质量门禁、固定 Prompt、生命周期政策、版本治理政策、导航政策、最终审查决策政策、review record template、Epic 2 schema/index rules 或 Epic 6 batch rules。

本文的 owner entry point 是 `docs/index.md` 的 `methodology` 分组。Navigation treatment 为 `listed_in_docs_index`；index treatment 是在 `docs/index.md` 的 `## methodology` 下列出 `docs/methodology/source-discipline-and-real-world-anchor-policy.md`。

本文适用于以下场景：

- 新建或升级概念文档时判断真实世界锚点是否足够具体。
- 审查 current-practice、historical/deprecated practice、产品/标准/监管/市场状态等时间敏感声明。
- 区分事实、来源归纳、项目规则、用户语境、推断、建议、价值判断和 open question。
- 处理无法验证但会影响 ready、accepted、reviewed、validated、`upgraded_v1`、`maintained_asset` 或等价通过状态的声明。

本文不要求每篇概念文档都做广泛外部研究。证据负担必须和声明类型、当前性风险、质量状态声明、下游使用风险成比例。低当前性、纯项目内规则或明确 illustrative 的例子，可以使用项目内来源；外部 current-practice claim 则需要可追溯来源和核对日期。

## 真实世界锚点规则

真实世界锚点的职责不是装饰正文，而是把抽象概念压到可定位、可检查、可解释的对象或场景上。一个锚点必须支持正文中的某个解释、边界、tradeoff、失败模式、迁移判断、历史迁移点或来源/时间语境判断。

可接受的锚点类型包括：

| 类型 | 最低要求 |
| --- | --- |
| 组织 | 具体组织名称、相关实践或公开对象、锚定的声明 |
| 系统 | 具体系统名称或可定位系统类别、运行语境、支撑的机制或边界 |
| 产品 | 具体产品或版本语境、相关行为、核对日期或来源限制 |
| 标准 | 标准名称、版本或状态语境、支撑的规则或术语 |
| 公开实践 | 可定位实践名称、适用范围、来源限制和时间语境 |
| 论文 | 论文标题或可定位引用、提出或验证的概念点 |
| 具体运行场景 | actor、环境、约束、触发条件和要解释的判断 |
| 项目内治理对象 | 本仓库 formal docs、quality gate、template、story 或 governance asset，用来证明项目规则或治理边界 |

每个锚点至少写清五件事：

1. `name`：锚点叫什么，能否被 reviewer 定位。
2. `type`：它是组织、系统、产品、标准、公开实践、论文、运行场景还是项目内治理对象。
3. `context`：它出现在哪个语境、版本、时间、项目或运行条件下。
4. `why it anchors`：为什么这个对象能帮助理解本文概念，而不是随手举例。
5. `supported claim`：它支撑正文中哪条定义、边界、机制、tradeoff、失败模式、迁移或来源判断。

以下写法不构成合格锚点：

- “行业实践”“很多系统”“现代产品”“常见架构”“大型项目”。
- 未命名团队、泛泛生产系统、抽象公司、没有定位信息的市场实践。
- 只说明“现实中会用到”，但不说明支撑哪条判断。
- 只为了让章节看起来完整的 decorative example。
- 例子真实存在，但与正文 claim 没有解释关系。

项目内锚点可以用于证明 `project rule`、workflow contract、质量门禁、导航政策、生命周期状态或版本治理规则。例如，`docs/methodology/concept-document-quality-gate.md` 可以作为“不可定位锚点会触发 Hard Fail”的项目内锚点。项目内锚点不得伪装成外部 verified fact；它只能支撑本项目规则或本仓库状态。

当正文声称外部 current practice、产品行为、标准状态、市场状态、监管状态或公共行业实践时，通常需要外部锚点。项目内锚点不能替代外部来源来证明“行业现在通常如何做”。

若主题确实不适合真实世界锚点，必须显式写出 not-applicable rationale，并提供补偿性验证方式。例如纯数学概念可说明锚点不适用的原因，但仍需要通过形式定义、推导、反例、应用约束或 open question 支撑理解。不能因为“难找锚点”就补一个泛泛案例。

## 来源与声明类型

文档必须把不同声明类型分开写，不能把事实、推断、建议和值判断混成一个不可审查段落。

| 标签 | 含义 | 可支持什么 | 不能支持什么 |
| --- | --- | --- | --- |
| `verified fact` | 已从可定位来源核对的事实，并有时间语境或核对日期 | 存在性、字段、版本、官方定义、标准条文、公开事实 | 超出来源范围的通用建议 |
| `source-based synthesis` | 基于多个来源或同一来源不同段落的归纳整理 | 有限定范围的趋势、模式、对比或解释 | 绝对化 best practice 或无边界结论 |
| `project rule` | 来自本项目 formal docs、governance assets、story 或 Maxwell 明确指令的规则 | 本项目内任务、状态、索引、质量门禁和 Agent 行为判断 | 外部行业事实或通用标准声明 |
| `user context` | 来自 Maxwell 当前请求、intake、story、对话或明确偏好 | 解释角度、下游用途、范围限制、优先级 | 客观事实或外部当前实践 |
| `inference` | Agent 从事实、来源或项目规则推导出的判断 | 有条件的解释、风险提示、操作化框架 | 未标限制的事实或 validated/current 状态 |
| `open question` | 当前无法验证、超出范围或等待后续 story 的事项 | 暴露边界、后续核查和防止过度声明 | 支撑 ready、accepted、reviewed、validated 或等价通过状态 |

相关概念的区别如下：

- `fact` 是可由来源直接确认的陈述，例如某文件存在、某标准写了某术语、某产品文档在某日期说明某行为。
- `source-based synthesis` 是对来源的有限归纳，例如多个官方文档共同显示某类约束，但归纳范围必须小于来源范围。
- `inference` 是从事实或规则推出的解释，必须保留推断链和限制。
- `recommendation` 是面向行动的建议，必须说明目标、前提、风险和来源/推断基础。
- `value judgment` 是偏好或价值取向，例如“更清晰”“更适合学习”，必须说明评价标准，不得伪装成事实。

外部声明的来源层级如下：

1. 一手、官方或权威来源优先：标准、规范、产品文档、论文、监管文本、组织公开文档。
2. 技术声明优先使用标准、规格、论文、官方实现文档或产品文档。
3. currentness claim 必须使用有日期、可追溯、能支持当前状态的来源。
4. 二手来源只能作为辅助，必须说明其限制；不能用二手摘要替代可获得的一手来源来支撑高风险 current-practice claim。

项目规则的来源层级不同。正式 `docs/` 资产、Foundation governance assets、当前 story、BMad workflow 和 Maxwell 明确指令可以支撑 `project rule`，但不得被写成外部 verified fact。例如，“本项目要求新增 formal docs 同步 `docs/index.md`”是项目规则，不是行业通用事实。

当声明依赖当前标准、产品行为、市场状态、监管状态或活跃公共实践时，正文必须写出 checked date 或等价时间语境，并说明来源限制。`time_context`、frontmatter `source_basis`、正文、参考资料和 `open_questions` 必须一致。

## 当前实践声明

以下表达默认构成 current-practice claim：

- “当前”“现在”“现代”“现行”“推荐”“标准做法”“best practice”。
- “通常”“业界使用”“现在一般”“主流系统会”。
- 活跃标准行为、产品行为、库/框架行为、监管状态、市场状态。
- 对历史路径的“已被替代”“不再推荐”“当前更合适”的判断。

current-practice claim 必须满足四项要求：

1. 显式时间语境：写明核对日期、版本语境、阶段语境或来源日期。
2. 声明范围足够窄：不能把一个产品、标准、社区或项目中的证据扩写成全行业结论。
3. 声明类型分离：verified fact、source-based synthesis、inference、recommendation 和 open question 分开。
4. 来源限制可见：说明来源覆盖什么、没覆盖什么、何时可能失效。

过宽声明必须收窄。例如：

| 不合格写法 | 可接受改写方向 |
| --- | --- |
| “现代系统都推荐 X。” | “在本文核对的某标准/产品/项目语境中，X 被作为推荐或示例；是否可泛化到其他系统仍是 open question。” |
| “业界通常使用 Y。” | “来源 A 和 B 显示 Y 在这些公开场景中被使用；本文只把它作为可定位实践，不宣称普遍性。” |
| “现在不再使用 Z。” | “在本文核对的来源范围内，Z 被标为 legacy/deprecated；旧系统兼容、教学对比或特定环境仍可能保留解释价值。” |

不能把 source-limited synthesis 写成 universal recommendation。若来源只覆盖某标准、某产品、某论文或某项目，正文建议也只能覆盖相同或更窄范围。

## 历史实践、旧方案与已废弃路径

历史、legacy、deprecated、superseded 或 context-limited 例子可以保留，但必须被正确框住。旧例子的价值通常是解释迁移、对比边界、展示失败压力或支持兼容/维护语境，而不是冒充当前推荐。

凡正文使用历史或废弃实践，必须说明四件事：

1. 为什么它属于历史、deprecated、legacy、superseded 或 context-limited 语境。
2. 哪个 limitation、failure pressure、成本、兼容性、规模、可靠性、安全、治理或认知问题导致迁移。
3. 当前更合适的 alternative、preferred framing 或 review posture 是什么，并写出时间语境。
4. 旧例子在哪些条件下仍有解释价值，例如 contrastive value、migration warning、compatibility context、legacy maintenance context 或教学上的 explanatory value。

禁止把旧例子写成当前推荐，除非有当前来源明确支持该结论。即使旧例子仍可在特定条件下使用，也必须写明条件，而不是删除历史标签。

## 不可验证声明处理

不可验证声明是指在当前任务范围、来源、时间或证据下无法被 reviewer 确认的声明。它不一定要被删除，但不能支撑通过状态或关键结论。

允许的处理方式只有五类：

| 处理方式 | 适用条件 |
| --- | --- |
| remove | 该声明不必要，删除不会损害文档核心判断 |
| replace with narrower verifiable claim | 有来源能支持更窄、更具体、更低风险的说法 |
| mark as inference with limitation | 推断链可解释，但不能伪装成事实 |
| move to open questions | 当前 story、来源或时间不足以解决，但需要保留为后续核查点 |
| stop for clarification | 该声明影响 canonical path、quality/status、currentness、外部事实或用户决策，继续写会制造假通过 |

以下情况可触发质量门禁 Hard Fail：

- current-practice claim 无来源、无核对日期或无来源限制。
- 真实世界锚点泛泛不可定位，或不能支撑正文 claim。
- historical/deprecated practice 被写成当前推荐。
- 事实、推断、建议和值判断混写到 reviewer 无法区分。
- blocking open question 或不可验证声明仍被用来支撑 ready、accepted、reviewed、validated、`upgraded_v1`、`maintained_asset` 或等价通过状态。

本文不重新定义质量门禁。上述处理必须回到 `docs/methodology/concept-document-quality-gate.md` 的 Hard Fail 类别：来源依据与真实世界锚点、时间语境与一致性、必需 frontmatter、仓库集成、边界清晰度和状态过度声明。

## 生成与审查检查表

生成概念文档候选稿前，至少检查：

1. 是否存在 current-practice、historical/deprecated、产品/标准/监管/市场状态或外部实践声明。
2. 每个真实世界锚点是否有 name、type、context、why it anchors 和 supported claim。
3. 外部 current-practice claim 是否有可追溯来源、核对日期和范围限制。
4. 项目内规则是否标为 `project rule`，而不是外部事实。
5. 推断、建议和值判断是否标出依据和限制。
6. 不可验证声明是否已删除、收窄、标为 inference、移入 open questions 或停止澄清。
7. `source_basis`、`time_context`、正文、参考资料、`quality_status` 和 `open_questions` 是否一致。

审查时，reviewer 至少检查：

| 检查项 | 通过证据 |
| --- | --- |
| anchor adequacy | 锚点可定位，并能支撑具体 claim |
| claim labels | verified fact、source-based synthesis、project rule、user context、inference、open question 可区分 |
| current-practice discipline | 有时间语境、checked date 或等价限制；没有过度泛化 |
| historical/deprecated handling | 说明历史原因、迁移压力、当前替代/framing 和剩余解释价值 |
| unverifiable claims | 不再支撑通过状态或关键结论 |
| quality-gate fit | 若触发 Hard Fail，输出 failure condition、evidence location、why it blocks、repair guidance |

验证本文自身时，检查 frontmatter、heading hierarchy、索引条目、相关链接、source/time/status 一致性、非软件边界、版本治理影响和未来 story 边界。本文当前 `quality_status: draft` 是保守状态，因为 Epic 2 和 Epic 3 已细化 schema、review record、decision policy、rework/completion evidence 形态，Epic 5 尚未细化 related docs taxonomy。

## 维护触发点

以下变化要求复核本文：

- Epic 2 建立 frontmatter schema、doc_id、topic/path/naming、candidate promotion 或 index synchronization rules。
- Epic 3 review record、document decision policy、rework loop 或 completion report template 发生实质字段或 vocabulary 变更。
- Epic 5 建立 related docs taxonomy、link maintenance 或 reusable model entry points。
- Epic 6 建立 batch governance runbook、batch review record 或 batch completion report。
- Maxwell 明确授权 executable tooling、lint/scoring automation、CI 或批量检查工具；当前本文不授权这些自动化。
- 质量门禁 Hard Fail、prompt/template version governance、lifecycle/status vocabulary 或导航政策发生语义变化。

本文不创建 Epic 2 schema 字段，不创建 Epic 3 决策模板，不创建 Epic 6 batch runbook，不改变 prompt/template/quality 版本，也不执行批量重写。后续 story 可以细化本文，但不得用局部自动化或新字段绕过当前的来源、时间、锚点和不可验证声明纪律。

## 参考资料

- [统一概念文档规范：新建、升级、审查与仓库集成](./document-generation-methodology.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](./concept-document-contract.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](./intake-and-intent-classification.md)
- [概念文档样例目录：合格、不合格与质量门禁证据](./concept-document-example-catalog.md)
- [统一概念文档模板](./concept-document-template.md)
- [统一概念文档质量门禁](./concept-document-quality-gate.md)
- [固定概念文档生成 Prompt](./fixed-concept-generation-prompt.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](./governance-asset-boundary-policy.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](../governance/agent-behavior-constraints.md)
- [治理资产导航、索引与入口归属政策](../governance/governance-asset-navigation-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](../governance/lifecycle-states.md)
- [Prompt、模板与质量规则版本治理：规则演进、字段语义与渐进迁移](../governance/prompt-template-quality-version-governance.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](../governance/batch-readiness-checklist.md)
