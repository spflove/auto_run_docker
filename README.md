# run-docker · 自动化测试工具

最新更新：2026-11-16（dev `0.2.11.16.4`）

Linux x86_64 预编译包（版本可用 `run-docker --version` / `--versiondev` 查看）。

## 优点

- 同一集群一次安装，所有节点、任意路径均可用
- 上手成本极低，对用户层脚本零入侵
- 自动创建 / 复用容器、容器快速删除（`rm`）
- 任务结束（正常或异常）后自动释放资源，保留容器
- 机型支持 DCU、NVIDIA
- 自动识别测试框架 vLLM、SGLang
- Server 启动检测与日志错误检测；只读 Server CLI 快照（EP/CP/SP/kvcache 等）
- Client 报告生成（OpenCompass、EvalScope、bench、Hygon 性能报告；`test_type=3` 评分+性能双报告）
- 异常清理（卡顿、常见错误、超时等）
- 自动发送飞书消息（测试报告）；评分写表（`eval`）、性能写云盘目录（`perf`）
- 多种模式：单测、标准、多模型（`-m`）、多机多任务（`-nm`）、Agent 双容器（`-ag`）、多 target 编排（`-pp`）、镜像构建导出（`-bi`）
- 自动拉取镜像、安装构建并打包导出 `tar.gz` 镜像包（`-bi`）
- 多 target / 多节点分发编排（`-pp`）：用 **job yaml** 调度多份队列任务 yaml（`-m`/`-nm`/`-ag`），内网任意节点发起并远程分发
- 批量脚本清洗并生成 yaml（`-sc`）；可生成 `-m` / `-nm` / `-ag` 配置；支持 `-cut` / `-prio`

## 下载与安装

### 1. 下载

到 (https://github.com/spflove/auto_run_docker/blob/main/run-docker-linux.tar.gz) 下载最新：

- `run-docker-linux.tar.gz`

> 集群内也可从共享路径获取同名压缩包（若你们已同步发布）。

### 2. 解压并安装

复制到家目录下任意有权限的目录后执行：

```bash
tar -xvzf run-docker-linux.tar.gz
cd run-docker-linux
bash install.sh
```

安装完成后，**新开终端应可直接使用** `run-docker`。若当前终端尚未生效：

```bash
source ~/.local/lib/auto_docker/path.env
# 或
bash install.sh --fix-path
```

验证：

```bash
run-docker --help
run-docker --version
run-docker --versiondev
```

### 3. 卸载

```bash
bash install.sh --uninstall
# 或
run-docker --un
```

卸载程序后，默认保留 `~/.config/auto_docker/` 用户配置。

### 4. 配置用户 yaml

解压安装后，按自身环境修改配置（首次安装改一次即可，后续重装默认不覆盖）：

```bash
vim ~/.config/auto_docker/user_container.yaml
```

直接运行 `run-docker`（无参数）可查看基本使用说明。

## 公共账户集群

公共账户集群下，各用户宜使用**自己目录里的 yaml**，不要共用别人的配置文件：

```bash
run-docker use-config <自己目录下的 yaml 配置>
```

说明：

- `user_container.yaml` 中各项通常配置一次即可
- 后续换不同测试版本，多数情况只需改 `work_subdirectory`，其余可不动
- 相当于每个用户自己的配置文件；**建议自行备份一份**，新版本迭代时可直接复用
- 配置好后，在该集群任意节点、任意有权限路径均可使用 `run-docker`

示例：

```bash
# 指向自己家目录下保存的配置
run-docker use-config /public/home/<你的账号>/my_configs/user_container.yaml

run-docker --help
```

## 使用前提（重要）

1. **容器**：可手动创建，也可传 `-id` 自动创建（节点上须已有该镜像）。
   - 映射路径需要有权限；OpenCompass 需镜像内已装好并准备好数据集。
   - **不传镜像 ID = 复用容器。复用会重启当前容器。**
2. **脚本路径**：节点上的完整路径；确认文件存在且脚本无误。支持 `.sh` / `.py`。
3. **测试类型**：
   - `0`：自动识别（评分 / 性能；Client 为 `.sh` 或 `.py` 均可按关键字细分）
   - `1`：OpenCompass 精度
   - `3`：评分 + 性能双报告（同一 Client 须同时含评分与性能信号）
4. **飞书**：内置助手；在配置里填 `user_open_id` 即可收个人报告。开通助手与权限后使用。
5. **`-link` 关键字**：`eval`（评分表）/ `perf`（性能云盘）/ `both`（双写，仅 `test_type=3`）。**不再接受旧名 `yxff` / `yxpf`。**

## 运行模式（互斥）

| 模式 | 入口 | 说明 |
|------|------|------|
| 标准任务 | `run-docker <容器> <Server> <Client> <类型> ...` | 完整测试，自定义 Client |
| 单测模型 | `run-docker -c <容器> <Server> ...` | 内置 curl Client；`-c` 须为**第一个**参数 |
| 多模型 | `run-docker -m <yaml> ...` | YAML 批量调度；`-m` 须为**第一个**参数 |
| 多机多任务 | `run-docker -nm <yaml> ...` | 多机串行；`-nm` 须为**第一个**参数 |
| Agent 双容器 | `run-docker -ag <yaml> ...` | Server/Client 分容器；`-ag` 须为**第一个**参数 |
| 多 target 编排 | `run-docker -pp <job.yaml> ...` | 用 **job yaml** 混跑多节点队列任务（`-m`/`-nm`/`-ag`）；`-pp` 须为**第一个**参数 |
| 镜像构建导出 | `run-docker -bi ...` | 装包 / commit / 导出 tar.gz；`-bi` 须为**第一个**参数 |

`-c` / `-m` / `-nm` / `-ag` / `-pp` / `-bi` 互斥，且均须作为命令第一个参数。

### 基本用法

```bash
run-docker <容器名> <Server脚本> <Client脚本> <测试类型> [可选参数...]
run-docker -c <容器名> <Server脚本> [可选参数...]
run-docker -m xxx.yaml [可选参数...]
run-docker -nm xxx.yaml [可选参数...]
run-docker -ag xxx.yaml [可选参数...]
run-docker -pp job.yaml [-fg|--try]
run-docker -bi -ir <镜像> [-cn <名>] [-o <路径>] [-is <脚本>] [-fg]
```

### 必传参数（标准模式）

| 位置 | 说明 | 示例 |
|------|------|------|
| 1 | 容器名 | `my_container` |
| 2 | Server 脚本完整路径 | `/home/user/server.sh` |
| 3 | Client 脚本完整路径 | `/home/user/client.py` |
| 4 | 测试类型：`0` 自动识别 / `1` OpenCompass / `3` 评分+性能双报告 | `0` |

### 常用可选参数

| 参数 | 说明 |
|------|------|
| `-id` / `--imageid` | 镜像 ID；**不传则复用（会重启容器）**；传入则创建新容器（重名自动加后缀） |
| `-st` / `--scheduledtime` | 定时，如 `20260327_1035` 或 `2026-03-27_10:35` |
| `-fr` / `--forcerestart` | 与 `-st` 独立（仅 `-m` / `-nm` / `-ag`）：单独使用则立即强制释放整节点 GPU；与 `-st` 同用则到点后再释放，然后进入原有满卡等待 / 调度 |
| `-opencs` / `--opencompass` | OpenCompass **容器内路径**（仅 `test_type=1`；标准 / 多模型） |
| `-re` / `--reinstat` | 复用他人容器（标准 / 单测） |
| `-df` / `--diagnosefeishu` | 任务前发一条飞书诊断消息 |
| `-fb` / `--faban` `[N]` | 推送到发版群；`N` 为配置中第几个群（从配置读 `faban_chat_id`） |
| `-link <URL或关键字>` | `eval` 写飞书评分表；`perf` 写性能云盘；`both` 双写（仅 `test_type=3`）；也支持 sheets/wiki 直链 |
| `-hp` / `--hygon-perf` | 性能测试优先解析 `hygon_das_pref_test` CSV → `hygon_perf` 报告并发飞书 / 写 `perf`；失败则回退常规报告（评分任务忽略） |
| `-rt` / `--report-type` | `csv` 或 `md`（Client 报告；Server 报错报告固定 md） |

## 使用示例

### 最精简使用

```bash
run-docker my_container /path/to/server.sh /path/to/client.py 0
run-docker -c my_container /path/to/server.sh
run-docker -m xxx.yaml
run-docker -nm xxx.yaml
run-docker -ag xxx.yaml
run-docker -pp job.yaml
run-docker -bi -ir registry.example.com/sglang:tag
```

### 创建新容器

```bash
run-docker my_container /path/to/server.sh /path/to/client.py 0 -id abc123
```

### 定时 + 镜像

```bash
run-docker my_container /path/to/server.sh /path/to/client.py 0 -id abc123 -st 20260327_1035
```

### 定时 + 到点强制释放 GPU（`-m` / `-nm` / `-ag`）

```bash
run-docker -m xxx.yaml -st 20260804_2200 -fr
run-docker -nm xxx.yaml -st 20260804_2200 -fr
run-docker -ag xxx.yaml -st 20260924_2200 -fr
```

> `-fr` 与 `-st` 互相独立：单独使用则立即对各节点（`-m` / `-ag` single 为本机）强制释放整节点 GPU；与 `-st` 同用则到点后再释放，然后走原有等待 / 调度。

### OpenCompass 精度（容器内路径）

```bash
run-docker my_container /path/to/server.sh /path/to/client.py 1 \
  -id c7cf3618181b \
  -opencs /home/docker_data/opencompass
```

> `--opencompass` 现为**容器内路径**，便于复用他人已创建的容器。

### 飞书群推送 / 写表格 / 性能云盘

```bash
run-docker -m xxx.yaml -fb
run-docker -m xxx.yaml -fb 1
run-docker -m xxx.yaml -link 'https://xxx.feishu.cn/wiki/...?sheet=...'
run-docker -m xxx.yaml -link eval      # 评分表快捷关键字
run-docker -m xxx.yaml -link eval-2    # 同上 Wiki 的第 2 个子表
run-docker -m xxx.yaml -link perf      # 性能 CSV 写云盘目录
run-docker -m xxx.yaml -hp -link perf  # Hygon 性能报告优先 + 写入 perf
run-docker my_container /path/s.sh /path/c.sh 3 -link both  # type=3 双写
```

### Agent 双容器（`-ag`）

Server / Client 使用独立容器（可不同镜像）。模板见发布仓库 `agent_template.yaml`。

```bash
# single：本机一对 Server+Client
run-docker -ag agent.yaml
run-docker -ag agent.yaml --foreground -df -fb 1 -link perf -hp

# multi：多机主/work Server + 主节点 Client（须在 nodes[0] 本机启动，对齐 -nm）
run-docker -ag agent_multi.yaml

# 停止
run-docker -ag --stop
run-docker -ag --stop -as              # 按句柄 Client→Server 清理
run-docker -ag --stop 20260924_183000
```

要点：

- `defaults.server_image_id` / `client_image_id`（可相同）
- `task_kind: single | multi`；multi 须配 `model_configs[].nodes`，且 `tp_sum = total_cards × 节点数`
- Client 命名 `{server}_client_{port}`；调度不占卡（创建仍挂 GPU/DCU 与 docker.sock）
- Client→Server 地址由调度器写入（`server_api_host:port`），非同容器回环
- 会话目录：`tmp_batch_agent/<时间戳_节点名>/`（`--stop` 仍只传时间戳）

### 多 target 部署编排（`-pp`）

`-pp` 入口是 **job yaml**（编排清单），不是 `-m`/`-nm`/`-ag` 那类队列任务 yaml。  
job yaml 里通过 `targets` 引用多份队列任务 yaml，可混跑单机（`-m`）/ 多机（`-nm`）/ Agent（`-ag` / `ag_nm_`）。模板见 `batch_pipeline_template.yaml`。

```bash
# 前台 / 试跑
run-docker -pp job.yaml -fg
run-docker -pp job.yaml --try

# 后台（默认）；日志 yx_deploy_logs/batch_pipeline_<TS>.log
run-docker -pp job.yaml

# 停本机编排后台（不杀节点上已启动的业务容器）
run-docker -pp --stop
run-docker -pp --stop yx_deploy_logs/batch_pipeline_20261116_120000.log

# 汇总各节点调度统计
run-docker -pp --stats yx_deploy_logs/batch_pipeline_20261116_120000.log
run-docker -pp -ss 20261116_120000
```

#### job yaml 示例（`batch_pipeline_template.yaml`）

下面是 **job yaml**（编排清单）。`targets[].yaml` 指向的才是 `-m`/`-nm`/`-ag` 队列任务 yaml（可由 `-sc` 生成），二者不是同一种文件。

```yaml
# 镜像来源：docker_images/*.tar.gz / build_*.log / 镜像 SHORT_ID
b_log_or_s_imageid: /path/to/docker_images/my_build_20261116.tar.gz
client_image_id: ""   # 可选；仅 -ag 使用；空=与 server 相同

targets:
  # 单机 -m（队列任务 yaml，如 more_*.yaml）
  - ips: 10.16.1.37
    yaml: /path/to/more_xxx_10.16.1.37_MMDD_HHMMSS.yaml
  - ips: 10.16.1.38
    yaml: /path/to/more_xxx_10.16.1.38_MMDD_HHMMSS.yaml

  # 多机 -nm（队列任务 yaml，如 nm_*.yaml；ips 须与其 nodes 一致）
  - ips: 10.16.1.28,10.16.1.25
    yaml: /path/to/nm_xxx_10.16.1.28_MMDD_HHMMSS.yaml

  # Agent 单机 / 多机（ag_*.yaml / ag_nm_*.yaml）
  - ips: 10.16.1.40
    yaml: /path/to/ag_xxx_10.16.1.40_MMDD_HHMMSS.yaml
  - ips: 10.16.1.3,10.16.1.4
    yaml: /path/to/ag_nm_xxx_10.16.1.3_MMDD_HHMMSS.yaml

# 各节点执行 run-docker 的工作目录（通常为共享盘）
run_cwd: /public/home/<你的账号>/yx_dockerspace

# 按下标对应 targets；不足则复用第一项
run_docker_args:
  - "-fb -link eval"
```

要点：

- job yaml 字段：`targets[{ips, yaml}]` + `b_log_or_s_imageid`（可为 `build_*.log` / `docker_images/*.tar.gz` / 镜像 SHORT_ID）；其中 `yaml` 指向各队列任务 yaml
- `b_log_or_s_imageid` 为 `build_*.log` 时：先等 `-bi` 打包完成 → ensure/load → 再 patch **队列任务 yaml** 的镜像字段
- 内网任意互通节点可发起：本机 IP ∈ targets → 本地执行；否则 SSH 跳到对接节点（同 `-nm` 免密密钥）
- 不自动改写队列任务 yaml 内 `nodes`/IP（`targets.ips` 须与多机队列 yaml.nodes 一致）
- `-pp` 仅停编排进程本身，业务容器请用各模式自己的 `--stop` / `rm`

### 镜像构建导出（`-bi`）

从基础镜像创建容器 → 安装软件 → commit → 导出 `tar.gz`。产物可被 `-pp` 的 `b_log_or_s_imageid` 消费。

```bash
# 从镜像新建构建容器
run-docker -bi -ir registry.example.com/sglang:0.5.18
run-docker -bi -ir registry.example.com/sglang:0.5.18 -cn my-build -fg

# 复用已有容器继续构建
run-docker -bi -cn my-existing-container

# 指定安装脚本 / 导出路径（默认安装脚本读 user_container.yaml → build_image_install）
run-docker -bi -ir <镜像> -is /path/to/build_install.sh -o /path/to/out

# 停本机构建后台
run-docker -bi --stop
run-docker -bi --stop 20261022_171618
```

产物 / 日志：

- 镜像包：`docker_images/<cn>_<TS>.tar.gz`
- 构建日志：`yz_pipeline_logs/build_*.log`

### 复用他人容器

```bash
run-docker my_container /path/to/server.sh /path/to/client.py 0 -re
run-docker -c my_container /path/to/server.sh -re
```

## 批量清洗脚本 → 生成 yaml（`-sc`）

```bash
run-docker -sc <Server脚本目录...> \
  -id <镜像ID> -bn <容器名前缀> -type <0|1|3> [-client <Client脚本...>]

# 多机：指定节点 IP（生成 nodemulti yaml）
run-docker -sc <目录> -id <镜像ID> -bn <名称> -type 0 -client <路径> \
  -nn 10.16.1.28,10.16.1.25

# 多目录 + 多 Client / 多组 -nn（空格分隔，按序与目录对齐；Client 不足则复用第一个）
run-docker -sc <目录A> <目录B> -id <镜像ID> -bn <名称> -type 0 \
  -client /path/client_a.py /path/client_b.py \
  -nn 10.16.1.28,10.16.1.25 10.16.1.30,10.16.1.31

# 按卡数均分 YAML / 高优模型排序（可按唯一 Server 目录重复指定）
run-docker -sc <目录> -id <镜像> -bn <名称> -type 0 -client <路径> \
  -cut 8 -prio model_a,model_b

# Agent YAML（ag_*.yaml = single；-ag -nn → multi / ag_nm_*.yaml）
run-docker -sc <目录> -ag -id <Server镜像> [-cid <Client镜像>] \
  -bn <名称> -type 0 -client <路径>
run-docker -sc <目录> -ag -nn 10.16.1.28,10.16.1.25 \
  -id <Server镜像> -cid <Client镜像> -bn <名称> -type 0 -client <路径>

# 生成 YAML 写入 defaults.hygon_perf: true（本批等效 -hp）
run-docker -sc <目录> -id <镜像> -bn <名称> -type 0 -client <路径> -hp

# 从 json 生成（paths/client 可为数组）
run-docker -sc -js <config.json>
```

可选：`-ex 模型1,模型2` 排除；`-on/--only 模型1,模型2` 仅保留（可按目录重复指定；大小写不敏感）。  
`-cut N`：按卡数均分 YAML；`-prio NAMES`：高优模型整块靠前（只排序不筛选）。  
同目录且相同 IP 组合会合并为一份 nm yaml；单组 `-nn` 仍兼容。  
生成的 yaml 在 Server 脚本目录内。

## 其他常用命令

```bash
# 删除容器
run-docker rm <容器名> ...
run-docker rm -ps <关键词>
run-docker rm -ps <关键词> -ssh 10.16.1.28,10.16.1.25   # 本机 + 内网节点依次清理
run-docker rm -da <天数>          # 删除 N 天前 exited 容器（天数 > 1）
run-docker rm -fo -da <天数>      # 强制，慎用

# 重启 / 停止 / 杀进程兜底（支持逗号分隔多个）
run-docker restart <容器名>
run-docker restart c1,c2,c3
run-docker stop <容器名>
run-docker stop c1,c2
run-docker kill <容器名>          # 容器内扫荡进程（含 PID1；docker.orig 不可用时的兜底）

# 停止任务
run-docker --stop
run-docker --stop 20260511_130549
run-docker -m --stop [时间戳] -as     # 多模型：停该调度下全部子任务
run-docker -nm --stop                 # 多机任务请用 -nm --stop
run-docker -ag --stop [-as] [时间戳]  # Agent：-as 按句柄 Client→Server 清理
run-docker -pp --stop [log]           # 仅停编排后台，不杀业务容器
run-docker -bi --stop [时间戳]        # 停镜像构建后台

# 软停子任务 Client / Server（监控继续；TS 为 tmp_runtime 子任务时间戳或 -nm/-ag soft_stop_key）
run-docker --stop-role c 20260930_221500
run-docker --stop-role s 20260930_221500
run-docker --stop-role cs 20260930_221500   # 双侧：先 c 后 s

# 版本
run-docker --version
run-docker --versiondev

# DCU 检查
run-docker --dcu -n 192.168.1.100
run-docker --dcu -N "10.16.1.28,10.16.1.25"
```

## 配置说明（`user_container.yaml`）

安装后主配置：`~/.config/auto_docker/user_container.yaml`

常见项：

| 项 | 说明 |
|------|------|
| `perform_opencompass_path` | 精度测试默认 OpenCompass **容器内**目录 |
| `work_subdirectory` / `work_directory_prefix` | 容器内日志工作目录；若配置了 `volume_workdir`，前缀须落在其容器挂载点下 |
| `build_image_install` | `-bi` 默认安装脚本（节点完整路径；`-is` 可覆盖） |
| `allowed_directories` | 允许的宿主机目录挂载范围 |
| `volume_workdir` | 独立可读写工作卷（日志/产物；勿加 `:ro`；不参与模型路径转换；留空不挂载） |
| `volume_llm_models` / `volume_models_path` ~ `path5` | 模型等自定义映射（留空不挂载） |
| `feishu.user_open_id` | 个人飞书报告 |
| `feishu.faban_chat_id` | 发版群 chat_id（可配置多个；配合 `-fb [N]`） |

命令行参数优先级高于配置文件。容器名 / Server / Client / 测试类型必须命令行传入。

队列任务 yaml / job yaml 模板字段见发布仓库示例（`batch_more_template.yaml` / `nodemulti_template.yaml` / `agent_template.yaml` / `batch_pipeline_template.yaml`）：

- 单机多模型（队列任务 yaml）：`model_path`、`server_script_path`、`client_script_path`、`tp_sum`、`test_type`；可选 `defaults.hygon_perf`
- 多机（队列任务 yaml）：另需 `nodes: [主IP, workIP, ...]`，且 `tp_sum = total_cards × 节点数`
- Agent（队列任务 yaml）：`server_image_id` / `client_image_id`，`task_kind=single|multi`
- 编排（**job yaml**，与上三类队列任务 yaml 不同）：`targets[{ips,yaml}]` + `b_log_or_s_imageid`；`yaml` 再指向各队列任务 yaml，可混跑 `-m`/`-nm`/`-ag`
- SGLang 主脚本须含 `--nnodes` / `--node-rank` / `--dist-init-addr`
- vLLM 主脚本须含 `--nnodes` / `--node-rank` / `--master-addr`

## 日志与排障

```bash
# 最新任务日志
tail -f -n 100 tmp_runtime/*/script.log

# 指定时间戳
tail -f -n 100 tmp_runtime/20260515_183749/script.log

# 多模型 / 多机 / Agent / 编排 / 构建日志
ls -lt tmp_batch_more/ tmp_batch_nodemulti/ tmp_batch_agent/
ls -lt yx_deploy_logs/ yz_pipeline_logs/ docker_images/

ls -lt tmp_runtime/
```

> 调度器会话目录现为 `<时间戳_节点名>/`（如 `tmp_batch_more/20261020_183000_nodeA/`）；`--stop` 仍只传时间戳即可。

容器内完整日志示例：

```text
{work_directory_prefix}/{work_subdirectory}/<容器名>/<时间戳>/auto_run.log
{work_directory_prefix}/{work_subdirectory}/<容器名>/<时间戳>/server.log
{work_directory_prefix}/{work_subdirectory}/<容器名>/<时间戳>/client.log
```

## 近期能力摘要

| 时间 | 要点 |
|------|------|
| 07-02 | 自定义映射 path1–5；`-fb` 群推送；`-link` 写飞书表；EvalScope 报告；`-sc` 洗脚本生成 yaml |
| 07-09 | `-re` 复用他人容器；`-m --stop -as` 批量停子任务 |
| 07-22 | SGLang / vLLM 多机模型支持；`-sc -nn` 生成多机 yaml |
| 07-27 | 动态 toponum / NUMA；`-fb N` 多群 |
| 07-31 | `rm -da/-fo` 按日期删 exited 容器；`-m/-nm` 磁盘预检；`-sc` 映射/同模型提醒增强 |
| 08-01 | `-sc` 支持多 Client 槽位与多组 `-nn`（按目录对齐；同目录同 IP 合并一份 nm yaml） |
| 08-04 | `-m/-nm` 增加 `-fr/--forcerestart`（配合 `-st` 到点强制释放整节点 GPU）；`-nm` 选 IB `master_ip` 策略优化；增加 Server 日志报错检测 `hit an exception` / `AssertionError` |
| 08-05 | 安装时遇只读 shell rc（如公共目录锁死的 `~/.bash_profile`）改为告警并跳过，继续写入可写 rc；`--dcu` / 多机 hy-smi：优先 `/opt/hyhal/bin`，否则走 PATH |
| 08-06 | `-sc`：自动分离大于 8 卡模型，生成 nm yaml（未传 `-nn` 时） |
| 08-10 | 多机脚本 patch 更稳（脚本补丁细节优化） |
| 08-11 | `-m`：NVIDIA 空闲卡检测，极小残差显存仍算空闲；`-sc`：yaml 中路径相对执行时的 cwd |
| 08-12 | `-m/-nm`：坏子任务跳过，不再整批中止 |
| 08-14 | `-m/-nm`：path 类 CLI 参数自动转容器路径 |
| 08-15 | `-nm`：多机 SSH 探测重试；汇总列出成败模型名 |
| 08-18 | Server 单边假死卡顿上限 30min；超时发 stall 报告（飞书） |
| 08-19 | `-link`：冻结环境 idna 软跳过；飞书写入在报告生成后重试 |
| 08-24 | `-m/-nm` 调度进度统计；`-nm` serve.log 写入节点头 |
| 08-29 | `-on/-ex` 大小写不敏感 |
| 09-03 | Step2.6 curl 注入提速 |
| 09-04 | Phase1 启动上限按模型 M/N × 节点数动态计算；新增 `restart` / `stop`（`docker.orig`）；锁定 profile 下新登录 shell 仍可找到 `run-docker` |
| 09-07 | 只读 Server CLI 快照（EP/CP/SP/kvcache）；auto_run 后落盘、未设置并行维省略 |
| 09-08 | `-nm` 写入 Server CLI / serve 方式 |
| 09-09 | `-sc` 按 tp/8 用 `0.0.0.0` 填充 nm nodes；报告模块拆分，EvalScope perf 解析增强 |
| 09-10 | `-link yxpf`：EvalScope/bench 性能 CSV 写飞书云盘；按系列表 + 模型 sheet |
| 09-11 | 性能报告文件名与 Client 报告区分 |
| 09-14 | 新增 `-hp/--hygon-perf`（优先 `hygon_das_pref_test` CSV → hygon_perf + yxpf） |
| 09-15 | yxpf 宽表批量写入与动态列锁 |
| 09-22 | 新增 Agent 双容器模式（`-ag`）与 `-sc -ag` 生成 YAML |
| 09-23 | `-ag`：worker 子进程、单机并行、日志分流、Client 挂 GPU/DCU |
| 09-24 | `-ag` 支持 `task_kind=multi`（对齐 `-nm` Phase2）；磁盘预检 / 终态汇总 / 退出码对齐 `-m` |
| 10-08 | 新增 `--stop-role`、`run-docker kill`；`type=0` Client 支持 `.py`；`-nm` 任务日志与调度日志隔离 |
| 10-20 | 优化：探测重试 / Server 启动二次确认；`-nm` IB/网卡白名单；Phase2 风暴完成识别；`-fr` 与 `-st` 解耦；`test_type=3` 与 `-link eval/perf/both`；会话目录 `时间戳_节点名` |
| 10-28 | `-sc`：清洗日志落盘与终局汇总、按路径单块留痕；`-cut` / `-prio` 高优排序与按卡均分；Phase2 client 秒退快速清理 |
| 10-31 | 新增 `volume_workdir`；`-m` 无 TP 缺省 T=1、只改已有 DP/EP/PP；跨节点 `tmp_runtime` 防串台 |
| 11-06 | 优化：`-m/-nm/-ag` 终态四档与进度飞书统计；vLLM master-port / DP address·rpc-port；磁盘预检红区阈；`-nm` work 脚本精简 |
| 11-12 | 优化：`-link perf` 增加 `run_meta`；性能表 A 列「时间\|镜像\|节点」并写入清洗后 Server 脚本 |
| 11-16 | 新增 `-pp/--pipeline` 多 target 编排、`-bi/--build-image` 镜像构建导出；`rm -ps -ssh` 多节点清理；`-pp` 内网免密任意节点发起 |

## 注意事项

- 复用容器会**重启**该容器
- `-c` / `-m` / `-nm` / `-ag` / `-pp` / `-bi` 须为第一个参数且互斥
- `-fr` 仅用于 `-m` / `-nm` / `-ag`，与 `-st` 互相独立（单独使用则立即清卡；同用则到点后再清卡）
- `-hp` 不适用于单测模型（`-c`）
- 停止 Agent 请用 `run-docker -ag --stop`（不要写成 `run-docker --stop -ag`）
- 停止编排请用 `run-docker -pp --stop`（只停编排后台，不杀业务容器）
- `test_type=1` 时 Client 须为 `.py`；`test_type=3` 时 Client 须同时含评分与性能信号
- `-link both` 仅支持 `test_type=3`；旧关键字 `yxff` / `yxpf` 已废弃，请改用 `eval` / `perf`
- 脚本需落在 `allowed_directories` 挂载树内
- 写入飞书表格 / perf 云盘需给机器人**编辑权限**；评分表仅支持表格类链接
- 当前发布包仅支持 **Linux x86_64**
