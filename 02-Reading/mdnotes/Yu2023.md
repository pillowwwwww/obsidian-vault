---
Title: "Multimodal federated learning via contrastive representation ensemble" 
Year: 2023 
Authors: Qiying Yu, Yang Liu, Yimu Wang, Ke Xu, Jingjing Liu 
Tags: FOS: Computer and information sciences, Machine Learning (cs.LG), Artificial Intelligence (cs.AI), CreamFL, 多模态联邦
---

Zotero PDF Link: [2023ICLR-Multimodal federated learning via contrastive representation ensemble.pdf](zotero://select/library/items/4XUGSA5P)
Related::  

### 持续笔记
%% begin notes %% 
Write notes here! 
 %% end notes %% 
### 文中笔记
  
 

提出了两个问题：之前的多模态联邦学习无法解决 客户端异构和模型漂移 [Page 1](zotero://open-pdf/library/items/4XUGSA5P?page=1&annotation=9TMFYN8Z) 
 
  <mark class="hltr-yellow">"limited"</mark> [Page 1](zotero://open-pdf/library/items/4XUGSA5P?page=1&annotation=ZJ3THFV5) 
 
  <mark class="hltr-yellow">"[[模型漂移model drift]]"</mark> [Page 1](zotero://open-pdf/library/items/4XUGSA5P?page=1&annotation=297C4I5I) 
 
  <mark class="hltr-yellow">"KD-based"</mark> [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=UFZAS84F) 
 
  <mark class="hltr-yellow">"representation-level ensemble knowledge transfer."</mark> [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=LVDKARXS) 
 
  <mark class="hltr-yellow">"ensemble"</mark> [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=97KFWIIQ) 

ensemble集成学习:将两个或多个学习器（例如回归模型、神经网络）聚合在一起，以产生更准确的预测。换句话说，集成模型结合了多个独立的模型，从而比单个模型产生更准确的预测。 [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=97KFWIIQ) 
 
  <mark class="hltr-red">"global-local cross-modal contrastive aggregation strategy,"</mark> [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=CH7QAPVW) 
 
  <mark class="hltr-red">": modality gap and task gap"</mark> [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=G824PPVQ) 
 
  

inter-model 模态间；intra-model模态内部的 [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=9QX9STI6) 
 
  <mark class="hltr-red">"An inter-modal contrastive objective is designed to mitigate the modality gap, by performing cross-modality contrasts using public data in the local training phase, which complements for the information of the absent modality in unimodal clients."</mark> [Page 2](zotero://open-pdf/library/items/4XUGSA5P?page=2&annotation=3CZMNC8G) 
 
  <mark class="hltr-red">"A key goal of multimodal learning is to unify signals from different modalities into the same vector space, where semantically correlated data across modalities share similar representations. T"</mark> [Page 3](zotero://open-pdf/library/items/4XUGSA5P?page=3&annotation=ZFW8UWF2) 
 
  <mark class="hltr-orange">"3.2.1 INTER-MODAL CONTRAST"</mark> [Page 4](zotero://open-pdf/library/items/4XUGSA5P?page=4&annotation=P86EG2PM) 
 
  
 
 

%% Import Date: 2025-05-17T13:39:07.631+08:00 %%
