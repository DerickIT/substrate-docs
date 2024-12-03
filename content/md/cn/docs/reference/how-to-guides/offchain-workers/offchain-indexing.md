---
title: 链下索引
description:
keywords:
  - 链下工作者
  - ocw
  - 索引
---

<div class="warning">
	 请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_offchain_workers/index.html">Rust文档</a> 获取有关此主题的最新文档。
</div>

本指南将逐步引导您如何将数据从外部指令传递到链下工作者，而无需写入存储。

有时，链上外部指令需要将数据传递到具有可预测写入行为的链下工作者。
此数据可以写入链上存储供链下工作者读取，但这可能会给区块链带来巨大的成本。
另一种将数据从链上世界传递到链下世界的方法是通过**链下索引**将其保存在节点的本地存储中。

链下索引是从链上外部指令调用的，这意味着本地写入的数据预计在网络中的所有节点上都是一致的。

另一个用例是需要将大量数据存储在链上以供链下工作者处理。
这将过于昂贵。
解决方案是使用链下索引将该数据的哈希值存储在链上，并将相应原始数据存储在本地，以便链下工作者稍后读取。

请注意，当存在分叉区块时，相同的外部指令可能会运行多次。
结果是，如果使用非唯一键，数据可能会被不同的分叉区块覆盖，并且本地存储中的内容在节点之间将有所不同。
因此，开发人员在形成正确的索引键时应谨慎，以防止潜在的覆盖。

注意：为了查看链下索引功能的实际效果，请使用打开链下索引标志的 Substrate 节点运行。
例如：`./target/release/substrate-node --enable-offchain-indexing true`

## 步骤

1. 创建用于索引的唯一键。

   在模块的 `src/lib.rs` 中：

   ```rust
   const ONCHAIN_TX_KEY: &[u8] = b"my_pallet::indexing1";

   #[pallet::call]
   impl<T: Config> Pallet<T> {
   	#[pallet::weight(100)]
   	pub fn extrinsic(origin: OriginFor<T>, number: u64) -> DispatchResult {
   		let who = ensure_signed(origin)?;

   		let key = Self::derived_key(frame_system::Module::<T>::block_number());
   		// ...

   		Ok(())
   	}
   }

   impl<T: Config> Pallet<T> {
   	fn derived_key(block_number: T::BlockNumber) -> Vec<u8> {
   		block_number.using_encoded(|encoded_bn| {
   			ONCHAIN_TX_KEY.clone().into_iter()
   				.chain(b"/".into_iter())
   				.chain(encoded_bn)
   				.copied()
   				.collect::<Vec<u8>>()
   		})
   	}
   }
   ```

   在上面的代码中，在一个常规的外部指令中，调用 `Self::derived_key()` 辅助方法来生成稍后用于索引的键。
   它将预定义的前缀与当前编码的区块编号连接起来，并将其作为字节向量返回。

1. 定义索引数据并使用链下索引保存它：

   ```rust
   use sp_io::offchain_index;
   const ONCHAIN_TX_KEY: &[u8] = b"my_pallet::indexing1";

   #[derive(Debug, Deserialize, Encode, Decode, Default)]
   struct IndexingData(Vec<u8>, u64);

   #[pallet::call]
   impl<T: Config> Pallet<T> {
   	#[pallet::weight(100)]
   	pub fn extrinsic(origin: OriginFor<T>, number: u64) -> DispatchResult {
   		let who = ensure_signed(origin)?;

   		let key = Self::derived_key(frame_system::Module::<T>::block_number());
   		let data = IndexingData(b"submit_number_unsigned".to_vec(), number);
   		offchain_index::set(&key, &data.encode());
   		Ok(())
   	}
   }

   impl<T: Config> Pallet<T> {
   	// -- 为简洁起见跳过 --
   }
   ```

   索引数据可以是任何可以由 `Encode`、`Decode` 和 `Deserialize` 特性绑定的数据类型。
   在上面的代码中，使用 [`offchain_index::set()`](https://paritytech.github.io/substrate/master/sp_io/offchain_index/fn.set.html) 方法通过链下索引存储数据。

1. 使用 `offchain_worker` 钩子方法在链下工作者的数据库中读取数据：

   ```rust
   use sp_runtime::offchain::StorageValueRef;

   #[derive(Debug, Deserialize, Encode, Decode, Default)]
   struct IndexingData(Vec<u8>, u64);

   fn offchain_worker(block_number: T::BlockNumber) {
   	// 读取链下索引值。这与从
   	// ocw 本地存储读取完全相同。
   	let key = Self::derived_key(block_number);
   	let storage_ref = StorageValueRef::persistent(&key);

   	if let Ok(Some(data)) = storage_ref.get::<IndexingData>() {
   		debug::info!("本地存储数据：{:?}, {:?}",
   			str::from_utf8(&data.0).unwrap_or("error"), data.1);
   	} else {
   		debug::info!("从本地存储读取错误。");
   	}

   	// -- 省略 --
   }
   ```

   有了这个，链下工作者就可以从节点的本地存储中读取相应的数据。
   [链下本地存储](/reference/how-to-guides/offchain-workers/offchain-local-storage/) 操作指南解释了如何做到这一点。

## 相关资料

- [链下操作](/learn/offchain-operations/)
- [链下本地存储](/reference/how-to-guides/offchain-workers/offchain-local-storage/)
