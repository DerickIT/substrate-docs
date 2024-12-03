---
title: 创建基本函数
description:
tutorial:
---

您现在已经准备好了模块的框架，其中包含一些自定义类型和存储项。
现在是时候开始编写使用自定义类型并写入您创建的存储项的更改的函数了。
为了使代码模块化和灵活，大部分逻辑都在内部辅助函数中定义，只有一个可调用函数允许用户与区块链交互。
使用内部函数的一个优点是，您不需要对执行的操作进行任何类型的身份验证或授权，因为内部函数只能由您作为运行时开发者访问。

在本研讨会中，您只需要向用户公开的函数是`create_collectible`函数。
此函数使用户能够创建新的唯一可收藏物品，这些物品存储在`CollectibleMap`中并添加到`OwnerOfCollectibles`映射中。
在较高的层次上，您需要编写函数来执行以下任务：

- 为每个可收藏物品创建一个唯一标识符，并且不允许重复。
- 限制每个帐户可以拥有的可收藏物品的数量，以管理每个用户允许使用的存储。
- 确保可收藏物品的总数不超过`u64`数据类型允许的最大值。
- 允许用户生成新的可收藏物品。

除了生成新的可收藏物品所需的基本功能外，您的模块还应提供一些基本的事件和错误处理。
考虑到这一点，您还需要：

- 创建自定义错误消息以报告如果出现问题会发生什么。
- 创建自定义事件以在成功创建新的可收藏物品时发出信号。

错误和事件相当简单，所以让我们首先从这些声明开始。

## 添加自定义错误

以下是`create_collectible`函数应处理的一些潜在错误：

- `DuplicateCollectible`- 当尝试创建的可收藏物品项目已存在时抛出。
- `MaximumCollectiblesOwned`- 当帐户超过单个帐户可以持有的可收藏物品的最大数量时抛出。
- `BoundsOverflow`- 如果可收藏物品的供应量超过`u64`限制，则抛出。

要向运行时添加错误处理：

1. 在代码编辑器中打开`collectibles`模块的`src/lib.rs`文件。

2. 在您先前定义的存储宏之后添加`#[pallet::error]`宏。

	 ```rust
	 #[pallet::error]
	 ```

1. 使用`create_collectibles`函数可能返回的潜在错误的变体定义枚举数据类型。

	 ```rust
	 pub enum Error<T> {
		/// Each collectible must have a unique identifier
		DuplicateCollectible,
		/// An account can't exceed the `MaximumOwned` constant
		MaximumCollectiblesOwned,
		/// The total supply of collectibles can't exceed the u64 limit
		BoundsOverflow,
	}
	```

1. 保存更改。

1. 通过运行以下命令验证程序是否已编译：

   ```bash
   cargo build --package collectibles
   ```

   目前，您可以忽略编译器警告。

## 添加事件

运行时可以发出事件以通知前端应用程序已成功执行的事务的结果。
像[Subscan](https://www.subscan.io/)和[Polkadot/Substrate 门户资源管理器](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/explorer)这样的区块浏览器还会显示已完成事务的事件。

要向运行时添加`CollectibleCreated`事件：

1. 在代码编辑器中打开`collectibles`模块的`src/lib.rs`文件。

1. 将`frame_system`配置中的`RuntimeEvent`添加到模块配置中。

	 ```rust
	 #[pallet::config]
     pub trait Config: frame_system::Config {
		type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
	}
	 ```

2. 在您先前定义的错误宏之后添加`#[pallet::event]`宏。

	 ```rust
	 #[pallet::event]
	 ```

1. 将此模块的事件添加到代码中。

	 ```rust
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
	 pub enum Event<T: Config> {
		 /// A new collectible was successfully created
		 CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
	 }
	 ```

1. 保存更改。

1. 通过运行以下命令验证程序是否已编译：

   ```bash
   cargo build --package collectibles
   ```

   目前，您可以忽略编译器警告。

## 添加内部函数和可调用函数

解决了错误和事件问题后，就可以编写创建可收藏物品的核心逻辑了。

1. 创建一个内部函数，用于为新的可收藏物品生成`unique_id`。

	 ```rust
	 // Pallet internal functions
	impl<T: Config> Pallet<T> {
		// Generates and returns the unique_id and color
		fn gen_unique_id() -> ([u8; 16], Color) {
			// Create randomness
			let random = T::CollectionRandomness::random(&b"unique_id"[..]).0;

			// Create randomness payload. Multiple collectibles can be generated in the same block,
			// retaining uniqueness.
			let unique_payload = (
				random,
				frame_system::Pallet::<T>::extrinsic_index().unwrap_or_default(),
				frame_system::Pallet::<T>::block_number(),
		);

		// Turns into a byte array
		let encoded_payload = unique_payload.encode();
		let hash = frame_support::Hashable::blake2_128(&encoded_payload);

		// Generate Color
		if hash[0] % 2 == 0 {
				(hash, Color::Red)
		} else {
				(hash, Color::Yellow)
			}
		}
	}
   ```

1. 创建一个启用铸造新可收藏物品的内部函数。

	 ```rust
	     // Function to mint a collectible
		pub fn mint(
			owner: &T::AccountId,
			unique_id: [u8; 16],
			color: Color,
		) -> Result<[u8; 16], DispatchError> {
			// Create a new object
			let collectible = Collectible::<T> { unique_id, price: None, color, owner: owner.clone() };

			// Check if the collectible exists in the storage map
			ensure!(!CollectibleMap::<T>::contains_key(&collectible.unique_id), Error::<T>::DuplicateCollectible);

			// Check that a new collectible can be created
			let count = CollectiblesCount::<T>::get();
			let new_count = count.checked_add(1).ok_or(Error::<T>::BoundsOverflow)?;

			// Append collectible to OwnerOfCollectibles map
			OwnerOfCollectibles::<T>::try_append(&owner, collectible.unique_id)
				.map_err(|_| Error::<T>::MaximumCollectiblesOwned)?;

			// Write new collectible to storage and update the count
			CollectibleMap::<T>::insert(collectible.unique_id, collectible);
			CollectiblesCount::<T>::put(new_count);

			// Deposit the "CollectibleCreated" event.
			Self::deposit_event(Event::CollectibleCreated { collectible: unique_id, owner: owner.clone() });

			// Returns the unique_id of the new collectible if this succeeds
			Ok(unique_id)
		}
   ```

2. 创建使用内部函数的可调用函数。

	 ```rust
	 // Pallet callable functions
	 #[pallet::call]
	 impl<T: Config> Pallet<T> {

		 /// Create a new unique collectible.
		 ///
		 /// The actual collectible creation is done in the `mint()` function.
		 #[pallet::weight(0)]
		 pub fn create_collectible(origin: OriginFor<T>) -> DispatchResult {
			  // Make sure the caller is from a signed origin
				let sender = ensure_signed(origin)?;

				// Generate the unique_id and color using a helper function
				let (collectible_gen_unique_id, color) = Self::gen_unique_id();

				// Write new collectible to storage by calling helper function
				Self::mint(&sender, collectible_gen_unique_id, color)?;

				Ok(())
			}
	 }
   ```

1. 保存更改并通过运行以下命令验证程序是否已编译：

   ```bash
   cargo build --package collectibles
   ```

  您的代码现在应该可以编译，没有任何警告。
