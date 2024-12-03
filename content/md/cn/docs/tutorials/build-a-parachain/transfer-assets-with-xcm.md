---
title: 使用 XCM 传输资产
description: 演示如何使用交叉共识消息通过中继链执行对平行链的远程转移。
keywords:
  - XCM
  - 平行链通信
  - 消息传递
  - 交叉链
  - 交叉共识
---

在[打开消息传递通道](/tutorials/build-a-parachain/open-message-passing-channels)中，您了解了如何通过向中继链发送消息来在链之间打开双向通信通道。
您可以使用类似的策略来发送允许本地链管理远程链上帐户的消息。
在本教程中，平行链 B 将资产转移到平行链 A 的中继链上的主权帐户。

本教程的结果类似于使用 balances 模块中的 `transfer` 函数，只是在这种情况下，转移是由平行链启动的，并演示了在执行 `WithdrawAsset` 和 `DepositAsset` XCM 指令时如何使用持有寄存器。

## 开始之前

在开始之前，请验证以下内容：

- 您已使用 Zombienet 或使用 `rococo-local` 链规范的本地中继链设置了[平行链测试网络](/test/simulate-parachains)。

- 您已设置了两个本地或虚拟平行链用于测试目的。

  在本教程中，平行链 A 的唯一标识符为 1000，平行链 B 的唯一标识符为 1001。

- 两个本地平行链都可以使用 Sudo 模块。

- 您已打开消息传递通道以允许平行链 B 和平行链 A 之间的通信。

## 配置 XCM 指令

为了说明两条链之间的交互，在以下示例中，平行链 B 发送 XCM 指令以将资产存入平行链 A 上的帐户。

1. 使用[Polkadot/Substrate 门户](https://polkadot.js.org/apps)连接到平行链 B (1001) 的端点。

2. 点击**开发者**并选择**外部函数**。

3. 选择**sudo**，然后选择**sudo(call)** 以使用 Sudo 模块执行特权交易。

4. 选择**polkadotXcm**，然后选择**send(dest, message)**。

5. 指定目标参数以指示要传递消息的相对位置。

   - 用于指定目标位置的 XCM 版本：V1
   - 中继链是消息的目标，因此父级位置：1
   - 在父级的上下文中，内部设置：Here

6. 指定消息的 XCM 版本 (V2)。

7. 点击**添加项目**以构建要执行的消息。

## WithdrawAsset 指令

要将资产移动到虚拟持有寄存器：

1. 选择[WithdrawAsset](https://github.com/paritytech/xcm-format#withdrawasset)作为此消息的第一个指令。

2. 点击**添加项目**以标识要提取的链上资产。

3. 选择**具体**以使用资产的位置来标识要提取的资产。

4. 将**父级：0** 和**内部：Here** 设置为从中继链上的平行链 B 主权帐户提取资产。

5. 选择**可替代**以将资产标识为可替代资产。

6. 指定要提取的可替代资产总数。

   例如，本教程使用 12000000000000。

   ![从平行链 B 发送的 WithdrawAsset 指令](/media/images/docs/tutorials/parachains/transfer-withdraw-asset-instruction-ui.png)

## BuyExecution 指令

要使用存入持有寄存器的资产支付执行费用：

1. 点击**添加项目**以选择[BuyExecution](https://github.com/paritytech/xcm-format#buyexecution)作为此消息的第二个指令。

2. 选择**具体**以使用资产的位置来标识要用于支付执行 XCM 指令的资产。

3. 将**父级：0** 和**内部：Here** 设置为使用从中继链上平行链 B 主权帐户提取的资产。

4. 选择**可替代**以将资产标识为可替代资产。

5. 指定要使用的可替代资产总数。

   例如，本教程使用 12000000000000。

6. 选择**无限**以跳过为该指令设置权重限制。

   ![从平行链 B 发送的 BuyExecution 指令](/media/images/docs/tutorials/parachains/transfer-buy-execution-instruction-ui.png)

## DepositAsset 指令

要在支付费用后将持有寄存器中的资产存入特定帐户：

1. 点击**添加项目**以选择[DepositAsset](https://github.com/paritytech/xcm-format#depositasset)作为此消息的第三个指令。

1. 选择**通配符**以允许存入不确定的数量的资产。

1. 选择**全部**以允许在支付费用后存入所有剩余资产。

2. 将要从持有寄存器中删除的唯一资产的最大数量设置为 1。

   在本教程中，只有一个资产实例可供删除。

1. 指定接收已存入资产的受益人。

   您可以将剩余资产存入平行链 A 的主权帐户或特定帐户。
   对于本教程，资产是使用先前未注资的帐户 KRIS-PUBS 的特定帐户地址存入的。
   要选择此受益人，DepositAsset 指令如下所示：

   ![将帐户指定为受益人](/media/images/docs/tutorials/parachains/transfer-deposit-asset-instruction-ui.png)

   如果您想将资产存入平行链 A (1000) 的主权帐户，则可以使用以下设置指定受益人：

   - parents: 0,
   - interior: X1,
   - X1 junction: Parachain
   - Parachain index: 1000

   配置好要执行的所有 XCM 指令后，您就可以提交交易了。

## 提交交易

要提交交易：

1. 点击**提交交易**。

1. 点击**签名并提交**。

1. 点击**网络**并选择**资源管理器**以验证消息是否已发送。

   如果您展开事件，则可以查看消息指令。
   如果您点击包含交易的区块的链接，则可以查看其他详细信息

## 检查中继链上的事件

要检查中继链上的结果：

1. 打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps)并连接到中继链。

2. 点击**网络**并选择**资源管理器**以查看 XCM 消息的事件。

   ![中继链事件](/media/images/docs/tutorials/parachains/relay-chain-event-summary.png)

1. 点击记录更改的区块编号以查看详细信息。

   ![在中继链上记录提取和存入的资产](/media/images/docs/tutorials/parachains/relay-chain-block.png)

## 检查已存入的资产

要验证已存入帐户的资产：

1. 打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps)并连接到中继链。

2. 点击**帐户**，然后查看已将资产（减去交易费用）存入帐户。

   例如：

   ![已将资产存入指定的帐户](/media/images/docs/tutorials/parachains/transfer-account-funded.png)

   如果您已将远程转移到平行链 A (1000) 的主权帐户而不是 KRIS-PUBS 帐户，则应点击**帐户**，然后选择**地址簿**以查看从平行链 B 主权帐户提取的资产已存入平行链 A 主权帐户。
