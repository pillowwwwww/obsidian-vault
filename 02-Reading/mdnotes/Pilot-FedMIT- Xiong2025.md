# 🔤 AAAI2025: 试点：构建联合多模式指令调优框架 (2025, Xiong) AAAI ()

**原名：**Pilot: Building the federated multimodal instruction tuning framework

**译名：** AAAI2025: 试点：构建联合多模式指令调优框架

**作者：**Xiong et al.

**期刊：**Proceedings of the AAAI Conference on Artificial Intelligence

**IFQ：**

**DOI：** [10.1609/aaai.v39i20.35476](https://doi.org/10.1609/aaai.v39i20.35476)

**发表时间：**2025-04-11

**本地链接:** [2025-Pilot Building the federated multimodal instruction tuning framework.pdf](zotero://open-pdf/0_PC94NAPS)

**摘要翻译：** _本文提出了一项新颖的联邦跨模态指令调优任务（FedMIT），这对于在分布式设备上利用多种跨模态指令数据协同微调多模态大型语言模型（MLLMs）具有重要意义。为解决这一新任务，我们提出了一种联邦跨模态指令调优框架（Pilot）。我们的框架在视觉编码器与LLM的连接模块中引入了两阶段的“适配器嵌入”设计。第一阶段，我们从视觉信息中提取任务特定特征与客户端特定特征；第二阶段，我们构建跨任务混合适配器（CT-MoA）模块以实现跨任务交互。每个客户端不仅能捕捉本地数据的个性化信息并学习任务相关的跨模态知识，还能从其他任务中获取共性知识。此外，我们提出了一种文本训练参数自适应聚合策略，基于参数间的欧式距离计算权重以优化参数聚合过程，在最大化有益效应的同时有效减少负面效应。该框架能够在指令调优过程中不受任务异构性影响，通过协作利用不同客户端的分布式数据学习跨任务知识。我们的方法效果在两种不同的跨任务场景中得到了验证。_  

## 💡创新点

### 前置论文：

**最直接的前置论文里，FedIT 一定是第一梯队；FateLLM / OpenFedLLM 更像是同一条线上的「框架/工程基建」，也算近亲，但地位略次于 FedIT 本身。**

**这篇 Pilot 确实站在「多模态大模型 + 指令微调 + 联邦学习 + PEFT」四个热门赛道的交叉点，但从论文内部逻辑来看，更像是一个有明确问题设定的「交叉点工作」，而不是简单的混搭堆料。**

**谁是真「亲爹」？**

**FedIT**([Zhang 等, 2023](zotero://select/library/items/99I5MWL6))

- 这是 _概念上最直接的前置_：
    
    - 首次系统地把「instruction tuning + FL」这件事形式化成 **Federated Instruction Tuning (FedIT)**；
    - 给出具体协议：客户端本地做指令微调、服务器做参数聚合，在 **文本 LLM** 场景实现隐私保护的指令微调。
- Pilot 是在这个 **FedIT 概念** 上做「扩维」：
    
    - 从「文本-only 指令微调」→「多模态指令微调」；
    - 从「任务差异不算太大」→「故意构造跨 caption/VQA/grounding 等强 task heterogeneity 的场景」。

## 💧新名词：

### 指令调优

> **指令调优 = 用“指令→回答”这种数据形式来教模型听人话**（是“训练任务 / 数据”的概念）；  
> **Prompt Learning / Prompt Fine-tuning = 只动 prompt，不动大模型本体的一类 PEFT 技术**（是“怎么微调参数”的概念）。

以后看论文时，如果想给它贴标签，可以先问两句：

1. 它的训练数据是不是 instruction–response？
    
    - 是 → 可以叫“指令调优类”
2. 它主要调的是 prompt 还是 LoRA / Adapter / 全参数？
    
    - 只调 prompt → Prompt learning 子类
    - 调 LoRA/Adapter → LoRA/Adapter 型 PEFT
    - 全调 → Full fine-tuning

指令调优（Instruction Tuning）是用指令-回答做监督微调SFT（Supervised Fine-tuning on instruction–response pairs）。

Prompt Learning / Prompt Fine-tuning是主要调整prompt

**“指令调优”说的是“在什么数据 / 任务上训练”**，不限定你用什么 PEFT 技术。他们俩可以同时出现！一个针对数据集一个针对调什么

### 为什么学习多任务？为什么“看看别的任务在看什么”是有用的？

  
先想一个朴素场景：  
- 任务 A：图像描述（Caption）  
- 任务 B：VQA（问题：这张图里有什么？）  
- 任务 C：Grounding（标框：哪里是“绿色杯子”？）  
从人的角度想：  
- 写 Caption：你会自然而然学会“有哪些物体 + 大概场景 + 关系”；  
- 做 VQA：你会学会“根据问题，把注意力放到相关区域”；  
- 做 Grounding：你会逼着自己“真的去对齐文字和图像里的位置”。  
这三个任务看上去目标不一样，但底层都需要：  
对物体类别的辨识（cup, chair, dog...）；  
对场景/布局的大概理解（室内、街道、厨房）；  
对“语言描述 ↔ 图像区域/属性”的对齐能力。  
如果你只在 Caption 上训练，可能学得很好看图说话，但对“位置”、“数目”、“属性”等的敏感度不够。  
如果再看一看 Grounding 任务学会的“精细对齐”、VQA 学会的“根据问题选区域”，你对图像的理解会更立体。  
这就是最简单的直觉：  
不同任务在“盯图的方式”不一样：有的偏语义全局（caption），有的偏局部细节（grounding），有的偏条件选择（VQA）；  
但这些视角互补，合起来能教模型一套更强的视觉表示。

  
二、从表示学习 / 多任务学习的角度：为什么跨任务共享是“合理的赌注”？  
1. 经典的 representation learning 和 multi-task learning 有几个共识（说法不同，但核心类似）：  
 - 多个相关任务共享一个 encoder，能学到更通用、更鲁棒的特征。  
比如 ResNet 在 ImageNet 上 pretrain，再迁移到 detection / segmentation / caption，都能用，就是这个原因。  
 - 理解方式：  
每个任务都在“约束” encoder：  
Caption 强调全局语义；  
Detection / grounding 强调精细空间结构；  
分类强调 category 决边界；  
很多任务一起训练，encoder 想要同时把所有 loss 优化好，就不得不学一个兼顾多种视角的“折中”特征，而这个折中往往就是好的通用表征。  
2. 从统计角度，多任务 = 隐式的数据增强与正则化。  
单任务训练容易对自己那点数据和偏好过拟合；  
多任务共享 encoder：  
从其它任务看到更多 domain /风格/标注类型；  
模型不容易专门为某个任务学一些奇怪的“投机取巧模式”（spurious correlation）；  
这在理论和实验上都被视为一种“多任务正则化”。  
3. 尤其在视觉–语言场景，不同任务对“图–文对齐”的侧重点不同，互补很强。  
Caption 更关注“整体对齐”：一整句描述 ↔ 整幅图；  
VQA 更关注“局部对齐 +条件对齐”：具体问题 ↔ 对应区域；  
Grounding 更关注“精确区域 ↔ 文本片段”；  
OCR/VQA 又会强行要求模型学会“文字内容 + 视觉语义”两层对齐。  
所以，从理论范式上，“跨任务共享 encoder 的某些层/模块”本身就是一种非常主流且合理的选择——差别只在于：  
共享到什么程度（全部还是部分），用什么结构管理共享（hard-sharing / soft-sharing / MoE / adapter），以及怎么防止负迁移（例如只跟相近任务互相学）。

🌏研究背景：

#### FedIT([Zhang 等, 2023](zotero://select/library/items/99I5MWL6))

论文参考了FedIT这篇论文的思路，该论文[https://cs.fmy1024.cn/html_online/5358_151231670online.html](https://cs.fmy1024.cn/html_online/5358_151231670online.html)

- **客户端 (local side)** ：
    
    - 每个客户端分配一个预训练的 LLM。
    - 客户端在本地指令数据集上进行微调，更新的是 **LoRA 适配器参数** （而不是全部模型权重），从而降低计算和通信开销。
    - 本地训练完成后，把更新的适配器参数发送给服务器。
- **服务器 (server side)** ：
    
    - 聚合所有客户端上传的适配器参数。
    - 执行 **客户端选择 (client selection)** ，决定下一轮哪些客户端参与训练。
    - 重复这一过程，直到训练收敛。

但是：已有的 **FedIT**（联邦指令微调）多聚焦**NLP 单模态**任务，对**多模态**与**跨任务**情形鲜有探索。

---

#### **PROMPTFL**：（pFedPrompt 和PromptFL是同一个人写的 港理工Guo）

这篇论文中，把prompt当作联邦学习的对象，而不是聚合模型参数。  
在 PROMPTFL 里，**分类打分由“图像嵌入 vs. 文本嵌入（由提示+类名生成）”的相似度决定**；prompt作为文本端输入进入clip的text encoder里面。

预训练 VLM（如 CLIP、ALIGN）把图像和文本分别编码成向量（image encoder / text encoder），可用来做检索、打分或分类。也因此，只要给每个类别包一段“模板文本（prompt）”，就能把它“改装”为图像分类器。常见做法是把类名嵌进一句话，例如 “this is a photo of [class]”，然后对图像向量与各类别文本向量做余弦相似度并经 softmax 得到分类概率。

在联邦场景里，已有工作采用“只训练提示词参数（prompt），冻结 CLIP 主干”的范式：服务端仅聚合各客户端上传的提示参数更新，端上模型主干不动。这样能显著降低计算与通信开销，但本质上学到的是“用户共识（global user consensus）”，个性化不足。pFedPrompt 的目标就是在此基础上进一步个性化 prompt，让它更贴合各客户端的本地特征。

---

下面把 **PromptFL / pFedPrompt / FedIT** 的关键差异与共性，按你刚才关心的点（监督信号、训练对象、是否多模态、客户端架构等）汇总成一张对照表👇

**一句话对比（**：**PromptFL**与**pFedPrompt是同一个人写的，差距比较小[[pFedPrompt]]）**

- **PromptFL**：多模态 CLIP 上，**只学文本侧软提示**，聚合提示，追求**极简与低带宽**。
- **pFedPrompt**：在 PromptFL 上做**个性化**——**语言侧学共识提示**，**视觉侧用本地非参注意**把共识“落地”为个性；**只上传共识提示**。
- **FedIT**：**不是学提示**；它在 **LLM 内部**用 **LoRA** 做**SFT 监督**的联邦指令微调，**只聚合 LoRA**，把端侧**异质指令**转化为**泛化增益**。

|维度|**PromptFL (2022)**|**pFedPrompt (WWW’23)**|**FedIT / Shepherd (ICASSP’24 / arXiv)**|
|---|---|---|---|
|**任务/模态**|多模态 **CLIP**（图像+文本）|多模态 **CLIP**（图像+文本）|纯文本 **LLM**（指令微调，非多模态）|
|**监督信号（训练目标）**|图像分类/图文匹配的**判别式监督**：最大化 `image` 与 `text(prompt+class)` 余弦相似度（交叉熵）|同 PromptFL 的**判别式监督**，在此基础上做个性化|**SFT（监督式微调）**：用**指令→参考回答**的自回归交叉熵训练|
|**训练对象（被更新的参数）**|文本侧连续软提示**向量**（soft prompts）；编码器冻结|**语言侧“全局共识提示（GUC）”** + **视觉侧本地非参注意/检索（LFA）**；主干冻结|**LoRA 适配器**（线性层的低秩矩阵 **A、B**）；基座 LLM 冻结|
|**上传/聚合的对象**|仅**提示参数**（prompt 向量）|仅**共识提示**（GUC）；**LFA 为本地非参，不上传**|仅 **LoRA 的 A、B**（PEFT 子参数）|
|**是否“对输入文本微调”**|**是**：学习输入侧的可训练提示向量|**是**（语言侧提示）+ 视觉侧本地自适应|**否**：不学提示，**改模型内部映射**（LoRA）|
|**客户端架构**|CLIP（图像编码器+文本编码器）冻结；前端只接入可学习提示|同 CLIP；额外有本地 LFA 路由/检索|LLaMA-7B 等 LLM 冻结 + LoRA 插入；本地做 SFT|
|**个性化（pFL）能力**|弱（全局同一组提示）|**强**：全局**共识提示** + 本地**视觉侧**自适应（无需通信）|以**全局模型对齐/泛化**为主（可扩展个性化，但非本文重点）|
|**设计核心**|GUC：只对文本端进行prompt微调(只有GUC，即可学习向量+【class】传入text encoder）|GUC的基础上加了一个注意力模块（该论文指出，视觉端的特征同样重要。所以添加了LFA）|“**把指令微调联邦化**”：**SFT 监督 × LoRA-only 聚合**；把**异质指令**当泛化资产|
|||||
|||||
|**典型适用场景**|联邦图像分类/检索的快速适配|同上，且**端与端差异很大**的个性化场景|分布在多端的**文本指令数据**，无法集中但希望做指令微调。  <br>可以端侧推理也可服务器端推理，这不重要，重要的是FedIT解决的是“聚合得到一个更会遵循指令的全局模型|

该论文FedMIT就是在FedIT的基础上，改成了多模态的。

---

CoOP:

![](https://pic1.zhimg.com/v2-ce31f24f95d2a025020860d6e75b3292_r.jpg)

在CLIP在zero-shot预测中，text encoder输入的text是固定的，如“A photo of a {object}.”。而在CoOp中，输入的text是learnable，随着在下游任务的few-shot样本而更新。

![](https://pica.zhimg.com/v2-92934325add2f0b81d457fecf4f6c5c2_r.jpg)如上图，稍微改动一丢丢的prompt的内容，效果竟然能差这么多

## 🌟重点：

在标准 LLaVA 的官方说明里，结构是三段：

1. Vision encoder：CLIP ViT-L/14，把图像 → patch 特征矩阵 $H_v$。
2. Projector / connector：一个 MLP（文档里叫 **projection / projector / mm_projector**），把 $H_v$ 从 CLIP 的维度（例如 1024）映射到 LLM 的 embedding 维度（例如 4096）。
3. LLM：Vicuna，把 projector 输出的这些“图像 token”拼在文本 token 前面，一起做自回归生成。

**第二步的Projector / connector是在做什么？**

**它做的是“把图像信息嵌入到 LLM 能理解的向量空间里”。**

- 对 LLM 来说，输入永远是一串向量（token embedding），是“**格式统一的信号**”；
- connector 的作用是 **把视觉编码器输出的特征 → 格式转换 → LLM 的 embedding 空间**；
- 这样 LLM 的多头注意力可以在 text token 和 image token 之间做 cross-attention，从而学到“图–文对齐”。

---

具体框架（客户端）：

图像部分基座是clip+两个adpater。文字部分是一个llama作为大模型+lora模块（论文称为text-adapter?) +标签。指令 + 答案被 tokenize 成文本 token；

详细解释：

### 一、文本模块（LLM + Text-Adapter）

#### 1. 用的是什么 LLM？

- 底座是 **Vicuna-v1.5-7B**，来自 LLaVA 1.5。
- **LLM 主体是冻结的**，不直接更新大模型所有参数。2025-Pilot Building the federat…

也就是说，各个客户端拿到的是同一个 Vicuna，大部分参数都不动，只在其中插入 **LoRA 形式的 text-adapter**。

#### 2. Text-Adapter：基于 LoRA 的参数高效微调

- 在 LLM 上，他们只训练 **LoRA 层**，把它当作 **text-adapter**。
- 论文中给的配置：
    
    - LoRA rank = 64
    - 标量 α = 128
- 训练时：
    
    - **只更新 LoRA + connector 参数**
    - 其余 LLM 权重全部冻结。2025-Pilot Building the federat…

你可以理解成：

> “LLM 本体保持统一，联邦学习只在每个客户端上装一个小的 LoRA 插件，各自学习自己的指令风格和文本输出偏好。”

#### 3. 文本侧的联邦聚合：Adaptive Text-Adapter Aggregation（ATA）

Stage 1 / Stage 2 结束后，每个客户端都会把自己训练好的 **text-adapter 参数 Θ_l,k** 发回服务器。如何聚合是关键点之一。

**传统 FedAvg/FedIT 的做法**

- 通常是：所有客户端的 text-adapter 做一个 **简单加权平均**。
- 但是这里任务跨模态、跨任务差异很大，如果直接平均，很容易互相拖后腿。2025-Pilot Building the federat…

**Pilot 的做法：自适应聚合（ATA）**

1. 对每个客户端 k：
    
    - 计算它的 text-adapter 与其他客户端 text-adapter 的 **欧氏距离**：
        
        $$d_{k,i} = E(\Theta_{l,k}, \Theta_{l,i})$$
        
2. 找出 **最近的 Top-M 个客户端**（语义最相近的那几个 task）。2025-Pilot Building the federat…
3. 用 **距离的倒数** 作为权重做加权平均：
    
    $$w_{k'} \propto \frac{1}{d_{k,k'}}$$
    
4. 得到客户端 k 的聚合结果：
    
    - 一部分来自自己（带上自己数据量权重 n_k）
    - 一部分来自那 M 个“邻居客户端”。2025-Pilot Building the federat…

直观理解：

> “每个客户端只和在 text-adapter 参数空间里‘长得像自己’的那些客户端做信息融合，既吸收正面影响，又尽量避免跟完全不相关任务硬拼在一起。”

### 二、视觉模块（Visual Encoder + Connector）

#### 1. 视觉编码器：CLIP ViT-L/14

- 视觉 backbone 用的是 **CLIP ViT-L/14**（和 LLaVA 一致）。
- 在 **主方法 Pilot 中，视觉 encoder 是冻结的**，不参与训练。2025-Pilot Building the federat…

#### 2. Connector：视觉特征→LLM token

MLLM 通用结构是：

- 视觉 encoder：
    
    $$H_v = f(X_v)$$
    
- connector：
    
    $$x_{\text{img}} = \psi(H_v) \in \mathbb{R}^{N \times C}$$
    
- 然后把 $x_{\text{img}}$ 和文本指令 $x_{\text{ins}}$ 一起送进 LLM。2025-Pilot Building the federat…

在 LLaVA 里，ψ 是一个 **MLP connector**。  
在 Pilot 里，他们 **把所有“adapter on adapter”的结构都挂在 ψ 这个 connector 上**，也就是：

> “不直接改 CLIP，不直接改 LLM，本方法全都发生在中间那条视觉→文本桥上。”

#### 3. CT-CLIP（只在 ablation 里）

论文后面做了一个 ablation：

- 把 CLIP 内部的所有 MLP 层也类似地做 cross-task 训练，叫 **CT-CLIP**。
- 结果：精度略高一点，但 **激活参数和通信参数暴涨**（AP 从 0.5B → 1.75B，CP 从 0.3B → 0.5B），对联邦场景来说太贵。2025-Pilot Building the federat…

结论：

> “可以改 CLIP，但太贵了，所以主方法还是选择在 connector 上做轻量适配。”

### Stage 1：Task-specific Feature Mining（任务特征挖掘）

**目标（Goal）**

- 从每个 client 的视觉信息中，拆出两类特征：
    
    - task-specific features（任务特定特征）：同一个任务、不同 client 可共享；
    - client-specific features（客户端特定特征）：本地数据分布的个性部分。

**结构（Architecture）**

* 冻结 visual encoder：CLIP ViT-L/14，得到 $H_v = f(X_v)$。

* 两个 2-layer MLP adapter：

 * task-specific adapter $\psi_t$：学任务相关的视觉特征；

 * client-specific adapter $\psi_s$：学本地数据分布独特的视觉特征。

* 输出：

 * $x_t = \psi_t(H_v) \in \mathbb{R}^{N\times C}$

 * $x_s = \psi_s(H_v) \in \mathbb{R}^{N\times C}$

 * 图像 token：$x_{\text{img}} = x_t + x_s$（再与文本 tokens 拼接送入 LLM）。

---

**损失函数（Loss）**

1. 语言建模损失 $L_{\text{ce}}$：

  * 标准 next-token cross-entropy（给定 $x_{\text{img}}$、指令 $x_{\text{ins}}$、答案前缀，预测下一个答案 token）。

2. 差异损失（difference loss）$L_d$：

  * 形式：$L_d = | x_t^\top x_s |_F^2$（Frobenius 范数）；

  * 含义：让 $x_t$ 和 $x_s$ 在特征子空间里尽量「正交」，

    避免两个 adapter 学到过于相似的东西（强制分工）。

3. Stage 1 总 loss：

  * $L = L_{\text{ce}} + \lambda_0 L_d$，$\lambda_0$ 是权重超参（论文里取 0.1）。

---

**联邦侧（上传 & 聚合）**

1. **本地完成 Stage 1 后，每个 client 上传两类参数：**

  * text-adapter 参数 $\Theta^l$（LLM 里的 LoRA）；

  * task-specific adapter 参数 $\Theta^a$（视觉侧的 $\psi_t$）。

  * client-specific adapter $\psi_s$ 留在本地，不上传。

2. **Server 端：Task-aware Visual-adapter Aggregation（任务感知视觉聚合）**

  * 对同一任务 $t$ 的所有 client，按样本数加权平均它们的 $\Theta^a$：

    * 得到每个任务一份全局 task adapter $\bar{\Theta}^t_a$；

  * 这些结果在 Stage 2 会被下发到所有 client，组成 CT-MoA 的「任务池」。  

3. **Server 端：Adaptive Text-adapter Aggregation（自适应文本聚合，ATA）**

  * 对每个 client 的 LoRA 参数 $\Theta^l_k$，计算它与其他 client LoRA 的欧氏距离；

  * 选 Top-M 最近的 M 个邻居，做加权平均（距离越近权重越高）；

  * 这样每个 client 获得一个「从相似 client 借来的」个性化 text-adapter $\bar{\Theta}^l_k$，

    减少来自任务差异很大的 client 的负迁移。

---

**一句话 summary：Stage 1 做了什么？**

> 冻结 CLIP & LLM 主干，只在 connector 处放两个 MLP adapter，

> 把视觉特征拆成「可共享的任务特征 $x_t$」和「仅本地使用的 client 特征 $x_s$」，

> 用 $L_d$ 强行让两者在表征空间上分工，

> 然后只把 task adapter + LoRA 上传给服务器做任务感知聚合 + 自适应聚合，

> 为 Stage 2 的跨任务 CT-MoA 准备好干净、可用的任务级表示。

---

### 具体流程：

结合后面的实现细节（$R=3, E=1$）：

可以这样理解：

#### Round 1（第一次通信）

1. **下发**：
    
    - 服务器把 base LLaVA（CLIP + LLM + 原 connector）下发给所有 client。
2. **本地 Stage1（很多 backward，不是一次）**
    
    - 结构：有 $\psi_t$（task adapter）和 $\psi_s$（client adapter），没有 CT-MoA；
    - 训练：
        
        - 每个 client 在自己的数据上跑 $E$ 个 epoch（实现里 $E=1$），每个 epoch 里面有很多 step：
            
            - 对每个 batch 算 $L = L_{\text{ce}} + \lambda_0 L_d$；
            - backward，把梯度传到 $\psi_t,\ \psi_s,\ \Theta^l$（LoRA）。
    - 这段结束以后，**只是结束了 Stage1 这段训练**，不是只 backward 一次。
3. **上传 ①**：
    
    - 每个 client 把：
        
        - 任务 adapter 参数 $\Theta^a$（来自 $\psi_t$）
        - 文本 LoRA 参数 $\Theta^l$  
            上传给服务器。
4. **聚合 ①**：
    
    - 服务器做：
        
        - Task-aware aggregation：同一任务的 $\Theta^a$ 加权平均 → 得到每个任务的全局 task adapter；
        - Adaptive Text-adapter Aggregation：对 LoRA $\Theta^l$ 做「邻居加权聚合」。
5. **下发（第二次）**：
    
    - 服务器把：
        
        - $T$ 个任务的全局 task adapter
        - 每个 client 的聚合后 LoRA  
            再次下发给各个 client。
6. **本地 Stage2（又是很多 backward）**
    
    - 客户端更新结构：
        
        - 把本任务的 adapter 设为 index 1：$\psi^t_1$
        - 把其他任务的 adapter 作为 $\psi^t_2,\dots,\psi^t_T$；
        - 在 $\psi^t_i$（$i\ge2$）上加 cross-task adapter $\psi^c_i$；ψᶜᵢ **没有单独的 loss 名字**，它就是在 stage2 中，作为 $h_i(H_v)=\psi^t_i(H_v)+\psi^c_i(H_v)$ 的一部分，被 $L_{ce}$ 反向训练出来的。
        - 加 router $\phi$ 组成 CT-MoA。
    - 本地训练：
        
        - 对每个 batch 用 Stage2 的 loss： $L = L_{\text{ce}} + \lambda_1 L_b + \lambda_2 L_z$（监督正确答案+负载均衡损失 $L_b$+router z-loss $L_z$）后面两个都是MoE常用的损失
        - backward，把梯度传到：本地任务的 $\psi^t_1$、所有 $\psi^c_i$、LoRA $\Theta^l$、router $\phi$
        - client-specific adapter $\psi_s$ 在这一阶段是冻结的。
7. **上传 ②**：
    
    - Stage2 结束后，每个客户端 **再次上传**：
        
        - 当前任务的 task adapter 参数 $\Theta^a$（来自更新后的 $\psi^t_1$）；
        - LoRA 参数 $\Theta^l$
8. **聚合 ②**：
    
    - 同样再做一次 task-aware aggregation + ATA，
    - 得到这一轮的全局参数，供下一轮 round 使用。

> 所以：

- 第 1 轮：做一遍 Stage1（warm-up + 初始化本任务 adapter）、然后 Stage2；
- 第 2~R 轮：只在 Stage2 架构下继续本地训练 + 聚合（因为 task adapter / CT-MoA 结构已经有了，再回头 Stage1 意义不大，也浪费通信）。

Stage1 的主要目的在于：  
① 视觉侧做任务/客户端表征分解，并拿到每个任务的全局 task adapter；  
② 文本侧预训练并初步聚合 text-adapter，给 Stage2 一个好的起点。

Stage2中一个Loss函数同时训练客户端里面的router和CT-MOA。router还多两个自己的损失。

这样一次backward并不会让它们两个学成同一个东西，因为物理上就不是一堆参数。

- router 的参数 φ：输入 $H_v$，输出的是一组 **概率** $P$，形状 $(N, T)$，在每个 token 上 sum=1，非负。
- cross-task adapter 的参数：输入同样是 $H_v$，输出的是 **高维特征向量**，形状 $(N, C)$，可以是任意实数。
- 一个在 **概率空间**，一个在 **表示空间**，数学结构完全不一样，它们不可能“变成同一个函数”。

它们扮演的角色不同：

- router：决定“哪一个专家用多少”，只是给 $P[i]$
- cross-task adapter：决定“任务 $i$ 的特征到底长什么样”，给 $\psi^c_i(H_v)$。

## 🔬实验方法：

## 📜 总结：