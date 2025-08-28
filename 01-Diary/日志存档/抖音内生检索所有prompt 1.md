---
date: 2025-08-27 16:14:01
tags: dailynote
rating: ⭐
excerpt: 
---

## 发给豆包的prompt
```

任务描述：  
我将提供一个Instruction（前沿问题）、若干个解决Instruction需要用到的Concept，请不要使用联网搜索功能。请回答Instruction。

# Instruction
请不要使用联网搜索，回答以下问题：

设整数 m≥1m\ge1、r∈Nr\in\mathbb N。对每个 i∈[m]i\in[m]，令 Ai∈{0,1}m×rA_i\in\{0,1\}^{m\times r}，并定义

Tribes(Ai)=⋁j∈[r]⋀k∈[m](Ai)k,j,H(A1,…,Am)=1 ⁣[∑i=1mTribes(Ai) ≥m2+1].\mathrm{Tribes}(A_i)=\bigvee_{j\in[r]}\bigwedge_{k\in[m]}(A_i)_{k,j},\qquad H(A_1,\ldots,A_m)=\mathbf 1\!\Big[\sum_{i=1}^{m}\mathrm{Tribes}(A_i)\ \ge \tfrac m2+1\Big].

给定 tt 个独立“块” x(1),…,x(t)x^{(1)},\ldots,x^{(t)}，定义

Ft(x(1),…,x(t))=1 ⁣[∑j=1tH(x(j)) ≥αt],α∈(0,1) 为常数.F_t(x^{(1)},\ldots,x^{(t)})=\mathbf 1\!\Big[\sum_{j=1}^{t} H(x^{(j)})\ \ge \alpha t\Big],\quad \alpha\in(0,1)\ \text{为常数}.

我们考虑 **两种输入来源**：

- UU：{0,1}n\{0,1\}^n 的均匀分布；
    
- DD：按论文 **Definition 5** 采样的产品分布（当地址 a=(Tribes(A1),…,Tribes(Am))a=(\mathrm{Tribes}(A_1),\ldots,\mathrm{Tribes}(A_m)) 满足 ∣a∣=m/2|a|=m/2 时，将对应的 pap_a 置为 1，其余同均匀采样）。
    

设单块长度 s:=m2rs:=m^2 r，总输入长度 n:=t⋅sn:=t\cdot s。

**问题：** 选择能使

∣Pr⁡x∼D[Ft(x)=1]−Pr⁡u∼U[Ft(u)=1]∣ ≥ Ω(1)\Big|\Pr_{x\sim D}[F_t(x)=1]-\Pr_{u\sim U}[F_t(u)=1]\Big|\ \ge\ \Omega(1)

的**最小** tt 的量级，并给出一个可计算 FtF_t 的 DNF **宽度上界**，用 nn 表达其数量级。

> 说明：这里 **没有** 指定 rr 与 mm 的关系，也 **没有** 给出单块偏差规模 δ(m)\delta(m) 或“组合后宽度如何传递”的规则。这些都不在题面中；只有拿到 concepts 才能闭环。只给 instruction 时，模型无法把 mm 换成 log⁡n\log n，也无法选出最小 tt。

---

# Concepts（优先出自论文参考文献；每条仅一个工具）

**C1｜Tribes 的“平衡参数”选取**  
取 r=⌈2mln⁡2⌉r=\lceil 2^{m}\ln 2\rceil 时，Pr⁡[Tribes(A)=1]=12+o(1)\Pr[\mathrm{Tribes}(A)=1]=\tfrac12+o(1)。  
**来源（论文参考文献）**：O’Donnell, _Analysis of Boolean Functions_, §4.2（论文正文以 “[O’D14, §4.2]” 明确引用）。

**C2｜中点层质量（局部中心极限定理 / 二项中间层估计）**  
对独立 {0,1}\{0,1\} 变量和 SmS_m，在阈值 m/2+Θ(1)m/2+\Theta(1) 的邻域，

Pr⁡[ Sm≥m/2+1 ]−12 = Θ ⁣(1/m).\Pr[\,S_m\ge m/2+1\,]-\tfrac12\ =\ \Theta\!\big(1/\sqrt m\big).

（这给出 δ(m)=Θ(1/m)\delta(m)=\Theta(1/\sqrt m) 的量级。）  
**来源（论文参考文献优先）**：O’Donnell, _Analysis of Boolean Functions_（二项分布中间层/CLT 处；论文正文推导也得到 Ω(1/m)\Omega(1/\sqrt m) 的差距量级）。

**C3｜优势放大量化（Hoeffding/Chernoff）**  
独立重复 tt 次并作多数阈值，区分优势达 Ω(1)\Omega(1) 需要 t=Θ(1/δ2)t=\Theta(1/\delta^2)。  
**来源（论文正文点名使用；亦为标准外部文献）**：Hoeffding, _JASA_ (1963)；论文在放大步骤中“by Hoeffding inequality”给出精确量化。

**C4｜DNF 宽度 = 最大 1-证书大小**  
最小 DNF 宽度等于最大 1-证书复杂度 C1(⋅)C_1(\cdot)。  
**来源（论文参考文献）**：O’Donnell, _Analysis of Boolean Functions_；论文在 Claim 4 开头以“DNF 视作 1-证书族”使用该等价。

**C5｜分块合成的证书乘法上界**  
分块合成 F=g∘(f,…,f)F=g\circ(f,\ldots,f)（各块变量不相交）满足

C1(F) ≤ C1(g)⋅C1(f),C_1(F)\ \le\ C_1(g)\cdot C_1(f),

其中阈值函数 g(z)=1[∑zi≥αt]g(z)=\mathbf 1[\sum z_i\ge \alpha t] 有 C1(g)=⌈αt⌉C_1(g)=\lceil\alpha t\rceil。  
**来源（论文参考文献优先）**：O’Donnell, _Analysis of Boolean Functions_（certificate complexity 的 composition 命题/练习归纳）；或 Buhrman–de Wolf, _SIAM J. Comput._, 2002（综述中对证书复杂度与合成的不等式）。

> 这 5 条里，**C1/C2/C4** 直接来自论文引用的 O’Donnell；**C3** 在论文正文中被点名使用（也有外部原始文献）；**C5** 为教材/综述命题（多数版面在 O’Donnell 里可直接找到）。如需，我能标注 O’D14 的具体章节/命题号。

---

# Answer（结构简单的基础数值）

 DNF 宽度上界 =O((log⁡n)3) .\boxed{\,\text{DNF 宽度上界 }=O\big((\log n)^3\big)\,}.

**闭环要点（只解释给你看，不放入 concepts）**

- 由 C2 得单块偏差 δ(m)=Θ(1/m)\delta(m)=\Theta(1/\sqrt m)，C3 推出最小 t=Θ(1/δ2)=Θ(m)t=\Theta(1/\delta^2)=\Theta(m)；
    
- 由 C4 与标准“单列 all-1”证书可得 HH 的 1-证书大小 ℓ=O(m2)\ell=O(m^2)；
    
- 由 C5：C1(Ft)≤C1(g)⋅C1(H)=Θ(t)⋅O(m2)=O(m3)C_1(F_t)\le C_1(g)\cdot C_1(H)=\Theta(t)\cdot O(m^2)=O(m^3)；
    
- 用 C1 把 rr 明确为 Θ(2m)\Theta(2^m)，故 n=t⋅s=Θ(m)⋅(m2⋅2m)⇒m=Θ(log⁡n)n=t\cdot s=\Theta(m)\cdot(m^2\cdot 2^m)\Rightarrow m=\Theta(\log n)，得宽度 O((log⁡n)3)O((\log n)^3)。
    


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
请记住：concepts优先在当前题源论文的参考文献中找，如果无法在参考文献中找到，可以在互联网上检索。 请根据我的两条建议，修改你的设计。


```
```
请检查每一条Concept是否出现在题源论文中，并且是解决instruction所必须。每一条Concept中只包含一个概念或定理。注意!需要由外部的参考文献引入。
```