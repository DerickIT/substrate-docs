---
title: 授权特定节点
description: 配置一个具有授权节点和访问受限节点的网络。
keywords:
  - 受许可的
  - 私有的
  - 授权
  - 访问受限
---

在[添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes/)中，您了解了如何构建一个具有已知验证器节点集的简单网络。
该教程演示了**受许可网络**的简化版本。
在受许可网络中，只有**授权节点**才能执行特定的网络活动。
例如，您可以授予某些节点验证区块的权限，而授予其他节点传播交易的权限。

具有被授予特定权限的节点的区块链不同于**公共**或**无许可**区块链。
在无许可区块链中，任何人都可以通过在合适的硬件上运行节点软件来加入网络。
通常，无许可区块链提供了更大的网络去中心化。
但是，在某些情况下，创建受许可区块链可能是合适的。
例如，受许可区块链适用于以下类型的项目：

- 对于私有或联盟网络，例如私营企业或非营利组织。
- 在高度监管的数据环境中，例如医疗保健、金融或企业对企业账本。
- 用于大规模测试预公开区块链网络。

本教程演示了如何使用[节点授权模块](https://paritytech.github.io/substrate/master/pallet_node_authorization/index.html)通过Substrate构建受许可网络。

## 节点授权和所有权

`node-authorization` 模块是一个预构建的FRAME模块，允许您管理网络的可配置节点集。
每个节点都由一个对等标识符（`PeerId`）标识。
每个对等标识符都由**唯一一个**声称该节点的帐户所有者（`AccountId`）拥有。

您可以通过两种方式授权节点加入网络：

- 通过将对等标识符添加到链规范文件中的预定义节点列表中，作为链的创世配置的一部分。
  您必须获得链的治理机制的批准，或者才能访问网络中Sudo模块的根帐户才能执行此操作。

- 通过向特定节点请求_配对对等_连接。
  您可以使用预定义的节点对等标识符或使用从每个节点的公钥和私钥生成的标识符来添加节点之间的连接。

如下图所示，本教程演示了这两种授权方法，其中Alice和Bob的对等标识符在链规范中定义，其他节点使用节点授权模块添加。

![使用对等标识符授权节点](/media/images/docs/tutorials/permissioned-network/four-nodes.png)

请注意，_任何_用户都可以声称自己是`PeerId`的所有者。
为了防止虚假声明，您应该在启动节点_之前_声明节点标识符。
启动节点后，其`PeerID`对网络可见，_任何_人都可以随后声明它。

作为节点的所有者，您可以为您的节点添加和删除连接。
例如，您可以操作预定义节点和您的节点之间或您的节点和其他非预定义节点之间的连接。
您无法更改预定义节点的连接。
它们始终允许彼此连接。

`node-authorization` 模块使用[链下工作程序](/learn/offchain-operations)来配置其节点连接。
启动节点时，请确保启用链下工作程序，因为它默认情况下对非权威节点是禁用的。

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装[Rust和Rust工具链](/install/)为Substrate开发配置了您的环境。

- 您已完成[构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)，并且已在本地安装了Substrate节点模板。

- 您已完成[添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes/)教程。

- 您通常熟悉Substrate中的[对等网络](https://wiki.polkadot.network/docs/faq#networking)。

## 教程目标

通过完成本教程，您将实现以下目标：

- 查看并编译节点模板。

- 将节点授权模块添加到节点模板运行时。

- 启动多个节点并授权新节点加入。

## 构建节点模板

如果您已完成之前的教程，则应该在本地拥有Substrate节点模板存储库。

1. 在您拥有Substrate节点模板存储库的计算机上打开终端shell。

1. 必要时，通过运行以下命令更改到节点模板目录的根目录：

   ```bash
   cd substrate-solo-template-node
   ```

2. 如果要通过运行类似于以下命令的命令来保存更改，请切换到存储库的工作分支：

   ```bash
   git switch -c my-wip-branch
   ```

3. 通过运行以下命令编译节点模板：

   ```bash
   cargo build --release
   ```

   节点模板应该可以编译而没有任何错误。
   如果您在编译时遇到问题，您可以尝试[解决Rust问题](/install/troubleshoot-rust-issues/)中的故障排除技巧。

## 添加节点授权模块

在您可以使用新的模块之前，您必须向编译器用来构建运行时二进制文件的配置文件中添加一些关于它的信息。

对于Rust程序，您可以使用`Cargo.toml`文件来定义配置设置和依赖项，这些设置和依赖项决定了最终二进制文件中包含的内容。
因为Substrate运行时编译为包含标准库函数的原生Rust二进制文件和不包含标准库的[WebAssembly (Wasm)](https://webassembly.org/)二进制文件，所以`Cargo.toml`文件控制两条重要的信息：

- 要作为运行时依赖项导入的模块，包括要导入的模块的位置和版本。

- 编译原生Rust二进制文件时应启用的每个模块中的功能。
  通过启用每个模块的标准（`std`）功能集，您可以编译运行时以包含在构建WebAssembly二进制文件时否则会缺少的函数、类型和基元。

有关在`Cargo.toml`文件中添加依赖项的常规信息，请参阅Cargo文档中的[依赖项](https://doc.rust-lang.org/cargo/guide/dependencies.html)。
有关启用和管理依赖包中的功能的信息，请参阅Cargo文档中的[功能](https://doc.rust-lang.org/cargo/reference/features.html)。

### 添加节点授权依赖项

要将`node-authorization`模块添加到Substrate运行时：

1. 打开终端shell并更改到节点模板的根目录。

1. 在文本编辑器中打开`runtime/Cargo.toml`配置文件。

1. 找到`[dependencies]`部分，并添加`pallet-node-authorization`crates以使其可用于节点模板运行时。

   ```toml
   [dependencies]
   pallet-node-authorization = { default-features = false, version = "4.0.0-dev", git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v1.0.0" }
   ```

   此行将`pallet-node-authorization`crates作为依赖项导入，并为该crates指定以下配置详细信息：

   - 默认情况下，编译运行时时不会启用模块功能。
   - crates的版本标识符。
   - 用于检索`pallet-node-authorization`crates的存储库位置。
   - 用于检索crates的分支。

   请注意，您应该为所有模块使用相同的分支和版本信息，以确保它们彼此兼容。
   使用来自不同分支的模块可能会导致编译器错误。
   此示例说明了如果其他模块使用`branch = "polkadot-v1.0.0"`，则将模块添加到`Cargo.toml`文件。

2. 将`pallet-node-authorization/std`功能添加到`features`列表中，以便在编译运行时时启用。

   ```toml
   [features]
   default = ['std']
   std = [
     ...
     "pallet-node-authorization/std",    # 添加此行
     ...
   ]
   ```

   此部分指定要为此运行时编译的默认功能集是`std`功能集。
   当使用`std`功能集编译运行时时，将启用作为依赖项列出的所有模块的`std`功能。
   有关如何使用标准库作为原生Rust二进制文件编译运行时以及如何使用`no_std`属性作为WebAssembly二进制文件编译运行时的更多详细信息，请参阅[构建过程](/build/build-process/)。

   如果您忘记更新`Cargo.toml`文件中的`features`部分，则在编译运行时二进制文件时可能会看到`cannot find function`错误。

3. 通过运行以下命令检查新依赖项是否正确解析：

   ```bash
   cargo check -p solo-template-node-runtime --release
   ```

### 添加管理规则

为了在本教程中模拟治理，您可以将模块配置为使用`EnsureRoot`特权函数，该函数可以使用Sudo模块调用。
Sudo模块默认包含在节点模板中，允许您通过根级管理帐户进行调用。
在生产环境中，您将使用更现实的基于治理的检查。

要在运行时启用`EnsureRoot`规则：

1. 在文本编辑器中打开`runtime/src/lib.rs`文件。

1. 将以下行添加到文件中：

   ```rust
   use frame_system::EnsureRoot;
   ```

## 实现Config特征

每个模块都有一个名为`Config`的[Rust特征](https://doc.rust-lang.org/book/ch10-02-traits.html)。
`Config`特征用于标识模块所需的参数和类型。

添加模块所需的大部分模块特定代码都是使用`Config`特征实现的。
您可以通过参考其Rust文档或模块的源代码来查看需要为任何模块实现的内容。
例如，要查看需要为`node-authorization`模块的`Config`特征实现的内容，您可以参考[`pallet_node_authorization::Config`](https://paritytech.github.io/substrate/master/pallet_node_authorization/pallet/trait.Config.html)的Rust文档。

要在运行时实现`node-authorization`模块：

1. 在文本编辑器中打开`runtime/src/lib.rs`文件。

1. 使用以下代码添加模块的`parameter_types`部分：

   ```rust
   parameter_types! {
     pub const MaxWellKnownNodes: u32 = 8;
     pub const MaxPeerIdLength: u32 = 128;
   }
   ```

1. 使用以下代码添加模块的`Config`特征的`impl`部分：

   ```rust
   impl pallet_node_authorization::Config for Runtime {
     type RuntimeEvent = RuntimeEvent;
     type MaxWellKnownNodes = MaxWellKnownNodes;
     type MaxPeerIdLength = MaxPeerIdLength;
     type AddOrigin = EnsureRoot<AccountId>;
     type RemoveOrigin = EnsureRoot<AccountId>;
     type SwapOrigin = EnsureRoot<AccountId>;
     type ResetOrigin = EnsureRoot<AccountId>;
     type WeightInfo = ();
   }
   ```

1. 使用以下代码行将模块添加到`construct_runtime`宏中：

   ```rust
   construct_runtime!(
   pub enum Runtime where
       Block = Block,
       NodeBlock = opaque::Block,
       UncheckedExtrinsic = UncheckedExtrinsic
     {
       /*** 添加此行 ***/
       NodeAuthorization: pallet_node_authorization::{Pallet, Call, Storage, Event<T>, Config<T>},
     }
   );
   ```

1. 保存更改并关闭文件。

1. 通过运行以下命令检查配置是否可以编译：

   ```bash
   cargo check -p solo-template-node-runtime --release
   ```

### 为授权节点添加创世存储

在您可以启动网络以使用节点授权之前，需要一些额外的配置来处理对等标识符和帐户标识符。
例如，`PeerId`以bs58格式编码，因此您需要在`node/Cargo.toml`中为[bs58](https://docs.rs/bs58/)库添加一个新的依赖项来解码`PeerId`以获取其字节。
为简单起见，授权节点与预定义帐户相关联。

要为授权节点配置创世存储：

1. 在文本编辑器中打开`node/Cargo.toml`文件。

1. 找到`[dependencies]`部分，并将`bs58`库添加到节点模板。

   ```toml
   [dependencies]
   bs58 = { version = "0.4.0" }
   ```

1. 保存更改并关闭文件。

1. 在文本编辑器中打开`node/src/chain_spec.rs`文件。

1. 使用以下代码添加用于授权加入网络的节点的创世存储：

   ```rust
   use sp_core::OpaquePeerId; // 一个结构体包装Vec<u8>来表示节点`PeerId`。
   use node_template_runtime::NodeAuthorizationConfig; // 为模块服务的创世配置。
   ```

1. 找到配置FRAME模块初始存储状态的`testnet_genesis`函数。

   例如：

   ```rust
   /// 配置FRAME模块的初始存储状态。
   fn testnet_genesis(
     wasm_binary: &[u8],
     initial_authorities: Vec<(AuraId, GrandpaId)>,
     root_key: AccountId,
     endowed_accounts: Vec<AccountId>,
     _enable_println: bool,
     ) -> GenesisConfig {

   ```

1. 在`GenesisConfig`声明中，添加以下代码块：

   ```rust
     node_authorization: NodeAuthorizationConfig {
       nodes: vec![
         (
           OpaquePeerId(bs58::decode("12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2").into_vec().unwrap()),
           endowed_accounts[0].clone()
         ),
         (
           OpaquePeerId(bs58::decode("12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust").into_vec().unwrap()),
           endowed_accounts[1].clone()
         ),
       ],
     },
   ```

   在此代码中，`NodeAuthorizationConfig`包含一个`nodes`属性，它是一个包含两个元素的元组的向量。
   元组的第一个元素是`OpaquePeerId`。
   `bs58::decode`操作将人类可读的`PeerId`（例如，`12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2`）转换为字节。
   元组的第二个元素是表示此节点所有者的`AccountId`。
   此示例使用预定义的[Alice和Bob](/reference/command-line-tools/subkey/#well-known-keys)，此处标识为赋予帐户[0]和[1]。

1. 保存更改并关闭文件。

### 验证节点是否编译

现在您已完成代码更改，您可以验证节点是否编译。

要编译节点：

1. 必要时更改到`substrate-solo-template-node`目录的根目录：

1. 通过运行以下命令编译节点：

   ```bash
   cargo build --release
   ```

   如果没有语法错误，则可以继续。
   如果有错误，请按照编译输出中的说明修复它们，然后重新运行`cargo build`命令。

## 确定要使用的帐户密钥

您已在创世存储中配置了与Alice和Bob帐户关联的节点。
您可以使用[`subkey`](/reference/command-line-tools/subkey/)程序检查与预定义帐户关联的密钥，以及生成和检查您自己的密钥。
但是，如果您运行`subkey generate-node-key`命令，您的节点密钥和对等标识符将被随机生成，并且与教程中使用的密钥不匹配。
因为本教程使用预定义帐户和众所周知的节点密钥，所以您可以为每个帐户使用以下密钥。

### Alice

| 密钥类型 | 密钥值 |
| :------- | :-------------------------------------------------------------|
| 节点密钥 | c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a |
| 由节点密钥生成的标识符 | 12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2 |
| 解码为十六进制的对等标识符 | 0x0024080112201ce5f00ef6e89374afb625f1ae4c1546d31234e87e3c3f51a62b91dd6bfa57df |

### Bob

| 密钥类型 | 密钥值 |
| :------- | :-------------------------------------------------------------|
| 节点密钥 | 6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58 |
| 由节点密钥生成的标识符 | 12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust |
| 解码为十六进制的对等标识符 | 0x002408011220dacde7714d8551f674b8bb4b54239383c76a2b286fa436e93b2b7eb226bf4de7 |

另外两个开发帐户——Charlie和Dave——在创世配置中没有定义众所周知的节点密钥或对等标识符。
出于演示目的，您可以为这些帐户使用以下密钥。

### Charlie

| 密钥类型 | 密钥值 |
| :------- | :-------------------------------------------------------------|
| 节点密钥 | 3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e |
| 由节点密钥生成的标识符 | 12D3KooWJvyP3VJYymTqG7eH4PM5rN4T2agk5cdNCfNymAqwqcvZ |
| 解码为十六进制的对等标识符 | 0x002408011220876a7b4984f98006dc8d666e28b60de307309835d775e7755cc770328cdacf2e |

### Dave

| 密钥类型 | 密钥值 |
| :------- | :-------------------------------------------------------------|
| 节点密钥 | a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a |
| 由节点密钥生成的标识符 | 12D3KooWPHWFrfaJzxPnqnAYAoRUyAHHKqACmEycGTVmeVhQYuZN |
| 解码为十六进制的对等标识符 | 0x002408011220c81bc1d7057a1511eb9496f056f6f53cdfe0e14c8bd5ffca47c70a8d76c1326d |

对于本教程，您可以将节点密钥复制到文件中，然后使用`subkey inspect-node-key`来验证Charlie和Dave的对等标识符。
例如，使用以下命令将Charlie的节点密钥保存到名为`charlie-node-key`的文件中：

```bash
echo -n "3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e" > charlie-node-key
```

然后，您可以运行以下命令来验证对等标识符：

```bash
./subkey inspect-node-key --file charlie-node-key
```

该命令显示Charlie节点的对等标识符：

```text
12D3KooWJvyP3VJYymTqG7eH4PM5rN4T2agk5cdNCfNymAqwqcvZ
```

如果您为自己的帐户生成节点密钥，请将节点的对等标识符保存到文件中，以便在需要时将其传递给`subkey inspect-node-key`或其他命令。

## 启动网络节点

您现在可以使用预定义帐户的节点密钥和对等标识符来启动受许可网络并授权其他节点加入。

在本教程中，您将启动四个节点。
三个节点与预定义帐户相关联，所有这三个节点都允许创建和验证区块。
第四个节点是一个**子节点**，它只被授权在选定节点所有者的批准下读取选定节点的数据。

### 启动第一个节点

因为您已将创世存储配置为使用Alice和Bob的众所周知的节点密钥，所以您可以使用`--alice`命令快捷方式（`--name alice --validator`）来启动第一个节点。

要启动第一个节点：

1. 必要时，在您的计算机上打开终端shell。

1. 更改到您编译Substrate节点模板的根目录。

1. 通过运行以下命令启动第一个节点：

   ```bash
   ./target/release/solo-template-node \
   --chain=local \
   --base-path /tmp/validator1 \
   --alice \
   --node-key=c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a \
   --port 30333 \
   --rpc-port 9944
   ```

   在此命令中，`--node-key`选项指定用于与网络安全连接的密钥。
   此密钥还用于内部生成人类可读的PeerId，如上节所示。

   正如您在其他教程中可能看到的那样，使用的命令行选项是：

   - `--chain=local` 用于本地测试网（与`--dev`标志不同！）。
   - `--alice` 将节点命名为`alice` 并使节点成为可以创建和完成区块的验证器。
   - `--port` 为对等通信分配端口。
   - `--rpc-port` 为WebSocket连接分配监听端口。

### 启动第二个节点

您可以使用`--bob`命令快捷方式（`--name bob --validator`）来启动第二个节点。

要启动第二个节点：

1. 在您的计算机上打开一个**新的**终端shell。

1. 更改到您编译Substrate节点模板的根目录。

1. 通过运行以下命令启动第二个节点：

   ```bash
   ./target/release/solo-template-node \
   --chain=local \
   --base-path /tmp/validator2 \
   --bob \
   --node-key=6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58 \
   --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2 \
   --port 30334 \
   --rpc-port 9945
   ```

   启动两个节点后，您应该开始在两个终端日志中看到创建和完成的新区块。

### 将第三个节点添加到众所周知的节点列表中

您可以使用`--name charlie`命令启动第三个节点。
`node-authorization`模块使用[链下工作程序](/learn/offchain-operations)来配置节点连接。
因为第三个节点不是众所周知的节点，并且它将把网络中的第四个节点配置为只读子节点，所以您必须包含命令行选项以启用链下工作程序。

要启动第三个节点：

1. 在您的计算机上打开一个**新的**终端shell。

1. 更改到您编译Substrate节点模板的根目录。

1. 通过运行以下命令启动第三个节点：

   ```bash
   ./target/release/solo-template-node \
   --chain=local \
   --base-path /tmp/validator3 \
   --name charlie  \
   --node-key=3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e \
   --port 30335 \
   --rpc-port=9946 \
   --offchain-worker always
   ```

   启动此节点后，您应该会看到该节点没有**连接的对等节点**。
   因为这是一个受许可的网络，所以必须明确授权节点才能连接。
   Alice和Bob节点已在创世`chain_spec.rs`文件中配置。
   所有其他节点都必须使用对Sudo模块的调用手动添加。

### 授权第三个节点访问

本教程使用Sudo模块进行治理。
因此，您可以使用Sudo模块调用`node-authorization`模块提供的`addWellKnownNode`函数来添加第三个节点。
为简单起见，您可以使用Polkadot/Substrate门户应用程序访问Sudo模块。

1. 在浏览器中打开[Polkadot/Substrate门户](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer)。

1. 点击**开发者**并选择**Sudo**。

1. 选择**nodeAuthorization**并选择**addWellKnownNode(node, owner)**。

1. 在必需的0x前缀后复制并粘贴由Charlie拥有的节点的十六进制编码对等标识符。

2. 选择**Charlie**作为节点所有者。

3. 点击**提交Sudo**。

   ![提交Sudo事务以添加第三个节点](/media/images/docs/tutorials/permissioned-network/sudo-add-well-known-node.png)

4. 在授权事务中，请注意Alice开发帐户是默认的根管理帐户，并用作此调用的`sudo`来源，然后点击**签名并提交**。

   ![验证并授权事务](/media/images/docs/tutorials/permissioned-network/sudo-to-add-node.png)

5. 点击**网络**并选择**资源管理器**以查看最近的事务。

   ![验证Sudo事件](/media/images/docs/tutorials/permissioned-network/sudo-events-add-node.png)

   事务包含在区块中后，您应该会看到`charlie`节点已连接到`alice`和`bob`节点，并开始同步区块。
   这三个节点可以使用默认情况下在本地网络中启用的[mDNS](https://paritytech.github.io/substrate/master/sc_network/index.html)发现机制找到彼此。还要确保任何本地防火墙都配置为允许mDNS。

   如果您的节点不在同一个本地网络上，则应使用命令行选项`--no-mdns`将其禁用。

### 允许来自子节点的连接

本网络中的第四个节点将不用作验证器，也不会添加到众所周知的节点列表中。
此第四个节点由`dave`用户帐户拥有，但它是`charlie`节点的子节点。
子节点只能通过连接到由`charlie`父节点拥有的节点来访问网络。
重要的是要记住，父节点负责其授权连接到网络的任何子节点。
父节点的所有者负责控制访问，如果需要删除或审核子节点。

因为这是一个_受许可的网络_，所以Charlie必须配置其节点以允许来自由Dave拥有的节点的连接。
您可以使用Polkadot/Substrate门户应用程序授予此权限。

要允许子节点访问网络：

1. 在浏览器中打开[Polkadot/Substrate门户](https://polkadot.js.org/apps/#/explorer)。

2. 点击**开发者**并选择**外部函数**。

3. 选择**nodeAuthorization**并选择**addConnections(node, connections)**。

4. 在必需的0x前缀后复制并粘贴由Charlie拥有的节点的十六进制编码对等标识符。

5. 对于连接参数，在必需的0x前缀后复制并粘贴由Dave拥有的节点的十六进制编码对等标识符，然后点击**提交事务**。

6. 查看事务详细信息，然后点击**签名并提交**。

   ![charlie_add_connections](/media/images/docs/tutorials/permissioned-network/charlie_add_connections.png)

### 声明子节点

在启动子节点之前，节点所有者应声明其节点的对等标识符。
您可以使用Polkadot/Substrate门户应用程序提交事务以声明由`dave`帐户拥有的节点。

1. 在浏览器中打开[Polkadot/Substrate门户](https://polkadot.js.org/apps/#/explorer)。

1. 点击**开发者**并选择**外部函数**。

2. 选择**nodeAuthorization**并选择**claimNode(node)**。

3. 在必需的0x前缀后复制并粘贴由Dave拥有的节点的十六进制编码对等标识符，然后点击**提交事务**。

4. 查看事务详细信息，然后点击**签名并提交**。

   ![dave_claim_node](/media/images/docs/tutorials/permissioned-network/dave_claim_node.png)

### 启动子节点

声明节点对等标识符后，您就可以启动子节点了。

要启动子节点：

1. 在您的计算机上打开一个**新的**终端shell。

1. 更改到您编译Substrate节点模板的根目录。

1. 通过运行以下命令启动子节点：

   ```bash
   ./target/release/solo-template-node \
   --chain=local \
   --base-path /tmp/validator4 \
   --name dave \
   --node-key=a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a \
   --port 30336 \
   --rpc-port 9947 \
   --offchain-worker always
   ```

### 允许连接到子节点

您现在拥有一个具有四个节点的网络。
但是，要允许子节点参与，您必须将其配置为允许来自由Charlie拥有的父节点的连接。
步骤与您之前执行的允许来自由Dave拥有的节点的连接的步骤类似。

1. 在浏览器中打开[Polkadot/Substrate门户](https://polkadot.js.org/apps/#/explorer)。

2. 点击**开发者**并选择**外部函数**。

3. 选择**nodeAuthorization**并选择**addConnections(node, connections)**。

4. 在必需的0x前缀后复制并粘贴由Dave拥有的节点的十六进制编码对等标识符。

5. 对于连接参数，在必需的0x前缀后复制并粘贴由Charlie拥有的节点的十六进制编码对等标识符，然后点击**提交事务**。

6. 查看事务详细信息，然后点击**签名并提交**。

   ![dave_add_connections](/media/images/docs/tutorials/permissioned-network/dave_add_connections.png)

   您现在应该会看到子节点只有一个对等节点——属于Charlie的节点——并且正在从链中同步区块。
   如果子节点没有立即连接到其对等节点，请尝试停止并重新启动子节点。

## 提交事务所需的密钥

您应该注意，任何帐户都可以用来签名和提交影响其他节点行为的事务。
但是，要签名和提交影响您不拥有的节点的事务：

- 事务必须引用_链上数据_。
- 您必须在密钥库中拥有具有所需来源的帐户的_签名密钥_。

在本教程中，所有节点都可以访问开发帐户签名密钥。
因此，您可以使用帐户代表Charlie或Dave来签名和提交影响任何连接节点的事务。
如果您正在为现实世界的应用程序构建受许可的网络，节点操作员很可能只能访问他们自己的节点密钥，并且需要节点所有者帐户来签名和提交影响他们控制签名密钥的节点的事务。

## 下一步

在本教程中，您学习了如何构建某些节点具有有限权限和对网络资源访问受限的网络的基础知识。
要了解有关本教程中介绍的主题的更多信息，请参阅以下资源：

- [帐户、地址和密钥](/learn/accounts-addresses-keys)
- [节点授权模块](https://paritytech.github.io/substrate/master/pallet_node_authorization/index.html#)
- [节点授权源代码](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/node-authorization/src/lib.rs)
- [监控节点指标](/tutorials/build-a-blockchain/monitor-node-metrics/)
