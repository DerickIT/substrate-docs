---
title: 随机性
description: 描述如何在 Substrate 运行时中引入随机性。
keywords:
---

由于区块链需要确定性结果（相同的输入始终产生相同的输出），因此寻找合适的来源来产生看似随机的结果带来独特的挑战。
但是，在许多应用程序中，随机性对于执行某些操作至关重要。
例如，你希望包含随机性来选择负责生成块的验证者，以防止验证者选择具有可预测性，从而避免在攻击中被利用。
你还可以将随机性用于统计或科学分析、密码学操作或涉及游戏或赌博的应用程序中。

## 确定性随机性

在传统的计算机应用程序中，人们所说的随机数实际上是使用**伪随机性**生成的。
伪随机性取决于足够随机的种子（由用户或外部来源提供）和一个用于操作种子以生成一系列看似随机数字的算法。
结果是一个伪随机数，由于用于生成它的算法而难以预测，但由于相同的种子输入始终产生相同的输出序列而具有确定性。

但是，在区块链上运行的应用程序受到更严格的限制，因为网络中的所有权威机构都必须就任何链上值达成一致，包括注入的任何随机性数据。
由于此约束，你不能直接在区块链应用程序中使用真正的随机性。

对于区块链应用程序，提供随机性的最常见方法是称为[可验证随机函数](https://en.wikipedia.org/wiki/Verifiable_random_function)的密码学基元。
可验证随机函数 (VRF) 是一种数学运算，它接收输入并生成随机数以及此随机数由提交者生成的真实性证明。
任何挑战者都可以验证该证明，以确保随机数生成有效。

在 Polkadot 生态系统和基于 Substrate 的链中，可验证随机函数是 BABE 模块提供的共识机制的一部分。
有关可验证随机函数与共识之间关系的更多信息，请参阅 [共识](/learn/consensus/)

## 生成和使用随机性

Substrate 提供了一个名为 [`Randomness`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html) 的特性，该特性定义了**生成随机性**的逻辑与**使用随机性**的逻辑之间的接口。
此特性允许你独立编写生成随机性和使用随机性的逻辑。

### 生成随机性

你可以通过多种不同的方式实现 `Randomness` 特性，具体取决于你的应用程序所需的安全性保证和性能权衡。
Substrate 包含两个关于如何在提供不同性能、复杂性和安全性权衡的模块中实现 `Randomness` 特性的示例。

- [不安全的随机性](https://paritytech.github.io/substrate/master/pallet_insecure_randomness_collective_flip/index.html) 模块提供了一个 `random` 函数，该函数基于前 81 个块的块哈希生成伪随机值。

  这种类型的随机性性能良好，但不安全。
  你应该只在安全性要求低的应用程序中或在测试使用随机性的应用程序时使用此模块。
  你不应该在生产环境中使用此模块。

- [BABE 模块](https://paritytech.github.io/substrate/master/pallet_babe/index.html) 通过使用可验证随机函数来提供随机性。

此模块提供生产级随机性，并用于 Polkadot。
如果你选择此模块作为区块链的随机性来源，则你的区块链必须使用盲分配区块链扩展（[BABE](/reference/glossary/#blind-assignment-of-blockchain-extension-babe)）基于时隙的共识来生成块。

### 使用随机性

`Randomness` 特性提供以下方法来使用随机性：

- `random_seed` 方法不接受任何参数并返回原始随机值。
  如果你在一个块中多次调用此方法，它每次都会返回相同的值。
  因此，在大多数情况下，你不应该直接使用此方法。

- `random` 方法将字节数组作为上下文标识符，并返回一个对该上下文唯一且独立于其他上下文的（由底层随机性来源允许的）结果。

需要随机值的模块不需要提供随机性来源，但它们需要指定一个实现了 `Randomness` 特性的随机性来源。

### 安全性保证

重要的是要记住，`Randomness` 特性提供了一个方便的抽象，用于在运行时中定义随机性来源，但特性本身没有任何安全保证。
作为运行时开发者，你必须确保你使用的随机性来源满足_所有使用其随机性的模块_的安全性要求。

## 接下来的步骤

- [操作方法：随机性](/reference/how-to-guides/pallet-design/incorporate-randomness/)
- [`Randomness`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html)
