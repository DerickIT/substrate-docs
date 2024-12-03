---
title: 自定义模块
description:
keywords:
---

构建自定义运行时的最常见方法是从 [现有模块](/reference/frame-pallets/) 开始。
例如，你可能开始构建一个特定于应用程序的抵押模块，该模块使用现有集体和余额模块中公开的类型，但包含应用程序及其抵押规则所需的自定义运行时逻辑。

虽然 [FRAME 模块](/reference/frame-pallets) 提供了对最常见模块的概述，但查找有关现有模块的最新信息最好的地方是使用命名约定 `pallet_*` 的 crate 的 [Rust API](/reference/rust-api/) 文档。

如果你找不到满足你需求的模块，则可以使用 FRAME 宏来构建自定义模块的脚手架。

## 模块宏和属性

FRAME 广泛使用 Rust 宏来封装复杂的代码块。
构建自定义模块最重要的宏是 [`pallet`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html) 宏。
`pallet` 宏定义了模块必须提供的核心属性集。
例如：

- `#[pallet::pallet]` 是一个强制性模块属性，它使你能够为模块定义一个结构 (struct)，以便它可以存储易于检索的数据。
- `#[pallet::config]` 是一个强制性模块属性，它使你能够为模块定义配置特性。

`pallet` 宏还定义了模块通常提供的核心属性集。
例如：

- `#[pallet::call]` 是使你能够为模块实现可调度函数调用的属性。
- `#[pallet::error]` 是使你能够生成可调度错误的属性。
- `#[pallet::event]` 是使你能够生成可调度事件的属性。
- `#[pallet::storage]` 是使你能够在运行时生成存储实例及其元数据的属性。

这些核心属性与你在编写自定义模块时需要做出的决策相一致。
例如，你需要考虑：

- 存储。你的模块存储什么数据？数据是存储在链上还是链下？
- 函数。你的模块公开哪些可调用函数？
- 事务性。你的函数调用是否旨在以原子方式修改存储？
- 挂钩。你的模块是否会调用任何运行时挂钩？

宏简化了你需要编写的用于实现自定义运行时逻辑的代码。
但是，某些宏会对函数声明强制执行特定要求。
例如，`Config` 特性必须由 `frame_system::Config` 绑定，并且 `#[pallet::pallet]` 结构必须声明为 `pub struct Pallet<T>(_);`。
有关在 FRAME 模块中使用的宏的概述，请参阅 [FRAME 宏](/reference/frame-macros/)。

<!-- ## 有用的 FRAME 特性

- 模块来源
- 来源：EnsureOrigin、EnsureOneOf
  ...

## 运行时实现

编写模块并为运行时实现它两者是相辅相成的。
你的模块的 `Config` 特性是在 `Runtime` 中实现的，`Runtime` 是一个特殊的结构，用于在 `construct_runtime` 宏中编译所有已实现的模块。

- [`parameter_types`](https://paritytech.github.io/substrate/master/frame_support/macro.parameter_types.html) 和 [`ord_parameter_types`](https://paritytech.github.io/substrate/master/frame_support/macro.ord_parameter_types.html) 宏对于将值传递给可配置的模块常量很有用。
- [其他注意事项，例如 no_std]
- 最小运行时引用
- 侧链架构引用
- API 端点：on_initialize、离链工作程序？

编写链接到基本和中级操作方法指南的内容。 -->
