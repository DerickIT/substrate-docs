---
title: 探索代码
description: 仔细查看节点模板的内容。
keywords:
---

在 [启动节点](/quick-start/start-a-node/) 中，您在开发模式下编译并启动了本地 Substrate 节点。
此特定节点（`substrate-node-template`）提供了一个简化的环境，其中只有几个常用模块可帮助您入门。
无需深入了解细节，从探索节点模板代码的基本构建块中，您可以学到很多东西。

## 关于节点模板

节点模板包含一些默认的区块链基本要素，例如点对点网络、简单的共识机制和交易处理。
节点模板还包含一些用于处理帐户、余额和交易费用以及执行管理操作的基本功能。
这套核心功能是通过几个预定义的模块（称为**调色板**）提供的，这些模块实现了特定功能。

例如，以下核心模块在节点模板中预定义：

- `pallet_balances` 用于管理帐户资产和帐户之间的转账。
- `pallet_transaction_payment` 用于管理执行的交易的交易费用。
- `pallet_sudo` 用于执行需要管理权限的操作。

节点模板还提供了一个入门 `pallet_template`，它说明了如何在自定义调色板中实现功能。

现在您已经概述了节点模板中包含的功能，让我们更仔细地查看 `substrate-node-template` 目录及其子目录中的代码。

## 清单文件

由于 Substrate 是一个基于 Rust 的框架，因此每个包都有一个清单文件（`Cargo.toml` 文件），其中包含编译包所需的信息。
如果您打开位于 `substrate-node-template` 根目录中的 `Cargo.toml` 文件，您会发现它描述了构成节点模板工作区的成员包。
例如：

```toml
[workspace]
members = [
    "node",
    "pallets/template",
    "runtime",
]
[profile.release]
panic = "unwind"
```

从此清单中，您可以看到节点模板工作区包含三个包：

- `node` 包为许多核心区块链服务（如点对点网络、区块创作、区块终结和交易池管理）提供 Rust 模块。
- `pallets` 子目录中的 `template` 包是入门模板，说明了在构建自己的自定义模块时如何实现功能。
- `runtime` 包为处理帐户、余额、交易费用以及已包含在节点模板中的其他功能提供所有应用程序逻辑。

每个成员包也有自己的清单（自己的 `Cargo.toml` 文件），其中包含特定于包的信息，包括依赖项和配置设置，这些信息是编译该成员包所必需的。
例如，工作区 `node` 成员的 `Cargo.toml` 文件将包的名称指定为 `node-template`，并列出了使节点模板能够提供基本区块链服务的核心库和基元。
您将在 [架构和 Rust 库](/learn/architecture) 中详细了解库和基元。
现在，了解清单在描述每个包的依赖项和其他关键信息方面的重要性就足够了。

如果您打开 `runtime/Cargo.toml` 文件和 `pallets/template/Cargo.toml`，您将看到不同的库和基元作为依赖项，但您会对编译这些包需要什么有一个大致的了解。
例如，运行时的清单列出了所有调色板（包括 `frame_system` 、 `frame_support` 和前面提到的 `pallet_balances` 、 `pallet_transaction_payment` 和 `pallet_sudo` 模块），这些模块构成了节点模板的默认运行时。

## 核心客户端源代码

Substrate 最重要的方面之一是节点由两个主要部分组成：**核心客户端**和**运行时**。
节点模板还包含核心客户端服务的单独包（在 `node/src` 目录中）和 `runtime` 包（在 `runtime/src` 目录中）。

默认情况下，`node/src` 目录包含以下 Rust 模块：

- `benchmarking.rs`
- `chain_spec.rs`
- `cli.rs`
- `command.rs`
- `lib.rs`
- `main.rs`
- `rpc.rs`
- `service.rs`

大多数核心客户端服务都封装在 `node/src/service.rs` Rust 模块中。
您很少需要修改此文件或 `node/src` 目录中的其他 Rust 模块。
您可能会修改的文件是 `chain_spec.rs` 文件。
`chain_spec.rs` 文件描述了默认开发和本地测试网络链的配置，包括有关默认预资助开发帐户以及预配置为具有创建区块权限的节点的信息。
如果您创建自定义链，则使用此文件来识别节点连接到的网络以及本地节点与之通信的其他节点。

## 默认节点模板运行时

由于 Substrate 为构建区块链提供了一个模块化且灵活的框架，因此您可以更改工作区中的任何包。
但是，大多数应用程序开发工作都在运行时和用于构建运行时的模块（调色板）中完成。
在开始为自己的项目自定义运行时之前，您应该花一些时间探索默认节点模板中的内容。

### 默认清单

您已经了解了运行时的默认清单如何在类似于以下内容的行中列出运行时的默认依赖项和功能：

```rust
pallet-balances = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }

pallet-sudo = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }

pallet-transaction-payment = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }
```

还有一些对核心包的依赖项，例如 `frame-system` 、 `frame-support` 和 `frame-executive` 。
您将在 [核心 FRAME 服务](/learn/runtime-development/#core-frame-services) 中详细了解这些核心服务。
现在，只需注意这些和其他模块是编译节点模板的运行时所必需的。

### 默认源代码

运行时的主要源代码位于 `runtime/src/lib.rs` 文件中。
如果您在代码编辑器中打开此文件，一开始可能会觉得很复杂。
有些细微差别在文档的其他部分有介绍，但本质上，源代码执行以下操作：

- 导入 frame_system 和 frame_support 核心服务。
- 指定运行时的版本信息。
- 声明要包含的调色板。
- 声明每个包含的调色板的类型和参数。
- 为每个包含的调色板设置常量和变量值。
- 为每个包含的调色板实现 `Config` 特性。
- 从包含的调色板构建运行时。
- 为评估调色板性能准备基准测试框架。
- 实现允许核心客户端调用运行时的接口。

您将在 [构建](/build/) 和 [测试](/test/) 部分的主题中详细了解如何构建运行时、定义基准测试和使用运行时接口。
现在，您只需要对运行时的组成方式以及如何使用 `Config` 特性实现默认调色板有一个大致的了解。
