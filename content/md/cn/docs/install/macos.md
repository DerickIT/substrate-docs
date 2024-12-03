---
title: macOS 开发环境
description: 在 macOS 上为 Substrate 设置本地开发环境。
keywords:
---

您可以在配备 Intel 或 Apple Silicon（M 系列）处理器的 Apple macOS 计算机上安装 Rust 并设置 Substrate 开发环境。

## 在您开始之前

在您在 macOS 上安装 Rust 并设置开发环境之前，请验证您的计算机是否满足以下基本要求：

- 操作系统版本为 10.7 Lion 或更高版本。
- 处理器速度至少为 2Ghz，建议为 3Ghz。
- 内存至少为 8 GB RAM，建议为 16 GB。
- 存储空间至少为 10 GB。
- 宽带互联网连接。

### 支持 Apple Silicon

在构建过程开始之前，必须安装 Protobuf。要安装它，请运行以下命令：

`brew install protobuf`

### 安装 Homebrew

在大多数情况下，您应该使用 Homebrew 在 macOS 计算机上安装和管理软件包。
如果您在本地计算机上尚未安装 Homebrew，则应在继续之前下载并安装它。

要安装 Homebrew：

1. 打开终端应用程序。

1. 通过运行以下命令下载并安装 Homebrew：

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
   ```

1. 通过运行以下命令验证 Homebrew 是否已成功安装：

   ```bash
   brew --version
   ```

   该命令将显示类似于以下内容的输出：

   ```bash
   Homebrew 3.3.1
   Homebrew/homebrew-core (git revision c6c488fbc0f; last commit 2021-10-30)
   Homebrew/homebrew-cask (git revision 66bab33b26; last commit 2021-10-30)
   ```

## 安装

由于区块链需要标准密码学来支持公钥/私钥对的生成和交易签名的验证，因此您还必须有一个提供密码学的软件包，例如 `openssl`。

要在 macOS 上安装 `openssl` 和 Rust 工具链：

1. 打开终端应用程序。

1. 通过运行以下命令确保您拥有 Homebrew 的更新版本：

   ```bash
   brew update
   ```

1. 通过运行以下命令安装 `openssl` 软件包：

   ```bash
   brew install openssl
   ```

1. 下载 `rustup` 安装程序并使用它通过运行以下命令来安装 Rust：

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

1. 按照显示的提示进行默认安装。

1. 通过运行以下命令更新当前 shell 以包含 Cargo：

   ```bash
   source ~/.cargo/env
   ```

1. 通过运行以下命令验证您的安装：

   ```bash
   rustc --version
   ```

1. 通过运行以下命令将 Rust 工具链配置为默认使用最新的稳定版本：

   ```bash
   rustup default stable
   rustup update
   rustup target add wasm32-unknown-unknown
   ```

1. 通过运行以下命令将 `nightly` 版本和 `nightly` WebAssembly (wasm) 目标添加到您的开发环境中：

   ```bash
   rustup update nightly
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

1. 通过运行以下命令验证开发环境的配置：

   ```bash
   rustup show
   rustup +nightly show
   ```

   该命令将显示类似于以下内容的输出：

   ```bash
   # rustup show

   active toolchain
   ----------------

   stable-x86_64-apple-darwin (default)
   rustc 1.61.0 (fe5b13d68 2022-05-18)

   # rustup +nightly show

   active toolchain
   ----------------

   nightly-x86_64-apple-darwin (overridden by +toolchain on the command line)
   rustc 1.63.0-nightly (e71440575 2022-06-02)
   ```

   1. 使用以下命令安装 `cmake`：

   ```
   brew install cmake
   ```

## 编译 Substrate 节点

现在您已经安装了 Rust 并为 Substrate 开发配置了 Rust 工具链，您就可以通过克隆 Substrate **节点模板** 文件并编译 Substrate 节点来完成开发环境的设置。

节点模板提供了一个工作环境，其中包含构建区块链所需的所有最常见功能，而无需任何多余的模块或工具。
为了确保节点模板为您提供一个相对稳定的工作环境来进行实验，建议的最佳实践是从 Substrate Developer Hub 存储库克隆 Substrate 节点模板，而不是从核心 Substrate 存储库克隆。

要编译 Substrate 节点模板：

1. 通过运行以下命令克隆节点模板存储库：

   ```bash
   git clone https://github.com/paritytech/polkadot-sdk-solochain-template
   ```

   在大多数情况下，您可以克隆 `main` 分支以获取最新的代码。
   但是，如果您想使用与特定 Polkadot 版本兼容的 Substrate 分支，则可以使用 `--branch` 命令行选项。
   单击 [标签](https://github.com/paritytech/polkadot-sdk-solochain-template/tags) 以查看与特定 Polkadot 版本兼容的分支列表。

1. 通过运行以下命令更改到节点模板目录的根目录：

   ```bash
   cd substrate-solo-template-node
   ```

   如果您想保存更改并使此分支易于识别，则可以通过运行类似于以下内容的命令来创建一个新分支：

   ```bash
   git switch -c my-wip-branch
   ```

1. 通过运行以下命令编译节点模板：

   ```bash
   cargo build --release
   ```

   由于需要大量软件包，因此编译节点可能需要几分钟。

构建成功完成后，您的本地计算机将准备好进行 Substrate 开发活动。

## 下一步去哪里

Substrate Developer Hub 充当访问社区提供的众多资源的中心门户。
根据您的兴趣和学习风格，您可能更喜欢一条途径而不是另一条途径。
例如，如果您更喜欢阅读源代码并且熟悉 Rust，您可能想从深入研究 [Rust API](https://paritytech.github.io/substrate/master) 开始。

<!-- TODO NAV.YAML -->
<!-- add these back -->
<!--如果您不熟悉 Substrate 和 Substrate 生态系统，您可能希望通过查看 [探索](/explore/) 来更广泛地了解可用的资源以及在哪里可以找到它们。-->

以下是一些有关您可以在哪里了解更多信息的额外建议。

#### 告诉我

- [架构](/learn/architecture/)
- [网络和区块链](/learn/node-and-network-types/)
- [构建过程](/build/build-process)

#### 指导我

- [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)
- [模拟网络](/tutorials/build-a-blockchain/simulate-network/)
- [添加可信节点](/tutorials/build-a-blockchain/add-trusted-nodes/)
