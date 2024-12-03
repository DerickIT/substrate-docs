---
title: 启动节点
description: 从模板启动您的第一个 Substrate 节点。
keywords:
---

所有 Substrate 教程和操作指南都需要您在开发环境中构建和运行 Substrate 节点。
为了帮助您快速设置工作环境，[Substrate 开发者中心](https://github.com/substrate-developer-hub/) 为您维护了_模板_。
例如，[substrate-node-template](https://github.com/paritytech/polkadot-sdk-solochain-template/tags/) 是主要 Substrate `node-template` 二进制文件的快照，其中包含一组核心功能，可帮助您入门。

启动节点后，您可以使用 Web 浏览器和一个简单的应用程序连接到它，该应用程序允许您查找预定义帐户的余额。

## 在您开始之前

在您开始之前，请验证以下内容：

- 您有互联网连接，并且可以访问本地计算机上的交互式外壳终端。

- 您通常熟悉软件开发和使用命令行界面。

- 您已安装 Rust 编译器和工具链。

  您可以通过运行 `rustup show` 命令来检查是否已安装 Rust。
  如果已安装 Rust，则此命令将显示工具链和编译器的版本信息。
  如果未安装 Rust，则该命令不会返回任何输出。
  有关安装 Rust 的信息，请参阅 [安装](/install)。

## 构建节点模板

1. 通过运行以下命令克隆节点模板存储库：

   ```sh
   git clone https://github.com/paritytech/polkadot-sdk-solochain-template
   ```

   此命令将克隆 `main` 分支。

   或者，您可以使用 `--branch` 命令行选项和一个 [标签](https://github.com/paritytech/polkadot-sdk-solochain-template/tags) 来指定您希望节点与之兼容的 Polkadot 版本。

2. 更改到克隆目录的根目录：

   ```sh
   cd polkadot-sdk-solochain-template
   ```

3. 通过运行类似于以下内容的命令来创建一个新分支以保存您的工作：

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   您可以使用您选择的任何标识信息来命名分支。
   在大多数情况下，您应该在名称中包含有关您克隆分支的年-月-日的的信息。
   例如：

   ```text
   git switch -c my-learning-branch-2023-03-31
   ```

4. 编译节点模板：

   ```sh
   cargo build --release
   ```

   第一次编译节点时，可能需要一些时间才能完成。
   编译完成后，您应该看到类似于以下内容的行：

   ```bash
   Finished release [optimized] target(s) in 11m 23s
   ```

## 查看节点信息

1. 验证您的节点是否已准备好使用，并通过运行以下命令查看有关可用命令行选项的信息：

   ```sh
   ./target/release/solochain-template-node --help
   ```

   使用信息将显示您可以用来执行以下操作的命令行选项：

   - 启动节点
   - 使用帐户和密钥
   - 修改节点操作

1. 通过运行以下命令查看预定义的 `Alice` 开发帐户的帐户信息：

   ```sh
   ./target/release/solochain-template-node key inspect //Alice
   ```

   该命令将显示以下帐户和地址信息：

   ```text
   Secret Key URI `//Alice` is account:
   Network ID:        substrate
   Secret seed:       0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
   Public key (hex):  0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
   Account ID:        0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
   Public key (SS58): 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
   SS58 Address:      5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
   ```

   `Alice` 和 `Bob` 等预定义的开发帐户在**链规范**文件中配置。
   您将在 [探索代码](/quick-start/explore-the-code/) 中详细了解节点模板文件，并在 [链规范](/build/chain-spec/) 中更详细地了解链规范文件。
   现在，了解开发帐户的存在以启用对简单交易（如余额转账）的测试就足够了。

## 启动区块链

1. 通过运行以下命令在开发模式下启动节点：

   ```sh
   ./target/release/solochain-template-node --dev
   ```

   在开发模式下，链不需要任何对等计算机来终结区块。
   节点启动时，终端将显示有关执行的操作的输出。
   如果您看到有关正在提出和终结区块的消息，则说明您有一个正在运行的节点。

   ```text
   ... Idle (0 peers), best: #3 (0xcc78…5cb1), finalized #1 ...
   ... Starting consensus session on top of parent ...
   ... Prepared block for proposing at 4 (0 ms) ...
   ```

## 连接到节点

现在您的节点正在运行，您可以连接到它以检查预定义的 `Alice` 帐户的余额。
对于此简单应用程序，您可以创建一个 `index.html` HTML 文件，该文件使用 JavaScript 和 [Polkadot-JS API](https://polkadot.js.org/docs/api) 与区块链交互。

例如，此示例 [index.html](/assets/quickstart/index.html) 演示了如何使用 JavaScript、Polkadot-JS API 和 HTML 来执行以下操作：

- 将帐户地址作为输入。
- 使用 `onClick` 事件查找帐户余额。
- 将帐户的余额作为输出显示。

要连接到节点并检查帐户余额：

1. 将 [示例代码](/assets/quickstart/index.html)（用于**快速入门：获取余额**应用程序）复制并粘贴到代码编辑器中的新文件中，并将该文件保存在本地计算机上。

2. 在 Web 浏览器中打开 `index.html` 文件。

3. 将 `Alice` 帐户的 SS58 地址复制并粘贴到输入字段中，然后单击**获取余额**。

## 停止节点

1. 转到显示区块链操作的终端。

1. 按 `control-c` 键组合停止本地区块链并清除所有状态。
