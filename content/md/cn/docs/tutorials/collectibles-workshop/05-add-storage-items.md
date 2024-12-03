---
title: 添加自定义存储项
description:
tutorial: 1
---

为了使`collectibles`模块有用，它需要存储有关创建的可收藏物品数量以及谁拥有每个可收藏物品的信息。
确定要存储的信息后，您需要确定如何存储它：作为单个值还是存储映射。
对于本研讨会，您将创建三个自定义存储项来跟踪状态：

- 一个简单的单值（`CollectiblesCount`）用于跟踪模块中可收藏物品的总数。
- 一个简单的键值对映射（`CollectiblesMap`）用于将与每个可收藏物品关联的属性映射到其唯一标识符。
- 一个简单的键值对映射（`OwnerOfCollectibles`）用于将可收藏物品映射到拥有它们的用户帐户。

有关 Substrate 使用的存储架构和抽象的更详细说明，请参阅[状态转换和存储](/learn/state-transitions-and-storage/)。

## 存储单值

FRAME 存储模块提供了一个`StorageValue`特征来在运行时存储单值。

在本研讨会中，您将对`CollectiblesCount`使用`StorageValue`——用于跟踪模块中可收藏物品的总数。`StorageValue`跟踪一个 64 位无符号整数 (u64) 值，每次生成新的可收藏物品时都会递增此值，最多可达 18_446_744_073_709_551_615 个唯一可收藏物品。

```rust
#[pallet::storage]
pub(super) type CollectiblesCount<T: Config> = StorageValue<_, u64, ValueQuery>;
```

此声明中的`ValueQuery`指定如果存储中没有值，则查询应返回什么。
处理查询返回内容有三种可能的设置：`OptionQuery`、`ResultQuery`或`ValueQuery`。
我们在此处使用`ValueQuery`，因此如果存储中没有值（例如，当您第一次启动网络时），查询应返回零 (0) 值，而不是`OptionQuery`值为`None`或`ResultQuery`值为`Err`。

## 将可收藏物品映射到它们的属性

FRAME 存储模块提供了一个`StorageMap`特征来在运行时存储单个键映射。
名为`CollectiblesMap`的 StorageMap 将每个可收藏物品映射到其唯一信息。
`CollectiblesMap`映射的键是可收藏物品的`unique_id`。

```rust
/// Maps the Collectible struct to the unique_id.
#[pallet::storage]
pub(super) type CollectibleMap<T: Config> = StorageMap<_, Twox64Concat, [u8; 16], Collectible<T>>;
```

此声明中的`Twox64Concat`指定要使用的哈希算法来创建此存储值。
通过允许您指定要使用的哈希算法，存储映射允许您控制存储信息的安全性级别。
例如，您可以选择性能更高但安全性较低的哈希算法来存储有关可收藏物品的信息，以及性能较低但安全性较高的哈希算法来存储更敏感的信息。
有关 Substrate 支持的哈希算法及其提供的安全性的信息，请参阅[哈希算法](/build/runtime-storage/#hashing-algorithms)。

## 将所有者映射到他们的可收藏物品

名为`OwnerOfCollectibles`的 StorageMap 将用户帐户映射到他们拥有的可收藏物品。

此存储映射的键是用户帐户：`T::AccountID`。
此存储映射的值是具有每个用户帐户拥有的每个可收藏物品的`unique_id`的`BoundedVec`数据类型。
此映射使查找每个单独的可收藏物品的信息变得很容易，因为`unique_id`用作`collectiblesMap`映射的键。
通过使用 BoundedVec，您可以确保每个存储项都有最大长度，这对于管理运行时内的限制非常重要。

```rust
/// Track the collectibles owned by each account.
#[pallet::storage]
pub(super) type OwnerOfCollectibles<T: Config> = StorageMap<
	_,
	Twox64Concat,
	T::AccountId,
	BoundedVec<[u8; 16], T::MaximumOwned>,
	ValueQuery,
>;
```

为了使代码能够编译，您需要使用此派生宏来注释`Collectible`数据结构：

```rust
#[scale_info(skip_type_params(T))]
```

通过运行以下命令验证程序是否已编译：

```bash
cargo build --package collectibles
```

目前，您可以忽略编译器警告。
