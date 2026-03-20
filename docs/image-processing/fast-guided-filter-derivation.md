---
doc_id: image-processing-fast-guided-filter
title: 快速导向滤波的数学原理、公式推导与 C++ 对照
concept: fast_guided_filter
topic: image-processing
depth_mode: deep
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-03-20T17:09:33+08:00'
source_basis:
  - fast_guided_filter_arxiv2015_checked_2026_03_19
  - guided_image_filtering_eccv2010_checked_2026_03_19
  - guided_image_filtering_tpami2013_checked_2026_03_19
  - opencv_ximgproc_fast_guided_filter_docs_checked_2026_03_19
time_context: foundations_plus_current_practice_checked_2026_03_19
applicability: edge_aware_filtering_acceleration_modeling_and_implementation_review
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: upgraded_v1
related_docs:
  - docs/image-processing/guided-filter-derivation.md
  - docs/methodology/concept-document-template.md
open_questions:
  - 下采样倍率对误差与性能 tradeoff 的经验边界是否需要补充实验？
  - 是否需要补充快速导向滤波在工业图像链路中的典型使用场景？
---

# 快速导向滤波的数学原理、公式推导与 C++ 对照

## 1. 这份文档要帮你学会什么

这篇文档要帮你理解的不是“guided filter 又出了一个快版本”，而是：为什么 fast variant 能近似得足够好、它真正近似掉的对象是什么、什么时候应该接受这笔误差换速度。

读完后，你应该至少能做到：

- 说清 Fast Guided Filter 和普通 Guided Filter 的关系
- 知道它近似的是系数图，而不是最终输出图
- 用 `r`、`eps`、`s` 解释速度、误差和边缘伪影之间的 tradeoff
- 判断什么时候应该用低分辨率系数估计，什么时候应该回到全分辨率 guided filter
- 把论文推导、OpenCV 接口和真实延迟预算场景连起来看

## 2. 一句话结论 / 问题定义

**快速导向滤波不是新滤波器，而是 guided filter 的一条加速路径：它把局部线性模型的系数图放到低分辨率上估计，再上采样回原分辨率 guidance 上生成最终输出，从而用更低计算成本保住大部分边缘结构。**

它真正要解决的问题是：

- 全分辨率 guided filter 在高分辨率图像和视频上仍然可能太贵
- 但直接把低分辨率输出放大又会丢掉高频边缘
- 所以需要找到“什么中间量比最终结果更适合近似”

## 3. 对象边界与相邻概念

Fast Guided Filter 处理的是：

- 普通 guided filter 中最昂贵的局部统计和系数求解步骤
- 在低分辨率上更平滑、可近似的系数图
- 最后仍然依赖高分辨率 guidance 恢复输出结构

它不等于：

- 先降采样得到最终结果再硬上采样
- 完全不同的 edge-aware filter
- 对任何误差预算都合适的默认加速器

最该一起看的相邻概念是：

- 普通 Guided Filter
- coarse-to-fine / image pyramid
- joint bilateral approximation
- learned refinement

## 4. 核心结构

最小结构可以压成 7 个构件：

1. 原分辨率 guidance `I` 与输入 `p`
2. 下采样后的 `I^L`、`p^L`
3. 低分辨率窗口半径 `r^L`
4. 低分辨率局部线性系数 `a^L`、`b^L`
5. 低分辨率上的系数平滑
6. 上采样回原分辨率的系数图
7. 最终输出 `q = A I + B`

最关键的结构判断是：被近似的是“更平滑的系数图”，不是最终输出图。

## 5. 核心机制 / 主链路 / 因果链

Fast Guided Filter 的主链路可以压成 6 步：

1. 把 guidance 和输入按比例 `s` 下采样
2. 在低分辨率上沿用普通 guided filter 的局部线性模型
3. 计算低分辨率局部均值、方差、协方差，解出 `a^L`、`b^L`
4. 再对低分辨率系数图做局部平均
5. 把平滑后的系数图上采样回原分辨率
6. 用原分辨率 guidance 生成最终输出

为什么这条近似通常成立，关键是：系数图比原图和最终输出都更平滑，因此在更低分辨率上仍能保留主要结构信息。

## 6. 关键 tradeoff 与失败模式

Fast Guided Filter 用误差换速度。最大的 tradeoff 来自 `s`：

- `s` 越大，主要计算越少，延迟越低
- 但 `s` 越大，边缘附近的系数误差和伪影风险也越高

常见失败模式是：

- 直接把低分辨率最终输出 `q^L` 放大回原图，以为这就是 fast variant
- 误把 `s` 当“越大越好”的单调增益参数
- 在对边缘极度敏感的任务上还强行用很大的下采样倍率
- 忽略 guidance 本身质量，导致快速版把错误边缘也稳定放大

## 7. 应用场景

Fast Guided Filter 常见于：

- 高分辨率影像和视频上的 edge-aware 后处理
- 深度图、视差图、mask、transmission map 的高速 refinement
- 机器人 / SLAM / 实时视觉里有明确延迟预算的场景
- 需要在移动端或边缘设备上把 guided filter 成本压下去的链路

## 8. 工业 / 现实世界锚点

### 8.1 OpenCV ximgproc 的 `(Fast) Guided Filter` 接口

OpenCV ximgproc 文档把 `guidedFilter` 明确写成 “Simple one-line (Fast) Guided Filter call”，并暴露 `scale` 作为 Fast Guided Filter 的 subsample factor，这就是它进入现实工程工具链的直接锚点。

### 8.2 原始论文定位

Fast Guided Filter 的 arXiv 论文把方法直接定位为 guided filter 的 fast and accurate approximation，用来处理高分辨率图像和实时场景，而不是一个完全不同的新滤波器家族。

### 8.3 与普通 guided filter 的现实分工

现实工程里，普通 guided filter 更像质量基线，Fast Guided Filter 更像在延迟和算力受限场景下的可控近似版本。

## 9. 当前推荐实践、过时路径与替代

### 9.1 截至 2026-03-19 更推荐的实践

当前更稳的做法通常是：

- 在高分辨率、强延迟约束场景里优先考虑 fast variant
- 把 `s` 当成速度与误差的显式交换参数，而不是默认值
- 同时用耗时、边缘伪影和任务误差一起评估配置
- 当误差预算很小或边缘极其关键时，回到全分辨率 guided filter

### 9.2 过时路径与替代

下面这些路径通常已经不够稳：

- 直接把低分辨率最终输出放大回原图
- 只看速度，不看边缘误差和任务指标
- 在高语义依赖任务上，把 fast variant 当通用后处理万能解

更稳的替代是：

- 对边缘误差敏感的场景回退到全分辨率 guided filter
- 当任务真正依赖全局语义时，改用 learned refinement 或更强模型

## 10. 自测题 / 验证入口

1. Fast Guided Filter 近似掉的对象为什么是系数图，而不是最终输出？
2. 为什么 `s = 1` 会退化回普通 guided filter？
3. `s` 增大时，速度和边缘误差通常分别怎么变？
4. 为什么“低分辨率输出上采样”不是 Fast Guided Filter 的正确实现？
5. 什么时候你应该宁可回到全分辨率 guided filter，也不继续增大 `s`？

## 11. 迁移与关联模型

理解了这篇文档后，你应该能把模型迁移到：

- 普通 Guided Filter 的复杂度优化
- coarse-to-fine 近似设计
- image pyramid 与中间量近似
- 其他“中间表示比最终输出更适合近似”的快速算法

## 12. 未解问题与继续深挖

- 下采样倍率、画质误差和任务指标之间怎样建立更稳的经验规则？
- 在真实工业图像链路里，Fast Guided Filter 最典型的退化案例是什么？
- 哪些视觉任务上 fast variant 仍足够好，哪些任务应该直接退回全分辨率或换更强模型？

## 13. 参考资料

以下“当前实践”相关内容的核对日期均为 `2026-03-19`。

- Kaiming He, Jian Sun. Fast Guided Filter. arXiv 2015: https://arxiv.org/abs/1505.00996
- Kaiming He, Jian Sun, Xiaoou Tang. Guided Image Filtering. ECCV 2010 project page: https://people.csail.mit.edu/kaiming/eccv10/index.html
- Kaiming He, Jian Sun, Xiaoou Tang. Guided Image Filtering. TPAMI 2013: https://pubmed.ncbi.nlm.nih.gov/23599054/
- OpenCV ximgproc filters docs: https://docs.opencv.org/4.x/da/d17/group__ximgproc__filters.html

## 14. 详细推导、代码对应与补充说明

### 1. 快速导向滤波在做什么

快速导向滤波（Fast Guided Filter）不是一种完全不同的新滤波器，而是**普通导向滤波的快速近似实现**。

它要解决的问题和普通导向滤波完全一样：

- 已知一张导向图 $I$（guidance image）
- 已知一张待滤波图 $p$（filtering input）
- 希望得到输出图 $q$

目标也完全一样：

- 在平坦区域平滑噪声
- 在边缘处尽量保边
- 让输出结构跟随导向图 $I$

它和普通导向滤波的区别不在于模型变了，而在于：

- 普通导向滤波在**原分辨率**上计算所有局部统计量和系数图
- 快速导向滤波在**低分辨率**上估计系数图，再把这些系数图上采样回原分辨率

所以快速导向滤波的关键思想可以先记成一句话：

> 用低分辨率估计局部线性模型的系数，用高分辨率导向图生成最终输出。

---

### 2. 为什么普通导向滤波还能继续加速

普通导向滤波本身已经很快了。  
它的主要计算都是一系列盒式滤波（box filter），复杂度对像素数是线性的。

但在线性时间之外，仍然还有一个现实问题：

- 当图像很大时，像素总数 $N = H \times W$ 很大
- 即便每一步是 $O(N)$，常数次盒式滤波依然会花费明显时间

快速导向滤波的观察是：

- 最终输出公式是 $q = \bar{a}I + \bar{b}$
- 这里真正决定边缘位置的是高分辨率导向图 $I$
- 系数图 $\bar{a}, \bar{b}$ 本身已经是经过局部均值平滑后的量，通常比原图变化慢得多

既然系数图比较平滑，就没有必要总是在全分辨率上计算它们。  
于是可以：

1. 先把 $I$ 和 $p$ 缩小  
2. 在缩小后的图上求系数  
3. 再把这些系数放大回原图大小  
4. 最后仍然用原始高分辨率导向图 $I$ 生成输出

这就是快速导向滤波的近似来源。

---

### 3. 从普通导向滤波出发

为了理解快速版，先把普通导向滤波最核心的式子写出来。

在局部窗口 $\omega_k$ 内，普通导向滤波假设：

$$
q_i = a_k I_i + b_k,\qquad \forall i\in\omega_k
$$

通过局部最小二乘可得：

$$
a_k = \frac{\operatorname{cov}_{I,p,k}}{\sigma_k^2 + \epsilon}
$$

$$
b_k = \bar{p}_k - a_k \mu_k
$$

其中：

- $\mu_k$ 是窗口内导向图均值
- $\bar{p}_k$ 是窗口内输入图均值
- $\sigma_k^2$ 是导向图局部方差
- $\operatorname{cov}_{I,p,k}$ 是导向图与输入图的局部协方差

由于每个像素属于多个窗口，最终输出为：

$$
q_i = \bar{a}_i I_i + \bar{b}_i
$$

其中

$$
\bar{a}_i = \operatorname{mean}_{\omega_i}(a),\qquad
\bar{b}_i = \operatorname{mean}_{\omega_i}(b)
$$

快速导向滤波不会改动这个输出结构。  
它只改动**系数图是怎么得到的**。

---

### 4. 符号约定

为了和普通导向滤波区分，这里用上标 `$L$` 表示**低分辨率**量（low resolution）。

- $I$：原分辨率导向图
- $p$：原分辨率输入图
- $q$：原分辨率输出图
- $s$：下采样因子，通常为正整数
- $I^L = D_s(I)$：对 $I$ 下采样后的低分辨率导向图
- $p^L = D_s(p)$：对 $p$ 下采样后的低分辨率输入图
- $r$：原分辨率窗口半径
- $r^L$：低分辨率窗口半径

本文采用下面这个实现约定：

$$
r^L = \max(1,\operatorname{round}(r/s))
$$

此外：

- $U_s(\cdot)$ 表示把低分辨率图上采样回原分辨率
- $\omega_k^L$ 表示低分辨率图上的局部窗口
- $\mu_k^L$、$\bar{p}_k^L$、$(\sigma_k^L)^2$、$\operatorname{cov}_{I^L,p^L,k}$ 都是在低分辨率窗口上定义

---

### 5. 灰度快速导向滤波的数学推导

下面先讲最经典的**灰度导向**情形，即：

- $I$ 是单通道
- $p$ 是单通道

---

#### 5.1 先把图像缩小

定义下采样后的图像：

$$
I^L = D_s(I),\qquad p^L = D_s(p)
$$

这里 $D_s(\cdot)$ 可以是任意合理的缩小算子。  
快速导向滤波的核心并不依赖某一种唯一的下采样插值；本文在 OpenCV 示例代码中选择 `INTER_AREA` 做下采样，因为它是工程上更常见、也更稳定的缩小方式。

缩小之后，低分辨率图的像素总数大约是原来的：

$$
N^L \approx \frac{N}{s^2}
$$

其中 $N = H\times W$。

---

#### 5.2 在低分辨率上保留同样的局部线性模型

快速导向滤波并没有发明新的模型。  
它只是把普通导向滤波的局部线性模型搬到了低分辨率图上：

$$
q_i^L = a_k^L I_i^L + b_k^L,\qquad \forall i\in\omega_k^L
$$

其中：

- $a_k^L$ 是低分辨率窗口中的斜率
- $b_k^L$ 是低分辨率窗口中的截距

注意：这里的 $q^L$ 只是“如果我们在低分辨率上做普通导向滤波，将会得到的输出”。  
快速导向滤波最终并**不直接使用**这个 $q^L$ 作为最终答案，而是只保留系数图信息。

---

#### 5.3 低分辨率上的优化目标

和普通导向滤波完全一样，在低分辨率窗口 $\omega_k^L$ 内构造最小二乘目标：

$$
E(a_k^L,b_k^L)
=
\sum_{i\in\omega_k^L}
\Bigl(
(a_k^L I_i^L + b_k^L - p_i^L)^2
+
\epsilon (a_k^L)^2
\Bigr)
$$

这里的 $\epsilon$ 与普通导向滤波同义：

- 抑制斜率过大
- 提高数值稳定性
- 控制平滑强度

需要特别说明的是：  
快速导向滤波通常**不额外修改** $\epsilon$ 的定义。  
也就是说，快速版只是把局部统计的求解放到低分辨率上，而不是把正则项换成另一种形式。

---

#### 5.4 对 $b_k^L$ 求偏导

对目标函数关于 $b_k^L$ 求偏导并令其为零：

$$
\frac{\partial E}{\partial b_k^L}
=
2\sum_{i\in\omega_k^L}(a_k^L I_i^L + b_k^L - p_i^L)
= 0
$$

移项：

$$
a_k^L \sum_{i\in\omega_k^L} I_i^L
+
|\omega^L| b_k^L
-
\sum_{i\in\omega_k^L} p_i^L = 0
$$

两边除以 $|\omega^L|$，定义低分辨率局部均值

$$
\mu_k^L = \frac{1}{|\omega^L|}\sum_{i\in\omega_k^L} I_i^L,
\qquad
\bar{p}_k^L = \frac{1}{|\omega^L|}\sum_{i\in\omega_k^L} p_i^L
$$

得到

$$
a_k^L \mu_k^L + b_k^L - \bar{p}_k^L = 0
$$

因此

$$
b_k^L = \bar{p}_k^L - a_k^L \mu_k^L
$$

---

#### 5.5 把 $b_k^L$ 代回去

把上式代回局部模型：

$$
q_i^L = a_k^L I_i^L + \bar{p}_k^L - a_k^L \mu_k^L
= a_k^L(I_i^L - \mu_k^L) + \bar{p}_k^L
$$

于是残差项变成：

$$
a_k^L I_i^L + b_k^L - p_i^L
=
a_k^L(I_i^L - \mu_k^L) - (p_i^L - \bar{p}_k^L)
$$

目标函数可改写为：

$$
E(a_k^L)
=
\sum_{i\in\omega_k^L}
\Bigl(
a_k^L(I_i^L - \mu_k^L) - (p_i^L - \bar{p}_k^L)
\Bigr)^2
+
|\omega^L|\epsilon (a_k^L)^2
$$

---

#### 5.6 对 $a_k^L$ 求偏导

对上式求导：

$$
\frac{\partial E}{\partial a_k^L}
=
2\sum_{i\in\omega_k^L}
\Bigl(
a_k^L(I_i^L - \mu_k^L) - (p_i^L - \bar{p}_k^L)
\Bigr)
(I_i^L - \mu_k^L)
+
2|\omega^L|\epsilon a_k^L
=0
$$

整理后：

$$
a_k^L
\Biggl[
\sum_{i\in\omega_k^L}(I_i^L - \mu_k^L)^2 + |\omega^L|\epsilon
\Biggr]
=
\sum_{i\in\omega_k^L}(I_i^L - \mu_k^L)(p_i^L - \bar{p}_k^L)
$$

再定义低分辨率局部方差和协方差：

$$
(\sigma_k^L)^2
=
\frac{1}{|\omega^L|}
\sum_{i\in\omega_k^L}(I_i^L - \mu_k^L)^2
$$

$$
\operatorname{cov}_{I^L,p^L,k}
=
\frac{1}{|\omega^L|}
\sum_{i\in\omega_k^L}(I_i^L - \mu_k^L)(p_i^L - \bar{p}_k^L)
$$

得到低分辨率上的闭式解：

$$
a_k^L = \frac{\operatorname{cov}_{I^L,p^L,k}}{(\sigma_k^L)^2 + \epsilon}
$$

$$
b_k^L = \bar{p}_k^L - a_k^L \mu_k^L
$$

可以看出，推导与普通导向滤波完全一样，只是所有统计量都换成了低分辨率版本。

---

#### 5.7 用均值形式改写，方便实现

和普通导向滤波一样，方差与协方差可以改写成更适合代码的形式：

$$
\operatorname{cov}_{I^L,p^L}
=
\operatorname{mean}(I^L p^L)
-
\operatorname{mean}(I^L)\operatorname{mean}(p^L)
$$

$$
\operatorname{var}(I^L)
=
\operatorname{mean}\bigl((I^L)^2\bigr)
-
\operatorname{mean}(I^L)^2
$$

于是低分辨率上的系数图可写成：

$$
a^L = \frac{\operatorname{cov}_{I^L,p^L}}{\operatorname{var}(I^L) + \epsilon}
$$

$$
b^L = \operatorname{mean}(p^L) - a^L\operatorname{mean}(I^L)
$$

---

#### 5.8 低分辨率上，一个像素也会属于多个窗口

在低分辨率图上，同样每个像素也属于多个窗口，所以最终低分辨率输出本该是：

$$
q_i^L = \bar{a}_i^L I_i^L + \bar{b}_i^L
$$

其中

$$
\bar{a}^L = \mathcal{M}_{r^L}[a^L],\qquad
\bar{b}^L = \mathcal{M}_{r^L}[b^L]
$$

这里的 $\mathcal{M}_{r^L}$ 表示低分辨率窗口半径 $r^L$ 下的局部均值算子。

到这里为止，你看到的仍然只是“低分辨率普通导向滤波”。

---

### 6. 快速导向滤波真正的关键近似

快速导向滤波的真正关键点是：

- 它**不把低分辨率输出 $q^L$ 直接上采样**
- 它**只把低分辨率系数图 $\bar{a}^L,\bar{b}^L$ 上采样**

定义上采样后的系数图：

$$
A = U_s(\bar{a}^L),\qquad B = U_s(\bar{b}^L)
$$

然后最终输出使用原分辨率导向图 $I$：

$$
q_i = A_i I_i + B_i
$$

这就是快速导向滤波最核心的最终公式。

如果把它和普通导向滤波对比：

- 普通版：$q_i = \bar{a}_i I_i + \bar{b}_i$
- 快速版：$q_i = A_i I_i + B_i$

其中 $A,B$ 是由低分辨率系数图上采样得到的对 $\bar{a},\bar{b}$ 的近似。

---

### 7. 为什么不能直接把低分辨率输出 $q^L$ 放大

这是理解快速导向滤波最容易误解的地方。

如果直接做：

$$
q \approx U_s(q^L)
$$

那只是普通的“先缩小再滤波再放大”，高频边缘会直接丢失。

快速导向滤波不是这样做的。  
它做的是：

$$
q_i = A_i I_i + B_i
$$

这里：

- $A_i, B_i$ 是较平滑的系数图，可以在低分辨率估计
- $I_i$ 仍然是原分辨率导向图，所以高分辨率边缘位置还在

也就是说，快速导向滤波保留高分辨率边缘的原因在于：

- 结构定位来自原图 $I$
- 低分辨率只负责提供相对平滑的局部线性系数

这就是它“加速但不明显损伤视觉效果”的核心原因。

---

### 8. 用盒式滤波写成完整流程

定义低分辨率局部均值算子：

$$
\mathcal{M}_{r^L}[f](i)
=
\frac{1}{N_i^L}
\sum_{j\in\omega_i^L} f_j
$$

则灰度快速导向滤波可以压缩写成：

#### 8.1 下采样

$$
I^L = D_s(I),\qquad p^L = D_s(p)
$$

#### 8.2 低分辨率局部统计

$$
\mu_{I^L} = \mathcal{M}_{r^L}[I^L],\qquad
\mu_{p^L} = \mathcal{M}_{r^L}[p^L]
$$

$$
\operatorname{cov}_{I^L,p^L}
=
\mathcal{M}_{r^L}[I^L p^L] - \mu_{I^L}\mu_{p^L}
$$

$$
\operatorname{var}(I^L)
=
\mathcal{M}_{r^L}[(I^L)^2] - \mu_{I^L}^2
$$

#### 8.3 低分辨率系数图

$$
a^L = \frac{\operatorname{cov}_{I^L,p^L}}{\operatorname{var}(I^L) + \epsilon}
$$

$$
b^L = \mu_{p^L} - a^L \mu_{I^L}
$$

#### 8.4 低分辨率上对系数再均值

$$
\bar{a}^L = \mathcal{M}_{r^L}[a^L],\qquad
\bar{b}^L = \mathcal{M}_{r^L}[b^L]
$$

#### 8.5 上采样系数图

$$
A = U_s(\bar{a}^L),\qquad B = U_s(\bar{b}^L)
$$

#### 8.6 用原分辨率导向图生成输出

$$
q = A I + B
$$

注意：这里的乘法、减法、除法全部都是按像素逐点进行。

---

### 9. 为什么主耗时部分会近似缩到 $O(N/s^2)$

设原图像素数为

$$
N = H\times W
$$

下采样因子为 $s$ 后，低分辨率图像素数近似为：

$$
N^L \approx \frac{N}{s^2}
$$

普通导向滤波的主要计算量是若干次盒式滤波和逐点运算，它们都发生在原分辨率上，因此主项为：

$$
O(N)
$$

快速导向滤波中，最重的盒式滤波与局部统计计算都发生在低分辨率上，因此这部分主耗时变成：

$$
O\!\left(\frac{N}{s^2}\right)
$$

当然，最后仍有两类原分辨率操作：

- 把系数图上采样到原图大小
- 计算 $q = AI + B$

所以更完整地说，快速版的总复杂度可写成：

$$
O\!\left(\frac{N}{s^2}\right) + O(N)
$$

但由于：

- 盒式滤波是主耗时部分
- 上采样和最终逐点合成开销较小

也就是说，从严格的大 $O$ 记号看，它仍然包含原分辨率阶段；但在工程实践中，真正最重的局部统计部分会随 $1/s^2$ 显著下降。  
He 和 Sun 在 2015 年的说明中给出的结论是：当 $s=4$ 时，实际实现中常能得到超过 10 倍的加速，同时视觉退化很小。这里是对原文结果的概括，而不是新推导。

---

### 10. 彩色快速导向滤波的矩阵推导

现在把导向图扩展为三通道。  
输入图 $p$ 仍然先看作单通道，低分辨率导向图为：

$$
I_i^L =
\begin{bmatrix}
I_i^{L,(1)} \\
I_i^{L,(2)} \\
I_i^{L,(3)}
\end{bmatrix}
$$

低分辨率局部线性模型为：

$$
q_i^L = (a_k^L)^T I_i^L + b_k^L,\qquad \forall i\in\omega_k^L
$$

其中：

- $a_k^L \in \mathbb{R}^3$
- $b_k^L \in \mathbb{R}$

---

#### 10.1 低分辨率彩色情形的目标函数

$$
E(a_k^L,b_k^L)
=
\sum_{i\in\omega_k^L}
\Bigl(
\bigl((a_k^L)^T I_i^L + b_k^L - p_i^L\bigr)^2
+
\epsilon (a_k^L)^T a_k^L
\Bigr)
$$

对 $b_k^L$ 求导并令其为零，仍然得到：

$$
b_k^L = \bar{p}_k^L - (a_k^L)^T \mu_k^L
$$

其中

$$
\mu_k^L = \frac{1}{|\omega^L|}\sum_{i\in\omega_k^L} I_i^L
$$

是三维均值向量。

---

#### 10.2 对向量 $a_k^L$ 求解

令中心化变量

$$
\tilde{I}_i^L = I_i^L - \mu_k^L,\qquad
\tilde{p}_i^L = p_i^L - \bar{p}_k^L
$$

则目标函数变为

$$
E(a_k^L)
=
\sum_{i\in\omega_k^L}
\Bigl((a_k^L)^T\tilde{I}_i^L - \tilde{p}_i^L\Bigr)^2
+
|\omega^L|\epsilon (a_k^L)^T a_k^L
$$

对向量 $a_k^L$ 求梯度并令其为零：

$$
\frac{\partial E}{\partial a_k^L}
=
2\sum_{i\in\omega_k^L}
\tilde{I}_i^L\Bigl((\tilde{I}_i^L)^T a_k^L - \tilde{p}_i^L\Bigr)
+
2|\omega^L|\epsilon a_k^L
= 0
$$

整理得：

$$
\left(
\sum_{i\in\omega_k^L}\tilde{I}_i^L(\tilde{I}_i^L)^T
+
|\omega^L|\epsilon U
\right)a_k^L
=
\sum_{i\in\omega_k^L}\tilde{I}_i^L \tilde{p}_i^L
$$

两边除以 $|\omega^L|$ 后，定义低分辨率局部协方差矩阵

$$
\Sigma_k^L
=
\frac{1}{|\omega^L|}
\sum_{i\in\omega_k^L}
\tilde{I}_i^L(\tilde{I}_i^L)^T
$$

以及协方差向量

$$
\operatorname{cov}_{I^L,p^L,k}
=
\frac{1}{|\omega^L|}
\sum_{i\in\omega_k^L}
\tilde{I}_i^L \tilde{p}_i^L
$$

于是得到：

$$
a_k^L = (\Sigma_k^L + \epsilon U)^{-1}\operatorname{cov}_{I^L,p^L,k}
$$

$$
b_k^L = \bar{p}_k^L - (a_k^L)^T\mu_k^L
$$

随后依然要在低分辨率上对系数做均值，再上采样：

$$
\bar{a}^L = \mathcal{M}_{r^L}[a^L],\qquad
\bar{b}^L = \mathcal{M}_{r^L}[b^L]
$$

$$
A = U_s(\bar{a}^L),\qquad B = U_s(\bar{b}^L)
$$

最终在原分辨率上输出：

$$
q_i = A_i^T I_i + B_i
$$

这里的 $I_i$ 是原分辨率三通道导向向量。

---

### 11. 普通导向滤波和快速导向滤波逐公式对比

这一节专门把两者放在一起比较。

#### 11.1 模型层面

两者的局部线性模型完全相同：

$$
q_i = a_k I_i + b_k
$$

快速版并没有改变目标函数，也没有改变闭式解的形式。  
它改变的是**在哪个分辨率上估计系数**。

---

#### 11.2 灰度公式逐项对比

| 步骤 | 普通导向滤波 | 快速导向滤波 |
|---|---|---|
| 输入 | $I, p$ | $I, p$ |
| 下采样 | 无 | $I^L = D_s(I), p^L = D_s(p)$ |
| 半径 | $r$ | $r^L = \max(1,\operatorname{round}(r/s))$ |
| 均值 | $\mu_I = \mathcal{M}_r[I]$ | $\mu_{I^L} = \mathcal{M}_{r^L}[I^L]$ |
| 协方差 | $\operatorname{cov}_{I,p} = \mathcal{M}_r[Ip] - \mu_I\mu_p$ | $\operatorname{cov}_{I^L,p^L} = \mathcal{M}_{r^L}[I^Lp^L] - \mu_{I^L}\mu_{p^L}$ |
| 方差 | $\operatorname{var}(I) = \mathcal{M}_r[I^2] - \mu_I^2$ | $\operatorname{var}(I^L) = \mathcal{M}_{r^L}[(I^L)^2] - \mu_{I^L}^2$ |
| 系数 | $a = \dfrac{\operatorname{cov}_{I,p}}{\operatorname{var}(I)+\epsilon}$ | $a^L = \dfrac{\operatorname{cov}_{I^L,p^L}}{\operatorname{var}(I^L)+\epsilon}$ |
| 截距 | $b = \mu_p - a\mu_I$ | $b^L = \mu_{p^L} - a^L\mu_{I^L}$ |
| 系数平滑 | $\bar{a} = \mathcal{M}_r[a], \bar{b} = \mathcal{M}_r[b]$ | $\bar{a}^L = \mathcal{M}_{r^L}[a^L], \bar{b}^L = \mathcal{M}_{r^L}[b^L]$ |
| 恢复原尺寸 | 无 | $A = U_s(\bar{a}^L), B = U_s(\bar{b}^L)$ |
| 最终输出 | $q = \bar{a}I + \bar{b}$ | $q = AI + B$ |

---

#### 11.3 真正被近似的是哪一部分

快速导向滤波近似的不是：

- 目标函数
- 闭式解公式
- 最终输出结构

它近似的是：

- 局部统计量在低分辨率上计算
- 系数图在低分辨率上估计
- 再通过上采样近似原分辨率系数图

所以误差主要来自：

- 下采样丢失的局部高频统计
- 上采样时的插值误差
- 当 $s$ 太大时，低分辨率窗口不足以表达原图局部结构

---

#### 11.4 为什么快速版通常还能保住边缘

因为最后一步仍然是：

$$
q_i = A_i I_i + B_i
$$

只要：

- $A_i$、$B_i$ 的近似不至于太差
- $I_i$ 仍然是原分辨率导向图

那么边缘定位仍然由高分辨率导向图决定。  
所以快速版通常只是在系数精度上有轻微近似，而不是完全丢掉边缘结构。

---

### 12. 对应到 OpenCV C++：完整代码

下面给出一份 OpenCV 风格的完整实现。  
它故意按照“公式顺序”组织，而不是追求最短代码。

```cpp
#include <opencv2/opencv.hpp>
#include <algorithm>
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

cv::Mat downsample(const cv::Mat& src, int s) {
    if (s == 1) {
        return src.clone();
    }
    cv::Mat dst;
    cv::resize(src, dst, cv::Size(), 1.0 / s, 1.0 / s, cv::INTER_AREA);
    return dst;
}

cv::Mat upsampleTo(const cv::Mat& src, cv::Size size) {
    cv::Mat dst;
    cv::resize(src, dst, size, 0.0, 0.0, cv::INTER_LINEAR);
    return dst;
}

int scaledRadius(int r, int s) {
    return std::max(1, cvRound(static_cast<double>(r) / s));
}

}  // namespace

cv::Mat fastGuidedFilterGray(const cv::Mat& I, const cv::Mat& p, int r, double eps, int s) {
    CV_Assert(I.type() == CV_32F);
    CV_Assert(p.type() == CV_32F);
    CV_Assert(I.size() == p.size());
    CV_Assert(r >= 1);
    CV_Assert(s >= 1);

    const cv::Mat I_sub = downsample(I, s);
    const cv::Mat p_sub = downsample(p, s);
    const int r_sub = scaledRadius(r, s);

    const cv::Mat N_sub = computeWindowPixelCount(I_sub.size(), r_sub);

    const cv::Mat mean_I_sub  = boxMean(I_sub, N_sub, r_sub);
    const cv::Mat mean_p_sub  = boxMean(p_sub, N_sub, r_sub);
    const cv::Mat mean_Ip_sub = boxMean(I_sub.mul(p_sub), N_sub, r_sub);
    const cv::Mat mean_II_sub = boxMean(I_sub.mul(I_sub), N_sub, r_sub);

    const cv::Mat cov_Ip_sub = mean_Ip_sub - mean_I_sub.mul(mean_p_sub);
    const cv::Mat var_I_sub  = mean_II_sub - mean_I_sub.mul(mean_I_sub);

    const cv::Mat a_sub = cov_Ip_sub / (var_I_sub + eps);
    const cv::Mat b_sub = mean_p_sub - a_sub.mul(mean_I_sub);

    const cv::Mat mean_a_sub = boxMean(a_sub, N_sub, r_sub);
    const cv::Mat mean_b_sub = boxMean(b_sub, N_sub, r_sub);

    const cv::Mat A = upsampleTo(mean_a_sub, I.size());
    const cv::Mat B = upsampleTo(mean_b_sub, I.size());

    return A.mul(I) + B;
}

cv::Mat fastGuidedFilterColor(const cv::Mat& I, const cv::Mat& p, int r, double eps, int s) {
    CV_Assert(I.type() == CV_32FC3);
    CV_Assert(p.type() == CV_32F);
    CV_Assert(I.size() == p.size());
    CV_Assert(r >= 1);
    CV_Assert(s >= 1);

    const float epsf = static_cast<float>(eps);
    const cv::Mat I_sub = downsample(I, s);
    const cv::Mat p_sub = downsample(p, s);
    const int r_sub = scaledRadius(r, s);

    std::vector<cv::Mat> ch(3);
    cv::split(I_sub, ch);
    const cv::Mat& I0 = ch[0];
    const cv::Mat& I1 = ch[1];
    const cv::Mat& I2 = ch[2];

    const cv::Mat N_sub = computeWindowPixelCount(I_sub.size(), r_sub);

    const cv::Mat mean_I0 = boxMean(I0, N_sub, r_sub);
    const cv::Mat mean_I1 = boxMean(I1, N_sub, r_sub);
    const cv::Mat mean_I2 = boxMean(I2, N_sub, r_sub);
    const cv::Mat mean_p  = boxMean(p_sub, N_sub, r_sub);

    const cv::Mat mean_I0p = boxMean(I0.mul(p_sub), N_sub, r_sub);
    const cv::Mat mean_I1p = boxMean(I1.mul(p_sub), N_sub, r_sub);
    const cv::Mat mean_I2p = boxMean(I2.mul(p_sub), N_sub, r_sub);

    const cv::Mat cov_Ip0 = mean_I0p - mean_I0.mul(mean_p);
    const cv::Mat cov_Ip1 = mean_I1p - mean_I1.mul(mean_p);
    const cv::Mat cov_Ip2 = mean_I2p - mean_I2.mul(mean_p);

    const cv::Mat var_00 = boxMean(I0.mul(I0), N_sub, r_sub) - mean_I0.mul(mean_I0);
    const cv::Mat var_01 = boxMean(I0.mul(I1), N_sub, r_sub) - mean_I0.mul(mean_I1);
    const cv::Mat var_02 = boxMean(I0.mul(I2), N_sub, r_sub) - mean_I0.mul(mean_I2);
    const cv::Mat var_11 = boxMean(I1.mul(I1), N_sub, r_sub) - mean_I1.mul(mean_I1);
    const cv::Mat var_12 = boxMean(I1.mul(I2), N_sub, r_sub) - mean_I1.mul(mean_I2);
    const cv::Mat var_22 = boxMean(I2.mul(I2), N_sub, r_sub) - mean_I2.mul(mean_I2);

    cv::Mat a0(I_sub.size(), CV_32F);
    cv::Mat a1(I_sub.size(), CV_32F);
    cv::Mat a2(I_sub.size(), CV_32F);
    cv::Mat b(I_sub.size(), CV_32F);

    for (int y = 0; y < I_sub.rows; ++y) {
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

        for (int x = 0; x < I_sub.cols; ++x) {
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

    const cv::Mat mean_a0_sub = boxMean(a0, N_sub, r_sub);
    const cv::Mat mean_a1_sub = boxMean(a1, N_sub, r_sub);
    const cv::Mat mean_a2_sub = boxMean(a2, N_sub, r_sub);
    const cv::Mat mean_b_sub  = boxMean(b,  N_sub, r_sub);

    const cv::Mat A0 = upsampleTo(mean_a0_sub, p.size());
    const cv::Mat A1 = upsampleTo(mean_a1_sub, p.size());
    const cv::Mat A2 = upsampleTo(mean_a2_sub, p.size());
    const cv::Mat B  = upsampleTo(mean_b_sub,  p.size());

    std::vector<cv::Mat> fullCh(3);
    cv::split(I, fullCh);

    return A0.mul(fullCh[0]) + A1.mul(fullCh[1]) + A2.mul(fullCh[2]) + B;
}
```

---

### 13. 灰度快速导向滤波公式和 C++ 代码的一一对应

| 数学公式 | 数学含义 | C++ 代码 |
|---|---|---|
| $I^L = D_s(I)$ | 低分辨率导向图 | `const cv::Mat I_sub = downsample(I, s);` |
| $p^L = D_s(p)$ | 低分辨率输入图 | `const cv::Mat p_sub = downsample(p, s);` |
| $r^L = \max(1,\operatorname{round}(r/s))$ | 缩放后的窗口半径 | `const int r_sub = scaledRadius(r, s);` |
| $N^L_i = \sum_{j\in\omega_i^L} 1$ | 低分辨率窗口像素数 | `const cv::Mat N_sub = computeWindowPixelCount(I_sub.size(), r_sub);` |
| $\mu_{I^L} = \mathcal{M}_{r^L}[I^L]$ | 低分辨率导向图均值 | `mean_I_sub` |
| $\mu_{p^L} = \mathcal{M}_{r^L}[p^L]$ | 低分辨率输入图均值 | `mean_p_sub` |
| $\operatorname{mean}(I^Lp^L)$ | 低分辨率乘积均值 | `mean_Ip_sub` |
| $\operatorname{mean}((I^L)^2)$ | 低分辨率平方均值 | `mean_II_sub` |
| $\operatorname{cov}_{I^L,p^L} = \operatorname{mean}(I^Lp^L) - \mu_{I^L}\mu_{p^L}$ | 低分辨率协方差 | `cov_Ip_sub` |
| $\operatorname{var}(I^L) = \operatorname{mean}((I^L)^2) - \mu_{I^L}^2$ | 低分辨率方差 | `var_I_sub` |
| $a^L = \dfrac{\operatorname{cov}_{I^L,p^L}}{\operatorname{var}(I^L)+\epsilon}$ | 低分辨率斜率图 | `a_sub` |
| $b^L = \mu_{p^L} - a^L \mu_{I^L}$ | 低分辨率截距图 | `b_sub` |
| $\bar{a}^L = \mathcal{M}_{r^L}[a^L]$ | 低分辨率平滑系数图 | `mean_a_sub` |
| $\bar{b}^L = \mathcal{M}_{r^L}[b^L]$ | 低分辨率平滑截距图 | `mean_b_sub` |
| $A = U_s(\bar{a}^L)$ | 上采样后的斜率系数图 | `const cv::Mat A = upsampleTo(mean_a_sub, I.size());` |
| $B = U_s(\bar{b}^L)$ | 上采样后的截距系数图 | `const cv::Mat B = upsampleTo(mean_b_sub, I.size());` |
| $q = AI + B$ | 最终输出 | `return A.mul(I) + B;` |

需要特别注意：

- 快速版真正上采样的是 `mean_a_sub` 和 `mean_b_sub`
- 最终输出仍然使用原分辨率导向图 `I`
- 这正是快速版与“先缩小后放大输出”的本质区别

---

### 14. 彩色快速导向滤波公式和 C++ 代码的一一对应

| 数学公式 | 数学含义 | C++ 代码 |
|---|---|---|
| $I^L = D_s(I)$ | 低分辨率三通道导向图 | `const cv::Mat I_sub = downsample(I, s);` |
| $p^L = D_s(p)$ | 低分辨率输入图 | `const cv::Mat p_sub = downsample(p, s);` |
| $\mu_{I^{L,(0)}}$、$\mu_{I^{L,(1)}}$、$\mu_{I^{L,(2)}}$ | 低分辨率三个通道的局部均值 | `mean_I0`, `mean_I1`, `mean_I2` |
| $\mu_{p^L}$ | 低分辨率输入图均值 | `mean_p` |
| $\operatorname{cov}(I^{L,(m)}, p^L)$ | 第 $m$ 个通道与输入图的协方差 | `cov_Ip0`, `cov_Ip1`, `cov_Ip2` |
| $\Sigma^L + \epsilon U$ | 加正则后的低分辨率协方差矩阵 | `cv::Matx33f Sigma(...)` |
| $a^L = (\Sigma^L + \epsilon U)^{-1}\operatorname{cov}_{I^L,p^L}$ | 低分辨率三维斜率 | `cv::Vec3f ak = Sigma.inv(...) * cov_Ip;` |
| $b^L = \mu_{p^L} - (a^L)^T\mu_{I^L}$ | 低分辨率截距 | `pb[x] = mp[x] - ...;` |
| $\bar{a}^{L,(c)} = \mathcal{M}_{r^L}[a^{L,(c)}]$ | 每个通道的平滑系数图 | `mean_a0_sub`, `mean_a1_sub`, `mean_a2_sub` |
| $\bar{b}^L = \mathcal{M}_{r^L}[b^L]$ | 平滑后的截距图 | `mean_b_sub` |
| $A^{(c)} = U_s(\bar{a}^{L,(c)})$ | 上采样后的每通道系数图 | `A0`, `A1`, `A2` |
| $B = U_s(\bar{b}^L)$ | 上采样后的截距图 | `B` |
| $q = A_0 I_0 + A_1 I_1 + A_2 I_2 + B$ | 最终输出 | `return A0.mul(fullCh[0]) + ... + B;` |

彩色情形下，快速版和普通版一样，都不是把三个通道独立做灰度滤波。  
它仍然通过 $3\times 3$ 协方差矩阵联合建模三个通道，只不过这一切发生在低分辨率上。

---

### 15. 每段代码背后的数学意义

#### 15.1 下采样

```cpp
const cv::Mat I_sub = downsample(I, s);
const cv::Mat p_sub = downsample(p, s);
const int r_sub = scaledRadius(r, s);
```

对应：

$$
I^L = D_s(I),\qquad p^L = D_s(p),\qquad r^L = \max(1,\operatorname{round}(r/s))
$$

这一步决定了快速版的计算域。

---

#### 15.2 低分辨率局部统计

```cpp
const cv::Mat mean_I_sub  = boxMean(I_sub, N_sub, r_sub);
const cv::Mat mean_p_sub  = boxMean(p_sub, N_sub, r_sub);
const cv::Mat mean_Ip_sub = boxMean(I_sub.mul(p_sub), N_sub, r_sub);
const cv::Mat mean_II_sub = boxMean(I_sub.mul(I_sub), N_sub, r_sub);
```

对应：

$$
\mu_{I^L},\qquad \mu_{p^L},\qquad \operatorname{mean}(I^Lp^L),\qquad \operatorname{mean}((I^L)^2)
$$

这一步就是在低分辨率上构造方差和协方差。

---

#### 15.3 低分辨率系数图

```cpp
const cv::Mat cov_Ip_sub = mean_Ip_sub - mean_I_sub.mul(mean_p_sub);
const cv::Mat var_I_sub  = mean_II_sub - mean_I_sub.mul(mean_I_sub);

const cv::Mat a_sub = cov_Ip_sub / (var_I_sub + eps);
const cv::Mat b_sub = mean_p_sub - a_sub.mul(mean_I_sub);
```

对应：

$$
\operatorname{cov}_{I^L,p^L}
=
\operatorname{mean}(I^Lp^L) - \mu_{I^L}\mu_{p^L}
$$

$$
\operatorname{var}(I^L)
=
\operatorname{mean}((I^L)^2) - \mu_{I^L}^2
$$

$$
a^L = \frac{\operatorname{cov}_{I^L,p^L}}{\operatorname{var}(I^L)+\epsilon},
\qquad
b^L = \mu_{p^L} - a^L\mu_{I^L}
$$

---

#### 15.4 低分辨率上平滑系数图

```cpp
const cv::Mat mean_a_sub = boxMean(a_sub, N_sub, r_sub);
const cv::Mat mean_b_sub = boxMean(b_sub, N_sub, r_sub);
```

对应：

$$
\bar{a}^L = \mathcal{M}_{r^L}[a^L],\qquad
\bar{b}^L = \mathcal{M}_{r^L}[b^L]
$$

这是最终要被上采样的量。

---

#### 15.5 把平滑后的系数图上采样回原图

```cpp
const cv::Mat A = upsampleTo(mean_a_sub, I.size());
const cv::Mat B = upsampleTo(mean_b_sub, I.size());
```

对应：

$$
A = U_s(\bar{a}^L),\qquad B = U_s(\bar{b}^L)
$$

这里用的是 `INTER_LINEAR`，也就是双线性插值。

---

#### 15.6 最终输出

```cpp
return A.mul(I) + B;
```

对应：

$$
q = AI + B
$$

这一步再次说明：  
快速导向滤波最终依然在原分辨率导向图上完成结构恢复。

---

### 16. 参数 `r`、`eps`、`s` 的作用

#### 16.1 `r`：窗口空间范围

- `r` 越大，局部统计来自更大的范围，平滑更强
- `r` 越小，结果更贴近局部细节

快速版中真正参与低分辨率统计的是：

$$
r^L = \max(1,\operatorname{round}(r/s))
$$

所以 `r` 和 `s` 不是独立的。

---

#### 16.2 `eps`：正则强度

- `eps` 越小，线性系数更容易变大，输出更强地跟随导向图
- `eps` 越大，线性系数更容易被压小，结果更平滑

快速版通常不对 `eps` 额外缩放。  
因为它控制的是亮度域上的拟合稳定性，而不是空间尺度本身。

---

#### 16.3 `s`：速度和精度的交换参数

- `s = 1`：退化成普通导向滤波
- `s = 2`：通常能明显加速，误差仍很小
- `s = 4`：常是一个很实用的工程折中
- `s` 太大：低分辨率统计过粗，系数图近似误差会明显增加

因此 `s` 控制的是：

- 速度
- 系数图近似精度
- 低分辨率窗口对原图局部结构的表达能力

---

### 17. 适合快速导向滤波的场景

快速版尤其适合：

- 图像分辨率很高
- 窗口半径 $r$ 较大
- 任务更看重实时性或吞吐量
- 对普通导向滤波的“几乎相同视觉效果”已经足够

典型场景包括：

- 大图边缘保持平滑
- 细节增强前的基底层提取
- 实时图像编辑
- 大分辨率联合滤波或引导式增强

如果你追求的是：

- 与普通导向滤波尽可能严格一致
- 对微小局部结构误差特别敏感

那就应该减小 `s`，甚至直接使用普通导向滤波。

---

### 18. 值得验证的测试场景

#### 18.1 `s = 1`

这时快速导向滤波应退化为普通导向滤波，结果应接近一致。  
这是最重要的正确性检查。

#### 18.2 常值图

如果 $I$ 和 $p$ 都是常值图，那么输出应保持常值。  
这能检验：

- 下采样
- 上采样
- 系数求解

是否引入了偏移。

#### 18.3 `p = I`

当输入图本身就是导向图时，输出应接近原图，只会有轻微平滑。  
同时可以比较不同 `s` 下与普通版的偏差。

#### 18.4 阶跃边缘图

构造左黑右白的阶跃图：

- 普通均值滤波会跨边缘模糊
- 普通导向滤波会保边
- 快速导向滤波应尽量接近普通导向滤波

#### 18.5 固定 `r` 和 `eps`，改变 `s`

这可以观察：

- `s` 增大时速度如何提高
- `s` 增大时误差如何增加

这是理解快速版价值的核心实验。

#### 18.6 彩色导向图

给导向图三个通道不同结构，验证彩色快速版确实利用了跨通道协方差，而不是退化为三个独立灰度滤波。

---

### 19. 一句话总结整套方法

快速导向滤波的本质可以压缩成一句话：

> 在低分辨率上求导向滤波的局部线性系数图，再把这些系数图上采样回原分辨率，并用原分辨率导向图生成最终输出。

如果拆成公式，就是：

1. 下采样

$$
I^L = D_s(I),\qquad p^L = D_s(p)
$$

2. 低分辨率上求系数

$$
a^L = \frac{\operatorname{cov}_{I^L,p^L}}{\operatorname{var}(I^L)+\epsilon},
\qquad
b^L = \mu_{p^L} - a^L\mu_{I^L}
$$

3. 对系数再做局部均值

$$
\bar{a}^L = \mathcal{M}_{r^L}[a^L],\qquad
\bar{b}^L = \mathcal{M}_{r^L}[b^L]
$$

4. 上采样回原图

$$
A = U_s(\bar{a}^L),\qquad B = U_s(\bar{b}^L)
$$

5. 用原分辨率导向图输出

$$
q = AI + B
$$

真正的关键不是“把结果缩小再放大”，而是“把平滑的系数图缩小求解，再回到原分辨率导向图上恢复结构”。

---

### 20. 参考资料

- Kaiming He, Jian Sun. *Fast Guided Filter*. arXiv, 2015: https://arxiv.org/abs/1505.00996
- Kaiming He, Jian Sun, Xiaoou Tang. *Guided Image Filtering*. ECCV 2010: https://doi.org/10.1007/978-3-642-15549-9_1
- Kaiming He, Jian Sun, Xiaoou Tang. *Guided Image Filtering*. TPAMI 2013: https://pubmed.ncbi.nlm.nih.gov/23599054/

这三篇资料之间的关系是：

- ECCV 2010 / TPAMI 2013 给出了导向滤波的完整理论
- arXiv 2015 的 Fast Guided Filter 给出了快速近似版本和复杂度分析

---

### 21. 工业 / 现实世界锚点

#### 21.1 移动端影像与视频后处理

快速导向滤波最常见的现实锚点，是高分辨率图像或视频上的边缘保持后处理。

这类场景通常有两个约束同时存在：

- 结果必须跟着高分辨率 guidance 保边
- 延迟预算又不允许总在全分辨率上做完整局部统计

因此它特别适合：

- 移动端影像增强
- 实时视频滤波
- 深度图、透射图、mask 的高速 refinement

#### 21.2 机器人 / SLAM / 实时视觉链路

在机器人和实时视觉场景里，快速版的意义更直接：你不是不想要高质量，而是必须在延迟和算力预算内做出“足够好”的边缘保持 refinement。

这时真正要问的是：

- 系数图是否足够平滑，值得低分辨率估计
- 速度提升是否显著
- 近似误差是否会在关键边缘处造成不可接受伪影

### 22. 当前推荐实践、过时路径与替代

#### 22.1 当前更推荐的实践

当前更稳的用法通常是：

- 当分辨率很高、延迟预算很紧时，优先考虑 fast variant
- 把 `s` 看成速度与误差的显式交换参数，而不是“越大越好”
- 同时用误差、边缘伪影和速度收益三件事评估配置，而不是只看耗时

#### 22.2 已经不推荐的近似方式

有一种看似自然、但实际上不推荐的旧近似：直接在低分辨率上得到最终输出 `q^L`，再把 `q^L` 放大回原图。

这条路的问题是：

- 低分辨率输出本身已经丢掉了不少高频边缘结构
- 再怎么上采样也很难把这些结构凭空补回来

Fast Guided Filter 的关键替代路径正是：

- 不是低分辨率求最终输出
- 而是低分辨率求更平滑、更适合近似的系数图
- 再回到原分辨率 guidance 上生成最终结果

当任务对边缘极度敏感、而误差预算又很小的时候，更稳的替代不是盲目继续增大 `s`，而是回到全分辨率 guided filter，或在更强语义任务中使用 learned refinement。

### 23. 自测题

1. 快速导向滤波真正近似掉的对象是什么，为什么不是直接近似最终输出？
2. 为什么系数图比原图更适合在低分辨率上估计？
3. `s` 增大后，速度和边缘误差通常会怎么变化？
4. 为什么 `s = 1` 时，快速版会退化回普通导向滤波？
5. 在什么情况下你应该宁可回到全分辨率 guided filter，而不是继续调大 `s`？

### 24. 迁移入口

理解了快速导向滤波后，你可以把同一套思路迁移到更多“快速近似”问题：

- 先问真正昂贵的部分在哪里
- 再问哪些中间量比最终结果更平滑、更适合近似
- 最后问哪一部分必须保留在高分辨率或高精度上

这套思路不只适用于 guided filter，也适用于很多图像金字塔、低秩近似和 coarse-to-fine 设计。

### 25. 未解问题与继续深挖

后续仍值得继续深挖的点包括：

- 下采样倍率、误差和速度收益之间更系统的经验边界
- 快速导向滤波在真实工业图像链路里的典型使用模式与退化案例
