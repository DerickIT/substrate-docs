---
title: 事件和错误
description: 解释如何从运行时发出事件和错误。
keywords:
---

当模块想要向外部实体（如用户、链浏览器或 dApp）发送有关运行时中更改或条件的通知时，它可以发出事件。

在自定义模块中，你可以定义：

- 你想要发出什么类型的事件
- 这些事件中包含什么信息
- 何时发出这些事件

## 声明事件

使用 `#[pallet::event]` 宏创建事件。
例如：

```rust
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
	/// 设置值。
	ValueSet { value: u32, who: T::AccountId },
}
```

然后，需要 `RuntimeEvent` 类型来为运行时聚合它们。

```rust
#[pallet::config]
	pub trait Config: frame_system::Config {
		/// 主要的事件类型。
		type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
	}
```

## 将事件公开到你的运行时

在 `/runtime/src/lib.rs` 文件中，必须将你在模块中定义的任何事件公开到运行时。

要将事件公开到运行时：

1. 在文本编辑器中打开 `/runtime/src/lib.rs` 文件。
2. 在模块的配置特性中实现 `RuntimeEvent` 类型：

   ```rust
   impl template::Config for Runtime {
   	 type RuntimeEvent = RuntimeEvent;
   }
   ```

3. 将 `RuntimeEvent` 类型添加到 `construct_runtime!` 宏：

   ```rust
   construct_runtime!(
   	 pub enum Runtime where
    	 Block = Block,
   	   NodeBlock = opaque::Block,
   	   UncheckedExtrinsic = UncheckedExtrinsic
   	 {
       // --snip--
   	   TemplateModule: template::{Pallet, Call, Storage, Event<T>},
   	   //--add-this------------------------------------->
   		 }
   );
   ```

   在此示例中，事件是泛型类型，需要 `<T>` 参数。
   如果你的事件不使用泛型类型，则不需要 `<T>` 参数。

## 存储事件

Substrate 提供了使用宏存储事件的默认实现。
存储事件具有以下结构：

```rust
// 1. 声明 Events 枚举时使用 `generate_deposit` 属性。
#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)] // <------ here ----
	#[pallet::metadata(...)]
	pub enum Event<T: Config> {
		// --snip--
	}

// 2. 在可调度函数内使用 `deposit_event`
#[pallet::call]
	impl<T: Config> Pallet<T> {
		#[pallet::weight(1_000)]
		pub(super) fn set_value(
			origin: OriginFor<T>,
			value: u64,
		) -> DispatchResultWithPostInfo {
			let sender = ensure_signed(origin)?;
			// --snip--
			Self::deposit_event(RawEvent::ValueSet(value, sender));
		}
	}
```

此函数的默认行为是从 FRAME 系统调用 [`deposit_event`](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#method.deposit_event)，它会将事件写入存储。

此函数将事件放入该块的系统模块的运行时存储中。
在新块开始时，系统模块会自动删除在上一个块中存储的所有事件。

使用默认实现存储的事件直接受下游库（如 [Polkadot-JS API](https://github.com/polkadot-js/api)）支持。
但是，如果你想以不同的方式处理事件，则可以实现你自己的 `deposit_event` 函数。

## 支持的类型

事件可以使用 [SCALE 编解码器](/reference/scale-codec) 发出任何支持类型编码的类型。

如果你想使用 `AccountId` 或 `Balances` 等运行时泛型类型，则需要
包含一个 [`where` 子句](https://doc.rust-lang.org/rust-by-example/generics/where.html) 来定义
如上例所示的这些类型。

## 监听事件

Substrate RPC 不会直接公开用于查询事件的端点。
如果你使用了默认实现，则可以通过查询系统模块的存储来查看当前块的事件列表。
否则，[Polkadot-JS API](https://github.com/polkadot-js/api) 支持对运行时事件的 WebSocket 订阅。

## 错误

运行时代码应明确且优雅地处理所有错误情况。
运行时代码中的函数必须是非抛出函数，决不会导致编译器 [panic](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html)。
编写非抛出 Rust 代码的常用习惯用法是编写返回 [`Result` 类型](https://paritytech.github.io/substrate/master/frame_support/dispatch/result/enum.Result.html) 的函数。
`Result` 枚举类型拥有一个 `Err` 变体，允许函数指示它未能成功执行而无需 panic。
可以在 FRAME 开发环境中调度到运行时的函数调用_必须_返回一个 [`DispatchResult` 类型](https://paritytech.github.io/substrate/master/frame_support/dispatch/type.DispatchResult.html)，如果函数遇到错误，则_可能_是 [`DispatchError` 变体](https://paritytech.github.io/substrate/master/frame_support/dispatch/enum.DispatchError.html)。

每个 FRAME 模块都可以使用 `#[pallet::error]` 宏定义自定义 `DispatchError`。
例如：

```rust
#[pallet::error]
pub enum Error<T> {
		/// 错误名称应具有描述性。
		InvalidParameter,
		/// 错误应具有与其关联的有用文档。
		OutOfSpace,
	}
```

FRAME 支持模块还包含一个有用的 [`ensure!` 宏](https://paritytech.github.io/substrate/master/frame_support/macro.ensure.html)，可用于检查先决条件，如果不满足则发出错误。

```rust
frame_support::ensure!(param < T::MaxVal::get(), Error::<T>::InvalidParameter);
```

## 接下来的步骤

- [Frame 宏](/reference/frame-macros)
- [Polkadot-JS API](https://github.com/polkadot-js/api)。
- [`construct_runtime!` 宏](https://paritytech.github.io/substrate/master/frame_support/macro.construct_runtime.html)
- [`#[frame_support::pallet]` 宏](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html)
- [`[pallet::error]` 宏](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#error-palleterror-optional)
