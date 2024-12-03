---
title: 单元测试
description: 说明运行时逻辑的基本单元测试。
keywords:
---

在构建运行时逻辑时，您需要定期测试逻辑是否按预期工作。
您可以使用 Rust 提供的 [单元测试框架](https://doc.rust-lang.org/rust-by-example/testing/unit_testing.html) 为运行时创建单元测试。
创建了一个或多个单元测试后，您可以使用 `cargo test` 命令来执行测试。
例如，您可以通过运行以下命令来运行为运行时创建的所有测试：

```shell
cargo test
```

有关使用 Rust cargo test 命令和测试框架的更多信息，请运行以下命令：

```shell
cargo help test
```

## 在模拟运行时中测试 pallet 日志

除了可以使用 Rust 测试框架进行的单元测试之外，您还可以通过构建模拟运行时环境来验证运行时中的逻辑。
配置类型 `Test` 定义为 Rust 枚举，其中包含模拟运行时中使用的每个 pallet 配置特性的实现。

```rust
frame_support::construct_runtime!(
 pub enum Test where
  Block = Block,
  NodeBlock = Block,
  UncheckedExtrinsic = UncheckedExtrinsic,
 {
  System: frame_system::{Pallet, Call, Config, Storage, Event<T>},
  TemplateModule: pallet_template::{Pallet, Call, Storage, Event<T>},
 }
);

impl frame_system::Config for Test {
 // -- snip --
 type AccountId = u64;
}
```

如果 `Test` 实现 `pallet_balances::Config`，则赋值可能会对 `Balance` 类型使用 `u64`。
例如：

```rust
impl pallet_balances::Config for Test {
 // -- snip --
 type Balance = u64;
}
```

通过将 `pallet_balances::Balance` 和 `frame_system::AccountId` 分配给 `u64`，测试帐户和余额只需要在模拟运行时中跟踪 `(AccountId: u64, Balance: u64)` 映射。

## 在模拟运行时中测试存储

[`sp-io`](https://paritytech.github.io/substrate/master/sp_io/index.html) crate 公开了 [`TestExternalities`](https://paritytech.github.io/substrate/master/sp_io/type.TestExternalities.html) 实现，您可以使用它在模拟环境中测试存储。
它是 [`substrate_state_machine`](https://paritytech.github.io/substrate/master/sp_state_machine/index.html) 中基于内存的哈希映射外部函数实现的类型别名，称为 [`TestExternalities`](https://paritytech.github.io/substrate/master/sp_state_machine/struct.TestExternalities.html)。

以下示例演示了定义名为 `ExtBuilder` 的结构体以构建 `TestExternalities` 实例，并将区块号设置为 1。

```rust
pub struct ExtBuilder;

impl ExtBuilder {
 pub fn build(self) -> sp_io::TestExternalities {
  let mut t = system::GenesisConfig::default().build_storage::<TestRuntime>().unwrap();
  let mut ext = sp_io::TestExternalities::new(t);
  ext.execute_with(|| System::set_block_number(1));
  ext
 }
}
```

要在单元测试中创建测试环境，会调用 build 方法以使用默认创世配置生成 `TestExternalities`。

```rust
#[test]
fn fake_test_example() {
 ExtBuilder::default().build_and_execute(|| {
  // ...测试逻辑...
 });
}
```

[Externalities](https://paritytech.github.io/substrate/master/sp_externalities/index.html) 的自定义实现允许您构建提供对外部节点功能的访问的运行时环境。
另一个示例可以在 [`offchain`](https://paritytech.github.io/substrate/master/sp_core/offchain/index.html) 模块中找到。
`offchain` 模块维护其自己的 [Externalities](https://paritytech.github.io/substrate/master/sp_core/offchain/trait.Externalities.html) 实现。

## 在模拟运行时中测试事件

除了存储之外，测试从链中发出的事件也很重要。
假设您使用 `generate_deposit` 宏使用 `deposit_event` 的默认生成，则所有 pallet 事件都存储在 `system` / `events` 键下，其中包含一些作为 [`EventRecord`](https://paritytech.github.io/substrate/master/frame_system/struct.EventRecord.html) 的额外信息。

可以使用 `System::events()` 直接访问和迭代这些事件记录，但 system pallet 中还定义了一些辅助方法，可在测试中使用，即 [`assert_last_event`](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#method.assert_last_event) 和 [`assert_has_event`](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#method.assert_has_event)。

```rust
fn fake_test_example() {
 ExtBuilder::default().build_and_execute(|| {
  System::set_block_number(1);
  // ...发出 FakeEvent1 然后是 FakeEvent2 的测试逻辑...
  System::assert_has_event(Event::FakeEvent1{}.into())
  System::assert_last_event(Event::FakeEvent2 { data: 7 }.into())
  assert_eq!(System::events().len(), 2);
 });
}
```

需要注意的是：

- 创世块不会发出事件，因此需要设置区块号才能通过此测试。
- 实例化 pallet 事件后，您需要有一个 `.into()`，这会将其转换为通用事件。

### 高级事件测试

在测试 pallet 中的事件时，通常您只对 pallet 自己发出的事件感兴趣。
以下辅助函数会过滤事件，只包含 pallet 发出的事件，并将它们转换为自定义事件类型。
此类辅助函数通常放在 `mock.rs` 文件中，用于在模拟运行时中进行测试。

```rust
fn only_example_events() -> Vec<super::Event<Runtime>> {
 System::events()
  .into_iter()
  .map(|r| r.event)
  .filter_map(|e| if let RuntimeEvent::TemplateModule(inner) = e { Some(inner) } else { None })
  .collect::<Vec<_>>();
}
```

此外，如果您的测试执行按顺序发出事件的操作，您可能只想查看自上次检查以来发生的事件。
以下示例利用了前面的辅助函数。

```rust
parameter_types! {
 static ExamplePalletEvents: u32 = 0;
}

fn example_events_since_last_call() -> Vec<super::Event<Runtime>> {
 let events = only_example_events();
 let already_seen = ExamplePalletEvents::get();
 ExamplePalletEvents::set(events.len() as u32);
 events.into_iter().skip(already_seen as usize).collect()
}
```

您可以在 [提名池](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/nomination-pools/src/mock.rs) 或 [staking](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/staking/src/mock.rs) 的测试中找到此类事件测试的示例。
如果您使用此新函数重写之前的事件测试，则生成的代码如下所示：

```rust
fn fake_test_example() {
 ExtBuilder::default().build_and_execute(|| {
  System::set_block_number(1);
  // ...发出 FakeEvent1 的测试逻辑...
  assert_eq!(
   example_events_since_last_call(),
   vec![Event::FakeEvent1{}]
  );
  // ...发出 FakeEvent2 的测试逻辑...
  assert_eq!(
   example_events_since_last_call(),
   vec![Event::FakeEvent2{}]
  );
 });
}
```

## 创世配置

在前面的示例中，`ExtBuilder::build()` 方法使用默认创世配置来构建模拟运行时环境。
在许多情况下，在测试之前设置存储很方便。
例如，您可能希望在测试之前预先播种帐户余额。

在 `frame_system::Config` 的实现中，`AccountId` 和 `Balance` 都设置为 `u64`。
您可以将 `(u64, u64)` 对放入 `balances` vec 中，以将 `(AccountId, Balance)` 对作为帐户余额播种。
例如：

```rust
impl ExtBuilder {
 pub fn build(self) -> sp_io::TestExternalities {
  let mut t = frame_system::GenesisConfig::default().build_storage::<Test>().unwrap();
  pallet_balances::GenesisConfig::<Test> {
   balances: vec![
    (1, 10),
    (2, 20),
    (3, 30),
    (4, 40),
    (5, 50),
    (6, 60)
   ],
  }
   .assimilate_storage(&mut t)
   .unwrap();

  let mut ext = sp_io::TestExternalities::new(t);
  ext.execute_with(|| System::set_block_number(1));
  ext
 }
}
```

在此示例中，帐户 1 的余额为 10，帐户 2 的余额为 20，依此类推。

用于定义 pallet 创世配置的确切结构取决于 pallet `GenesisConfig` 结构体定义。
例如，在 Balances pallet 中，它定义为：

```rust
pub struct GenesisConfig<T: Config<I>, I: 'static = ()> {
 pub balances: Vec<(T::AccountId, T::Balance)>,
}
```

## 区块生产

模拟区块生产对于验证预期行为在区块生产过程中是否成立很有用。

一种简单的方法是在所有模块的 `on_initialize` 和 `on_finalize` 调用之间递增 System 模块的区块号，其中 `System::block_number()` 作为唯一的输入。
虽然对运行时代码来说缓存对存储或系统模块的调用很重要，但测试环境脚手架应优先考虑可读性，以方便将来的维护。

```rust
fn run_to_block(n: u64) {
 while System::block_number() < n {
  if System::block_number() > 0 {
   ExamplePallet::on_finalize(System::block_number());
   System::on_finalize(System::block_number());
  }
  System::reset_events();
  System::set_block_number(System::block_number() + 1);
  System::on_initialize(System::block_number());
  ExamplePallet::on_initialize(System::block_number());
 }
}
```

如果 pallet 特性实现了 `frame_support::traits::{OnInitialize, OnFinalize}` 特性以分别在每个区块之前和之后执行运行时方法中编码的逻辑，则仅从 `ExamplePallet` 调用 `on_finalize` 和 `on_initialize` 方法。

然后按以下方式调用此函数。

```rust
#[test]
fn my_runtime_test() {
 with_externalities(&mut new_test_ext(), || {
  assert_ok!(ExamplePallet::start_auction());
  run_to_block(10);
  assert_ok!(ExamplePallet::end_auction());
 });
}
```

## 接下来的步骤

<!-- TODO NAV.YAML -->
<!-- add these back -->
<!-- - [设置 pallet 的测试](/reference/how-to-guides/testing/) -->
