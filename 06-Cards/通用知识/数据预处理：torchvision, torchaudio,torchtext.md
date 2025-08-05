---
Title: 数据预处理：torchvision, torchaudio,torchtext
tags:
  - torchvision
  - torchaudio
  - torchtext
原始链接:
---
`torchtext`、`torchvision` 和 `torchaudio` 这三个包，都属于 **PyTorch 官方生态中的“领域工具集”**，目的是让用户快速实现以下功能：

> **从原始数据 → 可训练张量的完整 pipeline**。
> 
## 1. torchvision （图像模块 ）

### 📌 作用：
专注于图像任务的辅助工具，包括数据集加载、图像预处理、可视化和预训练模型。
- 提供了常用的 **数据集加载**（CIFAR-10/100、ImageNet、COCO 等）；
    
- 包含 **图像预处理／增强**（`transforms.Resize`、`RandomCrop`、`Normalize` 等）；
    
- 内置了众多 **预训练模型**（ResNet、VGG、DenseNet、Faster-RCNN…）及工具函数。
## 2. torchaudio （音频模块）
### 📌 作用：
为音频任务（语音识别、声音分类）提供数据处理、特征提取、音频文件读写等工具。
- 支持 **音频文件读写**（WAV、MP3）；
    
- 内置 **滤波器、谱变换**（STFT、Mel-Spectrogram、MFCC）；
    
- 包含语音识别、声学模型相关 **预训练权重**。

## 2. torchtext （文本模块）
### 📌 作用：
为 NLP 任务提供标准数据集、文本预处理、分词器、词表构建、预训练词向量等功能。
相当于上面两个包在 NLP 领域的整合：提供数据集、分词、词表、批次生成和与预训练词向量／Transformer 的对接。
### 进入文本编码器前的必要处理

几乎所有深度学习的文本模型（RNN、Transformer、BERT 等）都**不能直接**接受原始字符串，需要先做一系列 **“离散→连续”** 的转换：

|步骤|作用|
|---|---|
|**读取**|从文件/数据集里拿到原始 `str`（句子、文档）|
|**分词 (Tokenization)**|把一句话切分成若干“子词”或“词” (e.g. `["this","is","an","ex"]`)|
|**数值化 (Numericalization)**|将每个 token → 一个整数 id，构建词表 `vocab[token] = id`|
|**填充/截断 (Padding/Truncation)**|保证同一 batch 里所有序列长度相同，便于张量运算|
|**批次化 (Batching)**|把若干样本拼成 `[batch_size, seq_len]` 张量，并生成对应的 mask|
|**嵌入查表 (Embedding Lookup)**|用 `nn.Embedding` 或预训练词向量将 token id → 实数向量|

只有完成以上处理，才能把形状是 `(batch_size, seq_len)` 且元素为整数的张量，输入到后端的文本编码器里，得到连续的特征表示 `(batch_size, seq_len, hidden_dim)`。

## 三个模态的比较
| 步骤      | 图像 (`torchvision`)                 | 文本 (`torchtext`)                          | 音频 (`torchaudio`)                  |
| ------- | ---------------------------------- | ----------------------------------------- | ---------------------------------- |
| 加载原始数据  | `.jpg`, `.png`, `.bmp`             | `.txt`, `.csv`, `.json`                   | `.wav`, `.mp3`, `.flac`            |
| 数据集接口   | `ImageFolder`, `CIFAR10`, `COCO` 等 | `AG_NEWS`, `IMDB`, `WikiText` 等           | `LibriSpeech`, `CommonVoice` 等     |
| 样本结构    | 图像为 `HWC`，转为 `CHW` 张量              | 文本为字符串 → token ids                        | 音频为波形（1D）张量                        |
| 模型输入前处理 | Resize, Normalize, ToTensor        | Tokenize, Vocab Lookup, Padding           | STFT, MelSpectrogram, Normalize    |
| 模型输入结构  | `[batch_size, 3, H, W]`            | `[batch_size, seq_len]`                   | `[batch_size, waveform_len]`       |
| 预训练模型支持 | `ResNet`, `ViT`, `YOLO` 等          | `BERT`, `GPT`, `RoBERTa`（需从 Transformers） | `wav2vec 2.0`, `HuBERT`, `WavLM` 等 |

