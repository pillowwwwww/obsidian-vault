---
Title: pytorch一些容易错的地方
tags:
  - pytorch
原始链接:
---
# 1.
**定义**：
“与设备无关”指的是代码可以在 CPU 或 GPU 上自动运行，而无需手动更改 `"cpu"` 或 `"cuda"`。
### ✅ 设备无关写法（推荐）：

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
X = X.to(device)  # 自动适应当前硬件环境
```

---
 错误解释：Expected all tensors to be on the same device

当模型的参数在 GPU 上，但数据还在 CPU 上，就会报错：

```
RuntimeError: Expected all tensors to be on the same device, but found at least two devices, cuda:0 and cpu!
```

**原因**：`eval_model()` 函数中忘了把 `X` 和 `y` 移动到同一设备。

# 2. device
📌 PyTorch 中什么时候需要注意 `.to(device)`？

🧠 总原则：
所有参与张量计算的对象（模型参数、输入数据、标签）必须在同一个设备上（CPU 或 GPU）。

---

🧭 需要注意 `device` 的关键场景：

1️⃣ 模型初始化后
```python
model = MyModel()
model.to(device)  # ✅ 将模型参数移动到指定设备
```
常见错误：模型已定义但未转移到 GPU，后续训练时报错。

---

2️⃣ 训练/测试循环中，每个 batch 的数据
```python
for X, y in dataloader:
    X, y = X.to(device), y.to(device)
```
常见错误：模型在 GPU 上但数据仍在 CPU，计算时报 device mismatch 错误。

---

3️⃣ 评估函数中（如 eval_model）
```python
X, y = X.to(device), y.to(device)
model.to(device)
```
推理阶段同样需要注意设备统一，封装函数时尤其容易忽略这一点。

---

4️⃣ 自定义张量时
```python
x = torch.tensor([1, 2, 3], device=device)
```
常见错误：手动创建的 tensor 默认在 CPU，和 GPU 模型计算时报错。

---

5️⃣ 模型加载后
```python
model.load_state_dict(torch.load("model.pth"))
model.to(device)
```
常见错误：从 CPU 存的模型直接用在 GPU 上未 `.to(device)`。

---

✅ 一张表总结

| 场景             | 是否需要 `.to(device)`？ | 示例代码                         |
|------------------|--------------------------|----------------------------------|
| 模型创建后       | ✅                       | `model.to(device)`              |
| 每个 batch 数据  | ✅                       | `X.to(device), y.to(device)`    |
| 测试 / 推理阶段  | ✅                       | `model.eval(); X.to(device)`    |
| 创建新张量       | ✅                       | `torch.tensor(..., device=...)` |
| 加载模型         | ✅                       | `model.to(device)`              |
| 损失函数/优化器  | ❌                       | 模型参数正确即可                |

---

✅ 推荐写法模板

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

model = MyModel().to(device)

for X, y in dataloader:
    X, y = X.to(device), y.to(device)
    y_pred = model(X)
```

建议所有封装函数（如 `train_step()`、`eval_model()`）都支持传入 `device` 参数，确保通用性和跨平台运行能力。



