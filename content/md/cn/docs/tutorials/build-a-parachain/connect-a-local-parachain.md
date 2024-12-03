---
title: 连接本地平行链
description: 演示如何通过连接到本地中继链来部署平行链。
keywords:
  - 平行链
  - 测试网
  - 本地
  - 验证人
  - 链规范
---

本教程说明了如何使用本地中继链预留平行链标识符，以及如何将本地平行链连接到该中继链。

## 教程目标

通过完成本教程，您将实现以下目标：

- 编译本地平行链节点。
- 为平行链使用预留本地中继链上的唯一标识符。
- 为平行链配置链规范。
- 导出平行链的运行时和创世状态。
- 启动本地平行链并查看它是否连接到本地中继链。

## 开始之前

在开始之前，请验证以下内容：

- 您已按照[准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/)中的说明配置了具有两个验证器的本地中继链。

- 您知道平行链版本和依赖项与它们连接到的中继链版本紧密耦合，并且知道您用于配置中继链的软件版本。

  教程通常使用最新的 Polkadot 分支来演示功能。
  如果教程没有按预期工作，您应该检查您的本地环境中是否有最新的 Polkadot 分支，并在需要时更新您的本地软件。

## 构建平行链模板

本教程使用[Substrate 平行链模板](https://github.com/substrate-developer-hub/substrate-parachain-template)来说明如何启动连接到本地中继链的平行链。
平行链模板类似于在单链开发中使用的[节点模板](https://github.com/paritytech/polkadot-sdk-solochain-template)。
您还可以使用平行链模板作为开发自定义平行链项目的起点。

要构建平行链模板：

1. 根据需要，在您的计算机上打开一个新的终端 shell。

2. 克隆 `substrate-parachain-template` 存储库的分支，该分支与您用于配置中继链的发行分支匹配。

   例如，如果您使用 `release-v1.0.0`Polkadot 发行分支来配置本地中继链，请对平行链模板使用 `polkadot-v1.0.0` 分支。

   ```bash
   git clone --depth 1 --branch polkadot-v1.0.0 https://github.com/substrate-developer-hub/substrate-parachain-template.git
   ```

3. 通过运行以下命令更改到平行链模板目录的根目录：

   ```bash
   cd substrate-parachain-template
   ```

   您现在拥有一个分离的分支。
   如果您想保存更改并使此分支易于识别，则可以通过运行类似于以下命令的命令来创建一个新分支：

   ```bash
   git switch -c my-branch-v1.0.0
   ```

4. 通过运行以下命令构建平行链模板验证人：

   ```bash
   cargo build --release
   ```

   编译节点可能需要长达 60 分钟的时间，具体取决于您的硬件和软件配置。

## 预留唯一标识符

每个平行链都必须预留一个唯一标识符（`ParaID`），这使它能够连接到其特定的中继链。
每个中继链都管理其自身连接到它的平行链的唯一标识符集。
该标识符被称为 `ParaID`，因为相同的标识符可用于标识由[平行链](https://wiki.polkadot.network/docs/learn-parachains)占据的插槽，也可用于标识由[平行线程](https://wiki.polkadot.network/docs/learn-parathreads)占据的插槽。

您应该注意，您必须拥有足够的资金才能在中继链上预留插槽。
您可以通过检查该中继链的 `paras_registrar` 模块中的 `ParaDeposit` 配置来确定特定中继链需要多少代币。
例如，Rococo 需要 5 个 ROC 来预留标识符：

```rust
parameter_types! {
	pub const ParaDeposit: Balance = 5 * DOLLARS;
	pub const DataDepositPerByte: Balance = deposit(0, 1);
}

impl paras_registrar::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
	type RuntimeOrigin = RuntimeOrigin;
	type Currency = Balances;
	type OnSwap = (Crowdloan, Slots);
	type ParaDeposit = ParaDeposit;
	type DataDepositPerByte = DataDepositPerByte;
	type WeightInfo = weights::runtime_common_paras_registrar::WeightInfo<Runtime>;
}
```

每个中继链都通过从 `2000` 开始递增标识符来分配其自身的标识符，适用于所有不是[公共利益平行链](https://wiki.polkadot.network/docs/learn-common-goods)的链。
公共利益链使用不同的方法来分配插槽标识符。

要预留平行链标识符：

1. 验证您的本地中继链验证人正在运行。
2. 在浏览器中打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/parachains/parathreads)。
3. 连接到本地中继链节点。
4. 点击**网络**并选择**平行链**。

   ![导航到平行链](/media/images/docs/tutorials/parachains/network-parachains.png)

5. 点击**平行线程**，然后点击**ParaId**。

   ![预留标识符](/media/images/docs/tutorials/parachains/paraid-reserve.png)

6. 查看预留标识符的事务设置，然后点击**提交**。

   用于预留标识符的帐户将是为此事务收费的帐户，并且将是与标识符关联的平行线程的来源帐户。

7. 点击**签名并提交**以授权交易。

   提交交易后，点击**网络**并选择**资源管理器**。

8. 检查最近事件列表中是否有成功的 `registrar.Reserved`，然后点击该事件以查看有关交易的详细信息。

您现在可以准备链规范并生成所需的文件，以便您的平行链使用预留的标识符（`paraId` `2000`）连接到中继链。

## 修改默认链规范

要将您的平行链注册到本地中继链，您必须修改默认链规范以使用您已预留的平行链标识符。

1. 通过运行以下命令生成平行链模板节点的纯文本链规范：

   ```bash
   ./target/release/parachain-template-node build-spec --disable-default-bootnode > plain-parachain-chainspec.json
   ```

1. 在文本编辑器中打开平行链模板节点的纯文本链规范。

1. 将 `para_id` 设置为您先前预留的平行链标识符。

   例如，如果您的预留标识符为 `2000`，请将 `para_id` 字段设置为 `2000`：

   ```json
   ...
   "relay_chain": "rococo-local",
   "para_id": 2000,
   "codeSubstitutes": {},
   "genesis": {
      ...
   }
   ```

1. 将 `parachainId` 设置为您先前预留的平行链标识符。

   例如，如果您的预留标识符为 `2000`，请将 `para_id` 字段设置为 `2000`：

   ```json
   ...
      "parachainSystem": null,
      "parachainInfo": {
        "parachainId": 2000
      },
   ...
   ```

1. 如果您同时在同一本地网络上的任何人一起完成本教程，则需要一个额外的步骤来防止意外地与他们的节点对等。找到以下行并添加字符以使您的 protocolId 唯一：

```json
   "protocolId": "template-local"
```

5. 保存更改并关闭纯文本链规范文件。
6. 通过运行以下命令从修改后的链规范文件生成原始链规范文件：

   ```bash
   ./target/release/parachain-template-node build-spec --chain plain-parachain-chainspec.json --disable-default-bootnode --raw > raw-parachain-chainspec.json
   ```

   该命令将生成一个新的原始链规范文件，其中包含两个验证人。

   ```text
   2022-08-30 13:00:50 Building chain spec
   2022-08-30 13:00:50 assembling new collators for new session 0 at #0
   2022-08-30 13:00:50 assembling new collators for new session 1 at #0
   ```

## 准备平行链验证人

本地中继链运行且平行链模板的原始链规范已更新后，您可以准备启动平行链验证人节点并导出有关其运行时和创世状态的信息。

要准备注册平行链验证人：

1. 导出平行链的 WebAssembly 运行时。

   中继链需要特定于平行链的运行时验证逻辑来验证平行链区块。
   您可以通过运行类似于以下命令的命令来导出平行链验证人节点的 WebAssembly 运行时：

   ```bash
   ./target/release/parachain-template-node export-genesis-wasm --chain raw-parachain-chainspec.json para-2000-wasm
   ```

2. 生成平行链创世状态。

   要注册平行链，中继链需要知道平行链的创世状态。
   您可以通过运行类似于以下命令的命令将整个创世状态（十六进制编码）导出到文件：

   ```bash
   ./target/release/parachain-template-node export-genesis-state --chain raw-parachain-chainspec.json para-2000-genesis-state
   ```

   您应该注意，您导出的运行时和状态必须是_创世_块的。
   您不能将具有任何先前状态的平行链连接到中继链。
   所有平行链都必须从中继链上的区块 0 开始。
   有关平行链模板的创建方式以及如何将链逻辑（而不是其历史记录或状态迁移）转换为平行链的详细信息，请参阅[转换单链](/reference/how-to-guides/parachains/convert-a-solo-chain/)。

3. 使用类似于以下命令的命令启动验证人节点：

   ```bash
   ./target/release/parachain-template-node \
   --alice \
   --collator \
   --force-authoring \
   --chain raw-parachain-chainspec.json \
   --base-path /tmp/parachain/alice \
   --port 40333 \
   --rpc-port 8844 \
   -- \
   --execution wasm \
   --chain ../polkadot/raw-local-chainspec.json \
   --port 30343 \
   --rpc-port 9977
   ```

   在此命令中，在单独的 `--` 参数之前的参数用于平行链模板验证人。
   `--` 后面的参数用于嵌入式中继链节点。
   请注意，此命令指定了平行链的原始链规范和中继链的原始链规范。
   在此示例中，本地中继链的原始链规范是位于 `polkadot` 目录中的 `raw-local-chainspec.json`。
   确保第二个 `--chain` 命令行指定本地中继链的原始链规范的路径。

   如果您启动平行链的另一个节点，则将使用相同的链规范文件，但基路径和端口号不同。

   在您启动平行链模板节点的终端中，您应该看到类似于以下内容的输出：

   ```text
   Parachain Collator Template
   ✌️  version 0.1.0-336530d3bdd
   ❤️  by Anonymous, 2020-2023
   📋 Chain specification: Local Testnet
   🏷  Node name: Alice
   👤 Role: AUTHORITY
   💾 Database: RocksDb at /tmp/parachain/alice/chains/local_testnet/db/full
   no effect anymore and will be removed in the future!
   Parachain Account: 5Ec4AhPUwPeyTFyuhGuBbD224mY85LKLMSqSSo33JYWCazU4
   Is collating: yes
   [Relaychain] 🏷  Local node identity is: 12D3KooWR8wJbGWrjzKTpuXQvbuM1rE2GAE9JVEFEwAyNX6LV9nN
   [Relaychain] 💻 Operating system: ...
   ......
   [Relaychain] 📦 Highest known block at #95
   [Relaychain] Running JSON-RPC server: addr=127.0.0.1:9977, allowed origins=["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"]
   [Relaychain] 〽️ Prometheus exporter started at 127.0.0.1:9616
   [Parachain] 🏷  Local node identity is: 12D3KooWF464pkLaHbsfc4DzDkYuhdVE4zSqBHR1gPapLvwsfZtg
   [Relaychain] discovered: 12D3KooWCq8n5PzroHvxEyCbHigb5ZWc6q8WL7E25iuWYbJod9D2 /ip4/10.105.172.196/tcp/30334
   [Relaychain] discovered: 12D3KooWCzfdGstKxFiZ5QdKQVmQVgcBgE8r7FfBrG8bCRsia6Bo /ip4/10.105.172.196/tcp/30333
   [Relaychain] discovered: 12D3KooWCzfdGstKxFiZ5QdKQVmQVgcBgE8r7FfBrG8bCRsia6Bo /ip4/10.96.0.2/tcp/30333
   [Relaychain] discovered: 12D3KooWCq8n5PzroHvxEyCbHigb5ZWc6q8WL7E25iuWYbJod9D2 /ip4/10.96.0.2/tcp/30334
   [Parachain] 💻 Operating system: ...
   ......
   [Parachain] 📦 Highest known block at #0
   [Parachain] Running JSON-RPC server: addr=127.0.0.1:8844, allowed origins=["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"]
   ...
   ```

   您应该会看到模板验证人节点作为独立节点运行，并且其中继节点作为对等节点与本地中继链验证人节点连接。
   如果您没有看到嵌入式中继链与本地中继链节点对等，请尝试禁用防火墙或使用中继节点的地址添加 `bootnodes` 参数。

   它尚未开始创建平行链区块。
   当验证人实际**在中继链上注册**后，将开始创建。

## 在本地中继链上注册

本地中继链和验证人节点运行后，您可以准备在本地中继链上注册平行链。
在实时公共网络中，注册通常涉及[平行链拍卖](https://wiki.polkadot.network/docs/en/learn-auction)。
对于本教程和本地测试，您可以使用 Sudo 事务和 Polkadot/Substrate 门户。
使用 Sudo 事务使您可以绕过获取平行链或平行线程插槽所需的步骤。

要注册平行链：

1. 验证您的本地中继链验证人正在运行。
2. 在浏览器中打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/parachains/parathreads)。
3. 根据需要连接到本地中继链节点。
4. 点击**开发者**并选择**Sudo**。

   ![选择 Sudo 以注册平行链](/media/images/docs/tutorials/parachains/developer-sudo.png)

5. 选择**paraSudoWrapper**，然后选择**sudoScheduleParaInitialize(id, genesis)** 以在下一次中继链会话开始时初始化预留的 paraID。

   对于交易参数：

   - `id`: 输入您预留的 `ParaId`。
     对于本教程，预留的标识符为 2000。

   - `genesisHead`: 点击**文件上传**并上传您为平行链导出的创世状态。
     对于本教程，请选择 `para-2000-genesis` 文件。

   - `validationCode`: 点击**文件上传**并上传您为平行链导出的 WebAssembly 运行时
     对于本教程，请选择 `para-2000-wasm` 文件。
   - `paraKind`: 选择**是**。

   ![设置注册参数](/media/images/docs/tutorials/parachains/register-with-sudo.png)

6. 点击**提交 Sudo**。
7. 查看交易详细信息，然后点击**签名并提交**以授权交易。

   提交交易后，点击**网络**并选择**资源管理器**。

8. 检查最近事件列表中是否有成功的 `sudo.Sudid` _和_ `paras.PvfCheckAccepted`，然后点击该事件以查看有关交易的详细信息。

   ![查看 Sudo 注册事件](/media/images/docs/tutorials/parachains/sudo-registration-event.png)

   初始化平行链后，您可以通过点击**网络**，然后选择**平行链**在 Polkadot/Substrate 门户中查看它。

9. 点击**网络**并选择**平行链**，然后等待新纪元开始。

   中继链跟踪每个平行链的最新区块（头部）。
   当中继链区块完成验证时，已完成[验证过程](https://polkadot.network/the-path-of-a-parachain-block)的平行链区块也会完成验证。
   这就是 Polkadot 为其平行链实现**合并共享安全**的方式。

   平行链在下个纪元连接到中继链并完成其第一个区块后，您可以在 Polkadot/Substrate 门户中查看有关它的信息。

   运行平行链的终端还会显示类似于以下内容的详细信息：

   ```text
   [Relaychain] 💤 Idle (2 peers), best: #90 (0x5f73…1ccf), finalized #87 (0xeb50…68ea), ⬇ 1.4kiB/s ⬆ 1.1kiB/s
   [Parachain] 💤 Idle (0 peers), best: #0 (0x3626…fef3), finalized #0 (0x3626…fef3), ⬇ 1.2kiB/s ⬆ 0.7kiB/s
   [Relaychain] 💤 Idle (2 peers), best: #90 (0x5f73…1ccf), finalized #88 (0xd43c…c3e6), ⬇ 0.7kiB/s ⬆ 0.5kiB/s
   [Parachain] 💤 Idle (0 peers), best: #0 (0x3626…fef3), finalized #0 (0x3626…fef3), ⬇ 1.0kiB/s ⬆ 0.6kiB/s
   [Relaychain] 👶 New epoch 9 launching at block 0x1c93…4aa9 (block slot 281848325 >= start slot 281848325).
   [Relaychain] 👶 Next epoch starts at slot 281848335
   [Relaychain] ✨ Imported #91 (0x1c93…4aa9)
   [Parachain] Starting collation. relay_parent=0x1c936289cfe15fabaa369f7ae5d73050581cb12b75209c11976afcf07f6a4aa9 at=0x36261113c31019d4b2a1e27d062e186f46da0e8f6786177dc7b35959688ffef3
   [Relaychain] 💤 Idle (2 peers), best: #91 (0x1c93…4aa9), finalized #88 (0xd43c…c3e6), ⬇ 1.2kiB/s ⬆ 0.7kiB/s
   [Parachain] 💤 Idle (0 peers), best: #0 (0x3626…fef3), finalized #0 (0x3626…fef3), ⬇ 0.2kiB/s ⬆ 37 B/s
   [Relaychain] ✨ Imported #92 (0x557c…9f73)
   [Parachain] Starting collation. relay_parent=0x557c8c611fdcb6aa86161df3ca71cad71ea159e7213ca36f1ed6b82393b19f73 at=0x36261113c31019d4b2a1e27d062e186f46da0e8f6786177dc7b35959688ffef3
   [Parachain] 🙌 Starting consensus session on top of parent 0x36261113c31019d4b2a1e27d062e186f46da0e8f6786177dc7b35959688ffef3
   [Parachain] 🎁 Prepared block for proposing at 1 (0 ms) [hash: 0x55f3ff62e12bd14385fcb4d386df5469d7360401a3f93ddb944aa0a023f9854a; parent_hash: 0x3626…fef3; extrinsics (2): [0x3e2c…53d1, 0x9080…1966]
   [Parachain] 🔖 Pre-sealed block for proposal at 1. Hash now 0xc905d5b68c052c996d9a4d4863329d24fb6c5f0372f52469ccb909c886025df2, previously 0x55f3ff62e12bd14385fcb4d386df5469d7360401a3f93ddb944aa0a023f9854a.
   [Parachain] ✨ Imported #1 (0xc905…5df2)
   [Parachain] PoV size { header: 0.2177734375kb, extrinsics: 2.8740234375kb, storage_proof: 1.7783203125kb }
   [Parachain] Compressed PoV size: 4.279296875kb
   [Parachain] Produced proof-of-validity candidate. block_hash=0xc905d5b68c052c996d9a4d4863329d24fb6c5f0372f52469ccb909c886025df2
   [Relaychain] 💤 Idle (2 peers), best: #92 (0x557c…9f73), finalized #89 (0x37ea…8ef3), ⬇ 1.4kiB/s ⬆ 1.7kiB/s
   [Parachain] 💤 Idle (0 peers), best: #0 (0x3626…fef3), finalized #0 (0x3626…fef3), ⬇ 0.2kiB/s ⬆ 0.2kiB/s
   [Relaychain] ✨ Imported #93 (0xed3c…0ba3)
   [Relaychain] ✨ Imported #93 (0x2bf8…02c4)
   [Relaychain] 💤 Idle (2 peers), best: #93 (0xed3c…0ba3), finalized #90 (0x5f73…1ccf), ⬇ 1.7kiB/s ⬆ 0.9kiB/s
   [Parachain] 💤 Idle (0 peers), best: #0 (0x3626…fef3), finalized #0 (0x3626…fef3), ⬇ 0.2kiB/s ⬆ 0.1kiB/s
   [Relaychain] 👴 Applying authority set change scheduled at block #91
   [Relaychain] 👴 Applying GRANDPA set change to new set [(Public(88dc3417d5058ec4b4503e0c12ea1a0a89be200fe98922423d4334014fa6b0ee (5FA9nQDV...)), 1), (Public(d17c2d7823ebf260fd138f2d7e27d114c0145d968b5ff5006125f2414fadae69 (5GoNkf6W...)), 1)]
   [Relaychain] ✨ Imported #94 (0xc141…1732)
   [Parachain] Starting collation. relay_parent=0xc141dbb2f9f5108e0cfe4c4c08ced2c48e718c94dc58127e07428b42a80b1732 at=0xc905d5b68c052c996d9a4d4863329d24fb6c5f0372f52469ccb909c886025df2
   [Relaychain] Trying to remove unknown reserved node 12D3KooWCzfdGstKxFiZ5QdKQVmQVgcBgE8r7FfBrG8bCRsia6Bo from SetId(3).
   [Relaychain] 💤 Idle (2 peers), best: #94 (0xc141…1732), finalized #91 (0x1c93…4aa9), ⬇ 0.7kiB/s ⬆ 0.5kiB/s
   [Parachain] 💤 Idle (0 peers), best: #1 (0xc905…5df2), finalized #0 (0x3626…fef3), ⬇ 0.2kiB/s ⬆ 0.1kiB/s
   ...
   ```

   您应该会看到模板验证人节点作为独立节点运行，并且其中继节点作为对等节点与本地中继链验证人节点连接。
   如果您没有看到嵌入式中继链与本地中继链节点对等，请尝试禁用防火墙或使用中继节点的地址添加 `bootnodes` 参数。

   它尚未开始创建平行链区块。
   当验证人实际**在中继链上注册**后，将开始创建。

## 连接并提交交易

到目前为止，您已使用 Polkadot/Substrate 门户连接到本地网络并向本地中继链提交交易。
现在您已经运行并连接到中继链的平行链，您可以使用 Polkadot/Substrate 门户向平行链提交交易。

要连接到平行链并提交交易：

1. 在新的浏览器窗口或选项卡中打开[Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer)。

2. 点击应用程序左上角的网络选择器。

3. 将自定义端点更改为连接到平行链的 WebSocket 端口。

   如果您遵循本教程中的设置，请连接到端口 `8844`。

4. 点击**帐户**并选择**转移**以将资金从 Alice 转移到另一个帐户。

   - 选择一个帐户来发送资金。
   - 输入金额。
   - 点击**进行转移**。
   - 查看交易，然后点击**签名并提交**以授权转移。

5. 点击**帐户**以验证转移是否已完成以及平行链交易是否成功。

   如果交易成功，则表示您拥有一个可工作的平行链。

## 重置区块链状态

您在本教程中连接到中继链的平行链验证人包含平行链的所有区块链数据。
此平行链网络中只有一个节点，因此您提交的任何交易都只存储在此节点上。
中继链不存储任何平行链状态。
中继链仅存储连接到它的平行链的头部信息。

出于测试目的，您可能希望定期清除区块链状态以重新开始。
但是，您应该记住，如果您清除链状态或手动删除数据库，则您将无法恢复数据或恢复链状态。
如果您有要保留的数据，则应确保在清除平行链状态之前对其进行备份。

如果您想重新开始进行干净的测试环境，则应完全删除本地中继链节点和平行链的链状态。

要重置区块链状态：

1. 在运行平行链模板节点的终端中，按 Control-c。
2. 通过运行以下命令清除平行链验证人状态：

   ```bash
   rm -rf /tmp/parachain
   ```

3. 在运行 `alice` 验证人节点或 `bob` 验证人节点的终端中，按 Control-c。

4. 通过运行以下命令清除验证人状态：

   ```bash
   rm -rf /tmp/relay
   ```

## 下一步

- [方法：添加更多平行链节点](/reference/how-to-guides/parachains/add-paranodes/)
- [方法：转换单链](/reference/how-to-guides/parachains/convert-a-solo-chain/)
- [Polkadot 维基](https://wiki.polkadot.network/docs/learn-collator)
- [获取测试网插槽](/tutorials/build-a-parachain/acquire-a-testnet-slot/)
