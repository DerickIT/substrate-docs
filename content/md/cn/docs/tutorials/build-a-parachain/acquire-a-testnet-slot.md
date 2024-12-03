---
title: 获取测试网插槽
description: 在公共测试网络上获取平行链插槽
keywords:
  - rococo
  - 测试网
  - 水龙头
  - 中继链
---

本教程演示了如何在公共测试网络（例如 [Rococo](https://wiki.polkadot.network/docs/build-pdk#rococo-testnet) 测试网络）上部署平行链。
与私有网络相比，公共测试网络的准入门槛更高，但它代表着将平行链项目迁移到生产网络的重要一步。

## 开始之前

由于 Rococo 是一个共享测试网络，因此本教程需要一些额外的步骤来准备您的环境，而这些步骤对于本地测试来说是不需要的。
在开始之前，请验证以下内容：

- 您知道如何生成和修改链规范文件，如[添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes/)中所述。

- 您知道如何生成和存储密钥，如[添加信任节点](/tutorials/build-a-blockchain/add-trusted-nodes/)中所述。

- 您已在本地计算机上完成了[准备本地中继链](/tutorials/build-a-parachain/prepare-a-local-relay-chain/)和[连接本地平行链](/tutorials/build-a-parachain/connect-a-local-parachain/)教程。

## 使用帐户和代币入门

要在 Rococo 上执行任何操作，您需要 ROC 代币，并且要存储代币，您必须能够访问与 Substrate 兼容的数字货币钱包。
您不能在任何公共环境中使用[开发密钥和帐户](/reference/command-line-tools/subkey/#predefined-accounts-and-keys)。
有很多可用于持有数字货币的选项——包括硬件钱包和基于浏览器的应用程序——有些比其他的更可靠。
在选择一个之前，您应该自己进行研究。

但是，您可以使用[Polkadot/Substrate 门户](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)来开始测试。

要准备一个帐户：

1. 将[Polkadot/Substrate 门户](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)连接到 Rococo 网络。

2. 点击并选择**帐户**。

1. 点击**添加帐户**。

1. 复制您的秘密助记词并将其放在安全的地方，然后点击**下一步**。

2. 输入帐户名称和密码，然后点击**下一步**。

1. 点击**保存**。

2. 前往 https://faucet.polkadot.io，在表单输入中提供您的公共地址并提交，以在您的钱包中获得 100 个 ROC 代币。

   >或者，您可以加入[Rococo Element 频道](https://matrix.to/#/#rococo-faucet:matrix.org)并发送一条包含`!drip`和您的 Rococo 公共地址的消息，以在您的钱包中获得 100 个 ROC。
   >例如，发送类似于以下内容的消息：`!drip 5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1`

## 预留平行链标识符

在您可以在 Rococo 上注册为平行线程之前，您必须预留一个平行链标识符。
步骤类似于您在[连接本地平行链](/tutorials/build-a-parachain/connect-a-local-parachain/)中遵循的步骤，以在本地中继链上预留标识符。
但是，对于公共测试网，系统会为您分配下一个可用的标识符。

要预留标识符：

1. 将[Polkadot/Substrate 门户](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)连接到 Rococo 网络。

1. 点击**网络**并选择**平行链**。

   ![从网络菜单中选择平行链](/media/images/docs/tutorials/parachains/rococo-select-parachains.png)

2. 点击**平行线程**，然后点击**ParaId**。

3. 查看交易并记下分配给您的平行链标识符，然后点击**提交**。

4. 输入您的密码以验证您的身份，然后点击**签名并提交**。

   ![授权交易](/media/images/docs/tutorials/parachains/auth-rococo-id.png)

5. 点击**网络**并选择**资源管理器**以检查最近事件列表中是否有成功的 `registrar.Reserved`。

   ![Rococo 网络的预留标识符](/media/images/docs/tutorials/parachains/rococo-reserved-id-event.png)

## 修改链规范文件

注册平行链所需的文件必须指定要连接到的正确中继链和分配给您的平行链标识符。
要进行这些更改，您必须构建和修改平行链的链规范文件。
在本教程中，中继链为 `rococo`，而不是[连接本地平行链](/tutorials/build-a-parachain/connect-a-local-parachain/)教程中使用的 `rococo-local`，para 标识符为 `4105`。

要修改链规范：

1. 通过运行以下命令生成平行链模板节点的纯文本链规范：

   ```bash
   ./target/release/parachain-template-node build-spec --disable-default-bootnode > plain-parachain-chainspec.json
   ```

1. 在文本编辑器中打开平行链模板节点的纯文本链规范。

2. 将 `relay-chain` 设置为 `rococo`，并将 `para_id` 设置为您已分配的标识符。

   例如，如果您的预留标识符为 `4105`，请将 `para_id` 字段设置为 `4105`：

   ```json
   ...
   "relay_chain": "rococo",
   "para_id": 4105,
   "codeSubstitutes": {},
   "genesis": {
      ...
   }
   ```

3. 将 `parachainId` 设置为您先前预留的平行链标识符。

      ```json
      ...
         "parachainSystem": null,
         "parachainInfo": {
           "parachainId": 4105
         },
      ...
      ```

1. 将您帐户的公钥添加到会话密钥部分。每个配置的会话密钥都需要一个正在运行的验证人。

   ```json
   ...
      "session": {
        "keys": [
         [
           "5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1",
           "5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1",
           {
            "aura": "5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1"
           }
         ],
        ]
      }
    ...
    ```

2. 保存更改并关闭纯文本链规范文件。

3. 通过运行以下命令从修改后的链规范文件生成原始链规范文件：

   ```bash
   ./target/release/parachain-template-node build-spec --chain plain-parachain-chainspec.json --disable-default-bootnode --raw > raw-parachain-chainspec.json
   ```

## 导出所需文件

要准备注册平行链验证人：

1. 通过运行类似于以下命令的命令导出平行链的 WebAssembly 运行时：

   ```bash
   ./target/release/parachain-template-node export-genesis-wasm --chain raw-parachain-chainspec.json para-4105-wasm
   ```

2. 通过运行类似于以下命令的命令生成平行链创世状态：

   ```bash
   ./target/release/parachain-template-node export-genesis-state --chain raw-parachain-chainspec.json para-4105-genesis-state
   ```

## 启动验证人节点

您必须使验证人的端口可公开访问和可发现，才能使平行链节点与 Rococo 验证器节点对等以生成区块。
您可以使用 `--port` 命令行选项指定要使用的端口。
例如，您可以使用类似于以下命令的命令启动验证人：

```bash
./target/release/parachain-template-node --collator \
  --chain raw-parachain-chainspec.json \
  --base-path /tmp/parachain/pubs-demo \
  --port 50333 \
  --rpc-port 8855 \
  -- \
  --execution wasm \
  --chain rococo \
  --port 50343 \
  --rpc-port 9988
  ```

在此示例中，第一个 `--port` 设置指定验证人节点的端口，第二个 `--port` 指定嵌入式中继链节点的端口。
第一个 `--rpc-port` 设置指定您可以使用它通过 Polkadot-JS API 调用或 Polkadot/Substrate 门户应用程序连接到验证人的端口。
第二个 `--rpc-port` 指定使用 Polkadot-JS API 或 Polkadot/Substrate 门户应用程序连接到嵌入式中继链的端口。

## 注册为平行线程

在您可以在公共中继链上租赁插槽以成为平行链之前，您必须在 Rococo 上注册为平行线程。

要注册为平行线程：

1. 将[Polkadot/Substrate 门户](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)连接到 Rococo 网络。

1. 点击**网络**并选择**平行链**。

1. 点击**平行线程**，然后点击**ParaThread**。

   ![点击 ParaThread 进行注册](/media/images/docs/tutorials/parachains/rococo-parathread.png)

2. 验证平行链所有者和平行链标识符，并上传包含平行链的 WebAssembly 验证函数和创世状态的文件，然后点击**提交**。

3. 输入您的密码以验证您的身份，然后点击**签名并提交**。

4. 向下滚动平行线程列表，并验证您的平行线程注册是否为**入职中**：

   ![平行线程入职中](/media/images/docs/tutorials/parachains/rococo-onboarding.png)

   您还可以通过在网络资源管理器中检查 `registrar.Registered` 事件来验证您的注册请求。

   提交注册请求后，需要两个[会话](https://wiki.polkadot.network/docs/glossary#session)才能完成平行线程的入职。

## 请求平行链插槽

平行链作为平行线程激活后，相关项目团队应[打开请求](https://github.com/paritytech/subport/issues/new?assignees=&labels=Rococo&template=rococo.yaml) Rococo 上的**永久**或**临时平行链插槽**。

- **永久插槽**通常分配给已完成成功插槽租赁拍卖并在Polkadot上拥有插槽的平行链的团队。

  永久插槽使这些团队能够在实时公共环境中持续测试其代码库与最新Polkadot功能的兼容性。
  只有有限数量的永久插槽可用。

- **临时插槽**是以连续的循环方式动态分配的平行链插槽。

  在每个租赁期开始时，一定数量的平行线程（最多在中继链配置中定义的最大数量）会自动升级为平行链，持续一段时间。
  在租赁期结束时处于活动状态的平行链会自动降级为平行线程，以便在后续期间为其他人使用空出的插槽。
  具有动态分配的临时插槽使没有Polkadot上平行链插槽的团队能够在更真实的网络环境中更频繁地测试其运行时。

### 提交插槽请求

Rococo 运行时需要 `sudo` 访问权限才能分配插槽。
例如，Rococo 运行时指定用于分配插槽的帐户必须具有根级权限：

```text
AssignSlotOrigin = EnsureRoot<Self::AccountId>;
```

最终，插槽分配旨在通过 Rococo 治理由社区驱动。
但是，Rococo `sudo` 密钥目前由 Parity Technologies 控制。
因此，您必须提交[Rococo 插槽请求](https://github.com/paritytech/subport/issues/new?assignees=&labels=Rococo&template=rococo.yaml)才能接收插槽分配。
分配插槽后，您将收到通知并准备好连接。

### 使用管理帐户分配插槽

如果您拥有具有 `AssignSlotOrigin` 原点的帐户，则可以使用该帐户在 Rococo 网络上分配临时插槽。

要分配临时插槽：

1. 将[Polkadot/Substrate 门户](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)连接到 Rococo 网络。

2. 点击**开发者**并选择[**外部函数**](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/extrinsics)。

3. 选择您要用来提交交易的帐户。

4. 选择 `assignedSlots` 模块。

5. 选择 `assignTempParachainSlot` 函数。

6. 输入分配给您的预留平行链标识符。

7. 为 `LeasePeriodStart` 选择 `Current`。

   如果当前插槽已满，系统会为您分配下一个可用的插槽。

8. 点击**提交事务**。

9. 输入您的密码以验证您的身份，然后点击**签名并提交**。

   如果您的帐户权限不足，则交易将失败并显示 `BadOrigin` 错误。

### 租赁期限

Rococo 上分配的平行链插槽的当前租赁期限和插槽可用性设置如下：

- **永久插槽租赁期限**: 1 年（365 天）
- **临时插槽租赁期限**: 3 天
- **永久插槽最大数量**: 最多 25 个永久插槽
- **临时插槽最大数量**: 最多 20 个临时插槽
- **每个租赁期分配的临时插槽最大数量**: 每个 3 天的临时租赁期最多 5 个临时插槽

这些设置可能会根据社区的需求而更改。

## 测试您的平行链

分配并激活插槽后，您可以在 Rococo 测试网络上测试您的平行链。
请注意，当临时插槽租赁期结束时，平行链会自动降级为平行线程。
注册和批准的插槽会以循环方式自动循环，因此您可以预期自己会不时作为平行链重新上线。

## 下一步

在对 Rococo 等测试网络上的平行链进行严格测试和验证后，您可以考虑通过[插槽拍卖](https://guide.kusama.network/docs/learn-auction/)加入生产网络，例如[Kusama](https://kusama.network/)。
