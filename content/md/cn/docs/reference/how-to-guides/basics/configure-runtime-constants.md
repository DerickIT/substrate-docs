---
title: 配置运行时常量
description: 在运行时声明一个常量值是一个有用的工具，可以用来定义固定值或根据某些因素动态变化的值。
keywords:
  - 基础
  - 运行时
  - 可配置常量
---

在运行时声明一个常量值是一个有用的工具，可以用来定义固定值或根据某些因素动态变化的值。
本指南展示如何创建用于重置存储中 `u32` 值的模块常量。
我们将此值称为 `SingleValue`，也可以使用名为 `add_value` 的方法修改它。

## 配置模块的类型、事件和错误

1. 在模块中定义常量。

   - `MaxAddend` 将是在元数据中显示的值。
   - `ClearFrequency` 用于跟踪区块编号，并将用于重置 `SingleValue`。

   ```rust
   #[pallet::config]
   pub trait Config: frame_system::Config {
   	type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;

   	#[pallet::constant] // 将常量放入元数据
   	/// 每次调用添加的最大金额。
   	type MaxAddend: Get<u32>;

   	/// 清除存储值的频率。
   	type ClearFrequency: Get<Self::BlockNumber>;
   }
   ```

1. 声明存储项和事件。

   使用存储属性宏，声明 `SingleValue`，即每个区块周期都会修改的值。

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn single_value)]
   pub(super) type SingleValue<T: Config> = StorageValue<_, u32, ValueQuery>;
   ```

1. 定义模块的事件。

   ```rust
   #[pallet::event]
   #[pallet::generate_deposit(pub(super) fn deposit_event)]
   pub enum Event<T: Config> {
   	/// 值已添加。参数为
   	/// （初始金额、添加的金额、最终金额）。
   	Added(u32, u32, u32),
   	/// 值已清除。参数是清除前的值。
   	Cleared(u32)
   }
   ```

1. 添加一个处理操作溢出的错误：

   ```rust
   #[pallet::error]
   pub enum Error<T> {
   	/// 操作会导致溢出。
   	Overflow
   }
   ```

## 创建模块方法和运行时常量

1. 配置 `on_finalize`。

   `SingleValue` 在 `on_finalize` 函数中每 `ClearFrequency` 个区块被设置为 0，该函数在区块执行结束时运行。
   在 `#[pallet::hooks]` 属性下指定此逻辑：

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	fn on_finalize(n: T::BlockNumber) {
   		if (n % T::ClearFrequency::get()).is_zero() {
   			let current_value = <SingleValue<T>>::get();
   			<SingleValue<T>>::put(0u32);
   			Self::deposit_event(Event::Cleared(current_value));
   		}
   	}
   }
   ```

1. 创建一个允许用户指定值的方法。

   `add_value` 方法增加 `SingleValue`，只要每次调用添加的值小于 `MaxAddend` 值。

   对于此方法，您必须：

   - 包含检查。
   - 跟踪先前值。
   - 检查溢出。
   - 更新 `SingleValue`。

   ```rust
   // 从运行时外部可调用的外部函数。
   #[pallet::call]
   impl<T: Config> Pallet<T> {
   	#[pallet::weight(1_000)]
   	fn add_value(
   		origin: OriginFor<T>,
   		val_to_add: u32
   	) -> DispatchResultWithPostInfo {
   		let _ = ensure_signed(origin)?;
   		ensure!(val_to_add <= T::MaxAddend::get(), "value must be <= maximum add amount constant");
   		// 获取先前值
   		let c_val = SingleValue::<T>::get();
   		// 检查添加新值时是否溢出
   		let result = c_val.checked_add(val_to_add).ok_or(Error::<T>::Overflow)?;
   		<SingleValue<T>>::put(result);
   		Self::deposit_event(Event::Added(c_val, val_to_add, result));
   		Ok(().into())
   	}
   }
   ```

1. 为运行时提供常量值。

   在 `runtime/src/lib.rs` 中，声明模块运行时实现的 `MaxAddend` 和 `ClearFrequency` 的值：

   ```rust
   parameter_types! {
   	pub const MaxAddend: u32 = 1738;
   	pub const ClearFrequency: u32 = 10;
   }

   impl constant_config::Config for Runtime {
   	type RuntimeEvent = RuntimeEvent;
   	type MaxAddend = MaxAddend;
   	type ClearFrequency = ClearFrequency;
   }
   ```

## 示例

- [`constant-config` 示例模块](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/configurable-constant/src/lib.rs)

## 资源

- [升级正在运行的网络](/tutorials/build-a-blockchain/upgrade-a-running-network/)
- [Get 特性](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Get.html)
- [Extra_constants](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#extra-constants-palletextra_constants-optional)
