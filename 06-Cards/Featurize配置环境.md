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
5. 运行creamfl

