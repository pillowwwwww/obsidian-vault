# RobustMerge：面向多模态大语言模型的、具有方向鲁棒性的参数高效模型合并

  

**作者**：Fanhu Zeng，Haiyang Guo，Fei Zhu，Li Shen，Hao Tang  

**单位**：  

1. 中国科学院自动化研究所 MAIS  

2. 香港中文大学（深圳）人工智能与机器人中心 HKISI-CAS  

3. 中山大学深圳校区网络空间安全学院  

4. 北京大学计算机学院，多媒体信息处理全国重点实验室  

  

**邮箱**：challengezengfh@gmail.com，guohaiyang2023@ia.ac.cn，zhfei2018@gmail.com，shenli6@mail.sysu.edu.cn，haotang@pku.edu.cn  

  

**会议**：NeurIPS 2025  

**说明**：以下为论文的中文全文整理版，保留原有结构、公式、图表标题与表格数据；参考文献条目为便于检索保留原文。图像本身未嵌入，但其图注已译出。

  

---

  

## 摘要

  

使用自定义数据对预训练模型进行微调，会产生大量专门面向特定任务的专家模型。将多个模型合并为一个通用模型，以在避免数据泄漏的前提下获得多任务能力，已变得越来越流行。随着数据规模和模型规模的扩大，参数高效微调逐渐成为高效获得任务专用模型的常见做法。然而，专门面向参数高效合并的方法仍然很少，而现有为全参数微调合并设计的方法，在参数高效合并场景下会失效。为了解决这一问题，我们从低秩分解的角度进行分析，揭示了合并过程中的**方向鲁棒性**对于合并参数高效模块至关重要。我们进一步发现，补偿差距悬殊的奇异值之间的鸿沟，有助于提升方向鲁棒性。

  

因此，我们提出 **RobustMerge**：一种无需训练的参数高效模型合并方法，通过**互补参数自适应**来维持方向鲁棒性。具体来说，我们：（1）基于参数间关系，对参数进行剪枝并构造奇异值缩放系数，从而在远离任务干扰的情况下维持方向稳定；（2）执行跨任务归一化，以增强对未见任务的泛化能力。我们构建了一个由多样化多模态任务组成的基准，并在其上进行了实验，以验证我们方法卓越的性能与泛化性。进一步的研究和大量分析也展示了该方法的有效性。代码地址：<https://github.com/AuroraZengfh/RobustMerge>。

  

---
在修复这些问题后，

## 1 引言

  

基础模型的快速发展促进了基于定制数据构建专家模型。现代模型，尤其是大语言模型（LLM），通常在多种数据集上进行预训练以获得通用知识，而后在任务特定数据上进行微调以获得特定领域能力。面对不同领域的任务时，多任务学习 [46] 是一种常见范式，用于缓解不同任务间的性能波动。然而，某些特定知识往往需要随着时间逐步累积。随着模型规模持续增大 [4, 58]，一旦模型在某一特定数据集上完成专业化，再重新训练它以获得另一领域知识不仅耗时且资源开销巨大，还可能遭遇灾难性遗忘 [66]。此外，数据隐私问题也会阻碍其实用化部署。为了解决这些问题，**模型合并** [50] 被提出：在**无需训练、无需访问原始数据**的前提下，将多个各自掌握特定知识的独立模型直接整合为一个具有多任务能力的统一模型。它的有效性与便利性，已在多个下游任务中展现出巨大潜力 [10, 48]。

  

尽管模型合并很受欢迎，但其中仍有若干关键问题尚未解决，限制了它在现实场景中的部署。首先，随着多模态大语言模型（MLLM）等大模型以及海量数据的出现，**参数高效微调**（PEFT）[16] 已成为训练大模型最主流、最有效的方法之一。然而，现有模型合并方法大多关注**全参数微调**（FFT）场景 [61, 11]；当它们被直接应用于参数高效模型合并时，往往会受到分布偏移的影响，导致明显性能下降，如图 1 所示。其次，当前性能较强的方法往往依赖于额外的已见任务信息（例如验证集 [63] 或额外存储 [18]）来提升性能，因此它们通常只能处理已见任务，难以泛化到未见任务，这使得它们在真实场景中的鲁棒性和可扩展性受到质疑，如表 1 所示。与我们最相关的工作是 LoraHub [17]，但它要求在测试时通过自适应来优化系数，这严重阻碍了其实际应用。

  

受上述不足启发，我们旨在设计一种**具有泛化能力的参数高效模块合并算法**。首先，我们分析性能下降的原因，观察到：（1）奇异值存在显著悬殊；（2）参数高效模块的参数分布与全参数微调不同，呈现出明显更宽的分布。进一步地，从低秩分解视角出发，我们揭示了**方向鲁棒性**——即在合并过程中维持奇异向量方向——对高效合并至关重要。

  

基于这些观察，我们提出 **RobustMerge**：一种面向多模态大模型高性能合并的新型参数高效方法，并引入有效的**互补参数自适应**以维持方向、提升性能。具体而言，我们直接在 LoRA 组件上对无效参数进行剪枝，并根据参数间关系构造缩放系数，以缓解由奇异值差异悬殊所引起的任务间干扰。此外，我们执行跨任务归一化，以平衡不同数据规模的任务，并增强对未见任务的泛化能力。值得注意的是，我们的方法**不需要任何额外数据或存储，也不需要显式分解**，因此更具灵活性与高效性。

  

我们在一个包含 **8 个已见任务**和 **4 个未见任务**、覆盖多种领域的基准上进行实验，以评估多模态生成任务上的能力。我们还报告了若干通用评测基准上的结果，显示本方法在已见任务（+3.4%）、未见任务（+4.5%）以及综合通用能力上都取得了显著提升，证明了方法的有效性与泛化性。除此之外，我们还在视觉任务上进行了实验，并给出大量分析以验证方法的实用价值。

  

**本文贡献如下：**

- 我们聚焦于**参数高效模型合并**，强调了在**无需额外数据与存储**的前提下设计高性能参数高效合并算法的必要性。

- 我们从低秩分解中奇异值的**方向鲁棒性**视角进行分析，并提出一种**无需训练**的有效合并算法，通过互补参数自适应维持方向，从而增强合并性能。

- 我们开展了广泛实验，结果显著优于现有方法，强有力地验证了该方法的有效性与泛化性。

  

### 图 1

**图 1：** 已见任务增强与未见任务泛化之间的性能平衡。

  

### 表 1 不同方法的前提条件与适用范围

  

| 方法 | 无需验证集 | 无需额外存储 | 支持未见任务 | 支持参数高效合并 |

|---|---:|---:|---:|---:|

| Task Arithmetic | ✗ | ✓ | ✓ | ✗ |

| DARE | ✓ | ✓ | ✓ | ✗ |

| Ties-merging | ✓ | ✓ | ✓ | ✗ |

| PCB-Merging | ✓ | ✓ | ✓ | ✗ |

| LoraHub | ✗ | ✗ | ✓ | ✓ |

| AdaMerging | ✗ | ✓ | ✓ | ✗ |

| EMR-Merging | ✓ | ✗ | ✗ | ✗ |

| **RobustMerge** | **✓** | **✓** | **✓** | **✓** |

  

> 注 2：这里的 *complementary* 与 *individual* 相对，用于区分矩阵沿秩维度（`r` 维）进行的子空间乘法，与沿原始输入/输出维度（`d_i / d_o` 维）进行的普通乘法。

  

---

  

## 2 相关工作

  

### 2.1 多模态大语言模型

  

随着数据量与模型规模的激增，大语言模型（LLM）[45, 55, 1] 展示了极其强大的性能。它们通常由仅解码器结构组成，以自回归方式响应输入，因此在分类任务 [56] 与生成任务 [7] 中都显示出巨大潜力。进一步地，多模态大语言模型（MLLM）为大模型增加了视觉感知能力：它们使用视觉编码器提取图像特征，并借助跨模态模块对齐图文特征 [31, 26] 等。当前关于大模型的大量工作都致力于直接利用任务特定数据微调单个独立模型，以在某一任务上获得更好结果。与其关注某一特定领域性能的提升，我们更关注将多个模型整合为一个模型，以提升效率并同时处理多项任务。

  

### 2.2 参数高效微调

  

当使用任务特定数据对预训练模型进行微调时，训练整个模型不仅会扰动模型从数十亿级数据中学到的表征，也会带来巨大的资源消耗。为了解决这一问题，研究者提出了**参数高效微调** [13]，即避免对整个模型进行全量微调。它通常只训练轻量级模块来使模型适应下游任务，并取得了与全参数微调模型相当的竞争性能。已有大量参数高效技术被探索，例如提示学习 [20, 23, 65]，以及包括 LoRA [16, 59, 33, 40, 32]、(IA)$^3$ [30] 在内的适配器学习等。本文聚焦于 **LoRA**，因为它是最常用的 PEFT 方法，并已在多个领域 [67, 9]，尤其是大模型 [31] 中展示出有效性。

  

### 2.3 模型合并

  

**模型合并** [62, 51] 指的是将多个具有不同能力的模型合并为一个统一模型，以处理多任务学习 [21, 39]。Task Arithmetic [19] 提出了一种范式：通过从微调模型中减去预训练模型得到任务向量，并将模型合并视为任务向量上的算术运算。它已在多个领域得到广泛关注 [52]。Ties-merging [61] 通过裁剪与符号选择来减少干扰。DARE [64] 随机丢弃参数并对剩余参数进行重缩放，以逼近原始表示。PCB-merging [11] 则通过带有竞争平衡的参数调整来缓解潜在冲突。然而，这些方法大多聚焦于分类任务中的 FFT 模型合并 [5]，在分布偏移存在时难以获得满意性能 [53]。一些较新的工作 [28, 29] 也关注预训练阶段的 checkpoint 合并，以增强下游表现。相比之下，我们聚焦于**多模态任务中的参数高效合并**。

  

### 图 2

**图 2：** 在低秩空间中合并任务 A 与任务 B，并分别在每个任务上进行评估的示意图。向量长度表示数值上的奇异值大小。  

左图：单任务内部存在差距悬殊的奇异值，导致跨任务合并时不稳定。  

右图：由于大奇异值方向天然更具鲁棒性，在合并特定奇异向量时，方向不稳定更可能出现在较小奇异值上。对尾部奇异值进行缩放有助于提升方向鲁棒性并改善性能。

  

---

  

## 3 方法

  

我们首先介绍参数高效合并的基本记号；接着给出在合并参数高效模块时、降低任务干扰的观察与动机；最后提出我们的方法，以提升多模态大语言模型中参数高效模型合并的性能。

  

### 3.1 预备知识与记号

  

参数高效微调会冻结预训练模型，仅微调轻量模块以适应下游任务。本文聚焦于 LoRA [16]，这是一种低秩适配技术，它将额外参数分解为两个低秩矩阵。形式化地，对于权重矩阵 $W_0 \in \mathbb{R}^{d_o \times d_i}$，更新后的矩阵写作：

  

$$

W = W_0 + \Delta W = W_0 + B \cdot A,

\tag{1}

$$

  

其中 $B \in \mathbb{R}^{d_o \times r}$、$A \in \mathbb{R}^{r \times d_i}$，并且秩 $r \ll \min(d_i, d_o)$。

  

模型合并的目标是：给定多个具有相同结构、且由预训练模型 $\theta_{\text{pre}}$ 微调得到的模型 $\{\theta_1, \cdots, \theta_N\}$，在**无需训练**的条件下将它们合并为一个新模型 $\theta_m$，同时保持多任务能力。现有全参数微调（FFT）方法通常遵循 Task Arithmetic（TA）[19] 提出的范式：构造任务向量

$\tau_n = \theta_n - \theta_{\text{pre}} \in \mathbb{R}^d$，在任务向量子空间中执行某种合并算法，再加回预训练模型，即

  

$$

\theta_m = \theta_{\text{pre}} + \lambda \sum_{n=1}^{N} \Phi(\tau_n),

$$

  

其中 $\Phi(\cdot)$ 表示合并算法。

  

参数高效模型合并与传统模型合并不同：基础模型主干被冻结，而需要合并的更新矩阵通常是随机初始化后训练得到的。因此，为简洁起见，我们直接使用 $\Delta W$ 表示待合并模块，并在这些参数高效模块上执行模型合并方法，即

  

$$

W_m = W_0 + \lambda \sum_{n=1}^{N} \Phi(\Delta W_n).

$$

  

### 3.2 动机与观察

  

虽然现有方法在 FFT 合并上表现良好，但在 PEFT 合并中仍存在挑战，导致性能不理想。为了更好地理解两者差异，我们：（1）首先分析单个任务的参数分布与低秩分解；（2）然后揭示多个任务参数高效合并的关键因素；（3）最后基于这些观察提出有效的参数高效模块合并算法。

  

#### 方向鲁棒性对于多任务模型合并至关重要

  

为了说明参数高效微调在合并中的特殊性，我们对低秩矩阵执行奇异值分解（SVD），得到奇异值及其对应方向，并引入在合并中起关键作用的概念：**方向鲁棒性（Direction Robustness）**。具体来说，对于单个矩阵而言，每个奇异值对应的方向可视为低秩空间中的任务专有知识，而奇异值大小则表示当前任务对该知识的使用程度。理论分析见附录 A。

  

我们在图 3(a) 中可视化了参数高效模块的奇异值分布，观察到头部奇异值与尾部奇异值之间存在显著差距（单任务内部差异）。因此，当要合并不同任务的模型（任务间差异）时，大奇异值的方向本质上容易发生方向变化。当合并模型并在某一特定任务上评估时，即关注该任务的专有知识时，对应的小奇异值更容易发生方向改变，从而挑战稳定性。当评估任务改变时，其他奇异向量上也会出现类似的不稳定。由此可见，**方向鲁棒性**——即在低秩矩阵合并过程中尽量保持每个奇异向量的方向——对于减轻任务干扰至关重要，因为每个奇异向量都代表着特定任务知识，并对合并性能做出贡献。图 2 给出了将分别在任务 A、B 上微调得到的模型进行合并、并分别在每个任务上评估的示意。

  

#### 缩小奇异值之间的差距有助于获得高性能合并模型

  

由于不同任务具有各自的主奇异方向，因此某些方向在某个任务中可能具有较大的奇异值，而在另一个任务中则可能较小。基于上述观察以及图 3，可以推断：对于某些任务而言，尾部奇异值对应的方向在合并时更容易引发不稳定，因此缩小奇异值之间的差距对于解决不同任务之间的干扰至关重要。一种直接的方式是**自适应地放大尾部奇异值**。这种做法对本来就较大的奇异向量影响较小，但会明显帮助较小奇异值对应的向量。图 3(a) 与图 3(b) 清楚表明，我们在第 3.3 节介绍的方法会改变奇异值分布，并通过对较小奇异值施加更大的缩放倍数来自适应调节奇异值，从而缓解方向不稳定并取得更好的性能。更详细的奇异值示意见附录 B。

  

#### 参数高效模块的参数分布与全参数微调显著不同

  

为了分析两类合并之间的差异，我们还在图 3(c) 中描绘了参数元素分布。可以发现：全参数微调的大多数参数分布更小、更集中（深蓝色分布），在这种情况下，**符号冲突**问题尤为突出 [61]。相反，参数高效组件中的参数分布范围相对更宽（浅蓝色与灰色分布），因此在不同任务间造成干扰的主要问题不是符号冲突，而是**方向不稳定**。我们将在第 4.3 节给出详细对比。

  

#### 参数高效模块内部存在内在关系

  

两个 LoRA 矩阵在 PEFT 中具有非对称功能 [68, 54]。AsymmetryLoRA [68] 指出，随机未训练的 $A$ 与训练后的 $A$ 表现相当，而 $B$ 能改善理论界；HydraLoRA [54] 则揭示，共享的 $A$ 可以保留知识。为了确定两者在合并中的不同作用，我们分别在图 3(c) 中绘制了 $A$ 与 $B$ 的分布。结果表明：$B$ 服从高斯分布，而 $A$ 近似服从均匀分布。这与已有研究一致，即 $B$ 在 PEFT 合并中扮演更加独特且关键的角色。基于这种表达特性，我们希望**直接在两个 LoRA 模块上对奇异值进行缩放**，从而避免显式、耗时的分解过程。

  

### 图 3

**图 3：**  

(a) 原始矩阵与剪枝后矩阵的奇异值大小。原始矩阵中存在悬殊的奇异值，而剪枝能有效地“抬升”尾部部分。  

(b) RobustMerge 的有效性：通过对较小奇异值施加更大的缩放，自适应地减轻干扰。  

(c) FFT 模块与 PEFT 模块的分布。全参数微调参数与参数高效微调中不同组件的分布存在明显差异。

  

### 3.3 RobustMerge：面向多模态大语言模型的参数高效模型合并

  

基于上述观察，我们提出一种新的参数高效组件合并方法，其目标是通过**自适应缩放尾部奇异值**来维持方向鲁棒性并补偿奇异值之间的差距。如图 4 所示，我们的方法包含两个部分：  

1. **剪枝与互补参数缩放**；  

2. **跨任务归一化**。

  

#### 剪枝与互补参数缩放

  

由于参数分布显著更宽，较大参数的变化更可能改变低秩空间中的方向。因此，与其像 [61, 18] 那样进行复杂的“同号参数选择”，我们将**无效参数**简单定义为：**数值幅度较小的参数**。这样，通过保留较大参数，矩阵方向不会发生过大变化，从而在缓解不同任务模型间干扰时，仍能保留特定任务知识。记 $M(\cdot)$ 为二值操作矩阵，则更新后的矩阵可写为：

  

$$

\tilde{A} = M_A(k) \odot A, \qquad

\tilde{B} = M_B(k) \odot B,

\tag{2}

$$

  

其中 $\odot$ 表示逐元素乘法，$k$ 为参数剪枝率。形式上，它会将按幅值排序后最小的 $k\%$ 参数置零。图 3(a) 展示了这种做法在改变分布、抬升尾部奇异值方面的作用。

  

剪除无效参数之后，剩余参数还应进一步精炼，以放大尾部奇异值，并补偿由任务干扰导致的性能损失。考虑到在分解矩阵上直接操作开销较大，我们受第 3.2 节中两个 LoRA 模块的非对称性与相关性启发，并利用 $A$ 在高维空间中近似正交这一事实，直接在原始低秩矩阵上进行调整，以实现相同目标。具体而言，我们提出通过**互补参数缩放**来自适应调节奇异值，即对 $B$ 进行变换，以补偿由奇异值差距引起的性能缺陷。由于 $A$ 近似服从均匀分布，我们根据 $A$ 的统计特性构造奇异值缩放系数。定义缩放矩阵 $S$ 为对角矩阵，其对角线元素为：

  

$$

S_i =

\frac{\sum_{j=1}^{d_i} \left|A[i,j]\right|}

{\sum_{j=1}^{d_i} \left| M_A[i,j] \odot A[i,j] \right|},

\qquad i = 1, \cdots, r.

\tag{3}

$$

  

这可以被视为低秩空间中的奇异值自适应：每个模型中的较小奇异值会以更大的比例被有效放大（图 3(b)），从而在不显式计算分解的前提下，减轻由方向不稳定引起的任务冲突。

  

#### 算法 1：带互补参数自适应的参数高效合并流程

  

**输入**：微调后的模型 $\{A_n, B_n\}_{n=1}^{N}$，剪枝率 $k$，秩 $r$，以及系数 $\lambda$  

**输出**：合并后的参数高效模型 $W$

  

```text

步骤 1：剪枝与互补参数缩放

M_A(k) = binary(set_topk_nonzero(A, k))

M_B(k) = binary(set_topk_nonzero(B, k))

Ã = M_A(k) ⊙ A

Ḃ = M_B(k) ⊙ B

for i = 1, ···, r:

    S_i = Σ_j |A[i,j]| / Σ_j |M_A[i,j] ⊙ A[i,j]|

  

步骤 2：跨任务归一化

for n = 1, ···, N:

    Ṡ_i^n = S_i^n / Σ_n S_i^n

  

获得参数高效模块

for n = 1, ···, N:

    Ṡ_n = Diag(S_i^n)

    ΔŴ_n ← Ḃ_n · Ṡ_n · Ã_n

  

合并参数高效模块

W ← W_0 + λ Σ_n ΔŴ_n

return W

```

  

#### 跨任务归一化

  

互补参数缩放系数 $S$ 是以任务无关方式确定的。一方面，不同已见任务之间数据规模的不平衡会导致：数据丰富任务容易过拟合，数据稀缺任务容易欠拟合；另一方面，这种不平衡也会负面影响对未见任务的泛化。因此，我们在所有任务之间对缩放矩阵做归一化，以减轻系数失衡的影响，其数学形式为：

  

$$

\tilde{S}_i^n = S_i^n \Big/ \sum_{n=1}^{N} S_i^n,

\qquad n = 1, \cdots, N.

\tag{4}

$$

  

这种归一化为不同类型任务带来了更好的平衡，因此能够获得更加稳定的性能。同时，它还能增强对未见任务的能力，如图 5(c) 所示。最终的参数高效更新可写为：

  

$$

\Delta \tilde{W}_n = \tilde{B}_n \cdot \tilde{S}_n \cdot \tilde{A}_n,

\qquad n = 1, \cdots, N,

\tag{5}

$$

  

而合并后的模型权重则通过将所有任务的参数高效模块加和得到。需要强调的是，在整个合并过程中，**无需验证数据，也无需对已见任务进行额外信息存储**，这进一步体现了该方法的优势。

  

### 图 4

**图 4：** RobustMerge 的整体流程。任务被划分为已见任务和未见任务。已见任务的 checkpoint 通过标准的单任务训练获得，并按照参数间自适应的流程进行合并。在推理时，合并后的模型既需要提升已见任务性能，也需要对分布未知的未见任务具有良好泛化性。

  

---

  

## 4 实验

  

### 实现细节

  

我们在多模态模型 [31, 44] 上开展了多模态生成任务、未见任务泛化以及视觉任务实验。我们从模型规模、任务数量、LoRA 秩等多个方面系统扩展实验，以验证方法的有效性。除非另有说明，所有模型的训练秩均设为 16。更多实现细节见附录 C。

  

### 数据集与基线

  

对于多模态任务合并，我们构建了 **MM-MergeBench**（MultiModal Merging Benchmark），其中包含 8 个多模态生成任务：ScienceQA [35]、ImageNet [5]、VQAv2 [7]、REC-COCO [22, 38]、OCRVQA [41]、Flickr30k [43]、VizWiz-caption [12] 与 IconQA [37]。这些任务覆盖问答、定位、分类、图像描述等多种类型，能够全面评估不同合并方法在生成任务上的表现。为展示对未见任务的泛化能力，我们在 4 个不同数据集上评估合并模型：ImageNet-R [15]、AOKVQA [47]、Screen2Word [57] 和 TabMWP [36]。详细说明见附录 E。此外，我们还在 POPE [27]、MME [6] 和 MMBench [34] 等通用基准上进行评估。视觉任务实验见第 4.2 节，更多结果见附录 F。

  

对于对比方法，我们在多模态大语言模型的参数高效模块上重新实现了 Task Arithmetic [19]、Ties-merging [61]、DARE [64] 与 PCB-merging [11]，以保证公平比较。关于这些基线的详细信息见附录 D。

  

### 4.1 多模态大语言模型上的生成任务实验

  

我们系统评估了多模态生成任务中的参数高效合并方法。基础模型为 LLaVA [31]，图像编码器为 CLIP-L-336 [44]。

  

#### RobustMerge 在参数高效微调场景中有效

  

我们评估了多种模型合并方法的性能。具体来说，我们首先在不同数据集上分别微调得到独立模型，然后在**不重新访问数据**的条件下对其进行合并。从表 2 左半部分可以看出，现有方法在合并参数高效模型时会遭受严重性能下降，在某些情况下甚至比零样本还差。此外，它们的表现也未必优于最简单的 Task Arithmetic，这说明 **PEFT 合并本身具有挑战性**。相比之下，我们的方法取得了最佳结果，在平均意义上**稳定且显著地**超过所有先前方法（平均提升 3.4%）。值得注意的是，我们的方法甚至取得了与多任务学习相当的性能。这些结果有力验证了方法的有效性。

  

### 表 2：MM-MergeBench 上 8 个已见任务与 4 个未见任务的性能

  

| 方法 | SciQA | Image | VQA | REC | OCR | VizWiz | Flickr | IconQA | 已见平均 | AVQA | Image-R | S2W | TabMWP | 未见平均 |

|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|

| Individual | 83.74 | 96.02 | 67.58 | 43.40 | 65.50 | 64.80 | 57.29 | 75.54 | 69.23 | - | - | - | - | - |

| Zero-Shot | 61.73 | 40.87 | 62.88 | 36.10 | 41.16 | 41.03 | 49.07 | 14.09 | 43.37 | 51.62 | 28.27 | 5.98 | 15.01 | 25.22 |

| Multi-Task | 76.90 | 74.08 | 67.05 | 35.98 | 65.37 | 66.67 | 56.09 | 66.87 | 63.62 | 76.33 | 41.39 | 8.34 | 18.20 | 36.06 |

| Task Arithmetic | 71.94 | 57.49 | 67.06 | 38.90 | 62.87 | 44.80 | 49.20 | 39.21 | 53.93 | 74.78 | 37.37 | 7.52 | 13.57 | 33.31 |

| DARE | 71.59 | 57.25 | 66.26 | 39.38 | 62.56 | 44.93 | 49.13 | 39.59 | 53.84 | 73.75 | 37.67 | 7.56 | 13.62 | 33.15 |

| Ties-merging | 71.49 | 55.88 | 66.73 | 39.67 | 65.12 | 44.35 | 47.06 | 34.46 | 53.09 | 73.43 | 38.44 | 7.47 | 13.23 | 33.14 |

| PCB-merging | 71.10 | 57.82 | 67.59 | 38.22 | 64.35 | 44.58 | 48.90 | 37.01 | 53.70 | 74.57 | 36.28 | 7.84 | 15.44 | 33.53 |

| **RobustMerge** | **73.43** | **65.54** | **67.20** | **44.80** | **62.97** | **46.61** | **52.80** | **45.90** | **57.33** | **79.30** | **45.79** | **9.23** | **17.62** | **37.99** |

  

### 表 3：不同合并模型在通用多模态基准上的性能

  

| 方法 | POPE | MME | MMBench |

|---|---:|---:|---:|

| Zero-Shot | 86.4 | 1476.9 | 66.1 |

| Traditional MTL | 86.9 | 1433.5 | 62.9 |

| Task Arithmetic | 87.0 | 1465.2 | 67.3 |

| DARE | 86.4 | 1475.7 | 67.4 |

| Ties-merging | 86.7 | 1489.4 | 66.6 |

| PCB-merging | 86.6 | 1490.7 | 66.3 |

| **RobustMerge** | **87.2** | **1494.9** | **68.1** |

  

#### RobustMerge 能提升未见任务性能

  

泛化能力是评估模型合并方法的关键，因为分布偏移在现实世界中不可避免且经常发生。表 2 右半部分展示了在 4 个未见任务上直接评估合并模型的结果。这一设定更困难，因为合并模型对未见任务的数据分布没有先验信息。现有合并方法（TA、DARE、Ties）的较差表现进一步证实了这一点，它们在某些场景下甚至不如零样本。相反，我们的方法显著提升了性能，平均提升高达 4.5%，并且甚至超过了多任务学习。尤其值得注意的是，我们的方法在**域迁移任务**（ImageNet-R）与**特定知识任务**（TabMWP）上都取得了提升，进一步证明了其泛化性。

  

#### RobustMerge 在通用多模态基准上同样优越

  

我们还在通用多模态基准 POPE [27]、MME [6] 和 MMBench [34] 上报告结果（表 3），用于评估合并模型的通用能力，例如幻觉等。结果表明，多任务学习在这些基准上表现不佳，说明该问题本身具有挑战性。相比之下，我们的方法提升了零样本能力，显著优于现有方法，并在这些困难基准上保留了出色的通用知识能力，证明了其有效性。

  

### 4.2 视觉语言模型上的视觉任务实验

  

在视觉语言模型（VLM）与视觉任务设置中，我们遵循 Task Arithmetic [19] 的实验设定，使用 LoRA 分别在 8 个视觉数据集上微调模型。数据集包括 Cars [24]、MNIST [25]、EuroSAT [14]、GTSRB [49]、DTD [3]、RESISC45 [2]、SUN397 [60] 和 SVHN [42]。详见附录 E。

  

#### RobustMerge 在视觉任务上同样有效

  

我们在 CLIP-ViT-B-32 [44] 上评估方法，结果如表 4 所示。可以看到，当采用参数高效技术对模型进行微调时，先前方法相较零样本性能并未获得明显提升，这使其在视觉任务的参数高效调优场景中的有效性受到质疑。相比之下，我们的方法相较零样本取得了 **7.9%** 的显著提升，并以 **4.4%** 的较大优势超过现有合并方法。这有力验证了我们的方法在视觉模型的参数高效合并中同样有效。

  

#### RobustMerge 可扩展到更大的视觉语言模型

  

我们进一步将方法应用于更大的模型，以验证其可扩展性。具体而言，我们分别在 8 个视觉任务上微调 CLIP-ViT-L-14，并评估合并后的参数高效组件。表 5 结果显示，随着基础模型增大，所有方法的性能都得到了提升。同时，RobustMerge 仍取得最佳结果，平均提升 **2.0%**，表明其具有良好的可扩展性。

  

### 表 4：以 CLIP-ViT-B-32 为基础模型时，8 个视觉任务的合并结果

  

| 方法 | Cars | MNIST | EuroSAT | GTSRB | DTD | RESISC45 | SUN397 | SVHN | 平均 |

|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|

| Zero-Shot | 59.7 | 48.5 | 62.3 | 32.6 | 60.7 | 43.8 | 45.5 | 31.4 | 48.0 |

| Individual | 74.3 | 99.3 | 65.2 | 92.9 | 88.7 | 58.4 | 99.1 | 96.4 | 84.2 |

| Task Arithmetic | 60.3 | 52.3 | 63.2 | 37.6 | 62.8 | 44.0 | 50.9 | 37.6 | 51.1 |

| DARE | 60.4 | 52.4 | 63.1 | 37.5 | 62.8 | 44.0 | 50.3 | 37.7 | 51.0 |

| Ties-merging | 60.7 | 56.4 | 62.4 | 33.9 | 61.3 | 43.1 | 51.1 | 42.9 | 51.5 |

| **RobustMerge** | **61.4** | **65.0** | **65.0** | **43.1** | **63.3** | **44.7** | **52.2** | **52.4** | **55.9 (+4.4)** |

  

### 表 5：基础模型扩大到 CLIP-ViT-L-14 时，8 个视觉任务的合并结果

  

| 方法 | Cars | MNIST | EuroSAT | GTSRB | DTD | RESISC45 | SUN397 | SVHN | 平均 |

|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|

| Zero-Shot | 77.7 | 76.3 | 66.8 | 50.5 | 71.0 | 55.3 | 59.9 | 58.4 | 64.4 |

| Individual | 99.7 | 99.4 | 80.0 | 97.2 | 95.8 | 70.3 | 98.6 | 97.9 | 92.4 |

| Task Arithmetic | 78.6 | 79.7 | 68.5 | 53.6 | 73.5 | 55.8 | 65.7 | 60.9 | 67.0 |

| DARE | 79.5 | 81.4 | 68.8 | 56.5 | 75.0 | 56.6 | 65.8 | 62.8 | 68.3 |

| Ties-merging | 79.4 | 83.4 | 69.5 | 59.4 | 76.0 | 55.7 | 64.0 | 64.4 | 68.9 |

| **RobustMerge** | **79.7** | **82.8** | **70.6** | **62.4** | **78.4** | **58.2** | **64.7** | **70.3** | **70.9 (+2.0)** |

  

### 4.3 消融实验与进一步分析

  

#### 各组成部分的有效性

  

我们逐步加入方法中的关键组件，即**剪枝与互补参数缩放**以及**跨任务归一化**，以验证它们的作用。表 6 显示：剪枝与互补参数缩放从根本上促进了方向鲁棒性，并减轻了模型合并中的任务干扰；将所有组件结合后，结果进一步提升。

  

### 表 6：各组成部分的影响（Prune&Scale 表示剪枝与互补缩放，Norm 表示归一化）

  

| Prune&Scale | Norm | SciQA | Image | VQA | REC | OCR | VizWiz | Flickr | IconQA | 平均 |

|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|

|  |  | 71.94 | 57.49 | 67.06 | 38.90 | 62.87 | 44.80 | 49.20 | 39.21 | 53.93 |

| ✓ |  | 73.03 | 64.18 | 67.50 | 43.12 | 58.19 | 46.36 | 52.24 | 44.54 | 56.14 (+2.21) |

| ✓ | ✓ | 73.43 | 65.54 | 67.20 | 44.80 | 62.97 | 46.61 | 52.80 | 45.90 | 57.33 (+3.40) |

  

#### 秩（rank）的影响

  

为了探索 LoRA 秩变化时的性能，我们在不同秩下训练参数高效组件。图 6 表明，随着秩提升，模型性能得到增强，因为更新子空间能够存储更多知识。与此同时，我们的方法在所有秩设置下都持续以较大幅度超过现有方法（秩 16 时高出 3.4%，秩 128 时高出 3.3%），验证了其可扩展性。

  

#### 剪枝率的影响

  

正如第 3.2 节所揭示的，数值较小的参数对低秩分解中的方向鲁棒性影响更小。因此，我们按照参数幅值进行剪枝，以简化合并过程。为了进一步验证这一观点，我们逐步增大剪枝率，并在图 5(a) 中展示平均性能变化。结果表明：

  

1. 随着剪枝率增加，由于任务间干扰减少，性能会逐渐提升；  

2. 当剪掉较大参数时，方向与任务知识受到显著影响，性能会迅速下降。

  

因此，这些结果与前文分析一致，突出了本文“无效参数剪枝”策略的有效性。

  

#### 基于幅值的参数剪枝效果更优

  

我们还将本文提出的剪枝方式，与 DARE [64] 与 Ties-merging [61] 中使用的剪枝技术进行了比较。具体而言，我们分别采用**随机剪枝**与**聚合符号**作为剪枝准则，并评估其性能。图 5(b) 表明：在参数高效合并中，**符号冲突并不是关键问题**，其性能甚至差于随机剪枝。相比之下，我们基于参数幅值的剪枝技术在多模态任务中取得了更好的结果，并以显著优势超越已有方法（分别提升 2.3% 和 4.0%）。这表明，在参数高效模型合并中，真正关键的是参数的**幅值**，而不是符号干扰。我们认为，这种提升来源于：参数高效模型的分布相比全参数微调明显更宽，而按符号进行剪枝不可避免地会改变低秩空间中的方向；相反，我们的方法在较少影响主方向的前提下避免了任务冲突。

  

#### 互补参数缩放能够有效补偿性能下降

  

第 3.3 节中我们指出，缩放系数是通过参数间的相互影响构造的。为验证其有效性，我们将其替换为不同的缩放策略。具体地，我们解耦两个模块之间的交互，分别只使用来自 $A$、$B$ 的系数；此外，我们还对 $A$ 与 $B$ 同时进行缩放，并在表 7 中报告结果。实验表明，基于**互补参数自适应**构造缩放系数是有益的，而自适应调节 $B$ 的系数能够有效提升性能，这与前面的分析一致。结果也说明：更复杂的缩放系数**未必带来更好性能**（在已见任务上反而下降 2.4%）。

  

### 表 7：互补参数缩放的影响

  

| 方法 | 已见任务 | 未见任务 |

|---|---:|---:|

| Baseline（无自适应） | 54.1 | 32.5 |

| Ours（individual, A） | 54.5 (+0.4) | 32.1 (−0.4) |

| Ours（individual, B） | 55.4 (+1.3) | 34.1 (+1.6) |

| Ours（individual, A + B） | 51.7 (−2.4) | 35.0 (+2.5) |

| **Ours（inter-parameter）** | **57.3 (+3.2)** | **38.0 (+5.5)** |

  

### 图 5

**图 5：**  

(a) 不同剪枝率下合并模型的平均性能。随着剪枝率提高，依据“无效参数”标准保留下来的非零参数数量逐步下降。  

(b) 不同剪枝技术在已见任务与未见任务上的平均性能。聚合符号方法表现较差。  

(c) 与现有方法相比，平均性能与标准差的对比。跨任务归一化在保证稳定性的同时提升了性能。

  

### 图 6

**图 6：** 不同秩设置下的平均性能。

  

#### 跨任务归一化带来更稳定的性能

  

如第 3.3 节所述，跨任务归一化不仅能在已见任务上带来一致且稳定的性能，还能进一步提升未见任务表现。我们在图 5(c) 中分析了性能与方差的关系。具体而言，与现有方法相比，我们的方法获得了高出 3.4% 的性能（57.3% 对 53.9%），同时标准差更小（1.2%）。值得注意的是，加入跨任务归一化后，这一优势进一步增强：平均性能再提升 1.2%，标准差再降低 0.2%，显示出所提方法的优越性。

  

#### 关于方向鲁棒性的进一步分析

  

为了更好理解参数高效模型合并，我们构造了用于衡量**方向鲁棒性**的评估指标。我们使用任务专用模型与合并模型之间各对应奇异向量的平均相似度作为指标，它反映了合并过程中的方向偏移程度。相似度越大，说明该方向越鲁棒，在合并时越不容易改变方向，从而更能保持特定任务的性能。此外，我们还引入**合并模型与原模型之间奇异值的比率**，以更全面地反映特定知识的保留程度。正如图 3 所示，最大的奇异值更具方向鲁棒性，因此在图 7 中，我们将其分为“最大奇异值”与“其余奇异值平均值”两部分，以更清楚地展示不同合并方法的行为。

  

可以得出以下结论：

  

1. 在合并过程中，最大的奇异向量往往较为稳定，而其余向量差异极大（即方向不稳定），这会导致评估性能下降。相比之下，我们的方法显著提高了其余奇异向量的相似度，从而明显提升了合并性能。  

2. 奇异值比率的结果也表明：对于某个特定模型，现有方法会降低最大的奇异值，却未能充分增强较小奇异值。相比之下，我们的方法更好地增强了较小奇异值，并在合并过程中保留了任务特定知识，这与我们“缩放较小奇异值有助于提高方向鲁棒性”的观点一致。

  

此外，这些分析也清晰解释了我们方法中各组成部分的功能：  

（1）**Prune（剪枝）** 用于在尽可能少影响方向的前提下缓解任务间干扰，而稀疏化本身也能增强小奇异值的鲁棒性；  

（2）**Scale（缩放）** 则在剪枝之后补偿由剪枝引起的奇异值下降，从而进一步增强方向鲁棒性。

  

### 图 7

**图 7：** 不同合并技术下奇异向量相似度与奇异值比率的比较。

  

---

  

## 5 结论

  

本文聚焦于大规模基础模型中的参数高效模型合并问题。我们从低秩分解角度分析，揭示了**方向鲁棒性**对于合并参数高效模块至关重要。我们进一步发现，对尾部奇异值进行缩放能够有效缓解任务干扰并维持方向鲁棒性。因此，我们提出了 **RobustMerge**：一种在低秩空间中维持方向的有效合并技术。我们通过广泛实验与全面分析，展示了该方法的优越性与可扩展性。这是首个从方向鲁棒性视角研究参数高效模型合并的工作，我们希望它能够启发更多先进的参数高效合并方法。

  

### 局限性与未来工作

  

我们尚未在更多模型结构和更多任务上验证本方法。不过，由于我们的方法是一种**与模型无关、与任务无关的后处理算法**，考虑到 HuggingFace 等平台上存在大量模型，这一点并不会成为主要瓶颈。另一方面，我们提出了参数高效合并中的方向鲁棒性概念，但出于效率考虑，并没有在显式分解后的矩阵上设计专门算法。我们认为，这些方向都很有前景，值得在参数高效学习的更多下游场景中继续探索。

  

---

  

# 附录

  

## 附录 A 合并中的奇异值分解理论分析

  

由于我们关注的是**低秩矩阵的合并**，因此首先介绍奇异值分解的基本记号，然后说明它在合并中的应用。

  

### A.1 奇异值分解背景

  

记参数高效模块 $W = B \times A$，其中 $W \in \mathbb{R}^{n \times n}$，并且 $W$ 是一个低秩矩阵，即

  

$$

\mathrm{Rank}(W) = r,\qquad r \ll n.

$$

  

矩阵 $W$ 的奇异值记为：

  

$$

\sigma_1 \ge \sigma_2 \ge \cdots \ge \sigma_r > 0.

$$

  

根据奇异值分解，矩阵 $W$ 可以写为：

  

$$

W = U \Sigma V^T,

\tag{6}

$$

  

其中 $U = [u_1, u_2, \cdots, u_r] \in \mathbb{R}^{n \times r}$ 和

$V = [v_1, v_2, \cdots, v_r] \in \mathbb{R}^{n \times r}$ 分别是由左、右归一化奇异向量组成的正交矩阵，$\Sigma \in \mathbb{R}^{r \times r}$ 为包含原始矩阵奇异值的对角矩阵，可表示为：

  

$$

\Sigma =

\begin{bmatrix}

\sigma_1 \\

& \sigma_2 \\

&& \ddots \\

&&& \sigma_r

\end{bmatrix}_{r \times r}.

\tag{7}

$$

  

因此，低秩矩阵可以改写为：

  

$$

W = u_1 \sigma_1 v_1^T + u_2 \sigma_2 v_2^T + \cdots + u_r \sigma_r v_r^T

= \sigma_1 u_1 v_1^T + \sigma_2 u_2 v_2^T + \cdots + \sigma_r u_r v_r^T.

\tag{8}

$$

  

### A.2 合并中的理论分析

  

为便于说明，我们考虑一个简化情形，以两个参数高效模块为例。

  

设 $W_1, W_2$ 分别是针对任务 A 与任务 B 微调得到的两个分解模块：

  

$$

W_1 = \sigma_{11}u_{11}v_{11}^T + \sigma_{21}u_{21}v_{21}^T + \cdots + \sigma_{r1}u_{r1}v_{r1}^T,

$$

  

$$

W_2 = \sigma_{12}u_{12}v_{12}^T + \sigma_{22}u_{22}v_{22}^T + \cdots + \sigma_{r2}u_{r2}v_{r2}^T,

\tag{9}

$$

  

其中，$\sigma_{ij}$、$u_{ij}$、$v_{ij}$ 分别表示第 $j$ 个低秩矩阵的第 $i$ 个奇异值 / 左奇异向量 / 右奇异向量。

  

鉴于奇异向量在 LoRA 中具有实际意义，考虑两个从 $1$ 到 $r$ 的排列，即 $(1), (2), \cdots, (r)$ 和 $(1), (2), \cdots, (r)$。那么，两个模块的合并可写为：

  

$$

\hat{W} = \lambda(W_1 + W_2)

$$

  

$$

= \lambda(\sigma_{(1)1}u_{(1)1}v_{(1)1}^T + \sigma_{(2)1}u_{(2)1}v_{(2)1}^T + \cdots + \sigma_{(r)1}u_{(r)1}v_{(r)1}^T

+ \sigma_{(1)2}u_{(1)2}v_{(1)2}^T + \sigma_{(2)2}u_{(2)2}v_{(2)2}^T + \cdots + \sigma_{(r)2}u_{(r)2}v_{(r)2}^T)

$$

  

$$

= \lambda\{

(\sigma_{(1)1}u_{(1)1}v_{(1)1}^T + \sigma_{(1)2}u_{(1)2}v_{(1)2}^T)

+ (\sigma_{(2)1}u_{(2)1}v_{(2)1}^T + \sigma_{(2)2}u_{(2)2}v_{(2)2}^T)

+ \cdots

+ (\sigma_{(r)1}u_{(r)1}v_{(r)1}^T + \sigma_{(r)2}u_{(r)2}v_{(r)2}^T)

\},

\tag{10}

$$

  

其中，每一组下标 $\{(i), (i)\}$（$i = 1, \cdots, r$）表示相似的奇异成分，也即两个不同矩阵在低秩空间中的相似知识。

  

经验上，$U$ 在低秩空间中包含更通用的知识，跨任务相似性更大。因此，合并过程在数学上可以表示为：对低秩空间中的每一组任务特定向量分别进行合并：

  

$$

\lambda \sigma_{(i)1} v_{(i)1}^T + \lambda \sigma_{(i)2} v_{(i)2}^T

= \lambda_{i1}\xi_{i1} + \lambda_{i2}\xi_{i2},

\qquad i = 1, \cdots, r.

\tag{11}

$$

  

由于 $U, V$ 都是归一化的，因此由式（11）可以推出：低秩空间中的合并可被看作是每一组任务专有奇异向量的向量加法，其中奇异向量 $\xi_i$ 表示方向，奇异值 $\sigma_i$ 表示大小。根据向量合成规律，对于每个奇异值，任务 A 与任务 B 的方向变化会彼此互补。由上述推导可见，由于原始方向与幅值存在差异，**较大奇异值更容易决定合并后奇异向量的方向与幅值**。因此，由于奇异值差距悬殊，从不同任务的奇异值向量视角看，奇异向量夹角变化将有所不同。例如，对任务 A 而言，向量 1 的方向变化较小，而向量 2 的方向变化较大；对于任务 B，情况则相反。由此可知，合并的关键在于**维持小奇异值对应向量的方向鲁棒性**。在不失一般性的前提下，这一推导可以扩展到任意数量模型的合并。

  

---

  

## 附录 B 不同层中奇异值的分布

  

我们绘制了第 1、18、32 层中 `attn.v` 的分布，以展示不同层中奇异值分布的变化。图 8 清楚表明：

  

1. 随着层位置升高，最大奇异值会变得更大，而尾部奇异值会变得更小，从而使分布更加悬殊。这与 HiDe-LLaVA [8] 的观察一致：较低层包含更多通用知识，而较高层包含更多任务特定知识，因此在前几层中，头尾奇异值的差距没有最后几层那么大。随着层数增加，分布会越来越悬殊，这进一步凸显了在合并过程中妥善处理方向不稳定的重要性。  

2. 与原始模型相比，我们的方法能在不同层中稳定地同时缩放头部与尾部奇异值，从而促进稳健、高效的合并并带来更好的性能，这强有力地说明了该方法的有效性与合理性。

  

### 图 8

**图 8：** 第 1、18、32 层中 `attn.v` 的原始模型与本文方法的奇异值分布。

  

---

  

## 附录 C 更多实现细节

  

我们分别基于 **LLaVA** 与 **CLIP** 构建了多模态任务与视觉任务的代码框架。训练时，我们遵循 LLaVA 中描述的标准训练流程，即：对每个任务单独训练，并获得参数高效模块。LoRA 被添加到基础模块中的线性层上，所有模型都只训练 1 个 epoch 用于合并实验。

  

推理时，多模态任务使用**准确率**进行评估。视觉任务则通过为每个类别构造文本提示并计算相似度来完成，每个测试样本被分类到相似度最大的类别。超参数 $\lambda$ 默认设为 2。所有合并实验均在单张 NVIDIA A6000 上进行，采样温度设为 0。

  

- LLaVA：<https://github.com/haotian-liu/LLaVA>  

- CLIP：<https://github.com/openai/CLIP>

  

---

  

## 附录 D 对比方法细节

  

模型合并方法的核心操作在于设计合并算法，即论文中定义的 $\Phi(\cdot)$。在主实验结果中，我们主要与 DARE、Ties-merging 以及 Task Arithmetic 进行比较。为了公平比较，我们将这些传统方法应用到 **LoRA 组件** 上。具体而言，我们先用 LoRA 对基础模型进行微调，再用传统方法合并 LoRA 组件，最后将合并后的 LoRA 挂接到基础模型上进行性能评估。各方法的合并方式简要如下：

  

- **Task Arithmetic**：将所有参数视作向量。它通过“微调模型 - 预训练模型”得到任务向量，并对这些向量执行标准的加减运算。它将不同任务 checkpoint 中的参数直接相加，构成多任务学习的一个强基线。

- **Ties-merging**：采用“裁剪—选择符号—合并”的范式来减少不同任务参数间的冲突。具体而言，它先保留幅值最大的参数，然后依据剩余参数的求和结果决定聚合符号，最后只合并与聚合符号一致的参数，以减轻分歧。

- **DARE**：从经验上观察到参数中存在稀疏性。它以固定比例 $p$ 随机丢弃参数，并使用 $1/(1-p)$ 对保留参数进行重缩放，使其期望上补偿被丢弃的参数。

  

---

  

## 附录 E 不同任务的细节

  

### E.1 指令微调数据集的组成

  

这些指令微调数据集采用标准的 instruction tuning 格式，由**图文对**和附加的**指令模板**组成。指令模板使用自然语言明确表达任务环境与目标，是指令微调中的关键组成。表 8 给出了各数据集使用的模板。在多模态生成任务中，我们为每个数据集精心设计了指令模板，并将其与任务特定的图像、文本输入拼接后输入模型，以自回归方式生成响应。训练时，语言模型可训练，而视觉编码器被冻结。

  

### 表 8：各数据集的指令模板（以下为中文意译；论文实验中实际使用英文模板）

  

| 数据集 | 指令模板 |

|---|---|

| ScienceQA | 请直接从给定选项中回答对应的字母。 |

| ImageNet | 图像中的物体是什么？请使用单个单词或短语回答。 |

| VQAv2 | 请使用单个单词或短语回答问题。 |

| Grounding | 请给出这句话所描述区域的边界框坐标：`<description>`。 |

| IconQA | 请使用单个单词或短语回答问题。 |

| VizWiz | 图像中发生了什么？请为图像生成一句简短描述。 |

| Flickr30k | 图像中发生了什么？请为图像生成一句简短描述。 |

| OCR-VQA | 请使用单个单词或短语回答问题。 |

| AOKVQA | 请直接从给定选项中回答对应的字母。 |

| ImageNet-R | 图像中的物体是什么？ |

| Screen2Word | 给定一个手机界面。请用一句话描述这个界面。 |

| TabMWP | 请使用单个单词或短语回答问题。 |

  

### E.2 视觉任务数据集

  

所有视觉任务都是传统图像分类数据集，包含汽车、纹理、交通标志等多个领域中的常见对象，类别数从 10 到 397 不等。我们在每个任务上使用 LoRA 对 VLM 进行微调，并使用不同类型的合并方法对其进行合并。训练时，只有视觉编码器可训练；文本编码器保持冻结，用于提取标签嵌入。

  

---

  

## 附录 F 更多实验结果

  

### RobustMerge 对任务数量具有良好泛化性

  

我们逐步增加参与合并的任务数量，以验证方法的鲁棒性。如图 9 所示：在**已见任务**上，随着合并模型数量增加，由于对特定任务的干扰变强，性能会略有下降；在**未见任务**上，性能则先上升、后小幅下降。可能的原因是：在第一阶段，已见任务将知识迁移给分布相似的未见任务，从而增强其性能；在第二阶段，任务间干扰开始主导合并过程，而非知识迁移。不论哪种情形，我们的方法都随着任务数变化而持续以显著优势超过现有方法，说明其优越性与稳定性。

  

### 表 9：在 DoRA 微调模型上的合并结果

  

| 方法 | SciQA | Image | VQA | REC | OCR | VizWiz | Flickr | IconQA | 平均 |

|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|

| Task Arithmetic | 69.91 | 67.45 | 66.18 | 41.43 | 58.57 | 46.60 | 52.68 | 40.57 | 55.42 |

| Ties-merging | 69.01 | 64.06 | 66.60 | 40.68 | 61.94 | 46.51 | 51.97 | 35.82 | 54.57 |

| **RobustMerge** | **70.95** | **68.25** | **66.48** | **41.67** | **58.39** | **46.72** | **52.78** | **43.40** | **56.08** |

  

### 图 9

**图 9：** 随着任务数量增加，已见任务与未见任务上的平均性能变化。我们的方法始终以显著优势超过 TA 与 Ties。

  

### RobustMerge 能推广到不同的 PEFT 方法

  

我们主要在 LoRA 上测试方法，因为它是最常用、最具可比性的 PEFT 技术。为了展示该方法的可扩展性，我们还在 **DoRA** [33] 上进行了实验。DoRA 是一种基于 LoRA 的参数高效技术，采用更先进的算法来提升 PEFT 学习性能。表 9 结果显示：我们的方法在不同 PEFT 方法下，依然能够相对于已有合并方法取得稳定且显著的提升。这有力说明，**方向鲁棒性问题**广泛存在于不同 PEFT 模块的合并之中，因为这些方法往往主要关注提升单一任务性能。相比之下，我们的方法在一定程度上处理了这一问题，因此即便 PEFT 技术发生变化，也能提升多任务性能。

  

---

  

## 参考文献（保留原文以便检索）

  

- [1] Jean-Baptiste Alayrac, Jeff Donahue, Pauline Luc, Antoine Miech, Iain Barr, Yana Hasson, Karel Lenc, Arthur Mensch, Katherine Millican, Malcolm Reynolds, et al. Flamingo: a visual language model for few-shot learning. Advances in neural information processing systems, 35:23716–23736, 2022.

- [2] Gong Cheng, Junwei Han, and Xiaoqiang Lu. Remote sensing image scene classification: Benchmark and state of the art. Proceedings of the IEEE, 105(10):1865–1883, 2017.

- [3] Mircea Cimpoi, Subhransu Maji, Iasonas Kokkinos, Sammy Mohamed, and Andrea Vedaldi. Describing textures in the wild. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 3606–3613, 2014.

- [4] Mostafa Dehghani, Josip Djolonga, Basil Mustafa, Piotr Padlewski, Jonathan Heek, Justin Gilmer, Andreas Peter Steiner, Mathilde Caron, Robert Geirhos, Ibrahim Alabdulmohsin, et al. Scaling vision transformers to 22 billion parameters. In International Conference on Machine Learning, pages 7480–7512. PMLR, 2023.

- [5] Jia Deng, Wei Dong, Richard Socher, Li-Jia Li, Kai Li, and Li Fei-Fei. Imagenet: A large- scale hierarchical image database. In 2009 IEEE conference on computer vision and pattern recognition, pages 248–255. Ieee, 2009.

- [6] Chaoyou Fu, Peixian Chen, Yunhang Shen, Yulei Qin, Mengdan Zhang, Xu Lin, Jinrui Yang, Xiawu Zheng, Ke Li, Xing Sun, Yunsheng Wu, and Rongrong Ji. Mme: A comprehensive evaluation benchmark for multimodal large language models, 2024.

- [7] Yash Goyal, Tejas Khot, Douglas Summers-Stay, Dhruv Batra, and Devi Parikh. Making the v in vqa matter: Elevating the role of image understanding in visual question answering. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 6904–6913, 2017.

- [8] Haiyang Guo, Fanhu Zeng, Ziwei Xiang, Fei Zhu, Da-Han Wang, Xu-Yao Zhang, and Cheng- Lin Liu. Hide-llava: Hierarchical decoupling for continual instruction tuning of multimodal large language model. In Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), page 13572–13586, 2025.

- [9] Haiyang Guo, Fanhu Zeng, Fei Zhu, Wenzhuo Liu, Da-Han Wang, Jian Xu, Xu-Yao Zhang, and Cheng-Lin Liu. Federated continual instruction tuning. arXiv preprint arXiv:2503.12897, 2025.

- [10] Haiyang Guo, Fei Zhu, Fanhu Zeng, Bing Liu, and Xu-Yao Zhang. Desire: Dynamic knowledge consolidation for rehearsal-free continual learning. arXiv preprint arXiv:2411.19154, 2024.

- [11] DU Guodong, Junlin Lee, Jing Li, Runhua Jiang, Yifei Guo, Shuyang Yu, Hanting Liu, Sim Kuan Goh, Ho-Kin Tang, Daojing He, et al. Parameter competition balancing for model merging. In The Thirty-eighth Annual Conference on Neural Information Processing Systems, 2024.

- [12] Danna Gurari, Qing Li, Abigale J Stangl, Anhong Guo, Chi Lin, Kristen Grauman, Jiebo Luo, and Jeffrey P Bigham. Vizwiz grand challenge: Answering visual questions from blind people. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 3608–3617, 2018.

- [13] Zeyu Han, Chao Gao, Jinyang Liu, Jeff Zhang, and Sai Qian Zhang. Parameter-efficient fine-tuning for large models: A comprehensive survey. arXiv preprint arXiv:2403.14608, 2024.

- [14] Patrick Helber, Benjamin Bischke, Andreas Dengel, and Damian Borth. Eurosat: A novel dataset and deep learning benchmark for land use and land cover classification. IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing, 12(7):2217–2226, 2019.

- [15] Dan Hendrycks, Steven Basart, Norman Mu, Saurav Kadavath, Frank Wang, Evan Dorundo, Rahul Desai, Tyler Zhu, Samyak Parajuli, Mike Guo, Dawn Song, Jacob Steinhardt, and Justin Gilmer. The many faces of robustness: A critical analysis of out-of-distribution generalization. ICCV, 2021.

- [16] Edward J Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, and Weizhu Chen. LoRA: Low-rank adaptation of large language models. In International Conference on Learning Representations, 2022.

- [17] Chengsong Huang, Qian Liu, Bill Yuchen Lin, Tianyu Pang, Chao Du, and Min Lin. Lo- rahub: Efficient cross-task generalization via dynamic lora composition. arXiv preprint arXiv:2307.13269, 2023.

- [18] Chenyu Huang, Peng Ye, Tao Chen, Tong He, Xiangyu Yue, and Wanli Ouyang. Emr-merging: Tuning-free high-performance model merging. In The Thirty-eighth Annual Conference on Neural Information Processing Systems, 2024.

- [19] Gabriel Ilharco, Marco Tulio Ribeiro, Mitchell Wortsman, Ludwig Schmidt, Hannaneh Ha- jishirzi, and Ali Farhadi. Editing models with task arithmetic. In The Eleventh International Conference on Learning Representations, 2023.

- [20] Menglin Jia, Luming Tang, Bor-Chun Chen, Claire Cardie, Serge Belongie, Bharath Hariharan, and Ser-Nam Lim. Visual prompt tuning. In European Conference on Computer Vision, pages 709–727. Springer, 2022.

- [21] Xisen Jin, Xiang Ren, Daniel Preotiuc-Pietro, and Pengxiang Cheng. Dataless knowledge fusion by merging weights of language models. In The Eleventh International Conference on Learning Representations, 2023.

- [22] Sahar Kazemzadeh, Vicente Ordonez, Mark Matten, and Tamara Berg. Referitgame: Referring to objects in photographs of natural scenes. In Proceedings of the 2014 conference on empirical methods in natural language processing (EMNLP), pages 787–798, 2014.

- [23] Muhammad Uzair Khattak, Hanoona Rasheed, Muhammad Maaz, Salman Khan, and Fa- had Shahbaz Khan. Maple: Multi-modal prompt learning. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 19113–19122, 2023.

- [24] Jonathan Krause, Michael Stark, Jia Deng, and Li Fei-Fei. 3d object representations for fine- grained categorization. In Proceedings of the IEEE international conference on computer vision workshops, pages 554–561, 2013.

- [25] Yann LeCun. The mnist database of handwritten digits. http://yann. lecun. com/exdb/mnist/, 1998.

- [26] Junnan Li, Dongxu Li, Silvio Savarese, and Steven Hoi. Blip-2: Bootstrapping language-image pre-training with frozen image encoders and large language models. In International conference on machine learning, pages 19730–19742. PMLR, 2023.

- [27] Yifan Li, Yifan Du, Kun Zhou, Jinpeng Wang, Wayne Xin Zhao, and Ji-Rong Wen. Evaluating object hallucination in large vision-language models. In Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing, pages 292–305, 2023.

- [28] Deyuan Liu, Zhanyue Qin, Hairu Wang, Zhao Yang, Zecheng Wang, Fangying Rong, Qingbin Liu, Yanchao Hao, Bo Li, Xi Chen, et al. Pruning via merging: Compressing llms via manifold alignment based layer merging. In Proceedings of the 2024 Conference on Empirical Methods in Natural Language Processing, pages 17817–17829, 2024.

- [29] Deyuan Liu, Zecheng Wang, Bingning Wang, Weipeng Chen, Chunshan Li, Zhiying Tu, Dianhui Chu, Bo Li, and Dianbo Sui. Checkpoint merging via bayesian optimization in llm pretraining. arXiv preprint arXiv:2403.19390, 2024.

- [30] Haokun Liu, Derek Tam, Mohammed Muqeeth, Jay Mohta, Tenghao Huang, Mohit Bansal, and Colin A Raffel. Few-shot parameter-efficient fine-tuning is better and cheaper than in-context learning. Advances in Neural Information Processing Systems, 35:1950–1965, 2022.

- [31] Haotian Liu, Chunyuan Li, Qingyang Wu, and Yong Jae Lee. Visual instruction tuning. Advances in neural information processing systems, 36, 2024.

- [32] Jun Liu, Zhenglun Kong, Peiyan Dong, Xuan Shen, Pu Zhao, Hao Tang, Geng Yuan, Wei Niu, Wenbin Zhang, Xue Lin, et al. Rora: Efficient fine-tuning of llm with reliability optimization for rank adaptation. In ICASSP, 2025.

- [33] Shih-Yang Liu, Chien-Yi Wang, Hongxu Yin, Pavlo Molchanov, Yu-Chiang Frank Wang, Kwang-Ting Cheng, and Min-Hung Chen. Dora: Weight-decomposed low-rank adaptation. In Forty-first International Conference on Machine Learning, 2024.

- [34] Yuan Liu, Haodong Duan, Yuanhan Zhang, Bo Li, Songyang Zhang, Wangbo Zhao, Yike Yuan, Jiaqi Wang, Conghui He, Ziwei Liu, et al. Mmbench: Is your multi-modal model an all-around player? In European conference on computer vision, pages 216–233. Springer, 2025.

- [35] Pan Lu, Swaroop Mishra, Tanglin Xia, Liang Qiu, Kai-Wei Chang, Song-Chun Zhu, Oyvind Tafjord, Peter Clark, and Ashwin Kalyan. Learn to explain: Multimodal reasoning via thought chains for science question answering. Advances in Neural Information Processing Systems, 35:2507–2521, 2022.

- [36] Pan Lu, Liang Qiu, Kai-Wei Chang, Ying Nian Wu, Song-Chun Zhu, Tanmay Rajpurohit, Peter Clark, and Ashwin Kalyan. Dynamic prompt learning via policy gradient for semi-structured mathematical reasoning. In The Eleventh International Conference on Learning Representations, 2023.

- [37] Pan Lu, Liang Qiu, Jiaqi Chen, Tony Xia, Yizhou Zhao, Wei Zhang, Zhou Yu, Xiaodan Liang, and Song-Chun Zhu. Iconqa: A new benchmark for abstract diagram understanding and visual language reasoning. In The 35th Conference on Neural Information Processing Systems (NeurIPS) Track on Datasets and Benchmarks, 2021.

- [38] Junhua Mao, Jonathan Huang, Alexander Toshev, Oana Camburu, Alan L Yuille, and Kevin Murphy. Generation and comprehension of unambiguous object descriptions. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 11–20, 2016.

- [39] Michael S Matena and Colin A Raffel. Merging models with fisher-weighted averaging. Advances in Neural Information Processing Systems, 35:17703–17716, 2022.

- [40] Fanxu Meng, Zhaohui Wang, and Muhan Zhang. Pissa: Principal singular values and singular vectors adaptation of large language models. Advances in Neural Information Processing Systems, 37:121038–121072, 2024.

- [41] Anand Mishra, Shashank Shekhar, Ajeet Kumar Singh, and Anirban Chakraborty. Ocr-vqa: Visual question answering by reading text in images. In 2019 international conference on document analysis and recognition (ICDAR), pages 947–952. IEEE, 2019.

- [42] Yuval Netzer, Tao Wang, Adam Coates, Alessandro Bissacco, Baolin Wu, Andrew Y Ng, et al. Reading digits in natural images with unsupervised feature learning. In NIPS workshop on deep learning and unsupervised feature learning, volume 2011, page 4. Granada, 2011.

- [43] Bryan A. Plummer, Liwei Wang, Christopher M. Cervantes, Juan C. Caicedo, Julia Hockenmaier, and Svetlana Lazebnik. Flickr30k entities: Collecting region-to-phrase correspondences for richer image-to-sentence models. IJCV, 123(1):74–93, 2017.

- [44] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark, et al. Learning transferable visual models from natural language supervision. In International conference on machine learning, pages 8748–8763. PMLR, 2021.

- [45] Alec Radford, Jeffrey Wu, Rewon Child, David Luan, Dario Amodei, Ilya Sutskever, et al. Language models are unsupervised multitask learners. OpenAI blog, 1(8):9, 2019.

- [46] Victor Sanh, Albert Webson, Colin Raffel, Stephen Bach, Lintang Sutawika, Zaid Alyafeai, Antoine Chaffin, Arnaud Stiegler, Arun Raja, Manan Dey, et al. Multitask prompted training enables zero-shot task generalization. In International Conference on Learning Representations, 2022.

- [47] Dustin Schwenk, Apoorv Khandelwal, Christopher Clark, Kenneth Marino, and Roozbeh Mottaghi. A-okvqa: A benchmark for visual question answering using world knowledge. In European conference on computer vision, pages 146–162. Springer, 2022.

- [48] Viraj Shah, Nataniel Ruiz, Forrester Cole, Erika Lu, Svetlana Lazebnik, Yuanzhen Li, and Varun Jampani. Ziplora: Any subject in any style by effectively merging loras. In European Conference on Computer Vision, pages 422–438. Springer, 2024.

- [49] Johannes Stallkamp, Marc Schlipsing, Jan Salmen, and Christian Igel. The german traffic sign recognition benchmark: a multi-class classification competition. In The 2011 international joint conference on neural networks, pages 1453–1460. IEEE, 2011.

- [50] George Stoica, Daniel Bolya, Jakob Brandt Bjorner, Pratik Ramesh, Taylor Hearn, and Judy Hoffman. Zipit! merging models from different tasks without training. In The Twelfth International Conference on Learning Representations, 2024.

- [51] Yi-Lin Sung, Linjie Li, Kevin Lin, Zhe Gan, Mohit Bansal, and Lijuan Wang. An empirical study of multimodal model merging. In Findings of the Association for Computational Linguistics: EMNLP 2023, pages 1563–1575, 2023.

- [52] Anke Tang, Li Shen, Yong Luo, Nan Yin, Lefei Zhang, and Dacheng Tao. Merging multi-task models via weight-ensembling mixture of experts. In Forty-first International Conference on Machine Learning, 2024.

- [53] Anke Tang, Li Shen, Yong Luo, Yibing Zhan, Han Hu, Bo Du, Yixin Chen, and Dacheng Tao. Parameter-efficient multi-task model fusion with partial linearization. In The Twelfth International Conference on Learning Representations, 2024.

- [54] Chunlin Tian, Zhan Shi, Zhijiang Guo, Li Li, and Cheng zhong Xu. HydraloRA: An asymmetric loRA architecture for efficient fine-tuning. In The Thirty-eighth Annual Conference on Neural Information Processing Systems, 2024.

- [55] Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timo- thée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, et al. Llama: Open and efficient foundation language models. arXiv preprint arXiv:2302.13971, 2023.

- [56] Alex Wang, Amanpreet Singh, Julian Michael, Felix Hill, Omer Levy, and Samuel R. Bowman. Glue: A multi-task benchmark and analysis platform for natural language understanding. In The Twelfth International Conference on Learning Representations, 2024.

- [57] Bryan Wang, Gang Li, Xin Zhou, Zhourong Chen, Tovi Grossman, and Yang Li. Screen2words: Automatic mobile ui summarization with multimodal learning. In The 34th Annual ACM Symposium on User Interface Software and Technology, pages 498–510, 2021.

- [58] Peng Wang, Shuai Bai, Sinan Tan, Shijie Wang, Zhihao Fan, Jinze Bai, Keqin Chen, Xuejing Liu, Jialin Wang, Wenbin Ge, et al. Qwen2-vl: Enhancing vision-language model’s perception of the world at any resolution. arXiv preprint arXiv:2409.12191, 2024.

- [59] Taiqiang Wu, Jiahao Wang, Zhe Zhao, and Ngai Wong. Mixture-of-subspaces in low-rank adaptation. arXiv preprint arXiv:2406.11909, 2024.

- [60] Jianxiong Xiao, Krista A Ehinger, James Hays, Antonio Torralba, and Aude Oliva. Sun database: Exploring a large collection of scene categories. International Journal of Computer Vision, 119:3–22, 2016.

- [61] Prateek Yadav, Derek Tam, Leshem Choshen, Colin A Raffel, and Mohit Bansal. Ties-merging: Resolving interference when merging models. Advances in Neural Information Processing Systems, 36, 2024.

- [62] Enneng Yang, Li Shen, Guibing Guo, Xingwei Wang, Xiaochun Cao, Jie Zhang, and Dacheng Tao. Model merging in llms, mllms, and beyond: Methods, theories, applications and opportu- nities. arXiv preprint arXiv:2408.07666, 2024.

- [63] Enneng Yang, Zhenyi Wang, Li Shen, Shiwei Liu, Guibing Guo, Xingwei Wang, and Dacheng Tao. Adamerging: Adaptive model merging for multi-task learning. In The Twelfth International Conference on Learning Representations, 2024.

- [64] Le Yu, Bowen Yu, Haiyang Yu, Fei Huang, and Yongbin Li. Language models are super mario: Absorbing abilities from homologous models as a free lunch. In Forty-first International Conference on Machine Learning, 2024.

- [65] Fanhu Zeng, Zhen Cheng, Fei Zhu, Hongxin Wei, and Xu-Yao Zhang. Local-prompt: Extensible local prompts for few-shot out-of-distribution detection. In The Thirteenth International Conference on Learning Representations, 2025.

- [66] Fanhu Zeng, Fei Zhu, Haiyang Guo, Xu-Yao Zhang, and Cheng-Lin Liu. Modalprompt: Dual- modality guided prompt for continual learning of large multimodal models. arXiv preprint arXiv:2410.05849, 2024.

- [67] Fei Zhu and Zhaoxiang Zhang. Trustlora: Low-rank adaptation for failure detection under out-of-distribution data. arXiv preprint arXiv:2504.14545, 2025.

- [68] Jiacheng Zhu, Kristjan Greenewald, Kimia Nadjahi, Haitz Sáez de Ocáriz Borde, Rickard Brüel Gabrielsson, Leshem Choshen, Marzyeh Ghassemi, Mikhail Yurochkin, and Justin Solomon. Asymmetry in low-rank adapters of foundation models. In Forty-first International Conference on Machine Learning, 2024.