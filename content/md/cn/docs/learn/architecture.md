---
title: 架构和 Rust 库
description: 介绍 Substrate 节点的核心组件。
keywords:
---

如[区块链基础](/learn/blockchain-basics)中所述，区块链依赖于一个去中心化的计算机网络（称为节点），这些计算机相互通信。

由于节点是任何区块链的核心组件，因此了解使 Substrate 节点独一无二的因素非常重要，包括默认提供的核心服务和库，以及如何自定义和扩展节点以适应不同的项目目标。

## 客户端和运行时

在高级别上，Substrate 节点包含两个主要部分：

- 具有**外部节点服务**的**核心客户端**，它处理网络活动，例如对等发现、管理交易请求、与对等节点达成共识以及响应 RPC 调用。

- 包含用于执行区块链状态转换函数的所有业务逻辑的**运行时**。

下图以简化形式说明了这种职责分离，以帮助您可视化架构以及 Substrate 如何为构建区块链提供模块化框架。

![Substrate 架构](/media/images/docs/simplified-architecture.png)

## 客户端外部节点服务

核心客户端包含多个外部节点服务，这些服务负责在运行时之外发生的活动。
例如，核心客户端中的外部节点服务处理对等发现、管理交易池、与其他节点通信以达成共识以及响应来自外部世界的 RPC 请求。

核心客户端服务处理的一些最重要的活动涉及以下组件：

- [存储](/learn/state-transitions-and-storage/): 外部节点使用简单高效的键值存储层来持久化 Substrate 区块链的不断变化的状态。

- [对等网络](/learn/networks-and-nodes/): 外部节点使用 Rust 实现的[`libp2p 网络栈`](https://libp2p.io/)与其他网络参与者进行通信。

- [共识](/learn/consensus/): 外部节点与其他网络参与者进行通信，以确保他们就区块链的状态达成一致。

- [远程过程调用 (RPC) API](/build/remote-procedure-calls/): 外部节点接受传入的 HTTP 和 WebSocket 请求，以允许区块链用户与网络交互。

- [遥测](/maintain/monitor/): 外部节点通过嵌入式[Prometheus](https://prometheus.io/)服务器收集并提供对节点指标的访问。

- [执行环境](/build/build-process/): 外部节点负责为运行时选择执行环境（WebAssembly 或原生 Rust），然后分派调用到所选运行时。

Substrate 通过其核心区块链组件提供用于处理这些活动的默认实现。
原则上，您可以使用自己的代码修改或替换任何组件的默认实现。
实际上，应用程序很少需要更改任何底层区块链功能，但 Substrate 允许您进行更改，因此您可以自由地在您认为合适的地方进行创新。

执行这些任务通常需要客户端节点服务与运行时进行通信。
这种通信是通过调用专门的[运行时 API](/reference/runtime-apis/)来处理的。

## 运行时

运行时决定交易是有效还是无效，并负责处理区块链状态的更改。
来自外部的请求通过客户端进入运行时，运行时负责状态转换函数以及存储结果状态。

由于运行时执行它接收的函数，因此它控制交易如何包含在区块中以及区块如何返回到外部节点以进行闲聊或导入到其他节点。
本质上，运行时负责处理链上发生的所有事情。
它也是构建 Substrate 区块链的节点的核心组件。

Substrate 运行时旨在编译为[WebAssembly (Wasm)](/reference/glossary#webassembly-wasm) 字节码。
此设计决策使：

- 支持无分叉升级。
- 多平台兼容性。
- 运行时有效性检查。
- 中继链共识机制的验证证明。

与外部节点具有向运行时提供信息的方式类似，运行时使用专门的[主机函数](https://paritytech.github.io/substrate/master/sp_io/index.html)与外部节点或外部世界进行通信。

## 核心库

为了在节点模板中保持简单，区块链的许多方面都配置了默认实现。
例如，存在网络层、数据库和共识机制的默认实现，您可以按原样使用它们来运行区块链，而无需进行大量自定义。
但是，底层架构的库为定义您自己的区块链组件提供了很大的灵活性。

就像节点包含两个主要部分（核心客户端和运行时）提供不同的服务一样，Substrate 库也分为三个主要职责领域：

- 用于外部节点服务的核心客户端库。
- 用于运行时的 FRAME 库。
- 用于库之间通信的底层函数和接口的原始库。

下图说明了库如何反映核心客户端外部节点和运行时职责，以及**原始库**如何提供这两个库之间的通信层。

![用于外部节点和运行时的核心节点库](/media/images/docs/libraries.png)

### 核心客户端库

使 Substrate 节点能够处理其网络职责（包括共识和区块执行）的库是 Rust 板条箱，这些板条箱在板条箱名称中使用 `sc_` 前缀。
例如，[`sc_service`](https://paritytech.github.io/substrate/master/sc_service/index.html) 库负责为 Substrate 区块链构建网络层，管理网络参与者与交易池之间的通信。

### 用于运行时的 FRAME 库

使您能够构建运行时逻辑以及对传入和传出运行时的信息进行编码和解码的库是 Rust 板条箱，这些板条箱在板条箱名称中使用 `frame_` 前缀。

`frame_*` 库为运行时提供基础设施。
例如，[`frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html) 库提供了一组用于与其他 Substrate 组件交互的基本函数，而[`frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html) 使您能够声明运行时存储项、错误和事件。

除了 `frame_*` 库提供的基础设施之外，运行时还可以包含一个或多个 `pallet_*` 库。
每个使用 `pallet_` 前缀的 Rust 板条箱都代表一个单独的 FRAME 模块。
在大多数情况下，您使用 `pallet_*` 库来组装您想要包含在区块链中的功能，以适应您的项目。

您可以使用**原始库**构建 Substrate 运行时，而无需使用 `frame_*` 或 `pallet_*` 库。
但是，`frame_*` 或 `pallet_*` 库提供了最有效的路径来组合 Substrate 运行时。

### 原始库

在 Substrate 架构的最低级别，存在原始库，这些库使您可以控制底层操作，并使核心客户端服务与运行时之间能够进行通信。
原始库是 Rust 板条箱，这些板条箱在板条箱名称中使用 `sp_` 前缀。

原始库提供最低级别的抽象来公开核心客户端或运行时可以用来执行操作或相互交互的接口。

例如：

- [`sp_arithmetic`](https://paritytech.github.io/substrate/master/sp_arithmetic/index.html) 库为运行时定义了定点算术原语和类型。
- [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html) 库提供了一组可共享的 Substrate 类型。
- [`sp_std`](https://paritytech.github.io/substrate/master/sp_std/index.html) 库从 Rust 标准库导出原语，以使它们可用于依赖运行时的任何代码。

## 模块化架构

核心 Substrate 库的分离为编写区块链逻辑提供了一个灵活且模块化的架构。
原始库提供了一个基础，核心客户端和运行时都可以构建在该基础之上，而无需直接相互通信。
原始类型和特性在它们自己的单独板条箱中公开，因此它们可用于外部节点服务和运行时组件，而不会引入循环依赖问题。

## 下一步去哪里

现在您已经熟悉了用于构建和与 Substrate 节点交互的架构和库，您可能想更深入地探索这些库。
要了解有关任何库的技术细节的更多信息，您应该查看该库的[Rust API](https://paritytech.github.io/substrate/master/) 文档。
