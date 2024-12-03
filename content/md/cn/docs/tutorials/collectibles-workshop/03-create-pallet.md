---
title: 创建新的模块
tutorial: 1
---

> ⚠️ 本教程已过期，可能无法按预期工作。请参考[`Polkadot SDK 文档`](https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/guides/your_first_pallet/index.html)以获取最新信息。

在本研讨会中，您将学习如何创建一个自定义 Substrate 模块（称为模块），该模块将包含特定于应用程序的区块链的代码。
模块是使用 FRAME 库和 Rust 编程语言构建的。
FRAME 包含许多专用宏，这些宏使您可以轻松地将应用程序逻辑组合到可重用的容器中。
让我们从创建一个新项目并为 Substrate 可收藏应用程序的关键部分准备基本框架开始。

## 创建新项目

由于本研讨会完全是关于演示创建新模块（新的自定义模块）的完整工作流程，因此我们不会从`pallet-template`开始。
相反，第一步是为将要构建的可收藏模块创建一个新的 Rust 包。

要创建项目：

1. 根据需要打开新的终端。

2. 更改到工作区中包含`workshop-solo-template-node/pallets`目录的目录。

3. 通过运行以下命令为`collectibles`模块创建一个新的 Rust 项目：

   ```bash
   cargo new collectibles
   ```

   请注意，`cargo`会警告您新包不在当前工作区中，并且有几种不同的方法可以解决此问题。
   目前，您可以忽略警告并继续准备新项目。

4. 通过运行以下命令更改到`collectibles`目录：

   ```bash
   cd collectibles
   ```

5. 查看`collectibles`目录的内容，并注意它提供了一个默认的`Cargo.toml`和`src/main.rs`程序：

   ```text
   ├── Cargo.toml
   └── src
    └── main.rs
   ```

   在 Rust 中，每个包的`Cargo.toml`文件称为包清单，它定义了包所需的配置设置和依赖项。
   `workshop-solo-template-node/pallets/collectibles`文件夹中的`Cargo.toml`文件定义了您正在构建的`collectibles`包的依赖项。

   按照约定，Substrate 项目（包括模块）的源代码通常位于`src/lib.rs`文件中。
   默认情况下，Cargo 为新项目创建模板`src/main.rs`文件。
   为了在研讨会中清晰起见，让我们重命名新模块的主源文件。

6. 通过运行以下命令重命名`src/main.rs`源文件：

   ```bash
   mv src/main.rs src/lib.rs
   ```

   现在您的项目文件已就位，让我们通过将新项目添加到当前工作区来解决`cargo`警告您的问题。

## 将模块添加到工作区

与其他 Rust 程序一样，节点模板也有一个`Cargo.toml`清单文件。
在这种情况下，清单文件描述了工作区成员。
要将新模块添加到工作区：

1. 更改到工作区中包含`workshop-solo-template-node`目录的目录。

1. 在代码编辑器中打开`Cargo.toml`文件。

1. 将新的`pallets/collectibles`模块添加为工作区的成员。

   ```toml
   [workspace]
   members = [
     "node",
     "pallets/collectibles",
     "pallets/template",
     "runtime",
   ]
   ```

1. 保存更改并关闭文件。

## 准备项目清单

您的新项目现在已包含在工作区中，并已准备好开始配置。
您已经知道该项目有一个默认的`Cargo.toml`清单文件，并且此文件描述了包属性和依赖项。
在使用 Substrate 和 FRAME 开发区块链时，定义包属性和依赖项尤其重要，因为模块化开发环境使您可以将代码的一部分功能导入到代码的另一部分，并且您可以利用这一点来重用常见功能。

在本研讨会中，`collectibles`模块将成为 Substrate 运行时的一部分，其`Cargo.toml`文件需要定义它依赖的一些模块。
例如，`collectibles`模块需要的两个核心包是`frame_system`和`frame_support`模块：

- `frame_system`提供处理常用数据结构和原语的核心功能，以便所有需要它们的模块都可以使用它们，从而使新的模块能够轻松集成到运行时并相互交互。
- `frame_support`提供核心支持服务，用于处理调度到运行时的函数调用、定义存储结构、准备事件和错误以及核心实用程序。

除了`frame_system`和`frame_support`之外，`collectibles`模块还需要包来支持最小化区块链网络流量所需的类型编码和解码。
为了支持 SCALE 格式的编码和解码，`collectibles`模块需要访问`codec`和`scale-info`包。

要更新可收藏项目的清单：

1. 在代码编辑器中打开`collectibles`模块的默认`Cargo.toml`文件。

2. 将`frame-support`和`frame-system`添加到依赖项中。

   ```toml
   [dependencies]
   frame-support = { default-features = false, version = "36.0.0" }
   frame-system = { default-features = false, version = "36.0.0" }
   ```

3. 将`codec`和`scale-info`添加到依赖项中。

   ```toml
   codec = { package = "parity-scale-codec", version = "3.0.0", default-features = false, features = ["derive"] }
   scale-info = { version = "2.1.1", default-features = false, features = ["derive"] }
   ```

1. 将这些包的标准功能添加到`Cargo.toml`文件中，以支持运行时的原生二进制目标。

   ```toml
   [features]
   default = ["std"]
   std = [
      "frame-support/std",
      "frame-system/std",
      "codec/std",
      "scale-info/std",
   ]
   ```

   目前，Substrate 运行时编译为跨平台 WebAssembly 目标和特定于平台的原生二进制目标。

## 准备通用代码部分

您现在拥有模块所需的最小包依赖项，这些依赖项已在`Cargo.toml`清单中指定。
下一步是准备一组通用宏，作为新模块的框架。

1. 在文本编辑器中打开`src/lib.rs`并删除模板`main()`函数。

   您现在拥有一个干净的框架，用于创建 Substrate 可收藏模块。

2. 通过向`src/lib.rs`文件添加以下通用宏声明集来准备 Substrate 可收藏模块的框架：

   ```rust
   #![cfg_attr(not(feature = "std"), no_std)]

   pub use pallet::*;

   #[frame_support::pallet(dev_mode)]
   pub mod pallet {
        use frame_support::pallet_prelude::*;
        use frame_system::pallet_prelude::*;

        #[pallet::pallet]
        pub struct Pallet<T>(_);

        #[pallet::config]
        pub trait Config: frame_system::Config {
        }
   }
   ```

3. 保存更改并关闭文件。

1. 通过运行以下命令验证程序是否已编译：

   ```bash
   cargo build --package collectibles
   ```

   目前，您可以忽略有关未使用代码的编译器警告。
