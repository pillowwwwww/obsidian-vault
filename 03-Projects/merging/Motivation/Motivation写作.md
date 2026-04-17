# 图片
远端对应目录是：
- /root/SubspaceLoRA/Subspace_LoRA/experiments/tail_collapse/motivation_main_figures_fedavg_homo_clients0_2_3_4_5_r128_seed42_seed43

# 前置知识

#### 暂时只针对B矩阵
【SELECTIVE AGGREGATION FOR LOW-RANK ADAPTATION IN FEDERATED LEARNING 】
该论文通过**数学推导**得出，LoRA 的 A、B 在联邦学习中不对称，依赖不同因素（A 的最优解与输入数据分布无关，而 B与输入数据分布有关）。
提出A 更像“跨客户端共享的通用知识”，B 更像“客户端私有的个性化知识”
#### 2. 定义谱异质性：
在线性代数里，`spectrum（谱）` 泛指一个线性对象在某个分解下的标量序列。  
> `singular value spectrum（奇异值谱）`
对某个客户端 `c`、层 `l`、轮次 `t` 的 `LoRA-B` 矩阵记为：
$$
B_{c,l}^{(t)} \in \mathbb{R}^{d_{\text{out}} \times r}
$$
SVD:
$$
B_{c,l}^{(t)} = U_{c,l}^{(t)} \Sigma_{c,l}^{(t)} V_{c,l}^{(t)\top}
$$
也可写为：
$$
B_{c,l}^{(t)} = \sum_{i=1}^{q} \sigma_{c,l,i}^{(t)}\, u_{c,l,i}^{(t)} v_{c,l,i}^{(t)\top}
$$
![[image-118.png]]

- $u_i v_i^\top$ 是第$i$个谱方向；
- $\sigma_i$是该方向的强度。

# 逻辑链（谱异质性存在 → 低共识方向优先被削弱 → 被削弱内容对源客户端有功能意义）
#### 1. FedAvg rapidly compresses cross-client spectral heterogeneity in LoRA-B.`
联邦聚合会快速压缩 LoRA（后面可能会改成B矩阵）中跨客户端的谱异质性。
#### 2. FedAvg does not suppress local directions uniformly; it disproportionately attenuates low-consensus local directions.
那些获得其他客户端支持较弱的局部方向【暂时定为低共识方向】，在聚合后被保留下来的可能性较低。`
#### 3. The attenuated bundles are not random noise; they are more useful to their source client. FedAvg may wash out part of the client-specific, functionally meaningful knowledge encoded in low-consensus LoRA-B。
被削弱的方向并非随机噪声；它们对源客户端更有用。（个性化知识）

# 文字表述
![[image-114.png]]
**Figure 1.**  后面改成Frobenius 距离
Figure 1. FedAvg rapidly compresses cross-client spectral heterogeneity in LoRA-B. At round 0, client updates exhibit clearly different spectral profiles, with a mean pairwise JS divergence of 4.18×10^-3 and a median of 2.86×10^-3. After only one FedAvg step, the mean and median pairwise distances drop to 1.06×10^-3 and 8.00×10^-4, respectively. This shows that aggregation quickly pushes heterogeneous client updates toward a more homogeneous spectrum.
图 1。FedAvg 会在 LoRA-B 中快速压缩跨客户端的谱异质性。在第 0 轮时，客户端更新表现出明显不同的谱特征，其两两 JS 散度的平均值为 4.18×10^-3，中位数为 2.86×10^-3。仅经过一步 FedAvg 后，两两距离的平均值和中位数分别下降到 1.06×10^-3 和 8.00×10^-4。这表明，聚合会迅速将原本异质的客户端更新推向更加同质化的谱分布。
**spectral = LoRA-B 的 singular value spectrum（奇异值谱）**
底层分析脚本在 analyze_spectral_heterogeneity.py (line 49)。

已完成修改并在 `534` 服务器重生成 `Figure 1（图1）`。

- 更新后的本地脚本在 [make_motivation_main_figures.py](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/make_motivation_main_figures.py#L165)。
- 更新后的本地图文件在 [figure1_spectral_contraction.png](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/motivation_main_figures_fedavg_homo_clients0_2_3_4_5_r128_seed42_seed43/figure1_spectral_contraction.png)。
- 我给 `Figure 1` 加上的两个子标题是：
  - Left: `Pairwise spectral-distance heatmaps before and after the first FedAvg round`
  - Right: `Pairwise contraction of cross-client spectral distance after FedAvg`
- `colorbar（颜色图例）` 现在已经从整张图最右侧移到了左侧热图块旁边。

**Figure 1 中文介绍**
这里的 `spectral（谱）` 不是 `FGL（联邦图学习）` 里的图拉普拉斯频域，而是 **LoRA-B 矩阵的 `singular value spectrum（奇异值谱）`**。底层分析脚本在 [analyze_spectral_heterogeneity.py](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/analyze_spectral_heterogeneity.py#L49)。

实验设置：
- clients: `0, 2, 3, 4, 5`
- tasks: `Translation / Question Answering / Question Understanding / Sentiment Analysis / Coreference Resolution`
- rank: `128`
- seeds: `42, 43`
- 比较轮次：`round 0` 和 `round 1`
- 分析对象：每个 client（客户端）、每一层的 `LoRA-B`

实验流程：
1. 对每个 client、每一层的 `LoRA-B` 做 `SVD（奇异值分解）`：
$$
B_{c,l}^{(t)} = U_{c,l}^{(t)} \Sigma_{c,l}^{(t)} V_{c,l}^{(t)\top}
$$
2. 取奇异值向量 $\sigma$，构造归一化谱分布：
$$
p_{c,l}^{(t)}(i)=\frac{\sigma_{c,l}^{(t)}(i)}{\sum_j \sigma_{c,l}^{(t)}(j)}
$$
这一步在 [_normalized_sum_spectrum.py](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/analyze_spectral_heterogeneity.py#L105)。
3. 对同一层、同一轮的两个客户端 `a,b`，计算它们归一化谱分布的 `JS divergence（JS 散度）`：
$$
\mathrm{JS}(p,q)=\frac{1}{2}\mathrm{KL}(p\|m)+\frac{1}{2}\mathrm{KL}(q\|m), \quad m=\frac{p+q}{2}
$$
实现见 [_js_divergence](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/analyze_spectral_heterogeneity.py#L123) 和 [_pair_distance_payload](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/analyze_spectral_heterogeneity.py#L223)。
4. 底层分析会同时生成：
- `same_client_twin（同客户端双种子）`
- `cross_client_replica1（seed42 的跨客户端）`
- `cross_client_replica2（seed43 的跨客户端）`
相关构造在 analyze_spectral_heterogeneity.py 和 [analyze_spectral_heterogeneity.py](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/analyze_spectral_heterogeneity.py#L520)。
1. 但 `Figure 1` 当前主图读取的是 `cross_client_replica1`，即 [make_motivation_main_figures.py](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/make_motivation_main_figures.py#L132) 到 [make_motivation_main_figures.py](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/make_motivation_main_figures.py#L134) 过滤后的结果。

图表解读：
- 左图：显示 `round 0` 与 `round 1` 的跨客户端 `pairwise spectral distance（成对谱距离）` 热图。颜色越深，两个客户端在 LoRA-B 奇异值谱分布上越不一样。
- 右图：每条灰线对应一个 `client-pair × layer（客户端对 × 层）` 的距离从 `round 0` 到 `round 1` 的变化；粗蓝线显示中位数变化，文字同时标出均值变化。
- 当前核心数值：
  - round 0 mean JS = `0.004183843624724181`
  - round 0 median JS = `0.00286187644259175`
  - round 1 mean JS = `0.0010628018987472575`
  - round 1 median JS = `0.0007998994210758`
- 这意味着第一次 `FedAvg（联邦平均）` 后，跨客户端 LoRA-B 谱分布差异显著收缩。中位数约变为原来的 `0.28x`，均值约变为原来的 `0.25x`。
- 因而 `Figure 1` 支撑的结论是：
  **FedAvg rapidly compresses cross-client spectral heterogeneity in LoRA-B.**

补充一点我刚核对到的实现细节：左侧热图的构造函数在 [_build_heatmap_matrix](/e:/FL/Robust%20Tail%20Singular/Subspace_LoRA/experiments/tail_collapse/make_motivation_main_figures.py#L145) 里，当前是把同一 `client pair（客户端对）` 的多层值顺序写入同一个矩阵格子，因此**左热图目前不是显式按层均值聚合**；而右图和 summary 数值是基于全部 `1680` 个 `client-pair × layer` 条目算出来的。

---

![[image-119.png]]
**Figure 2.**  
左图（Scatter）关键数据：

- Spearman ρ = 0.795，p ≈ 0 — 极强的正相关
- 共 64,680 个方向（仅 `head` + `middle` band，已过滤噪声）
- 其他客户端的投影绝对值均值，x 轴越低，`retain_after_agg`（聚合后保留率，y 轴）也越低
- 趋势线（binned mean）从左下到右上几乎是线性递增的

右图（Violin）关键数据：

- Round 0：99.8% 方向被衰减（保留率 < 1），平均保留率 0.391
- Round 1：81.5% 方向被衰减，平均保留率 0.882

论点支撑力： 这张图直接证明了 "获得其他客户端支持较弱的局部方向，在聚合后被保留下来的可能性较低"，且用了与 y 轴非耦合的 x 轴指标（`diff_presence`），band filter 排除了噪声方向，使用了全量方向（无 selection bias）。

图片已保存在本地 `Subspace_LoRA/experiments/tail_collapse/redesign_motivation_main_figures_fedavg_homo_clients0_2_3_4_5_r128_seed42_seed43/` 目录中。


![[image-116.png]]
**Figure 3.**  
Figure 3. The bundles most attenuated by aggregation are more useful to their source client. In the left panel, restoring each candidate bundle to a global anchor improves the source client substantially more than non-source clients on average (2.03×10^-5 vs. 0.445×10^-5 in loss reduction), and this pattern holds for all 5/5 bundles. In the right panel, removing the same bundle from the source local model causes much larger degradation than matched-control removals (4.23×10^-5 vs. 0.938×10^-5), again for 5/5 bundles. Together, these results indicate that the attenuated bundles are not random noise; they carry source-preferential, functionally meaningful local content.
图 3。被聚合削弱最明显的 bundle 对其源客户端更有用。左图中，将每个候选 bundle 恢复到一个全局锚点时，相比非源客户端的平均效果，它对源客户端的提升显著更大（loss 降低分别为 2.03×10^-5 和 0.445×10^-5）；这一模式在 5/5 个 bundle 上都成立。右图中，从源客户端的局部模型中移除同一个 bundle，会比匹配对照移除造成更大的性能退化（4.23×10^-5 vs. 0.938×10^-5）；这一结果同样在 5/5 个 bundle 上成立。综合来看，这些结果说明，被削弱的 bundle 并非随机噪声，而是携带了偏向源客户端、在功能上具有实际意义的局部信息。

**Motivation Paragraph**
Federated fine-tuning starts from client updates that are substantially heterogeneous in LoRA-B, but this heterogeneity is compressed very quickly by FedAvg (Figure 1). Importantly, the compression is not uniform. Aggregation disproportionately attenuates local directions that receive weak support from other clients (Figure 2). Moreover, the content carried by these attenuated bundles is not functionally random. When restored to a global anchor, the bundles benefit their source client more than non-source clients; when removed from the source local model, they hurt the source client more than matched-control removals (Figure 3). These observations motivate our central research question: does FedAvg wash out part of the client-specific, functionally meaningful knowledge encoded in low-consensus LoRA-B subspaces?
联邦微调开始时，客户端在 LoRA-B 中的更新具有显著异质性，但这种异质性会被 FedAvg 很快压缩（图 1）。更重要的是，这种压缩并非均匀发生。聚合会不成比例地削弱那些几乎得不到其他客户端支持的局部方向（图 2）。此外，这些被削弱的 bundle 所承载的内容在功能上也并非随机。当它们被恢复到全局锚点时，这些 bundle 对其源客户端的帮助大于对非源客户端的帮助；而当它们从源客户端的局部模型中被移除时，造成的损害也大于匹配对照移除（图 3）。这些观察引出了我们的核心研究问题：FedAvg 是否会冲淡一部分编码在低共识 LoRA-B 子空间中的、具有功能意义的客户端特有知识？


如果确实如此，那么标准的聚合规则所减少的就不只是无害的客户端差异；它还可能抹除那些对本地适配有用、偏向源客户端的知识。


---
所以我们提出了FedDGC:

"Spectral contraction 是 FedAvg 聚合的副产品。虽然它对全局模型的平均性能影响有限（因为被衰减的多是任务特异方向），但对任务异构的个别客户端是有害的：当客户端收到聚合后的 G 作为下一轮训练起点时，它之前学到的任务特异方向已经被稀释或丢失了。FedDGC 在客户端开始本地训练之前，选择性地恢复那些'在本地显著、被全局稀释、且其他客户端不支持'的方向，让客户端不需要浪费本地训练预算去重新学习已经学过的东西。"


---
问题：
1. FedSA无法超越。
   - 从原理上反驳：
	   - “冻结其中的一个矩阵，代价是压缩了可训练参数空间，会减慢收敛、降低适配能力”；
	   -  A 全部共享，B 全部不共享。但是A也有客户端特有的知识, B也有个性化知识
   - 把方法改成方向感知，而不是聚合后修补。
	   -  给高重要性方向更大权重，给个性化的方向更多的权重（进行放缩），直接修改聚合算法。
	   -  其它方向正常平均

1. 我们应该是作用在全局还是作用在客户端呢？

---
# 逻辑链：
1. 稀释缺口存在，且非均匀 【展示稀释缺口的存在，展示方向的消失】
2. 被稀释的方向不是噪音，而是有个性化功能意义
3. 所以值得修补

# 定义稀释缺口
释缺口的本质是：

> 客户端本地训练后，ΔW_loc 在方向 φ_k 上的强度是 σ_k。FedAvg 聚合多个客户端的更新后，全局 ΔW_global 在同一个方向 φ_k 上的强度变成了 α_k。当 α_k < σ_k 时，就产生了缺口。


### 同轮内比较 = 跨轮比较

这是我需要帮你澄清的一点——同轮内和跨轮在这里是同一件事：

Round 0:

客户端训练 → ΔW_loc (方向 φ_k 强度 = σ_k)

FedAvg → ΔW_global (方向 φ_k 强度 = α_k)

Round 1:

客户端的起点就是 ΔW_global ← 这就是 round 0 的聚合结果。


我现在有一个关于动机部分的逻辑链：
(1) FedAvg 在同一轮内创造了稀释缺口 (δ_i > 0)
(2) 这些缺口不是无害的（被稀释的方向有功能意义）
(3) 因此值得在下一轮训练前修补。

---
# 实验观测
### 1. tail 方向在标准 FedAvg 聚合下遭受更严重的稀释缺口"
- **tail / floor 在相对意义上更脆弱**
- **head / middle 在绝对意义上损失更多**

由于大奇异值的方向（也就是头部奇异值的方向）不容易发生变化，而尾部的奇异值因为奇异值小，方向更容易发生变化。我们能否在此基础上，针对不同band，它们的方向产生的偏斜，做一个量化展示？你有什么思路吗？

>方向的扰动敏感性 $∝  1 / (σ_i − σ_{i+1})$
  Head 段奇异值间距大 → 方向稳定。Tail 段奇异值密集 → 微小扰动就能让方向大幅旋转。
![[image-121.png|760x332]]


### 2. cross_support by band → tail 方向缺乏跨客户端共识

### 3.不是噪音，而是有个性化功能意义

> Figure 1 左:   δ/σ by band → tail 方向被稀释得更严重（效果）

>Figure 1 右 (新):  cross_support by band → tail 方向缺乏跨客户端共识（原因）
>	tail 方向的跨客户端共识低，FedAvg 聚合时其他客户端没有相应的支持来"保护"这些方向（why）

> Figure 2:   恢复 top-δ 方向 → perplexity 下降（功能意义，排除"noise"解释）

![[image-120.png]]


Figure 1：FedAvg 对 tail 方向造成了更大的稀释缺口（因为跨客户端共识低）

Figure 2：这些被稀释的 tail 方向不是噪声——同客户端在下一轮主动恢复了它们（R1 Self ≈ 0.72），而其他客户端完全不恢复（R1 Other ≈ Post-FedAvg）。FedAvg 浪费了训练资源，迫使客户端在每一轮都重新学习被抹掉的个性化知识。

→ FedDGC-B 的动机：既然这些方向会被重新学习，不如在聚合时就保护它们。




local-only 的 band 排序是 Floor > Tail > Middle > Head（尾部方向在纯本地训练中增长最多），而 FL 排序相反

---
# 数据异构，任务异构，有些客户端的任务弱，容易被别的任务的强方向淹没
多任务合并时，最怕的是某些任务的弱方向被别的任务的强方向“淹没”。如果一个任务的尾部奇异方向本来就很小，那合并时它更容易被扰动，导致该任务特有知识丢失。于是作者的思路是：  
**不要只让头部方向特别强，要把尾部方向也适当补起来。**

---

![[image-122.png]]

1. Local-only 线几乎保持平坦（0.89–0.96） → 证明方向损失不是训练本身造成的，是聚合造成的
2. FL 与 Local-only 的差距 = 纯粹的"聚合代价"（aggregation cost）
3. Local-only 中 Floor 保留最好（0.96），但 FL 中 Floor 损失最大（0.39）→ 聚合专门伤害低共识方向

#### 从这一张相对占比图中，可以同时读出四条证据：

1. 聚合是损失的唯一来源：Local-only ≈ 0.96（几乎无损），FL ≈ 0.39–0.51 → 差距完全由聚合造成
2. 聚合系统性偏向头部：Head 保留 51%，Floor 仅 39% → 低共识方向受害最深
3. 被损害的方向是个性化方向："被损害的方向确实包含个性化知识（gap > 0），且 Head 最不个性化", Specificity gap 在 Tail/Floor（~0.155）远高于 Head（0.117）. middle → tail → floor 的递增损伤——是因为奇异值大小（谱脆弱性），尾部的奇异值更脆弱。
4. 客户端能恢复但被反复打断：锯齿模式显示每轮恢复后又被聚合压制

#### 右图：Specificity gap = self retention − other retention
Specificity gap = self − other 衡量的是："同一个客户端能恢复该方向，而其他客户端不能恢复"——这就是个性化的操作性定义。

- Gap > 0 → 这些方向是客户端特有的（只有自己能学回来）
- Gap ≈ 0 → 这些方向是共享的（别人也能学到）

所以 所有 band 的 gap 都 > 0 确实证明了所有方向都带有一定的个性化信息。Head（0.117）显著低于其他三个 band（~0.15），说明头部方向更"公共"，非头部方向更"个性化"。
1. Head vs 非Head：个性化程度不同（0.117 vs ~0.15），这是 specificity gap 能证明的
2. 非Head 内部（middle/tail/floor）：个性化程度相似，但聚合伤害递增——这是因为谱脆弱性（σ 越小越容易被聚合噪声淹没），而不是因为"更个性化"


#### Band 排序反转本身就是个性化的证据：

- Local-only 中 Floor 保留 0.96 → Floor 方向在纯任务训练中最稳定 → 它们不是噪声，而是对任务有价值
- FL 中 Floor 保留 0.39 → 聚合专门摧毁了这些"对任务最有价值"的方向
- 如果 Floor 是噪声，它们在 local-only 训练中也应该自然衰减——但事实相反
所以"对任务稳定 + 被聚合破坏 + 低跨客户端共识"三者结合 ≈ 个性化。消融实验可以作为 supplementary evidence，但不是必须的。
----

# 我来整理一下当前motivation部分的内容。

实验设置：rank=32，511 个客户端，10% 部分参与，5 rounds

1. @Subspace_LoRA/experiments/tail_collapse/redesign_figure1_dilution_gap_dw_fedavg_homo_clients0_2_3_4_5_r128_seed42_seed43/figure1_dilution_gap_dw.png 这张图片证明了稀释缺口的存在，并且fail和floor在联邦聚合后有更严重的稀释缺口。

2. @Subspace_LoRA/experiments/tail_collapse/redesign_figure2_5round_fedavg_full511_part10_r32_lr1em2_seed42/figure2_5round_relative.png 这张图片表明，聚合是造成方向损失的唯一原因，训练本身造成的损失微乎其微（此处我不太理解为什么local-only训练也会造成损失）。

3. 客户端能恢复但被反复打断：锯齿模式显示每轮恢复后又被聚合压制。

4. 被稀释的方向是个性化方向，因为在local-only训练下，floor方向保持的最好，而FL下，band排序完全相反，代表floor和tail中包含客户端的个性化知识。并且Specificity gap也可证明，它们存在个性化知识，并且middle tail floor比head存在的个性化知识更多。

## 换个书面表述。
### Claim 1：稀释缺口普遍存在且具有谱结构（Figure 1）

FedAvg 聚合后，客户端 ΔW 的几乎每个奇异方向都会产生稀释缺口 δ_k > 0。关键发现：缺口的严重程度与奇异值大小负相关——头部方向（大 σ）的相对缺口约 55%，而尾部/底层方向（小 σ）的相对缺口达 65–70%。这是因为小奇异值方向在聚合平均中更容易被其他客户端的噪声淹没（谱脆弱性）。

### Claim 2：方向损失几乎完全由聚合造成（Figure 2 左面板）

通过将 FL 训练与 local-only 训练做控制实验对比（相同客户端、相同数据、相同训练轮数）：

- Local-only 训练 5 轮后，方向的相对份额仍保留 89–96% → 训练本身几乎不损失方向
- FL 训练 5 轮后，方向的相对份额仅剩 39–51% → 差距完全由聚合引入

Local-only 也有 4–11% 的微小损失，这是因为模型持续学习新知识时，优化轨迹自然漂移，部分初始方向的"份额"被新方向取代。但这种自然漂移比聚合损伤小一个数量级。

### Claim 3：客户端能恢复但被反复打断（Figure 2 锯齿模式）

当客户端重新参与训练时，它能将自己方向的相对份额从 ~0.45 恢复到 ~0.65。但下一次聚合立即将份额压回 ~0.50。这种"恢复 ↗ → 打断 ↘"的锯齿循环在每轮重复，说明聚合伤害不是一次性的，而是系统性的、持续的。

### Claim 4：被稀释的方向不是噪声（Figure 2 右面板 + band 排序反转）

两条证据证明被稀释的方向承载功能性知识：

(a) 所有 band 的 specificity gap（self − other）> 0，说明方向具有客户端特异性。Head 的 gap 最小（0.117），非 Head 的 gap 约 0.15——非头部方向比头部方向多约 30% 的个性化程度。

(b) Band 排序反转：Local-only 中 Floor 保留最好（0.96），说明这些方向对客户端自身任务最有价值。但 FL 中 Floor 损失最大（0.39），排序完全相反。这种反转说明：聚合恰恰优先破坏了对客户端最有用的方向。