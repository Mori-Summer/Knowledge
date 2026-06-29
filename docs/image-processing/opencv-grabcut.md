---
doc_id: image-processing-opencv-grabcut
title: OpenCV GrabCut：从 GMM、MRF 能量函数到最小割的交互式分割模型
concept: opencv_grabcut
topic: image-processing
depth_mode: deep
created_at: '2026-06-26T00:00:00+08:00'
updated_at: '2026-06-26T00:00:00+08:00'
source_basis:
  - opencv_4_x_imgproc_grabcut_docs_checked_2026_06_26
  - opencv_4_x_python_grabcut_tutorial_checked_2026_06_26
  - rother_kolmogorov_blake_grabcut_siggraph_2004_checked_2026_06_26
  - boykov_jolly_interactive_graph_cuts_2001_as_cited_by_grabcut
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
time_context: opencv_4_x_official_docs_checked_2026_06_26_and_siggraph_2004_algorithm_baseline
applicability: grabcut_mathematical_model_energy_minimization_gmm_mrf_graph_cut_and_opencv_grabcut_reasoning
prompt_version: concept_generation_prompt_v4
template_version: concept_doc_v1
quality_status: draft
related_docs:
  - docs/methodology/document-generation-methodology.md
  - docs/methodology/source-discipline-and-real-world-anchor-policy.md
  - docs/image-processing/guided-filter-derivation.md
open_questions:
  - 是否需要补一篇独立文档推导 s-t graph cut、max-flow/min-cut 与二值 MRF 能量之间的等价关系？
  - 是否需要用一个 1D 或 3x3 玩具图手算一次 t-link、n-link 和 cut cost？
  - OpenCV 实现中的 GMM 参数存储、协方差正则化和空成分处理是否需要单独读源码验证？
---

# OpenCV GrabCut：从 GMM、MRF 能量函数到最小割的交互式分割模型

## 1. 这份文档要帮你学会什么

本文重心不是 `cv.grabCut` 怎么调用，而是 GrabCut 背后的数学建模：

- 如何把“前景/背景抠图”写成一个二值标注问题。
- 为什么需要两个 GMM，而不是一个颜色阈值。
- 为什么能量函数分成数据项 `U` 和平滑项 `V`。
- 为什么这个能量可以转成 s-t graph cut，并用 min-cut 求全局最优标签。
- 为什么 GrabCut 要迭代更新 `GMM component -> GMM parameters -> segmentation`。
- 为什么它保证能量单调下降，但不保证找到全局最优的完整联合解。

OpenCV 在本文里只是一个现实锚点：它提供的 `mask`、`rect`、`bgdModel`、`fgdModel` 本质上分别对应 GrabCut 数学模型里的约束、标签状态和颜色模型参数。

## 2. 一句话结论

**GrabCut 把交互式前景提取建模为一个带隐变量的二值 MRF 能量最小化问题：每个像素有前景/背景标签 `alpha_n`，颜色由前景或背景 GMM 解释，相邻像素通过对比敏感 Potts 项保持区域连贯；在固定 GMM 参数和混合成分时，标签子问题可由最小割精确求解，整体算法则通过交替最小化不断降低能量。**

核心公式是：

$$
E(\alpha, k, \theta, z)
=
U(\alpha, k, \theta, z)
+
V(\alpha, z)
$$

其中：

- `z` 是图像颜色观测。
- `alpha` 是每个像素的二值前景/背景标签。
- `k` 是每个像素属于哪个 GMM 成分的隐变量。
- `theta` 是前景/背景两个 GMM 的参数。
- `U` 是颜色似然代价。
- `V` 是空间连贯性和边界对齐代价。

GrabCut 的本质不是“OpenCV 的一个抠图函数”，而是：

```text
用户约束
  -> 给二值标注问题提供硬边界和初始化
  -> GMM 给每个像素提供颜色解释代价
  -> MRF 平滑项给相邻像素提供一致性约束
  -> graph cut 精确求当前参数下的最优二值标签
  -> 新标签反过来更新 GMM
```

## 3. 数学对象与符号

设图像有 `N` 个像素：

$$
z = (z_1, z_2, \dots, z_N)
$$

对彩色图像：

$$
z_n \in \mathbb{R}^3
$$

其中 `z_n` 是第 `n` 个像素的 RGB 或 BGR 颜色向量。OpenCV 内部使用 BGR 存储不改变数学模型；它仍然是三维颜色向量。

每个像素有一个硬分割标签：

$$
\alpha_n \in \{0,1\}
$$

约定：

$$
\alpha_n = 0 \Rightarrow \text{background}
$$

$$
\alpha_n = 1 \Rightarrow \text{foreground}
$$

所有像素标签组成：

$$
\alpha = (\alpha_1, \alpha_2, \dots, \alpha_N)
$$

GrabCut 还为每个像素引入一个 GMM 成分变量：

$$
k_n \in \{1,2,\dots,K\}
$$

所有成分分配组成：

$$
k = (k_1, k_2, \dots, k_N)
$$

原论文常取：

$$
K = 5
$$

也就是说，背景由 `K` 个 Gaussian 成分描述，前景也由 `K` 个 Gaussian 成分描述，总共有 `2K` 个 Gaussian 成分。

GMM 参数记为：

$$
\theta =
\{\pi(\alpha,k),\mu(\alpha,k),\Sigma(\alpha,k)
\mid \alpha \in \{0,1\}, k=1,\dots,K\}
$$

其中：

- `pi(alpha,k)` 是混合权重。
- `mu(alpha,k)` 是均值向量。
- `Sigma(alpha,k)` 是协方差矩阵。

## 4. 交互输入在数学上是什么

GrabCut 不是无监督分割。用户输入给模型提供约束。

传统 trimap 写成：

$$
T = \{T_B, T_U, T_F\}
$$

含义是：

- `T_B`：确定背景。
- `T_F`：确定前景。
- `T_U`：未知区域。

在完整 trimap 里：

$$
n \in T_B \Rightarrow \alpha_n = 0
$$

$$
n \in T_F \Rightarrow \alpha_n = 1
$$

未知区域：

$$
n \in T_U \Rightarrow \alpha_n \text{ 需要优化求解}
$$

GrabCut 的一个关键改造是允许 incomplete labelling：用户可以只给确定背景，不必一开始给确定前景。矩形初始化就是这种思想的工程化形式。

若用户画出矩形 `R`，可理解为：

$$
n \notin R \Rightarrow n \in T_B \Rightarrow \alpha_n = 0
$$

$$
n \in R \Rightarrow n \in T_U
$$

工程实现中，矩形内会被初始化成“可能前景”或待优化区域，但这不是数学上的确定前景。它只是给迭代一个起点。

OpenCV 四值 mask 和数学标签的关系可以这样理解：

| OpenCV mask | 数学语义 | 是否允许优化改变 |
| --- | --- | --- |
| `GC_BGD` | `alpha_n = 0` 的硬背景约束 | 通常不变 |
| `GC_FGD` | `alpha_n = 1` 的硬前景约束 | 通常不变 |
| `GC_PR_BGD` | 当前估计为背景 | 可变 |
| `GC_PR_FGD` | 当前估计为前景 | 可变 |

因此，OpenCV 的四类标签不是四分类分割。底层目标仍是二值前景/背景分割，只是额外保存“确定 / 可能”的约束强度。

## 5. 概率模型：从像素颜色到负对数似然

如果只看单个像素，GrabCut 想回答：

> 这个颜色 `z_n` 更像背景 GMM，还是更像前景 GMM？

对固定标签 `alpha_n` 和固定成分 `k_n`，像素颜色的概率密度是：

$$
p(z_n \mid \alpha_n, k_n, \theta)
=
\mathcal{N}
\left(
z_n;
\mu(\alpha_n,k_n),
\Sigma(\alpha_n,k_n)
\right)
$$

再乘上该成分的混合权重：

$$
p(z_n, k_n \mid \alpha_n, \theta)
=
\pi(\alpha_n,k_n)
\mathcal{N}
\left(
z_n;
\mu(\alpha_n,k_n),
\Sigma(\alpha_n,k_n)
\right)
$$

直接最大化概率不方便，所以取负对数，得到数据代价：

$$
D(\alpha_n,k_n,\theta,z_n)
=
-\log p(z_n \mid \alpha_n,k_n,\theta)
-\log \pi(\alpha_n,k_n)
$$

展开 Gaussian 的负对数密度，忽略与优化无关的常数项：

$$
D(\alpha_n,k_n,\theta,z_n)
=
-\log \pi(\alpha_n,k_n)
+
\frac{1}{2}\log \det \Sigma(\alpha_n,k_n)
+
\frac{1}{2}
\left[
z_n-\mu(\alpha_n,k_n)
\right]^T
\Sigma(\alpha_n,k_n)^{-1}
\left[
z_n-\mu(\alpha_n,k_n)
\right]
$$

这条公式拆开看：

| 项 | 含义 | 直觉 |
| --- | --- | --- |
| `-log pi` | 成分先验代价 | 很少出现的成分不应轻易解释大量像素 |
| `1/2 log det Sigma` | Gaussian 体积代价 | 协方差越大，模型越“宽松”，需要付复杂度代价 |
| Mahalanobis distance | 像素到成分中心的归一化距离 | 颜色越不像该成分，代价越大 |

于是所有像素的数据项是：

$$
U(\alpha,k,\theta,z)
=
\sum_{n=1}^{N}
D(\alpha_n,k_n,\theta,z_n)
$$

如果没有后面的平滑项，GrabCut 就退化成“每个像素独立选前景/背景 GMM 中代价更小的一边”。这样会产生噪点和破碎区域，因为它完全不关心相邻像素之间的关系。

## 6. 为什么是两个 GMM，而不是一个阈值

前景和背景的颜色分布通常都是多峰的。

例如：

- 一个人的前景可能包含皮肤、衣服、头发、阴影、高光。
- 背景可能包含墙面、地面、天空、桌面、文字。

单个阈值等价于在颜色空间里画一个很粗糙的边界；单个 Gaussian 也只能表达一个椭球形分布。GMM 则允许：

$$
p(z_n \mid \alpha,\theta)
=
\sum_{k=1}^{K}
\pi(\alpha,k)
\mathcal{N}(z_n;\mu(\alpha,k),\Sigma(\alpha,k))
$$

GrabCut 又进一步引入硬成分分配 `k_n`，使优化更容易：

$$
k_n
=
\arg\min_{k}
D(\alpha_n,k,\theta,z_n)
$$

这不是完整 EM 的软分配，而是 hard assignment。原论文说明软分配可以想象，但实践收益不足以抵消计算代价；GrabCut 采用硬成分变量来保持迭代流程简单。

## 7. 空间模型：对比敏感 Potts 平滑项

只用颜色似然会让分割碎裂。GrabCut 加入邻接像素之间的平滑项：

$$
V(\alpha,z)
=
\gamma
\sum_{(m,n)\in C}
\frac{1}{\operatorname{dist}(m,n)}
\left[\alpha_m \neq \alpha_n\right]
\exp
\left(
-\beta
\lVert z_m-z_n\rVert^2
\right)
$$

其中：

- `C` 是邻接像素对集合，通常可理解为 4 邻域或 8 邻域。
- `[alpha_m != alpha_n]` 是指示函数，条件成立为 `1`，否则为 `0`。
- `dist(m,n)` 是两个像素中心的空间距离；水平/垂直邻居为 `1`，对角邻居为 `sqrt(2)`。
- `gamma` 控制整体平滑强度。
- `beta` 控制颜色差对边界代价的影响。

这就是对比敏感 Potts model。

如果两个相邻像素颜色相近：

$$
\lVert z_m-z_n\rVert^2 \approx 0
$$

则：

$$
\exp(-\beta \lVert z_m-z_n\rVert^2) \approx 1
$$

把它们切成不同标签会付出较大代价。

如果两个相邻像素颜色差很大：

$$
\lVert z_m-z_n\rVert^2 \gg 0
$$

则：

$$
\exp(-\beta \lVert z_m-z_n\rVert^2) \approx 0
$$

在这里切开代价较低。于是分割边界倾向于穿过图像中的强颜色边缘。

`beta` 通常按图像里的平均邻接颜色差自适应设定：

$$
\beta
=
\left(
2
\left\langle
\lVert z_m-z_n\rVert^2
\right\rangle
\right)^{-1}
$$

其中 `<>` 表示对邻接像素对求平均。这让指数项能根据当前图像的整体颜色尺度调节敏感度。

一个重要边界：

$$
\beta = 0
$$

时，平滑项变成普通 Ising / Potts 平滑：

$$
V(\alpha)
=
\gamma
\sum_{(m,n)\in C}
\frac{1}{\operatorname{dist}(m,n)}
\left[\alpha_m \neq \alpha_n\right]
$$

它会在所有位置都惩罚标签变化，不关心图像边缘。GrabCut 使用 `beta > 0`，是为了让强边缘位置更容易成为分割边界。

## 8. 完整能量函数

综合数据项和平滑项：

$$
E(\alpha,k,\theta,z)
=
\sum_{n=1}^{N}
D(\alpha_n,k_n,\theta,z_n)
+
\gamma
\sum_{(m,n)\in C}
\frac{1}{\operatorname{dist}(m,n)}
\left[\alpha_m \neq \alpha_n\right]
\exp
\left(
-\beta
\lVert z_m-z_n\rVert^2
\right)
$$

这条式子就是 GrabCut 的核心。

它表达了两个偏好：

1. 像素标签要让颜色容易被对应的前景/背景 GMM 解释。
2. 相邻且颜色相近的像素应更倾向于同一标签。

也可以把它看成一个 MAP 问题：

$$
\hat{\alpha}
=
\arg\max_{\alpha}
p(\alpha \mid z, \theta)
$$

转成负对数后：

$$
\hat{\alpha}
=
\arg\min_{\alpha}
E(\alpha,k,\theta,z)
$$

更准确地说，GrabCut 在联合变量上做交替优化：

$$
(\hat{\alpha},\hat{k},\hat{\theta})
\approx
\arg\min_{\alpha,k,\theta}
E(\alpha,k,\theta,z)
$$

这里用 `approx`，因为整体联合问题不是一次 graph cut 就能全局解决。GrabCut 做的是 block coordinate descent：分块固定其他变量，优化当前变量。

## 9. 为什么可以用 graph cut

当 `k` 和 `theta` 固定时，能量只剩下关于二值标签 `alpha` 的函数：

$$
E(\alpha)
=
\sum_{n}
D_n(\alpha_n)
+
\sum_{(m,n)\in C}
V_{mn}
\left[\alpha_m \neq \alpha_n\right]
$$

其中：

$$
D_n(0)=D(0,k_n,\theta,z_n)
$$

$$
D_n(1)=D(1,k_n,\theta,z_n)
$$

以及：

$$
V_{mn}
=
\gamma
\frac{1}{\operatorname{dist}(m,n)}
\exp
\left(
-\beta
\lVert z_m-z_n\rVert^2
\right)
$$

这个能量是典型二值 MRF：

- unary term：每个像素自己取背景/前景的代价。
- pairwise term：相邻像素标签不同的代价。

因为：

$$
V_{mn} \ge 0
$$

且 pairwise 形式是 Potts：

$$
V_{mn}[\alpha_m \neq \alpha_n]
$$

它满足二值 graph cut 可优化的 submodular 条件。因此可以构造一个 s-t graph，使一次最小割的 cut cost 等于这套能量。

## 10. s-t graph 如何对应能量

构造图：

- 每个像素是一个节点。
- 增加两个终端：source `s` 和 sink `t`。
- 像素到终端的边叫 t-link。
- 像素之间的边叫 n-link。

常见映射如下：

| 图边 | 权重 | 对应能量 |
| --- | --- | --- |
| `s -> n` | 与背景代价相关 | 如果像素被切到背景侧，要付前景/背景中的某个 unary 代价，具体取决于约定 |
| `n -> t` | 与前景代价相关 | 同上 |
| `m <-> n` | `V_mn` | 如果两个像素被 cut 分到两侧，付平滑代价 |

不同教材对 source/foreground、sink/background 的方向约定可能相反，但本质不变：t-link 编码 unary term，n-link 编码 pairwise discontinuity term。

一次 cut 会把图分成两部分：

$$
S \cup T = \text{all nodes}, \quad s \in S,\quad t \in T
$$

由此得到标签：

$$
n \in S \Rightarrow \alpha_n = 1
$$

$$
n \in T \Rightarrow \alpha_n = 0
$$

或相反，取决于实现约定。

最小割求：

$$
\operatorname{cut}^{*}
=
\arg\min_{\operatorname{cut}}
\operatorname{cost}(\operatorname{cut})
$$

它等价于：

$$
\hat{\alpha}
=
\arg\min_{\alpha}
E(\alpha)
$$

这就是 GrabCut 能在固定颜色模型时得到当前二值标签全局最优解的原因。

## 11. 硬约束如何进入图模型

确定背景和确定前景不能被普通优化随便翻转。数学上可以把不允许的标签赋予无限大代价。

若像素 `n` 是确定背景：

$$
D_n(0)=0,\qquad D_n(1)=+\infty
$$

若像素 `n` 是确定前景：

$$
D_n(1)=0,\qquad D_n(0)=+\infty
$$

未知像素则使用当前固定的 GMM 成分分配和参数给出的数据代价：

$$
D_n(0)=D(0,k_n,\theta,z_n)
$$

$$
D_n(1)=D(1,k_n,\theta,z_n)
$$

这里的 `k_n` 来自上一轮成分分配步骤。下一轮迭代开始时，算法会在新的标签状态下重新选择 `k_n`。如果只做概念化解释，也可以把某个标签下的颜色代价理解为“由该标签对应 GMM 中最合适的成分解释的代价”，但 GrabCut 的交替最小化写法需要区分“当前固定的 `k_n`”和“下一轮重新估计的 `k_n`”。

工程实现不一定真的写入数学上的无穷大，通常使用足够大的容量或固定标签逻辑。数学含义是一样的：用户硬约束比颜色似然和平滑项优先级更高。

## 12. 交替最小化：GrabCut 真正的迭代

完整能量：

$$
E(\alpha,k,\theta,z)
$$

同时依赖三类未知量：

- `alpha`：像素前景/背景标签。
- `k`：每个像素分配到哪个 Gaussian 成分。
- `theta`：GMM 参数。

GrabCut 不是一次性求：

$$
\arg\min_{\alpha,k,\theta}E(\alpha,k,\theta,z)
$$

而是循环做三个子问题。

### 12.1 固定 alpha 和 theta，更新 k

对每个像素，选择当前标签下代价最低的 GMM 成分：

$$
k_n
\leftarrow
\arg\min_{k}
D(\alpha_n,k,\theta,z_n)
$$

这是一个逐像素枚举问题。因为 `K` 很小，直接算每个 Gaussian 的代价即可。

### 12.2 固定 alpha 和 k，更新 theta

对每个类别 `alpha` 和成分 `k`，收集像素集合：

$$
F_{\alpha,k}
=
\{z_n \mid \alpha_n=\alpha,\ k_n=k\}
$$

更新混合权重：

$$
\pi(\alpha,k)
=
\frac{|F_{\alpha,k}|}
{\sum_{j=1}^{K}|F_{\alpha,j}|}
$$

更新均值：

$$
\mu(\alpha,k)
=
\frac{1}{|F_{\alpha,k}|}
\sum_{z_n\in F_{\alpha,k}}z_n
$$

更新协方差：

$$
\Sigma(\alpha,k)
=
\frac{1}{|F_{\alpha,k}|}
\sum_{z_n\in F_{\alpha,k}}
(z_n-\mu(\alpha,k))(z_n-\mu(\alpha,k))^T
$$

这一步就是最大似然估计。直观地说：当前被认为是前景的像素会反过来重塑前景 GMM；当前被认为是背景的像素会重塑背景 GMM。

这也是 GrabCut 能从粗矩形逐步变好的原因：初始标签很粗，但只要第一轮切分稍微改善，新的 GMM 就会更接近真实前景/背景分布，下一轮切分又会进一步改善。

### 12.3 固定 k 和 theta，更新 alpha

构造当前二值 MRF：

$$
E(\alpha)
=
\sum_n D_n(\alpha_n)
+
\sum_{(m,n)\in C}V_{mn}[\alpha_m\neq\alpha_n]
$$

用 graph cut 求：

$$
\alpha
\leftarrow
\arg\min_{\alpha}E(\alpha)
$$

这是当前固定参数下的全局最优标签。

### 12.4 单调下降与局部最优

每一步都在固定其他变量时降低或不增加总能量：

$$
E^{(t+1)} \le E^{(t)}
$$

能量有下界，因此迭代会收敛到某个稳定点。

但要注意：

$$
\text{单调下降} \neq \text{全局最优}
$$

原因是整体问题同时包含：

- 离散标签 `alpha`
- 离散成分 `k`
- 连续参数 `theta`
- 用户硬约束
- 非凸的混合模型参数估计

GrabCut 的保证是：每个分块子问题有清晰优化意义，能量会下降，并收敛到局部最小或稳定点。它不保证在所有可能初始化中找到全局最佳分割。

## 13. 为什么矩形初始化能工作

矩形初始化看似粗糙，但数学上有两个作用。

第一，矩形外给出大量确定背景样本：

$$
n \notin R \Rightarrow \alpha_n = 0
$$

这让背景 GMM 一开始就有相对可靠的训练数据。

第二，矩形内作为未知或临时前景，使前景 GMM 至少有一批候选样本。虽然里面混有背景，但 graph cut 通过边界项和颜色项可以逐步排除一部分背景。

初始状态可抽象为：

$$
\alpha_n^{(0)}
=
\begin{cases}
0, & n \notin R \\
1, & n \in R
\end{cases}
$$

但这里的 `1` 对矩形内像素只是 provisional foreground，不是用户硬标的确定前景。

如果矩形切掉了真实目标的一部分：

$$
n \notin R,\ n \in \text{true foreground}
$$

该像素会被硬约束成背景。后续颜色模型再好，也很难把它恢复成前景。这是矩形不能过紧的数学原因。

## 14. OpenCV API 在模型里的位置

OpenCV 的接口只是把上述变量包装成工程参数：

| OpenCV 参数 | 数学对象 |
| --- | --- |
| `img` | 观测 `z` |
| `mask` | 约束和当前标签 `alpha` 的工程编码 |
| `rect` | 初始化 `T_B` 和 `T_U` |
| `bgdModel` | 背景 GMM 参数 `theta` 的内部存储 |
| `fgdModel` | 前景 GMM 参数 `theta` 的内部存储 |
| `iterCount` | 交替最小化循环次数 |
| `mode` | 使用矩形、mask、继续迭代或固定模型的控制方式 |

OpenCV 4.x 官方文档在 2026-06-26 核对时定义的四类 mask 标签是：

| 标签 | 数值 | 数学含义 |
| --- | --- | --- |
| `GC_BGD` | `0` | 确定背景约束 |
| `GC_FGD` | `1` | 确定前景约束 |
| `GC_PR_BGD` | `2` | 当前估计背景 |
| `GC_PR_FGD` | `3` | 当前估计前景 |

本文不展开代码。只需记住：OpenCV 的 `bgdModel` 和 `fgdModel` 不是跨图片复用的训练模型，而是当前图像、当前交互状态下的 GMM 参数缓存。

## 15. 失败模式的数学解释

| 现象 | 数学原因 | 对应修复 |
| --- | --- | --- |
| 前景和背景颜色接近 | `D(0,k,theta,z_n)` 与 `D(1,k,theta,z_n)` 差距小，数据项弱 | 增加硬约束，或引入更强语义先验 |
| 边界不清 | `||z_m-z_n||^2` 小，边界项不鼓励在真实边界切开 | 用笔刷修正，或换 matting/语义模型 |
| 结果过度平滑 | `gamma` 相对过强，切边代价压过数据项 | 降低平滑影响或增加局部约束 |
| 区域破碎 | 数据项局部波动大，平滑项不足 | 增强平滑或后处理连通性 |
| 目标被矩形裁掉 | 矩形外被硬约束为 `alpha=0` | 重画矩形 |
| 多个相似实例被保留 | GMM 只看颜色，不知道实例身份 | 对非目标实例加背景约束 |
| 半透明边缘发硬 | `alpha_n` 是硬二值，不是连续 alpha | 后接 image matting |

这些失败不是“OpenCV 参数没调好”这么简单，而是模型表达能力的边界：

$$
\text{GrabCut sees color + contrast + user constraints, not semantic identity.}
$$

如果语义信息没有进入能量函数，它就不会凭空出现在优化结果里。

## 16. 与语义分割和 matting 的边界

GrabCut 求的是：

$$
\alpha_n \in \{0,1\}
$$

语义分割求的是：

$$
y_n \in \{1,2,\dots,C\}
$$

matting 求的是：

$$
\alpha_n \in [0,1]
$$

三者不是同一问题。

GrabCut 的前景不是类别，而是用户当前想保留的区域。它没有“人”“车”“商品”这些语义标签。它输出的是硬分割，不是透明度。原论文在 hard segmentation 之后还讨论 border matting，但 OpenCV 常用 `grabCut` 接口的核心输出仍是四值 mask。

因此：

- 想自动知道“这是什么物体”：用检测或语义/实例分割。
- 想得到透明发丝、玻璃、薄纱：用 matting。
- 已经有人给出粗框或正负笔刷，只想用颜色和边界细化：GrabCut 合适。

## 17. 自测题

1. 为什么 GrabCut 的能量函数必须同时有 `U` 和 `V`？如果去掉其中一个会发生什么？
2. 在数据项里，Mahalanobis distance 相比普通欧氏距离多表达了什么？
3. 为什么 `exp(-beta ||z_m-z_n||^2)` 会让 cut 倾向于经过强颜色边缘？
4. 固定 `theta` 和 `k` 后，为什么 `alpha` 子问题可以用 graph cut 精确求解？
5. GrabCut 为什么能量单调下降，但整体仍然只能说收敛到局部最优或稳定点？
6. 矩形外如果包含真实前景，为什么后续迭代很难恢复？
7. 为什么 OpenCV 的 `GC_PR_FGD` 不是数学上的第三类标签？

参考答案要点：

- `U` 提供颜色解释，`V` 提供空间连贯与边界偏好；缺一会退化成碎裂分类或纯平滑。
- Mahalanobis distance 考虑协方差尺度和通道相关性。
- 颜色差越大，指数项越小，切开代价越低。
- 二值 Potts pairwise term 非负且 submodular，可构造成 s-t min-cut。
- 每个分块优化下降能量，但联合非凸、含离散和连续变量。
- 矩形外是硬背景约束，相当于前景标签代价无穷大。
- `GC_PR_FGD` 只是“当前可变标签状态”，底层前景/背景仍是二值 `alpha`。

## 18. 迁移与关联模型

理解 GrabCut 后，可以迁移到这些模型：

- 二值 MRF / CRF 的 unary + pairwise 能量结构。
- s-t graph cut 与 max-flow/min-cut。
- Gaussian Mixture Model 的 hard assignment 和参数估计。
- 传统视觉里“局部观测代价 + 空间先验”的建模方式。
- 深度分割模型之后的能量优化或边界 refinement。

特别值得继续补的数学链路是：

$$
\text{MAP inference}
\rightarrow
\text{negative log energy}
\rightarrow
\text{submodular binary MRF}
\rightarrow
\text{s-t graph construction}
\rightarrow
\text{min-cut / max-flow}
$$

这条链路学明白后，GrabCut 就不再只是一个 OpenCV 函数，而是一个典型的视觉能量最小化案例。

## 19. 参考资料

- OpenCV 4.x imgproc miscellaneous transformations: https://docs.opencv.org/4.x/d7/d1b/group__imgproc__misc.html
- OpenCV Python tutorial, Interactive Foreground Extraction using GrabCut Algorithm: https://docs.opencv.org/4.x/d8/d83/tutorial_py_grabcut.html
- Carsten Rother, Vladimir Kolmogorov, Andrew Blake, "GrabCut": Interactive Foreground Extraction using Iterated Graph Cuts, ACM Transactions on Graphics / SIGGRAPH 2004: https://www.microsoft.com/en-us/research/publication/grabcut-interactive-foreground-extraction-using-iterated-graph-cuts/
- GrabCut paper PDF from Microsoft Research: https://www.microsoft.com/en-us/research/wp-content/uploads/2004/08/siggraph04-grabcut.pdf
- ACM DOI entry for the GrabCut paper: https://dl.acm.org/doi/10.1145/1015706.1015720
- [统一概念文档规范：AI 生成、升级、审查与仓库集成](../methodology/document-generation-methodology.md)
- [来源纪律与真实世界锚点政策：当前实践、历史路径与不可验证声明处理](../methodology/source-discipline-and-real-world-anchor-policy.md)
