---
title: creamfl复现记录
date: 2025-06-23 15:57:11
tags:
---

[[2025-06-23]] 学校服务器 
``` bash fold:"CreamFL_Quick成功"
python src/main.py --name CreamFL_Quick   --server_lr 1e-5 --agg_method con_w   --contrast_local_inter --contrast_local_intra --interintra_weight 0.5   --comm_rounds 8   --local_epochs 3   --pub_data_num 50000   --num_img_clients 4 --num_txt_clients 4 --num_mm_clients 6   --client_num_per_round 6   --feature_dim 128
结果：
wandb: Waiting for W&B process to finish... (success).
wandb:
wandb:
wandb: Run history:
wandb:        Server i2t_r1 ▁▄▅▆▇▇██
wandb: Server n_fold_i2t_r1 ▁▄▆▆▇███
wandb: Server n_fold_t2i_r1 ▁▄▅▆▇███
wandb:       Server rsum_r1 ▁▄▅▆▇███
wandb:        Server t2i_r1 ▁▃▅▅▇▇██
wandb:
wandb: Run summary:
wandb:        Server i2t_r1 19.52
wandb: Server n_fold_i2t_r1 41.42
wandb: Server n_fold_t2i_r1 32.264
wandb:       Server rsum_r1 107.472
wandb:        Server t2i_r1 14.268
wandb:
wandb: You can sync this run to the cloud by running:
wandb: wandb sync /home/liuchang/code/CreamFL/wandb/offline-run-20250622_143143-o5t8pq5m
wandb: Find logs at: ./wandb/offline-run-20250622_143143-o5t8pq5m/logs
```
