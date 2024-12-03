---
title: 拍卖和众筹
description:
keywords:
  - parachain
  - slot
  - loan
  - crowdloan
  - auction
---

准备在 Polkadot 或 Kusama 等_生产网络_上启动 Parachain？您必须在 [Parachain 时隙拍卖](https://wiki.polkadot.network/docs/learn-auction) 中获得一个时隙。

> 在尝试进入生产网络之前，请先在 [Rococo 测试网](/tutorials/build-a-parachain/acquire-a-testnet-slot/) 上尝试所有操作！

由于大多数人需要锁定的 DOT 或 KSM 总量相对较大，因此通常使用 [众筹](https://wiki.polkadot.network/docs/learn-crowdloans) 来让社区为参与拍卖的资金池贡献资金。

阅读完以上两篇文章后，请参见以下内容，了解有关配置众筹和竞价的更多面向开发人员的指南。

### 注册为 Parathread

所有 Parachain 都需要首先注册为 Parathread。您将需要：

- 注册 `ParaID` 的押金（取决于网络）

- 预留一个唯一的 `ParaID`。这将分配给下一个可用的 ID。
  此整数将大于 `2000`，因为 `0-999` 保留给 [系统 Parachain](https://wiki.polkadot.network/docs/learn-common-goods#system-level-chains)，而 `1000-1999` 保留给 [公共实用 Parachain](https://wiki.polkadot.network/docs/learn-common-goods#public-utility-chains)。

- 您的 Parachain 创世状态。
  请参阅 [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 中的创世状态导出过程。

- 您的 Parachain Wasm 运行时。
  请参阅 [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 中的 Wasm 运行时导出过程。

步骤如下：

- 前往 **_正确的中继链_** 上的 Polkadot-JS Apps parathreads 部分 [此处](https://polkadot.js.org/apps/#/parachains/parathreads)。

- 预留下一个可用的 `ParaID`。

  ![paraid-reserve.png](/media/images/docs/tutorials/parachains/paraid-reserve.png)

- 成功预留您的 `ParaID` 后，您现在可以注册为 **Parathread**。

  ![register-parathread.png](/media/images/docs/tutorials/parachains/register-parathread.png)

- 外部函数成功后，您将看到 `registrar.Registered` 事件被发出。

  ![parathread-register-success.png](/media/images/docs/tutorials/parachains/parathread-register-success.png)

- 在 Polkadot-JS Apps [Parachains -> Parathreads](https://polkadot.js.org/apps/#/parachains/parathreads) 页面中，您将看到您的 Parathread 注册正在 **加入中**:

  ![parathread-onboarding.png](/media/images/docs/tutorials/parachains/parathread-onboarding.png)

外部函数成功后，需要 [**2 个会话**](#相关设置)才能将链完全加入作为 Parathread。

### Parachain 时隙拍卖

然后可以通过赢得 Parachain 时隙拍卖将 Parathread 转换为 Parachain。
这保证了现在的 Parachain 将始终在其分配的时隙持续时间内拥有其区块数据哈希并包含在中继链区块中（称为验证证明，简称 **PoV**）。

**只有完全加入的 Parathread 才有资格参与 Parachain 时隙拍卖。**
公共物品链可以通过链上治理决策绕过拍卖并成为 Parachain。

#### 相关设置

> **网络中的一些_关键_设置取决于！这些会影响某些 Parachain 操作的计时行为，并且可能会发生变化！**

- 会话长度
- 租赁期长度
- 结束周期
- 当前租赁期索引 = 当前区块编号 / 14400

请注意，Parachain/Parathread 转换为不同状态的转换至少需要 2 个会话，包括加入、退出、升级、降级等。

在 [Polkadot wiki](https://wiki.polkadot.network/docs/learn-crowdloans#starting-a-crowdloan-campaign) 和您目标中继链上当前运行的 [运行时源代码](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/runtime) 中查找这些设置。
在您承诺这些参数之前，_绝对必须_了解这些参数。

#### 投标

任何拥有完全加入的 Parathread 的人都可以竞标以赢得其 `ParaID` 的 Parachain 时隙。
他们需要在时隙拍卖中胜过所有其他参与者。

您可以在 Polkadot-JS App [网络 -> Parachains -> 拍卖](https://polkadot.js.org/apps/#/parachains/auctions) 页面中执行此操作（确保您在正确的网络上）。

选择您的 `ParaID`、您想要竞价的金额以及您想要竞价的时隙：

![parachain-bid.png](/media/images/docs/tutorials/parachains/parachain-bid.png)

### 众筹

您也可以选择利用您的社区并获得他们的支持来赢得 Parachain 时隙拍卖。
在这种情况下，您将启动一个众筹活动，以便您的支持者可以将他们的代币借给您的项目以赢得拍卖。
这些代币将在拍卖过程中被托管，如果获胜，则整个时隙租赁期间也是如此。

您不会为测试网 Parachain 时隙执行此操作，但您可以考虑为主网 Parachain 时隙选择此选项。

#### 启动众筹活动

在下文中，我们正在准备提交一个外部函数来启动众筹。

![parachain-bid.png](/media/images/docs/tutorials/parachains/parachain-crowdloan.png)

参数说明：

- `parachain id`：您只能为已注册的 `ParaID` 创建众筹活动。

- `众筹上限`：您的众筹活动可以筹集的最大金额。
  这只是获得外部支持的一种负责任的措施。

- `结束区块`：这是您希望众筹结束的时间。
  如果您知道拍卖将在 3 天后开始，持续 5 天，您可能希望将众筹设置为在 10 天后结束，最后留有一些缓冲时间。
  这样，您就可以确保您的众筹在整个拍卖过程中都是活跃的。
  另一方面，您也不希望将众筹期限设置得太长，否则会不必要地延长社区资金锁定期限，并劝退他们参与您的活动。

- `第一期` / `最后一期`：您想要竞标的时隙持续时间。
  因此，如果当前拍卖有 3-6 个时隙开放，则这些值可以是两者之间的任何数字。

- 只要您没有收到任何捐款，您就可以取消众筹。

如果您的外部函数成功，您可以在 [网络 -> Parachains -> 众筹页面](https://polkadot.js.org/apps/#/parachains/crowdloan) 中看到您的新众筹条目：

![crowdloan-success.png](/media/images/docs/tutorials/parachains/crowdloan-success.png)

#### 为众筹活动提供资金

任何拥有可用余额并选择为您的活动捐款的账户，包括启动此活动的同一账户。

您可以转到上面相同的 [众筹页面](https://polkadot.js.org/apps/#/parachains/crowdloan)，并在您想要支持的活动上选择 **+ 捐款**。

您将看到一个类似于以下内容的外部函数弹出窗口：

![crowdloan-contribute.png](/media/images/docs/tutorials/parachains/crowdloan-contribute.png)

输入您想要支持的资助金额并提交交易。
