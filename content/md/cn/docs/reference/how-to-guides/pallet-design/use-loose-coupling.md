---
title: 使用松散模块耦合
description:
keywords:
---

<div class="warning">
	 请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_pallet_coupling/index.html">Rust 文档</a> 获取有关此主题的最新文档。
</div>

本指南演示了如何使用称为 **松散耦合** 的技术在一个模块中重用另一个模块中的函数或类型。

松散耦合使您能够在当前模块中重用在外部模块中定义的部分逻辑。
本指南通过在模块的配置特性中使用特性边界来重用在外部模块中声明的类型，从而说明了松散耦合。
在此示例中，当前模块使用来自 [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html) 模块的 `Currency` 特性的方法。

## 步骤预览

1. 配置工作区清单以包含外部模块。
2. 导入您想要使用的特性。
3. 向配置特性添加一个类型。
4. 在运行时提供实现。

## 配置您的工作区清单

为了使工作目录中的模块能够重用另一个模块中的代码，必须在为模块导入的包依赖项列表中包含外部模块。
因此，项目 `Cargo.toml` 文件中的清单必须指定您要导入的模块。
在此示例中，您正在重用 `frame-support` 模块中的 `Currency` 特性信息，因此您必须确保在清单的 `dependencies` 和 `features` 部分中包含 `frame-support`。

要配置您的工作区清单：

1. 在您的计算机上打开一个终端 shell 并导航到项目的根目录。

2. 在文本编辑器中打开清单 `Cargo.toml` 文件。

3. 将您要与其松散耦合的模块添加到依赖项中。

   例如：

   ```text
   [dependencies]
   frame-support = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v1.0.0"}
   ```

   请注意，您应该为所有模块使用相同的 branch 和版本信息，以确保导入的模块彼此兼容。
   使用来自不同 branch 的模块可能会导致编译器错误。
   此示例说明了如果其他模块使用 `branch = "polkadot-v1.0.0"`，则将 `frame-support` 模块添加到 `Cargo.toml` 文件中。

   由于构建过程会编译标准二进制文件和 WebAssembly 目标，因此您还必须在模块的特性中包含 `frame-support/std`。

1. 将 `frame-support/std` 添加到模块的 `std` 特性中。

   例如：

   ```text
   [features]
   default = ["std"]
   std = [
     "frame-support/std",
   ]
   ```

1. 保存您的更改并关闭 `Cargo.toml` 文件。

## 导入您想要使用的特性

在此示例中，您想要使用 `frame-support` 模块中的 [`Currency`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html) 特性，以便当前模块可以访问其方法。

要从另一个模块导入特性：

1. 在您的计算机上打开一个终端 shell 并导航到项目的根目录。

2. 在文本编辑器中打开当前模块的 `src/lib.rs` 文件。

3. 通过添加以下行来导入 `Currency` 特性：

   ```rust
   use frame_support::traits::Currency;
   ```

4. 保存您的更改。

## 向配置特性添加一个类型

下一步是创建一个类型，该类型由您想要在模块中公开的类型绑定。

要更新模块的配置特性：

1. 在您的计算机上打开一个终端 shell 并导航到项目的根目录。

2. 在文本编辑器中打开当前模块的 `src/lib.rs` 文件。

3. 创建一个要在模块中使用的类型，该类型由您想要在外部模块中访问的类型绑定：

   例如：

   ```rust
   pub trait Config: frame_system::Config {
       // --snip--

       /// 访问我们松散耦合的模块 `my-pallet` 的类型
       type LocalCurrency: Currency<Self::AccountId>;
   }
   ```

5. 使用已创建的类型使用松散耦合模块的特性提供的方法来访问该方法。

   例如：

   ```rust
   // 使用 `my-pallet` 的 getter
   let total_balance = T::LocalCurrency::total_issuance();
   ```

   在此示例中，[`total_issuance`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#tymethod.total_issuance) 是 `Currency` 特性从 `frame-support` 模块公开的方法。

1. 保存您的更改并关闭项目的 `src/lib.rs` 文件。

## 在运行时提供实现

完成项目的更新后，您就可以实现运行时配置以包含基于 `Currency` 特性的 `LocalCurrency` 了。

要更新模块的运行时配置：

1. 在您的计算机上打开一个终端 shell 并导航到节点模板的根目录。

2. 在文本编辑器中打开 `runtime/src/lib.rs` 文件。

1. 添加模块的运行时配置以指定 `LocalCurrency` 类型以使用为 `Balances` 模块定义的实现。

   ```rust
   impl my_pallet::Config for Runtime {
     type LocalCurrency = Balances;
   }
   ```

1. 检查 `construct_runtime!` 宏内的 `Balances` 定义。

   ```rust
   construct_runtime! (
     pub enum Runtime where
     Block = Block,
     NodeBlock = opaque::Block,
     UncheckedExtrinsic = UncheckedExtrinsic
     {
       Balances: pallet_balances,
     }
   )
   ```

   在此示例中，您的模块可以从 [`pallet_balances`](https://paritytech.github.io/substrate/master/pallet_balances/index.html#implementations-1) 模块继承 `Currency` 特性的实现并访问松散耦合的 `frame-support` 模块中的方法。

   默认情况下，`construct_runtime!` 宏包含宏定义中列出的所有模块的所有模块属性。

## 示例

- [Democracy 模块](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/democracy/src/lib.rs#L298-L335) 中的 [`EnsureOrigin`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.EnsureOrigin.html) 特性
- [Identity 模块](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/identity/src/lib.rs#L149-L151) 中的 [权重方法](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/identity/src/weights.rs#L46-L64)。
-[Grandpa 模块](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/grandpa/src/lib.rs#L106) 中的 [`KeyOwnerProofSystem`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.KeyOwnerProofSystem.html)。

## 资源

- [模块耦合](/build/pallet-coupling)
- [使用紧密模块耦合](/reference/how-to-guides/pallet-design/use-tight-coupling/)
