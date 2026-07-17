**Greedy decoding 是什么**

Greedy decoding，中文通常称“贪心解码”，是最简单的文本生成策略：

> 每一步都选择当前概率最高的那个 token，不保留其他候选，也不回头修改。

假设某一步的概率是：

```
"Therefore"  45%
"So"         30%
"The"        15%
其他         10%
```

Greedy decoding 直接选择 `"Therefore"`。

下一步再根据：

```
Prompt + 已生成的 "Therefore"
```

计算新的 token 概率，并继续选择最高的一个。

整个过程只有一条路径：

```
Prompt
→ 当前最优 token
→ 当前最优 token
→ 当前最优 token
→ ...
```

不像 beam search 会同时保留多条候选：

```
Beam search:
Prompt
├─ 路径A
├─ 路径B
├─ 路径C
└─ 路径D

Greedy:
Prompt
└─ 唯一路径
```

**优缺点**

优点：

- 速度快；
- 显存占用低；
- 完全确定，相同输入通常得到相同结果；
- 适合需要复现的 anchor 构建。

缺点：

- 每一步只看当前最优，不保证整条序列全局最优；
- 走错一步后不能回退；
- 容易进入重复、冗长或自我检查循环；
- 即使答案已经出现，只要 EOS 不是概率最高的 token，就会继续生成。

**为什么当前是 Greedy**

当前配置：

```
do_sample: false
num_beams: 1
```

含义是：

- `do_sample=false`：不随机抽样；
- `num_beams=1`：只保留一条路径。

两者组合就是 greedy decoding。

**为什么 beam-search 的 `early_stopping=True` 无效**

Beam search 的 early stopping 判断的是：

```
已经收集到足够多条以 EOS 结束的 beam 吗？
```

例如：

```
num_beams=4
early_stopping=true
```

收集到4条完整 beam 后停止搜索。

但当前：

```
num_beams=1
```

没有多个 beam，也没有“继续扩展其他候选 beam”的过程。Hugging Face 会选择 greedy generation 路径，因此 beam search 的停止逻辑基本不会参与。

更重要的是，beam-search early stopping 不理解答案内容。即使生成了：

```
Therefore, the answer is #### 42
```

只要模型下一步认为：

```
"Let" 的概率 > EOS 的概率
```

Greedy decoding 就可能继续：

```
Therefore, the answer is #### 42.
Let me verify this calculation again...
```

直到：

- 模型主动生成 EOS；
- 达到 `max_new_tokens=320`；
- 命中 `stop_strings`；
- 触发自定义 `StoppingCriteria`。

所以当前问题不是“beam search 搜索太久”，而是：

> Greedy 模型已经给出明确答案，却没有主动选择 EOS。

这就是为什么需要 answer-aware stopping：

```
检测到完整的 #### <answer>
→ 明确认为答案已完成
→ 停止这一条 greedy 序列
```

它直接处理回答内容，而 beam-search 的 `early_stopping=True` 只处理多候选搜索过程。