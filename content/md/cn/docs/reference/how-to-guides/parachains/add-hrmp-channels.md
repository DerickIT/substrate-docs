# 添加 HRMP 通道

本指南将指导您如何请求 Parachain B 接受与 Parachain A 建立通道的请求。

有关 HRMP 通道的更多信息，请阅读 [Polkadot wiki](https://wiki.polkadot.network/docs/build-hrmp-channels)。

**通道是单向的**

一般的流程是，想要打开通道的其中一个 Parachain 必须向它想要与其打开通道的 Parachain 发出请求。

然后，另一个 Parachain 需要接受这个打开通道的请求，这意味着通道默认是单向的。

对于双向通信，我们需要以相反的方式打开另一个通道。

只有在接收方确认后，并且只有在会话更改后才能打开通道。

在本指南中，我们假设我们有：

- 中继链
- ParaId 为 2000 的 Parachain A
- ParaId 为 3000 的 Parachain B

## Parachain A 的操作

1. 在中继链上，准备以下调用：

   ```
   hrmp.hrmpInitOpenChannel(
       recipient: 3000                    // 您想要与其打开通道的另一个 Parachain
       proposedMaxCapacity: 1000          // 指定通道中一次可以有多少条消息
       proposed_max_message_size: 102400  // 指定消息的最大大小
   )
   ```

   请注意，这些数字受中继链的配置限制。

   设置所需参数后，保存编码后的调用数据。
   例如，Rococo 中此调用的编码调用数据为：[ `0x3c00b80b0000e803000000900100`](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/extrinsics/decode/0x3c00b80b0000e803000000900100)。

1. 现在，在 Parachain 上，我们必须编写一条 XCM 消息来通知中继链我们想要与 Parachain B 打开一个通道。

   该消息应类似于以下内容（使用上一步中的编码调用数据）：

   ```
   polkadotXcm.send(
       dest: V1
           parents: 1
           interior: Here
       message: V2
           XcmV2Instruction: WithdrawAsset
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
           XcmV2Instruction: BuyExecution
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
               weightLimit: Unlimited
           XcmV2Instruction: Transact
               originType: Native
               requireWeightAtMost: 4_000_000_000
                   encoded: 0x3c00b80b0000e803000000900100 // 我们 hrmpInitOpenChannel 编码后的调用数据
           XcmV2Instruction: DepositAsset
             assets: Wild::All
             maxAssets: 1
             beneficiary:
               parents: 0
               interior: Parachain(2000)           
   )
   ```

   请记住，您应该考虑到活动的 XCM 配置来编写您的消息，这只是一个示例。

## Parachain B 的操作

到目前为止，Parachain A 已经完成了它的部分工作：发送到 Parachain B 的请求。

现在这个请求必须由 Parachain B 接受。

该过程重复进行，但这次我们将必须编码不同的调用。

1. 在中继链上，提交以下外部函数并保存编码后的调用数据：

   ```
   hrmp.hrmpAcceptOpenChannel(
       sender: 2000
   )
   ```

   Rococo 上的编码调用数据为：[ `0x1701d0070000`](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/extrinsics/decode/0x1701d0070000)

1. 现在在 Parachain B 上，我们可以使用我们最后的 XCM 消息作为基础来编写这个消息，并将其作为对中继链的调用来执行（使用上一步中的编码调用数据）。

   ```
   polkadotXcm.send(
       dest: V1
           parents: 1
           interior: Here
       message: V2
           XcmV2Instruction: WithdrawAsset
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
           XcmV2Instruction: BuyExecution
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
               weightLimit: Unlimited
           XcmV2Instruction: Transact
               originType: Native
               requireWeightAtMost: 4_000_000_000
                   encoded: 0x1701d0070000 // 我们 hrmpAcceptOpenChannel 编码后的调用数据

   )
   ```

   请记住，您应该考虑到活动的 XCM 配置来编写您的消息，这只是一个示例。

就是这样——通道已被接受，并且将保持打开状态，以便 Parachain A 到 Parachain B 的通信现在可以流动。

要使其成为双向通道，您需要打开另一个通道，从 Parachain B 到 Parachain A。您可以通过从 B 发出请求并通过遵循本指南中的相同步骤从 A 接受它来做到这一点。
