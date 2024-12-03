---
title: 整合随机性
description: 链上随机性技术和工具详解。
keywords:
  - 模块设计
  - 中级
  - 运行时
  - 随机性
---

随机性在计算机程序中用于许多应用程序。例如，游戏应用程序、NFT 创建和选择区块作者都需要一定程度的随机性。

在确定性计算机中很难获得真正的随机性。
在区块链的上下文中尤其如此，当网络中的所有节点都必须就链的状态达成一致时。
FRAME 为运行时工程师提供了一个 [链上随机性](/build/randomness/) 源，使用 [Randomness 特性](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html)。

本指南解释了如何通过使用 `random` 方法和 nonce 作为主题来使用 FRAME 的 Randomness 特性。
本指南还说明了如何通过将 `RandomCollectiveFlip` 模块分配给公开“random”类型的模块的配置特性来增加随机值的熵。

## 导入 `Randomness`

1. 在您想要使用的模块中，从 `frame_support` 导入 [`Randomness`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html) 特性：

   ```rust
   use frame_support::traits::Randomness;
   ```

1. 将其包含在模块的配置特性中：

   ```rust
   #[pallet::config]
   pub trait frame_system::Config {
   	type MyRandomness: Randomness<Self::Hash, BlockNumberFor<Self>>;
   }
   ```

   请注意，`Randomness` 特性指定了 `Output` 和 `BlockNumber` 类型的泛型返回。
   在您的模块中使用 `frame_system` 中的 [`BlockNumber`](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html#associatedtype.BlockNumber)
   和 [`Hash`](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html#associatedtype.Hash) 来满足该特性的要求。

   正如 [此特性的文档](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html) 中所述，充其量，此特性可以为您提供很久以前难以预测但最近已变得易于预测的随机性。
   在评估您对它的使用时，请记住这一点。

## 创建 nonce 并将其用于您的随机性实现

使用 nonce 作为 `frame_support::traits::Randomness::random(subject: &[u8])` 方法的主题。

1. 在模块中包含 nonce 分为两个步骤：

   - **创建 `Nonce` 存储项。** 存储项可以是 `u32` 或 `u64` 类型。

   - **创建私有 nonce 函数。** 此函数在每次使用时都会递增 nonce。

   `increment_nonce()` 私有函数可以这样实现，它既返回又更新 nonce。
   例如：

   ```rust
   fn get_and_increment_nonce() -> Vec<u8> {
   	let nonce = Nonce::<T>::get();
   	Nonce::<T>::put(nonce.wrapping_add(1));
   	nonce.encode()
   }
   ```

   要了解有关 wrapping 和 encoding 方法的更多信息，请参阅 Rust 文档中的 [`wrapping_add`](https://doc.rust-lang.org/std/intrinsics/fn.wrapping_add.html) 和 [`encode`](https://paritytech.github.io/substrate/master/frame_support/dispatch/trait.Encode.html#method.encode)。

1. 在可调度函数中使用 Randomness。

   使用 nonce，您可以调用 `Randomness` 公开的 `random()` 方法。
   下面的代码片段是一个模拟示例，假设已实现相关的事件和存储项：

   ```rust
   #[pallet::weight(100)]
   pub fn create_unique(
   	origin: OriginFor<T>)
   	-> DispatchResultWithPostInfo {
   	// 调用此可调度函数的帐户。
   	let sender = ensure_signed(origin)?;
   		// 随机值。
   		let nonce = Self::get_and_increment_nonce();
   		let (randomValue, _) = T::MyRandomness::random(&nonce);
   	// 将随机值写入存储。
   	<MyStorageItem<T>>::put(randomValue);
   	Self::deposit_event(Event::UniqueCreated(randomValue));
   }
   ```

1. 更新模块的运行时实现。

   因为您已向模块的配置特性添加了一个类型，所以 `Config` 为进一步增强 `Randomness` 特性派生的随机性提供了机会。
   这是通过使用 [Randomness Collective Flip 模块](https://paritytech.github.io/substrate/master/pallet_insecure_randomness_collective_flip/index.html) 来实现的。

   将此模块与 `Randomness` 特性一起使用将大大提高 `random()` 处理的熵。

   在 `runtime/src/lib.rs` 中，假设 `pallet_random_collective_flip` 在 `construct_runtime` 中被实例化为 `RandomCollectiveFlip`，请按以下方式指定公开的类型：

   ```rust
   impl my_pallet::Config for Runtime{
   	type Event;
   	type MyRandomness = RandomCollectiveFlip;
   }
   ```

## 示例

- [BABE 中使用的随机性](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/babe/src/randomness.rs)
- [FRAME 的 Lottery 模块](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/lottery/src/lib.rs#L120)

## 相关资料

- [可验证随机函数](https://en.wikipedia.org/wiki/Verifiable_random_function)
