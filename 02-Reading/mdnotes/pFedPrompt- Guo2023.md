# 🔤 WWW2023:pFedPrompt：联邦学习中视觉语言模型的学习个性化提示 (2023, Guo) ()

**原名：**pFedPrompt: Learning personalized prompt for vision-language models in federated learning

**译名：** WWW2023:pFedPrompt：联邦学习中视觉语言模型的学习个性化提示

**作者：**Guo et al.

**期刊：**

**IFQ：**

**DOI：** [10.1145/3543507.3583518](https://doi.org/10.1145/3543507.3583518)

**发表时间：**2023-04-30

**本地链接:** [WWW '23 The ACM Web Conference 20232023-pFedPrompt Learning personalized prompt for vision-language models in federated learning.pdf](zotero://open-pdf/0_3S2PQS2S)

**摘要翻译：** _像 CLIP 这样的预训练视觉语言模型在学习捕获用户潜在特征的表示方面显示出巨大的潜力。最近提出的一种称为上下文优化（CoOp）的方法引入了训练提示的概念，用于适应预训练的视觉语言模型。鉴于这种方法的轻量级性质，研究人员将范式从集中式系统迁移到分散式系统，以创新联邦学习（FL）的协作训练框架。然而，目前FL的提示训练主要侧重于对用户共识进行建模，缺乏对用户特征的适应，提示的个性化在很大程度上还没有得到充分的探索。过去几年的研究已经应用个性化 FL (pFL) 方法来为异构用户定制模型。不幸的是，我们发现，随着模式和训练行为的变化，直接应用 pFL 方法来提示训练会导致个性化和性能不足。为了弥补这一差距，我们提出了 pFedPrompt，它通过从语言空间学习用户共识并以非参数方式适应视觉空间中的用户特征，利用视觉语言模型中多模态的独特优势。通过这种双重协作，学习到的提示将完全个性化并符合用户的本地特征。我们在具有统计异质性的 FL 设置下对各种数据集进行了广泛的实验。结果证明了我们的 pFedPrompt 相对于具有稳健性能的替代方法的优越性。_

## 💡创新点

### (): WWW2023:pFedPrompt：联邦学习中视觉语言模型的学习个性化提示；2023-04-30

创新点：

[Go to annotation](zotero://open-pdf/library/items/3S2PQS2S?page=1367&annotation=undefined) 这一片建立在promptFL论文之上，promptFL只对文本端进行prompt微调(只有GUC，即可学习向量+【class】传入text encoder），但是该论文指出，视觉端的特征同样重要。所以添加了LFA

新增加了注意力模块：

**范式很常见**：注意力的本质就是“用 **Q** 去检索 **K/V** 并加权汇总”。pFedPrompt 采用的是一种**外部记忆（external）、非参数化**的注意力：

- **Q**：当前图像的中间空间特征 $F_s$；
- **K**：由本地 few-shot 的 $F_s$ 聚合出的类原型（记忆键）$M_k$；
- **V**：对应的 one-hot 标签（记忆值）$M_v$。

---

## 🔬实验方法：本文的实验比较新颖

## 这篇论文总结了联邦学习的个性化的常用的四个方法（3.1）：**本地微调、参数分解、正则化（FedProx）、相似度/加权聚合（AMP）**

**新问题领域（VLM 的 pFL）缺少成熟基线时，很常见用这种**方法学迁移/适配（method transfer & adaptation）**：把 pFL 里几类代表性技术——本地微调、参数分解、正则化（FedProx）、相似度/加权聚合（AMP）**

然后在实验部分分别对这四个方法使用之前的一个设置了baseline：

**PromptFL**（只联邦学软提示）；

1. **PromptFL+Finetuning**（全局提示学完再做本地微调）；
2. **PromptPer**（参数分解/解耦：全局+个性两段提示，上传聚合只走“全局段”）；
3. **PromptProx**（正则/近端项，限制本地提示别偏离全局提示太多）；
4. **PromptAMP**（相似度/加权，把其他客户端提示线性组合成“云提示”再本地更新）；

作者自己造了四个方法。论文还把这些“移植式”的做法称作**“vanilla 方法”**，指出它们**只更新文本侧的提示**、**不利用视觉特征做自适应**，因此难以充分个性化；这正是作者随后提出 pFedPrompt（GUC+LFA）的动机。

## 📜 总结：