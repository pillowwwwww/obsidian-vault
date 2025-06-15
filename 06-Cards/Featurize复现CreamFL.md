---
Title: Featurize配置环境
tags: 
原始链接:
---

conda activate /home/featurize/work/myenv


持久化安装：`(/home/featurize/work/myenv) ➜ CreamFL pip install --user torch torchvision torchaudio`

 ✅  始终使用 pip 安装依赖，而不是用 conda，如无必要，不要创建虚拟环境，除非你知道自己在做什么。（i donnot know why)
 ✅  常用的包可使用 pip install --user xxx 安装，这样下次使用无需重复安装。 
服务器cuda12.3

creamfl：
版本：
 pip install --user torch==1.10.0+cu113 torchvision==0.11.1+cu113 torchaudio==0.10.0+cu113 torchtext==0.11.0 -f https://download.pytorch.org/whl/torch_stable.html

2. Apex的Cuda版本与Pytorch的Cuda版本不匹配（无root权限）
安装apex，(pip install -v --no-cache-dir ./) [import apex always have AttributeError: module 'torch' has no attribute 'library' · Issue #1870 · NVIDIA/apex](https://github.com/NVIDIA/apex/issues/1870)按照官网的格式，用的linux
3. 把requirement里面的文件装好
4. pip install --user fire
5. 设置软连接：ln -s /home/featurize/work/data/mmdata /home/featurize/data/mmdata
6. 运行creamfl ：
pip install transformers==4.21.0
```text
pip install -U huggingface_hub
```




图像（jpg） → ResNet18（预训练） → 图像特征向量（例如 512维）  
文本（caption） → GloVe（预训练） → 文本特征向量（300维）  
图文匹配：通过对比学习或表征对齐（如 PCME）
所以：

ResNet18 是图像编码器，它的预训练权重能让你快速获得表达图像语义的向量；

如果不用预训练，模型将无法“理解”图像的语义。

