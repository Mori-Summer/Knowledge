---
doc_id: methodology-concept-document-template
title: 统一概念文档模板
concept: unified_concept_document_template
topic: methodology
created_at: '2026-03-16T10:05:00+08:00'
updated_at: '2026-03-19T18:10:00+08:00'
source_basis:
  - architecture_rules
  - personal_knowledge_base_practice
  - methodology_workflow_review_2026_03_19
time_context: template_baseline_2026_03_19
applicability: future_concept_doc_generation_and_manual_upgrade
prompt_version: template_alignment_v1
template_version: concept_doc_v1
quality_status: maintained_asset
related_docs:
  - docs/methodology/learning-new-things-playbook.md
  - docs/methodology/cognitive-modeling-playbook.md
  - docs/methodology/methodology-operator-guide.md
  - docs/methodology/concept-document-quality-gate.md
open_questions:
  - 未来是否需要给制度/政策类概念再补一份更强的一手来源模板？
---

# 统一概念文档模板

这份模板不是为了让文档“长得一样”，而是为了让每篇概念文档都稳定具备可学习、可建模、可迁移、可复用的最小结构。

## 1. 适用对象

这份模板适用于：

- 技术概念
- 数学对象
- 经济金融概念
- 系统机制
- 工程方法

它不要求所有文档同样长，但要求所有文档都能回答同一组核心问题。

## 2. 每篇概念文档必须回答的问题

一篇合格文档至少应能回答：

1. 这东西试图解决什么问题
2. 它的边界在哪里，不是什么
3. 它由哪些关键结构构成
4. 它是如何运转的，核心因果链是什么
5. 它在什么条件下有效，什么时候会失效
6. 现实里它被怎样使用，工业上为什么关心它
7. 旧做法为什么不够好，当前替代路径是什么
8. 如何验证自己是否真的理解了
9. 如何把这个模型迁移到相邻问题上
10. 现在还有哪些未解问题值得继续深挖

## 3. 推荐正文骨架

对大多数概念文档，建议按下面顺序组织：

```markdown
# 标题

## 1. 这份文档要帮你学会什么
## 2. 一句话结论 / 问题定义
## 3. 对象边界与相邻概念
## 4. 核心结构
## 5. 核心机制 / 主链路 / 因果链
## 6. 关键 tradeoff 与失败模式
## 7. 应用场景
## 8. 工业 / 现实世界锚点
## 9. 当前推荐实践、过时路径与替代
## 10. 自测题 / 验证入口
## 11. 迁移与关联模型
## 12. 未解问题与继续深挖
## 13. 参考资料
```

不是每篇文档都必须逐字逐句照抄这个目录，但这些信息位点不能缺。

## 4. 特殊规则与分型约束

### 4.1 数学推导型文档

如果文档本体是数学推导或公式证明，仍然不能只停留在推导本身。至少要额外补：

- 这套推导在现实里解决什么问题
- 哪些工程场景真正会用它
- 当前更推荐怎样使用它，而不是误把它当万能工具
- 如何通过样例、自测题或反例检查自己是否真的理解
- 它能迁移到哪些相邻方法、近似方法或实现设计中

### 4.2 时间敏感型文档

如果正文涉及：

- 当前推荐实践
- 现行制度 / 现行标准
- 最新产品行为 / 当前工业实现
- 仍可能变化的软件库、平台规则、监管状态

则至少额外满足：

- 正文显式写出核对日期
- `time_context` 能看出时间基线
- `source_basis` 和参考资料能反映一手或官方来源基础
- 正文尽量区分“事实”“推断”“当前建议”

### 4.3 制度 / 政策 / 监管 / 市场实践型文档

如果概念本体涉及规则、制度、市场设计、监管实践或产业政策，则不能只靠抽象原理。至少还要补：

- 真实制度对象或真实市场对象
- 当前约束是如何通过规则落地的
- 旧路径为什么不够用，当前替代是怎样补这个缺口的
- 这套制度设计在现实里最常见的失败模式是什么

### 4.4 历史 / 经典理论型文档

如果概念本体主要来自经典理论或历史文本，则至少要区分：

- 原始语境里的问题定义
- 后来常见的误读或泛化
- 当前仍可迁移的结构是什么
- 哪些结论只能放在原始语境下理解

## 5. 元数据最小集合

frontmatter 至少保留这些字段：

- `doc_id`
- `title`
- `concept`
- `topic`
- `created_at`
- `updated_at`
- `source_basis`
- `time_context`
- `applicability`
- `prompt_version`
- `template_version`
- `quality_status`
- `related_docs`
- `open_questions`

### 5.1 字段语义与填写约定

为了便于后续索引、升级和自动化检查，推荐再遵守下面这些约定：

- `doc_id`：建议稳定对应 `topic + slug`
- `concept`：建议写成稳定、可复用的概念标识，而不是整句标题
- `created_at` / `updated_at`：使用带时区的 ISO 8601 时间
- `source_basis`：优先写来源类别或来源簇，不写空泛词
- `time_context`：要能看出这篇文档是 evergreen、historical，还是 checked-on-date 的当前判断
- `related_docs`：使用仓库内相对路径
- `open_questions`：至少保留一个仍值得继续深挖的问题

### 5.2 仓库集成约束

一篇正式概念文档不仅要内容合格，还要满足：

- 直接写入合适的 `docs/{topic}/`
- 文件名使用 `kebab-case`
- 新增正式文档后同步更新 `docs/index.md`

## 6. 升级判定标准

一篇文档可以标记为 `upgraded_v1`，至少要满足：

- frontmatter 合法且字段齐全
- 正文能看出清晰的对象边界与主链路
- 至少有一个应用场景
- 至少有一个工业或现实世界锚点
- 存在验证入口：自测题、预测题、纠错题或等价机制
- 存在迁移入口：能引向相邻概念或新问题
- 存在未解问题，不把文档伪装成“已经彻底完成”
- 如果正文包含时间敏感内容，显式给出核对日期
- 如果正文包含“当前实践”结论，参考资料能支撑关键判断
- 工业锚点具体到真实组织、系统、产品、市场、法规或标准，而不是泛泛行业印象

更完整的验收方式，见 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)。

## 7. 不能做的事

- 不能把模板变成空话目录
- 不能用泛泛案例代替真实工业语境
- 不能提“旧方法”却不解释为什么旧、局限在哪、当前怎么替代
- 不能只追求语言流畅，忽略模型可调用性
- 不能为了整齐删掉高价值的专业细节
- 不能写时间敏感结论却不给出时间基线
- 不能生成正式文档却不更新 `docs/index.md`
