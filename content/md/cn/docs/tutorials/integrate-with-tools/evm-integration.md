---
title: 以太坊集成
description:
keywords:
---

通过使用 Frontier 项目的crates并将 EVM 和以太坊模块添加到您的运行时，您可以构建一个基于 Substrate 的区块链，该区块链支持基于以太坊的帐户并允许执行基于 Solidity 的智能合约。

以太坊虚拟机 (EVM) 是一台虚拟计算机，它具有使以太坊网络参与者能够存储数据并就数据状态达成一致的组件。
对于基于 Substrate 的区块链，EVM 的核心职责在**EVM 模块**中实现。
EVM 模块负责执行用 Solidity 等高级语言编写的智能合约的以太坊合约字节码，然后编译为 EVM 字节码。
下图简要概述了如何将 EVM 模块和以太坊 RPC 调用集成到您的 Substrate 运行时中。

![与以太坊兼容的运行时架构](/media/images/docs/tutorials/evm-ethereum/pallet-evm.png)

除了 EVM 模块外，以太坊模块还负责存储以太坊格式的区块、交易回执和交易状态。
当用户提交原始以太坊交易时，该交易将首先通过调用运行时中`pallet_ethereum`中的`transact`函数转换为 Substrate 交易。

![以太坊模块](/media/images/docs/tutorials/evm-ethereum/pallet-ethereum.png)

请注意，以太坊帐户和 Substrate 帐户对于使用单个私钥并不直接兼容。
有关将以太坊帐户和密钥映射到 Substrate 帐户和密钥的信息，请参阅 Moonbeam 文档中的[统一帐户](https://docs.moonbeam.network/learn/unified-accounts/#substrate-evm-compatible-blockchain)。

## 以太坊特定的运行时 API 和 RPC

运行时存储所有可以查询的以太坊格式的信息。
您可以使用节点 RPC 服务器和运行时 API 和 RPC 客户端调用来调用运行时并检索该信息。

![访问以太坊格式信息的远程过程调用](/media/images/docs/tutorials/evm-ethereum/rpc.png)

## Frontier 区块导入

![区块导入过程](/media/images/docs/tutorials/evm-ethereum/block-import.png)

## 下一步

- [Moonbeam：以太坊兼容性](https://docs.moonbeam.network/learn/features/eth-compatibility/)
- [以太坊虚拟机 (EVM)](https://ethereum.org/en/developers/docs/evm/)
- [访问 EVM 帐户](/tutorials/integrate-with-tools/access-evm-accounts/)
- [Substrate EVM 实用程序](https://github.com/paritytech/frontier/blob/master/template/utils/README.md#substrate-evm-utilities)
- [Frontier rpc 和 rpc-core](https://github.com/paritytech/frontier/tree/master/client/)
- [Frontier 共识](https://github.com/paritytech/frontier/tree/master/primitives/consensus)
