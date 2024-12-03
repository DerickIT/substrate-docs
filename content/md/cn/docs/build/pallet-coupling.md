---
title: 模块耦合
description:
keywords:
  - 耦合
  - 模块设计
---

<div class="warning">
	 请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_pallet_coupling/index.html">Rust 文档</a> 获取有关此主题的最新文档。
</div>

术语**耦合**通常用于描述两个软件模块相互依赖的程度。
例如，在面向对象编程中，紧耦合和松耦合用于描述对象类之间的关系：

- 紧耦合是当两组类相互依赖时。
- 松耦合是当一个类使用另一个类公开的接口时。

在 Substrate 中，**紧模块耦合**和**松模块耦合**用于描述模块如何调用另一个模块中的函数。
这两种技术都以不同的方式实现了相同的功能，每种技术都有其自身的权衡。

## 紧密耦合的模块

由于紧耦合使使用模块的工作变得不那么灵活和可扩展，因此只有当模块需要_整体_继承其耦合的对应模块而不是特定类型或方法时，你才会使用紧密耦合。

编写需要紧密耦合的模块时，你明确指定模块的 `Config` 特性要由要与其耦合的模块的 `Config` 特性绑定。

所有 FRAME 模块都与 `frame_system` 模块紧密耦合。
以下示例说明了如何使用名为 `some_pallet` 的模块的 `Config` 特性与 `frame_system` 模块紧密耦合：

```rust
pub trait Config: frame_system::Config + some_pallet::Config {
    // --snip--
}
```

这与在面向对象编程中使用类继承非常相似。
提供此特性绑定意味着此模块只能安装在也安装了 `some_pallet` 模块的运行时中。
与 `frame_system` 类似，此示例中的紧密耦合将要求你在耦合模块的 **Cargo.toml** 文件中指定 `some_pallet`。

紧密耦合有几个缺点，开发者应该考虑：

- **可维护性**: 一个模块中的更改通常会导致需要修改另一个模块。
- **可重用性**: 必须同时包含这两个模块才能使用其中一个，这使得集成紧密耦合的模块变得更加困难。

## 松散耦合的模块

在松散模块耦合中，你可以指定某些类型需要绑定的特性和函数接口。

此类类型的实际实现发生在**模块外部**，在运行时配置期间（通常是在 `/runtime/src/lib.rs` 文件中定义的代码）。使用松散耦合，你可以使用已实现特性的另一个模块中的类型和接口，或者你可以声明一个全新的结构，实现这些特性，并在运行时实现模块时分配它。

例如，假设你有一个可以访问帐户余额并将资金转移到另一个帐户的模块。
此模块定义了一个 `Currency` 特性，该特性具有一个**抽象函数接口**，稍后将实现实际的转移逻辑。

```rust
pub trait Currency<AccountId> {
    // -- snip --
    fn transfer(
        source: &AccountId,
        dest: &AccountId,
        value: Self::Balance,
        // 暂时不用担心最后一个参数
        existence_requirement: ExistenceRequirement,
    ) -> DispatchResult;
}
```

在第二个模块中，你定义 `MyCurrency` 关联类型并通过
`Currency<Self::AccountId>` 特性绑定它，以便你可以通过调用 `T::MyCurrency::transfer(...)` 来使用余额转移逻辑。

```rust
pub trait Config: frame_system::Config {
    type MyCurrency: Currency<Self::AccountId>;
}

impl<T: Config> Pallet<T> {
    pub fn my_function() {
        T::MyCurrency::transfer(&buyer, &seller, price, ExistenceRequirement::KeepAlive)?;
    }
}
```

请注意，此时你尚未指定如何实现 `Currency::transfer()` 逻辑。
只是约定它将在某个地方实现。

现在，你可以使用运行时配置（`runtime/src/lib.rs`）来实现模块并指定类型为 `Balances`。

```rust
impl my_pallet::Config for Runtime {
    type MyCurrency = Balances;
}
```

`Balances` 类型在 `construct_runtime!` 宏中指定为实现 [`Currency` 特性](https://paritytech.github.io/substrate/master/pallet_balances/index.html#implementations-1) 的 [`pallet_balances`](https://paritytech.github.io/substrate/master/pallet_balances/index.html) 的一部分。

使用运行时提供的实现，你可以在松散耦合的模块中使用 `Currency<AccountId>` 特性。

许多 FRAME 模块都以这种方式与 [Currency 特性](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html) 耦合。

## 选择模块耦合策略

通常，松散耦合比紧密耦合提供了更大的灵活性，并且从系统设计的角度来看被认为是一种更好的实践。
它保证了代码的更好可维护性、可重用性和可扩展性。
但是，对于方法和类型之间的重叠多于差异的不太复杂的模块，紧密耦合可能很有用。

在 FRAME 中，有两个模块与 [`pallet_treasury`](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/treasury) 紧密耦合：

- [赏金模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/bounties)
- [小费模块](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/tips)

一般来说，模块越复杂，紧密耦合就越不可取。
这在计算机科学中引发了一个名为 [内聚性](<https://en.wikipedia.org/wiki/Cohesion_(computer_science)>) 的概念，这是一个用于检查软件系统整体质量的指标。

## 接下来的步骤

- [操作方法：使用松散耦合](/reference/how-to-guides/pallet-design/use-loose-coupling/)
- [操作方法：使用紧密耦合](/reference/how-to-guides/pallet-design/use-tight-coupling/)
