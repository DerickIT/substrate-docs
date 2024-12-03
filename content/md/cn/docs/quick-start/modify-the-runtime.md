---
title: 修改运行时
description: 对默认节点模板进行简单更改以创建自定义运行时。
keywords:
---

在 [探索代码](/quick-start/explore-the-code/) 中，您了解了构成默认节点模板的清单文件和 Rust 模块。
现在您已经对运行时源代码的外观有了一般了解，让我们看看如何轻松地进行一些简单更改来自定义运行时。

在此简单演示中，您将执行以下操作：

- 添加一个具有您要使用的功能的pallet。
- 更改一些常量值。
- 更新运行时版本。
- 重新编译运行时以包含您的更改。
- 提交交易以更新存储在链上的运行时。

您还将看到另一个使用 Polkadot-JS API 的应用程序，以及如何使用该应用程序的托管版本来查看链状态并提交交易。

## 在您开始之前

当您使用 `--dev` 命令行选项在开发模式下运行节点时，它会从具有第一个区块的干净状态开始。
为了更好地说明如何修改和更新运行时，您应该使用其默认运行时重新启动默认节点模板，以便它开始生成区块。

要使用默认运行时重新启动节点：

1. 在您的计算机上打开一个终端外壳。

2. 更改到您编译 Substrate 节点模板的根目录。
3. 通过运行以下命令在开发模式下启动本地节点：

   ```bash
   cargo run --release -- --dev
   ```

启动节点后，您可以使用使用 Polkadot-JS API 构建的基于浏览器的应用程序连接到它。

要连接到正在运行的节点：

1. 在 Chrome 或基于 Chromium 的浏览器中打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer)。

   如果您使用更严格的浏览器（例如 Firefox），您可能会发现 Polkadot/Substrate 门户和节点之间的连接被阻止。

2. 连接到开发网络和默认本地节点端点 `127.0.0.1:9944` （如果需要）。

   在大多数情况下，[Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer) 会自动初始化与正在运行的本地节点的连接。
   如果需要，请单击**未知**以显示网络选择菜单，然后选择**开发**和**本地节点**，然后单击**切换**。

3. 注意，在开发下，节点模板版本是默认版本 100。

   ![节点模板默认版本](/media/images/docs/quickstart-100.png)

## 添加pallet

使用 Substrate 和 FRAME 开始构建的最常见方法是添加pallet，方法是导入现有库中的pallet或创建自己的pallet。
从头开始创建自己的pallet并不难，但需要更多工作来设计应用程序逻辑、存储要求、错误处理等。
为了简单起见，让我们通过从现有库中导入一个pallet来添加一个pallet。

默认情况下，节点模板不包含 [实用程序pallet](https://paritytech.github.io/substrate/master/pallet_utility/index.html)。
如果此pallet包含您要使用的函数，则可以将其添加到默认运行时。

要添加实用程序pallet：

1. 在您的计算机上打开第二个终端外壳，并更改到节点模板根目录。
2. 在代码编辑器中打开运行时清单—`runtime/Cargo.toml`。

3. 找到 `[dependencies]` 部分，并将实用程序pallet添加为依赖项。

   例如，您应该添加类似于以下内容的单行。

   ```toml
   pallet-utility = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }
   ```

   请务必将 `branch = "polkadot-vX.Y.Z"` 替换为用于其他pallet的 Polkadot 分支。

   您可以复制任何现有的pallet依赖项作为模型，以确保 `pallet-utility` 依赖项的分支设置与所有其他pallet的分支设置相同。

4. 找到 `[features]` 部分，并将实用程序pallet添加到标准二进制文件的默认功能列表中。

   例如：

   ```toml
   [features]
   default = ["std"]
   std = [
      ...
      "pallet-utility/std",
      ...
   ]
   ```

   您将在 [Rust 和 WebAssembly](/build/build-process) 中详细了解如何为标准和 WebAssembly 二进制文件构建功能。

5. 保存您的更改并关闭 `Cargo.toml` 文件。

6. 在代码编辑器中打开 `runtime/src/lib.rs` 文件。

7. 添加实用程序pallet的 `Config` 特性的实现。

   例如：

   ```rust
   impl pallet_utility::Config for Runtime {
      type RuntimeEvent = RuntimeEvent;
      type RuntimeCall = RuntimeCall;
      type PalletsOrigin = OriginCaller;
      type WeightInfo = pallet_utility::weights::SubstrateWeight<Runtime>;
   }
   ```

   每个pallet都有一个 `Config` 特性，用于它所需的特定参数和类型。
   您始终可以查看pallet的 Rust 文档以了解有关其配置要求的更多信息。
   例如，您可以查看 [pallet-utility](https://paritytech.github.io/substrate/master/pallet_utility/index.html) 的 Rust 文档。

8. 在 `construct_runtime!` 宏中添加实用程序pallet。

   例如：

   ```rust
   construct_runtime!(
     pub struct Runtime
     where
        Block = Block,
        NodeBlock = opaque::Block,
        UncheckedExtrinsic = UncheckedExtrinsic
     {
            System: frame_system,
            RandomnessCollectiveFlip: pallet_randomness_collective_flip,
            Timestamp: pallet_timestamp,
            Aura: pallet_aura,
            ...
            Utility: pallet_utility, // 添加此行
            ...
     }
   ```

   您可以在 [FRAME 宏](/reference/frame-macros/) 和 [运行时构建宏](/reference/frame-macros/#runtime-construction-macros) 中详细了解 `construct_runtime` 宏的工作原理。

## 更改常量值

默认情况下，节点模板中的余额pallet定义了一个 `EXISTENTIAL_DEPOSIT` 常量。
`EXISTENTIAL_DEPOSIT` 表示帐户必须具有的最小余额，才能被视为有效的活动帐户。
默认情况下，该常量定义为具有 500 值的 128 位无符号整数类型。
为了简单起见，您将更改此常量的值，从 500 更改为 1000。

要更新常量值：

1. 在代码编辑器中打开 `runtime/src/lib.rs` 文件。

2. 找到余额pallet的 `EXISTENTIAL_DEPOSIT`。

   ```text
   /// 最低存款。
   pub const EXISTENTIAL_DEPOSIT: u128 = 500;
   ```

3. 更新 `EXISTENTIAL_DEPOSIT` 的值。

   ```rust
   pub const EXISTENTIAL_DEPOSIT: u128 = 1000 // 更新此值。
   ```

## 更新运行时版本

默认情况下，节点模板使用 `spec_version` 和 100 的值在 `VERSION` 常量中标识默认运行时版本。
为了表明您已对默认运行时进行了更改，您将 `spec_version` 从 100 更改为 101。

请注意，对于您在_快速入门_中对默认运行时所做的更改，更新 `spec_version` 并非严格要求。
但是，通过更新版本，您可以看到执行无分叉升级所涉及的基本步骤。

要更新运行时版本：

1. 在代码编辑器中打开 `runtime/src/lib.rs` 文件。
2. 找到 `runtime_version` 宏。

   ```text
   #[sp_version::runtime_version]
   pub const VERSION: RuntimeVersion = RuntimeVersion {
        spec_name: create_runtime_str!("node-template"),
        impl_name: create_runtime_str!("node-template"),
        authoring_version: 1,
        spec_version: 100,
        impl_version: 1,
        apis: RUNTIME_API_VERSIONS,
        transaction_version: 1,
        state_version: 1,
   };
   ```

3. 更新 `spec_version` 以指定新的运行时版本。

   ```rust
   spec_version: 101,  // 将 spec_version 从 100 更改为 101
   ```

4. 保存您的更改并关闭 `runtime/src/lib.rs` 文件。

此时，您已修改了运行时代码并更改了版本信息。
但是，正在运行的节点仍在使用先前编译的运行时版本。
如果您仍然使用 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer) 连接到正在运行的节点，您会发现节点模板版本仍然是默认版本 100，并且余额常量 existentialDeposit 的 [链状态](https://polkadot.js.org/apps/#/chainstate/constants) 仍然是 500。

![链状态](/media/images/docs/quickstart-org-chainstate.png)

## 重新编译运行时

在您可以更新节点模板以使用您修改的运行时之前，您必须重新编译运行时。

要重新编译运行时包：

1. 打开第二个终端外壳，并更改到您编译节点模板的根目录。
2. 通过运行以下命令重新编译运行时：

   ```shell
   cargo build --release --package node-template-runtime
   ```

   `--release` 命令行选项需要更长的编译时间。
   但是，它会生成一个更适合提交到区块链网络的较小构建工件。
   存储优化对于任何区块链都至关重要。
   使用此命令，构建工件将输出到 `target/release` 目录。
   WebAssembly 构建工件位于 `target/release/wbuild/node-template-runtime` 目录中。
   例如，如果您列出 `target/release/wbuild/node-template-runtime` 目录的内容，您应该看到以下 WebAssembly 工件：

   ```text
   node_template_runtime.compact.compressed.wasm
   node_template_runtime.compact.wasm
   node_template_runtime.wasm
   ```

## 提交交易

现在您有一个更新的 WebAssembly 对象，它描述了修改后的运行时。
但是，正在运行的节点尚未使用升级的运行时。
要更新存储在链上的运行时，您必须提交一个更改要使用的 WebAssembly 对象的交易。

要更新运行时：

1. 在 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer) 中，单击**开发人员**，然后选择**外部函数**。

2. 选择管理**Alice** 帐户。

3. 选择**sudo** pallet和**sudoUncheckedWeight(call, weight)** 函数。
4. 选择**system** 和**setCode(code)** 作为使用 Alice 帐户进行调用的方法。

5. 单击**文件上传**，然后选择或拖放您为更新的运行时生成的紧凑压缩 WebAssembly 文件—`node_template_runtime.compact.compressed.wasm`。

   例如，导航到 `target/release/wbuild/node-template-runtime` 目录，然后选择 `node_template_runtime.compact.compressed.wasm` 作为要上传的文件。

6. 将两个**权重**参数都保留为默认值 `0`。

   ![运行时升级设置](/media/images/docs/tutorials/forkless-upgrade/set-code-transaction.png)

7. 单击**提交交易**。

8. 查看授权，然后单击**签名并提交**。

## 验证修改后的运行时

交易包含在区块中后，您可以验证您是否正在使用修改后的运行时。

要验证您的更改：

1. 在 [Polkadot/Substrate 门户](https://polkadot.js.org/apps) 中，单击**网络**，然后选择**资源管理器**，以查看是否已成功发生 `sudo.Sudid` 事件。

   ![成功的 sudo 事件](/media/images/docs/tutorials/forkless-upgrade/set-code-sudo-event.png)

2. 检查节点模板版本现在是否为 `101`。

   例如：

   ![更新的运行时版本为 101](/media/images/docs/quickstart-101.png)

3. 单击**开发人员**，然后选择**外部函数**。
4. 单击**提交以下外部函数**，然后滚动到列表底部，以验证**utility** pallet是否可用作为选项。

   ![实用程序pallet](/media/images/docs/quickstart-utility-pallet.png)

5. 单击**开发人员**，选择**链状态**，然后单击 [常量](https://polkadot.js.org/apps/#/chainstate/constants?rpc=ws://127.0.0.1:9944)。
6. 选择**balances** pallet，选择**existentialDeposit**，然后单击**+** 以查询常量值。

   ![验证常量值更改](/media/images/docs/quickstart-chain-state.png)

## 下一步去哪里

验证更改后，您知道您有一个自定义版本的节点模板正在运行，并且已成功升级本地节点以使用您修改的运行时。

这是一个相当大的成就，但您还可以做更多的事情。
要深入了解概念和核心组件，请查看 [学习](/learn/) 部分中的主题，或通过探索 [构建](/build/) 部分中的主题来开始构建您已经学到的知识。
