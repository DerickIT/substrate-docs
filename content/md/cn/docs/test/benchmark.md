---
title: 基准测试
description: 描述您可以用来估计执行运行时逻辑中的函数所需的计算资源的基准测试框架。
keywords:
  - 权重
  - 资源消耗
  - 执行时间
---

Substrate 和 FRAME 提供了一个灵活的框架，用于为您的区块链开发自定义逻辑。
这种灵活性使您可以设计复杂且交互式的 pallet 并实现复杂的运行时逻辑。
但是，确定要为 pallet 中的函数分配的适当 [权重](/reference/glossary/#weight) 可能是一项艰巨的任务。
基准测试使您可以衡量在运行时以及在不同条件下执行不同函数所需的时间。
如果您使用基准测试为函数调用分配准确的权重，则可以防止您的区块链过载并无法生成区块，或者不会因为恶意行为者而容易受到拒绝服务 (DoS) 攻击。

## 为什么对 pallet 进行基准测试

了解执行不同函数（包括运行时函数，如 `on_initialize` 和 `verify_unsigned`）所需的计算资源非常重要，这样可以确保运行时的安全，并使运行时能够根据可用资源来包含或排除交易。

能够根据可用资源包含或排除交易可确保运行时能够继续生成和导入区块，而不会中断服务。
例如，如果您有一个需要特别密集计算的函数调用，则执行该调用可能会超过生成或导入区块所允许的最大时间，从而中断区块处理过程或完全停止区块链的进展。
基准测试可以帮助您验证不同函数所需的执行时间是否在合理的范围内。

同样，恶意用户可能会尝试通过重复执行需要密集计算或不准确反映其所需计算的函数调用来破坏网络服务。
如果执行函数调用的成本不能准确反映所涉及的计算，则没有动力阻止恶意用户攻击网络。
因为基准测试可以帮助您评估与执行交易相关的权重，所以它还可以帮助您确定适当的交易费用。
根据您的基准测试，您可以设置代表执行区块链上特定调用所消耗的资源的费用。

## 开发线性模型

在高级别上，基准测试需要您执行以下步骤：

- 编写自定义基准测试逻辑，以对函数执行特定代码路径。
- 在特定硬件集和特定运行时配置上，在 WebAssembly 执行环境中执行基准测试逻辑。
- 在可能影响函数所需执行时间的可能值的受控范围内执行基准测试逻辑。
- 对函数中的每个组件多次执行基准测试，以隔离和去除异常值。

根据执行基准测试逻辑生成的結果，基准测试工具会为所有组件创建函数的线性模型。
函数的线性模型使您可以估计执行特定代码路径所需的时间，并在运行时无需实际花费任何大量资源的情况下做出明智的决策。
基准测试假设所有交易都具有线性复杂性，因为复杂度较高的函数被认为对运行时来说是危险的，因为这些函数的权重可能会随着运行时状态或输入变得过于复杂而爆炸式增长。

## 基准测试和权重

如 [交易、权重和费用](/build/tx-weights-fees/) 中所述，基于 Substrate 的链使用**权重**的概念来表示执行区块中的交易所需的时间。
执行交易中任何特定调用所需的时间取决于几个因素，包括：

- 计算复杂性。
- 存储复杂性。
- 所需的数据库读取和写入操作。
- 使用的硬件。

要计算交易的适当权重，您可以使用基准测试参数来衡量在不同硬件上、使用不同的变量值以及多次重复执行函数调用所需的时间。
然后，您可以使用基准测试结果来确定一个近似的最坏情况权重，以表示执行每个函数调用和每个代码路径所需的资源。
费用则基于最坏情况权重。
如果实际调用比最坏情况执行得更好，则会调整权重，并且可以返还任何多余的费用。

因为权重是基于特定物理机器的计算时间的通用度量单位，所以任何函数的权重都可能根据用于基准测试的特定硬件而变化。

通过对每个运行时函数的预期权重进行建模，区块链能够计算出在特定时间段内可以执行多少交易或系统级调用。

在 FRAME 中，每个可以调度的函数调用都必须具有 `#[weight]` 注解，该注解可以返回给定函数的输入后，该函数的最坏情况执行的预期权重。
基准测试框架会自动为您生成包含这些公式的文件。

## 基准测试工具

[基准测试框架](https://paritytech.github.io/substrate/master/frame_benchmarking/index.html) 提供了工具，可帮助您添加、测试、运行和分析运行时函数的基准测试。
帮助您确定执行函数调用所需时间的基准测试工具包括：

- [基准测试宏](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/benchmarking/src/lib.rs) 可帮助您编写、测试和添加运行时基准测试。
- 用于处理基准测试数据的 [线性回归分析函数](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/benchmarking/src/analysis.rs)。
- [命令行界面 (CLI)](hhttps://github.com/paritytech/polkadot-sdk/tree/master/substrate/utils/frame/benchmarking-cli) 使您可以对节点执行基准测试。

在编译节点时，端到端基准测试管道默认情况下处于禁用状态。
如果您想运行基准测试，则需要使用 `runtime-benchmarks` Rust 功能标志编译节点。

## 编写基准测试

编写运行时基准测试类似于为您的 pallet 编写单元测试。
与单元测试一样，基准测试必须执行代码中的特定逻辑路径。
在单元测试中，您会检查代码的特定成功和失败结果。
对于基准测试，您需要执行**最计算密集型**的路径。

在编写基准测试时，您应该考虑可能影响函数复杂性的特定条件（例如存储或运行时状态）。
例如，如果在 `for` 循环中触发更多迭代会增加数据库读取和写入操作的数量，则应设置一个触发此条件的基准测试，以更准确地表示函数的执行情况。

如果函数根据用户输入或其他条件执行不同的代码路径，则您可能不知道哪个路径是最计算密集型的。
为了帮助您了解代码中的复杂性可能变得难以管理的地方，您应该为每个可能的执行路径创建一个基准测试。
基准测试可以帮助您识别代码中可能需要强制执行边界的地方（例如，通过限制向量中的元素数量或限制 `for` 循环中的迭代次数）来控制用户与 pallet 的交互方式。

您可以在所有预构建的 [FRAME pallet](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame) 中找到端到端基准测试的示例。

## 测试基准测试

您可以使用与为 pallet 的单元测试创建的相同的模拟运行时来测试基准测试。
您在 `benchmarking.rs` 模块中使用的基准测试宏会自动为您生成测试函数。
例如：

```rust
fn test_benchmark_[benchmark_name]<T>::() -> Result<(), &'static str>
```

您可以将基准测试函数添加到单元测试中，并确保函数的结果为 `Ok(())`。

### 验证区块

通常，您只需要检查基准测试是否返回 `Ok(())`，因为该结果表示该函数已成功执行。
但是，如果您想验证任何最终条件（例如运行时的最终状态），则可以在基准测试中选择包含 `verify` 块。
附加的 `verify` 块不会影响最终基准测试过程的结果。

### 使用基准测试运行单元测试

要运行基准测试，您需要指定要测试的包并启用 `runtime-benchmarks` 功能。
例如，您可以通过运行以下命令来测试 Balances pallet 的基准测试：

```bash
cargo test --package pallet-balances --features runtime-benchmarks
```

## 添加基准测试

每个 pallet 附带的基准测试不会自动添加到您的节点中。
要执行这些基准测试，您需要实现 `frame_benchmarking::Benchmark` 特性。
您可以在 [Substrate 节点](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/bin/node/runtime/src/lib.rs) 中看到如何执行此操作的示例。

假设您的节点上已经设置了一些基准测试，您只需要将 pallet 添加到 `define_benchmarks!` 宏中：

```rust
#[cfg(feature = "runtime-benchmarks")]
mod benches {
	define_benchmarks!(
		[frame_benchmarking, BaselineBench::<Runtime>]
		[pallet_assets, Assets]
		[pallet_babe, Babe]
    ...
    [pallet_mycustom, MyCustom]
    ...
```

添加 pallet 后，请使用 `runtime-benchmarks` 功能标志编译节点二进制文件。
例如：

```bash
cd bin/node/cli
cargo build --profile=production --features runtime-benchmarks
```

`production` 配置文件应用各种编译器优化。
这些优化会_大大_减慢编译过程。
如果您只是在测试某些内容并且不需要最终数字，请使用 `--release` 命令行选项而不是 `production` 配置文件。

## 运行基准测试

启用基准测试后，您需要编译节点二进制文件才能执行基准测试。
如果您使用 `production` 配置文件编译节点，则可以通过运行以下命令列出可用的基准测试：

```bash
./target/production/solo-template-node benchmark pallet --list
```

### 对所有 pallet 中的所有函数进行基准测试

要执行运行时的所有基准测试，您可以运行类似于以下命令的命令：

```bash
./target/production/solo-template-node benchmark pallet \
    --chain dev \
    --execution=wasm \
    --wasm-execution=compiled \
    --pallet "*" \
    --extrinsic "*" \
    --steps 50 \
    --repeat 20 \
    --output pallets/all-weight.rs
```

此命令会创建一个输出文件（在本例中，名为 `all-weight.rs` 的文件），该文件为您的运行时实现了 `WeightInfo` 特性。

### 对 pallet 中的特定函数进行基准测试

要执行特定 pallet 中特定函数的基准测试，您可以运行类似于以下命令的命令：

```bash
./target/production/solo-template-node benchmark pallet \
    --chain dev \
    --execution=wasm \
    --wasm-execution=compiled \
    --pallet pallet_balances \
    --extrinsic transfer \
    --steps 50 \
    --repeat 20 \
    --output pallets/transfer-weight.rs
```

此命令会为选定的 pallet（例如，`transfer-weight.rs`）创建一个输出文件，该文件为 `pallet_balances` pallet 实现了 `WeightInfo` 特性。

### 使用模板格式化基准测试

基准测试命令行界面使用 Handlebars 模板来格式化最终输出文件。
您可以选择传递 `--template` 命令行选项以指定自定义模板而不是默认模板。
在模板中，您可以访问基准测试命令行界面中 `TemplateData` 结构体提供的所有数据。

输出生成中包含一些自定义 Handlebars 辅助函数：

- `underscore`：从字符串的右侧每隔 3 个字符添加一个下划线。
  主要用于分隔大数字。
- `join`：将字符串数组连接成一个空格分隔的字符串，用于模板。
  主要用于连接传递给 CLI 的所有参数。

要获取 `benchmark` 子命令的完整列表，请运行：

```bash
./target/production/solo-template-node benchmark --help
```

要获取 `benchmark pallet` 子命令的可用选项的完整列表，请运行：

```bash
./target/production/solo-template-node benchmark pallet --help
```

## 接下来的步骤

- [frame-benchmarking 自述文件](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/benchmarking/README.md)
- [Substrate 研讨会：对您的 Substrate Pallet 进行基准测试](https://www.youtube.com/watch?v=Qa6sTyUqgek)
- [操作指南：添加基准测试](/reference/how-to-guides/weights/add-benchmarks)
- [命令参考：solo-template-node benchmark](/reference/command-line-tools/solo-template-node/#benchmark)
