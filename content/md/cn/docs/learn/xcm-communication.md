---
title: 跨共识消息传递
description: 提供跨共识通信和跨共识消息（XCM）格式的概述。
keywords:
  - XCM
  - XCMP
  - Polkadot
  - 平行链
  - 中继链
---

跨共识通信依赖于一种消息格式——XCM，该格式旨在提供一组通用且可扩展的指令，以完成不同共识系统、交易格式和传输协议之间的交易。

XCM 格式表达了消息的内容。
每条消息由一组**指令**组成，这些指令由**发送者**请求，可以被消息的**接收者**接受或拒绝。
消息格式完全独立于用于发送和接收消息的**消息协议**。

## 消息协议

在 Polkadot 生态系统中，有三种主要的通信通道——消息协议——用于在链之间传输消息：

- 向上消息传递（UMP）使平行链能够向其中继链传递消息。
- 向下消息传递（DMP）使中继链能够向平行链传递消息。
- 跨共识消息传递（XCMP）使平行链能够与连接到同一中继链的其他平行链交换消息。

向上和向下消息传递协议提供了一个垂直消息传递通道。
跨共识消息传递可以被认为是一个水平的——平行链到平行链——传输协议。
由于完整的跨共识消息传递（XCMP）协议仍在开发中，水平中继路由消息传递（HRMP）提供了一种临时解决方案，通过中继链路由旨在传递给平行链的消息。
水平中继路由消息传递（HRMP）旨在作为一种临时解决方案，当 XCMP 发布到生产环境时将被弃用。

尽管这些消息传递协议是 Polkadot 生态系统中链之间通信的主要手段，但 XCM 本身并不受这些传输协议的限制。
相反，无论消息的来源和目的地在哪里，你都可以使用 XCM 表达许多常见类型的交易。
例如，你可以构建从智能合约或 pallets 路由的消息，通过桥接，或使用完全不属于 Polkadot 生态系统的传输协议。

![XCM 提供与消息传递分离的消息内容](/media/images/docs/xcm-channel-overview.png)

由于 XCM 专门设计用于传达接收系统应执行的操作，它可以为许多常见类型的交易提供灵活且无偏见的交易格式。

## XCM 格式的消息

关于使用 XCM 格式的消息，有四个重要原则你应该了解：

- 消息是**异步**的。发送消息后，不期望发送系统等待指示消息已传递或执行的响应。
- 消息是**绝对**的，保证按**顺序**传递和解释，并**高效**执行。
- 消息是**不对称**的，不会返回任何结果给发送者。你只能使用额外的消息单独传达结果给发送者。
- 消息是**无偏见**的，不对消息传递的共识系统做任何假设。

了解这些基本原则后，你可以开始使用 XCM 构建消息。
在 Rust 中，消息定义如下：

```rust
pub struct Xcm<Call>(pub Vec<Instruction<Call>>);
```

如该定义所示，消息只是一个调用，用于执行一组有序的指令。
`Instruction` 类型是一个枚举数据类型，定义的变体顺序通常反映了构建消息时使用的顺序。
例如，`WithdrawAsset` 是第一个变体，因为它通常在有序指令列表中的其他指令（如 `BuyExecution` 或 `DepositAsset`）之前执行。

大多数 XCM 指令使你能够执行常见任务，例如将资产转移到新位置或将资产存入不同账户。
执行这些类型任务的指令允许你构建一致的消息，无论你与之通信的共识系统如何配置，它们都能按预期执行。
然而，你也有灵活性来定制指令的执行方式或使用 `Transact` 指令。

`Transact` 指令允许你执行消息接收者公开的任何可调用函数。
通过使用 `Transact` 指令，你可以对接收系统上的任何函数进行通用调用，但这需要你了解该系统的配置。
例如，如果你想调用另一个平行链的特定 pallet，你必须知道接收运行时的配置，以构建正确的消息以到达正确的 pallet。
这些信息因链而异，因为每个运行时的配置可能不同。

## 在虚拟机中执行

跨共识虚拟机（XCVM）是一个高级虚拟机，具有一个 XCM 执行程序，用于执行接收到的 XCM 指令。
该程序按顺序执行指令，直到运行到末尾或遇到错误并停止执行。

当 XCM 指令执行时，XCVM 通过使用几个专门的寄存器来维护其内部状态。
XCVM 还可以访问执行指令的底层共识系统的状态。
根据执行的操作，XCM 指令可能会更改寄存器、共识系统的状态或两者。

例如，`TransferAsset` 指令指定要转移的资产及其转移位置。
当该指令执行时，**来源寄存器**会自动设置为反映消息的来源，并从该信息中识别应从哪里提取要转移的资产。
执行 XCM 指令时操作的另一个寄存器是**持有寄存器**。
持有寄存器用于临时存储资产，同时等待额外的指令来处理这些资产。

XCVM 中还有几个其他寄存器用于处理特定任务。
例如，有一个盈余权重寄存器用于存储任何费用的高估部分，还有一个退款权重寄存器用于存储已退款的盈余权重部分。
通常，你不能直接修改寄存器中存储的值。
相反，值在 XCM 执行程序启动时设置，并由特定指令在特定情况下或根据特定规则进行操作。
有关每个寄存器包含内容的更多信息，请参见 [XCM 参考](/reference/xcm-reference/)。

### 配置

与 Substrate 和基于 FRAME 的链中的其他组件一样，XCM 执行程序是模块化和可配置的。
你可以使用 `Config` trait 配置 XCM 执行程序的许多方面，并自定义实现以不同方式处理 XCM 指令。
例如，`Config` trait 提供以下类型定义：

```rust
/// 参数化 `XcmExecutor` 的 trait。
pub trait Config {
    /// 外部调用调度类型。
    type Call: Parameter + Dispatchable<PostInfo = PostDispatchInfo> + GetDispatchInfo;
    /// 如何发送后续的 XCM 消息。
    type XcmSender: SendXcm;
    /// 如何提取和存入资产。
    type AssetTransactor: TransactAsset;
    /// 如何从 `OriginKind` 值获取调用来源。
    type OriginConverter: ConvertOrigin<<Self::Call as Dispatchable>::Origin>;
    /// 作为储备信任的 (Location, Asset) 对的组合。
    type IsReserve: FilterAssetLocation;
    /// 作为传送器信任的 (Location, Asset) 对的组合。
    type IsTeleporter: FilterAssetLocation;
    /// 反转位置的方法。
    type LocationInverter: InvertLocation;
    /// 是否执行给定的 XCM。
    type Barrier: ShouldExecute;
    /// 估算 XCM 执行权重的处理程序。
    type Weigher: WeightBounds<Self::Call>;
    /// 购买 XCM 执行权重信用的处理程序。
    type Trader: WeightTrader;
    /// 查询响应的处理程序。
    type ResponseHandler: OnResponse;
    /// 执行后处理持有寄存器中资产的处理程序。
    type AssetTrap: DropAssets;
    /// 处理资产认领指令的处理程序。
    type AssetClaims: ClaimAssets;
    /// 处理版本订阅请求的处理程序。
    type SubscriptionService: VersionChangeNotifier;
}
```

配置设置和 XCM 指令集——消息或更准确地说，是在接收系统上执行的程序——作为 XCM 执行程序的输入。
通过 XCM 构建模块提供的额外类型和函数，XCM 执行程序解释并按提供的顺序逐一执行指令中包含的操作。
下图提供了工作流程的简化概述。

![XCM 配置和执行](/media/images/docs/xcvm-overview.png)

## 位置

由于 XCM 是一种用于不同共识系统之间通信的语言，它必须有一种抽象的方式来以通用、灵活和明确的方式表达位置。
例如，XCM 必须能够识别以下类型活动的位置：

- 指令应在何处执行。
- 资产应从何处提取。
- 接收资产的账户可以在哪里找到。

对于任何这些活动，位置可能在中继链、平行链、外部链、特定链上的账户、特定智能合约或单个 pallet 的上下文中。
例如，XCM 必须能够识别以下类型的位置：

- 层-0 链，例如 Polkadot 或 Kusama 中继链。
- 层-1 链，例如 Bitcoin 或 Ethereum 主网或平行链。
- 层-2 智能合约，例如 Ethereum 上的 ERC-20。
- 平行链或 Ethereum 上的地址。
- 中继链或平行链上的账户。
- 基于 Frame 的 Substrate 链上的特定 pallet。
- 基于 Frame 的 Substrate 链上的单个 pallet 实例。

为了在共识系统的上下文中描述位置，XCM 使用 `MultiLocation` 类型。
`MultiLocation` 类型表达相对于当前位置的位置，由两个参数组成：

- `parents: u8` 描述从当前共识位置向上移动的层数，然后解释 `interior` 参数。
- `interior: InteriorMultiLocation` 描述在按 `parents` 参数指定的相对路径上升后外部共识系统内部的位置。

`InteriorMultiLocation` 使用**连接点**的概念来识别本地共识系统内部的共识系统，每个连接点指定相对于前一个位置的进一步位置。
没有连接点的 `InteriorMultiLocation` 仅指本地共识系统（Here）。
你可以使用连接点来指定 XCM 指令的内部上下文，例如平行链、账户或相对于外部共识的 pallet 实例。

例如，以下参数从中继链的上下文中引用唯一标识符为 1000 的平行链：

```rust
{ "parents": 1,
"interior": { "X1": [{ "Parachain": 1000 }]}
}
```

在此示例中，`parents` 参数上升一级到父链，`interior` 指定具有连接点类型 `Parachain` 和索引 `1000` 的内部位置。

在文本中，MultiLocation 遵循用于描述文件系统路径的约定。
例如，表示为 `../PalletInstance(3)/GeneralIndex(42)` 的 MultiLocation 描述了一个具有一个父级（..）和两个连接点（`PalletInstance{index: 3}` 和 `GeneralIndex{index: 42}`）的 MultiLocation。

有关指定位置和连接点的更多信息，请参见 [通用共识位置标识符](https://github.com/paritytech/xcm-format#7-universal-consensus-location-identifiers)。

## 资产

大多数区块链依赖某种类型的数字资产来提供对网络安全至关重要的经济激励。
一些区块链支持单一的原生资产。
其他区块链允许在链上管理多个资产，例如，在智能合约中定义的资产或非原生代币。
还有一些区块链支持独一无二的收藏品的非同质化数字资产。
为了支持这些不同类型的资产，XCM 必须能够以通用、灵活和明确的方式表达资产。

为了描述链上资产，XCM 使用 `MultiAsset` 类型。
`MultiAsset` 类型指定资产身份以及资产是同质化的还是非同质化的。
通常，资产身份使用具体位置指定。
如果资产是同质化的，定义中包括一个数量。

尽管可以使用抽象标识符来识别资产，但具体标识符是一种明确识别资产的方式，无需全球资产名称注册表。

具体标识符通过其在相对于解释上下文的共识系统中的位置明确识别单个资产。
然而，值得注意的是，具体资产标识符不能在共识系统之间简单复制。
相反，资产是使用每个共识系统的相对路径移动的。
相对路径必须从接收系统的角度构建。

对于原生资产——例如 Polkadot 中继链上的 DOT——资产标识符通常是铸造资产的链或其平行链的上一级（..）。
如果资产在 pallet 内管理，资产标识符使用 pallet 实例标识符和该 pallet 内的索引指定位置。
例如，Karura 平行链可能引用 Statemine 平行链上的一个资产，位置为 `../Parachain(1000)/PalletInstance(50)/GeneralIndex(42)`。

有关指定位置和连接点的更多信息，请参见 [通用资产标识符](https://github.com/paritytech/xcm-format#6-universal-asset-identifiers)。

## 指令

大多数 XCM 指令使你能够构建一致的消息，无论你与之通信的共识系统如何配置，它们都能按预期执行。
然而，你也有灵活性使用 `Transact` 指令来执行消息接收者公开的任何可调用函数。
通过使用 `Transact` 指令，你可以对接收系统上的任何函数进行通用调用，但这需要你了解该系统的配置。
例如，如果你想调用另一个平行链的特定 pallet，你必须知道接收运行时的配置，以构建正确的消息以到达正确的 pallet。
这些信息因链而异，因为每个运行时的配置可能不同。