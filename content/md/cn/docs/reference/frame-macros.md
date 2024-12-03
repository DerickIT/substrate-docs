---
title: FRAME 宏
description:
keywords:
---
<div class="warning">
  请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/frame_support/attr.pallet.html">Rust 文档</a> 以获取有关此主题的最新文档。
</div>

Substrate 使用自定义的 [Rust 宏](https://doc.rust-lang.org/book/ch19-06-macros.html) 来生成代码并聚合您为运行时实现的 pallet 的逻辑。
这些运行时宏允许您专注于运行时逻辑，而不是花费时间在编码和解码链上变量或复制 [基本区块链开发](/learn/runtime-development#core-primitives) 所需的代码上。

本节概述了 Rust 中可用的宏类型，并重点介绍了在运行时开发中如何使用特定的 FRAME 宏。

## 宏基础知识

在计算机编程中，宏是封装预设指令序列以执行的代码行。
作为编写代码的代码，宏使您可以抽象出重复的操作并简化您需要编写的代码。
使用宏，您可以隐式声明复杂的数据结构。

在 Rust 中，宏可以是**声明性宏**或**过程宏**。

[声明性宏](https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming) 使您可以声明一个模式并将表达式的结果与该模式进行比较，然后根据是否匹配该模式来执行任何其他代码行。
声明性宏在 Rust 编程中被广泛使用。

[过程宏](https://doc.rust-lang.org/book/ch19-06-macros.html#procedural-macros-for-generating-code-from-attributes) 类似于函数。
与声明性宏中进行的模式匹配不同，过程宏将代码作为输入，对输入执行一些指令集，并生成代码作为输出。
有三种类型的过程宏：

- [自定义派生宏](https://doc.rust-lang.org/book/ch19-06-macros.html#how-to-write-a-custom-derive-macro) 使您可以为给定类型定义和重用特性的实现。
  `derive` 宏对于定义必须满足特定特性的自定义运行时类型的实现特别有用。
- [类似属性的宏](https://doc.rust-lang.org/book/ch19-06-macros.html#attribute-like-macros) 使您可以创建新的属性来生成代码。

- [类似函数的宏](https://doc.rust-lang.org/book/ch19-06-macros.html#function-like-macros) 使您可以定义像函数调用一样运行以生成代码的宏。

因为宏在编译器解释其包含的代码行之前就被展开，所以它们可以定义复杂的数据结构和操作。
FRAME 宏利用不同类型的宏来提供对通常是复杂代码块的快捷抽象。
但是，宏提供的抽象可能会使运行时代码难以理解。

要了解有关 Substrate 运行时中使用的 FRAME 宏的更多信息，您可以安装和使用 [cargo-expand](https://docs.rs/crate/cargo-expand)。
安装 cargo-expand 后，您可以使用 `cargo expand` 命令来显示运行时使用的宏中包含的代码的结果。

## FRAME 支持和系统宏

Substrate 原语和 FRAME 都依赖于一系列不同的宏。
本节概述了 FRAME 支持和 FRAME 系统库中提供的宏。
在大多数情况下，这些宏提供了其他 pallet 依赖的框架，您应该熟悉它们在运行时逻辑中的使用方法和位置。
概述之后，本节将描述您作为运行时开发人员最有可能使用的特定宏。

### FRAME 支持宏

`frame_support` crate 提供了运行时中使用的许多最重要的声明性、派生、类似属性和类似函数的宏。
您应该熟悉的来自 `frame_support` crate 的一些重要宏包括：

- `construct_runtime` 用于根据您已实现的 pallet 列表构建运行时。
- `match_types` 用于创建实现 `Contains` 特性的类型，其语法类似于 `matches!`。
- `parameter_types` 用于创建 `Get` 特性的新实现。

有关 `frame_support` crate 中宏的更多信息，请参阅 [宏](https://paritytech.github.io/substrate/master/frame_support/index.html#macros)、[派生宏](https://paritytech.github.io/substrate/master/frame_support/index.html#derives) 和 [属性宏](https://paritytech.github.io/substrate/master/frame_support/index.html#attributes) 的 Rust 文档。

### FRAME 系统宏

`frame_system` crate 使用宏来定义提供对核心数据类型和共享实用程序的访问的原语。
这些原语和关联宏构成了外部节点和运行时中许多节点操作的基础，并充当其他 pallet 与 Substrate 框架交互的基础层。

您应该熟悉的来自 `frame_system` crate 的一些重要原语和宏包括：

- [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html)

  - `map` 用于从数组初始化键值集合。
  - `RuntimeDebug` 用于调试运行时。

  有关 `sp_core` 类似函数的宏的更多信息，请参阅 [宏](https://paritytech.github.io/substrate/master/sp_core/index.html#macros)。

- [`sp_runtime`](https://paritytech.github.io/substrate/master/sp_runtime/index.html)

  - `bounded_btree_map` 用于根据给定的文字构建有界 `btree-map`。
  - `bounded_vec` 用于根据给定的文字构建有界 `vec`。
  - `impl_opaque_keys` 用于为描述的数据结构实现 `OpaqueKeys`。
  - `parameter_types` 用于创建 `Get` 特性的新实现。

  有关 `sp_runtime` 类似函数的宏的更多信息，请参阅 [宏](https://paritytech.github.io/substrate/master/sp_runtime/index.html#macros)。
  有关 `sp_runtime` 派生宏的信息，请参阅 [派生宏](https://paritytech.github.io/substrate/master/sp_runtime/index.html#derives)。

- [`sp_api`](https://paritytech.github.io/substrate/master/sp_api/index.html)

  - `decl_runtime_apis` 用于将指定特性声明为运行时 API。
  - `impl_runtime_apis` 用于将指定特性实现标记为运行时 API。

  有关 `sp_api` 类似函数的宏的更多信息，请参阅 [宏](https://paritytech.github.io/substrate/master/sp_api/index.html#macros)。

- [`sp_std`](https://paritytech.github.io/substrate/master/sp_std/index.html)

  - `if_std` 用于指示仅在启用 `std` 功能集时应运行的代码。
  - `map` 用于从数组初始化键值集合。
  - `vec` 用于创建包含参数的向量。

  有关 `sp_std` 类似函数的宏的更多信息，请参阅 [宏](https://paritytech.github.io/substrate/master/sp_std/index.html#macros)。

- [`sp_version`](https://paritytech.github.io/substrate/master/sp_version/index.html)

  - `create_apis_vec` 用于创建 API 声明的向量。
  - `create_runtime_str` 用于创建 `RuntimeString` 常量。
  - `runtime_version` 用作属性，它接受运行时的版本声明并生成具有等效内容的自定义 WebAssembly 部分。

您将在节点模板的运行时和节点 `Cargo.toml` 文件中看到这些 crate 中的许多作为依赖项列出。

## 用于组合 pallet 的宏

如 [构建自定义 pallet](/learn/runtime-development#building-custom-pallets) 中所述，大多数 FRAME pallet 使用一组通用的部分组成。

宏使构建每个部分更加模块化和可扩展。
本节描述可用的宏以及如何使用它们来构建自定义运行时。

### #[pallet]

`#[pallet]` 宏是声明 pallet 所必需的。
此 [属性宏](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html) 是 pallet 模块 (`mod pallet`) 的属性。
在 `pallet` 模块中，`#[pallet]` 宏充当其他 `#[pallet::*]` 宏的入口点，这些宏描述用于标识 pallet 需要特定项的属性。
例如，pallet 通常包含一组类型、函数和特性实现，这些类型、函数和特性实现由 `construct_runtime!` 宏聚合以构建运行时。

```rust
#[pallet]
pub mod pallet {
...
}
```

#### 开发模式

您可以将 `dev_mode` 指定为 `#[pallet]` 或 `#[frame_support::pallet]` 属性宏的参数，以启用 pallet 的开发模式。
例如，将 `#[pallet]` 替换为 `#[pallet(dev_mode)]` 或将 `#[frame_support::pallet]` 替换为 `#[frame_support::pallet(dev_mode)]` 以启用您正在使用的 pallet 的开发模式。

开发模式会放宽对生产 pallet 的一些限制和要求，以便在开发和测试周期中更容易迭代您的代码。
例如，如果您为 pallet 启用开发模式：

- 您不需要在每个 `#[pallet::call]` 声明上指定权重。
  默认情况下，开发模式会为未明确指定权重的调用分配权重零（0）。

- 您不需要在存储类型上实现 `MaxEncodedLen`。
  默认情况下，开发模式会将所有存储项标记为无界。

请注意，您只能将 `dev_mode` 参数添加到包含 pallet 模块的 `#[pallet]` 或 `#[frame_support::pallet]` 属性宏中。
您不能为任何 `#[pallet::*]` 属性宏指定此参数。

您永远不应在生产网络中部署启用了开发模式的 pallet。在生产运行时中部署 pallet 之前，请务必从 `#[pallet]` 声明中删除 `dev_mode` 参数，修复任何编译器错误，并在禁用开发模式的情况下完成测试。

#### 使用 pallet 模块

在模块内部，宏会解析具有属性 `#[pallet::*]` 的项。
一些 `#[pallet::*]` 属性是必需的，而另一些是可选的。

您可以通过使用这些 crate 的 `pallet_prelude` 自动导入来自 `frame_support` 和 `frame_system` crate 的系统级类型。
例如：

```rust
#[pallet]
pub mod pallet {
		use frame_support::pallet_prelude::*;
		use frame_system::pallet_prelude::*;
		...
}
```

`#[pallet]` 宏类似于派生宏，因为它通过读取输入来扩展 pallet 类型和特性实现。
在大多数情况下，宏不会修改任何输入。
但是，在一些特定情况下，与派生宏不同，此宏会修改其输入。

宏将在以下情况下修改输入：

- 如果将**泛型**替换为**类型**

  例如，如果 `pub struct Pallet<..>(_)` 中的项的内部类型在 `pallet::storage` 宏中被替换为实现 `StorageInstance` 特性的类型，则可能会发生这种情况。

- 如果更改了**函数或数据结构**

  例如，如果 `pallet::type_value` 宏将函数项更改为结构体和特性实现，则可能会发生这种情况。

- 如果用户没有提供**文档**

  例如，如果没有提供文档，则宏 `pallet::pallet` 会修改输入，以便在 `struct Pallet<T>(_);` 项上方添加文档。

### #[pallet::config]

`#[pallet::config]` 宏是定义 pallet 使用的泛型数据类型所必需的。

此宏提供作为 pallet 的系统级 [`Config` 特性](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html) 的一部分的常量。

此宏的 Config 特性必须定义为名为 `Config` 的常规特性定义，其中包含系统级 `frame_system::Config` 特性。
该定义可以包含其他顶级特性和 where 子句。
例如：

```rust
#[pallet::config]
pub trait Config: frame_system::Config + $optionally_some_other_supertraits
$optional_where_clause
{
...
}
```

要绕过 `frame_system::Config` 要求，您可以使用属性 `#[pallet::disable_frame_system_supertrait_check]`。
例如：

```rust
#[pallet::config]
#[pallet::disable_frame_system_supertrait_check]
pub trait Config: pallet_timestamp::Config {}
```

### #[pallet::constant]

`#[pallet::constant]` 宏为 [`#[pallet::config]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#config-trait-palletconfig-mandatory) 宏中的 `Config` 特性提供其所需的类型和属性，并生成关联的元数据。

此宏将有关 pallet 中使用的常量的信息添加到运行时元数据中，包括：

- 常量名称
- 关联类型的名称
- 常量值
- 为常量返回的 `Get::get()` 值

例如，您可以使用 `#[pallet::constant]` 将 `type MyGetParam` 添加到元数据中：

```rust
#[pallet::config]
pub trait Config: frame_system::Config {
	#[pallet::constant] // 将属性放入元数据中
	type MyGetParam: Get<u32>;
}
```

### #[pallet::extra_constants]

`#[pallet::extra_constants]` 宏使您可以向元数据添加常量。

例如，您可以声明一个返回生成值的函数。
然后，您可以使用 `#[pallet::extra_constants]` 宏将生成值的信[Response interrupted by user]
