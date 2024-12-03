---
title: 准备启动
description:
keywords:
  - collator
  - parachain
  - 启动
  - 先决条件
  - 要求
  - 检查清单
  - 权重
  - 迁移
---

本指南旨在指导您如何在本地测试环境之外启动 Parachain。
您将学习如何确保 `protocolId` 设置唯一以及运行时权重正确。

Parachain 上的运行时约束比单链严格得多，因为您必须与中继链协调以最终确定状态转换。在将 Parachain 启动到生产环境时，确保链的运行时已正确配置和测试至关重要。

## 设置唯一的协议标识符

网络冲突会导致严重的麻烦。
_所有_链都应使用其他任何类型的网络（无论是测试网络、中继链还是 Parachain）都不使用的**唯一** `protocolId`。
拥有唯一的协议标识符可确保您的节点连接到正确的对等节点，而不是连接到其他 libp2p 网络的节点。
您希望使用此 ID 将它们隔离到一个不同的对等组。
协议 ID 冲突将导致您的节点出现_许多_问题。

为了设置唯一的协议 ID，请确保您使用一些随机数或盐值。这在
（对于 [Parachain 节点模板](https://github.com/paritytech/polkadot-sdk-parachain-template/)）
中设置为 `/client/network/src/command.rs` 中的 CLI 项目，并传递到扩展 `/client/network/src/chain_spec.rs`

所有 [链规范](/build/chain-spec/) 文件都将此项目作为字段包含在内。
例如，主要 [中继链运行时](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs) 链规范具有唯一的协议 ID。
对于 Polkadot：

```json
// polkadot 代码库中 `/node/service/chain-specs/polkadot.json` 的原始链规范文件
{
  //--snip--
  "protocolId": "dot"
  //--snip--
}
```

监控 [此问题](https://github.com/paritytech/substrate/issues/7746) 以获取有关将来更安全地配置此常量的方法的更新。

## 内存分析

应进行 Collator [内存分析](/reference/command-line-tools/memory-profiler) 以分析内存泄漏、确定内存消耗发生的位置、定义临时分配以及调查应用程序中的过度内存碎片。

## 最小化运行时大小

在启动 Parachain 时，至关重要的是使用**运行时的压缩版本**来尽可能降低中继链的资源消耗量。

- 建议使用有限功能启动 Parachain，并通过运行时升级逐步增加功能。其背后的原因是，在运行时升级期间，之前的运行时和新的运行时都包含在 PoVBlock 中，因此，如果更改足够大，则由于 PoVBlock 大小限制，中继链可能会拒绝该区块。

- 如果运行时包含在状态证明中，请确保 PoV 块（即，一系列外部函数，包括新的运行时、PoV 状态证明，可能还有旧的运行时）适合 PoVBlock 大小限制。如果运行时不包含在状态证明中，则新运行时的大小限制将高得多。

## 关键 Parachain 约束

您可以在 [Polkadot 代码库](https://github.com/paritytech/polkadot-sdk/blob/43cd6fd4370d3043272f64a79aeb9e6dc0edd13f/polkadot/primitives/src/v8/mod.rs#L401-L429) 中检查所有中继链的最大大小（这些是公共常量）。
请注意：

- 您目标中继链的运行时版本（这些_可能会_更改）
- `MAX_CODE_SIZE`
- `MAX_HEAD_DATA_SIZE`
- `MAX_POV_SIZE`

您**必须**使您的 Parachain 舒适地适应这些最大值。
您还可以使用连接到中继节点的 Polkadot-JS Apps UI 来查看这些常量：_开发者_ -> _ParachainsConfiguration_ -> _ActiveConfiguration_

## 使用正确的权重

使用 [运行时基准测试](/test/benchmark) 以确保运行时权重实际上指示运行时使用的资源。

### 自定义权重

如果您需要偏离基准测试，请确保运行时中的每个 pallet 都采用了正确的加权系统。一般来说，**不**应在生产中使用默认权重和“猜测”权重。

### 设置区块权重限制

建议一开始将区块权重限制（区块生产时间）设置为 0.5 秒，因为区块执行时间存在不确定性。随着网络执行时间的稳定，权重限制可以增加到 2 秒。

## 设置调用过滤器

尤其是在启动 Parachain 时，您可能需要严格限制为_特定类别_的用户启用的功能。这可以通过**调用过滤器**来实现。

在这里，您可以看到如何在 [限制](https://github.com/paritytech/cumulus/blob/59cdbb6a56b1c49009413d66ba2232494563b57c/polkadot-parachains/statemine/src/lib.rs#L148) 和 [启用](https://github.com/paritytech/cumulus/pull/476/files#diff-09b95657e9aa1b646722afa7944a00ddc2541e8753254a86180b338d3376f93eL151) 功能方面使用过滤器，如 [Statemine 运行时部署](https://github.com/paritytech/cumulus/pull/476) 中所实现的那样。

## 增量运行时部署

如果您在启动前接近上述限制（例如运行时过大），则强烈建议尽可能减少功能并进行**增量升级**。
在这种情况下，您可以：

1. 使用完整的运行时功能（包括所有 pallet）生成链的创世状态

2. 从运行时中删除启动 Parachain 时不需要的所有 pallet

3. 重新构建 WASM blob（验证逻辑）和链的运行时

4. 使用在 (3) 中生成的更新的创世状态和 WASM blob 注册您的 Parachain

5. 在您的 Parachain 运行后，您可以通过链上升级运行时来包含缺少的 pallet（确保 pallet 索引和名称与步骤 (1) 中用于生成创世状态的索引和名称匹配），而无需进行存储迁移。有关链上运行时升级的更多信息，请参阅下一节。

有关如何实际执行这些增量运行时升级，请参阅**[Parachain 运行时升级指南](/reference/how-to-guides/parachains/runtime-upgrade)**。

## 启动模拟

在尝试在生产测试网或主网上尝试任何操作之前，您应该在尽可能接近模拟真实网络行为的网络上启动您的链。
在受限网络中进行测试将帮助您为在具有许多 collator 和验证者以及带宽和延迟等约束的真实网络中可能发生的故障做好准备。
您模拟真实网络进行测试的越接近，您就可以越确定运行时升级将成功。

请参阅 [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 以了解用于此类测试自动化的工具选择。

## 示例

- [Statemine 运行时部署](https://github.com/paritytech/cumulus/pull/476)

## 资源

- [运行时升级的参考文档](/build/upgrade)
- [使用基准权重的操作指南](/reference/how-to-guides/weights/add-benchmarks)
- [`try-runtime` 文档的参考](/reference/command-line-tools/try-runtime)
  - [`try-runtime` 视频研讨会](https://www.crowdcast.io/e/substrate-seminar/41)
- [Fork Off Substrate 工具](https://github.com/maxsam4/fork-off-substrate)
