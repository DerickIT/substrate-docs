---
title: 添加基准测试
description: 演示如何使用基准测试框架来估计 pallet 的执行需求。
keywords:
  - 权重
  - 基准测试
  - 运行时
---

本指南说明了如何为 pallet 编写简单的基准测试，测试基准测试，并运行 [基准测试](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/benchmarking) 命令以生成关于 pallet 中函数所需执行时间的实际估计值。
本指南不涵盖如何使用基准测试结果来更新交易权重。

## 向 pallet 添加基准测试

1. 在文本编辑器中打开 pallet 的 [`Cargo.toml`](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/Cargo.toml) 文件。

2. 使用与 pallet 中其他依赖项相同的版本和分支，将 `frame-benchmarking` crate 添加到 pallet 的 [dependencies] 中。

   例如：

   ```toml
   frame-benchmarking = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
   ```
1. 将 `runtime-benchmarks` 添加到 pallet 的 [features] 列表中。

   例如：

   ```toml
   [features]
   runtime-benchmarks = ["frame-benchmarking/runtime-benchmarks"]
   ```

1. 将 `frame-benchmarking/std` 添加到 pallet 的 `std` 功能列表中。

   例如：

   ```toml
   std = [
      ...
      "frame-benchmarking/std",
      ...
   ]
   ```

## 添加基准测试模块

1. 在 pallet 的 `src` 文件夹中创建一个新的文本文件（例如，`benchmarking.rs`）。

2. 在文本编辑器中打开 `benchmarking.rs` 文件，并创建一个定义 pallet 基准测试的 Rust 模块。

   您可以使用任何预构建 pallet 的 `benchmarking.rs` 作为 Rust 模块中包含内容的示例。
   通常，该模块应包含类似于以下内容的代码：

   ```rust
   #![cfg(feature = "runtime-benchmarks")]
   mod benchmarking;

   use crate::*;
   use frame_benchmarking::{benchmarks, whitelisted_caller};
   use frame_system::RawOrigin;

   benchmarks! {
      // 在此处添加各个基准测试
      benchmark_name {
         /* 设置初始状态的代码 */
      }: {
         /* 测试被基准测试的函数的代码 */
      }
      verify {
         /* 可选验证 */
      }
   }
   ```

3. 编写各个基准测试以测试 pallet 中函数的最计算密集型路径。

   基准测试宏会自动为基准测试模块中包含的每个基准测试生成一个测试函数。
   例如，宏会创建类似于以下内容的测试函数：

   ```rust
   fn test_benchmarking_[benchmark_name]<T>::() -> Result<(), &'static str>
   ```

   [pallet-example-basic](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/benchmarking.rs) 的基准测试模块提供了一些简单的示例基准测试。
   例如：

   ```rust
   benchmarks! {
     set_dummy_benchmark {
       // 基准测试设置阶段
       let b in 1 .. 1000;
     }: set_dummy(RawOrigin::Root, b.into()) // 执行阶段
     verify {
		   // 可选验证阶段
       assert_eq!(Pallet::<T>::dummy(), Some(b.into()))
     }
   }
   ```

   在此示例代码中：

   - 基准测试的名称为 `set_dummy_benchmark`。
   - 变量 `b` 存储用于测试 `set_dummy` 函数执行时间的输入。
   - `b` 的值在 1 到 1,000 之间变化，因此您可以重复运行基准测试以使用不同的输入值来测量执行时间。

## 测试基准测试

在将基准测试添加到 pallet 的基准测试模块中的 `benchmarks!` 宏之后，您可以使用模拟运行时进行单元测试，并确保基准测试的测试函数返回 `Ok(())` 作为结果。

1. 在文本编辑器中打开 `benchmarking.rs` 基准测试模块。

2. 将 `impl_benchmark_test_suite!` 宏添加到基准测试模块的底部：

   ```rust
   impl_benchmark_test_suite!(
     MyPallet,
     crate::mock::new_test_ext(),
     crate::mock::Test,
   );
   ```

   `impl_benchmark_test_suite!` 宏采用以下输入：

   - pallet 生成的 Pallet 结构体，在此示例中为 `MyPallet`。
   - 生成测试创世存储的函数，`new_text_ext()`。
   - 完整的模拟运行时结构体，`Test`。

   这与您用于为单元测试设置模拟运行时的信息相同。
   如果所有基准测试都在模拟运行时测试环境中通过，则它们在实际运行时运行基准测试时很可能也能正常工作。

3. 通过为名为 `pallet-mycustom` 的 pallet 运行类似于以下命令的命令，在模拟运行时中执行为 pallet 生成的基准测试单元测试：

   ```bash
   cargo test --package pallet-mycustom --features runtime-benchmarks
   ```

4. 验证测试结果。

   例如：

   ```text
   正在运行 4 个测试
   test mock::__construct_runtime_integrity_test::runtime_integrity_tests ... ok
   test tests::it_works_for_default_value ... ok
   test tests::correct_error_for_none_value ... ok
   test benchmarking::bench_do_something ... ok

   测试结果：ok。4 个通过；0 个失败；0 个忽略；0 个已测量；0 个已过滤；在 0.00 秒内完成
   ```

## 向运行时添加基准测试

在将基准测试添加到 pallet 后，还必须更新运行时以包含 pallet 和 pallet 的基准测试。

1. 在文本编辑器中打开运行时的 `Cargo.toml` 文件。

2. 将您的 pallet 添加到运行时的 `[dependencies]` 列表中：

   ```toml
   pallet-mycustom = { default-features = false, path = "../pallets/pallet-mycustom"}
   ```

3. 更新运行时的 `[features]` 以包含 pallet 的 `runtime-benchmarks`：

   ```toml
   [features]
   runtime-benchmarks = [
     ...
     'pallet-mycustom/runtime-benchmarks'
     ...
   ]
   ```

4. 更新运行时的 `std` 功能以包含您的 pallet：

   ```toml
    std = [
     # -- snip --
     'pallet-mycustom/std'
   ]
   ```

5. 将 pallet 的配置特性添加到运行时。

6. 将 pallet 添加到 `construct_runtime!` 宏中。

   如果您需要有关将 pallet 添加到运行时的更多详细信息，请参阅 [将 pallet 添加到运行时](/tutorials/build-application-logic/add-a-pallet) 或 [导入 pallet](/reference/how-to-guides/basics/import-a-pallet)。

7. 将您的 pallet 添加到 `runtime-benchmarks` 功能中的 `define_benchmark!` 宏中。

   ```rust
   #[cfg(feature = "runtime-benchmarks")]
   mod benches {
       define_benchmarks!(
         [frame_benchmarking, BaselineBench::<Runtime>]
         [pallet_assets, Assets]
         [pallet_babe, Babe]
         ...
         [pallet_mycustom, MyPallet]
         ...
       );
   }
   ```

## 运行基准测试

更新运行时后，您就可以使用启用了 `runtime-benchmarks` 功能来编译它，并开始为您的 pallet 启动基准测试分析。

1. 通过运行以下命令来启用 `runtime-benchmarks` 功能构建您的项目：

   ```bash
   cargo build --package node-template --release --features runtime-benchmarks
   ```

2. 查看节点 `benchmark pallet` 子命令的命令行选项：

   ```bash
   ./target/release/node-template benchmark pallet --help
   ```

   `benchmark pallet` 子命令支持多个命令行选项，这些选项可以帮助您自动化基准测试。
   例如，您可以设置 `--steps` 和 `--repeat` 命令行选项以使用不同的值多次执行函数调用。

3. 通过运行类似于以下命令的命令来启动 pallet 的基准测试：

   ```bash
   ./target/release/node-template benchmark pallet \
    --chain dev \
    --pallet pallet_mycustom \
    --extrinsic '*' \
    --steps 20 \
    --repeat 10 \
    --output pallets/pallet-mycustom/src/weights.rs
   ```

   此命令会在指定的目录中创建一个 `weights.rs` 文件。
   有关如何配置 pallet 以使用这些权重的信息，请参阅 [使用自定义权重](/reference/how-to-guides/weights/use-custom-weights/)。

## 示例

您可以使用任何预构建 pallet 的 `benchmarking.rs` 和 `weights.rs` 文件来了解有关基准测试不同类型的函数的更多信息。

- [示例 pallet：基准测试](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/benchmarking.rs)
- [示例 pallet：权重](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/weights.rs)
- [Balances pallet：基准测试](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/balances/src/benchmarking.rs)
- [Balances pallet：权重](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/balances/src/weights.rs)
