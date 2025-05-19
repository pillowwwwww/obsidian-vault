---
Title: "Federated optimization in heterogeneous networks" 
Year: 2018 
Authors: Tian Li, Anit Kumar Sahu, Manzil Zaheer, Maziar Sanjabi, Ameet Talwalkar, Virginia Smith 
Tags: FedProx, 基本算法
---

Zotero PDF Link: [2018-Federated optimization in heterogeneous networks.pdf](zotero://select/library/items/WK9JU8GB)
Related::  

### 持续笔记
%% begin notes %% 
#### 背景
##### 提出了两个异构：系统异构和统计异构
		
- **系统异构**(system heterogeneity)：一个联邦学习的系统中存在许多设备，这些设备的计算能力、存储能力、网络通信能力不尽相同，等待落后的局部模型会拖慢整个系统的训练速度，而抛弃落后的局部模型可能会损失全局模型的准确率（accuracy）。
- **[统计异构](https://zhida.zhihu.com/search?content_id=214605093&content_type=Article&match_order=1&q=%E7%BB%9F%E8%AE%A1%E5%BC%82%E6%9E%84&zhida_source=entity)**(statistical heterogeneity)：不同设备所持有的数据之间是非独立同分布的（non-IID），此外还有数据不平衡的情况。


FedAvg算法成为了联邦学习的SOTA（state of the art）算法，但该论文仍然有一些**不足：**

- 该论文几乎没有考虑系统异构。在实际中，若一些局部模型没能在规定时间内完成训练，系统会丢弃这些模型，从而损失全局模型的准确率。
- 该论文利用实验结果证明了FedAvg算法的效果，但没有从理论上分析其收敛速率等指标。




 %% end notes %% 

### 文中笔记
  

