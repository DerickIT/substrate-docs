---
title: 探索 sidecar 端点
description: 演示如何使用 Postman 探索 Substrate REST 服务（sidecar）API。
keywords:
---

<div class="warning">
	 请参考<a href="https://paritytech.github.io/substrate-api-sidecar/dist/">Sidecar 文档</a>、<a href="https://github.com/paritytech/substrate-api-sidecar?tab=readme-ov-file#note">自述文件</a>和<a href="https://github.com/paritytech/substrate-api-sidecar/tree/master/guides">guides 文件夹</a>以获取有关此主题的最新文档。
</div>

Substrate [sidecar](https://github.com/paritytech/substrate-api-sidecar)服务为使用 FRAME 构建的 Substrate 区块链节点提供了 REST API。
sidecar REST 服务公开了大量端点，使您可以与节点、帐户、事务、平行链以及基于 Substrate 的区块链的许多其他组件进行交互。

在本教程中，您将使用 Postman 来探索`sidecar`服务。
Postman 是一个流行的平台，用于检查、构建和测试应用程序编程接口。
Postman 提供了一个易于使用的界面，使经验丰富和经验不足的 API 开发人员都能够使用基于浏览器的客户端或桌面客户端进行协作和试验。

您将使用[预定义的 API 集合](https://documenter.getpostman.com/view/24602305/2s8YsqWaj8#intro)来探索`sidecar`通过 RESTful 端点提供的功能。
预定义的 API 集合提供了一组使用 Postman 创建的已保存请求，您可以自定义和重用这些请求。
通过使用使用 Postman 创建的 API 集合，您可以访问诸如内联文档、可重用变量以访问数据、参数形成中的错误检测等功能。

## 开始之前

在开始之前，请验证以下内容：

- 您已完成[快速入门](/quick-start/)，并在本地安装并运行了 Substrate 节点模板。
- 您的本地计算机上有 Node.js（版本 14 或更高版本），以便下载和安装`sidecar`包。
- 您拥有或可以创建一个[Postman](https://www.postman.com/)帐户。
- 您可以访问或可以安装**Postman for Web**或**Postman for Mac**客户端。

## 教程目标

通过完成本教程，您将学习如何使用 Postman 探索`sidecar`API REST 服务，包括如何：

- 下载和安装`sidecar`服务。
- 导入 Postman API 集合。
- 设置工作环境。
- 向`sidecar`API 发出 API 请求。
- 保存数据以供进一步使用。

## 下载和安装 sidecar

要下载和安装`sidecar`：

1. 打开终端 shell。
2. 使用`npm`或`yarn`全局或本地安装`@substrate/api-sidecar`服务。

   例如，如果您使用`yarn`，请运行以下命令以全局安装`@substrate/api-sidecar`服务：

   ```bash
   yarn global add @substrate/api-sidecar
   ```

   如果您使用`npm`作为您的包管理器，请运行以下命令以全局安装`@substrate/api-sidecar`服务：

   ```bash
   npm install -g @substrate/api-sidecar
   ```

1. 验证您是否有正在运行的 Substrate 节点供服务连接。

   默认情况下，服务尝试使用 ws://127.0.0.1:9944 连接到本地主机。
   您可以通过修改 SAS_SUBSTRATE_URL 环境设置来配置服务以使用不同的 URL。
   如果您想连接到本地运行的节点模板并使用默认端口，则无需进行任何配置。

2. 通过运行以下命令启动服务：

   ```bash
   substrate-api-sidecar
   ```

   如果您使用的是默认配置，则应该会看到类似于以下内容的输出：

   ```text
   SAS:
     📦 LOG:
        ✅ LEVEL: "info"
        ✅ JSON: false
        ✅ FILTER_RPC: false
        ✅ STRIP_ANSI: false
     📦 SUBSTRATE:
        ✅ URL: "ws://127.0.0.1:9944"
        ✅ TYPES_BUNDLE: undefined
        ✅ TYPES_CHAIN: undefined
        ✅ TYPES_SPEC: undefined
        ✅ TYPES: undefined
     📦 EXPRESS:
        ✅ BIND_HOST: "127.0.0.1"
        ✅ PORT: 8080
   2023-01-04 14:35:41 info: Version: 14.2.2
   2023-01-04 14:35:41 warn: API/INIT: RPC methods not decorated: transaction_unstable_submitAndWatch, transaction_unstable_unwatch
   2023-01-04 14:35:41 info: Connected to chain Development on the solo-template-node client at ws://127.0.0.1:9944
   2023-01-04 14:35:41 info: Listening on http://127.0.0.1:8080/
   2023-01-04 14:35:41 info: Check the root endpoint (http://127.0.0.1:8080/) to see the available endpoints for the current node
   ```

## 导入 API 集合

要使用`sidecar`的预定义 API 集合：

1. 在浏览器中打开预定义的[Substrate API Sidecar](https://documenter.getpostman.com/view/21393319/2sA3Qs9C9M#intro)API 集合。
2. 点击页面右上角的**在 Postman 中运行**。
3. 选择使用 Postman for Web 还是 Postman for Mac 桌面客户端运行集合。

   如果您使用的是 macOS 计算机，则应使用桌面客户端运行集合，因为桌面客户端通常更稳定并支持更多功能。
   如果您的本地计算机上未安装 Postman for Mac，请点击**获取应用程序**进行下载。

   ![在 Postman 中运行 API 集合](/media/images/docs/tutorials/postman-sidecar/run-in-postman.png)

1. 在 Postman 中选择一个工作区，然后点击**导入**将预定义的集合添加到您的 Postman 工作区。

   在 Postman 中打开 Substrate API `sidecar` 集合后，您可以开始定义环境变量。

## 定义环境变量

预定义的 Postman API 集合带有一个名为`Dev`的内置开发环境。
此环境包含预定义的变量，供您在使用预定义查询向`sidecar`服务发出不同的 API 请求时使用。
`Dev`环境中定义的变量包括以下内容：

- `url`用于指定要使用的`sidecar`REST API 的 URL。
  默认值为`http://127.0.0.1:8080`。
- `account`用于指定区块链上特定用户的帐户标识符。
- `number`用于指定区块链中特定区块的编号。
- `extrensicIndex`用于指定区块中特定外部函数的索引号。
- `assetId`用于指定模块资产的标识符。
- `storageItemId`用于指定模块存储项的标识符。
- `paraId`用于指定特定平行链的唯一数字标识符。

所有请求都需要`url`变量。
`Dev`环境提供`http://127.0.0.1:8080`作为`url`的默认值，因为当您启动`sidecar`服务的新实例时，此 URL 是创建的默认 REST API 地址。
如果您已将`sidecar`服务设置为使用外部托管位置或使用环境设置更改了默认本地 URL，则必须相应地更改`url`变量的默认值。
除了`url`变量外，不同类型的请求还需要您定义不同的变量。

例如，在本教程中，返回区块链上特定帐户余额的查询要求您为`account`变量定义一个值。

## 获取端点列表

您可以使用 Postman 向`sidecar`发送`GET`请求以返回 API 集合可用的所有活动端点的列表。
此请求只需要定义`url`变量。
在大多数情况下，您可以使用本地主机 IP 地址和端口的默认值，除非您在本地开发环境之外使用`sidecar`。

要获取端点列表：

1. 在 Postman 客户端中，点击**集合**并打开**Substrate API Sidecar**集合。
2. 从列表中选择**GET 端点列表**请求。

   ![选择 GET 端点请求](/media/images/docs/tutorials/postman-sidecar/select-get-endpoints.png)

   选择**获取端点列表**请求后，Postman 将显示此请求可用的选项。
   由于此请求只需要 URL 并且不使用任何其他参数，因此此处没有要配置的设置。

   ![设置请求参数](/media/images/docs/tutorials/postman-sidecar/endpoint-parameters.png)

   如果请求需要其他参数，则您可以通过为参数提供不同的值并查看这些值如何提供 API 的不同响应来进行试验。

3. 通过键入`{{url}}`来指定要连接到的 URL。

   ![指定请求的 URL](/media/images/docs/tutorials/postman-sidecar/specify-url.png)

4. 点击**发送**。

   发送请求后，您应该会看到 Postman 中的响应部分已填充数据。
   默认情况下，数据以 JSON 格式显示，但您可以将响应更改为 XML、HTML 或纯文本。
   您应该会看到类似于以下部分响应的信息：

   ```json
   {
     "docs": "https://paritytech.github.io/substrate-api-sidecar/dist",
     "github": "https://github.com/paritytech/substrate-api-sidecar",
     "version": "14.2.2",
     "listen": "127.0.0.1:8080",
     "routes": [
        {
            "path": "/accounts/:address/asset-balances",
            "method": "get"
        },
        {
            "path": "/accounts/:address/asset-approvals",
            "method": "get"
        },
      ]
   }
   ```

5. 如果您想使用它来构建自己的应用程序的功能，请点击**保存响应**将此响应保存到输出文件中。

   您还可以将响应保存为示例以供将来参考。
   如果您将响应保存为示例，则它将添加到“获取端点列表”请求选项的下方。

   此端点列表请求是简单请求的有用演示。
   但是，更常见的是，您需要使用 sidecar 端点来访问特定数据，这将要求您操作为不同变量定义的值以检索所需的信息。

   这是本教程的下一步。

## 获取帐户信息

现在您已经熟悉了发出简单的请求，让我们执行另一个`GET`请求，但这次修改其中一个环境变量的值。
在此请求中，您将请求链上特定帐户的余额。
使用[substrate 节点模板](https://github.com/paritytech/polkadot-sdk-solochain-template)作为工作区块链，此请求查询`Alice`用户的帐户余额。

要获取 Alice 地址的帐户信息：

1. 打开新的终端 shell。
2. 复制 Alice 的帐户地址。

   例如，通过运行以下命令来检查`//Alice`帐户的关键信息：

   ```bash
   ./target/release/solo-template-node key inspect //Alice
   ```

   复制帐户的公钥：

   ```
   5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
   ```

3. 在 Postman 客户端中，选择`Dev`环境以显示环境变量及其值列表。

   对于每个变量，都有初始值和当前值。

   - `初始值`是在启动新的`Dev`环境时加载变量的值。
   - `当前值`是当前会话的值。

   `当前值`仅在本地保存，绝不会发送到 Postman 或与使用相同 Postman API 集合的其他团队成员共享。

1. 将 Alice 的帐户地址粘贴到`当前值`字段中。

   ![Postman 环境中的帐户变量](/media/images/docs/tutorials/postman-sidecar/dev-variables.png)

1. 点击**集合**并打开**Substrate API Sidecar**集合。

1. 选择**帐户**文件夹，然后选择**GET 帐户余额信息**。

   ![选择帐户余额信息请求](/media/images/docs/tutorials/postman-sidecar/account-balance.png)

2. 使用您先前使用的`url`变量和您在步骤 4 中设置的`account`变量配置请求：

   ![指定请求的端点](/media/images/docs/tutorials/postman-sidecar/account-request.png)


3. 点击**发送**。

   您应该会收到一个响应，该响应在 JSON 格式中看起来类似于以下内容：

   ```json
   {
     "at": {
        "hash": "0xfc354903a665c7847ba4f83dd9e5fb0389e31bc2015086aca56a68bb345493a5",
        "height": "1189"
     },
     "nonce": "0",
     "tokenSymbol": "UNIT",
     "free": "1152921504606846976",
     "reserved": "0",
     "miscFrozen": "0",
     "feeFrozen": "0",
     "locks": []
   }
   ```

  您可以将此响应保存为文件或 Postman 中的示例，以帮助您创建测试或构建您自己的应用程序。

## 接下来的步骤

在本教程中，您学习了如何：

- 将 API 集合导入到 Postman。
- 定义要在集合中使用的环境变量。
- 使用自定义变量定义发送 API 请求。
- 检查和保存 API 的响应。

您现在拥有了在 Postman 中使用`sidecar`端点以及开始构建出于您自己目的而使用`sidecar`API 的应用程序的良好工作基础。
要自己探索更多内容，请考虑以下任务：

- 使用预定义集合中的其他端点发送请求。
- 在 Postman 中编写测试以进行调试。
- 设置 Postman 流程，使您可以连接一系列 API REST 请求。
