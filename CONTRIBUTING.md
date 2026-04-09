# Contributing

这份文档说明如何继续维护这个知识库，使新增内容、旧文档升级和文档审计都能对齐当前仓库结构。

这里的“贡献”主要不是写产品代码，而是继续产出、升级和审查高质量的 Markdown 知识文档。  
目标不是把仓库写大，而是把每篇文档写到以后还能稳定复用。

## 当前接受的贡献类型

这个仓库当前主要接受 4 类贡献：

1. 新建概念文档
2. 升级已有概念文档
3. 审计 / 审查现有文档
4. 优化方法论文档与仓库规范

对应地，常见工作流也是 4 条：

- `新建`
- `升级`
- `审计`
- `方法论维护`

## 开始前先读什么

在开始任何贡献之前，默认先读下面这一份主规范：

- [docs/methodology/document-generation-methodology.md](/Users/maxwell/Knowledge/docs/methodology/document-generation-methodology.md)

只有在主规范不足以支撑具体任务时，再进入对应参考件：

- [docs/methodology/fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
- [docs/methodology/concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
- [docs/methodology/concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md)
- [docs/methodology/learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)
- [docs/methodology/cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
- [docs/methodology/methodology-operator-guide.md](/Users/maxwell/Knowledge/docs/methodology/methodology-operator-guide.md)

默认不再要求每次都把整套方法论文档完整读一遍。

## 目录与主题选择

所有正式知识文档都放在 `docs/` 下，按主题分目录。

当前主题包括：

- `docs/ai-systems/`
- `docs/computer-systems/`
- `docs/economics/`
- `docs/image-processing/`
- `docs/programming-languages/`
- `docs/systems/`
- `docs/methodology/`
- `docs/_reports/`

主题选择原则：

- 优先复用已有主题目录
- topic 应表达稳定主题域，而不是一次性任务名
- 概念跨多层时，优先放到最能承载其主问题的目录
- 只有在现有目录明显不合适时，才新增新 topic

## 文件命名与 frontmatter 约定

### 文件名

- 使用 `kebab-case`
- 名称尽量直接对应概念本体
- 不要用口语化、一次性或临时性的文件名

### frontmatter

frontmatter 最少应包含：

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

附加约定：

- `concept` 统一使用稳定的 `snake_case`
- `doc_id` 尽量稳定对应 `topic + slug`
- `related_docs` 使用仓库内相对路径
- `updated_at` 要反映本次真实修改时间
- `time_context` 与正文时间敏感结论必须一致

## 工作流 1：新建概念文档

### 第一步：明确最小输入

至少明确下面四项：

- 概念名
- 所属主题
- 你是在什么语境下遇到它的
- 你当前最不理解的点

### 第二步：按主规范执行

新建时的推荐顺序：

1. 先按 `document-generation-methodology` 判断任务类型与 `standard` / `deep`
2. 再建立边界、结构、主链路和 tradeoff
3. 再按统一合同组织输出
4. 再按质量门禁验收
5. 写入合适目录并更新 `docs/index.md`

### 推荐指令

```text
按 docs/methodology/document-generation-methodology.md，为概念 {concept} 新建一篇知识库文档。先判断 standard 或 deep，建立可复用内部模型，直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，补齐统一 frontmatter，并在完成后更新 docs/index.md。
```

## 工作流 2：升级已有文档

升级默认遵守“保留高价值内容，补齐短板”，而不是整篇推倒重写。

推荐顺序：

1. 先用主规范或质量门禁找 Hard Fail 和低分项
2. 再补缺失栏目、主链路、tradeoff、验证入口和迁移模型
3. 必要时回到参考 playbook 补边界、机制和约束
4. 更新 `updated_at`、`source_basis`、`time_context`
5. 若文档标题变更或新增正式文档，更新 `docs/index.md`

升级时优先补这些缺口：

- 问题定义不清
- 对象边界不清
- 没有核心机制 / 因果链
- 工业锚点泛泛而谈
- 当前实践没有日期或来源纪律
- 没有自测题 / 验证入口
- 没有迁移入口
- 没有未解问题

### 推荐指令

```text
按 docs/methodology/document-generation-methodology.md 升级现有文档 {path}。
```

## 工作流 3：审计 / 审查现有文档

审计默认以 [concept-document-quality-gate.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-quality-gate.md) 为基线，而不是凭阅读感觉给评价。

推荐顺序：

1. 先查仓库集成与 frontmatter
2. 再查结构完整性
3. 再查模型质量
4. 最后看工业锚点、当前实践、来源和时间敏感性

输出建议：

- findings first
- 优先报 Hard Fail
- 给出文件定位
- 区分“必改项”和“可改进项”

如果只是想做一轮系统审计，可以直接要求按质量门禁做 review。

## 工作流 4：方法论文档维护

`docs/methodology/` 下的文档不是普通概念文档，它们共同组成仓库的规范层。

维护这部分内容时，优先遵守下面这条新分层：

- 主规范：`document-generation-methodology.md`
- 参考件：playbook、template、quality gate、prompt、operator guide

判断原则：

- 任何每次新建、升级、审查都必须遵守的稳定规则，优先写进主规范
- 只有在需要保留更深背景、完整模板、完整评分细则或快捷触发词时，才写进参考件
- 默认先收束入口，而不是继续增加并列主入口

## 正式概念文档的最小信息位点

正式概念文档默认至少包含：

1. 这份文档要帮你学会什么
2. 一句话结论 / 问题定义
3. 对象边界与相邻概念
4. 核心结构
5. 核心机制 / 主链路 / 因果链
6. 关键 tradeoff 与失败模式
7. 应用场景
8. 工业 / 现实世界锚点
9. 当前推荐实践、过时路径与替代
10. 自测题 / 验证入口
11. 迁移与关联模型
12. 未解问题与继续深挖
13. 参考资料

不是每篇都必须逐字照抄这些标题，但这些信息位点不能缺。

## 内容质量规则

### 以模型为中心，不以术语为中心

优先回答：

- 它在解决什么问题
- 为什么需要它
- 它由哪些关键结构构成
- 它如何运转
- 何时成立，何时失效

### 必须有边界

如果概念容易和相邻概念混淆，必须明确写：

- 它不等于什么
- 它和哪些概念经常被混用
- 它只解决哪一层问题

### 必须有真实锚点

工业 / 现实世界锚点不能写成：

- “工业里很常见”
- “很多系统会用到”

更好的写法是：

- 真实组织、系统、产品、市场、标准或制度
- 工程上为什么关心它
- 理解错后在现实里会出什么问题

### 必须写清旧路径与替代

如果提到旧做法、旧技术或已过时路径，必须解释：

- 为什么旧
- 局限在哪
- 当前更推荐什么

### 必须有验证入口和迁移入口

每篇文档都应有：

- 自测题 / 预测题 / 纠错题等验证入口
- 指向相邻问题的迁移入口

否则很容易停留在“读的时候像懂了”。

## 来源与时间敏感规则

这些内容默认视为时间敏感：

- 当前推荐实践
- 标准状态 / 规范状态
- 库行为 / 工具链行为
- 现行制度 / 当前市场实践
- 当前产品行为 / 平台规则

对此默认要求：

- 正文显式写核对日期
- `time_context` 与正文一致
- 优先使用一手或官方来源
- 区分事实、推断和结论

如果不确定，先核对，不要凭印象写。

## 什么时候要更新 `docs/index.md`

以下情况应更新 [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)：

- 新增正式文档
- 文档标题发生变化
- 新增新的 topic 目录

如果只是正文修订、frontmatter 规范化、参考资料补强，而标题和导航结构未变，则通常不需要改 `docs/index.md`。

## 不要做的事

- 不要新增重复文档覆盖已有主题
- 不要只写定义和例子就结束
- 不要用空泛案例代替真实锚点
- 不要省略时间语境
- 不要引用过时路径却不解释替代方案
- 不要为了整齐删掉关键专业细节
- 不要把文档写成资料阅读流水账

## 最后检查清单

在认为文档完成之前，至少检查一遍：

- 目录位置是否合理
- 文件名是否符合 `kebab-case`
- `concept` 是否已规范成 `snake_case`
- frontmatter 是否齐全且语义一致
- 是否真正覆盖“问题-边界-结构-机制-失效-应用-替代”
- 是否包含真实锚点
- 是否包含验证入口和迁移入口
- 是否包含未解问题
- 如果有时间敏感内容，是否写清日期与来源
- 如需导航更新，是否同步改了 `docs/index.md`

## 推荐起手方式

最稳的维护习惯是：

1. 先判断任务属于 `新建`、`升级`、`审计` 还是 `方法论维护`
2. 再回到 `docs/methodology/` 选择对应工具文档
3. 执行完成后，用质量门禁做最后一轮自查

这样比自由发挥更稳，也更符合这个仓库现在的结构。
