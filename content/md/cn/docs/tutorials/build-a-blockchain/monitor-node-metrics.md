---
title: 监控节点指标
description: 使用可观察性工具捕获和查看有关 Substrate 节点的信息。
keywords:
  - 可观察性
  - 指标
  - 节点操作
---

Substrate 公开了有关网络操作的指标。
例如，您可以收集有关节点连接了多少个对等节点、节点使用了多少内存以及生成的区块数量的信息。
要捕获和可视化 Substrate 节点公开的指标，您可以配置和使用诸如 [Prometheus](https://prometheus.io/) 和 [Grafana](https://grafana.com/) 之类的工具。
本教程演示了如何使用 Prometheus 获取数据样本，以及如何使用 Grafana 创建图表和仪表板以使用数据样本可视化节点指标。

在较高的层次上，Substrate 公开了遥测数据，这些数据可以被 Prometheus 端点使用，并以 Grafana 仪表板或图表中的可视化信息的形式呈现。

下图简要概述了如何配置 Substrate、Prometheus 和 Grafana 之间的交互以显示有关节点操作的信息。

![使用 Prometheus 和 Grafana 可视化节点指标](/media/images/docs/tutorials/monitor-node-metrics/node-metrics.png)

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已完成至少一些之前的教程，包括 [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/) 和 [模拟网络](/tutorials/build-a-blockchain/simulate-network/)。

## 教程目标

通过完成本教程，您将实现以下目标：

- 安装 Prometheus 和 Grafana。
- 配置 Prometheus 以捕获 Substrate 节点的时序数据。
- 配置 Grafana 以使用 Prometheus 端点可视化收集的节点指标。

## 安装 Prometheus 和 Grafana

出于测试和演示目的，您应该下载 Prometheus 和 Grafana 的已编译 `bin` 程序，而不是自己构建工具或使用 Docker 镜像。
使用以下链接为您的架构下载相应的二进制文件。
本教程假设您在工作目录中使用已编译的二进制文件。

要安装本教程的工具：

1. 在您的计算机上打开浏览器。

1. 从 [prometheus 下载](https://prometheus.io/download/) 下载适用于 Prometheus 的相应预编译二进制文件。

2. 在您的计算机上打开终端 shell 并导航到您的“下载”文件夹，然后运行相应的命令以解压缩您下载的文件内容。

   例如，在 macOS 上，您可以运行类似于以下内容的命令：

   ```text
   gunzip prometheus-2.38.0.darwin-amd64.tar.gz && tar -xvf prometheus-2.38.0.darwin-amd64.tar
   ```

3. 导航到 [Grafana OSS 下载](https://grafana.com/grafana/download?edition=oss)。

4. 选择适合您架构的相应预编译二进制文件。

5. 在您的计算机上打开终端 shell 并运行相应的命令以安装到您的架构上。

   例如，在安装了 Homebrew 的 macOS 上，您可以运行以下命令：

   ```text
   brew update
   brew install grafana
   ```

## 启动 Substrate 节点

Substrate 公开了可在端口 `9615` 上使用的 [Prometheus 公开格式](https://prometheus.io/docs/concepts/data_model/) 端点，该端点提供指标。
您可以使用 `--prometheus-port` 命令行选项更改端口，并使用 `--prometheus-external` 命令行选项启用它以通过本地主机以外的接口进行访问。
为简单起见，本教程假设 Substrate 节点、Prometheus 实例和 Grafana 服务都在本地使用默认端口运行。

1. 在您的计算机上打开终端 shell。

2. 必要时，通过运行以下命令更改到节点模板目录的根目录：

   ```bash
   cd substrate-solo-template-node
   ```

3. 通过运行以下命令以开发模式启动节点模板：

   ```bash
   ./target/release/solo-template-node --dev
   ```

## 配置 Prometheus 端点

当您解压缩 Prometheus 下载时创建的目录包含一个 `prometheus.yml` 配置文件。
您可以修改此文件（或创建自定义配置文件）以配置 Prometheus 从默认 Prometheus 端口端点（端口 `9615`）或您使用 `--prometheus-port <port-number>` 命令行选项指定的端口提取数据。

要将 Substrate 公开的端点添加到 Prometheus 目标列表：

1. 在您的计算机上打开一个新的终端 shell。

1. 更改到 Prometheus 工作目录的根目录。

1. 在文本编辑器中打开 `prometheus.yml` 配置文件。

1. 将 `substrate_node` 添加为 `scrape_config` 端点。

   例如，添加类似于以下内容的部分：

   ```yml
    # 一个刮取配置，其中包含一个要刮取的端点：
    scrape_configs:
      - job_name: "substrate_node"

        scrape_interval: 5s

        static_configs:
          - targets: ["localhost:9615"]
   ```

    这些设置会覆盖 `substrate_node` 作业的刮取目标的全局默认值。
    将 `scrape_interval` 设置为小于块时间的值非常重要，以确保您拥有每个块的数据点。
    例如，Polkadot 块时间为六秒，因此 `scrape_interval` 设置为五秒。

1. 使用修改后的 `prometheus.yml` 配置文件启动 Prometheus 实例。

   例如，如果您当前位于 Prometheus 工作目录中并使用默认配置文件名，则可以通过运行以下命令启动 Prometheus：

   ```bash
   ./prometheus --config.file prometheus.yml
   ```

   让此进程继续运行。

2. 打开一个新的终端 shell 并通过运行以下命令检查为 Substrate 节点检索到的指标：

   ```bash
   curl localhost:9615/metrics
   ```

   此命令返回类似于以下截断示例的输出：

   ```text
   # HELP substrate_block_height Block height info of the chain
   # TYPE substrate_block_height gauge
   substrate_block_height{status="best",chain="dev"} 16
   substrate_block_height{status="finalized",chain="dev"} 14
   substrate_block_height{status="sync_target",chain="dev"} 16
   # HELP substrate_block_verification_and_import_time Time taken to verify and import blocks
   # TYPE substrate_block_verification_and_import_time histogram
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.005"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.01"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.025"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.05"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.1"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.25"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="1"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="2.5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="10"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="+Inf"} 0
   substrate_block_verification_and_import_time_sum{chain="dev"} 0
   substrate_block_verification_and_import_time_count{chain="dev"} 0
   # HELP substrate_build_info A metric with a constant '1' value labeled by name, version
   # TYPE substrate_build_info gauge
   substrate_build_info{name="ruddy-afternoon-1788",version="4.0.0-dev-6a8b2b12371",chain="dev"} 1
   # HELP substrate_database_cache_bytes RocksDB cache size in bytes
   # TYPE substrate_database_cache_bytes gauge
   substrate_database_cache_bytes{chain="dev"} 0
   # HELP substrate_finality_grandpa_precommits_total Total number of GRANDPA precommits cast locally.
   # TYPE substrate_finality_grandpa_precommits_total counter
   substrate_finality_grandpa_precommits_total{chain="dev"} 76
   # HELP substrate_finality_grandpa_prevotes_total Total number of GRANDPA prevotes cast locally.
   # TYPE substrate_finality_grandpa_prevotes_total counter
   substrate_finality_grandpa_prevotes_total{chain="dev"} 77
   # HELP substrate_finality_grandpa_round Highest completed GRANDPA round.
   # TYPE substrate_finality_grandpa_round gauge
   substrate_finality_grandpa_round{chain="dev"} 76
   ...
   ```

   或者，您可以在浏览器中打开相同的端点以查看所有可用的指标数据。
   例如，如果您使用的是默认 Prometheus 端口，请在浏览器中打开 [`http://localhost:9615/metrics`](http://localhost:9615/metrics)。<!-- markdown-link-check-disable-line -->

## 配置 Grafana 数据源

在您为您的架构运行相应的命令以安装 Grafana 后，您可以启动本地计算机上的服务以开始使用它。
用于启动服务的命令取决于您的本地系统架构和包管理器。
例如，如果您使用的是 macOS 和 Homebrew，则可以通过运行以下命令启动 Grafana：

```bash
brew services start grafana
```

有关在不同操作系统上启动 Grafana 的信息，请参阅相应的 [Grafana 文档](https://grafana.com/docs/grafana/v9.0/)。

启动 Grafana 后，您可以在浏览器中导航到它。

1. 打开浏览器并导航到 Grafana 使用的端口。

   默认情况下，Grafana 使用 http://localhost:3000，除非您已配置不同的主机或端口。<!-- markdown-link-check-disable-line -->

2. 使用默认的 `admin` 用户名和密码 `admin` 登录，然后点击**登录**。

3. 在欢迎页面上的**配置**菜单下，点击**数据源**。

1. 点击**Prometheus** 将 Prometheus 端点配置为 Substrate 节点指标的数据源。

   在 Substrate 节点和 Prometheus 实例都运行的情况下，配置 Grafana 以在其默认端口 `http://localhost:9090` 或您配置 Grafana 使用的端口（如果您自定义了端口信息）上查找 Prometheus。

   您不应指定您在 `prometheus.yml` 文件中设置的 Prometheus 端口。
   该端口是您的节点发布其数据的端口。

2. 点击**保存并测试**以确保您已正确设置数据源。

   如果数据源有效，则您可以配置仪表板以显示节点指标。

## 导入模板仪表板

如果您想要一个基本的仪表板来开始，您可以导入一个 [Substrate 仪表板模板](/assets/tutorials/monitor-node/substrate-solo-template-node-metrics.json) 以获取有关您的节点的基本信息。

要导入仪表板模板：

1. 在 Grafana 欢迎页面上，点击**仪表板**。

1. 在左侧导航中，点击**仪表板**并选择**浏览**。

2. 对于搜索选项，点击新建并选择**导入**。

3. 复制 [Substrate 仪表板模板](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/monitor-node/substrate-solo-template-node-metrics.json) 并将其粘贴到**通过面板 json 导入**文本框中。

4. 点击**加载**。

5. 必要时查看和修改仪表板的名称、文件夹和唯一标识符。

6. 选择**Prometheus (default)**，然后点击**导入**。

   ![Substrate 仪表板模板](/media/images/docs/tutorials/monitor-node-metrics/grafana-template-dashboard.png)

   [Substrate 仪表板模板](https://grafana.com/grafana/dashboards/13759/) 可与任何基于 Substrate 的链一起使用，也可以从 Grafana Labs 仪表板库下载。

   如果您想创建自己的仪表板，请参阅 [Grafana 的 Prometheus 文档](https://prometheus.io/docs/visualization/grafana/)。

   如果您创建自定义仪表板，请考虑将其上传到 [Grafana 仪表板](https://grafana.com/grafana/dashboards)。
   您可以通过在 [Awesome Substrate](https://github.com/substrate-developer-hub/awesome-substrate) 存储库中列出您的仪表板来让 Substrate 构建者社区知道您的仪表板的存在。

## 下一步

- [添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes)
- [监控您的节点](https://wiki.polkadot.network/docs/en/maintain-guides-how-to-monitor-your-node)
- [Substrate Prometheus Exporter](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/utils/prometheus)
- [Polkadot 网络仪表板](https://github.com/w3f/polkadot-dashboard)
