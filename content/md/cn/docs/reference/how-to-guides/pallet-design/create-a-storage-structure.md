---
title: 创建存储结构（结构体）
description:
keywords:
  - 模块设计
  - 中级
  - 运行时
---

创建类似分组存储项的结构体是跟踪它们的有序方法。
以这种方式分组时，引用它们比单独保留各个 `StorageValue` 项更容易。
这可以使测试和创世配置更容易。

本指南向您展示如何创建一个 [`StorageValue`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageValue.html) 存储项，该项保存一个结构体并在 [`on_initialize`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Hooks.html#method.on_initialize) 中使用。
此结构体执行以下操作：

- 跟踪初始金额 (`issuance`)
- 跟踪接收该金额的帐户 (`minter`)
- 跟踪可以燃烧一定金额的帐户 (`burner`)
- 在 `on_initialize` 中（部分）使用

## 开始之前

确保您有一个可用于构建结构体的模块。
如果您没有正在使用的模块，请使用 [模板模块](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/pallets/template/src/lib.rs)。

1. 创建一个结构体

   将结构体命名为 `MetaData` 并声明其不同的类型：

   ```rust
   #[derive(Clone, Encode, Decode, Eq, PartialEq, RuntimeDebug, Default)]
   pub struct MetaData<AccountId, Balance> {
   	issuance: Balance,
   	minter: AccountId,
   	burner: AccountId,
   }
   ```

1. 将结构体声明为存储项

   使用 `StorageValue` 将结构体声明为存储中的新单个项：

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn meta_data)]
   	pub(super) type MetaDataStore<T: Config> = StorageValue<_, MetaData<T::AccountId, T::Balance>, ValueQuery>;
   ```

1. 配置 `GenesisConfig`

   使用 `#[pallet::genesis_config]` 属性从您的 `MetaData` 结构体初始化值。

   ```rust
   // 将 `admin` 声明为 `T::AccountId` 类型。
   #[pallet::genesis_config]
   pub struct GenesisConfig<T: Config> {
   	pub admin: T::AccountId,
   	}
   // 为其提供默认值。
   #[cfg(feature = "std")]
   impl<T: Config> Default for GenesisConfig<T> {
   	fn default() -> Self {
   		Self {
   			admin: Default::default(),
   			}
   		}
   	}
   ```

   此 `admin` 变量必须与 `node/chainspec.rs` 文件内 `fn testnet_genesis` 中使用的变量相对应。

1. 配置 `GenesisBuild`

   使用 `#[pallet::genesis_build]` 属性初始化结构体的值，使用 `admin` 初始化 `T::AccountId` 类型的值：

   ```rust
   #[pallet::genesis_build]
   impl<T: Config> GenesisBuild<T> for GenesisConfig<T> {
   	fn build(&self) {
   		MetaDataStore::<T>::put(MetaData {
   			issuance: Zero::zero(),
   			minter: self.admin.clone(),
   			burner: self.admin.clone(),
   		});
   	}
   }
   ```

1. 在 `on_initialize()` 中使用结构体

   为在启动链时初始化的 `MetaData` 的 `issuance` 字段分配一个金额：

   ```rust
   fn on_initialize(_n: T::BlockNumber) -> Weight {
   	// 为 StorageValue 结构体创建一个别名。
   	let mut meta = MetaDataStore::<T>::get();

   	// 向 `issuance` 字段添加一个值。
   	let value: T::Balance = 50u8.into();
   	meta.issuance = meta.issuance.saturating_add(value);

   	// 将金额添加到存储中的 `minter` 帐户。
   	Accounts::<T>::mutate(&meta.minter, |bal| {
   		*bal = bal.saturating_add(value);
   	});
   }
   ```

`on_initialize` 函数确保在启动链时将指定项的值写入存储。

## 示例

- [`reward-coin`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/d3602a66d66be5b013f2e3330081ea4e0d6dd978/example-code/template-node/pallets/reward-coin/src/lib.rs#L24-L29) 示例模块

## 资源

- [`Default::default()`](https://paritytech.github.io/substrate/master/sp_std/default/trait.Default.html)
