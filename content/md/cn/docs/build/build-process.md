---
title: 构建过程
description: 描述 Substrate 节点如何编译成平台原生二进制文件和 WebAssembly 二进制文件，以及如何使用这些二进制文件来执行对 Substrate 运行时的调用。
keywords:
---

在 [架构](/learn/architecture) 中，你了解到 Substrate 节点由外部节点主机和运行时执行环境组成。
这些节点组件通过运行时 API 调用和主机函数调用相互通信。
在本节中，你将了解有关 Substrate 运行时如何编译成平台原生可执行文件以及如何编译成存储在区块链上的 WebAssembly (Wasm) 二进制文件的更多信息。
在了解二进制文件如何编译的内部工作原理之后，你将了解为什么有两个二进制文件、何时使用它们以及如何在需要时更改执行策略。

## 编译优化的工件

你可能已经知道，可以通过在 Substrate 节点项目的根目录中运行 `cargo build --release` 命令来编译 Substrate 节点。
此命令将为项目构建平台特定的可执行文件和 WebAssembly 二进制文件，并生成**优化的**可执行工件。
生成优化的可执行工件包括一些编译后处理。

作为优化过程的一部分，WebAssembly 运行时二进制文件会在包含在链的创世状态之前通过一系列内部步骤进行编译和压缩。
为了让你更好地理解这个过程，下图总结了这些步骤。

![WebAssembly 在链上包含之前进行编译和压缩](/media/images/docs/node-executable.png)

以下部分将更详细地描述构建过程。

### 构建 WebAssembly 二进制文件

`wasm-builder` 是一个工具，它将为你的项目构建 WebAssembly 二进制文件的过程集成到主要的 `cargo` 构建过程中。
此工具发布在 `substrate-wasm-builder` crate 中。

当你启动构建过程时，`cargo` 会从项目中的所有 `Cargo.toml` 文件构建依赖关系图。
然后，运行时 `build.rs` 模块使用 `substrate-wasm-builder` crate 将运行时的 Rust 代码编译成 WebAssembly 二进制文件，从而创建初始二进制工件。

#### WebAssembly 中包含的功能

默认情况下，`wasm-builder` 会启用为项目在 WebAssembly 二进制文件和平台原生可执行文件中定义的所有功能，但 `default` 和 `std` 功能除外，这些功能仅为原生构建启用。

#### 用于自定义构建过程的环境变量

你可以使用以下环境变量来自定义 WebAssembly 二进制文件的构建方式：

| 使用此变量       | 如果你想这样做                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SKIP_WASM_BUILD`       | 跳过构建 WebAssembly 二进制文件。这在只需要重新编译原生二进制文件时很有用。但是，如果 WebAssembly 二进制文件不存在，则不会编译任何二进制文件。你可以通过在环境变量中包含 PROJECT_NAME 来跳过各个项目的 WebAssembly 构建。例如，要跳过 cargo 项目 node-runtime 的 WebAssembly 二进制文件的构建，可以使用环境变量 SKIP_NODE_RUNTIME_WASM_BUILD。 |
| `WASM_BUILD_TYPE`       | 指定 WebAssembly 二进制文件是 `release` 构建还是 `debug` 构建。默认情况下，使用为 `cargo` 命令指定的构建类型。                                                                                                                                                                                                                                                                                                         |
| `FORCE_WASM_BUILD`      | 强制进行 WebAssembly 构建。很少需要此环境变量，因为 `wasm-builder` 会指示 `cargo` 检查文件更改。                                                                                                                                                                                                                                                                                                                 |
| `WASM_BUILD_RUSTFLAGS`  | 在构建 WebAssembly 二进制文件时扩展传递给 `cargo build` 命令的 `RUSTFLAGS`。                                                                                                                                                                                                                                                                                                                                                               |
| `WASM_BUILD_NO_COLOR`   | 禁用 WebAssembly 构建的彩色输出。                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `WASM_TARGET_DIRECTORY` | 将 WebAssembly 二进制文件复制到指定的目录。路径需要是绝对路径。                                                                                                                                                                                                                                                                                                                                                                          |
| `WASM_BUILD_TOOLCHAIN`  | 指定用于构建 WebAssembly 二进制文件的工具链。格式需要与 `cargo` 使用的格式相同，例如 `nightly-2020-02-20`。                                                                                                                                                                                                                                                                                                          |
| `CARGO_NET_OFFLINE`     | 阻止对某些或所有启动以支持离线环境的进程的网络访问。                                                                                                                                                                                                                                                                                                                                                                      |

#### 压缩 WebAssembly 二进制文件

`substrate-wasm-builder` crate 使用较低级别的进程来优化指令序列并删除任何不必要的代码（例如用于调试的代码），以创建一个紧凑的 WebAssembly 二进制文件。
然后进一步压缩二进制文件以最大限度地减少最终 WebAssembly 二进制文件的大小。
当编译器处理节点的 `runtime/src/lib.rs` 文件时，它会看到包含生成的 WebAssembly 二进制文件的需求：

```rust
include!(concat!(env!("OUT_DIR"), "/wasm_binary.rs"));
```

此代码在其编译结果中包含紧凑的 WebAssembly 二进制文件 (`WASM_BINARY`) 和编译器生成的未压缩 WebAssembly 二进制文件 (`WASM_BINARY_BLOATY`)，并生成项目的最终可执行二进制文件。

在构建过程的每个阶段，WebAssembly 二进制文件都会被压缩到比早期更小的尺寸。
例如，你可以比较 Polkadot 的每个 WebAssembly 二进制工件的大小：

```bash
.rw-r--r-- 1.2M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.compressed.wasm
.rw-r--r-- 5.1M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.wasm
.rwxr-xr-x 5.5M pep  1 Dec 16:13 │  └── polkadot_runtime.wasm
```

你应该始终将完全压缩的运行时 (`*_runtime.compact.compressed.wasm`) WebAssembly 二进制文件用于链上升级和中继链验证。
在大多数情况下，无需使用初始 WebAssembly 二进制文件或中间紧凑型工件。

## 执行策略

编译包含原生和 WebAssembly 运行时的节点后，可以使用命令行选项来指定节点的运行方式。
有关可用于启动节点的命令行选项的详细信息，请参阅 [solo-template-node](/reference/command-line-tools/solo-template-node) 命令行参考。

启动节点时，节点可执行文件使用你指定的命令行选项来初始化链并生成创世块。
在此过程中，节点会将 WebAssembly 运行时添加为存储项值和相应的 `:code` 密钥。

启动节点后，正在运行的节点将选择要使用的运行时。
默认情况下，节点始终将 WebAssembly 运行时用于所有操作，包括：

- 同步
- 创作新块
- 导入块
- 与离链工作程序交互

### WebAssembly 运行时的选择

使用 WebAssembly 运行时非常重要，因为 WebAssembly 和原生运行时可能会出现差异。
例如，如果你对运行时进行了更改，则必须生成新的 WebAssembly blob 并更新链以使用新版本的 WebAssembly 运行时。
更新后，WebAssembly 运行时与原生运行时不同。
为了解决这种差异，所有执行策略都将 WebAssembly 运行时的表示视为规范运行时。
如果原生运行时和 WebAssembly 运行时版本不同，则始终选择 WebAssembly 运行时。

由于 WebAssembly 运行时作为区块链状态的一部分存储，因此网络必须就该二进制文件的表示达成共识。
为了就二进制文件达成共识，表示 WebAssembly 运行时的 blob 在所有同步节点上必须完全相同。

### WebAssembly 执行环境

WebAssembly 执行环境可能比 Rust 执行环境更严格。
例如，WebAssembly 执行环境是一个 32 位架构，最多具有 4GB 的内存。
可以在 WebAssembly 运行时中执行的逻辑始终可以在 Rust 执行环境中执行。
但是，并非所有可以在 Rust 运行时中执行的逻辑都可以在 WebAssembly 运行时中执行。
区块创作节点通常使用 WebAssembly 执行环境来帮助确保它们生成有效的块。

### 原生运行时

尽管默认情况下选择 WebAssembly 运行时，但可以通过将**执行策略**指定为命令行选项来覆盖为所有操作或特定操作选择的运行时。

如果原生运行时和 WebAssembly 运行时共享相同的 [版本](/maintain/runtime-upgrades/#runtime-versioning)，则除了 WebAssembly 运行时之外，还可以选择性地使用原生运行时，或者如果使用 WebAssembly 运行时失败，则作为后备方案。
通常，你只会出于性能原因或因为它比 WebAssembly 运行时更不严格的环境而选择使用原生运行时。
例如，你可能希望将原生运行时用于初始同步。
要将原生运行时用于同步块，可以使用 `--execution-syncing native` 或 `--execution-syncing native-else-wasm` 命令行选项启动节点。

有关使用命令行选项为所有操作或特定操作指定执行策略的信息，请参阅 [solo-template-node](/reference/command-line-tools/solo-template-node)。
有关执行策略变体的更多信息，请参阅 [ExecutionStrategy](https://paritytech.github.io/substrate/master/sc_cli/arg_enums/enum.ExecutionStrategy.html)

## 在没有原生运行时的情况下构建 WebAssembly

启动新链需要 WebAssembly 运行时。
提供初始 WebAssembly 运行时后，表示 WebAssembly 运行时的 blob 可以作为 [链规范](/build/chain-spec) 的一部分传递给其他节点。
在某些罕见情况下，你可能希望在没有原生运行时的情况下编译 WebAssembly 目标。
例如，如果你正在测试 WebAssembly 运行时以准备无分叉升级，你可能只想编译新的 WebAssembly 二进制文件。

尽管这是一个罕见的用例，但你可以使用 [build-only-wasm.sh](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/.maintain/build-only-wasm.sh) 脚本来构建 `no_std` WebAssembly 二进制文件，而无需编译原生运行时。

你还可以使用 `wasm-runtime-overrides` 命令行选项从文件系统加载 WebAssembly。

## 在没有 WebAssembly 的情况下编译 Rust

如果你想在不构建新的 WebAssembly 运行时的情况下编译节点的 Rust 代码，则可以使用 `SKIP_WASM_BUILD` 作为构建选项。
此选项主要用于在不需要更新 WebAssembly 时加快编译时间。

## 接下来的步骤

- [Wasm-builder 源代码](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/utils/wasm-builder/src/lib.rs)
- [Rust 编译选项](https://doc.rust-lang.org/cargo/commands/cargo-build.html#compilation-options)
- [讨论：删除原生运行时](https://github.com/paritytech/substrate/issues/10579)
