---
title: 链外操作
description: 强调如何将来自链外来源的数据集成到链内状态中。
keywords:
---

<div class="warning">
	 请参阅<a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_offchain_workers/index.html">Rust 文档</a>，以获取有关此主题的最新文档。
</div>

在许多用例中，您可能希望从链外来源查询数据或在更新链内状态之前处理数据，而无需使用链内资源。
将链外数据合并的传统方法涉及连接到[预言机](/reference/glossary#oracle)以从某些传统来源提供数据。
虽然使用预言机是处理链外数据源的一种方法，但预言机提供的安全性、可扩展性和基础设施效率存在局限性。

为了使链外数据集成更安全、更高效，Substrate 通过以下功能支持链外操作：

- **链外工作者**是组件的子系统，使执行长时间运行且可能是非确定性任务成为可能，例如：

  - 网站服务请求
  - 数据的加密、解密和签名
  - 随机数生成
  - CPU 密集型计算
  - 链内数据的枚举或聚合

  链外工作者使您能够将可能需要比允许的时间更长的执行时间才能完成的任务从区块处理管道中移出。
  任何可能需要比允许的最大区块执行时间更长的任务都是链外处理的合理候选者。

- **链外存储**是 Substrate 节点本地的存储，链外工作者和链内逻辑都可以访问它：

  - 链外工作者对链外存储具有读写访问权限。
  - 链内逻辑通过链外索引具有写访问权限，但没有读访问权限。
    链外存储允许不同的工作者线程相互通信，并存储不需要在整个网络上达成共识的用户特定或节点特定数据。

- **链外索引**是一种可选服务，允许运行时独立于链外工作者直接写入链外存储。
  链外索引为链内逻辑提供临时存储，并补充链内状态。

## 链外工作者

链外工作者在 Substrate 运行时之外的自己的 Wasm 执行环境中运行。
这种关注点分离确保区块生成不会受到长时间运行的链外任务的影响。
但是，由于链外工作者是在与运行时相同的代码中声明的，因此它们可以轻松地访问链内状态以进行计算。

![链外工作者](/media/images/docs/off-chain-workers-v2.png)

链外工作者可以访问扩展的 API，用于与外部世界进行通信：

- 能够将[交易](https://paritytech.github.io/substrate/master/sp_runtime/offchain/trait.TransactionPool.html)（已签名或未签名）提交到链以发布计算结果。
- 一个功能齐全的 HTTP 客户端，允许工作者访问和从外部服务获取数据。
- 访问本地密钥库以签署和验证语句或交易。
- 所有链外工作者之间共享的另一个本地[键值数据库](https://paritytech.github.io/substrate/master/sp_runtime/offchain/trait.OffchainStorage.html)。
- 用于随机数生成的安全的本地熵源。
- 访问节点的精确[本地时间](https://paritytech.github.io/substrate/master/sp_runtime/offchain/struct.Timestamp.html)。
- 能够休眠和恢复工作。

请注意，链外工作者的结果不受常规交易验证的约束。
因此，您应该确保链外操作包含一个验证方法来确定哪些信息进入链。
例如，您可以通过实现投票、平均或检查发送者签名的机制来验证链外交易。

您还应该注意，链外工作者默认情况下没有任何特定权限或特权，因此代表恶意用户可能利用的潜在攻击向量。
在大多数情况下，在写入存储之前检查交易是否由链外工作者提交，不足以保护网络。
与其假设链外工作者在没有安全措施的情况下可以被信任，不如有意设置限制性权限，以限制对该过程的访问以及它可以执行的操作。

链外工作者在每个区块导入期间生成。
但是，它们在初始区块链同步期间不会执行。

## 链外存储

链外存储始终是 Substrate 节点本地的，不会与任何其他区块链节点共享到链上，也不会受到共识的约束。
您可以使用具有读写访问权限的链外工作者线程或通过链内逻辑使用链外索引来访问存储在链外存储中的数据。

由于链外工作者线程在每个区块导入期间生成，因此在任何给定时间都可能有多个链外工作者线程在运行。
与任何多线程编程环境一样，存在[互斥锁](<https://en.wikipedia.org/wiki/Lock_(computer_science)>) 实用程序，用于在链外工作者线程访问链外存储时对其进行锁定，以确保数据一致性。

链外存储充当链外工作者线程相互通信以及链外和链内逻辑之间通信的桥梁。
它也可以使用远程过程调用 (RPC) 进行读取，因此它适合存储无限增长的数据，而不会过度消耗链内存储。

## 链外索引

在区块链的上下文中，存储最常与链内状态有关。
但是，链内状态很昂贵，因为它必须在网络中的多个节点上达成一致并填充。
因此，您不应该使用链内存储来存储历史数据或用户生成的数据（这些数据会随着时间的推移无限增长）。

为了解决访问历史数据或用户生成数据的需求，Substrate 通过链外索引提供对链外存储的访问。
链外索引允许运行时在不使用链外工作者线程的情况下直接写入链外存储。
您可以通过使用 `--enable-offchain-indexing` 命令行选项启动 Substrate 节点来启用此功能以持久化数据。

与链外工作者不同，链外索引在每次处理区块时都会填充链外存储。
通过在每个区块填充数据，链外索引确保数据始终一致，并且对于启用索引的每个运行节点来说完全相同。

## 下一步去哪里

现在您已经熟悉了链外工作者、链外存储和链外索引如何使您能够处理未存储在链上的数据，您可能想探索以下链外工作者的示例以及如何在运行时开发中使用它们：

- [示例：链外工作者](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/examples/offchain-worker)
- [示例：提交交易](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/transactions.md)
- [示例：使用 HTTP 请求获取数据](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/http-json.md)
- [示例：链外存储](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/storage.md)
- [示例：链外索引](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/indexing.md)
