---
title: 构建代币合约
description: 使用 ink!语言编写的智能合约构建 ERC-20 代币供应。
keywords:
  - erc20
  - 以太坊
  - ink
  - 代币
  - 可替代的
---

本教程说明了如何使用 ink!语言构建[ERC-20 代币](https://eips.ethereum.org/EIPS/eip-20)合约。

ERC-20 规范定义了可替代代币的通用标准。

定义代币属性的标准使遵循该规范的开发者能够构建可以与其他产品和服务互操作的应用程序。

ERC-20 代币标准并非唯一的代币标准，但它是最常用的标准之一。

## 开始之前

在开始之前，请验证以下内容：

- 您通常熟悉智能合约、代币和加密货币的概念和术语。

- 您已按照[安装](/install/)中的说明安装 Rust 并设置了开发环境。

- 您已完成[准备您的第一个合约](/tutorials/smart-contracts/prepare-your-first-contract/)，并在本地安装了 Substrate 合约节点。

- 您已完成[开发智能合约](/tutorials/smart-contracts/develop-a-smart-contract/)，并且熟悉 ink!如何使用 Rust 属性宏来构建智能合约。

## 教程目标

通过完成本教程，您将实现以下目标：

- 了解 ERC-20 标准中定义的基本属性和接口。

- 创建符合 ERC-20 标准的代币。

- 在合约之间转移代币。

- 处理涉及批准或第三方的转移活动的路由。

- 创建与代币活动相关的事件。

## ERC-20 标准的基础知识

[ERC-20 代币标准](https://eips.ethereum.org/EIPS/eip-20)定义了在以太坊区块链上运行的大多数智能合约的接口。

这些标准接口允许个人在其现有的智能合约平台之上部署他们自己的加密货币。

如果您查看该标准，您会注意到定义了以下核心函数。

```javascript
// ----------------------------------------------------------------------------
// ERC Token Standard #20 Interface
// https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
// ----------------------------------------------------------------------------

contract ERC20Interface {
    // Storage Getters
    function totalSupply() public view returns (uint);
    function balanceOf(address tokenOwner) public view returns (uint balance);
    function allowance(address tokenOwner, address spender) public view returns (uint remaining);

    // Public Functions
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    // Contract Events
    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```

用户余额映射到帐户地址，这些接口允许用户转移他们拥有的代币或允许第三方代表他们转移代币。

最重要的是，必须实现智能合约逻辑以确保不会无意中创建或销毁资金，并且用户的资金受到恶意行为者的保护。

请注意，所有公共函数都返回一个`bool`，它仅指示调用是否成功。

在 Rust 中，这些函数通常会返回一个`Result`。

## 创建代币供应

处理 ERC-20 代币的智能合约类似于使用映射存储[在使用映射存储值](/tutorials/smart-contracts/use-maps-for-storing-values/)中存储值的 Incrementer 合约。

在本教程中，ERC-20 合约由固定数量的代币组成，这些代币在部署合约时都存入与合约所有者关联的帐户中。

然后，合约所有者可以将代币分发给其他用户。

在本教程中创建的简单 ERC-20 合约并不代表您铸造和分发代币的唯一方式。

但是，此 ERC-20 合约为扩展您在其他教程中学到的内容以及如何使用 ink!语言构建更强大的智能合约提供了良好的基础。

对于 ERC-20 代币合约，初始存储包括：

- `total_supply`表示合约中代币的总供应量。
- `balances`表示每个帐户的个体余额。

要开始，让我们使用一些模板代码创建一个新项目。

要构建 ERC-20 代币智能合约：

1. 如果您还没有打开终端 shell，请在本地计算机上打开一个。

1. 通过运行以下命令创建一个名为`erc20`的新项目：

   ```bash
   cargo contract new erc20
   ```

1. 通过运行以下命令更改到新项目目录：

   ```bash
   cd erc20/
   ```

1. 在文本编辑器中打开`lib.rs`文件。

1. 将默认模板源代码替换为以下[ERC-20 模板](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-template.rs)。

1. 保存对`lib.rs`文件的更改，然后关闭该文件。

1. 通过运行以下命令验证程序是否已编译并通过了简单的测试：

   ```bash
   cargo test
   ```

   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 2 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok

   test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

1. 通过运行以下命令验证您是否可以构建合约的 WebAssembly：

   ```bash
   cargo contract build
   ```

   如果程序成功编译，则您可以以其当前状态上传它或开始向合约添加功能。

## 上传和实例化合约

如果您想测试您目前所拥有的内容，则可以使用[Contracts UI](https://contracts-ui.substrate.io)上传合约。

要在添加新函数之前测试 ERC-20 合约：

1. 启动本地合约节点。如有需要，您可以参考[准备您的第一个合约](/tutorials/smart-contracts/prepare-your-first-contract/)教程中的说明。

2. 使用`new()`构造函数实例化合约。

  ```bash
  cargo contract instantiate --constructor new --args 1_000_000 --suri //Alice --salt $(date +%s)
  ```

3. 通过调用`total_supply()`消息来验证`total_supply`。不要忘记添加`--dry-run`标志，因为我们只想从链状态读取。

  ```bash
  cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS \
      --message total_supply --suri //Alice --dry-run
  ```


4. 使用`balance_of()`验证所有代币的初始持有者 Alice 拥有的代币数量。

  ```bash
  cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS \
      --message balance_of --args 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY \
      --suri //Alice --dry-run
  ```

  请注意，我们必须将 Alice 的完整地址（`5Grw...utQY`）作为参数，因为`cargo-contract`（目前）不支持该位置的众所周知的密钥。

  如果您提供任何其他`AccountId`，则余额应为零，因为所有代币都由合约所有者拥有。

## 转移代币

此时，ERC-20 合约有一个用户帐户，该帐户拥有合约的`total_supply`代币。

要使此合约有用，合约所有者必须能够将代币转移到其他帐户。

对于此简单的 ERC-20 合约，您将添加一个公共`transfer`函数，使您（作为合约调用者）能够将您拥有的代币转移到另一个用户。

公共`transfer`函数调用私有`transfer_from_to()`函数。

由于这是一个内部函数，因此无需任何授权检查即可调用它。

但是，转移的逻辑必须能够确定`from`帐户是否有可转移到接收`to`帐户的代币。

`transfer_from_to()`函数使用合约调用者（`self.env().caller()`）作为`from`帐户。

在此上下文中，`transfer_from_to()`函数将执行以下操作：

- 获取`from`和`to`帐户的当前余额。

- 检查`from`余额是否小于要发送的代币数量`value`。

  ```rust
  let from_balance = self.balance_of(*from);
    if from_balance < value {
    return Err(Error::InsufficientBalance)
  }
  ```

- 从转移帐户中减去`value`，并将`value`添加到接收帐户中。

  ```rust
  self.balances.insert(&from, &(from_balance - value));
  let to_balance = self.balance_of(*to);
  self.balances.insert(&to, &(to_balance + value));
  ```

要将转移函数添加到智能合约：

1. 如果您还没有打开终端 shell，请在本地计算机上打开一个。

1. 验证您是否位于`erc20`项目目录中。

1. 在文本编辑器中打开`lib.rs`。

1. 添加一个`Error`声明，如果帐户中没有足够的代币来完成转移，则返回错误。

   ```rust
   /// Specify ERC-20 error type.
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
   /// Return if the balance cannot fulfill a request.
       InsufficientBalance,
   }
   ```

1. 添加一个`Result`返回类型以返回`InsufficientBalance`错误。

   ```rust
   /// Specify the ERC-20 result type.
   pub type Result<T> = core::result::Result<T, Error>;
   ```

1. 添加`transfer()`公共函数以使合约调用者能够将代币转移到另一个帐户。

   ```rust
   #[ink(message)]
   pub fn transfer(&mut self, to: AccountId, value: Balance) -> Result<()> {
       let from = self.env().caller();
       self.transfer_from_to(&from, &to, value)
   }
   ```

1. 添加`transfer_from_to()`私有函数以将代币从与合约调用者关联的帐户转移到接收帐户。

   ```rust
   fn transfer_from_to(
       &mut self,
       from: &AccountId,
       to: &AccountId,
       value: Balance,
   ) -> Result<()> {
        let from_balance = self.balance_of(*from);
        if from_balance < value {
            return Err(Error::InsufficientBalance)
        }

        self.balances.insert(&from, &(from_balance - value));
        let to_balance = self.balance_of(*to);
        self.balances.insert(&to, &(to_balance + value));

        Ok(())
   }
   ```

1. 添加一个测试，该测试将代币从一个帐户转移到另一个帐户。

   ```rust
   #[ink::test]
   fn transfer_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       assert!(contract.transfer(bob(), 10).is_ok());
       assert_eq!(contract.balance_of(bob()), 10);
       assert!(contract.transfer(bob(), 100).is_err());
   }
   ```

1. 通过运行以下命令验证程序是否已编译并通过了测试用例：

   ```bash
   cargo test
   ```

   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 3 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

如果您想查看我们的解决方案，可以查看
[此处](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-transfer.rs)。

## 创建事件

ERC-20 代币标准指定在提交事务时，合约调用不能直接返回值。
但是，您可能希望您的智能合约以某种方式发出事件已发生的信号。
例如，您可能希望您的智能合约指示何时进行了交易或批准了转移。
您可以使用[事件](https://use.ink/basics/events)来发送此类信号。

您可以使用事件来传达任何类型的数据。
定义事件的数据类似于定义`struct`。
应使用`#[ink(event)]`属性标记来声明事件。

### 添加转移事件

在本教程中，您将声明一个`Transfer`事件，以提供有关已完成转移操作的信息。
`Transfer`事件包含以下信息：

- `Balance`类型的数值。
- `from`帐户的 Option 包装的`AccountId`变量。
- `to`帐户的 Option 包装的`AccountId`变量。

为了更快地访问事件数据，它们可以具有_索引字段_。
您可以通过在该字段上使用`#[ink(topic)]`属性标记来实现此目的。

要添加`Transfer`事件：

1. 在文本编辑器中打开`lib.rs`文件。

1. 使用`#[ink(event)]`属性宏声明事件。

   ```rust
   #[ink(event)]
   pub struct Transfer {
       #[ink(topic)]
       from: Option<AccountId>,
       #[ink(topic)]
       to: Option<AccountId>,
       value: Balance,
     }
   ```

### 发出事件

现在您已经声明了事件并定义了事件包含的信息，您需要添加发出事件的代码。

您可以通过使用事件名称作为对调用的唯一参数来调用[`self.env().emit_event()`](https://docs.rs/ink_env/4.0.0-rc/ink_env/fn.emit_event.html)函数来实现此目的。

在此 ERC-20 合约中，您希望每次发生转移时都发出`Transfer`事件。
代码中有两个地方会发生这种情况：

- 在`new`调用期间初始化合约。

- 每次调用`transfer_from_to`时。

`from`和`to`字段的值为`Option<AccountId>`数据类型。
但是，在代币的初始转移期间，为*初始供应*设置的值并非来自任何其他帐户。
在这种情况下，Transfer 事件的`from`值为`None`。

要发出 Transfer 事件：

1. 在文本编辑器中打开`lib.rs`文件。

1. 将`Transfer`事件添加到`new()`构造函数中。

   ```rust
   #[ink(constructor)]
   pub fn new(total_supply: Balance) -> Self {
       // -- snip --

       Self::env().emit_event(Transfer {
           from: None,
           to: Some(caller),
           value: total_supply,
       });

       // -- snip --
   }
   ```

1. 将`Transfer`事件添加到`transfer_from_to()`函数中。

   ```rust
   fn transfer_from_to(
       &mut self,
       from: &AccountId,
       to: &AccountId,
       value: Balance,
   ) -> Result<()> {
      // -- snip --

       self.env().emit_event(Transfer {
           from: Some(*from),
           to: Some(*to),
           value,
       });

       // -- snip --
   }
   ```

   请注意，`value`不需要`Some()`，因为该值未存储在`Option`中。


1. 通过运行以下命令验证程序是否已编译并通过了所有测试：

   ```bash
   cargo test
   ```

   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 3 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

如果您想查看我们的解决方案，可以查看
[此处](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-event.rs)。

## 启用第三方转移

ERC-20 代币合约现在可以在帐户之间转移代币，并在发生这种情况时发出事件。
作为最后一步，您可以添加`approve`和`transfer_from`函数以启用第三方转移。

启用一个帐户代表另一个帐户花费代币，允许您的智能合约支持去中心化交易。

您不必直接在合约中将代币转移到另一个用户，而是可以批准您拥有的部分代币代表您进行交易。

在等待交易执行时，您仍然可以控制和花费您的代币（如有需要）。

您还可以批准多个合约或用户来访问您的代币，因此，如果一个合约提供了最佳交易，则您无需将代币从一个合约转移到另一个合约，这可能是一个成本高昂且耗时的过程。

为了确保可以安全地进行批准和转移，ERC-20 代币合约使用两步过程，分别进行**批准**和**从...转移**操作。

### 添加批准逻辑

批准另一个帐户代表您花费您的代币是第三方转移过程的第一步。

作为代币所有者，您可以指定任何帐户以及指定帐户可以代表您转移的任何数量的代币。

您不必批准帐户中的所有代币，您可以指定已批准的帐户允许转移的最大数量。

当您多次调用`approve`时，您会将先前批准的值替换为新值。
默认情况下，任何两个帐户之间的批准值为`0`。
如果您想撤销对帐户中代币的访问权限，则可以使用值为`0`调用`approve`函数。

要在 ERC-20 合约中存储批准，您需要使用稍微复杂的`Mapping`键。

由于每个帐户都可以为任何其他帐户使用不同的批准金额，因此您需要使用元组作为键来映射到余额值。

例如：

```rust
pub struct Erc20 {
 // -- snip --

 /// Balances that can be transferred by non-owners: (owner, spender) -> allowed
 allowances: Mapping<(AccountId, AccountId), Balance>,
}
```

该元组使用`(owner, spender)`来标识允许代表`owner`访问代币的`spender`帐户，最多可达指定的`allowance`。

要将批准逻辑添加到智能合约：

1. 在文本编辑器中打开`lib.rs`文件。

1. 使用`#[ink(event)]`属性宏声明`Approval`事件。

   ```rust
   #[ink(event)]
   pub struct Approval {
       #[ink(topic)]
       owner: AccountId,
       #[ink(topic)]
       spender: AccountId,
       value: Balance,
   }
   ```

1. 添加一个`Error`变体，如果转移请求超过帐户限额，则指示错误。

   ```rust
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
       InsufficientBalance,
       InsufficientAllowance,
   }
   ```

1. 为所有者和非所有者组合到帐户余额的存储声明中添加一个`allowances` `Mapping`。

   ```rust
   allowances: Mapping<(AccountId, AccountId), Balance>,
   ```

1. 在`new()`构造函数中实例化并添加`allowances` `Mapping`。

   ```rust
   #[ink(constructor)]
   pub fn new(total_supply: Balance) -> Self {
       // -- snip --

       let allowances = Mapping::default();

       Self {
           total_supply,
           balances,
           allowances
      }
   }
   ```

1. 添加`approve()`函数以授权`spender`帐户最多提取`value`数量的代币，这些代币来自调用者的帐户。

   ```rust
   #[ink(message)]
   pub fn approve(&mut self, spender: AccountId, value: Balance) -> Result<()> {
       let owner = self.env().caller();
       self.allowances.insert((owner, spender), &value);

       self.env().emit_event(Approval {
         owner,
         spender,
         value,
       });

       Ok(())
   }
   ```

1. 添加一个`allowance()`函数以返回允许`spender`从`owner`帐户提取的代币数量。

   ```rust
   #[ink(message)]
   pub fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance {
       self.allowances.get((owner, spender)).unwrap_or_default()
   }
   ```

### 添加从...转移逻辑

现在您已经为一个帐户设置了代表另一个帐户转移代币的批准，您需要创建一个`transfer_from`函数来启用已批准的用户转移代币。

`transfer_from`函数调用私有`transfer_from_to`函数来执行大部分转移逻辑。

授权非所有者转移代币有几个要求：

- `self.env().caller()`合约调用者必须分配可在`from`帐户中使用的代币。

- 存储为`allowance`的分配必须大于要转移的值。

如果满足这些要求，则合约会将更新的限额插入到`allowance`变量中，并使用指定的`from`和`to`帐户调用`transfer_from_to()`函数。

请记住，当调用`transfer_from`时，`self.env().caller()`和`from`帐户用于查找当前限额，但`transfer_from`函数是在指定的`from`和`to`帐户之间调用的。

每当调用`transfer_from`时，都会使用三个帐户变量，您需要确保正确使用它们。

要将`transfer_from`逻辑添加到智能合约：

1. 在文本编辑器中打开`lib.rs`文件。

1. 添加`transfer_from()`函数，以代表`from`帐户向`to`帐户转移`value`数量的代币。

   ```rust
   /// Transfers tokens on the behalf of the `from` account to the `to account
   #[ink(message)]
   pub fn transfer_from(
       &mut self,
       from: AccountId,
       to: AccountId,
       value: Balance,
   ) -> Result<()> {
       let caller = self.env().caller();
       let allowance = self.allowance(from, caller);
       if allowance < value {
           return Err(Error::InsufficientAllowance);
       }

       self.transfer_from_to(&from, &to, value)?;

       self.allowances.insert((from, caller), &(allowance - value));

       Ok(())
      }
   ```

1. 为`transfer_from()`函数添加测试。

   ```rust
   #[ink::test]
   fn transfer_from_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       let _ = contract.approve(alice(), 20);
       let _ = contract.transfer_from(alice(), bob(), 10);
       assert_eq!(contract.balance_of(bob()), 10);
   }
   ```

1. 为`allowance()`函数添加测试。

   ```rust
   #[ink::test]
   fn allowances_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       let _ = contract.approve(alice(), 200);
       assert_eq!(contract.allowance(alice(), alice()), 200);

       assert!(contract.transfer_from(alice(), bob(), 50).is_ok());
       assert_eq!(contract.balance_of(bob()), 50);
       assert_eq!(contract.allowance(alice(), alice()), 150);

       assert!(contract.transfer_from(alice(), bob(), 100).is_err());
       assert_eq!(contract.balance_of(bob()), 50);
       assert_eq!(contract.allowance(alice(), alice()), 150);
   }
   ```

1. 通过运行以下命令验证程序是否已编译并通过了所有测试：

   ```bash
   cargo test
   ```

   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 5 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok
   test erc20::tests::transfer_from_works ... ok
   test erc20::tests::allowances_works ... ok

   test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

1. 通过运行以下命令验证您是否可以构建合约的 WebAssembly：

   ```bash
   cargo contract build
   ```

   构建合约的 WebAssembly 后，您可以使用`cargo-contract`（如[上传和实例化合约](#upload-and-instantiate-the-contract)中所述）上传和实例化它。

## 编写测试用例

在本教程中，您向`lib.rs`文件添加了简单的单元测试。

基本测试用例通过指定输入值并验证返回的结果来说明函数是否按预期工作。

您可以通过编写其他测试用例来提高代码的质量。

例如，您可以添加测试来测试无效输入、空值或超出预期范围的值的错误处理。

## 下一步

在本教程中，您学习了如何使用 ink!编写简单的 ERC-20 代币智能合约，以便在 Substrate 区块链上运行。

例如，本教程说明了：

- 如何创建具有固定数量代币的合约。

- 如何将代币从合约所有者转移到其他帐户。

- 如何向智能合约添加测试。

- 如何启用第三方转移。

您可以在[智能合约](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-final.rs)的资产中找到本教程代码的示例。

您可以在以下主题中了解有关智能合约开发的更多信息：

- [使用映射存储值](/tutorials/smart-contracts/use-maps-for-storing-values/)

- [对智能合约进行故障排除](/tutorials/smart-contracts/troubleshoot-smart-contracts/)

- [ink!文档](https://paritytech.github.io/ink-docs/)
