---
title: Add trusted nodes
description: Generate account keys and a custom chain specification for a private blockchain network.
keywords:
  - enterprize
  - private
  - permission
---
# 添加可信节点

本教程演示了如何启动一个小型独立的区块链网络,其中包含一组私有**验证者**的**权威集合**。

正如您在[区块链基础知识](/learn/blockchain-basics/)中所学到的,所有区块链都要求网络中的节点在任何特定时间点就数据状态达成一致,这种对状态的一致称为**共识**。

Substrate 节点模板使用一种称为**权威轮询**或 **Aura** 共识的权威证明共识模型。

Aura 共识协议将区块生产限制在一个轮流的授权账户列表中。

这些授权账户——**权威**——以轮询方式创建区块,通常被认为是网络中受信任的参与者。

这种共识模型为有限数量参与者启动单独的区块链提供了一种简单的方法。

在本教程中,您将了解如何生成授权节点参与网络所需的密钥,如何配置网络并与其他授权账户共享网络信息,以及如何使用批准的验证者集启动网络。

## 开始之前

在开始之前,请验证以下内容:

- 您已通过安装 [Rust 和 Rust 工具链](/install/)配置了 Substrate 开发环境。

- 您已完成[构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)并在本地安装了 Substrate 节点模板。

- 您已按照[模拟网络](/tutorials/build-a-blockchain/simulate-network/)中的描述使用预定义账户在单台计算机上启动节点。

- 您通常熟悉软件开发和使用命令行界面。

- 您通常熟悉区块链和智能合约平台。

## 教程目标

通过完成本教程,您将实现以下目标:

- 生成用作网络权威的密钥对。

- 创建自定义链规范文件。

- 启动私有双节点区块链网络。

## 生成您的账户和密钥

在[模拟网络](/tutorials/build-a-blockchain/simulate-network/)中,您使用预定义的账户和密钥启动了对等节点。

对于本教程,您将为网络中的验证者节点生成自己的密钥。

请记住,区块链网络中的每个参与者都负责生成唯一的密钥。

### 密钥生成选项

有几种方法可以生成密钥。

例如,您可以使用 `node-template` 子命令、独立的 [subkey](/reference/command-line-tools/subkey/) 命令行程序、Polkadot-JS 应用程序或第三方密钥生成实用程序生成密钥对。

虽然您可以使用预定义的密钥对来完成本教程,但在生产环境中绝不会使用这些密钥。

本教程将演示如何使用 Substrate 节点模板和 `key` 子命令生成密钥,而不是使用预定义的密钥或更安全的 `subkey` 程序。

### 使用节点模板生成本地密钥

作为最佳实践,在生成用于生产区块链的密钥时,您应该使用从未连接过互联网的离线计算机。

至少,在生成任何您打算在不受您控制的公共或私有区块链上使用的密钥之前,您应该断开互联网连接。

但是,对于本教程,您可以使用 `node-template` 命令行选项在本地生成随机密钥,同时保持与互联网的连接。

要使用节点模板生成密钥:

1. 在计算机上打开终端 shell。

2. 切换到编译 Substrate 节点模板的根目录。

3. 通过运行以下命令生成随机助记词和密钥:

```bash
./target/release/node-template key generate --scheme Sr25519 --password-interactive
```

4. 为生成的密钥输入密码。

该命令生成密钥并显示类似以下内容的输出:

```text
Secret phrase: pig giraffe ceiling enter weird liar orange decline behind total despair fly
Secret seed: 0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex): 0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Account ID: 0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Public key (SS58): 5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
SS58 Address: 5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
```

您现在拥有一个节点使用 `aura` 生成区块的 Sr25519 密钥。

在本例中,该账户的 Sr25519 公钥是 `5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW`。

5. 使用您刚刚生成的账户的**助记词**来派生使用 Ed25519 签名方案的密钥。

例如,运行类似以下的命令:

```bash
./target/release/node-template key inspect --password-interactive --scheme Ed25519 "pig giraffe ceiling enter weird liar orange decline behind total despair fly"
```

6. 输入您用于生成密钥的密码。

该命令显示类似以下内容的输出:

```text
Secret phrase `pig giraffe ceiling enter weird liar orange decline behind total despair fly` is account:
Secret seed: 0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex): 0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
Public key (SS58): 5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
Account ID: 0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
SS58 Address: 5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
```

您现在拥有一个节点使用 `grandpa` 最终确定区块的 Ed25519 密钥。

在本例中,该账户的 Ed25519 公钥是 `5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN`。

### 生成第二组密钥

对于本教程,私有网络仅包含两个节点,因此您需要两组密钥。

您有几个选项可以继续本教程:

- 您可以使用预定义账户之一的密钥。

- 您可以在本地计算机上使用不同的身份重复上一节中的步骤来生成第二个密钥对。

- 您可以派生子密钥对来模拟本地计算机上的第二个身份。

- 您可以招募其他参与者生成加入您的私有网络所需的密钥。

为了说明目的,本教程中使用的第二组密钥是:

- Sr25519: 5EJPj83tJuJtTVE2v7B9ehfM7jNT44CBFaPWicvBwYyUKBS6 用于 `aura`。

- Ed25519: 5FeJQsfmbbJLTH1pvehBxrZrT5kHvJFj84ZaY5LK7NU87gZS 用于 `grandpa`。

## 创建自定义链规范

生成用于区块链的密钥后,您就可以使用这些密钥对创建自定义**链规范**,然后与称为**验证者**的受信任网络参与者共享您的自定义链规范。

要使其他人能够参与您的区块链网络,请确保他们生成自己的密钥。

收集网络参与者的密钥后,您可以创建自定义链规范来替换 `local` 链规范。

为简单起见,本教程中创建的自定义链规范是 `local` 链规范的修改版本,说明如何创建双节点网络。

如果您有所需的密钥,可以按照相同的步骤向网络添加更多节点。

### 修改本地链规范

您可以修改预定义的 `local` 链规范,而不是编写全新的链规范。

要基于本地规范创建新的链规范:

1. 在计算机上打开终端 shell。

2. 切换到编译 Substrate 节点模板的根目录。

3. 通过运行以下命令将 `local` 链规范导出到名为 `customSpec.json` 的文件:

```bash
./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
```

如果您在文本编辑器中打开 `customSpec.json` 文件,您会看到它包含几个字段。其中一个字段是您使用 `cargo build --release` 命令构建的运行时的 WebAssembly (Wasm) 二进制文件。

由于 WebAssembly (Wasm) 二进制文件是一个大的 blob,您可以预览前几行和最后几行以查看需要更改的字段。

4. 通过运行以下命令预览 `customSpec.json` 文件中的前几个字段:

```bash
head customSpec.json
```

该命令显示文件中的前几个字段。

例如:

```json
{
  "name": "Local Testnet",
  "id": "local_testnet",
  "chainType": "Local",
  "bootNodes": [],
  "telemetryEndpoints": null,
  "protocolId": null,
  "properties": null,
  "consensusEngine": null,
  "codeSubstitutes": {},
```

5. 通过运行以下命令预览 `customSpec.json` 文件中的最后几个字段:

```bash
tail -n 80 customSpec.json
```

此命令显示 Wasm 二进制字段之后的最后几个部分,包括运行时中使用的几个 pallet 的详细信息,例如 `sudo` 和 `balances` pallet。

6. 在文本编辑器中打开 `customSpec.json` 文件。

7. 修改 `name` 字段以将此链规范标识为自定义链规范。

例如:

```json
"name": "My Custom Testnet",
```

8. 修改 `aura` 字段以指定有权创建区块的节点,方法是为每个网络参与者添加 Sr25519 SS58 地址密钥。

```json
"aura": { 
  "authorities": [
    "5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW",
    "5CXGP4oPXC1Je3zf5wEDkYeAqGcGXyKWSRX2Jm14GdME5Xc5"
  ]
},
```

9. 修改 `grandpa` 字段以指定有权最终确定区块的节点,方法是为每个网络参与者添加 Ed25519 SS58 地址密钥。

```json
"grandpa": {
  "authorities": [
    [
      "5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN",
      1
    ],
    [
      "5DpdMN4bVTMy67TfMMtinQTcUmLhZBWoWarHvEYPM4jYziqm",
      1
    ]
  ]
},
```

请注意,`grandpa` 部分中 `authorities` 字段有两个数据值。

第一个值是地址密钥。

第二个值用于支持**加权投票**。

在此示例中,每个验证者的权重为 **1** 票。

10. 保存更改并关闭文件。

### 添加验证者

正如您刚才看到的,您可以通过修改 `aura` 和 `grandpa` 部分来在链规范中添加和更改权限地址。

您可以使用此技术添加任意数量的验证者。

要添加验证者:

- 修改 `aura` 部分以包含 **Sr25519** 地址。

- 修改 `grandpa` 部分以包含 **Ed25519** 地址和投票权重。

确保为每个验证者使用唯一的密钥。

如果两个验证者具有相同的密钥,它们将产生冲突的区块。

有关使用密钥对和签名的更多信息,请参阅[公钥密码学](/learn/cryptography/#public-key-cryptography)。

## 将链规范转换为原始格式

准备包含验证者信息的链规范后,必须先将其转换为原始规范格式,然后才能使用。

原始链规范包含运行时的 WebAssembly 二进制文件,这是启动自定义链所必需的。

要将 JSON 链规范转换为原始格式:

1. 在终端 shell 中,运行以下命令:

```bash
./target/release/node-template build-spec --chain=customSpec.json --raw --disable-default-bootnode > customSpecRaw.json
```

2. 验证 `customSpecRaw.json` 文件是否已创建。

## 启动第一个节点

现在您已经准备好使用自定义链规范启动第一个节点。

要启动第一个节点:

1. 在终端 shell 中,运行以下命令:

```bash
./target/release/node-template \
--base-path /tmp/node01 \
--chain ./customSpecRaw.json \
--port 30333 \
--ws-port 9945 \
--rpc-port 9933 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode01 \
--password-interactive
```

2. 当提示输入密码时,输入您用于生成密钥的密码。

该命令使用以下命令行选项启动节点:

- `--base-path` 指定存储链数据的自定义位置。
- `--chain` 指定要使用的自定义链规范。
- `--port` 指定用于点对点(P2P)通信的自定义端口。
- `--ws-port` 指定用于 WebSocket 连接的自定义端口。
- `--rpc-port` 指定用于 RPC 调用的自定义端口。
- `--validator` 指定此节点是验证者。
- `--rpc-methods Unsafe` 允许使用不安全的 RPC 方法。
- `--name` 指定要在节点浏览器中显示的自定义节点名称。
- `--telemetry-url` 将遥测数据发送到特定端点。
- `--password-interactive` 允许您交互式输入密码。

3. 注意节点启动后显示的节点标识。

例如,输出应类似于以下内容:

```text
Local node identity is: 12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX
```

您需要此节点标识来允许其他节点连接到此节点。

## 启动第二个节点

现在第一个节点正在运行,您可以启动第二个节点。

要启动第二个节点并将其连接到第一个节点:

1. 打开一个新的终端 shell。

2. 切换到编译 Substrate 节点模板的根目录。

3. 运行以下命令以启动第二个节点并连接到第一个节点:

```bash
./target/release/node-template \
--base-path /tmp/node02 \
--chain ./customSpecRaw.json \
--port 30334 \
--ws-port 9946 \
--rpc-port 9934 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode02 \
--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX \
--password-interactive
```

4. 当提示输入密码时,输入您用于生成第二组密钥的密码。

这个命令使用与第一个节点类似的命令行选项,但有以下区别:

- 使用不同的 `--base-path`、`--port`、`--ws-port`、`--rpc-port` 和 `--name` 值。
- 添加 `--bootnodes` 选项以连接到第一个节点。

`--bootnodes` 选项指定第一个节点的 IP 地址、端口和节点标识。

## 验证网络是否正在生成区块

如果您已正确完成所有步骤,两个节点现在应该正在生成和最终确定区块。

您可以通过查看每个终端窗口中的输出来验证这一点。

您应该看到类似以下内容的消息:

```text
2021-11-03 15:48:23 🔨 Prepared block for proposing at 1 (2 ms) [hash: 0x1d59b567d574f89d9d9fe58ae4f00020f7f0a987f5adf0f0553b69fe9003e309; parent_hash: 0x92f0c1731fe8c1b27c5c1f1b5571d78280365d0ed31e7c7fb12ba5e5c0c42945; extrinsics (1): [0x4634f4db3963bd32e1af2fc1f717fc85c6ef68e6dd7943e456b9e802c5bb9f35]]
2021-11-03 15:48:23 🔖 Pre-sealed block for proposal at 1. Hash now 0x4f8c9b2946c9e13911a7440726d20fd33de04510a78cb6a4b7e0e7eaa20eb2a8, previously 0x1d59b567d574f89d9d9fe58ae4f00020f7f0a987f5adf0f0553b69fe9003e309.
2021-11-03 15:48:23 ✨ Imported #1 (0x4f8c…2a8)
2021-11-03 15:48:23 ♻️  Reorg on #1,0x4f8c…2a8 to #1,0x4f8c…2a8, common ancestor #0,0x92f0…945
2021-11-03 15:48:24 💤 Idle (0 peers), best: #1 (0x4f8c…2a8), finalized #0 (0x92f0…945), ⬇ 0 ⬆ 0
2021-11-03 15:48:26 ✨ Imported #2 (0x4d1a…e7c6)
2021-11-03 15:48:28 🙌 Starting consensus session on top of parent 0x4d1a0c7cf8d3cebbf33be7ca3ba4981bc7b3da2e2c9bb31c7f56f806e8b8e7c6
2021-11-03 15:48:28 🎁 Prepared block for proposing at 3 (0 ms) [hash: 0x8a6a2f3c5fe11948a3d96e4457969a78a72df9f76ab3f4ea0d4f55316336a132; parent_hash: 0x4d1a0c7cf8d3cebbf33be7ca3ba4981bc7b3da2e2c9bb31c7f56f806e8b8e7c6; extrinsics (1): [0x7c9bae38b600b995d5e5b07f2cec2c12c5c95c4f8c89e1e42bfc06228f161bd1]]
2021-11-03 15:48:28 🔖 Pre-sealed block for proposal at 3. Hash now 0x7c4c0c238b8c09c2d9fd90a3fe2f2a209b251f79c2efd974b8dcd7f5385e3f0f, previously 0x8a6a2f3c5fe11948a3d96e4457969a78a72df9f76ab3f4ea0d4f55316336a132.
2021-11-03 15:48:28 ✨ Imported #3 (0x7c4c…3f0f)
2021-11-03 15:48:29 💤 Idle (1 peers), best: #3 (0x7c4c…3f0f), finalized #1 (0x4f8c…2a8), ⬇ 1.0kiB/s ⬆ 1.0kiB/s
```

这些消息表明网络正在生成和最终确定区块。

## 下一步

恭喜!您已成功启动了一个具有两个验证者节点的小型私有区块链网络。

在本教程中,您学习了:

- 如何生成用作网络权威的密钥对。
- 如何创建自定义链规范文件。
- 如何启动私有双节点区块链网络。

要了解更多信息,请探索以下资源:

- [账户、地址和密钥](/learn/accounts-addresses-keys/)
- [共识](/learn/consensus/)
- [节点模板命令行参考](/reference/command-line-tools/node-template/)