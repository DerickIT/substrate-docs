---
title: 运行时开发
description: 介绍开发 Substrate 区块链运行时必不可少的核心编程接口、原语和模块。
keywords:
---

如[架构](/learn/architecture/)中所述，Substrate 节点的**运行时**包含执行交易、保存状态转换以及与外部节点交互的所有业务逻辑。
Substrate 提供了构建常见区块链组件所需的所有工具，因此您可以专注于开发定义区块链行为的运行时逻辑。

## 状态转换和运行时

在最基本的层面上，每个区块链本质上都是一个账本或记录，记录链上发生的每个更改。
在基于 Substrate 的链中，这些状态更改记录在运行时中。
由于运行时处理此操作，因此运行时有时被描述为提供[状态转换函数](/reference/glossary#state-transition-function-stf)。

由于状态转换发生在运行时中，因此运行时是您定义**存储项**的地方，这些存储项代表区块链[状态](/reference/glossary#state)，以及[交易](/learn/transaction-types)，这些交易允许区块链用户对该状态进行更改。

![运行时中的状态和函数](/media/images/docs/state-transition-function.png)

Substrate 运行时确定哪些交易有效，哪些交易无效，以及如何响应交易更改链状态。

## 运行时接口

如您在[架构](/learn/architecture/)中所学，外部节点负责处理对等发现、交易池、区块和交易闲聊、共识以及响应来自外部世界的 RPC 调用。
这些任务经常需要外部节点查询运行时以获取信息或向运行时提供信息。
运行时 API 促进了外部节点和运行时之间这种通信。

在 Substrate 中，`sp_api` 板条箱提供了一个接口来实现运行时 API。
它旨在让您能够使用[`impl_runtime_apis`](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html)
宏来灵活地定义自己的自定义接口。
但是，每个运行时都必须实现[`Core`](https://paritytech.github.io/substrate/master/sp_api/trait.Core.html) 和[`Metadata`](https://paritytech.github.io/substrate/master/sp_api/trait.Metadata.html) 接口。
除了这些必需的接口之外，大多数 Substrate 节点（如节点模板）还实现了以下运行时接口：

- [`BlockBuilder`](https://paritytech.github.io/substrate/master/sp_block_builder/trait.BlockBuilder.html)，用于构建区块所需的功能。
- [`TaggedTransactionQueue`](https://paritytech.github.io/substrate/master/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html)，用于验证交易。
- [`OffchainWorkerApi`](https://paritytech.github.io/substrate/master/sp_offchain/trait.OffchainWorkerApi.html)，用于启用链外操作。
- [`AuraApi`](https://paritytech.github.io/substrate/master/sp_consensus_aura/trait.AuraApi.html)，用于使用循环共识方法生成和验证区块。
- [`SessionKeys`](https://paritytech.github.io/substrate/master/sp_session/trait.SessionKeys.html)，用于生成和解码会话密钥。
- [`GrandpaApi`](https://paritytech.github.io/substrate/master/sp_consensus_grandpa/trait.GrandpaApi.html)，用于将区块最终确定到运行时。
- [`AccountNonceApi`](https://paritytech.github.io/substrate/master/frame_system_rpc_runtime_api/trait.AccountNonceApi.html)，用于查询交易索引。
- [`TransactionPaymentApi`](https://paritytech.github.io/substrate/master/pallet_transaction_payment_rpc_runtime_api/trait.TransactionPaymentApi.html)，用于查询有关交易的信息。
- [`Benchmark`](https://paritytech.github.io/substrate/master/frame_benchmarking/trait.Benchmark.html)，用于估计和测量完成交易所需的执行时间。

## 核心原语

Substrate 还定义了运行时必须实现的核心原语。
Substrate 框架对运行时必须提供给 Substrate 的其他层的內容做出了最少的假设。
但是，必须定义一些数据类型，并且必须满足特定的
接口才能在 Substrate 框架内工作。

这些核心原语是：

- `Hash`：一种对某些数据进行加密摘要编码的类型。通常只是一个 256 位
  数量。

- `DigestItem`：一种必须能够对与共识和更改跟踪相关的许多“硬编码”备选方案进行编码的类型，以及与
  运行时中特定模块相关的任何数量的“软编码”变体。

- `Digest`：一系列 DigestItem。它对轻客户端在区块中需要掌握的所有信息进行编码。

- `Extrinsic`：一种表示区块链外部的单个数据片段的类型，该数据片段被区块链识别。这通常涉及一个或多个签名，以及某种
  编码指令（例如，用于转移资金所有权或调用智能合约）。

- `Header`：一种代表与区块相关的所有信息的类型（以密码方式或其他方式）。它包括父哈希、存储根和外部函数 Trie 根、
  摘要和区块编号。

- `Block`：本质上只是 `Header` 和一系列 `Extrinsic` 的组合，以及
  要使用的哈希算法的规范。

- `BlockNumber`：一种对任何有效区块的所有祖先的总数进行编码的类型。通常是一个
  32 位数量。

## FRAME

[FRAME](/reference/glossary/#frame) 是您作为运行时开发人员可用的最强大的工具之一。
如[Substrate 赋能开发人员](/)中所述，FRAME 是**Framework for Runtime Aggregation of Modularized Entities**的首字母缩写，它包含大量模块和支持库，简化了运行时开发。
在 Substrate 中，这些模块（称为**模块**）为不同的用例提供了可自定义的业务逻辑，以及您可能希望包含在运行时中的功能。
例如，有一些模块提供了用于质押、共识、治理和其他常见活动的业务逻辑框架。

有关可用模块的摘要，请参阅[FRAME 模块](/reference/frame-pallets/)。

除了模块之外，FRAME 还通过以下库和模块提供服务来与运行时进行交互

- [FRAME 系统板条箱 `frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html) 为运行时提供低级类型、存储和函数。

- [FRAME 支持板条箱 `frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html) 是一个 Rust 宏、类型、特性和模块的集合，简化了 Substrate 模块的开发。

- [FRAME 执行模块 `frame_executive`](https://paritytech.github.io/substrate/master/frame_executive/index.html) 协调传入函数调用到运行时中相应
  模块的执行。

下图说明了 FRAME 及其系统、支持和执行模块如何为运行时环境提供服务。

![FRAME 和运行时架构](/media/images/docs/runtime-and-frame.png)

### 使用模块组合运行时

您可以构建基于 Substrate 的区块链，而无需使用 FRAME。
但是，FRAME 模块使您能够使用预定义的组件作为起点来组合自定义运行时逻辑。
每个模块都定义了特定类型、存储项和函数，以实现运行时的特定功能集或功能。

下图说明了如何选择和组合 FRAME 模块来组合运行时。

![使用 FRAME 组合运行时](/media/images/docs/compose-runtime.png)

### 构建自定义模块

除了预构建的 FRAME 模块库之外，您还可以使用 FRAME 库和服务来构建自己的自定义模块。
使用自定义模块，您可以灵活地定义最适合您目的的运行时行为。
由于每个模块都有自己的离散逻辑，因此您可以组合预构建模块和自定义模块来控制区块链提供的功能和功能，并实现您想要的结果。

例如，您可以在运行时中包含[Balances 模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/balances)，以使用其与加密货币相关的存储项和函数来管理代币，但添加自定义逻辑以在账户余额发生变化时调用您编写的模块。

大多数模块都由以下部分的某种组合组成：

- 导入和依赖项
- 模块类型声明
- 运行时配置特性
- 运行时存储
- 运行时事件
- 用于在特定上下文中执行的逻辑的钩子
- 可用于执行交易的函数调用

例如，如果您想定义一个自定义模块，您可以从类似于以下内容的模块的骨架结构开始：

```rust
// Add required imports and dependencies
pub use pallet::*;

#[frame_support::pallet]
pub mod pallet {
 use frame_support::pallet_prelude::*;
 use frame_system::pallet_prelude::*;

 // Declare the pallet type
 // This is a placeholder to implement traits and methods.
 #[pallet::pallet]
 pub struct Pallet<T>(_);

 // Add the runtime configuration trait
 // All types and constants go here.
 #[pallet::config]
 pub trait Config: frame_system::Config { ... }

 // Add runtime storage to declare storage items.
 #[pallet::storage]
 #[pallet::getter(fn something)]
 pub type MyStorage<T: Config> = StorageValue<_, u32>;

 // Add runtime events
 #[pallet::event]
 #[pallet::generate_deposit(pub(super) fn deposit_event)]
 pub enum Event<T: Config> { ... }

 // Add hooks to define some logic that should be executed
 // in a specific context, for example on_initialize.
 #[pallet::hooks]
 impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> { ... }

 // Add functions that are callable from outside the runtime.
 #[pallet::call]
 impl<T:Config> Pallet<T> { ... }
}
```

您可以根据需要组合包含某些或所有部分的模块。
在开始设计和构建自定义运行时时，您将了解有关 FRAME 库和用于定义配置特性、存储项、事件和错误的运行时原语的更多信息，以及如何编写分派到运行时以执行的函数调用。

## 下一步去哪里

现在您已经熟悉了 Substrate 运行时开发的基础知识以及如何使用模块，请探索以下主题和教程以了解更多信息。

- [FRAME 模块](/reference/frame-pallets/)
- [向运行时添加模块](/tutorials/build-application-logic/add-a-pallet)
- [Substrate 的 Rust](/learn/rust-basics/)
- [宏参考](/reference/frame-macros/)
- [在自定义模块中使用宏](/tutorials/build-application-logic/use-macros-in-a-custom-pallet/)
