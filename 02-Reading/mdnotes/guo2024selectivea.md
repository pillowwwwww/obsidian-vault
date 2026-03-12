# 🔤 icir2025: FedSA-LoRA: 联邦学习中低秩适应的选择性聚合 (2024, Guo) ()

**原名：**Selective aggregation for low-rank adaptation in federated learning

**译名：** icir2025: FedSA-LoRA: 联邦学习中低秩适应的选择性聚合

**作者：**Guo et al.

**期刊：**

**IFQ：**

**DOI：** [10.48550/ARXIV.2410.01463](https://doi.org/10.48550/ARXIV.2410.01463)

**发表时间：**2024

**本地链接:** [2024-Selective aggregation for low-rank adaptation in federated learning 1.pdf](zotero://open-pdf/0_H58MRY9T)

**摘要翻译：** _我们通过分析学习到的 $A$ 和 $B$ 矩阵的不对称性来研究联邦学习中的 LoRA。在此过程中，我们发现 $A$ 矩阵负责学习通用知识，而 $B$ 矩阵专注于捕获客户端特定的知识。基于这一发现，我们引入了 Federated Share-A Low-Rank Adaptation（FedSA-LoRA），该方法使用两个低秩可训练矩阵 $A$ 和 $B$ 来建模权重更新，但仅共享 $A$ 矩阵以供服务器聚合。此外，我们探讨了其他 LoRA 变体（如 rsLoRA 和 VeRA）中学习到的 $A$ 和 $B$ 矩阵之间的关系，揭示了一致的模式。因此，我们将 FedSA-LoRA 方法扩展到这些 LoRA 变体，从而得到 FedSA-rsLoRA 和 FedSA-VeRA。通过这种方式，我们建立了一种将 LoRA 与联邦学习集成的一般范式，为后续将 LoRA 变体与联邦学习结合的研究提供了指导。广泛的自然语言理解和生成任务的实验结果证明了所提出方法的有效性。我们的代码可在 https://github.com/Pengxin-Guo/FedSA-LoRA 获取。_

## 💡创新点

### 论文明确指出，**矩阵 A 负责学习“通用知识”**（General Knowledge），而**矩阵 B 则专注于捕获“客户端特定/个性化的知识”**（Client-specific Knowledge） 。

**依据：**

**1.理论维度的区别（引理 1）**

2.实验观察：发现不同客户端训练出的 A 矩阵之间具有非常高的相似度；而 B 矩阵之间的相似度则低得多 。更关键的是，随着客户端数据异构性（non-IID 程度）的增加，不同客户端之间 B 矩阵的相似度会进一步下降，这进一步证实了 B 矩阵是在拟合本地的个性化特征。

【恍然大悟！】

Lora分为A B矩阵，这是参数空间的。

流经AB矩阵的**输入数据** $x_t$，这是数据空间的。

$V \Lambda V^T $  是 $\mathbb{E}[x_t x_t^T]$的“真身”，在矩阵的世界里，为什么 $\mathbb{E}[x_t x_t^T]$ 一定能被分解成 $V \Lambda V^T$ 呢？

$\begin{bmatrix} \mathbb{E}[x_1^2] & \mathbb{E}[x_1 x_2] \\ \mathbb{E}[x_2 x_1] & \mathbb{E}[x_2^2] \end{bmatrix}$

这背后隐藏着线性代数中最伟大、最优美的定理之一：**谱定理（Spectral Theorem）**。

**谱定理（Spectral Theorem）** 铁口直断地给出了一个无与伦比的保证：

**只要一个矩阵是实对称矩阵，它就【一定】能被分解为一个正交矩阵** $V$**、一个对角矩阵** $\Lambda$**、以及** $V$ **的转置** $V^T$ **的乘积。**

也就是说：因为 $\mathbb{E}[x_t x_t^T]$ 完美对称，所以数学之神保证了必定存在一组 $V$ 和 $\Lambda$，使得：$\mathbb{E}[x_t x_t^T] = V \Lambda V^T$

这个等号是绝对成立的，没有任何条件和悬念。

在引理 1 中，论文用微积分算出了 $B^*$ 的完美公式 ：

$B^{*} = \Delta_{W}\mathbb{E}[x_{t}x_{t}^{T}]Q^{T}(Q\mathbb{E}[x_{t}x_{t}^{T}]Q^{T})^{-1}$

现在，我们把 $\mathbb{E}[x_t x_t^T]$ 替换成它的真身 $V \Lambda V^T$：

$B^{*} = \Delta_{W} (V \Lambda V^T) Q^{T} (Q (V \Lambda V^T) Q^{T})^{-1}$

**仔细看替换后的公式，你能得出两个极其震撼的物理结论：**

#### 1. 为什么 B 矩阵必须是“本地化（个性化）”的？

你看，最优的 $B^*$ 矩阵的身体里，**死死地镶嵌着** $V$**（数据的主轴方向）和** $\Lambda$**（数据的方差/信息量大小）**！

这意味着，神经网络在训练 B 矩阵时，其实是在暗中学习本地数据的 SVD 特征结构：

- 本地数据的 $V$ 往哪边倾斜，B 矩阵在映射时就必须顺着那个方向去发力。
- 本地数据的 $\Lambda$ 在哪个维度上值很大，B 矩阵就必须在这个维度上分配更多的权重。

如果客户端 1 是一所医院（数据形状是 $V_1 \Lambda_1 V_1^T$），客户端 2 是自动驾驶（数据形状是 $V_2 \Lambda_2 V_2^T$）。因为它们的“锁”完全不同，所以它们各自打磨出来的“钥匙” $B_1^*$和 $B_2^*$ 必然完全不同！

## 💧新名词：

谱定律：

## 🌏研究背景：

## 🌟重点：

## 🔬实验方法：

## 📜 总结：

## Other

Accepted at ICLR 2025