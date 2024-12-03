---
title: 向运行时添加模块
description: 演示向 Substrate 节点模板的运行时添加简单模块的基本步骤。
keywords:
  - 运行时
  - FRAME
  - 模块
  - 依赖项
  - Nicks
  - 配置
---

正如您在[构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)中看到的那样，[Substrate 节点模板](https://github.com/substrate-developer-hub/substrate-node-template)提供了一个可工作的**运行时**，其中包含一些默认的 FRAME 开发模块（**模块**），可帮助您开始构建自定义区块链。

本教程介绍了向节点模板的运行时添加新模块的基本步骤。
无论何时要向运行时添加新的 FRAME 模块，步骤都类似。
但是，每个模块都需要特定的配置设置——例如，执行模块实现的函数所需的参数和类型。
在本教程中，您将向节点模板的运行时添加[Nicks 模块](https://paritytech.github.io/substrate/master/pallet_nicks/index.html)，因此您将了解如何配置特定于 Nicks 模块的设置。
Nicks 模块允许区块链用户支付押金以预留他们控制的帐户的昵称。它实现了以下功能：

- `set_name` 函数，用于收集押金并在名称未被占用时设置帐户的名称。
- `clear_name` 函数，用于删除与帐户关联的名称并返还押金。
- `kill_name` 函数，用于强制删除帐户名称而不返还押金。

请注意，本教程是更高级教程的垫脚石，这些教程说明了如何添加具有更复杂配置设置的模块、如何创建自定义模块以及如何发布模块。

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已完成[构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)教程，并在本地安装了来自开发者中心的 Substrate 节点模板。

- 您通常熟悉软件开发和使用命令行界面。

- 您通常熟悉区块链和智能合约平台。

## 教程目标

通过完成本教程，您将使用 Nicks 模块来实现以下目标：

- 学习如何更新运行时依赖项以包含新的模块。

- 学习如何配置特定于模块的 Rust 特征。

- 通过使用前端模板与新模块交互来查看运行时的更改。

## 添加 Nicks 模块依赖项

在您可以使用新的模块之前，您必须向编译器用来构建运行时二进制文件的配置文件中添加一些关于它的信息。

对于 Rust 程序，您可以使用 `Cargo.toml` 文件来定义配置设置和依赖项，这些设置和依赖项决定了最终二进制文件中包含的内容。
因为 Substrate 运行时编译为包含标准库 Rust 函数的原生平台二进制文件和不包含标准 Rust 库的[WebAssembly (Wasm)](https://webassembly.org/)二进制文件，所以 `Cargo.toml` 文件控制两条重要的信息：

- 要作为运行时依赖项导入的模块，包括要导入的模块的位置和版本。
- 编译原生 Rust 二进制文件时应启用的每个模块中的功能。通过启用每个模块的标准（`std`）功能集，您可以编译运行时以包含在构建 WebAssembly 二进制文件时否则会缺少的函数、类型和基元。

有关在 `Cargo.toml` 文件中添加依赖项的信息，请参阅 Cargo 文档中的[依赖项](https://doc.rust-lang.org/cargo/guide/dependencies.html)。
有关启用和管理依赖包中的功能的信息，请参阅 Cargo 文档中的[功能](https://doc.rust-lang.org/cargo/reference/features.html)。

要向运行时添加 Nicks 模块的依赖项：

1. 打开终端 shell 并更改到节点模板的根目录。

1. 在文本编辑器中打开 `runtime/Cargo.toml` 配置文件。

1. 找到[依赖项]部分并记下其他模块是如何导入的。

1. 复制现有的模块依赖项描述，并将模块名称替换为 `pallet-nicks` 以使该模块可用于节点模板运行时。

   例如，添加类似于以下内容的行：

   ```toml
   pallet-nicks = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0" }
   ```

   此行将 `pallet-nicks` crates作为依赖项导入，并指定以下内容：

   - 版本，用于标识要导入的crates的版本。
   - 使用标准 Rust 库编译运行时时包含模块功能的默认行为。
   - 用于检索 `pallet-nicks` crates的存储库位置。
   - 用于检索crates的分支。请确保对 Nicks 模块使用与运行时中包含的其他模块相同的**版本**和**分支**信息。

   在节点模板的任何给定版本中，这些详细信息对于每个模块都应该相同。

1. 将 `pallet-nicks/std` 功能添加到 `features` 列表中，以便在编译运行时时启用。

   ```toml
   [features]
   default = ["std"]
   std = [
      ...
      "pallet-aura/std",
      "pallet-balances/std",
      "pallet-nicks/std",
      ...
   ]
   ```

   此部分指定要为此运行时编译的默认功能集是 `std` 功能集。
   当使用 `std` 功能集编译运行时时，将启用作为依赖项列出的所有模块的 `std` 功能。
   有关如何使用标准 Rust 库作为平台原生二进制文件编译运行时以及如何使用 `no_std` 属性作为 WebAssembly 二进制文件编译运行时的更多详细信息，请参阅[构建过程](/build/build-process/)。

   如果您忘记更新 `Cargo.toml` 文件中的 `features` 部分，则在编译运行时二进制文件时可能会看到 `cannot find function` 错误。

2. 通过运行以下命令检查新依赖项是否正确解析：

   ```bash
   cargo check -p node-template-runtime --release
   ```

## 查看 Balances 的配置

每个模块都有一个名为 `Config` 的[Rust **特征**](https://doc.rust-lang.org/book/ch10-02-traits.html)。
`Config` 特征用于标识模块执行其功能所需的参数和类型。

使用 `Config` 特征实现了添加模块所需的大部分模块特定代码。
您可以通过参考其 Rust 文档或模块的源代码来查看需要为任何模块实现的内容。
例如，要查看需要为 `nicks` 模块实现的内容，您可以参考[`pallet_nicks::Config`](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/trait.Config.html)的 Rust 文档或[Nicks 模块源代码](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/nicks/src/lib.rs)中的特征定义。

对于本教程，您可以看到 `nicks` 模块中的 `Config` 特征声明了以下类型：

```rust
pub trait Config: Config {
    type RuntimeEvent: From<Event<Self>> + IsType<<Self as Config>::RuntimeEvent>;
    type Currency: ReservableCurrency<Self::AccountId>;
    type ReservationFee: Get<<<Self as Config>::Currency as Currency<<Self as Config>::AccountId>>::Balance>;
    type Slashed: OnUnbalanced<<<Self as Config>::Currency as Currency<<Self as Config>::AccountId>>::NegativeImbalance>;
    type ForceOrigin: EnsureOrigin<Self::RuntimeOrigin>;
    type MinLength: Get<u32>;
    type MaxLength: Get<u32>;
}
```

确定模块所需的类型后，您需要向运行时添加代码以实现 `Config` 特征。
要查看如何为模块实现 `Config` 特征，让我们以**Balances**模块为例。

要查看 Balances 模块的 `Config` 特征：

1. 在文本编辑器中打开 `runtime/src/lib.rs` 文件。

1. 找到 `Balances` 模块并注意它包含以下实现（`impl`）代码块：

   ```text
   pub type Balance = u128;

   // ...

   /// Existential deposit.
   pub const EXISTENTIAL_DEPOSIT: u128 = 500;

   impl pallet_balances::Config for Runtime {
      type MaxLocks = ConstU32<50>;
      type MaxReserves = ();
      type ReserveIdentifier = [u8; 8];
      /// The type for recording an account's balance.
      type Balance = Balance;
      /// The ubiquitous event type.
      type RuntimeEvent = RuntimeEvent;
      /// The empty value, (), is used to specify a no-op callback function.
      type DustRemoval = ();
      /// Set the minimum balanced required for an account to exist on-chain
      type ExistentialDeposit = ConstU128<EXISTENTIAL_DEPOSIT>;
      /// The FRAME runtime system is used to track the accounts that hold balances.
      type AccountStore = System;
      /// Weight information is supplied to the Balances pallet by the node template runtime.
      type WeightInfo = pallet_balances::weights::SubstrateWeight<Runtime>;
   }
   ```

   正如您在此示例中看到的那样，`impl pallet_balances::Config` 块允许您配置由 Balances 模块 `Config` 特征指定的类型和参数。
   例如，此 `impl` 块将 Balances 模块配置为使用 `u128` 类型来跟踪余额。

## 实现 Nicks 的配置

现在您已经看到了如何为 Balances 模块实现 `Config` 特征的示例，您就可以为 Nicks 模块实现 `Config` 特征了。

要在运行时实现 `nicks` 模块：

1. 在文本编辑器中打开 `runtime/src/lib.rs` 文件。

1. 找到 Balances 代码块的最后一行。

1. 为 Nicks 模块添加以下代码块：

   ```rust
   impl pallet_nicks::Config for Runtime {
    // The Balances pallet implements the ReservableCurrency trait.
    // `Balances` is defined in `construct_runtime!` macro.
    type Currency = Balances;

    // Set ReservationFee to a value.
    type ReservationFee = ConstU128<100>;

    // No action is taken when deposits are forfeited.
    type Slashed = ();

    // Configure the FRAME System Root origin as the Nick pallet admin.
    // https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Root
    type ForceOrigin = frame_system::EnsureRoot<AccountId>;

    // Set MinLength of nick name to a desired value.
    type MinLength = ConstU32<8>;

    // Set MaxLength of nick name to a desired value.
    type MaxLength = ConstU32<32>;

    // The ubiquitous event type.
    type RuntimeEvent = RuntimeEvent;
   }

1. 将 Nicks 添加到 `construct_runtime!` 宏中。

   例如：

   ```rust
   construct_runtime!(
   pub enum Runtime where
       Block = Block,
       NodeBlock = opaque::Block,
       UncheckedExtrinsic = UncheckedExtrinsic
     {
       /* --snip-- */
       Balances: pallet_balances,

       /*** 添加此行 ***/
       Nicks: pallet_nicks,
     }
   );
   ```

1. 保存更改并关闭文件。

2. 通过运行以下命令检查新依赖项是否正确解析：

   ```bash
   cargo check -p node-template-runtime --release
   ```

   如果没有错误，则可以准备编译。

3. 通过运行以下命令以发布模式编译节点：

   ```bash
   cargo build --release
   ```

## 启动区块链节点

节点编译完成后，您可以启动已通过[Nicks 模块](https://paritytech.github.io/substrate/master/pallet_nicks/index.html)增强了昵称功能的节点，并使用前端模板与之交互。

要启动本地 Substrate 节点：

1. 根据需要打开终端 shell。

1. 更改到 Substrate 节点模板的根目录。

1. 通过运行以下命令以开发模式启动节点：

   ```bash
   ./target/release/node-template --dev
   ```

   在本例中，`--dev`选项指定节点使用预定义的`development`链规范在开发模式下运行。
   默认情况下，此选项还会在您按 Control-c 停止节点时删除所有活动数据（例如密钥、区块链数据库和网络信息）。
   使用`--dev`选项可确保您在每次停止和重新启动节点时都拥有干净的工作状态。

1. 通过查看终端中显示的输出，验证您的节点是否已成功启动并运行。

   如果控制台输出中`finalized`后面的数字在增加，则您的区块链正在生成新区块并就它们描述的状态达成共识。

1. 保持显示节点输出的终端打开以继续。

## 启动前端模板

现在您已向运行时添加了新的模块，您可以使用[Substrate 前端模板](/tutorials/build-a-blockchain/build-local-blockchain/#install-the-front-end-template)与节点模板交互并访问 Nicks 模块。

要启动前端模板：

1. 在您的计算机上打开一个新的终端 shell。

1. 在新终端中，更改到您安装前端模板的根目录。

1. 通过运行以下命令启动前端模板的 Web 服务器：

   ```bash
   yarn start
   ```

1. 在浏览器中打开`http://localhost:8000/`以查看前端模板。

## 使用 Nicks 模块设置昵称

启动前端模板后，您可以使用它与您刚刚添加到运行时的 Nicks 模块进行交互。

要为帐户设置昵称：

1. 检查帐户选择列表以验证当前是否选择了 Alice 帐户。

1. 在 Pallet Interactor 组件中，验证是否选择了**外部函数**。

2. 从可调用的模块列表中选择**nicks**。

3. 选择[**setName**](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Call.html#variant.set_name)作为要从 nicks 模块调用的函数。

4. 输入一个长度超过`MinNickLength`（8 个字符）且不超过`MaxNickLength`（32 个字符）的**名称**。

   ![选择要调用的模块和函数](/media/images/docs/tutorials/add-a-pallet/set-name-function.png)

5. 点击**已签名**以执行该函数。

6. 观察调用的状态从 Ready 更改为 InBlock 再更改为 Finalized，并记下 Nicks 模块发出的[事件](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Event.html)。

   ![Alice 的昵称已成功更新](/media/images/docs/tutorials/add-a-pallet/set-name-result.png)

## 使用 Nicks 模块查询帐户信息

接下来，您可以使用查询功能从 Nicks 模块的运行时存储中读取 Alice 的昵称值。

要返回为 Alice 存储的信息：

1. 在 Pallet Interactor 组件中，选择**查询**作为交互类型。

1. 从可查询的模块列表中选择**nicks**。

1. 选择[**nameOf**](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Call.html#variant.set_name)作为要调用的函数。

1. 在 AccountId 字段中复制并粘贴**alice**帐户的地址，然后点击**查询**。

   ![读取名称](/media/images/docs/tutorials/add-a-pallet/Alice-query-result.png)

   返回类型是一个包含两个值的元组：

   - Alice 帐户的十六进制编码昵称`53756273747261746520737570657273746172202d20416c696365`。
    如果您将十六进制编码值转换为字符串，您将看到您为`setName`函数指定的名称。
   - 从 Alice 的帐户中预留以确保昵称的金额（`100`）。

   如果您要查询 Bob 帐户的 Nicks 模块的 `nameOf`，则会看到返回的值为 `None`，因为 Bob 没有调用 `setName` 函数来预留昵称。

## 探索其他功能

本教程说明了如何向运行时添加简单的模块，并演示了如何使用预定义的前端模板与新模块进行交互。
在本例中，您向运行时添加了`nicks`模块，并使用前端模板调用了`set_name`和`nameOf`函数。
`nicks`模块还提供了另外两个函数——`clear_name`函数和`kill_name`函数——使帐户所有者能够删除预留的名称，或使根级用户能够强制删除帐户名称。
您可以通过探索这些函数的工作方式来了解其他功能（例如 Sudo 模块和来源帐户的使用）。
但是，这些功能超出了本教程的预期范围。
如果您想探索 Nicks 和 Sudo 模块公开的其他功能，请参阅[下一步](#next-steps)并选择[为调用指定来源](/tutorials/build-application-logic/specify-the-origin-for-a-call)。

## 下一步

有几个[教程](/tutorials/)可以作为学习更多关于 Substrate 开发的下一步。

- [为调用指定来源](/tutorials/build-application-logic/specify-the-origin-for-a-call)探讨了使用不同的来源帐户调用函数。
- [开发智能合约](/tutorials/smart-contracts/)指导您使用 ink!来构建智能合约。
- [在自定义模块中使用宏](/tutorials/build-application-logic/use-macros-in-a-custom-pallet)说明了如何使用宏来创建您自己的模块。
