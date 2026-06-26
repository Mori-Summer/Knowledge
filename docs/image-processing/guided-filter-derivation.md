---
doc_id: image-processing-guided-filter
title: 导向滤波与快速导向滤波：局部线性模型、低分辨率系数近似与工程选型
concept: guided_filter
topic: image-processing
depth_mode: deep
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-06-22T09:32:40+08:00'
source_basis:
  - guided_image_filtering_eccv2010_checked_2026_03_19
  - guided_image_filtering_tpami2013_checked_2026_03_19
  - fast_guided_filter_arxiv2015_checked_2026_03_19
  - opencv_ximgproc_guided_filter_docs_checked_2026_03_19
  - opencv_ximgproc_fast_guided_filter_docs_checked_2026_03_19
  - matlab_imguidedfilter_docs_checked_2026_03_19
  - fast_guided_filter_doc_merged_2026_06_22
  - docs_folder_consolidation_progress_2026_06_23
time_context: foundations_plus_current_practice_checked_2026_03_19_and_guided_filter_docs_consolidated_2026_06_23
applicability: edge_aware_filtering_modeling_derivation_acceleration_and_implementation_review
prompt_version: concept_generation_prompt_v1
template_version: concept_doc_v1
quality_status: consolidated_v1
related_docs:
  - docs/methodology/document-generation-methodology.md
open_questions:
  - 彩色导向滤波在不同数值尺度下的稳定性阈值是否需要单独实验记录？
  - Fast Guided Filter 的下采样倍率、画质误差和任务指标之间是否需要建立项目内经验表？
---

# 导向滤波与快速导向滤波：局部线性模型、低分辨率系数近似与工程选型

## 1. 规范定位

本文是 `image-processing` 下关于 Guided Filter / Fast Guided Filter 的唯一入口。

它同时覆盖：

- 普通导向滤波的局部线性模型、闭式解、复杂度与实现检查点
- 彩色 guidance 的矩阵形式
- Fast Guided Filter 的低分辨率系数近似、参数 `s` 与失效边界
- OpenCV / MATLAB 工程接口背后的概念映射

不要再为 Fast Guided Filter 单独维护一份平行文档。快速版不是新滤波器族，而是普通 guided filter 的加速近似路径。

## 2. 一句话结论

**导向滤波是一种 edge-aware 局部线性滤波：它假设输出 `q` 在每个局部窗口内可以写成 guidance `I` 的线性变换，从而在平坦区域平滑、在 guidance 边缘附近避免跨边界混合。Fast Guided Filter 则把最重的局部统计和系数求解放到低分辨率上，再把系数图上采样回原分辨率，用原分辨率 guidance 生成输出。**

核心判断：

- 普通 guided filter 近似的是局部输出和 guidance 的线性关系。
- Fast Guided Filter 近似的是更平滑的系数图，不是最终输出图。
- 如果任务真正依赖语义边界，而 guidance 只有低层梯度信息，guided filter 不会凭空补出语义结构。

## 3. 对象边界

导向滤波处理的是：

- guidance image `I`
- filtering input `p`
- output `q`
- 局部窗口 `omega_k`
- 由最小二乘得到的局部线性系数 `a_k`、`b_k`

它不等于：

- 普通均值滤波或 Gaussian smoothing
- 语义级 refinement 模块
- 三通道分别独立运行的灰度滤波
- 低分辨率输出直接上采样

相邻模型：

- joint bilateral filter
- matting Laplacian
- moving least squares / ridge-like local regression
- coarse-to-fine approximation
- learned refinement

## 4. 符号与参数

| 符号 | 含义 | 实现约束 |
|---|---|---|
| `I` | guidance image | 可以是灰度或彩色 |
| `p` | filtering input | 常见为单通道 refinement target |
| `q` | output | 与 `p` 同尺寸 |
| `r` | 原分辨率窗口半径 | 控制空间统计范围 |
| `eps` | 正则强度 | 必须和像素值尺度匹配 |
| `s` | Fast Guided Filter 的下采样因子 | `s = 1` 退化为普通 guided filter |
| `D_s` | 下采样算子 | 工程中常用 area / pyramid 下采样 |
| `U_s` | 上采样算子 | 工程中常用 bilinear 上采样 |
| `M_r[f]` | 半径 `r` 下的局部均值 | 通常由 box filter 实现 |

Fast Guided Filter 中低分辨率窗口半径常按下式处理：

$$
r^L = \max(1,\operatorname{round}(r/s))
$$

`eps` 控制亮度域上的拟合稳定性，通常不因为 `s` 额外缩放；`r` 控制空间尺度，快速版中需要映射到 `r^L`。

## 5. 普通灰度 Guided Filter

### 5.1 局部线性模型

在窗口 `omega_k` 内，假设：

$$
q_i = a_k I_i + b_k,\qquad \forall i\in\omega_k
$$

其中 `a_k` 和 `b_k` 在同一个窗口内为常数，不同窗口有不同系数。

### 5.2 最小二乘目标

$$
E(a_k,b_k)
=
\sum_{i\in\omega_k}
\left((a_k I_i + b_k - p_i)^2 + \epsilon a_k^2\right)
$$

数据项要求局部线性模型拟合 `p`；正则项抑制过大的 `a_k`，避免低方差区域数值不稳定。

### 5.3 闭式解

定义窗口内均值、方差、协方差：

$$
\mu_k = \operatorname{mean}_{\omega_k}(I),\qquad
\bar{p}_k = \operatorname{mean}_{\omega_k}(p)
$$

$$
\sigma_k^2 = \operatorname{mean}_{\omega_k}(I^2) - \mu_k^2
$$

$$
\operatorname{cov}_{I,p,k}
=
\operatorname{mean}_{\omega_k}(Ip) - \mu_k\bar{p}_k
$$

则：

$$
a_k = \frac{\operatorname{cov}_{I,p,k}}{\sigma_k^2 + \epsilon}
$$

$$
b_k = \bar{p}_k - a_k\mu_k
$$

### 5.4 重叠窗口平均

一个像素属于多个窗口，因此最终不是直接使用单个 `a_k`、`b_k`，而是先平均系数：

$$
\bar{a}_i = \operatorname{mean}_{k:i\in\omega_k}(a_k),\qquad
\bar{b}_i = \operatorname{mean}_{k:i\in\omega_k}(b_k)
$$

最终输出：

$$
q_i = \bar{a}_i I_i + \bar{b}_i
$$

这条输出式是实现里的核心闭环。

## 6. 彩色 Guidance

彩色 guidance 把标量 `I_i` 扩展为三维向量：

$$
I_i =
\begin{bmatrix}
I_i^{(1)} \\
I_i^{(2)} \\
I_i^{(3)}
\end{bmatrix}
$$

局部模型变为：

$$
q_i = a_k^T I_i + b_k
$$

其中 `a_k` 是三维向量。闭式解为：

$$
a_k = (\Sigma_k + \epsilon U)^{-1}\operatorname{cov}_{I,p,k}
$$

$$
b_k = \bar{p}_k - a_k^T\mu_k
$$

`Sigma_k` 是三通道 guidance 的局部协方差矩阵，`U` 是单位矩阵。

实现约束：

- 彩色 guided filter 不是对 RGB 三个通道分别运行灰度 guided filter。
- 每个像素位置需要解一个 `3 x 3` 正则化线性系统。
- `Sigma + eps * U` 的数值稳定性取决于数据尺度、`eps`、边界策略和浮点精度。

## 7. Fast Guided Filter

### 7.1 核心近似

Fast Guided Filter 保留普通 guided filter 的局部线性模型和最终输出结构，只改变系数图的估计分辨率。

普通版：

$$
q = \bar{a}I + \bar{b}
$$

快速版：

$$
q = AI + B
$$

其中：

$$
A = U_s(\bar{a}^L),\qquad B = U_s(\bar{b}^L)
$$

`A`、`B` 是低分辨率系数图上采样后的结果，用来近似原分辨率 `bar_a`、`bar_b`。

### 7.2 灰度快速版流程

1. 下采样：

$$
I^L = D_s(I),\qquad p^L = D_s(p)
$$

2. 在低分辨率上计算局部统计：

$$
\mu_{I^L} = M_{r^L}[I^L],\qquad
\mu_{p^L} = M_{r^L}[p^L]
$$

$$
\operatorname{cov}_{I^L,p^L}
=
M_{r^L}[I^Lp^L] - \mu_{I^L}\mu_{p^L}
$$

$$
\operatorname{var}(I^L)
=
M_{r^L}[(I^L)^2] - \mu_{I^L}^2
$$

3. 解低分辨率系数：

$$
a^L =
\frac{\operatorname{cov}_{I^L,p^L}}
{\operatorname{var}(I^L)+\epsilon}
$$

$$
b^L = \mu_{p^L} - a^L\mu_{I^L}
$$

4. 对系数再做局部平均：

$$
\bar{a}^L = M_{r^L}[a^L],\qquad
\bar{b}^L = M_{r^L}[b^L]
$$

5. 只上采样系数图：

$$
A = U_s(\bar{a}^L),\qquad B = U_s(\bar{b}^L)
$$

6. 用原分辨率 guidance 输出：

$$
q = AI + B
$$

### 7.3 禁止误解

下面这个不是 Fast Guided Filter：

$$
q \approx U_s(q^L)
$$

直接把低分辨率输出 `q^L` 放大会丢掉高频边缘。Fast Guided Filter 的关键是：低分辨率只估计更平滑的系数图，最终边缘定位仍由原分辨率 `I` 提供。

## 8. 复杂度与参数取舍

普通 guided filter 的主要计算是常数次 box filter 和逐点运算，复杂度为：

$$
O(N),\qquad N = H\times W
$$

Fast Guided Filter 把最重的局部统计放到低分辨率，主统计部分近似变为：

$$
O(N/s^2)
$$

总流程仍包含上采样和最终逐点合成，因此更完整的工程视角是：

$$
O(N/s^2) + O(N)
$$

参数判断：

- `r` 越大，统计范围越大，平滑越强。
- `eps` 越大，`a` 越容易被压小，结果越平滑。
- `eps` 必须和像素值范围匹配；`[0,1]` 与 `[0,255]` 不能共享同一数值直觉。
- `s = 1` 应接近普通 guided filter。
- `s` 越大，局部统计越便宜，但边缘附近的系数近似误差越大。
- `s = 2` 或 `s = 4` 常是工程起点，不是通用定律。

## 9. 实现检查表

### 9.1 普通灰度版

| 数学对象 | 实现变量 | 检查点 |
|---|---|---|
| `M_r[I]` | `mean_I` | box filter / integral image 均可 |
| `M_r[p]` | `mean_p` | 与 `I` 尺寸一致 |
| `M_r[Ip]` | `mean_Ip` | 逐点乘法后再局部均值 |
| `M_r[I^2]` | `mean_II` | 用于方差 |
| `cov` | `mean_Ip - mean_I * mean_p` | 不要漏掉中心化 |
| `var` | `mean_II - mean_I * mean_I` | 低方差区域依赖 `eps` |
| `a` | `cov / (var + eps)` | 逐点除法 |
| `b` | `mean_p - a * mean_I` | 逐点乘减 |
| `bar_a` / `bar_b` | `boxMean(a)` / `boxMean(b)` | 重叠窗口平均 |
| `q` | `bar_a * I + bar_b` | 最终使用原 guidance |

### 9.2 彩色版

| 数学对象 | 实现变量 | 检查点 |
|---|---|---|
| 通道均值 | `mean_I0` / `mean_I1` / `mean_I2` | 每个通道局部均值 |
| 通道-输入协方差 | `cov_Ip0` / `cov_Ip1` / `cov_Ip2` | 组成协方差向量 |
| 通道间协方差 | `var_00` / `var_01` / ... / `var_22` | 组成对称矩阵 |
| 正则化矩阵 | `Sigma + eps * I` | 对角线加 `eps` |
| 三维系数 | `a0` / `a1` / `a2` | 解线性系统，不是独立灰度滤波 |
| 截距 | `b` | `mean_p - a^T mean_I` |
| 输出 | `a0*I0 + a1*I1 + a2*I2 + b` | 使用原分辨率通道 |

### 9.3 Fast 版差异

Fast 版只在这些点上改变普通实现：

- 先构造 `I_sub = D_s(I)`、`p_sub = D_s(p)`。
- 用 `r_sub = max(1, round(r / s))` 计算低分辨率局部统计。
- 在低分辨率上求 `a_sub`、`b_sub`、`mean_a_sub`、`mean_b_sub`。
- 上采样 `mean_a_sub`、`mean_b_sub`，得到 `A`、`B`。
- 最终输出必须使用原分辨率 `I`：`q = A * I + B`。

## 10. 选型规则

优先用普通 guided filter，当：

- 质量基线比延迟更重要
- 图像尺寸不大，或者窗口半径较小
- 对边缘误差敏感，需要先建立可信 reference
- 需要排查 fast variant 的近似误差来源

优先考虑 Fast Guided Filter，当：

- 图像或视频分辨率高
- 延迟预算明确
- 系数图预期比原图和输出更平滑
- 可接受小幅系数近似误差换取显著加速

不要使用 guided filter 作为默认解，当：

- 任务需要语义理解、上下文推理或对象级边界
- guidance 本身边缘错误或噪声结构强
- 误差预算极小，而 fast variant 的 `s` 已经引入可见伪影

## 11. 失败模式

常见错误：

- 把 guided filter 当通用去噪器。
- 把 `eps` 当无尺度 magic number。
- 忽略输入是否已归一化到 `[0,1]`。
- 彩色 guidance 按三个灰度滤波分别运行，丢掉通道协方差。
- Fast 版直接上采样 `q^L`，而不是上采样系数图。
- 只看 `s` 带来的速度，不看边缘伪影和任务指标。

判断失败原因时优先检查：

1. guidance 是否可信。
2. `eps` 是否匹配数据范围。
3. `r` 是否与目标结构尺度匹配。
4. Fast 版的 `s` 是否过大。
5. 边界处理和 box filter 是否与公式一致。

## 12. 验证入口

最小正确性测试：

1. 常值图：`I` 和 `p` 都为常值时，输出应保持常值。
2. `p = I`：输出应接近原图，只发生受 `eps` 和 `r` 控制的轻微平滑。
3. 阶跃边缘：guided filter 应比均值滤波更少跨边缘混合。
4. 改变 `eps`：`eps` 增大时输出更平滑，边缘跟随变弱。
5. 彩色 guidance：三个通道结构不同，验证实现使用了跨通道协方差矩阵。
6. Fast `s = 1`：结果应接近普通 guided filter。
7. 固定 `r`、`eps` 改变 `s`：记录速度提升、边缘误差和任务指标变化。

## 13. 现实锚点与当前实践

截至 `2026-03-19`，现实工具链锚点包括：

- OpenCV `ximgproc::guidedFilter` / `createGuidedFilter`
- OpenCV ximgproc 中带 `scale` 参数的 `(Fast) Guided Filter` 接口
- MATLAB `imguidedfilter`
- He, Sun, Tang 的 Guided Image Filtering ECCV 2010 / TPAMI 2013
- He, Sun 的 Fast Guided Filter arXiv 2015

当前更稳的实践：

- 用普通 guided filter 建立质量基线。
- 在高分辨率、强延迟约束场景中评估 Fast Guided Filter。
- 把 `r`、`eps`、`s` 放入同一个实验矩阵，而不是单独调参。
- 当任务依赖语义边界时，转向 learned refinement 或 task-specific 后处理。

参考资料：

- Kaiming He, Jian Sun, Xiaoou Tang. Guided Image Filtering. ECCV 2010 project page: https://people.csail.mit.edu/kaiming/eccv10/index.html
- Kaiming He, Jian Sun, Xiaoou Tang. Guided Image Filtering. TPAMI 2013: https://pubmed.ncbi.nlm.nih.gov/23599054/
- Kaiming He, Jian Sun. Fast Guided Filter. arXiv 2015: https://arxiv.org/abs/1505.00996
- OpenCV ximgproc filters docs: https://docs.opencv.org/4.x/da/d17/group__ximgproc__filters.html
- MathWorks `imguidedfilter`: https://www.mathworks.com/help/images/ref/imguidedfilter.html
