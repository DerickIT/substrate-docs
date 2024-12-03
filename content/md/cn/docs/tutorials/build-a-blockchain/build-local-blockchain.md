---
title: 构建本地区块链
description: 在开发环境中编译、启动和与单个本地区块链节点交互。
keywords:
---

正如您在[区块链基础知识](/learn/blockchain-basics/)中学到的那样，区块链由称为节点的去中心化计算机组成以形成网络。

Substrate 提供了一个灵活、开放和可扩展的开发环境，允许您设计和构建完全定制的区块链节点，以满足您的应用程序或业务模型需求。

## 开始

成为区块链开发人员的第一步是学习如何编译和启动单个本地区块链节点。
在本教程中，您将使用节点模板构建和启动单个节点区块链。

Substrate 节点模板提供了一个可工作的单节点区块链，您可以在本地开发环境中运行它。
节点模板包含几个预定义的组件——例如用户帐户和帐户余额——以便您可以尝试执行常见任务。
无需对模板进行任何更改，您就可以运行一个功能性节点，该节点可以生成区块并允许交易。

启动本地区块链节点后，本教程将演示如何使用 Substrate 前端模板查看有关区块链活动的信息并提交交易。

### 本教程面向谁？

本教程提供了对 Substrate 的基本介绍，并准备了一个最小的可工作开发环境，您可以使用它在其他教程中进行进一步探索。
它适用于任何对学习 Substrate 和区块链开发感兴趣的人。
本教程假设您没有任何关于 Substrate 的先前经验或知识。
您无需任何编程或区块链经验即可完成本教程。
这只是第一步，但希望它能激励您继续您的旅程。

### 完成本教程需要多长时间？

本教程需要编译 Rust 代码，大约需要一到两个小时才能完成。

## 开始之前

对于本教程，您将下载并使用可工作的代码。在开始之前，请验证以下内容：

- 您拥有良好的互联网连接，并且可以访问本地计算机上的shell终端。

- 您通常熟悉软件开发和使用命令行界面。

- 您通常熟悉区块链和智能合约平台。

- 您已按照[安装](/install/)中的说明安装 Rust 并设置了开发环境。

## 教程目标

通过完成本教程，您将实现以下目标：

- 编译节点模板并启动基于 Substrate 的本地区块链。

- 安装前端模板以与本地区块链节点交互。

- 使用前端模板提交交易并查看结果。

## 编译Substrate节点

[Substrate 节点模板](https://github.com/paritytech/polkadot-sdk-solochain-template)提供了一个可工作的开发环境，以便您可以立即开始在Substrate上构建。

如果您已经在本地计算机上编译了节点模板，则可以跳过此部分，继续进行[启动本地节点](#start-the-local-node)。

要编译 Substrate 节点模板：

1. 在您的计算机上打开终端shell。

1. 通过运行以下命令克隆节点模板存储库：

   ```bash
   git clone https://github.com/paritytech/polkadot-sdk-solochain-template
   ```

   此命令克隆`main`分支。
   如果您熟悉Polkadot，并且想要使用特定Polkadot版本，则可以使用`--branch`命令行选项来选择要使用的特定分支。

   点击[Releases](https://github.com/paritytech/polkadot/releases)或[Tags](https://github.com/paritytech/polkadot/tags)查看您可以与节点模板一起使用的最新和早期Polkadot分支。

2. 通过运行以下命令更改到节点模板目录的根目录：

   ```bash
   cd substrate-solo-template-node
   ```

   创建一个新分支来包含您的工作：

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   将`yyyy-mm-dd`替换为您想要的任何标识信息，但我们建议使用数字年-月-日格式。例如：

   ```bash
   git switch -c my-learning-branch-2023-03-01
   ```

3. 通过运行以下命令编译节点模板：

   ```bash
   cargo build --release
   ```

   您应该始终使用`--release`标志来构建优化的工件。
   第一次编译时，需要一些时间才能完成。

   它应该以类似于以下内容的行完成：

   ```bash
   Finished release [optimized] target(s) in 11m 23s
   ```

## 启动本地节点

节点编译完成后，您可以使用前端模板开始探索它的功能。

要启动本地 Substrate 节点：

1. 在您编译节点的同一终端中，您现在可以通过运行以下命令以开发模式启动节点：

   ```bash
   ./target/release/solo-template-node --dev
   ```

   `solo-template-node`命令行选项指定您希望运行的节点如何运行。
   在本例中，`--dev`选项指定节点使用预定义的`development`链规范在开发模式下运行。
   默认情况下，此选项还会在您按Control-c停止节点时删除所有活动数据——例如密钥、区块链数据库和网络信息。
   使用`--dev`选项可确保您在每次停止和重新启动节点时都拥有干净的工作状态。

2. 通过查看终端中显示的输出，验证您的节点是否已成功启动并运行。

   终端应显示类似于以下内容的输出：

   ```text
   2022-08-16 13:43:58 Substrate Node    
   2022-08-16 13:43:58 ✌️  version 4.0.0-dev-de262935ede    
   2022-08-16 13:43:58 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2022    
   2022-08-16 13:43:58 📋 Chain specification: Development
   2022-08-16 13:43:58 🏷  Node name: limping-oatmeal-7460    
   2022-08-16 13:43:58 👤 Role: AUTHORITY    
   2022-08-16 13:43:58 💾 Database: RocksDb at /var/folders/2_/g86ns85j5l7fdnl621ptzn500000gn/T/substrate95LPvM/chains/dev/db/full    
   2022-08-16 13:43:58 ⛓  Native runtime: solo-template-node-100 (solo-template-node-1.tx1.au1)
   2022-08-16 13:43:58 🔨 Initializing Genesis block/state (state: 0xf6f5…423f, header-hash: 0xc665…cf6a)
   2022-08-16 13:43:58 👴 Loading GRANDPA authority set from genesis on what appears to be first startup.
   2022-08-16 13:43:59 Using default protocol ID "sup" because none is configured in the chain specs
   2022-08-16 13:43:59 🏷  Local node identity is: 12D3KooWCu9uPCYZVsayaCKLdZLF8CmqiHkX2wHsAwSYVc2CxmiE
   ...
   ...
   ...
   ...
   2022-08-16 13:54:26 💤 Idle (0 peers), best: #3 (0xcdac…26e5), finalized #1 (0x107c…9bae), ⬇ 0 ⬆ 0
   ```

   如果`finalized`后面的数字在增加，则您的区块链正在生成新区块并就它们描述的状态达成共识。

   我们将在后面的教程中探讨日志输出的详细信息。
   目前，重要的是要知道您的节点正在运行并生成区块。

3. 保持显示节点输出的终端打开以继续。

## 安装前端模板

前端模板使用[ReactJS](https://reactjs.org/)来呈现一个Web浏览器界面，使您可以与基于Substrate的区块链节点交互。
您可以将此前端模板作为起点，在将来为自己的项目创建用户界面。

前端模板需要[Yarn](https://yarnpkg.com/)和[Node.js](https://nodejs.org/)。如果您没有这些工具，请先安装它们。

要安装前端模板：

1. 在您的计算机上打开一个新的终端窗口。

2. 通过运行以下命令检查您的本地计算机上是否安装了`node`：

   ```bash
   node --version
   ```

   如果您已安装`node`，则此命令将返回一个版本号，例如`v19.7.0`。

   如果该命令没有返回版本号，请按照[Node.js](https://nodejs.org/)网站上您使用的操作系统的说明下载并安装`node`，或者通过安装和使用[`nvm`](https://github.com/nvm-sh/nvm)节点版本管理器来安装和使用它。

   要运行前端模板，`node`版本至少应为**v14**。

3. 通过运行以下命令检查您的本地计算机上是否安装了`yarn`：

   ```bash
   yarn --version
   ```

   要运行前端模板，`yarn`版本至少应为**v3**。
   如果您安装了较旧的版本，则可以使用`yarn version`并指定版本号来更新`yarn`。
   如果该命令没有返回版本号，或者如果您对安装特定`yarn`版本有任何疑问，请按照[Yarn](https://yarnpkg.com/getting-started/install)网站上的说明安装`yarn`。

4. 通过运行以下命令克隆前端模板存储库：

   ```bash
   git clone https://github.com/substrate-developer-hub/substrate-front-end-template
   ```

5. 通过运行以下命令更改到前端模板目录的根目录：

   ```bash
   cd substrate-front-end-template
   ```

6. 通过运行以下命令安装前端模板的依赖项：

   ```bash
   yarn install
   ```

## 启动前端模板

Substrate 前端模板由用户界面组件组成，使您可以与 Substrate 节点交互并执行一些常见任务。

要使用前端模板：

1. 验证您的当前工作目录是您在上节中安装前端模板的根目录。

1. 通过运行以下命令启动前端模板：

   ```bash
   yarn start
   ```

   通常，运行`yarn start`命令会自动在您的默认浏览器中打开`http://localhost:8000`。
   如有必要，您可以手动指定`http://localhost:8000` URL 来查看前端模板。

   顶部部分有一个帐户选择列表，用于在您想要执行链上操作时选择要使用的帐户。
   模板的顶部部分还显示有关您连接到的链的信息。

   ![前端模板顶部部分](/media/images/docs/tutorials/build-local-blockchain/blockchain-top.png)

   您可能还会注意到，前端模板显示了一个带有某些预定义帐户的余额表，并且其中一些帐户预先配置了资金。
   您可以使用此示例数据来尝试诸如转移资金之类的操作。

   ![预定义帐户和余额](/media/images/docs/tutorials/build-local-blockchain/balances.png)

## 从帐户转移资金

现在您已经在本地计算机上运行了一个区块链节点，并且可以使用前端模板执行链上操作，您可以开始探索与区块链交互的不同方法。

默认情况下，前端模板包含几个组件，允许您尝试不同的常见任务。
在本教程中，您可以执行一个简单的转移操作，将资金从一个帐户转移到另一个帐户。

要将资金转移到帐户：

1. 在余额表中，注意预定义的帐户（例如dave），这些帐户没有与其关联的资金。

   ![找到一个资金为零的帐户](/media/images/docs/tutorials/build-local-blockchain/dave-account.png)

   在余额表下方，前端模板还显示一个**转移**组件。
   您可以使用此组件将资金从一个帐户转移到另一个帐户。

1. 从可用帐户列表中选择**dave**以填充您要向其转移资金的地址。

1. 指定至少**1000000000000**作为要转移的金额，然后点击**提交**。

   ![将资金转移到帐户](/media/images/docs/tutorials/build-local-blockchain/transfer-filled.png)

1. 请注意，余额表中的值已使用转移进行了更新。

   ![显示更新后的余额](/media/images/docs/tutorials/build-local-blockchain/balance-result.png)

1. 检查**事件**组件以查看与您刚刚完成的转移相关的事件。

   Substrate 区块链将异步操作的结果报告为事件，因此您可以使用事件组件查看作为转移一部分执行的每个操作的详细信息。
   例如：

   ![作为异步操作结果记录的事件](/media/images/docs/tutorials/build-local-blockchain/event-panel.png)

1. 当事务已完成并包含在一个区块中时，您会看到类似于以下内容的确认消息：

   😉 Finalized. Block hash: 0xa50d3f99fcea8a1611806895aa3f4d4d55fdc4989fbb2148d4856a043d01f808

## 停止本地节点

成功转移后，您可以继续探索前端模板组件或停止本地 Substrate 节点以清除您所做的任何状态更改。
因为您在启动节点时指定了`--dev`选项，所以停止本地节点会停止区块链并清除所有持久性区块数据，以便您下次启动节点时可以从干净的状态开始。

要停止本地 Substrate 节点：

1. 返回到显示节点输出的终端shell。

2. 按Control-c终止正在运行的进程。

3. 验证您的终端是否已返回到`substrate-solo-template-node`目录中的终端提示符。

## 下一步

恭喜！

在本教程中，您学习了：

- 如何使用节点模板启动基于 Substrate 的可工作区块链节点。

- 如何使用前端用户界面查看和与区块链节点交互。

- 如何将资金从一个帐户简单地转移到另一个帐户。

前端模板包含几个额外的组件，供您在连接到本地开发节点时进行试验。
您可以自己探索这些组件，或者在以下主题中了解更多信息：

- [架构](/learn/architecture/)
- [网络和区块链](/learn/node-and-network-types/)
- [模拟网络](/tutorials/build-a-blockchain/simulate-network/)
- [提交问题](https://github.com/substrate-developer-hub/substrate-docs/issues/new/choose)
- [Substrate Stack Exchange](https://substrate.stackexchange.com/)
