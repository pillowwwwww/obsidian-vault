### 1. - 在decentralized学习场景中（不一定是DFL，因为我不清楚普通的severless和DFL有什么区别），我们能否使用子空间感知的，OSFT/OFT的学习？在实现合并前，先计算梯度与旧知识空间正交，然后减去，用剩余向量进行更新？
    
    该研究问题是否成立？我们的研究场景是什么？应该是model merging ,还是TAD-LoRA这篇论文的研究场景？
    
    也就是研究”如何学习新任务，同时让更新不要进入旧任务已经使用的重要空间？”，类似于O-LoRA / InfLoRA / OSFT 这几篇论文在serverless中的迁移

流程：
![[e717469a8501980c184d98723bbe5a03-2.png]]




2. 根据子空间去选择merge的对象：论文：[MASS: MoErging through Adaptive Subspace Selection](https://iclr.cc/virtual/2026/poster/10010856)，ICLR 2026。MASS也根据子空间选择合并对象，但它不是在训练前选择固定模型子集，而是：
			对每个测试样本，动态判断该样本属于哪些任务子空间，然后只激活并合并对应的任务更新。
### 2. 研究干扰和共享的平衡
![[image-154.png|626x502]]
