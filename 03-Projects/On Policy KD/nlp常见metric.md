**1. nlp的指标**

不是所有 NLP 任务都会关注这些指标。分类、序列标注通常不需要；但开放式生成、数学推理、代码生成、对话等任务，至少应该明确 `max_new_tokens`，并监控输出是否被长度上限截断。

它们可以分成两类：

|项目|类型|类似 LoRA 中的什么|
|---|---|---|
|Token budget / `max_new_tokens`|可设置的生成超参数|类似 rank、LR，是实验旋钮|
|Early stopping 规则|可设置的生成策略|类似 scheduler/stop condition|
|截断率/撞限率|运行后统计指标|类似梯度裁剪率、OOM率|
|EOS率|模型行为指标|类似收敛率、有效样本率|
|输出长度|模型行为指标|类似 loss/gradient norm 的诊断量|

Hugging Face 将 `max_new_tokens`、EOS、`stop_strings` 和自定义 `StoppingCriteria` 都作为标准生成接口，但并不规定统一取值。[Transformers generation documentation](https://huggingface.co/docs/transformers/main_classes/text_generation)

**Token budget**

设 `max_new_tokens=B`，表示模型在 prompt 后最多生成 \(B\) 个 token。

它影响：

- 是否有足够空间完成推理和答案；
- 推理速度、显存和 Teacher feedback 成本；
- 不同方法之间是否受到相同生成约束；
- Anchor 实际覆盖的状态数量。

它不是越大越好。太小会截断答案；太大则允许模型重复、自我怀疑和答案漂移。

GSM8K 没有公认的固定值。对当前 Qwen2.5-Math-1.5B，可以把 `128/320/512` 作为预算敏感性点，但最终值应由自然结束长度的 P95/P99 决定，而不是凭经验固定。

**撞限率**

定义为：

```
未产生 EOS 或合法停止信号，并生成到 max_new_tokens 的样本比例
```

它表示多少样本受到硬预算约束。以下只是项目工程建议，不是 NLP 统一标准：

|撞限率|解读|
|---|---|
|`<1%`|最理想，预算几乎不影响结果|
|`1%–5%`|通常可接受|
|`5%–10%`|需要检查撞限样本|
|`10%–20%`|预算已经明显干预结果|
|`>20%`|必须报告并做长度/停止敏感性分析|
|`>50%`|生成过程或停止机制明显异常|

你当前 R5–R8 的 `53%–78%` 不是普通波动。

**EOS率**

EOS率表示模型是否在预算内主动认为“回答已经结束”。

对于格式明确的 GSM8K：

|EOS/合法完成率|项目建议|
|---|---|
|`>95%`|理想|
|`90%–95%`|基本健康|
|`80%–90%`|需要分析|
|`<80%`|停止行为明显不稳定|

引入答案停止后，不应只报告 EOS率，而应报告：

```
completion_rate = natural_eos_rate + valid_answer_stop_rate
```

并区分自然 EOS 与规则停止，不能把强制停止伪装成自然 EOS。

**输出长度**

平均长度本身没有“越短越好”或统一合理范围。需要看：

- P50/P90/P95，而不只是均值；
- 正确与错误样本的长度；
- 撞限与非撞限样本的正确率；
- 首次最终答案出现位置；
- 最终答案之后又生成了多少 token；
- 后续推理是否把原本正确的答案改错。

近期 reasoning-model 研究也发现，答案出现后的继续推理有时只是冗余，有时会破坏已经正确的答案；是否适合早停取决于任务和轨迹结构。[Thinking Past the Answer](https://arxiv.org/abs/2606.02835), [When Does Learning to Stop Help?](https://arxiv.org/abs/2606.30852)

**Early stopping**

Early stopping 不是一个指标，而是规则：

```
当生成前缀满足某个完成条件时，提前终止生成
```

常见条件包括 EOS、固定停止字符串、完整答案格式、答案稳定或专门训练的停止分类器。Transformers 支持固定 `stop_strings` 和自定义逐序列停止条件。[StoppingCriteria documentation](https://huggingface.co/docs/transformers/internal/generation_utils)

