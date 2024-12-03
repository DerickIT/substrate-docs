---
title: 打开消息传递通道
description: 演示如何使用水平中继路由消息传递 (HRMP) 进行平行链之间的通信。
keywords:
  - Polkadot
  - 交叉共识消息传递
  - HRMP
  - XCM
  - 平行链
---

在 Polkadot 生态系统中，链可以通过安全通道传递消息来相互通信。
主要有三个通信通道：

- 上行消息传递 (UMP)，使平行链能够向上传递消息到其中继链。
- 下行消息传递 (DMP)，使中继链能够向下传递消息到平行链。
- 交叉共识消息传递 (XCMP)，使平行链能够相互发送消息。

水平中继路由消息传递 (HRMP) 是交叉共识消息传递 (XCMP) 的一个临时版本。
这个临时解决方案（有时也称为 XCMP-Lite）提供了与 XCMP 相同的功能，但将链之间传递的所有消息存储在中继链存储中。

尽管 HRMP 预计在 XCMP 完全实现后将逐步淘汰，但本教程使用 HRMP 来说明如何打开消息传递通道以使平行链能够相互通信。

## 关于 XCM 格式

通过 Polkadot 通信通道传递的消息使用[XCM 格式](https://github.com/paritytech/xcm-format)来表达消息接收者应该做什么。
XCM 格式旨在支持链之间的通信，但也支持来自智能合约和模块的消息，通过桥接以及通过其他传输协议。

## 打开 HRMP 通道

重要的是要注意，HRMP 通道是单向的。
如果您希望平行链 1000 与平行链 1001 通信，您必须首先请求从平行链 1000 到平行链 1001 打开 HRMP 通道。
平行链 1001 必须接受请求后，平行链 1000 才能向其传递消息。
但是，由于该通道是单向的，因此平行链 1000 无法通过该通道从平行链 1001 接收消息。

对于平行链 1000 从平行链 1001 接收消息，您必须从平行链 1001 到平行链 1000 打开另一个通道。
在平行链 1000 确认它将接受来自平行链 1001 的消息后，链可以在下一个会话更改时交换消息。

## 开始之前

在本教程中，您将打开 HRMP 通道，使具有唯一标识符 1000 的平行链和具有唯一标识符 1001 的平行链能够交换消息。
在开始之前，请验证以下内容：

- 您已使用 Zombienet 或使用 `rococo-local` 链规范的本地中继链设置了[平行链测试网络](/test/simulate-parachains)。

- 您已设置了两个本地或虚拟平行链用于测试目的。

  在本教程中，平行链 A 的唯一标识符为 1000，平行链 B 的唯一标识符为 1001。

- 两个本地平行链都可以使用 Sudo 模块。

  在本教程的测试环境中，您可以将 Sudo 模块添加到每个验证人节点。
  如果您使用 `substrate-parachain-template` 来构建节点，则默认情况下不包含该模块。
  要添加 Sudo 模块，您必须更新 `runtime/src/lib.rs` 和 `node/src/chain_spec.rs` 文件。
  对于运行时，您可以实现配置特征并修改 construct_runtime 宏，这与添加任何其他模块的方式类似。
  有关包含 Sudo 模块的链规范的示例，请参阅[parachain-template-1001.rs](/assets/tutorials/relay-chain-specs/parachain-template-1001.rs)。

  在生产环境中，您将使用治理提案和投票来代替 Sudo 模块进行特权交易。

## 添加主权帐户

在平行链能够与另一个平行链交换消息之前，它必须在中继链上拥有一个帐户，该帐户拥有可用于支付执行 XCM 指令的资产。

要将主权帐户地址添加到中继链：

1. 打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps)并连接到中继链端点。

2. 计算要在中继链上使用的平行链[主权帐户地址](https://substrate.stackexchange.com/questions/1200/how-to-calculate-sovereignaccount-for-parachain)。

   平行链 A (1000) 地址：5Ec4AhPZk8STuex8Wsi9TwDtJQxKqzPJRCH7348Xtcs9vZLJ

   平行链 B (1001) 地址：5Ec4AhPZwkVeRmswLWBsf7rxQ3cjzMKRWuVvffJ6Uuu89s1P

   请注意，如果为平行链注册的平行链标识符发生更改，则主权帐户和地址也会发生更改。
   您还应该注意，在中继链上用于平行链的帐户地址与在另一个平行链上用于平行链的地址不同。

3. 点击**帐户**并选择**地址簿**。

4. 点击**添加联系人**。

5. 添加平行链 A (1000) 的地址和名称，然后点击**保存**。

6. 点击**帐户**并将一些资产从 Alice 转移到平行链 A (1000) 帐户。

   对平行链 B (1001) 重复步骤 3 到步骤 6。

## 准备打开通道的编码调用

要设置平行链之间的通信，您必须首先发送打开消息传递通道的请求。
该请求必须采用编码调用的形式，其参数指定接收请求的平行链、通道的消息容量和最大消息大小。
您需要在创建的消息中包含此信息的编码版本才能发出请求，但您可以生成编码调用而无需提交交易来准备请求。

要准备打开通道的编码调用：

1. 打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps)并连接到中继链端点。

2. 根据需要检查中继链的通道配置限制。

   要检查当前中继链的配置设置：

   - 点击**开发者**并选择**链状态**。
   - 选择**配置**，然后选择**activeConfig()** 并点击**+**。
   - 检查 `hrmpChannelMaxCapacity` 和 `hrmlChannelMaxMessageSize` 的参数值。

   例如：

      ```text
      hrmpChannelMaxCapacity: 8
      hrmlChannelMaxMessageSize: 1,048,576
      ```

3. 点击**开发者**并选择**外部函数**。

4. 选择**hrmp**，然后选择**hrmpInitOpenChannel(recipient, proposedMaxCapacity, proposedMaxMessageSize)** 以初始化打开新通道的请求。

   对于交易参数，请指定以下内容以准备调用数据：

   - recipient：输入您要与其打开通道的平行链的标识符 (1001)。
   - proposedMaxCapacity：输入通道中一次可以容纳的最大消息数 (8)。
   - proposedMaxMessageSize：指定消息的最大大小 (1048576)。

   请注意，您为**proposedMaxCapacity** 和**proposedMaxMessageSize** 设置的值不应超过为中继链的 `hrmpChannelMaxCapacity` 和 `hrmpChannelMaxMessageSize` 参数定义的值。

5. 复制**编码调用数据**。

   ![复制编码调用数据](/media/images/docs/tutorials/parachains/hrmp-encoded-call.png)

   您需要此信息来构建 XCM Transact 指令。
   以下是 Rococo 中编码调用数据的示例：`0x3c00e90300000800000000001000`

## 配置打开通道请求

现在您有了编码调用，您可以配置请求以通过中继链从平行链 A 到平行链 B 打开通道。

1. 使用[Polkadot/Substrate 门户](https://polkadot.js.org/apps)连接到平行链 A (1000) 的端点。

2. 点击**开发者**并选择**外部函数**。

3. 选择**sudo**，然后选择**sudo(call)** 以使用 Sudo 模块执行特权交易。

3. 选择**polkadotXcm**，然后选择**send(dest, message)** 以通知中继链您希望与平行链 B (1001) 打开通道。

   ![使用 Sudo 模块发送消息](/media/images/docs/tutorials/parachains/hrmp-sudo-call.png)

4. 指定目标参数以指示要传递消息的相对位置。

   目标参数指定应在何处执行 XCM。
   在此示例中，平行链 1000 的父级是中继链，在父级的上下文中，Here 的内部设置意味着中继链将执行 XCM。
   有关为 XCM 指定相对位置的更多信息，请参阅[通用共识位置标识符](https://github.com/paritytech/xcm-format#7-universal-consensus-location-identifiers)。

   ![目标参数](/media/images/docs/tutorials/parachains/hrmp-destination.png)

5. 指定 XCM 版本，然后点击**添加项目**以构建要执行的消息。

   此消息至少需要添加以下指令集：

   - [WithdrawAsset](https://github.com/paritytech/xcm-format#withdrawasset) 将指定的链上资产移动到虚拟[持有寄存器](https://polkadot.network/blog/xcm-the-cross-consensus-message-format/#-the-holding-register)。

   - [BuyExecution](https://github.com/paritytech/xcm-format#buyexecution) 使用使用 WithdrawAsset 指令存入虚拟持有寄存器的资产来支付当前消息的执行费用。
     有关支付费用的更多信息，请参阅[XCM 中的费用支付](https://polkadot.network/blog/xcm-the-cross-consensus-message-format/#-fee-payment-in-xcm)。

   - [Transact](https://github.com/paritytech/xcm-format#transact) 指定您在中继链上准备的编码调用。

   在大多数情况下，您还希望包含以下指令：

   - [RefundSurplus](https://github.com/paritytech/xcm-format#refundsurplus) 将使用 BuyExecution 指令先前支付的任何费用高估值移动到称为退款权重寄存器的第二个虚拟寄存器中。

   - [DepositAsset](https://github.com/paritytech/xcm-format#depositasset) 从退款权重寄存器中减去资产，并在受益人的所有权下存入链上等效资产。

   请注意，每个指令都需要您指定其他信息，以使消息接收者能够执行预期的 XCM 指令。
   请确保您从接收系统的角度构建每个指令的信息。

   让我们仔细看看这些指令。

### WithdrawAsset 指令

要将资产移动到虚拟持有寄存器：

1. 选择[WithdrawAsset](https://github.com/paritytech/xcm-format#withdrawasset)作为此消息的第一个指令。

2. 点击**添加项目**以标识要提取的链上资产。

   ![添加指令或添加有关指令的信息](/substrate-docs/content/media/images/docs/tutorials/parachains/construction-xcm-instruction.png)

3. 选择**具体**以使用资产的位置来标识要提取的资产。

     有关指定资产位置的更多信息，请参阅[具体标识符](https://github.com/paritytech/xcm-format#concrete-identifiers)。

4. 将**父级：0** 和**内部：Here** 设置为从中继链上的主权帐户提取资产。

5. 选择**可替代**以将资产标识为可替代资产。

6. 指定要提取的可替代资产总数。

   ![WithdrawAsset 和设置](/media/images/docs/tutorials/parachains/withdraw-asset-instruction-settings.png)

### BuyExecution 指令

要使用存入持有寄存器的资产支付执行费用：

1. 点击**添加项目**以选择[BuyExecution](https://github.com/paritytech/xcm-format#buyexecution)作为此消息的第二个指令。

2. 选择**具体**以使用资产的位置来标识要用于支付执行 XCM 指令的资产。

3. 将**父级：0** 和**内部：Here** 设置为指示您正在使用本地链的原生资产进行执行支付。

4. 选择**可替代**以将资产标识为可替代资产。

5. 指定要使用的可替代资产总数。

6. 选择**无限**以跳过为该指令设置权重限制。

   ![BuyExecution 和设置](/media/images/docs/tutorials/parachains/buy-execution-open.png)

### Transact 指令

要执行您在中继链上准备的编码调用：

1. 点击顶部的**添加项目**以选择[Transact](https://github.com/paritytech/xcm-format#transact)作为此消息的第三个指令。

2. 选择**原生**作为执行指令的消息来源。

3. 将**requireWeightAtMost** 设置为指定在调度 XCM 调用时要使用的最大权重。

   如果 XCM 调度需要的权重超过指定值，则交易失败。
   如果 XCM 调度需要的权重小于指定值，则差值可以添加到剩余权重寄存器中。

1. 指定要由 Transact 指令执行的交易的编码调用数据。

   例如，粘贴用于启动打开通道请求的编码调用。

   ![Transact 和设置](/media/images/docs/tutorials/parachains/transact-open-request.png)

### RefundSurplus 和 DepositAsset 指令

要移动任何费用高估值：

1. 点击顶部的**添加项目**以选择[RefundSurplus](https://github.com/paritytech/xcm-format#transact)作为此消息的第四个指令。

1. 点击顶部的**添加项目**以选择[DepositAsset](https://github.com/paritytech/xcm-format#transact)作为此消息的第五个指令。

1. 选择**通配符**以允许存入不确定的数量的资产。

1. 选择**全部**以允许存入所有退款资产。

2. 将要从持有寄存器中删除的唯一资产的最大数量设置为 1。

   在本教程中，只有一个资产实例可供删除。

1. 指定接收已存入资产的受益人。

   通常，DepositAsset 指令的受益人是消息发送者的主权帐户。
   在本例中，您可以将平行链 A (1000) 指定为**父级：0**、**内部：X1**、**平行链：1000**，以便任何剩余资产都将返回到该帐户，并可用于传递其他 XCM 指令或打开其他 HRMP 通道。

   ![RefundSurplus 和 DepositAsset 指令和设置](/media/images/docs/tutorials/parachains/refund-and-deposit.png)

   或者，您可以将受益人指定为**父级：0**、**内部：X1**、**AccountId** 并标识要接收资产的网络和帐户地址。

   有关 RefundSurplus 和 DepositAsset 指令的更多信息，请参阅[权重](https://polkadot.network/blog/xcm-part-three-execution-and-error-management/#-weight)。

### 查看完整的指令集

此 XCM 指令集：

- 将资产从平行链 A 主权帐户提取到虚拟持有寄存器。

- 使用持有寄存器中的资产来支付 XCM 指令所需的执行时间。

- 在中继链上执行打开通道请求的初始化请求。

- 退还任何剩余资产并将退款资产存入指定受益人拥有的帐户。

有关说明此指令集所有设置的示例，请参阅示例[xcm-instructions](/assets/tutorials/relay-chain-specs/xcm-instructions.txt)文件。
有关更多信息和特定技术问题的答案，请尝试[Substrate 和 Polkadot Stack Exchange](https://substrate.stackexchange.com/)上的以下标签：

- xcm
- hrmp
- weight
- cumulus

### 提交交易

要提交交易：

1. 点击**提交交易**。

1. 点击**签名并提交**。

1. 点击**网络**并选择**资源管理器**以验证消息是否已发送。

## 验证请求

提交交易后，您应该验证消息是否已接收并在中继链上执行。

要验证请求：

1. 在两个单独的浏览器选项卡或窗口中打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps)，其中一个实例连接到中继链，另一个实例连接到平行链端点。

2. 在连接到平行链的浏览器实例中，点击**网络**，然后选择**资源管理器**。

3. 检查最近事件列表，并验证是否存在**sudo.Sudid**事件和**polkadotXcm.Sent**事件。

   ![平行链上的 XCM 指令事件](/media/images/docs/tutorials/parachains/hrmp-parachain-events.png)

   您可以展开**polkadotXcm.Sent**事件以查看有关已发送消息中包含的指令的详细信息。

4. 在连接到中继链的浏览器实例中，点击**网络**，然后选择**资源管理器**。

3. 检查最近事件列表，并验证是否显示**ump.UpwardMessagesReceived**、**hrmp.OpenChannelRequested**和**ump.ExecutedUpward**事件。

   ![中继链上的 XCM 指令事件](/media/images/docs/tutorials/parachains/hrmp-relay-chain-request-complete.png)

   您可以展开这些事件以查看已执行的指令的详细信息。

4. 检查平行链 A (1000) 的主权帐户余额。

   ![消息执行后的帐户余额](/media/images/docs/tutorials/parachains/hrmp-after-msg-execution-balance.png)

## 准备接受编码调用

成功创建打开通道请求后，平行链 B (1001) 必须接受请求，然后才能使用您请求的通道发送任何消息。
步骤类似于启动打开通道请求的步骤，但您将使用不同的编码调用，并且您将从平行链 B 而不是平行链 A 发送消息。

第一步是在中继链上准备编码调用。

要准备编码调用：

1. 打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps)并连接到中继链。

2. 点击**开发者**并选择**外部函数**。

3. 选择**hrmp**，然后选择**hrmpAcceptOpenChannel(sender)** 并指定**发送者**，在本例中为平行链 A (1000)。

4. 复制**编码调用数据**。

   ![复制编码调用数据](/media/images/docs/tutorials/parachains/hrmp-relay-chain-accept-request.png)

   您需要此信息来创建 XCM Transact 指令。
   以下是 Rococo 中编码调用数据的示例：
   0x3c01e8030000

## 接受打开通道请求

现在您有了编码调用，您可以构建一组 XCM 指令来接受平行链 A 请求打开的通道。

1. 使用[Polkadot/Substrate 门户](https://polkadot.js.org/apps)连接到平行链 B (1001) 的端点。

2. 点击**开发者**并选择**外部函数**。

3. 选择**sudo**，然后选择**sudo(call)** 以使用 Sudo 模块执行特权交易。

4. 选择**polkadotXcm**，然后选择**send(dest, message)** 以通知中继链您希望与平行链 B (1001) 打开通道。

5. 指定目标参数以指示要传递消息的相对位置。

6. 指定 XCM 版本，然后点击**添加项目**以构建要执行的消息。

   您可以为此消息使用类似的指令集：

   - [WithdrawAsset](https://github.com/paritytech/xcm-format#withdrawasset) 将指定的链上资产移动到虚拟持有寄存器。

   - [BuyExecution](https://github.com/paritytech/xcm-format#buyexecution) 使用使用 WithdrawAsset 指令存入虚拟持有寄存器的资产来支付当前消息的执行费用。

   - [Transact](https://github.com/paritytech/xcm-format#transact) 指定您在中继链上准备的编码调用（例如，0x3c01e8030000）。

   - [RefundSurplus](https://github.com/paritytech/xcm-format#refundsurplus) 将任何费用高估值移动到退款权重寄存器中。

   - [DepositAsset](https://github.com/paritytech/xcm-format#depositasset) 从退款权重寄存器中减去资产，并在受益人的所有权下存入链上等效资产。

7. 点击**提交交易**。

   提交交易后，您必须等待下一个纪元才能看到链状态中的更改。

## 验证打开的通道

提交交易后，您可以验证消息是否已接收并在中继链上执行。

要验证请求：

1. 在两个单独的浏览器选项卡或窗口中打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps)，其中一个实例连接到中继链，另一个实例连接到平行链端点。

2. 在连接到平行链的浏览器实例中，点击**网络**，然后选择**资源管理器**。

3. 检查最近事件列表，并验证是否存在**sudo.Sudid**事件和**polkadotXcm.Sent**事件。

4. 在连接到中继链的浏览器实例中，点击**网络**，然后选择**资源管理器**。

5. 检查最近事件列表，并验证是否显示**hrmp.OpenChannelAccepted**和**ump.ExecutedUpward**事件。

   ![中继链上的 XCM 指令事件](/media/images/docs/tutorials/parachains/hrmp-relay-chain-request-accepted.png)

   在下个纪元开始后，您还可以查询**hrmpChannels**的链状态，以验证您是否已从平行链 A (1000) 到平行链 B (1001) 打开了一个通道。

   ![查询 HRMP 通道](/media/images/docs/tutorials/parachains/hrmp-verify-channel.png)

## 打开第二个通道

如果您只想向平行链 B (1001) 发送 XCM 指令，那么您的工作就完成了。

但是，如果您想启用双向通信——两个链都可以接收和执行另一个链发送的指令，则需要对平行链 B 执行与您刚刚对平行链 A 完成的步骤相同的步骤。在平行链 A 接受打开通道请求后，两个平行链都可以发送、接收和执行 XCM 指令。

重复所有步骤以准备编码调用、发送打开通道请求和接受请求，以启用从平行链 B (1001) 到平行链 A (1000) 的通信。
打开从平行链 B 到平行链 A 的通道后，两个平行链可以直接相互发送消息，或者通过中继链路由消息。

尽管您现在可以在平行链之间发送 XCM 指令，但构建可以在远程链上成功执行的消息需要额外的配置才能达成对可以转移的资产的共同理解，或者定义允许资产从一个链转移到另一个链的相互信任关系。
在未来的教程中，您将看到一些平行链之间通信的常见场景示例。

## 下一步

- [XCM 第一部分：交叉共识消息格式](https://polkadot.network/blog/xcm-the-cross-consensus-message-format)
- [XCM 第二部分：版本控制和兼容性](https://polkadot.network/blog/xcm-part-two-versioning-and-compatibility)
- [XCM 第三部分：执行和错误管理](https://polkadot.network/blog/xcm-part-three-execution-and-error-management)
