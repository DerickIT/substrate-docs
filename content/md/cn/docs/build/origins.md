---
title: 特权调用和来源
description:
keywords:
  - 来源
---

<div class="warning">
  请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_origin/index.html">Rust 文档</a> 获取有关此主题的最新文档。
</div>

运行时来源由可调度函数用于检查调用的来源。

## 原始来源

Substrate 定义了三种原始来源，可在运行时模块中使用：

```rust
pub enum RawOrigin<AccountId> {
	Root,
	Signed(AccountId),
	None,
}
```

- Root：系统级来源。这是最高的权限级别，可以认为是运行时来源的超级用户。

- Signed：事务来源。这由某个链上帐户的私钥签名，并包含签名者的帐户标识符。这允许运行时对调度的来源进行身份验证，然后将事务费用计入关联帐户。

- None：缺乏来源。这需要由验证者达成一致或由模块验证才能包含。
  此来源类型更复杂，因为它旨在绕过某些运行时机制。
  例如，此来源类型可用于允许验证者直接将数据插入块中。

## 来源调用

你可以在运行时中使用任何来源构建调用。例如：

```rust
// Root
proposal.dispatch(system::RawOrigin::Root.into())

// Signed
proposal.dispatch(system::RawOrigin::Signed(who).into())

// None
proposal.dispatch(system::RawOrigin::None.into())
```

你可以查看 [Sudo 模块](https://paritytech.github.io/substrate/master/pallet_sudo/index.html) 的源代码以了解此处的实际实现。

## 自定义来源

除了三种核心来源类型外，运行时开发者还可以定义自定义来源。
这些可以用作运行时中特定模块的函数内的授权检查，或用于定义围绕运行时请求来源的自定义访问控制逻辑。

自定义来源允许运行时开发者根据其运行时逻辑指定有效的来源。例如，可能需要将某些函数的访问权限限制为特殊的自定义来源，并且仅允许来自 [集体](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/collective) 成员的调度调用。使用自定义来源的优点是它为运行时开发者提供了一种方法来配置对运行时调度调用的特权访问。

## 下一步

### 了解更多

- 了解如何在 [`#[pallet::call]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#call-palletcall-optional) 宏中使用来源。

### 示例

- 查看 [Sudo 模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/sudo) 以了解它如何允许用户使用 `Root` 和 `Signed` 来源进行调用。

- 查看 [Timestamp 模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/timestamp) 以了解它如何验证使用 `None` 来源的调用。

- 查看 [Collective 模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/collective) 以了解它如何构建自定义 `Member` 来源。

- 查看我们创建和使用自定义来源的方案。

### 参考

- 访问 [`RawOrigin` 枚举](https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html) 的参考文档。
