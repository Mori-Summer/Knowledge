---
doc_id: methodology-concept-document-example-catalog
title: 概念文档样例目录：合格、不合格与质量门禁证据
concept: concept_document_example_catalog
topic: methodology
depth_mode: standard
created_at: '2026-05-25T15:31:43+08:00'
updated_at: '2026-06-17T13:52:34+08:00'
source_basis:
  - _bmad-output/project-context.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/implementation-artifacts/1-1-concept-document-generation-contract.md
  - _bmad-output/implementation-artifacts/1-2-intake-and-intent-classification.md
  - _bmad-output/implementation-artifacts/1-3-concept-document-example-catalog.md
  - docs/index.md
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
  - _bmad-output/implementation-artifacts/stabilization-status-2026-06-15.md
time_context: stabilization_concept_example_catalog_review_2026_06_17
applicability: concept_document_example_catalog_for_reviewer_calibration
prompt_version: not_applicable
template_version: governance_asset_v1
quality_status: reviewed
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/concept-document-contract.md
  - docs/methodology/intake-and-intent-classification.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
  - docs/methodology/governance-asset-boundary-policy.md
  - docs/governance/agent-behavior-constraints.md
  - docs/governance/governance-asset-navigation-policy.md
  - docs/governance/lifecycle-states.md
  - docs/governance/prompt-template-quality-version-governance.md
  - docs/governance/batch-readiness-checklist.md
  - docs/governance/frontmatter-schema.md
  - docs/governance/index-synchronization-rules.md
  - docs/governance/candidate-promotion-checklist.md
  - docs/governance/document-decision-policy.md
  - docs/governance/rework-loop-examples.md
  - docs/governance/related-docs-taxonomy.md
  - docs/governance/link-maintenance-policy.md
  - docs/governance/existing-doc-reuse-procedure.md
  - docs/governance/network-boundary-and-decay-prevention.md
  - docs/templates/review-record-template.md
  - docs/templates/completion-report-template.md
open_questions:
  - review record、document decision policy 和 rework-loop examples 已建立后，是否需要把本文的 reviewer action calibration 全量映射到正式决策 label？
  - frontmatter schema 已建立后，是否需要把本目录中的示例 frontmatter 表达方式进一步收敛到 schema baseline？
  - Epic 6 batch governance runbook 建立后，是否需要为样例目录补充批量示例复核记录？
---

# 概念文档样例目录：合格、不合格与质量门禁证据

## 资产角色、权威与适用范围

本文是 `Knowledge` 项目的概念文档样例目录，用合格与不合格样例校准 reviewer 对证据的判断。它的职责是把 `docs/methodology/concept-document-contract.md` 的必需信息位点、`docs/methodology/concept-document-template.md` 的结构期待，以及 `docs/methodology/concept-document-quality-gate.md` 的 Hard Fail 和六项评分信号具体化。

本文补充以下资产：

- `docs/methodology/document-generation-methodology.md`
- `docs/methodology/concept-document-contract.md`
- `docs/methodology/intake-and-intent-classification.md`
- `docs/methodology/concept-document-template.md`
- `docs/methodology/concept-document-quality-gate.md`
- `docs/methodology/fixed-concept-generation-prompt.md`
- Foundation governance assets
- Epic 2 frontmatter、path、index 与 promotion governance assets
- Epic 3 review/decision/rework/completion governance assets
- Epic 5 related-docs、link maintenance、reuse 和 network-boundary governance assets

本文不替代主方法论、概念文档生成合同、intake 流程、模板、质量门禁、固定 Prompt、生命周期政策、版本治理政策、导航政策、source-discipline policy、frontmatter schema、index synchronization rules、review/completion templates、document decision policy、rework-loop examples、related-docs/link governance，或未来 Epic 6 的 batch governance runbook、batch review record 和 batch completion report。

当前 `quality_status: reviewed` 表示本文已完成 Epic 6 前置稳定化审查：样例目录角色、passing/failing example semantics、reviewer action calibration、source/currentness labels、owner/index entry、相关治理依赖、链接/索引边界和非软件边界已检查。未解决项保留在 `open_questions` 和维护触发点中；本文不声明 `validated`，因为 Epic 6 batch governance runbook、batch review record 和 batch completion report 仍未落地。

本文的 owner entry point 是 `docs/index.md` 的 `methodology` 分组。Navigation treatment 是 `listed_in_docs_index`，index treatment 是在 `docs/index.md` 的 `## methodology` 下列出 `docs/methodology/concept-document-example-catalog.md`。

本文内嵌的 passing/failing examples 只是本方法论资产中的 illustrative calibration examples。它们不是 standalone formal concept documents，不创建独立 `doc_id`，不写入 `docs/{topic}/`，也不作为单独条目加入 `docs/index.md`。

## 样例使用规则

这些样例用于校准 evidence judgment，不创建第二套质量门禁。Reviewer 仍必须以 `docs/methodology/concept-document-quality-gate.md` 为 Hard Fail、六项评分、状态声明和审查输出的权威来源。

样例的 passing/failing 结论只属于本文的示例场景。实际候选文档是否可接受，必须单独按主方法论、合同、模板、质量门禁、frontmatter、路径、索引、来源和时间语境检查。

每个样例使用同一组 compact evidence map：

| 字段 | 含义 |
| --- | --- |
| information point | 候选文档必须让 reviewer 找到的合同信息位点或治理证据 |
| evidence location | 样例中能看到证据的位置 |
| gate impact | 对 Hard Fail 或六项评分的影响 |
| likely reviewer action | reviewer 在真实候选稿中倾向选择的动作 |

Story 1.3 层级允许的 reviewer action calibration 只有三类：

- `targeted_revision`：候选结构有保留价值，缺口具体且可补。
- `regeneration_candidate`：候选基础不可信，局部修补会留下错误结构或状态风险。
- `acceptance_candidate`：无 Hard Fail，必要证据可定位，剩余风险不阻塞，状态声明保守。

这些动作是样例校准启发，不是正式 document decision policy。正式 accept、revise、regenerate、reject、hold 规则、review record 字段、rework-loop examples 和 completion-report templates 由 `docs/governance/document-decision-policy.md`、`docs/templates/review-record-template.md`、`docs/governance/rework-loop-examples.md` 和 `docs/templates/completion-report-template.md` 负责；本文不创建实际 review record 或 completion report。

## 合格样例

本节样例选择一个低当前性、项目内可解释的纯概念路径主题，避免把 Story 1.3 扩展成外部事实研究。示例主题是“边界清晰度”，它用于说明 reviewer 应该如何看到 frontmatter、命名/分类问题、边界、判别框架、真实锚点、验证、迁移、自测和 open questions。

### 示例 A：passing candidate excerpt

以下摘录不是 standalone formal concept document，而是一个合格候选稿应当呈现的证据形态。

```markdown
---
doc_id: methodology-boundary-clarity
title: 边界清晰度：让概念能被稳定判别，而不是只被解释
concept: boundary_clarity
topic: methodology
depth_mode: standard
created_at: '2026-05-25T15:31:43+08:00'
updated_at: '2026-05-25T15:31:43+08:00'
source_basis:
  - docs/methodology/concept-document-contract.md
  - docs/methodology/concept-document-quality-gate.md
time_context: illustrative_project_rule_context_2026_05_25
applicability: concept_document_review_and_revision
prompt_version: not_applicable
template_version: concept_doc_v2
quality_status: draft
related_docs:
  - docs/methodology/concept-document-contract.md
  - docs/methodology/concept-document-quality-gate.md
open_questions:
  - 后续是否需要在示例集中补一个外部来源纪律案例？
---

# 边界清晰度：让概念能被稳定判别，而不是只被解释

## 1. 这份文档要帮你学会什么

本文帮助 reviewer 判断一篇概念文档是否真的说清了“什么属于这个概念、什么不属于、和相邻概念怎么分”。如果读完只能复述定义，不能判断边界，这篇文档仍然不合格。

## 2. 命名 / 分类问题

“边界清晰度”命名的是一种审查能力：读者能把候选对象分成属于本文对象、不属于本文对象、相邻但不同的对象三类。它解决的问题不是措辞是否漂亮，而是候选文档能不能支持稳定分类。

## 3. 概念边界与相邻概念

属于边界清晰度的证据：

- 正向边界：列出什么对象、场景或条件算作本文对象。
- 反向边界：列出什么看似相关但不属于本文对象。
- 相邻区分：说明最容易混淆的概念之间用什么判据分开。

不属于边界清晰度的证据：

- 只给定义，但没有例子、反例或判别条件。
- 只列相邻名词，没有说明差异。
- 把适用场景、实现机制和历史背景混成同一个概念。

相邻概念区别：

| 相邻概念 | 判别差异 |
| --- | --- |
| 定义完整性 | 关注句子是否说明“是什么”；边界清晰度关注能否判断“什么算 / 不算”。 |
| 适用范围 | 关注结论在哪些场景成立；边界清晰度还要求区分相邻对象。 |
| 来源纪律 | 关注事实、推断、时间语境是否可追溯；边界清晰度关注对象分类是否稳定。 |

## 4. 判别框架

Reviewer 可以用三问判断边界是否足够清楚：

1. 如果给出一个新对象，我能判断它是否属于本文概念吗？
2. 如果给出一个相邻概念，我能说出它们的关键差异吗？
3. 如果候选文档删除某个例子或反例，边界判断是否会塌掉？

如果三问中任一项只能靠 reviewer 自己补常识，候选稿应进入 `targeted_revision`。

## 5. 真实世界锚点

项目内锚点：`docs/methodology/concept-document-quality-gate.md` 把“对象边界不清，读者无法判断什么属于本文对象、什么不属于”列为边界清晰度 Hard Fail。这个锚点不是外部行业事实，而是 `Knowledge` 项目的 project rule。

## 6. 来源与时间语境

- `project rule`：Hard Fail 和六项评分来自 `docs/methodology/concept-document-quality-gate.md`，本文按 2026-05-25 的本地项目状态使用。
- `verified fact`：本文只声称本仓库存在上述方法论资产；检查日期为 2026-05-25。
- `inference`：三问判别框架是对合同和质量门禁的操作化整理，不是新增门禁。
- `open question`：后续是否要补外部来源案例。

## 7. 自测题 / 验证入口

1. 给一篇只定义“mutex 是互斥锁”的文档，指出它缺少哪类边界证据。
2. 给一篇同时讨论 mutex、semaphore、condition variable 的文档，判断它是否说明了三者的判别差异。
3. 删除候选稿中的反例后，如果 reader 无法判断不属于对象的场景，应如何修复？

## 8. 迁移与关联模型

这个判别框架可迁移到 source discipline、frontmatter schema 和 index governance：每个治理规则都需要说明属于、不属于、相邻但不同。不能迁移的是具体 Hard Fail 名称；那些仍由对应权威资产定义。

## 9. 未解问题与继续深挖

- frontmatter schema 已建立后，边界证据是否需要独立字段表达？
- 来源纪律政策是否需要在未来示例集中展示更严格的 project rule 与 verified fact 区分？
```

### Passing evidence map

| information point | evidence location | gate impact | likely reviewer action |
| --- | --- | --- | --- |
| Complete frontmatter expectations | 示例 frontmatter 包含 `doc_id`、`title`、`concept`、`topic`、`depth_mode`、时间字段、YAML arrays、版本/status 字段；没有未授权全局字段 | 避免必需 Frontmatter Hard Fail；支持“元数据 / 仓库纪律”高分 | `acceptance_candidate` |
| Source/time/status consistency | `time_context`、`source_basis`、`quality_status: draft` 与正文的 illustrative/project-rule 说明一致 | 避免时间语境与一致性 Hard Fail | `acceptance_candidate` |
| Naming/classification problem | “命名 / 分类问题”说明它解决稳定分类而非漂亮定义 | 支持“问题定义与边界” | `acceptance_candidate` |
| Object boundary and non-object scope | “概念边界与相邻概念”列出属于、不属于和相邻区别 | 避免边界清晰度 Hard Fail | `acceptance_candidate` |
| Discrimination frame | “三问判断”给出可调用判别框架 | 支持“结构与因果 / 判别框架” | `acceptance_candidate` |
| Real-world anchor | 锚点明确指向本地质量门禁资产，而不是泛泛“文档审查常见” | 支持“真实世界锚点 / 当前实践”；本例为 project-internal anchor | `acceptance_candidate` |
| Validation and self-test | 三道题检查边界判断、相邻概念和修复方向 | 避免验证 Hard Fail | `acceptance_candidate` |
| Transfer entry | 迁移到 source discipline、schema 和 index governance，同时说明不可迁移部分 | 避免迁移 Hard Fail | `acceptance_candidate` |
| Open questions | 明确 frontmatter schema 和来源纪律示例后续复核点 | 避免把未决治理问题伪装成已解决 | `acceptance_candidate` |
| Source/currentness labels | 区分 `project rule`、`verified fact`、`inference`、`open question` | 满足来源与时间语境最低标签要求 | `acceptance_candidate` |

这个 passing example 不因为文字完整就直接声明 `reviewed` 或 `validated`。它保持 `quality_status: draft`，因为它是示例化校准资产中的片段，不是 standalone formal docs asset；来源纪律政策、frontmatter schema 和正式决策政策即使已经存在，也不自动把 illustrative excerpt 晋升为 reviewed 或 validated。

## 不合格样例

本节样例故意写得结构看似完整，展示为什么 reviewer 不能只看标题和 frontmatter。它的问题是内容没有产生可复用判断力，并且混入未经验证的 current-practice claim。

### 示例 B：failing candidate excerpt

以下摘录是反例，不应作为正式概念文档接受。

```markdown
---
doc_id: computer-systems-memory-order
title: Memory Order：现代系统里的并发顺序
concept: memory_order
topic: computer-systems
depth_mode: deep
created_at: '2026-05-25T15:31:43+08:00'
updated_at: '2026-05-25T15:31:43+08:00'
source_basis:
  - general_programming_knowledge
time_context: current_best_practice
applicability: all_concurrent_programming
prompt_version: concept_generation_prompt_v4
template_version: concept_doc_v2
quality_status: validated
related_docs:
  - docs/computer-systems/atomicity.md
open_questions: []
---

# Memory Order：现代系统里的并发顺序

## 1. 这份文档要帮你学会什么

Memory order 是多线程编程中非常重要的概念。理解它可以写出高性能并发程序。

## 2. 一句话结论 / 问题定义

Memory order 决定程序中内存操作的顺序。现代系统一般推荐使用 acquire/release，复杂场景可以使用 relaxed。

## 3. 对象边界与相邻概念

Memory order 和 atomic、mutex、compiler optimization 都有关。它们都影响并发程序的正确性。

## 4. 核心机制 / 主链路

CPU 和编译器会重排序，所以 memory order 用来控制重排序。不同 memory order 有不同强度，越强越安全，越弱越快。

## 5. 工业 / 现实世界锚点

很多高性能系统都使用 memory order 来提高性能。现代 C++ 项目通常会根据性能选择合适的 memory order。

## 6. 自测题 / 验证入口

1. 什么是 memory order？
2. memory order 为什么重要？

## 7. 迁移与关联模型

可以迁移到所有多线程系统和所有语言的并发模型。

## 8. 未解问题

无。
```

### Failure evidence map

| information point | evidence location | gate impact | likely reviewer action |
| --- | --- | --- | --- |
| Status overclaim | frontmatter 写 `quality_status: validated`，但没有事实核查、用途检查或迁移检查证据 | 命中必需 Frontmatter Hard Fail：状态暗示已验证但无证据 | `regeneration_candidate` |
| Source/time context broken | `source_basis: general_programming_knowledge`，`time_context: current_best_practice`，正文声称“现代系统一般推荐”但没有来源或核对日期 | 命中来源依据与真实世界锚点 Hard Fail、时间语境 Hard Fail | `regeneration_candidate` |
| Boundary missing | “和 atomic、mutex、compiler optimization 都有关”没有说明什么算 memory order、什么不算、相邻差异是什么 | 命中边界清晰度 Hard Fail；“问题定义与边界”低分 | `targeted_revision` only if other foundations are reliable |
| Fake mechanism | “越强越安全，越弱越快”把 tradeoff 简化成口号，没有对象、同步边、可见性或适用条件 | 命中文档类型结构 Hard Fail；“结构与因果 / 判别框架”低分 | `regeneration_candidate` |
| Generic anchor | “很多高性能系统”无法定位真实组织、系统、标准、论文或公开实践 | 命中真实世界锚点 Hard Fail | `targeted_revision` only if source/time claims are removed and replaced |
| Weak validation | 自测题只问术语回忆，不能检查解释、边界、诊断、预测或迁移 | 命中验证与迁移 Hard Fail | `targeted_revision` |
| Unsafe transfer | “所有多线程系统和所有语言”过度泛化，未说明语言内存模型差异或迁移限制 | 迁移入口低分，并触发边界泛化风险 | `regeneration_candidate` |
| Empty open questions | 明显存在来源、当前性、语言边界和锚点缺口，却写 `open_questions: []` | 状态/来源/未决风险不一致 | `regeneration_candidate` |

### Repair guidance

如果这篇候选稿只是某个较好旧文档的压缩摘要，且 reviewer 能在原文中定位稳定 `doc_id`、正确 path、真实来源、语言边界和机制链，可选择 `targeted_revision`：删除 `validated` 状态，补 source/time labels，重写边界、机制、真实锚点、自测和迁移。

如果这就是候选稿的全部内容，应标为 `regeneration_candidate`。原因是它同时存在状态过度声明、来源/时间语境断裂、边界缺失、伪机制、泛化迁移和空 open questions。局部补丁很可能保留错误主链路。

本文不重新定义 Hard Fail。上述失败只是在具体样例中引用 `docs/methodology/concept-document-quality-gate.md` 已有类别。

## Reviewer Action Calibration

Reviewer 可以用下面启发判断候选稿该补写、重生成还是接受。它们只服务 Story 1.3 样例校准，不替代 `docs/governance/document-decision-policy.md` 的最终决策政策。

| action | 适用信号 | 不适用信号 |
| --- | --- | --- |
| `targeted_revision` | 高价值结构已经存在；缺失证据位置具体；identity、path、source status 没有根本错误；补写不会改变核心判断链 | 候选稿的对象边界、来源、状态和主链路都不可信 |
| `regeneration_candidate` | path/type 错误；结构是模板填空；多个核心信息点命中 Hard Fail；source/time claims 广泛 unsupported；内容太泛，保留会延续错误判断 | 候选稿只有少数可定位缺口，且修复不会改变主结构 |
| `acceptance_candidate` | 无 Hard Fail；frontmatter/path/index discipline sound；必需信息点可定位；未决风险 nonblocking；状态声明保守 | 存在阻塞来源缺口、断链、状态过度声明或不可定位锚点 |

实际完成审查时，reviewer 还必须按质量门禁输出 Hard Fail、六项评分或等价治理检查、必改项、可改进项、最终结论和允许状态。正式 review record、decision policy、rework loop 和 completion report 分别由 `docs/templates/review-record-template.md`、`docs/governance/document-decision-policy.md`、`docs/governance/rework-loop-examples.md` 和 `docs/templates/completion-report-template.md` 定义。

## 来源与时间语境标签

样例中出现事实、项目规则、用户语境、推断和未决问题时，至少使用以下标签。更完整的 source discipline 与 real-world anchor policy 由 `docs/methodology/source-discipline-and-real-world-anchor-policy.md` 承担；本文只规定样例校准的最低标签。

| label | 含义 | 示例用法 |
| --- | --- | --- |
| `verified fact` | 已从本地正式资产或合适来源核对的事实，并有检查日期或时间语境 | “本仓库存在 `docs/methodology/concept-document-quality-gate.md`；检查日期 2026-05-25。” |
| `project rule` | 来自本项目正式方法论、治理资产、story 或 Maxwell 指令的规则 | “Hard Fail 以质量门禁为准。” |
| `user context` | 来自 Maxwell 当前请求、story、intake 或对话语境的使用背景 | “本文样例只展示 reviewer calibration 标签，不替代专门来源纪律政策。” |
| `inference` | Agent 对项目规则的操作化整理、归纳或推导，不是新增规则 | “三问判别框架是对合同与门禁的整理。” |
| `open question` | 当前 story 不解决、尚待后续核查或归属后续治理工作的事项 | “frontmatter schema 后是否调整示例 frontmatter 表达？” |

项目内部规则可以引用本地 docs、story 和 governance assets。外部 current-practice claim 必须有 source/time-context discipline；若当前 story 不准备核查外部事实，示例应保持 evergreen、project-internal、illustrative，或把该 claim 标为 `open question`。

## 完成与验证证据

验证本文时，至少检查以下事项：

| 验证项 | 通过证据 |
| --- | --- |
| role/authority/scope | 本文声明自己是 reviewer calibration support asset，且不替代主规范、合同、模板、质量门禁、fixed prompt、source policy、frontmatter/index governance、review/completion governance 或未来 Epic 6 batch assets |
| frontmatter | 必填字段存在；`source_basis`、`related_docs`、`open_questions` 是 YAML arrays；没有未授权全局字段 |
| navigation | owner entry point 是 `docs/index.md` 的 `methodology` 分组；navigation treatment 是 `listed_in_docs_index` |
| example semantics | 内嵌样例不是 standalone formal concept documents，不进入 `docs/{topic}/`，不单独加入索引 |
| passing example | frontmatter、命名问题、边界、判别框架、真实锚点、来源标签、验证、自测、迁移、open questions 可定位 |
| failing example | 结构看似完整但失败原因映射到 Hard Fail 或六项低分证据，并给出 targeted revision 与 regeneration 区分 |
| reviewer actions | 只定义 `targeted_revision`、`regeneration_candidate`、`acceptance_candidate` 的校准启发，不替代 formal document decision policy |
| source/currentness labels | 定义 `verified fact`、`project rule`、`user context`、`inference`、`open question` |
| version governance | 不 bump prompt/template/quality 版本；本文没有改变规则语义 |
| non-software boundary | 未创建代码、package manifest、source tree、tests、CLI/API/UI/database/deployment/CI、runtime automation、lint/scoring tools、standalone concept documents 或 batch workflow |

参考资料：

- [统一概念文档规范：新建、升级、审查与仓库集成](./document-generation-methodology.md)
- [概念文档生成合同：输入、输出、边界与必需信息位点](./concept-document-contract.md)
- [输入摄入与任务意图判定：任务类型、文档路径、深度与缺失输入处理](./intake-and-intent-classification.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](./source-discipline-and-real-world-anchor-policy.md)
- [统一概念文档模板](./concept-document-template.md)
- [统一概念文档质量门禁](./concept-document-quality-gate.md)
- [固定概念文档生成 Prompt](./fixed-concept-generation-prompt.md)
- [方法论资产边界：主规范、模板、质量门禁、playbook 与固定 Prompt 的职责分工](./governance-asset-boundary-policy.md)
- [Agent 行为约束：文档治理任务必须先判边界、再执行、可验证](../governance/agent-behavior-constraints.md)
- [治理资产导航、索引与入口归属政策](../governance/governance-asset-navigation-policy.md)
- [文档生命周期状态：草稿、审查、验证、废弃与归档转换规则](../governance/lifecycle-states.md)
- [Prompt、模板与质量规则版本治理：规则演进、字段语义与渐进迁移](../governance/prompt-template-quality-version-governance.md)
- [批量治理 Readiness Checklist：范围、冲突、停止条件与恢复策略](../governance/batch-readiness-checklist.md)
- [Frontmatter schema 与 doc_id 身份规则：正式 docs 资产的元数据基线](../governance/frontmatter-schema.md)
- [docs/index.md 同步与导航治理规则](../governance/index-synchronization-rules.md)
- [候选文档晋升 Checklist：从临时输出到正式 docs 资产](../governance/candidate-promotion-checklist.md)
- [文档决策政策：accept、revise、regenerate、defer、reject 与 lifecycle 结果](../governance/document-decision-policy.md)
- [返工闭环示例：失败类型、修复路径、重生成边界与复审入口](../governance/rework-loop-examples.md)
- [related_docs 关系分类与维护规则](../governance/related-docs-taxonomy.md)
- [链接维护政策：变更影响、入链/出链与失效预防](../governance/link-maintenance-policy.md)
- [Existing Doc Reuse Procedure](../governance/existing-doc-reuse-procedure.md)
- [Network Boundary and Decay Prevention](../governance/network-boundary-and-decay-prevention.md)
- [审查记录模板：任务分类、Hard Fail、评分证据、未验证项与决策记录](../templates/review-record-template.md)
- [完成汇报模板：质量状态、入库决策证据、验证证据、未解决风险与非软件边界](../templates/completion-report-template.md)
