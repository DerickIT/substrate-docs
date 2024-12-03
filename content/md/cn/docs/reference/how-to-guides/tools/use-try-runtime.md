---
title: 使用 try-runtime
description: 如何使用 `try-runtime` 测试存储迁移。
keywords:
  - 存储迁移
  - 测试
  - 运行时
  - 工具
---

`try-runtime` 工具使您能够在将运行时启动到生产环境之前在模拟环境中运行测试和验证操作。
本指南演示了将 `try-runtime` 工具集成到运行时的基本步骤，以便您可以使用它来测试存储迁移。

通常，将 `try-runtime` 工具添加到运行时类似于导入 pallet。
这涉及在正确的位置添加适当的依赖项并更新运行时逻辑以包含 `try-runtime` 功能。
与 pallet 一样，请确保在向运行时添加依赖项时使用 `try-runtime` 工具的适当标签或分支。

## 添加运行时依赖项

1. 打开终端 shell 并更改到节点模板的根目录。

1. 在文本编辑器中打开 `runtime/Cargo.toml` 配置文件。

1. 找到 [dependencies] 部分并注意其他 pallet 是如何导入的。

1. 添加 `frame-try-runtime` 依赖项：

   ```toml
   [dependencies]
	 frame-try-runtime = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
	 ```

	 请注意，您应该为所有 pallet 使用相同的 branch 和版本信息，以确保导入的 pallet 彼此兼容。
   使用来自不同 branch 的 pallet 会导致编译器错误。
   此示例说明如果其他 pallet 使用 `branch = "polkadot-v1.0.0"`，则将 `frame-try-runtime` pallet 添加到 `Cargo.toml` 文件中。

1. 添加 `try-runtime-cli` 依赖项：

	 ```toml
	 try-runtime-cli = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
	 ```

1. 将 `frame-try-runtime` 添加到标准功能列表中：

    ```toml
		[features]
		default = ["std"]
		std = [
			"codec/std",
			"scale-info/std",
			"frame-try-runtime/std",
			...
		]
		```

1. 在 `[features]` 部分中添加或更新 `try-runtime` 以包含运行时中的每个 pallet。

	 ```toml
	 try-runtime = [
		 "frame-executive/try-runtime",
		 "frame-try-runtime",
		 "frame-system/try-runtime",
		 "pallet-aura/try-runtime",
		 "pallet-balances/try-runtime",
		 "pallet-nicks/try-runtime",
		 "pallet-grandpa/try-runtime",
		 "pallet-randomness-collective-flip/try-runtime",
		 "pallet-sudo/try-runtime",
		 "pallet-template/try-runtime",
		 "pallet-timestamp/try-runtime",
		 "pallet-transaction-payment/try-runtime",
		]
	```

## 在运行时 api 中实现 try-runtime

1. 为 try-runtime 功能添加一个配置块。

	 ```rust
	 // 运行时 api 的实现
	 impl_runtime_apis! {
    ...
		/* --snip-- */
    #[cfg(feature = "try-runtime")]
    impl frame_try_runtime::TryRuntime<Block> for Runtime {
        fn on_runtime_upgrade() -> (frame_support::weights::Weight, frame_support::weights::Weight) {
            log::info!("try-runtime::on_runtime_upgrade.");
            // 注意：故意解包：我们不想将错误向后传播，并且希望在这里有一个回溯。如果任何迁移前/后检查失败，我们将立即停止。
            let weight = Executive::try_runtime_upgrade().map_err(|err|{
                log::info!("try-runtime::on_runtime_upgrade failed with: {:?}", err);
                err
            }).unwrap();
            (weight, RuntimeBlockWeights::get().max_block)
        }
        fn execute_block_no_check(block: Block) -> frame_support::weights::Weight {
            Executive::execute_block_no_check(block)
        }
    }
		```

## 添加节点依赖项

1. 在文本编辑器中打开 `node/Cargo.toml` 配置文件。

1. 找到 [dependencies] 部分并注意其他 pallet 是如何导入的。

1. 添加 `frame-try-runtime` 依赖项：

	 ```toml
	 frame-try-runtime = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
	 ```
1. 添加 `try-runtime-cli` 依赖项：

	 ```toml
	 try-runtime-cli = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
	 ```

1. 在 `[features]` 部分中添加或更新 `cli` 和 `try-runtime`。

	 ```toml
	 [features]
	 ...
	 cli = [ "try-runtime-cli" ]
	 try-runtime = [ "node-template-runtime/try-runtime", "try-runtime-cli" ]
	 ...
	 ```

## 添加命令行子命令

1. 在文本编辑器中打开 `node/src/cli.rs` 文件。

```rust
/* --snip-- */
/// 尝试对运行时状态执行某些命令。
#[cfg(feature = "try-runtime")]
TryRuntime(try_runtime_cli::TryRuntimeCmd),

/// 尝试对运行时状态执行某些命令。注意：必须启用 `try-runtime` 功能。
#[cfg(not(feature = "try-runtime"))]
TryRuntime,
/* --snip-- */
```

## 添加命令

1. 在文本编辑器中打开 `node/src/commands.rs` 文件。

2. 添加命令

	 ```rust
	 /* --snip-- */
	 #[cfg(feature = "try-runtime")]
	 Some(Subcommand::TryRuntime(cmd)) => {
		let runner = cli.create_runner(cmd)?;
		runner.async_run(|config| {
			// 只需要运行时或任务管理器来执行 `async_run`。
			let registry = config.prometheus_config.as_ref().map(|cfg| &cfg.registry)let task_manager = sc_service::TaskManager::new(
			  config.tokio_handle.clone(),
				registry,
			).map_err(|e| sc_cli::Error::Service(sc_service::Error::Prometheus(e)))?;
			
		  Ok((cmd.run::<Block, service::ExecutorDispatch>(config), task_manager))
		})
	},

	#[cfg(not(feature = "try-runtime"))]
	Some(Subcommand::TryRuntime) => {
		Err("TryRuntime 在构建节点时未启用。\
		您可以使用 `--features try-runtime` 启用它。".into())
		},
	/* --snip-- */
	```

如果您在工作区中使用自定义 pallet，请确保在工作区的 `pallets/pallet_name/Cargo.toml` 文件中包含了 `try-runtime` 依赖项。

## 使用 try-runtime

使用 try-runtime 工具类似于编写单元测试。

要使用 `try-runtime`：

1. 创建一个 externalities 实例。

1. 在实例上调用 `execute_with`。

<!--
## 示例

## 资源
-->

## 接下来的步骤

- [命令行参考：try-runtime](/reference/command-line-tools/try-runtime/)
- [TryRuntime API](https://crates.parity.io/frame_try_runtime/trait.TryRuntime.html)
