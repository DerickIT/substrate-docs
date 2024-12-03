---
title: 升级正在运行的网络
description: 说明您可以更新正在运行的节点的方法。
keywords:
  - 无分叉升级
  - 运行时升级
---

与许多区块链不同，Substrate 开发框架支持对区块链核心的运行时进行**无分叉升级**。
大多数区块链项目需要对代码库进行[硬分叉](/reference/glossary/#fork)才能支持新功能的持续开发或对现有功能的增强。
使用 Substrate，您可以部署增强的运行时功能（包括重大更改），而无需硬分叉。
因为运行时的定义本身就是基于 Substrate 的链状态中的一个元素，所以网络参与者可以通过在事务中调用 [`set_code`](https://paritytech.github.io/substrate/master/frame_system/pallet/enum.Call.html#variant.set_code) 函数来更新此值。
因为对运行时状态的更新是使用区块链的共识机制和加密保证进行验证的，所以网络参与者可以使用区块链本身来分发更新的或扩展的运行时逻辑，而无需分叉链或发布新的区块链客户端。

本教程说明了如何在不创建代码库的分支或停止链的进度的情况下升级运行时。
在本教程中，您将对正在运行的网络节点上的 Substrate 运行时进行以下更改：

- 增加运行时的 `spec_version`。
- 将 Utility 模块添加到运行时。
- 增加网络帐户的最低余额。

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已完成 [构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/) 教程，并在本地安装了 Substrate 节点模板。

- 您已查看 [将模块添加到运行时](/tutorials/build-application-logic/add-a-pallet) 以了解如何将新模块添加到运行时。

## 教程目标

通过完成本教程，您将实现以下目标：

- 使用 Sudo 模块模拟链升级的治理。

- 升级正在运行的节点的运行时以包含新的模块。

- 提交事务以将修改后的运行时上传到正在运行的节点。

## 使用 Sudo 授权升级

通常，运行时升级是通过治理来管理的，社区成员投票批准或拒绝升级提案。
为了代替治理，本教程使用 Sudo 模块和 `Root` 原点来标识具有升级运行时权限的运行时管理员。
只有这个根级管理员才能通过调用 `set_code` 函数来更新运行时。
Sudo 模块允许您通过指定具有根级管理权限的帐户来使用 `Root` 原点调用 `set_code` 函数。

默认情况下，节点模板的链规范文件指定 `alice` 开发帐户是 Sudo 管理帐户的所有者。
因此，本教程使用 `alice` 帐户执行运行时升级。

### 运行时升级的资源核算

调度到 Substrate 运行时的函数调用始终与 [权重](/reference/glossary/#weight) 相关联，以说明资源使用情况。
FRAME 系统模块设置了这些事务可以使用的区块长度和区块权重的边界。
但是，`set_code` 函数的设计意图是使用可以放入区块中的最大权重。
强制运行时升级以使用整个区块可以防止同一区块中的事务在不同版本的运行时上执行。

`set_code` 函数的权重注释还指定该函数位于 `Operational` 类中，因为它提供了网络功能。
被标识为操作的函数调用：

- 可以使用区块的整个权重限制。
- 具有最高优先级。
- 免于支付交易费用。

### 管理资源核算

在本教程中，[`sudo_unchecked_weight`](https://paritytech.github.io/substrate/master/pallet_sudo/pallet/enum.Call.html#variant.sudo_unchecked_weight) 函数用于调用运行时升级的 `set_code` 函数。
`sudo_unchecked_weight` 函数与 `sudo` 函数相同，只是它支持一个额外的参数来指定要用于调用的权重。
此参数使您可以绕过资源核算安全措施，为调度 `set_code` 函数的调用指定权重为零。
此设置允许区块花费_无限的时间来计算_，以确保运行时升级不会失败，无论操作多么复杂。
它可以花费所有需要的时间来成功或失败。

## 将 Utility 模块添加到运行时

默认情况下，节点模板在其运行时中不包含 [Utility 模块](https://paritytech.github.io/substrate/master/pallet_utility/index.html)。
为了说明运行时升级，您可以将 Utility 模块添加到正在运行的节点。

### 启动本地节点

本教程说明了如何更新正在运行的节点，因此第一步是使用当前运行时启动本地节点。

要使用当前运行时启动节点：

1. 在您的计算机上打开终端 shell。

1. 更改到您编译 Substrate 节点模板的根目录。

1. 通过运行以下命令以开发模式启动先前编译的本地节点：

   ```bash
   cargo run --release -- --dev
   ```

   让此节点继续运行。
   您可以编辑和重新编译以升级运行时，而无需停止或重新启动正在运行的节点。

1. 在浏览器中打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer) 并连接到本地节点。

1. 点击最左边的下拉菜单以选择网络。

   ![选择网络](/media/images/docs/tutorials/forkless-upgrade/polkadot-js-select-network.png)

1. 在**开发**下，选择**本地节点**，然后点击**切换**。

   ![选择网络](/media/images/docs/tutorials/forkless-upgrade/select-local-node.png)

   在左上角，请注意节点模板版本是默认版本 100。

   ![节点模板版本](/media/images/docs/tutorials/forkless-upgrade/default-version.png)

### 将 Utility 模块添加到运行时依赖项

要更新运行时的依赖项以包含 Utility 模块：

1. 打开第二个终端 shell 窗口或选项卡。

1. 更改到您编译 Substrate 节点模板的根目录。

1. 在文本编辑器中打开 `runtime/Cargo.toml` 文件。

1. 找到 `[dependencies]` 部分。

   例如：

   ```text
   [dependencies]
   codec = { package = "parity-scale-codec", version = "3.0.0", default-features = false, features = ["derive"] }
   scale-info = { version = "2.1.1", default-features = false, features = ["derive"] }

   pallet-aura = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0" }
   ```

1. 将 Utility 模块添加为依赖项。

   例如，添加一行，其中包含以下字段：

   ```toml
   pallet-utility = {
      version = "4.0.0-dev",
      default-features = false,
      git = "https://github.com/paritytech/polkadot-sdk.git",
      branch = "polkadot-v1.0.0"
   }
   ```

1. 找到 `[features]` 部分和标准二进制文件的默认功能列表。

   例如：

   ```text
   [features]
   default = ["std"]
   std = [
      "frame-try-runtime?/std",
      "frame-system-benchmarking?/std",
      "frame-benchmarking?/std",
      "codec/std",
      "scale-info/std",
   ```

1. 将 Utility 模块添加到列表中。

   ```toml
   "pallet-utility/std",
   ```

1. 保存更改并关闭 `Cargo.toml` 文件。


### 添加 Utility 模块配置

要添加 Utility 类型和配置特征：

1. 在文本编辑器中打开 `runtime/src/lib.rs` 文件。

1. 添加 Utility 模块的 Config 特征的实现。

   ```rust
   impl pallet_utility::Config for Runtime {
      type RuntimeEvent = RuntimeEvent;
      type RuntimeCall = RuntimeCall;
      type PalletsOrigin = OriginCaller;
      type WeightInfo = pallet_utility::weights::SubstrateWeight<Runtime>;
   }
   ```

1. 找到 `construct_runtime!` 宏。

   ```text
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
   ```

1. 在 `construct_runtime!` 宏中添加 Utility 模块。

   ```rust
   Utility: pallet_utility,
   ```

1. 找到 `runtime_version` 宏。

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

1. 更新 Balances 模块的 EXISTENTIAL_DEPOSIT 值。

   ```rust
   pub const EXISTENTIAL_DEPOSIT: u128 = 1000 // 更新此值。
   ```

   此更改增加了帐户需要存入的最低余额才能被视为有效的活动帐户。
   此更改不会删除余额在 500 到 1000 之间的任何帐户。
   删除帐户需要存储迁移。
   有关升级数据存储的信息，请参阅 [存储迁移](/maintain/runtime-upgrades/#storage-migrations)

1. 增加 [`spec_version`](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html#structfield.spec_version) 以指定新的运行时版本。

   ```rust
   spec_version: 101,  // 将 spec_version 从 100 更改为 101
   ```

   `runtime_version` 的字段指定以下信息：

   - `spec_name` 指定运行时的名称。
   - `impl_name` 指定外部节点客户端的名称。
   - `authoring_version` 指定 [区块作者](/reference/glossary#author) 的版本。
   - `spec_version` 指定运行时的版本。
   - `impl_version` 指定外部节点客户端的版本。
   - `apis` 指定支持的 API 列表。
   - `transaction_version` 指定 [可调度函数](/reference/glossary#dispatch) 接口的版本。
   - `state_version` 指定运行时使用的键值 trie 数据结构的版本。

   要升级运行时，您必须_增加_`spec_version`。
   有关更多信息，请参阅 [FRAME 系统](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/system/src/lib.rs) 模块和 `can_set_code` 方法。

1. 保存更改并关闭 `runtime/src/lib.rs` 文件。

### 重新编译并连接到本地节点

1. 验证本地节点是否在第一个终端中继续运行。

1. 在您更新了运行时 `Cargo.toml` 和 `lib.rs` 文件的第二个终端中，通过运行以下命令重新编译运行时

   ```shell
   cargo build --release --package node-template-runtime
   ```

   `--release` 命令行选项需要更长的编译时间。
   但是，它会生成一个更小的构建工件，更适合提交到区块链网络。
   对于任何区块链，存储优化都是_至关重要的_。
   使用此命令，构建工件将输出到 `target/release` 目录。
   WebAssembly 构建工件位于 `target/release/wbuild/node-template-runtime` 目录中。
   例如，您应该看到以下 WebAssembly 工件：

   ```text
   node_template_runtime.compact.compressed.wasm
   node_template_runtime.compact.wasm
   node_template_runtime.wasm
   ```

## 执行运行时升级

您现在拥有一个描述修改后的运行时逻辑的 WebAssembly 工件。
但是，正在运行的节点尚未使用升级的运行时。
要完成升级，您需要提交一个事务，以更新节点以使用升级的运行时。

要使用升级的运行时更新网络：

1. 在浏览器中打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer) 并连接到本地节点。

1. 点击**开发者**并选择**外部函数**以提交事务，以便运行时使用新的构建工件。

1. 选择管理帐户**Alice**。

1. 选择**sudo** 模块和**sudoUncheckedWeight(call, weight)** 函数。

1. 选择**system** 和**setCode(code)** 作为要使用 Alice 帐户进行的调用。

1. 点击**文件上传**，然后选择或拖放您为更新的运行时生成的紧凑型压缩 WebAssembly 文件——`node_template_runtime.compact.compressed.wasm`。

   例如，导航到 `target/release/wbuild/node-template-runtime` 目录并将 `node_template_runtime.compact.compressed.wasm` 作为要上传的文件。

1. 将两个**权重**参数保留为默认值 `0`。

   ![运行时升级设置](/media/images/docs/tutorials/forkless-upgrade/set-code-transaction.png)

1. 点击**提交事务**。

1. 查看授权，然后点击**签名并提交**。

1. 点击**网络**并选择**资源管理器**以查看是否已成功发生 `sudo.Sudid` 事件。

   ![成功的 sudo 事件](/media/images/docs/tutorials/forkless-upgrade/set-code-sudo-event.png)   

   事务包含在区块中后，节点模板版本号表明运行时版本现在为 `101`。
   例如：

   ![更新后的运行时版本为 101](/media/images/docs/tutorials/forkless-upgrade/runtime-version-101.png)

   如果您的本地节点在终端中生成的区块与浏览器中显示的区块匹配，则您已完成成功的运行时升级。

1. 点击**开发者**并选择**外部函数**。点击*提交以下外部函数*并滚动到列表底部。您将看到**utility** 作为选项。


## 下一步

- [运行时版本 101](/assets/tutorials/runtime-upgrade/lib-spec-version-101.rs)
- [运行时版本 102](/assets/tutorials/runtime-upgrade/lib-spec-version-102.rs)
- [存储迁移](/maintain/runtime-upgrades/#storage-migration)
