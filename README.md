# 8gkg Pearl Miner

主页：[8gkg.com](https://8gkg.com)。用于 NVIDIA 显卡的 Pearl（PRL / PearlHash V3）挖矿软件，支持 Windows、Linux 与 HiveOS。当前版本 **0.4.1**，默认开发者费用 **1%**，可调整或关闭。

## 文件与启动

| 平台 | 本目录中的发行文件 | 启动方式 |
| --- | --- | --- |
| Windows x64 | [8gkg_pearl-0.4.1-win64.zip](8gkg_pearl-0.4.1-win64.zip) | 解压后编辑 `start-pearl.bat` 中的 `WALLET`，保存并双击运行 |
| Linux x64 | [8gkg_pearl-0.4.1-linux-x64.tar.gz](8gkg_pearl-0.4.1-linux-x64.tar.gz) | 解压后编辑 `start-pearl.sh` 中的 `WALLET`，执行 `./start-pearl.sh` |
| HiveOS | [8gkg_pearl-0.4.1.tar.gz](8gkg_pearl-0.4.1.tar.gz) | 安装自定义矿工，名称填写 `8gkg_pearl` |

Windows 单文件程序也可直接从 [windows-0.4.1/8gkg_pearl.exe](windows-0.4.1/8gkg_pearl.exe) 使用。每个发行压缩包旁提供同名 `.sha256` 校验文件。

**正式挖矿前请修改收款钱包。** 无参数启动和随附脚本使用以下测试钱包，本版内置开发费钱包也暂用此地址；不修改时，矿工部分的收益同样归该地址：

```text
prl1pr884rm9jw4darrkhzrhpm3hppd3ss0g4m2ysxapw2wqwtexuq5nsgt8g6z
```

脚本中的 `WORKER` 为矿工名称，默认 `rig01`；直接运行程序时默认名称为 `rig-gpu`。不指定显卡时使用所有受支持的 NVIDIA GPU。程序持续运行，按 **Ctrl+C** 停止。

## Kryptex 连接说明

本版默认配置为 `tls://prl.kryptex.network:8048`，使用 TLS 验证服务器证书。以下命令中的 `YOUR_PEARL_ADDRESS` 请替换为自己的 PRL 钱包，`rig01` 替换为矿工名称。

Windows PowerShell：

```powershell
.\8gkg_pearl.exe --pool tls://prl.kryptex.network:8048 --wallet YOUR_PEARL_ADDRESS --worker rig01
```

Linux：

```bash
tar -xzf 8gkg_pearl-0.4.1-linux-x64.tar.gz
cd 8gkg_pearl
./8gkg_pearl --pool tls://prl.kryptex.network:8048 --wallet YOUR_PEARL_ADDRESS --worker rig01
```

只使用指定显卡，在命令后添加 `--gpu-id 0,1` 或 `-d 0,1`。使用 `--list-devices` 查看当前系统中的设备编号。

多个矿池地址用分号分隔，并用引号括住。以下为 TLS 与 TCP 地址示例；TCP 连接不加密：

```powershell
.\8gkg_pearl.exe --pool "tls://prl.kryptex.network:8048;tcp://prl.kryptex.network:7048" --wallet YOUR_PEARL_ADDRESS --worker rig01
```

只检查连接、授权与任务接收，在相同连接参数后添加 `--check-pool --seconds 10`。此模式不执行 GPU 挖矿或开发费任务。

本版在矿池确认 v2 能力后使用 gzip 压缩证明；未确认时使用原始格式。已验证 gzip 编码、完整证明校验和 TLS 授权。矿池网页上的 gzip 兼容性提示是否消失尚未确认，不能只凭本地自检判断矿池页面状态。

## HiveOS 安装与飞行表

使用 **`8gkg_pearl-0.4.1.tar.gz`**，保留这个文件名。将安装包上传到矿机后，在矿机终端执行：

```bash
mkdir -p /hive/miners/custom
tar -xzf 8gkg_pearl-0.4.1.tar.gz -C /hive/miners/custom
```

飞行表选择 **Custom**，填写：

| 字段 | 内容 |
| --- | --- |
| Miner name | `8gkg_pearl` |
| Hash algorithm | `pearlhash` |
| Wallet and worker template / Pool URL / Pass | 可留空；本矿工不读取这三个字段 |
| Extra config arguments（附加配置参数） | 填写下方完整启动参数，钱包、矿池、矿工名都写在这里 |

若使用自动下载安装，将该安装包放到矿机可访问的 HTTP/HTTPS 地址，在 **Installation URL** 中填写真实下载链接。手动安装后不需要该链接；本机文件路径不能作为下载地址。

与 QuantusMiner 相同，**全部矿工参数从“附加配置参数”读取**，不从飞行表的钱包模板、矿池、密码或算法字段生成。将 `YOUR_PEARL_ADDRESS` 替换为自己的 PRL 钱包，然后把下面一行完整粘贴到附加配置：

```text
--pool tls://prl.kryptex.network:8048 --wallet YOUR_PEARL_ADDRESS --worker rig01 --password x --devfee 1
```

需要选卡或关闭开发费，在同一行末尾添加 `--gpu-id 0,1 --devfee 0`。即使飞行表其他字段有不同的钱包或矿池，也不会覆盖附加参数。算法默认 `pearlhash`；需要显式设置时也写在附加参数中。

附加配置同样支持 JSON 对象：

```json
{"pool":"tls://prl.kryptex.network:8048","wallet":"YOUR_PEARL_ADDRESS","worker":"rig01","password":"x","devfee":1,"gpu-id":"0,1","api-port":21373}
```

未指定的参数使用程序默认值；留空全部附加配置会使用本版测试钱包，请在正式挖矿前填好自己的 `--wallet`。不要将可执行文件名或 shell 命令填入附加配置，只填写矿工参数。

HiveOS 默认启用本机统计 API；`--api-port`、`--log-file` 等设置以附加配置为准。安装包包含主程序、配置与统计脚本以及使用说明；配置脚本使用 HiveOS 自带的 bash、python3、jq、curl 和 flock。启动脚本随后切换为矿工进程。

## 显卡适配与自动调优

以下设备已使用 **0.4.1 发行程序**完成实机功能验证，测试日期为 2026-09-15：

| 显卡 | 架构 | 本版测试范围 |
| --- | --- | --- |
| RTX 5060 Ti | SM120 / Blackwell | Windows GPU 源码内核自检；本地 Ubuntu 20.04 WSL 双卡计算、API、HiveOS 脚本和 gzip 证明测试 |
| RTX 4070 SUPER | SM89 / Ada | Windows GPU 源码内核自检；本地 Ubuntu 20.04 WSL 双卡计算、API、HiveOS 脚本和 gzip 证明测试 |
| RTX 3080 | SM86 / Ampere | HiveOS / Ubuntu 22.04 双卡自动调优、挖矿计算、API 和 HiveOS 脚本测试 |
| NVIDIA CMP 170HX | SM80 / Ampere | HiveOS / Ubuntu 22.04 双卡自动调优、挖矿计算、API 和 HiveOS 脚本测试 |

本机测试驱动为 595.71，HiveOS 测试机为 610.57.04。3080 和 CMP 170HX 本版未在 Windows 重测。**RTX 20 系的 SM75 通用内核已编译，尚未实机验收**；其他型号不能仅凭相同架构就视为已测试。

每次启动按显卡独立校验和测速，默认调优预算 **15 秒**，多卡并行进行。调优过程与结果不显示，也不保存或读取调优缓存。完成校验的候选中优先选择算力最高者；算力在最快结果的 1% 范围内时，优先选择实测功耗更低者。功耗数据不足时保留最快结果。

未测试过的型号先按架构、核心数、显存、带宽等信息安排候选，再以本机校验与测速决定。温度、当前功率限制及其他 GPU 任务会影响短时测量，不能据此承诺长期算力或节能幅度。程序不会自动修改显卡功率上限、频率或风扇。

使用 `--tune-seconds 8` 可缩短启动比较时间，但可能无法测完全部候选。`--no-autotune` 跳过比较，默认使用通用内核；也可以搭配 `--kernel` 固定实现。

## 开发者费用

默认 **1%** 的 GPU 搜索时间用于开发者钱包。矿工与开发费使用独立矿池会话，开发费计算期间矿工连接仍保持、接收任务和保活。启动信息显示费用比例，后台开发费连接、重试和提交过程保持静默。

费用按每张 GPU 的实际搜索时间累计。调优、空闲、网络等待和 CPU 证明校验不计费；按完整批次切换会造成短时间比例偏差。开发费连接不可用时继续矿工任务，不积累长时间补扣。

控制台 `Speed`、`Average`、API 与 HiveOS 算力包含矿工和开发费的总计算量；A/R/P 与矿池收款地址的份额统计属于矿工。因此本机总速度与钱包在矿池端的有效算力并不等同。

关闭开发费，在启动命令或 HiveOS 附加配置末尾添加：

```text
--devfee 0
```

关闭后不建立开发费连接，也不执行开发费计算。可用 `--devfee 0.5`、`--devfee 2` 等设置比例，范围 0–100。开发费与矿池自身收费分开。

本版开发费收款地址暂为前述测试钱包。`--dev-wallet ADDRESS` 可覆盖该地址；修改矿工的 `--wallet` 不会自动修改开发费收款地址。

## 运行统计与 API

启动时显示一次版本、钱包、矿工名、矿池、驱动和选中显卡。挖矿开始后 **第 10 秒**首次显示统计，此后默认 **每 30 秒**更新。`ALL` 为全部所选显卡的汇总，其上方有独立横线。

| 列名 | 含义 |
| --- | --- |
| Speed (30s) / Average | 最近约 30 秒的速度 / 本次挖矿平均速度 |
| A/R/P | 矿工已接受 / 已拒绝 / 待确认份额 |
| Err | 本地证明校验错误数 |
| Core/Mem MHz | 实时核心主频 / 显存频率 |
| Core/Mem C | 核心温度 / 显存温度，摄氏度 |
| Fan % / Power W | 风扇比例 / 显卡功耗 |
| TH/s/W | 按实时速度与功耗计算的能效 |

传感器读取不到时显示 `--`。控制台和 API 不显示 CPU、进程内存占用或调优结果。

普通 Windows/Linux 使用 `--api-enable` 开启 API；HiveOS 自动开启。默认只监听 **`127.0.0.1:21373`**，可用 `--api-port` 改端口。

Windows PowerShell：

```powershell
Invoke-RestMethod http://127.0.0.1:21373/summary
```

Linux / HiveOS：

```bash
curl --noproxy '*' http://127.0.0.1:21373/summary
```

`/`、`/api.json` 和 `/summary` 返回相同 JSON，未知路径返回 404。`algorithms[0].hashrate.now` 为总速度，单位 H/s；`gpu_devices` 包含每卡信息、份额计数、频率、温度与功耗。读取不到的传感器为 `null`。HiveOS 脚本将算力转换为 kH/s，并按 PCI 地址匹配显卡；API 不可用时清空旧统计。

## 常用参数

| 参数 | 说明 |
| --- | --- |
| `--wallet ADDRESS` / `--worker NAME` | 收款钱包 / 矿工名称 |
| `--pool URL` / `--password VALUE` | 矿池地址 / 密码，默认密码 `x` |
| `--gpu-id 0,1` 或 `-d 0,1` | 指定设备；默认使用全部受支持显卡 |
| `--tune-seconds N` | 每次启动调优预算，1–3600 秒，默认 15 |
| `--no-autotune` / `--kernel ID` | 跳过调优 / 限定内核 |
| `--devfee N` / `--dev-wallet ADDRESS` | 开发费比例 / 开发费钱包 |
| `--seconds N` | 挖矿时长；默认 0 为持续运行，启动调优另计 |
| `--print-interval N` | 首次 10 秒之后的统计间隔，默认 30 秒 |
| `--api-enable` / `--api-port N` | 开启本机 API / 指定端口 |
| `--log-file FILE` / `--debug` | 保存滚动日志 / 增加诊断信息，仍隐藏调优结果 |
| `--config FILE` | 读取 JSON 配置，命令行参数优先 |
| `--watchdog N` / `--max-restarts N` | 工作线程超时秒数 / 10 分钟内可恢复服务故障的重试上限，默认 60 / 10 |
| `--cpu-limit N` / `--memory-limit N` | CPU 协作限速百分比 / 进程内存阈值 MiB，默认 100 / 8192 |

可指定的内核：`pearl-source-v1`、`pearl-sass-v1`、`pearl-forge-sass-v1`、`pearl-peak-sass-v1`、`pearl-krig-sass-v1`；`pearl-wide-v1` 仅用于 SM120。一般保留自动选择即可。

查询和自检：

```powershell
.\8gkg_pearl.exe --list-devices
.\8gkg_pearl.exe --self-test -d 0
.\8gkg_pearl.exe --benchmark -d 0 --seconds 10
.\8gkg_pearl.exe --help
```

Linux 将命令中的 `.\8gkg_pearl.exe` 换成 `./8gkg_pearl`。`--cpu-test` 检查 CPU 共识与 gzip；`--licenses` 显示内置组件许可。`--tune-only` 仅静默调优后退出，不连接矿池或保存结果。

## 运行环境与已知限制

程序采用单文件部署，无需安装 .NET 或 CUDA Toolkit；需要支持当前显卡及内置 CUDA 运行时的 NVIDIA 驱动。Windows 版使用 CUDA 13.1，Linux 版使用 CUDA 12.9。

Linux 主程序与 CUDA 原生库均在本地 **Ubuntu 20.04 / glibc 2.31** 编译，主程序要求的最高 GLIBC 版本为 **2.29**。已在本地 Ubuntu 20.04 WSL 和远端 HiveOS / Ubuntu 22.04 验证。主程序内置运行库，运行时会释放并校验供系统加载的依赖文件；这些文件不保存调优结果。

调优、多卡工作线程和两套矿池连接都在同一矿工进程内。池连接故障自动重连，可恢复的服务故障在进程内重试；GPU 驱动卡死或不可恢复故障会退出，需要 HiveOS 或系统管理程序重新启动。Windows 的终端或 conhost 为系统控制台组件。

本次验收包括两组 Linux 双卡 API/HiveOS 测试、本地双卡 201 个 gzip 证明的独立校验，以及 Windows/Linux 的宿主 API 与账户隔离测试。短时功能验证不能替代长期矿池收益统计，尚无大幅超过 SRBMiner 的性能结论。

---

文档结构参考 [8gkg QuantusMiner 使用说明](https://github.com/8gkgcom/QuantusMiner)。本文的命令、默认值和测试范围以 **8gkg Pearl Miner 0.4.1** 为准。
