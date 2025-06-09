---
Title: pytorch基本知识
tags: 
原始链接:
---
记得看google colab 笔记： 
00、01、02、03
https://colab.research.google.com/drive/13-KqUlaOSNPHa-hJGtZHMDtRBgSXDpvh
### PyTorch Tensor分类

| 名称     | 维度  | 数学含义      | 示例                         | Shape          | 常见用途          |
| ------ | --- | --------- | -------------------------- | -------------- | ------------- |
| **标量** | 0   | 单个数值      | `tensor(42)`               | `()`           | 损失值、超参数       |
| **向量** | 1   | 一维数组      | `tensor([1, 2, 3])`        | `(n,)`         | 特征向量、偏置项      |
| **矩阵** | 2   | 二维数组（行×列） | `tensor([[1, 2], [3, 4]])` | `(m, n)`       | 权重矩阵、灰度图像     |
| **张量** | ≥3  | 多维数组      | `torch.randn(2, 3, 4)`     | `(d1, d2,...)` | 批量数据、RGB图像、视频 |

#### 关键说明
1. **维度**：通过 `tensor.ndim` 或 `len(tensor.shape)` 获取  
2. **标量特殊点**：
   - PyTorch 中scalar是 0 维（数学中可能视为 1 维）
   - 需用 `item()` 提取 Python 数值：`scalar.item()`
3. **高维张量命名**：
   - 3维：通常称 "3D 张量"（如时序数据）
   - 4维：常见于批量处理（如 `(batch, channel, height, width)`）

#### 示例代码
```python
import torch

# 各维度张量创建
scalar = torch.tensor(42)                    # 标量
vector = torch.arange(3)                     # 向量
matrix = torch.ones(2, 2)                    # 矩阵
tensor_3d = torch.rand(2, 3, 4)              # 3维张量

# 维度验证
assert scalar.ndim == 0
assert vector.shape == (3,)
```

### 线性模型Linear
#### `nn.Linear` 的本质
- `nn.Linear(in_features, out_features)` 表示对输入数据做一次 **线性变换**
- 数学表达：`output = weight × input + bias`
- 它只进行加减乘除操作，没有引入任何非线性函数

#### 线性模型只能学到线性的关系

线性模型只能学到 **线性的关系** ，比如：
- 分类任务中：一条直线（2D）、一个平面（3D）、一个超平面（高维）
- 回归任务中：输入和输出之间的线性关系
- 举个例子：
	`model = nn.Linear(2, 1)`
	这个模型可以解决这样的问题：
	- 给你一个点 `(x1, x2)`，预测它是在线的哪一边
	但它无法解决像圆一样的问题，因为圆的边界是曲线，不是直线！
#### 要模型学到曲线，就要在模型中加入 **非线性成分**

#### 为什么可以直接写 model_0(X_test) 来执行前向传播？

 ✅ 原因：
PyTorch 中的所有模型类都继承自 `torch.nn.Module`，这个基类实现了 `__call__()` 方法。
```
class SimpleModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(1, 1)

    def forward(self, x):
        print("Calling forward")
        return self.linear(x)
        

# 创建模型
model = SimpleModel()

# 调用模型
x = torch.tensor([[1.0]])
y = model(x)  # 实际上就是在调用 forward()
```
####  `y_logits` 和 `y_pred_probs` 有什么不同？

| 概念                 | 含义                                | 特点                     |
| ------------------ | --------------------------------- | ---------------------- |
| **`y_logits`**     | 模型的原始输出，未经过归一化或激活函数处理             | 可以是任意实数（正、负、0），不直接表示概率 |
| **`y_pred_probs`** | 经过`sigmoid()`函数后的结果，表示预测为类别 1 的概率 | 值在 [0, 1] 范围内，可解释为概率   |


# pytorch使用图像
### ✅ PyTorch 使用图像数据的两个关键要求：

| 要求                       | 原因            |
| ------------------------ | ------------- |
| 转为 Tensor                | 便于模型处理        |
| 封装为 Dataset 和 DataLoader | 支持批处理、打乱、并行加载 |
###  一、为什么要使用 `torchvision.transforms`？

我们拿到了图像文件（如 `.jpg`），但在 PyTorch 中，**模型不能直接使用图片文件**，必须将图像转成：

> **张量（Tensor）格式**，且尺寸统一，并可以做图像增强（比如随机翻转）。