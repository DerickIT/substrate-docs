---
title: 构建应用程序逻辑
description: 演示如何添加模块以自定义 Substrate 运行时环境。
keywords:
---

**构建应用程序逻辑** 教程重点介绍了如何使用模块自定义运行时，包括如何向运行时添加简单和复杂的模块，以及如何将模块与智能合约结合使用。
您将学习如何：

- [向运行时添加模块](/tutorials/build-application-logic/add-a-pallet/) 介绍了向节点模板运行时添加简单的预定义模块的常见步骤。
- [在自定义模块中使用宏](/tutorials/build-application-logic/use-macros-in-a-custom-pallet) 说明了如何使用宏创建自定义模块。
- [为调用指定来源](/tutorials/build-application-logic/specify-the-origin-for-a-call) 演示了如何指定用作函数调用发起者的帐户。
- [添加链下工作程序](/tutorials/build-application-logic/add-offchain-workers/) 说明了如何修改模块以包含链下工作程序，以及如何配置模块和运行时以使链下工作程序能够提交更新链上状态的事务。
- [发布自定义模块](/tutorials/build-application-logic/publish-custom-pallets/) 说明了如何发布自定义模块和crates，以便社区可以使用它们。

如果您想尝试为您的区块链进行智能合约开发，则应使用预配置的[substrate-contracts-node](https://github.com/paritytech/substrate-contracts-node)，而不是标准节点模板。
支持智能合约的模块与当前节点模板之间存在兼容性问题。
解决这些兼容性问题需要对标准节点模板的配置进行大量更改，包括使用所有crates的旧版本并修改多个文件。
要直接深入智能合约开发，请参阅[开发智能合约](/tutorials/smart-contracts/)中的教程。
