---
title: 账户数据结构
description: 描述 FRAME 中用于账户的存储映射数据结构。
keywords:
---

账户是任何区块链的基本组成部分，在 Substrate 中，账户可以以许多不同的方式使用。
本文档描述了账户如何在 Substrate 中存储，以及如何使用账户数据结构在运行时逻辑中管理账户生命周期。

## 账户

`Account` 数据类型是一个存储映射，它在 [`frame-system` pallet](https://paritytech.github.io/substrate/master/src/frame_system/lib.rs.html#530) 中泛型定义：

```rust
/// 特定账户 ID 的完整账户信息。
#[pallet::storage]
#[pallet::getter(fn account)]
pub type Account<T: Config> = StorageMap<
  _,
  Blake2_128Concat,
  T::AccountId,
  AccountInfo<T::Nonce, T::AccountData>,
  ValueQuery,
>;
```

`Account` 的 `StorageMap` 包含以下参数：

- 第一个参数 (\_) 用于宏展开。
- `Blake2_128Concat` 指定要使用的哈希算法。
- `T::AccountId` 用作 `AccountInfo<T::Nonce, T::AccountData>` 结构体的键。

有关详细信息，请参阅 [`StorageMap` API](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageMap.html#impl)。

## AccountInfo

账户的 `AccountInfo` 在 [`frame-system` pallet](https://paritytech.github.io/substrate/master/src/frame_system/lib.rs.html#788-803) 中定义：

```rust
#[derive(Clone, Eq, PartialEq, Default, RuntimeDebug, Encode, Decode)]
pub struct AccountInfo<Nonce, AccountData> {
  /// 此帐户已发送的事务数量。
  pub nonce: Nonce,
  /// 当前依赖于此帐户存在的其他模块的数量。在该数量为零之前，无法清除此帐户。
  pub consumers: RefCount,
  /// 允许此帐户存在的其他模块的数量。在该数量和 `sufficients` 都为零之前，可能无法清除此帐户。
  pub providers: RefCount,
  /// 仅出于自身目的允许此帐户存在的模块的数量。在该数量和 `providers` 都为零之前，可能无法清除此帐户。
  pub sufficients: RefCount,
  /// 属于此帐户的附加数据。用于在许多链中存储余额。
  pub data: AccountData,
}
```

每个帐户都有一个 `AccountInfo`，它包含：

- `nonce`，指示帐户已发送的事务数量。
- `consumers` 引用计数器，指示当前依赖于此帐户存在的其他模块的数量。
- `providers` 引用计数器，指示允许此帐户存在的其他模块的数量。
- `sufficients` 引用计数器，指示仅出于自身目的允许此帐户存在的模块的数量。
- `AccountData` 结构体，您可以将其配置为保存不同类型的数据。

## 账户引用计数器

账户引用计数器跟踪运行时中的账户依赖项。
例如，如果您在由账户控制的映射下存储数据，则在删除账户控制的映射下存储的数据之前，您不希望删除该账户。

`consumers` 和 `providers` 引用计数器设计为一起使用。
例如，当帐户在成为验证者之前设置其会话密钥时，`Session` pallet 中的 `consumers` 引用计数器会递增。
在递增 `consumer` 计数器之前，`providers` 引用计数器必须大于零。

`providers` 引用计数器指示帐户是否已准备好被依赖。
例如，当使用超过生存存款 [[2]](#ref-system-created) 创建新帐户时，`providers` 引用计数器会递增。

`providers` 引用计数器可防止 Substrate pallet 在帐户处于活动状态之前（`providers` > 0）存储有关帐户的数据。
`consumers` 引用计数器可防止 Substrate pallet 在所有 pallet 中删除有关帐户的数据之前（`consumers` == 0）删除帐户。
`consumers` 引用计数器使用户对其在链上存储的数据负责。
如果用户想要删除其帐户并取回生存存款，则需要删除他们在所有链上 pallet 中存储的所有数据以递减 `consumers` 计数器。

pallet 还具有清理函数，用于递减 `providers` 引用计数器，以在 pallet 管理的范围内将帐户标记为已停用。
当帐户 `providers` 引用计数器为零且 `consumers` 为零时，所有链上 pallet 都认为该帐户已**停用**。

`sufficients` 引用计数器指示帐户是否自给自足，并且可以独立存在。
例如，在 Assets pallet 中，帐户可以拥有足够数量的某些资产，但没有拥有任何原生帐户余额。

运行时开发人员可以使用 `frame-system` pallet 公开的 `inc_consumers()`、`dec_consumers()`、`inc_providers()`、`dec_providers()`、`inc_sufficients()` 和 `dec_sufficients()` 方法来更新这些计数器。
某个计数器的每次递增调用都应伴随帐户生命周期中该计数器的相应递减调用。

还有三个查询函数可以简化对这些计数器的使用：

- `can_inc_consumer()` 用于检查帐户是否已准备好使用（`providers` > 0）。
- `can_dec_provider()` 用于检查帐户在递减 `providers` 到 0 之前是否在运行时中没有任何引用（`consumers` == 0）。
- `is_provider_required()` 用于检查帐户是否有未完成的消费者引用（`consumers` > 0）。

有关详细信息，请参阅 [`frame-system` API](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#impl-11)。

## AccountData 特性及其实现

只要结构体满足 [`frame-system::pallet::Config` 特性](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html#associatedtype.AccountData) 中定义的关联类型 `AccountData` 特性绑定，`AccountInfo` 就可以是任何结构体。
默认情况下，Substrate 运行时将 `AccountInfo` 配置为在 [`pallet-balances`](https://paritytech.github.io/substrate/master/pallet_balances/struct.AccountData.html) 中定义的那样。

## 接下来的步骤

有关其他技术细节，请参阅以下资源：

- [`frame_system::AccountInfo` API](https://paritytech.github.io/substrate/master/frame_system/struct.AccountInfo.html)
- [`pallet_balances::AccountData` API](https://paritytech.github.io/substrate/master/pallet_balances/struct.AccountData.html)。
- [`pallet_session::Pallet::set_keys` 可调度调用](https://paritytech.github.io/substrate/master/src/pallet_session/lib.rs.html#508-571)
- [`frame_system::Provider` `HandleLifetime` 实现](https://paritytech.github.io/substrate/master/src/frame_system/lib.rs.html#1549-1561)
- [`pallet_assets` `new_account` 函数](https://paritytech.github.io/substrate/master/src/pallet_assets/functions.rs.html#46-61)
