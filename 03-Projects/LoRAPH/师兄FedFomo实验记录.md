
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
OOM:
1. 出现在重组投影头的时候，main没有split（这是为什么）
2. aph 评估，cifar10结束之后，恢复cifar100的snapshot的时候会出现OOM
3.  问一下师兄，一般同时加载多少客户端，会OOM？我们有什么办法去避免OOM,或者遇到OOM之后应该如何做？ 
	1. 上一个数据集结束的时候Release memory
	2. restore_from_pickle 反序列化时会把快照里的所有客户端模型直接 to(self.device) 上 GPU（PFL/system/flcore/servers/serverbase.py:617-618）。在这一瞬间显存就被一次性占满，导致报错退出，所以现在先把快照加载到CPU，或者，
	3. 在 restore_snapshot 加参数跳过 client_models/clients[i].model 的 GPU 上屏，评测时按需拉取。


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




这个月的我的主线任务是：完成了师兄分配给我的实验任务，并给师兄汇报了，然后把期间自己的步骤，和卡住的点记录下来，跟师兄交流。
通过这个小项目，现在能搞清楚整个实验流程，并且会动手设计实验，评估等等，并且弄明白了师兄之前的工作，以及现有框架。![[image-55.png|555x293]]

下个月我目前计划是：从近期的三大会中，找到一个热门的细分方向，去读这个细分方向论文，争取能有自己的想法。
国庆期间我会学习数理统计这门课程，因为要交中期作业了。。。