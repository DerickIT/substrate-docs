---
title: 在自定义模块中使用宏
description: 使用 FRAME 宏的框架创建用于 Substrate 运行时的自定义模块。
keywords:
---

本教程说明了如何使用 FRAME 开发环境中包含的**宏**为 Substrate 运行时创建自定义模块。

在本教程中，您将构建一个简单的**存在证明**应用程序。存在证明是一种通过在区块链上存储有关对象的信息来验证数字对象真实性和所有权的方法。
由于区块链将时间戳和帐户与对象关联起来，因此区块链记录可用于“证明”特定对象在特定日期和时间存在。
它还可以验证该日期和时间记录的所有者是谁。

## 数字对象和哈希

与其在区块链上存储整个文件，不如只存储该文件的[加密哈希](https://en.wikipedia.org/wiki/Cryptographic_hash_function)效率更高。
这也称为“数字指纹”。
哈希使区块链能够通过使用小型且唯一的哈希值来高效地存储任意大小的文件。
由于对文件的任何更改都会导致不同的哈希值，因此用户可以通过计算哈希值并将该哈希值与链上存储的哈希值进行比较来证明文件的有效性。

![文件哈希](/media/images/docs/tutorials/custom-pallet/file-hash.png)

## 数字对象和帐户签名

区块链使用[公钥加密](https://en.wikipedia.org/wiki/Public-key_cryptography)将数字身份映射到拥有私钥的帐户。
区块链将您用于存储数字对象哈希的帐户记录为事务的一部分。
由于帐户信息存储为事务的一部分，因此该帐户私钥的控制器可以稍后证明其所有权，即最初上传文件的人。

## 完成本教程需要多长时间？

本教程需要编译 Rust 代码，大约需要一到两个小时才能完成。

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已克隆了[Substrate 节点模板](https://github.com/paritytech/polkadot-sdk-solochain-template)，它将用作本教程的起点。

## 教程目标

通过完成本教程，您将实现以下目标：

- 了解自定义模块的基本结构。

- 查看 Rust 宏如何简化您需要编写的代码的示例。

- 启动包含自定义模块的区块链节点。

- 与公开存在证明模块的前端进行交互。

## 应用程序设计

存在证明应用程序公开了两个可调用函数：

- `create_claim()`允许用户通过上传哈希来声明文件的生存。

- `revoke_claim()`允许当前索赔的所有者撤销所有权。

## 构建自定义模块

Substrate 节点模板具有基于 FRAME 的运行时。
正如您在[运行时开发](/learn/runtime-development)中学到的那样，FRAME 是一个代码库，允许您通过组合称为模块的模块来构建 Substrate 运行时。
您可以将模块视为定义区块链功能的专用逻辑单元。
Substrate 为您提供了许多预构建的模块，可在基于 FRAME 的运行时中使用。

![运行时组合](/media/images/docs/tutorials/custom-pallet/frame-runtime.png)

本教程演示了如何创建您自己的 FRAME 模块以包含在您的自定义区块链中。

### 设置模块的脚手架

本教程演示了如何从头开始创建自定义模块。
因此，第一步是从节点模板目录中的文件中删除一些文件和内容。

1. 打开终端 shell 并导航到节点模板的根目录。

1. 通过运行以下命令更改到`pallets/template/src`目录：

   ```bash
   cd pallets/template/src
   ```

1. 删除以下文件：

   ```bash
   benchmarking.rs
   mock.rs
   tests.rs
   ```

1. 在文本编辑器中打开`lib.rs`文件。

   此文件包含可作为新模块模板使用的代码。
   在本教程中，您将不会使用模板代码。
   但是，在删除模板代码之前，您可以查看它提供了什么。

1. 删除`lib.rs`文件中的所有行，并将其替换为自定义模块的框架：

   ```rust
   // All pallets must be configured for `no_std`.
   #![cfg_attr(not(feature = "std"), no_std)]

   // Re-export pallet items so that they can be accessed from the crate namespace.
   pub use pallet::*;

   #[frame_support::pallet]
   pub mod pallet {
     use frame_support::pallet_prelude::*;
     use frame_system::pallet_prelude::*;

     #[pallet::pallet]
     pub struct Pallet<T>(_);

     #[pallet::config]  // <-- 步骤 2. 代码块将替换此内容。
     #[pallet::event]   // <-- 步骤 3. 代码块将替换此内容。
     #[pallet::error]   // <-- 步骤 4. 代码块将替换此内容。
     #[pallet::storage] // <-- 步骤 5. 代码块将替换此内容。
     #[pallet::call]    // <-- 步骤 6. 代码块将替换此内容。
   }

   pub mod weights {
      // Placeholder struct for the pallet weights
      pub struct SubstrateWeight<T>(core::marker::PhantomData<T>);
   }
   ```

   您现在拥有一个框架，其中包含_事件_、_错误_、_存储_和_可调用函数_的占位符。

   > 注意：模块权重不在本教程的范围内。如果您想了解有关权重的更多信息，您可以阅读我们的文档[此处](/reference/how-to-guides/weights/)。


1. 保存更改。

## 配置模块以发出事件

每个模块都有一个名为 `Config` 的[Rust “特征”](https://doc.rust-lang.org/book/ch10-02-traits.html)。
您可以使用此特征来公开可配置选项并将模块连接到运行时的其他部分。
对于本教程，我们需要配置模块以发出事件。

要为存在证明模块定义 `Config` 特征：

1. 在文本编辑器中打开`pallets/template/src/lib.rs`文件。

1. 将`#[pallet::config]`行替换为以下代码块：

   ```rust
   /// Configure the pallet by specifying the parameters and types on which it depends.
   #[pallet::config]
   pub trait Config: frame_system::Config {
     /// Because this pallet emits events, it depends on the runtime's definition of an event.
     type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
      /// Pallets use weights to measure the complexity of the callable functions.
      /// Configuring weights is outside the scope of this tutorial, so we will leave it empty for now.
      type WeightInfo;
   }
   ```

1. 保存更改。

## 实现模块事件

现在您已将模块配置为发出事件，您可以准备定义这些事件了。
根据[应用程序设计](#application-design)，我们希望存在证明模块在以下情况下发出事件：

- 当新的声明添加到区块链时。
- 当声明被撤销时。

每个事件都可以包含一个`AccountId`来标识触发事件的用户以及一个`Hash`，表示正在存储或删除的存在证明声明。

要实现模块事件：

1. 在文本编辑器中打开`pallets/template/src/lib.rs`文件。

1. 将`#[pallet::event]`行替换为以下代码块：

   ```rust
   // Pallets use events to inform users when important changes are made.
   // Event documentation should end with an array that provides descriptive names for parameters.
   #[pallet::event]
   #[pallet::generate_deposit(pub(super) fn deposit_event)]
   pub enum Event<T: Config> {
     /// Event emitted when a claim has been created.
     ClaimCreated { who: T::AccountId, claim: T::Hash },
     /// Event emitted when a claim is revoked by the owner.
     ClaimRevoked { who: T::AccountId, claim: T::Hash },
   }
   ```

1. 保存更改。

## 包含模块错误

您定义的事件指示何时成功完成对模块的调用。
错误表示调用失败以及失败的原因。
对于本教程，您定义了以下错误条件：

- 当声明已存在时尝试进行声明。

- 尝试撤销不存在的声明。

- 尝试撤销由另一个帐户拥有的声明。

要实现存在证明模块的错误：

1. 在文本编辑器中打开`pallets/template/src/lib.rs`文件。

1. 将`#[pallet::error]`行替换为以下代码块：

   ```rust
   #[pallet::error]
   pub enum Error<T> {
     /// The claim already exists.
     AlreadyClaimed,
     /// The claim does not exist, so it cannot be revoked.
     NoSuchClaim,
     /// The claim is owned by another account, so caller can't revoke it.
     NotClaimOwner,
   }
   ```

1. 保存更改。

## 为存储的项目实现存储映射

要向区块链添加新的声明，存在证明模块需要存储机制。
为了满足此要求，您可以创建一个键值映射，其中每个声明都指向所有者以及创建声明时的区块编号。
要创建此键值映射，您可以使用 FRAME[`StorageMap`](https://paritytech.github.io/polkadot-sdk/master/frame_support/storage/types/struct.StorageMap.html)。

要实现存在证明模块的存储：

1. 在文本编辑器中打开`pallets/template/src/lib.rs`文件。

1. 将`#[pallet::storage]`行替换为以下代码块：

   ```rust
   #[pallet::storage]
   pub(super) type Claims<T: Config> = StorageMap<_, Blake2_128Concat, T::Hash, (T::AccountId, BlockNumberFor<T>)>;
   ```

1. 保存更改。

## 实现可调用函数

存在证明模块向用户公开了两个可调用函数：

- `create_claim()`允许用户声明具有哈希的文件的存在。

- `revoke_claim()`允许声明的所有者撤销声明。

这些函数使用`StorageMap`来实现以下逻辑：

- 如果声明已存在于存储中，则它已拥有所有者，并且无法再次声明。
- 如果声明不存在于存储中，则它可供声明并写入存储。

要在存在证明模块中实现此逻辑：

1. 在文本编辑器中打开`pallets/template/src/lib.rs`文件。

1. 将`#[pallet::call]`行替换为以下代码块。您可以尝试自己实现`revoke_claim`函数。只需复制函数签名，而不是内容。应使用`Claims::<T>::get`和`Claims::<T>::remove`来获取或删除声明。

   ```rust
   // Dispatchable functions allow users to interact with the pallet and invoke state changes.
   // These functions materialize as "extrinsics", which are often compared to transactions.
   // Dispatchable functions must be annotated with a weight and must return a DispatchResult.
   #[pallet::call]
   impl<T: Config> Pallet<T> {
     #[pallet::weight(Weight::default())]
     #[pallet::call_index(0)]
     pub fn create_claim(origin: OriginFor<T>, claim: T::Hash) -> DispatchResult {
       // Check that the extrinsic was signed and get the signer.
       // This function will return an error if the extrinsic is not signed.
       let sender = ensure_signed(origin)?;

       // Verify that the specified claim has not already been stored.
       ensure!(!Claims::<T>::contains_key(&claim), Error::<T>::AlreadyClaimed);

       // Get the block number from the FRAME System pallet.
       let current_block = <frame_system::Pallet<T>>::block_number();

       // Store the claim with the sender and block number.
       Claims::<T>::insert(&claim, (&sender, current_block));

       // Emit an event that the claim was created.
       Self::deposit_event(Event::ClaimCreated { who: sender, claim });

       Ok(())
     }

     #[pallet::weight(Weight::default())]
     #[pallet::call_index(1)]
     pub fn revoke_claim(origin: OriginFor<T>, claim: T::Hash) -> DispatchResult {
       // Check that the extrinsic was signed and get the signer.
       // This function will return an error if the extrinsic is not signed.
       let sender = ensure_signed(origin)?;

       // Get owner of the claim, if none return an error.
       let (owner, _) = Claims::<T>::get(&claim).ok_or(Error::<T>::NoSuchClaim)?;

       // Verify that sender of the current call is the claim owner.
       ensure!(sender == owner, Error::<T>::NotClaimOwner);

       // Remove claim from storage.
       Claims::<T>::remove(&claim);

       // Emit an event that the claim was erased.
       Self::deposit_event(Event::ClaimRevoked { who: sender, claim });
       Ok(())
     }
   }
   ```

1. 保存更改并关闭文件。

1. 通过运行以下命令检查模块是否已编译：

   ```bash
   cargo check -p pallet-template
   ```

   `-p`标志告诉 cargo 只检查您一直在修改的文件，从而节省一些编译时间。

   如果您遇到问题，可以参考节点模板的[解决方案](https://github.com/paritytech/polkadot-sdk-solochain-template/blob/tutorials/solutions/proof-of-existence/pallets/template/src/lib.rs)。

## 使用新模块构建运行时

将存在证明模块的所有部分复制到`pallets/template/lib.rs`文件后，您可以编译并启动节点。

要编译并启动更新的 Substrate 节点：

1. 打开终端 shell。

1. 更改到节点模板的根目录。

1. 通过运行以下命令编译节点模板：

   ```bash
   cargo build --release
   ```

1. 通过运行以下命令以开发模式启动节点：

   ```bash
   ./target/release/solo-template-node --dev
   ```

   `--dev`选项使用预定义的`development`链规范启动节点。
   使用`--dev`选项可确保您在每次停止和重新启动节点时都拥有干净的工作状态。

1. 验证节点是否正在生成区块。

## 与区块链交互

现在您已经运行了一个包含自定义存在证明模块的新区块链，我们可以与链进行交互以确保所有功能都能按预期工作！

为此，我们将使用 Polkadot JS Apps，这是一个开发人员工具，可以连接并与任何基于 Substrate 的区块链进行交互。

默认情况下，您的区块链应该在`ws://127.0.0.1:9944`上运行，因此要连接到它，我们可以使用此链接：

https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/

如果您的 Substrate 区块链正在运行并且 Polkadot JS Apps 已连接，您应该会看到左上角的区块编号增加：

![Polkadot JS 资源管理器](/media/images/docs/tutorials/custom-pallet/poe-explorer.png)

### 提交声明

要使用前端测试存在证明模块：

1. 导航到["开发者 > 外部函数"](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/extrinsics)选项卡。

   ![外部函数选项卡](/media/images/docs/tutorials/custom-pallet/poe-extrinsics-tab.png)

1. 调整外部函数页面以选择“ALICE”作为帐户，并将“templateModule > createClaim”作为外部函数。

   ![创建声明](/media/images/docs/tutorials/custom-pallet/poe-create-claim.png)

1. 然后，您可以切换“哈希文件”，这将允许您选择一个文件来哈希并在区块链上声明。

   ![哈希文件](/media/images/docs/tutorials/custom-pallet/poe-hash-file.png)

1. 点击右下角的“提交事务”，然后在弹出窗口中点击“签名并提交”。

   ![提交外部函数](/media/images/docs/tutorials/custom-pallet/poe-submit.png)

1. 如果一切顺利，您应该会看到一条绿色的外部函数成功通知！

   ![外部函数成功](/media/images/docs/tutorials/custom-pallet/poe-success.png)

### 读取声明

本教程的最后一步是检查哪些声明已存储在您的区块链中。

1. 导航到["开发者 > 链状态"](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/chainstate)选项卡。

   ![链状态](/media/images/docs/tutorials/custom-pallet/poe-chain-state.png)

1. 将状态查询调整为“templateModule > claims”。

1. 切换哈希输入上的“包含选项”以使其为空。

   这将允许我们一次查看所有声明，而不仅仅是一个。

   ![查询所有声明](/media/images/docs/tutorials/custom-pallet/poe-claims.png)

1. 按“+”按钮执行查询。

   ![查询结果](/media/images/docs/tutorials/custom-pallet/poe-query.png)

   现在您可以看到该声明已存储在区块链中，其中包含有关所有者地址和创建声明时的区块编号的数据！

## 下一步

在本教程中，您学习了如何创建新的自定义模块的基础知识，包括：

- 如何向自定义模块添加事件、错误、存储和可调用函数。

- 如何将自定义模块集成到运行时。

- 如何编译和启动包含自定义模块的节点。

- 如何使用 Polkadot JS Apps 开发人员工具与自定义区块链进行交互。

本教程涵盖了基础知识，而没有深入研究代码。
但是，在您努力构建自己的完全自定义区块链时，您还可以做更多的事情。
自定义模块使您可以公开您希望区块链支持的功能。

要完成对存在证明链的理解，请尝试：

- 使用“ALICE”甚至“BOB”帐户再次声明相同的文件。
  - 您应该会收到错误！
- 使用“ALICE”和/或“BOB”帐户声明其他文件。
- 使用相应的声明所有者帐户撤销声明。
- 查看读取存储后声明的最终列表。

要了解通过创建自定义模块可以实现哪些功能，请浏览 FRAME 文档和[方法指南](/reference/how-to-guides)。
