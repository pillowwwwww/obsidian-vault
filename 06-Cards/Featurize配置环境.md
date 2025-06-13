---
Title: Featurize配置环境
tags: 
原始链接:
---

conda activate /home/featurize/work/myenv

持久化安装：`(/home/featurize/work/myenv) ➜ CreamFL pip install --user torch torchvision torchaudio`

 ✅  始终使用 pip 安装依赖，而不是用 conda，如无必要，不要创建虚拟环境，除非你知道自己在做什么。（i donnot know why)
 ✅  常用的包可使用 pip install --user xxx 安装，这样下次使用无需重复安装。 
