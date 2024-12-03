---
title: Substrate 赋能构建者
description:
keywords:
---

Substrate 框架使您能够为项目做出多种类型的设计决策。
Substrate 不会对您要使用的共识模型、您可能要实施的治理形式或是否要收取交易费用做出任何假设。
在每个设计决策中，您都可以自由地在技术创新的复杂性和使用预定义模块和特定于功能的逻辑的开发简便性之间进行选择。

下图说明了构建项目的不同方法如何影响开发时间和复杂性。

![平衡技术自由和开发简便性](/media/images/docs/development-complexity.png)

如该图所示，Substrate 提供了预定义的模板和工具来降低复杂性和开发时间。
但是，Substrate 还公开了构建区块链核心功能的底层原语，因此您拥有完全的技术自由，可以在开发堆栈的任何点进行创新。

如果您想最大限度地减少开发时间和复杂性，则可以使用预定义的 Substrate 节点并进行最少的配置，并且几乎无需开发工作即可启动区块链。
如果您想修改底层组件或操作——例如，共识、网络或存储——则可以从核心原语设计和实现基于 Substrate 的区块链，或者从头开始构建自己的组件。

## 使用 FRAME

当然，最常见的场景介于这两个极端之间，并且涉及使用预定义的模块和库来简化开发。
总的来说，预定义的模块和库构成了 [FRAME](/reference/glossary/#frame) 开发环境。
FRAME 是 Framework for Runtime Aggregation of Modularized Entities 的缩写，FRAME 的主要目的是使您能够创建自定义的 Substrate **运行时**。

通过使用 FRAME，您可以选择和配置要使用的特定模块（称为 **模块**）。
例如，如果您知道您的应用程序需要管理帐户余额，则只需在运行时逻辑的配置中包含 Balances 模块即可。
然后，您可以在运行时的上下文中修改模块的不同方面以适合您的应用程序。
如果当前不存在您需要的功能的模块，则可以使用 FRAME 创建自己的自定义模块，然后将该自定义模块添加到您的自定义运行时。

## 基本运行时设计决策

由于从头开始构建区块链是一项复杂的业务，需要很多专门的技能和知识，因此大多数构建者从一组核心的外部节点组件开始，并将精力集中在构建自定义的运行时逻辑上。
但是，即使您没有构建底层区块链组件，构建自定义运行时也需要您做出一些重要的设计决策。
例如，您可以构建 Substrate 运行时：

- 不使用任何预定义的 FRAME 模块。
- 只使用预定义的 FRAME 模块。
- 使用预定义模块和自定义模块的组合。
- 使用智能合约和预定义的 FRAME 模块。
- 使用预定义模块和自定义模块以及智能合约的组合。

作为设计过程的一部分，您可以探索预定义的 [FRAME 模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame) 以查看它们是否提供了您想要使用或自定义的功能，并确定您可能需要的任何自定义模块。

## 下一步去哪里

探索以下资源以了解更多信息。

#### 告诉我更多

- [区块链基础知识](/main-docs/learn/blockchain-basics/)
- [为什么选择 Substrate？](/main-docs/learn/why-substrate)
- [架构](/main-docs/learn/architecture/)
- [运行时开发](/main-docs/learn/runtime-development/)

#### 指导我

- [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)
- [模拟网络](/tutorials/build-a-blockchain/simulate-network/)
- [添加可信节点](/tutorials/build-a-blockchain/add-trusted-nodes/)

<!--
#### 向我展示（相关视频内容）

*

#### 教我（相关操作方法内容）

*
-->
