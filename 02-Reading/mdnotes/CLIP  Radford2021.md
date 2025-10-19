# 🔤 从自然语言监督中学习可迁移的视觉模型 (2021, Radford) ()

**原名：**Learning transferable visual models from natural language supervision

**译名：** 从自然语言监督中学习可迁移的视觉模型

**作者：**Radford et al.

**期刊：**

**IFQ：**

**DOI：** [10.48550/ARXIV.2103.00020](https://doi.org/10.48550/ARXIV.2103.00020)

**发表时间：**2021

**本地链接:** [2021-Learning transferable visual models from natural language supervision.pdf](zotero://open-pdf/0_ARSJ6YDZ)

**摘要翻译：** _最先进的计算机视觉系统经过训练可以预测一组固定的预定对象类别。这种受限的监督形式限制了它们的通用性和可用性，因为需要额外的标记数据来指定任何其他视觉概念。直接从原始文本中学习图像是一种很有前途的替代方案，它利用了更广泛的监督来源。我们证明，预测哪个标题与哪个图像对应的简单预训练任务是一种高效且可扩展的方法，可以在从互联网收集的 4 亿对（图像、文本）数据集上从头开始学习 SOTA 图像表示。预训练后，使用自然语言来引用学习的视觉概念（或描述新的视觉概念），从而实现模型零样本传输到下游任务。我们通过对 30 多个不同的现有计算机视觉数据集进行基准测试来研究这种方法的性能，涵盖 OCR、视频中的动作识别、地理定位和多种类型的细粒度对象分类等任务。该模型可以轻松地迁移到大多数任务，并且通常可以与完全监督的基线竞争，而无需任何数据集特定的训练。例如，我们在 ImageNet 零样本上匹配原始 ResNet-50 的准确性，而无需使用其所训练的 128 万个训练样本中的任何一个。我们在 https://github.com/OpenAI/CLIP 发布了代码和预训练模型权重。_

## 💡创新点

### 多模态方向的基石论文 CLIP

([Radford 等, 2021, p. 1](zotero://select/library/items/7JW885XC)) 使用对比学习的自监督训练，让模型自己从数据集上做一个学习。没有经过任何的有监督训练。训练出的模型达到跟有监督训练训出的经典模型同样的效果，甚至更好，并且做了大量的实验证实这件事。

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=undefined) “After pre-training” ([Radford 等, 2021, p. 1](zotero://select/library/items/7JW885XC))

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=undefined) “zero-shot tra” ([Radford 等, 2021, p. 1](zotero://select/library/items/7JW885XC))

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=undefined) “For instance, we match the ac” ([Radford 等, 2021, p. 1](zotero://select/library/items/7JW885XC))

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=undefined) “curacy of the original ResNet-50 on ImageNet” ([Radford 等, 2021, p. 1](zotero://select/library/items/7JW885XC))

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=undefined) “zero-shot without n” ([Radford 等, 2021, p. 1](zotero://select/library/items/7JW885XC))

([Radford 等, 2021, p. 2](zotero://select/library/items/7JW885XC)) 从网上收集了4亿张图片文本对，送入文本和image编码器

([Radford 等, 2021, p. 2](zotero://select/library/items/7JW885XC)) 提示词工程

## 💧新名词：

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=2&annotation=undefined) “r zero-shot prediction” ([Radford 等, 2021, p. 2](zotero://select/library/items/7JW885XC))

## 🌏研究背景：

## 🌟重点：

## 🔬实验方法：

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=8&annotation=undefined) “Figure 5. Zero-shot CLIP is competitive with a fully supervised baseline. Across a 27 dataset eval suite, a zero-shot CLIP classifier outperforms a fully supervised linear classifier fitted on ResNet-50 features on 16 datasets, including ImageNet.” ([Radford 等, 2021, p. 8](zotero://select/library/items/7JW885XC))

([Radford 等, 2021, p. 15](zotero://select/library/items/7JW885XC)) 视觉图片的变体数据集  
在这些数据集上，比传统的模型表现好

([Radford 等, 2021, p. 19](zotero://select/library/items/7JW885XC)) 数据重叠：4亿的照片里，可能会和imagenet数据集的图片一样（泄题了）

[Go to annotation](zotero://open-pdf/library/items/ARSJ6YDZ?page=19&annotation=undefined) “detected data overlap” ([Radford 等, 2021, p. 19](zotero://select/library/items/7JW885XC))

([Radford 等, 2021, p. 19](zotero://select/library/items/7JW885XC)) 并不是在训练CLIP前，把数据集去重。该实验是通过先在“评测数据集”里用近重复检测器把样本分成两部分——与预训练集（WIT-400M）相似度高的记为 Overlap，其余为 Clean；然后同一个已训练好的 CLIP分别在 Overlap / Clean / All 三个切分上做零样本评测，用 All − Clean 估计“重叠”可能带来的整体精度抬高，并做单侧二项检验与 99.5% Clopper–Pearson 置信区间判断显著性。

![[image-59.png]]

总结：`All − Clean` 恰好等于“重叠比例 ×（重叠相对更容易的幅度）”，因此是“数据重叠对总体精度的抬高量”的自然估计；而由于重叠比例很小，抬高也就普遍很小。([arXiv](https://arxiv.org/pdf/2103.00020 "Learning Transferable Visual Models From Natural Language Supervision"))

在测试数据重叠的时候，可以用这个实验方法。

## 📜 总结：