---
title: 熟悉环境
description:
tutorial: 1
code: |

answer: |
---

如果您完成了[准备工作环境](/tutorials/collectibles-workshop/01-prepare/)中的清单，则您已成功编译了 Substrate 节点。
此特定节点（substrate-node-template）预先配置了一些常见的模块，以提供简单的可用环境来帮助您入门。

## 关于节点模板

节点模板包含一些默认的区块链基本要素，例如点对点网络、区块创建、共识以及已完成区块的区块导入。
节点模板还包含一些用于处理帐户和事务以及执行管理操作的基本功能。
此核心功能集是通过几个预定义的模块（称为模块）提供的，这些模块实现了特定功能。

节点模板中预定义了以下核心模块：

- `pallet_balances`用于管理帐户资产和帐户之间的转移。
- `pallet_transaction_payment`用于管理执行的事务的交易费用。
- `pallet_sudo`用于执行需要管理权限的操作。

节点模板还提供了一个入门级`pallet_template`，它说明了如何在自定义模块中实现功能。
您在本研讨会中将要执行的大部分操作都类似于您在`pallet_template`中看到的内容，只是您将实现特定于您正在构建的应用程序的逻辑。

## 重命名您的工作区

为了区分默认的 Substrate 节点模板和您将用于自定义节点的工作区，您可以重命名工作目录并为您的更改创建单独的分支。

要准备自定义工作区：

1. 在本地计算机上打开终端，然后导航到包含`substrate-node-template`根目录的目录。

2. 通过运行以下命令重命名`substrate-node-template`根目录：

   ```bash
   mv substrate-node-template workshop-node-template
   ```

   在后续步骤中，`workshop-node-template`目录用于指代节点的根目录。

3. 通过运行以下命令更改到节点的根目录：

   ```bash
   cd workshop-node-template
   ```

4. 通过运行类似于以下命令的命令为您的工作区创建一个自定义分支：

   ```bash
   git switch -c build-substrate-workshop
   ```

   因为您只是更改了工作区文件夹和存储库分支的名称，所以您不必重新编译节点。
   如果您想通过重新编译来验证构建，请运行以下命令：

   ```bash
   cargo build --release
   ```

## 以开发模式启动节点

重命名工作区后，您可以以开发模式启动节点并让它在后台运行。
在开发模式下，节点使用为开发环境预定义的一些默认设置。
开发环境的一些默认设置包括：

- 若干预定义的帐户，包括 Alice、Bob、Charlie 和 Dave。
- Alice 和 Bob 帐户的初始帐户余额。
- Alice 帐户的根级权限。
- 使用 Alice 帐户预定义的权限来创建和验证区块。

默认情况下，开发模式还会将链数据存储在临时数据库中，并在您停止节点时删除该数据。

要启动本地 Substrate 节点：

1. 根据需要打开终端 shell。

2. 更改到`workshop-node-template`根目录。

3. 通过运行以下命令以开发模式启动节点：

   ```bash
   ./target/release/node-template --dev
   ```

   在开发模式下，链不需要任何对等计算机来完成区块。
   在一些初始化消息之后，您应该会看到正在提出、密封和完成区块。
   例如：

   ```text
   2022-10-12 13:07:18 Substrate Node
   2022-10-12 13:07:18 ✌️  version 4.0.0-dev-6c701fa6109
   2022-10-12 13:07:18 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2022
   2022-10-12 13:07:18 📋 Chain specification: Development
   2022-10-12 13:07:18 🏷  Node name: cooperative-clocks-6375
   2022-10-12 13:07:18 👤 Role: AUTHORITY
   ...
   2022-10-12 13:07:24 🙌 Starting consensus session on top of parent 0x72d1778c51e4c624e39d20bd3fac5d2c800a424fbd3bde08f4863bdf90966bf9
   2022-10-12 13:07:24 🎁 Prepared block for proposing at 1 (1 ms) [hash: 0xdc23986196af67f6b07a78c50cd7b42c24b2e8da2d9f8ebf81c2540c358ada06; parent_hash: 0x72d1…6bf9; extrinsics (1): [0xd047…45da]]
   2022-10-12 13:07:24 🔖 Pre-sealed block for proposal at 1. Hash now 0x0e2c8238d12d4babc83881384f6f178aa2996df5083387070c3d726bb7b10aec, previously 0xdc23986196af67f6b07a78c50cd7b42c24b2e8da2d9f8ebf81c2540c358ada06.
   2022-10-12 13:07:24 ✨ Imported #1 (0x0e2c…0aec)
   ```

   您现在拥有一个正在生成区块的运行中区块链。
   您可以随时将其保留在后台运行，或者停止并重新启动它。

   - 要停止节点，请按 Control-c。
   - 要重新启动节点，请运行`./target/release/node-template --dev`。

   下一步是开始自定义此区块链的功能。

   正如您在[准备工作环境](/tutorials/collectibles-workshop/01-prepare/)中学到的那样，Substrate 是使用 Rust 编程语言构建的。
   因此，自定义区块链的功能意味着在 Rust 中工作。

   如果您熟悉 Rust，您可以直接转到[创建新的模块](/tutorials/collectibles-workshop/03-create-pallet/)并开始构建。
   如果您不熟悉 Rust，则可能需要在开始编写代码之前查看[绕行：学习用于 Substrate 的 Rust](/tutorials/collectibles-workshop/detours/learn-rust/)中的简要介绍。
