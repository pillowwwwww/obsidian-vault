[[2025-11-17]]这里先看了一下当前根据pilot的一些点：联邦+多模态+大模型+lora+adapter+moe/moa
https://chatgpt.com/c/69184cfb-0684-832a-ae60-5f8112fed7c0 这里面问了一些关于在pilot上面魔改创新点的问题，比如加ocr，把task放在图片外一圈传上去，具体在[[2025-11-15]]的tracking里面
---
现在看一下modal merging:

## 首先是Mix data or merge models? Optimizing for diverse multi-task learning：

在abstract中提到：
- 目前的模型已经被全球广泛采用，并且部署到多种应用中。偏好训练和安全措施往往过度拟合于西方中心化数据集中常见的危害，而安全协议经常无法扩展到多语言环境中。
-  多任务+多语言，每种语言引入了独特的挑战。
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
---
## 知乎的总结
[(99+ 封私信 / 7 条消息) 首页 - 知乎](https://www.zhihu.com/column-square)
- **在同任务上，融合后的单一模型，和融合前的各个模型相比，表现如何？** 很随机，并不好。DARE + Task-Arithmetic 貌似好一点
- 从`TIES`论文的实验来看，融合模型的OOD Generalization能力还不错，说明**融合模型在一定程度上学到了鲁棒的task解决能力**。

**其他使用场景**：
模型融合还有一些使用场景：
1. 在一次训练当中，融合多个checkpoint，以提升模型的训练效果；
2. 将融合模型作为进一步Fine-Tuning的起点，以取得更好的FT效果；
3. `Task Vector`不仅可以用于加（即模型融合），也可以用于减（即让模型遗忘某些能力）。
常见大模型融合算法：大模型模型融合算法（Ties、Slerp、Task Arithmetic、DARE、Passthrough）
[浅析大模型模型融合算法（Ties、Slerp、Task Arithmetic、DARE、Passthrough） - 文章 - 开发者社区 - 火山引擎](https://developer.volcengine.com/articles/7390576746635984932)

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

- **Task Arithmetic (ICLR 2023)** 7777
    
    - **论文：** _Editing Models with Task Arithmetic_
        
    - **核心思想：** 定义“任务向量”（Task Vector $\tau = \theta_{ft} - \theta_{pre}$）。通过 $\theta_{new} = \theta_{pre} + \tau_A + \tau_B$，可以通过简单的加法赋予模型新能力，或通过减法消除有害能力（如毒性）。
        
- **Git Re-Basin (ICLR 2023)**
    
    - **论文：** _Git Re-Basin: Merging Models modulo Permutation Symmetries_
        
    - **核心思想：** 解决了不同初始化的模型权重排列不一致的问题（Permutation Symmetry），通过重新排列神经元使得两个独立训练的模型可以融合（但在大模型上计算成本极高）。
        
- **Ties-Merging (NeurIPS 2023)** 8888
    
    - **论文：** _Ties-Merging: Resolving Interference When Merging Models_
        
    - **核心思想：** 指出多任务融合的主要障碍是“参数干扰”。提出了三个步骤：Trim（修剪冗余参数）、Elect Signs（解决符号冲突）、Merge（合并）。这是 FFT 融合的经典 Baseline。
        
- **DARE (ICML 2024)** 9999
    
    - **论文：** _Language Models are Super Mario: Absorbing Abilities from Homologous Models as a free lunch_
        
    - **核心思想：** 发现 SFT（监督微调）后的参数增量主要也是冗余的。通过随机丢弃（Drop）绝大部分更新（例如 90%）并重新缩放（Rescale）剩余参数，可以几乎无损地融合多个大模型。
        

#### 阶段三：参数高效与模块化融合 (The Era of PEFT & Modularity)

由于 LLM 参数量过大，全量微调成本太高，社区转向 PEFT（如 LoRA）。这一阶段的研究重点是如何高效融合这些低秩模块。

- **LoraHub (EMNLP 2023)** 10101010
    
    - **核心思想：** 试图将多个 LoRA 模块视为组件。但它需要“测试时适应”（Test-time adaptation），即在推理阶段需要少量的样本来计算每个 LoRA 的权重系数，这不是完全的 Training-free。
        
- **ZipLoRA (arXiv 2023 / ECCV 2024)** 11
    
    - **核心思想：** 针对风格化生成，解决不同 LoRA 融合后风格和内容混淆的问题。
        
- **RobustMerge (NeurIPS 2025 - 本文)** 12
    
    - **核心思想：** 针对 LoRA 的低秩特性，提出了基于 SVD 的“方向鲁棒性”理论。
        
    - **创新点：** 不同于 Ties/DARE 处理数值大小，RobustMerge 通过修剪小幅度参数并**动态调整奇异值（Scaling Singular Values）**来维持低秩空间中的方向稳定性，从而实现无训练、无额外数据的多模态模型融合。

