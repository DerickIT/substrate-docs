---
title: 为调用指定来源
description: 演示如何指定用作函数调用发起者的帐户。
keywords:
  - FRAME
  - 运行时
  - 来源
---

在[向运行时添加模块](/tutorials/build-application-logic/add-a-pallet)中，您向[Substrate 节点模板](https://github.com/paritytech/polkadot-sdk-solochain-template)运行时添加了来自`pallet_nicks`的函数。

Nicks 模块允许区块链用户支付押金以预留他们控制的帐户的昵称。
它实现了以下功能：

- `set_name` 函数，用于启用帐户所有者在名称未被预留时设置其自身帐户的名称。
- `clear_name` 函数，用于启用帐户所有者删除与帐户关联的名称并返还押金。
- `kill_name` 函数，用于强制删除另一方的帐户名称而不返还押金。
- `force_name` 函数，用于在不需要押金的情况下为另一方设置帐户名称。

本教程说明了如何使用不同的来源帐户调用这些函数，以及为什么使用不同的来源帐户调用这些函数很重要。

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已在本地安装了 Substrate 节点模板。

- 您已在本地安装了 Substrate 前端模板。

- 您已完成[向运行时添加模块](/tutorials/build-application-logic/add-a-pallet)教程，并成功编译了包含`nicks`模块的运行时。

- 您通常熟悉软件开发和使用命令行界面。

## 教程目标

通过完成本教程，您将实现以下目标：

- 使用有权执行调用的帐户调用`set_name`函数。
- 使用有权执行调用的帐户调用`clear_name`函数。
- 使用无权执行调用的帐户调用`force_name`函数。
- 使用没有管理权限的来源调用`kill_name`函数。
- 使用`Root`来源帐户调用`kill_name`函数。
- 查看如何使用不同的来源帐户调用函数会导致失败或成功的结果。

## 确定管理帐户

正如您在[向运行时添加模块](/tutorials/build-application-logic/add-a-pallet)中看到的那样，`nicks`模块的`Config`特征声明了几个类型。
对于本教程，重点是`ForceOrigin`类型。
`ForceOrigin`类型用于指定可以执行某些操作的帐户。
对于此模块，`ForceOrigin`类型指定可以为其他帐户设置或删除名称的帐户。
通常，只有具有管理权限的帐户（例如根超级用户帐户）才能代表其他帐户执行操作。
在 Nicks 模块的情况下，只有帐户所有者或 Root 帐户才能设置或删除预留的昵称。
在您将 FRAME 系统[`Root`来源](https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Root)标识为`nicks`模块管理员时，您在实现（`impl`）块中配置了此 Root 帐户。
例如：

```rust
type ForceOrigin = frame_system::EnsureRoot<AccountId>;
```

在节点模板的开发[链规范](https://github.com/paritytech/polkadot-sdk-solochain-template/blob/main/node/src/chain_spec.rs)中，[Sudo 模块](https://paritytech.github.io/substrate/master/pallet_sudo/index.html)配置为使用 Alice 帐户作为 FRAME 系统`Root`来源。
由于此配置，默认情况下，只有 Alice 帐户才能调用需要`ForceOrigin`类型的函数。

如果您尝试使用除 Alice 帐户以外的帐户调用`kill_name`或`force_name`，则该调用将无法执行。

## 为帐户设置名称

为了演示调用来源如何影响操作，让我们设置并尝试强制删除另一个帐户的帐户名称。
对于此演示，请确保您拥有：

- 以开发模式运行的节点模板：`./target/release/solo-template-node --dev`
- 正在运行并连接到本地节点的前端模板：`yarn start`
- 连接到本地 Web 服务器的浏览器：<http://localhost:8000/> <!-- markdown-link-check-disable-line -->

1. 将前端模板中的活动帐户从 Alice 更改为 Bob。

1. 在 Pallet Interactor 中选择**外部函数**：

   - 选择`nicks`模块。
   - 选择`setName`函数。
   - 为帐户输入名称。
   - 点击**已签名**以提交 Bob 签名的此事务。

   由于 Bob 是此帐户的所有者，因此交易成功。
   作为帐户的所有者，Bob 还可以执行已签名的`clearName`函数来删除帐户的昵称。

1. 选择**外部函数**：

   - 选择`nicks`模块。
   - 选择`clearName`函数。
   - 点击**已签名**以提交 Bob 签名的此事务。

      由于 Bob 是此帐户的所有者，因此交易成功。
      对于 Bob 要设置或删除另一个帐户的昵称，他必须使用为模块配置的`ForceOrigin`调用`forceName`或`killName`函数。

1. 选择**外部函数**：

   - 选择`nicks`模块。
   - 选择`forceName`函数。
   - 将 Charlie 的帐户地址复制并粘贴为目标。
   - 为帐户输入名称。
   - 点击**已签名**以提交 Bob 签名的此事务。

   由于您使用 Bob 的帐户签署了此事务，因此该函数使用[`Signed`来源](https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Signed)而不是`Root`来源进行调度。
   在这种情况下，函数调用本身是成功的。
   但是，名称预留无法完成，并且会发出`BadOrigin`错误。

   ![BadOrigin 错误](/media/images/docs/tutorials/add-a-pallet/badOrigin.png)

   正如您在事件中看到的那样，该事务导致从 Bob 的帐户中提取作为提交事务的[费用](/build/tx-weights-fees/)，但没有状态更改，因为`Root`来源没有提交事务。
   未能更改状态还说明了用于数据库读取和写入的[先验证后写入](/build/runtime-storage#verify-first-write-last)原则，以确保只有成功操作才会提交到磁盘。

## 使用 Root 来源调度调用

   Sudo 模块使您可以使用`Root`来源调度调用。
   在 Nick 模块中，必须使用`ForceOrigin`配置指定的`Root`来源调用 forceName 和 killName 函数。
   在前端模板中，您可以通过点击**SUDO**来访问 Sudo 模块以使用`Root`来源调度调用。

对于此演示，请确保您拥有：

- 以开发模式运行的节点模板：`./target/release/solo-template-node --dev`
- 正在运行并连接到本地节点的前端模板：`yarn start`
- 连接到本地 Web 服务器的浏览器：<http://localhost:8000/> <!-- markdown-link-check-disable-line -->

1. 将活动帐户更改为 Alice。

   如[确定管理帐户](#identify-the-administrative-account)中所述，在开发模式下运行链时，Alice 是与`Root`来源关联的帐户。

1. 在 Pallet Interactor 中选择**外部函数**：

   - 选择`nicks`模块。
   - 选择`forceName`函数。
   - 将 Charlie 的帐户地址复制并粘贴为目标。
   - 为帐户输入名称。
   - 点击**SUDO**以使用`Root`来源提交此事务。

   ![使用 SUDO 提交事务](/media/images/docs/tutorials/add-a-pallet/sudo-tx.png)

1. 选择**外部函数**：

   - 选择`nicks`模块。
   - 选择`killName`函数。
   - 将 Bob 的帐户地址复制并粘贴为目标。
   - 点击**SUDO**以使用`Root`来源提交此事务。

   在这种情况下，Sudo 模块会发出[`Sudid`事件](https://paritytech.github.io/substrate/master/pallet_sudo/pallet/enum.Event.html)以通知网络参与者`Root`来源调度了一个调用，但发生了错误。

   ![提交 killName 函数发出了错误](/media/images/docs/tutorials/add-a-pallet/sudo-error.png)

   此调度错误包含两条元数据：

   - 一个`index`编号，指示发出错误的模块。
   - 一个`error`编号，指示该模块的`Error`枚举发出的错误。

   `index`编号与模块在`construct_runtime!`宏中的位置相对应，`construct_runtime!`宏中的_第一个_模块的索引号为零 (0)。

   在此示例中，`index`为`6`（_第七个_模块），`error`为`2`（_第三个_错误）。

   ```rust
   construct_runtime!(
    pub enum Runtime where
      Block = Block,
      NodeBlock = opaque::Block,
      UncheckedExtrinsic = UncheckedExtrinsic
    {
      System: frame_system,                                        // index 0
      RandomnessCollectiveFlip: pallet_randomness_collective_flip, // index 1
      Timestamp: pallet_timestamp,                                 // index 2
      Aura: pallet_aura,                                           // index 3
      Grandpa: pallet_grandpa,                                     // index 4
      Balances: pallet_balances,                                   // index 5
      Nicks: pallet_nicks,                                         // index 6
    }
   ```

   无论`index`的值如何，`error`值`2`都对应于 Nicks 模块中的[`Unnamed`错误](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Error.html)。
   如果您期望 Bob 没有预留昵称或先前已清除名称预留，则会发生此错误。

   您可以确认 Alice 可以使用 SUDO 调用`killName`函数来删除当前已预留名称的任何帐户的预留昵称。

1. 选择**外部函数**：

   - 选择`nicks`模块。
   - 选择`killName`函数。
   - 将 Charlie 的帐户地址复制并粘贴为目标。
   - 点击**SUDO**以使用`Root`来源提交此事务。

   ![提交成功的 killName 事务](/media/images/docs/tutorials/add-a-pallet/sudo-error.png)

## 下一步

本教程介绍了使用`Root`和`Signed`来源指定用于提交事务的帐户，并演示了使用不同的来源帐户调用函数的结果。
有几个[教程](/tutorials/)可以作为学习更多关于 Substrate 开发的下一步。

除了教程之外，您可能还希望浏览以下资源以了解更多信息。

- [特权调用和来源](/build/origins)更详细地介绍了默认的原始来源类型以及如何创建自定义来源。
- [事件和错误](/build/events-and-errors)解释了如何从运行时发出事件和错误。
- [FRAME 模块](/reference/frame-pallets/)概述了最常用的预定义 FRAME 模块。
