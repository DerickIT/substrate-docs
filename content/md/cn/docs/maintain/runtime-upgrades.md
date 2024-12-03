---
title: 运行时升级
description: 解释运行时版本控制和存储迁移如何支持基于 Substrate 的网络的无分叉升级。
keywords:
---

无分叉运行时升级是 Substrate 区块链开发框架的一个定义性功能。
无需分叉代码库即可更新运行时逻辑的能力使您的区块链能够随着时间的推移而发展和改进。
此功能可以通过将运行时执行环境（运行时 WebAssembly blob）的定义作为区块链运行时状态中的一个元素来实现。

由于运行时是区块链状态的一部分，因此网络维护人员可以利用区块链的功能进行无需信任的去中心化共识，以安全地增强运行时。

在 FRAME 运行时开发系统中，系统库定义了 [ `set_code` 调用](https://paritytech.github.io/substrate/master/frame_system/pallet/enum.Call.html#variant.set_code)，用于更新运行时的定义。
教程 [升级正在运行的网络](/tutorials/build-a-blockchain/upgrade-a-running-network/) 演示了两种无需关闭节点或中断操作即可升级运行时的方法。
但是，教程中的两次升级都说明了向运行时添加功能，而不是_更新_现有的运行时状态。
如果运行时升级需要更改现有状态，则可能需要存储迁移。

## 运行时版本控制

在 [构建过程](/main-docs/build/build-process/) 中，您了解到编译节点会生成平台原生二进制文件和 WebAssembly 二进制文件，并且可以通过执行策略命令行选项来控制在区块生成过程的不同点选择使用哪个二进制文件。
选择要与其通信的运行时执行环境的组件称为**执行器**。
尽管您可以为自定义场景覆盖默认执行策略，但在大多数情况下，执行器会通过评估原生和 WebAssembly 运行时二进制文件的以下信息来选择要使用的适当二进制文件：

- `spec_name`
- `spec_version`
- `authoring_version`

为了将此信息提供给执行器进程，运行时包含一个 [运行时版本结构](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html)，类似于以下内容：

```rust
pub const VERSION: RuntimeVersion = RuntimeVersion {
  spec_name: create_runtime_str!("solo-template-node"),
  impl_name: create_runtime_str!("solo-template-node"),
  authoring_version: 1,
  spec_version: 1,
  impl_version: 1,
  apis: RUNTIME_API_VERSIONS,
  transaction_version: 1,
};
```

结构中的参数提供以下信息：

| 此参数        | 提供此信息                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spec_name`           | 不同 Substrate 运行时的标识符。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `impl_name`           | 规范实现的名称。这对节点影响不大，仅用于区分不同实施团队的代码。                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `authoring_version`   | 创作接口的版本。创作节点只有在与本地运行时相等时才会尝试创作区块。                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `spec_version`        | 运行时规范的版本。除非 Wasm 和原生二进制文件之间的 `spec_name` 、 `spec_version` 和 `authoring_version` 都相同，否则完整节点不会尝试使用其原生运行时代替链上 Wasm 运行时。 `spec_version` 的更新可以作为 CI 过程自动执行，就像 [波卡网络](https://gitlab.parity.io/parity/mirrors/polkadot/-/blob/master/scripts/ci/gitlab/check_extrinsics_ordering.sh) 所做的那样。此参数通常在 `transaction_version` 更新时递增。                      |
| `impl_version`        | 规范实现的版本。节点可以忽略此版本。它仅用于指示代码不同。只要 `authoring_version` 和 `spec_version` 相同，代码本身可能已更改，但原生和 Wasm 二进制文件执行的操作相同。通常，只有非逻辑中断优化才会导致 `impl_version` 更改。                                                                                                                                                                                                    |
| `transaction_version` | 处理交易的接口版本。此参数可用于同步硬件钱包或其他签名设备的固件更新，以验证运行时交易是否有效。该参数允许硬件钱包知道可以安全签名的交易。如果 `construct_runtime!` 宏中pallet的索引发生变化，或者可调度函数发生任何变化（例如参数数量或参数类型），则必须增加此数字。如果更新此数字，则 `spec_version` 也必须更新。 |
| `apis`                | 支持的 [运行时 API](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html) 列表及其版本。                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

编排引擎（有时称为执行器）在选择执行它之前，会验证原生运行时是否与 WebAssembly 具有相同的共识驱动逻辑。
但是，由于运行时版本控制是手动设置的，因此如果运行时版本被错误地表示，编排引擎仍然可能做出不适当的决策。

## 访问运行时版本

FRAME 系统通过 `state.getRuntimeVersion` RPC 端点公开运行时版本信息。
端点接受可选的区块标识符。
但是，在大多数情况下，您使用运行时 [元数据](/main-docs/build/application-development/#exposing-runtime-information-as-metadata) 来了解运行时公开的 API
以及如何与这些 API 交互。
运行时元数据_仅_应在链的 [运行时 `spec_version`](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html#structfield.spec_version) 更改时更改。

## 无分叉运行时升级

传统区块链在升级其链的状态转换函数时需要进行 [硬分叉](<https://en.wikipedia.org/wiki/Fork_(blockchain)>)。
硬分叉要求所有节点运营商停止其节点并手动升级到最新的可执行文件。
对于分布式生产网络，硬分叉升级的协调可能是一个复杂的过程。

运行时版本控制属性使基于 Substrate 的区块链能够实时升级运行时逻辑，而不会导致网络分叉。

为了执行无分叉运行时升级，Substrate 使用现有的运行时逻辑将存储在区块链上的 Wasm 运行时更新为具有新逻辑的新共识中断版本。此升级作为共识过程的一部分推送到网络上的所有完整节点。
升级 Wasm 运行时后，编排引擎会发现原生运行时 `spec_name` 、 `spec_version` 或 `authoring_version` 不再与新的 Wasm 运行时匹配。
因此，编排引擎执行规范的 Wasm 运行时，而不是在任何执行过程中使用原生运行时。

## 存储迁移

存储迁移是自定义的一次性函数，允许您更新存储以适应运行时的更改。
例如，如果运行时升级将用于表示用户余额的数据类型从_无符号_整数更改为_有符号_整数，则存储迁移将读取现有值作为无符号整数，并写回已转换为有符号整数的更新值。
如果您在需要时不进行此类更改以更改数据存储方式，则运行时无法正确解释存储值以包含在运行时状态中，并且可能导致未定义的行为。

### 使用 FRAME 进行存储迁移

FRAME 存储迁移是使用 [`OnRuntimeUpgrade`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.OnRuntimeUpgrade.html) 特性实现的。
`OnRuntimeUpgrade` 特性指定了一个单一函数— `on_runtime_upgrade` —允许
您指定在运行时升级_之后_但_在_任何 [`on_initialize`](/main-docs/learn/transaction-lifecycle#initialize-a-block) 函数或交易执行之前立即运行的逻辑。

### 准备存储迁移

准备存储迁移意味着了解运行时升级定义的更改。
Substrate 存储库使用 [`E1-runtimemigration`](https://github.com/paritytech/substrate/pulls?q=is%3Apr+label%3AE1-runtimemigration) 标签来指定此类更改。

### 编写迁移

每个存储迁移都不同，具有不同的要求和不同级别的复杂性。
但是，当您需要执行存储迁移时，可以使用以下推荐做法作为指导：

- 将迁移提取到可重用的函数中并为其编写测试。
- 在迁移中包含日志以帮助调试。
- 请记住，迁移是在_升级_运行时的上下文中执行的。
  迁移代码可能需要包含已弃用的类型，如 [此示例](https://github.com/hicommonwealth/substrate/blob/5f3933f5735a75d2d438341ec6842f269b886aaa/frame/indices/src/migration.rs#L5-L22) 所示。
- 使用存储版本使迁移更安全，使其更具声明性，如 [此示例](https://github.com/paritytech/substrate/blob/c79b522a11bbc7b3cf2f4a9c0a6627797993cb79/frame/elections-phragmen/src/lib.rs#L119-L157) 所示。

### 迁移排序

默认情况下，FRAME 根据pallet在 `construct_runtime!` 宏中出现的顺序对 `on_runtime_upgrade` 函数的执行进行排序。
对于升级，函数按_反向_顺序运行，从最后执行的pallet开始。
如果需要，您可以强制执行自定义顺序（请参阅 [此处的示例](https://github.com/hicommonwealth/edgeware-node/blob/7b66f4f0a9ec184fdebcccd41533acc728ebe9dc/node/runtime/src/lib.rs#L845-L866)）。

FRAME 存储迁移按以下顺序运行：

1. 如果使用自定义顺序，则使用自定义 `on_runtime_upgrade` 函数。
1. 系统 `frame_system::on_runtime_upgrade` 函数。
1. 运行时中定义的所有 `on_runtime_upgrade` 函数，从 `construct_runtime!` 宏中的最后一个pallet开始。

### 测试迁移

测试存储迁移非常重要。
您可以使用的一些测试存储迁移的工具包括：

- [Substrate 调试工具包](https://github.com/paritytech/substrate-debug-kit) 包含一个 [远程外部性](https://github.com/paritytech/substrate-debug-kit/tree/master/remote-externalities) 工具，允许在实时链数据上安全地执行存储迁移单元测试。
- [fork-off-substrate](https://github.com/maxsam4/fork-off-substrate) 脚本使创建链规范以引导本地测试链以测试运行时升级和存储迁移变得容易。

## 下一步去哪里

- [升级正在运行的网络](/tutorials/build-a-blockchain/upgrade-a-running-network/)
- [Substrate 迁移](https://github.com/apopiak/substrate-migrations)
