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

- 查看集群的全部节点和内存
```bash
# %N  节点名
# %c  CPU 核心数
# %m  内存大小，单位 MiB
# %t  节点状态

sinfo -N -h -o "%N %c %m" | awk '{printf "%-15s CPU=%-5s Memory=%.1f GB\n",$1,$2,$3/1024}'
```

- 日常巡检
```bash
%N：节点名
%C：CPU 状态，格式是 allocated/idle/other/total
%m：节点总内存，MiB
%e：节点当前空闲内存，MiB
%T：节点状态

# 简写
sinfo -N -o "%N %C %m %e %T"

# 全称
sinfo -N -O NodeList,CPUsState,Memory,AllocMem,FreeMem,StateLong

# 格式化输出
sinfo -N -h -O NodeList,CPUsState,Memory,AllocMem,FreeMem,StateLong | \
awk '{
    printf "%-8s CPU(A/I/O/T)=%-13s Total=%.1fG Alloc=%.1fG Free=%.1fG State=%s\n",
    $1,$2,$3/1024,$4/1024,$5/1024,$6
}'
```

### squeue
- 任务状态
```bash

idle
= 完全空闲
= CPUAlloc = 0

mixed
= 部分资源已使用，部分仍空闲
= 0 < CPUAlloc < CPUTot

idle
0/64/0/64

mixed
16/48/0/64

allocated
64/0/0/64
```

看所有 mixed 节点上的任务
```bash
squeue -o "%.18i %.10u %.20j %.8T %.8C %.10m %.20R"
```

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

### scontrol
- scontrol查看全部信息
```
scontrol show job 92
```

- 查看特定信息：JobID,JobName,State,ExitCode,Elapsed,MaxRSS
```bash
sacct -j 20603 --format=JobID,JobName,State,ExitCode,Elapsed,MaxRSS
```

| 用途 | 命令 | 说明 |
|---|---|---|
| 查看任务详情 | `scontrol show job JOBID` | 查看任务状态、CPU、内存、节点、依赖关系、时间限制等 |
| 查看所有任务 | `scontrol show jobs` | 显示当前可见任务的详细信息 |
| 查看节点 | `scontrol show node` | 查看所有节点状态 |
| 查看指定节点 | `scontrol show node cu01` | 查看 `cu01` 的 CPU、内存、状态、正在运行任务等 |
| 查看分区 | `scontrol show partition` | 查看所有 partition |
| 查看指定分区 | `scontrol show partition cpu-1` | 查看 `cpu-1` 分区配置 |
| 查看 Slurm 配置 | `scontrol show config` | 查看当前生效的 Slurm 配置 |
| 查看 reservation | `scontrol show reservation` | 查看预约资源 |
| 查看任务依赖 | `scontrol show job JOBID \| grep Dependency` | 检查 `afterok` 等依赖 |
| 暂停排队任务 | `scontrol hold JOBID` | 让 Pending 任务保持等待，不再被调度 |
| 恢复 hold 任务 | `scontrol release JOBID` | 解除 hold |
| 重新排队 | `scontrol requeue JOBID` | 将运行/结束的 batch job 重新放回 Pending |
| 挂起运行任务 | `scontrol suspend JOBID` | 暂停运行；通常需要较高权限 |
| 恢复挂起任务 | `scontrol resume JOBID` | 恢复 suspended job |
| 修改任务 | `scontrol update JobId=JOBID ...` | 修改允许动态修改的参数 |
| 查看 batch 脚本 | `scontrol write batch_script JOBID` | 把提交时的 sbatch 脚本恢复出来 |
| 展开节点列表 | `scontrol show hostnames 'cu[01-05]'` | 输出 cu01、cu02… |
| 压缩节点列表 | `scontrol show hostlist cu01 cu02 cu03` | 转成类似 `cu[01-03]` |
| 查看版本 | `scontrol version` | 查看 Slurm/scontrol 版本 |
| 帮助 | `scontrol help` | 查看命令帮助 |


### sacct 
- sacct
```bash
sacct -j 92
```


### slurm 任务的状态汇总

| 缩写 | 全称 | 中文含义 | 典型说明 |
|---|---|---|---|
| `BF` | `BOOT_FAIL` | 启动失败 | 节点启动/硬件启动失败导致任务终止 |
| `CA` | `CANCELLED` | 已取消 | 被用户或管理员取消 |
| `CD` | `COMPLETED` | 已完成 | 正常结束，退出码为 0 |
| `CF` | `CONFIGURING` | 配置中 | 已分配资源，但节点还在启动/配置 |
| `CG` | `COMPLETING` | 正在结束 | 程序基本结束，但还在清理资源 |
| `DL` | `DEADLINE` | 超过截止时间 | 因 deadline 限制被终止 |
| `F` | `FAILED` | 失败 | 非 0 退出码或其他失败 |
| `NF` | `NODE_FAIL` | 节点失败 | 运行节点故障导致任务终止 |
| `OOM` | `OUT_OF_MEMORY` | 内存不足 | 超过内存限制，被 OOM kill |
| `PD` | `PENDING` | 排队等待 | 尚未获得资源或条件未满足 |
| `PR` | `PREEMPTED` | 被抢占 | 被更高优先级任务抢占 |
| `R` | `RUNNING` | 运行中 | 已获得资源并正在执行 |
| `RD` | `RESV_DEL_HOLD` | 预约删除后挂起 | reservation 被删除导致任务 hold |
| `RF` | `REQUEUE_FED` | 联邦重排队 | federation 环境中重新排队 |
| `RH` | `REQUEUE_HOLD` | 重排队并挂起 | 已重排队，但暂不参与调度 |
| `RQ` | `REQUEUED` | 已重新排队 | 任务正在重新进入队列 |
| `RS` | `RESIZING` | 调整资源中 | 作业资源规模正在改变 |
| `RV` | `REVOKED` | 已撤销 | federation sibling job 被撤销 |
| `SI` | `SIGNALING` | 正在发送信号 | Slurm 正在向作业发送信号 |
| `SE` | `SPECIAL_EXIT` | 特殊退出 | 特殊状态下重新排队 |
| `SO` | `STAGE_OUT` | 数据导出中 | 正在执行 stage-out |
| `ST` | `STOPPED` | 已停止 | 收到 `SIGSTOP`，资源仍被占用 |
| `S` | `SUSPENDED` | 已挂起 | 执行暂停，CPU 可被释放给其他任务 |
| `TO` | `TIMEOUT` | 超时 | 达到 `--time` 限制被终止 |
