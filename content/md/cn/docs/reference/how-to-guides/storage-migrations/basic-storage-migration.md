---
title: 基本存储迁移
description: 说明如何修改特定 pallet 的存储并准备迁移到新的存储布局。
keywords:
  - 存储迁移
  - 运行时
  - 升级
---

本指南说明了如何通过修改 FRAME [Nicks pallet](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/nicks) 来执行特定 pallet 的存储迁移。
在本教程中，您将修改存储映射以提供包含姓氏的可选字段，并编写一个迁移函数，然后可以使用运行时升级来触发该函数。
当更改仅限于特定 pallet 和单个存储项时，您可以使用这种类型的简单存储迁移。
您可以按照类似的步骤进行更复杂的数据迁移，但是您需要编写更复杂的迁移函数并使用比本教程中解释的更多工具来测试您的迁移。

## 在本地添加 Nicks pallet

我们将对 [FRAME 的 Nick's pallet](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/nicks) 进行更改，在教程 [将 pallet 添加到运行时](/tutorials/build-application-logic/add-a-pallet/) 中，我们展示了如何将 Nicks pallet 添加到节点模板的运行时中。

对于本指南，因为我们将更改 pallet 的代码，我们将获取 pallet 的代码并将其本地添加到我们的节点模板中。您可以查看如何在本地添加它的示例 [此处](https://github.com/substrate-developer-hub/substrate-node-template/commit/022b6da0d1d55f54de3568e97aa5fe45a7975fa5)。

为了进行测试，我们现在可以启动我们的节点，并使用 Nicks pallet 中的外部函数 `setName` 来设置昵称。

## 创建存储结构体并更新存储项

默认情况下，Nicks pallet 使用存储映射来提供一个查找表，其中包含 `BoundedVec` 来存储昵称。
例如，默认存储定义如下所示：

```rust
/// 名称的查找表。
	#[pallet::storage]
	pub(super) type NameOf<T: Config> =
		StorageMap<_, Twox64Concat, T::AccountId, (BoundedVec<u8, T::MaxLength>, BalanceOf<T>)>;
```
我们希望更新存储以添加一个包含姓氏的可选字段。
为此，我们创建一个新的结构体 `Nickname` 来管理之前的和新的存储项，即名字和姓氏：

```rust
    #[derive(Encode, Decode, Default, TypeInfo, MaxEncodedLen, PartialEqNoBound, RuntimeDebug)]
	#[scale_info(skip_type_params(T))]
	#[codec(mel_bound())]
	pub struct Nickname<T: Config> {
		pub first: BoundedVec<u8, T::MaxLength>,
		pub last: Option<BoundedVec<u8, T::MaxLength>>,
	}
```
要更改现在存储在存储中的数据，我们将更新 StorageMap `NameOf` 以存储 Nickname 结构体，而不是仅存储 `BoundedVec`

```rust
    #[pallet::storage]
	pub(super) type NameOf<T: Config> =
		StorageMap<_, Twox64Concat, T::AccountId, (Nickname<T>, BalanceOf<T>)>;
```

## 更新函数

现在您已经添加了新的数据结构并修改了存储以同时包含名字和可选的姓氏，您必须更新 Nicks pallet 函数以包含新的 `last: Option<BoundedVec<u8>>` 参数声明。
在大多数情况下，修改存储项时更新函数将需要添加一些逻辑来解释这些更改。例如，您可能需要修改参数名称或添加新变量。

在这种情况下，大多数所需的更改都在 `set_name` 和 `force_name` 函数中。
例如，您可以修改 `set_name` 函数以将 `bounded_name` 更改为 `bounded_first` 并添加具有类似于以下代码的 `bounded_last` 声明：

```rust
//--snip
pub fn set_name(origin,
    first: Vec<u8>,
    last: Option<Vec<u8>>)  -> DispatchResult{
```

此外，使用 `Nickname` 结构体更新所有存储写入：
```rust
//--snip
pub fn set_name(origin,
    first: Vec<u8>,
    last: Option<Vec<u8>>)  -> DispatchResult{
    //--snip

    let bounded_first: BoundedVec<_, _> =
        first.try_into().map_err(|_| Error::<T>::TooLong)?;
    ensure!(bounded_first.len() >= T::MinLength::get() as usize, Error::<T>::TooShort);

    let mut bounded_last: BoundedVec<_, _> = Default::default();
    if let Some(last) = last {
        bounded_last= last.try_into().map_err(|_| Error::<T>::TooLong)?;
        ensure!(bounded_last.len() >= T::MinLength::get() as usize, Error::<T>::TooShort);
    }
    let bounded_last: Option<BoundedVec<u8, T::MaxLength>> = Some(bounded_last);

    //--snip
    <NameOf<T>>::insert(&sender, (Nickname{first: bounded_first, last: bounded_last}, deposit));
    }
```
查看如何更新外部函数的示例 [此处](https://github.com/substrate-developer-hub/substrate-node-template/commit/a9ee9b2b9096c2b85ecb4448366df2b8502e7aa7)。

## 添加存储版本

`pallet::pallet` 宏实现了 `traits::GetStorageVersion`，但需要将当前存储版本传达给宏。这可以通过使用 `pallet::storage_version` 宏来完成。
```rust
    /// 当前存储版本，我们将新版本设置为 2。
	const STORAGE_VERSION: StorageVersion = StorageVersion::new(2);


    #[pallet::pallet]
	#[pallet::storage_version(STORAGE_VERSION)]
	pub struct Pallet<T>(_);
```

## 声明迁移模块

迁移模块应包含两部分：

- 指示要从中迁移的已弃用存储的模块。
- 返回权重的迁移函数。

在 src/pallets/nicks/migration.rs 中创建一个新文件

此模块的脚手架如下所示：

```rust
pub mod migration {
  use super::*;

  pub mod v1 {...} // 只包含 V1 存储格式

  pub fn migrate_to_v2<T: Config>() -> Weight {...} // 包含检查并将存储转换为 V2 格式
}
```

## 编写 `migrate_to_v2`

以下是此函数需要执行的操作概述：

- 检查存储版本以确保需要迁移（最佳实践）
- 将存储值转换为新的存储格式
- 更新存储版本
- 返回迁移消耗的权重

### 检查存储版本

围绕检查构建 `migrate_to_v2` 逻辑。如果不需要进行存储迁移，则返回 0：

```rust
    let onchain_version =  Pallet::<T>::on_chain_storage_version();
    if onchain_version < 2 {

    }
    else {
        // 我们在这里不做任何事情。
		Weight::zero()
    }
```

### 转换存储值

使用 [`translate storage 方法`][translate-storage-rustdocs]，
将存储值转换为新格式。由于存储中现有的 `nick` 值可以由空格分隔的字符串组成，因此在 `' '` 处将其拆分，并将之后的内容放入新的 `last` 存储项中。如果不是，则 `last` 采用 `None` 值：

```rust
        // 我们将存储值从旧格式转换为新格式。
        NameOf::<T>::translate::<(Vec<u8>, BalanceOf<T>), _>(
            |k: T::AccountId, (nick, deposit): (Vec<u8>, BalanceOf<T>)| {
                info!(target: LOG_TARGET, "     Migrated nickname for {:?}...", k);

                // 我们在 ' ' (<space>) 处拆分 nick。
                match nick.iter().rposition(|&x| x == b" "[0]) {
                    Some(ndx) => {
                        let bounded_first: BoundedVec<_, _> = nick[0..ndx].to_vec().try_into().unwrap();
                        let bounded_last: BoundedVec<_, _> = nick[ndx + 1..].to_vec().try_into().unwrap();
                        Some((Nickname {
                            first: bounded_first,
                            last: Some(bounded_last)
                        }, deposit))
                },
                    None => {
                        let bounded_name: BoundedVec<_, _> = nick.to_vec().try_into().unwrap();
                        Some((Nickname { first: bounded_name, last: None }, deposit))
                    }
                }
            }
        );
```

### 更新存储版本

```rust
    // 更新存储版本。
	StorageVersion::new(2).put::<Pallet::<T>>();
```

### 返回消耗的权重

为此，请计算存储读取和写入的数量并返回相应的权重：

```rust
let count = NameOf::<T>::iter().count();
T::DbWeight::get().reads_writes(count as Weight + 1, count as Weight + 1)
```

### 7. 在 `on_runtime_upgrade` 中使用 `migrate_to_v2`
在您的 pallet lib.rs 中，声明 mod migration。
```rust
mod migration;
```

然后返回到 pallet 的函数，并在 `on_runtime_upgrade` 中指定 `migrate_to_v2` 函数。这使您可以表达运行时升级时应该发生的情况：

```rust
    #[pallet::hooks]
	impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
		fn on_runtime_upgrade() -> frame_support::weights::Weight {
			migration::migrate_to_v2::<T>()
		}
	}
```

查看完整迁移代码的示例 [此处](https://github.com/substrate-developer-hub/substrate-node-template/commit/cfbe01dd4be358d0df45a81b87b6ba7393e20368)。

## 更新单元测试

编写运行时迁移模块时，务必对其进行测试，以避免因处理存储项而导致任何严重问题。

对于 Nicks pallet，我们有以下测试：

- `fn kill_name_should_work()`
- `fn force_name_should_work()`
- `fn normal_operation_should_work()`
- `fn error_catching_should_work()`

我们必须更新它们以与我们添加的新代码一起使用，例如：
```rust
    #[test]
	fn normal_operation_should_work() {
		new_test_ext().execute_with(|| {
			assert_ok!(Nicks::set_name(RuntimeOrigin::signed(1), b"Gav".to_vec(), None));
			assert_eq!(Balances::reserved_balance(1), 2);
			assert_eq!(Balances::free_balance(1), 8);
			assert_eq!(<NameOf<Test>>::get(1).unwrap().0.first, b"Gav".to_vec());

			assert_ok!(Nicks::set_name(RuntimeOrigin::signed(1), b"Gavin".to_vec(), None));
			assert_eq!(Balances::reserved_balance(1), 2);
			assert_eq!(Balances::free_balance(1), 8);
			assert_eq!(<NameOf<Test>>::get(1).unwrap().0.first, b"Gavin".to_vec());

			assert_ok!(Nicks::clear_name(RuntimeOrigin::signed(1)));
			assert_eq!(Balances::reserved_balance(1), 0);
			assert_eq!(Balances::free_balance(1), 10);
		});
	}
```
查看完整测试修复的示例 [此处](https://github.com/substrate-developer-hub/substrate-node-template/commit/fa021b11878e8621bb455d4638e1821b681c085e)。

## 示例

- [迁移 Nicks pallet](https://github.com/substrate-developer-hub/substrate-node-template/tree/alexd10s/how-to-storage-migration-example)

## 资源

#### Rust 文档

- [Rust 中的 `Option`](https://doc.rust-lang.org/std/option/)
- [`frame_support::storage::migration`](https://paritytech.github.io/substrate/master/frame_support/storage/migration/index.html) 实用程序文档

[translate-storage-rustdocs]: https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageMap.html#method.translate
[nicks-migration-htg-diff]: https://github.com/substrate-developer-hub/migration-example/pull/2/files
