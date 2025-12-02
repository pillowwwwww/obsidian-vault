[[2025-11-17]]这里先看了一下当前根据pilot的一些点：联邦+多模态+大模型+lora+adapter+moe/moa
https://chatgpt.com/c/69184cfb-0684-832a-ae60-5f8112fed7c0 这里面问了一些关于在pilot上面魔改创新点的问题，比如加ocr，把task放在图片外一圈传上去，具体在[[2025-11-15]]的tracking里面
---
现在看一下modal merging:
模型合并指的是将不同能力的多个模型合并为一个通用模型以处理多任务学习。[[2025-RobustMerge Parameter-Efficient Model Merging for MLLMs with Direction Robustness.pdf]] ,也叫 M-MTC[[2024-Merge, ensemble, and cooperate! A survey on collaborative strategies in the era of large language models.pdf]]

## 首先是Mix data or merge models? Optimizing for diverse multi-task learning：

在abstract中提到：
- 目前的模型已经被全球广泛采用，并且部署到多种应用中。偏好训练和安全措施往往过度拟合于西方中心化数据集中常见的危害，而安全协议经常无法扩展到**多语言环境**中。
- **多任务+多语言**，每种语言引入了独特的挑战。
- 基于语言的合并，基于目标的合并很有效。
- 模型混合比weights混合更有效。

## What Matters for Model Merging at Scale?
- 比较了四种流行的合并方法——平均值、任务算术、Dare-TIES和TIES合并——对从1B到64B参数的不同模型规模进行合并。
- 在零样本上进行评估
- 当合并八个大型专家模型时，合并后的模型通常比多任务训练模型泛化得更好。
- 使用较大的模型时，可以更好地合并更多的专家模型。

## 联邦学习的模型聚合 vs 近年的 Model Merging，本质区别是什么？
> 很粗暴地说：
> * 联邦聚合 = **“训练过程中的一个优化步骤”**；
> * 模型合并 = **“训练完之后的一个参数算术/后处理操作”**。

使用时机与过程不一样
**联邦学习聚合：**
* 是**迭代过程中的一环**：
1. 服务器下发当前全局模型 (w_t)
2. 客户端在本地数据上做几步 SGD，得到 $w_t^{(k)}$
3. 服务器聚合：$w_{t+1} = \sum_k p_k w_t^{(k)}$
4. 重复多轮。
* 每一轮的本地模型都是“**从同一个当前全局模型出发的小步更新（small update around (w_t)）**”。

**模型合并：**
* 多数情况下是**一次性（one-shot）或少数几步**：
* 先分别把 experts 训练/微调到各自的收敛点；
* 合并时**不再继续访问训练数据**；
* 直接做权重平均、task vector 加法、TIES 那样的操作。
* 合并的对象通常是“**已经各自走到不同 basin 的独立最优点**”，不是围绕某个共同的当前点的小更新。

是否认为FedAvg的本质就是FFT合并？FedAvg标准过程就是客户端进行全量微调（FFT），服务器对全量参数进行平均聚合。这可以被视为最基础的“模型合并”。

---
## 知乎的总结
[(99+ 封私信 / 7 条消息) 首页 - 知乎](https://www.zhihu.com/column-square)
#### 两个角度去评价融合后的模型：
- In-Domain Performance:**在同任务上，融合后的单一模型，和融合前的各个模型相比，表现如何？** 很随机，并不好。DARE + Task-Arithmetic 貌似好一点
- OOD:从`TIES`论文的实验来看，融合模型的OOD Generalization能力还不错，说明**融合模型在一定程度上学到了鲁棒的task解决能力**。

还可以从已见任务和未见任务上去评估。

**其他使用场景**：
模型融合还有一些使用场景：
1. 在一次训练当中，融合多个checkpoint，以提升模型的训练效果；
2. 将融合模型作为进一步Fine-Tuning的起点，以取得更好的FT效果；
3. `Task Vector`不仅可以用于加（即模型融合），也可以用于减（即让模型遗忘某些能力）。
常见大模型融合算法：大模型模型融合算法（Ties、Slerp、Task Arithmetic、DARE、Passthrough）
[浅析大模型模型融合算法（Ties、Slerp、Task Arithmetic、DARE、Passthrough） - 文章 - 开发者社区 - 火山引擎](https://developer.volcengine.com/articles/7390576746635984932)
#### 这些融合算法分为两类：参数空间（Parameter-space） vs. 函数空间（Function-space）
![[image-98.png]]
![[image-93.png|682x323]]
![[image-94.png|681x265]]
![[image-95.png|682x449]]
![[image-96.png|480x229]]

---
## 发展过程
#### 阶段一：同分布下的均值融合 (The Era of Generalization)
早期的模型融合主要关注如何让同一个模型在同一任务上获得更好的泛化能力，通常是对不同训练阶段或不同初始化的模型进行平均。
- **Stochastic Weight Averaging (SWA) (UAI 2018 / NeurIPS 2019)**
    - **核心思想：** 在训练轨迹上对权重进行平均，可以找到更宽的极小值点，从而提高泛化性。
- **Model Soups (ICML 2022)**
    - **核心思想：** 针对预训练模型进行多次微调（不同的超参数），然后贪婪地平均这些模型的权重。
    - **贡献：** 证明了简单的参数平均（Parameter Averaging）可以在不增加推理成本的情况下显著提升 ImageNet 等任务的精度。
#### 阶段二：任务算术与多任务融合 (The Era of Task Arithmetic & FFT)
随着大模型（LLM）的出现，研究者开始关注如何将不同任务的模型融合为一个“多面手”。这一阶段主要针对 **Full Fine-Tuning (FFT)** 的模型。
- **Task Arithmetic (ICLR 2023)**       
- **Git Re-Basin (ICLR 2023)**       
- **Ties-Merging (NeurIPS 2023)** 
- **DARE (ICML 2024)** 

#### 阶段三：参数高效与模块化融合 (The Era of PEFT & Modularity)
由于 LLM 参数量过大，全量微调成本太高，社区转向 PEFT（如 LoRA）。这一阶段的研究重点是如何高效融合这些低秩模块。
- **LoraHub (EMNLP 2023)** 10101010
    - **核心思想：** 试图将多个 LoRA 模块视为组件。但它需要“测试时适应”（Test-time adaptation），即在推理阶段需要少量的样本来计算每个 LoRA 的权重系数，这不是完全的 Training-free。    
- **ZipLoRA (arXiv 2023 / ECCV 2024)** 11
    - **核心思想：** 针对风格化生成，解决不同 LoRA 融合后风格和内容混淆的问题。       
- **RobustMerge (NeurIPS 2025 - 本文)** 12
    - **核心思想：** 针对 LoRA 的低秩特性，提出了基于 SVD 的“方向鲁棒性”理论。   
    - **创新点：** 不同于 Ties/DARE 处理数值大小，RobustMerge 通过修剪小幅度参数并**动态调整奇异值（Scaling Singular Values）**来维持低秩空间中的方向稳定性，从而实现无训练、无额外数据的多模态模型融合。

---
## 合并和集成
[[2024-Merge, ensemble, and cooperate! A survey on collaborative strategies in the era of large language models.pdf]]
明确界定了“合并”的独特性：
- **参数空间 (Parameter Space)：** Merging 特指在**参数空间**中通过算术运算将多个模型整合成一个新模型 。
- **对比 Ensemble (集成)：** 集成是在**输出空间**（如 token 概率或生成的文本）结合多个模型的结果，而不改变模型本身的参数 。合并的一个显著优势是不像集成那样需要在推理时同时运行多个模型，因此推理效率更高。
- ![[image-88.png]]

## 未来挑战和方向：

挑战：挑战与未来方向：灵活的模型合并 (Flexible LLMs Merging)
在论文的第 6 章（挑战与未来方向）中，作者探讨了突破当前限制的可能性：

- **异构模型合并的探索：** 尽管目前很难合并不同架构的模型，但有研究发现可以通过利用**重叠的 token** 作为桥梁，将不同 LLM 的 token embedding 投影到公共空间 。
    
- **神经元对齐 (Neuron Alignment)：** 论文提到了一些尚未在 LLM 中广泛探索的高级技术，如 OT Fusion（最优传输融合）、Re-Basin 技术和 REPAIR。这些方法旨在解决神经元排列不一致的问题，未来有望用于实现更灵活的模型融合.

--- 
## 
模型合并的三种场景：联邦学习、微调、蒸馏

---
## [[RobustMerge-Zeng2025]]
该论文中提到：
- PEFT 合并的核心挑战不是解决微小的符号抖动，而是防止因忽略参数幅值差异而导致的**方向不稳定性（Direction Instability）**：
- 如果不进行补偿性缩放，那些幅值偏小（但未被剪枝）的奇异向量本身就天然面临“方向偏移过大”的风险。
- 剪枝（Pruning）去掉的是那些**极小**的、被视为噪音或对方向贡献微乎其微的参数。这本身是为了减少干扰。
- 因此，合并的焦点应从“消除冲突”转向“维护方向”
- 作者的解决方案是互补参数缩放

---
## FedMerge
当前痛点：
- FL中，一个全局模型可能不足以服务于具有非IID任务和分布的众多客户端。
- **破坏个性化（Erasing Personalization）：** FedAvg这种“强制覆盖”会抹除客户端在上一轮训练中积累的、适应其本地数据的“个性化”偏置；客户端漂移（进行一下对比）
- 那能否在下发了之后，在客户端（FedMerge是从服务器上）进行一个模型合并（剪枝+缩放？）再去本地训练呢？

论文指出，由于 Non-IID 分布，单一全局模型不足以服务所有客户端 。这篇论文引用的 **FedEM** (Federated Multi-Task Learning) 直接将 FL 建模为多任务学习问题，认为每个客户端的数据是多个潜在任务分布的混合 。

随着大模型引入联邦学习，FL 的目标从“训练一个分类器”转变为“通过联邦微调让大模型具备跨任务能力”。

#### FedMerge的局限：无法针对零样本
改进策略：引入语义先验与元学习初始化
为了解决这一问题，必须打破对本地梯度的绝对依赖，引入额外的“先验”信息源。结合 **Pilot** 1 框架中的路由思想与其他元学习策略，我们可以设计以下三种改进方案：
- 策略一：基于语义特征的轻量级路由初始化（The Pilot Approach）

**Pilot** 框架 1 在其第二阶段（Cross-Task Visual Interaction）中引入了一个名为 **CT-MoA**（Cross-task Mixture-of-Adapters）的模块。该模块包含一个路由器（Router），其工作机制是根据输入图像的视觉特征 $H^v$ 来动态预测适配器的选择概率：

$$\mathcal{P} = \text{Softmax}(\phi(H^v)) \in \mathbb{R}^T$$

虽然 FedMerge 为了推理效率放弃了动态路由（详见第3章分析），但我们可以借鉴这一思想用于 **初始化阶段**。
- **改进机制：** 在客户端注册阶段，允许客户端上传少量的、脱敏的元数据（Metadata）或任务描述符（Task Descriptor）。例如，客户端可以上传其数据分布的均值向量、类别分布的文本描述，或者在本地运行一个冻结的编码器得到的特征嵌入均值。
- 服务器端映射网络： 服务器端训练一个轻量级的超网络（Hypernetwork）或映射函数 $M(\cdot)$，输入为任务描述符，输出为初始融合权重 $w_{init}$。
    
    $$w_{init} = M(\text{TaskDescriptor}_i)$$
    
- **效果：** 这样，即使没有标签数据进行反向传播，服务器也能根据任务的语义相似性（例如，“这是一个医疗影像任务”）智能地赋予相关的全局模型（如“医疗专家模型”）更高的初始权重。这就是利用语义“先验”替代了梯度“后验”。
- 策略二：基于聚类的冷启动策略（Clustering Cold-Start）
借鉴 **IFCA**（Iterative Federated Clustering Algorithm）的思想（提及于 1），我们可以假设客户端并非均一分布，而是聚集成若干个簇。
- **改进机制：** 服务器维护 $K$ 组典型的权重配置（代表 $K$ 种常见的数据分布模式）。
- **推理时选择：** 对于零样本客户端，服务器下发这 $K$ 个预设的融合模型。客户端在无标签数据上进行推理，利用无监督指标（如预测熵、置信度或困惑度 Perplexity）来评估哪个模型最“适应”本地数据。
- **决策：** 客户端选择无监督指标最优的那个模型对应的权重作为初始 $w$。这相当于利用群体知识构建了离散的先验分布。
#### 融合的好处：高效性
pilot这样传统的联邦学习，激活的experts太多了，一个experts就大，都放在客户端本地保存N个专家的参数，频繁切换，密集激活。
如果是merge：- **存储负担：** 无论服务器端用了 10 个还是 100 个专家来合成，客户端只需要存储 **1 个** 模型的大小。
- **计算量：** 推理 FLOPs（浮点运算次数）严格等于单模型推理，没有任何额外开销。
- **延迟：** 享受标准的静态图优化，延迟最低。



---

## 我的想法：
#### 1. 如何定义传统联邦学习和模型合并的关系？
FedAvg 本质就是 FFT 版本的“基础模型合并”，把**FedAvg 可以视为最基础、最原始的“模型合并”算法**。
那么现在，取代传统的 FedAvg(FFT模型合并)，选择PEFT model merging，本质上就是将“全量参数的合并”变成了“PEFT 模块的合并”。

Robust Merge这篇论文发现，直接把FFT的模型合并算法用到PEFT上面，表现不好。
那么：
- 需要**针对联邦PEFT 特性**设计零样本（Unseen Task）合并算法。
- 【在实际联邦中，经常有新客户端加入，带来一个全新的任务（例如 Client New 想做“表情包识别”，这是之前系统里没有的任务）。】


- **Pilot 的缺陷：** 在 Pilot 论文中，针对 Text-Adapter 的聚合，作者提出了一种“自适应聚合”策略，其核心是计算参数之间的**欧几里得距离 (Euclidean Distance)**，并选择距离最近的 Top-M 个客户端进行加权平均 。
    
- **RobustMerge 的反驳：** RobustMerge 明确指出，直接对 PEFT 参数进行基于数值（或简单距离）的合并是次优的，因为它忽略了**奇异值向量的方向 (Direction of Singular Vectors)** 。仅仅因为欧氏距离近就聚合，可能会导致虽然距离近、但“主方向”不一致的 Adapter 互相抵消，破坏特定任务的知识。

方法：
merge里面提到的合并算法+模型池改为（池子里 Global Models改为“VQA Expert”、“Caption Expert”的 Adapter）

#### 2. W0​ 冻结，给每个任务训练一个ΔWi，所有“model merging”本质上就是在玩LoRA的 ΔWi


在服务器上去合并不同的模型，选一个最合适的分数最高的发给客户端

#### 3. 把robustmerge的方法迁移到联邦学习里面
- 最近有研究表明，去除 Adapter 中的非线性激活函数（Linear Adapter）在许多任务上效果相当。如果采用 Linear Adapter，其数学形式与 LoRA 几乎一致（都是低秩矩阵相乘），RobustMerge 的方法可以直接迁移



- 可以做成 **A 型**：每轮把 LoRA/adapter 传到 server，server 用“合并算法”替换掉 FedAvg/加权平均，在训练循环里用；
    
- 也可以做成 **B 型**：本地训完再一次性把 LoRA/adapter（或其压缩统计）传上来，只做一次/少数几次 **训练免合并**，这才是典型的 “model merging”。
总结：
 您应该采纳 RobustMerge 关于“干扰机制”的理论洞察（RobustMerge 的发现完全适用于 FL：简单的参数平均会破坏 LoRA 适配器中的细微方向信息，导致全局模型性能下降，特别是在异构数据环境下。），但在算法实现上，应转向 **LoRM**。LoRM 通过数学手段解决了 RobustMerge 试图通过剪枝解决的问题，且更适合联邦学习的去中心化特性。


$$ \mathcal{L}_{local} = \mathcal{L}_{CE}(\theta_{base} + B_k A_k; D_k) + \lambda | B_k^T B_k - I |_F^2 $$

#### 4. 根据文本模特的稳定性去稳定视觉模态
**“因为我们观察到 Client A 和 Client B 的文本模态达成了某种共识（例如参数距离近、或谱特征一致），所以我们有理由相信并在数学上赋予更高的权重，去强制融合它们原本发散的视觉模态**