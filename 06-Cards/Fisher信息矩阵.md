---
Title: 
tags: 
原始链接:
---
前提：参数处于最优解的位置。


# Score 函数定义为：
$$s(\theta) = \frac{\partial}{\partial \theta} \log f(x; \theta)$$
同时可以写为：

$$s(\theta) = \nabla_\theta \log P(x; \theta)$$
倒三角符号 $\nabla$ 读作 **Nabla**（或者 Del），它的数学含义正是你说的——**求偏导**。
但是，它比普通的 $\frac{\partial}{\partial \theta}$ 更“高级”一点：它是**针对向量（多参数）的求导**。
在神经网络里，$\theta$ 可能包含了 $[w_1, w_2, b_1, \dots]$ 成千上万个参数。
这时候写 $\frac{\partial}{\partial \theta}$ 就有点不够用了，因为我们需要对每一个参数都求偏导。
于是我们用 $\nabla_\theta$ 来表示一个**“导数包”**（Gradient Vector）：

$$\nabla_\theta f = \begin{bmatrix} \frac{\partial f}{\partial \theta_1} \\ \frac{\partial f}{\partial \theta_2} \\ \vdots \\ \frac{\partial f}{\partial \theta_n} \end{bmatrix}$$

#### 数学上的score函数就是梯度：

- **在统计学中：**

    我们要**最大化**似然函数（Maximum Likelihood Estimation, MLE）。
    目标函数是：$J(\theta) = \log P(x; \theta)$（对数似然）。
    **Score 函数定义为：**$$s(\theta) = \nabla_\theta \log P(x; \theta)$$
    **结论：Score 函数 = 对数似然的梯度。**
    
- **在深度学习中：**
    我们要**最小化**损失函数（Loss Function）。
    最常用的损失函数是**负对数似然 (Negative Log-Likelihood, NLL)**（比如交叉熵损失）。
    目标函数是：$L(\theta) = - \log P(x; \theta)$。
    **梯度定义为：**
    $$g(\theta) = \nabla_\theta L(\theta) = \nabla_\theta \left( - \log P(x; \theta) \right) = - \nabla_\theta \log P(x; \theta)$$
**看到联系了吗？**
$$g(\theta) = - s(\theta)$$

**深度学习里的“梯度”，就是统计学里的“Score 函数”加个负号。**

- **Score 函数：** 指向**概率增加**的方向（爬山）。
- **梯度（Loss）：** 指向**误差减小**的方向（下山）

它们的大小（模长）是一模一样的，只是方向相反。

#### score函数是导数除以原函数
$$s(\theta) = \frac{\partial}{\partial \theta} \log f(x; \theta) = \frac{f'(x; \theta)}{f(x; \theta)}$$
推导过程我自己推过了，理解score函数是**导数除以原函数**。

这在数学上衡量的是：当参数 $\theta$ 变化时，概率密度函数产生的**百分比变化**。

# Fisher 信息的定义

Fisher 信息矩阵 $I(\theta)$ 是 Score 函数方差的期望值：

$$I(\theta) = E \left[ \left( \frac{\partial}{\partial \theta} \log f(x; \theta) \right)^2 \right]$$
信息矩阵：score函数平方的期望。但是我们称这是Score函数方差的期望
- **方差的定义：** $\text{Var}(X) = E[(X - \mu)^2]$
    
- **Fisher 信息的定义：** $I(\theta) = E[s(\theta)^2]$
    

这两个式子长得不一样。要让它们相等，必须满足一个极其特殊的条件。

这个条件就是：**Score 函数的期望值（均值）永远等于 0。**