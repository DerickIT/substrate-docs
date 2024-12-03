---
title: 构建
description: 公开有关如何构建和编译 Substrate 节点的详细信息。
keywords:
---

本节中的主题更详细地探讨了用于构建运行时逻辑的代码，包括用于构建和与节点交互的库和工具，以及更仔细地了解如何编译逻辑以构建 Substrate 节点。

- [智能合约](/build/smart-contracts-strategy/) 总结了如何使用构建智能合约作为在基于 Substrate 的链中进行应用程序开发的方法。
- [自定义模块](/build/custom-pallets) 公开了构成构建自定义模块基础的宏和属性。
- [运行时存储](/build/runtime-storage) 更仔细地介绍了存储结构以及如何导航到运行时中存储的数据。
- [交易、权重和费用](/build/tx-weights-fees) 解释了权重和费用在执行交易中的作用以及如何计算和退款费用的机制。
- [模块耦合](/build/pallet-coupling) 描述了模块如何在运行时中紧密或松散耦合。
- [事件和错误](/build/events-and-errors) 解释如何从运行时发出事件和错误。
- [随机性](/build/randomness) 建议了一些方法，你可以在基于 Substrate 的区块链上运行的应用程序中包含随机性。
- [特权调用和来源](/build/origins) 描述了如何使用预定义或自定义来源来标识函数调用的发起者。
- [远程过程调用](/build/remote-procedure-calls/) 总结了如何使用远程过程调用和 RPC 方法与 Substrate 节点交互。
- [应用程序开发](/build/application-development/) 介绍了元数据和前端库作为构建在区块链上运行的应用程序的工具的作用。
- [链规范](/build/chain-spec) 讨论了链规范的使用，包括你可以修改和不能修改的内容，以及如何分发自定义链规范。
- [创世配置](/build/genesis-configuration) 描述了创世配置的主要元素。
- [构建过程](/build/build-process) 深入探讨了 Rust 代码如何编译成 Rust 二进制文件和 WebAssembly 目标以及如何使用这两个目标来优化节点操作的细节。
- [构建确定性运行时](/build/build-a-deterministic-runtime) 解释了如何使用 Substrate 运行时工具箱 (`srtool`) 和 Docker 来构建基于 Substrate 的链的 WebAssembly 运行时。
- [排查代码问题](/build/troubleshoot-your-code) 突出显示了用于排查问题和遵循最佳实践的通用和特定于 Substrate 的编码技术。
