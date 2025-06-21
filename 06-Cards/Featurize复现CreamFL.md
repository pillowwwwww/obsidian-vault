---
Title: Featurize配置环境
tags: 
原始链接:
---

conda activate /home/featurize/work/myenv

wandb key：
ae54560cc7626fdea0641de28b9f493368799ceb

持久化安装：`(/home/featurize/work/myenv) ➜ CreamFL pip install --user torch torchvision torchaudio`

 ✅  始终使用 pip 安装依赖，而不是用 conda，如无必要，不要创建虚拟环境，除非你知道自己在做什么。（i donnot know why)
 ✅  常用的包可使用 pip install --user xxx 安装，这样下次使用无需重复安装。 
服务器cuda12.3

### Creamfl：
版本：
```
 pip install --user torch==1.10.0+cu113 torchvision==0.11.1+cu113 torchaudio==0.10.0+cu113 torchtext==0.11.0 -f https://download.pytorch.org/whl/torch_stable.html
```
2. Apex的Cuda版本与Pytorch的Cuda版本不匹配（无root权限）
安装apex，(pip install -v --no-cache-dir ./) [import apex always have AttributeError: module 'torch' has no attribute 'library' · Issue #1870 · NVIDIA/apex](https://github.com/NVIDIA/apex/issues/1870)按照官网的格式，用的linux
3. 把requirement里面的文件装好
4. pip install --user fire
5. 设置软连接：ln -参数+源文件或目录+目标文件或目录
   ln -s /home/featurize/work/data/mmdata /home/featurize/data/mmdata
6. ln -s /home/featurize/work/data/flickr30k/flickr30k-images /home/featurize/data/flickr30k/flickr30k-images
7. 把train2014和val2014的内容放到allimages里面！
 ```# 为 train2014 中所有图片创建软链接到 allimages/
find train2014 -type f -name '*.jpg' -exec ln -s ../train2014/{} allimages/ \;

#为val2014 中所有图片创建软链接到 allimages/
find val2014 -type f -name '*.jpg' -exec ln -s ../val2014/{} allimages/ \;
```
8. 运行creamfl ：
pip install transformers==4.21.0
```text
pip install --user -U huggingface_hub

python src/main.py --name CreamFL --server_lr 1e-5 --agg_method con_w --contrast_local_inter --contrast_local_intra --interintra_weight 0.5

```
  9. 下载这个包
     pip install --user nltk
  >>> import nltk
  >>>  nltk.download('punkt_tab')


### 预训练
图像（jpg） → ResNet18（预训练） → 图像特征向量（例如 512维）  
文本（caption） → GloVe（预训练） → 文本特征向量（300维）  
图文匹配：通过对比学习或表征对齐（如 PCME）
所以：
ResNet18 是图像编码器，它的预训练权重能让你快速获得表达图像语义的向量；
如果不用预训练，模型将无法“理解”图像的语义。
### 10. pretrain: 从huggingface上下载模型：
```
 export HF_ENDPOINT=https://hf-mirror.com
 
  huggingface-cli download --resume-download google-bert/bert-base-uncased --local-dir google-bert/bert-base-uncased --local-dir-use-symlinks False
  注意云服务器会删除缓存,你可以把模型删了再下一遍
 然后告诉电脑我已经下载好了：export TRANSFORMERS_CACHE=/home/featurize/work/google-bert
 等到了服务器上之后：
## 🔒 如何彻底阻止联网下载？
你需要：
### ✅ 1. 设置离线模式（阻止联网）
`export TRANSFORMERS_OFFLINE=1`
### ✅ 2. 加载模型时**改用本地路径**
AutoModel.from_pretrained("/home/featurize/work/google-bert/bert-base-uncased")
  ``` 
**可选参数 `--local-dir-use-symlinks False`**

_【更新：请注意，_[v0.23.0](https://link.zhihu.com/?target=https%3A//github.com/huggingface/huggingface_hub/releases/tag/v0.23.0)_开始加--local-dir 时会关闭符号链接】_

值得注意的是，有个`--local-dir-use-symlinks False` 参数可选，因为huggingface的工具链默认会使用符号链接来存储下载的文件，导致`--local-dir`指定的目录中都是一些“链接文件”，真实模型则存储在`~/.cache/huggingface`下，如果不喜欢这个可以用 `--local-dir-use-symlinks False`取消这个逻辑。

但我不太喜欢取消这个参数，**其最大方便点在于，调用时可以用模型名直接引用模型，而非指定模型路径。**

什么意思呢？我们知道，`from_pretrain` 函数可以接收一个模型的id，也可以接收模型的存储路径。

假如我们用浏览器下载了一个模型，存储到服务器的 `/data/gpt2` 下了，调用的时候你得写模型的绝对路径

```python3
AutoModelForCausalLM.from_pretrained("/data/gpt2")
```

然而如果你用的 `huggingface-cli download gpt2` 下载，即使你把模型存储到了自己指定的目录，但是你仍然可以简单的用模型的名字来引用他。即：

```text
AutoModelForCausalLM.from_pretrained("gpt2")
```

原理是因为huggingface工具链会在 `.cache/huggingface/` 下维护一份模型的**符号链接**，无论你是否指定了模型的存储路径 ，缓存目录下都会链接过去，这样可以避免自己忘了自己曾经下过某个模型，此外调用的时候就很方便。

所以用了官方工具，既可以方便的用模型名引用模型，又可以自己把模型集中存在一个自定义的路径，方便管理。
![[image-15.png]]


### 🧩 问题描述
解压后出现了类似结构：

```
coco2014/
└── annotations/
    └── annotations/
        ├── instances_train2014.json
        ├── captions_val2014.json
        └── ...
```

目标是把 `.json` 文件移到上层，只保留一层 `annotations/`。

---

### 🛠️ 解决步骤

```bash
# 进入最里层目录
cd /home/mount/lc_data/coco2014/annotations/annotations

# 把所有文件移动到上一层
mv * ../

# 返回上一层目录
cd ..

# 删除多余的 annotations 文件夹
rm -r annotations
```

---

在学校服务器上：
学校服务器不能联网，所以要从本本地下载，如何传到了pretained文件夹下面，然后向缓存中设置了软连接：
```
(creamfl_torch1.10) liuchang@work2:~/pretrained_models/bert/models--bert-base-uncased$ mkdir -p ~/.cache/huggingface/hub/
(creamfl_torch1.10) liuchang@work2:~/pretrained_models/bert/models--bert-base-uncased$ ln -s /home/liuchang/pretrained_models/bert/models--bert-base-uncased ~/.cache/huggingface/hub/models--bert-base-uncased
```
注意，mscoco也设置了软连接，真实的存放地址在：
/home/mount/lc_data/coco2014/train2014