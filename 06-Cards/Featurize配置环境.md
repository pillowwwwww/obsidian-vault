---
Title: Featurize配置环境
tags: 
原始链接:
---

conda activate /home/featurize/work/myenv


持久化安装：`(/home/featurize/work/myenv) ➜ CreamFL pip install --user torch torchvision torchaudio`

 ✅  始终使用 pip 安装依赖，而不是用 conda，如无必要，不要创建虚拟环境，除非你知道自己在做什么。（i donnot know why)
 ✅  常用的包可使用 pip install --user xxx 安装，这样下次使用无需重复安装。 


版本：
pip install --user --index-url https://download.pytorch.org/whl/cu116 \
    torch==1.12.1+cu116 \
    torchvision==0.13.1+cu116 \
    torchaudio==0.12.1+cu116
    
 安装与之匹配的 torchtext（CPU 库即可，C++ 扩展会自动与 torch 对齐）
pip install --user torchtext==0.13.1
2. 安装apex，按照官网的格式，最简单的那种即可，我这次用的linux
3. 把requirement里面的文件装好
4. pip install --user fire

