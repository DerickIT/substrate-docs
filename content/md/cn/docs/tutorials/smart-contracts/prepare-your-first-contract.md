---
title: 准备您的第一个合约
description: 使用 ink!智能合约语言构建和测试简单的智能合约。
keywords:
---

正如您在[区块链基础知识](/main-docs/learn/blockchain-basics/)中学到的那样，去中心化应用程序通常被编写为**智能合约**。

虽然 Substrate 主要是一个用于构建自定义区块链的框架和工具包，但它也可以为智能合约提供平台。

本教程演示了如何构建一个基本的智能合约以在基于 Substrate 的链上运行。

在本教程中，您将探索如何使用 ink!作为编写基于 Rust 的智能合约的编程语言。

## 开始之前

在开始之前，请验证以下内容：

- 您拥有良好的互联网连接，并且可以访问本地计算机上的 shell 终端。

- 您通常熟悉软件开发和使用命令行界面。

- 您通常熟悉区块链和智能合约平台。

- 您已按照[安装](/main-docs/install/)中的说明安装 Rust 并设置了开发环境。

## 教程目标

通过完成本教程，您将实现以下目标：

- 学习如何创建智能合约项目。

- 使用 ink!智能合约语言构建和测试智能合约。

- 在本地 Substrate 节点上部署智能合约。

- 使用`cargo-contract`CLI 与智能合约进行交互。

## 更新您的 Rust 环境

对于本教程，您需要向 Substrate 开发环境添加一些 Rust 源代码。

要更新您的开发环境：

1. 在您的计算机上打开终端 shell。

1. 通过运行以下命令来更新您的 Rust 环境：

   ```bash
   rustup component add rust-src
   ```

1. 通过运行以下命令验证您是否已安装 WebAssembly 目标：

   ```bash
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

   如果目标已安装并已更新，则该命令将显示类似于以下内容的输出：

   ```text
   info: component 'rust-std' for target 'wasm32-unknown-unknown' is up to date
   ```

## 安装`cargo-contract`CLI 工具

`cargo-contract`是一个命令行工具，您将使用它来构建、部署和与 ink!合约进行交互。

请注意，除了 Rust 之外，安装`cargo-contract`还需要一个支持 C++17 的 C++编译器。

`gcc`、`clang`以及 Visual Studio 2019+ 的现代版本应该都可以工作。

1. 添加`rust-src`编译器组件：

```bash
rustup component add rust-src
```

2. 安装最新版本的`cargo-contract`：

```bash
cargo install --force --locked cargo-contract --version 2.0.0-rc
```

3. 通过运行以下命令验证安装情况并浏览可用的命令：

```bash
cargo contract --help
```

## 安装 Substrate 合约节点

为了简化本教程，您可以[下载](https://github.com/paritytech/substrate-contracts-node/releases)适用于 Linux 或 macOS 的预编译 Substrate 节点。

预编译的二进制文件默认包含用于智能合约的 FRAME 模块。

要在 macOS 或 Linux 上安装合约节点：

1. 打开[发行版](https://github.com/paritytech/substrate-contracts-node/releases)页面。

1. 下载适合您本地计算机的相应压缩存档。

1. 打开下载的文件并将内容解压缩到工作目录。

如果您无法下载预编译的节点，则可以使用类似于以下命令的命令在本地编译它。您可以在[发行版](https://github.com/paritytech/substrate-contracts-node/releases)页面上找到最新的标签：

```bash
cargo install contracts-node --git https://github.com/paritytech/substrate-contracts-node.git --tag <latest-tag> --force --locked
```

您可以在[标签](https://github.com/paritytech/substrate-contracts-node/tags)页面上找到要使用的最新标签。

您可以通过运行`substrate-contracts-node --version`来验证安装情况。

## 创建新的智能合约项目

您现在可以开始开发新的 ink!智能合约项目了。

要生成 ink!项目的文
