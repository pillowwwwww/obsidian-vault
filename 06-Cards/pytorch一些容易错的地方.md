---
Title: pytorch一些容易错的地方
tags:
  - pytorch
原始链接:
---
1. **定义**：
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