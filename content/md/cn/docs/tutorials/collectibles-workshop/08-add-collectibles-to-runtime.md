---
title: 向运行时添加可收藏物品
description:
tutorial:
---

您现在拥有一个具有交易数字可收藏物品的基本功能的自定义模块。
下一步是将此模块添加到运行时，以在其区块链上公开其功能。
但是，在执行此操作之前，请花一点时间验证您是否按照[将模块添加到工作区](/tutorials/collectibles-workshop/03-create-pallet/#add-the-pallet-to-the-workspace)中的说明在节点的清单中包含了 Collectibles 模块。

要检查工作区的清单：

1. 根据需要打开新的终端。

2. 更改到工作区中包含`workshop-solo-template-node`目录的目录。

3. 查看节点工作区的`Cargo.toml`文件的内容。

   ```bash
   more Cargo.toml
   ```

   您应该会看到 Collectibles 模块被列为工作区的成员。
   例如：

   ```toml
   [workspace]
   members = [
        "node",
        "pallets/collectibles",
        "pallets/template",
        "runtime",
   ]
   [profile.release]
   panic = "unwind"
   ```

   如果包含 Collectibles 模块，则您可以准备更新运行时。

## 更新运行时文件

要将 collectibles 模块添加到运行时：

1. 在代码编辑器中打开`runtime/Cargo.toml`文件，并将 Collectibles 模块添加到运行时的本地依赖项和标准功能中。

   例如：

   ```toml
   # Local Dependencies
   pallet-template = { version = "4.0.0-dev", default-features = false, path = "../pallets/template" }
   collectibles = { default-features = false, path = "../pallets/collectibles" }

   [features]
   default = ["std"]
   std = [
   ...
        "collectibles/std",
   ...
   ```

1. 保存更改。

1. 打开`runtime/src/lib.rs`。

1. 将 Collectibles 模块导入到运行时中。

   ```rust
   /// Import the Collectibles pallet.
   pub use collectibles;
   ```

1. 为 collectibles 模块实现配置特征。

   ```rust
   impl collectibles::Config for Runtime {
        type RuntimeEvent = RuntimeEvent;
        type Currency = Balances;
        type CollectionRandomness = RandomnessCollectiveFlip;
        type MaximumOwned = frame_support::pallet_prelude::ConstU32<100>;
   }
   ```

1. 将模块添加到`construct_runtime!`宏中。

   ```rust
   construct_runtime!(
        pub struct Runtime
        where
            Block = Block,
            NodeBlock = opaque::Block,
            UncheckedExtrinsic = UncheckedExtrinsic,
        {
            System: frame_system,
            RandomnessCollectiveFlip: pallet_randomness_collective_flip,
            Timestamp: pallet_timestamp,
            Aura: pallet_aura,
            Grandpa: pallet_grandpa,
            Balances: pallet_balances,
            TransactionPayment: pallet_transaction_payment,
            Sudo: pallet_sudo,
            TemplateModule: pallet_template,
            Collectibles: collectibles,
        }
   );
   ```

1. 通过运行以下命令使用更新的运行时编译区块链节点：

   ```bash
   cargo build --release
   ```

   节点编译完成后，您的自定义模块就可以使用了。

## 访问 Collectibles 模块

现在您已经拥有了一个新编译的节点，您可以重新启动区块链并使用您的新模块。

1. 通过运行以下命令启动区块链节点：

   ```bash
   ./target/release/solo-template-node --dev
   ```

2. 打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer)并连接到您的本地**开发**节点。

   ![连接到本地节点](/media/images/docs/tutorials/collectibles-workshop/connect-to-local-endpoint.png)

1. 点击**开发者**并选择**外部函数**。

2. 选择**Collectibles**模块并查看可调用函数列表。

   ![Collectibles 模块中的可调用函数](/media/images/docs/tutorials/collectibles-workshop/collectibles-pallet.png)

3. 选择**createCollectible**函数，点击**提交事务**，然后点击**签名并提交**。

4. 点击**网络**并选择**资源管理器**以查看为创建新的可收藏物品而发出的事件。

   ![CollectibleCreated 事件](/media/images/docs/tutorials/collectibles-workshop/create-collectible-event.png)

   您可以使用 Polkadot/Substrate 门户来测试函数、事件和错误是否按预期工作。
   但是，为了让您的应用程序吸引用户，您需要开发一个自定义界面，让用户能够浏览和竞标其他用户创建并出售的可收藏物品。
   构建引人入胜的界面本身就是一门艺术，因此在本研讨会中，我们将保持简单，不深入探讨应用程序的外观或应提供的用户体验。
   在大多数情况下，您需要使用 TypeScript 和 Web 开发框架（例如 React、Vue 或 Angular）来构建在区块链上运行的应用程序的前端。
