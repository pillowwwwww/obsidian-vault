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

# Instruction（难度提升版）

设整数 $m\ge1$，对每个 $i\in[m]$ 给定矩阵 $A_i\in\{0,1\}^{m\times r}$ ，其中
$r=\lceil 2^{\,m}\ln 2\rceil$。定义

$$
\mathrm{Tribes}(A_i)=\bigvee_{j\in[r]} \ \bigwedge_{k\in[m]} (A_i)_{k,j},\qquad 
H(A_1,\ldots,A_m)=\mathbf 1\!\Big[\ \sum_{i=1}^{m}\mathrm{Tribes}(A_i)\ \ge \tfrac m2+1\ \Big].
$$

令“一个 **块**”的输入长度为 $s:=m^2 r$（共有 $m$ 个 $m\times r$ 矩阵）。对 $t$ 个独立块 $x^{(1)},\ldots,x^{(t)}$ 定义

$$
F_t(x^{(1)},\ldots,x^{(t)})=\mathbf 1\!\Big[\ \sum_{j=1}^{t} H(x^{(j)})\ \ge \alpha t\ \Big],
$$

其中阈值 $\alpha$ 任取常数（例如 $\alpha\in(0,1)$ 固定）。**已知**存在某个与均匀分布独立的产品分布 $\mathcal{X}$ 使得

$$
\big|\Pr_{\mathcal{X}}[H=1]-\Pr_{\mathrm{Unif}}[H=1]\big|\ \ge\ \frac{c}{\sqrt{m}}
$$

对某常数 $c>0$ 成立。设总输入长度 $n:=t\cdot s$。

**问题：** 任选能把 $\Pr[F_t=1]$ 在 $\mathcal{X}$ 与均匀分布之间的优势放大到 $\Omega(1)$ 的最小 $t$ 的量级，给出一个可计算 $F_t$ 的 DNF 的**宽度上界**，用 $n$ 表达其数量级。


---

# Concepts

**concept1) Tribes 的平衡参数**  
当 $r=\lceil 2^{\,m}\ln 2\rceil$ 时，$\Pr[\mathrm{Tribes}(A)=1]=\tfrac12+o(1)$（经典“平衡”设定）。TR25-058 正文明确引用 O’Donnell《Analysis of Boolean Functions》\[O’D14, §4.2] 来使用这一事实：  
“**It is well-known \[O’D14, §4.2]** … the function becomes balanced …”。  

**concept2) 单调外层与 DNF 的宽度乘法：$g\circ f^t$ 的 1-证书拼接**  
若 $f$ 可由宽度 $\ell$ 的 DNF 计算，且 $g:\{0,1\}^t\to\{0,1\}$ **单调**，则 $g\circ f^t$ 可由宽度 $t\ell$ 的 DNF 计算（把 $t$ 个块各自的 1-证书合取即可）。TR25-058 在 **Lemma 12** 中给出这一“单调组合 → 证书拼接 → 宽度乘 $t$”的标准论断：  
“**Then $g\circ f^t$ can be computed by a $t\ell$-DNF.**”  

**concept3) 放大量化：Hoeffding 不等式 → 需要 $t=\Theta(1/\delta^2)$**  
若 $\delta:=\big|\Pr[H=1|\mathcal{X}]-\Pr[H=1|\mathrm{Unif}]\big|>0$，独立重复 $t$ 次并在阈值 $\alpha\in(\Pr_{\mathrm{Unif}},\Pr_{\mathcal X})$ 处作判决，则

$$
\Pr_{\mathcal X}\big[\text{多数判为 1}\big]-\Pr_{\mathrm{Unif}}\big[\text{多数判为 1}\big]\ \ge\ 1-2e^{-\Theta(t\delta^2)}.
$$

取 $t=\Theta(1/\delta^2)$ 可把优势放大为 $\Omega(1)$。**外部来源**：Hoeffding, *JASA*, 1963；TR25-058 在 **Lemma 11** 的证明中也直接调用“by Hoeffding inequality”。
**concept4) DNF ↔ 1-证书视角（用于界定 $\ell$）**  
“一个函数可由宽度 $\le k$ 的 DNF 计算，当且仅当其每个 1-输入都存在大小 $\le k$ 的 1-证书”。TR25-058 在 **Claim 4** 开头以常识形式使用这一视角（将 DNF 视作 1-证书族）：  
“**A DNF is commonly viewed as a collection of 1-certificates …**”。  
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

