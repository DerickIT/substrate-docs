---
title: 开发智能合约
description: 开发一个递增值的智能合约。
keywords:
---

在[准备您的第一个合约](/tutorials/smart-contracts/prepare-your-first-contract/)中，您学习了使用默认的第一个项目在基于 Substrate 的区块链上构建和部署智能合约的基本步骤。

在本教程中，您将开发一个新的智能合约，每次执行函数调用时都会递增计数器值。

## 开始之前

在开始之前，请验证以下内容：

- 您拥有良好的互联网连接，并且可以访问本地计算机上的 shell 终端。

- 您通常熟悉软件开发和使用命令行界面。

- 您通常熟悉区块链和智能合约平台。

- 您已按照[安装](/install/)中的说明安装 Rust 并设置了开发环境。

- 您已完成[准备您的第一个合约](/tutorials/smart-contracts/prepare-your-first-contract/)，并在本地安装了 Substrate 合约节点。

## 教程目标

通过完成本教程，您将实现以下目标：

- 学习如何使用智能合约模板。

- 使用智能合约存储简单值。

- 使用智能合约递增和检索存储的值。

- 向智能合约添加公共函数和私有函数。

## 智能合约和 ink!

在[准备您的第一个合约](/tutorials/smart-contracts/prepare-your-first-contract/)中，您安装了`cargo-contract`包，以便通过命令行访问 ink!编程语言。

ink!语言是[嵌入式领域特定语言](https://wiki.haskell.org/Embedded_domain_specific_language)。

此语言使您可以使用 Rust 编程语言编写基于 WebAssembly 的智能合约。

该语言使用带有专用`#[ink(...)]`属性宏的标准 Rust 模式。

这些属性宏描述了智能合约的不同部分所代表的内容，以便可以将其转换为与 Substrate 兼容的 WebAssembly 字节码。

## 创建新的智能合约项目

在 Substrate 上运行的智能合约以项目的形式开始。
您可以使用`cargo contract`命令创建项目。

在本教程中，您将为`incrementer`智能合约创建一个新项目。

创建新项目会向项目目录添加新的项目目录和默认的入门文件（也称为**模板文件**）。

您将修改这些入门模板文件以构建`incrementer`项目的智能合约逻辑。

要为智能合约创建新项目：

1. 如果您还没有打开终端 shell，请在本地计算机上打开一个。

1. 通过运行以下命令创建一个名为`incrementer`的新项目：

   ```bash
   cargo contract new incrementer
   ```

1. 通过运行以下命令更改到新项目目录：

   ```bash
   cd incrementer/
   ```

1. 在文本编辑器中打开`lib.rs`文件。

   默认情况下，模板`lib.rs`文件包含`flipper`智能合约的源代码，其中`flipper`合约名称的实例已重命名为`incrementer`。

1. 将默认模板源代码替换为新的[incrementer](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/incrementer-template.rs)源代码。

1. 保存对`lib.rs`文件的更改，然后关闭该文件。

1. 通过运行以下命令验证程序是否已编译并通过了简单的测试：

   ```bash
   cargo test
   ```

   您可以忽略任何警告，因为此模板代码只是一个框架。
   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 1 test
   test incrementer::tests::default_works ... ok

   test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

1. 通过运行以下命令验证您是否可以构建合约的 WebAssembly：

   ```bash
   cargo contract build
   ```

   如果程序成功编译，则您可以开始编程。

## 存储简单值

现在您已经拥有了`incrementer`智能合约的一些入门源代码，您可以引入一些新功能。

例如，此智能合约需要存储简单值。

本节中的以下代码旨在说明 ink!语言的功能。
您将在本教程的下一节[更新您的智能合约](#update-your-smart-contract)中开始使用代码。

您可以使用`#[ink(storage)]`属性宏为合约存储简单值：

```rust
#[ink(storage)]
pub struct MyContract {
  // Store a bool
  my_bool: bool,
  // Store a number
  my_number: u32,
}
```

### 支持的类型

ink!智能合约支持大多数 Rust 常用数据类型，包括布尔值、无符号整数和有符号整数、字符串、元组和数组。

这些数据类型使用[Parity scale 编解码器](https://github.com/paritytech/parity-codec)进行编码和解码，以便高效地通过网络传输。

除了可以使用 scale 编解码器进行编码和解码的常用 Rust 类型外，ink!语言还支持 Substrate 特定的类型（如`AccountId`、`Balance`和`Hash`），就好像它们是基本类型一样。

以下代码说明了如何为此合约存储`AccountId`和`Balance`：

```rust
#[ink::contract]
mod MyContract {

  // Our struct will use those default ink! types
  #[ink(storage)]
  pub struct MyContract {
    // Store some AccountId
    my_account: AccountId,
    // Store some Balance
    my_balance: Balance,
  }
/* --snip-- */
}
```

### 构造函数

每个 ink!智能合约都必须至少有一个在创建合约时运行的构造函数。
但是，智能合约可以根据需要有多个构造函数。

以下代码说明了如何使用多个构造函数：

```rust
#[ink::contract]
mod my_contract {

    #[ink(storage)]
    pub struct MyContract {
        number: u32,
    }

    impl MyContract {
        /// Constructor that initializes the `u32` value to the given `init_value`.
        #[ink(constructor)]
        pub fn new(init_value: u32) -> Self {
            Self {
                number: init_value,
            }
        }

        /// Constructor that initializes the `u32` value to the `u32` default (0).
        ///
        /// Constructors can delegate to other constructors.
        #[ink(constructor)]
        pub fn default() -> Self {
            Self {
                number: Default::default(),
            }
        }
    /* --snip-- */
    }
}
```

## 更新您的智能合约

现在您已经学习了存储简单值、声明数据类型和使用构造函数，您可以更新智能合约源代码以实现以下内容：

- 创建一个名为`value`的存储值，其数据类型为`i32`。
- 创建一个新的`Incrementer`构造函数，并将其`value`设置为`init_value`。
- 创建一个名为`default`的第二个构造函数，该函数没有输入，并创建一个新的`Incrementer`，其`value`设置为`0`。

要更新智能合约：

1. 在文本编辑器中打开`lib.rs`文件。

1. 通过声明名为`value`且数据类型为`i32`的存储项来替换“存储声明”注释。

   ```rust
   #[ink(storage)]
   pub struct Incrementer {
       value: i32,
   }
   ```

1. 修改`Incrementer`构造函数以将其`value`设置为`init_value`。

   ```rust
   impl Incrementer {
        #[ink(constructor)]
        pub fn new(init_value: i32) -> Self {
            Self { value: init_value }
        }
   }
   ```

1. 添加一个名为`default`的第二个构造函数，该函数创建一个新的`Incrementer`，其`value`设置为`0`。

   ```rust
   #[ink(constructor)]
   pub fn default() -> Self {
       Self {
           value: 0,
       }
   }
   ```

1. 保存更改并关闭文件。

1. 尝试再次运行`test`子命令，您会看到测试现在失败了。这是因为我们需要更新`get`函数并修改测试以匹配我们实现的更改。我们将在下一节中进行此操作。

## 添加一个函数来获取存储值

现在您已经创建并初始化了存储值，您可以使用公共函数和私有函数与之交互。
在本教程中，您将添加一个公共函数（也称为消息）来获取存储值。

请注意，所有公共函数都必须使用`#[ink(message)]`属性宏。

要向智能合约添加公共函数：

1. 在文本编辑器中打开`lib.rs`文件。

1. 更新`get`公共函数以返回具有`i32`数据类型的`value`存储项的数据。

   ```rust
   #[ink(message)]
   pub fn get(&self) -> i32 {
       self.value
   }
   ```

   由于此函数仅_读取_合约存储，因此它使用`&self`参数来访问合约函数和存储项。

   此函数不允许更改`value`存储项的状态。

   如果函数中的最后一个表达式没有分号（;），Rust 会将其视为返回值。

1. 将私有`default_works`函数中的“测试您的合约”注释替换为测试`get`函数的代码。

   ```rust
   #[ink::test]
   fn default_works() {
       let contract = Incrementer::default();
       assert_eq!(contract.get(), 0);
   }
   ```

1. 保存更改并关闭文件。

1. 使用`test`子命令检查您的工作，您会看到它仍然失败，因为我们需要更新`it_works`测试并添加一个新的公共函数来递增`value`存储项。

   ```bash
   cargo test
   ```

## 添加一个函数来修改存储值

此时，智能合约不允许用户修改存储。
要启用用户_修改_存储项，您必须将`value`显式标记为可变变量。

要添加用于递增存储值的函数：

1. 在文本编辑器中打开`lib.rs`文件。

1. 添加一个新的`inc`公共函数，使用数据类型为`i32`的`by`参数来递增存储的`value`。

   ```rust
   #[ink(message)]
   pub fn inc(&mut self, by: i32) {
       self.value += by;
   }
   ```

1. 向源代码添加新的测试以验证此函数。

   ```rust
   #[ink::test]
   fn it_works() {
       let mut contract = Incrementer::new(42);
       assert_eq!(contract.get(), 42);
       contract.inc(5);
       assert_eq!(contract.get(), 47);
       contract.inc(-50);
       assert_eq!(contract.get(), -3);
   }
   ```

1. 保存更改并关闭文件。

1. 使用`test`子命令检查您的工作：

   ```bash
   cargo test
   ```

   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 2 tests
   test incrementer::tests::it_works ... ok
   test incrementer::tests::default_works ... ok

   test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

### 构建合约的 WebAssembly

测试`incrementer`合约后，您可以将此项目编译为 WebAssembly。

要为此智能合约构建 WebAssembly：

1. 根据需要在您的计算机上打开终端 shell。

1. 验证您是否位于`incrementer`项目文件夹中。

1. 通过运行以下命令编译`incrementer`智能合约：

   ```bash
   cargo contract build
   ```

   该命令显示类似于以下内容的输出：

   ```text
   Your contract artifacts are ready. You can find them in:
   /Users/dev-docs/incrementer/target/ink

   - incrementer.contract (code + metadata)
   - incrementer.wasm (the contract's code)
   - incrementer.json (the contract's metadata)
   ```

## 部署和测试智能合约

如果您在本地安装了[`substrate-contracts-node`](https://github.com/paritytech/substrate-contracts-node)节点，则可以为智能合约启动本地区块链节点。

然后，您可以使用`cargo-contract`来部署和测试智能合约。

要在本地节点上部署：

1. 根据需要在您的计算机上打开终端 shell。

2. 通过运行以下命令以本地开发模式启动合约节点：

   ```bash
   substrate-contracts-node --log info,runtime::contracts=debug 2>&1
   ```

3. 上传并实例化合约

   ```bash
   cargo contract instantiate --constructor default --suri //Alice --salt $(date +%s)
   ```

   ```text
   Dry-running default (skip with --skip-dry-run)
       Success! Gas required estimated at Weight(ref_time: 321759143, proof_size: 0)
   Confirm transaction details: (skip with --skip-confirm)
    Constructor default
           Args
      Gas limit Weight(ref_time: 321759143, proof_size: 0)
   Submit? (Y/n):
      Events
       Event Balances ➜ Withdraw
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         amount: 2.953956313mUNIT
       ... snip ...
       Event System ➜ ExtrinsicSuccess
         dispatch_info: DispatchInfo { weight: Weight { ref_time: 2772097885, proof_size: 0 }, class: Normal, pays_fee: Yes }

   Code hash 0x71ddef2422fdb8358b503d5ef122c088a2dc6486dd460c37b01d672a8d319959
   Contract 5Cf6wFEyZnqvNJaKVxnWswefo7uT4jVsgzWKh8b78GLDV6kN
   ```

4. 递增值

   ```bash
   cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS --message inc --args 42 --suri //Alice
   ```

  ```text
   Dry-running inc (skip with --skip-dry-run)
    Success! Gas required estimated at Weight(ref_time: 8013742080, proof_size: 262144)
   Confirm transaction details: (skip with --skip-confirm)
        Message inc
           Args 42
      Gas limit Weight(ref_time: 8013742080, proof_size: 262144)
   Submit? (Y/n):
      Events
       Event Balances ➜ Withdraw
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         amount: 98.97416μUNIT
       Event Contracts ➜ Called
         caller: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         contract: 5Cf6wFEyZnqvNJaKVxnWswefo7uT4jVsgzWKh8b78GLDV6kN
       Event TransactionPayment ➜ TransactionFeePaid
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         actual_fee: 98.97416μUNIT
         tip: 0UNIT
       Event System ➜ ExtrinsicSuccess
         dispatch_info: DispatchInfo { weight: Weight { ref_time: 1383927346, proof_size: 13255 }, class: Normal, pays_fee: Yes }
  ```

5. 获取当前值

   ```bash
   cargo contract call --contract 5Cf6wFEyZnqvNJaKVxnWswefo7uT4jVsgzWKh8b78GLDV6kN --message get --suri //Alice --dry-run
   ```

   ```text
   Result Success!
   Reverted false
     Data Tuple(Tuple { ident: Some("Ok"), values: [Int(42)] })
   ```

正如您所看到的，从合约中读取的`value`为`42`，这与我们之前的步骤匹配！

## 下一步

在本教程中，您学习了使用 ink!编程语言和属性宏创建智能合约的一些基本技术。

例如，本教程说明了：

- 如何在新智能合约项目中添加存储项、指定数据类型和实现构造函数。

- 如何向智能合约添加函数。

- 如何向智能合约添加测试。

- 如何使用`cargo-contract`上传和实例化合约。

您可以在[智能合约](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/incrementer-basics.rs)的资产中找到本教程最终代码的示例。

您可以在以下主题中了解有关智能合约开发的更多信息：

- [使用映射存储值](/tutorials/smart-contracts/use-maps-for-storing-values/)
- [构建 ERC20 代币合约](/tutorials/smart-contracts/build-a-token-contract/)
- [对智能合约进行故障排除](/tutorials/smart-contracts/troubleshoot-smart-contracts/)
