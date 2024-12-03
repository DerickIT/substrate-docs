---
title: 准备本地中继链
description: 解释如何配置本地中继链以设置用于平行链的本地测试网络。
keywords:
  - polkadot
  - cumulus
  - 中继链
  - 平行链
  - 平行线程
  - paraID
  - rococo
  - xcm
  - xcmp
  - 验证人
  - 测试网
  - 本地
---

本教程说明了如何配置本地中继链。
本地中继链是设置测试平行链节点可以连接到的本地测试环境所必需的。

## 开始之前

在开始之前，请考虑以下事项：

- 虽然这不是严格的先决条件，但建议您首先学习如何为可信验证器的私有网络生成链规范，如[添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes/)中所述。

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已完成 [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/) 教程，并且知道如何编译和运行 Substrate 节点。

- 您通常熟悉 Polkadot 的[架构和术语](https://wiki.polkadot.network/docs/learn-architecture)。

- 您知道平行链版本和依赖项与它们连接到的中继链版本紧密耦合。

  您的平行链必须与中继链升级保持同步才能继续成功运行。
  如果您在发布中继链的新版本时没有保持同步，则您的网络很可能会停止生成区块。

  教程通常使用最新的 Polkadot 分支来演示功能。
  如果教程没有按预期工作，您应该检查您的本地环境中是否有最新的 Polkadot 分支，并在需要时更新您的本地软件。

## 教程目标

通过完成本教程，您将实现以下目标：

- 设置您的平行链构建环境。
- 准备本地中继链规范。
- 在本地启动中继链。

## 构建中继链节点

Polkadot 是一个基于 Substrate 的中继链。
因此，本教程使用 Polkadot 存储库中的代码来准备本地中继链。

1. 克隆 Polkadot 存储库的最新发行分支以准备稳定的工作环境。

   发行分支往往最可靠，并使用命名约定 `release-v<n..n.n>`。
   例如，本教程中使用的发行分支是 `release-v1.0.0`。
   可能会有更新的发行版可用，并且在大多数情况下，您可以替换更新的发行分支，而不是使用 `release-v1.0.0` 分支，只要您对每个模块使用相同的分支即可。
   您可以在 GitHub 的[发行版](https://github.com/paritytech/polkadot/releases)选项卡中找到有关每个发行版的信息。

   ```bash
   git clone --branch release-v1.0.0 https://github.com/paritytech/polkadot-sdk.git
   ```

2. 通过运行以下命令更改到 `polkadot` 目录的根目录：

   ```bash
   cd polkadot
   ```

3. 通过运行以下命令构建中继链节点：

   ```bash
   cargo build --release
   ```

   编译节点可能需要 15 到 60 分钟才能完成。

1. 通过运行以下命令验证节点是否已正确构建：

   ```bash
   ./target/release/polkadot --help
   ```

   如果显示命令行帮助，则节点已准备好配置。

## 中继链规范

每个基于 Substrate 的链都需要一个[链规范](/build/chain-spec/)。
中继链网络的链规范提供了与链规范对其他网络提供的相同类型的配置设置。
链规范文件中的许多设置对于网络操作至关重要。
例如，链规范标识参与网络的对等节点、验证器的密钥、引导节点地址和其他信息。

### 链规范示例

对于本教程，本地中继链使用一个链规范示例文件，其中包含两个验证器中继链节点（Alice 和 Bob）作为权威机构。
由于中继链_必须_运行的验证器节点数量比连接的平行链验证人总数多至少一个，因此您只能将本教程中的链规范用于具有**单个平行链**的本地中继链网络。

如果您想连接两个平行链，每个平行链都有一个验证人，则您需要运行三个或更多中继链验证器节点。
通常，您需要修改链规范并硬编码其他验证器才能为两个或更多平行链设置本地测试网络。

### 纯文本和原始链规范文件

链规范示例有两种格式——纯文本格式的 JSON 文件和 SCALE 编码的原始格式的 JSON 文件。

- [纯文本中继链规范示例](/assets/tutorials/relay-chain-specs/plain-local-chainspec.json)
- [原始中继链规范示例](/assets/tutorials/relay-chain-specs/raw-local-chainspec.json)

您可以读取和编辑链规范文件的纯文本版本。
但是，在您可以使用链规范文件启动节点之前，必须将其转换为 SCALE 编码的原始格式。
有关将链规范转换为使用原始格式的信息，请参阅[自定义链规范](/reference/how-to-guides/basics/customize-a-chain-specification/)。

链规范示例仅对具有两个验证器节点的单个平行链有效。
如果您添加其他验证器、向中继链添加其他平行链，或者想要使用自定义帐户密钥而不是预定义帐户，则需要创建一个自定义链规范文件。

如果您同时在同一本地网络上的任何人一起完成本教程，则必须下载并修改纯文本中继链规范示例，以防止意外地与他们的节点对等。在纯文本链规范中找到以下行，并添加字符以使您的 protocolId 唯一：

```json
   "protocolId": "dot"
```

## 启动中继链节点

在您可以为平行链启动区块生成之前，您需要启动一个中继链以便它们连接。

要使用[原始链规范示例文件](/assets/tutorials/relay-chain-specs/raw-local-chainspec.json)启动验证器节点：

1. 将原始链规范文件下载到本地计算机上的工作目录。

   例如，将文件保存为 `/tmp` 目录中的 `raw-local-chainspec.json`。
   您需要在启动节点的命令中指定文件的路径。

2. 通过运行以下命令使用 `alice` 帐户启动第一个验证器：

   ```bash
   ./target/release/polkadot \
   --alice \
   --validator \
   --base-path /tmp/relay/alice \
   --chain /tmp/raw-local-chainspec.json \
   --port 30333 \
   --rpc-port 9944
   ```

   此命令使用 `/tmp/raw-local-chainspec.json` 作为链规范示例文件的位置。
   请确保 `--chain` 命令行指定您下载到本地工作目录中的原始链规范的路径。
   此命令还使用端口（`port`）和 WebSocket 端口（`ws-port`）的默认值。
   此处明确包含这些值是为了提醒您始终检查这些设置。
   节点启动后，同一台本地计算机上的其他节点将无法使用这些端口。

3. 查看节点启动时的日志消息，并记下本地节点标识。

   您需要指定此标识符才能使其他节点连接。

   ```bash
   🏷 Local node identity is: 12D3KooWGjsmVmZCM1jPtVNp6hRbbkGBK3LADYNniJAKJ19NUYiq
   ```

4. 打开一个新的终端并使用 `bob` 帐户启动第二个验证器。

   该命令类似于用于启动第一个节点的命令，但有一些重要的区别。

   ```bash
   ./target/release/polkadot \
   --bob \
   --validator \
   --base-path /tmp/relay/bob \
   --chain /tmp/raw-local-chainspec.json \
   --port 30334 \
   --rpc-port 9945
   ```

   请注意，此命令使用不同的基路径（`/tmp/relay/bob`）、验证器密钥（`--bob`）和端口（`30334` 和 `9945`）。

   由于两个验证器都在一台本地计算机上运行，因此无需指定 `--bootnodes` 命令行选项以及第一个节点的 IP 地址和对等节点标识符。
   如果您想连接在本地网络外部运行的节点或链规范文件中未标识的节点，则需要 `bootnodes` 选项。

   如果您没有看到中继链生成区块，请尝试禁用防火墙或使用 `alice` 节点的地址添加 `bootnodes` 命令行选项来启动节点。
   添加 `bootnodes` 选项如下所示（使用上面的节点标识）：`--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWGjsmVmZCM1jPtVNp6hRbbkGBK3LADYNniJAKJ19NUYiq`。

## 下一步

在本教程中，您学习了如何构建和启动本地中继链。
从这里开始，您可能希望学习如何将本地平行链连接到本地中继链，或者尝试使用可帮助您自动设置测试网络的工具。

- [连接本地平行链](/tutorials/build-a-parachain/connect-a-local-parachain/)
- [启动平行链测试网络](https://github.com/open-web3-stack/parachain-launch)
- [设置 `zombienet`](https://github.com/paritytech/zombienet) 是一个 CLI 工具，使您可以生成短暂的 Polkadot 和 Substrate 网络并在其上执行测试。
