---
title: Linux 开发环境
description: 在 Linux 上为 Substrate 设置本地开发环境。
keywords:
---

Rust 支持大多数 Linux 发行版。
根据你使用的具体发行版和操作系统版本，你可能需要为环境添加一些软件依赖项。
一般来说，你的开发环境应该包括一个链接器或 C 兼容编译器，如 `clang`，以及一个合适的集成开发环境（IDE）。

## 开始之前

查看你的操作系统文档，了解已安装的软件包以及如何下载和安装你可能需要的其他软件包。
例如，如果你使用 Ubuntu，可以使用 Ubuntu 高级打包工具（`apt`）来安装 `build-essential` 包：

```bash
sudo apt install build-essential
```

在安装 Rust 之前，您至少需要安装以下软件包：

```text
clang curl git make
```

由于区块链需要标准的加密技术来支持公钥/私钥对的生成和交易签名的验证，你还必须有一个提供加密技术的软件包，如 `libssl-dev` 或 `openssl-devel`。

## 安装所需软件包和 Rust

在 Linux 上安装 Rust 工具链：

1. 登录到你的计算机并打开终端。

1. 通过运行适合你 Linux 发行版的包管理命令，检查本地计算机上已安装的软件包。

1. 通过运行适合你 Linux 发行版的包管理命令，将缺少的软件包依赖项添加到本地开发环境中。

   例如，在 Ubuntu Desktop 或 Ubuntu Server 上，你可以运行类似以下的命令：

   ```bash
   sudo apt install --assume-yes git clang curl libssl-dev protobuf-compiler
   ```

   点击标签标题查看其他 Linux 操作系统的示例：

   <figure class='tabbed'>

   [[tabbedCode]]
   |```Debian
   | sudo apt install --assume-yes git clang curl libssl-dev llvm libudev-dev make protobuf-compiler

   [[tabbedCode]]
   |```Arch
   | pacman -Syu --needed --noconfirm curl git clang make protobuf

   [[tabbedCode]]
   | ```fedora
   | sudo dnf update
   | sudo dnf install clang curl git openssl-devel make protobuf-compiler

   [[tabbedCode]]
   | ```opensuse
   | sudo zypper install clang curl git openssl-devel llvm-devel libudev-devel make protobuf

   </figure>

   请记住，不同的发行版可能使用不同的包管理器，并以不同的方式打包软件包。
   例如，根据你的安装选择，Ubuntu Desktop 和 Ubuntu Server 可能有不同的软件包和不同的要求。
   然而，命令行示例中列出的软件包适用于许多常见的 Linux 发行版，包括 Debian、Linux Mint、MX Linux 和 Elementary OS。

1. 下载 `rustup` 安装程序并使用它安装 Rust，运行以下命令：

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

1. 按照显示的提示进行默认安装。

1. 运行以下命令更新当前 shell 以包含 Cargo：

   ```bash
   source $HOME/.cargo/env
   ```

1. 运行以下命令验证安装：

   ```bash
   rustc --version
   ```

1. 运行以下命令将 Rust 工具链配置为默认使用最新的稳定版本：

   ```bash
   rustup default stable
   rustup update
   ```

1. 运行以下命令将 `nightly` 版本和 `nightly` WebAssembly (wasm) 目标添加到你的开发环境中：

   ```bash
   rustup update nightly
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

1. 运行以下命令验证开发环境的配置：

   ```bash
   rustup show
   rustup +nightly show
   ```

   该命令显示类似以下的输出：

   ```bash
   # rustup show

   active toolchain
   ----------------

   stable-x86_64-unknown-linux-gnu (default)
   rustc 1.62.1 (e092d0b6b 2022-07-16)

   # rustup +nightly show

   active toolchain
   ----------------

   nightly-x86_64-unknown-linux-gnu (overridden by +toolchain on the command line)
   rustc 1.65.0-nightly (34a6cae28 2022-08-09)
   ```

## 编译 Substrate 节点

现在你已经安装了 Rust 并配置了用于 Substrate 开发的 Rust 工具链，你可以通过克隆 Substrate **节点模板**文件并编译一个 Substrate 节点来完成开发环境的设置。

节点模板提供了一个工作环境，包含了构建区块链所需的所有常见功能，而没有任何多余的模块或工具。
为了确保节点模板为你提供一个相对稳定的工作环境进行实验，推荐的最佳实践是从 Substrate 开发者中心仓库克隆 Substrate 节点模板，而不是从核心 Substrate 仓库。

编译 Substrate 节点模板：

1. 运行以下命令克隆节点模板仓库：

   ```bash
   git clone https://github.com/paritytech/polkadot-sdk-minimal-template
   ```

1. 运行以下命令切换到节点模板目录的根目录：

   ```bash
   cd polkadot-sdk-minimal-template
   ```

   如果你想保存更改并使该分支易于识别，可以运行类似以下的命令创建一个新分支：

   ```bash
   git switch -c my-wip-branch
   ```

   按照 README.md 文件中的说明构建节点。

构建成功完成后，你的本地计算机已准备好进行 Substrate 开发活动。

## 接下来去哪里

Substrate 开发者中心充当了社区可用的众多资源的中央门户。
根据你的兴趣和学习风格，你可能更喜欢某种途径。
例如，如果你喜欢阅读源代码并且熟悉 Rust，你可能希望从深入研究 [Rust API](https://paritytech.github.io/polkadot-sdk/master) 开始。

#### 告诉我

- [架构](/learn/architecture/)
- [网络和区块链](/learn/networks-and-nodes/)
- [构建过程](/build/build-process)

#### 指导我

- [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)
- [模拟网络](/tutorials/build-a-blockchain/simulate-network/)
- [添加受信任的节点](/tutorials/build-a-blockchain/add-trusted-nodes/)

<!-- TODO NAV.YAML -->
<!-- add these back -->
<!--If you are new to Substrate and the Substrate ecosystem, you might want broader exposure to what resources are available and where to find them by checking out [Explore](/main-docs/explore/).-->