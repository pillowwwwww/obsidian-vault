---
Title: Energy-Based Models
tags: 
原始链接: https://www.zhihu.com/search?type=content&q=Energy-Based%20Models
---

## 🧠一、 什么是EBMs？Energy-Based Models

我感觉就是为了方便建模。

比如在一个复杂系统里你要表示一个变量

![x\in X](https://www.zhihu.com/equation?tex=x%5Cin+X)

的概率 P(x)，要满足如下性质：

- P(x)>0
- 所有变量概率之和为1
- 方便计算

你就可以想到用指数函数（因为大于0）。

所以 P(x) 可以表示为：

![P(x)=\frac{1}{Z}e^{f(x)}](https://www.zhihu.com/equation?tex=P%28x%29%3D%5Cfrac%7B1%7D%7BZ%7De%5E%7Bf%28x%29%7D)

f(x) 是关于 x 的函数，其中 Z 是normalize项 :

![Z=\sum_{x\in X} e^{f(x)}](https://www.zhihu.com/equation?tex=Z%3D%5Csum_%7Bx%5Cin+X%7D+e%5E%7Bf%28x%29%7D)

所以：

![P(x)=\frac{e^{f(x)}}{\sum_{x\in X}e^{f(x)}}](https://www.zhihu.com/equation?tex=P%28x%29%3D%5Cfrac%7Be%5E%7Bf%28x%29%7D%7D%7B%5Csum_%7Bx%5Cin+X%7De%5E%7Bf%28x%29%7D%7D)

哇是不是很像softmax。

统计物理中玻尔兹曼(吉布斯)分布有着差不多的形式：

![P(x)=\frac{e^{-E(x)}}{\sum_{x\in X} e^{-E(x)}}](https://www.zhihu.com/equation?tex=P%28x%29%3D%5Cfrac%7Be%5E%7B-E%28x%29%7D%7D%7B%5Csum_%7Bx%5Cin+X%7D+e%5E%7B-E%28x%29%7D%7D)

这里的E表示状态的能量。

之前的f(x)就可以写成：

![f(x)=-E(x)](https://www.zhihu.com/equation?tex=f%28x%29%3D-E%28x%29)

所以这样的建模方法就叫做能量模型啦。

通常在机器学习中 E(x)通常可以表示为[似然函数](https://zhida.zhihu.com/search?content_id=67108005&content_type=Answer&match_order=1&q=%E4%BC%BC%E7%84%B6%E5%87%BD%E6%95%B0&zhida_source=entity)或者[对数似然](https://zhida.zhihu.com/search?content_id=67108005&content_type=Answer&match_order=1&q=%E5%AF%B9%E6%95%B0%E4%BC%BC%E7%84%B6&zhida_source=entity)或者[值函数](https://zhida.zhihu.com/search?content_id=67108005&content_type=Answer&match_order=1&q=%E5%80%BC%E5%87%BD%E6%95%B0&zhida_source=entity)，所以有时候求最大似然就可以被表示为求最小能量。

---
 以上是原文，下面是我的整理：
 
 #### 1️⃣ 能量函数与概率分布的联系

在 Energy-Based Models（EBMs）中，样本的概率通过 Boltzmann 分布建模：

$P(x) = \frac{e^{-E(x)}}{Z}$

- $E(x)$：输入样本 $x$ 的能量（越小表示越可能）
    
- $Z$：归一化常数（配分函数）
- - **能量 E(x) 越小，P(x) 越大 → **负相关**；
    
- 所以我们最大化概率 P(x)) ↔ 等价于最小化能量 E(x)。
    
**结论**：最小能量 $\Rightarrow$ 最大概率
#### 2️⃣ 最大似然学习 ≈ 最小能量函数
![[image-30.png|611x247]]
#### ![[image-31.png|602x226]]3️⃣ 


---

## 🧠 二、什么是“值函数”（Value Function）？
## 为什么把“值函数”变成“能量函数”？

在机器学习中，“值函数”并非某一固定公式，而是一个**统称**，它在不同任务中代表“我们想优化的目标”，例如：

|应用场景|值函数（Value Function）表示什么|
|---|---|
|**分类**|softmax/logits 的得分函数，例如 `f(x) = W·x + b`|
|**对比学习**|相似性得分 `f(x, x⁺) = xᵀx⁺`，希望正样本得分高，负样本低|
|**强化学习**|状态/状态-动作值函数 `V(s)` 或 `Q(s, a)`|
|**GAN判别器**|`D(x)`：区分真实 vs 生成样本的得分|
|**MoE门控**|`G(x) = softmax(Wx)`：对各专家的权重分配|

通俗讲：

> ✅ **值函数 = 模型当前对某个输入输出的偏好程度、置信度、价值评估。**