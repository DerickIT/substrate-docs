---
title: 术语表
description: 定义并解释特定于区块链技术或 Substrate 生态系统的概念和术语。
keywords:
---

本术语表定义并解释特定于区块链技术或 Substrate 生态系统的概念和术语。

## 自适应仲裁偏差 (AQB)

提供一种机制，用于根据投票参与率调整公投的通过阈值。
自适应仲裁偏差允许更灵活的治理，因为它消除了对投票目的进行任意仲裁的要求，这会产生不良的治理机制。
自适应仲裁偏差在 [Democracy pallet](/reference/frame-pallets#democracy) 中实现。
Democracy pallet 提供了链上机构（例如 [集体](#council)）或单个代币持有者——使用正面、负面或中性偏差来调用公投的接口。

使用**正面参与率偏差**时，随着更多投票的投出，通过阈值会_降低_，因此更高的参与率会增加公投通过的可能性。
使用**负面参与率偏差**时，随着更多投票的投出，通过阈值会_增加_。
负面参与率偏差有时也称为“默认通过”位置，因为如果投票机构漠不关心，则公投默认通过。
**中性参与率偏差**指定简单的多数通过阈值。

## 聚合

在 [FRAME](#frame) 的上下文中，_聚合_ 或 _[pallet](#pallet) 聚合_ 是将来自多个运行时模块的类似类型组合成单个类型。
Pallet 聚合允许表示每个模块的类似类型。
包含聚合类型的调用有时被称为_外部调用_或对_外部对象_的调用。
目前，有六种可以聚合的数据类型：

- `Call` 用于可以使用一组参数调用的已发布函数。
- `Error` 用于指示函数调用 (`Call`) 失败原因的消息。
- `Event` 用于描述状态更改的 pallet 发出的事件。
- `Log` 用于可扩展的标题项。
- `Metadata` 用于允许检查上述信息的信息。
- `Origin` 用于函数调用 (`Call`) 的来源。

## 批准投票

投票系统，其中投票者可以投票给任意数量的候选人。
获得最高总票数的候选人获胜。
在批准投票中，值得注意的是：

- 对所有候选人投票与对任何候选人都不投票相同。
- 可以通过投票给所有其他候选人来反对单个候选人。

批准投票由 [FRAME Elections Phragmen pallet](/reference/frame-pallets#elections-phragmen) 用作许多基于 Substrate 的链上的管理 [Council](#council)。

## 作者

描述负责创建 [区块](#block) 的 [节点](#node)。
区块作者也称为_区块生产者_。
在工作量证明区块链中，这些节点称为_矿工_。

## 权威

充当集体以管理 [区块链](#blockchain) 网络上的 [共识](#consensus) 的 [节点](#node)。
在 [权益证明](#nominated-proof-of-stake-npos) 区块链中（例如，使用 [FRAME](#frame) 的 [Staking pallet](/reference/frame-pallets#staking) 的区块链），权威是通过代币加权提名和投票系统确定的。

术语_权威_和_[验证者](#validator)_ 有时似乎指的是同一事物。
但是，_验证者_ 是一个更广泛的术语，可以包括链维护的其他方面，例如 Parachain 验证。
一般来说，权威是验证者的（非严格）子集，许多验证者都是权威。

## 权威轮 (Aura)

确定性 [共识](#consensus) 协议，其中 [区块](#block) 生产仅限于轮流创建区块的旋转权威列表。
使用权威轮 (Aura) 共识，假设大多数在线权威是诚实的。

阅读 [Aura 共识算法的官方维基文章](https://openethereum.github.io/Aura) 以了解更多信息。

Aura 协议通常与 GRANDPA 结合使用作为 [混合共识](#hybrid-consensus) 协议，其中 [Aura](#aura) 用于区块生产和短期 [概率最终性](#probabilistic-finality)，而 [确定性最终性](#deterministic-finality) 由 [GRANDPA](#grandpa) 提供。

## 区块链扩展盲分配 (BABE)

一种类似于 [Aura](#aura) 的 [区块创作](#author) 协议。
但是，使用区块链扩展盲分配 (BABE) 协议，[权威](#authority) 会根据可验证随机函数 (VRF) 赢得 [时隙](#slot)，而不是轮询选择方法。
获胜的权威可以选择一条链并为其提交一个新区块。

请参考其 [官方 Web3 基金会研究文件](https://research.web3.foundation/Polkadot/protocols/block-production/Babe) 以了解有关 BABE 的更多信息。

## 区块

区块是区块链的单个元素，其中包含可能导致状态更改的有序指令集（通常以交易的形式）。

每个区块都由 [密码学摘要](#cryptographic-primitives)（哈希）标识，并包含指向其父区块哈希的反向指针。
每个区块都有一个 [标题](#header) 和一个包含已执行指令的主体，这些指令在某些参数内被包含在区块中。
区块最常被可视化为一个垂直堆栈，其中每个区块都引用其父区块，直到出现必须使用 [分叉选择规则](/learn/consensus#fork-choice-rules) 和可选的 [最终性](#finality) 机制来解决的冲突。

## 区块高度

因为区块通常被可视化为垂直堆栈，所以区块高度用于描述特定区块与第一个区块的距离。
区块链中任何特定区块的区块高度都指示有多少区块位于该区块之前。

类似地，术语_顶部_或_尖端_用于描述链中最近添加的区块。

## 区块编号

与区块高度类似，区块编号用于描述特定区块与第一个区块的距离。
区块链中任何特定区块的区块编号都指示有多少区块位于该区块之前。

## 区块链

描述一个分布式计算机网络，该网络使用 [密码学](#cryptographic-primitives) 允许一组参与者在系统状态随时间演变时的 [共识](#consensus) 上达成信任。
构成区块链网络的计算机称为 [节点](#node)。

## 拜占庭容错 (BFT)

定义分布式计算机网络在一定比例的 [节点](#node) 或 [权威](#authority) 出现缺陷或恶意行为时保持运行的能力。
通常，如果分布式网络可以在假设最多三分之一的节点出现缺陷、离线、积极恶意并作为协调攻击的一部分的情况下保持功能，则该网络被认为是拜占庭容错的。

### 拜占庭故障

由于节点故障超过达成共识所需节点比例而导致网络服务丢失。

### 实用拜占庭容错 (pBFT)

拜占庭容错的早期方法。pBFT 系统最多可以容忍三分之一的参与者的拜占庭行为。
此类系统的通信开销为 `O(n²) `，其中 `n` 是系统中节点（参与者）的数量。

## 盲分配

用于描述在 [共识](#consensus) 机制中支持多个相互排斥选项的错误或恶意行为类型。

## ethash

某些 [工作量证明](#proof-of-work) [共识](#consensus) 系统（例如以太坊区块链）使用的函数。
它是由 [Tim Hughes 领导的团队](https://github.com/ethereum/ethash/graphs/contributors) 开发的。

## 事件

一种记录方法，为了离线世界的好处，记录某些特定的 [状态](#state) 转换已经发生。
在 [FRAME](#frame) 的上下文中，事件是可以由每个 [pallet](#pallet) 单独定义的可组合数据类型。
FRAME 中的事件实现为一组瞬态存储项，这些项在区块执行后立即进行检查，并在区块初始化期间重置。

## 执行器

一种在给定的 [运行时](#runtime) 中使用一组依赖项执行函数调用的方法。
Substrate 中有两个编排引擎，_WebAssembly_ 和_本机_。

- _本机执行器_ 使用嵌入在 [节点](#node) 中的本机编译运行时来执行调用。
  这是一个性能优化，最新的节点可以利用它。

- _WebAssembly 执行器_ 使用 [Wasm](#webassembly-wasm) 二进制文件和 Wasm 解释器来执行调用。
  无论区块链 [节点](#node) 的版本如何，该二进制文件都保证是最新的，因为它会保存在基于 Substrate 的链的 [状态](#state) 中。

## 外部函数

区块链外部的数据，包含在 [区块](#block) 中。
典型的 Substrate 链具有包含 [`Call`](#call) 值的外部函数。
一般来说，有两种类型的外部函数：

- 已签名或未签名的 [交易](#transaction)。
- 由 [区块作者](#author) 插入的固有数据。

## 生存存款

[Balances pallet](/reference/frame-pallets#balances) 中允许帐户拥有的最低余额。
无法使用低于生存存款金额的余额创建帐户。
如果帐户余额低于此金额，则 Balances pallet 将使用 [FRAME 系统 API](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#method.dec_ref) 来删除其对该帐户的引用。
如果删除了对帐户的所有引用，则可以 [清除](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#method.allow_death) 该帐户。

## 最终性

[共识](#consensus) 的一部分，使区块链的持续进展不可逆转。
在 [区块](#block) 最终确定后，它封装的所有 [状态](#state) 更改都是不可逆转的，除非进行硬分叉。
共识算法_必须_保证最终确定的区块永远不需要回滚。
但是，不同的共识算法可以定义不同的最终确定方法。

在使用**确定性最终性**的共识协议中，当包含区块时，每个区块都保证是该链的规范区块。
确定性最终性在不可用完整链的情况下是理想的，例如在 [轻客户端](#light-client) 的情况下。
[GRANDPA](#grandpa) 是 [Polkadot 网络](#polkadot-network) 使用的确定性最终性协议。

在使用**概率最终性**的共识协议中，最终性以概率表示，用 `p` 表示，表示建议的区块（用 `B` 表示）将保留在规范链中的概率。
随着在 `B` 之上生成更多区块，`p` 接近 1。

在使用**即时最终性**的共识协议中，最终性在区块生成后立即得到保证。
这种类型的非概率共识倾向于使用 [实用拜占庭容错 (pBFT)](#practical-byzantine-fault-tolerance-pbft)，并且具有昂贵的通信要求。

## 分叉

表示区块链可能采取的不同路径。
如果两个或多个 [区块](#block) 具有相同的父区块但状态不同，则区块链无法继续前进，直到解决这些差异。
未解决的分叉会将区块链拆分为两条单独的链。
通过解决不同的分叉，您可以确保只存在一条规范链。

## Flaming Fir

一个基于 Substrate 的 [区块链](#blockchain) 测试网络，用于开发和测试 Substrate 区块链开发框架。
有关访问 Substrate 网络和 Flaming Fir 的更多信息，请参阅 [Polkadot wiki](https://wiki.polkadot.network/docs/maintain-networks#flaming-fir)。

## FRAME

_模块化实体运行时聚合框架_ 的首字母缩写，使开发人员能够从称为 [pallet](#pallet) 的模块化组件创建区块链 [运行时](#runtime) 环境。

运行时开发人员使用以下 [宏](#macro) 与 FRAME 进行交互：

- `#[pallet::event]`
- `#[pallet::error]`
- `#[pallet::storage]`
- `#[frame_support::pallet]`

这些宏使定义自定义 pallet 和组合 pallet 以创建可工作的运行时变得容易，可以使用 [`construct_runtime!`](/reference/frame-macros#construct_runtime) 宏来部署基于 Substrate 的区块链。

[Substrate 代码库](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame) 中使用的约定是在核心 FRAME 模块前加上 `frame_`，在可选 pallet 前加上 `pallet_*`。
例如，前面的宏都在 [`frame_support`](/reference/frame-pallets#support-library) 模块中定义，并且所有基于 FRAME 的运行时_都必须_包含 [`frame_system`](/reference/frame-pallets#system-library) 模块。
在使用 `frame_support::construct_runtime` 宏创建包含 `frame_system` 模块的运行时后，可以使用可选的 pallet（例如 [Balances](/reference/frame-pallets#balances) pallet）来扩展运行时的核心功能。

## 全节点

能够通过执行和验证所有逻辑以安全方式同步区块链的 [节点](#node)。
全节点与 [轻客户端](#light-client) 相反。

## 创世配置

一种指定 [区块链](#blockchain) 初始 [状态](#state) 的机制。
按照惯例，此初始状态或第一个区块通常称为创世状态或创世块。
基于 Substrate 的链的创世配置是通过 [链规范](/build/chain-spec/) 文件完成的。
链规范文件使您可以轻松地使用单个 Substrate 代码库作为多个独立配置链的基础。

## GRANDPA

一种用 Rust 语言实现的 [区块链](#blockchain) [确定性最终性](#deterministic-finality) 机制。
[正式规范](https://github.com/w3f/consensus/blob/master/pdf/grandpa-old.pdf) 由 [Web3 基金会](https://web3.foundation/) 维护。

## 标题

汇总用于总结 [区块](#block) 的信息的结构。
标题主要由 [密码学](#cryptographic-primitives) 信息组成，[轻客户端](#light-client) 使用这些信息来获得链的最小安全但非常高效的同步。

## 混合共识

一种区块链共识协议，它由用于 [区块生产](#author) 和 [最终性](#finality) 的独立或松散耦合机制组成。
混合共识允许链像概率共识协议（例如 [Aura](#aura-aka-authority-round)）一样快速增长，同时保持与 [确定性最终性](#deterministic-finality) 共识协议（例如 [GRANDPA](#grandpa)）相同的安全级别。
一般来说，区块生产算法往往比最终确定机制快。
使区块生产与区块最终确定分开，使 Substrate 开发人员能够更好地控制其链的性能。

## 固有交易

固有交易（有时称为固有事务）是一种特殊的未签名交易类型。
这种类型的交易使区块创作节点能够将不需要直接验证的信息插入到区块中。
只有调用固有交易函数的区块创作节点才能将其数据插入到其区块中。
一般来说，即使无法确定性地验证，验证者也假设使用固有交易插入的数据是有效且合理的。

## JSON-RPC

一种以 JSON 编码的无状态、轻量级远程过程调用协议。JSON-RPC 提供了一种使用 JavaScript 对象表示法在远程系统上调用函数的标准方法。
对于 Substrate，此协议是通过 [Parity JSON-RPC](https://github.com/paritytech/jsonrpc) crate 实现的。

## 密钥库

Substrate 中用于管理密钥的子系统，用于生成新区块。

## Kusama

[Kusama](https://kusama.network/) 是一个基于 Substrate 的 [区块链](#blockchain)，其设计类似于 [Polkadot 网络](#polkadot-network)。
Kusama 是一个 [金丝雀](https://en.wiktionary.org/wiki/canary_in_a_coal_mine) 网络，被称为 [Polkadot 的“狂野表亲”](https://polkadot.network/kusama-polkadot-comparing-the-cousins/)。
作为金丝雀网络，Kusama 预计比 [Westend](#westend) 等测试网络更稳定，但不如 [Polkadot](#polkadot) 等生产网络稳定。

作为金丝雀网络，Kusama 受 [其网络参与者](/reference/frame-pallets#democracy) 控制，旨在足够稳定以鼓励有意义的实验。

## libp2p

一个点对点网络堆栈，允许使用许多传输机制，包括 WebSockets（可在 Web 浏览器中使用）。
Substrate 使用 `libp2p` 网络堆栈的 [Rust 实现](https://github.com/libp2p/rust-libp2p)。

## 轻客户端

一种区块链 [节点](#node)，它不存储 [链状态](#state) 或生成区块。
轻客户端能够验证 [密码学原语](#cryptographic-primitives)，并公开一个 [远程过程调用 (RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call) 服务器，允许区块链用户与区块链网络进行交互。

## 宏

一种编程语言特性，使开发人员能够编写可以命名并一起执行的一系列指令。
[FRAME](#frame) 开发环境为 [Rust](https://doc.rust-lang.org/1.7.0/book/macros.html) 提供了几个 [宏](/reference/frame-macros)，您可以使用这些宏来组合 [运行时](#runtime)。

## 元数据

提供有关系统一个或多个方面的信息的数据。
公开有关 Substrate [区块链](#blockchain) 的信息的元数据使您可以与该系统进行交互。

## 节点

区块链客户端的运行实例。
每个节点都是允许区块链参与者相互交互的 [点对点](https://en.wikipedia.org/wiki/Peer-to-peer) 网络的一部分。
Substrate 节点可以在区块链网络中扮演许多角色。
例如，生成区块的节点为区块链执行 [验证者](#validator) 角色。
运行 [轻客户端](#light-client) 的节点可在资源受限的环境（如 [用户界面](https://github.com/paritytech/substrate-connect) 或嵌入式设备）中促进可扩展的交互。

## 提名权益证明 (NPoS)

一种根据承诺其权益以确保一个或多个区块生产节点正常运行的意愿来确定 [验证者](#validator) 或_[权威](#authority)_ 的方法。

## 预言机

在区块链网络中，预言机是一种将区块链连接到非区块链数据源的机制。
预言机使区块链能够访问和处理来自现有数据源的信息，并整合来自非区块链系统和服务的数据。

## 来源

一种 [FRAME](#frame) 原语，用于标识 [已调度](#dispatch) 函数调用到 [运行时](#runtime) 的来源。
FRAME `system` 模块定义了三个内置 [来源](/build/origins#raw-origins)。
作为 [pallet](#pallet) 开发人员，您还可以定义自定义来源，例如 [Collective pallet](https://paritytech.github.io/substrate/master/pallet_collective/enum.RawOrigin.html) 中定义的那些来源。

## pallet

一个模块，可用于扩展基于 [FRAME](#frame) 的 [运行时](#runtime) 的功能。
Pallet 将特定于域的逻辑与运行时原语（如 [事件](#event) 和 [存储项](#storage-items)）捆绑在一起。

## Parachain

Parachain 是一个 [区块链](#blockchain)，它从 _[中继链](#relay-chain)_ 获取共享基础设施和安全性。
您可以在 [Polkadot Wiki](https://wiki.polkadot.network/docs/en/learn-parachains) 上了解有关 Parachain 的更多信息。

## Polkadot 网络

[Polkadot 网络](https://polkadot.network/) 是一个 [区块链](#blockchain)，它充当异构区块链网络的中心枢纽。
它充当中继链的角色，并通过提供共享基础设施和安全性来支持其他链（[Parachain](#parachain)）。

## 最终性证明

可用于证明特定区块已最终确定的数据。

## 工作量证明

一种 [共识](#consensus) 机制，通过要求网络参与者进行工作来阻止攻击。
例如，一些工作量证明系统要求参与者使用 [Ethash](#ethash) 函数来计算哈希作为已完成工作的证明。

## 中继链

多个区块链异构网络的中心枢纽。
中继链是 [区块链](#blockchain)，它为网络中的其他区块链（[Parachain](#parachain)）提供共享基础设施和安全性。
除了提供 [共识](#consensus) 功能外，中继链还允许 Parachain 进行通信和交换数字资产，而无需相互信任。

## 远程过程调用 (RPC)

一种与计算机程序交互的机制。
远程过程调用使开发人员能够查询远程计算机程序或使用他们提供的参数来调用程序逻辑。
Substrate 节点在 HTTP 和 WebSocket 端点上公开 RPC 服务器。

## 杜鹃花

一种 [即时最终性](#instant-finality)、[拜占庭容错 (BFT)](#byzantine-fault-tolerance-bft) [共识](#consensus) 算法。
是 [pBFT](#practical-byzantine-fault-tolerance-pbft) 用于区块链的多种改编之一。
请参阅其 [GitHub 上的实现](https://github.com/paritytech/rhododendron)。

## Rococo

Polkadot 网络的 [Parachain](#parachain) 测试网络。
Rococco 网络是一个基于 Substrate 的 [区块链](#blockchain)，它是异构区块链网络能力的不断发展的测试平台。

## 运行时

区块链的区块执行逻辑。
运行时为节点提供 [状态转换函数](#state-transition-function-stf)。
在 Substrate 中，运行时存储为 [WebAssembly](#webassembly-wasm) 二进制文件，位于 [链状态](#state) 中。

## 时隙

共识引擎（例如 [Aura](#aura-aka-authority-round) 和 [BABE](#blind-assignment-of-blockchain-extension-babe)）使用的固定相等的时间间隔。
在每个时隙中，允许（或强制）一部分 [权威](#authority) [创作](#author) [区块](#block)。

## 主权账户

中继链生态系统中每个链的唯一帐户标识符。
每个链的主权帐户都是根级别帐户，只能使用 Sudo pallet 或通过治理访问。
帐户标识符是通过连接特定文本字符串的 Blake2 哈希和已注册的 Parachain 标识符来计算的。

对于中继链，Parachain 帐户标识符计算为（blake2(para+ParachainID) 与哈希截断为正确长度的连接。
例如，中继链上 Parachain 标识符为 1012 的 Parachain 的帐户标识符为：
字符串到十六进制 para：0x70617261
编码的 Parachain 标识符 1012：f4030000

0x70617261f4030000000000000000000000000000000000000000000000000000
ccount 地址：5Ec4AhPc9b6e965pNRSsn8tjTzuKaKambivxcL7Gz9Gne9YB

对于其他 Parachain，Parachain 帐户标识符计算为（blake2(sibl+ParachainID) 与哈希截断为正确长度的连接。
例如，中继链上 Parachain 标识符为 1012 的 Parachain 的帐户标识符为：
字符串到十六进制 sibl：0x7369626c
编码的 Parachain 标识符 1012：f4030000

0x7369626cf4030000000000000000000000000000000000000000000000000000
帐户地址：5Eg2fntREKHYGgoxvRPxtnEYiUadHjdsfNaPsHdmrsJMVugs

主权帐户最常用于签署发送到生态系统中的中继链或其他链的 XCM 消息。

## SS58 地址格式

SS58 地址格式是基于比特币 [`Base-58-check`](https://en.bitcoin.it/wiki/Base58Check_encoding) 编码的公钥地址。
每个 Substrate SS58 地址都使用 `base-58` 编码值来标识特定基于 Substrate 的链上的特定帐户。
这些由 `base-58` 编码值表示，用于标识特定基于 Substrate 的链上的特定帐户。
[规范 `ss58-registry`](https://github.com/paritytech/ss58-registry) 提供有关不同基于 Substrate 的链使用的地址格式的更多详细信息，包括用于不同网络的网络前缀和网站。

## 权益加权投票

一种民主投票系统，它使用每代币一票的方法来统计投票，而不是每人一票的方法。

## 状态

在区块之间持续存在并可用于作为状态转换函数的一部分创建新区块的密码学安全数据。
在基于 Substrate 的区块链中，状态存储在支持高效创建增量摘要的 [trie](#trie-patricia-merkle-tree) 数据结构中。
此 trie 对 [运行时](#runtime) 公开为 [简单的键/值映射](/learn/state-transitions-and-storage)，其中键和值都可以是任意字节数组。

## 状态转换函数 (STF)

[区块链](#blockchain) 的逻辑，用于确定处理 [区块](#block) 时状态如何变化。
在 Substrate 中，状态转换函数实际上等同于 [运行时](#runtime)。

## 存储项

[FRAME](#frame) 原语，为 [运行时](#runtime) 提供类型安全的数据持久性功能。
请参阅有关 [运行时存储](/build/runtime-storage) 的本文档，以了解有关存储项的更多信息。

## Substrate

一个灵活的框架，用于构建模块化、高效且可升级的 [区块链](#blockchain)。
Substrate 使用 [Rust](https://www.rust-lang.org/) 编程语言编写，并由 [Parity Technologies](https://www.parity.io/) 维护。

## 交易

一种 [外部函数](#extrinsic)，其中包含一个 [签名](/learn/transaction-types)，该签名可用于通过固有地或通过 [签名扩展](/reference/transaction-format#signed-extension) 来验证授权它的帐户。

## 交易纪元

一个可定义的时期（以 [区块](#block) 编号的范围表示），在此期间，交易可以包含在区块中。
交易纪元用于防止在帐户被清除且其重放保护 nonce 重置为零的情况下发生交易重放攻击。

## 交易池

尚未包含在 [区块](#block) 中但已被确定为有效的交易集合。

_标记交易池_ 是一个交易池实现，它允许 [运行时](#runtime) 指定给定交易是否有效，如何对其进行优先级排序，以及它在依赖性和互斥性方面与池中的其他交易的关系。
标记交易池实现的设计是可扩展的，并且足够通用，可以表达 [未花费交易输出 (UTXO)](https://github.com/danforbes/danforbes/blob/master/writings/utxo.md) 和基于帐户的交易模型。

## trie（Patricia Merkle 树）

一种用于表示键值对集的数据结构。

Patricia Merkle trie 数据结构使可以使用密码学哈希来存储和检索数据集中的项。由于对数据集的增量更改会导致新的哈希，因此即使数据集非常大，检索数据也很高效。使用此数据结构，您还可以证明数据集是否包含任何特定的键值对，而无需访问整个数据集。

## 验证者

一个半受信任（或不受信任但激励良好的）参与者，有助于维护 [区块链](#blockchain) 网络。
在 Substrate 中，验证者大致对应于运行 [共识](#consensus) 系统的 [权威](#authority)。
在 [Polkadot](#polkadot-network) 中，验证者还管理其他职责，例如保证数据可用性和验证 [Parachain](#parachain) 候选 [区块](#block)。

## WebAssembly (Wasm)

一种执行架构，允许高效、平台中立地表达确定性、机器可执行的逻辑。
[WebAssembly](https://webassembly.org/) 可以从许多语言编译，包括 [Rust](http://rust-lang.org/) 编程语言。
基于 Substrate 的链使用 WebAssembly 二进制文件来提供可移植的 [运行时](#runtime)，这些运行时可以作为链的 [状态](#state) 的一部分包含在内。

## 权重

基于 Substrate 的区块链中使用的一种约定，用于衡量和管理验证区块所需的时间。
Substrate 将一个权重单位定义为参考硬件上 1 皮秒的执行时间。

最大区块权重应等于目标区块时间的 1/3，分配如下：

- 1/3 用于区块构建
- 1/3 用于网络传播
- 1/3 用于导入和验证

通过定义权重，您可以根据您的用例适当地在每秒交易数量和维护目标区块时间所需的硬件之间做出权衡决策。
因为权重是在运行时定义的，所以您可以使用运行时更新来调整它们，以跟上硬件和软件的改进。

## Westend

Westend 是一个由 Parity 维护的基于 Substrate 的 [区块链](#blockchain)，它充当 [Polkadot 网络](#polkadot-network) 的测试网络。
