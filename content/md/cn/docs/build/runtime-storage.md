---
title: 运行时存储结构
description:
keywords:
---

在开发运行时逻辑时，你需要对存储的信息以及如何尽可能高效地存储信息做出重要决策。
如[状态转换和存储](/learn/state-transitions-and-storage/)中所述，读取和写入存储数据代价很高。
此外，存储不必要的大型数据集可能会减慢网络速度并占用系统资源。

Substrate 旨在提供一个灵活的框架，使你能够构建适合你需求的区块链。
但是，在设计运行时存储时，你应该记住一些基本准则，以确保你构建的区块链在长期内是安全、高性能且可维护的。

## 决定存储什么

区块链运行时存储的基本原则是最大限度地减少存储的数据项的数量和大小。
例如，你应该只在运行时存储_与共识相关的_信息。
你不应在运行时存储中间或临时数据，也不应存储如果操作失败则不需要的数据。

## 使用哈希数据

只要有可能，就使用哈希等技术来减少必须存储的数据量。
例如，许多治理功能（例如，民主模块中的 [`propose`](https://paritytech.github.io/substrate/master/pallet_democracy/pallet/enum.Call.html#variant.propose) 函数）允许网络参与者对可调度调用的_哈希_进行投票，而不是对调用本身进行投票。
调用的哈希大小始终是有界的，而调用的长度可能是不确定的。

在运行时升级的情况下，使用调用的哈希尤其重要，其中可调度调用将整个运行时 Wasm blob 作为其参数。
由于这些治理机制是在_链上_实现的，因此就给定提案的状态达成共识所需的所有信息也必须存储在链上 - 这包括_正在投票的内容_。
但是，通过将链上提案绑定到其哈希，Substrate 的治理机制允许以一种方式来完成此操作，即在提案被批准_之后_才将与提案相关的所有数据带到链上。
这意味着不会浪费存储在失败的提案上。

提案通过后，有人可以启动实际的可调度调用（包括其所有参数），该调用将被哈希并与提案中的哈希进行比较。

另一种使用哈希来最大限度地减少存储在链上的数据的常见模式是将与对象关联的预映像存储在 [IPFS](https://docs.ipfs.io) 中；这意味着只需要将 IPFS 位置（大小有界的哈希）存储在链上。

### 避免存储瞬态数据

不要使用运行时存储来存储在逻辑上是原子的操作上下文中或如果操作失败则不需要的中间或瞬态数据。
这并不意味着不应使用运行时存储来跟踪需要多个原子操作的动作的状态，例如[实用程序模块的多签名功能](https://paritytech.github.io/substrate/master/pallet_utility/pallet/enum.Call.html#variant.as_multi)。
在这种情况下，运行时存储用于跟踪可调度调用上的签名者，即使给定的调用可能永远不会收到足够的签名来实际调用。
在这种情况下，每个签名都被认为是正在进行的多签名操作中的原子事件。
在满足与该签名相关的所有先决条件后，才会存储记录单个签名所需的数据。

### 创建边界

为存储项的大小创建边界是控制运行时存储使用的一种极其有效的方法，并且在整个 Substrate 代码库中反复使用。
通常，大小由用户操作确定的任何存储项都应该对其进行限制。
上面描述的 [多签名模块](https://paritytech.github.io/substrate/master/pallet_multisig/pallet/trait.Config.html#associatedtype.MaxSignatories) 中的多签名功能就是一个这样的例子。
在这种情况下，与多签名操作关联的签名者列表由多签名参与者提供。
由于此签名者列表[对于就多签名操作的状态达成共识](#what-to-store) 是必要的，因此必须将其存储在运行时中。但是，为了控制签名者列表可以使用多少空间，实用程序模块要求用户配置此数字的边界，作为写入存储之前的先决条件。

## 事务性存储

如[状态转换和存储](/learn/state-transitions-and-storage/)中所述，运行时存储涉及底层键值数据库和内存存储覆盖抽象，这些抽象跟踪密钥和状态更改，直到将值提交到底层数据库。
默认情况下，运行时中的函数会在将更改提交到主存储覆盖层之前将其写入单个内存**事务性存储层**。
如果错误阻止事务完成，则会丢弃事务性存储层中的更改，而不是将其传递到主存储覆盖层，并且底层数据库中的状态保持不变。

### 添加事务性存储层

你可以使用 `#[transactional]` 宏来生成其他内存存储覆盖层，从而扩展事务性存储层。
通过生成其他内存事务性存储覆盖层，你可以选择是否要将特定更改提交到主存储覆盖层。
其他事务性存储层使你能够隔离对特定函数调用的更改，并在任何时候选择要提交的更改。

你还可以嵌套最多十个嵌套的事务性存储层。
对于你创建的每个嵌套事务性存储层，你都可以选择是否要将更改提交到其下方的事务性层，这使你能够很好地控制提交到底层数据库的内容。
限制嵌套事务性存储层的总数可以限制解决要提交的更改的计算开销。

### 调度事务性存储层调用

如果你想在自己的事务性层内调度函数调用，则可以使用 `dispatch_with_transactional(call)` 函数为调用显式生成新的事务性层，并使用该事务性层上下文来处理结果。

### 在没有事务性存储层的情况下提交更改

如果你想在不使用默认事务性存储层的情况下将更改提交到主存储覆盖层，则可以使用 `#[without_transactional]` 宏。
`#[without_transactional]` 宏使你能够识别无需其自身事务性层的函数。

例如，你可能会这样定义一个函数：

```rust
/// 此函数无需其他事务性存储层即可安全执行。
#[without_transactional]
fn set_value(x: u32) -> DispatchResult {
    Self::check_value(x)?;
    MyStorage::set(x);
    Ok(())
}
```

调用此函数不会生成事务性存储层。

但是，如果你使用 `#[without_transactional]` 宏，请记住，对存储的更改会影响主内存存储覆盖层中的值。
如果在你修改存储后发生错误，则这些更改将保留，并且可能导致你的数据库处于不一致状态。

## 访问运行时存储

在[状态转换和存储](/learn/state-transitions-and-storage/)中，你了解了 Substrate 如何使用存储抽象来提供对底层键值数据库的读写访问。
FRAME [`Storage`](https://paritytech.github.io/substrate/master/frame_support/storage) 模块简化了对这些分层存储抽象的访问。
你可以使用 FRAME 存储数据结构来读取或写入任何可以使用 [SCALE 编解码器](/reference/scale-codec/) 编码的值。
存储模块提供以下类型的存储结构：

- [StorageValue](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageValue.html) 用于存储任何单个值，例如 `u64`。
- [StorageMap](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageMap.html) 用于存储单个键值映射，例如特定帐户密钥到特定余额值。
- [StorageDoubleMap](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageDoubleMap.html) 用于在具有两个键的存储映射中存储值，作为有效删除具有公共第一个键的所有条目的优化。
- [StorageNMap](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageNMap.html) 用于在具有任意数量键的映射中存储值。

你可以在模块中包含任何这些存储结构，以引入将成为区块链状态一部分的新存储项。
你选择实现的存储项类型完全取决于你希望在运行时逻辑上下文中如何使用这些信息。

## 简单的存储值

你可以将 `StorageValue` 存储项用于运行时视为单个单元的值。
例如，你应该将此类型的存储用于以下常见用例：

- 单个基元值
- 单个 `struct` 数据类型对象
- 单个相关项目集合

如果你将此类型的存储用于项目列表，则应注意存储的列表的大小。
大型列表和 `struct` 会产生存储成本，并且在运行时迭代大型列表或 `struct` 会影响网络性能或完全停止块的生成。
如果迭代存储超过块生成时间并且你的项目是 [平行链](/reference/glossary/#parachain)，则区块链将停止生成块并停止运行。

请参阅 [StorageValue](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageValue.html#required-methods) 文档，了解 StorageValue 公开的完整方法列表。

## 单键存储映射

映射数据结构非常适合管理其元素将被随机访问的项目集，而不是按顺序完整地迭代它们。
Substrate 中的单键存储映射类似于传统的 [哈希映射](https://en.wikipedia.org/wiki/Hash_table)，具有键值映射以执行随机查找。
为了提供灵活性和控制，Substrate 允许你选择要用来生成映射键的哈希算法。
例如，如果映射存储敏感数据，你可能希望使用加密强度更高的哈希算法来生成密钥，而不是使用性能更好但加密属性较弱的哈希算法。
有关为要使用的映射选择哈希算法的更多信息，请参阅 [哈希算法](#hashing-algorithms)。

请参阅 [StorageMap](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageMap.html#required-methods) 文档，了解 StorageMap 公开的完整方法列表。

## 双键存储映射

[DoubleStorageMap](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageDoubleMap.html) 存储项类似于单键存储映射，只是它们包含两个键。
使用这种类型的存储结构对于使用公共键查询值很有用。

## 多键存储映射

[StorageNMap](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageNMap.html) 存储结构也类似于单键和双键存储映射，但使你能够定义任意数量的键。
要在 `StorageNMap` 结构中指定键，必须在声明 `StorageNMap` 时向 Key 类型参数提供包含 `NMapKey` 结构的元组。

有关在声明此类型存储结构时使用的语法的更多详细信息，请参阅 [StorageNMap 文档](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageNMap.html)。

## 迭代存储映射

你可以使用映射键和值来迭代 Substrate 存储映射。
但是，务必记住，映射通常用于跟踪无界或非常大的数据集，例如帐户和余额。
迭代大型数据集可能会消耗你用于生成块的有限资源的大量资源。
例如，如果迭代数据集所需的时间超过为生成块分配的最大时间，则运行时可能会停止生成新块，从而阻止链的进度。
此外，访问存储映射中元素所需的数据库读取次数远远超过访问列表中元素所需的数据库读取次数。
因此，迭代存储映射中的元素比读取列表中的元素的成本要高得多（就性能和执行时间而言）。

考虑到相对成本，通常最好避免在运行时迭代存储映射。
但是，关于如何使用 Substrate 存储功能没有严格的规则，最终，由你决定为你的应用程序访问运行时存储的最佳方法。

Substrate 提供以下方法，使你能够迭代存储映射：

| 方法        | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `iter()`      | 按任意顺序枚举映射中的所有元素。如果你在执行此操作时更改映射，则会得到未定义的结果。有关更多信息，请参阅 [`IterableStorageMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.iter)、[`IterableStorageDoubleMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.iter) 或 [`IterableStorageNMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageNMap.html#tymethod.iter)。                                   |
| `drain()`     | 删除映射中的所有元素并按任意顺序迭代它们。如果你在此过程中向映射添加元素，则会得到未定义的结果。有关更多信息，请参阅 [`IterableStorageMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.drain)、[`IterableStorageDoubleMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.drain)、[`IterableStorageNMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageNMap.html#tymethod.drain)。 |
| `translate()` | 按任意顺序转换映射的所有元素。要从映射中删除元素，请从转换函数返回 `None`。有关更多信息，请参阅 [`IterableStorageMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.translate)、[`IterableStorageDoubleMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.translate)、[`IterableStorageNMap`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.IterableStorageNMap.html#tymethod.translate)。            |

## 声明存储项

你可以在任何基于 FRAME 的模块中使用 [`#[pallet::storage]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#storage-palletstorage-optional) 属性宏来创建运行时存储项。
以下示例说明了如何声明不同类型的存储项。

### 单个存储值

```rust
#[pallet::storage]
type SomePrivateValue<T> = StorageValue<
    _,
    u32,
    ValueQuery
>;

#[pallet::storage]
#[pallet::getter(fn some_primitive_value)]
pub(super) type SomePrimitiveValue<T> = StorageValue<_, u32, ValueQuery>;

#[pallet::storage]
pub(super) type SomeComplexValue<T: Config> = StorageValue<_, T::AccountId, ValueQuery>;
```

### 单键存储映射

```rust
#[pallet::storage]
#[pallet::getter(fn some_map)]
pub(super) type SomeMap<T: Config> = StorageMap<
    _,
    Blake2_128Concat, T::AccountId,
    u32,
    ValueQuery
>;
```

### 双键存储映射

```rust
#[pallet::storage]
pub(super) type SomeDoubleMap<T: Config> = StorageDoubleMap<
    _,
    Blake2_128Concat, u32,
    Blake2_128Concat, T::AccountId,
    u32,
    ValueQuery
>;
```

### 多键存储映射

```rust
#[pallet::storage]
#[pallet::getter(fn some_nmap)]
pub(super) type SomeNMap<T: Config> = StorageNMap<
    _,
    (
        NMapKey<Blake2_128Concat, u32>,
        NMapKey<Blake2_128Concat, T::AccountId>,
        NMapKey<Twox64Concat, u32>,
    ),
    u32,
    ValueQuery,
>;
```

请注意，映射的存储项指定了将[使用的哈希算法](#hashing-algorithms)。

### 处理查询返回值

声明存储项时，如果指定密钥的存储中没有值，则可以指定查询如何处理返回值。
在存储声明中，你指定以下内容：

- [`OptionQuery`](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.OptionQuery.html) 用于从存储中查询可选值，如果存储包含值则返回 `Some`，如果存储中没有值则返回 `None`。
- [`ResultQuery`](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.ResultQuery.html) 用于从存储中查询结果值，如果存储中没有值则返回错误。

- [`ValueQuery`](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.ValueQuery.html) 用于从存储中查询值并返回该值。
  你还可以使用 `ValueQuery` 来返回默认值（如果你已为存储项配置了特定默认值）或返回使用 `OnEmpty` 泛型配置的值。

### 可见性

在上例中，除了 `SomePrivateValue` 之外，所有存储项都通过 `pub` 关键字公开。
区块链存储始终可以从运行时_外部_公开查看。
Substrate 存储项的可见性只会影响运行时_内部_的其他模块是否能够访问存储项。

### Getter 方法

`#[pallet::getter(..)]` 宏提供了一个可选的 `get` 扩展，可用于为包含该存储项的模块上的存储项实现 getter 方法。
此扩展将所需的 getter 函数名称作为参数。
如果你省略此可选扩展，则可以访问存储项值，但你将无法通过在模块上实现的 getter 方法来访问它；相反，你需要使用[存储项的 `get` 方法](#methods)。

可选的 `getter` 扩展只会影响从 Substrate 代码_内部_访问存储项的方式——你将始终能够[查询运行时的存储](/build/runtime-storage#Querying-Storage) 以获取存储项的值。

以下是一个为名为 `SomeValue` 的 Storage Value 实现名为 `some_value` 的 getter 方法的示例。
此模块现在除了 `SomeValue::get()` 方法外，还可以访问 `Self::some_value()` 方法：

```rust
#[pallet::storage]
#[pallet::getter(fn some_value)]
pub(super) type SomeValue = StorageValue<_, u64, ValueQuery>;
```

### 默认值

Substrate 允许你指定在未设置存储项的值时返回的默认值。
尽管默认值实际上**不会**占用运行时存储，但运行时逻辑会在执行期间看到此值。

以下是指定存储中默认值的示例：

```rust
#[pallet::type_value]
pub(super) fn MyDefault<T: Config>() -> T::Balance { 3.into() }
#[pallet::storage]
pub(super) type MyStorageValue<T: Config> =
    StorageValue<Value = T::Balance, QueryKind = ValueQuery, OnEmpty = MyDefault<T>>;
```

请注意，为了提高每个存储字段的清晰度，上述语法是声明存储项的非缩写版本。

## 访问存储项

使用 Substrate 构建的区块链公开了一个远程过程调用 (RPC) 服务器，可用于查询运行时存储。你可以使用 [Polkadot JS](https://polkadot.js.org/) 等软件库轻松地从你的代码中与 RPC 服务器交互并访问存储项。Polkadot JS 团队还维护了 [Polkadot Apps UI](https://polkadot.js.org/apps)，这是一个功能齐全的 Web 应用程序，用于与基于 Substrate 的区块链交互，包括查询存储。

## 哈希算法

Substrate 中存储映射的一个新颖特性是它们允许开发者指定将用来生成映射键的哈希算法。
用于封装哈希逻辑的 Rust 对象称为“哈希器”。
总的来说，Substrate 开发者可用的哈希器可以用两种方式来描述：（1）它们是否是加密的；以及（2）它们是否产生透明的输出。

为了完整起见，下面描述了非透明哈希算法的特性，但请记住，对于基于 FRAME 的区块链，任何不产生透明输出的哈希器都已被弃用。

### 加密哈希算法

加密哈希算法使我们能够构建工具，这些工具使操纵哈希算法的输入以影响其输出变得极其困难。
例如，即使输入是 1 到 10 的数字，加密哈希算法也会产生广泛的输出分布。
当用户能够影响存储映射的键时，务必使用加密哈希算法。
否则会创建一个攻击媒介，使恶意行为者很容易降低区块链网络的性能。
应该使用加密哈希算法来生成其密钥的映射示例是用于跟踪帐户余额的映射。
在这种情况下，务必使用加密哈希算法，这样攻击者就无法用许多小的转账到连续的帐户编号来轰炸你的系统。
如果没有合适的加密哈希算法，这将创建一个在性能方面会受到影响的不平衡存储结构。在[常见 Substrate 哈希器](#common-substrate-hashers) 中阅读有关 Substrate 中常见哈希器的更多信息。

加密哈希算法比其非加密对应算法更复杂且资源密集型，这就是为什么运行时工程师必须了解其适当用途才能最好地利用 Substrate 提供的灵活性的原因。

### 透明哈希算法

透明哈希算法是一种易于发现和验证用来生成给定输出的输入的算法。在 Substrate 中，通过将算法的输入与其输出连接起来来使哈希算法变得透明。这使得用户可以轻松检索密钥的原始未哈希值并根据需要进行验证（通过重新哈希）。Substrate 的创建者已**弃用在基于 FRAME 的运行时中使用非透明哈希器**，因此此信息主要出于完整性考虑而提供。事实上，如果你想访问[可迭代映射](#iterable-storage-maps) 功能，则_必须_使用透明哈希算法。

### 常见的 Substrate 哈希器

此表列出了 Substrate 中使用的一些常见哈希器，并指出了哪些是加密的，哪些是透明的：

| 哈希器                                                                                                        | 加密的 | 透明的 |
| ------------------------------------------------------------------------------------------------------------- | ------------- | ----------- |
| [Blake2 128 Concat](https://paritytech.github.io/substrate/master/frame_support/struct.Blake2_128Concat.html) | X             | X           |
| [TwoX 64 Concat](https://paritytech.github.io/substrate/master/frame_support/struct.Twox64Concat.html)        |               | X           |
| [Identity](https://paritytech.github.io/substrate/master/frame_support/struct.Identity.html)                  |               | X           |

Identity 哈希器封装了一个哈希算法，该算法的输出等于其输入（恒等函数）。
此类型的哈希器应仅在起始密钥已经是加密哈希时使用。

## 接下来的步骤

查看一些涵盖存储各种主题的指南：

- [操作方法：创建存储结构](/reference/how-to-guides/pallet-design/create-a-storage-structure)
- [StorageValue](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageValue.html)
- [StorageMap](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageMap.html)
- [StorageDoubleMap](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageDoubleMap.html)
- [StorageNMap](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageNMap.html)
