---
title: 库介绍
description:
keywords:
---

在使用节点模板时，你不需要了解任何关于底层架构或正在使用的库的知识，因为基本组件已经组装好并可以使用了。
但是，如果你想设计和构建自定义区块链，你可能希望熟悉可用的库并了解这些不同库的作用。

在 [架构](/learn/architecture/) 中，你了解了 Substrate 节点的核心组件以及节点的不同部分如何承担不同的责任。
在更技术层面上，节点不同层之间职责的分离体现在用于构建基于 Substrate 的区块链的核心库中。
下图说明了库如何反映外部节点和运行时的职责，以及**基元**库如何在这两者之间提供通信层。

![外部节点和运行时的核心节点库](/media/images/docs/libraries.png)

## 核心节点库

使 Substrate 节点能够处理其网络责任（包括共识和块执行）的库是 Rust crate，其 crate 名称中使用 `sc_` 前缀。
例如，[`sc_service`](https://paritytech.github.io/substrate/master/sc_service/index.html) 库负责为 Substrate 区块链构建网络层，管理网络参与者和交易池之间的通信。

提供外部节点和运行时之间通信层的库是 Rust crate，其 crate 名称中使用 `sp_` 前缀。
这些库协调需要外部节点和运行时交互的活动。
例如，[`sp_std`](https://paritytech.github.io/substrate/master/sp_std/index.html) 库从 Rust 的标准库中获取有用的基元，并使它们可用于依赖运行时的任何代码。

使你能够构建运行时逻辑以及编码和解码传入和传出运行时信息的库是 Rust crate，其 crate 名称中使用 `frame_` 前缀。
`frame_*` 库为运行时提供基础设施。
例如，[`frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html) 库提供了一组用于与其他 Substrate 组件交互的基本函数，而 [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html) 使你能够声明运行时存储项、错误和事件。

除了 `frame_*` 库提供基础设施之外，运行时还可以包含一个或多个 `pallet_*` 库。
每个使用 `pallet_` 前缀的 Rust crate 都代表一个单独的 FRAME 模块。
在大多数情况下，你使用 `pallet_*` 库来组装你想要包含在区块链中的功能，以适应你的项目。

你可以使用 `sp_*` 核心库公开的基元来构建 Substrate 运行时，而无需使用 `frame_*` 或 `pallet_*` 库。
但是，`frame_*` 或 `pallet_*` 库提供了组合 Substrate 运行时的最有效途径。

## 模块化架构

核心库的分离为编写区块链逻辑提供了一个灵活且模块化的架构。
基元库提供了一个基础，外部节点和运行时都可以在其上构建，而无需直接相互通信。
基元类型和特性在其自己的单独 crate 中公开，因此它们可用于外部节点和运行时组件，而不会引入循环依赖问题。

## 前端库

除了使你能够构建基于 Substrate 的区块链的核心库之外，还有一些客户端库可用于与 Substrate 节点交互。
你可以使用客户端库来构建特定于应用程序的前端。
通常，客户端库公开的功能是在 Substrate 远程过程调用 (RPC) API 之上实现的。
有关使用元数据和前端库构建应用程序的更多信息，请参阅 [应用程序开发](/build/application-development/#rpc-apis)。

## 接下来的步骤

现在你已经熟悉了用于构建和与 Substrate 节点交互的库，你可能希望更深入地探索这些库。
要了解任何库的技术细节，你应该查看该库的 [Rust API](https://paritytech.github.io/substrate/master/) 文档。
