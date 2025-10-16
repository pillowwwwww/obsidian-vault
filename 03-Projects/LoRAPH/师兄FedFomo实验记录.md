
## task：
(1) 把pflib中的FedALA或者FedFomo算法迁移到咱们的框架下，我自己感觉现在这个框架易用性和可复现性比原来的PFLib强
(2) 选两个中的一个，在Pr=0.2的情况下，在Cifar10， Cifar100 以及 TinyImageNet训练，保存模型。
(3) 在模型基础上，在该算法的Models的基础上用APH与LoRAPH训练重组投影头，保存。
(4) 保存之后，用脚本评估指标，为表中补充一行数据

---
### 1）
今天我对94服务器做了一些调整：
根盘满了，我把我的环境 数据放到data2盘：
- `/data2/lc/cache/conda_pkgs` → 专门用来放 conda 的 **包缓存**（所有下载的 `.tar.bz2` 包）。
- `/data2/lc/tmp` → 专门作为 **临时目录**（解压 / 构建时的临时文件）。  
    👉 这样我们确保以后下载和解压都放在大盘 **/data2**，不再用满的根盘。
- 设置环境变量 
    export CONDA_PKGS_DIRS=/data2/lc/cache/conda_pkgs
	CONDA_PKGS_DIRS = conda 用来存放下载包的缓存目录。
	👉 我们强制指定它去 /data2/lc/cache/conda_pkgs，而不是默认的 /home/liuchang/miniconda3/pkgs（根盘）。
-    export TMPDIR=/data2/lc/tmp
	export TMP=/data2/lc/tmp
	export TEMP=/data2/lc/tmp
	TMPDIR / TMP / TEMP = Linux 常见的临时目录变量，很多程序（包括 conda / pip）在编译或解压时会用这些。
	👉 我们让所有临时文件都写到 /data2/lc/tmp，避免根盘 /tmp 被塞满。
- 让这些环境变量永久生效
	echo 'export CONDA_PKGS_DIRS=/data2/lc/cache/conda_pkgs' >> ~/.bashrc
	echo 'export TMPDIR=/data2/lc/tmp' >> ~/.bashrc
	echo 'export TMP=/data2/lc/tmp'   >> ~/.bashrc
	echo 'export TEMP=/data2/lc/tmp'  >> ~/.bashrc
- 让以后的新虚拟环境也都装到data2下面
	在 `~/.condarc` 文件里写：

```
envs_dirs:
  - /data2/lc/conda_envs
pkgs_dirs:
  - /data2/lc/cache/conda_pkgs

```

#### 什么时候用 **conda install**？
- **优先选 conda**，尤其是：
    - **大科学计算库**（pytorch、tensorflow、cudatoolkit、numpy、scipy、pandas、matplotlib…）
    - **有 C/CUDA 依赖的库**（比如 opencv、xgboost、faiss…）
- 因为 conda 会帮你解决底层二进制依赖（MKL、CUDA、cuDNN、OpenMP…），pip 往往不会处理这些。

**conda 装的包 → 用 `conda remove` 卸载**，不要用 `pip uninstall`。
我使用conda安装了一些大件。
```
conda install -y -c pytorch -c nvidia pytorch torchvision pytorch-cuda=12.1
```
其它的包使用pip install吧

### 一些实验的命令行

- 使用 nohup python main.py --config template/fedfomo_cifar10_pr0.2.yaml > /data2/lc/experiments/logs/FedFomo_$(date +%Y%m%d_%H%M%S).log 2>&1 &   进行挂起
  - 1. 看日志输出（最常用）

	你在命令里已经指定了日志文件：
	
	`/data2/lc/experiments/logs/FedFomo_20250914_xxxxxx.log`
	
	可以用以下命令实时查看：
	
	`tail -f /data2/lc/experiments/logs/FedFomo_*.log`
	
	或者只看最后 50 行：
	
	`tail -n 50 /data2/lc/experiments/logs/FedFomo_*.log`

	这样你能看到训练进度。

- 查看当前显卡的进程：
```
# 只列出有计算上下文的进程（没有就什么都不输出）
nvidia-smi --query-compute-apps=gpu_uuid,pid,process_name,used_memory --format=csv,noheader

# 只看第 3 号卡的进程
nvidia-smi -i 3 --query-compute-apps=pid,process_name,used_memory --format=csv,noheader
```

- 查看当前进程：
```
ps -p PID -o pid,ppid,stat,stime,etime,%cpu,%mem,cmd
# 若进程不在了，会输出not running
ps -p <PID> -o pid,ppid,stat,etime,%cpu,%mem,cmd || echo "not running"

```



---
## 调参
### 一些知识点
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

----
现在跑完了算法本身的模型，接下来要把FedFomo与APH和LoRAPH结合：
### 2）

PS:是否需要测试OOD数据呢？
tiny的globalround是20，为什么呢，我用的100。。。。

重组投影头的时候，我加了一行代码，分离base head...不分离总是报错
![[image-50.png]]

---
## 评估：

### 1. FedFomo基线评估（w/o） - 使用 eval_script_fedavg.py
- 直接使用fedavg的测评方法测评fedfomo是不行的，acc算出来只有10%，因为使用了全局模型
```
	      model = server.global_model
```
- 想看 FedFomo 的表现：需要对 client_models 逐个评估，然后按样本数/权重加权平均。框架训练日志已经在 serverFomo.evaluate() 中做了这件事，所以看到 0.76+ 的准确率。

---
## 查找进程：
你遇到的情况其实是 **`jobs` 和 `ps` 的区别** 导致的，看起来像“进程没了”，但其实还在运行：

### 关键点

1. **`jobs` 只能显示当前 shell（前台/后台会话）中由 `&` 启动的任务**。
    
    - 一旦你退出过 shell，或者进程被 `nohup` 脱离了控制终端（典型就是 `nohup python ... &`），这个任务就 **不会再出现在 `jobs`** 里。
        
    - 所以你虽然还看到日志在更新，但 `jobs` 不会再列出它。
        
2. **进程还在运行**，因为 `nohup` 会让它在后台独立运行，不依赖于当前 shell。
    
    - 日志能持续更新就是最好的证明。
#### 查找 python 进程：

`ps -ef | grep python | grep -v grep`
只看自己的：
`ps -u liuchang | grep train_ | grep -v grep`



---

---
## OOM错误:
#### 问题：
1. 出现在重组投影头的时候，main没有split（这是为什么）
2. aph 评估，cifar10结束之后，恢复cifar100的snapshot的时候会出现OOM
3.  问一下师兄，一般同时加载多少客户端，会OOM？我们有什么办法去避免OOM,或者遇到OOM之后应该如何做？ 
	1. 上一个数据集结束的时候Release memory
	2. restore_from_pickle 反序列化时会把快照里的所有客户端模型直接 to(self.device) 上 GPU（PFL/system/flcore/servers/serverbase.py:617-618）。在这一瞬间显存就被一次性占满，导致报错退出，所以现在先把快照加载到CPU，或者，
	3. 在 restore_snapshot 加参数跳过 client_models/clients[i].model 的 GPU 上屏，评测时按需拉取。

**现在已知**：是因为FedFomo会在服务器中为每一个客户端保存一个备份，导致容量太大，恢复的时候出现OOM，是FedFomo的特性。
#### 关于重新加载模型的过程:
该框架评估aph的时候，为什么需要**快照恢复**而不只是**加载权重**？
- 你这里之所以用了“快照恢复”，是因为训练阶段只保存了“整台 server 的快照（包含每个客户端的模型）”，并没有把每个客户端的权重单独导出。要拿到每个客户端的个性化模型，只能先把快照恢复出来再抽取。

**什么时候用“快照恢复”**？

- 恢复训练现场：继续训练、精准复现一次训练中断时的“运行态”（全局模型、每个 client 的模型、副本、优化器、轮次、亲和矩阵等）。
- 审计/分析内部状态：需要看 FedFomo 的亲和矩阵 P、每轮参与名单、历史缓存等“非权重信息”。
- 没有单独保存客户端权重时：快照里“带着所有客户端的模型”，恢复一下就能统一拿到。

**什么时候用“加载权重（state_dict）”**

- 纯评估：只要“模型参数 + 数据”即可（比如 APH 的 Acc/NLL/ECE），不需要 server 的其它状态。
- 轻量、稳定、设备无关：恢复出的就是 CPU 张量，按需迁移到 GPU；不依赖训练时的代码环境，出错率低。
- 推荐做法：训练阶段“额外”把每个客户端模型另存为 client_{id}.pt（state_dict），评估时直接 initialize_model(config) → load_state_dict → rearrange_model → 评估，完全跳过快照。

**你现在为什么在这个位置需要快照**？

- 你的产物是 best_snapshot.pkl，里面才包含“每个客户端的个性化模型”。想评 PFL 的 20 个客户端，就得先把 server 恢复出来再抽取它们。否则你手里没有各客户端的单独权重文件可用。
- - 快照（server-level）
    - 优点：一份文件全包含；可恢复训练、分析内部状态。
    - 缺点：重/大；恢复时会反序列化+深拷贝+设备迁移，容易在 GPU 上出现显存峰值（尤其 FedFomo 持有 20 份模型）。
- 权重（model-level，state_dict）
    - 优点：轻量、设备无关、评估足够；只占一个模型的显存。
    - 缺点：需要训练时单独导出每个客户端的权重；否则后处理要先通过快照把它们“导出来”。

在我们的MINIE_FL框架中，这里的“恢复”分成两步，分别是 restore_from_pickle(...)反序列化 和 restore_snapshot()，它们合起来完成“把磁盘上的快照应用到当前 Server 实例”。
- 反序列化（restore_from_pickle → pickle.load）：把快照文件读成内存里的“快照容器”self._snapshot。
- 应用快照（restore_snapshot）：把“快照容器”深拷贝到 Server 的各个属性，并把客户端模型搬到目标设备，Server 才真正处于“恢复后的运行态”。

----
## 汇报：

1. 把FedFomo迁移到MINI_FL框架上
2. 写yaml实验配置：         
					times写几次？- 我看之前的yaml是1/3/5都有；
					 search: true   - 是否要进行超参数搜索？（FedFomo有三个参数 M、ε、val 比例，最后只用了mu，这是为什么呢？）
					 客户端参与率、lr、local epouch、ground rounds这些，一般先使用comme setting, 在自己做实验的时候，在一点点尝试哪个设置更好嘛? 现在在测试别人的算法，所以就沿用同意的setting就可以吗？
3. 划分数据集 ：
					dir/pat怎么选呢，我现在是完全仿照师兄之前的设置，划分的数据集（Cifar10_NonIID_Pat2_Client20、Cifar100_NonIID_Dir0.05_Client20、TinyImageNet_NonIID_Dir0.05_Client20）
4. 训练基线w/o模型，保存
5. 训练APH/LoRAPH重组投影头，保存：
					我一开始在main函数中没有把FedFomo进行分离（ split_model = rearrange_model(init_model)）导致总是报错OOM，又删掉之前训练的模型重新训练的。
	![[image-53.png|670x737]]
6. 评估基线模型 
				fedfomo的基线模型评估脚本之前是用来评估FedAvg的，要重写，需要对 client_models 逐个评估，然后按样本数/权重加权平均。
7. 评估APH/LoRAPH重组后
		改写评估脚本后，用的【eval_script_aph_compatible.py】，
```
global_epoch = 10

sampling_num = 10

upper_lr = 0.1

lower_lr = 0.01

p_lambda = 0.1
```
但是总是遇到OOM的错误（发现可能是FedFomo算法需要为每个客户端维护一个模型副本？可能是这个原因？）
			我尝试了以下办法： 1.只把当前要测的模型暂时迁到 GPU，用完立刻挪回 CPU，并torch.cuda.empty_cache()
			2. 把 config.general.device、server.device 强制设为 'cpu'，；并在循环末尾清理引用、触发垃圾回收。
		
都不行，但是可用的memory变多了，但还是报错OOM。

国庆给老师发的汇报：
这个月的我的主线任务是：完成了师兄分配给我的实验任务，并给师兄汇报了，然后把期间自己的步骤，和卡住的点记录下来，跟师兄交流。
通过这个小项目，现在能搞清楚整个实验流程，并且会动手设计实验，评估等等，并且弄明白了师兄之前的工作，以及现有框架。![[image-55.png|555x293]]

---
# 实验2 ： ECE vs. Dir参数 以及 ECE vs. 参与率的变化图

base模型已训练好（/home/jqchen/code/MINE_FL/Uncertainty/LoRAPH/temp_script/Avg_pr_models），你试着对这12个模型分别用APH和LoRAPH去提升其ECE，保存一下数据。

1. 使用train_aph_script重组。
2. 目前是针对pkl文件，通过get models.py把它转换成pt文件。注意：可以双层嵌套遍历+使用测试客户端去接收参数
3. 
```
训练APH模型

- **训练脚本**: `train_aph_sequence.py`

- **训练日志**: `aph_sequence.log` (包含所有训练过程的ECE记录)

- **保存路径**: `aph_heads/aph_head_{dataset}_pr_{pr}.pkl`

训练LoRAPH模型

- **训练脚本**: `train_loraph_sequence.py`

- **训练日志**: `loraph_sequence.log` (包含所有训练过程的ECE记录)

- **保存路径**: `aph_heads/loraph_head_{dataset}_pr_{pr}.pkl

---
```

4. 把基线模型，APH，LoRAPH的ECE提取出来，绘图。
师兄建议我：固定那个啥，比如说你固定PR，然后去变啊，比如说你PR在0.0.1，然后你的纵轴是ECE，然后横轴是那个那个。迪利克粒分布那个参数阿尔法，然后把把那三个方法（基线，APH, LoRAPH)分别是3条线或者3个那个把那种柱状图这样去画。比较直观，你这样画看不出来趋势的，因为咱们需要的是方法之间的纵向对比。

5. 根据师兄的绘图脚本修改：
  
 1️⃣ **matplotlib论文级配置** (完全借鉴)

**来源**: `plot_heatmap.py` 第9-11行, `plot_pr_dir_bar.py` 第8-10行

```python

# 师兄的配置（我100%采用）

plt.rcParams["font.family"] = "Times New Roman"

plt.rcParams["font.size"] = 20

plt.rcParams['pdf.fonttype'] = 42  # 确保PDF字体可编辑

```

**学到的知识点**:

- `pdf.fonttype = 42` 保证生成的PDF字体是TrueType，可在Adobe Illustrator等软件中编辑

- 字号20适合学术论文的图表尺寸

- Times New Roman是SCI论文标准字体


2️⃣ **专业配色方案** (完全采用) **来源**: `plot_pr_dir_bar.py` 第14行

```python

# 师兄的配色（我完全照搬）

COLORS = ["#4E79A7", "#F28E2B", "#59A14F"]  # 蓝橙绿，论文风格

```

**学到的知识点**:
- 这组配色来自Tableau专业调色板
- 色盲友好（colorblind-safe）
- 适合黑白打印（对比度高）
- 符合学术期刊审美标准  

**颜色含义**:
- `#4E79A7` (蓝色): 稳重，适合基线方法
- `#F28E2B` (橙色): 醒目，适合改进方法
- `#59A14F` (绿色): 清新，适合最优方法

3️⃣ **柱状图+折线图组合** (核心借鉴 ⭐⭐⭐) **来源**: `plot_pr_dir_bar.py` 第52-84行


```python

# 师兄的巧妙设计

# 步骤1: 先画柱状图

for i, alg in enumerate(ALG_ORDER):

    centers = x + (i - 1) * width

    heights = np.asarray(data[alg], dtype=float)

    plt.bar(centers, heights, width=width,

            color=COLORS[i], edgecolor='black', linewidth=0.8, alpha=0.6)

    bar_centers_per_alg[alg] = centers

    tops_per_alg[alg] = heights

  

# 步骤2: 再叠加折线（连接柱顶）

for i, alg in enumerate(ALG_ORDER):

    plt.plot(bar_centers_per_alg[alg], tops_per_alg[alg],

             linestyle='-', marker='o', linewidth=2,

             color=COLORS[i], label='_nolegend_')

```

**总结**：绘图模板去找temp_script/plot_开头的这三个文件

6. ![[a0bd2dd42a0e6a433d53fd3c6af624a7.png]]