# 图片
远端对应目录是：
- /root/SubspaceLoRA/Subspace_LoRA/experiments/tail_collapse/motivation_main_figures_fedavg_homo_clients0_2_3_4_5_r128_seed42_seed43

# 文字表述
![[image-114.png]]
**Figure 1.**  
Figure 1. FedAvg rapidly compresses cross-client spectral heterogeneity in LoRA-B. At round 0, client updates exhibit clearly different spectral profiles, with a mean pairwise JS divergence of 4.18×10^-3 and a median of 2.86×10^-3. After only one FedAvg step, the mean and median pairwise distances drop to 1.06×10^-3 and 8.00×10^-4, respectively. This shows that aggregation quickly pushes heterogeneous client updates toward a more homogeneous spectrum.
图 1。FedAvg 会在 LoRA-B 中快速压缩跨客户端的谱异质性。在第 0 轮时，客户端更新表现出明显不同的谱特征，其两两 JS 散度的平均值为 4.18×10^-3，中位数为 2.86×10^-3。仅经过一步 FedAvg 后，两两距离的平均值和中位数分别下降到 1.06×10^-3 和 8.00×10^-4。这表明，聚合会迅速将原本异质的客户端更新推向更加同质化的谱分布。



![[image-115.png]]
**Figure 2.**  
Figure 2. FedAvg selectively attenuates low-consensus local directions. The left panel shows representative local directions whose support from other clients is weak and whose post-aggregation projection is substantially reduced. The right panel summarizes direction retention after aggregation. At round 0, the mean retention is 0.391 and 99.8% of directions have retention below 1; even at round 1, the mean retention is still only 0.882 and 81.5% of directions remain below 1. These results suggest that aggregation does not suppress all local directions uniformly, but disproportionately weakens directions with limited cross-client support.
图 2。FedAvg 会选择性地削弱跨客户端共识较低的局部方向。左图展示了具有代表性的局部方向：这些方向几乎得不到其他客户端的支持，并且在聚合后，其投影被显著缩小。右图总结了聚合后的方向保留情况。在第 0 轮时，平均保留率为 0.391，且 99.8% 的方向保留率低于 1；即使在第 1 轮时，平均保留率仍只有 0.882，并且仍有 81.5% 的方向低于 1。这些结果表明，聚合并不是对所有局部方向进行均匀抑制，而是会不成比例地削弱那些跨客户端支持有限的方向。


![[image-116.png]]
**Figure 3.**  
Figure 3. The bundles most attenuated by aggregation are more useful to their source client. In the left panel, restoring each candidate bundle to a global anchor improves the source client substantially more than non-source clients on average (2.03×10^-5 vs. 0.445×10^-5 in loss reduction), and this pattern holds for all 5/5 bundles. In the right panel, removing the same bundle from the source local model causes much larger degradation than matched-control removals (4.23×10^-5 vs. 0.938×10^-5), again for 5/5 bundles. Together, these results indicate that the attenuated bundles are not random noise; they carry source-preferential, functionally meaningful local content.
图 3。被聚合削弱最明显的 bundle 对其源客户端更有用。左图中，将每个候选 bundle 恢复到一个全局锚点时，相比非源客户端的平均效果，它对源客户端的提升显著更大（loss 降低分别为 2.03×10^-5 和 0.445×10^-5）；这一模式在 5/5 个 bundle 上都成立。右图中，从源客户端的局部模型中移除同一个 bundle，会比匹配对照移除造成更大的性能退化（4.23×10^-5 vs. 0.938×10^-5）；这一结果同样在 5/5 个 bundle 上成立。综合来看，这些结果说明，被削弱的 bundle 并非随机噪声，而是携带了偏向源客户端、在功能上具有实际意义的局部信息。

**Motivation Paragraph**
Federated fine-tuning starts from client updates that are substantially heterogeneous in LoRA-B, but this heterogeneity is compressed very quickly by FedAvg (Figure 1). Importantly, the compression is not uniform. Aggregation disproportionately attenuates local directions that receive weak support from other clients (Figure 2). Moreover, the content carried by these attenuated bundles is not functionally random. When restored to a global anchor, the bundles benefit their source client more than non-source clients; when removed from the source local model, they hurt the source client more than matched-control removals (Figure 3). These observations motivate our central research question: does FedAvg wash out part of the client-specific, functionally meaningful knowledge encoded in low-consensus LoRA-B subspaces?
联邦微调开始时，客户端在 LoRA-B 中的更新具有显著异质性，但这种异质性会被 FedAvg 很快压缩（图 1）。更重要的是，这种压缩并非均匀发生。聚合会不成比例地削弱那些几乎得不到其他客户端支持的局部方向（图 2）。此外，这些被削弱的 bundle 所承载的内容在功能上也并非随机。当它们被恢复到全局锚点时，这些 bundle 对其源客户端的帮助大于对非源客户端的帮助；而当它们从源客户端的局部模型中被移除时，造成的损害也大于匹配对照移除（图 3）。这些观察引出了我们的核心研究问题：FedAvg 是否会冲淡一部分编码在低共识 LoRA-B 子空间中的、具有功能意义的客户端特有知识？


**如果你想更像顶会动机段的收尾句**  
你也可以把最后一句换成这一版，更有研究问题的张力：

如果确实如此，那么标准的聚合规则所减少的就不只是无害的客户端差异；它还可能抹除那些对本地适配有用、偏向源客户端的知识。


---
