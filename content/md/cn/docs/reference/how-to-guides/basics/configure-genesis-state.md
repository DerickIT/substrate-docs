---
title: 配置创世状态
description: 创世配置定义了存储项的初始状态，例如帐户、余额、自定义模块的创世等。
keywords:
  - 基础
  - 初学者
  - 运行时
---

创世配置定义了存储项的初始状态，例如帐户、余额、自定义模块的创世等。
本指南演示如何为具有以下存储项的模块配置创世状态：

- `SingleValue<T>` 类型，用于单个 `StorageValue` 存储项。
- `AccountMap<T>` 类型，用于简单的单键 `StorageMap` 存储项。 

## 步骤预览

1. 在模块中添加存储项。
2. 在模块中添加创世配置宏。
3. 在链规范中设置初始值。

## 在模块中添加存储项

1. 在文本编辑器中打开模块的 `src//lib.rs` 文件。
   
1. 添加 `StorageValue` 存储项。
   
   例如：
   
   ```rust
   #[pallet::storage]
   #[pallet::getter(fn something)]
   pub type SingleValue<T: Config> = StorageValue<
      _, 
      T::Balance
   >;
   
1. 添加 `StorageMap` 存储项，用于具有可枚举条目的映射。
   
      例如：
   
   ```rust
   #[pallet::storage]
   #[pallet::getter(fn accounts)]
   pub type AccountMap<T: Config> = StorageMap<
      _, 
      Blake2_128Concat, 
      T::AccountId, 
      T::Balance
   >;
   ```

## 添加创世配置宏

`GenesisConfig` 代码应放在存储项之后。

1. 添加 `#[pallet::genesis_config]` 属性宏并定义 `GenesisConfig` 结构，用于初始化要初始化的存储项。

   ```rust
   #[pallet::genesis_config]
   pub struct GenesisConfig<T: Config> {
   	pub single_value: T::Balance,
   	pub account_map: Vec<(T::AccountId, T::Balance)>,
   }
   ```

2. 设置 `GenesisConfig` 结构的默认值。

   ```rust
   #[cfg(feature = "std")]
   impl<T: Config> Default for GenesisConfig<T> {
   	fn default() -> Self {
   		Self { single_value: Default::default(), account_map: Default::default() }
   	}
   }
   ```

3. 添加 `#[pallet::genesis_build]` 属性宏并实现 [`GenesisBuild`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.GenesisBuild.html) 特性。

   ```rust
   #[pallet::genesis_build]
   impl<T: Config> GenesisBuild<T> for GenesisConfig<T> {
   	fn build(&self) {
   		<SingleValue<T>>::put(&self.single_value);
   		for (a, b) in &self.account_map {
   			<AccountMap<T>>::insert(a, b);
   		}
   	}
   }
   ```

   `#[pallet::genesis_build]` 宏允许您执行一些逻辑来定义 `GenesisConfig` 结构如何将某些内容放入存储中。

## 设置初始值

在模块中配置存储项和创世配置后，您可以在链的创世状态中指定要为存储项设置的值。
在本例中，假设运行时的 `construct_runtime!` 宏将 `PalletSomething` 作为模块名称，将 `pallet_something` 作为模块的路径。

```rust
construct_runtime!(
   pub struct Runtime
   where
      Block = Block,
      NodeBlock = opaque::Block,
      UncheckedExtrinsic = UncheckedExtrinsic,
   {
      PalletSomething: pallet_something,
   }
)
```

1. 在文本编辑器中打开 `node/chain_spec.rs` 文件。
   
2. 验证 `use node_template_runtime::BalanceConfig;` 是否在 `chain_spec.rs` 文件的顶部导入。

3. 创建一个类型为 `T::Balance` 的常量值，该值将存储在 `<SingleValue<T>>` 中（在 `testnet_genesis` 方法内）。

   ```rust
   const VALUE: Balance = 235813;
   ```

4. 创建一个帐户向量，用于在 `testnet_genesis` 方法内使用 `<AccountMap<T>>` 初始化。

   ```rust
   let accounts_to_map: Vec<AccountId> =
   	vec![
   		get_account_id_from_seed::<sr25519::Public>("Alice"),
   		get_account_id_from_seed::<sr25519::Public>("Bob"),
   		get_account_id_from_seed::<sr25519::Public>("Charlie"),
   	];
   ```

5. 将模块添加到 `testnet_genesis` 函数的 `GenesisConfig` 子句中。

   惯例是在 `runtime/src/lib.rs` 中的 `construct_runtime!` 内使用模块名称的小写拼写。
   例如，对于声明为 `CamelCase` 的模块，惯例是在 `testnet_genesis` 函数中将其称为 `camel_case`。

   对于此示例模块，代码如下所示：

   ```rust
   pallet_something: PalletSomethingConfig {
   	single_value: VALUE,
   	account_map: accounts_to_map.iter().cloned().map(|x| (x, VALUE)).collect(),
   }
   ```

   此示例代码将 `accounts_to_map` 中的每个帐户映射到等于 `VALUE` 的金额。
   此模式与 Balances 模块的 `GenesisConfig` 非常相似。

## 示例

- [节点模板 'chain_spec.rs'](https://github.com/substrate-developer-hub/substrate-node-template/blob/master/node/src/chain_spec.rs)
- [示例模块](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/lib.rs)
