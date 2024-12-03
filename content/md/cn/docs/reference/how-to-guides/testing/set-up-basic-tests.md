---
title: 设置基本测试
description:
keywords:
  - 基础
  - 测试
  - 运行时
---

学习如何设置编写 pallet 测试所需的脚手架。

本指南逐步介绍了如何使用 `mock.rs` 和 `test.rs` 作为测试 pallet 的基础。
我们将使用节点模板作为 `mock.rs` 文件的脚手架，以及一个名为 `pallet-testing` 的任意 pallet 来为本指南提供一些上下文。
此 pallet 将包含一个名为 `add_value` 的单个函数，该函数接受一个来源和一个 `u32`，如果该值小于或等于我们在模拟运行时中指定的名为 `MaxValue` 的常量，则返回 `Ok(())`。

## 使用模板节点作为样板

在 `pallet-testing/src` 内部，我们需要做的第一件事是创建两个空文件：`mock.rs` 和 `tests.rs`。

粘贴 [`template/src/mock.rs`](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/pallets/template/src/mock.rs) 中的内容。
我们将使用它作为我们将为 `pallet-testing` pallet 定制的样板。

## 创建模拟运行时以测试您的 pallet

1. 首先修改 `pallet-testing/src/mock.rs` 以包含 `pallet-testing` pallet。这涉及以下代码部分的更改：

1. 将第一行替换为 pallet 的名称，在本例中为 `pallet_testing`：

   ```rust
   use crate as pallet_testing;
   /*--snip--*/
   ```

## 配置模拟运行时

1. 在 `frame_support::construct_runtime!` 中，将 `pallet_template` 替换为您的 pallet 的名称，在本例中为 `pallet_testing`：

   ```rust
   /*--snip--*/
   TestingPallet: pallet_testing::{Pallet, Call, Storage, Event<T>},
   /*--snip--*/
   ```

1. 为模拟运行时实现您的 pallet。将 `impl pallet_template::Config for Test {...}` 替换为您的配置类型和 pallet 需要的任何常量值：

   ```rust
   parameter_types! {
     pub const MaxValue: u32 = 50;
   }

   impl pallet_testing::Config for Test {
     type RuntimeEvent = RuntimeEvent;
     type MaxValue = MaxValue;
   }
   ```

要使用模拟运行时，我们需要为 `pallet-testing` pallet 设置 `tests.rs` 文件。

## 设置和创建测试

在 `tests.rs` 中，首先使用 `super` 从 `lib.rs` 导入您需要的依赖项：

```rust
use super::*;
```

1. 测试错误是否按预期工作：

   ```rust
   #[test]
   fn error_works(){
     new_test_ext().execute_with(|| {
       assert_err!(
         TestingPallet::add_value(RuntimeOrigin::signed(1), 51),
         "value must be <= maximum add amount constant"
       );
     })
   }
   ```

1. 创建一个应该有效的测试：

   ```rust
   #[test]
   fn test_should_work() {
     new_test_ext().execute_with(|| {
       assert_ok!(
         TestingPallet::add_value(RuntimeOrigin::signed(1), 10)
       );
     })
   }
   ```

1. 再创建一个应该失败的测试：

   ```rust
   #[test]
   fn test_should_fail() {
     new_test_ext().execute_with(|| {
       assert_ok!(
         TestingPallet::add_value(RuntimeOrigin::signed(1), 100)
       );
     })
   }
   ```

## 运行您的测试

从您的 pallet 目录执行以下命令：

```bash
cargo test
```

## 示例

- [`pallet_template`](https://github.com/substrate-developer-hub/substrate-node-template/blob/master/pallets/template/src/tests.rs#L1-L23) 中的测试
- [`reward-coin`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/reward-coin/src/tests.rs) 示例 pallet 中的测试

## 相关资料

- [单元测试](/test/unit-testing/)
- [测试转移函数](/reference/how-to-guides/testing/test-a-transfer-function)
- [`assert_ok!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_ok.html)
- [`assert_err!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_err.html)
