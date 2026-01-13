## Slurm

### sinfo
- 显示全部节点
```bash
$ sinfo 
$ sinfo -N
```

- 常见节点状态对比
```bash
idle: 空闲，可接受作业。
alloc (allocated): 已分配/被全部占用。
mix: 部分占用，但仍有剩余资源。
down: 节点宕机或完全不可用。
drain/drained: 停止接收新作业，等待旧作业结束。
```

- 查看集群的全部节点数
```bash
$ scontrol show node |  grep "mem=" | grep "cpu=" | cut -d , -f 1 | cut -d = -f 3 | awk '{sum+=$1}END{print sum}'
```



### squeue
任务查看

```bash
# 查看信息（字段，参数）
man squeue

# 帮助信息
squeue --help

# 查看全部任务
squeue

# 查看自己的任务
squeue -u `whoami`

# 指定输出格式查看自己的任务
squeue -o "%.18i %.9P %.12j %.12u %.12T %.12M %.16l %.6D %R" -u zhusitao

# 简写
echo "alias sq='squeue -o "%.18i %.9P %.12j %.12u %.12T %.12M %.16l %.6D %R" -u zhusitao'" >> ~/.bashrc

```
### sbatch 

```bash
cat xxx.slurm
#!/bin/bash

#SBATCH --job-name=IndexBuild
#SBATCH --account=songlab
#SBATCH --partition=computerPartiton
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=6
#SBATCH --mail-type=end
#SBATCH --mail-user=zhusitao@whu.edu.cn
#SBATCH --output=%j.out
#SBATCH --error=%j.err
sh work.sh
```
```bash
sbatch xxx.slurm 
```

- 指定node
```bash
#SBATCH --nodelist=n[0050-0053],n0180
```

### sacctmgr
- 查看用户的优先级
```bash
sacctmgr show ass user=`whoami`  format=user,part,qos

```

### scancel

- 取消作业ID为92的作业
```bash
scancel 92
```

- 取消自己名下的全部作业
```bash
# 注意whoami前后不是单引号
scancel -u `whoami`
```

- 取消自己账号下所有状态为PENDING的作业
```bash
scancel -t PENDING -u `whoami`
```

- scancel 常见参数
```bash
--help                # 显示scancel命令的使用帮助信息；
-A <account>          # 取消指定账户的作业，如果没有指定job_id,将取消所有；
-n <job_name>         # 取消指定作业名的作业；
-p <partition_name>   # 取消指定分区的作业；
-q <qos>              # 取消指定qos的作业；
-t <job_state_name>   # 取消指定作态的作业，"PENDING", "RUNNING" 或 "SUSPENDED"；
-u <user_name>        # 取消指定用户下的作业；
```

### sacct & scontrol
- scontrol
```
scontrol show job 92
```

- sacct
```bash
sacct -j 92
```
