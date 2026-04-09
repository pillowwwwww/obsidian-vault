2026-04-09 15:08
# Review 04/09
### 1. Figure 1
很容易被质疑这只是平均化趋同，但是展示不出来在趋同过程中，不同方向被不均匀的削弱。
Figure 1： 基本保留当前形式。小改：

- 右侧 slope plot 加一条 x=0 的虚线锚定（让视觉上"压到零附近"更明显）

- 1. JS 绝对值本身已经很小。 即使在 round 0，JS divergence 也只在 1e-3 量级。Reviewer 可能会问：这些客户端之间的谱本来就很像（毕竟共享同一个初始化），压缩 4 倍是否真的有实际意义？建议在 caption 中加一句参考基准（如随机矩阵的预期 JS 值）来锚定数量级。
    
2. 只展示了 round 0 → round 1。 如果 round 2 以后又发散呢？建议在正文中至少提一句为什么选择 round 0→1（如：round 0 是最大的谱差异来源，后续轮次已经在 shared initialization 基础上训练，因此谱差异的绝对量更小）。

图表表达：
- 热力图的 colorbar label 写的是 "Pairwise spectral distance (JS divergence)"——建议改为更简短的 "JS divergence"。
- 右侧 slope plot 中 y 轴标签 "Cross-client JS divergence (x1e-3)" 和 "Pairwise spectral distance (JS divergence)" 同时出现，有些冗余。建议统一为一个名称。
- Client ID 的 x 轴标签字体偏小。
Caption 建议：
> Figure 1. FedAvg compresses cross-client spectral heterogeneity in LoRA-B after a single aggregation round. (a) Pairwise JS divergence of singular-value spectra between all client pairs, before (round 0) and after (round 1) the first FedAvg round. (b) Each grey line tracks one (client pair, layer) combination; the blue line marks the median. Median JS divergence drops by 72% (from 2.86×10⁻³ to 0.80×10⁻³). This compression is expected from averaging, but motivates asking _which_ local spectral components are disproportionately suppressed (see Figure 2).

### Figure 2
1. 按 attenuation 选出来的方向，是否混杂大量噪音？
当前有什么能部分回应这个攻击： Figure 3 的 causal experiment 试图回答"这些方向不是噪音"，但只有 5 个 bundle，效应量在 1e-5 量级。
改进建议（不需要重跑训练）： 在生成 heatmap 时加一个 energy gate，比如只展示 `band_name in ["head", "middle"]` 或 `energy_score >= τ_e` 的方向。这样可以排除纯噪声方向，让 motivation 更站得住脚。如果加了 energy gate 之后，高衰减方向仍然大量存在，那说服力就大大增强。

