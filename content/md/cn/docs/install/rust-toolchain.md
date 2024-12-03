---
title: Rust 工具链
description:
keywords:
---

<!-- TODO Manage expectations about build times on lower-spec hardware. Define that practical dev environment requirements are higher than just for running a node  -->

Rust 是一种现代的、类型安全的、高性能的编程语言，它为构建复杂系统提供了丰富的功能集。
该语言还拥有活跃的开发人员社区和不断增长的可共享库（称为 **板条箱**）生态系统。

## 学习 Rust

Rust 是用于构建基于 Substrate 的区块链的核心语言，因此如果您打算进行 Substrate 开发，则需要熟悉 Rust 编程语言、编译器和工具链管理。

如果您刚开始使用 Rust，则应将 [Rust 编程语言](https://doc.rust-lang.org/book/) 添加到书签中，并参考 Rust 网站上的其他 [学习 Rust](https://www.rust-lang.org/learn) 资源来指导您。
但是，在准备开发环境时，有一些重要事项需要注意。

## 关于 Rust 工具链

Rust **工具链** 中的核心工具是 `rustc` 编译器、`cargo` 构建和包管理器以及 `rustup` 工具链管理器。
在任何给定时间，可能有多个版本的 Rust 可用。
例如，存在稳定版、测试版和 nightly 构建的发布通道。
您可以使用 `rustup` 程序来管理环境中可用的构建以及与不同 Rust 构建一起使用的工具链程序的版本。

`rustc` 编译器使您能够为不同的架构构建二进制文件，称为 **目标**。
目标由一个字符串标识，该字符串指定编译器应生成的输出类型。
此功能很重要，因为 Substrate 编译为本机 Rust 二进制文件和 WebAssembly 目标。

WebAssembly 是一种可移植的二进制格式，可以在任何现代计算机硬件上执行，并且可以通过访问互联网的任何浏览器执行。
WebAssembly (Wasm) 目标使 Substrate 能够生成可移植的区块链运行时。
有关这些二进制文件如何使用的更多信息，请参阅 [构建过程](/build/build-process/)。
