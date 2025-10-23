---
Title: "Learning transferable visual models from natural language supervision" 
Year: 2021 
Authors: Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark, Gretchen Krueger, Ilya Sutskever 
Tags: FOS: Computer and information sciences, Machine Learning (cs.LG), Computer Vision and Pattern Recognition (cs.CV)
---

Zotero PDF Link: [2021-Learning transferable visual models from natural language supervision.pdf](zotero://select/library/items/ARSJ6YDZ)
Related::  

### 持续笔记
%% begin notes %% 
Write notes here! 
 %% end notes %% 
### 文中笔记
  
 

使用对比学习的自监督训练，让模型自己从数据集上做一个学习。没有经过任何的有监督训练。训练出的模型达到跟有监督训练训出的经典模型同样的效果，甚至更好，并且做了大量的实验证实这件事。 [Page 1](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=MKDZRHSD) 
 
 
创新点 
  <mark class="hltr-yellow">"After pre-training"</mark> [Page 1](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=F54USYLN) 
 
 
创新点 
  <mark class="hltr-yellow">"zero-shot tra"</mark> [Page 1](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=9WICKQF3) 
 
 
创新点 
  <mark class="hltr-yellow">"For instance, we match the ac"</mark> [Page 1](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=63PYHN6U) 
 
 
创新点 
  <mark class="hltr-yellow">"curacy of the original ResNet-50 on ImageNet"</mark> [Page 1](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=99R5SWCK) 
 
 
创新点 
  <mark class="hltr-yellow">"zero-shot without n"</mark> [Page 1](zotero://open-pdf/library/items/ARSJ6YDZ?page=1&annotation=6AFVAYPN) 
 
 
创新点 
  

从网上收集了4亿张图片文本对，送入文本和image编码器 [Page 2](zotero://open-pdf/library/items/ARSJ6YDZ?page=2&annotation=5N98DVSY) 
 
 
创新点 
  

CLIP为什么这么好？因为它已经训练好了这两个encoder，我们可以直接拿过来用。期间可能会涉及到一些模态融合，对齐的问题。 [Page 2](zotero://open-pdf/library/items/ARSJ6YDZ?page=2&annotation=ZLIVQZAN) 
 
 
创新点 
  
 
 
重点 
  

提示词工程 [Page 2](zotero://open-pdf/library/items/ARSJ6YDZ?page=2&annotation=QLYASGHM) 
 
 
创新点 
  <mark class="hltr-yellow">"r zero-shot prediction"</mark> [Page 2](zotero://open-pdf/library/items/ARSJ6YDZ?page=2&annotation=WPQKFV9K) 
 
 
创新点 
  <mark class="hltr-yellow">"Figure 5. Zero-shot CLIP is competitive with a fully supervised baseline. Across a 27 dataset eval suite, a zero-shot CLIP classifier outperforms a fully supervised linear classifier fitted on ResNet-50 features on 16 datasets, including ImageNet."</mark> [Page 8](zotero://open-pdf/library/items/ARSJ6YDZ?page=8&annotation=95PSAYGF) 
 
 
创新点 
  

视觉图片的变体数据集
在这些数据集上，比传统的模型表现好 [Page 15](zotero://open-pdf/library/items/ARSJ6YDZ?page=15&annotation=K3UHCPA8) 
 
 
创新点 
  
 
 
重点 
  
 
 
重点 
  

数据重叠：4亿的照片里，可能会和imagenet数据集的图片一样（泄题了） [Page 19](zotero://open-pdf/library/items/ARSJ6YDZ?page=19&annotation=HM2QLG3Y) 
 
 
创新点 
  <mark class="hltr-yellow">"detected data overlap"</mark> [Page 19](zotero://open-pdf/library/items/ARSJ6YDZ?page=19&annotation=978FWYVP) 
 
 
创新点 
  

并不是在训练CLIP前，把数据集去重。该实验是通过先在“评测数据集”里用近重复检测器把样本分成两部分——与预训练集（WIT-400M）相似度高的记为 Overlap，其余为 Clean；然后同一个已训练好的 CLIP分别在 Overlap / Clean / All 三个切分上做零样本评测，用 All − Clean 估计“重叠”可能带来的整体精度抬高，并做单侧二项检验与 99.5% Clopper–Pearson 置信区间判断显著性 [Page 19](zotero://open-pdf/library/items/ARSJ6YDZ?page=19&annotation=UIW4YC25) 
 
 
创新点 
 

%% Import Date: 2025-10-23T21:00:41.280+08:00 %%
