<div align="center">

# TMS

### 面向矿场局域网的 TCMinerProxy 安全接入客户端

集中接入本地矿机，压缩公网流量与出口连接数，并提供配置管理和运行监控。

[客户端下载](https://www.tcminerproxy.com/zh/download/tms-secure-client) · [使用文档](https://www.tcminerproxy.com/zh/document/tcminerproxy) · [GitHub 仓库](https://github.com/MinerProxyPro/TMS) · [TCMinerProxy 服务端](https://github.com/MinerProxyPro/TCMinerProxy)

</div>

> **使用前提：** TMS 需要配合 TCMinerProxy 服务端使用。如果矿机较少、网络稳定且带宽充足，也可以让矿机直接连接 TCMinerProxy。

## 目录

- [项目简介](#overview)
- [功能特性](#features)
- [协议选择](#protocols)
- [下载与安装](#installation)
- [首次配置](#configuration)
- [运行维护](#operations)
- [常见问题](#faq)
- [文档与仓库说明](#resources)

<a id="overview"></a>

## 项目简介

TMS 通常部署在矿场局域网内。矿机连接本地 TMS，由 TMS 统一接入远程 TCMinerProxy 服务端，再由服务端连接上游矿池。

```mermaid
flowchart LR
    A["矿机 / ASIC"] -->|"局域网连接"| B["TMS 本地客户端"]
    B -->|"压缩、加密、连接复用"| C["TCMinerProxy 服务端"]
    C --> D["上游矿池"]
```

通过在本地集中管理连接，TMS 可以减少公网传输量和出口连接数，并支持配置同步、多远程地址负载均衡及运行状态监控。

<a id="features"></a>

## 功能特性

| 功能 | 说明 |
| --- | --- |
| 流量压缩 | 支持 TMS3、TMS3(Zstd) 和 TMS3(NB)，用于减少公网传输量 |
| 连接复用 | 将多台矿机的连接汇聚为更少的公网出口连接 |
| 加密传输 | 通过受保护的协议链路连接 TCMinerProxy 服务端 |
| 自动同步 | 使用服务端推送地址，同步端口配置 |
| 手动配置 | 自行设置本地监听端口、远程服务器、币种、协议和密码 |
| 负载均衡 | 为同一本地端口配置多个兼容的远程地址，分配连接 |
| 运行监控 | 查看入口与出口连接数，以及 CPU、内存、网络和端口状态 |
| 后台访问控制 | 设置登录凭据和自定义安全访问路径 |

<a id="protocols"></a>

## 协议选择

先确认服务端端口使用的协议，再选择对应的客户端配置。

| 协议 | 适用场景 | 说明 |
| --- | --- | --- |
| `TMS3(NB)` | BTC、LTC 大规模接入 | 仅支持 BTC 和 LTC。原项目说明中，特定测试条件下公网流量最高可减少 99.6% |
| `TMS3(Zstd)` | 兼顾压缩效果与 CPU 占用 | 连接逻辑与 TMS3 相同，通常对 CPU 更友好 |
| `TMS3` | 通用高压缩需求 | 提高压缩级别通常也会增加 CPU 负载 |
| `TMS2` | 连接历史 TMS2 服务端端口 | 用于兼容历史部署；TMS3 协议端口本身不向下兼容 TMS2 |

> **配置要求：** 客户端与服务端的币种、协议、端口密码和压缩设置必须匹配。实际压缩效果受币种、矿机协议、连接数量、压缩参数和硬件性能影响，上述数据不代表所有部署环境都能达到相同效果。

<a id="installation"></a>

## 下载与安装

### 支持平台

| 平台 | 架构或版本 | 安装方式 |
| --- | --- | --- |
| Linux | x86-64 | 安装脚本 |
| Linux / OpenWrt | ARMv7 | 安装脚本或手动下载 |
| Linux / OpenWrt | AArch64 | 安装脚本或手动下载 |
| Windows | x64 图形界面版 | 下载 GUI 主程序及运行库 |
| Windows | x64 命令行版 | 下载控制台程序 |

OpenWrt 的硬件和发行版差异较大。部署前请确认 CPU 架构，并先接入少量矿机验证兼容性。

### Linux 安装

**安装前准备**

- 在 TCMinerProxy 服务端创建并启用 TMS 协议端口。
- 为运行 TMS 的设备设置固定局域网 IP。
- 准备 `root` 权限。下方安装命令需要 Bash 和 curl。
- 阅读以下系统变更说明。

**安装脚本的系统变更**

脚本将程序安装到 `/root/tms`，根据系统配置服务或启动方式，并设置开机自启动。脚本还会调整文件句柄限制，且在部分 Linux 发行版中尝试关闭防火墙；当前脚本会跳过 OpenWrt 的防火墙自动关闭操作。

> **管理端口保护：** 执行前请审查 [安装脚本](https://github.com/MinerProxyPro/TMS/blob/main/install.sh)。安装后按实际网络配置防火墙，只允许可信局域网访问管理端口和矿机监听端口，避免将默认管理端口 `42703` 暴露到公网。

**运行安装命令**

```bash
sudo -i
bash <(curl -s -L https://raw.githubusercontent.com/MinerProxyPro/TMS/main/install.sh)
```

已使用 `root` 登录时，可直接执行第二行。按照菜单选择下载线路与 CPU 架构，完成安装。脚本会识别常见架构并提供推荐项，也支持手动选择 `x86-64`、`armv7-musleabihf` 或 `aarch64`。

<details>
<summary>无法访问 GitHub 时的备用安装入口</summary>

无法访问 GitHub 时，可使用以下备用线路：

```bash
sudo -i
bash <(curl -s -L -k http://cdn.tcminerproxy.com/install.sh)
```

该线路使用 HTTP。建议仅在可信网络中使用，并在执行前下载、检查脚本内容。

</details>

**打开管理界面**

安装完成后，在同一局域网内通过浏览器访问：

```text
http://TMS设备IP:42703
```

将 `TMS设备IP` 替换为运行 TMS 的设备地址，然后继续阅读[首次配置](#configuration)。

### Windows 下载

| 文件 | 用途 | 下载 |
| --- | --- | --- |
| `tms.exe` | 图形界面主程序，适合一般桌面使用 | [下载 GUI 主程序](https://raw.githubusercontent.com/MinerProxyPro/TMS/main/windows-gui/tms.exe) |
| `WebView2Loader.dll` | GUI 所需运行库，与主程序放在同一目录 | [下载运行库](https://raw.githubusercontent.com/MinerProxyPro/TMS/main/windows-gui/WebView2Loader.dll) |
| `tms.exe` | 命令行程序，适合控制台操作或自定义进程管理 | [下载命令行版](https://raw.githubusercontent.com/MinerProxyPro/TMS/main/windows-no-gui/tms.exe) |

图形界面版依赖 Microsoft Edge WebView2。若启动后出现白屏，请确认运行库与主程序位于同一目录，并安装 [Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/consumer/) 后重新启动。

<a id="configuration"></a>

## 首次配置

### 1. 准备服务端端口

在 TCMinerProxy 中创建所需的 `TMS2`、`TMS3`、`TMS3(Zstd)` 或 `TMS3(NB)` 协议端口，确认端口正常运行，并记录连接地址及相关配置。

### 2. 导入或手动添加配置

首次打开 TMS 时，可以选择以下任一方式：

| 方式 | 操作 |
| --- | --- |
| 自动同步 | 填写服务端推送地址，同步端口配置 |
| 手动添加 | 点击“跳过”，自行填写本地端口及远程服务器信息 |

手动填写远程地址时，使用 `地址:端口` 格式。同一本地端口可添加多个远程地址，但这些地址必须使用兼容的币种、协议和密码。

### 3. 核对两端设置

逐项确认客户端与服务端的以下配置匹配：

- 币种。
- TMS 协议类型。
- 端口密码。
- TMS3 压缩参数。

### 4. 接入矿机

本地监听端口创建成功后，将矿机连接地址设置为：

```text
stratum+tcp://TMS局域网IP:本地监听端口
```

将 `TMS局域网IP` 和 `本地监听端口` 替换为实际配置。先接入少量矿机，检查连接数、算力和拒绝率，确认正常后再逐批增加数量。

### 5. 完成上线检查

- [ ] TMS 设备已设置固定局域网 IP。
- [ ] 已设置后台登录用户名和密码。
- [ ] 已按需设置自定义安全访问路径。
- [ ] 防火墙已限制管理后台与本地监听端口的访问来源。
- [ ] 管理后台未直接暴露到公网。
- [ ] 已记录协议、压缩参数，并准备好故障时可切换的备用连接地址。

设置安全访问路径后，访问地址末尾需保留 `/`，例如：

```text
http://TMS设备IP:42703/private-path/
```

其中 `private-path` 为示例，请替换为实际设置的路径。

<a id="operations"></a>

## 运行维护

### 连接压缩与调优

TMS3 按本地端口将矿机连接汇聚为较少的公网出口连接。减少出口连接数可以提高连接压缩程度，也可能影响 CPU 负载、延迟和拒绝率。

以下为原项目提供的测试起点，需结合硬件和网络情况调整：

| 使用场景 | 初始测试配置 |
| --- | --- |
| TMS3 / TMS3(Zstd) | 每 100 台矿机约使用 1 条出口连接 |
| TMS3(NB)，仅运行 BTC / LTC | 每个端口先测试 3–6 条出口连接 |

不同币种、不同本地端口会分别建立出口连接。扩容或调整参数时，应同时观察 TMS CPU 占用、入口与出口连接数、服务端算力和上游拒绝率。

### Linux 管理菜单

再次运行安装命令即可打开管理菜单，进行以下操作：

- 安装或更新 TMS。
- 启动、停止、重启 TMS，以及查看运行状态。
- 查看运行日志和错误日志。
- 启用或关闭开机自启动。
- 卸载 TMS。

### 默认路径与端口

| 项目 | 默认值 |
| --- | --- |
| 安装目录 | `/root/tms` |
| 主程序 | `/root/tms/tms` |
| systemd 服务名称 | `tmservice` |
| OpenWrt 启动脚本 | `/etc/init.d/tms` |
| 本地配置文件 | 以当前安装版本实际生成的配置文件为准 |
| 运行日志 | `/root/tms/nohup.out` |
| 错误日志 | `/root/tms/err.log` |
| 管理端口 | `42703` |

服务管理方式取决于系统：当前安装脚本支持 systemd、OpenWrt 启动脚本，以及其他环境下的启动方式。关于本地配置文件和管理端口修改，见下方常见问题。

<a id="faq"></a>

## 常见问题

<details>
<summary><strong>Windows 图形界面启动后白屏，如何处理？</strong></summary>

先确认 `WebView2Loader.dll` 与 `tms.exe` 位于同一目录。如果问题仍然存在，安装 Microsoft Edge WebView2 Runtime 后重新启动 TMS。

</details>

<details>
<summary><strong>矿机能连接 TMS，但 TMS 无法连接服务端，如何排查？</strong></summary>

依次检查远程地址和端口、服务端运行状态、网络及防火墙规则，再核对两端的币种、协议、密码和压缩设置。客户端所选协议必须与服务端端口对应，TMS2、TMS3、TMS3(Zstd) 和 TMS3(NB) 端口不能混用。

</details>

<details>
<summary><strong>一个本地端口如何连接多台服务器？</strong></summary>

在“手动添加”或“编辑端口”中添加多个远程地址。确保这些地址使用兼容的币种、协议和密码后，TMS 会把连接分配到可用的远程地址。

</details>

<details>
<summary><strong>如何修改默认管理端口？</strong></summary>

在命令行版本的 TMS 安装目录中找到配置文件，将 `PORT` 修改为所需端口，保存后重启 TMS。修改后需同步更新浏览器访问地址和防火墙规则。

配置文件的位置与名称以当前版本实际安装结果为准。如未找到对应文件或字段，请先查阅 [使用文档](https://www.tcminerproxy.com/zh/document/tcminerproxy)。

</details>

<details>
<summary><strong>TMS 可以单独连接矿池吗？</strong></summary>

TMS 是 TCMinerProxy 的可选本地客户端，需要配合对应的服务端使用，不能替代 TCMinerProxy 服务端。

</details>

<a id="resources"></a>

## 文档与仓库说明

### 文档入口

| 主题 | 链接 |
| --- | --- |
| 产品与下载 | [客户端下载](https://www.tcminerproxy.com/zh/download/tms-secure-client) · [文档概览](https://www.tcminerproxy.com/zh/document/tcminerproxy) |
| 安装与接入 | [安装教程](https://www.tcminerproxy.com/zh/document/tms/installation) · [部署与配对](https://www.tcminerproxy.com/zh/document/tms/setup) |
| 配置与调优 | [端口映射](https://www.tcminerproxy.com/zh/document/tms/port-mapping) · [压缩设置](https://www.tcminerproxy.com/zh/document/tms/compression) |
| 监控与排障 | [监控与运维](https://www.tcminerproxy.com/zh/document/tms/monitoring) · [故障排查](https://www.tcminerproxy.com/zh/document/tms/troubleshooting) |
| 项目仓库 | [TMS 客户端](https://github.com/MinerProxyPro/TMS) · [TCMinerProxy 服务端](https://github.com/MinerProxyPro/TCMinerProxy) |

### 仓库用途

本仓库主要分发 TMS 安装脚本与各平台预编译程序。`main` 分支提供当前维护的发布文件，历史独立客户端不再作为当前版本的快速安装入口。

历史部署可在当前客户端中选择 TMS2 协议，对接兼容的服务端端口。TMS3 协议端口本身不向下兼容 TMS2。
