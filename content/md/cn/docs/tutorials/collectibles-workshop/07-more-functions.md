---
title: 更多函数
description:
tutorial:
---

为了使此模块更有用，并使其更像简单的可收藏物品交易应用程序的基础，让我们添加一些更基本的函数来执行以下任务：

- 在帐户之间转移可收藏物品。
- 设置可收藏物品的价格。
- 购买可收藏物品。

## 转移

要允许用户将可收藏物品从一个帐户转移到另一个帐户，您需要创建一个公开可调用的函数。
在本研讨会中，公共函数（`transfer`）将依赖于名为`do_transfer`的内部函数来执行检查并将更改写入存储。
内部函数包括以下检查，以确定是否可以转移可收藏物品：

- 可收藏物品必须存在才能转移。
- 不可将可收藏物品转移到其当前所有者。
- 不可将可收藏物品转移到已经拥有允许的最大数量的可收藏物品的帐户。

如果可收藏物品存在并且没有被转移到其当前所有者，则`do_transfer`内部函数会更新`OwnerOfCollectibles`存储映射以反映新的所有者并将更改写入底层数据库。
为了处理错误并发出事件，您还需要：

- 当检查失败时，`NoCollectible`、`NotOwner`和`TransferToSelf`错误。
- 当转移成功时，`TransferSucceeded`事件。

公开可调用的函数只需检查调用者的来源并调用`do_transfer`内部函数即可。

1. 将`NoCollectible`、`NotOwner`和`TransferToSelf`变体添加到错误枚举中。

	 ```rust
	 // Pallet error messages.
	 #[pallet::error]
	 pub enum Error<T> {
		 /// Each collectible must have a unique identifier
		 DuplicateCollectible,
		 /// An account can't exceed the `MaximumOwned` constant
		 MaximumCollectiblesOwned,
		 /// The total supply of collectibles can't exceed the u64 limit
		 BoundsOverflow,
		 /// The collectible doesn't exist
		 NoCollectible,
		 /// You are not the owner
		 NotOwner,
		 /// Trying to transfer a collectible to yourself
		 TransferToSelf,
	 }
   ```

2. 将`TransferSucceeded`事件添加到您的模块中。

	 ```rust
	 // Pallet events.
	 #[pallet::event]
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
	 pub enum Event<T: Config> {
		  /// A new collectible was successfully created.
			CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
			/// A collectible was successfully transferred.
			TransferSucceeded { from: T::AccountId, to: T::AccountId, collectible: [u8; 16] },
	 }
	 ```

3. 创建一个启用转移可收藏物品的内部函数。

   ```rust
	 // Update storage to transfer collectible
	 pub fn do_transfer(
		 collectible_id: [u8; 16],
		 to: T::AccountId,
	 ) -> DispatchResult {
		 // Get the collectible
		 let mut collectible = CollectibleMap::<T>::get(&collectible_id).ok_or(Error::<T>::NoCollectible)?;
		 let from = collectible.owner;

		 ensure!(from != to, Error::<T>::TransferToSelf);
		 let mut from_owned = OwnerOfCollectibles::<T>::get(&from);

		 // Remove collectible from list of owned collectible.
		 if let Some(ind) = from_owned.iter().position(|&id| id == collectible_id) {
			 from_owned.swap_remove(ind);
			} else {
			  return Err(Error::<T>::NoCollectible.into())
			}
				// Add collectible to the list of owned collectibles.
				let mut to_owned = OwnerOfCollectibles::<T>::get(&to);
				to_owned.try_push(collectible_id).map_err(|_id| Error::<T>::MaximumCollectiblesOwned)?;

				// Transfer succeeded, update the owner and reset the price to `None`.
				collectible.owner = to.clone();
				collectible.price = None;

				// Write updates to storage
				CollectibleMap::<T>::insert(&collectible_id, collectible);
				OwnerOfCollectibles::<T>::insert(&to, to_owned);
				OwnerOfCollectibles::<T>::insert(&from, from_owned);

				Self::deposit_event(Event::TransferSucceeded { from, to, collectible: collectible_id });
			Ok(())
		}
	```

1. 创建可调用函数。

	 ```rust
	 /// Transfer a collectible to another account.
	 /// Any account that holds a collectible can send it to another account.
	 /// Transfer resets the price of the collectible, marking it not for sale.
	 #[pallet::weight(0)]
	 pub fn transfer(
		  origin: OriginFor<T>,
			to: T::AccountId,
			unique_id: [u8; 16],
	 ) -> DispatchResult {
		// Make sure the caller is from a signed origin
		let from = ensure_signed(origin)?;
		let collectible = CollectibleMap::<T>::get(&unique_id).ok_or(Error::<T>::NoCollectible)?;
		ensure!(collectible.owner == from, Error::<T>::NotOwner);
		Self::do_transfer(unique_id, to)?;
		Ok(())
	 }
   ```

1. 保存更改。

2. 通过运行以下命令验证程序是否已编译，没有任何错误：

   ```bash
   cargo build --package collectibles
   ```

## 设置价格

要允许可收藏物品的所有者设置他们拥有的可收藏物品的价格，模块必须提供一个函数来更新`Collectible`数据结构中的价格字段并发出事件。
对于此函数，您可以使用`CollectibleMap`存储项的`get`和`insert`方法来修改和更新`Collectible`对象。

与之前的函数一样，在允许调用者将新价格写入存储之前，您需要执行一些检查：

- 调用者必须是已签名的来源。
- 可收藏物品必须已存在。
- 调用者必须是可收藏物品的所有者。

如果检查通过，则该函数会将新价格写入存储并发出`PriceSet`事件。

1. 将`PriceSet`事件添加到您的模块中。

	 ```rust
	 // Pallet events
	 #[pallet::event]
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
	 pub enum Event<T: Config> {
		/// A new collectible was successfully created.
		CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
     	/// A collectible was successfully transferred.
     	TransferSucceeded { from: T::AccountId, to: T::AccountId, collectible: [u8; 16] },
		/// The price of a collectible was successfully set.
		PriceSet { collectible: [u8; 16], price: Option<BalanceOf<T>> },
		}
	 ```

2. 添加用于设置价格的可调用函数。

   ```rust
    /// Update the collectible price and write to storage.
    #[pallet::weight(0)]
    pub fn set_price(
        origin: OriginFor<T>,
        unique_id: [u8; 16],
        new_price: Option<BalanceOf<T>>,
    ) -> DispatchResult {
        // Make sure the caller is from a signed origin
        let sender = ensure_signed(origin)?;
        // Ensure the collectible exists and is called by the owner
        let mut collectible = CollectibleMap::<T>::get(&unique_id).ok_or(Error::<T>::NoCollectible)?;
        ensure!(collectible.owner == sender, Error::<T>::NotOwner);
        // Set the price in storage
        collectible.price = new_price;
        CollectibleMap::<T>::insert(&unique_id, collectible);

        // Deposit a "PriceSet" event.
        Self::deposit_event(Event::PriceSet { collectible: unique_id, price: new_price });
        Ok(())
    }
    ```

2. 通过运行以下命令验证程序是否已编译，没有任何错误：

   ```bash
   cargo build --package collectibles
   ```

## 购买可收藏物品

要使用户能够购买可收藏物品，您需要公开另一个可调用函数（`buy_collectible`），该函数与之前的可调用函数类似，在将任何更改写入存储之前，它使用内部函数来执行检查。

在本研讨会中，内部函数为`do_buy_collectible`，它执行大部分繁重的工作来确定购买可收藏物品的尝试是否会成功。
例如，`do_buy_collectible`内部函数会检查：

- 提出的购买价格大于或等于可收藏物品的所有者为其设置的价格，如果提出的价格过低，则返回`BidPriceTooLow`错误。
- 可收藏物品正在出售，如果可收藏物品的价格为`None`，则返回`NotForSale`错误。
- 购买者的帐户有足够的可用余额来支付可收藏物品的价格。
- 购买者的帐户没有太多可收藏物品，无法接收另一个可收藏物品。

如果所有检查都通过，则`do_buy_collectible`内部函数将更新帐户余额并使用`Currency`特征的转移方法转移可收藏物品的所有权。

由于内部函数完成了大部分工作，因此公开的`buy_collectible`函数只需验证函数调用者的帐户并调用`do_buy_collectible`函数即可。

1. 将`BidPriceTooLow`和`NotForSale`添加到`Error`枚举类型的变体中。

	 ```rust
	 // Pallet errors
	 #[pallet::error]
      pub enum Error<T> {
        /// Each collectible must have a unique identifier
        DuplicateCollectible,
        /// An account can't exceed the `MaximumOwned` constant
        MaximumCollectiblesOwned,
        /// The total supply of collectibles can't exceed the u64 limit
        BoundsOverflow,
        /// The collectible doesn't exist
        NoCollectible,
        // You are not the owner
        NotOwner,
        /// Trying to transfer a collectible to yourself
        TransferToSelf,
		/// The bid is lower than the asking price.
		BidPriceTooLow,
		/// The collectible is not for sale.
		NotForSale,
      }

2. 将`Sold`事件添加到您的模块中。

	 ```rust
	 // Pallet events
   #[pallet::event]
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
      pub enum Event<T: Config> {
        /// A new collectible was successfully created.
        CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
        /// A collectible was successfully transferred.
        TransferSucceeded { from: T::AccountId, to: T::AccountId, collectible: [u8; 16] },
        /// The price of a collectible was successfully set.
        PriceSet { collectible: [u8; 16], price: Option<BalanceOf<T>> },
        /// A collectible was successfully sold.
		Sold { seller: T::AccountId, buyer: T::AccountId, collectible: [u8; 16], price: BalanceOf<T> },
      }
		```

1. 创建一个在用户想要购买可收藏物品时调用的内部函数。

	 ```rust
	 // An internal function for purchasing a collectible
	 pub fn do_buy_collectible(
		  unique_id: [u8; 16],
		  to: T::AccountId,
		  bid_price: BalanceOf<T>,
	 ) -> DispatchResult {

	 // Get the collectible from the storage map
	 let mut collectible = CollectibleMap::<T>::get(&unique_id).ok_or(Error::<T>::NoCollectible)?;
	 let from = collectible.owner;
	    ensure!(from != to, Error::<T>::TransferToSelf);
		  let mut from_owned = OwnerOfCollectibles::<T>::get(&from);

	 // Remove collectible from owned collectibles.
	 if let Some(ind) = from_owned.iter().position(|&id| id == unique_id) {
			from_owned.swap_remove(ind);
		} else {
			return Err(Error::<T>::NoCollectible.into())
		}
	 // Add collectible to owned collectible.
	 let mut to_owned = OwnerOfCollectibles::<T>::get(&to);
	 to_owned.try_push(unique_id).map_err(|_id| Error::<T>::MaximumCollectiblesOwned)?;
	 // Mutating state with a balance transfer, so nothing is allowed to fail after this.
	 if let Some(price) = collectible.price {
			ensure!(bid_price >= price, Error::<T>::BidPriceTooLow);
			// Transfer the amount from buyer to seller
			T::Currency::transfer(&to, &from, price, frame_support::traits::ExistenceRequirement::KeepAlive)?;
			// Deposit sold event
			Self::deposit_event(Event::Sold {
				seller: from.clone(),
				buyer: to.clone(),
				collectible: unique_id,
				price,
			});
	 } else {
		  return Err(Error::<T>::NotForSale.into())
   }

	 // Transfer succeeded, update the collectible owner and reset the price to `None`.
	 collectible.owner = to.clone();
	 collectible.price = None;
	 // Write updates to storage
	 CollectibleMap::<T>::insert(&unique_id, collectible);
	 OwnerOfCollectibles::<T>::insert(&to, to_owned);
	 OwnerOfCollectibles::<T>::insert(&from, from_owned);
	 Self::deposit_event(Event::TransferSucceeded { from, to, collectible: unique_id });
	 Ok(())
	 }
   ```

1. 添加一个可调用函数，允许用户购买可收藏物品。

   ```rust
	 /// Buy a collectible. The bid price must be greater than or equal to the price
	 /// set by the collectible owner.
	 #[pallet::weight(0)]
	 pub fn buy_collectible(
		 origin: OriginFor<T>,
		 unique_id: [u8; 16],
		 bid_price: BalanceOf<T>,
	 ) -> DispatchResult {
		 // Make sure the caller is from a signed origin
		 let buyer = ensure_signed(origin)?;
		 // Transfer the collectible from seller to buyer.
		 Self::do_buy_collectible(unique_id, buyer, bid_price)?;
	 Ok(())
	 }
	 ```

2. 通过运行以下命令验证程序是否已编译，没有任何错误：

   ```bash
   cargo build --package collectibles
