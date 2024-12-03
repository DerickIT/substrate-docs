---
title: 模拟网络
description: 使用预定义帐户作为授权验证器来启动私有区块链网络。
keywords:
---

本教程简要介绍了如何使用私有**验证器**的**权威集**启动私有区块链网络。

Substrate 节点模板使用权威共识模型，该模型将区块生成限制为授权帐户的轮换列表。
授权帐户（**权威机构**）负责轮流创建区块。

在本教程中，您将看到权威共识模型在实践中的工作方式，方法是使用两个预定义帐户作为允许节点生成区块的权威机构。
在这个模拟网络中，两个节点使用不同的帐户和密钥启动，但在单个计算机上运行。

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已完成 [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/) 教程，并在本地安装了 Substrate 节点模板。

- 您通常熟悉软件开发和使用命令行界面。

- 您通常熟悉区块链和智能合约平台。

## 教程目标

通过完成本教程，您将实现以下目标：

- 使用预定义帐户启动区块链节点。

- 了解用于启动节点的关键命令行选项。

- 确定节点是否正在运行并生成区块。

- 将第二个节点连接到正在运行的网络。

- 验证对等计算机是否生成和完成区块。

## 启动第一个区块链节点

在生成密钥以启动您自己的私有 Substrate 网络之前，您可以使用名为 `local` 的预定义网络规范并使用预定义用户帐户来学习基本原理。

本教程通过使用名为 `alice` 和 `bob` 的预定义帐户在单个本地计算机上运行两个 Substrate 节点来模拟私有网络。

要启动区块链：

1. 在您的计算机上打开终端 shell。

1. 更改到您编译 Substrate 节点模板的根目录。

1. 通过运行以下命令清除旧的链数据：

   ```bash
   ./target/release/node-template purge-chain --base-path /tmp/alice --chain local
   ```

   该命令会提示您确认操作：

   ```bash
   Are you sure to remove "/tmp/alice/chains/local_testnet/db"? [y/N]:
   ```

1. 输入 `y` 以确认您要删除链数据。

   启动新网络时，应始终删除旧的链数据。

1. 通过运行以下命令使用 `alice` 帐户启动本地区块链节点：

   ```bash
   ./target/release/node-template \
   --base-path /tmp/alice \
   --chain local \
   --alice \
   --port 30333 \
   --rpc-port 9945 \
   --node-key 0000000000000000000000000000000000000000000000000000000000000001 \
   --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
   --validator
   ```

### 查看命令行选项

继续之前，请查看如何使用以下选项来启动节点。

| <div style="min-width:110pt;font-weight:bold;">选项</div> | <div style="font-weight:bold;">说明</div>                                                                                                                                                                                           |
| :---------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--base-path`                                               | 指定用于存储与此链相关的所有数据的目录。                                                                                                                                                                 |
| `--chain local`                                             | 指定要使用的链规范。有效的预定义链规范包括 `local`、`development` 和 `staging`。                                                                                                             |
| `--alice`                                                   | 将 `alice` 帐户的预定义密钥添加到节点的密钥库。使用此设置，`alice` 帐户用于区块生成和完成。                                                                             |
| `--port 30333`                                              | 指定用于对等（`p2p`）流量的监听端口。因为本教程使用在同一台物理计算机上运行的两个节点来模拟网络，所以您必须为至少一个帐户显式指定不同的端口。 |
| `--rpc-port 9945`                                            | 指定服务器将通过 WebSocket 和 HTTP 监听传入 JSON-RPC 流量的端口。默认端口为 `9944`。本教程使用自定义 WebSocket 端口号（`9945`）。                                                                                   |
| `--node-key <key>`                                          | 指定用于 `libp2p` 网络的 Ed25519 密钥。您应该仅在开发和测试中使用此选项。                                                                                                              |
| `--telemetry-url`                                           | 指定发送遥测数据的位置。对于本教程，您可以将遥测数据发送到 Parity 托管的服务器，任何人都可以使用该服务器。                                                                                   |
| `--validator`                                               | 指定此节点参与网络的区块生成和完成。                                                                                                                                                |

有关节点模板可用的命令行选项的更多信息，请通过运行以下命令查看使用帮助：

`./target/release/node-template --help`

### 查看显示的节点消息

如果节点成功启动，终端将显示描述网络操作的消息。
例如，您应该看到类似于此的输出：

```text
2022-08-16 15:29:55 Substrate Node    
2022-08-16 15:29:55 ✌️  version 4.0.0-dev-de262935ede    
2022-08-16 15:29:55 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2022    
2022-08-16 15:29:55 📋 Chain specification: Local Testnet    
2022-08-16 15:29:55 🏷  Node name: Alice    
2022-08-16 15:29:55 👤 Role: AUTHORITY    
2022-08-16 15:29:55 💾 Database: RocksDb at /tmp/alice/chains/local_testnet/db/full    
2022-08-16 15:29:55 ⛓  Native runtime: node-template-100 (node-template-1.tx1.au1)    
2022-08-16 15:29:55 🔨 Initializing Genesis block/state (state: 0x6894…033d, header-hash: 0x2cdc…a07f)    
2022-08-16 15:29:55 👴 Loading GRANDPA authority set from genesis on what appears to be first startup.    
2022-08-16 15:29:56 Using default protocol ID "sup" because none is configured in the chain specs    
2022-08-16 15:29:56 🏷  Local node identity is: 12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp    
2022-08-16 15:29:56 💻 Operating system: macos    
2022-08-16 15:29:56 💻 CPU architecture: x86_64    
2022-08-16 15:29:56 📦 Highest known block at #0    
2022-08-16 15:29:56 〽️ Prometheus exporter started at 127.0.0.1:9615    
2022-08-16 15:29:56 Running JSON-RPC server: addr=127.0.0.1:9945, allowed origins=Some(["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"])      
2022-08-16 15:29:56 creating instance on iface 192.168.1.125    
2022-08-16 15:30:01 💤 Idle (0 peers), best: #0 (0x2cdc…a07f), finalized #0 (0x2cdc…a07f), ⬇ 0 ⬆ 0
...
```

尤其要注意输出中的以下消息：

- `🔨 Initializing Genesis block/state (state: 0xea47…9ba8, header-hash: 0x9d07…7cce)` 标识节点正在使用的初始或**创世块**。
  启动下一个节点时，请验证这些值是否相同。

- `🏷 Local node identity is: 12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp` 指定一个唯一标识此节点的字符串。
  此字符串由用于使用 `alice` 帐户启动节点的 `--node-key` 确定。
  启动第二个节点时，您可以使用此字符串来标识要连接到的节点。
- `2021-03-10 17:34:37 💤 Idle (0 peers), best: #0 (0x9d07…7cce), finalized #0 (0x9d07…7cce), ⬇ 0 ⬆ 0` 表示网络中没有其他节点，并且没有生成区块。
  必须有另一个节点加入网络才能开始生成区块。

## 将第二个节点添加到区块链网络

现在您使用 `alice` 帐户密钥启动的节点正在运行，您可以使用 `bob` 帐户将另一个节点添加到网络。
因为您正在加入一个已经运行的网络，所以您可以使用运行中的节点来识别新节点要加入的网络。
这些命令与您之前使用的命令类似，但有一些重要的区别。

要将节点添加到正在运行的区块链：

1. 在您的计算机上打开一个**新的**终端 shell。

1. 更改到您编译 Substrate 节点模板的根目录。

1. 通过运行以下命令清除旧的链数据：

   ```bash
   ./target/release/node-template purge-chain --base-path /tmp/bob --chain local -y
   ```

   通过向命令添加 `-y`，您可以删除链数据而无需提示您确认操作。

1. 通过运行以下命令使用 `bob` 帐户启动第二个本地区块链节点：

   ```bash
   ./target/release/node-template \
   --base-path /tmp/bob \
   --chain local \
   --bob \
   --port 30334 \
   --rpc-port 9946 \
   --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
   --validator \
   --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
   ```

   请注意此命令与上一个命令之间的区别：

   - 因为两个节点都在同一台物理计算机上运行，所以您必须为 `--base-path`、`--port` 和 `--rpc-port` 选项指定不同的值。

   - 此命令包含 `--bootnodes` 选项并指定单个引导节点，即由 `alice` 启动的节点。

   `--bootnodes` 选项指定以下信息：

   - `ip4` 表示节点的 IP 地址使用 IPv4 格式

   - `127.0.0.1` 指定正在运行的节点的 IP 地址。
     在本例中，`localhost` 的地址。

   - `tcp` 指定 TCP 作为用于对等通信的协议。

   - `30333` 指定用于对等通信的端口号。
     在本例中，TCP 流量的端口号。

   - `12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp` 标识要与此网络通信的正在运行的节点。
     在本例中，使用 `alice` 帐户启动的节点的标识符。

## 验证区块是否已生成和完成

启动第二个节点后，节点应相互连接作为对等节点并开始生成区块。

要验证区块是否正在完成：

1. 验证您是否在启动第一个节点的终端中看到了类似于以下内容的行：

   ```text
   2022-08-16 15:32:33 discovered: 12D3KooWBCbmQovz78Hq7MzPxdx9d1gZzXMsn6HtWj29bW51YUKB /ip4/127.0.0.1/tcp/30334
   2022-08-16 15:32:33 discovered: 12D3KooWBCbmQovz78Hq7MzPxdx9d1gZzXMsn6HtWj29bW51YUKB /ip6/::1/tcp/30334
   2022-08-16 15:32:36 🙌 Starting consensus session on top of parent 0x2cdce15d31548063e89e10bd201faa63c623023bbc320346b9580ed3c40fa07f
   2022-08-16 15:32:36 🎁 Prepared block for proposing at 1 (5 ms) [hash: 0x9ab34110e4617454da33a3616efc394eb1ce95ee4bf0daab69aa4cb392d4104b; parent_hash: 0x2cdc…a07f; extrinsics (1): [0x4634…cebf]] 
   2022-08-16 15:32:36 🔖 Pre-sealed block for proposal at 1. Hash now 0xf0869a5cb8ebd0fcc5f2bc194ced84ca782d9749604e888c8b9b515517179847, previously 0x9ab34110e4617454da33a3616efc394eb1ce95ee4bf0daab69aa4cb392d4104b.
   2022-08-16 15:32:36 ✨ Imported #1 (0xf086…9847)
   2022-08-16 15:32:36 💤 Idle (1 peers), best: #1 (0xf086…9847), finalized #0 (0x2cdc…a07f), ⬇ 1.0kiB/s ⬆ 1.0kiB/s
   2022-08-16 15:32:41 💤 Idle (1 peers), best: #1 (0xf086…9847), finalized #0 (0x2cdc…a07f), ⬇ 0.6kiB/s ⬆ 0.6kiB/s
   2022-08-16 15:32:42 ✨ Imported #2 (0x0d5e…2a7f)
   2022-08-16 15:32:46 💤 Idle (1 peers), best: #2 (0x0d5e…2a7f), finalized #0 (0x2cdc…a07f), ⬇ 0.6kiB/s ⬆ 0.6kiB/s
   2022-08-16 15:32:48 🙌 Starting consensus session on top of parent 0x0d5ef31979c2aa17fb88497018206d3057151119337293fe85d9526ebd1e2a7f
   2022-08-16 15:32:48 🎁 Prepared block for proposing at 3 (0 ms) [hash: 0xa307c0112bce39e0dc689132452154da2079a27375b44c4d94790b46a601346a; parent_hash: 0x0d5e…2a7f; extrinsics (1): [0x63cc…39a6]]    
   2022-08-16 15:32:48 🔖 Pre-sealed block for proposal at 3. Hash now 0x0c55670e745dd12892c9e7d5205085a78ccea98df393a822fa9b3865cfb3d51b, previously 0xa307c0112bce39e0dc689132452154da2079a27375b44c4d94790b46a601346a.
   2022-08-16 15:32:48 ✨ Imported #3 (0x0c55…d51b)
   2022-08-16 15:32:51 💤 Idle (1 peers), best: #3 (0x0c55…d51b), finalized #1 (0xf086…9847), ⬇ 0.7kiB/s ⬆ 0.9kiB/s    
   ...
   ```

   在这些行中，您可以看到有关区块链的以下信息：

   - 在网络上发现了第二个节点身份（`12D3KooWBCbmQovz78Hq7MzPxdx9d1gZzXMsn6HtWj29bW51YUKB`）。
   - 该节点有一个对等节点（`1 peers`）。
   - 节点已生成一些区块（`best: #3 (0x0c55…d51b)`）。
   - 区块正在完成（`finalized #1 (0xf086…9847)`）。

1. 验证您是否在启动第二个节点的终端中看到了类似的输出。

1. 通过在终端 shell 中按 Control-c 关闭其中一个节点。

   关闭节点后，您会看到剩余节点现在有零个对等节点，并且已停止生成区块。
   例如：

   ```text
   2022-08-16 15:53:45 💤 Idle (1 peers), best: #143 (0x8f11…1684), finalized #141 (0x5fe3…5a25), ⬇ 0.8kiB/s ⬆ 0.7kiB/s
   2022-08-16 15:53:50 💤 Idle (0 peers), best: #143 (0x8f11…1684), finalized #141 (0x5fe3…5a25), ⬇ 83 B/s ⬆ 83 B/s
   ```  

2. 通过在终端 shell 中按 Control-c 关闭第二个节点。

   如果您想从模拟网络中删除链状态，请对 `/tmp/bob` 和 `/tmp/alice` 目录使用 `purge-chain` 子命令和 `--base-path` 命令行选项。

## 下一步

本教程介绍了启动私有区块链网络的第一步基本步骤。
在本教程中，您通过在单个计算机上运行两个节点并将预定义帐户用作参与者来模拟私有网络。

您学习了：

- 如何使用几个节点模板命令和命令行选项。

- 如何启动两个区块链节点，这些节点作为对等节点相互通信。

- 如何验证您的私有区块链节点是否正在生成区块。

下一个教程基于您在本教程中学到的信息，说明如何使用其他参与者和在单独计算机上运行的节点启动私有网络。

在 [添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes/) 中，您将学习：

- 如何生成您自己的密钥对。

- 如何创建使用您生成的密钥的自定义链规范。

- 如何将验证器添加到使用自定义链规范的私有网络。

如果您在本教程中遇到任何问题，请提交问题、提出问题或提供反馈。

- [提交问题](https://github.com/substrate-developer-hub/substrate-docs/issues/new/choose)。

- [Substrate Stack Exchange](https://substrate.stackexchange.com/)。
