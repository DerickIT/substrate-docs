---
title: 导入模块
description: 本指南演示如何快速将本地模块和外部模块集成到运行时。
keywords:
  - 基础
  - 初学者
  - 运行时
---

本指南演示如何快速将本地模块和外部模块集成到运行时。
有关更详细的分步说明，请参阅 [将模块添加到运行时](/tutorials/build-application-logic/add-a-pallet)。

本指南说明：

- 如何包含一个自定义本地模块，该模块为运行时实现了一个事件和一个调用。
- 如何在运行时包含来自 `Crates.io` 的外部模块。

## 创建本地模块

1. 创建一个名为 `pallet_something` 的本地模块。

1. 通过将以下内容添加到 `/runtime/src/lib.rs` 来导入此模块：

   ```rust
   // 导入模块。
   pub use pallet_something;
   ```

1. 配置模块的运行时实现。
   假设本地模块仅将 `Event` 和 `Call` 类型公开到运行时。将以下内容添加到 `/runtime/src/lib.rs`：

   ```rust
   // 配置模块。
   impl pallet_something::Config for Runtime {
   	type RuntimeEvent = RuntimeEvent;
   	type RuntimeCall = RuntimeCall;
   }
   ```

1. 为 [`construct_runtime` 宏](/reference/frame-macros/#construct_runtime) 声明模块：

   ```rust
   construct_runtime!(
   	pub enum Runtime where
   	Block = Block,
   	NodeBlock = opaque::Block,
   	UncheckedExtrinsic = UncheckedExtrinsic
   	{
   		/* --snip-- */
   		Something: pallet_something,
   		/* --snip-- */
   	}
   );
   ```

1. 更新 `/runtime/Cargo.toml`

   在 `/runtime/Cargo.toml` 中，将模块作为本地依赖项包含在 `std` 中，并添加 `runtime-benchmarks`。
   例如：

   ```toml
   # --snip--
   pallet-something = { default-features = false,   path = '../pallets/something'
   version = '3.0.0'
   # --snip--
   [features]
   default = ['std']
   runtime-benchmarks = [
     # --snip--
     'pallet-something/runtime-benchmarks',
   ]
   std = [
     'pallet-something/std',
     # --snip--
   ]
   ```

## 导入外部模块

要添加外部模块，您使用的方法与本地模块类似，但您必须包含模块公开的所有类型。
您还必须包含相关的参数类型和常量。
有关如何声明参数和常量的示例，请参阅 [`pallet_timestamp`](https://paritytech.github.io/substrate/master/pallet_timestamp/index.html)。

以下是将外部模块添加到 `/runtime/Cargo.toml` 依赖项中的示例，如果模块托管在 [crates.parity.io](https://crates.parity.io/) 上：

```toml
[dependencies]
pallet-external = {default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", version = "4.0.0-dev"}

# --snip--
runtime-benchmarks = [
  /* --snip */
  'pallet-external/runtime-benchmarks',
]
std = [
  'pallet-external/std',
  # --snip--
]
```

## 示例

- [模板模块](https://paritytech.github.io/substrate/master/pallet_template/index.html)
- [时间戳模块](https://paritytech.github.io/substrate/master/pallet_timestamp/index.html)

## 相关资料

- [时间戳模块关联类型](https://paritytech.github.io/substrate/master/pallet_timestamp/index.html)
- [FRAME `pallet-timestamp`](https://crates.io/crates/pallet-timestamp)<!-- markdown-link-check-disable-line -->
- [单元测试](/test/unit-testing)
