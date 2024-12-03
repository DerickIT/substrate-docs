---
title: 升级 Parachain
description:
keywords:
  - collator
  - parachain
  - 升级
  - cumulus
  - 存储
  - 迁移
---

本指南的目标是帮助 Parachain 开发人员确保运行时升级成功。

本指南说明：

- 如何修改 Parachain 运行时（添加/删除 pallet）
- 如何进行 Parachain 存储迁移

Parachain 上的运行时升级具有_非常_严格的要求，并且需要稍微不同的流程，因为您_必须_与中继链协调以促进此操作。
因此，以及状态转换协调的非常有限的性质，既要**快速**又要**简洁**到足以将区块包含在中继链中。

## 继续之前

在尝试升级 Parachain 的运行时之前，请验证以下事项：

- 您熟悉如何对不连接到中继链的 Substrate 节点执行 [运行时升级](/maintain/runtime-upgrades)。
- 您已按照 [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain) 中的步骤操作，并且熟悉 Parachain collator 节点和中继链之间的交互。
- 您可以访问 `polkadot-launch` 进行测试。

## 选择升级方法

如果您的现有 Substrate 链具有非常大的状态，您要在不同的存储格式之间迁移，则可能无法在一个区块内运行所有运行时迁移。
您可以使用一些策略来解决此问题：

1. 如果可以实际地在两个或三个区块内处理要迁移的存储项的数量，则可以使用 [Scheduler pallet](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/scheduler) 来运行迁移，以确保无论区块生产者如何都能执行它们。

1. 使用版本化存储，并且仅在访问尚未升级的存储值时才执行迁移。
   这可能会导致用户之间的交易费用差异，并可能导致更复杂的运行时代码。
   但是，如果正确计量（权重已正确基准测试），此方法将确保迁移的停机时间最少。

1. 如果您必须在多个区块中拆分迁移，则可以在链上或链下进行：

   - 链上多区块迁移将需要编写自定义 pallet 逻辑，该逻辑可以随着时间的推移排队更改，或者使用 Scheduler pallet 来一次迁移一部分存储。

   - 您可以手动在链下生成迁移，而不是将迁移代码添加到运行时，并使用多个 `system.setStorage` 调用来根据需要添加和删除存储项，方法是通过具有 root 权限的来源（例如民主）。
     如果您要进行的交易数量有限，则可以通过调度程序批量处理多个交易以随着时间的推移发生。

建立迁移策略后，您应该在非生产测试网上测试迁移，以确保它在继续之前有效。

在受限网络中进行测试将帮助您为在具有许多 collator 和验证者以及带宽和延迟等约束的真实网络中可能发生的故障做好准备。
您模拟真实网络进行测试的越接近，您就可以越确定运行时升级将成功。

## 授权 -> 制定升级流程

当最终准备好升级 Parachain 时，中继链需要在升级发生之前被告知链的运行时升级。
[Cumulus](https://github.com/paritytech/polkadot-sdk/tree/master/cumulus) 库提供了帮助您通过以下方式通知中继链即将进行的升级的功能：

1. **使用 [`authorize_upgrade`](https://paritytech.github.io/cumulus/cumulus_pallet_parachain_system/pallet/struct.Pallet.html#method.authorize_upgrade)** 提供升级的哈希值并授权它。
1. **使用 [`enact_authorized_upgrade`](https://paritytech.github.io/cumulus/cumulus_pallet_parachain_system/pallet/struct.Pallet.html#method.enact_authorized_upgrade)** 提供升级的实际代码。

调用这两个函数后，中继链将收到已安排新升级的通知。

## 示例

- [Substrate 迁移示例仓库](https://github.com/apopiak/substrate-migrations)
- [Staking pallet 迁移逻辑](https://github.com/paritytech/substrate/blob/6be513d663836c5c5b8a436f5712402a1c5365a3/frame/staking/src/lib.rs#L757)

## 资源

- [运行时升级](/maintain/runtime-upgrades)
- [Fork off Substrate](https://github.com/maxsam4/fork-off-substrate)
- [`try-runtime`](/reference/command-line-tools/try-runtime)
- [`try-runtime` 视频研讨会](https://www.crowdcast.io/e/substrate-seminar/41)
