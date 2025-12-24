# 🔤 联邦基础模型中的双层个性化：一种任务向量聚合方法 (2025, Yang) ()

**原名：**Bi-level personalization for federated foundation models: A task-vector aggregation approach

**译名：** 联邦基础模型中的双层个性化：一种任务向量聚合方法

**作者：**Yang et al.

**期刊：**

**IFQ：**

**DOI：** [10.48550/ARXIV.2509.12697](https://doi.org/10.48550/ARXIV.2509.12697)

**发表时间：**2025

**本地链接:** [2025-Bi-level personalization for federated foundation models A task-vector aggregation approach.pdf](zotero://open-pdf/0_UCKS2QH8)

0. 翻译摘要原文

联邦基础模型（Federated Foundation Models, FedFM）代表了一种跨客户端联合微调预训练基础模型的新范式 1。然而，针对一小群新用户或特定场景（通常数据量远少于预训练的大规模数据）微调基础模型仍是一个挑战 2。在这种背景下，个性化与联合学习之间的权衡变得更加敏感 3。为了解决这些问题，我们提出了一种用于基础模型联邦微调的双层个性化框架 4。具体而言，我们在客户端层级利用私有数据进行个性化微调，随后在服务器层级利用由客户端特定**任务向量（Task Vectors）**衡量的相似用户进行个性化聚合 5。鉴于从客户端层级微调中获得的个性化信息，服务器层级的个性化聚合能够在获取群体个性化信息的同时，减轻来自非独立同分布（non-IID）数据中无关或利益冲突客户端的干扰 6。该算法的有效性已在基准数据集的广泛实验分析中得到证实 7。

## 1. 方法动机 (Motivation)

a) 驱动力：为何要做这个？

FedFM 的目标是让大模型适应本地特定的上下文（Personalization）。但在实际操作中，不同客户端的任务可能完全不同（例如，有人做情感分析，有人做文本摘要）。如果服务器强行把所有人的模型参数平均在一起（像 FedAvg 那样），会引入“噪声”，导致模型在本地任务上表现变差（Negative Transfer） 8888。

b) 现有方法的痛点（Critical Gap）

这是本论文最犀利的洞察点：

传统“个性化FL”失效： 现有的个性化联邦学习方法（如 FedAMP）通常通过比较**模型参数（Weights）**的差异来判断两个客户端是否相似。

大模型微调的“参数惰性”： 在 FedFM 中，大家都是从同一个强大的预训练模型开始微调的，且通常使用 PEFT（如 LoRA）。这意味着，无论任务多不同，微调后的模型参数在数值上都非常相似（欧氏距离很近）。

结论： 靠“参数差异”根本分不清谁跟谁是同路人，谁是捣乱的 10。

c) 研究直觉/假设

作者引入了 “任务向量（Task Vector）” 的概念（源自 Task Arithmetic, Ilharco et al. 2022）。

假设： 虽然微调后的绝对参数很相似，但参数的变化量（$\Delta\theta = \theta_{fine-tuned} - \theta_{pre-trained}$）包含了高密度的任务方向信息。在“变化量空间”里计算相似度，比在“参数空间”里计算要准确得多 。

## 2. 方法设计 (Method Design)

这是一个双层（Bi-level）个性化框架，名为 FedBip。

a) 详细 Pipeline (输入 → 处理 → 输出)

输入： $K$ 个客户端，每个拥有私有数据 $D_k$；预训练模型 $\theta_{pre}$。

Step 1: 客户端本地微调 (Client-Level Personalization)

每个客户端 $i$ 接收上一轮专门为其聚合的模型 $\overline{\theta}_i^t$。

在本地数据 $D_i$ 上进行微调（可以是全量，也可以是 LoRA）。

输出： 本地微调后的模型 $\theta_i^t$ 12。

Step 2: 计算任务向量 (Task Vector Computation)

关键操作： 并不是直接上传参数。逻辑上，服务器计算（或客户端上传）当前模型与上一轮聚合模型的差值。

公式： $\tau_i^t = \theta_i^t - \overline{\theta}_i^t$ 13。

意义： $\tau_i^t$ 代表了这一轮训练中，为了适应本地任务，模型“想往哪里走”。

Step 3: 服务器端相似度加权 (Similarity Measurement)

服务器计算所有客户端任务向量两两之间的余弦相似度（Cosine Similarity）。

直觉： 如果你的任务向量方向跟我的一样，说明我们在学类似的东西，我应该多参考你的更新。

权重计算： 对于客户端 $i$，计算它参考客户端 $k$ 的权重 $p_{i,k}^t$：

$$p_{i,k}^t = \frac{\text{Cosine}(\tau_i^t, \tau_k^t)}{\sum_{j=1}^K \text{Cosine}(\tau_i^t, \tau_j^t)}$$

Step 4: 个性化聚合 (Server-Level Personalization)

核心差异： 不再产出一个全局模型，而是为每个客户端 $i$ 产出一个专属模型。

操作： 在上一轮模型的基础上，加上加权后的各个客户端的任务向量。

公式： $\overline{\theta}_i^{t+1} = \overline{\theta}_i^t + \sum_{k=1}^K p_{i,k}^t \tau_k^t$ 15。

注意： 这里的 $\tau_k^t$ 是其他客户端的任务向量。如果 $k$ 和 $i$ 任务冲突，权重 $p$ 会很小，从而过滤掉干扰。

Step 5 (进阶): 逐层聚合 (FedBip-L)

考虑到 Transformer 不同层（底层关注纹理/语法，高层关注语义）的差异性，上述过程可以在每一层独立进行 16。即每一层都有自己的权重矩阵。

b) 模块功能

Local Fine-tuner: 负责将通用知识转化为特定任务知识（获取 $\tau$）。

Task Vector Aggregator: 这是一个路由器和滤波器。它识别哪些 $\tau$ 是友军，哪些是噪声，并按需组装。

3. 与其他方法对比

a) 本质不同

FedAvg (传统): 盲目平均。权重通常基于数据量，不考虑任务冲突。

FedAMP (现有SOTA): 基于模型参数相似度。但在大模型微调场景下失效，因为所有人的参数都太像了。

FedBip (本文): 基于**任务向量（参数更新量）**相似度。这是在微小变动中提取特征的关键。

b) 创新点

特征空间转换： 将相似度度量从 Parameter Space 转移到了 Task Vector Space。

双层个性化： 结合了本地训练（显式个性化）和服务器端路由（隐式知识共享）。

逐层自适应： 承认大模型不同层的异质性（Layer-wise extension）。

d) 方法对比表

维度FedAvg / FedITFedAMP / FedALAFedBip (Ours)聚合目标全局统一模型个性化模型个性化模型相似度度量无（均匀/数据量加权）模型参数 ($W$)任务向量 ($\Delta W$)抗干扰能力差 (受冲突任务影响)中 (在 FedFM 中失效)高 (精准过滤冲突)计算开销低中 (需维护相似度矩阵)中 (需维护相似度矩阵)适用场景IID 数据非 IID，小模型非 IID，基础模型 (LLM/ViT)4. 实验表现与优势

a) 实验设置

CV 领域: OfficeHome 数据集 (ViT 模型)，模拟领域漂移（Domain Shift）。

NLP 领域: Flan 数据集 (LLaMA-7B + LoRA)，8个不同任务（如情感分析、摘要、纠错等），模拟任务漂移（Task Shift）     

对比基线: FedAvg, FedAMP, FedALA, FedAWA 等 17。

b) 关键结果

性能提升: 在 NLP 任务中，FedBip 平均准确率达到 81.21%，相比 FedIT (FedAvg for LLM) 的 79.05% 和 FedAMP 的 79.55% 有明显提升 18。

逐层更强: 使用逐层聚合 (FedBip-L) 进一步提升至 81.90% 19。

解决负迁移: 对于“Linguistic Acceptability”这个任务，单独训练比联合训练效果好（说明存在冲突）。FedBip 在该任务上大幅超越基线，证明它成功剔除了冲突任务的干扰 20。

c) 局限性与风险 (顾问视角的批判)

虽然论文只字未提明显的缺点，但我必须指出：

服务器端存储与计算膨胀： 这种方法需要为每个客户端维护一个模型副本（或者在计算时动态重组）。如果有 1000 个客户端，服务器就要存 1000 个模型状态，且计算 $1000 \times 1000$ 的相似度矩阵。这对大规模 FL 是一个巨大的 Scalability 瓶颈。

通信量： 虽然是 PEFT，但每个客户端每轮都要下载自己的专属模型，这使得传统的“广播”机制（一次广播，大家接收）失效，变成了“单播”，下行带宽压力增大 $K$ 倍。

冷启动问题： 在第一轮，大家都是从 Pre-trained 走的，$\tau$ 可能非常随机，相似度计算可能不稳定。

5. 学习与应用

a) 复现关键步骤

这篇论文没有提供 GitHub 链接（通常 arXiv 论文可能后续会补）。如果要复现：

数据结构： 服务器端不能只存一个 global_model。你需要一个 Dict[Client_ID, Last_Aggregated_Model]。

向量计算： 确保做减法时对应的基准是正确的（当前本地权重 - 上一轮该客户端的聚合权重）。

相似度矩阵： 使用 PyTorch 的 F.cosine_similarity 高效计算矩阵。

b) 实现建议

LoRA 适配： 既然是针对 Foundation Models，务必只对 LoRA 的 A/B 矩阵计算任务向量，不要对整个模型算，否则显存爆炸。

内存优化： 鉴于要为每个 Client 存模型，建议只存 Task Vectors 和一个 Base Model，推理时动态相加，以节省服务器显存。

c) 迁移性

该方法非常容易迁移到任何 Transformer + Adapter/LoRA 的结构中。

多模态： 可以用于联邦多模态学习，利用 $\tau$ 区分图文任务的偏好。

持续学习： 利用 Task Vector 的加减特性，甚至可以做联邦持续学习（虽然论文未提）。

6. 总结

a) 一句话核心思想

在联邦大模型微调中，利用参数更新量（任务向量）的相似度而非参数本身，来指导服务器端的加权聚合，实现精准的个性化。

b) 速记版 Pipeline (小白版)

各自修炼： 客户端拿模型在本地数据上练，算出“变化量”。

上传变化： 客户端把这个“变化量”发给服务器。

找同路人： 服务器看谁的“变化量”方向一致（计算相似度）。

定制更新： 为每个客户端，只聚合那些跟它方向一致的别人的“变化量”。

下发模型： 把拼好的专属模型发回给对应客户端。