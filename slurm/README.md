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
# 帮助信息
squeue --help

# 查看全部任务
squeue

# 查看自己的任务
squeue -u `whoami`

# 指定输出格式查看自己的任务
squeue -o "%.18i %.9P %.12j %.12u %.12T %.12M %.16l %.6D %R" -u zhusitao

# 简写
echo "alias sq='squeue -o \"%.18i %.9P %.12j %.12u %.12T %.12M %.16l %.6D %R\" -u zhusitao'" >> ~/.bashrc

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

sbatch xxx.slurm 
```

### scancel
