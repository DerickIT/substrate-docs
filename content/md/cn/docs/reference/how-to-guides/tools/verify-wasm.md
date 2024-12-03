---
title: 验证 Wasm 二进制文件
description: 公开任何 Substrate 链的运行时功能。
keywords:
  - 工具
  - wasm
  - 运行时
---

[subwasm](https://github.com/chevdor/subwasm) 工具提供了一种方法来公开任何 Substrate 链的运行时功能。
本指南将向您展示如何使用 Subwasm 执行以下操作：

- 获取运行时中模块的快照。
- 公开链运行时的可验证历史信息，以用于前端应用程序。
- 使用 diff 检查两个运行时之间的更改。

## 开始之前

确保您拥有以下内容：

- 已安装 [最新版本](https://github.com/chevdor/subwasm/releases) 的 Subwasm。
- 本地运行的链正在执行您想要检查的运行时。
- 在您的环境中安装了 [JQ](https://stedolan.github.io/jq/download/) 以启用元数据的过滤。

## 获取本地链运行时的信息

1. 从链的目录内部启动本地节点。
   此示例在开发模式下启动节点模板。

   ```bash
   ./target/release/node-template --tmp --dev
   ```

1. 在新的终端中，创建一个名为 `TempWasms` 的临时文件夹。

   ```bash
   mkdir TempWasms
   ```

1. 在新文件夹内，使用 Subwasm 获取并保存本地链的 Wasm。

   ```bash
   cd TempWasms
   subwasm get --chain local -o mychain.wasm
   ```

   `-o` 标志会将 Wasm 文件保存到 `mychain.wasm`。

1. 运行以下命令以将链的元数据保存到 JSON 文件中：

   ```bash
    subwasm --json meta mychain.wasm | jq 'del( .. | .documentation? )' > mychain-metadata.json
   ```

   您可以使用 JQ 从 `subwasm --json meta mychain.wasm` 命令中过滤元数据详细信息。
   此示例忽略所有文档元数据。
   有关 Substrate 链公开的元数据字段的信息，请参阅有关 [Substrate 元数据](https://polkadot.js.org/docs/substrate) 的文档。

1. 将元数据保存到 JSON 文件后，您可以在前端应用程序中使用它。

## 比较活动链的两个不同运行时

您还可以使用 Subwasm 来比较链在不同区块处的运行时。
Subwasm 使用一个函数，该函数获取常见 Substrate 链（如 Polkadot、Kusama 和 Westend）的 URL，并返回节点的端点。
以下示例将区块 500,000 处的 Polkadot 运行时与其最新的运行时进行比较。

1. 前往区块浏览器并检索您想要比较的运行时的区块哈希。

   例如，前往 [PolkadotJS Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/explorer) 并检索 [区块 500000](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/explorer/query/500000) 处的区块哈希。

   `0xfdeab19649426aadbdab046c9b5dac0a5b0c850b8fd0927e3b4e6c896d5fb0b7`

1. 保存先前状态的 Wasm blob。

   ```bash
   BLOCK_HASH=0xfdeab19649426aadbdab046c9b5dac0a5b0c850b8fd0927e3b4e6c896d5fb0b7
   subwasm get --chain polkadot --block $BLOCK_HASH -o polkadot-500000.wasm
   ```

1. 获取运行时的最新 Wasm blob。

   ```bash
   subwasm get --chain polkadot -o polkadot-latest.wasm
   ```

1. 比较这两个运行时并将输出放入 JSON 文件中。

   ```bash
   subwasm diff polkadot-latest.wasm polkadot-500000.wasm --json > polkadot-wasm-diff.json
   ```

## 示例

如上所述，`subwasm` 有许多用例。
一些额外的示例包括：

- 压缩和解压缩 Wasm 文件。
- 检查 Wasm 文件以验证治理中的运行时升级。
