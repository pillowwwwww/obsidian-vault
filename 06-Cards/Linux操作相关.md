---
Title: 
tags: 
原始链接:
---

-  重命名文件夹 mv 原名字 新名字
-  cat 打开文件 只阅读


#### linux删除文件

危险！！

- rm 文件名
- rm 文件1 文件2 文件3
- 删除文件夹（递归删除）：rm -r 文件名

**conda 装的包 → 用 `conda remove` 卸载**，不要用 `pip uninstall`。

- **查**：`pgrep -af 关键字`、`ps -p PID -o ...`
    
- **看**：`top -H -p PID`、`nvidia-smi`
    
- **日志**：`tail -f 日志文件`
    
- **停**：`kill PID` → 不行再 `kill -9`
    
- **后台**：`nohup ... & echo $!`、`bg / disown`
    
- **空间**：`df -h`、`du -sh`



### cifar10 实验
- 使用 nohup python main.py --config template/fedfomo_cifar10_pr0.2.yaml > /data2/lc/experiments/logs/FedFomo_$(date +%Y%m%d_%H%M%S).log 2>&1 &   进行挂起
  - 1. 看日志输出（最常用）

	你在命令里已经指定了日志文件：
	
	`/data2/lc/experiments/logs/FedFomo_20250914_xxxxxx.log`
	
	可以用以下命令实时查看：
	
	`tail -f /data2/lc/experiments/logs/FedFomo_*.log`
	
	或者只看最后 50 行：
	
	`tail -n 50 /data2/lc/experiments/logs/FedFomo_*.log`

	这样你能看到训练进度。
- 