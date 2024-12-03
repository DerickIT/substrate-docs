---
title: 构建确定性运行时
description: 解释如何使用 Substrate 运行时工具箱 (srtool) 和 Docker 来构建基于 Substrate 的链的 WebAssembly 运行时。
keywords:
---

默认情况下，Rust 编译器会生成优化的 WebAssembly 二进制文件。
这些二进制文件非常适合在进行本地开发的隔离环境中工作。
但是，编译器默认生成的 WebAssembly 二进制文件不能保证是确定性可重现的。
每次编译器生成 WebAssembly 运行时时，它都可能会生成略微不同的 WebAssembly 字节码。
这在所有节点都必须使用完全相同的原始链规范文件的区块链网络中是有问题的。

使用不能保证确定性可重现的构建也会导致其他问题。
例如，如果你想自动化区块链的构建过程，你将需要确保相同的代码始终产生相同的结果。
如果没有确定性构建，每次推送都会编译 WebAssembly 运行时，这会产生不一致且不可预测的结果，从而难以与任何自动化集成，并可能持续破坏你的 CI/CD 管道。
确定性构建（始终编译成完全相同的字节码的代码）还可以确保可以检查、审核和独立验证 WebAssembly 运行时。

## WebAssembly 运行时的工具

为了帮助你以确定性的方式编译 WebAssembly 运行时，你可以使用与为 Polkadot、Kusama 和其他基于 Substrate 的链生成运行时相同的工具。
这个工具（统称为 Substrate 运行时工具箱或 `srtool`）确保相同的源代码始终编译成相同的 WebAssembly blob。

Substrate 运行时工具箱 (`srtool`) 的核心组件是一个 Docker 容器。
此容器作为 Docker 映像的一部分执行。
`srtool` Docker 映像的名称指定用于编译映像中包含的代码的 Rust 编译器的版本。
例如，映像 `paritytech/srtool:1.62.0` 表示映像中的代码是用 `rustc` 编译器的 `1.62.0` 版本编译的。

## 使用 Docker 容器

因为 `srtool` 是一个 Docker 容器，所以你必须在你的构建环境中使用 Docker 才能使用它。
但是，你不需要了解任何关于使用 Docker 来使用 `srtool` 构建基于 Substrate 的链的知识，因为你可以使用 `srtool-cli` 命令行界面来使用 Docker 映像。

`srtool-cli` 包是一个用 Rust 编写的命令行实用程序，它会在你的计算机上安装一个名为 `srtool` 的可执行程序。
此程序简化了你与 srtool Docker 容器的交互。
随着时间的推移，围绕 `srtool` Docker 映像的工具已扩展到包括以下工具和辅助程序：

- [srtool-cli](https://github.com/chevdor/srtool-cli) 提供了一个命令行界面，用于拉取 srtool Docker 映像，获取有关映像和用于与其交互的工具的信息，以及使用 `srtool` Docker 容器构建运行时。

- [subwasm](https://github.com/chevdor/subwasm) 提供了用于使用 srtool 构建的元数据和 WebAssembly 运行时的命令行选项。`subwasm` 程序也在内部用于在 `srtool` 映像中执行任务。

- [srtool-actions](https://github.com/chevdor/srtool-actions) 提供 GitHub 操作，以将使用 `srtool` 映像生成的构建与你的 GitHub CI/CD 管道集成。
- [srtool-app](https://gitlab.com/chevdor/srtool-app) 提供了一个简单的图形用户界面，用于使用 `srtool` Docker 映像构建运行时。

## 准备环境

要使用执行 `srtool` Docker 容器中代码的 Docker 映像，你必须拥有 Docker 帐户以及 Docker 命令行或桌面工具。
你还应该使用你想要使用的特定命令行工具来准备你的开发环境。
至少，你应该安装 `srtool-cli` 程序，以便你可以使用简单的命令行界面来使用 Docker 映像。

准备环境：

1. 在你的 Substrate 开发环境中打开一个终端 shell。

2. 通过运行以下命令来验证你是否已安装 Docker：

   ```bash
   docker --version
   ```

   如果安装了 Docker，则该命令会显示版本信息。
   例如：

   ```text
   Docker version 20.10.17, build 100c701
   ```

3. 通过运行以下命令来安装 `srtool` 命令行界面：

   ```bash
   cargo install --git https://github.com/chevdor/srtool-cli
   ```

4. 通过运行以下命令来查看 `srtool` 命令行界面的使用信息：

   ```bash
   srtool help
   ```

5. 通过运行以下命令来下载最新的 `srtool` Docker 映像：

   ```bash
   srtool pull
   ```

## 开始确定性构建

准备环境后，你可以开始使用 `srtool` Docker 映像编译 WebAssembly 运行时。

构建运行时：

1. 在你的 Substrate 开发环境中打开一个终端 shell。

2. 通过运行类似于以下命令的命令来为项目编译运行时：

   ```bash
   srtool build --app --package solo-template-node-runtime --runtime-dir runtime
   ```

   - 你为 `--package` 指定的名称应该是 `Cargo.toml` 文件中为运行时定义的名称。

   - 你为 `--runtime-dir` 指定的路径应该是运行时的 `Cargo.toml` 文件的路径。
     如果运行时的 `Cargo.toml` 文件位于 `runtime` 子目录中（例如，runtime/kusama），则可以省略 `--runtime-dir` 命令行选项。

## 添加工作流程操作

如果你使用 GitHub 存储库来存放你的基于 Substrate 的项目，则可以设置一个 GitHub 工作流程来自动开始编译 WebAssembly 运行时。

添加用于构建运行时的工作流程：

1. 在你的 Substrate 存储库中创建一个 `.github/workflows` 目录。

1. 在 `.github/workflows` 目录中，单击 **添加文件**，然后选择 **创建新文件**。
1. 从 [srtools-actions](https://github.com/chevdor/srtool-actions) 存储库中的 `basic.yml` 示例中复制示例 GitHub 操作，并将其粘贴到你上一步中创建的文件中。

1. 修改示例操作中的设置以适合你的链。

   例如，修改以下设置：

   - 链的名称
   - 运行时包的名称
   - 运行时的位置

1. 为你的 Substrate 存储库中的操作文件键入一个名称。

1. 单击 **提交新文件**。

## 从 Docker Hub 下载

你必须在你的构建环境中拥有 Docker 帐户并安装 Docker 才能使用 Substrate 运行时工具箱。
如果你登录到 Docker Hub，则可以搜索 `paritytech/srtool` 容器并找到相应的映像，其标签名称标识 Rust 编译器版本和构建脚本版本。

如果你不想使用 [`srtool-cli`](/reference/command-line-tools/srtool/#srtool-cli) 或 [`srtool-app`](https://gitlab.com/chevdor/srtool-app) 来使用 `paritytech/srtool` 容器，则可以直接从 Docker Hub 拉取 `paritytech/srtool` 容器映像。

要从 Docker Hub 拉取映像：

1. 登录到 Docker Hub。

2. 在搜索字段中键入 `paritytech/srtool` 并按 Enter 键。

3. 单击 **paritytech/srtool**，然后单击 **Tags**。

4. 复制要拉取的映像的命令。
5. 在你的本地计算机上打开一个终端 shell。
6. 粘贴你从 Docker Hub 复制的命令。

   例如，你可能会运行类似于以下命令的命令：

   ```bash
   docker pull paritytech/srtool:1.62.0
   ```

   该命令将下载并解压缩映像。

### 映像的命名约定

与许多 Docker 映像不同，`srtool` 映像没有 `latest` 标签。
如果你直接从 Docker Hub 下载映像，则需要选择与你安装的 Rust 编译器版本兼容的映像。
`paritytech/srtool` Docker 映像的命名约定指定用于编译映像中包含的代码的 Rust 编译器的版本。
还有一些映像同时指定了编译器版本和使用的构建脚本版本。
例如，名为 `paritytech/srtool:1.62.0-0.9.19` 的映像是用 `rustc` 编译器的 `1.62.0` 版本编译的，但使用了 `0.9.19` 版本的构建脚本。

仅指定编译器版本的映像始终包含最新版本的软件。
在大多数情况下，这就是你应该使用的映像。

### 编译器版本

如果你直接从 Docker Hub 下载映像，则应首先检查你正在使用的 Rust 编译器的版本，以确保该映像与你安装的 Rust 版本兼容。
如果你不确定你使用的是哪个版本的 Rust 编译器，则可以运行以下命令来检查你安装的版本：

```bash
rustc --version
