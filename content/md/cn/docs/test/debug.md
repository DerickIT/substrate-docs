---
title: 调试
description:
keywords:
---

调试在所有软件开发领域都是必需的，区块链也不例外。大多数用于通用 Rust 调试的工具也适用于 Substrate。

## 日志记录实用程序

您可以使用 Rust 的日志记录 API 来调试运行时。这包含许多宏，包括 [`debug`](https://docs.rs/log/0.4.14/log/macro.debug.html) 和 [`info`](https://docs.rs/log/0.4.14/log/macro.info.html)。

例如，在使用 [`log` crate](https://crates.io/crates/log) 更新 pallet 的 `Cargo.toml` 文件后，只需使用 `log::info!` 即可记录到控制台：<!-- markdown-link-check-disable-line -->

```rust
pub fn do_something(origin) -> DispatchResult {

	let who = ensure_signed(origin)?;
	let my_val: u32 = 777;

	Something::put(my_val);

	log::info!("called by {:?}", who);

	Self::deposit_event(RawEvent::SomethingStored(my_val, who));
	Ok(())
}
```

## Printable 特性

Printable 特性旨在成为一种在 `no_std` 和 `std` 中从运行时打印的方法。`print` 函数适用于实现 [`Printable` 特性](https://paritytech.github.io/substrate/master/sp_runtime/traits/trait.Printable.html) 的任何类型。
Substrate 默认情况下会为某些类型 (`u8`、`u32`、`u64`、`usize`、`&[u8]`、`&str`) 实现此特性。
您也可以为自己的自定义类型实现它。以下是如何使用节点模板作为示例代码库为 pallet 的 `Error` 类型实现它的示例。

```rust
use sp_runtime::traits::Printable;
use sp_runtime::print;
```

```rust
#[frame_support::pallet]
pub mod pallet {
	// pallet 的错误
	#[pallet::error]
	pub enum Error<T> {
		/// 值为 None
		NoneValue,
		/// 值已达到最大值，无法进一步递增
		StorageOverflow,
	}

	impl<T: Config> Printable for Error<T> {
		fn print(&self) {
			match self {
				Error::NoneValue => "Invalid Value".print(),
				Error::StorageOverflow => "Value Exceeded and Overflowed".print(),
				_ => "Invalid Error Case".print(),
			}
		}
	}
}
```

```rust
/// 不接受参数，尝试递增存储值，并可能抛出错误
pub fn cause_error(origin) -> dispatch::DispatchResult {
	// 检查它是否已签名并获取签名者。另请参阅：ensure_root 和 ensure_none
	let _who = ensure_signed(origin)?;

	print!("My Test Message");

	match Something::get() {
		None => {
			print(Error::<T>::NoneValue);
			Err(Error::<T>::NoneValue)?
		}
		Some(old) => {
			let new = old.checked_add(1).ok_or(
				{
					print(Error::<T>::StorageOverflow);
					Error::<T>::StorageOverflow
				})?;
			Something::put(new);
			Ok(())
		},
	}
}
```

使用 RUST_LOG 环境变量运行节点二进制文件以打印值。

```sh
RUST_LOG=runtime=debug ./target/release/solo-template-node --dev
```

每次调用运行时函数时，都会在终端或标准输出中打印这些值。

```rust
2020-01-01 tokio-blocking-driver DEBUG runtime  My Test Message  <-- str 默认实现 Printable
2020-01-01 tokio-blocking-driver DEBUG runtime  Invalid Value    <-- 来自 NoneValue 的自定义字符串
2020-01-01 tokio-blocking-driver DEBUG runtime  DispatchError
2020-01-01 tokio-blocking-driver DEBUG runtime  8
2020-01-01 tokio-blocking-driver DEBUG runtime  0                <-- 来自 Error 枚举定义的索引值
2020-01-01 tokio-blocking-driver DEBUG runtime  NoneValue        <-- 包含错误标识符名称的 str
```

请记住，向运行时添加打印函数会增加 Rust 和 Wasm 二进制文件的大小，其中调试代码在生产环境中不需要。

## Substrate 自身的 print 函数

对于旧版用例，Substrate 提供了额外的 `Print` 调试（或跟踪）工具。您可以使用 [`print` 函数](https://paritytech.github.io/substrate/master/sp_runtime/fn.print.html) 来记录运行时执行的状态。

```rust
use sp_runtime::print;

// --snip--
pub fn do_something(origin) -> DispatchResult {
	print!("Execute do_something");

	let who = ensure_signed(origin)?;
	let my_val: u32 = 777;

	Something::put(my_val);

	print!("After storing my_val");

	Self::deposit_event(RawEvent::SomethingStored(my_val, who));
	Ok(())
}
// --snip--
```

使用 `RUST_LOG` 环境变量启动链以查看打印日志。

```sh
RUST_LOG=runtime=debug ./target/release/solo-template-node --dev
```

如果触发错误，则会在终端或标准输出中打印这些值。

```sh
2020-01-01 00:00:00 tokio-blocking-driver DEBUG runtime  Execute do_something
2020-01-01 00:00:00 tokio-blocking-driver DEBUG runtime  After storing my_val
```

## 如果 std

旧版 `print` 函数允许您打印并实现 `Printable` 特性。
但是，在某些旧版情况下，您可能希望执行的操作不仅仅是打印，或者只是为了调试目的而无需使用 Substrate 特定的特性。
[`if_std!` 宏](https://paritytech.github.io/substrate/master/sp_std/macro.if_std.html) 对于这种情况很有用。

使用此宏的一个警告是，其中的代码仅在实际运行运行时的本机版本时才会执行。

```rust
use sp_std::if_std; // 将 if_std! 宏导入作用域。
```

`println!` 语句应该在 `if_std` 宏内。

```rust
#[pallet::call]
impl<T: Config<I>, I: 'static> Pallet<T, I> {
		// --snip--
		pub fn do_something(origin) -> DispatchResult {

			let who = ensure_signed(origin)?;
			let my_val: u32 = 777;

			Something::put(my_val);

			if_std! {
				// 仅在启用 `std` 功能时才编译和执行此代码。
				println!("Hello native world!");
				println!("My value is: {:#?}", my_val);
				println!("The caller account is: {:#?}", who);
			}

			Self::deposit_event(RawEvent::SomethingStored(my_val, who));
			Ok(())
		}
		// --snip--
}
```

每次调用运行时函数时，都会在终端或标准输出中打印这些值。

```sh
$		2020-01-01 00:00:00 Substrate 节点
		2020-01-01 00:00:00   版本 x.y.z-x86_64-linux-gnu
		2020-01-01 00:00:00   作者：Anonymous，2017，2020
		2020-01-01 00:00:00 链规范：开发
		2020-01-01 00:00:00 节点名称：my-node-007
		2020-01-01 00:00:00 角色：AUTHORITY
		2020-01-01 00:00:00 已导入 999 (0x3d7a…ab6e)
		# --snip--
->		Hello native world!
->		My value is: 777
->		The caller account is: d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d (5GrwvaEF...)
		# --snip--
		2020-01-01 00:00:00 已导入 1000 (0x3d7a…ab6e)
