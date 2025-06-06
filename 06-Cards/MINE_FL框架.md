---
Title: MINE_FL框架
tags:
  - MINE_FL
原始链接:
---
# 流程图

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

框架总览：
[图片](https://www.mermaidchart.com/app/projects/1a42df6b-4fd4-4927-9a99-6d35412a4b6a/diagrams/52fc3cff-ff77-49a6-9ded-1f5781ba1bb2/share/invite/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkb2N1bWVudElEIjoiNTJmYzNjZmYtZmY3Ny00OWE2LTlkZWQtMWY1NzgxYmExYmIyIiwiYWNjZXNzIjoiRWRpdCIsImlhdCI6MTc0OTIwNDA5NX0.qnQEhcFvJj1mWpJnhOKUOkDPUWmookmexcm9X7SpkCc)：
```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "12px"}, "config": {"viewport": {"width": 800,"height":400}} }}%%
graph TB
    subgraph "Main Entry"
        A["程序启动"] --> B["load_config"]
        B --> C["mk_exp_dirs 创建实验目录"]
        C --> D["设置信号处理器"]
        D --> E["多次实验循环"]
    end
    
    subgraph "单次实验流程"
        E --> F["set_seed 设置随机种子"]
        F --> G["initialize_model 初始化模型"]
        G --> H["initialize_server 创建FedAvg服务器"]
        H --> I["server.train 开始训练"]
        I --> J["实验结束清理内存"]
    end
    
    subgraph "FedAvg服务器初始化"
        H --> K["FedAvg.__init__"]
        K --> L["super().__init__ 调用Server基类"]
        L --> M["set_clients 创建所有客户端"]
        M --> N["初始化Budget等属性"]
    end
    
    subgraph "FedAvg训练主循环"
        I --> O["for i in range(global_rounds + 1)"]
        O --> P["select_clients 选择参与客户端"]
        P --> Q["send_models 发送全局模型"]
        Q --> R["客户端并行训练"]
        R --> S["receive_models 收集客户端模型"]
        S --> T["aggregate_parameters 聚合参数"]
        T --> U["evaluate 评估性能"]
        U --> V{"继续训练?"}
        V -->|是| P
        V -->|否| W["save_results 保存结果"]
    end
    
    subgraph "客户端训练详情"
        R --> X["client.set_curr_round"]
        X --> Y["client.train 客户端训练"]
        Y --> Z["load_train_data 加载训练数据"]
        Z --> AA["本地SGD优化"]
        AA --> BB["更新本地模型参数"]
    end
    
    subgraph "服务器聚合过程"
        T --> CC["收集客户端权重和模型"]
        CC --> DD["处理攻击/防御逻辑"]
        DD --> EE["加权平均聚合"]
        EE --> FF["更新全局模型"]
    end
    
    style A fill:#e1f5fe
    style K fill:#f3e5f5
    style O fill:#fff3e0
    style Y fill:#e8f5e8
    style T fill:#ffecb3