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
请不要使用联网搜索，回答以下问题：
设整数 m≥1m\ge1。对每个 i∈[m]i\in[m] 给定 Ai∈{0,1}m×rA_i\in\{0,1\}^{m\times r}，其中 r=⌈2 mln⁡2⌉r=\lceil 2^{\,m}\ln 2\rceil。定义

Tribes(Ai)=⋁j∈[r]⋀k∈[m](Ai)k,j,H(A1,…,Am)=1 ⁣[ ∑i=1mTribes(Ai) ≥m2+1 ].\mathrm{Tribes}(A_i)=\bigvee_{j\in[r]}\bigwedge_{k\in[m]}(A_i)_{k,j},\qquad H(A_1,\ldots,A_m)=\mathbf 1\!\Big[\ \sum_{i=1}^{m}\mathrm{Tribes}(A_i)\ \ge \tfrac m2+1\ \Big].

令“一个块”的输入长度 s:=m2rs:=m^2r。取 tt 个独立块 x(1),…,x(t)x^{(1)},\ldots,x^{(t)}，并定义外层阈值函数

Ft(x(1),…,x(t)) := 1 ⁣[ ∑j=1tH(x(j)) ≥αt ],F_t(x^{(1)},\ldots,x^{(t)})\ :=\ \mathbf 1\!\Big[\ \sum_{j=1}^t H(x^{(j)})\ \ge \alpha t\ \Big],

其中 α∈(0,1)\alpha\in(0,1) 是常数。设总输入长度 n:=t⋅sn:=t\cdot s。

**比较的两种分布（论文特有设定）**：令

x=(A1,…,Am,p) 按 Definition 5 的方式采样（当 ∣a∣=m/2 时强制 pa=1),x=(A_1,\ldots,A_m,p)\ \text{按 Definition 5 的方式采样（当 }|a|=m/2 \text{ 时强制 }p_a=1),

并令 u∼{0,1}nu\sim\{0,1\}^n 为均匀分布（见文中 Definition 5 及其后续论证）。  
我们考虑区分 xx 与 uu 的任务：选取**能把区分优势放大到 Ω(1)\Omega(1)** 的最小 tt 的量级，并给出计算 FtF_t 的 DNF **宽度上界**，用 nn 表达其数量级。


---

# Concepts（优先取自题源论文参考文献；每条只含一个工具）

**C1｜Tribes 的平衡性（来自 O’D14，被论文正文引用）**  
当 r=⌈2 mln⁡2⌉r=\lceil 2^{\,m}\ln 2\rceil 时，Pr⁡[Tribes(A)=1]=12+o(1)\Pr[\mathrm{Tribes}(A)=1]=\tfrac12+o(1)。论文正文用 “It is well-known [O’D14, §4.2] … balanced” 指出这一事实。

**C2｜中间层概率尺度（局部中心极限定理 / Stirling 近似，O’D14 教材）**  
对独立 {0,1}\{0,1\} 变量的和，**恰在中点层**（如 ∣a∣=m/2|a|=m/2）的质量

Pr⁡[∣a∣=m/2] = Θ(1/m).\Pr[|a|=m/2]\ =\ \Theta(1/\sqrt m).

（这给出区分优势的“底噪级别”。教材型来源可用 O’Donnell《Analysis of Boolean Functions》中关于二项分布中间层的标准估计；论文正文亦在推导中得到 Pr⁡[E]=Ω(1/m)\Pr[E]=\Omega(1/\sqrt m)。）

**C3｜Chernoff/Hoeffding 放大量化（教材级；论文正文点名使用 Hoeffding）**  
将单次优势 δ>0\delta>0 的判别独立重复 tt 次并取阈值 α\alpha 的多数投票，达到 Ω(1)\Omega(1) 的区分优势需要

t=Θ(1/δ2).t=\Theta(1/\delta^2).

论文在 Lemma 11 证明中明确“by Hoeffding inequality”给出该量化。

**C4｜DNF 宽度 = 最大 1-证书大小（教材级：O’D14）**  
最小 DNF 宽度等于最大 1-证书复杂度 C1(⋅)C_1(\cdot)。论文正文把 DNF 视作“1-证书族”来使用这一视角（Claim 4 开头）。

**C5｜分块合成的证书乘法上界（教材级：O’D14）**  
分块合成 F=g∘(f,…,f)F=g\circ(f,\ldots,f) 且各块变量不相交时，有

C1(F) ≤ C1(g)⋅C1(f).C_1(F)\ \le\ C_1(g)\cdot C_1(f).

当 g(z)=1[∑zi≥αt]g(z)=\mathbf 1[\sum z_i\ge \alpha t] 为单调阈值函数时，C1(g)=⌈αt⌉C_1(g)=\lceil\alpha t\rceil（只需固定 ⌈αt⌉\lceil\alpha t\rceil 个输入为 1）。

> 注：上面 C1–C5 均来自题源论文的**外部来源**（O’D14 教材 + 经典不等式），其中 C3 在正文处被点名使用；C2 的数值尺度教材中有直接论述，且论文正文也得到相同量级（见引文）。

---

# Answer（结构简单的数值结论）

 可计算 Ft 的 DNF 宽度上界 = O((log⁡n)3) .\boxed{\ \text{可计算 }F_t\text{ 的 DNF 宽度上界 }=\ O\big((\log n)^3\big)\ }.

**为何“只给 instruction”时不可解？**

- 题面**未给出** δ\delta 的规模与“怎么放大到 Ω(1)\Omega(1)”的量化，也**未给** DNF 在分块多数组合下的宽度传递规则；没有 C2（中间层 Θ(1/m)\Theta(1/\sqrt m)）与 C3（t=Θ(1/δ2)t=\Theta(1/\delta^2)），模型无法选出最小 tt；没有 C5（证书乘法），即便知道单块 ℓ=O(m2)\ell=O(m^2) 也很难正确得到总体 tℓt\ell。
    
- 给足 C1–C5 后：由 C2 得 δ=Θ(1/m)\delta=\Theta(1/\sqrt m)，C3 推出最小 t=Θ(m)t=\Theta(m)；单块 HH 的证书大小 ℓ=O(m2)\ell=O(m^2)（由 C4 + “单列全 1”证书构造），再用 C5 得 C1(Ft)≤⌈αt⌉⋅ℓ=Θ(m)⋅O(m2)=O(m3)C_1(F_t)\le \lceil\alpha t\rceil\cdot \ell = \Theta(m)\cdot O(m^2)=O(m^3)；由 n=t⋅s=Θ(m)⋅Θ(m2⋅2m)n=t\cdot s=\Theta(m)\cdot\Theta(m^2\cdot2^m) 得 m=Θ(log⁡n)m=\Theta(\log n)，故宽度 =O((log⁡n)3)=O((\log n)^3)。
    

---

## 你关心的两点我已对齐

1. **只给 instruction**：刻意不含 δ 的规模与组合传递法则 → **一般模型无法闭环**。
    
2. **concepts 来源**：优先使用题源论文参考文献中的 O’Donnell 教材（C1、C2、C4、C5），以及论文正文点名使用的 Hoeffding（C3 的量化在正文 lemma 里直接出现）。如果你坚持所有概念都必须在“参考文献表”中出现，也可把 C3 改为“O’D14 中的 Chernoff 版本”来替代 Hoeffding 的表述。

# Concepts（全部来自外部文献，可做 docset）

- **C1｜Tribes 平衡参数**：当 r=⌈2mln⁡2⌉r=\lceil 2^{m}\ln 2\rceil 时，Pr⁡[Tribes(A)=1]=12+o(1)\Pr[\mathrm{Tribes}(A)=1]=\tfrac12+o(1)。  
    参考：O’Donnell, _Analysis of Boolean Functions_, §4.2.
    
- **C2｜DNF 宽度 = 最大 1-证书大小**（证书视角的标准等价）。  
    参考：O’Donnell, _Analysis of Boolean Functions_（证书复杂度章节）。
    
- **C3｜证书复杂度的分块合成上界**：对分块合成 F=g∘(f,…,f)F=g\circ(f,\ldots,f)，有  
    C1(F) ≤ C1(g)⋅C1(f)\displaystyle C_1(F)\ \le\ C_1(g)\cdot C_1(f)。阈值函数 g(z)=1[∑zi≥k]g(z)=\mathbf 1[\sum z_i\ge k] 满足 C1(g)=kC_1(g)=k。  
    参考：Buhrman & de Wolf, _SIAM J. Comput._, 2002（综述）；或 O’Donnell 教材中 composition 命题。
    
- **C4｜放大量化（Hoeffding/Chernoff）**：若单次优势为 δ>0\delta>0，多数投票达到 Ω(1)\Omega(1) 需 t=Θ(1/δ2)t=\Theta(1/\delta^2)。  
    参考：Hoeffding, _JASA_, 1963。（题源正文处亦“by Hoeffding inequality”。）
    
- **C5｜局部中心极限定理 / Berry–Esseen（阈值近邻偏差刻度）**：在阈值 m/2+Θ(1)m/2+\Theta(1) 邻域，  
    Pr⁡[Sm≥m/2+1]−12=Θ(1/m)\Pr[S_m\ge m/2+1]-\tfrac12=\Theta(1/\sqrt m)。  
    参考：Feller, _Probability Theory_, Vol. I；Berry (1941), Esseen (1942)。
    

# Answer（基础数值）

 DNF 宽度上界 =O((log⁡n)3) .\boxed{\ \text{DNF 宽度上界 }=O\big((\log n)^3\big)\ }.

## 难度开关（保证“有时对、有时错”）

- **开关A（最关键）**：是否附上 **C5**。
    
    - 给 C5：模型可推出 δ=Θ(1/m)⇒t=Θ(m)⇒\delta=\Theta(1/\sqrt m)\Rightarrow t=\Theta(m)\Rightarrow 宽度 =O(m3)=O((log⁡n)3)=O(m^3)=O((\log n)^3)。
        
    - 不给 C5：多数模型拿不到 δ\delta 量级，难以选出最小 tt，常答错/不完整。
        
- **开关B**：把 **C3** 只写“C1(F)≤C1(g)⋅C1(f)C_1(F)\le C_1(g)\cdot C_1(f)”而不点明“阈值 gg 的 C1(g)=kC_1(g)=k”——很多模型会漏乘或误用，从而失败。
    
- **开关C（轻改阈值）**：把内层阈值改成 m/2+⌈m⌉m/2+\lceil \sqrt m\rceil 或 m/2+cm/2+c（常数）。这会改变 δ\delta 的规模，使得是否给 C5 更关键，进一步拉大正确率差异。
    

如果你要，我也可以再给“阈值变体”的成套题面（同一 instruction 模板 + 两档 concept 配置），方便你直接做 A/B 测。
---

# Concepts
**concept1) Tribes 的平衡参数（题源论文已引用）**  
当 r=⌈2mln⁡2⌉r=\lceil 2^{m}\ln 2\rceil 时，Pr⁡[Tribes(A)=1]=12+o(1)\Pr[\mathrm{Tribes}(A)=1]=\tfrac12+o(1)。  
来源：R. O’Donnell, _Analysis of Boolean Functions_, §4.2（题源论文正文明确引用此处）。

**concept2) DNF 宽度与 1-证书复杂度相等（外部教材）**  
对任意布尔函数 ff，其最小 DNF 宽度等于最大的 1-证书大小 C1(f)C_1(f)。  
来源：R. O’Donnell, _Analysis of Boolean Functions_, 证书复杂度与 DNF 的标准等价（教材结论）。

**concept3) 证书复杂度的分块合成上界（外部综述/教材）**  
对分块合成 F=g∘(f,…,f)F=g\circ (f,\ldots,f)（各块变量互不相交），有

C1(F) ≤ C1(g)⋅C1(f).C_1(F)\ \le\ C_1(g)\cdot C_1(f).

特别地，单调“阈值”函数 g(z)=1[∑zi≥k]g(z)=\mathbf 1[\sum z_i\ge k] 满足 C1(g)=kC_1(g)=k。  
来源（任选其一，便于你入库）：

- H. Buhrman, R. de Wolf, _Complexity Measures and Decision Tree Complexity_, **SIAM J. Comput. 2002**（关于证书复杂度、合成不等式的综述结论）；
    
- 或 O’Donnell 教材中关于证书复杂度与函数合成的命题（对应章节可检索“certificate complexity, composition/submultiplicativity”）。
    

**concept4) 放大量化（Hoeffding/Chernoff，不等式）**  
独立重复 tt 次，若单次优势为 δ>0\delta>0，多数判决的区分优势达到 Ω(1)\Omega(1) 需要 t=Θ(1/δ2)t=\Theta(1/\delta^2)。  
来源：W. Hoeffding, _JASA_, 1963（题源论文正文在相关引理证明中点名“by Hoeffding inequality”）。

**concept5) 局部中心极限定理 / Berry–Esseen（确定 δ\delta 的尺度）**  
对 Sm=∑i=1mXiS_m=\sum_{i=1}^m X_i（独立同分布 Xi∈{0,1}X_i\in\{0,1\}，p=12+o(1)p=\tfrac12+o(1)），在阈值 m/2+Θ(1)m/2+\Theta(1) 的邻域，

Pr⁡[Sm≥m/2+1]−12 = Θ ⁣(1m).\Pr[S_m\ge m/2+1]-\tfrac12\ =\ \Theta\!\Big(\tfrac{1}{\sqrt m}\Big).

即此时 δ=Θ(1/m)\delta=\Theta(1/\sqrt m)。  
来源：W. Feller, _An Introduction to Probability Theory_, Vol. I（局部极限定理）；或 Berry (1941)/Esseen (1942)。

> 说明：concept1 引自题源论文引用；concept2–5 为“题源以外”的权威来源，满足你“概念必须来自其他论文/教材”的要求（其中 concept4 在题源正文被点名使用，便于对齐）。

---

# Answer（基础数值）

 DNF 宽度上界 =O((log⁡n)3) .\boxed{\ \text{DNF 宽度上界 }=O\big((\log n)^3\big)\ }.

**推演提要（不计入 concepts，仅供你核对）：**

- 由 concept5 得 δ=Θ(1/m)\delta=\Theta(1/\sqrt m)；由 concept4 得最小 t=Θ(1/δ2)=Θ(m)t=\Theta(1/\delta^2)=\Theta(m)。
    
- 由 (B2) 与阈值 (m/2+1)(m/2{+}1) 可构出 HH 的 1-证书大小 ℓ=Θ(m2)\ell=\Theta(m^2)。
    
- 由 concept3：C1(Ft)≤C1(g)⋅C1(H)=⌈αt⌉⋅ℓ=Θ(m)⋅Θ(m2)=Θ(m3)C_1(F_t)\le C_1(g)\cdot C_1(H)=\lceil\alpha t\rceil\cdot \ell=\Theta(m)\cdot \Theta(m^2)=\Theta(m^3)。
    
- 由 (B3) 和 r=⌈2mln⁡2⌉r=\lceil 2^{m}\ln 2\rceil 得 log⁡n=Θ(m)⇒m=Θ(log⁡n)\log n=\Theta(m)\Rightarrow m=\Theta(\log n)，从而 Θ(m3)=O((log⁡n)3)\Theta(m^3)=O((\log n)^3)。
    

---

## 难度控制（让模型“有时对、有时错”）

- **开关 A（最关键）**：是否给 concept5（局部 CLT/Berry–Esseen）。
    
    - 给：多半能推出 δ=Θ(1/m)\delta=\Theta(1/\sqrt m) → 答对。
        
    - 不给：往往无法确定 δ\delta 量级 → 选不出最小 tt → 常答错。
        
- **开关 B**：把 concept3 只写成“C1(g∘ft)≤C1(g)⋅C1(f)C_1(g\circ f^t)\le C_1(g)\cdot C_1(f)”而**不点明**“阈值 gg 有 C1(g)=kC_1(g)=k”；不少模型会把 tt 漏乘或误用 max⁡{⋅}\max\{\cdot\}。
    
- **开关 C**：把 HH 的阈值改成 m/2+⌈m⌉m/2+\lceil \sqrt m\rceil 或 m/2+cm/2+c（常数）。这会改变 δ\delta 的幅度，对有无 concept5 的依赖更强，答案分化更明显。
    

---

### 合规性自检

- **Instruction**：仅含标准定义与基础事实（B1–B3），不含题源论文的新技术或专有构造。
    
- **Concepts**：全部来自题源论文之外的来源（其中 concept1 也是题源“引用的外部知识”），满足你“概念不得来自题源正文新结果”的要求。
    
- **Answer**：为简洁的数量级结论。
    

如果你愿意，我还能再给一个**阈值变体版**（例如把外层 α\alpha 固定为 1/21/2，或把内层阈值改为 m/2+⌈m⌉m/2+\lceil\sqrt m\rceil），配同样风格的概念清单，用来在评测里混随机难度。

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

