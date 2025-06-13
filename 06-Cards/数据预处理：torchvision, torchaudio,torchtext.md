---
Title: 数据预处理：torchvision, torchaudio,torchtext
tags:
  - torchvision
  - torchaudio
  - torchtext
原始链接:
---
## 1. torchvision （图像模块）

### 📌 作用：
专注于图像任务的辅助工具，包括数据集加载、图像预处理、可视化和预训练模型。
- 提供了常用的 **数据集加载**（CIFAR-10/100、ImageNet、COCO 等）；
    
- 包含 **图像预处理／增强**（`transforms.Resize`、`RandomCrop`、`Normalize` 等）；
    
- 内置了众多 **预训练模型**（ResNet、VGG、DenseNet、Faster-RCNN…）及工具函数。
## 2. torchaudio （音频模块）
### 📌 作用：
为音频任务（语音识别、声音分类）提供数据处理、特征提取、音频文件读写等工具。