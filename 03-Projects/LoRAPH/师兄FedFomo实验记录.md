
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

----
现在跑完了算法本身的模型，接下来要把FedFomo与APH和LoRAPH结合：
### 2）

PS:是否需要测试OOD数据呢？
tiny的globalround是20，为什么呢，我用的100。。。。

重组投影头的时候，我加了一行代码，分离base head...不分离总是报错
![[image-50.png]]


---
