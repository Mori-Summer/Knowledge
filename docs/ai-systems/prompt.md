---
doc_id: ai-systems-prompt
title: Prompt：从一次性提问到系统指令栈的输入控制面
concept: prompt
topic: ai-systems
depth_mode: deep
created_at: '2026-04-01T20:50:06+08:00'
updated_at: '2026-04-08T19:25:35+08:00'
source_basis:
  - prompt_boundary_and_layering_synthesis_2026_04_08
  - repository_agent_prompt_stack_observation_2026_04_08
  - mcp_prompts_concepts_docs_checked_2026_04_08
  - pure_concept_doc_split_review_2026_04_08
time_context: foundational_concept_with_protocol_examples_checked_2026_04_08
applicability: prompt_boundary_discrimination_instruction_layering_and_upgrade_judgment
prompt_version: concept_generation_prompt_v3
template_version: unified_spec_v1
quality_status: upgraded_v1
related_docs:
  - docs/ai-systems/agent-mcp-skill-openclaw-concepts.md
  - docs/ai-systems/agent.md
  - docs/ai-systems/skill.md
  - docs/ai-systems/mcp.md
  - docs/methodology/document-generation-methodology.md
open_questions:
  - prompt 作为“当前输入”与“可复用提示资产”这两层含义，未来是否值得在术语上彻底拆开？
  - prompt stack 的优先级、压缩和冲突解释，应该更多交给人工规范，还是交给运行时自动治理？
  - 在 agent 系统里，prompt、skill 与 workflow 的升级边界，是否能进一步固化成可检查规则？
---

# Prompt：从一次性提问到系统指令栈的输入控制面

## 1. 这份文档要帮你学会什么

这篇文档的重点，不是继续把 prompt 写成“优化技巧大全”，而是先把这个概念本体讲清。

读完后，你应该至少能做到：

- 分清 prompt 到底在指“当前轮次输入”，还是在指“可复用提示资产”
- 区分 prompt、system prompt、developer instruction、user message、retrieved context、tool result、skill、memory 和 workflow
- 判断某个对象是在 prompt 里生效，还是只是以后可能被装进 prompt 的东西
- 知道什么时候继续改 prompt 就够了，什么时候其实该升级到 skill、workflow、runtime 或 policy 层

## 2. 一句话结论 / 概念结论

**Prompt 首先不是“写给模型的一大段话”，而是模型在当前轮次实际读到的输入控制面；在工程系统里，它经常由多层消息、上下文和反馈叠成一个 prompt stack。**

如果只记一句最稳的话：

**讨论 prompt 时，先问“这是不是当前轮次已经送进模型的输入”，再问“其中哪些成分是指令，哪些只是上下文、状态或反馈”。**

## 3. 这个概念试图解决什么命名或分类问题

“Prompt” 这个词在日常语境里最容易一词多指。

有人用它指：

- 用户写给模型的一句话
- 一次调用里所有会被模型读到的内容
- 可复用的提示模板
- 提示工程本身

如果不先把这些层次拆开，后面几乎一定会出现错位：

- 明明是 system rule、skill 或 runtime 的问题，却怪到 prompt 头上
- 明明讨论的是当前输入，却突然混进 prompt library 或 MCP prompt asset
- 明明只是 user message 写得含糊，却把整个 prompt stack 都看成失败
- 明明需要结构化状态、工具协议或 workflow，却继续无止境加长 prompt

所以，prompt 这个概念首先解决的不是“怎么把字写漂亮”，而是下面这组分类问题：

- 这一轮真正进入模型的输入到底有哪些成分
- 这些成分里，哪些在发指令，哪些只是提供背景
- “prompt 资产”与“当前 prompt”是不是同一层对象
- 什么问题该在 prompt 层解决，什么问题不该继续压回 prompt

## 4. 概念边界与相邻概念

### 4.1 这篇文档直接处理什么

这篇文档直接处理的是：

- prompt 作为当前轮次输入控制面的本体
- prompt stack 里不同成分的边界
- prompt 与 prompt asset 的区分
- prompt 与 skill、memory、workflow、tool、policy 的交界

这篇文档不负责系统展开：

- skill 如何设计成可复用行为资产
- agent 如何形成完整执行闭环
- MCP 作为能力协议如何接入工具、资源与提示

这些内容分别交给：

- [Skill：把 SOP、约束与操作策略沉淀成可复用行为资产](./skill.md)
- [Agent：目标驱动执行闭环，不是会聊天的模型](./agent.md)
- [MCP：把工具、资源与提示接成标准能力面的协议层](./mcp.md)

### 4.2 它不等于什么

Prompt 不等于：

- **用户消息。**
  user message 通常只是 prompt 的一个来源，不是全部 prompt。

- **任何长文本。**
  长，不代表就是 prompt；很多长文本只是知识材料、日志或说明。

- **skill。**
  skill 是可复用工作方法资产；它常常通过 prompt 生效，但不等于 prompt 本体。

- **memory。**
  memory 是可保存、可检索、可跨轮存在的状态；只有当它被放进当前输入，才成为 prompt 的一部分。

- **tool 或工具协议。**
  tool 决定“能做什么动作”；prompt 决定“当前轮次怎样理解和组织任务”。

- **模型参数或微调。**
  prompt 改的是当前输入，不是长期权重分布。

### 4.3 最容易混淆的相邻对象

| 对象 | 它是什么 | 和 prompt 的关系 |
| --- | --- | --- |
| `system prompt` | 最上层的稳定规则或角色设定 | 通常是 prompt stack 的高优先级组成部分 |
| `developer instruction` | 产品、框架或代理施加的具体执行规则 | 常是当前 prompt 的一层，但比 user message 更稳定 |
| `user message` | 当前轮次的显式任务意图 | 是 prompt 来源之一，不代表 prompt 全体 |
| `retrieved context` | 检索出的背景资料、仓库上下文或外部信息 | 单独看不是 prompt；被注入当前轮次后才成为 prompt 成分 |
| `tool result` | 工具调用后的反馈、报错或数据回传 | 常进入下一轮 prompt，但它本身不等于任务指令 |
| `skill` | 可复用行为资产、步骤模板和判断规则 | 往往通过 prompt 注入当前执行，但本体不等于 prompt |
| `memory` | 可持久保存和检索的状态 | 只有读入当前轮次时，才进入 prompt |
| `workflow` | 更确定的执行图、步骤编排或程序逻辑 | prompt 常为其中某一步服务，但 workflow 不等于 prompt |
| `MCP prompt asset` | 通过协议暴露的可发现提示模板 | 它是 prompt 资产；只有实例化并送入本轮输入时，才成为当前 prompt |

### 4.4 最关键的边界句

**看到 “prompt” 这个词时，先追问它是在说“当前轮次已经送进模型的输入”，还是在说“以后可复用的提示模板或提示资产”；这两层不分开，几乎所有讨论都会漂。**

## 5. 什么算它，什么不算它

下面这张表是最实用的判别模板。

| 表达或对象 | 在本文里怎么处理 | 为什么 |
| --- | --- | --- |
| 当前调用里的 system + developer + user + context + output constraint | 这是当前 prompt stack | 它们已经共同进入模型当前轮次 |
| 单独一条 user message | 只是 prompt 的一部分 | 它不能代表全部输入控制面 |
| 一段还躺在向量库里的知识片段 | 还不是 prompt | 只有被取出并注入当前轮次后，才进入 prompt |
| 工具返回的一段 JSON 报错 | 单独看不是 prompt | 下一轮读入后，它才成为 prompt 的反馈成分 |
| 一个 `SKILL.md` 文件 | 不是 prompt 本体 | 它是行为资产，通常需要被激活并装进当前执行 |
| 一个 MCP prompt 模板 | 是 prompt asset，不是当前 prompt | 它描述“怎样生成或组织提示”，不是已经送入模型的本轮输入 |
| 一个 workflow 图 | 不是 prompt | 它描述执行结构，不是当前轮次文本输入本体 |
| 模型权重、微调检查点 | 不是 prompt | 它们改变长期行为分布，不属于本轮输入 |

最值得反复用的一条判断规则是：

**不看“它像不像提示词”，只看“它有没有在这一轮真正进入模型输入”。**

## 6. 代表性例子、反例与常见误读

### 6.1 代表性例子

最有代表性的例子有三个。

- **聊天场景里的当前对话输入。**
  system 规定角色，user 提任务，外加输出格式约束；这几层合起来，才更接近当前 prompt，而不只是用户那一句话。

- **编码代理里的消息栈。**
  系统规则、技能指导、仓库上下文、工具报错和当前任务一起进入模型；这正是 prompt 作为输入控制面的典型形态。

- **MCP 暴露的 prompt 模板被实例化之后。**
  模板本身是资产；当它带着参数展开，并被真正送入当前调用时，它才变成当前 prompt 的一部分。

### 6.2 反例

最值得记住的反例有下面这些。

- **工具本身不是 prompt。**
  `ripgrep`、`git` 或某个 API 只是能力面，不是输入控制面。

- **仓库里的技能文件不是 prompt。**
  它可以改变行为方式，但它首先是资产，不是这一轮天然就会被模型读到的输入。

- **数据库里的历史记录不是 prompt。**
  只有被检索、选择并放进当前轮次时，它才进入 prompt。

- **“把 prompt 写得更长”不是通用解法。**
  如果问题来自权限治理、状态管理、工具协议或步骤确定性，继续加 prompt 常常只是掩盖结构缺口。

### 6.3 常见误读

最常见的误读有下面几类。

- **“prompt 就是一大段话。”**
  错。prompt 的关键不是长短，而是它是否构成当前轮次的输入控制面。

- **“prompt 就等于用户提问。”**
  错。用户提问常常只占其中一层。

- **“只要进了上下文窗口，就全都算同一种 prompt。”**
  不稳。进入输入不代表职责相同，指令、背景、反馈和约束仍然要分层看。

- **“skill 就是更长的 prompt。”**
  错。skill 更像可复用工作方法；prompt 更像当前执行载体。

- **“prompt 失败，就说明模型不行。”**
  错。很多失败先是层次混乱、优先级不清或上下文污染。

- **“prompt engineering 可以替代 runtime、workflow 或 policy 设计。”**
  错。prompt 能塑形本轮推理，但不能替代所有系统层职责。

## 7. 它会进入哪些更大的模型或判断框架

prompt 这个概念本体，最重要的下游去向有四个。

### 7.1 指令分层与优先级判断

一旦你把 prompt 看成输入控制面，就会自然进入 instruction hierarchy：  
哪些是硬约束，哪些是软建议，哪些只是背景材料。

### 7.2 Agent 执行闭环

在 agent 系统里，prompt 常常不是静态文本，而是会随着任务推进不断重组。  
这会把你带入“当前状态如何进入下一轮判断”的执行闭环视角。

### 7.3 从 prompt 升级到 skill / workflow 的判断

当你发现问题已经不只是“这轮怎么说”，而是：

- 同类任务反复出现
- 需要稳定步骤和完成标准
- 需要更强的确定性执行

这时就不该继续把一切都压回 prompt，而应分别转向：

- [skill.md](./skill.md)：承接可复用工作方法
- workflow 或程序逻辑：承接高确定性步骤编排

### 7.4 Prompt 资产化与协议化

当 prompt 被做成模板库、选择器、参数化资产或 MCP prompt 时，你看到的已经不只是当前输入，而是更上层的资产治理问题。  
这也是为什么“prompt asset”与“current prompt”必须分开理解。

## 8. 自测题 / 验证入口

如果你真的理解了这篇文档，至少应该能回答：

1. 为什么 prompt 不等于 user message？
2. 为什么一个还没被注入当前轮次的知识片段，不应直接算作 prompt？
3. MCP prompt 模板和当前 prompt，到底差在哪一层？
4. 什么情况下应该继续改 prompt，什么情况下应该升级到 skill 或 workflow？
5. 为什么“写得更长”不等于“prompt 更对”？

一个最实用的自测动作是：

把下面这些对象分别判定为“当前 prompt”“prompt 的组成候选”“prompt asset”或“非 prompt”：

- system rule
- user message
- 检索结果
- tool error JSON
- `SKILL.md`
- MCP prompt 模板
- workflow 图

如果你能稳定给出分类，并解释“为什么”，这篇文档的核心目标就达到了。

## 9. 迁移与关联模型

理解了 `Prompt` 之后，最值得迁移出去的不是一句优化口诀，而是下面这组判断：

- 这里讨论的是当前输入，还是可复用提示资产？
- 这里进入模型的是指令、背景、反馈，还是输出约束？
- 这个问题还属于 prompt 层，还是已经应该升级到 skill、workflow、runtime、policy 或 memory 层？

最值得连着看的文档是：

- [Skill：把 SOP、约束与操作策略沉淀成可复用行为资产](./skill.md)
- [Agent：目标驱动执行闭环，不是会聊天的模型](./agent.md)
- [MCP：把工具、资源与提示接成标准能力面的协议层](./mcp.md)
- [AI 代理栈分层：Agent、MCP、Skill、Prompt 与 OpenClaw 的概念边界](./agent-mcp-skill-openclaw-concepts.md)

最值得保留的迁移句是：

**Prompt 负责塑形当前轮次的理解与输出；一旦问题转向长期复用、确定性执行、权限治理或持久状态，就应该及时切到更合适的上层对象。**

## 10. 未解问题与继续深挖

- prompt 作为“当前输入”与“可复用提示资产”这两层含义，未来是否值得在术语上彻底拆开？
- prompt stack 的优先级、压缩和冲突解释，应该更多交给人工规范，还是交给运行时自动治理？
- 在 agent 系统里，prompt、skill 与 workflow 的升级边界，是否能进一步固化成可检查规则？

## 11. 参考资料

- [AI 代理栈分层：Agent、MCP、Skill、Prompt 与 OpenClaw 的概念边界](./agent-mcp-skill-openclaw-concepts.md)
- [Skill：把 SOP、约束与操作策略沉淀成可复用行为资产](./skill.md)
- [Agent：目标驱动执行闭环，不是会聊天的模型](./agent.md)
- [MCP：把工具、资源与提示接成标准能力面的协议层](./mcp.md)
- [统一概念文档规范：新建、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [MCP Concepts: Prompts](https://modelcontextprotocol.io/docs/concepts/prompts)
