---
title: 监控
description: 提供 Substrate 节点和波卡生态系统默认遥测的概述。
keywords:
---

监控网络操作和性能是维护健康、稳定和安全生态系统的重要组成部分。
默认情况下，Substrate 节点提供低级**遥测**组件，以自动收集和传输有关网络中每个节点操作的详细信息。
后端遥测信息会实时（发生时）流式传输到端点，以便您可以观察、监控和提醒系统行为。

作为开发人员或节点运营商，很少需要了解这些低级遥测组件如何提供有关计算机和网络操作的信息的实现细节。
在高级别，从每个公共节点收集的信息被发送到默认的遥测服务器，并在前端仪表板中进行聚合以进行显示。

下图提供了工作流程的简化概述。

![Substrate 节点的默认遥测](/media/images/docs/telemetry-overview.png)

## 查看遥测数据

[波卡遥测](https://telemetry.polkadot.io/) 仪表板提供了当前在线节点性能的实时视图。
您可以通过打开和关闭可用列列表中的可见列来选择要显示的信息。

要查看列列表并修改显示哪些列，请单击**设置**。

![单击“设置”以修改可见列](/media/images/docs/list-view-control.png)

您可以切换打开和关闭的列包括：

- **验证者** 以显示指示器以区分验证者节点和其他节点。
- **位置** 以显示节点的位置。
- **实现** 以显示节点上运行的软件版本。
- **网络 ID** 以显示节点的公共网络标识符。
- **对等节点数** 以显示连接到节点的对等节点数。
- **队列中的交易** 以显示在 Ready 队列中等待区块作者的交易数。
- **上传带宽** 以显示最近上传活动的图表（以 MB/秒为单位）。
- **下载带宽** 以显示最近下载活动的图表（以 MB/秒为单位）。
- **状态缓存大小** 以显示状态缓存大小的图表（以 MB 为单位）。
- **区块** 以显示节点的当前最佳区块号，以确保它与对等节点保持同步。
- **区块哈希** 以显示节点当前最佳区块号的区块哈希。
- **最终确定的区块** 以显示节点最近最终确定的区块号，以确保它与对等节点保持同步。
- **最终确定的区块哈希** 以显示节点最近最终确定的区块号的区块哈希。
- **区块时间** 以显示区块之间执行的时间。
- **区块传播时间** 以显示节点导入最近区块所需的时间。
- **上次区块时间** 以显示节点创建最近区块所需的时间。
- **节点正常运行时间** 以显示节点在线且未重新启动的天数。

### 固定节点信息

您可以通过单击仪表板列表中节点显示的行来保持一个或多个节点的信息在仪表板中可见。
例如，如果您希望在其他节点的信息滚动时保持两个节点的信息可见，则可以选择列表中的节点并将信息固定到位。

![保持所选节点的信息显示](/media/images/docs/pin-nodes.png)

### 在地图上显示节点

作为节点列表的替代方案，您可以在全球地图上显示节点。
该地图提供了节点位置、区块在实时生成和导入的位置以及地理位置上节点集中度更高的位置的可视化表示。
请注意，地图上仅显示具有已知位置的节点。

要查看在地图上表示的内容，请单击**地图**。

![在地球仪上显示节点](/media/images/docs/node-map-view.png)

在此视图中，您可以将鼠标悬停在地图上的任何节点位置以显示节点的可用信息子集，包括节点名称、位置、最佳区块号、最佳区块哈希和区块时间统计信息。

### 显示节点统计信息

除了单个节点的可用信息外，您还可以查看描述共享特定属性的节点数量和百分比的统计信息。
例如，您可以查看使用每个波卡版本的节点的百分比、使用 Linux 操作系统的节点的百分比以及当前部署的最常见发行版。
您还可以查看有关网络中节点的 CPU 和内存配置的统计信息。

要查看所有可用统计信息，请单击**统计信息**。

![查看跨节点的统计信息](/media/images/docs/node-statistics.png)

可用的统计信息包括有关网络中节点上运行的硬件和软件的详细信息，包括：

- 软件版本。
- 操作系统。
- CPU 架构和型号。
- 物理 CPU 内核数。
- 总内存。
- 节点是否为虚拟机。
- Linux 发行版和内核版本。
- CPU 和内存速度。
- 磁盘速度。

### 按链过滤

默认情况下，遥测仪表板显示有关波卡的信息，并提供到其他链子集的快速链接。
要显示其他链的信息，请单击**更多**并选择不同的链。

![选择其他链](/media/images/docs/more-chains.png)

## 自定义监控堆栈

默认遥测仪表板提供了对节点和链操作的可见性，而无需您托管或配置任何后端监控或前端服务。
但是，许多项目选择使用自己的后端服务器和前端仪表板来增强或替换默认遥测服务器。

通常，设置您自己的遥测服务器涉及为**链上事件**和单个**节点操作**建立监控和警报策略。

### 链上活动

您可以监控特定事件的链上活动，例如从特定地址提交的交易、当前验证者集的更改。
链上监控通常涉及连接到 RPC 节点以检查特定值、识别处理延迟或跟踪事件的时间。
在大多数情况下，您只需要两个 RPC 实例来处理所有主机的请求。
但是，建议您运行自己的 RPC 服务器来服务这些请求，以防公共 RPC 节点出现问题。
一些查询链上信息的应用程序示例是 [polkabot](https://gitlab.com/Polkabot/polkabot) 和 [polkadot-basic-notification](https://github.com/paritytech/polkadot-basic-notification)。

### 节点操作

您应该监控网络上运行的每个节点的基本操作信息，例如当前区块高度、对等连接数、CPU 使用率和可用空闲内存。
默认情况下，Substrate 在 `host:9615/metrics` 端点上公开许多有用的指标。
例如，如果 Substrate 在本地运行，您可以在 http://localhost:9615/metrics 端点上查看指标。<!-- markdown-link-check-disable-line -->
此端点使用简单的键值格式输出指标。
例如：

```text
polkadot_database_cache_bytes 0
```

但是，键也可以包含描述性标签。
例如：

```text
susbtrate_block_height{status="best"} 136
susbtrate_block_height{status="finalized"} 133
```

默认情况下，[指标](http://localhost:9615/metrics) 端点仅在本地网络接口上公开。<!-- markdown-link-check-disable-line -->
但是，您可以使用 `--prometheus-external` 命令行选项启动节点，将其公开在所有接口上。

### 配置监控工具

要设置监控和警报策略，您通常会配置一组工具来创建您自己的监控堆栈。
例如，默认的 `metrics` 端点不包含主机指标（例如 CPU、内存、带宽使用情况），因此您可以通过在每个主机上安装 Prometheus [node_exporter](https://github.com/prometheus/node_exporter) 来对其进行补充。
下图说明了一组通常用作监控堆栈的开源工具。

![监控堆栈层](/media/images/docs/monitoring-stack.png)

如该图所示，每层堆栈都有不同的工具可用。
在此示例中，以下工具配置用于监控链上活动和节点操作：

- Prometheus 是一个监控引擎，它以指定的间隔从指定的目标收集指标，并使用您定义的规则评估收集到的数据。其时间序列数据库可以保存大量数据，并且可以非常快速地访问这些数据。
- Grafana 是一个可观察性平台，允许您通过仪表板和图表查询、可视化和分析收集到的数据。
- Node exporter 是一个进程，它侦听端口并将特定于应用程序的指标报告给 Prometheus。
- Alertmanager 是一个工具，允许您根据指定的规则创建和路由警报。Alertmanager 允许您配置在出现问题时如何以及在何处发送警报。例如，您可以为警告警报发送即时消息，但为严重警报呼叫值班技术人员
- Loki 是一个可扩展的日志聚合系统，允许您在一个位置查看和搜索基础设施中所有组件的日志。

有关使用 Prometheus、Grafana 和 node exporter 设置节点监控的简单示例，请参阅 [监控节点指标](/tutorials/build-a-blockchain/monitor-node-metrics/)。
有关使用 Loki 的简化示例，请参阅 [远程日志记录](/deploy/deployment-options/#remote-logging)。

### 更改遥测服务器

在为节点配置后端监控规则后，您可以在启动节点时使用 `--telemetry-url` 命令行选项来指定要将遥测数据发送到的遥测服务器。
您可以多次传递此选项以指定多个遥测端点。
如果您指定 `--telemetry-url` 命令行选项，则还必须指定指标的详细程度，级别 0 表示最不详细，级别 9 表示最详细。

例如，要指定您自己的遥测服务器 URL 以及详细程度级别 5，您将运行类似于以下内容的命令：

```bash
./target/release/solo-template-node --dev \
  --telemetry-url "wss://192.168.48.1:9616 5" \
  --prometheus-port 9616 \
  --prometheus-external
```

有关遥测的后端组件或配置您自己的服务器的更多信息，请参阅 [substrate-telemetry](https://github.com/paritytech/substrate-telemetry) 或 [遥测 helm 图表](https://github.com/paritytech/helm-charts/tree/main/charts/substrate-telemetry) 以获取 Kubernetes 部署。

## 禁用遥测

默认情况下，所有全局链节点都启用了遥测。
您可以使用 `--no-telemetry` 命令行选项来阻止节点连接到 Substrate 遥测服务器。
例如，要阻止遥测数据发送到默认遥测服务器，您将运行类似于以下内容的命令：

```bash
./target/release/solo-template-node --chain myCustomChain \
  --no-telemetry
