---
title: 添加本地 Parachain 节点
description: 如何将额外的 Parachain 节点连接到已建立的本地中继测试网络
keywords:
  - parachain
  - collator
  - 全节点
---

正如您在 [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 中了解到的那样，Parachain 可以与单个 collator 协同工作。

## 额外的中继链节点

对于网络上的每个 [**collator**](/reference/glossary/#collator)（Parachain 区块创作节点），您应该至少运行两个 [**验证者**](/reference/glossary/#validators)（中继链节点）。

您可以**修改**提供的**普通**[中继链规范文件](/tutorials/build-a-parachain/prepare-a-local-relay-chain/#pre-configured-chain-spec-files)以包含更多验证者，或者采用更“正确”的生产方法，即修改**中继链**的 `chain_spec.rs` 中创世状态的**源代码**，以便在创世时添加更多测试网验证者。
回顾如何在 [添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes/) 教程中生成链规范。

### 启动 collator

### 启动 collator 节点

有了链规范，您现在可以启动 collator 节点了。
请注意，我们需要在命令的后半部分提供与启动中继链节点时使用的相同的中继链链规范：

```bash
./target/release/parachain-collator \
--alice \
--collator \
--force-authoring \
--chain rococo-local-parachain-2000-raw.json \
--base-path /tmp/parachain/alice \
--port 40333 \
--rpc-port 8844 \
-- \
--execution wasm \
--chain <中继链原始链规范> \
--port 30343 \
--rpc-port 9977
```

### 启动第二个 collator

运行其他 collator 的命令如下，假设 `Alice` 节点已经在运行。
此命令与我们用于启动第一个 collator 的命令几乎相同，但我们需要避免端口和 `base-path` 目录冲突：

```bash
./target/release/parachain-collator \
--bob \
--collator \
--force-authoring \
--chain rococo-local-parachain-2000-raw.json \
--base-path /tmp/parachain/bob \
--bootnodes <正在运行的 collator 节点> \
--port 40334 \
--rpc-port 9946 \
-- \
--execution wasm \
--chain <中继链链规范> \
--port 30344 \
--rpc-port 9978
--bootnodes <其他中继链节点>
```

## 相关资料

- [Polkadot Wiki: Collator](https://wiki.polkadot.network/docs/learn-collator)。
- [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 教程。
