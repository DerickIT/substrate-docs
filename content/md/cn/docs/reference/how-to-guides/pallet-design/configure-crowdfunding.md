---
title: 配置众筹
description: 如何使用 FRAME 模块创建众筹活动。
keywords:
  - 模块设计
  - 中级
  - 运行时
  - 子 trie
---

本指南向您展示如何构建一个控制多个令牌帐户并在子存储中存储数据的模块。
此结构可用于构建众筹应用程序。

在本指南中，我们将重点介绍子存储 trie 的使用，它允许任何贡献者使用小型 Merkle 证明来证明他们做出了贡献。
能够进行简单的贡献证明可以帮助用户声明参与众筹的奖励。

在本指南中，您将看到如何为每个不同的众筹活动创建一个新的子 trie。
任何用户都可以通过指定众筹的目标金额、结束时间和受益人（如果在结束时间之前达到目标，则将获得池化资金）来启动众筹。
如果资金筹集不成功，则可以解散，并将资金返还给原始所有者。

## 开始之前

要遵循本指南并在您自己的模块中实现众筹功能，请在您的模块中包含以下依赖项：

```rust
use frame_support::{
	ensure,
	pallet_prelude::*,
	sp_runtime::traits::{AccountIdConversion, Hash, Saturating, Zero},
	storage::child,
	traits::{Currency, ExistenceRequirement, Get, ReservableCurrency, WithdrawReasons},
	PalletId,
};
use frame_system::{pallet_prelude::*, ensure_signed};
use super::*;
```

本指南假设您知道如何根据您正在创建的模块逻辑创建您自己的错误和事件。

## 配置您的模块

1. 声明模块的配置特性。

   除了 `Event` 类型外，此模块还需要以下特性：

   - **`Currency`** - 众筹将以其计价的货币。
   - **`SubmissionDeposit`** - 众筹所有者需要存入的金额。
   - **`MinContribution`** - 可以贡献给众筹的最低金额。

   使用以下类型扩展您的特性：

   ```rust
   /// 模块的配置特性。
   #[pallet::config]
   pub trait Config: frame_system::Config {
   	type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
   	type Currency: ReservableCurrency<Self::AccountId>;
   	type SubmissionDeposit: Get<BalanceOf<Self>>;
   	type MinContribution: Get<BalanceOf<Self>>;
   	type RetirementPeriod: Get<Self::BlockNumber>;
   }
   ```

   以及以下类型别名：

   ```rust
   pub type FundIndex = u32;
   type AccountIdOf<T> = <T as frame_system::Config>::AccountId;
   type BalanceOf<T> = <<T as Config>::Currency as Currency<AccountIdOf<T>>>::Balance;
   ```

1. 创建一个结构体来存储众筹的配置。

   ```rust
   #[derive(Encode, Decode, Default, PartialEq, Eq, TypeInfo)]
   #[cfg_attr(feature = "std", derive(Debug))]
   pub struct FundInfo<AccountId, Balance, BlockNumber> {
   	/// 如果活动成功，将接收资金的帐户。
   	beneficiary: AccountId,
   	/// 已存入的存款金额。
   	deposit: Balance,
   	/// 已筹集的总金额。
   	raised: Balance,
   	/// 资金必须成功后的区块号。
   	end: BlockNumber,
   	/// `raised` 的上限。
   	goal: Balance,
   }
   ```

   以及一个新的别名，以便更容易使用它：

   ```rust
   type FundInfoOf<T> =
   	FundInfo<AccountIdOf<T>, BalanceOf<T>, <T as frame_system::Config>::BlockNumber>;
   ```

1. 声明您的存储项。

   您的存储项将跟踪哪个用户贡献了哪些资金以及他们贡献了多少。
   定义以下类型，这些类型将用于声明您的存储项：

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn funds)]
   /// 所有资金的信息。
   pub(super) type Funds<T: Config> = StorageMap<
   	_,
   	Blake2_128Concat,
   	FundIndex,
   	FundInfoOf<T>,
   	OptionQuery,
   >;

   #[pallet::storage]
   #[pallet::getter(fn fund_count)]
   /// 到目前为止已分配的资金总数。
   pub(super) type FundCount<T: Config> = StorageValue<_, FundIndex, ValueQuery>;
   ```

## 编写子 trie API 辅助函数

创建一个函数，为模块的可调度函数提供资金池的帐户 ID。

1. 在 `impl<T: Config> Pallet<T>` 内，编写：

   ```rust
   pub fn fund_account_id(index: FundIndex) -> T::AccountId {
   	const PALLET_ID: ModuleId = ModuleId(*b"ex/cfund");
   	PALLET_ID.into_sub_account(index)
   }
   ```

1. 生成唯一的 [ChildInfo](https://paritytech.github.io/substrate/master/sp_storage/enum.ChildInfo.html) ID：

   ```rust
   pub fn id_from_index(index: FundIndex) -> child::ChildInfo {
   		let mut buf = Vec::new();
   		buf.extend_from_slice(b"crowdfnd");
   		buf.extend_from_slice(&index.to_le_bytes()[..]);

   		child::ChildInfo::new_default(T::Hashing::hash(&buf[..]).as_ref())
   }
   ```

1. 编写以下使用 [Child API](https://paritytech.github.io/substrate/master/frame_support/storage/child/index.html) 的辅助函数：

   - **`pub fn contribution_put`**: 使用 [`put`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.put.html) 在关联的子 trie 中记录贡献

   - **`pub fn contribution_get`**: 使用 [`get`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.get.html) 在关联的子 trie 中查找贡献

   - **`pub fn contribution_kil`**: 使用 [`kill`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.kill.html) 从关联的子 trie 中删除贡献

   - **`pub fn crowdfund_kill`**: 使用 [`kill_storage`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.kill_storage.html) 在单个存储写入中删除关联子 trie 中贡献的完整记录

## 编写您的可调度函数

以下步骤概述了如何为该模块编写可调度函数。
在可调度函数逻辑中的各种检查之后，每个函数都使用其关联的方法更改 `Funds<T>` 存储映射。
我们的模块的 `create` 函数也使用了步骤 2 中创建的 `FundInfo` 结构体。

1. 创建一个新的资金池

   `fn create`:

   - 使用 [`T::Currency::withdraw`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#tymethod.withdraw) 创建一个不平衡变量
   - 使用步骤 2 中的 `FundInfo` 结构体更新 `Funds` 存储项
   - 存入 `Created` 事件

1. 贡献给现有资金池

   `fn contribute`:

   - 使用 `ensure!` 执行初步安全检查
   - 使用 `T::Currency::transfer` 将贡献添加到资金池
   - 使用辅助函数 `contribution_put` 在子 trie 中记录贡献
   - 存入 `Contributed` 事件

1. 提取贡献者对资金池的全部余额

   `fn withdraw`:

   - 使用 `ensure!` 执行初步安全检查
   - 使用 [`T::Currency::resolve_into_existing`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#method.resolve_into_existing) 返回资金
   - 计算新的余额并使用子 trie 辅助函数 `funds`、`contribution_get` 和 `contribution_kill` 更新存储
   - 存入 `Withdrew` 事件

1. 在其退休期结束后解散整个众筹。

   `fn dissolve`:

   - 使用 `ensure!` 执行初步安全检查
   - 允许解散者通过使用 [`T::Currency::resolve_creating`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#method.resolve_creating) 收集资金
   - 使用子 trie 辅助函数 `crowdfund_kill` 从存储中删除贡献者信息
   - 存入 `Dissolved` 事件

1. 向成功的众筹的受益人支付款项

   `fn dispense`:

   - 使用 [`T::Currency::resolve_creating`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#method.resolve_creating) 为受益人和调用者（分别）收集资金
   - 将初始存款给予调用此函数的帐户作为清理存储的奖励
   - 使用 `<Funds<T>>::remove(index);` 和 `Self::crowdfund_kill(index);` 从存储中删除资金，并在单个写入中删除所有贡献者


## 示例

- [`pallet_simple_crowdfund`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/simple-crowdfund/src/lib.rs#L1)

## 资源

- [Currency Imbalance 特性](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/imbalance/trait.Imbalance.html)
- [子 trie API](https://paritytech.github.io/substrate/master/frame_support/storage/child/index.html)
- [`extend_from_slice`](https://paritytech.github.io/substrate/master/frame_support/dispatch/struct.Vec.html#method.extend_from_slice)
- [`using_encode`](https://paritytech.github.io/substrate/master/frame_support/pallet_prelude/trait.Encode.html#method.using_encoded)
