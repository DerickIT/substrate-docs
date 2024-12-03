---
title: 自定义链规范
description: 在您创建了 Substrate 节点后，您希望启动一个具有许多对等节点的网络！
keywords:
  - 基础
  - 运行时
  - 可配置常量
---

在您创建了 Substrate 节点后，您希望启动一个具有许多对等节点的网络！
本指南展示了一种方法，可以统一创建链规范文件并分发它们，以便其他节点可以明确地发现并与您的网络对等。

本指南说明：

- 如何生成 `chain-spec.json` 并将其包含在内，以便其他节点加入公共网络
- 如何修改现有的纯链规范，而无需编辑节点源代码

## 创建链规范

**纯链规范文件** 

1. 在节点的工作目录中启动，使用以下命令生成纯链规范：

   ```bash
   ./target/release/solo-template-node build-spec > chain-spec-plain.json
   ```

   我们刚刚为 `chain_spec.rs` 文件中设置的默认网络生成了一个 **纯链规范** 文件。
   此文件可以传递给其他节点。

1. 修改纯链规范（可选）：

   此可选步骤，我们可以利用一个 _现有的_ 纯链规范，用于一个网络，否则需要修改节点的 _源代码_ 才能在 _新网络_ 上运行。
   例如，这在 [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 中非常有用，我们希望创建一个自定义 _中继链_，而无需自定义 Polkadot 的源代码。

   这里我们使用 _相同的_ 链规范，但传递一个标志来禁用引导节点，因为我们想要一个 _新的_ 网络，其中这些节点将不同。

   ```bash
   ./target/release/solo-template-node build-spec --chain chain-spec-plain.json --raw --disable-default-bootnode > no-bootnodes-chain-spec-plain.json
   ```

   此 `no-bootnodes-chain-spec-plain.json` 可用于生成 SCALE 存储编码的可分发原始链规范。

**原始链规范文件**

1. 生成原始链规范。

   有了纯规范，您可以通过运行以下命令生成最终的原始链规范：

   ```bash
   ./target/release/solo-template-node build-spec --chain chain-spec-plain.json --raw > chain-spec.json
   ```

原始链规范应始终在传递到节点以使用它时使用。

## 发布原始链规范

由于针对 WebAssembly 的 Rust 构建进行了优化，因此二进制文件不是确定性可重现的。
如果每个网络参与者都生成纯规范和/或原始链规范，则生成的 Wasm blob 的差异将破坏共识。

_惯例_ 是将节点的链规范文件 _包含在源代码本身中_，以便任何人都可以以相同的方式构建您的节点，从而通过将创世 blob 与另一个创世 blob 进行比较，可以轻松地检查非确定性。
Polkadot、Kusama、Rococo 以及更多网络链规范文件都可以在 [源代码中找到](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs)，以及一个 `.gitignore` 文件，以确保您在节点软件上进一步构建并进行 [运行时升级](/tutorials/build-a-blockchain/upgrade-a-running-network/) 时不会意外更改这些 `!/*.json` 文件。

## 启动一个新节点

如果您发布了一个节点二进制文件，或者让用户构建自己的节点，然后他们希望加入您的网络，那么他们只需要 _相同的_ 原始链规范文件，并使用以下命令运行您的二进制文件：

```bash
# 二进制文件名为 `solo-template-node`
# `chain-spec.json` 从规范的公共源获取
solo-template-node --chain chain-spec.json
```

这也可以简单地配置为默认网络。
作为参考，您可以查看 [Polkadot 如何实现](https://github.com/paritytech/polkadot/commits/master/cli/src/command.rs) 使用链规范的默认命令，用于 [源代码中](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs) 的各种网络。

## 示例

- [添加可信节点](/tutorials/build-a-blockchain/add-trusted-nodes#add-keys-to-keystore)
- [Polkadot 类网络链规范](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs)
- [Polkadot 用于多个网络的命令](https://github.com/paritytech/polkadot/commits/master/cli/src/command.rs)
