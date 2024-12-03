---
title: 计算费用
description:
keywords:
  - 权重
  - 费用
  - 运行时
  - FRAME v1
---

本指南逐步介绍了为运行时的 `pallet_transaction_payment` 实现自定义 `WeightToFee` 的过程。
费用分为三个部分：

- **字节费用** - 与交易长度（以字节为单位）成比例的费用。
  比例常数是交易支付 pallet 中的一个参数。
- **权重费用** - 根据 [交易权重](/build/tx-weights-fees) 计算的费用。
  转换不需要是线性的，尽管通常是线性的。
  相同的转换函数应用于运行时中所有 pallet 的所有交易。
- **费用乘数** - 计算费用的乘数，随着链的进展而变化。

FRAME 提供了 [交易支付 pallet](https://paritytech.github.io/substrate/master/pallet_transaction_payment/index.html) 用于计算和收取执行交易的费用。
修改费用计算方式以更准确地收取费用可能很有用。

## 目标

自定义 `WeightToFee` 以修改运行时费用计算方式。

## 用例

修改费用计算方式，而不是使用 [`IdentityFee`](https://paritytech.github.io/substrate/master/frame_support/weights/struct.IdentityFee.html)，它将一个费用的单位映射到一个权重的单位。

## 步骤

### 1. 编写 `LinearWeightToFee` 结构体

在 `runtime/src/lib.rs` 中，创建名为 `LinearWeightToFee` 的结构体，该结构体实现 [`WeightToFeePolynomial`](https://paritytech.github.io/substrate/master/frame_support/weights/trait.WeightToFeePolynomial.html)。
它必须返回一个 `WeightToFeeCoefficient` 整数的小向量。

`runtime/src/lib.rs`

```rust
pub struct LinearWeightToFee<C>(sp_std::marker::PhantomData<C>);

impl<C> WeightToFeePolynomial for LinearWeightToFee<C>
where
	C: Get<Balance>,
{
	type Balance = Balance;

	fn polynomial() -> WeightToFeeCoefficients<Self::Balance> {
		let coefficient = WeightToFeeCoefficient {
			coeff_integer: C::get(),
			coeff_frac: Perbill::zero(),
			negative: false,
			degree: 1,
		};

		smallvec!(coefficient)
	}
}
```

### 2. 在运行时中配置 `pallet_transaction_payment`

将调度权重 `type WeightToFee` 转换为可计费费用 `LinearWeightToFee`（替换 `IdentityFee<Balance>;`）：

`runtime/src/lib.rs`

```rust
parameter_types! {
    // 与 LinearWeightToFee 转换一起使用。
	pub const FeeWeightRatio: u128 = 1_000;
	// 建立字节费用。它用于所有配置中。
	pub const TransactionByteFee: u128 = 1;
}

impl pallet_transaction_payment::Config for Runtime {
	type OnChargeTransaction = CurrencyAdapter<Balances, ()>;
	type TransactionByteFee = TransactionByteFee;

	// 将调度权重转换为可计费费用。
	type WeightToFee = LinearWeightToFee<FeeWeightRatio>;

	type FeeMultiplierUpdate = ();
}
```

## 相关资料

- [权重](/reference/glossary#weight)
- [添加基准测试](/reference/how-to-guides/weights/add-benchmarks/)
- [使用自定义权重](/reference/how-to-guides/weights/use-custom-weights)
- [交易权重和费用](/build/tx-weights-fees)
- [`WeightToFeeCoefficients`](https://paritytech.github.io/substrate/master/frame_support/weights/type.WeightToFeeCoefficients.html)
- [`WeightToFeePolynomial`](https://paritytech.github.io/substrate/master/frame_support/weights/trait.WeightToFeePolynomial.html)
