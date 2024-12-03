---
title: 连接到中继链
difficulty: 1
keywords:
  - collator
  - parachain
  - parathread
  - 升级
  - cumulus
  - 存储
  - 迁移
  - paraid
  - 注册
  - 启动
---

本指南总结了如何将 Parachain 连接到中继链。
它旨在作为快速参考，供您准备好将 Parachain 部署到测试或生产网络上，并且已经在本地测试环境中执行了类似步骤时使用。
如果您之前没有设置本地中继链并将 Parachain 连接到它，则应在使用本指南之前，按照 [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 和 [连接本地 Parachain](/tutorials/build-a-parachain/connect-a-local-parachain/) 中的步骤操作。

本指南重点介绍以下主题：

- 如何获取 Parachain 的唯一标识符
- 如何注册 Parachain
- 如何获取 Parachain 时隙

在您可以在中继链上获得时隙之前，您必须准备一些有关 Parachain 的信息，并将这些信息提供给您想要连接到的中继链。

要准备将 Parachain 连接到中继链，您必须成功完成以下步骤：

- 在您想要连接到的特定中继链上预留一个唯一标识符。
- 编译 WebAssembly 运行时二进制文件。
- 从 Parachain 的链规范生成创世状态。

完成这些步骤以注册特定中继链的 Parachain 后，您可以请求该中继链上的时隙以将您的 Parachain 添加到网络中。
出于测试目的，您可以使用 Sudo pallet 在中继链上获取时隙。
对于生产网络，您可以通过拍卖和众筹来获取时隙。
在您获得中继链上的时隙后，Parachain 就可以开始生成区块了。

## 预留唯一标识符

在执行任何涉及您的 Parachain 或 Parathread 的操作之前，您必须拥有特定中继链的唯一标识符。
例如，您必须拥有唯一标识符才能加载 WebAssembly blob、注册创世状态、创建到其他 Parachain 的消息通道或启动众筹。

要预留标识符：

1. 在 Web 浏览器中打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/) 并连接到正在运行的 Parachain 节点的 WebSocket 端口。

2. 点击 **网络** 并选择 **Parachains**。

3. 点击 **Parathreads**，然后点击 **+ParaID**。

   ![预留 `ParaID`](/media/images/docs/tutorials/parachains/paraid-reserve.png)

4. 提交交易以预留您想要连接到的中继链的 **ParaID**。

   该交易需要押金。
   您用于预留标识符的帐户将为此交易收费，并将成为与标识符关联的 Parathread 的原始帐户。

   请注意，该标识符只能用于您正在注册的中继链，并且您必须指定该标识符才能完成其他步骤。

## 自定义 Parachain 规范

您必须配置 Parachain 的链规范才能使用您已预留的标识符。
要执行此任务，您必须：

- 将链规范生成一个纯文本 JSON 文件。
- 在文本编辑器中修改链规范。
- 生成修改后的原始格式链规范。

要创建自定义链规范：

1. 通过运行类似于以下命令的命令来生成纯链规范：

   ```text
   ./target/release/parachain-template-node build-spec --disable-default-bootnode > rococo-local-parachain-plain.json
   ```

   此示例中的命令假设 `rococo-local` 是您已在 `node/chan_spec.rs` 文件中注册的中继链。

2. 在文本编辑器中打开纯文本链规范文件，并将 `ParaID` 设置为您之前预留的 `ParaID`。

   例如，打开 `rococo-local-parachain-plain.json` 文件并修改两个字段：

   ```text
  // --snip--
  "para_id": <您预留的标识符> ,
  // --snip--
      "parachainInfo": {
        "parachainId": <您预留的标识符> 
      },
  // --snip--
  ```

3. 通过运行类似于以下命令的命令，从修改后的链规范文件生成原始链规范：

   ```text
   ./target/release/parachain-template-node build-spec \
     --chain rococo-local-parachain-plain.json \
     --raw \
     --disable-default-bootnode > rococo-local-parachain-raw.json
     ```

## 保存和分发原始链规范

如果您打算让其他人连接到您的网络，则必须为您的网络构建和分发确定性运行时构建。
有关如何构建确定性运行时的信息，请参见 [构建确定性运行时](/build/build-a-deterministic-runtime/)。

按照惯例，链规范位于代码库中节点的 `/chain-specs` 文件夹中。
例如：

- Polkadot 将这些 **中继链** 链规范包含在 [node/service/chain-specs](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs) 下
- Cumulus 将这些 **Parachain** 链规范包含在 [chain-specs](https://github.com/paritytech/polkadot-sdk/tree/master/cumulus/polkadot-parachain/src/chain_spec) 下。

在继续之前，将原始链规范提交到您的源代码中是一个好习惯。

## 获取 WebAssembly 运行时验证函数

中继链还需要 Parachain 特定的运行时验证逻辑来验证 Parachain 区块。
您可以在 Parachain collator 节点上运行 `export-genesis-wasm` 命令来生成此 WebAssembly blob。
例如：

```bash
./target/release/parachain-template-node export-genesis-wasm --chain rococo-local-parachain-raw.json > para-wasm
```

## 生成 Parachain 创世状态

要注册 Parachain，中继链需要知道 Parachain 的 [创世状态](/build/chain-spec#the-genesis-state)。
您可以在 Parachain collator 节点上运行 `export-genesis-state` 命令来生成 Parachain 的十六进制编码创世状态。
例如：

```bash
./target/release/parachain-template-node export-genesis-state --chain parachain-raw.json > para-genesis
```

## 启动 collator

您现在可以使用类似于以下命令的命令启动 collator 节点：

```text
parachain-collator \
--alice \
--collator \
--force-authoring \
--chain parachain-raw.json \
--base-path /tmp/parachain/alice \
--port 40333 \
--rpc-port 8844 \
-- \
--execution wasm \
--chain <中继链规范json> \
--port 30343 \
--rpc-port 9977
```

在此命令中，在单独的 `--` 参数之前传递的参数用于 Parachain collator。
`--` 后的参数用于嵌入式中继链节点。
请注意，中继链规范应该是您已注册的中继链的链规范。

您应该看到 collator 正在运行并与中继链节点对等。
但是，Parachain 尚未创作 Parachain 区块。
在 collator 在中继链上注册后，创作开始。

## Parachain 注册

根据您的目标中继链和那里的权限，您可以选择注册。
通常，您使用 Sudo 事务进行测试，以及 Parachain [拍卖](https://wiki.polkadot.network/docs/learn-auction) 和 [众筹](https://wiki.polkadot.network/docs/learn-crowdloans) 用于生产。

本指南仅涵盖使用 Sudo 事务。

### 注册押金计算

或者，您可以在函数中计算 Polkadot 运行时的押金计算的精确公式：

```rust
pub const fn deposit(items: u32, bytes: u32) -> Balance {}
```

您可以在 Polkadot 代码库中找到每个中继链的此函数。
例如：

- [Kusama](https://github.com/polkadot-fellows/runtimes/blob/main/relay/kusama/constants/src/lib.rs)
- [Polkadot](https://github.com/polkadot-fellows/runtimes/blob/main/relay/polkadot/constants/src/lib.rs)
- [Rococo](https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/runtime/rococo/constants/src/lib.rs)
- [Westend](https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/runtime/westend/constants/src/lib.rs)

### 使用 sudo 注册

要使用 `sudo` 调用在中继链上注册 Parachain。

1. 在浏览器中打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer) 并连接到目标中继链。

1. 点击 **开发者** 并选择 **Sudo**。

5. 选择 **paraSudoWrapper**，然后选择 **sudoScheduleParaInitialize(id, genesis)** 以在下一个中继链会话开始时初始化预留的标识符。

   对于事务参数：

   - `id`：键入您预留的 `ParaId`。

   - `genesisHead`：点击 **文件上传** 并上传您为 Parachain 导出的创世状态。
     对于本教程，请选择 `para-2000-genesis` 文件。

   - `validationCode`：点击 **文件上传** 并上传您为 Parachain 导出的 WebAssembly 运行时
     对于本教程，请选择 `para-2000-wasm` 文件。

   - `paraKind`：选择 **是**。

   ![设置注册参数](/media/images/docs/tutorials/parachains/register-with-sudo.png)

1. 通过点击 **网络** 并选择 **资源管理器** 来验证事务是否成功，以便在最近事件列表中查看 `sudo.Sudid` _和_ `paras.PvfCheckAccepted` 事件。

### 使用时隙租赁注册

您还可以使用 Sudo 事务强制执行时隙租赁。

要强制执行时隙租赁：

1. 在浏览器中打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer) 并连接到目标中继链。

1. 点击 **开发者** 并选择 **Sudo**。

2. 选择 **时隙**，然后选择 **forceLease(para, leaser, amount, period_begin, period_end)**。

   ![强制执行时隙租赁的 Sudo 事务](/media/images/docs/tutorials/parachains/forceLease.png)

   对于事务参数：

   - `period_begin`：指定您想要开始的时隙。例如，测试环境中的活动时隙 `0`。

   - `period_end`：指定一个超出您为测试 Parachain 预留时间的时隙。
     如果您想测试加入和退出，则应选择有间隙的时隙租赁。

   在 Parachain 加入并开始区块生产后，您应该会看到 Parachain 的类似以下信息：

   ![查看有关 Parachain 的信息](/media/images/docs/tutorials/parachains/parachain-active-lease.png)

## 区块生产和最终确定

注册成功且新的中继链纪元开始后，collator 应该开始生成 Parachain 区块。

> 新纪元开始可能需要一段时间。

您可以通过点击 **网络** 并选择 **Parachains** 来跟踪在 Polkadot/substrate 门户中注册的 Parachain 及其最新的头部数据。

## 示例

- [准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain)
- [连接本地 Parachain](/tutorials/build-a-parachain/connect-a-local-parachain/)
- [获取测试网时隙](/tutorials/build-a-parachain/acquire-a-testnet-slot/)
