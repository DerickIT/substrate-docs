---
title: 使用紧密模块耦合
description:
keywords:
---

<div class="warning">
	 请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_pallet_coupling/index.html">Rust 文档</a> 获取有关此主题的最新文档。
</div>

紧密耦合两个模块是一种编写模块的技术，该技术重用现有模块中的类型和方法。

它对于将一些运行时逻辑分解成需要访问公共类型和方法的单独模块很有用。
要了解有关模块之间紧密耦合和松散耦合的更多信息，请参阅 [模块耦合](/build/pallet-coupling)。

本指南将向您展示如何在您自己的模块中使用现有模块中的方法。

## 开始之前

本指南假设您有一个模块，您想将另一个模块与其耦合。
如果您还没有要使用的模块，请使用 [模板模块](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/pallets/template/src/lib.rs)。

1. 配置您的工作区

   假设您想使用一个名为 `special-pallet` 的模块，它是您本地工作区中的一个模块。
   通过在模块的 `Cargo.toml` 文件中提供其路径来导入它：

   ```toml
   special-pallet = { path = '../special-pallet', default-features = false }
   ```

1. 向您的模块添加一个特性边界

   现在您需要做的就是围绕模块的配置特性创建一个特性边界：

   ```rust
   pub trait Config: frame_system::Config + special_pallet::Config {
       // --snip--
   }
   ```

1. 使用 getter 函数

   为了使用 `special-pallet` 中的方法，请按以下方式调用它：

   ```rust
   // 从 `special-pallet` 模块获取成员
   let who = special_pallet::Pallet::<T>::get();
   ```

   上面的代码片段假设 `special-pallet` 包含一个名为 `get()` 的方法，该方法返回一个帐户 ID 向量。

## 示例

- FRAME 的 [Bounties](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/bounties)
  和 [Tipping](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/tips) 模块与 Treasury 模块

## 资源

- [模块耦合](/build/pallet-coupling)
- [松散耦合两个模块](/reference/how-to-guides/pallet-design/use-loose-coupling/)
