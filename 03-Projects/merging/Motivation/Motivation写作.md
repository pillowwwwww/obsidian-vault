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



### 2. cross_support by band → tail 方向缺乏跨客户端共识

### 3.不是噪音，而是有个性化功能意义