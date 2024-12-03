---
title: 选择 Collator
description:
keywords:
  - collator
  - parachain
  - cumulus
  - 设计
  - 选择
  - 经济
  - 博弈论
---

本指南对准备在生产环境中启动 Parachain 的团队很有用，它探讨了 collator 选择中的不同选项。

为 Parachain 网络设置 collator 至关重要，以确保存在_一些_中立的 collator 来防止审查——但不一定是多数。
避免 collator 过多也很重要，因为它们可能会减慢网络速度。
本指南逐步介绍了在设计 Parachain 网络时需要考虑的事项。

## Collator 选择

您可以自由选择 collator 选择方法。
常见的方法包括权益投票或通过委员会或其他来源（例如民主）直接分配 collator。
在这两种情况下，创建一个 pallet 来实现最符合您需求的逻辑。

### 权益投票

Cumulus [`collator-selection` pallet](https://github.com/paritytech/polkadot-sdk/blob/master/cumulus/pallets/collator-selection/src/lib.rs) 是实现权益投票以选择 collator 的一个实用示例。

### 使用链上治理

实现一个特殊的来源，允许该来源的成员成为 collator。使用民主 pallet 来选举这些成员，并在专门用于处理 collator 选择的 pallet 中定义它们：

```rust
    /// 此 pallet 的配置特性。
	#[pallet::config]
	pub trait Config: frame_system::Config {
        // --snip-- //
        type MySpecialOrigin: EnsureOrigin<Self::RuntimeOrigin>;
    }
    // --snip-- //
    #[pallet::call]
	impl<T: Config> Pallet<T> {
		/// 某些设置 collator 的可调度函数。
		#[pallet::weight(some_weight)]
		pub fn set_collator( origin: OriginFor<T>) -> DispatchResultWithPostInfo {
            T::MySpecialOrigin::ensure_origin(origin)?;
            // --snip-- //
        }
```

还有不同的方法可以为 collator 提供激励。
查看 [此示例](https://github.com/PureStake/moonbeam/blob/master/pallets/parachain-staking/src/lib.rs) 以了解如何操作。

## 示例

- 使用交易费用通过激励来选择 collator 的 [Cumulus 实现](https://github.com/paritytech/polkadot-sdk/blob/master/cumulus/pallets/collator-selection/src/lib.rs)。
- 使用通货膨胀货币政策抵押方案选择 collator 的 [Moonbeam 实现](https://github.com/PureStake/moonbeam/blob/master/pallets/parachain-staking/src/lib.rs)。

## 资源

- [Parachain DevOps 最佳实践](https://gist.github.com/lovelaced/cddc1c7234b883ee37e71cf4a1d63cac)
