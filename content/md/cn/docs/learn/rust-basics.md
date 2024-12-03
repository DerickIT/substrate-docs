---
title: Substrate 的 Rust
description: 突出显示在开发 Substrate 区块链时特别重要的 Rust 编程约定。
keywords:
---

Substrate 能够成为一个灵活且可扩展的框架，用于创建关键任务软件，这在很大程度上归功于[Rust](https://www.rust-lang.org/)。
作为 Substrate 的首选语言，Rust 是一种高性能的编程语言，是以下原因的首选：

- Rust 速度很快：它在编译时进行静态类型检查，使编译器能够优化代码以提高速度，并使开发人员能够针对特定编译目标进行优化。

- Rust 可移植：它旨在运行在嵌入式设备上，并支持任何类型的操作系统。

- Rust 内存安全：它没有垃圾收集器，并且会检查您使用的每个变量和您引用的每个内存地址，以避免任何内存泄漏。

- Rust 是 Wasm 首选：它对编译到 WebAssembly 提供了一流的支持。

## Substrate 中的 Rust

在[架构](/learn/architecture/)部分，您将了解到 Substrate 由两个不同的架构组件组成：外部节点和运行时。
虽然外部节点代码中使用了 Rust 中更复杂的功能，例如多线程和异步 Rust，但它们不会直接暴露给运行时工程师，这使得运行时工程师更容易专注于其节点的业务逻辑。

通常，根据他们的关注点，开发人员应该期望了解：

- 基本的[Rust 习惯用法](https://rust-unofficial.github.io/patterns/idioms/index.html)，[使用 `no_std`](https://docs.rust-embedded.org/book/intro/no-std.html) 以及使用哪些宏以及原因（用于运行时工程）。
- [异步 Rust](https://rust-lang.github.io/async-book/01_getting_started/01_chapter.html)（对于使用外部节点（客户端）代码的更高级开发人员）。

虽然在深入研究 Substrate 之前，对 Rust 的一般熟悉程度至关重要，并且有许多资源可用于学习 Rust，包括[Rust 语言编程书籍](https://doc.rust-lang.org/book/) 和[Rust by Example](https://doc.rust-lang.org/rust-by-example/)，但本节的其余部分重点介绍了 Substrate 如何使用一些 Rust 的核心功能，以帮助开发人员开始进行运行时工程。

## 编译目标

在构建 Substrate 节点时，我们使用 `wasm32-unknown-unknown` 编译目标，这意味着 Substrate 运行时工程师被限制为编写必须编译到 Wasm 的运行时。
这意味着您不能依赖一些典型的标准库类型和函数，并且必须仅对大多数运行时代码使用与 `no_std` 兼容的板条箱。
Substrate 有许多自己的原始类型和关联的特性，使您可以绕过 `no_std` 要求。

## 宏

在学习如何使用和编写 FRAME 模块时，您会看到有许多宏可用作可重用代码，以抽象出常见任务或强制执行运行时特定的要求。
这些宏使您可以专注于编写惯用的 Rust 和特定于应用程序的逻辑，而不是与运行时交互所需的通用代码。

Rust 宏是帮助确保满足某些要求（无需重新编写代码）的强大工具，例如以特定方式格式化逻辑、进行特定检查或某些逻辑包含特定数据结构。
这对于帮助开发人员编写能够与 Substrate 运行时的复杂性集成的代码特别有用。
例如，`#[frame_support::pallet]` 宏在所有 FRAME 模块中都是必需的，以帮助您正确实现某些必需的属性（例如存储项或外部可调用函数），并使其与 `construct_runtime` 中的构建过程兼容。

开发 Substrate 运行时涉及大量使用 Rust 的属性宏，它们有两种形式：派生属性和自定义属性。
在开始使用 Substrate 时，了解它们的具体工作原理并不重要，而是要了解它们的存在，以及它们如何使您能够编写正确的运行时代码。

派生属性对于需要满足某些特性的自定义运行时类型很有用，例如，使类型在运行时执行期间可由节点解码。

其他属性宏在 Substrate 的代码库中也很常见，用于：

- 指定代码片段是否仅编译到 `no_std` 或可以使用 `std` 库。
- 构建自定义 FRAME 模块。
- 指定运行时的构建方式。

## 泛型和配置特性

特性在 Rust 中通常与 Java 等语言中的接口相比较，它们提供了为类型提供高级功能的方法。

如果您已经阅读过有关模块的信息，您可能已经注意到每个模块都有一个 `Config` 特性，它允许您定义模块所依赖的类型和接口。

该特性本身从 `frame_system::pallet::Config` 特性继承了许多核心运行时类型，这使得在编写运行时逻辑时很容易访问通用类型。
此外，在任何 FRAME 模块中，`Config` 特性都是对 `T` 进行泛型的（有关泛型的更多信息，请参阅下一节）。
这些核心运行时类型的常见示例可能是 `T::AccountId`，用于在运行时识别用户帐户的通用类型，或者 `T::BlockNumber`，运行时使用的区块编号类型。

有关 Rust 中泛型类型和特性的更多信息，请参阅 Rust 书籍中有关[泛型类型](https://doc.rust-lang.org/book/ch10-01-syntax.html)、[特性](https://doc.rust-lang.org/book/ch10-02-traits.html) 和[高级特性](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html) 的部分。

使用 Rust 泛型，Substrate 运行时开发人员可以编写完全独立于特定实现细节的模块，因此可以充分利用 Substrate 的灵活性和可扩展性。

`Config` 特性中的所有类型都是使用特性边界泛型定义的，并在运行时实现中具体化。
这不仅意味着您可以编写支持相同类型的不同规范的模块（例如，Substrate 和以太坊链的地址），而且您还可以使用最少的开销（例如，将区块编号更改为 `u32`）根据您的需要自定义泛型实现。

这使开发人员能够编写不依赖于您所做出的核心区块链架构决策的代码。

Substrate 最大限度地利用泛型类型来提供最大的灵活性。
您定义如何解析泛型类型以适合您的目的。

有关 Rust 中泛型类型和特性的更多信息，请参阅 Rust 书籍中有关[泛型类型](https://doc.rust-lang.org/book/ch10-01-syntax.html) 的部分。

## 下一步去哪里

现在您已经了解了 Substrate 如何依赖一些关键的 Rust 功能（例如特性、泛型类型和宏），您可以探索以下资源以了解更多信息。

- [Rust 书籍](https://doc.rust-lang.org/book/)
- [为什么选择 Rust？](https://www.parity.io/blog/why-rust)（Parity 的博客）
- [Cargo 和 crates.io](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html)
- [为什么选择 Rust 来编写智能合约？](https://paritytech.github.io/ink-docs/why-rust-for-smart-contracts)（ink! 文档）
