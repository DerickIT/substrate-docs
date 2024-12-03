---
title: 解决 Rust 问题
description: 诊断和修复 Rust 和 Substrate 开发环境问题。
keywords:
  - Rust
  - 工具链
  - 编译器
  - rustup
---

如果编译 [Substrate 节点模板](https://github.com/substrate-developer-hub/substrate-node-template) 失败，则问题很可能是由开发环境中 Rust 的配置引起的。
本节建议如何诊断和修复配置问题。

## 检查当前配置

要查看有关您当前使用的 Rust 工具链的信息，请运行以下命令：

```bash
rustup show
```

此命令将显示类似于以下 Ubuntu 示例的输出：

```text
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/user/.rustup

installed toolchains
--------------------

stable-x86_64-unknown-linux-gnu (default)
nightly-2020-10-06-x86_64-unknown-linux-gnu
nightly-x86_64-unknown-linux-gnu

installed targets for active toolchain
--------------------------------------

wasm32-unknown-unknown
x86_64-unknown-linux-gnu

active toolchain
----------------

stable-x86_64-unknown-linux-gnu (default)
rustc 1.50.0 (cb75ad5db 2021-02-10)
```

在此示例中，默认工具链来自在 x86_64 架构上运行的 Linux 的 `stable` 发布通道。
示例输出还表明 `nightly-x86_64-unknown-linux-gnu` 工具链已安装，并且已安装了两个目标：

- `x86_64-unknown-linux-gnu` Linux 的本机 Rust 目标。
- `wasm32-unknown-unknown` WebAssembly 目标。

此环境还安装了 `nightly-2020-10-06-x86_64-unknown-linux-gnu` 工具链，但此工具链仅在显式指定为命令行选项时才使用。
有关将特定工具链指定为命令行选项的示例，请参阅 [指定 nightly 版本](#specifying-nightly-version)。

## 使用 nightly 发布通道进行 WebAssembly

Substrate 使用 [WebAssembly](https://webassembly.org) (Wasm) 来生成可移植的区块链运行时。
您必须将 Rust 编译器配置为使用 [`nightly 构建`](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html) 以允许您
将 Substrate 运行时代码编译为 Wasm 目标。

## 更新工具链

通常，您应该始终使用 Rust `stable` 和 `nightly` 构建的最新版本，因为 Substrate 中的更改通常取决于 Rust `nightly` 编译器构建中的上游更改。
要确保您的 Rust 编译器始终是最新的，您应该运行以下命令：

```bash
rustup update
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```

运行 `rustup update` 会更新 `nightly` 和 `stable` 工具链以使用最新的版本。
如果您在更新 `nightly` 工具链后无法编译 WebAssembly 目标，则可以回滚到工具链的早期版本并将该版本指定为命令行选项。
有关获取 `nightly` 工具链的早期版本以及将版本指定为命令行选项的更多信息，请参阅 [降级工具链](#downgrading-rust-nightly)。

## 使用特定 nightly 工具链

如果您想保证您的构建在更新 Rust 和其他依赖项时在您的计算机上正常工作，则应使用您知道与您使用的 Substrate 版本兼容的特定 Rust `nightly` 工具链。
您如何识别和传达要用于项目的特定 `nightly` 工具链版本可能会有所不同。
例如，Polkadot 在其 [发布说明](https://github.com/paritytech/polkadot/releases) 中发布了此信息。

在您确定要使用的特定 `nightly` 工具链版本后，您可以通过运行类似于以下内容的命令将其安装到您的开发环境中：

```bash
rustup install nightly-<yyyy-MM-dd>
```

例如：

```bash
rustup install nightly-2022-02-16
```

在您安装特定版本的 nightly 工具链后，通过运行类似于以下内容的命令将 WebAssembly 目标配置为使用它：

```bash
rustup target add wasm32-unknown-unknown --toolchain nightly-<yyyy-MM-dd>
```

例如：

```bash
rustup target add wasm32-unknown-unknown --toolchain nightly-2022-02-16
```

### 在环境变量中指定工具链

您可以设置 `WASM_BUILD_TOOLCHAIN` 环境变量以指定要用于编译 WebAssembly 的 `nightly` 工具链的版本。例如：

```bash
WASM_BUILD_TOOLCHAIN=nightly-<yyyy-MM-dd> cargo build --release
```

此命令使用指定的 nightly 工具链构建 _运行时_。
项目的其余部分使用 _默认_ 工具链编译，即您安装的 `stable` 工具链的最新版本。

### 降级 nightly 工具链

如果您要将计算机配置为使用最新的 Rust `nightly` 工具链，并且您想降级到特定的 nightly 版本，则必须首先卸载最新的 `nightly` 工具链。
例如，您可以删除最新的 `nightly` 工具链，然后使用特定版本的 `nightly` 工具链，方法是运行类似于以下内容的命令：

```sh
rustup uninstall nightly
rustup install nightly-<yyyy-MM-dd>
rustup target add wasm32-unknown-unknown --toolchain nightly-<yyyy-MM-dd>
```

## 确保 PATH 设置正确

如果在安装 Rust 后命令似乎不起作用，显示错误，例如 `command not found: rustup`，请确保您的 PATH 配置正确。

目前，`rustup` 安装程序默认安装到 bash 配置文件（在 Mac 上）。如果您使用的是其他 shell，请确保将此行添加到您的配置文件（例如 `.zshrc`）：

```bash
source "$HOME/.cargo/env"
```

## 为 Apple Silicon（M 系列）macOS 用户安装 cmake 或 protobuf

目前，在使用配备 M 系列芯片的 macOS 计算机上预安装的软件包时，编译 Substrate 节点时存在问题。

```sh
error: failed to run custom build command for prost-build v0.10.4
```

如果您看到此错误，则有两种解决方案。

- 通过运行以下命令安装 `cmake`：

```bash
brew install cmake
```

- 通过运行以下命令集安装正确的预编译 `protoc`：

```bash
git clone https://github.com/protocolbuffers/protobuf.git
cd protobuf

brew install autoconf
brew install automake
brew install Libtool

autoreconf -i
./autogen.sh
./configure
make
make check
sudo make install

export PATH=/opt/usr/local/bin:$PATH
