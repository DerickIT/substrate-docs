---
title: 运行时 API
description: 突出显示启用与外部节点服务通信的运行时接口。
keywords:
---

如 [架构](/learn/architecture) 中所述，Substrate 节点由外部节点服务和运行时组成，这种职责分离是设计基于 Substrate 的链和构建可升级逻辑的重要概念。
但是，外部节点服务和运行时必须相互通信才能完成许多关键操作，包括读取和写入数据以及执行状态转换。
外部节点服务通过调用运行时应用程序编程接口来执行特定任务，从而与运行时进行通信。
默认情况下，Substrate 运行时为外部节点服务调用提供以下特性：

- [`AccountNonceApi`](https://paritytech.github.io/substrate/master/frame_system_rpc_runtime_api/trait.AccountNonceApi.html)
- [`AuraApi`](https://paritytech.github.io/substrate/master/sp_consensus_aura/trait.AuraApi.html)
- [`Benchmark`](https://paritytech.github.io/substrate/master/frame_benchmarking/trait.Benchmark.html)
- [`BlockBuilder`](https://paritytech.github.io/substrate/master/sp_block_builder/trait.BlockBuilder.html)
- [`GrandpaApi`](https://paritytech.github.io/substrate/master/sp_consensus_grandpa/trait.GrandpaApi.html)
- [`NominationPoolsApi`](https://paritytech.github.io/substrate/master/pallet_nomination_pools_runtime_api/trait.NominationPoolsApi.html)
- [`OffchainWorkerApi`](https://paritytech.github.io/substrate/master/sp_offchain/trait.OffchainWorkerApi.html)
- [`SessionKeys`](https://paritytech.github.io/substrate/master/sp_session/trait.SessionKeys.html)
- [`TaggedTransactionQueue`](https://paritytech.github.io/substrate/master/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html)
- [`TransactionPaymentApi`](https://paritytech.github.io/substrate/master/pallet_transaction_payment_rpc_runtime_api/trait.TransactionPaymentApi.html)

## AccountNonceApi

使用 `AccountNonceApi` 获取指定帐户标识符的 nonce。
每次使用帐户完成交易时，每个帐户的 nonce 都会递增。
因此，nonce 有时也称为交易索引。

此 API 提供以下方法：

- `account_nonce` 用于获取指定 AccountId 的当前帐户 nonce。
- `account_nonce_with_context` 用于获取指定 AccountId 和执行上下文的当前帐户 nonce。

## AuraApi

使用 `AuraApi` 使用基于时隙的共识（使用权威的循环轮换）来管理区块创作。
尽管大多数与共识相关的任务都由外部节点服务处理，但运行时必须为此 API 提供与共识相关的任务，这些任务是状态转换逻辑的一部分。

此 API 为基于权威的循环轮询调度 ([Aura](/reference/glossary/#aura)) 提供以下方法：

- `slot_duration` 用于获取 Aura 共识的时隙持续时间。
- `slot_duration_with_context` 用于在指定的执行上下文中获取 Aura 共识的时隙持续时间。
- `authorities` 用于获取 Aura 共识的权威集。
- `authorities_with_context` 用于在指定的执行上下文中获取 Aura 共识的权威集。

## Benchmark

使用 `Benchmark` API 提供 FRAME 运行时中函数执行 [基准测试](/test/benchmark/) 所需的信息。

此 API 提供以下方法：

- `benchmark_metadata` 用于获取此运行时可用的基准测试元数据。
- `benchmark_metadata_with_context` 用于在指定的执行上下文中获取此运行时可用的基准测试元数据。
- `dispatch_benchmark` 用于调度指定的基准测试。
- `dispatch_benchmark_with_context` 用于在指定的执行上下文中调度指定的基准测试。

## BlockBuilder

使用 `BlockBuilder` API 提供构建和完成区块所需的功能。
运行时负责检查交易有效性并执行交易以构建区块。
对于外部节点，交易是不透明的向量数组 (Vec<u8>)。

此 API 提供以下方法：

- `apply_extrinsic` 用于将指定的外部函数包含在当前区块中。
  该方法还会返回一个结果，指示交易是否已包含在区块中。
- `apply_extrinsic_with_context` 用于将指定的外部函数包含在当前区块和指定的执行上下文中。
  该方法还会返回一个结果，指示交易是否已包含在区块中。
- `finalize_block` 用于完成当前区块的构建。
- `finalize_block_with_context` 用于在指定的执行上下文中完成当前区块的构建。
- `inherent_extrinsics` 用于将固有外部函数交易包含在当前区块中。
  固有交易类型因链而异。
- `inherent_extrinsics_with_context` 用于将固有外部函数交易包含在当前区块和指定的执行上下文中。
  固有交易类型因链而异。
- `check_inherents` 用于检查固有交易是否有效。
- `check_inherents_with_context` 用于检查指定的执行上下文中固有交易是否有效。

## GrandpaApi

使用 `GrandpaApi` 将 GRANDPA 最终确定协议中的权威集更改集成到运行时中。
GRANDPA 最终确定协议通过指定一些区块的延迟来发出对权威集的更改信号。
然后，在最终确定指定数量的区块后，这些更改会自动应用于运行时。

此 API 提供以下方法：

- `grandpa_authorities` 用于获取 GRANDPA 最终确定的当前权威和权重。
- `grandpa_authorities_with_context` 用于在指定的执行上下文中获取 GRANDPA 最终确定的当前权威和权重。
- `current_set_id` 用于获取当前 GRANDPA 权威集标识符。
- `current_set_id_with_context` 用于在指定的执行上下文中获取当前 GRANDPA 权威集标识符。

`GrandpaApi` 还提供用于提交交易以报告行为不端证据和相关的密钥所有权证明的方法。
有关这些方法的信息，请参阅 [`GrandpaApi`](https://paritytech.github.io/substrate/master/sp_consensus_grandpa/trait.GrandpaApi.html)。

## NominationPoolsApi

使用 `NominationPoolsApi` 获取有关提名池和提名池成员的信息。

此 API 提供以下方法：

- `pending_rewards` 用于获取具有指定 AccountId 的提名池成员的待处理奖励。
- `pending_rewards_with_context` 用于在指定的执行上下文中获取具有指定 AccountId 的提名池成员的待处理奖励。

## OffchainWorkerApi

使用 `OffchainWorkerApi` 启动 [链下工作者操作](/learn/offchain-operations/)。

此 API 提供以下方法：

- `offchain_worker` 用于启动指定区块头的链下任务。
- `offchain_worker_with_context` 用于启动指定区块头和执行上下文的链下任务。

## SessionKeys

使用 `SessionKeys` API 生成和解码 [会话密钥](/learn/accounts-addresses-keys/)
(https://paritytech.github.io/substrate/master/sp_session/trait.SessionKeys.html)

此 API 提供以下方法：

- `generate_session_keys` 用于生成一组会话密钥。
  如果您使用指定的种子生成密钥，则种子必须是有效的 `utf8` 字符串。
  您应该将生成的密钥存储在运行时外部函数公开的密钥库中。
  该方法将公钥作为连接的 SCALE 编码值返回。

- `generate_session_keys_with_context` 用于在指定的执行上下文中生成一组会话密钥。
  如果您使用指定的种子生成密钥，则种子必须是有效的 `utf8` 字符串。
  您应该将生成的密钥存储在运行时外部函数公开的密钥库中。
  该方法将公钥作为连接的 SCALE 编码值返回。

- `decode_session_keys` 用于解码指定的公共会话密钥。
  该方法返回原始公钥列表和密钥类型。

- `decode_session_keys_with_context` 用于在指定的执行上下文中解码指定的公共会话密钥。
  该方法返回原始公钥列表和密钥类型。

## TaggedTransactionQueue

使用 `TaggedTransactionQueue` API 验证交易队列中的交易。

此 API 提供以下方法：

- `validate_transaction` 用于验证指定的交易是否是给定 `block_hash` 参数指定的状态的有效交易。

- `validate_transaction_with_context` 用于在指定的执行上下文中验证指定的交易是否是给定 `block_hash` 参数指定的状态的有效交易。

## TransactionPaymentApi

使用 `TransactionPaymentApi` 查询运行时以获取有关交易和交易费用的信息。

此 API 提供以下方法：

- `query_info` 用于返回有关已调度到运行时的指定交易的信息。
- `query_info_with_context` 用于在指定的执行上下文中返回有关已调度到运行时的指定交易的信息。
- `query_fee_details` 用于返回有关指定交易的交易费用的信息。
- `query_fee_details_with_context` 用于在指定的执行上下文中返回有关指定交易的交易费用的信息。

## 接下来的步骤

- [运行时开发](/learn/runtime-development/)
- [FRAME 宏](/reference/frame-macros)
- [impl_runtime_apis](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html)
