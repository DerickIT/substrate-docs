---
title: 交易、权重和费用
description: 描述如何通过 Substrate 权重系统和交易费用的计算来计算执行交易所需的资源。
keywords:
---

当执行交易或在链上存储数据时，活动会更改链的状态并消耗区块链资源。
由于区块链可用的资源有限，因此管理链上操作如何消耗这些资源非常重要。
除了实际限制（例如存储容量）外，区块链资源还代表恶意用户潜在的攻击媒介。
例如，恶意用户可能会尝试使用消息来过载网络，以阻止网络生成新的块。
为了保护区块链资源免受耗尽或过载，你需要管理如何提供和使用这些资源。
需要注意的资源包括：

- 内存使用情况
- 存储输入和输出
- 计算
- 事务和块大小
- 状态数据库大小

Substrate 为块作者提供了多种方法来管理对资源的访问，并防止链的各个组件消耗过多的任何单一资源。
块作者可用的两种最重要的机制是**权重**和**交易费用**。

[权重](/reference/glossary/#weight) 用于管理验证块所需的时间。
通常，权重用于表征执行块主体中的调用所需的时间。
通过控制块可以消耗的执行时间，权重可以限制存储输入和输出以及计算。

块允许的某些权重在块的初始化和完成过程中消耗。
权重还可以用于执行强制固有的外部函数调用。
为了帮助确保块不会消耗过多的执行时间（并防止恶意用户使用不必要的调用来过载系统），权重与**交易费用**一起使用。

交易费用提供了一种经济激励，以限制执行时间、计算和执行操作所需的调用次数。
交易费用还用于使区块链在经济上可持续，因为它们通常适用于用户发起的交易，并在执行交易请求之前扣除。

## 如何计算费用

交易的最终费用是使用以下参数计算的：

- _基本费用_：这是用户为交易支付的最低金额。它在运行时声明为**基本权重**，并使用 `WeightToFee` 转换为费用。

- _权重费用_：与交易消耗的执行时间（输入和输出以及计算）成比例的费用。

- _长度费用_：与交易的编码长度成比例的费用。

- _小费_：一个可选的小费，用于提高交易的优先级，使其更有可能被交易队列包含。

基本费用和成比例的权重和长度费用构成**包含费用**。
包含费用是交易包含在块中必须具备的最低费用。

## 使用交易支付模块

[交易支付模块](https://paritytech.github.io/substrate/master/pallet_transaction_payment/index.html) 提供了计算包含费用的基本逻辑。

你还可以使用交易支付模块来：

- 使用 `Config::WeightToFee` 将权重值转换为基于货币类型的可扣除费用。

- 通过定义一个乘数来更新下一个块的费用，该乘数基于上一个块结束时链的最终状态，使用 `Config::FeeMultiplierUpdate`。

- 使用 `Config::OnChargeTransaction` 管理交易费用的提取、退款和存款。

你可以在 [交易支付](https://paritytech.github.io/substrate/master/pallet_transaction_payment/index.html) 文档中了解有关这些配置特性的更多信息。

你应该注意，交易费用会在执行交易之前提取。
执行交易后，可以调整交易权重以反映交易使用的实际资源。
如果交易使用的资源少于预期，则会更正交易费用，并存入调整后的交易费用。

### 仔细研究包含费用

计算最终费用的公式如下所示：

```
inclusion_fee = base_fee + length_fee + [targeted_fee_adjustment * weight_fee];
final_fee = inclusion_fee + tip;
```

在此公式中，`targeted_fee_adjustment` 是一个乘数，可以根据网络的拥塞情况调整最终费用。

- 从基本权重导出的 `base_fee` 涵盖了包含开销，例如签名验证。

- `length_fee` 是一个每字节费用，乘以编码外部函数的长度。

- `weight_fee` 费用是使用两个参数计算的：

  在运行时声明的并应用于所有外部函数的 `ExtrinsicBaseWeight`。

  说明外部函数复杂性的 `#[pallet::weight]` 注释。

要将权重转换为货币，运行时必须定义一个 `WeightToFee` 结构，该结构实现一个转换函数 `Convert<Weight,Balance>`。

请注意，外部函数发送者会在调用外部函数之前被收取包含费用。即使交易在执行时失败，费用也会从发送者的余额中扣除。

### 余额不足的帐户

如果帐户的余额不足以支付包含费用并保持活动状态（即，足以支付包含费用并保持最低**存在存款**），则你应确保取消交易，以便不会扣除任何费用并且交易不会开始执行。

Substrate 不会强制执行此回滚行为。
但是，这种情况很少发生，因为交易队列和块制作逻辑会在将外部函数添加到块之前执行检查以防止这种情况。

### 费用乘数

包含费用公式始终为相同的输入产生相同的费用。
但是，权重可能是动态的，并且根据 [`WeightToFee`](https://paritytech.github.io/substrate/master/pallet_transaction_payment/pallet/trait.Config.html#associatedtype.WeightToFee) 的定义方式，最终费用可能包含一定程度的可变性。

为了解决这种可变性，交易支付模块提供了 [`FeeMultiplierUpdate`](https://paritytech.github.io/substrate/master/pallet_transaction_payment/pallet/trait.Config.html#associatedtype.FeeMultiplierUpdate) 可配置参数。

默认更新函数的灵感来自 Polkadot 网络，并实现了一种目标调整，其中定义了块权重的目标饱和度级别。
如果前一个块的饱和度更高，则费用会略微增加。
同样，如果前一个块的事务少于目标，则费用会减少少量。
有关费用乘数调整的更多信息，请参阅 [Web3 研究页面](https://research.web3.foundation/Polkadot/overview/token-economics#relay-chain-transaction-fees-and-per-block-transaction-limits)。

## 具有特殊要求的交易

包含费用必须在执行之前可计算，因此只能表示固定逻辑。
某些交易需要使用其他策略来限制资源。
例如：

- 债券是一种费用类型，可能在某个链上事件后返回或被削减。

  例如，你可能希望要求用户放置债券才能参与投票。然后，可以在公投结束时返还债券，或者如果投票者试图进行恶意行为，则可以削减债券。

- 存款是以后可能退还的费用。

  例如，你可能要求用户支付存款才能执行使用存储的操作。如果后续操作释放了存储空间，则可以退还用户的存款。

* 燃烧操作用于根据其内部逻辑支付交易。

  例如，如果交易创建新的存储项以支付增加的状态大小，则交易可能会燃烧发送者的资金。

- 限制使你能够对某些操作强制执行常量或可配置的限制。

  例如，默认的抵押模块只允许提名者提名 16 个验证者，以限制验证者选举过程的复杂性。

重要的是要注意，如果你查询链的交易费用，它只会返回包含费用。

## 默认权重注释

Substrate 中的所有可调度函数都必须指定权重。实现此目的的方法是使用基于注释的系统，该系统允许你组合数据库读/写权重的固定值和/或基于基准测试的固定值。最基本的示例如下所示：

```rust
#[pallet::weight(100_000)]
fn my_dispatchable() {
    // ...
}
```

请注意，`ExtrinsicBaseWeight` 会自动添加到声明的权重中，以说明将空外部函数包含到块中的成本。

### 权重和数据库读/写操作

为了使权重注释独立于已部署的数据库后端，它们被定义为常量，然后在表达可调度执行的数据库访问时在注释中使用：

```rust
#[pallet::weight(T::DbWeight::get().reads_writes(1, 2) + 20_000)]
fn my_dispatchable() {
    // ...
}
```

此可调度执行一次数据库读取和两次数据库写入，以及其他增加额外 20,000 的操作。
数据库访问通常是在每次访问在 `#[pallet::storage]` 块内声明的值时进行的。
但是，只计算唯一的访问次数，因为访问值后会将其缓存，再次访问它不会导致数据库操作。
也就是说：

- 同一值的多次读取计为一次读取。
- 同一值的多次写入计为一次写入。
- 同一值的多次读取，然后写入该值，计为一次读取和一次写入。
- 写入后跟读取只计为一次写入。

### 调度类

调度分为三类：

- `Normal`
- `Operational`
- `Mandatory`

如果调度在权重注释中未定义为 `Operational` 或 `Mandatory`，则默认情况下调度被标识为 `Normal`。
你可以这样指定可调度使用其他类：

```rust
#[pallet::dispatch((DispatchClass::Operational))]fn my_dispatchable() {
    // ...
}
```

此元组表示法还允许你指定一个最终参数，该参数决定是否根据注释的权重向用户收费。
如果你没有另行指定，则假定为 `Pays::Yes`：

```rust
#[pallet::dispatch(DispatchClass::Normal, Pays::No)]
fn my_dispatchable() {
    // ...
}
```

#### 普通调度

此类中的调度表示普通用户触发的交易。
这些类型的调度只消耗块总权重限制的一部分。
有关块中可以为普通调度消耗的最大部分的信息，请参阅 [`AvailableBlockRatio`](https://paritytech.github.io/substrate/master/frame_system/limits/struct.BlockLength.html#method.max_with_normal_ratio)。
普通调度被发送到[交易池](/reference/glossary/#transaction-pool)。

#### 操作调度

与表示网络功能_使用_的普通调度不同，操作调度是_提供_网络功能的调度。
操作调度可以消耗块的整个权重限制。
它们不受 [`AvailableBlockRatio`](https://paritytech.github.io/substrate/master/frame_system/limits/struct.BlockLength.html#method.max_with_normal_ratio) 的限制。
此类中的调度具有最高优先级，并且无需支付 `length_fee`。

#### 强制调度

即使强制调度导致块超过其权重限制，也会将其包含在块中。
你只能将强制调度类用于由块作者提交的[固有事务](/reference/glossary/#inherent-transactions)。
此调度类旨在表示块验证过程的一部分的函数。
由于这些调度始终包含在块中，而不管函数权重如何，因此务必确保验证过程可以防止恶意节点滥用该函数来创建有效但重量过大的块。
你通常可以通过确保以下几点来实现此目的：

- 执行的操作始终很轻。
- 操作只能包含在一个块中一次。

为了使恶意节点更难以滥用强制调度，它们不能包含在返回错误的块中。
此调度类旨在满足这样的假设：允许创建重量过大的块比根本不允许创建任何块更好。

### 动态权重

除了纯粹的固定权重和常量外，权重计算还可以考虑可调度的输入参数。
权重应该可以通过一些基本的算术运算从输入参数中轻松计算出来：

```rust
use frame_support:: {
    dispatch:: {
        DispatchClass::Normal,
        Pays::Yes,
    },
   weights::Weight,
};

#[pallet::weight(FunctionOf(
  |args: (&Vec<User>,)| args.0.len().saturating_mul(10_000),
  )
]
fn handle_users(origin, calls: Vec<User>) {
    // 对每个用户执行某些操作
}
```

## 调度后权重校正

根据执行逻辑，可调度函数消耗的权重可能少于调度前规定的权重。
要更正权重，函数声明不同的返回类型并返回其实际权重：

```rust
#[pallet::weight(10_000 + 500_000_000)]
fn expensive_or_cheap(input: u64) -> DispatchResultWithPostInfo {
    let was_heavy = do_calculation(input);

    if (was_heavy) {
        // None 表示“没有更正”来自权重注释。
        Ok(None.into())
    } else {
        // 返回实际消耗的权重。
        Ok(Some(10_000).into())
    }
}
```

## 自定义费用

你还可以通过自定义权重函数或包含费用函数来定义自定义费用系统。

### 自定义权重

你可以使用 [weights](https://paritytech.github.io/substrate/master/frame_support/weights/index.html) 模块创建自定义权重计算类型，而不是使用默认权重注释。
自定义权重计算类型必须实现以下特性：

- `WeighData<T>` 用于确定调度的权重。
- `ClassifyDispatch<T>` 用于确定调度的类别。
- `Pays<T>` 用于确定调度发送者是否支付费用。

然后，Substrate 将这三个特性的输出信息捆绑到 `DispatchInfo` 结构中，并通过为所有 `Call` 变体和不透明外部函数类型实现 `GetDispatchInfo` 来提供它。
系统和执行模块在内部使用它。

`ClassifyDispatch`、`WeighData` 和 `PaysFee` 对于 `T` 是通用的，`T` 将解析为所有调度参数的元组，除了来源。
以下示例说明了一个结构，该结构将权重计算为 `m * len(args)`，其中 `m` 是给定的乘数，`args` 是所有调度参数的连接元组。
在此示例中，如果事务的参数长度超过 100 字节，则调度类为 `Operational`，如果编码长度大于 10 字节，则将支付费用。

```rust
struct LenWeight(u32);
impl<T> WeighData<T> for LenWeight {
    fn weigh_data(&self, target: T) -> Weight {
        let multiplier = self.0;
        let encoded_len = target.encode().len() as u32;
        multiplier * encoded_len
    }
}

impl<T> ClassifyDispatch<T> for LenWeight {
    fn classify_dispatch(&self, target: T) -> DispatchClass {
        let encoded_len = target.encode().len() as u32;
        if encoded_len > 100 {
            DispatchClass::Operational
        } else {
            DispatchClass::Normal
        }
    }
}

impl<T> PaysFee<T> {
    fn pays_fee(&self, target: T) -> Pays {
        let encoded_len = target.encode().len() as u32;
        if encoded_len > 10 {
            Pays::Yes
        } else {
            Pays::No
        }
    }
}
```

权重计算函数还可以强制转换为参数的最终类型，而不是将其定义为可以编码的模糊类型。
代码大致如下所示：

```rust
struct CustomWeight;
impl WeighData<(&u32, &u64)> for CustomWeight {
    fn weigh_data(&self, target: (&u32, &u64)) -> Weight {
        ...
    }
}

// 给定调度：
#[pallet::call]
impl<T: Config<I>, I: 'static> Pallet<T, I> {
    #[pallet::weight(CustomWeight)]
    fn foo(a: u32, b: u64) { ... }
}
```

在此示例中，`CustomWeight` 只能与具有特定签名 `(u32, u64)` 的调度一起使用，而不是 `LenWeight`，`LenWeight` 可以与任何内容一起使用，因为对 `<T>` 没有任何假设。

### 自定义包含费用

以下示例说明了如何自定义包含费用。
你必须在相应的模块中配置适当的关联类型。

```rust
// 假设这是余额类型
type Balance = u64;

// 假设我们希望所有权重都具有 `100 + 2 * w` 的费用转换
struct CustomWeightToFee;
impl WeightToFee<Weight, Balance> for CustomWeightToFee {
    fn convert(w: Weight) -> Balance {
        let a = Balance::from(100);
        let b = Balance::from(2);
        let w = Balance::from(w);
        a + b * w
    }
}

parameter_types! {
    pub const ExtrinsicBaseWeight: Weight = 10_000_000;
}

impl frame_system::Config for Runtime {
    type ExtrinsicBaseWeight = ExtrinsicBaseWeight;
}

parameter_types! {
    pub const TransactionByteFee: Balance = 10;
}

impl transaction_payment::Config {
    type TransactionByteFee = TransactionByteFee;
    type WeightToFee = CustomWeightToFee;
    type FeeMultiplierUpdate = TargetedFeeAdjustment<TargetBlockFullness>;
}

struct TargetedFeeAdjustment<T>(sp_std::marker::PhantomData<T>);
impl<T: Get<Perquintill>> WeightToFee<Fixed128, Fixed128> for TargetedFeeAdjustment<T> {
    fn convert(multiplier: Fixed128) -> Fixed128 {
        // 不要更改任何内容。在此处放置任何费用更新信息。
        multiplier
    }
}
```

## 接下来的步骤

你现在知道了权重系统是什么，它如何影响交易费用计算以及如何为你的可调度调用指定权重。
下一步是确定适当的权重以说明你的可调度执行的操作。
你可以使用 Substrate **基准测试函数**和 `frame-benchmarking` 调用来使用不同的参数测试你的函数，并在最坏情况下根据经验确定正确的权重。

- [基准测试](/test/benchmark/)
- [SignedExtension](https://paritytech.github.io/substrate/master/sp_runtime/traits/trait.SignedExtension.html)
- [示例模块的自定义权重](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/weights.rs)
- [Web3 基金会研究](https://research.web3.foundation/Polkadot/overview/token-economics#relay-chain-transaction-fees-and-per-block-transaction-limits)

<!-- - [计算权重](/reference/how-to-guides/weights/) -->
