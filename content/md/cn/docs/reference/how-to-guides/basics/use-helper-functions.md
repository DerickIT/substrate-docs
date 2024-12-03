---
title: 使用辅助函数
description: 本简短指南介绍了创建和重复使用辅助函数的示例，以在模块代码中执行常见的“验证”检查。
keywords:
  - 基础
  - 运行时
---

本简短指南介绍了创建和重复使用辅助函数的示例，以在模块代码中执行常见的“验证”检查。

有时，模块中的一个可调度函数会重复使用对其他可调度函数通用的逻辑。
在这种情况下，将此逻辑重构到自己的私有函数中很有用。
在其他情况下，随着执行可调度函数内各种检查的代码量增加，可调度函数变得越来越难以阅读。
在这两种情况下，使用无法从模块外部访问的辅助函数都是优化代码可读性和可重用性的有用工具。
在本指南中，我们将看到如何创建一个加法辅助函数，该函数检查算术溢出，并且可以在任何可调度函数中重复使用。

### 创建辅助函数

我们将要提到的辅助函数称为 `fn _adder`。
它检查在添加两个 `u32` 类型的整数时是否发生溢出。

它接受两个 `u32` 整数，使用 `checked_add` 和 `ok_or` 检查是否发生溢出。
如果发生溢出，它将返回一个错误；否则它将返回结果。

以下是它作为辅助函数的样子。
这将放在模块的底部：

```rust
impl<T: Config> Pallet<T> {
    fn _adder(num1: u32, num2: u32) -> Result<u32, &'static str> {
        num1.checked_add(num2).ok_or("Overflow when adding")
    }
}
```

## 在可调度函数中使用它

确定在执行加法时需要检查溢出的位置。
使用辅助函数，而不是重写相同的代码。
以下是一个简单的可调度函数示例，该函数允许签名外部函数将一个值添加到现有的存储值：

```rust
// 从运行时外部可调用的外部函数。
#[pallet::call]
impl<T: Config> Pallet<T> {
  #[pallet::weight(0)]
  fn add_value(
    origin: OriginFor<T>,
    val_to_add: u32
    ) -> DispatchResultWithPostInfo {
      let _ = ensure_signed(origin)?;

      ensure!(val_to_add <= T::MaxAddend::get(), "value must be <= maximum add amount constant");

      // 获取先前值
      let c_val = SingleValue::<T>::get();

      // 检查添加新值时是否溢出
      let result = _adder(c_val, val_to_add);

      <SingleValue<T>>::put(result);
      Self::deposit_event(Event::Added(c_val, val_to_add, result));
      Ok(().into())
  }
}
```

## 示例

- [example-offchain-worker](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs): 此模块的可调度函数中使用的 `add_price` 辅助函数。

## 资源

- [`checked_add` 在 `num` crate 中](https://docs.rs/num/0.4.0/num/traits/trait.CheckedAdd.html)
- [`ok_or` 在 `Option` crate 中](https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or)
