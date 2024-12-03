---
title: 测试转移函数
description:
keywords:
  - 测试
  - 运行时
  - 初学者
---

测试每个函数是为生产环境开发 pallet 的重要部分。
本指南逐步介绍了为基本 `transfer` 函数编写测试用例的最佳实践。

## 概述 `transfer` 函数

转移函数有两个关键要素：从一个帐户中减去余额并将该余额添加到另一个帐户。
在这里，我们将首先概述此函数：

```rust
#[pallet::weight(10_000)]
pub (super) fn transfer(
  origin: OriginFor<T>,
  to: T::AccountId,
  #[pallet::compact] amount: T::Balance,
  ) -> DispatchResultWithPostInfo {
    let sender = ensure_signed(origin)?;

    Accounts::<T>::mutate(&sender, |bal| {
      *bal = bal.saturating_sub(amount);
      });
      Accounts::<T>::mutate(&to, |bal| {
        *bal = bal.saturating_add(amount);
        });

    Self::deposit_event(Event::<T>::Transferred(sender, to, amount))
    Ok(().into())
}
```

## 检查发送者是否有足够的余额

首先要验证的是发送者是否有足够的余额。
在单独的 `tests.rs` 文件中，编写此第一个测试用例：

```rust
#[test]
fn transfer_works() {
  new_test_ext().execute_with(|| {
    MetaDataStore::<Test>::put(MetaData {
			issuance: 0,
			minter: 1,
			burner: 1,
		});
    // 向帐户 2 铸造 42 个代币。
    assert_ok!(RewardCoin::mint(RuntimeOrigin::signed(1), 2, 42));
    // 向帐户 3 发送 50 个代币。
    asset_noop!(RewardCoin::transfer(RuntimeOrigin::signed(2), 3, 50), Error::<T>::InsufficientBalance);
```

### 配置错误处理

要实现一些错误检查，请将 `mutate` 替换为 `try_mutate` 以使用 `ensure!`。
这将检查_bal 是否大于或等于 amount_，如果否，则会抛出错误消息：

```rust
Accounts::<T>::try_mutate(&sender, |bal| {
  ensure!(bal >= amount, Error::<T>::InsufficientBalance);
  *bal = bal.saturating_sub(amount);
  Ok(())
});
```

从您的 pallet 目录运行 `cargo test`。

## 检查发送帐户是否未低于最低余额

在您的 `transfer_works` 函数中：

```rust
assert_noop!(RewardCoin::transfer(RuntimeOrigin::signed(2), 3, 50), Error::<Test>::InsufficientBalance);
```

## 检查两个测试是否一起工作

使用 `#[transactional]` 为两个检查生成一个包装器：

```rust
#[transactional]
pub(super) fn transfer(
/*--snip--*/
```

## 处理尘埃帐户

确保发送和接收帐户不是尘埃帐户。使用 `T::MinBalance::get()`：

```rust
/*--snip--*/
let new_bal = bal.checked_sub(&amount).ok_or(Error::<T>::InsufficientBalance)?;
ensure!(new_bal >= T::MinBalance::get(), Error::<T>::BelowMinBalance);
/*--snip--*/
```

## 示例

- [`reward-coin` 示例 pallet](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/reward-coin/src/tests.rs) 中的测试。

## 资源

- [`assert_ok!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_ok.html)
- [`assert_noop!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_noop.html)
- [`ensure!`](https://paritytech.github.io/substrate/master/frame_support/macro.ensure.html)
- [`try_mutate`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageMap.html#tymethod.try_mutate)
