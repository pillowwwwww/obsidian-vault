---
Title: MINE_FL框架
tags:
  - MINE_FL
原始链接:
---

### 4. 服务器-客户端交互机制

#### 模型传输流程：

```
1. 下行传输 (send_models):
       for client in self.selected_clients:
           client.set_parameters(self.global_model)  # 复制全局模型到客户端
2. 上行传输 (receive_models):
   for client in self.selected_clients:
       self.uploaded_models.append(client.model)      # 收集客户端模型
       self.uploaded_weights.append(client.train_samples)  # 收集样本权重
```

