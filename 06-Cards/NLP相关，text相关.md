---
Title: NLP相关，text相关
tags:
  - NLP
  - torchtext
  - text
原始链接:
---

## 一、整体流程（从文本到模型输入）
#### 🧩 举例句子："A dog jumps over a log."
```text
1. 加载 coco_vocab.pkl
2. 分词（Tokenize） → ["a", "dog", "jumps", "over", "a", "log", "."]
3. 映射索引（Lookup） → [2, 3, 4, 5, 2, 6, 7]
4. Padding 补齐 → [2, 3, 4, 5, 2, 6, 7, 0, 0, 0]
5. 转成 Tensor → torch.tensor([...])
6. 送入 Embedding → 得到 [10, 300] 的词向量
7. 输入模型（RNN/Transformer）进行编码
8. 输出整句表示，用于对比 / 检索等下游任务
```
🎯 第二阶段：**训练/使用阶段**

|步骤|名称|输入示例|输出示例|说明|
|---|---|---|---|---|
|1️⃣|加载词典|.pkl 文件|{"a":2, "dog":3, ...}|提前构建好的词 → id 映射字典|
|2️⃣|分词（tokenize）|"A dog jumps ..."|["a", "dog", "jumps", ...]|拆成单词列表|
|3️⃣|查索引（lookup）|["a", "dog", "jumps"]|[2, 3, 4]|用 vocab 将 token 映射为数字|
|4️⃣|补齐（padding）|[2, 3, 4, 5, 2, 6, 7]|[2, 3, 4, 5, 2, 6, 7, 0, 0, 0]|补到统一长度，0 是 的索引|
|5️⃣|转张量（tensor）|list[int]|torch.tensor([...])|为送入模型做准备|
|6️⃣|嵌入（embedding）|Tensor([2,3,...])|[10, 300] 的嵌入向量|查表得到词向量|
|7️⃣|编码器处理|嵌入向量|句子语义表示向量|上下文建模（如 GRU、Transformer）|
|8️⃣|下游任务输入|语义向量|对比损失、图文匹配分数、分类等结果|用于 contrastive loss / 检索等|
补充解释：
- `<pad>`：表示填充用的空白词，index 一般为 0
- `<unk>`：词典中未出现的词统一映射为 `<unk>`，index 一般为 1
- `.pkl` 文件：pickle 序列化后的 Python 字典，保存了 `{word: index}` 映射
- vocab.pkl 仅用于 **"词 → index" 的映射**，训练时不再参与
- 第一阶段：**词汇表构建阶段（数据准备）**，也就是仅仅生成一次pkl文件，每个项目生成不同的pkl文件，方便复现和复用
- 即使都是使用 COCO 数据集，不同项目用的 coco_vocab.pkl 通常是各自构建的，不能混用。除非两个项目明确说：“我们使用相同构建规则/共享词表”。