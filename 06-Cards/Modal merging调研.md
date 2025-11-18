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