---
title: 共享类型和自定义类型
description:
tutorial: 1
---

现在您已经准备好了模块的基本结构，您可以开始自定义代码以实现它所需的特定于应用程序的接口。这就是应用程序设计发挥作用的地方。
由于 FRAME 是模块化的，并利用 Rust 特征和泛型类型提供的灵活性，您通常可以发现所需的接口已在`frame_system`、`frame_support`或其他预定义模块中提供，您可以直接将它们导入到您的模块中。

## 导入和声明外部接口

对于`collectibles`模块，您知道您需要某种分类账来跟踪谁拥有哪些可收藏物品以及将可收藏物品从一个帐户转移到另一个帐户的方法。
您还希望通过合并随机值来使可收藏物品独一无二。幸运的是，这些都是相当常见的用例，其接口在许多上下文中都很有用，因此它们已在`frame_support`库中定义为特征。

在`frame_support`中，特征是：

- `Currency`用于访问帐户余额、转移操作和 Balance 类型。
- `Randomness`用于访问链上随机值。

您可能还记得，Rust 特征使您可以为特定类型定义可以与其他类型共享的功能。
为了利用这一点，您可以从`frame_support`模块导入`Currency`和`Randomness`特征，然后将它们定义为类型并指定它们在`collectibles`模块中的行为方式。

除了`Currency`和`Randomness`特征之外，`collectibles`模块还需要一个接口来指定单个用户可以拥有的可收藏资产的最大数量。
对于此接口，`collectibles`模块定义了一个`Get<u32>`特征，该特征获取一个`u32`值来指定`MaximumOwned`常量。

通过在`collectibles`模块的配置中包含这些外部接口，`collectibles`模块将能够：

- 访问和操作用户帐户和余额。
- 生成链上随机数。
- 设置单个用户可以拥有的可收藏物品数量的限制。

要导入和声明这些接口：

1. 在代码编辑器中打开`collectibles`模块的`src/lib.rs`文件。

2. 将`frame_support`模块中的`Currency`和`Randomness`特征导入到您的项目中。

    ```rust
    use frame_support::traits::{Currency, Randomness};
    ```

3. 更新 collectibles `Config` 特征以声明 `Currency`、`Randomness` 和 `Get<u32>` 特征。

    ```rust
    #[pallet::config]
    pub trait Config: frame_system::Config {
        type Currency: Currency<Self::AccountId>;
        type CollectionRandomness: Randomness<Self::Hash, Self::BlockNumber>;

        #[pallet::constant]
        type MaximumOwned: Get<u32>;
    }
    ```

4. 通过运行以下命令验证程序是否已编译：

   ```bash
   cargo build --package collectibles
   ```

   目前，您可以忽略有关未使用代码的编译器警告。

## 添加自定义类型

Substrate 支持 Rust 中可用的所有基本类型——例如，bool、u8、u32 和其他常用类型。
Substrate 还提供了一些特定于 Substrate 的常用自定义类型——例如，`AccountId`、`BlockNumber`和`Hash`——您可以通过导入的`frame_system`和`frame_support`模块来使用它们。
您已经为`collectibles`模块导入了一些外部接口来使用。
现在，您可以定义一些自定义属性来描述可收藏物品。
要定义这些自定义属性，您将添加两种自定义类型：

- 一个枚举数据类型，用于列出`Color`属性的可能变体。
- 一个结构体 (struct) 数据类型，用于组合`Collectible`的属性。

### 枚举变体

```rust
 pub enum Color {
            Red,
            Yellow,
            Blue,
            Green
        }
```

`Collectible`结构体包含以下内容：

- `unique_id`是一个 16 字节的无符号整数，用于确保每个可收藏物品都是区块链中的唯一实体。
- `price`是一个`Option`，如果设置了价格则返回`Some(value)`，否则返回`None`以指示可收藏物品不出售。
- `color`是可收藏物品的自定义`Color`类型的变体。
- `owner`用于标识拥有可收藏物品的帐户。

因为我们已经导入了 Currency 特征，所以我们还可以在`collectibles`模块中使用 Currency 接口中的`Balance`和`AccountId`类型。

为`Balance`类型创建一个名为`BalanceOf`的类型别名：

```rust
type BalanceOf<T> =
	<<T as Config>::Currency as Currency<<T as frame_system::Config>::AccountId>>::Balance;
```

在 Collectible 结构体中使用 `BalanceOf` 和 `AccountId`：

```rust
        pub struct Collectible<T: Config> {
            // Unsigned integers of 16 bytes to represent a unique identifier
            pub unique_id: [u8; 16],
            // `None` assumes not for sale
            pub price: Option<BalanceOf<T>>,
            pub color: Color,
            pub owner: T::AccountId,
        }
```

您可以通过运行以下命令来检查代码是否已编译：

```bash
cargo build --package collectibles
```

您应该会看到代码已编译，但有警告，没有错误。
但是，如果您此时尝试使用自定义类型，编译器将报错。
这是因为模块尚未实现自定义类型期望的所有特征。

## 实现所需的特征

Substrate 需要对每种数据类型实现多个特征。
例如，每种数据类型都必须实现`Encode`和`Decode`特征，这些特征使数据能够被序列化和反序列化，以便能够高效地通过网络传输。
幸运的是，您可以使用`#[derive]`宏来实现模块从自定义类型期望的所有特征。
将`#[derive]`宏和以下特征添加到每个自定义类型中：

```rust
#[derive(Clone, Encode, Decode, PartialEq, Copy, RuntimeDebug, TypeInfo, MaxEncodedLen)]
