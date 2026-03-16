# Knowledge

一个用来沉淀“可学习、可建模、可迁移”知识文档的 Markdown 仓库。

这个项目的目标，不是收集一堆术语解释，也不是做百科式笔记，而是用统一的方法把一个概念写成一篇以后还能反复调用的文档。  
理想状态是：你读完一篇文档后，不只是“知道这个词”，而是能拿它去分析问题、定位边界、建立模型、迁移到相邻主题。

## 这类文档解决什么问题

很多知识库会在两个方向上失效：

- 只有定义，没有机制，读完还是不会分析
- 只有碎片化笔记，没有统一结构，过段时间就无法复用

这个仓库希望避免这两种失败模式。  
这里的文档默认都应该回答：

1. 这个概念试图解决什么问题
2. 它的边界在哪里，不是什么
3. 它由哪些关键结构构成
4. 它是如何运转的，核心因果链是什么
5. 它什么时候有效，什么时候失效
6. 现实里它怎样被使用，工业上为什么关心它
7. 旧做法为什么不够好，现在更推荐什么
8. 怎样验证自己是否真的理解了
9. 怎样把这个模型迁移到相邻问题
10. 还有哪些未解问题值得继续深挖

## 适用范围

这个仓库不是只写技术概念。  
它适用于：

- 计算机系统
- 编程语言
- AI 系统
- 数学对象
- 图像处理方法
- 经济金融概念
- 其他可以被“问题-结构-机制-边界-应用”方式建模的主题

## 仓库结构

```text
docs/
  methodology/                方法论文档
  ai-systems/                 AI 系统相关概念
  computer-systems/           计算机系统相关概念
  programming-languages/      编程语言相关概念
  image-processing/           图像处理相关概念
  _reports/                   规范化/升级报告
  index.md                    文档导航
```

关键文件：

- [docs/methodology/learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md)：学习新概念的方法论
- [docs/methodology/cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)：建模和分析问题的方法论
- [docs/methodology/concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)：统一概念文档模板
- [docs/methodology/fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)：固定生成指令
- [docs/index.md](/Users/maxwell/Knowledge/docs/index.md)：当前文档导航
- [CONTRIBUTING.md](/Users/maxwell/Knowledge/CONTRIBUTING.md)：继续新增或升级知识文档时的执行规范

## 文档质量标准

一篇合格的概念文档，不应该只是“写得通顺”。  
它至少要满足下面这些要求：

- 能看出清晰的问题定义和对象边界
- 有核心结构和主链路，而不是概念堆砌
- 有真实的应用场景和工业/现实世界锚点
- 对时间敏感内容给出日期语境，并尽量使用一手资料
- 如果提到旧技术、旧做法或被淘汰路径，必须解释为什么旧、替代方案是什么
- 有自测题、验证入口或纠错入口
- 有迁移入口，能把模型引向相邻问题
- 不把文档写成泛泛而谈的百科条目

## 如何生成一篇新文档

最推荐的用法，是把这个仓库交给一个支持读写文件的 AI agent，然后直接让它按方法论文档生成，并把结果写回 `docs/{topic}/`。

### 最短可用指令

直接使用这句：

```text
按 docs/methodology/concept-document-template.md 和两份 playbook，为概念 {concept} 生成一篇新的知识文档，并直接放入合适的 docs/{topic}/ 目录。不要写成泛泛解释，必须包含：问题定义、对象边界、核心结构、机制/因果链、关键 tradeoff 与失败模式、应用场景、工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题、参考资料，并补齐统一 frontmatter。
```

例如：

```text
按 docs/methodology/concept-document-template.md 和两份 playbook，为概念 memory order 生成一篇新的知识文档，并直接放入合适的 docs/{topic}/ 目录。不要写成泛泛解释，必须包含：问题定义、对象边界、核心结构、机制/因果链、关键 tradeoff 与失败模式、应用场景、工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题、参考资料，并补齐统一 frontmatter。
```

### 推荐完整指令

如果你希望生成结果更稳，建议把上下文也一起给 AI：

```text
为概念 {concept} 新建一篇知识库文档。

要求：
1. 严格遵循：
   - docs/methodology/learning-new-things-playbook.md
   - docs/methodology/cognitive-modeling-playbook.md
   - docs/methodology/concept-document-template.md
2. 输出目标不是术语解释，而是建立可复用的内部模型。
3. 文档必须直接写入合适的 docs/{topic}/ 目录，文件名使用 kebab-case，并补齐统一 frontmatter。
4. 文档必须至少包含：
   - 这份文档要帮我学会什么
   - 一句话结论 / 问题定义
   - 对象边界与相邻概念
   - 核心结构
   - 核心机制 / 主链路 / 因果链
   - 关键 tradeoff 与失败模式
   - 应用场景
   - 工业 / 现实世界锚点
   - 当前推荐实践、过时路径与替代
   - 自测题 / 验证入口
   - 迁移与关联模型
   - 未解问题与继续深挖
   - 参考资料
5. 不能泛泛而谈。工业场景必须真实；如果提到旧技术或旧做法，必须解释为什么旧、局限在哪、现在更推荐什么。
6. 文档完成后，顺手更新 docs/index.md。

补充上下文：
- 所属主题：{topic}
- 我是在什么场景下遇到它的：{context_where_i_met_it}
- 我现在最不理解的点：{what_confuses_me}
- 我希望后续能拿它分析什么问题：{future_use_case}
```

### 最少需要告诉 AI 的信息

最少给出这 4 项，也可以开工：

```text
概念：{concept}
主题：{topic}
遇到语境：{context}
当前困惑：{confusion}
```

例如：

```text
概念：memory order
主题：computer-systems
遇到语境：看 C++ 原子操作和并发资料时遇到
当前困惑：不知道 relaxed/acquire/release/seq_cst 到底差在哪，为什么会影响正确性
```

## 如何升级已有文档

如果不是新建，而是升级某篇旧文档，直接使用：

```text
按 docs/methodology/concept-document-template.md 升级现有文档 {path}。

要求：
- 保留原有高价值内容，不要整篇重写
- 补齐当前缺失的标准章节和 frontmatter 字段
- 把文档提升到可保留、可复用、可迁移的状态
- 重点补：工业/现实世界锚点、当前推荐实践、过时路径与替代、自测题、迁移入口、未解问题
```

## 生成文档时的几个硬规则

- 新文档必须放进合适的 `docs/{topic}/` 目录
- 文件名使用 `kebab-case`
- 文档应尽量直接落盘，而不是只输出在聊天窗口里
- 时间敏感结论要显式带日期
- 规范性、标准状态、库行为、语言特性等，优先引用官方或一手资料
- 工业案例不能是空泛比喻，必须说明为什么现实里关心它
- 如果内容涉及已过时路径，必须说明它为何过时、当前替代方案是什么

## 这个仓库不追求什么

- 不追求“文档都长得一样”
- 不追求“解释得像百科”
- 不追求“为了整齐而删掉高价值细节”
- 不追求“看起来懂了”，而追求“以后还能拿出来分析问题”

## 快速入口

- 看方法：先读 [learning-new-things-playbook.md](/Users/maxwell/Knowledge/docs/methodology/learning-new-things-playbook.md) 和 [cognitive-modeling-playbook.md](/Users/maxwell/Knowledge/docs/methodology/cognitive-modeling-playbook.md)
- 看模板：读 [concept-document-template.md](/Users/maxwell/Knowledge/docs/methodology/concept-document-template.md)
- 直接生成：参考 [fixed-concept-generation-prompt.md](/Users/maxwell/Knowledge/docs/methodology/fixed-concept-generation-prompt.md)
- 看已有例子：读 [memory-order.md](/Users/maxwell/Knowledge/docs/computer-systems/memory-order.md)

## 一句话总结

这是一个“把概念写成可调用模型”的 Markdown 知识库。  
如果你想使用它，最重要的不是多写几篇，而是始终按统一方法把每一篇写到“以后还能拿来分析问题”的质量线以上。
