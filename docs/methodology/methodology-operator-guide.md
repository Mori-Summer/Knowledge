---
doc_id: methodology-operator-guide
title: 方法论文档使用说明：怎么读、怎么用、怎么验收
concept: methodology_operator_guide
topic: methodology
created_at: '2026-03-19T18:10:00+08:00'
updated_at: '2026-03-19T18:10:00+08:00'
source_basis:
  - methodology_repository_practice
  - concept_document_generation_workflow_review_2026_03_19
time_context: operating_guide_baseline_2026_03_19
applicability: methodology_navigation_and_execution
prompt_version: not_applicable
template_version: guide_v1
quality_status: maintained_asset
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/concept-document-template.md
  - docs/methodology/concept-document-quality-gate.md
  - docs/methodology/fixed-concept-generation-prompt.md
open_questions:
  - 后续是否需要把这份说明进一步压成面向自动化脚本的 machine-readable 规则文件？
---

# 方法论文档使用说明：怎么读、怎么用、怎么验收

## 1. 这份说明解决什么问题

`docs/methodology/` 里的文件已经不止一份。  
如果没有一张明确的使用地图，第一次读的人或后续协作的模型很容易出现两个问题：

- 知道原则，但不知道先看哪份、什么时候用哪份
- 知道模板，但不知道什么时候算真正达标

这份说明的作用不是重复方法论内容，而是回答四个执行层问题：

- 每份文件各自负责什么
- 新建文档时按什么顺序用
- 升级旧文档时按什么顺序用
- 最后靠什么验收

## 2. 当前方法论栈的角色分工

### 2.1 `learning-new-things-playbook.md`

它负责回答：

- 学习一个新对象时，怎样从陌生走到可理解、可操作、可迁移
- 什么叫真正学会
- 学习过程中该产出什么、验证什么

它主要解决的是**学习路径和内部模型构建**。

### 2.2 `cognitive-modeling-playbook.md`

它负责回答：

- 分析和建模时，哪些认知规范必须遵守
- 怎样定义目标、边界、变量、反馈回路、约束和杠杆点
- 怎样区分事实、推断、假设和价值判断

它主要解决的是**分析纪律和因果建模质量**。

### 2.3 `concept-document-template.md`

它负责回答：

- 一篇合格概念文档至少要长成什么结构
- 必须包含哪些信息位点
- frontmatter 最小集合是什么

它主要解决的是**输出形状和标准栏目**。

### 2.4 `concept-document-quality-gate.md`

它负责回答：

- 什么情况直接不合格
- 什么情况只能算可保留草稿
- 什么情况才配标 `upgraded_v1`

它主要解决的是**验收门禁和评分规则**。

### 2.5 `fixed-concept-generation-prompt.md`

它负责回答：

- 以后如何用最短命令触发新建或升级
- 触发时要把哪些约束显式写进去

它主要解决的是**执行入口和稳定触发方式**。

## 3. 推荐阅读顺序

如果你第一次进入这套方法论，推荐顺序是：

1. 先读 [learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)，建立“学习目标不是解释术语，而是形成内部模型”的基本立场。
2. 再读 [cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)，把因果、边界、约束、时间尺度这些硬约束装进分析过程。
3. 再读 [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)，明确最终文档最少必须长成什么样。
4. 然后读 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)，知道最后靠什么验收，而不是靠感觉。
5. 最后用 [fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md) 作为实际触发入口。

## 4. 新建概念文档的标准工作流

### 4.1 第一步：先定义学习目标

优先从 `learning-new-things-playbook` 拿到这些信息：

- 你为什么要学这个概念
- 你希望达到哪一层掌握
- 你后续想用它分析什么问题
- 你当前最不理解的点是什么

如果这一步不清楚，后面很容易写成百科解释。

### 4.2 第二步：先建模，再写文档

优先从 `cognitive-modeling-playbook` 强制回答：

- 这个概念试图解决什么问题
- 边界在哪里
- 关键变量、约束和因果链是什么
- 哪些是事实，哪些是推断
- 哪些内容受时间敏感约束

如果这一步没做，文档最容易退化成漂亮但不可调用的说明文。

### 4.3 第三步：按模板组织输出

写文档时，使用 `concept-document-template` 保证：

- 章节齐全
- frontmatter 齐全
- 输出目标是“可复用模型”，不是“概念解释”

### 4.4 第四步：过质量门禁

正式落盘前，用 `concept-document-quality-gate` 做一次自查或审查。  
没有过门禁，就不要标 `upgraded_v1`。

### 4.5 第五步：落盘并更新索引

正式文档写入后，必须同步：

- 放进合适的 `docs/{topic}/`
- 文件名使用 `kebab-case`
- 更新 `docs/index.md`

## 5. 升级旧文档的标准工作流

升级旧文档时，不要直接整篇重写，推荐顺序是：

1. 先用 `concept-document-quality-gate` 找 hard fail 和低分项。
2. 再用 `concept-document-template` 对照缺失栏目。
3. 必要时回到两份 playbook，补因果链、边界、验证入口和迁移模型。
4. 保留原文档仍然有价值的内容，只补其结构短板和当前实践短板。
5. 更新 `updated_at`、`source_basis`、`time_context` 和 `docs/index.md`。

## 6. 主题、命名与 frontmatter 的最小约定

### 6.1 topic 目录选择

优先复用已有主题目录。  
只有在现有目录明显不合适时，才新增新的 `docs/{topic}/`。

选择原则是：

- topic 应该表达稳定主题域，而不是一次性任务名
- 同一类概念尽量归到同一主题下，避免碎片化

### 6.2 文件名

文件名使用 `kebab-case`，应尽量表达对象本体，而不是写成动作句。

### 6.3 frontmatter

frontmatter 的最小字段由模板规定。  
实际填写时再遵守三条附加约束：

- `doc_id` 建议稳定对应 `topic + slug`
- `concept` 建议用稳定、可复用的概念标识，而不是长句标题
- `time_context` 和 `source_basis` 要能反映这篇文档的时效性和证据基础

## 7. 时间敏感与来源纪律

这套方法论里最容易被低估的一层，是时效性和来源纪律。

对于“当前推荐实践”“最新状态”“现行制度”“当前工业做法”这类内容，默认遵守：

- 明确写核对日期
- 优先用一手来源或官方来源
- 正文里区分事实、推断和结论
- 如果找不到足够强的一手依据，就降低结论强度，不强行写成确定判断

如果文档本身是历史概念或经典理论，可以更多使用经典文本；  
如果文档涉及现行市场、平台、法规、标准、库行为或当前工程实践，就必须提高来源门槛。

## 8. 一篇概念文档什么时候算完成

一篇正式概念文档只有同时满足下面几点，才算真正完成：

- 文档已写入正确目录
- frontmatter 字段齐全且语义合理
- 标准章节齐全
- 工业 / 现实世界锚点具体且真实
- 当前实践和过时路径写清“为什么旧、局限在哪、现在更推荐什么”
- 自测题和迁移入口存在
- `concept-document-quality-gate` 通过
- `docs/index.md` 已更新

如果少了其中任意一条，通常都不应视为正式完成。

## 9. 最常见的误用方式

最常见的误用不是“不会写”，而是：

- 只看模板，不看两份 playbook，结果写成结构完整但机制空心的文章
- 只看 playbook，不看模板，结果写成分析笔记而不是知识库文档
- 有案例，但案例只是泛泛行业印象，不是真实锚点
- 有“当前实践”，但没有日期和来源纪律
- 文档写完了，却忘了更新 `docs/index.md`

## 10. 后续维护建议

今后如果继续扩展方法论，优先保持下面这条分层：

- playbook 负责思想与方法
- template 负责输出合同
- quality gate 负责验收
- prompt 负责触发
- operator guide 负责编排

不要把这五层重新混回一篇超长总文档里。  
一旦混回去，可读性会升高，但可操作性会下降。

## 11. 未解问题

- 这套方法论是否需要再拆出面向“技术概念”“制度概念”“数学概念”的三个专用 prompt？
- `concept-document-quality-gate` 未来是否应该进一步程序化，变成可自动检查的规则集？

## 12. 参考资料

- [学习新事物的方法手册：从陌生到可理解、可操作、可迁移](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)
- [认知规范与问题建模手册](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
- [统一概念文档模板](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
- [统一概念文档质量门禁](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)
- [固定概念文档生成 Prompt](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
