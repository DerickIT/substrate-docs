---
title: 设置 Parachain 测试网络
description: 解释如何设置本地测试网络以模拟具有验证者和 Parachain collator 节点的中继链。
keywords:
  - parachain
  - Polkadot
  - 测试网
  - collator 节点
  - 验证者节点
  - 中继链
  - zombienet
---

您可以使用 `zombienet` 命令行工具来设置本地测试网络，以模拟具有验证者和 Parachain collator 节点的中继链。
您可以将测试网络配置为包含多个验证者和具有多个 collator 的 Parachain。

要查看 Zombienet 提供的功能的完整列表，请参阅 [README](https://github.com/paritytech/zombienet)。

本教程说明了如何设置具有以下配置的测试网络：

- 四个验证者
- 两个 Parachain
- 每个 Parachain 一个 collator
- 一个允许 Parachain 交换消息的 HRMP 通道

## 使用二进制文件准备工作文件夹

`zombienet` 命令行界面依赖于配置文件来指定测试网络的特性，包括要使用的二进制文件、Docker 镜像或 kubernetes 部署的名称和位置。

本教程说明了如何配置使用中继链和 collator 二进制文件的测试网络，因此设置测试网络的第一步是使用您需要的二进制文件准备一个工作文件夹。

要使用测试网络的二进制文件准备工作文件夹：

1. 根据需要，在您的计算机上打开一个新的终端 shell。

1. 更改到您的主目录，并创建一个新文件夹来保存生成测试网络所需的二进制文件。

   例如：

   ```bash
   mkdir bin
   ```

   如果您在 Linux 上设置测试网络，则可以将 Polkadot 二进制文件从 [Releases](https://github.com/paritytech/polkadot/releases) 下载到您的工作文件夹中。
   如果您在 macOS 上设置测试网络或想要自己编译二进制文件，请继续执行下一步。

1. 通过运行类似于以下命令的命令来克隆 Polkadot 仓库：

   ```bash
   git clone --depth 1 --branch release-v1.0.0 https://github.com/paritytech/polkadot-sdk.git
   ```

   发行分支使用命名约定 `release-v<n.n.n>`。
   例如，本教程中使用的发行分支是 `release-v1.0.0`。
   您可以检出更新的发行分支，而不是使用 `release-v1.0.0`。
   您可以在 [Releases](https://github.com/paritytech/polkadot-sdk/releases) 选项卡上找到有关最近发行版以及每个发行版中包含内容的信息。

1. 通过运行以下命令更改到 `polkadot` 目录的根目录：

   ```bash
   cd polkadot
   ```

1. 通过运行以下命令编译中继链节点：

   ```bash
   cargo build --release
   ```

   编译节点可能需要 15 到 60 分钟才能完成。

1. 通过运行类似于以下命令的命令，将 Polkadot 二进制文件和工作者复制到您的工作 `bin` 文件夹中：

   ```bash
   cp ./target/release/polkadot ../bin/polkadot-v1.0.0
   cp ./target/release/polkadot-*-worker ../bin/
   ```

   如本示例所示，通常最好将 `polkadot` 的版本附加到二进制文件名，以保持 `bin` 文件夹中的文件井然有序。

1. 更改到您的主目录。

### 添加第一个 Parachain 二进制文件

您的工作文件夹现在具有中继链的二进制文件，但您还需要 Parachain collator 节点的二进制文件。
您可以通过克隆 `substrate-parachain-template` 仓库将 Parachain collator 二进制文件添加到您的工作文件夹中。
默认情况下，编译 `substrate-parachain-template` 会创建一个使用 `paraId` 1000 配置的 Parachain collator 二进制文件。
您可以将此 `paraId` 用于测试网络中的第一个 Parachain。

要将第一个 Parachain 二进制文件添加到工作文件夹：

1. 通过运行以下命令克隆 `substrate-parachain-template` 仓库：

   ```bash
   git clone https://github.com/substrate-developer-hub/substrate-parachain-template
   ```

1. 通过运行以下命令更改到 Parachain 模板目录的根目录：

   ```bash
   cd substrate-parachain-template
   ```

1. 检出与您用于配置中继链的发行分支匹配的发行分支。

   例如：

   ```bash
   git checkout polkadot-v1.0.0
   ```

1. 通过运行以下命令编译 Parachain 模板 collator：

   ```bash
   cargo build --release
   ```

   您现在拥有 paraId 1000 的 Parachain collator 二进制文件。

1. 通过运行类似于以下命令的命令，将 Parachain 二进制文件复制到您的工作 `bin` 文件夹中：

   ```bash
   cp ./target/release/parachain-template-node ../bin/parachain-template-node-v1.0.0-1000
   ```

   如本示例所示，通常最好将版本和 `paraId` 附加到二进制文件名，以保持 `bin` 文件夹中的文件井然有序。

### 添加第二个 Parachain 二进制文件

每个 Parachain 都必须具有唯一的标识符，因此要为测试网络创建第二个 Parachain，您需要编辑默认链规范以指定不同的 `paraId`。

要将第二个 Parachain 二进制文件添加到工作文件夹：

1. 在文本编辑器中打开 `substrate-parachain-template/node/src/chain_spec.rs` 文件。
2. 修改 Local Testnet 配置以使用标识符 `1001` 而不是默认标识符 `1000`。

   在 `local_testnet_config` 函数中需要更改两个实例：

   ```rust
   pub fn local_testnet_config() -> ChainSpec {
      // 为您的基本货币提供单位名称和小数位数
      let mut properties = sc_chain_spec::Properties::new();
      properties.insert("tokenSymbol".into(), "UNIT".into());
      properties.insert("tokenDecimals".into(), 12.into());
      properties.insert("ss58Format".into(), 42.into());

      ChainSpec::from_genesis(
        // 名称
        "Local Testnet",
        // ID
        "local_testnet",
        ChainType::Local,
        move || {
            testnet_genesis(
              // 初始 collator。
              vec![
                (
                  get_account_id_from_seed::<sr25519::Public>("Alice"),get_collator_keys_from_seed("Alice"),
                ),
                (
                  get_account_id_from_seed::<sr25519::Public>("Bob"),
                  get_collator_keys_from_seed("Bob"),
                ),
              ],
              vec![
                  get_account_id_from_seed::<sr25519::Public>("Alice"),get_account_id_from_seed::<sr25519::Public>("Bob"),get_account_id_from_seed::<sr25519::Public>("Charlie"),get_account_id_from_seed::<sr25519::Public>("Dave"),get_account_id_from_seed::<sr25519::Public>("Eve"),get_account_id_from_seed::<sr25519::Public>("Ferdie"),get_account_id_from_seed::<sr25519::Public>("Alice//stash"),get_account_id_from_seed::<sr25519::Public>("Bob//stash"),get_account_id_from_seed::<sr25519::Public>("Charlie//stash"),get_account_id_from_seed::<sr25519::Public>("Dave//stash"),get_account_id_from_seed::<sr25519::Public>("Eve//stash"),get_account_id_from_seed::<sr25519::Public>("Ferdie//stash"),
              ],
            1001.into(),
          )
        },
        // Bootnodes
        Vec::new(),
        // 遥测
        None,
        // 协议 ID
        Some("template-local"),
        // 分叉 ID
        None,
        // 属性
        Some(properties),
        // 扩展
        Extensions {
            relay_chain: "rococo-local".into(), // 您_必须_将其设置为正确的网络！
            para_id: 1001,
      },
    )
   ```

3. 保存您的更改并关闭链规范文件。

4. 通过运行以下命令使用修改后的链规范编译节点：

   ```bash
   cargo build -release
   ```

5. 通过运行类似于以下命令的命令，将新的 Parachain collator 二进制文件复制到工作 `bin` 文件夹中：

   ```bash:
   cp ./target/release/parachain-template-node ../bin/parachain-template-node-v1.0.0-1001
   ```

## 配置测试网络设置

现在您已经在工作文件夹中拥有了所有需要的二进制文件，您可以配置 Zombienet 将使用的测试网络设置。

要下载和配置 Zombienet：

1. 下载适用于 Linux 或 macOS 操作系统的适当 [Zombienet 可执行文件](https://github.com/paritytech/zombienet/releases)。

   根据您的安全设置，您可能需要明确允许访问可执行文件。

   如果您希望可执行文件在系统范围内可用，请在下载可执行文件后运行类似于以下命令的命令：

   ```bash
   chmod +x zombienet-macos
   cp zombienet-macos /usr/local/bin
   ```

2. 通过运行以下命令验证 Zombienet 是否已正确安装：

   ```bash
   ./zombienet-macos --help
   ```

   如果显示命令行帮助，则 Zombienet 已准备好进行配置。

3. 通过运行以下命令为 Zombienet 创建一个配置文件：

   ```bash
   touch config.toml
   ```

   您将使用配置文件来指定以下信息：

   - 测试网络二进制文件的位置。
   - 要使用的中继链规范——`rococo-local`。
   - 有关四个中继链验证者的信息。
   - 测试网络中包含的 Parachain 的标识符。
   - 有关每个 Parachain 的 collator 的信息。

   例如：

   ```toml
   [relaychain]
   default_command = "./bin/polkadot-v1.0.0"
   default_args = [ "-lparachain=debug" ]

   chain = "rococo-local"

      [[relaychain.nodes]]
      name = "alice"
      validator = true

      [[relaychain.nodes]]
      name = "bob"
      validator = true

      [[relaychain.nodes]]
      name = "charlie"
      validator = true

      [[relaychain.nodes]]
      name = "dave"
      validator = true

   [[parachains]]
   id = 1000
   cumulus_based = true

      [parachains.collator]
      name = "parachain-A-1000-collator-01"
      command = "./bin/parachain-template-node-v1.0.0-1000"

   [[parachains]]
   id = 1001
   cumulus_based = true

      [parachains.collator]
      name = "parachain-B-1001-collator-01"
      command = "../bin/parachain-template-node-v1.0.0-1001"
   ```

4. 保存您的更改并关闭文件。
5. 通过运行类似于以下命令的命令来使用此配置文件启动测试网络：

   ```bash
   ./zombienet-macos spawn config.toml -p native
   ```

   该命令会显示有关正在启动的测试网络节点的信息。
   所有节点都运行后，您可以通过打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps) 并连接到任何节点端点来与您的节点进行交互。

## 打开消息传递通道

现在您的测试网络已启动并运行，您可以打开水平中继消息传递通道，以启用 Parachain A (1000) 和 Parachain B (1001) 之间的通信。
因为通道是单向的，所以您需要：

- 发送请求以从 Parachain A (1000) 到 Parachain B (1001) 打开通道。
- 在 Parachain B (1001) 上接受请求。
- 发送请求以从 Parachain B (1001) 到 Parachain A (1000) 打开通道。
- 在 Parachain A (1000) 上接受请求。

Zombienet 通过使您可以将基本通道设置包含在配置文件中（用于测试目的）来简化打开这些通道的过程。

要设置测试网络中 Parachain 之间的通信：

1. 在文本编辑器中打开 `config.toml` 文件。

2. 将类似于以下内容的通道信息添加到配置文件中：

   ```toml
   [[hrmpChannels]]
   sender = 1000
   recipient = 1001
   max_capacity = 8
   max_message_size = 8000

   [[hrmpChannels]]
   sender = 1001
   recipient = 1000
   max_capacity = 8
   max_message_size = 8000
   ```

   请注意，您为 **maxCapacity** 和 **maxMessageSize** 设置的值不应超过为中继链定义的 `hrmpChannelMaxCapacity` 和 `hrmpChannelMaxMessageSize` 参数的值。

   要使用 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/) 检查当前中继链的配置设置：

   - 点击**开发者**，然后选择**链状态**。
   - 选择**配置**，然后选择**activeConfig()**。
   - 检查以下参数值：

     ```text
     hrmpChannelMaxCapacity: 8
     hrmpChannelMaxTotalSize: 8,192
     hrmlChannelMaxMessageSize: 1,048,576
     ```

3. 保存您的更改并关闭文件。
4. 通过运行以下命令重新启动 Zombienet：

   ```bash
   ./zombienet-macos spawn config.toml -p native
   ```

   您现在拥有一个测试网络，其中 Parachain A (1000) 和 Parachain B (1001) 之间打开了一个双向 HRMP 通道。

   您可以使用 [Polkadot/Substrate 门户](https://polkadot.js.org/apps) 连接到 Parachain 并发送消息。

5. 点击**开发者**，然后选择**外部函数**。

6. 选择 **polkadotXcm**，然后选择 **sent(dest, message)** 来创建要发送的 XCM 消息。

   您应该注意，XCM 消息与其他交易类似，需要发送者支付操作的执行费用。
   所有必需的信息都必须包含在消息本身中。
   有关在打开 HRMP 通道后如何使用 XCM 创建消息的信息，请参阅 [创建 XCM 消息](/learn/xcm-communication)。
