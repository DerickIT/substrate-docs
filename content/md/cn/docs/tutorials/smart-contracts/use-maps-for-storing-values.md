---
title: 使用映射存储值
description: 使用映射将数据存储为键值对。
keywords:
---

在[开发智能合约](/tutorials/smart-contracts/develop-a-smart-contract/)中，您开发了一个用于存储和检索单个数值的智能合约。

本教程说明了如何扩展智能合约的功能以管理每个用户的单个数字。
要添加此功能，您将使用[`Mapping`](https://docs.rs/ink/4.0.0-beta.1/ink/storage/struct.Mapping.html)类型。

ink!语言提供 Mapping 类型，使您可以将数据存储为键值对。例如，以下代码说明了如何将用户映射到数字：

```rust
// Import the `Mapping` type
use ink::storage::Mapping;

#[ink(storage)]
pub struct MyContract {
  // Store a mapping from AccountIds to a u32
  my_map: Mapping<AccountId, u32>,
}
```

使用`Mapping`数据类型，您可以为每个键存储存储值的唯一实例。

在本教程中，每个`AccountId`都表示一个键，该键映射到一个且只有一个存储的数值`my_map`。

每个用户只能存储、递增和检索与其自身`AccountId`关联的值。

## 初始化`Mapping`

第一步是初始化`AccountId`和存储值之间的映射。

- 指定映射键及其映射到的值。

以下示例说明了如何初始化`Mapping`并检索值：

```rust
#![cfg_attr(not(feature = "std"), no_std)]

#[ink::contract]
mod mycontract {
    use ink::storage::Mapping;

    #[ink(storage)]
    pub struct MyContract {
        // Store a mapping from AccountIds to a u32
        my_map: Mapping<AccountId, u32>,
    }

    impl MyContract {
        #[ink(constructor)]
        pub fn new(count: u32) -> Self {
            let mut my_map = Mapping::default();
            let caller = Self::env().caller();
            my_map.insert(&caller, &count);

            Self { my_map }
        }

        // Get the number associated with the caller's AccountId, if it exists
        #[ink(message)]
        pub fn get(&self) -> u32 {
            let caller = Self::env().caller();
            self.my_map.get(&caller).unwrap_or_default()
        }
    }
}
```

### 确定合约调用者

在前面的示例中，您可能已经注意到了`Self::env().caller()`函数调用。

此函数在整个合约逻辑中都可用，并且始终返回**合约调用者**。

重要的是要注意，合约调用者与**来源调用者**不同。

如果用户访问一个合约，然后调用后续合约，则第二个合约中的`Self::env().caller()`是第一个合约的地址，而不是原始用户。

### 使用合约调用者

在许多情况下，拥有可用的合约调用者都很有用。

例如，您可以使用`Self::env().caller()`创建一个访问控制层，该层仅允许用户访问他们自己的值。

您还可以使用`Self::env().caller()`在合约部署期间保存合约所有者。

例如：

```rust
#![cfg_attr(not(feature = "std"), no_std)]

#[ink::contract]
mod my_contract {

    #[ink(storage)]
    pub struct MyContract {
        // Store a contract owner
        owner: AccountId,
    }

    impl MyContract {
        #[ink(constructor)]
        pub fn new() -> Self {
            Self {
                owner: Self::env().caller(),
            }
        }
        /* --snip-- */
    }
}
```

因为您已使用`owner`标识符保存了合约调用者，所以您可以稍后编写函数来检查当前合约调用者是否是合约的所有者。

## 向智能合约添加映射

您现在可以向`incrementer`合约引入存储映射了。

要向`incrementer`合约添加存储映射：

1. 根据需要在您的计算机上打开终端 shell。

1. 验证您是否位于`incrementer`项目文件夹中。

1. 在文本编辑器中打开`lib.rs`文件。

1. 导入`Mapping`类型。

   ```rust
   #[ink::contract
   mod incrementer {
       use ink::storage::Mapping;
   ```

1. 将映射键从`AccountId`添加到存储为`my_map`的`i32`数据类型。

   ```rust
   pub struct Incrementer {
       value: i32,
       my_map: Mapping<AccountId, i32>,
   }
   ```

1. 在`new`构造函数中创建一个新的`Mapping`，并使用它来初始化您的合约。

   ```rust
   #[ink(constructor)]
   pub fn new(init_value: i32) -> Self {
       let mut my_map = Mapping::default();
       let caller = Self::env().caller();
       my_map.insert(&caller, &0);

       Self {
           value: init_value,
           my_map,
       }
   }
   ```

1. 在`default`构造函数中添加新的默认`Mapping`以及已定义的默认`value`。

    ```rust
    #[ink(constructor)]
    pub fn default() -> Self {
    Self {
            value: 0,
            my_map: Mapping::default(),
        }
    }
    ```

1. 添加一个`get_mine()`函数，以使用 Mapping API 的`get()`方法读取`my_map`并返回合约调用者的`my_map`。

   ```rust
   #[ink(message)]
   pub fn get_mine(&self) -> i32 {
        let caller = self.env().caller();
        self.my_map.get(&caller).unwrap_or_default()
   }
   ```

1. 添加新的测试以初始化帐户。

   ```rust
    #[ink::test]
   fn my_map_works() {
       let contract = Incrementer::new(11);
       assert_eq!(contract.get(), 11);
       assert_eq!(contract.get_mine(), 0);
   }
   ```

1. 保存更改并关闭文件。

1. 使用`test`子命令和`nightly`工具链通过运行以下命令来测试您的工作：

   ```bash
   cargo test
   ```

   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 3 tests
   test incrementer::tests::default_works ... ok
   test incrementer::tests::it_works ... ok
   test incrementer::tests::my_map_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

## 插入、更新或删除值

`Incrementer`合约的最后一步是允许用户更新他们自己的值。

您可以使用对 Mapping API 的调用在智能合约中提供此功能。

`Mapping`提供对存储项的直接访问。

例如，您可以通过使用现有键调用`Mapping::insert()`来替换存储项先前保存的值。

您还可以通过首先使用`Mapping::get()`从存储中读取它们，然后使用`Mapping::insert()`更新值来更新值。

如果给定键处没有现有值，则`Mapping::get()`返回`None`。

由于 Mapping API 提供对存储的直接访问，因此您可以使用`Mapping::remove()`方法从存储中删除给定键处的值。

要向合约添加插入和删除函数：

1. 根据需要在您的计算机上打开终端 shell。

1. 验证您是否位于`incrementer`项目文件夹中。

1. 在文本编辑器中打开`lib.rs`文件。

1. 添加一个`inc_mine()`函数，允许合约调用者获取`my_map`存储项并将递增的`value`插入到映射中。

   ```rust
   #[ink(message)]
   pub fn inc_mine(&mut self, by: i32) {
       let caller = self.env().caller();
       let my_value = self.get_mine();
       self.my_map.insert(caller, &(my_value + by));
   }
   ```

1. 添加一个`remove_mine()`函数，允许合约调用者从存储中清除`my_map`存储项。

   ```rust
   #[ink(message)]
   pub fn remove_mine(&self) {
       let caller = self.env().caller();
       self.my_map.remove(&caller)
   }
   ```

1. 添加新的测试以验证`inc_mine()`函数是否按预期工作。

   ```rust
   #[ink::test]
   fn inc_mine_works() {
       let mut contract = Incrementer::new(11);
       assert_eq!(contract.get_mine(), 0);
       contract.inc_mine(5);
       assert_eq!(contract.get_mine(), 5);
       contract.inc_mine(5);
       assert_eq!(contract.get_mine(), 10);
   }
   ```

1. 添加新的测试以验证`remove_mine()`函数是否按预期工作。

   ```rust
   #[ink::test]
   fn remove_mine_works() {
       let mut contract = Incrementer::new(11);
       assert_eq!(contract.get_mine(), 0);
       contract.inc_mine(5);
       assert_eq!(contract.get_mine(), 5);
       contract.remove_mine();
       assert_eq!(contract.get_mine(), 0);
   }
   ```

1. 使用`test`子命令检查您的工作：

   ```bash
   cargo test
   ```

   该命令应显示类似于以下内容的输出，以指示测试已成功完成：

   ```text
   running 5 tests
   test incrementer::tests::default_works ... ok
   test incrementer::tests::it_works ... ok
   test incrementer::tests::remove_mine_works ... ok
   test incrementer::tests::inc_mine_works ... ok
   test incrementer::tests::my_map_works ... ok

   test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

## 下一步

在本教程中，您学习了如何在智能合约中使用`ink::storage::Mapping`类型和 Mapping API。例如，本教程说明了：

- 如何初始化用于存储键值对的映射。

- 如何在智能合约中识别和使用合约调用者。

- 如何添加函数，使用户能够使用智能合约在映射中插入和删除为他们存储的值。

您可以在[智能合约](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/incrementer-mapping.rs)的资产中找到本教程最终代码的示例。

您可以在以下主题中了解有关智能合约开发的更多信息：

- [构建 ERC20 代币合约](/tutorials/smart-contracts/build-a-token-contract/)
- [对智能合约进行故障排除](/tutorials/smart-contracts/troubleshoot-smart-contracts/)
