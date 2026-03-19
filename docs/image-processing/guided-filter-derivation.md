---
doc_id: image-processing-guided-filter
title: 导向滤波的数学原理、公式推导与 C++ 对照
concept: guided_filter
topic: image-processing
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-03-19T20:45:00+08:00'
source_basis:
  - guided_image_filtering_eccv2010_checked_2026_03_19
  - guided_image_filtering_tpami2013_checked_2026_03_19
  - opencv_ximgproc_guided_filter_docs_checked_2026_03_19
  - matlab_imguidedfilter_docs_checked_2026_03_19
time_context: foundations_plus_current_practice_checked_2026_03_19
applicability: edge_aware_filtering_modeling_algorithm_derivation_and_implementation_review
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/image-processing/fast-guided-filter-derivation.md
  - docs/methodology/concept-document-template.md
open_questions:
  - 是否需要补充彩色导向滤波与实现复杂度的对照说明？
  - 是否需要补充 box filter 线性复杂度实现与数值稳定性备注？
---

# 导向滤波的数学原理、公式推导与 C++ 对照

## 1. 这份文档要帮你学会什么

这篇文档要帮你把导向滤波从“一个经典公式”升级成“一个以后能用于选型、调参、解释失效原因的内部模型”。

读完后，你应该至少能做到：

- 说清导向滤波到底在解决什么问题，以及它为什么不是普通均值滤波
- 理解局部线性模型、`a_k` / `b_k`、`r`、`eps` 与 box filter 加速之间的关系
- 判断什么时候该用 guided filter，什么时候该换 joint bilateral、matting 或 learned refinement
- 把推导公式和 OpenCV / MATLAB 这类真实实现对上号
- 用边缘、深度图、mask 精修等例子检查自己是否真的掌握了它

## 2. 一句话结论 / 问题定义

**导向滤波是一种 edge-aware 局部线性滤波：它假设输出 `q` 在每个局部窗口内都可以写成导向图 `I` 的线性变换，从而在平坦区域做平滑、在 guidance 的边缘附近尽量不跨边界混合。**

它真正要解决的问题不是“把图像模糊掉”，而是：

- 当输入结果 `p` 很噪时，怎样在保住 guidance 边缘的前提下做平滑
- 怎样把 joint upsampling、depth/disparity refinement、mask 精修这类问题压成局部线性回归
- 怎样用线性时间复杂度得到可解释、可控的 edge-aware 后处理

## 3. 对象边界与相邻概念

导向滤波处理的是：

- 一张导向图 `I`
- 一张待滤波图 `p`
- 一个局部窗口假设
- 一组由最小二乘得到的局部线性系数

它不等于：

- 普通均值滤波或 Gaussian smoothing
- 语义级 refinement 模块
- “任何视觉任务都优于 learned post-processing”的通用替代

最值得对照的相邻概念是：

- joint bilateral filter
- matting Laplacian
- moving least squares / ridge-like local regression
- Fast Guided Filter

## 4. 核心结构

导向滤波最小结构可以压成 6 个构件：

1. guidance image `I`
2. filtering input `p`
3. 局部窗口 `ω_k`
4. 局部线性模型 `q_i = a_k I_i + b_k`
5. 正则项 `eps`
6. 用 box filter 线性时间计算局部均值、方差、协方差

最重要的结构判断是：它的核心假设不是“局部常数”，而是“局部线性”。

## 5. 核心机制 / 主链路 / 因果链

导向滤波的主链路可以压成 5 步：

1. 在每个局部窗口里假设输出对 guidance 满足线性关系
2. 对 `a_k`、`b_k` 建立带正则项的最小二乘问题
3. 用局部方差 / 协方差得到闭式解
4. 因为每个像素属于多个重叠窗口，所以再对系数做一次局部平均
5. 在最终输出阶段用 `q = mean(a) * I + mean(b)` 恢复结果

为什么它能保边，关键不在“窗口里少平均一点”，而在 guidance 的局部统计量会让线性系数随边缘变化。

## 6. 关键 tradeoff 与失败模式

导向滤波买到的是：

- 线性时间复杂度
- 对 guidance 边缘敏感
- 公式和参数都相对可解释

它的代价和失效边界也很明确：

- `eps` 太大时，边缘也会被抹平
- `eps` 太小或 guidance 质量太差时，结果会跟着坏 guidance 走
- 它只能跟随 guidance 中已有的结构，不能凭空恢复语义边界
- 彩色 guidance 版本计算更重，矩阵实现更容易踩数值稳定和边界处理细节

## 7. 应用场景

导向滤波典型适用于：

- depth / disparity refinement
- alpha matte、mask、transmission map 的边缘细化
- joint upsampling
- 移动端或机器人视觉里的轻量 edge-aware 后处理

## 8. 工业 / 现实世界锚点

### 8.1 OpenCV `cv::ximgproc::guidedFilter`

OpenCV 的 ximgproc 模块提供了 `guidedFilter` 和 `createGuidedFilter` 接口，这是 guided filter 进入真实工程工具链的直接锚点，而不是停留在论文公式。

### 8.2 MATLAB `imguidedfilter`

MathWorks 提供了 `imguidedfilter`，支持 self-guidance 和外部 guidance，还给出代码生成能力和实际示例。这说明 guided filter 已经进入工业与科研常用产品环境，而不只是论文原型。

### 8.3 原始论文页的产品化线索

Kaiming He 的 ECCV 2010 页面明确提到 guided filter 的官方实现进入了 MATLAB 2014 和 OpenCV 3.0，这给了它非常具体的现实落地坐标。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-19 更推荐的实践

当前更稳的做法通常是：

- 把 guided filter 用在 edge-aware refinement 和 joint upsampling 这类它真正擅长的问题上
- 让 `r` 与空间尺度、`eps` 与数据范围对应，而不是把参数当 magic number
- 在真实工程里优先复用 OpenCV / MATLAB 这类成熟实现，再做定制优化
- 当任务已经依赖全局语义时，优先考虑 learned refinement 或更强先验模型

### 9.2 过时路径与替代

下面这些路径通常已经不够稳：

- 把导向滤波当通用去噪万能器
- 把它误当成语义级后处理
- 看到边缘保持需求就默认它一定比更强模型更合适

更稳的替代是：

- 当你需要轻量、可解释、基于 guidance 的局部结构保持时，用 guided filter
- 当你需要更强语义理解时，换 task-specific network、matting / segmentation 后处理或 learned refinement

## 10. 自测题 / 验证入口

1. 为什么 guided filter 的核心假设是“局部线性”，而不是“局部常数”？
2. `a_k` 和 `b_k` 各自控制什么，为什么还要做一次局部平均？
3. 为什么 guidance 的质量会直接限制 guided filter 的上限？
4. `eps` 为什么必须和数据范围、噪声水平一起看？
5. 如果问题真正依赖语义边界而不是图像梯度边界，为什么 guided filter 往往不够？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- Fast Guided Filter
- joint bilateral filter
- matting Laplacian
- coarse-to-fine approximation
- 一切“用 guidance 约束局部结构”的视觉后处理问题

## 12. 未解问题与继续深挖

- 彩色 guided filter 的矩阵实现怎样更系统地处理数值稳定与边界条件？
- 在真实工业数据上，`r`、`eps` 与噪声模型之间怎样建立更稳的经验规则？
- 在哪些任务上 guided filter 仍是足够好的后处理，而哪些任务应该直接转向 learned refinement？

## 13. 参考资料

以下“当前实践”相关内容的核对日期均为 `2026-03-19`。

- Kaiming He, Jian Sun, Xiaoou Tang. Guided Image Filtering. ECCV 2010 project page: https://people.csail.mit.edu/kaiming/eccv10/index.html
- Kaiming He, Jian Sun, Xiaoou Tang. Guided Image Filtering. TPAMI 2013: https://pubmed.ncbi.nlm.nih.gov/23599054/
- OpenCV ximgproc filters docs: https://docs.opencv.org/4.x/da/d17/group__ximgproc__filters.html
- MathWorks `imguidedfilter`: https://www.mathworks.com/help/images/ref/imguidedfilter.html

## 14. 详细推导、代码对应与补充说明

### 1. 导向滤波在做什么

导向滤波（Guided Filter）要解决的问题可以写成：

- 已知一张导向图 $I$（guidance image）
- 已知一张待滤波图 $p$（filtering input）
- 希望得到输出图 $q$

它的目标不是单纯做“平均”，而是：

- 在平坦区域里平滑噪声
- 在边缘附近尽量不要跨边缘模糊
- 让输出结构尽量跟随导向图 $I$ 的边缘

它和普通均值滤波的关键区别在于：

- 均值滤波假设“窗口内大家都应该变得接近”
- 导向滤波假设“窗口内输出 $q$ 可以由导向图 $I$ 的一个局部线性模型来表示”

也就是说，导向滤波的核心假设不是“局部常数”，而是“局部线性”。

---

### 2. 符号约定

下面先讲最经典的**灰度导向滤波**。

- $I$：导向图，单通道
- $p$：输入图，单通道
- $q$：输出图，单通道
- $\omega_k$：以像素 $k$ 为中心的局部窗口
- $|\omega|$：窗口像素数。如果窗口半径为 $r$，则理想情况下 $|\omega| = (2r+1)^2$
- $i$：窗口中的任意像素索引
- $\mu_k$：窗口 $\omega_k$ 内导向图 $I$ 的均值
- $\bar{p}_k$：窗口 $\omega_k$ 内输入图 $p$ 的均值
- $\sigma_k^2$：窗口 $\omega_k$ 内导向图 $I$ 的方差
- $\operatorname{cov}_{I,p,k}$：窗口 $\omega_k$ 内 $I$ 与 $p$ 的协方差
- $\epsilon$：正则项，控制线性系数 $a_k$ 的大小

为了书写简洁，后文也会把窗口内均值写成：

$$
\mu_k = \frac{1}{|\omega|}\sum_{i\in \omega_k} I_i,
\qquad
\bar{p}_k = \frac{1}{|\omega|}\sum_{i\in \omega_k} p_i
$$

---

### 3. 灰度导向滤波的局部线性模型

导向滤波假设：在每个局部窗口 $\omega_k$ 内，输出 $q$ 与导向图 $I$ 满足线性关系

$$
q_i = a_k I_i + b_k,\qquad \forall i\in\omega_k
$$

这里：

- $a_k$ 是窗口 $\omega_k$ 内的斜率
- $b_k$ 是窗口 $\omega_k$ 内的截距

这个式子非常关键，它说明：

1. 在同一个局部窗口内，$a_k$ 和 $b_k$ 是常数  
2. 但是不同窗口会有不同的 $a_k, b_k$  
3. 输出 $q_i$ 不是直接对 $p_i$ 求平均，而是通过导向图 $I_i$ 重建出来

这也是它保边的根源：如果导向图在边缘处发生突变，那么线性模型会随着局部统计量变化，从而避免像均值滤波那样把边缘抹平。

---

### 4. 优化目标是怎么来的

既然在窗口 $\omega_k$ 内假设

$$
q_i = a_k I_i + b_k
$$

那么自然希望它尽量接近输入图 $p_i$。于是构造最小二乘目标函数：

$$
E(a_k,b_k)
=
\sum_{i\in\omega_k}
\Bigl( (a_k I_i + b_k - p_i)^2 + \epsilon a_k^2 \Bigr)
$$

这个目标函数包含两部分：

#### 4.1 数据项

$$
\sum_{i\in\omega_k}(a_k I_i + b_k - p_i)^2
$$

它要求局部线性模型尽量拟合输入图 $p$。

#### 4.2 正则项

$$
\sum_{i\in\omega_k}\epsilon a_k^2 = |\omega|\epsilon a_k^2
$$

它惩罚过大的斜率 $a_k$，避免在局部方差很小的时候分母接近零，导致数值不稳定。

直观上：

- $\epsilon$ 小：更愿意相信局部线性关系，边缘跟随更强
- $\epsilon$ 大：更偏向平滑，$a_k$ 会更小

---

### 5. 对灰度情形做闭式求解

下面开始正式推导 $a_k$ 和 $b_k$。

#### 5.1 先对 $b_k$ 求偏导

目标函数对 $b_k$ 的偏导为：

$$
\frac{\partial E}{\partial b_k}
=
2\sum_{i\in\omega_k}(a_k I_i + b_k - p_i)
= 0
$$

移项得到：

$$
a_k \sum_{i\in\omega_k} I_i + |\omega| b_k - \sum_{i\in\omega_k} p_i = 0
$$

两边同时除以 $|\omega|$：

$$
a_k \mu_k + b_k - \bar{p}_k = 0
$$

因此

$$
b_k = \bar{p}_k - a_k \mu_k
$$

这是第一个闭式结果。

---

#### 5.2 把 $b_k$ 代回模型

把上式代回

$$
q_i = a_k I_i + b_k
$$

得到

$$
q_i = a_k I_i + \bar{p}_k - a_k\mu_k
= a_k(I_i - \mu_k) + \bar{p}_k
$$

于是残差变成：

$$
a_k I_i + b_k - p_i
=
a_k(I_i - \mu_k) - (p_i - \bar{p}_k)
$$

此时目标函数可以改写成

$$
E(a_k)
=
\sum_{i\in\omega_k}
\Bigl(
a_k(I_i - \mu_k) - (p_i - \bar{p}_k)
\Bigr)^2
+
|\omega|\epsilon a_k^2
$$

这里已经只剩一个未知量 $a_k$。

---

#### 5.3 对 $a_k$ 求偏导

对上式求导：

$$
\frac{\partial E}{\partial a_k}
=
2\sum_{i\in\omega_k}
\Bigl(
a_k(I_i - \mu_k) - (p_i - \bar{p}_k)
\Bigr)
(I_i - \mu_k)
+
2|\omega|\epsilon a_k
=0
$$

展开后得：

$$
a_k \sum_{i\in\omega_k}(I_i - \mu_k)^2
-
\sum_{i\in\omega_k}(I_i - \mu_k)(p_i - \bar{p}_k)
+
|\omega|\epsilon a_k = 0
$$

移项：

$$
a_k
\Biggl[
\sum_{i\in\omega_k}(I_i - \mu_k)^2 + |\omega|\epsilon
\Biggr]
=
\sum_{i\in\omega_k}(I_i - \mu_k)(p_i - \bar{p}_k)
$$

两边除以 $|\omega|$，定义方差和协方差：

$$
\sigma_k^2
=
\frac{1}{|\omega|}
\sum_{i\in\omega_k}(I_i - \mu_k)^2
$$

$$
\operatorname{cov}_{I,p,k}
=
\frac{1}{|\omega|}
\sum_{i\in\omega_k}(I_i - \mu_k)(p_i - \bar{p}_k)
$$

于是得到最重要的闭式解：

$$
a_k = \frac{\operatorname{cov}_{I,p,k}}{\sigma_k^2 + \epsilon}
$$

再代回到 $b_k$：

$$
b_k = \bar{p}_k - a_k\mu_k
$$

这就是灰度导向滤波的核心公式。

---

### 6. 用均值形式改写方差和协方差

为了便于写代码，通常不用“中心化后再求和”的形式，而改写成下面这种形式。

#### 6.1 协方差

$$
\operatorname{cov}_{I,p,k}
=
\frac{1}{|\omega|}\sum_{i\in\omega_k} I_i p_i
-
\mu_k \bar{p}_k
$$

写成更像代码的记号：

$$
\operatorname{cov}_{I,p}
=
\operatorname{mean}(I\cdot p)
-
\operatorname{mean}(I)\operatorname{mean}(p)
$$

#### 6.2 方差

$$
\sigma_k^2
=
\frac{1}{|\omega|}\sum_{i\in\omega_k} I_i^2
-
\mu_k^2
$$

写成代码形式：

$$
\operatorname{var}(I)
=
\operatorname{mean}(I^2)
-
\operatorname{mean}(I)^2
$$

于是灰度情形的代码实现基本就只剩“求很多局部均值”。

---

### 7. 为什么一个像素会有多个模型

前面求出的 $a_k, b_k$ 是“窗口 $\omega_k$ 的模型参数”。  
但一个像素 $i$ 会同时落在很多窗口里，所以它其实会得到很多个预测值：

$$
q_i^{(k)} = a_k I_i + b_k,\qquad \text{其中 } i\in\omega_k
$$

导向滤波最终把这些预测值再平均一次：

$$
q_i
=
\frac{1}{|\Omega_i|}
\sum_{k:\, i\in\omega_k}(a_k I_i + b_k)
$$

其中 $\Omega_i$ 表示“所有包含像素 $i$ 的窗口中心集合”。

因为 $I_i$ 与像素 $i$ 本身有关，但在求和里对每个窗口都是同一个值，所以可以提出去：

$$
q_i
=
I_i
\left(
\frac{1}{|\Omega_i|}
\sum_{k:\,i\in\omega_k} a_k
\right)
+
\left(
\frac{1}{|\Omega_i|}
\sum_{k:\,i\in\omega_k} b_k
\right)
$$

定义

$$
\bar{a}_i
=
\frac{1}{|\Omega_i|}
\sum_{k:\,i\in\omega_k} a_k,
\qquad
\bar{b}_i
=
\frac{1}{|\Omega_i|}
\sum_{k:\,i\in\omega_k} b_k
$$

于是得到最终输出公式：

$$
q_i = \bar{a}_i I_i + \bar{b}_i
$$

这个式子是最终真正出现在代码里的输出式。

---

### 8. 为什么它能保边

从公式

$$
a_k = \frac{\operatorname{cov}_{I,p,k}}{\sigma_k^2 + \epsilon}
$$

可以看出：

- 如果窗口内 $I$ 与 $p$ 变化趋势一致，那么协方差大，$a_k$ 会大一些，输出就会更强地跟随导向图边缘
- 如果窗口内 $I$ 基本是常数，那么 $\sigma_k^2$ 很小，$a_k$ 会被正则项压小，输出更接近局部常数
- 如果导向图在某处有明显边缘，那么边缘两侧窗口统计量不同，得到的 $a_k,b_k$ 也不同，最终不会简单跨边缘平均

这就是“边缘保持”的数学来源。

---

### 9. 用盒式滤波把复杂度降到线性

上面所有关键量都只是窗口均值：

- $\operatorname{mean}(I)$
- $\operatorname{mean}(p)$
- $\operatorname{mean}(Ip)$
- $\operatorname{mean}(I^2)$
- $\operatorname{mean}(a)$
- $\operatorname{mean}(b)$

因此，整个导向滤波可以写成“常数次局部均值运算”。

定义一个局部均值算子：

$$
\mathcal{M}_r[f](i)
=
\frac{1}{N_i}
\sum_{j\in\omega_i} f_j
$$

其中 $N_i$ 是窗口内有效像素数。在代码里，这个算子可以由 `boxFilter` 实现。

于是灰度导向滤波可以压缩写成：

$$
\mu_I = \mathcal{M}_r[I],\qquad
\mu_p = \mathcal{M}_r[p]
$$

$$
\operatorname{cov}_{I,p}
=
\mathcal{M}_r[Ip] - \mu_I\mu_p
$$

$$
\sigma_I^2
=
\mathcal{M}_r[I^2] - \mu_I^2
$$

$$
a = \frac{\operatorname{cov}_{I,p}}{\sigma_I^2 + \epsilon}
$$

$$
b = \mu_p - a\mu_I
$$

$$
\bar{a} = \mathcal{M}_r[a],\qquad
\bar{b} = \mathcal{M}_r[b]
$$

$$
q = \bar{a}I + \bar{b}
$$

注意：这里的所有乘法、除法、减法，都是按像素逐点进行。

由于窗口大小只通过 `boxFilter` 体现，而不是在每个像素上重新遍历整个窗口，所以整体复杂度对图像像素数是线性的，通常记为 $O(HW)$。

---

### 10. 彩色导向滤波的矩阵推导

现在把导向图从单通道扩展到三通道。  
输入图 $p$ 仍然先看成单通道，导向图 $I_i$ 则是三维向量：

$$
I_i =
\begin{bmatrix}
I_i^{(1)} \\
I_i^{(2)} \\
I_i^{(3)}
\end{bmatrix}
$$

局部线性模型变成：

$$
q_i = a_k^T I_i + b_k,\qquad \forall i\in\omega_k
$$

其中：

- $a_k \in \mathbb{R}^3$
- $b_k \in \mathbb{R}$

也就是说，原来灰度时的一个斜率 $a_k$，现在变成了三维系数向量。

---

#### 10.1 彩色情形的目标函数

$$
E(a_k,b_k)
=
\sum_{i\in\omega_k}
\Bigl(
(a_k^T I_i + b_k - p_i)^2
+
\epsilon\, a_k^T a_k
\Bigr)
$$

这里正则项是向量范数平方。

---

#### 10.2 先对 $b_k$ 求解

与灰度情形完全一样，

$$
\frac{\partial E}{\partial b_k}
=
2\sum_{i\in\omega_k}(a_k^T I_i + b_k - p_i)=0
$$

记窗口内导向图均值向量为

$$
\mu_k
=
\frac{1}{|\omega|}
\sum_{i\in\omega_k} I_i
$$

窗口内输入均值为

$$
\bar{p}_k
=
\frac{1}{|\omega|}
\sum_{i\in\omega_k} p_i
$$

则有

$$
b_k = \bar{p}_k - a_k^T \mu_k
$$

---

#### 10.3 代回并对 $a_k$ 求解

令中心化变量

$$
\tilde{I}_i = I_i - \mu_k,\qquad
\tilde{p}_i = p_i - \bar{p}_k
$$

则目标函数变为

$$
E(a_k)
=
\sum_{i\in\omega_k}
(a_k^T\tilde{I}_i - \tilde{p}_i)^2
+
|\omega|\epsilon\, a_k^T a_k
$$

对向量 $a_k$ 求梯度并令其为零：

$$
\frac{\partial E}{\partial a_k}
=
2\sum_{i\in\omega_k}\tilde{I}_i(\tilde{I}_i^T a_k - \tilde{p}_i)
+
2|\omega|\epsilon a_k
= 0
$$

整理得：

$$
\left(
\sum_{i\in\omega_k}\tilde{I}_i\tilde{I}_i^T
+
|\omega|\epsilon U
\right)a_k
=
\sum_{i\in\omega_k}\tilde{I}_i\tilde{p}_i
$$

其中 $U$ 是 $3\times 3$ 单位矩阵。

两边除以 $|\omega|$：

$$
(\Sigma_k + \epsilon U)a_k = \operatorname{cov}_{I,p,k}
$$

这里

$$
\Sigma_k
=
\frac{1}{|\omega|}
\sum_{i\in\omega_k}
\tilde{I}_i\tilde{I}_i^T
$$

是三通道导向图的局部协方差矩阵，

$$
\operatorname{cov}_{I,p,k}
=
\frac{1}{|\omega|}
\sum_{i\in\omega_k}
\tilde{I}_i\tilde{p}_i
$$

是导向图与输入图的协方差向量。

因此闭式解为：

$$
a_k = (\Sigma_k + \epsilon U)^{-1}\operatorname{cov}_{I,p,k}
$$

$$
b_k = \bar{p}_k - a_k^T\mu_k
$$

这就是彩色导向滤波最核心的矩阵公式。

---

#### 10.4 把矩阵展开成代码可算的形式

记三通道分别为 $I^{(1)}, I^{(2)}, I^{(3)}$，那么

$$
\Sigma_k =
\begin{bmatrix}
\sigma_{11} & \sigma_{12} & \sigma_{13} \\
\sigma_{21} & \sigma_{22} & \sigma_{23} \\
\sigma_{31} & \sigma_{32} & \sigma_{33}
\end{bmatrix}
$$

其中

$$
\sigma_{mn}
=
\operatorname{mean}(I^{(m)}I^{(n)})
-
\operatorname{mean}(I^{(m)})\operatorname{mean}(I^{(n)})
$$

而协方差向量是

$$
\operatorname{cov}_{I,p}
=
\begin{bmatrix}
\operatorname{cov}(I^{(1)},p) \\
\operatorname{cov}(I^{(2)},p) \\
\operatorname{cov}(I^{(3)},p)
\end{bmatrix}
$$

其中

$$
\operatorname{cov}(I^{(m)},p)
=
\operatorname{mean}(I^{(m)}p)
-
\operatorname{mean}(I^{(m)})\operatorname{mean}(p)
$$

所以彩色实现本质上就是：

1. 用盒式滤波求出所有一阶均值  
2. 用盒式滤波求出所有二阶项均值  
3. 组装每个像素位置上的 $3\times 3$ 协方差矩阵  
4. 解一个 $3\times 3$ 线性方程组  
5. 再对得到的 $a$ 和 $b$ 做一次局部均值  

---

### 11. 对应到 OpenCV C++：完整代码

下面这份代码故意写得“接近公式”，而不是追求最少行数。  
目标是让你能把每一条数学公式和代码变量直接对应起来。

```cpp
#include <opencv2/opencv.hpp>
#include <vector>

namespace {

cv::Size windowSize(int r) {
    return cv::Size(2 * r + 1, 2 * r + 1);
}

cv::Mat computeWindowPixelCount(cv::Size size, int r) {
    cv::Mat ones(size, CV_32F, cv::Scalar(1.0f));
    cv::Mat N;
    cv::boxFilter(
        ones, N, CV_32F, windowSize(r),
        cv::Point(-1, -1), false, cv::BORDER_REFLECT);
    return N;
}

cv::Mat boxMean(const cv::Mat& src, const cv::Mat& N, int r) {
    cv::Mat sum;
    cv::boxFilter(
        src, sum, CV_32F, windowSize(r),
        cv::Point(-1, -1), false, cv::BORDER_REFLECT);
    return sum / N;
}

}  // namespace

cv::Mat guidedFilterGray(const cv::Mat& I, const cv::Mat& p, int r, double eps) {
    CV_Assert(I.type() == CV_32F);
    CV_Assert(p.type() == CV_32F);
    CV_Assert(I.size() == p.size());

    const cv::Mat N = computeWindowPixelCount(I.size(), r);

    const cv::Mat mean_I  = boxMean(I, N, r);
    const cv::Mat mean_p  = boxMean(p, N, r);
    const cv::Mat mean_Ip = boxMean(I.mul(p), N, r);
    const cv::Mat mean_II = boxMean(I.mul(I), N, r);

    const cv::Mat cov_Ip = mean_Ip - mean_I.mul(mean_p);
    const cv::Mat var_I  = mean_II - mean_I.mul(mean_I);

    const cv::Mat a = cov_Ip / (var_I + eps);
    const cv::Mat b = mean_p - a.mul(mean_I);

    const cv::Mat mean_a = boxMean(a, N, r);
    const cv::Mat mean_b = boxMean(b, N, r);

    return mean_a.mul(I) + mean_b;
}

cv::Mat guidedFilterColor(const cv::Mat& I, const cv::Mat& p, int r, double eps) {
    CV_Assert(I.type() == CV_32FC3);
    CV_Assert(p.type() == CV_32F);
    CV_Assert(I.size() == p.size());

    const float epsf = static_cast<float>(eps);
    const cv::Mat N = computeWindowPixelCount(I.size(), r);

    std::vector<cv::Mat> ch(3);
    cv::split(I, ch);
    const cv::Mat& I0 = ch[0];
    const cv::Mat& I1 = ch[1];
    const cv::Mat& I2 = ch[2];

    const cv::Mat mean_I0 = boxMean(I0, N, r);
    const cv::Mat mean_I1 = boxMean(I1, N, r);
    const cv::Mat mean_I2 = boxMean(I2, N, r);
    const cv::Mat mean_p  = boxMean(p, N, r);

    const cv::Mat mean_I0p = boxMean(I0.mul(p), N, r);
    const cv::Mat mean_I1p = boxMean(I1.mul(p), N, r);
    const cv::Mat mean_I2p = boxMean(I2.mul(p), N, r);

    const cv::Mat cov_Ip0 = mean_I0p - mean_I0.mul(mean_p);
    const cv::Mat cov_Ip1 = mean_I1p - mean_I1.mul(mean_p);
    const cv::Mat cov_Ip2 = mean_I2p - mean_I2.mul(mean_p);

    const cv::Mat var_00 = boxMean(I0.mul(I0), N, r) - mean_I0.mul(mean_I0);
    const cv::Mat var_01 = boxMean(I0.mul(I1), N, r) - mean_I0.mul(mean_I1);
    const cv::Mat var_02 = boxMean(I0.mul(I2), N, r) - mean_I0.mul(mean_I2);
    const cv::Mat var_11 = boxMean(I1.mul(I1), N, r) - mean_I1.mul(mean_I1);
    const cv::Mat var_12 = boxMean(I1.mul(I2), N, r) - mean_I1.mul(mean_I2);
    const cv::Mat var_22 = boxMean(I2.mul(I2), N, r) - mean_I2.mul(mean_I2);

    cv::Mat a0(I.size(), CV_32F);
    cv::Mat a1(I.size(), CV_32F);
    cv::Mat a2(I.size(), CV_32F);
    cv::Mat b(I.size(), CV_32F);

    for (int y = 0; y < I.rows; ++y) {
        const float* v00 = var_00.ptr<float>(y);
        const float* v01 = var_01.ptr<float>(y);
        const float* v02 = var_02.ptr<float>(y);
        const float* v11 = var_11.ptr<float>(y);
        const float* v12 = var_12.ptr<float>(y);
        const float* v22 = var_22.ptr<float>(y);

        const float* cp0 = cov_Ip0.ptr<float>(y);
        const float* cp1 = cov_Ip1.ptr<float>(y);
        const float* cp2 = cov_Ip2.ptr<float>(y);

        const float* mI0 = mean_I0.ptr<float>(y);
        const float* mI1 = mean_I1.ptr<float>(y);
        const float* mI2 = mean_I2.ptr<float>(y);
        const float* mp  = mean_p.ptr<float>(y);

        float* pa0 = a0.ptr<float>(y);
        float* pa1 = a1.ptr<float>(y);
        float* pa2 = a2.ptr<float>(y);
        float* pb  = b.ptr<float>(y);

        for (int x = 0; x < I.cols; ++x) {
            cv::Matx33f Sigma(
                v00[x] + epsf, v01[x],        v02[x],
                v01[x],        v11[x] + epsf, v12[x],
                v02[x],        v12[x],        v22[x] + epsf
            );

            cv::Vec3f cov_Ip(cp0[x], cp1[x], cp2[x]);
            cv::Vec3f ak = Sigma.inv(cv::DECOMP_CHOLESKY) * cov_Ip;

            pa0[x] = ak[0];
            pa1[x] = ak[1];
            pa2[x] = ak[2];
            pb[x] = mp[x] - ak[0] * mI0[x] - ak[1] * mI1[x] - ak[2] * mI2[x];
        }
    }

    const cv::Mat mean_a0 = boxMean(a0, N, r);
    const cv::Mat mean_a1 = boxMean(a1, N, r);
    const cv::Mat mean_a2 = boxMean(a2, N, r);
    const cv::Mat mean_b  = boxMean(b, N, r);

    return mean_a0.mul(I0) + mean_a1.mul(I1) + mean_a2.mul(I2) + mean_b;
}
```

---

### 12. 灰度公式和 C++ 代码的一一对应

这一节把灰度导向滤波的每个关键公式，直接映射到代码变量。

| 数学公式 | 数学含义 | C++ 代码 |
|---|---|---|
| $N_i = \sum_{j\in\omega_i} 1$ | 每个窗口的像素数 | `const cv::Mat N = computeWindowPixelCount(I.size(), r);` |
| $\mu_I = \mathcal{M}_r[I]$ | 导向图局部均值 | `const cv::Mat mean_I = boxMean(I, N, r);` |
| $\mu_p = \mathcal{M}_r[p]$ | 输入图局部均值 | `const cv::Mat mean_p = boxMean(p, N, r);` |
| $\operatorname{mean}(Ip)$ | 乘积的局部均值 | `const cv::Mat mean_Ip = boxMean(I.mul(p), N, r);` |
| $\operatorname{mean}(I^2)$ | 平方的局部均值 | `const cv::Mat mean_II = boxMean(I.mul(I), N, r);` |
| $\operatorname{cov}_{I,p} = \operatorname{mean}(Ip) - \mu_I\mu_p$ | 协方差 | `const cv::Mat cov_Ip = mean_Ip - mean_I.mul(mean_p);` |
| $\sigma_I^2 = \operatorname{mean}(I^2) - \mu_I^2$ | 方差 | `const cv::Mat var_I = mean_II - mean_I.mul(mean_I);` |
| $a = \dfrac{\operatorname{cov}_{I,p}}{\sigma_I^2 + \epsilon}$ | 局部斜率 | `const cv::Mat a = cov_Ip / (var_I + eps);` |
| $b = \mu_p - a\mu_I$ | 局部截距 | `const cv::Mat b = mean_p - a.mul(mean_I);` |
| $\bar{a} = \mathcal{M}_r[a]$ | 重叠窗口对 $a$ 的平均 | `const cv::Mat mean_a = boxMean(a, N, r);` |
| $\bar{b} = \mathcal{M}_r[b]$ | 重叠窗口对 $b$ 的平均 | `const cv::Mat mean_b = boxMean(b, N, r);` |
| $q = \bar{a}I + \bar{b}$ | 最终输出 | `return mean_a.mul(I) + mean_b;` |

需要注意的一点是：

- 数学推导先“对每个窗口求 $a_k,b_k$”
- 代码实现里这些量都已经展开成整张图上的矩阵 `a`, `b`
- `boxMean(a, N, r)` 实际上就是在做“对所有包含当前像素的窗口参数求平均”

---

### 13. 彩色公式和 C++ 代码的一一对应

彩色情形比灰度多出来的核心只有一件事：  
标量方差变成了 $3\times 3$ 协方差矩阵，标量斜率变成了三维向量。

| 数学公式 | 数学含义 | C++ 代码 |
|---|---|---|
| $\mu_{I^{(0)}} = \mathcal{M}_r[I^{(0)}]$ | 第 0 通道局部均值 | `const cv::Mat mean_I0 = boxMean(I0, N, r);` |
| $\mu_{I^{(1)}} = \mathcal{M}_r[I^{(1)}]$ | 第 1 通道局部均值 | `const cv::Mat mean_I1 = boxMean(I1, N, r);` |
| $\mu_{I^{(2)}} = \mathcal{M}_r[I^{(2)}]$ | 第 2 通道局部均值 | `const cv::Mat mean_I2 = boxMean(I2, N, r);` |
| $\mu_p = \mathcal{M}_r[p]$ | 输入图局部均值 | `const cv::Mat mean_p = boxMean(p, N, r);` |
| $\operatorname{cov}(I^{(m)},p)$ | 各通道与 $p$ 的协方差 | `cov_Ip0`, `cov_Ip1`, `cov_Ip2` |
| $\sigma_{00} = \operatorname{mean}(I_0^2) - \mu_{I_0}^2$ | 协方差矩阵左上角 | `const cv::Mat var_00 = ...;` |
| $\sigma_{01} = \operatorname{mean}(I_0 I_1) - \mu_{I_0}\mu_{I_1}$ | 协方差矩阵非对角项 | `const cv::Mat var_01 = ...;` |
| $\sigma_{02}$、$\sigma_{11}$、$\sigma_{12}$、$\sigma_{22}$ | 其余矩阵项 | `var_02`, `var_11`, `var_12`, `var_22` |
| $\Sigma + \epsilon U$ | 加正则后的局部协方差矩阵 | `cv::Matx33f Sigma(...)` |
| $a = (\Sigma + \epsilon U)^{-1}\operatorname{cov}_{I,p}$ | 三维局部斜率 | `cv::Vec3f ak = Sigma.inv(...) * cov_Ip;` |
| $b = \mu_p - a^T\mu_I$ | 局部截距 | `pb[x] = mp[x] - ak[0]*mI0[x] - ak[1]*mI1[x] - ak[2]*mI2[x];` |
| $\bar{a}^{(c)} = \mathcal{M}_r[a^{(c)}]$ | 每个通道系数再做均值 | `mean_a0`, `mean_a1`, `mean_a2` |
| $q = \bar{a}_0 I_0 + \bar{a}_1 I_1 + \bar{a}_2 I_2 + \bar{b}$ | 最终输出 | `return mean_a0.mul(I0) + mean_a1.mul(I1) + mean_a2.mul(I2) + mean_b;` |

这里有一个特别重要的理解：

- 灰度时，分母只是一个数：$\sigma^2 + \epsilon$
- 彩色时，分母变成了一个矩阵：$\Sigma + \epsilon U$

所以彩色导向滤波不是简单地“对三个通道分别做灰度导向滤波”，而是利用通道之间的相关性一起求解。

---

### 14. 代码里为什么要显式写 `N`

你可能会问：`boxFilter` 不是可以直接算均值吗，为什么还要单独写 `N`？

原因有两个：

#### 14.1 为了和推导完全对应

数学上局部均值就是

$$
\operatorname{mean}(f) = \frac{\operatorname{boxsum}(f)}{N}
$$

代码里显式写成

```cpp
sum = boxFilter(src, normalize=false)
mean = sum / N
```

会让“公式到代码”的映射最直接。

#### 14.2 更容易处理边界

如果边界策略导致窗口内有效像素数不是常数，那么 `N` 可以自然吸收这个变化。  
本示例里使用 `BORDER_REFLECT`，因此边界通常也很稳定，但保留 `N` 这个写法更通用。

---

### 15. 每段代码背后的数学意义

下面按代码执行顺序，再把数学意义串一次。

#### 15.1 `mean_I`、`mean_p`

```cpp
const cv::Mat mean_I = boxMean(I, N, r);
const cv::Mat mean_p = boxMean(p, N, r);
```

对应的是窗口均值：

$$
\mu_I = \mathcal{M}_r[I],\qquad
\mu_p = \mathcal{M}_r[p]
$$

---

#### 15.2 `mean_Ip`、`mean_II`

```cpp
const cv::Mat mean_Ip = boxMean(I.mul(p), N, r);
const cv::Mat mean_II = boxMean(I.mul(I), N, r);
```

对应的是：

$$
\operatorname{mean}(Ip),\qquad \operatorname{mean}(I^2)
$$

因为协方差、方差都能通过“二阶项均值减去均值乘积”来得到。

---

#### 15.3 `cov_Ip`、`var_I`

```cpp
const cv::Mat cov_Ip = mean_Ip - mean_I.mul(mean_p);
const cv::Mat var_I  = mean_II - mean_I.mul(mean_I);
```

对应：

$$
\operatorname{cov}_{I,p}
=
\operatorname{mean}(Ip) - \mu_I\mu_p
$$

$$
\sigma_I^2
=
\operatorname{mean}(I^2) - \mu_I^2
$$

---

#### 15.4 `a`、`b`

```cpp
const cv::Mat a = cov_Ip / (var_I + eps);
const cv::Mat b = mean_p - a.mul(mean_I);
```

对应：

$$
a = \frac{\operatorname{cov}_{I,p}}{\sigma_I^2+\epsilon},
\qquad
b = \mu_p - a\mu_I
$$

这一步就是把最小二乘问题的闭式解真正落到代码里。

---

#### 15.5 `mean_a`、`mean_b`

```cpp
const cv::Mat mean_a = boxMean(a, N, r);
const cv::Mat mean_b = boxMean(b, N, r);
```

对应：

$$
\bar{a} = \mathcal{M}_r[a],\qquad
\bar{b} = \mathcal{M}_r[b]
$$

意思是：一个像素属于多个窗口，所以每个窗口里求出的参数还要再做一次局部平均。

---

#### 15.6 最终输出

```cpp
return mean_a.mul(I) + mean_b;
```

对应：

$$
q = \bar{a}I + \bar{b}
$$

这就是最终结果。

---

### 16. 参数 `r` 和 `eps` 的作用

#### 16.1 半径 `r`

- `r` 越大，局部统计量来自更大的窗口，平滑更强
- `r` 越小，输出更贴近局部结构细节

它控制的是“空间范围”。

#### 16.2 正则项 `eps`

- `eps` 越小，$a$ 越容易变大，输出更强地跟随导向图
- `eps` 越大，$a$ 越容易被压小，输出更平滑

它控制的是“线性拟合的刚性”。

一个很重要的工程细节是：  
`eps` 的尺度必须和像素值尺度匹配。

例如：

- 如果图像已经归一化到 $[0,1]$，那么 `eps` 可能取 `1e-6 ~ 1e-2`
- 如果图像还在 $[0,255]$，那么同样的视觉效果需要更大的 `eps`

因此，工程里通常先把图像转成 `CV_32F` 并归一化到 $[0,1]$。

---

### 17. 常见输入准备方式

如果输入是 OpenCV 常见的 `8-bit` 图像，通常先做：

```cpp
cv::Mat src8 = cv::imread("input.png", cv::IMREAD_COLOR);
cv::Mat src32;
src8.convertTo(src32, CV_32FC3, 1.0 / 255.0);
```

如果 `p` 是单通道：

```cpp
cv::Mat gray8 = cv::imread("input.png", cv::IMREAD_GRAYSCALE);
cv::Mat p32;
gray8.convertTo(p32, CV_32F, 1.0 / 255.0);
```

这样 `eps` 的尺度会更好控制。

---

### 18. 应该如何理解灰度版和彩色版的关系

灰度版公式：

$$
a = \frac{\operatorname{cov}_{I,p}}{\operatorname{var}(I) + \epsilon}
$$

彩色版公式：

$$
a = (\Sigma + \epsilon U)^{-1}\operatorname{cov}_{I,p}
$$

两者其实是同一个东西：

- 灰度版的“分母”是一个标量
- 彩色版的“分母”推广成了矩阵

所以可以把灰度导向滤波看成彩色导向滤波在一维情形下的特例。

---

### 19. 几个值得验证的测试场景

如果你要验证自己的实现是否正确，下面几个场景很有用。

#### 19.1 常值图

如果 $I$ 和 $p$ 都是常值图，那么：

- 方差为 0
- 协方差也为 0
- $a \approx 0$
- $b \approx \text{常值}$
- 输出应基本保持常值

#### 19.2 $p = I$

如果输入图就是导向图本身，那么输出通常应该非常接近原图，只会有轻微平滑。

#### 19.3 阶跃边缘

构造一张左黑右白的图，和均值滤波对比：

- 均值滤波会跨边缘模糊
- 导向滤波会更倾向于保持边缘位置

#### 19.4 改变 `eps`

固定 `r`，只改变 `eps`：

- 小 `eps`：边缘更锐，细节保留更多
- 大 `eps`：整体更平滑

#### 19.5 彩色导向图

让导向图三个通道结构不同，再观察输出是否能利用跨通道信息。  
这可以验证你是不是正确实现了矩阵求解，而不是误写成三个独立灰度滤波。

---

### 20. 一句话总结整套推导

导向滤波的数学本质可以压缩成一句话：

> 在每个局部窗口内，用导向图 $I$ 对输出 $q$ 做一次带正则的局部线性回归；再把所有重叠窗口得到的回归结果平均起来。

如果把这句话拆成公式，就是：

1. 局部模型  

$$
q_i = a_k I_i + b_k
$$

2. 局部最小二乘  

$$
E(a_k,b_k)
=
\sum_{i\in\omega_k}
\Bigl((a_k I_i + b_k - p_i)^2 + \epsilon a_k^2\Bigr)
$$

3. 闭式解  

$$
a_k = \frac{\operatorname{cov}_{I,p,k}}{\sigma_k^2 + \epsilon},
\qquad
b_k = \bar{p}_k - a_k\mu_k
$$

4. 重叠窗口平均  

$$
q_i = \bar{a}_i I_i + \bar{b}_i
$$

而在 C++ 代码里，这四步分别对应：

- 建模思想：体现在 `a`、`b` 这两个局部线性参数上
- 闭式解：体现在 `cov / (var + eps)` 和 `b = mean_p - a * mean_I`
- 重叠窗口平均：体现在 `mean_a = boxMean(a)`、`mean_b = boxMean(b)`
- 最终输出：体现在 `q = mean_a * I + mean_b`

到这里，导向滤波从数学原理到 C++ 实现的对应关系就完整闭合了。

---

### 21. 应用与工业锚点

#### 21.1 深度图、视差图与 mask 精修

导向滤波最常见的现实用途，不是“我想学一个公式”，而是：

- 一张结果图本身很噪，但边缘应该跟随另一张 guidance image
- 需要在边缘处保住结构，又不想付很高的计算代价

典型例子包括：

- 深度图 / 视差图 refinement
- 分割 mask、alpha matte、透射图的边缘细化
- 联合上采样：低分辨率结果借助高分辨率 guidance 恢复边界

#### 21.2 工业现实里的位置

在工业视觉链路里，导向滤波常见于“轻量、可解释、边缘敏感”的后处理阶段。

它特别适合：

- 需要确定性行为而不是黑箱语义修复
- 计算预算有限
- 已经有一张可信 guidance image

例如移动端影像后处理、机器人视觉预处理、传统图像增强链路里，它常作为一个局部线性、线性时间的边缘保持模块，而不是语义理解模块。

### 22. 当前推荐实践、常见误用与替代

#### 22.1 当前更推荐的用法

当前更稳的使用方式，是把导向滤波看成：

- 轻量 edge-aware refinement
- 联合上采样工具
- 局部线性模型驱动的经典视觉模块

也就是说，你应该在“边缘跟随 guidance、且需要快速可解释后处理”时优先考虑它。

#### 22.2 常见误用

不推荐把导向滤波误当成：

- 通用去噪万能器
- 语义级 refinement 模块
- 对任何任务都优于 learned post-processing 的通用替代

当问题已经高度依赖语义、上下文或全局结构时，更现代的 learned refinement、task-specific network 或更强先验模型通常更合适。导向滤波不是被“淘汰”了，而是它的适用边界非常明确。

### 23. 自测题

1. 为什么导向滤波的核心假设不是“局部常数”，而是“局部线性”？
2. `a_k` 和 `b_k` 各自承担什么角色？为什么需要对它们再做一次局部平均？
3. `eps` 变大时，输出为什么会更平滑？
4. 为什么说导向滤波适合联合上采样，而不是纯粹做普通均值平滑？
5. 如果一项任务需要理解语义边界而不只是图像梯度边界，为什么导向滤波往往不够？

### 24. 迁移入口

从这篇文档可以继续迁移到：

- Fast Guided Filter
- Joint Bilateral Filter
- Matting Laplacian
- moving least squares / ridge-like local regression 视角

迁移时最重要的不是公式名字，而是继续追问：

- guidance 到底约束了什么
- 被近似或被平滑的对象到底是什么
- 算法保留的是局部结构，还是更高层语义

### 25. 未解问题与继续深挖

后续仍值得单独深挖的点包括：

- 彩色导向滤波与实现复杂度之间更系统的工程取舍
- box filter 线性复杂度实现里的数值稳定性和边界处理细节
