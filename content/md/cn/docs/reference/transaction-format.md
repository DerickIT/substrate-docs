---
title: 交易格式
description: 描述 Substrate 中已签名和未签名交易的格式。
keywords:
---

本文详细描述了 Substrate 中已签名和未签名交易的数据结构。
这对于理解交易池如何检查传入交易特别有用。
Parachain 构建者会发现这对于自定义交易格式以及编写需要遵守所选格式的客户端应用程序很有用。

外部函数通常包含签名、一些数据来描述外部函数是否已通过一些有效性检查以及它打算调用的 pallet 和调用。
此格式为应用程序提供了一种方法，用于确保满足外部函数的要求并正确构建。

- 未检查：已签名交易，在可以将其接受到交易池中之前需要进行一些有效性检查。
  任何未检查的外部函数都包含要发送的数据的签名以及一些额外数据。
- 已检查：固有外部函数，根据定义不需要签名验证。
  相反，它们包含有关外部函数来自何处以及一些额外数据的信息。
- 不透明：用于外部函数尚未提交到特定格式但仍可以解码的情况。

额外数据可以是任何附加到交易或固有事务的有用信息。
例如，交易的 nonce、区块作者的小费或外部函数的有效期。
此信息由 [专用扩展](#signed-extensions) 提供，这些扩展有助于在外部函数包含在区块中之前确定其有效性和排序。

已签名交易的构建方式如下所示：

```rust
node_runtime::UncheckedExtrinsic::new_signed(
		function.clone(),                                      // 某个调用
		sp_runtime::AccountId32::from(sender.public()).into(), // 某个发送帐户
		node_runtime::Signature::Sr25519(signature.clone()),   // 帐户的签名
		extra.clone(),                                         // 已签名的扩展
	)
```

## 交易的构建方式

Substrate 以通用的方式定义其交易格式，以允许开发人员实现自定义方法来定义有效交易。
但是，在使用 FRAME 构建的运行时中（假设交易版本为 4），必须通过提交以下编码数据来构建交易：

`<签名帐户 ID> + <签名> + <附加数据>`

提交已签名交易时，签名是通过签署以下内容构建的：

- 实际调用，由以下内容组成：
  - pallet 的索引。
  - pallet 中函数调用的索引。
  - 目标函数调用所需的参数。
- 通过交易的已签名扩展验证的一些额外信息：
  - 此交易的纪元是什么，即此调用在交易池中持续多长时间才会被丢弃？
  - nonce，即此帐户发生了多少次之前的交易？
    这有助于防止重放攻击或意外的双重提交。
  - 支付给区块生产者的提示金额，以激励其将此交易包含在区块中。

然后，需要一些不属于签名内容的附加数据，其中包括：

- 规范版本和交易版本。
  这确保交易提交到兼容的运行时。
- 创世哈希。这确保交易对正确的链有效。
- 区块哈希。这对应于检查点区块的哈希，通过针对纪元信息提供的区块号进行检查，使签名能够验证交易不会在错误的分支上执行。

然后对 SCALE 编码的数据进行签名（即（`call`、`extra`、`additional`）），并按正确的顺序附加签名、额外数据和调用数据并进行 SCALE 编码，准备发送到将验证已签名有效负载的节点。
如果要签名的有效负载长度超过 256 个字节，则在签名之前对其进行哈希处理，以确保已签名数据的长度不会超过一定大小。

此过程可以分解为以下步骤：

1. 构建未签名的有效负载。
1. 创建签名有效负载。
1. 签署有效负载。
4. 序列化已签名的有效负载。
5. 提交序列化的交易。

在进行十六进制编码之前，外部函数会编码成以下字节序列：

`[ 1 ] + [ 2 ] + [ 3 ] + [ 4 ]`

其中：

- `[1]` 包含所有后续数据的紧凑编码长度（以字节为单位）。了解如何使用 [SCALE](/reference/scale-codec/) 进行紧凑编码。
- `[2]` 是一个 `u8`，包含 1 个字节，用于指示交易是已签名还是未签名 (1 位)，以及编码的交易版本 ID (7 位)。
- `[3]` 如果存在签名，则此字段包含帐户 ID、SR25519 签名和一些额外数据。如果未签名，则此字段包含 0 个字节。
- `[4]` 是编码的调用数据。这包括 1 个字节，用于表示要调用的 pallet，1 个字节，用于表示要在该 pallet 中进行的调用，以及编码该调用期望的参数所需的所有字节。

应用程序知道如何正确构建交易的方式由 [元数据接口](/build/application-development/#metadata-system) 提供。
应用程序将使用元数据类型和交易格式来知道如何正确编码交易。
如果不需要对调用进行签名，则 `[2]` 中的第一位将为 0，因此应用程序将知道不要尝试解码签名。

**Polkadot JS Apps 示例：**

在这里，我们演示了从 Bob 到 Dave 的余额转移的详细的手动外部函数构建和提交：Bob 向 Dave 发送 `42 UNIT`。

1. 在 `--dev` 模式下启动 [节点模板](https://github.com/paritytech/polkadot-sdk-solochain-template)（请参阅 [快速入门](/quick-start) 指南，了解如何设置此项）
1. 导航到 <https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/extrinsics>
1. 将 `Bob` 设置为发送者，并选择 `balances` pallet 和 `transfer(dest, value)` 调用
1. 将 `MultiAddress` 设置为 `Id`，并将 `AccountID` 设置为 `Dave`（用于 `dest`）
1. 将 `value` 设置为 `42000000000000`（这在节点模板的 [链规范](/build/chain-spec/) 中定义为 `42 UNIT`）
1. 点击“提交交易”按钮（右下角），并**取消选中**_签名并提交_ 以生成已签名的交易，其默认 `nonce = 0` 和 `Lifetime = 64` 用于检查

![Bob 向 Dave 转账 42 个单位](/media/images/docs/reference/apps-extrinsic-transfer-bob-to-dave-42.png)

- 编码的调用数据：`0x050300306721211d5404bd9da88e0204360a1a9ab8b87c66c1bc2fcdd37f3c2222cc200b00a014e33226`
- 编码的调用哈希：`0x26c333c22ec93ac431ee348168530b7d77e85d766f130af60890c0fd6ab20d5b`
- 生成的已签名交易调用哈希：`0x450284008eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48018eeaeb6a3496444c08b5c3e10e0c5f94776774591504ef4ef26e3873799831285a1a7cbd8ba2babe6fba94ea3585bf20e46c80ce7baeb25b149529ece931478c45020c00050000306721211d5404bd9da88e0204360a1a9ab8b87c66c1bc2fcdd37f3c2222cc200b00a014e33226`

![Bob 向 Dave 转账 42 个单位的已签名哈希](/media/images/docs/reference/apps-extrinsic-transfer-bob-to-dave-42-signed-hash.png)

在这里，您可以_复制_ `已签名交易` 数据以直接通过 RPC 提交，或在 `开发者` -> `外部函数` -> `解码` 部分进行检查。
我们现在将使用此窗口提交交易并_观察_结果。

1. 关闭“授权交易”卡片
1. 点击“提交交易”按钮（右下角），并保持选中_签名并提交_
1. 导航到 `开发者` -> `RPC 调用` 选项卡

在 RPC 选项卡中，您应该会看到 `author_submitAndWatchExtrinsic` 调用的结果，类似于：

```json
{
  dispatchInfo: {
    weight: 159,200,000
    class: Normal
    paysFee: Yes
  }
  events: [
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Withdraw
        section: balances
        index: 0x0508
        data: {
          who: 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
          amount: 125,000,147
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: NewAccount
        section: system
        index: 0x0003
        data: {
          account: 5DAAnrj7VHTznn2AWBemMuyBwZWs6FNFjdyVXUeYum3PTXFy
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Endowed
        section: balances
        index: 0x0500
        data: {
          account: 5DAAnrj7VHTznn2AWBemMuyBwZWs6FNFjdyVXUeYum3PTXFy
          freeBalance: 42,000,000,000,000
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Transfer
        section: balances
        index: 0x0502
        data: {
          from: 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
          to: 5DAAnrj7VHTznn2AWBemMuyBwZWs6FNFjdyVXUeYum3PTXFy
          amount: 42,000,000,000,000
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: ExtrinsicSuccess
        section: system
        index: 0x0000
        data: {
          dispatchInfo: {
            weight: 159,200,000
            class: Normal
            paysFee: Yes
          }
        }
      }
      topics: []
    }
  ]
  status: {
    InBlock: 0x501c8f15883bb2b686fb5ea1ca35e99dace8bd6216bfc571a31d7088aea000f7
  }
}
```

1. 导航到 `网络` -> `资源管理器` 选项卡
1. 通过点击卡片右上角的 `<区块编号>-<外部函数编号>` 来打开 `balances.Transfer` 外部函数详细信息
1. 检查交易的链上详细信息

![Bob 向 Dave 转账 42 个单位的结果](/media/images/docs/reference/apps-extrinsic-transfer-bob-to-dave-42-result.png)

点击 `#/extrinsics/decode/0x....` 链接以打开上述 `已签名交易` 数据的解码详细信息，并注意它与我们提交的内容相同。
因此，在提交外部函数之前或之后，此工具都可用于解码和内省交易调用数据。

## 已签名扩展

Substrate 提供了**已签名扩展**的概念，可以使用 [`SignedExtension`](https://paritytech.github.io/substrate/master/sp_runtime/traits/trait.SignedExtension.html) 特性用附加数据扩展外部函数。

交易队列定期调用已签名扩展以持续检查交易在放入就绪队列之前是否有效。
这是验证交易不会在区块中失败的有用保障措施。
它们通常用于强制执行验证逻辑，以保护交易池免受垃圾邮件和重放攻击。

在 FRAME 中，已签名的扩展默认情况下可以保存以下任何类型：

- `AccountId`：用于编码发送者的身份。
- `Call`：用于编码要调度的 pallet 调用。此数据用于计算交易费用。
- `AdditionalSigned`：用于处理要添加到已签名有效负载的任何其他数据。这使得在调度交易之前附加任何自定义逻辑成为可能。
- `Pre`：用于编码可以在调用调度之前传递的信息，以及在调用调度之后传递的信息。

FRAME 的 [system pallet](https://paritytech.github.io/substrate/master/frame_system/) 提供了一组开箱即用的 [有用的 `SignedExtensions`](https://paritytech.github.io/substrate/master/frame_system/index.html#signed-extensions)。

### 实用示例

用于验证交易的重要已签名扩展是 `CheckSpecVersion`。
它为发送者提供了一种方法，可以将规范版本作为附加到交易的已签名有效负载提供。
由于规范版本在运行时中已知，因此已签名的扩展可以执行简单的检查以验证规范版本是否匹配。
如果不匹配，则交易在放入交易池之前就会失败。

其他示例包括用于计算交易优先级的已签名扩展。
这些是：

- `CheckWeight`：将所有调度类别的优先级值设置为 `0`。
- `ChargeTransactionPayment`：计算总优先级，并相应地修改优先级值。

优先级取决于调度类和发送者愿意支付的每权重小费或每长度小费（以限制较大的为准）。
没有小费的交易使用 `1` 的最小小费值进行优先级计算，以确保并非所有交易的优先级最终都为 `0`。
这样做的结果是_较小_的交易比_较大_的交易更受青睐。

## 接下来的步骤

既然您已经了解了交易的构建方式，您可能需要查看交易如何从交易池到运行时，以及如何添加到区块中，或者如何使用允许您离线提交交易或使用 REST API 的工具。

- [交易生命周期](/learn/transaction-lifecycle/)
- [交易、权重和费用](/build/tx-weights-fees/)
- 用于离线交易的 [tx-wrapper](/reference/command-line-tools/tx-wrapper)
- 用于基于 REST 的交易的 [sidecar](/reference/command-line-tools/sidecar)
