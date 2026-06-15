---
doc_id: governance-agent-behavior-constraints
title: Agent 行为约束：文档治理任务必须先判边界、再执行、可验证
concept: agent_behavior_constraints
topic: governance
depth_mode: standard
created_at: '2026-05-21T10:32:06+08:00'
updated_at: '2026-05-25T09:34:08+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/planning-artifacts/implementation-readiness-report-corrected-2026-05-20.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
time_context: phase_4_foundation_2026_05_25
applicability: knowledge_documentation_governance_agent_work
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: draft
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/batch-readiness-checklist.md
open_questions:
  - Story 0.3 已定义生命周期状态后，是否需要在后续专门审查中把本资产的 quality_status 从 draft 调整为 reviewed 或 maintained_asset？
  - 后续是否需要为 Agent 任务开始声明建立固定 completion report 模板或 checklist？
---

# Agent 行为约束：文档治理任务必须先判边界、再执行、可验证

## 资产角色、权威与适用范围

这份文档是 `Knowledge` 项目中约束 Agent 执行行为的正式治理资产。它约束的是文档治理任务的执行方式，而不是定义新的软件系统、运行时平台或自动化工具。

本资产的作用是把 Agent 在开始、执行、验证和汇报文档治理工作时必须遵守的边界写成可检查规则。它主要适用于：

- 新建、升级、审查或集成 `docs/` 下的正式知识资产
- 修改 `docs/index.md`、`docs/methodology/`、`docs/governance/`、`docs/templates/`、`docs/runbooks/`
- 使用 `_bmad-output/` 中的 PRD、Architecture、Epics、Story、readiness report、review output 作为上下文
- 执行 BMad workflow、skill、story implementation、review 或 planning 类任务
- 维护 Agent-facing instructions、workflow contract、质量门禁、模板、索引和治理规则

它补充但不替代 `docs/methodology/document-generation-methodology.md`。当任务目标是正式概念文档时，主方法论仍是默认入口；当任务目标是 Agent 行为、资产边界、workflow contract 或治理执行纪律时，本资产提供额外约束。

本资产进入 `docs/governance/` 后，属于正式治理规则；同名或相关的 `_bmad-output/` story、PRD、Architecture、Epics 仍然只是 workflow/planning context，不能反向自动覆盖本资产。

本资产当前 `quality_status` 为 `draft`。Story 0.3 已定义生命周期状态词表，但本资产尚未经过单独晋升审查；后续应基于结构、来源、链接、索引和维护触发点证据复核是否晋升为 `reviewed`、`maintained_asset` 或项目约定的等价状态。

## 任务开始前的必填声明

Agent 在修改任何文件前，必须先明确以下信息，并让后续执行与完成汇报能回溯到这些判断。

| 必填项 | 必须说明什么 | 不合格信号 |
| --- | --- | --- |
| 任务类型 | 新建正式文档、升级正式文档、审查、仅更新索引、规范维护、BMad skill/workflow 维护、规划产物、归档/废弃、批量治理等 | 只说“更新一下”“实现一下”，没有判断动作类型 |
| 资产层级 | 目标属于 `docs/` 正式资产、索引、方法论、治理资产、模板、runbook、workflow output、skill 还是报告 | 把 `_bmad-output/` 和 `docs/` 当成同一类资产 |
| 适用规范 | 应服从哪些 methodology、quality gate、workflow、skill、story 或用户显式指令 | 跳过 BMad step、质量门禁或主规范 |
| 允许文件范围 | 本次可以新增、修改或删除哪些文件；哪些文件不得碰 | 顺手重构无关目录或改动未授权资产 |
| 预期产物 | 输出是一篇正式文档、索引项、review report、workflow output、story 状态更新还是对话结论 | 写入位置与产物生命周期不匹配 |
| 验证证据 | 本次完成需要哪些 Markdown、frontmatter、链接、索引、质量门禁、workflow contract 证据 | 只说“已检查”，没有具体证据 |
| 停止/升级条件 | 什么时候必须停下问 Maxwell，而不是自行补造规则或扩大范围 | 发现冲突后继续写，或把假设伪装成项目规则 |

如果任务类型或资产层级无法从用户请求、目标文件和 workflow 上下文中可靠判断，Agent 必须先提出假设或请求澄清；不得直接写入正式资产。

## 资产层级分类

Agent 必须按资产层级选择验证方式和修改边界。

| 资产层级 | 角色 | 默认处理规则 |
| --- | --- | --- |
| `docs/` | 正式知识资产的 canonical location | 新增、移动、删除、改标题、改 topic 或改状态时必须判断 `docs/index.md` 影响 |
| `docs/index.md` | 正式导航入口 | 只反映正式资产和明确保留的报告入口；不得把 planning output 当正式文档加入 |
| `docs/methodology/` | 主规范、模板、质量门禁、fixed prompt、playbook 的规范层 | 修改前必须识别资产角色，不能创建平行主规范或让参考件覆盖主规范 |
| `docs/governance/` | 项目治理政策、Agent 约束、生命周期、索引、命名、迁移、批量治理规则 | 必须声明资产角色、权威、适用范围、与主方法论关系和导航处理 |
| `docs/templates/` | 可复用模板资产 | 必须说明模板适用对象、必填字段、输出格式和不适用场景 |
| `docs/runbooks/` | 可执行操作流程 | 必须说明前置条件、步骤顺序、检查点、停止条件和完成证据 |
| `_bmad-output/` | BMad workflow outputs、规划产物、story、报告、草稿、审查记录 | 可作为上下文，但不是正式 `docs/` 资产；不得自动晋升 |
| `.agents/skills/` | Agent skill/workflow instructions | 修改时必须保护触发条件、输入、输出、步骤顺序、等待点、状态字段和完成条件 |
| report/review outputs | 一次性审查、评估或汇报产物 | 默认是证据记录，不是规范来源；除非经过正式晋升流程 |

正式资产与 workflow output 的生命周期必须分开管理。一个 `_bmad-output/` 文件即使内容正确，也只有在明确晋升、写入 `docs/`、补齐 frontmatter、更新索引并通过适用门禁后，才成为正式知识或治理资产。

## 非软件实施边界

`Knowledge` 的默认实施面是 Markdown、YAML frontmatter、方法论规范、质量门禁、索引、BMad workflow/skill 约束和 Agent 行为规则。Agent 不得默认把本项目当成代码项目。

除非 Maxwell 明确授权“实现可执行工具/自动化程序/软件系统”，Agent 不得新增或修改以下资产：

- Python、`uv`、Typer、Pydantic、pytest 或任何 package scaffold
- `package.json`、`pyproject.toml`、`requirements.txt`、`Cargo.toml`、`go.mod` 等运行时依赖清单
- `src/`、`tests/`、CLI entrypoint、API service、Web/UI、数据库、部署、CI/CD 或 runtime automation
- 软件测试框架、构建脚本、发布配置、容器配置或监控配置

如果用户后续明确授权 executable tooling，Agent 仍必须先执行新的架构或 ADR 判断，说明：

- 工具解决的具体治理问题
- 为什么 Markdown/manual workflow 不足够
- 允许新增的运行时栈、文件范围和测试方式
- 对现有文档治理资产、BMad workflow、质量门禁和索引的影响

未完成上述授权与设计前，任何“为了自动化”“为了完整性”“为了测试”而创建软件项目骨架的行为都视为越界。

## BMad Workflow 与 Skill 执行契约

BMad workflow、skill、story、step file、menu、waiting point、frontmatter progress、sprint status 和 on-complete instruction 是执行协议。Agent 必须把它们当作工作流架构处理。

执行 BMad workflow 或 skill 时，必须遵守以下规则：

- 触发规则：只有用户请求、显式 skill 名称或任务描述命中 skill 时，才进入对应 skill；不得因为偏好自行切换到其他 workflow。
- 输入契约：必须读取 workflow 要求的 story、config、project context、sprint status、methodology、quality gate 或 step file。
- 输出契约：输出位置、文档语言、状态字段、story record、file list、change log 必须符合 workflow 要求。
- 步骤顺序：step file 要求的顺序必须保留；不得跳过前置读取、等待点、质量门禁、状态更新或 on-complete 指令。
- 菜单确认：workflow 要求用户选择菜单、确认范围或批准高风险动作时，Agent 必须等待确认。
- 等待点：workflow 声明必须暂停、HALT、ASK 或请求用户输入时，不得自行补造答案继续。
- 状态更新：story Status、frontmatter 状态、sprint status、review follow-up checkbox、file list 和 change log 必须在规定时机更新。
- 完成指令：workflow 有 on-complete instruction 时，必须在最终退出前执行或明确说明无法执行的原因。

Agent 不得绕过 readiness check、quality gate、index synchronization、review follow-up、story task checkbox、sprint status 或 workflow waiting point。若用户要求绕过，Agent 必须说明治理风险，并取得明确确认后才可继续。

## `_bmad-output/` 使用规则

`_bmad-output/` 的内容可以指导工作，但不能自动成为正式规则。

Agent 使用 `_bmad-output/` 时，必须区分以下状态：

| 类型 | 可以怎么用 | 不得怎么用 |
| --- | --- | --- |
| Planning artifact | 作为 PRD、Architecture、Epics、readiness 的上下文和约束来源 | 不能直接当成正式 `docs/` 资产或索引入口 |
| Draft | 作为候选文本、结构或待审查材料 | 不能在未通过晋升流程前声明为已发布规则 |
| Review output | 作为问题证据、风险记录和修订依据 | 不能把审查意见自动改写成规范，除非用户确认 |
| User-approved rule | 可作为正式化依据 | 仍需写入正确 `docs/` 路径、frontmatter、索引和门禁证据 |
| Formal `docs/` asset | 只有写入 `docs/` 且完成集成后成立 | 不能由 `_bmad-output/` 文件自行“等同”产生 |

从 `_bmad-output/` 晋升到 `docs/` 时，最低流程是：

1. 判断目标文档类型和资产层级。
2. 选择正确 topic 路径和 kebab-case 文件名。
3. 补齐正式 frontmatter 和正文角色说明。
4. 说明来源依据、时间语境和适用范围。
5. 执行适用质量门禁或 workflow contract check。
6. 更新 `docs/index.md` 或明确说明不适用理由。
7. 在完成汇报中列出验证证据和未解决风险。

## 冲突处理与停止条件

当用户请求、系统约束、BMad workflow、project context、methodology、quality gate、story、单篇文档习惯之间出现冲突时，Agent 必须先识别冲突，而不是直接选一个方便的规则执行。

冲突处理必须包含四个判断：

- 冲突来源：哪些文件、workflow step、用户指令或项目规则互相冲突
- 受影响资产：正式文档、索引、方法论、治理资产、skill、workflow output、story 状态或报告
- 优先级：系统级约束 > 用户当前明确指令 > BMad workflow/skill 强制步骤 > 项目主规范 > quality gate/template/playbook/fixed prompt > 单篇文档习惯
- 处理方式：按更高优先级执行、收窄范围、记录例外、请求 Maxwell 确认或停止

以下情况必须停止并请求 Maxwell 确认：

- 需要改变正式资产身份、`doc_id`、路径、topic、生命周期状态或 canonical 入口
- 需要绕过 BMad workflow waiting point、menu confirmation、readiness check 或 quality gate
- 需要把 `_bmad-output/` 自动发布为 `docs/` 正式资产
- 需要修改 `docs/methodology/` 主规范、模板、quality gate、fixed prompt 或 skill/workflow 契约语义
- 需要删除、移动、批量重命名或批量重构正式资产
- 需要新增软件项目骨架、运行时依赖、测试框架、CI/CD、部署或自动化工具
- 关键来源、时间语境、索引一致性或质量状态无法验证

## 完成汇报与验证证据

Agent 报告文档治理任务完成时，必须提供足以复查的证据，而不是只说“已完成”。

完成汇报至少包含：

- 任务类型和资产层级
- 变更文件，区分新增、修改、删除
- 采用的 methodology、quality gate、workflow、skill 或 story 来源
- `docs/index.md` 影响：已更新、不适用或需后续处理
- frontmatter 状态：必填字段、数组字段、`doc_id`、topic、quality_status 是否一致
- Markdown 结构：关键章节、标题层级、链接目标是否存在
- 质量门禁或 workflow contract 检查结果
- 生命周期/status 影响，包括 draft/reviewed/maintained/deprecated 等状态解释
- 未解决风险、待核查项或用户确认项
- 非软件边界确认：未新增未授权 package、runtime、CLI、API、UI、database、deployment、CI/CD 或 software test artifact

验证结果必须使用 `通过`、`未通过`、`未验证` 或 `不适用` 表达。若存在 Hard Fail、关键未验证项或未经授权的范围扩大，不得把任务标记为完成。

## 验证入口

后续 Agent 可用下面的问题快速判断自己是否遵守了本资产：

1. 我是否在编辑前明确了任务类型、资产层级、适用规范、允许文件范围和预期产物？
2. 我是否把 `Knowledge` 当作 Markdown documentation/governance system，而不是默认当作软件项目？
3. 我是否完整执行了相关 BMad workflow/skill 的 step order、waiting point、state update 和 on-complete instruction？
4. 我是否区分了 `_bmad-output/` planning context、draft、review output、user-approved rule 和正式 `docs/` asset？
5. 如果新增或变更正式 `docs/` 资产，我是否补齐 frontmatter、正文角色说明、索引影响和质量证据？
6. 如果发现冲突，我是否指出冲突来源、受影响资产、优先级和停止/确认路径？
7. 我的完成汇报是否列出了具体文件、验证证据、未解决风险和非软件边界确认？

任一问题回答为“否”时，本次任务不能直接标记完成；必须先补齐证据、收窄范围或请求 Maxwell 确认。

## 维护与迁移入口

本资产后续可能被以下 story 或治理变更影响：

- Story 0.2：主规范、模板、质量门禁、playbook 与 fixed prompt 的职责边界
- Story 0.3：文档生命周期状态与状态转换规则
- Story 0.4：Prompt、模板与质量规则版本治理
- Story 0.5：质量门禁 Hard Fail、评分维度与质量状态基线
- `docs/governance/governance-asset-navigation-policy.md`：治理资产导航、索引与入口归属基线
- `docs/governance/batch-readiness-checklist.md`：批量治理 readiness、范围判定、冲突暴露、停止条件与恢复策略

这些规则落地后，应复核本资产中的 `quality_status`、生命周期词表、索引处理规则、completion report 要求和 workflow contract 表述是否需要同步更新。Story 0.7 已落地为 batch readiness policy；后续如 Epic 2、Epic 3 或 Epic 6 改变 schema、report 或 runbook 形态，再按对应治理资产复核。

本资产已列入 `docs/index.md` 的 `governance` section。若未来移动、重命名、废弃或拆分，必须同步处理 `doc_id`、索引入口、相关文档链接和替代入口说明。
