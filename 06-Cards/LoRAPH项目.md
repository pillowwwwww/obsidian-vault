---
Title: 
tags: 
原始链接:
---
## 实验前步骤
先从github提交历史中，观察一下选哪些数据集。怎么划分的。

每个yaml中规定的model有时候是resnet18m（cifar100）有时候是cnn。
有一些参数比如学习率，推荐设置的是0.01，那我要用0.01嘛 还是0.1
times =1 还是=5？
师兄数据集放在哪里？我们没有mount了


我发现cifar10的时候学习率是0.01
## 实验步骤
### cifar 10  0.78+-0.00
```

Best personalized accuracy.
0.7822

Average time cost per round.
8.5883
Save record path: /data2/lc/experiments/FedFomo/Cifar10_NonIID_Pat2_Client20/20250916_144353_cnn_static_pr_0.2_ls_5_att_0/Time_4/record/record.h5
成功保存快照到pkl

Average time cost: 858.89s.
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
std for best accurancy:0.006062998479980407
mean for best accurancy:0.7847313719204159
All done!

Storage on cpu
-------------------------------------------------------------------------------
Total Tensors: 1        Used Memory: 512.00B
-------------------------------------------------------------------------------
save config
推送中
```
### cifar 100   0.21+-0.00
```
-------------------------time cost-------------------------19.2282
mean time:2.4379
var time:0.1470

Best personalized accuracy.
0.2135

Average time cost per round.
18.7890
Save record path: /data2/lc/experiments/FedFomo/Cifar100_NonIID_Dir0.05_Client20/20250916_105011_resnet18m_static_pr_0.2_ls_5_att_0/Time_4/record/record.h5
成功保存快照到pkl

Average time cost: 1947.62s.
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
Length:  101
std for best accurancy:0.004252334492058439
mean for best accurancy:0.21028419566430454
All done!

Storage on cpu
-------------------------------------------------------------------------------
Total Tensors: 1        Used Memory: 512.00B
-------------------------------------------------------------------------------
save config
推送中
```

---
## 常用指令


nohup python main.py --config template/fedfomo_cifar10_pr0.2.yaml > /data2/lc/experiments/logs/FedFomo_$(date +%Y%m%d_%H%M%S).log 2>&1 &


  tail -f /data2/lc/experiments/logs/FedFomo_*.log
  

python generate_cifar100.py noniid nobalance dir 20 0.05

---
## 一些知识点
### 一、 参数 vs 超参数

 参数 (Parameters)

- **定义**：模型内部的可学习变量
- **例子**：神经网络的权重(W)和偏置(b)
- **特点**：训练过程中自动更新

```python
# 例如一个简单神经网络
y = W * x + b  # W和b就是参数，会通过梯度下降自动学习
```

超参数 (Hyperparameters)

- **定义**：训练前需要人工设定的配置
- **例子**：学习率、batch_size、网络层数等
- **特点**：需要人工调整，不会自动学习

```python
# 这些都是超参数，需要你提前设定
learning_rate = 0.01
batch_size = 128
epochs = 100
```

### 二、一些实验步骤：
 如果你想快速测试：

```yaml
times: 1
search: false  # 只测试默认值
```

如果你想认真调参：

```yaml
times: 3  # 每组合运行3次，结果更可靠
search: true
# 但建议先缩小搜索空间
M: search: [3, 5]  # 只测试2个值
```

为什么要times>1？
因为深度学习有随机性，单次结果不可靠。
search: 是否超参数调参，如果不search的话，那么不会遍历参数取值。
