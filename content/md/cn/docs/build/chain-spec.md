---
title: 链规范
description: 描述链规范在基于 Substrate 的网络中的作用、启动节点时如何指定要使用的链规范以及如何自定义和分发链规范。
keywords:
  - 创世
  - 配置
  - 链规范
---

在 Substrate 中，**链规范**是描述基于 Substrate 的区块链网络的信息集合。
例如，链规范标识区块链节点连接到的网络、它最初与之通信的其他节点以及节点必须同意才能生成块的初始状态。

链规范是使用 [`ChainSpec` 结构](https://paritytech.github.io/substrate/master/sc_service/struct.GenericChainSpec.html) 定义的。
`ChainSpec` 结构将链所需的信息分为两部分：

- 客户端规范，其中包含 Substrate **外部节点**用于与网络参与者通信并将数据发送到遥测端点的信息。
  启动节点时，许多这些链规范设置都可以被命令行选项覆盖，或者可以在区块链启动后更改。

- 所有网络中的节点都同意的初始**创世状态**。
  创世状态必须在区块链首次启动时建立，此后不能更改，除非启动一个全新的区块链。

## 自定义外部节点设置

对于外部节点，链规范控制诸如以下信息：

- 节点与之通信的引导节点。

- 节点向其发送遥测数据的服务器端点。

- 节点连接到的网络的人类可读和机器可读名称。

由于 Substrate 框架是可扩展的，你还可以自定义链规范以包含其他信息。
例如，你可以配置外部节点以连接到特定高度的特定块，以防止在从创世同步新节点时发生远程攻击。

请注意，你可以在创世之后自定义外部节点设置。
但是，节点只会添加使用相同 `protocolId` 的对等节点。

## 自定义创世配置

所有网络中的节点都必须在同意任何后续块之前就创世状态达成一致。
链规范的创世部分中配置的信息用于创建创世块。
它在你启动第一个节点时生效，并且不能用命令行选项覆盖。
但是，你可以在链规范的创世部分配置一些信息。
例如，你可以自定义链规范的创世部分以包含诸如以下信息：

- 初始代币持有者余额。

- 最初属于治理委员会的帐户。

- 控制 `sudo` 密钥的管理帐户。

Substrate 节点还在链上包含运行时逻辑的已编译 WebAssembly，因此初始运行时也必须在链规范中提供。

## 存储链规范信息

链规范中的信息可以存储为 Rust 代码或 JSON 文件。
Substrate 节点通常包含至少一个，通常是许多硬编码的链规范。
将此信息作为 Rust 代码直接包含在节点中可确保节点无需节点操作员提供的任何其他信息即可连接到至少一个链。
如果你正在构建一个旨在定义主网络的区块链，则此主网络规范通常在外部节点中硬编码。

或者，你可以使用 `build-spec` 子命令将链规范序列化为 JSON 文件。
在启动测试网络或私有链时，通常会将 JSON 编码的链规范与节点二进制文件一起分发。

## 提供链规范以启动节点

每次启动节点时，你都会提供节点应使用的链规范。
在最简单的情况下，节点使用硬编码到节点二进制文件中的默认链规范。
启动节点时，你可以使用 `--chain` 命令行选项选择替代的硬编码链规范。
例如，你可以通过将 `--chain local` 指定为命令行选项来指示节点使用与字符串“local”关联的链规范。

如果你不想使用硬编码的链规范启动节点，则可以将其作为 JSON 文件提供。
例如，你可以通过将 `--chain=someCustomSpec.json` 指定为命令行选项来指示节点使用 `someCustomSpec.json` 文件中的链规范。
如果你指定了一个 JSON 文件，则节点会尝试反序列化提供的 JSON 链规范，然后使用它。

## 为运行时声明存储项

在大多数情况下，Substrate 运行时需要在创世时配置一些存储项。
例如，如果你正在使用 FRAME 开发运行时，则在运行时中使用 `Config` 特性声明的任何存储项都需要在创世时进行配置。
这些存储值在链规范的创世部分中配置。
有关如何在模块中设置存储项初始值的信息，请参阅 [创世配置](/build/genesis-configuration/)。

### 创建自定义链规范

如果你正在创建用于开发、测试或演示目的的一次性网络，你可能需要一个完全自定义的链规范。
要创建一个完全自定义的链规范，你可以将默认链规范导出为 JSON 格式，然后编辑 JSON 文件中的字段。
例如，你可以使用 `build-spec` 子命令将链规范导出到 JSON 文件：

```bash
substrate build-spec > myCustomSpec.json
```

导出链规范后，你可以在文本编辑器中修改其任何字段。
例如，你可能想要更改网络名称、引导节点和任何创世存储项，例如代币余额。
编辑 JSON 文件后，你可以使用自定义的 JSON 启动节点。
例如：

```bash
substrate --chain=myCustomSpec.json
```

## 原始链规范

Substrate 节点支持运行时升级。
通过运行时升级，区块链的运行时可以与链启动时不同。
链规范包含以节点运行时可以理解的方式结构化的信息。
例如，考虑 Substrate 节点模板的默认链规范中的此摘录：

```json
"sudo": {
  "key": "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY"
}
```

在 JSON 文件中，此密钥及其关联的值是人类可读的文本。
但是，此信息不能以这种格式存储在 Substrate 使用的基础存储结构中。
在你可以使用链规范初始化节点的创世存储之前，必须将人类可读的密钥转换为实际的存储密钥，以便可以将值存储在 [存储 trie](/learn/state-transitions-and-storage/) 中。
此转换很简单，但它需要链规范以节点运行时可以读取的格式进行编码。

为了使具有升级运行时的节点能够从创世同步链，人类可读的链规范以**原始**格式编码。
原始格式使你能够分发所有节点都可以用来同步链的链规范，即使在运行时升级之后也是如此。

基于 Substrate 的节点支持 `--raw` 命令行选项来生成原始链规范。
例如，你可以通过运行以下命令来为人类可读的 `myCustomSpec.json` 文件生成原始链规范：

```bash
substrate build-spec --chain=myCustomSpec.json --raw > customSpecRaw.json
```

转换为原始格式后，`sudo key` 代码段如下所示：

```json
"0x50a63a871aced22e88ee6466fe5aa5d9": "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d",
```

## 接下来的步骤

- [添加受信任的节点](/tutorials/build-a-blockchain/add-trusted-nodes/)
- [操作方法：配置创世状态](/reference/how-to-guides/basics/configure-genesis-state/)
- [操作方法：自定义链规范](/reference/how-to-guides/basics/customize-a-chain-specification/)
- [节点模板链规范](https://github.com/paritytech/polkadot-sdk-solochain-template/blob/master/node/src/chain_spec.rs)
- [ChainSpec 结构](https://paritytech.github.io/substrate/master/sc_service/struct.GenericChainSpec.html)
- [ProtocolId 结构](https://paritytech.github.io/substrate/master/sc_network/config/struct.ProtocolId.html)
