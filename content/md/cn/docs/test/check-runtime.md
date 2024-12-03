---
title: 检查运行时
description: 描述 try-runtime 命令行工具，用于针对链状态的生产快照测试指定的运行时状态。
keywords:
  - 测试
  - 快照
  - 集成测试
  - 生产
  - 运行时升级
  - 存储迁移
---

`try-runtime` 命令行工具使您可以使用 [内存中外部函数](https://paritytech.github.io/substrate/master/sp_state_machine/struct.TestExternalities.html) 数据结构来存储状态，从而查询运行时存储的快照。
通过使用内存中存储，您可以为指定的运行时状态编写测试，以便您可以在进入生产环境_之前_针对实际链状态进行测试。

最简单的形式是，您可以通过执行以下操作来使用 `try-runtime` 测试运行时状态：

1. 连接到远程节点。
2. 调用特定的运行时 API。
3. 从特定区块的节点检索状态。
4. 为检索到的数据编写测试。

## 动机

`try-runtime` 的初始动机是需要针对实际链的状态测试运行时更改。
之前的 [`TestExternalities`](https://paritytech.github.io/substrate/master/sp_state_machine/struct.TestExternalities.html) 和 [`BasicExternalities`](https://paritytech.github.io/substrate/master/sp_state_machine/struct.BasicExternalities.html) 用于使用模拟数据编写单元和集成测试，但缺乏针对链的实际状态进行测试的能力。
`try-runtime` 工具通过使用节点的以下 RPC 端点检索状态来扩展 `TestExternalities` 和 `BasicExternalities`：

- `rpc_get_storage`
- `rpc_get_keys_paged`

（有关详细信息，请参阅 [`远程外部函数库`](https://paritytech.github.io/substrate/master/src/frame_remote_externalities/lib.rs.html)；）

使用键值数据库检索状态后，try-runtime 会将数据插入到 `TestExternalities` 中。

## 工作原理

`try-runtime` 工具有其自己的外部函数实现，称为 [`remote_externalities`](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/utils/frame/remote-externalities/src/lib.rs)，它只是 `TestExternalities` 的一个包装器，使用通用的 [键值存储](/learn/state-transitions-and-storage)，其中数据是 [类型编码](/reference/scale-codec) 的。

下图说明了外部函数作为捕获该运行时存储的一种方式位于已编译运行时之外的方式。

![存储外部函数](/media/images/docs/reference/try-runtime-ext-1.png)

使用 `remote_externalities`，您可以捕获一些链状态并在其上运行测试。本质上，`RemoteExternalities` 将使用实际链的数据填充 `TestExternalities`。

![使用外部函数进行测试](/media/images/docs/reference/try-runtime-ext-2.png)

要查询状态，`try-runtime` 使用 [`StateApiClient`](https://paritytech.github.io/substrate/master/sc_rpc/state/trait.StateApiClient.html) 提供的 RPC 方法。
特别是：

- [`storage`](https://paritytech.github.io/substrate/master/sc_rpc/state/trait.StateApiClient.html#method.storage)
  此方法返回表示您指定的区块的键的存储值。

- [`storage_key_paged`](https://paritytech.github.io/substrate/master/sc_rpc/state/trait.StateApiClient.html#method.storage_keys_paged)
  此方法返回与您使用分页支持指定的参数匹配的键。

## 用法

`try-runtime` 最常见的用例是帮助您准备存储迁移和运行时升级。

由于查询存储的 RPC 调用在计算上代价很高，因此在使用 `try-runtime` 命令之前，您应该为正在运行的节点设置许多命令行选项。要为 `try-runtime` 测试准备节点，请设置以下选项：

- 将 `--rpc-max-payload 1000` 设置为确保大型 RPC 查询可以正常工作。
- 将 `--rpc-cors all` 设置为确保 WebSocket 连接可以正常进行。

您可以将 `try-runtime` 与 [`fork-off-substrate`](https://github.com/maxsam4/fork-off-substrate) 结合使用，以在生产环境之前测试您的链。
使用 `try-runtime` 测试链的迁移及其迁移前和迁移后的状态。
然后，如果您想检查迁移后区块生产是否继续，可以使用 `fork-off-substrate`。

### 运行时升级挂钩

默认情况下，运行时升级挂钩（可以在运行时或 pallet 内部定义）指定运行时升级时应该发生的情况。
也就是说，默认的 `on_runtime_upgrade` 方法仅描述升级_之后_的运行时状态。
但是，可以使用 `try-runtime` 提供的方法来检查和比较运行时升级_之前_和_之后_的运行时状态，以进行测试。

如果您为运行时启用了 `try-runtime` 功能，则可以为运行时定义 `pre-upgrade` 和 `post-upgrade` 挂钩，如下所示：

```rust
#[cfg(feature = "try-runtime")]
fn pre_upgrade() -> Result<Vec<u8>, &'static str> {
		Ok(Vec::new())
}

#[cfg(feature = "try-runtime")]
fn post_upgrade(_state: Vec<u8>) -> Result<(), &'static str> {
		Ok(())
}
```

使用这些函数，您可以使用 `pre-upgrade` 挂钩来检索运行时状态并将其作为 Vec<u8> 结果返回。
您可以将 Vec<u8> 作为输入参数传递给 `post-upgrade` 挂钩。

## 命令行示例

要从命令行使用 `try-runtime`，请使用 `--features=try-runtime` 标志运行您的节点。
例如：

```bash
cargo run --release --features=try-runtime try-runtime
```

您可以将以下子命令与 `try-runtime` 一起使用：

- `on-runtime-upgrade`：针对给定的运行时状态执行 `tryRuntime_on_runtime_upgrade`。
- `offchain-worker`：针对给定的运行时状态执行 `offchainWorkerApi_offchain_worker`。
- `execute-block`：使用给定的区块和父区块的运行时状态执行 `core_execute_block`。
- `follow-chain`：跟踪给定链的最终确定区块并将它们应用于其所有外部函数。
  这允许在较长时间内检查新运行时的行为，并使用实际交易作为输入。

要查看特定 `try-runtime` 子命令的使用信息，请指定子命令和 `--help` 标志。
例如，要查看 `try-runtime on-runtime-upgrade` 的使用信息，您可以运行以下命令：

```bash
cargo run --release --features=try-runtime try-runtime on-runtime-upgrade --help
```

例如，您可以使用类似于以下命令的命令，针对本地运行的链使用 `on-runtime-upgrade` 子命令运行 `try-runtime`：

```bash
cargo run --release --features=try-runtime try-runtime on-runtime-upgrade live ws://localhost:9944
```

您可以使用类似于以下命令的命令，在 `localhost:9944` 上使用 `try-runtime` 重新执行来自 `ElectionProviderMultiPhase` 链下工作者的代码：

```bash
cargo run -- --release \
   --features=try-runtime \
   try-runtime \
   --execution Wasm \
   --wasm-execution Compiled \
   offchain-worker \
   --header-at 0x491d09f313c707b5096650d76600f063b09835fd820e2916d3f8b0f5b45bec30 \
   live \
   -b 0x491d09f313c707b5096650d76600f063b09835fd820e2916d3f8b0f5b45bec30 \
   -m ElectionProviderMultiPhase \
   --uri wss://localhost:9944
```

您可以使用类似于以下命令的命令，在 SomeChain 的状态上运行本地运行时的迁移：

```bash
RUST_LOG=runtime=trace,try-runtime::cli=trace,executor=trace \
   cargo run try-runtime \
   --execution Native \
   --chain somechain-dev \
   on-runtime-upgrade \
   live \
   --uri wss://rpc.polkadot.io
```

您可以使用类似于以下命令的命令，针对特定区块号的状态运行 try-runtime：

```bash
RUST_LOG=runtime=trace,try-runtime::cli=trace,executor=trace \
   cargo run try-runtime \
   --execution Native \
   --chain dev \
   --no-spec-name-check \
   on-runtime-upgrade \
   live \
   --uri wss://rpc.polkadot.io \
   --at <block-hash>
```

请注意，此命令需要 `--no-spec-name-check` 命令行选项。

## 接下来的步骤

- [存储键](/build/runtime-storage#storage-value-keys)
- [`OnRuntimeUpgrade`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.OnRuntimeUpgrade.html)
- [`try-runtime-upgrade`](https://paritytech.github.io/substrate/master/frame_executive/struct.Executive.html#method.try_runtime_upgrade)
- [Staking pallet](https://paritytech.github.io/substrate/master/pallet_staking/index.html)
