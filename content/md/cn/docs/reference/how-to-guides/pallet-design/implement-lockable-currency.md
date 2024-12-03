---
title: 实现可锁定货币
description:
keywords:
  - 模块设计
  - 货币
  - 中级
  - 运行时
---

本指南向您展示如何编写一个允许用户锁定资金用于抵押和投票的模块。
[`LockableCurrency`](https://paritytech.github.io/polkadot-sdk/master/frame_support/traits/tokens/currency/trait.LockableCurrency.html) 特性在强制通过抵押易耗资源来执行问责制的经济系统中非常有用。
您可以使用 Substrate [抵押模块](https://paritytech.github.io/substrate/master/pallet_staking/index.html) 以基于时间增量的方式管理锁定的资金。

在本指南中，我们将在我们自己的自定义模块中实现 `set_lock`、`extend_lock` 和 `remove_lock` 方法。

## 开始之前

您需要在运行时中已经集成了一个模块才能遵循本指南。
本指南假设您正在使用包含 [Balances 模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/balances) 的运行时来处理链的帐户和余额。
您可以使用 [节点模板](https://github.com/paritytech/polkadot-sdk-solochain-template) 中的模板模块来遵循。

## 声明必要的依赖项

[`LockableCurrency`](https://paritytech.github.io/polkadot-sdk/master/frame_support/traits/tokens/currency/trait.LockableCurrency.html) 的方法要求我们从 `frame_support` 导入一些特性。

1. 确保在模块顶部的部分导入了以下特性：

   ```rust
   use frame_support::{
   	dispatch::DispatchResult,
   	traits::{Currency, LockIdentifier, LockableCurrency, WithdrawReasons},
   };
   ```

1. 声明 `LockIdentifier` 常量。

   要使用 `LockableCurrency`，您必须声明一个 `LockIdentifier`。
   鉴于它需要 `[u8; 8]` 类型，此标识符必须为八个字符长。

   ```rust
   const EXAMPLE_ID: LockIdentifier = *b"example ";
   ```

   您需要这个来声明我们稍后将包含的方法。

## 声明您的自定义模块类型

定义您的自定义配置类型将允许您的模块继承我们想要实现的方法。

1. 定义可锁定货币类型，我们将其称为 `StakeCurrency`：

   ```rust
   	type StakeCurrency: LockableCurrency<Self::AccountId, Moment = Self::BlockNumber>;
   ```

1. 定义一个类型来满足我们正在使用的 [`Currency`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html) 特性的关联类型 [`Balance`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#associatedtype.Balance)：

   ```rust
   	type BalanceOf<T> =
   		<<T as Config>::StakeCurrency as Currency<<T as frame_system::Config>::AccountId>>::Balance;
   ```

   这确保了我们的模块有一个类型来处理我们将在实现的方法中的 `amount` 字段。

1. 为您的运行时指定新类型：

   ```rust
   	impl pallet_template::Config for Runtime {
   		type RuntimeEvent = RuntimeEvent;
   		type StakeCurrency = Balances; // <- 添加此行
   	}
   ```

   传入 `Balances` 可确保模块的 `LockableCurrency` 方法与处理区块链帐户余额的模块对 `Balance` 的理解相同。

## 使用所需方法创建可调度函数

所需方法是：

- `fn lock_capital`: 锁定调用者指定的代币数量。
- `fn extend_lock`: 延长锁定期限。
- `fn unlock_all`: 释放所有锁定的代币。

1. 编写 `lock_capital`，使用 `T::StakeCurrency::set_lock`:

   ```rust
   	#[pallet::weight(10_000 + T::DbWeight::get().writes(1))]
   	pub(super) fn lock_capital(
   		origin: OriginFor<T>,
   		#[pallet::compact] amount: BalanceOf<T>
   	) -> DispatchResultWithPostInfo {

   		let user = ensure_signed(origin)?;

   		T::StakeCurrency::set_lock(
   			EXAMPLE_ID,
   			&user,
   			amount,
   			WithdrawReasons::all(),
   		);

   		Self::deposit_event(Event::Locked(user, amount));
   		Ok(().into())
   	}
   ```

1. 编写 `extend_lock`，使用 `T::StakeCurrency::extend_lock`:

   ```rust
   	#[pallet::weight(1_000)]
   	pub(super) fn extend_lock(
   		origin: OriginFor<T>,
   		#[pallet::compact] amount: BalanceOf<T>,
   	) -> DispatchResultWithPostInfo {
   		let user = ensure_signed(origin)?;

   		T::StakeCurrency::extend_lock(
   			EXAMPLE_ID,
   			&user,
   			amount,
   			WithdrawReasons::all(),
   		);

   		Self::deposit_event(Event::ExtendedLock(user, amount));
   		Ok(().into())
   	}
   ```

1. 编写 `unlock_all`，使用 `T::StakeCurrency::remove_lock`:

   ```rust
   	#[pallet::weight(1_000)]
   	pub(super) fn unlock_all(
   		origin: OriginFor<T>,
   	) -> DispatchResultWithPostInfo {
   		let user = ensure_signed(origin)?;

   		T::StakeCurrency::remove_lock(EXAMPLE_ID, &user);

   		Self::deposit_event(Event::Unlocked(user));
   		Ok(().into())
   	}
   ```

1. 为所有三个可调度函数编写事件：

   ```rust
   	#[pallet::event]
   	#[pallet::generate_deposit(pub(super) fn deposit_event)]
   	pub enum Event<T: Config> {
   		Locked(T::AccountId, BalanceOf<T>),
   		Unlocked(T::AccountId),
   		LockExtended(T::AccountId, BalanceOf<T>),
   	}
   ```

## 示例

- [`lockable-currency`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/lockable-currency/src/lib.rs) 示例模块

## 相关资料

- [Currency 特性](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html)
- [LockableCurrency](https://paritytech.github.io/polkadot-sdk/master/frame_support/traits/tokens/currency/trait.LockableCurrency.html)
- [LockIdentifier](https://paritytech.github.io/polkadot-sdk/master/frame_support/traits/tokens/currency/type.LockIdentifier.html)
