---
title: Substrate Connect 中的轻客户端
description: 使用 Substrate Connect 将轻客户端集成到您的应用程序中，并启用与任何基于 Substrate 的链的交互。
keywords:
  - Substrate Connect
  - 轻客户端节点
  - Smoldot 轻客户端
  - Polkadot
  - Kusama
  - Westend
  - Rococo
---

通常，为区块链提供对等网络的节点需要大量资源，包括功能强大的高速处理器和高容量存储设备。
相比之下，轻客户端节点可以在资源受限的环境中运行，并嵌入到应用程序中，同时同步来自区块链的数据。

使用轻客户端节点，您可以以安全且去中心化的方式与区块链进行交互，而无需投资于运行完整节点所需的高性能硬件和网络容量。

## JavaScript 生态系统中的轻客户端

对于基于 Substrate 的链，轻客户端节点实现为一个 WebAssembly 客户端（称为 `smoldot`），它可以在浏览器中运行，并使用 JSON-RPC 调用与链进行交互。
为了使 `smoldot` WebAssembly 轻客户端更易于与 JavaScript 和 TypeScript 应用程序集成，有一个基于 `smoldot` 源代码构建的 JavaScript 包，称为 Substrate Connect。

Substrate Connect 可作为 Node.js 包使用，可以使用 `npm` 包管理器安装。
Substrate Connect 包使轻客户端节点能够与 JavaScript 生态系统中的应用程序集成。
将 Substrate Connect 添加到应用程序后，应用程序可以通过 JSON-RPC 消息与轻客户端进行通信，并访问区块链数据。

## 直接从浏览器连接到区块链

使用 Substrate Connect，您的应用程序可以配置为在本地计算机上运行的浏览器内部运行轻客户端节点。
从浏览器中，应用程序用户可以直接与区块链进行交互，而无需连接到任何第三方节点或其他服务器。

通过消除对中介服务器的需求，Substrate Connect 为区块链构建者、应用程序开发人员和最终用户提供了优势。
一些关键优势包括：

- 提高安全性
- 简化网络基础设施
- 降低维护成本
- 使区块链新手用户更容易上手
- 为 Web3 应用程序提供更快的采用路径

## 具有知名名称的区块链网络

您可以使用 Substrate Connect 连接到任何基于 Substrate 的区块链。
但是，您必须指定要连接到的链的正确名称。
有一些知名链名称是为[`WellKnownChain`](https://paritytech.github.io/substrate-connect/api/enums/_substrate_connect.WellKnownChain.html) 枚举类型定义的。

您可以使用列出的名称连接到以下公共区块链网络：

| 要连接到的链                                                                | 使用此链标识符 |
| :-------------------------------------------------------------------------------------- | :------------------------ |
| [Polkadot](https://polkadot.network/)                                                   | `polkadot`                |
| [Kusama](https://kusama.network/)                                                       | `ksmcc3`                  |
| [Westend](https://wiki.polkadot.network/docs/en/maintain-networks#westend-test-network) | `westend2`                |
| [Rococo](https://polkadot.network/rococo-v1-a-holiday-gift-to-the-polkadot-community/)  | `rococo_v2_2`             |

请注意，您必须使用链标识符（如特定网络的链规范中所示），而不是更常用的网络名称。
例如，您必须指定 `ksmcc3` 作为链标识符才能连接到 Kusama。
使用正确的名称对于已分叉的链尤其重要。
例如，`rococo_v2` 和 `rococo_v2_2` 是两个不同的链。

## 在使用 Polkadot-JS API 的应用程序中集成

如果您已经构建了使用现有 Polkadot-JS API 的应用程序，则 `@polkadot/rpc-provider` 包已经包含 `substrate-connect` RPC 提供程序。

要将 `substrate-connect` 添加到您的应用程序：

1. 通过运行您使用的包管理器的适当命令来安装 `@polkadot/rpc-provider` 包。

   例如，如果您使用 `yarn`，请运行以下命令：

   ```bash
   yarn add @polkadot/rpc-provider
   ```

   如果您使用 `npm` 作为包管理器，请运行以下命令：

   ```bash
   npm i @polkadot/rpc-provider
   ```

1. 通过运行您使用的包管理器的适当命令来安装 `@polkadot/api` 包。

   例如，如果您使用 `yarn`，请运行以下命令：

   ```bash
   yarn add @polkadot/api
   ```

   如果您使用 `npm` 作为包管理器，请运行以下命令：

   ```bash
   npm i @polkadot/api
   ```

### 使用 RPC 提供程序连接到知名网络

以下示例说明了如何使用 `rpc-provider` 连接到知名网络，例如 Polkadot、Kusama、Westend 或 Rococo。

```js
import { ScProvider, WellKnownChain } from "@polkadot/rpc-provider/substrate-connect";
import { ApiPromise } from "@polkadot/api";
// Create the provider for a known chain
const provider = new ScProvider(WellKnownChain.westend2);
// Stablish the connection (and catch possible errors)
await provider.connect();
// Create the PolkadotJS api instance
const api = await ApiPromise.create({ provider });
await api.rpc.chain.subscribeNewHeads(lastHeader => {
  console.log(lastHeader.hash);
});
await api.disconnect();
```

### 使用 RPC 提供程序连接到自定义网络

以下示例说明了如何使用 `rpc-provider` 通过指定其链规范来连接到自定义网络。

```js
import { ScProvider } from "@polkadot/rpc-provider/substrate-connect";
import { ApiPromise } from "@polkadot/api";
import jsonCustomSpec from "./jsonCustomSpec.json";
// Create the provider for the custom chain
const customSpec = JSON.stringify(jsonCustomSpec);
const provider = new ScProvider(customSpec);
// Stablish the connection (and catch possible errors)
await provider.connect();
// Create the PolkadotJS api instance
const api = await ApiPromise.create({ provider });
await api.rpc.chain.subscribeNewHeads(lastHeader => {
  console.log(lastHeader.hash);
});
await api.disconnect();
```

### 使用 RPC 提供程序连接到平行链

以下示例说明了如何使用 `rpc-provider` 通过指定其链规范来连接到平行链。

```js
import { ScProvider, WellKnownChain } from "@polkadot/rpc-provider/substrate-connect";
import { ApiPromise } from "@polkadot/api";
import jsonParachainSpec from "./jsonParachainSpec.json";
// Create the provider for the relay chain
const relayProvider = new ScProvider(WellKnownChain.westend2);
// Create the provider for the parachain. Notice that
// we must pass the provider of the relay chain as the
// second argument
const parachainSpec = JSON.stringify(jsonParachainSpec);
const provider = new ScProvider(parachainSpec, relayProvider);
// Stablish the connection (and catch possible errors)
await provider.connect();
// Create the PolkadotJS api instance
const api = await ApiPromise.create({ provider });
await api.rpc.chain.subscribeNewHeads(lastHeader => {
  console.log(lastHeader.hash);
});
await api.disconnect();
```

## 与其他库一起使用 Substrate Connect

上一节演示了如何将 Substrate Connect 提供程序集成到使用 Polkadot-JS API 的应用程序中。
使用此提供程序，您可以创建应用程序，使用户能够通过调用 Polkadot-JS API 方法来与链进行交互。
但是，您可以在不依赖 Polkadot-JS API 的应用程序中安装和使用 @substrate-connect。例如，如果您正在构建自己的应用程序库或编程接口，则可以通过运行您使用的包管理器的适当命令来安装 Substrate Connect 依赖项。

例如，如果您使用 `yarn`，请运行以下命令：

```bash
yarn add @substrate/connect
```

如果您使用 `npm` 作为包管理器，请运行以下命令：

```bash
npm i @substrate/connect
```

### 连接到知名链

以下示例说明了如何使用 Substrate Connect 连接到知名网络，例如 Polkadot、Kusama、Westend 或 Rococo。

```js
import { WellKnownChain, createScClient } from "@substrate/connect";
// Create the client
const client = createScClient();
// Create the chain connection, while passing the `jsonRpcCallback` function.
const chain = await client.addWellKnownChain(WellKnownChain.polkadot, function jsonRpcCallback(response) {
  console.log("response", response);
});
// send a RpcRequest
chain.sendJsonRpc('{"jsonrpc":"2.0","id":"1","method":"system_health","params":[]}');
```

### 连接到平行链

以下示例说明了如何使用 Substrate Connect 连接到平行链。

```js
import { WellKnownChain, createScClient } from "@substrate/connect";
import jsonParachainSpec from "./jsonParachainSpec.json";
// Create the client
const client = createScClient();
// Create the relay chain connection. There is no need to pass a callback
// function because we will sending and receiving messages through
// the parachain connection.
await client.addWellKnownChain(WellKnownChain.westend2);
// Create the parachain connection.
const parachainSpec = JSON.stringify(jsonParachainSpec);
const chain = await client.addChain(parachainSpec, function jsonRpcCallback(response) {
  console.log("response", response);
});
// send a request
chain.sendJsonRpc('{"jsonrpc":"2.0","id":"1","method":"system_health","params":[]}');
```

## API 文档

有关 substrate-connect API 的更多信息，请参阅[Substrate Connect](https://paritytech.github.io/substrate-connect/api/)。

## 浏览器扩展

Substrate Connect 浏览器扩展使用[Substrate Connect](https://github.com/paritytech/substrate-connect) 和[Smoldot 轻客户端](https://github.com/smol-dot/smoldot) 节点模块，并在浏览器启动时更新和同步知名 Substrate 链规范（**Polkadot、Kusama、Rococo、Westend**），将它们保留在扩展内部的最新状态，以实现更快的链同步。

当在浏览器的选项卡中启动集成[Substrate Connect](https://github.com/paritytech/substrate-connect) 的 dApp 时（例如，[polkadotJS/apps](https://polkadot.js.org/apps/?rpc=light%3A%2F%2Fsubstrate-connect%2Fpolkadot#/explorer)），它将从扩展中接收最新的规范，而不是从 dApp 内部导入的最后一个规范进行包装同步；同时，dApp 将在扩展中显示为“已连接”，这意味着它正在使用扩展的引导节点和规范；

您可以从[Substrate Connect](https://substrate.io/developers/substrate-connect/) 下载 Chrome 和 Firefox 扩展，或在[Github 存储库](https://github.com/paritytech/substrate-connect/tree/main/projects/extension) 中找到更多信息。

## 示例项目

- [Burnr](https://paritytech.github.io/substrate-connect/burnr/)

  不安全的可赎回钱包：一个基于轻客户端的、在浏览器中运行的 Substrate 钱包。
  它旨在快速且易于使用，但安全性不如其他解决方案。
  [Github](https://github.com/paritytech/substrate-connect/tree/main/projects/burnr)

- [Multi-demo](https://paritytech.github.io/substrate-connect/demo/)

  涵盖多链和平行链示例的简单演示。
  [Github](https://github.com/paritytech/substrate-connect/tree/main/projects/demo)

## Brave 浏览器 WebSocket 问题

截至**Brave v1.36**，扩展和网页被限制为最多 10 个活动 WebSocket 连接，以防止侧信道攻击。
您可以在[Partition WebSockets Limits to prevent side channels](https://github.com/brave/brave-browser/issues/19990) 中找到有关此更改的更多信息。

如果您使用的是 Brave 浏览器，并且由于已打开允许的最大 WebSocket 连接数量而无法连接，则可以禁用此限制。

要禁用 WebSocket 限制：

1. 在 Brave 浏览器中打开一个新选项卡。

2. 复制 URL [brave://flags/#restrict-websockets-pool](brave://flags/#restrict-websockets-pool)。

3. 将 URL 粘贴到地址栏中以选择**Restrict WebSockets pool** 设置。

4. 点击设置列表并选择**Disabled**。

   ![禁用 Restrict WebSockets pool 设置](/media/images/docs/brave-setting.png)

5. 重新启动浏览器。
