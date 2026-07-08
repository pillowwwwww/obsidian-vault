https://www.xiaohongshu.com/explore/6a4cda03000000001102f030?xsec_token=ABNN2--cxOd0Blfw45xA86tcKqfXZNpN78XuBDLeN_AjA=&xsec_source=pc_like
# RL、sft、distillation
![[image-144.png]]

SFT：老师直接给标准答案，学生照着学。 
蒸馏：强老师给概率分布或示范，学生模仿老师。
RL：学生自己做题，阅卷器给分，学生强化高分做法、减少低分做法。

|方法|数据从哪里来|监督信号来自哪里|训练目标|
|---|---|---|---|
|SFT|人工/标准答案|标准答案 token|模仿标准答案|
|蒸馏 KD|teacher 或固定数据|teacher logits / logprobs / 答案|模仿 teacher|
|OPD|student 自己生成|teacher 对 student 轨迹打分|在自己轨迹上学 teacher|
|RLHF|model 自己生成|人类偏好 / reward model|提高人类偏好回答概率|
|RLVR / GRPO|model 自己生成|验证器 / 标准答案检查|提高正确回答概率，降低错误回答概率|

**GRPO 的过程很简单：**

1. **模型自己答题**  
    同一道题生成多条答案。
    
2. **给答案打分**  
    答对 reward 高，答错 reward 低。
    
3. **组内比较**  
    比平均分好的答案是好样本，比平均分差的是坏样本。
    
4. **更新模型**  
    提高好答案中 token 的概率，降低坏答案中 token 的概率。
    
5. **限制更新幅度**  
    用 old/current ratio 和 clip 防止模型一下子改太猛。
    

一句话：

> **GRPO 就是让模型自己多做几遍题，奖励做得好的答案，惩罚做得差的答案，并控制更新不要太激进。**

```
# ========== 采样阶段 ==========
# 此时 policy 参数是 θ_old
with torch.no_grad():
    completions, old_logprobs = policy.generate(
        prompts,
        return_logprobs=True
    )

# old_logprobs 是采样时保存下来的，之后当常数用
rewards = verifier(prompts, completions)
advantages = compute_group_advantages(rewards)


# ========== 训练阶段 ==========
# policy 现在是 current model，参数是 θ_current
# 第一次 update 前，θ_current 通常等于 θ_old

logits = policy(prompts + completions)

# 从完整词表分布里取出“当初采样出来的那些 token”的 logprob
current_logprobs = gather_logprobs(
    logits,
    completion_token_ids
)

ratio = torch.exp(current_logprobs - old_logprobs)

loss = -torch.min(
    ratio * advantages,
    torch.clamp(ratio, 1 - eps, 1 + eps) * advantages
).mean()

optimizer.zero_grad()
loss.backward()
optimizer.step()
```