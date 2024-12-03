---
title: 链下本地存储
description:
keywords:
  - 链下工作者
  - ocw
  - 本地
  - 存储
---

<div class="warning">
	 请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_offchain_workers/index.html">Rust文档</a> 获取有关此主题的最新文档。
</div>

本指南将教你如何使用链下工作者将检索到的数据保存在本地存储中以便将来访问。

在上一节中，我们提到链下工作者（简称 **OCW**）不能直接修改区块链状态，因此它们必须提交交易才能将其计算结果保存回链上。
尽管如此，有时数据也不适合保存在链上，但仍然需要存储在某个地方以供将来访问。
这包括临时数据或中间计算，一旦计算完成即可丢弃。

在本指南中，我们将指示链下工作者将数据写入本地存储，而无需在整个区块链网络中传递数据。
引入了 **存储锁** 的概念，以便在多个 OCW 之间一致地访问值。
OCW 在每个区块生成结束时异步运行，并且它们的运行时间不受限制，因此在任何时间点都可能启动多个 OCW 实例。

本地存储 API 与其链上对应 API 类似，使用 `get`、`set` 和 `mutate` 来访问它们。
`mutate` 使用[**比较并设置模式**](https://en.wikipedia.org/wiki/Compare-and-swap)——它将内存位置的内容与给定值进行比较，并且只有在它们相同的情况下，才将该内存位置的内容修改为新的给定值。
这是作为单个原子操作完成的。
原子性保证新值是基于最新信息计算的；如果该值在此期间已被另一个线程更新，则写入将失败。

请注意，由于存储在本地存储中的值没有经过网络中的共识机制，因此它们可能会受到节点操作员的操纵。

在本操作指南中，我们将首先从充当缓存的存储中检查计算值是否存在。
如果找到缓存值，则链下工作者返回；否则，它将尝试获取锁、执行密集型计算并将结果保存到存储/缓存中。

## 步骤

1. 在模块的 `offchain_worker` 函数钩子中定义存储引用：

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
     // 创建对本地存储值的引用。
     // 由于本地存储对所有链下工作者都是通用的，因此最好
     // 使用模块名称作为我们条目的前缀。
     let storage = StorageValueRef::persistent(b"pallet::my-storage");
   }
   ```

   在上面的代码中，使用 [`StorageValueRef::persistent()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage/struct.StorageValueRef.html#method.persistent) 定义了持久性本地存储，它由 `pallet::my-storage` 键标识。
   该键是字节数组类型而不是 `str` 类型。
   此本地存储是持久化的，并在链下工作者的运行中共享。

1. 检查存储是否包含缓存值。

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
     // ...
     let storage = StorageValueRef::persistent(b"pallet::my-storage");

     if let Ok(Some(res)) = storage.get::<u64>() {
       log::info!("缓存结果：{:?}", res);
       return Ok(());
     }
   }
   ```

   使用 [`get<T: Decode>()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage/struct.StorageValueRef.html#method.get) 函数获取结果，返回 `Result<Option<T>, StorageRetrievalError>` 类型。
   我们只关心具有有效值的情况。如果是，则返回 `Ok(())`。
   请注意，我们还需要定义返回值的类型。

   使用 [`set()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage/struct.StorageValueRef.html#method.get) 写入存储，并使用 [`mutate<T, E, F>()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage/struct.StorageValueRef.html#method.mutate) 原子地读取和更改存储。

1. 如果没有有效值 (`None`) 或出现 `StorageRetrievalError`，则继续计算所需结果并获取存储锁。

   首先定义存储锁如下。

   ```rust
   const LOCK_BLOCK_EXPIRATION: u32 = 3; // 以区块数为单位
   const LOCK_TIMEOUT_EXPIRATION: u64 = 10000; // 以毫秒为单位

   fn offchain_worker(block_number: T::BlockNumber) {
     // ...
     let storage = StorageValueRef::persistent(b"pallet::my-storage");

     if let Ok(Some(res)) = storage.get::<u64>() {
       log::info!("缓存结果：{:?}", res);
       return Ok(());
     }

     // 此处进行非常密集的计算
     let val: u64 = 100 + 100;

     // 定义存储锁
     let mut lock = StorageLock::<BlockAndTime<Self>>::with_block_and_time_deadline(
       b"pallet::storage-lock",
       LOCK_BLOCK_EXPIRATION,
       Duration::from_millis(LOCK_TIMEOUT_EXPIRATION)
     );
   }
   ```

   在上面的代码片段中，使用同时指定的[区块和时间截止时间](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.with_block_and_time_deadline)定义了一个存储锁。
   此函数接收锁标识符、区块编号到期时间和时间到期时间。
   当超过指定的区块编号或时间持续时间时，上述锁就会过期。
   我们还可以使用[仅区块编号](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.with_block_deadline)或[时间持续时间](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.with_deadline)来指定到期时间段。

1. 使用[`.try_lock()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.try_lock)获取存储锁。

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
     // ...

     let mut lock = /* .... */;

     if let Ok(_guard) = lock.try_lock() {
       storage.set(&val);
     }
   }
   ```

   它返回 `Result<StorageLockGuard<'a, '_, L>, <L as Lockable>::Deadline>`。
   此处的机制是首先获取锁保护，一次只能由一个进程持有，然后才能写入存储。
   然后使用 `set()` 将值写入存储。
   传递到 `set()` 中的值的数据类型必须与上面 `get<T>()` 调用中指定的类型相同。

1. 最后，从链下工作者函数返回。

   完整的代码类似于以下内容：

   ```rust
   const LOCK_BLOCK_EXPIRATION: u32 = 3; // 以区块数为单位
   const LOCK_TIMEOUT_EXPIRATION: u64 = 10000; // 以毫秒为单位

   fn offchain_worker(block_number: T::BlockNumber) {
     let storage = StorageValueRef::persistent(b"pallet::my-storage");

     if let Ok(Some(res)) = storage.get::<u64>() {
       log::info!("缓存结果：{:?}", res);
       return Ok(());
     }

     // 此处进行非常密集的计算
     let val: u64 = 100 + 100;

     // 定义存储锁
     let mut lock = StorageLock::<BlockAndTime<Self>>::with_block_and_time_deadline(
       b"pallet::storage-lock",
       LOCK_BLOCK_EXPIRATION,
       Duration::from_millis(LOCK_TIMEOUT_EXPIRATION)
     );

     if let Ok(_guard) = lock.try_lock() {
       storage.set(&val);
     }
     Ok(())
   }
   ```

## 示例

- [Substrate 中的**链下工作者示例模块**](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L372-L441)
- [**OCW 模块**演示](https://github.com/jimmychu0807/substrate-offchain-worker-demo/blob/master/pallets/ocw/src/lib.rs#L299-L342)
