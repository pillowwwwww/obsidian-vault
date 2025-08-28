---
date: 2025-08-27 16:14:01
tags: dailynote
rating: ⭐
excerpt: 
---

## 发给豆包的prompt
```

#Task：  
请不搜索互联网，只参考提供的Concepts回答问题 

# Instruction
设整数 $m\ge1$、$r\in\mathbb N$。对每个 $i\in[m]$，令 $A_i\in\{0,1\}^{m\times r}$，并定义

$$
\mathrm{Tribes}(A_i)=\bigvee_{j\in[r]}\bigwedge_{k\in[m]}(A_i)_{k,j},\qquad 
H(A_1,\ldots,A_m)=\mathbf 1\!\Big[\sum_{i=1}^{m}\mathrm{Tribes}(A_i)\ \ge \tfrac m2+1\Big].
$$

取 $t$ 个独立“块” $x^{(1)},\ldots,x^{(t)}$，并定义

$$
F_t(x^{(1)},\ldots,x^{(t)})=\mathbf 1\!\Big[\sum_{j=1}^{t} H(x^{(j)})\ \ge \alpha t\Big],\quad \alpha\in(0,1).
$$

我们考虑两种输入来源：

* $U$：对 $\{0,1\}^n$ 的均匀分布；
* $D$：按论文中 **Definition 5** 的产品分布采样（当地址向量 $a=(\mathrm{Tribes}(A_1),\ldots,\mathrm{Tribes}(A_m))$ 满足 $|a|=m/2$ 时，将对应 $p_a$ 置为 1）。

**前提：**

* （L12）*t·ℓ-DNF 合成引理*：若 $f$ 可由宽度 $\ell$ 的 DNF 计算，且 $g:\{0,1\}^t\to\{0,1\}$ 为单调阈值函数，则 $g\circ f^t$ 可由宽度 $t\ell$ 的 DNF 计算。

设单块长度 $s:=m^2 r$，总输入长度 $n:=t\cdot s$。

**问题：** 选择使

$$
\big|\Pr_{x\sim D}[F_t(x)=1]-\Pr_{u\sim U}[F_t(u)=1]\big|\ \ge\ \Omega(1)
$$

的**最小** $t$ 的量级，并给出一个计算 $F_t$ 的 DNF **宽度上界**，用 $n$ 表达其数量级。

---
### Concept 1) Balanced parameters of Tribes (with exact probability formula)

For $A\in\{0,1\}^{m\times r}$,

$$
\Pr[\mathrm{Tribes}(A)=1]\ =\ 1-\bigl(1-2^{-m}\bigr)^{r}.
$$

When $r=\lceil 2^{m}\ln 2\rceil$, we have $\Pr[\mathrm{Tribes}(A)=1]=\tfrac12+o(1)$ (“balanced”).

---

### Concept 2) 1-certificate template of Tribes (“single column of all 1’s”, size $=m$)

$\mathrm{Tribes}(A)=\bigvee_{j\in[r]}\bigwedge_{k\in[m]}A_{k,j}$.  
A 1-certificate is: choose a column $j^\star$ and fix all $m$ bits in that column to 1. Certificate size $=m$.

---

### Concept 3) Explicit estimate for the binomial “middle layer”

For $S_m\sim\mathrm{Bin}(m,1/2)$,

$$
\Pr\big[S_m=m/2\big]\ =\ \binom{m}{m/2}2^{-m}\ =\ \Theta\!\big(1/\sqrt{m}\big),
$$

more precisely (via Stirling):

$$
\binom{m}{m/2}2^{-m}\in\Big[\sqrt{\tfrac{2}{\pi m}}\,(1-O({1}/{m})),\ \sqrt{\tfrac{2}{\pi m}}\,(1+O({1}/{m}))\Big].
$$

---

### Concept 4) Hoeffding amplification (minimum repetitions required for majority vote)

Let independent indicator variables $Z_1,\ldots,Z_t$ have expectation gap $\delta>0$ under two distributions.  
Take threshold $\alpha=\tfrac12(\mathbb E_x Z+\mathbb E_u Z)$, then

$$
\Pr_x\!\Big[\sum_{i=1}^t Z_i\ge \alpha t\Big]-\Pr_u\!\Big[\sum_{i=1}^t Z_i\ge \alpha t\Big]\ \ge\ 1-2e^{-\tfrac{t\delta^2}{2}}.
$$

Thus, to achieve distinguishing advantage $\Omega(1)$, one needs $t=\Theta(1/\delta^2)$.

---

### Concept 5) DNF width = maximum 1-certificate complexity

For any Boolean function $f$, the minimal DNF width equals $C_1(f)$, the supremum of the minimal 1-certificate sizes over all 1-inputs.



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
目前豆包模型无法通过concepts推导出instruction的正确答案，请你重新设计concepts。
我认为concept需要更具体，例如具体的做法，计算公式等。

但你需要注意：
1.每一条Concept/Theorem是否出现在论文中，并且是解决instruction所必须。
2.每一条Concept/Theorem中只包含一个概念或定理。
3.注意！Concept/Theorem不能是题源论文推论，需要由外部的参考文献引入。 concepts优先在当前题源论文的参考文献中找，如果无法在参考文献中找到，可以在互联网上检索。
请根据我的建议，修改你的设计。

```
