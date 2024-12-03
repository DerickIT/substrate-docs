---
title: 访问 EVM 帐户
description: 说明如何通过 Substrate 区块链节点集成对基于以太坊的帐户和合约的访问。
keywords:
  - 以太坊
  - 合约
  - solidity
  - evm
  - ethersjs
  - web3js
  - hardhat
  - truffle
  - ganache
---

本教程说明了如何使用来自[Frontier](https://github.com/paritytech/frontier)项目的crates来构建**与以太坊兼容的**区块链，该区块链可以访问基于以太坊的帐户并执行基于 Solidity 的智能合约。Frontier 项目的两个主要目标是使您能够执行以下操作：

- 使用本地 Substrate 节点无修改地运行以太坊去中心化应用程序。
- 从以太坊主网导入状态。

本教程使用预定义的节点模板来提供工作环境。
该模板是使用[Frontier 发行指南](https://github.com/paritytech/frontier/blob/master/docs/solo-template-node-release.md)中的说明生成的。

如果您想自己生成独立模板，则可以使用[solo-template-node-release.sh](https://github.com/paritytech/frontier/blob/master/.maintain/solo-template-node-release.sh)模板生成脚本。
如果您使用[frontier](https://github.com/paritytech/frontier)存储库或模板生成脚本构建自己的节点，请注意 Frontier 使用其自身的 Substrate crates版本，您可能需要更新`Cargo`文件中的依赖项以匹配项目中的依赖项。

## 开始之前

在尝试本教程之前，您应该已完成以下 Substrate 教程：

- [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)
- [向运行时添加模块](/tutorials/build-application-logic/add-a-pallet/)
- [在自定义模块中使用宏](/tutorials/build-application-logic/use-macros-in-a-custom-pallet/)

从这些教程中，您应该熟悉如何执行以下任务：

- 启动 Substrate 区块链节点。
- 在运行时添加、删除和配置模块。
- 通过使用 Polkadot-JS 或其他前端连接到节点来提交事务。

在开始本教程之前，您还应该熟悉以下内容：

- 以太坊核心概念和术语
- 以太坊虚拟机 (EVM) 基础知识
- 去中心化应用程序和智能合约
- 模块设计原则

## 创世配置

`frontier-solo-template-node`中的开发[链规范](https://github.com/substrate-developer-hub/frontier-solo-template-node/blob/main/node/src/chain_spec.rs)定义了一个创世块，该块已预先配置了`alice`帐户的 EVM 帐户。
当您以开发模式启动此节点时，`alice`的 EVM 帐户将获得默认数量的以太币。
您将使用此帐户来查看 EVM 帐户详细信息并调用以太坊智能合约。
启动节点后，您将能够使用[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)查看`alice`的 EVM 帐户的详细信息。

## 编译 Frontier 节点

[Frontier 节点模板](https://github.com/substrate-developer-hub/frontier-solo-template-node)提供了一个可工作的开发环境，以便您可以立即开始在 Substrate 上构建。

要编译 Frontier 节点模板：

1. 在您的计算机上打开终端 shell。

1. 通过运行以下命令克隆节点模板存储库：

   ```bash
   git clone https://github.com/substrate-developer-hub/frontier-solo-template-node.git
   ```

1. 通过运行以下命令更改到节点模板目录的根目录：

   ```bash
   cd frontier-solo-template-node
   ```

1. 通过运行以下命令编译节点模板：

   ```bash
   cargo build --release
   ```

## 连接到节点

节点编译完成后，您必须启动节点才能开始浏览预配置的 EVM 帐户。

要启动本地 Substrate 节点：

1. 根据需要在本地计算机上打开终端 shell。

1. 更改到您编译`frontier-solo-template-node`的根目录。

1. 通过运行以下命令以开发模式启动节点：

   ```bash
   ./target/release/frontier-template-node --dev
   ```

   `--dev`命令行选项指定节点使用预定义的`development`链规范运行，该规范包括`alice`的预定义 EVM 帐户以及其他用于测试的帐户。

1. 通过查看终端中显示的输出，验证您的节点是否已成功启动并运行。

   终端应显示类似于此的输出：

   ```text
   2022-07-08 10:06:42 Frontier Node
   2022-07-08 10:06:42 ✌️  version 0.0.0-1b6bff4-x86_64-macos
   2022-07-08 10:06:42 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2021-2022
   2022-07-08 10:06:42 📋 Chain specification: Development
   2022-07-08 10:06:42 🏷  Node name: flippant-boat-0444
   2022-07-08 10:06:42 👤 Role: AUTHORITY
   ...
   ```

1. 使用[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)连接到本地节点。

1. 点击**设置**，然后点击**开发者**。

   ![开发者设置](/media/images/docs/tutorials/evm-ethereum/settings-developer.png)

1. 定义以下帐户信息以创建 EVM `Account`类型并使帐户能够发送事务和检查区块。

   要发送事务，您必须定义`Address`和`LookupSource`设置。
   <br>
   要检查区块，您必须定义`Transaction`和`Signature`设置。

   ```json
   {
      "Address": "MultiAddress",
      "LookupSource": "MultiAddress",
      "Account": {
         "nonce": "U256",
         "balance": "U256"
      },
      "Transaction": {
         "nonce": "U256",
         "action": "String",
         "gas_price": "u64",
         "gas_limit": "u64",
         "value": "U256",
         "input": "Vec<u8>",
         "signature": "Signature"
      },
      "Signature": {
         "v": "u64",
         "r": "H256",
         "s": "H256"
      }
   }
   ```

1. 点击**保存**。

## 使用 RPC 查询余额

配置 EVM 帐户的设置后，您可以使用 Polkadot-JS 应用程序查看有关`alice`的 EVM 帐户的信息。

1. 验证您的节点是否仍在运行，并且[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)已连接到节点。

1. 点击**开发者**，然后选择**RPC 调用**。

1. 在**提交**选项卡上，选择**eth**作为要调用的端点。

1. 从要调用的函数列表中选择`getBalance(address, number)`。

1. 为地址指定`alice`帐户的 EVM 帐户标识符。

   预定义的帐户地址为`0xd43593c715fdd31c61141abd04a99fd6822c8558`。
   帐户的地址是使用[Substrate EVM 实用程序](https://github.com/substrate-developer-hub/frontier-solo-template-node/tree/main/utils/README.md#--evm-address-address)根据`alice`帐户的公钥计算得出的。

1. 点击**提交 RPC 调用**。

   该调用应返回类似于以下内容的输出：

   ```text
   2: eth.getBalance: U256
   340,282,366,920,938,463,463,374,607,431,768,210,955
   ```

## 部署智能合约

现在您已经了解了如何查询以太坊地址的余额，您可能希望探索如何部署和调用以太坊智能合约以及测试相关功能。
本教程使用[Truffle](https://www.trufflesuite.com/truffle)示例合约，该合约定义了一个[ERC-20 代币](https://github.com/substrate-developer-hub/frontier-solo-template-node/blob/main/examples/contract-erc20/truffle/contracts/MyToken.sol)。您还可以使用 Polkadot JS SDK 和[Typescript](https://github.com/substrate-developer-hub/frontier-solo-template-node/tree/main/examples/contract-erc20)创建 ERC-20 代币合约。

1. 创建 ERC-20 合约。

   为方便起见，您可以使用[MyToken.json](https://github.com/substrate-developer-hub/frontier-solo-template-node/blob/main/examples/contract-erc20/truffle/contracts/MyToken.json)中代币合约的已编译`bytecode`来在 Substrate 区块链上部署合约。

1. 验证您的节点是否仍在运行，并且[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)已连接到节点。

1. 点击**开发者**，然后选择**外部函数**。

1. 选择**ALICE**开发帐户作为用于提交交易的帐户。

1. 选择**evm**。

1. 选择**create**函数。

1. 配置函数的参数。

   | 此处 | 指定此内容 |
   | -------- | ------------ |
   | `source` | 0xd43593c715fdd31c61141abd04a99fd6822c8558 |
   | `init` | 来自`MyToken.json`的原始`bytecode`十六进制值 |
   | `value` | 0 |
   | `gasLimit` | 4294967295 |
   | `maxFeePerGas` | 100000000 |

   您可以将可选参数留空。
   `nonce`的值将递增源帐户的已知 nonce，从`0x0`开始。
   根据您选择的函数，您可能需要删除未使用的参数。

1. 点击**提交事务**。

1. 点击**签名并提交**以授权交易。

## 查看智能合约

提交交易后，合约将在网络上部署，您可以使用[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)查看有关它的信息。

1. 验证您的节点是否仍在运行，并且[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)已连接到节点。

1. 点击**网络**，然后选择**资源管理器**。

1. 点击**evm.Created**事件以验证新创建的合约的地址是否为`0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`。

   ![成功创建合约事件](/media/images/docs/tutorials/evm-ethereum/evm-created.png)

   您还可以使用浏览器开发者工具中的控制台查看有关交易的详细信息。

   由于 EVM 合约地址由合约创建者的帐户标识符和 nonce 确定，因此部署合约的地址是使用众所周知的帐户标识符`0xd43593c715fdd31c61141abd04a99fd6822c8558`和`alice`帐户的 nonce `0x0`计算得出的。

1. 点击**开发者**，然后选择**链状态**。

1. 选择**evm**作为要查询的状态和**accountCodes**。

1. 为`alice`帐户指定帐户标识符`0xd43593c715fdd31c61141abd04a99fd6822c8558`，并注意帐户代码为空（`0x`）。

1. 为您使用`alice`开发帐户部署的合约指定合约地址`0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`，并注意合约帐户代码是来自 Solidity 合约的字节码。

## 查看帐户存储

您部署的 ERC-20 合约基于[OpenZeppelin ERC-20 实现](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)。
此合约包含一个构造函数，该构造函数会铸造最大数量的代币并将它们存储在与合约创建者关联的帐户中。

要查询与智能合约关联的帐户存储：

1. 在链状态中，将**evm**作为要查询的状态，选择**accountStorages**。

1. 将 ERC-20 合约地址`0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`指定为第一个参数。

1. 将要读取的存储槽指定为第二个参数`0x045c0350b9cf0df39c4b40400c965118df2dca5ce0fbcf0de4aafc099aea4a14`。

   地址的存储槽是使用[Substrate EVM 实用程序](https://github.com/substrate-developer-hub/frontier-solo-template-node/tree/main/utils/README.md#--erc20-slot-slot-address)根据槽 0 和帐户标识符`0xd43593c715fdd31c61141abd04a99fd6822c8558`计算得出的。

   返回的值应为`0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff`。

   如果您在部署合约后检查`alice`帐户的余额，您会看到已从帐户中提取费用，并且`getBalance(address, number)`调用返回类似于以下内容的值：

   ```text
   340,282,366,920,938,463,463,374,603,530,233,757,803
   ```

## 转移代币

到目前为止，您只使用了`alice`开发帐户。
下一步是使用已部署的合约将代币转移到另一个帐户。

1. 验证您的节点是否仍在运行，并且[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)已连接到节点。

1. 点击**开发者**，然后选择**外部函数**。

1. 选择**ALICE**开发帐户作为用于提交交易的帐户。

1. 选择**evm**。

1. 选择**call**以在 ERC-20 合约上调用`transfer(address, uint256)`函数。

1. 配置函数的参数。

   | 此处 | 指定此内容 |
   | -------- | ------------ |
   | `source` | 0xd43593c715fdd31c61141abd04a99fd6822c8558 |
   | `target` | 0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f |
   | `input` | 0xa9059cbb0000000000000000000000008eaf04151687736326c9fea17e25fc528761369300000000000000000000000000000000000000000000000000000000000000dd |
   | `value` | 0 |
   | `gasLimit` | 4294967295 |
   | `maxFeePerGas` | 100000000 |

   `source`代表持有代币的帐户。
   在本例中，`source`是`alice`的 EVM 帐户，即合约创建者。
   `target`是用于从`alice`向`bob`转移代币的合约地址。

   `input`参数是一个 EVM ABI 编码的函数调用，该调用指定要执行的函数调用（`0xa9059cbb`）以及函数所需的参数。
   对于此函数，参数是`bob`EVM 帐户标识符（`0x8eaf04151687736326c9fea17e25fc5287613693`）和要转移的代币数量（221 或十六进制中的`0xdd`）。

   本教程中的输入值是使用[Remix web IDE](http://remix.ethereum.org)计算得出的。

1. 点击**提交事务**。

1. 点击**签名并提交**以授权交易。

## 验证代币转移

提交交易后，您可以使用[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)验证代币转移。

1. 验证您的节点是否仍在运行，并且[Polkadot-JS 应用程序](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)已连接到节点。

1. 点击**网络**，然后选择**资源管理器**。

1. 点击**evm.Executed**事件以验证已执行合约的地址是否为`0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`。

   ![成功执行事件](/media/images/docs/tutorials/evm-ethereum/evm-executed.png)

1. 点击**开发者**，然后选择**链状态**。

1. 选择**evm**作为要查询的状态和**accountStorages**。

1. 检查合约地址`0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`和存储槽`0x045c0350b9cf0df39c4b40400c965118df2dca5ce0fbcf0de4aafc099aea4a14`的存储。

   ```text
   0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff22
   ```

   如果您在部署合约后检查`alice`帐户的余额，您会看到已从帐户中提取费用，并且`getBalance(address, number)`调用返回类似于以下内容的值：

   ```text
   340,282,366,920,938,463,463,374,603,530,233,366,411
   ```

## 下一步

- [Moonbeam：以太坊兼容性](https://docs.moonbeam.network/learn/features/eth-compatibility/)
- [以太坊集成](/tutorials/integrate-with-tools/evm-integration/)
- [EVM ABI 规范](https://solidity.readthedocs.io/en/latest/abi-spec.html)
- [ERC-20 代币标准](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)
