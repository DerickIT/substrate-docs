---
title: Windows 开发环境
description: 在 Windows 上为 Substrate 设置本地开发环境。
keywords:
---

通常，UNIX 类操作系统（如 macOS 或 Linux）为构建基于 Substrate 的区块链提供了更好的开发环境。
Substrate [教程](/tutorials/) 和 [操作指南](/reference/how-to-guides/) 中的所有代码示例和命令行说明都说明了如何在终端中使用与 UNIX 兼容的命令与 Substrate 进行交互。

但是，如果您的本地计算机使用 Microsoft Windows 而不是 UNIX 类操作系统，则可以使用其他软件对其进行配置，使其成为构建基于 Substrate 的区块链的合适开发环境。
要在运行 Microsoft Windows 的计算机上准备开发环境，可以使用 Windows Subsystem for Linux (WSL) 来模拟 UNIX 操作环境。

## 在您开始之前

在 Microsoft Windows 上安装之前，请验证以下基本要求：

- 您有一台运行受支持版本的 Microsoft Windows 操作系统的计算机。
- 您必须运行 Microsoft Windows 10 版本 2004 或更高版本，或 Microsoft Windows 11 才能在配备 Windows 桌面操作系统的计算机上安装 Windows Subsystem for Linux。
- 您必须运行 Microsoft Windows Server 2019 或更高版本才能在配备 Windows 服务器操作系统的计算机上安装 Windows Subsystem for Linux。
- 您拥有良好的互联网连接，并且可以访问本地计算机上的 shell 终端。

## 设置 Windows Subsystem for Linux

Windows Subsystem for Linux (WSL) 使您能够在使用 Windows 操作系统的计算机上模拟 Linux 环境。
对于 Substrate 开发，这种方法的主要优势是您可以使用 Substrate 文档中描述的所有代码和命令行示例。
例如，您可以运行常见的命令（例如 `ls` 和 `ps`），而无需修改。
通过使用 Windows Subsystem for Linux，您可以避免配置虚拟机映像或双启动操作系统。

要使用 Windows Subsystem for Linux 准备开发环境：

1. 检查您的 Windows 版本和内部版本号，以查看 Windows Subsystem for Linux 是否默认启用。

   如果您拥有 Microsoft Windows 10 版本 2004（内部版本 19041 及更高版本）或 Microsoft Windows 11，则 Windows Subsystem for Linux 默认可用，您可以继续执行下一步。

   如果您安装了较旧版本的 Microsoft Windows，请参阅 [WSL 手动安装步骤（适用于较旧版本）](https://docs.microsoft.com/en-us/windows/wsl/install-manual)。
   如果您要安装在较旧版本的 Microsoft Windows 上，则可以下载并安装 WLS 2，前提是您的计算机拥有 Windows 10 版本 1903 或更高版本。

1. 从开始菜单中选择 Windows PowerShell 或命令提示符，右键单击，然后选择 **以管理员身份运行**。

1. 在 PowerShell 或命令提示符终端中，运行以下命令：

   ```bash
   wsl --install
   ```

   此命令将启用 Windows 操作系统中包含的必需 WSL 2 组件，下载最新的 Linux 内核，并默认安装 Ubuntu Linux 发行版。

   如果您想查看可用的其他 Linux 发行版，请运行以下命令：

   ```bash
   wsl --list --online
   ```

1. 下载发行版后，关闭终端。

1. 单击开始菜单，选择 **关闭或注销**，然后单击 **重新启动** 以重新启动计算机。

   重新启动计算机是启动 Linux 发行版安装的必要步骤。
   重新启动后，安装可能需要几分钟才能完成。

   有关将 WSL 设置为开发环境的更多信息，请参阅 [设置 WSL 开发环境](https://docs.microsoft.com/en-us/windows/wsl/setup/environment)。

## 安装所需的软件包和 Rust

要在 WSL 上安装 Rust 工具链：

1. 单击开始菜单，然后选择 **Ubuntu**。

1. 键入 UNIX 用户名以创建用户帐户。

1. 为您的 UNIX 用户键入密码，然后重新键入密码以确认它。

1. 通过运行以下命令使用 Ubuntu 高级打包工具 (`apt`) 下载 Ubuntu 发行版的最新更新：

   ```bash
   sudo apt update
   ```

1. 通过运行以下命令添加 Ubuntu 发行版所需的软件包：

   ```bash
   sudo apt install --assume-yes git clang curl libssl-dev llvm libudev-dev make protobuf-compiler
   ```

1. 下载 `rustup` 安装程序并使用它通过运行以下命令来安装 Ubuntu 发行版的 Rust：

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

1. 通过运行以下命令将 Rust 工具链配置为使用最新的稳定版本作为默认工具链：

   ```bash
   rustup default stable
   rustup update
   ```

1. 通过运行以下命令将 `nightly` 版本的工具链和 `nightly` WebAssembly (`wasm`) 目标添加到您的开发环境中：

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

   stable-x86_64-unknown-linux-gnu (default)
   rustc 1.61.0 (fe5b13d68 2022-05-18)

   # rustup +nightly show

   active toolchain
   ----------------

   nightly-x86_64-unknown-linux-gnu (overridden by +toolchain on the command line)
   rustc 1.63.0-nightly (e71440575 2022-06-02)
   ```

## 编译 Substrate 节点

现在您已经安装了 Rust 并为 Substrate 开发配置了 Rust 工具链，您就可以通过克隆 Substrate **节点模板** 文件并编译 Substrate 节点来完成开发环境的设置。

节点模板提供了一个工作环境，其中包含构建区块链所需的所有最常见功能，而无需任何多余的模块或工具。
为了确保节点模板为您提供一个相对稳定的工作环境来进行实验，建议的最佳实践是从 Substrate Developer Hub 存储库克隆 Substrate 节点模板，而不是从核心 Substrate 存储库克隆。

要编译 Substrate 节点模板：

1. 通过运行以下命令克隆节点模板存储库：

   ```bash
   git clone https://github.com/substrate-developer-hub/substrate-node-template
   ```

   在大多数情况下，您可以克隆 `main` 分支以获取最新的代码。
   但是，如果您想使用与特定 Polkadot 版本兼容的 Substrate 分支，则可以使用 `--branch` 命令行选项。
   单击 [标签](https://github.com/substrate-developer-hub/substrate-node-template/tags) 以查看与特定 Polkadot 版本兼容的分支列表。

1. 通过运行以下命令更改到节点模板目录的根目录：

   ```bash
   cd substrate-node-template
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
