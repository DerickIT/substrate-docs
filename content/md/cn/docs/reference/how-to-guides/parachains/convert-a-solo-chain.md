---
title: 转换单链
description:
keywords:
  - collator
  - parachain
  - parathread
  - 升级
  - cumulus
  - 存储
  - 迁移
  - paraid
  - 注册
  - 启动
  - 转换
---

本指南说明了如何将 Cumulus 添加到单链中，并使用中继链来提供最终性。

您将学习如何：

- 引导单链的安全性
- 通过公共中继链上的 Parachain 访问 XCMP

对于任何 Substrate 链，Cumulus 的集成都使该链能够将其_最终性_与中继链（如 Polkadot）耦合。
本指南_不_说明如何迁移_正在运行的单链_，仅说明将节点的_代码库_转换为使用 Cumulus 进行共识（而不是像 GRANDPA 那样，这对于其他 Substrate 单链很常见）所需的步骤。

## Parachain 和单链节点模板

Substrate Parachain 模板类似于 Substrate 节点模板。
这两个模板都包含 `node`、`runtime` 和 `pallets` 目录以及许多相同的预定义 pallet 和特性。
您可以使用任一模板遵循大多数 [教程](/tutorials/)。
但是，节点和 Parachain 模板之间存在一些重要的区别，您应该从一开始就注意这些区别。

### Parachain 信息 pallet

默认情况下，Parachain 模板 [运行时](https://github.com/paritytech/polkadot-sdk-parachain-template/blob/master/runtime/Cargo.toml) 包含几个 Parachain 特定的 pallet，包括 [`parachain-info` pallet](https://paritytech.github.io/polkadot-sdk/master/staging_parachain_info/pallet/index.html)。
此 pallet 旨在将唯一的 Parachain 标识符注入到 Parachain 运行时中。
此信息允许运行时知道哪些跨链消息是针对它的。

### 验证区块宏

每个 Parachain 都必须在向该中继链注册时向中继链提供一个 `validate_block` 函数（以 WebAssembly blob 的形式）。
此函数仅 Parachain 需要，因此默认情况下它不包含在节点模板中。
但是，Parachain 模板通过将以下代码添加到运行时逻辑的底部来为 Substrate 运行时创建此函数：

```rust
cumulus_pallet_parachain_system::register_validate_block!(
  Runtime = Runtime,
  BlockExecutor = cumulus_pallet_aura_ext::BlockExecutor::<Runtime, Executive>,
  CheckInherents = CheckInherents,
);
```

### 最终性取决于中继链

Parachain 模板不包含任何区块最终确定机制，因为 Parachain 旨在使用中继链提供的最终性。
中继链最终确定是 Polkadot 和其他中继链架构中的一个基本概念。
相反，Substrate 节点模板和许多其他基于 Substrate 的链实现了它们自己的区块最终确定机制，通常使用 GRANDPA pallet 和相关的 API。

### Collator 服务

collator 服务（[`node/src/service.rs`](https://github.com/paritytech/polkadot-sdk-parachain-template/blob/master/node/src/service.rs)）在 Parachain 模板中的与节点模板中类似命名的 [`node/src/service.rs`](https://github.com/paritytech/polkadot-sdk-parachain-template/blob/master/node/src/service.rs) 完全不同。
collator 服务被明确设计为一个包装器，以提供标准 Substrate 节点不需要的 Parachain 特定操作。

如果您有一个想要转换为 Parachain 的现有 Substrate 链，则应将 [`node/src/service.rs`](https://github.com/paritytech/polkadot-sdk-parachain-template/blob/master/node/src/service.rs) 从 Parachain 模板复制为起点。
