---
date: 2025-08-27 16:14:01
tags: dailynote
rating: ⭐
excerpt: 
---

## 发给豆包的prompt
```

任务描述：  
我将提供一个Instruction（前沿问题）、若干个解决Instruction需要用到的Concept/Theorem（概念、定理），请不要使用联网搜索功能。请回答Instruction。

# Instruction

设整数 m≥1m\ge1。对每个 i∈[m]i\in[m]，令 Ai∈{0,1}m×rA_i\in\{0,1\}^{m\times r} 且  
r=⌈2 mln⁡2⌉r=\lceil 2^{\,m}\ln 2\rceil。定义 **Tribes**：

Tribes(Ai)=⋁j∈[r]⋀k∈[m](Ai)k,j.\mathrm{Tribes}(A_i)=\bigvee_{j\in[r]}\bigwedge_{k\in[m]} (A_i)_{k,j}.

再定义布尔函数

H(A1,…,Am)=1 ⁣[ ∑i=1mTribes(Ai) ≥m2+1 ].H(A_1,\ldots,A_m)=\mathbf 1\!\left[\ \sum_{i=1}^{m}\mathrm{Tribes}(A_i)\ \ge \frac m2+1\ \right].

令输入总长度 n:=m2rn:=m^2r（因为共有 mm 个矩阵、每个尺寸 m×rm\times r）。问：**给出一个能计算 HH 的 DNF 的宽度，用 nn 表示其数量级**。


---

## Concepts（给模型时再发；每条只含一个概念/定理，且在论文中出现并带他引）

1. **Tribes 的平衡参数与 1-证书（all-1 列）**  
    当 r=⌈2 mln⁡2⌉r=\lceil 2^{\,m}\ln 2\rceil 时，Pr⁡[Tribes(A)=1]=12+o(1)\Pr[\mathrm{Tribes}(A)=1]=\tfrac12+o(1)；且 Tribes(A)=1\mathrm{Tribes}(A)=1 的一个 1-证书就是“任选一列全为 1”。论文将这一事实作为常识并引自 O’Donnell 的专著 [O’D14, §4.2]（论文处见 “It is well-known [O’D14, §4.2] … balanced” 语句）。
    
2. **DNF 与 1-证书的等价刻画**  
    “一个函数可由宽度为 kk 的 DNF 计算，当且仅当其每个取 1 的输入都存在大小 ≤kk 的 1-证书”。该刻画在论文的 Claim 4 证明里直接作为标准事实使用（“A DNF is commonly viewed as a collection of 1-certificates …”）。可配套参考 [O’D14] 对证书复杂度的讲解。
    
3. **合取下证书宽度可加**  
    若要证明“至少 m/2+1m/2+1 个谓词为真”，只需为任取的 m/2+1m/2+1 个子谓词各给出其 1-证书；整体 1-证书的大小为这些证书大小之和（用于把多数阈值化为若干合取证书）。论文在 Claim 4 的构造中按此思路给出了宽度界（为每个被选矩阵给出“一列全 1”的证书）。
    

"""

```


## 第一步:deep research找instruction
```
我现在在用 instruction–answer–concept 框架去测试模型的 deep research 能力（在测试时我不会上传这篇论文）。 

任务描述：
我将提供一篇Thesis（硕博学位论文，题源论文TR25-058）。请你作为一位深耕领域前沿的学者，找出Thesis中解决的instruction（关键问题），该instruction能够在Thesis中找到一个结构简单的answer（答案），再找出解决instruction得到answer所需要的3-5条Concept知识点，并重点检查以下方面的问题： 
---
instruction 应该是一个论文中确实解决了的、但能用一个非常简单的量或结论来回答的问题。不能过于开放。
---
请检查answer的结构是否符合结构简单的要求，answer的形式应该是是标量、产物名、简短的结论等。
---
请检查每一条Concept是否出现在Thesis中，并且是解决instruction所必须的。
每一条Concept中只包含一个概念或定理。
请保证每一个concept都要有一个非本论文的参考文献（给出ref在Thesis中的具体参考文献编号，在原文中是如何使用的），如果你的这个结论没有其他论文提及的话，你可以把他放进instruction里做已知信息。
---
我希望你可以聚焦于整篇论文的核心成果，如果核心成果设计的instruction的answer结构过于复杂，你也可以先关注某个章节或具体技术的成功。 

----

输出格式建议：
请按照以下结构回答：
Instruction：（领域前沿的问题）
Answer：（结构简单的答案）
Concept/Theorem：（解决instruction得到answer所需要的知识点）
- concept_n:...
- theorem_n:...
（以此类推）
---
我想跟你讲解一下我现在要做的事情：我在测试模型是否能自己推出正确答案。我需要提交的字段【图片】已经上传给你了，但你不需要一下子给我所有字段的回答。现在我们一步一步来。首先，我需要找到一个合适的instruction，这个instruction中不能包含只存在于题源论文（也就是我上传给你的A zero-knowledge PCP THEOREM....这篇论文）中的特有的知识。我会把这个instruction发送给豆包，豆包将无法计算出正确的答案。
第二，我需要你给我整理几条concepts。检查每一条Concept是否出现在题源论文中，并且是解决instruction所必须。并且，每一条concepts出现在题源论文中时，需要是通过引用其他参考文献得到的concepts，而不能是该题源论文中独特的知识/推导中间过程。如果它是特有的知识，那么你需要把它放到instruction中，你明白吗？每一个Concept中只包含一个概念或定理。 其次，我希望concept中不要包含关键推导，而只是给出一些构造性证明的关键构造。我把【concepts+instruction】都发送给豆包，我希望豆包模型有时能推导出正确答案，有时不能推导出正确答案。
第三，我需要你再次检查你给出的每一条concepts是否能在题源论文中找到参考文献。我到时会把这些参考文献下载下来发送给你，你需要为我找到每一条concepts出自该参考文献的位置。

以上三部分你是否理解了？如果你理解了，你可以先为我进行第一步。
```


```
请检查：instruction的内容中，是否存在只存在于我上传的这篇论文中特有的内容？
我在对豆包进行训练的时候，不会上传题源论文。
```

```

