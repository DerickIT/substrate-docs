---
title: 创世配置
description:
keywords:
---

<div class="warning">
  请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/frame_support/pallet_macros/attr.genesis_build.html">Rust 文档</a> 获取有关此主题的最新文档。
</div>


任何区块链生成的第一个块称为创世块。
与此块关联的哈希是此块之后生成的所有块的顶级父级。

Substrate 节点模板默认情况下会为一部分模块提供创世配置（初始状态）。
当你向运行时添加自定义逻辑时（例如，通过添加预定义或自定义模块），你可能会发现你想要修改创世配置以包含其他存储项或设置不同的初始值。

正如你在 [链规范](/build/chain-spec/) 中了解到的那样，你用于启动节点的链规范决定该节点的创世配置。
但是，链规范不会创建在启动节点时初始化的存储项。
相反，存储项在运行时中包含的模块中定义，如 [运行时存储](/build/runtime-storage/) 中所述。

创建运行时的存储项后，你可以选择是否应将其设置为创世配置的一部分的某个初始值并包含在创世块中。
为了指定要为其设置初始状态的存储项，Substrate 提供了两个专门的 FRAME 属性宏。

你可以用来在链的创世配置中初始化存储项的宏是：

- `#[pallet::genesis_config]` 宏定义 `GenesisConfig` 数据类型并初始化存储项。
- `#[pallet::genesis_build]` 宏构建创世配置。

这些宏与链规范一起使用来定义运行时的初始状态。

## 配置简单的存储值

以下示例演示了将单个存储值添加到 `pallet_template` 的创世配置中。
默认情况下，`pallet_template` 具有一个在创世块中未初始化的存储项。
此示例说明了如何使用 `#[pallet::genesis_config]` 和 `#[pallet::genesis_build]` 宏将存储值的初始值设置为链的创世配置的一部分。

### 在模块中配置宏

要初始化 `pallet_template` 的存储项：

1. 打开一个新的终端 shell 并导航到节点模板的根目录。
2. 在文本编辑器中打开 `pallets/template/src/lib.rs` 文件。
3. 添加 `#[pallet::genesis_config]` 宏并将存储值 `something` 添加为模块的 `GenesisConfig` 存储项。
   例如，将以下宏添加到文件中：
   ```rust
   // 测试创世配置
   #[pallet::genesis_config]
    #[derive(Default)]
    pub struct GenesisConfig {
          pub something: u32,
   }
   ```

````

在此示例中，需要 `#[derive(Default)]` 宏来满足 `frame_support::traits::GenesisBuild` 中的特性绑定要求。

4. 添加 `#[pallet::genesis_build]` 宏：

 ```rust
 #[pallet::genesis_build]
 impl<T: Config> GenesisBuild<T> for GenesisConfig {
    fn build(&self) { }
 }
````

在此示例中，没有要执行的 `build` 函数的特殊处理。

5. 保存更改并关闭文件。

6. 通过运行以下命令来验证模块是否已编译：

   ```bash
   cargo build --package pallet-template
   ```

### 配置链规范

现在你已将模块配置为在创世块中初始化存储值，你可以在链规范中为该存储项设置初始值。

1. 在文本编辑器中打开 `node/src/chain_spec.rs` 文件。
2. 将 `TemplateModuleConfig` 添加到 `node_template_runtime`。

   例如：

   ```rust
   use node_template_runtime::{
      AccountId, AuraConfig, BalancesConfig, RuntimeGenesisConfig, GrandpaConfig, Signature, SudoConfig, SystemConfig, TemplateModuleConfig, WASM_BINARY,
   };
   ```

3. 找到 `GenesisConfig` 并设置 `something` 存储项的初始值。

   例如，在 `node/src/chain_spec.rs` 文件中：

   ```rust
   -> GenesisConfig {
        GenesisConfig {
                system: SystemConfig {
                        // 将 Wasm 运行时添加到存储。
                        code: wasm_binary.to_vec(),
                },

                template_module: TemplateModuleConfig {
                       something: 221u32,
                },

                transaction_payment: Default::default(),
        }
      }
   ```

## 将创世配置添加到运行时

使用 [`#[pallet::genesis_config]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#genesis-config-palletgenesis_config-optional) 宏将 `GenesisConfig` 添加到每个需要的模块后，必须在运行时中包含每个模块的 `Config` 特性，以使运行时能够在创世块中初始化存储项。

然后，将为构建运行时中包含的所有模块的 `GenesisConfig` 类型聚合到该运行时的单个 `RuntimeGenesisConfig` 类型中。

聚合的 `RuntimeGenesisConfig` 实现 [`BuildStorage`](https://paritytech.github.io/substrate/master/sp_runtime/trait.BuildStorage.html) 特性以构建运行时的所有初始存储项。
例如，节点模板运行时会为默认情况下具有 `RuntimeGenesisConfig` 的以下模块构建存储项：

- [配置简单的存储值](#配置简单的存储值)
  - [在模块中配置宏](#在模块中配置宏)
  - [配置链规范](#配置链规范)
- [将创世配置添加到运行时](#将创世配置添加到运行时)
  - [系统模块](#系统模块)
  - [Aura 模块](#aura-模块)
  - [Grandpa 模块](#grandpa-模块)
  - [余额模块](#余额模块)
  - [交易支付模块](#交易支付模块)
  - [Sudo 模块](#sudo-模块)
- [在模块内初始化存储项](#在模块内初始化存储项)

### 系统模块

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig {
		#[serde(with = "sp_core::bytes")]
		pub code: Vec<u8>,
	}
```

### Aura 模块

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig<T: Config> {
		pub authorities: Vec<T::AuthorityId>,
	}
```

### Grandpa 模块

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig {
		pub authorities: AuthorityList,
	}
```

### 余额模块

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig<T: Config<I>, I: 'static = ()> {
		pub balances: Vec<(T::AccountId, T::Balance)>,
	}
```

### 交易支付模块

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig {
		pub multiplier: Multiplier,
	}
```

### Sudo 模块

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig<T: Config> {
		/// Sudo 密钥的 `AccountId`。
		pub key: Option<T::AccountId>,
	}
```

由于这些模块包含带有 `GenesisConfig` 的 `#[pallet::genesis_config]` 宏，并且在运行时中定义了 `Config` 特性，因此它们被聚合到运行时的 [`node_template_runtime::RuntimeGenesisConfig`](https://paritytech.github.io/substrate/master/node_template_runtime/struct.RuntimeGenesisConfig.html) 结构中：

```rust
pub struct RuntimeGenesisConfig {
    pub system: SystemConfig,
    pub aura: AuraConfig,
    pub grandpa: GrandpaConfig,
    pub balances: BalancesConfig,
    pub transaction_payment: TransactionPaymentConfig,
    pub sudo: SudoConfig,
}
```

最终，`RuntimeGenesisConfig` 通过 [`ChainSpec`](https://paritytech.github.io/substrate/master/sc_chain_spec/trait.ChainSpec.html) 特性公开。

有关 Substrate 创世存储配置的更完整示例，请参阅 [与 Substrate 代码库一起提供的链规范](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/bin/node/cli/src/chain_spec.rs)。

## 在模块内初始化存储项

你可以使用 [`#[pallet::genesis_build]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#genesis-build-palletgenesis_build-optional) 宏来定义模块本身内存储项的初始状态。
在模块内定义创世配置允许你访问模块的私有函数。

以下示例演示了使用 `#[pallet::genesis_config]` 和 `#[pallet::genesis_build]` 来设置存储项的初始值。
在此示例中，有两个存储项：

- 成员帐户标识符列表。
- 指定列表中某个成员作为主要成员的特定帐户标识符。

此示例的宏和数据类型在 `my_pallet/src/lib.rs` 文件中定义：

```rust
#[pallet::genesis_config]
struct GenesisConfig {
    members: Vec<T::AccountId>,
    prime: T::AccountId,
}

#[pallet::genesis_build]
impl<T: Config> GenesisBuild<T> for GenesisConfig {
    fn build(&self) {
        Pallet::<T>::initialize_members(&self.members);
        SomeStorageItem::<T>::put(self.prime);
    }
}
```

创世配置在 `node/src/chain_spec.rs` 文件中定义：

```rust
GenesisConfig {
    my_pallet: MyPalletConfig {
        members: LIST_OF_IDS,
        prime: ID,
    },
}
```

你还可以使用 `genesis_build` 宏来定义不绑定到特定存储项的 `GenesisConfig` 属性。
如果你想调用模块中设置多个存储项的私有辅助函数，或者调用模块中包含的其他模块中定义的函数，这将非常有用。
例如，使用名为 `intitialize_members` 的虚构私有函数，代码可能如下所示：

在 `my_pallet/src/lib.rs` 中：

```rust
#[pallet::genesis_config]
struct GenesisConfig {
    members: Vec<T::AccountId>,
    prime: T::AccountId,
}

#[pallet::genesis_build]
impl<T: Config> GenesisBuild<T> for GenesisConfig {
    fn build(&self) {
        Pallet::<T>::initialize_members(&config.members);
        SomeStorageItem::<T>::put(self.prime);
    }
}
```

在 `chain_spec.rs` 中：

```rust
GenesisConfig {
    my_pallet: MyPalletConfig {
        members: LIST_OF_IDS,
        prime: ID,
    },
}
