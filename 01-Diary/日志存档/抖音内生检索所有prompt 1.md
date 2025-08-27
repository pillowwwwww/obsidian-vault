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
Concept/Theorem:
"""
concept1) QSMP message template and accounting

Each party can locally prepare a code-space phase state or a zero-set superposition over $\Sigma^n\simeq(\mathbb F_q)^N$:

$$
\lvert \psi_u\rangle=\frac{1}{\sqrt{|C_n|}}\sum_{c\in C_n}\omega^{\langle c,u\rangle}\lvert c\rangle,\quad
\lvert \phi_S\rangle=\frac{1}{\sqrt{|S|}}\sum_{e\in S}\lvert e\rangle.
$$

Encoding a register over $(\mathbb F_q)^N$ costs $Q=N\log_2 q$ qubits per message; with folded-RS parameters $N,\log q=\mathrm{poly}(n)$, the total QSMP cost (Alice + Bob, one shot each) is $\mathrm{poly}(n)$.

concept2) Deterministic list decoding for GRS (Guruswami–Sudan)

There is a polynomial-time procedure $\textsf{ListDecode}(z)$ that outputs

$$
\{x\in \mathrm{GRS}_{q,\gamma,k,\mathbf v}:\ \mathrm{hw}(x-z)\le N-\sqrt{kN}\},
$$

with $N=q-1$ and $d=N-k+1$.

concept3) List-recoverability of folded RS

For $C=\mathrm{RS}^{(m)}_{\mathbb F_q,\gamma,k}\subseteq\Sigma^n$ (with $\Sigma=\mathbb F_q^m,\ n=N/m$): if integers $r,s\ge1$ satisfy

$$
\frac{\zeta N}{m}\ \ge\ \Bigl(1+\frac{s}{r}\Bigr)(N\ell k^{s})^{\tfrac{1}{s+1}}\frac{1}{m-s+1},\quad
(r+s)\Bigl(\frac{N\ell}{k}\Bigr)^{\tfrac{1}{s+1}}<q,
$$

then $C$ is $(\zeta,\ell,L)$-list-recoverable with $L\le q^{s}$.
(Interface: given $(N,q,m,k,\zeta,\ell)$, choose $(r,s)$ to read off $L$.)

concept4) High-probability unique decodability of the dual under biased noise

Let $e\in(\mathbb F_q)^N$ have i.i.d. coordinates with $\Pr[e_j=0]=1-p$, $\Pr[e_j=a\neq0]=\tfrac{p}{q-1}$. For $z=x+e$ with unknown $x\in C_n^\perp$, define $\textsf{Decode}(z)$: unfold, run $\textsf{ListDecode}$, and return the unique $x$ if $\mathrm{hw}(z-x)\le(p+\varepsilon)N$. Whenever

$$
N-\sqrt{kN}\ \ge\ (p+\varepsilon)N,
$$

we have $\Pr[\textsf{Decode}(z)=x]\ge 1-2^{-\Omega(N)}$.

concept5) Quantitative subcube normalization (communication ⇒ fixed coordinates + min-entropy)

Let $B:=2^{\,n^{\delta^*}}$. Any public-coin two-way protocol with total communication $T$ can be normalized so that each accepting rectangle fixes at most

$$
s \le \frac{cT}{\max\{B,\ \log_2|\Sigma|\}},
$$

and every unfixed coordinate has per-coordinate min-entropy at least

$$
\alpha\cdot \max\{B,\ \log_2|\Sigma|\},
$$

for absolute constants $c,\alpha>0$.

concept6) Entropy–list tradeoff: hitting bound for dangerous patterns

Suppose on an accepting rectangle the dangerous-candidate set has size $\le L$, and the remaining $t'=n-s$ unfixed coordinates each have min-entropy $\ge \alpha\cdot \max\{B,\ \log_2|\Sigma|\}$. Then for any fixed dangerous pattern,

$$
\Pr[\text{hit that pattern}] \le 2^{-\alpha\cdot \max\{B,\ \log_2|\Sigma|\}\cdot t'}.
$$

By a union bound over all dangerous candidates,

$$
\Pr[\text{hit some dangerous pattern}] \le L\cdot 2^{-\alpha\cdot \max\{B,\ \log_2|\Sigma|\}\cdot t'}.
$$

(Interface: compare $\log_2 L$ against $\alpha\cdot \max\{B,\ \log_2|\Sigma|\}\cdot t'$.)

concept7) k-wise independent hash families and union bound

Over $\mathbb F_r$, the degree-$(k-1)$ polynomial family $h(i)=P(i)$ is $k$-wise independent, with seed length $\Theta(k\log r)$ and family size $|\mathcal H|=r^{k}$. If the protocol must succeed simultaneously for every $h\in\mathcal H$, require

$$
\alpha\cdot \max\{B,\ \log_2|\Sigma|\}\cdot (n-s)\ \gg\ \log_2|\mathcal H|
$$

so that the union bound across the family remains negligible.
"""

Instruction:
"""
Do not browse the web. Output only the ordered pair

(
 QSMP communication
,
 classical two-way lower bound 
).
( QSMP communication, classical two-way lower bound ).

Models.

QSMP: Alice and Bob each send one quantum state to a referee; no interaction; no shared randomness or entanglement. Cost = total qubits.

Classical public-coin two-way: interactive; cost = total bits.

Problem ingredients.

Folded RS code $C_n\subseteq \Sigma^n$ over $\Sigma=\mathbb F_q^m$; dual $C_n^\perp$.
---
Instances: $t=\mathrm{poly}(n)$ maps $H^{(i)}:\Sigma^n\to\{0,1\}^n$ solved in parallel.

Same hash across all copies: choose one $k$-wise independent $h$ and reuse it for all $t$ copies; require

$$
H^{(i)}(x^{(i)})\oplus h(x^{(i)})=0^n,\quad x^{(i)}\in C_n.
$$

Index gadget: each coordinate $j$ carries a label $g_j\in\{0,1\}^{B}$ with

$$
B=2^{\,n^{\delta^*}},\qquad \delta^*\in(0,1].
$$

Do not assume which of $B$ or $\log_2|\Sigma|$ dominates; use the Concepts if needed.

Family size: the protocol must succeed simultaneously for every $h\in\mathcal H$, where

$$
\log_2|\mathcal H|=n^{0.34}.
$$

Success criterion: worst-case error $\le 1/3$.

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
-
我想跟你讲解一下我现在要做的事情：我在测试模型是否能自己推出正确答案。我需要提交的字段【图片】已经上传给你了，但你不需要一下子给我所有字段的回答。现在我们一步一步来。首先，我需要找到一个合适的instruction，这个instruction中不能包含只存在于题源论文（也就是我上传给你的A zero-knowledge PCP THEOREM....这篇论文）中的特有的知识。我会把这个instruction发送给豆包，豆包将无法计算出正确的答案。
第二，我需要你给我整理几条concepts。检查每一条Concept是否出现在题源论文中，并且是解决instruction所必须。并且，每一条concepts出现在题源论文中时，需要是通过引用其他参考文献得到的concepts，而不能是该题源论文中独特的知识/推导中间过程。如果它是特有的知识，那么你需要把它放到instruction中，你明白吗？每一个Concept中只包含一个概念或定理。 其次，我希望concept中不要包含关键推导，而只是给出一些构造性证明的关键构造。我把【concepts+instruction】都发送给豆包，我希望豆包模型有时能推导出正确答案，有时不能推导出正确答案。
第三，我需要你再次检查你给出的每一条concepts是否能在题源论文中找到参考文献。我到时会把这些参考文献下载下来发送给你，你需要为我找到每一条concepts出自该参考文献的位置。

以上三部分你是否理解了？如果你理解了，你可以先为我进行第一步。
```


```
我想跟你讲解一下我现在要做的事情：我在测试模型是否能自己推出正确答案。我需要提交的字段【上传图片】已经上传给你了，但你不需要一下子给我所有字段的回答。现在我们一步一步来。首先，我需要找到一个合适的instruction，这个instruction中不能包含只存在于题源论文（也就是我上传给你的A zero-knowledge PCP THEOREM....这篇论文）中的特有的知识。我会把这个instruction发送给豆包，豆包将无法计算出正确的答案。
第二，我需要你给我整理几条concepts。检查每一条Concept是否出现在题源论文中，并且是解决instruction所必须。并且，每一条concepts出现在题源论文中时，需要是通过引用其他参考文献得到的concepts，而不能是该题源论文中独特的知识/推导中间过程。如果它是特有的知识，那么你需要把它放到instruction中，你明白吗？每一个Concept中只包含一个概念或定理。 其次，我希望concept中不要包含关键推导，而只是给出一些构造性证明的关键构造。我把【concepts+instruction】都发送给豆包，我希望豆包模型有时能推导出正确答案，有时不能推导出正确答案。
第三，我需要你再次检查你给出的每一条concepts是否能在题源论文中找到参考文献。我到时会把这些参考文献下载下来发送给你，你需要为我找到每一条concepts出自该参考文献的位置。

以上三部分你是否理解了？如果你理解了，你可以先为我进行第一步。
```
```
```



cot：
```

```

