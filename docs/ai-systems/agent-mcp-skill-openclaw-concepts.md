---
doc_id: ai-systems-agent-mcp-skill-openclaw-concepts
title: AI 代理栈分层：Agent、MCP、Skill 与 OpenClaw 的概念边界
concept: agent_stack_layering
topic: ai-systems
depth_mode: deep
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-03-20T17:09:33+08:00'
source_basis:
  - openclaw_homepage_checked_2026_03_19
  - openclaw_gateway_architecture_checked_2026_03_19
  - openclaw_agent_runtime_docs_checked_2026_03_19
  - openclaw_agent_loop_docs_checked_2026_03_19
  - mcp_architecture_spec_checked_2026_03_19
  - mcp_architecture_concepts_docs_checked_2026_03_19
time_context: current_practice_checked_2026_03_19
applicability: ai_agent_architecture_analysis_protocol_layering_and_product_boundary_judgment
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/programming-languages/callback-lifetime-management.md
open_questions:
  - OpenClaw 与其他长期代理平台的系统边界对比是否需要单独成文？
  - MCP 在生产环境中的权限与安全边界是否需要补一节工业案例？
---

# AI 代理栈分层：Agent、MCP、Skill 与 OpenClaw 的概念边界

## 1. 这份文档要帮你学会什么

这篇文档的目标，不是把 `Agent`、`MCP`、`Skill`、`OpenClaw` 四个词逐个解释，而是把它们压成一套以后分析 AI 产品、编码代理、企业助手和长期在线代理平台时都能调用的分层模型。

读完后，你应该至少能做到：

- 区分什么是执行闭环、什么是能力协议、什么是操作知识资产、什么是产品运行时
- 判断一个系统的问题出在 agent loop、tool contract、skill 设计，还是产品编排边界
- 识别“有很多工具”但其实没有 agent、“有 prompt”但没有 skill、“有 server”但没有协议治理的伪分层
- 用统一框架分析 OpenClaw 这类长期在线代理平台和一般 coding agent 工作台的差异

## 2. 一句话结论 / 问题定义

**在 AI 代理栈里，Agent 解决的是目标驱动执行闭环，MCP 解决的是能力与上下文如何标准接入，Skill 解决的是操作知识如何沉淀和复用，OpenClaw 解决的是如何把这些分层资产编排成一个可长期运行、可连接真实世界、可治理的产品系统。**

这组概念真正要解决的问题不是“AI 应用里名词为什么这么多”，而是：

- 不同抽象层到底各自负责什么
- 能力接入、行为约束、执行循环和产品运行时应该怎样解耦
- 为什么复杂代理系统不能只靠“一个更长的 prompt + 一堆工具”维持稳定

## 3. 对象边界与相邻概念

这篇文档里四个对象的边界是：

- `Agent`：围绕目标感知、决策、行动、读取反馈的执行闭环
- `MCP`：把外部能力与上下文以 host / client / server 协议暴露给 AI 应用的标准接口层
- `Skill`：把 SOP、约束、操作步骤和工作模式压缩成可复用行为资产
- `OpenClaw`：把长期连接、会话、节点、代理循环和产品通道编排起来的代理平台

它们不等于：

- `Agent` 不等于“会聊天的模型”
- `MCP` 不等于“某个具体工具集合”
- `Skill` 不等于“随便一段 prompt”
- `OpenClaw` 不等于“另一个聊天 UI”

最容易混淆的相邻概念是 workflow、tool/plugin、gateway、runtime、system prompt 和 product orchestration。

## 4. 核心结构

分析这组概念时，最稳的最小结构是：

```text
用户目标
  -> Agent loop
  -> Skill / policy / workflow knowledge
  -> Tool / Resource / Prompt contract
  -> MCP host / client / server protocol
  -> 外部系统
  -> Product runtime / gateway / session orchestration
```

真正值得记住的是：`Agent`、`MCP`、`Skill`、`OpenClaw` 不是同层对象，而是“执行层、协议层、知识资产层、产品编排层”的分层组合。

## 5. 核心机制 / 主链路 / 因果链

一个最小可运行链路可以压成 6 步：

1. 用户目标进入 host / gateway
2. Agent loop 基于当前状态判断下一步动作
3. Skill 提供约束、步骤模板和行为边界
4. Host 通过 MCP client 访问 server 暴露出的 tools / resources / prompts
5. 外部系统返回结果，Agent 再读取反馈继续循环
6. Product runtime 负责会话、权限、节点连接、长生命周期和多通道交互

这条链路说明两个关键点：

- 没有标准化能力面，Agent 很快会退化成 ad-hoc tool glue
- 没有 product runtime，很多“长期在线代理”其实只是一次性推理包装

## 6. 关键 tradeoff 与失败模式

这套分层买到的是可治理性、可扩展性和复用边界；代价是系统复杂度、权限设计和运行时编排成本显著上升。

最常见的失败模式是：

- 用一个超长 system prompt 同时承担 skill、policy、tool contract 和 runtime 责任
- 把“能调用很多工具”误当成“已经有 agent”
- 工具接口没有协议化，导致每个产品都发明一套 ad-hoc JSON
- 长期在线代理默认持有高权限、弱审计、弱隔离的执行面
- 把 skill 和 tool 混成一层，导致知识资产无法迁移、能力资产无法治理

## 7. 应用场景

这套模型最适合分析：

- AI 编码工作台与 repo agent
- 企业内部工单、运维、知识库与审批助手
- 多渠道长期在线个人助理
- 需要把权限、能力接入、执行循环和产品编排拆层的代理平台

## 8. 工业 / 现实世界锚点

### 8.1 OpenClaw Gateway / Agent Loop

OpenClaw 官方架构把 gateway、client websocket、node connection、agent runtime 和 agent loop 明确拆开，这就是“产品编排层不等于单次模型调用”的真实锚点。

### 8.2 MCP Host / Client / Server

MCP 官方架构把 host、client、server 角色明确区分，并把 tools、resources、prompts 视为协议原语。这说明能力接入可以被协议化，而不是每个代理产品各自硬编码。

### 8.3 代理式编码工作台

编码代理工作台是这套模型最直接的现实落点：Agent 负责任务闭环，Skill 负责 SOP 与行为约束，MCP 或等价协议负责接文件系统 / 终端 / 浏览器 / 数据源，产品运行时负责权限与审计。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-19 更推荐的实践

当前更稳的方向通常是：

- 协议化能力接入，而不是靠提示词暗示模型“大概能做什么”
- 最小权限、审批点和可回放执行链，而不是默认高权限长期在线
- versioned skill / workflow 资产，而不是把操作知识堆进一个单体 system prompt
- 明确分开 protocol、capability、skill、runtime 和 product orchestration

### 9.2 过时路径与替代

下面这些路径通常已经不够稳：

- 一个巨大的单体 prompt 同时承担所有层的责任
- 每个产品都发明一套 ad-hoc tool schema
- 只做工具接入却不设计 skill、权限和运行时治理
- 默认让长期在线代理静默执行高风险动作

更稳的替代是：

- 用 MCP 这类协议或等价 contract 明确能力边界
- 用 skill / workflow 资产固化操作知识
- 用 gateway / runtime 设计承接长生命周期、会话和权限控制

## 10. 自测题 / 验证入口

1. 为什么 `Agent`、`MCP`、`Skill`、`OpenClaw` 不是同一层概念？
2. 什么时候一个“多工具聊天产品”还不能算真正的 agent？
3. 为什么 skill 不能简单等同于一段 prompt？
4. MCP 解决的是“模型更聪明”还是“能力接入更标准”？
5. 长期在线代理如果没有 runtime、权限和审计设计，最容易出什么问题？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- coding agent 产品
- 企业代理框架
- plugin / extension 生态
- tool calling 与 workflow 编排
- 任何需要分清“能力接入、行为约束、执行循环、产品运行时”的 AI 系统

## 12. 未解问题与继续深挖

- MCP 在生产环境里的权限、审批、多租户和沙箱模型怎样进一步制度化？
- 长期在线代理平台与轻量交互式 coding agent 工作台的最优边界怎样划分？
- skill 资产如何版本化、测试化、审计化，而不重新滑回“超长 prompt”？

## 13. 参考资料

以下“当前实践”相关内容的核对日期均为 `2026-03-19`。

- OpenClaw homepage: https://openclaw.ai/
- OpenClaw architecture: https://docs.openclaw.ai/architecture
- OpenClaw agent runtime: https://docs.openclaw.ai/concepts/agent
- OpenClaw agent loop: https://docs.openclaw.ai/concepts/agent-loop
- MCP architecture specification: https://modelcontextprotocol.io/specification/2025-06-18/architecture
- MCP concepts, architecture: https://modelcontextprotocol.io/docs/concepts/architecture

## 14. 详细展开与原有分项拆解

### 1. 这份文档要帮你学会什么

这份文档不是一份安装教程，也不是术语词典。

它的目标是基于 [learning-new-things-playbook.md](../methodology/learning-new-things-playbook.md) 和 [cognitive-modeling-playbook.md](../methodology/cognitive-modeling-playbook.md) 的方法，帮你建立一套可操作的内部模型，让你真正看懂：

- `Agent` 在 AI 应用里到底是什么
- `MCP` 到底解决什么问题，位于哪一层
- `skill` 到底是能力、提示词、还是工具封装
- `OpenClaw` 为什么会火，它的理念是什么，它是如何运作的

这份文档的学习标准不是“听过这些词”，而是你读完之后至少能做到：

- 用自己的话解释这些概念
- 说清它们彼此的边界
- 画出一个最小运行链路
- 判断一个系统的问题出在“模型层、协议层、能力层、还是产品编排层”

一句话先给结论：

**Agent 是执行闭环，MCP 是接入协议，skill 是可复用的操作知识包，OpenClaw 是把这些东西编排成一个长期在线个人代理平台的产品与系统。**

---

### 2. 先建立一张总地图

很多人一开始学这些概念时会混乱，原因不是概念本身太难，而是把不同层级的东西混在一起了。

你可以先把 AI 应用粗略分成五层：

```text
用户目标
  ↓
Agent 层
  负责：理解目标、决定下一步、维护状态、循环执行
  ↓
Skill / Prompt 层
  负责：告诉模型“遇到什么问题时，按什么规则做事”
  ↓
Tool / Integration 层
  负责：真正执行动作，比如读文件、发消息、查日历、跑命令
  ↓
MCP / API 协议层
  负责：把外部能力以标准方式暴露给 AI 应用
  ↓
外部世界
  文件、数据库、浏览器、聊天平台、设备、SaaS、操作系统
```

从这个角度看：

- `Agent` 关注“如何决策与执行”
- `skill` 关注“模型知道怎么做”
- `tool` 关注“系统能做什么”
- `MCP` 关注“这些能力如何标准化接进来”
- `OpenClaw` 关注“如何把这些东西做成一个长期在线、可扩展、跨渠道、能真的帮你做事的产品系统”

如果你先把层次理顺，后面的概念就不会缠在一起。

---

### 3. Agent 到底是什么

#### 3.1 Agent 不是“会聊天的模型”

一个普通聊天模型只做一件事：

- 你给输入
- 它生成输出

这更像一次性映射。

而 `Agent` 的核心不是“会回答”，而是它形成了一个**围绕目标持续运行的执行闭环**。它通常具备五个要素：

- 目标：它在替谁完成什么事
- 状态：它记得当前上下文、历史动作和中间结果
- 决策：它能判断下一步该做什么
- 动作：它能调用工具、读写环境、触发外部系统
- 反馈：它会读取动作结果，修正后续行为

所以，Agent 的本质不是一个更华丽的聊天框，而是：

**让模型从“只生成文本”变成“围绕目标持续感知、决策、行动、再修正”的执行体。**

#### 3.2 Agent 的最小运行闭环

可以把一个最小 Agent 想成下面这条链：

```text
接收目标
  ↓
理解当前状态
  ↓
选择下一步动作
  ↓
调用工具或输出消息
  ↓
读取结果
  ↓
判断是否继续
  ↓
完成 / 失败 / 等待下一次触发
```

比如“帮我把今天的重要邮件整理成待办并发给我”：

- 它先要知道你要什么结果
- 再去读邮件
- 提取重要项
- 生成待办
- 可能还要发回 Telegram 或 Slack
- 如果权限不足、结果为空、格式冲突，它还要重试或换路径

这就是 agentic behavior。重点不是“有很多步骤”，而是“它自己维护执行闭环”。

#### 3.3 Agent 和几个近似概念的边界

#### Agent vs Chatbot

`Chatbot` 的重点是对话。

`Agent` 的重点是任务闭环。

一个聊天机器人可以没有工具、没有状态规划、没有长期目标；一个 Agent 即便通过聊天界面被触发，它的核心也不是聊天，而是执行。

#### Agent vs Workflow

`Workflow` 更像预先写好的固定流程。

它通常是：

- 第一步做 A
- 第二步做 B
- 第三步做 C

而 Agent 的差异在于：

- 它可以根据环境状态临时选择路径
- 它不一定提前知道完整步骤
- 它会根据结果调整下一步

固定工作流更像流水线，Agent 更像带一定自主性的操作员。

#### Agent vs Automation Script

脚本是人把逻辑提前写死。

Agent 是人只给目标和边界，它在运行时自己选择具体动作。

这并不代表 Agent 一定比脚本更好。很多场景里，固定脚本更稳定、更便宜、更可审计。Agent 的价值在于处理不完全结构化的任务。

#### 3.4 Agent 的强项和弱点

强项：

- 能处理半结构化目标
- 能在工具之间做动态编排
- 能把“复杂目标”分解成多个局部动作
- 对人来说交互成本低，你更像在下达任务，而不是写程序

弱点：

- 不稳定，容易受提示词和上下文影响
- 容易权限过大
- 工具一多，误操作空间也会变大
- 长链路任务中的错误会累积

所以判断一个系统是不是 Agent，不要看它是不是“用了大模型”，而要看它是否真的具备目标驱动、状态管理、动作选择和反馈闭环。

---

### 4. MCP 到底是什么

#### 4.1 先讲它解决的问题

在 `MCP` 出现之前，AI 应用要连接外部能力时，通常是每个应用自己写一套接入：

- 接 GitHub 一套
- 接文件系统一套
- 接数据库一套
- 接浏览器一套
- 接企业内部系统又一套

这样的问题是：

- 每个 AI 应用都要重复造轮子
- 能力描述不统一
- 安全边界很难标准化
- 工具一多，维护成本越来越高

`MCP` 的目标就是把“AI 应用如何接入外部能力和上下文”这件事标准化。

所以你可以把 MCP 理解为：

**模型上下文与工具接入的标准协议层。**

它不替你做推理，不替你决定任务策略，也不自动把系统变成强 Agent。它只是把“能力接口”做成一个公共标准。

#### 4.2 MCP 的最核心分工

根据 MCP 官方架构，最重要的角色有三类：

- `Host`：真正承载用户和模型的 AI 应用，比如桌面客户端、IDE、聊天产品、agent 平台
- `Client`：Host 内部连接某个 MCP Server 的会话实例
- `Server`：向 Host 暴露能力的服务

一个重要原则是：

**Host 管编排，Server 管能力。**

也就是说：

- Host 决定什么时候连哪个 Server
- Host 决定把哪些能力展示给模型
- Server 只专注提供具体能力，不负责整个应用的任务编排

#### 4.3 MCP 的三种核心 primitive

MCP 官方最重要的概念是三个 primitive：

- `Tools`
- `Resources`
- `Prompts`

它们分别解决不同问题。

#### 1. Tools

`Tool` 是可执行动作。

例如：

- 搜索文件
- 发 HTTP 请求
- 查询数据库
- 创建 issue
- 执行某个 API 调用

工具的特点是：**会产生动作或副作用**。

#### 2. Resources

`Resource` 是可读取上下文。

例如：

- 某个文档内容
- 某份配置
- 某条数据库记录
- 某个 API 返回的数据

它的重点不是“做事”，而是“提供上下文”。

#### 3. Prompts

`Prompt` 是可复用的提示模板或工作模板。

它更像：

- 一个可以带参数的任务模板
- 一组结构化消息
- 一个标准化交互入口

它不是底层执行器，而是“如何组织与模型的交互”。

#### 4.4 MCP 是怎么运作的

从运行链路看，一个典型的 MCP 交互大致是这样：

```text
AI 应用启动
  ↓
Host 连接一个或多个 MCP Server
  ↓
做协议握手、版本协商、能力发现
  ↓
Client 获取这个 Server 提供的 tools/resources/prompts
  ↓
模型或应用逻辑决定调用某个能力
  ↓
Host 通过 Client 向 MCP Server 发请求
  ↓
Server 执行动作或返回资源
  ↓
结果回到 Host
  ↓
Host 再把结果放回模型上下文或交给用户界面
```

你可以把它想成“AI 世界里的 USB-C 接口”。

这不是一个精确的技术等价，但对理解很有帮助：

- 它的价值不是替设备工作
- 它的价值是把连接方式标准化

#### 4.5 MCP 的价值和边界

价值：

- 让能力接入变得标准化
- 让不同 AI 应用可以复用同一套服务
- 把工具、资源、提示模板做成统一发现和调用机制
- 有利于权限、授权、审计和安全边界的统一设计

边界：

- MCP 不负责高质量任务规划
- MCP 不保证模型一定会正确用工具
- MCP 不等于 Agent
- MCP Server 不是“会思考的东西”

一句话说，`MCP` 让“接什么能力”变容易，但不保证“怎么用这些能力”就一定聪明。

---

### 5. skill 到底是什么

#### 5.1 先说一个关键事实：skill 不是统一标准词

`skill` 这个词在 AI 应用里很常见，但它不像 `MCP` 那样有统一协议规范。

不同产品里，skill 可能指：

- 一组专门指令
- 某类任务的操作手册
- 一套模板 + 例子 + 资源
- 一个带脚本和文档的能力包
- 一个“什么时候该调用什么工具”的知识包

所以学 `skill` 时最容易犯的错，就是把它误当成某个标准化技术名词。

更准确地说：

**skill 是一种产品层和应用层的抽象，不是底层协议标准。**

#### 5.2 skill 的本质是什么

如果从功能上看，skill 的本质是：

**把某一类任务的操作知识、约束、策略和最佳实践，打包成模型可复用的能力说明。**

它通常回答的是：

- 遇到什么场景应该使用我
- 目标应该如何拆解
- 哪些工具适合用
- 调用顺序和约束是什么
- 什么情况应该停止或回退

所以 skill 更像“给模型看的操作手册”，而不是“真正干活的执行器”。

#### 5.3 skill、tool、prompt 的关系

这是最容易混淆的部分。

#### skill vs tool

`tool` 负责执行。

`skill` 负责教模型如何用这些执行器。

例如：

- `tool`：一个能搜索邮件的接口
- `skill`：一份告诉模型“当用户要整理邮件时，先筛发件人、再按时间、再提取行动项”的说明

所以工具像手和脚，skill 像操作经验。

#### skill vs prompt

`prompt` 可以只是一次性的输入。

`skill` 通常是可复用、可发现、可装载的知识块。

一个 skill 往往会影响 prompt，但 skill 自己不等于一次用户输入。

#### skill vs MCP Server

`MCP Server` 提供的是标准化能力端点。

`skill` 提供的是“如何在某类任务里理解和使用这些能力”的策略。

前者偏接口层，后者偏认知层。

#### 5.4 一个很实用的判断方法

当你看到某个能力包时，可以问四个问题：

- 它自己能执行动作吗
- 它只是提供能力接口，还是还告诉模型如何使用
- 它是否围绕某类任务复用
- 它的主要价值是“连接能力”还是“传授策略”

如果答案更偏“传授策略”，那它大概率更像 skill。

#### 5.5 在代理式编码环境里，skill 往往长什么样

在很多 coding agent 或 personal agent 系统中，skill 往往不是一个神秘组件，而是一个很朴素的包：

- 一个 `SKILL.md`
- 若干说明文档
- 可选的脚本、模板、资源文件
- 一些元信息，用于描述触发场景和用途

例如在当前这个环境里，`AGENTS.md` 就明确说明：skill 是存放在 `SKILL.md` 中的一组本地说明，可按需加载，并可引用脚本、模板和资源。这非常能说明它的本质：

**skill 是把“做某类事情的方法”组织成可复用资产。**

---

### 6. 把 Agent、MCP、skill 放在一起看

你可以用下面这张表，快速定位三者区别：

| 概念 | 它解决什么问题 | 它最像什么 | 它不是什么 |
| --- | --- | --- | --- |
| Agent | 如何围绕目标持续决策和执行 | 一个带状态和反馈的执行闭环 | 单次聊天回复 |
| MCP | 如何把外部能力标准化接入 AI 应用 | 协议和接线标准 | 任务规划器 |
| skill | 如何把某类任务的方法教给模型 | 可复用操作知识包 | 底层执行接口 |

如果再加上 `tool`：

| 概念 | 核心职责 |
| --- | --- |
| tool | 真正执行动作 |
| skill | 告诉模型什么时候、为什么、怎样使用 tool |
| MCP | 把 tool/resource/prompt 以标准方式暴露出来 |
| Agent | 决定在当前目标下是否调用这些能力，并形成执行闭环 |

到这里，你应该能看到：

**Agent 决策，skill 教方法，tool 做动作，MCP 负责标准化接入。**

---

### 7. OpenClaw 是什么

#### 7.1 它不是一个“会聊天的应用”，而是一种产品理念

OpenClaw 官网首页把它定位为：

- 你的个人 AI assistant
- 运行在你自己的机器上
- 可以接入你已经在用的聊天平台
- 能“真的做事”

官方文档则把 `Gateway`、`Agent Runtime`、`Agent Loop`、`Skills`、`Workspace` 作为核心概念。

从这些官方表述可以归纳出它的设计理念：

**OpenClaw 不是想做一个新的聊天窗口，而是想做一个长期在线、可接入真实世界、可持续执行任务的个人代理基础设施。**

这是它和很多“把大模型塞进 App”产品的根本差别。

#### 7.2 OpenClaw 想解决什么问题

它针对的不是“我想和 AI 聊天”，而是下面这类需求：

- 我想通过我本来就在用的入口和 AI 交互，比如 WhatsApp、Telegram、Slack、Discord
- 我希望 AI 有长期记忆和持续身份，而不是每次都重开
- 我希望 AI 能真正访问文件、命令行、浏览器、设备和外部服务
- 我希望这些能力是我自己掌控的，而不是全在封闭平台里

所以 OpenClaw 的目标函数，不是做“最好看的聊天体验”，而是做：

**一个你自己拥有、始终在线、能接触现实环境的个人代理平台。**

#### 7.3 OpenClaw 的核心理念

如果压缩成几个关键词，我会这样概括：

- 常驻：它不是临时开一个页面，而是有长期运行的 Gateway
- 多入口：你可以从各种聊天渠道和控制界面触发它
- 私有化：强调运行在你自己的设备或你控制的环境中
- 可扩展：可以增加 skills、tools、plugins、channel connectors
- 代理化：重点不是回答，而是执行和持续协作

OpenClaw 官网那句很关键的 slogan 是 “The AI that actually does things”。这不是一句营销口号而已，它实际上在说明这个产品的第一性目标：

**让 AI 从“文本回答器”变成“可操作现实世界的执行代理”。**

#### 7.4 OpenClaw 的最小架构图

根据官方 `Gateway Architecture`、`Agent Runtime`、`Agent Loop` 文档，可以把它压缩成下面这张图：

```text
用户 / 消息渠道 / 控制客户端
(WhatsApp, Telegram, Slack, Discord, CLI, Web UI, macOS app)
  ↓
Gateway
  负责统一接入、连接管理、事件流、身份与配对、协议校验
  ↓
Agent Runtime
  负责会话、模型、prompt 组装、skills 注入、工具调用
  ↓
Tools / Nodes / 外部系统
  文件、命令行、浏览器、设备能力、消息发送、各种集成
  ↓
结果回流
  流式输出、最终回复、持久化、后续继续触发
```

一个关键设计点是：

**Gateway 是控制平面，assistant 才是产品本体。**

也就是说，Gateway 负责把各种入口和能力面连起来，但用户真正感受到的是那个长期存在、会记住你、会继续做事的 agent。

#### 7.5 OpenClaw 是如何运作的

这是理解 OpenClaw 的核心。

根据官方 `Agent Loop` 文档，一个真实运行的 agent loop 大致是：

```text
消息进入系统
  ↓
Gateway 接收并校验请求
  ↓
解析会话，确定 sessionKey / sessionId
  ↓
准备 workspace、上下文文件、skills 快照
  ↓
组装 system prompt
  包含基础提示、skills、bootstrap/context、按需覆盖项
  ↓
进入嵌入式 agent runtime
  ↓
模型推理
  ↓
调用工具
  ↓
把工具结果回填上下文
  ↓
继续推理 / 流式输出 / 结束
  ↓
持久化会话与运行结果
```

这里面有几个关键机制非常重要。

#### 1. 它不是一次性调用，而是完整的 loop

OpenClaw 官方把 agent loop 定义为：

- intake
- context assembly
- model inference
- tool execution
- streaming replies
- persistence

这说明它的重点不是“模型生成了一次回答”，而是“把一次真实任务运行的整个生命周期管起来”。

#### 2. 它强调每个 session 的串行化

官方文档明确写了：

- 每个 session 的 run 会被串行化
- 还可以走全局队列
- 这样可以避免工具竞争和 session 历史混乱

这背后反映的是一个很重要的工程思想：

**Agent 系统不是只要模型聪明就行，还必须先保证状态一致性。**

如果两个并发任务同时改同一段上下文、同一批文件、同一个消息线程，系统很容易乱。

所以 OpenClaw 的一个核心原理，是先用队列和锁把 agent loop 变成可管理的真实系统，而不是纯 demo。

#### 3. 它把 skill 注入到 prompt，而不是把 skill 神秘化

官方 `System Prompt` 文档说明：

- 当有可用 skills 时，系统提示中会注入一个精简的 available skills 列表
- 列表里带有 skill 的位置
- 模型被指示在需要时再去读对应的 `SKILL.md`

这件事非常值得理解。

它说明在 OpenClaw 里，skill 的工作方式不是：

- “神奇地自动执行”

而是：

- 先告诉模型有哪些可用能力知识包
- 真需要时再去读取对应文档
- 再把那些知识转化为后续动作策略

也就是说，skill 在 OpenClaw 里本质上还是**面向模型的知识注入与策略注入**。

#### 4. 它把长期运行所需的“人格、记忆、工作约束”外显成文件

`Agent Runtime` 文档里提到，workspace 中会有一些关键文件，例如：

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `BOOTSTRAP.md`

这意味着什么？

意味着 OpenClaw 把一个长期在线 agent 的一部分“自我约束”和“工作记忆”外显出来了。

这是一种很强的理念：

**代理不是黑箱人格，而是可以被观察、编辑、积累和逐步塑形的运行体。**

#### 5. 它把 streaming、timeout、retry、suppression 都纳入 agent runtime

官方 `Agent Loop` 文档还说明了：

- assistant deltas 会流式输出
- tool 事件会单独流出
- 超时会中止运行
- compaction 会触发压缩与重试
- 某些重复消息会被抑制

这说明 OpenClaw 的“原理”不只是模型和工具，而是：

**把真实代理系统的工程问题一起纳入设计。**

如果没有这些部分，很多 agent 产品只能演示一次成功路径，无法长期稳定运行。

#### 7.6 OpenClaw 为什么会火

这里我不讲传播故事，只讲从产品原理上看，它为什么会让人感到“这东西不一样”。

基于官网与官方文档，我的归纳是：

#### 1. 它把“AI 会做事”这件事做成了可感知体验

很多 AI 产品的体验是：

- 你问，它答

而 OpenClaw 的体验更像：

- 你交代，它去做

这会让用户明显感到范式变化。

#### 2. 它把入口放在用户已经在用的聊天渠道里

这很关键。

当一个 agent 活在 WhatsApp、Telegram、Slack、Discord 这种你已经高频使用的入口里，它就更像“持续在线的助手”，而不是“需要专门打开的网站”。

#### 3. 它强调“你的机器、你的上下文、你的技能”

OpenClaw 官网强调：

- 跑在你自己的机器上
- private by default
- skills and plugins can extend it

这对很多用户的吸引力不是抽象的“开源”，而是：

**我能把它逐步塑造成真正属于我的代理。**

#### 4. 它把 agent 做成平台，而不是单个功能

它不是“一个邮件总结器”或“一个日历助手”。

它更像一个平台：

- 多渠道输入
- 多工具输出
- 长期记忆
- 多种模型
- 插件与 skill 扩展
- 节点与设备能力

一旦用户意识到这是平台，而不是单点功能，想象空间会被迅速拉大。

所以，如果你问“为什么 OpenClaw 会火”，更本质的答案不是“某个模型特别强”，而是：

**它把个人 agent 从概念演示，推进成了一个可持续运行、可扩展、可自定义的操作平台。**

#### 7.7 OpenClaw 不是什么

理解一个东西，最重要的方法之一是看它不是什么。

OpenClaw 不是：

- 不是单纯的聊天机器人
- 不是等同于某个模型
- 不是 MCP 的同义词
- 不是单个 tool 集合
- 不是固定工作流引擎

它更像：

- 一个代理平台
- 一个长期在线的控制平面加运行时
- 一个把技能、工具、上下文、渠道、设备和会话管理组织到一起的系统

#### 7.8 OpenClaw 的边界与风险

如果只看“它好强”，那理解还是不完整。

真正成熟的理解必须看到失败模式。

#### 1. 权限风险

当 agent 能访问文件、命令行、浏览器、消息渠道和外部账户时，风险不是线性增长，而是组合爆炸。

问题不只是“某个工具危险”，而是：

- 错误理解用户意图
- 错误调用高权限工具
- 错误把结果发到外部渠道

#### 2. 提示注入和上下文污染

当 agent 会读网页、文档、消息和第三方 skill 时，它会暴露在大量不可信输入前。

这类系统的安全问题，很多时候不是传统漏洞，而是：

- 读到了恶意指令
- 把不该信任的内容当成高优先级约束
- 让长期记忆被污染

#### 3. 长期运行的状态漂移

长期在线的 agent 很容易出现：

- 人设偏移
- 记忆堆积失真
- 技能越来越多但边界越来越乱
- 上下文窗口被无效历史挤占

所以“长期在线”既是它的魅力，也是它的治理难点。

#### 4. 工程复杂度

一旦一个系统同时处理：

- 多渠道
- 多模型
- 流式输出
- 工具调用
- 权限与配对
- 会话一致性
- 超时与重试

它就不再只是一个 prompt 工程项目，而是一个真正的分布式应用和运行时工程问题。

这也是为什么 OpenClaw 的原理不能简单理解成“提示词写得好”。

---

### 8. 用 playbook 的方法复盘这四个概念

为了符合“真正学会”的标准，我们最后按统一模板压缩一次。

#### 8.1 Agent

- 目的：围绕目标持续决策和执行
- 边界：负责规划与动作选择，不等于底层能力接口
- 构成：目标、状态、模型、工具、反馈回路
- 机制：感知 -> 决策 -> 行动 -> 反馈 -> 再决策
- 失败模式：幻觉规划、误调用工具、长链路累错、权限失控

#### 8.2 MCP

- 目的：标准化 AI 应用与外部能力/上下文的连接方式
- 边界：负责协议与能力发现，不负责任务规划
- 构成：Host、Client、Server、Tools、Resources、Prompts
- 机制：连接 -> 协商 -> 发现能力 -> 调用 -> 返回结果
- 失败模式：把 MCP 神化成 Agent、权限设计不清、工具描述不可信

#### 8.3 skill

- 目的：把某类任务的方法和约束教给模型
- 边界：偏知识层，不是执行器，也不是统一协议
- 构成：说明文档、规则、模板、例子、可选脚本和资源
- 机制：被发现 -> 被读取 -> 进入模型策略 -> 指导后续动作
- 失败模式：写成空话、边界模糊、和 tool 职责重叠、知识过期

#### 8.4 OpenClaw

- 目的：把个人 AI assistant 做成长期在线、可接入真实世界的代理平台
- 边界：是平台和运行时，不等于某个单独模型或协议
- 构成：Gateway、Agent Runtime、Agent Loop、Workspace、Skills、Tools、Channels、Nodes
- 机制：消息进入 -> 会话解析 -> 上下文与 skill 装配 -> 模型推理 -> 工具执行 -> 流式输出 -> 持久化
- 失败模式：高权限误操作、提示注入、长期状态漂移、复杂度过高

---

### 9. 自测题：检验你是否真的理解了

如果你想验证自己是不是已经从“听过”走到了“解释”甚至“应用”层，可以试着回答下面的问题。

#### 1. 如果一个系统只是“用户提问，模型回答”，没有状态管理，也不会调用工具，它算 Agent 吗？为什么？

#### 2. 如果一个系统接入了很多 MCP Server，但模型不会规划，也不会合理用工具，它为什么仍然可能表现很差？

#### 3. 如果一个 skill 只写“你要乐于助人、尽量聪明”，这为什么通常是个差 skill？

#### 4. 为什么说 OpenClaw 的关键不只是模型，而是 Gateway、Agent Loop 和 Workspace 一起构成的系统？

#### 5. 如果你要自己设计一个类似平台，哪些东西属于协议层，哪些属于运行时，哪些属于知识层？

如果你能不看原文，自己清楚回答这五题，说明你已经建立了一个比较稳的模型。

---

### 10. 最后一层压缩

把全文再压缩成最短版本：

- `Agent` 是围绕目标运行的执行闭环
- `MCP` 是让外部能力标准化接入 AI 应用的协议
- `skill` 是教模型如何在某类场景里做事的知识包
- `OpenClaw` 是把聊天入口、长期记忆、工具能力、技能体系和代理运行时编排成一个“始终在线、真的能做事”的个人 AI 平台

如果你以后再看到新的 agent 产品，就按这四个问题去拆：

- 它的 Agent 闭环是否真实存在
- 它的能力接入是走什么协议
- 它的 skill 或 prompt 体系如何教模型做事
- 它的产品编排是否支持长期稳定运行

这样你就不会再被术语带着走，而是能看见底层结构。

---

### 11. 参考资料

以下是写作时主要依据的官方资料。涉及“为什么会火”的判断，是我基于这些官方资料做的归纳，不是官方原话。

- OpenClaw 官网首页：https://openclaw.ai/
- OpenClaw GitHub 仓库主页：https://github.com/clawdbot/clawdbot
- OpenClaw Gateway Architecture：https://docs.openclaw.ai/architecture
- OpenClaw Agent Runtime：https://docs.openclaw.ai/concepts/agent
- OpenClaw Agent Loop：https://docs.openclaw.ai/concepts/agent-loop
- OpenClaw System Prompt：https://docs.openclaw.ai/concepts/system-prompt
- OpenClaw Pi Integration Architecture：https://docs.openclaw.ai/pi
- MCP Architecture：https://modelcontextprotocol.io/specification/2025-06-18/architecture
- MCP Basic Overview：https://modelcontextprotocol.io/specification/2025-03-26/basic
- MCP Concepts: Architecture：https://modelcontextprotocol.io/docs/concepts/architecture
- MCP Concepts: Prompts：https://modelcontextprotocol.io/docs/concepts/prompts

写作日期：2026-03-12

---

### 12. 工业 / 现实世界锚点

#### 12.1 编码代理工作台

一个最典型的现实锚点，就是你现在正在构建的这种“知识库 + 代理式工作流”环境。

在这类系统里：

- `Agent` 负责围绕目标持续执行，例如读仓库、规划下一步、调用 shell、修改文档
- `MCP` 或等价协议负责把文件系统、终端、浏览器、文档索引等能力接成统一能力面
- `skill` 负责把“如何执行某一类工作”的方法压缩成可复用操作知识包
- 产品编排层负责权限控制、历史上下文、工作区边界和长期连续性

这也是为什么现实里最重要的问题不是“模型够不够聪明”，而是：

- 能力面是否标准化
- 权限边界是否清晰
- skill 是否真的能约束行为
- agent loop 是否能持续稳定工作

#### 12.2 企业内部运维 / 业务助手

另一个高价值场景，是企业内部的 ticket、知识库、文档审批、部署操作或排障助手。

在这类系统里，当前更推荐的做法不是把所有能力塞给一个“全能代理”，而是：

- 把 CMDB、工单、日志、发布系统拆成独立能力面
- 通过协议化 server 或清晰 tool contract 接入
- 对高风险动作加审批或人工确认
- 用 skill 固化 SOP、升级路径、禁止动作和异常处理方式

原因很简单：现实系统最怕的不是“不会做事”，而是“做错事还不可追踪”。

### 13. 当前推荐实践、过时路径与替代

#### 13.1 当前更推荐的实践

当前更稳的方向，是把代理系统拆成几层可治理的资产：

- 协议化或结构化描述的能力接入面，而不是靠提示词暗示模型“你大概能做什么”
- 最小权限原则，而不是默认给代理高权限工作区
- versioned skill 资产，而不是把所有操作知识堆进一个超长 system prompt
- 审计、确认和可回放的执行链路，而不是让代理静默完成高风险操作

#### 13.2 已经过时或明显不推荐的路径

下面这些路径不是“绝对不能用”，但在复杂系统里通常已经不够稳：

- 一个巨大的单体 system prompt，同时描述所有能力、所有边界、所有场景
- 每个产品各自发明一套 ad-hoc tool JSON，没有统一能力描述或复用边界
- 把“能访问很多工具”误当成“已经有了 Agent”
- 让长期在线代理默认持有高权限、弱审计、弱隔离的执行面

它们的问题在于：

- 迁移性差
- 审计难
- 技能知识和工具能力纠缠
- 随着能力增长迅速失控

替代路径不是“再写一个更长的 prompt”，而是把协议、能力、skill、运行时和产品编排明确分层。

### 14. 迁移入口与继续深挖

以后你再看任何新的 agent 平台、AI coding 产品或企业代理框架，都可以先问四个问题：

1. 它有没有真实的目标驱动执行闭环，还是只是多工具聊天？
2. 它的能力面是怎么接进来的，协议边界是否清楚？
3. 它如何沉淀 skill / workflow / SOP，而不是只靠模型临场发挥？
4. 它的长期运行依赖什么产品编排与权限控制？

当前仍值得继续深挖的问题：

- OpenClaw 这类长期在线代理平台，与更轻量的 coding agent 工作台在系统边界上到底如何区分？
- MCP 在生产环境下的权限模型、审批模型和多租户隔离应如何设计？
