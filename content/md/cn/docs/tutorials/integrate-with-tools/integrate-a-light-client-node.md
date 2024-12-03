---
title: 集成轻客户端节点
description: 演示如何在浏览器中使用 Substrate Connect 连接到基于 Substrate 的区块链。
keywords:
---

正如您在[Substrate Connect 中的轻客户端](/learn/light-clients-in-substrate-connect/)中学到的那样，轻客户端节点以最少的硬件和软件要求提供对区块链数据的安全和去中心化访问。

本教程演示了如何使用轻客户端连接到任何基于 Substrate 的区块链。
为了说明这一点，您将学习如何将您的应用程序连接到[Statemint 平行链](https://wiki.polkadot.network/docs/learn-statemint)。
Statemint 是一个连接到 Polkadot 的系统平行链，并具有可公开访问的链规范文件。

## 开始之前

在开始之前，请验证以下内容：

- 您已设置好用于处理 Javascript 和 Typescript 的代码编辑器。
- 您已安装[`yarn`包管理器](https://classic.yarnpkg.com/lang/en/docs/install/)。

## 教程目标

通过完成本教程，您将实现以下目标：

- 使用 Substrate Connect Javascript 库连接到 Polkadot 中继链。
- 学习如何为 Substrate Connect 指定要使用的自定义链规范文件。
- 连接到与自定义链规范关联的平行链。

## 连接到众所周知的链

在轻客户端能够连接到网络之前，您必须拥有一个 Web 应用程序，该应用程序指定轻客户端应连接到的网络、它要与其通信的节点以及它在创世时必须具有的共识关键状态。
此信息可在网络的[链规范](/build/chain-spec/)文件中找到。

Substrate Connect 预先配置为识别[WellKnownChain](https://paritytech.github.io/substrate-connect/api/enums/_substrate_connect.WellKnownChain.html)枚举列表中定义的多个链。
这些众所周知的链是：

- Polkadot，标识为`polkadot`
- Kusama，标识为`ksmcc3`
- Rococo，标识为`rococo_v2_2`
- Westend，标识为`westend2`

要连接到这些链之一：

1. 在您的计算机上打开新的终端 shell。

2. 通过运行以下命令克隆`empty-webapp`模板来创建一个 Web 应用程序以使用 Substrate Connect：

   ```bash
   git clone https://github.com/bernardoaraujor/empty-webapp
   ```

3. 通过运行以下命令更改到`empty-webapp`目录：

   ```bash
   cd empty-webapp
   ```

4. 通过运行以下命令安装 Substrate Connect：

   ```bash
   yarn add @substrate/connect
   ```

5. 通过运行以下命令安装来自 Polkadot-JS RPC 提供程序的依赖项：

   ```bash
   yarn add @polkadot/rpc-provider
   ```

6. 通过运行以下命令安装来自 Polkadot-JS API 的依赖项：

   ```bash
   yarn add @polkadot/api
   ```

   安装这些依赖项后，您可以在示例应用程序中使用它们。

7. 在您首选的代码编辑器中打开`empty-webapp/index.ts`文件。
8. 复制并粘贴以下应用程序代码以创建一个 Substrate Connect 实例，其中`substrate-connect`作为提供程序，该提供程序使用`polkadot`链规范文件连接到 Polkadot 中继链。

   ```typescript
   import { ScProvider } from "@polkadot/rpc-provider/substrate-connect";
   import * as Sc from "@substrate/connect";
   import { ApiPromise } from "@polkadot/api";

   window.onload = () => {
     void (async () => {
       try {
         const provider = new ScProvider(Sc, Sc.WellKnownChain.polkadot);

         await provider.connect();
         const api = await ApiPromise.create({ provider });
         await api.rpc.chain.subscribeNewHeads((lastHeader: { number: unknown; hash: unknown }) => {
           console.log(`💡 New block #${lastHeader.number} has hash ⚡️ ${lastHeader.hash}`);
         });
       } catch (error) {
         console.error(<Error>error);
       }
     })();
   };
   ```

   在此示例应用程序代码中，创建 Substrate Connect 实例类似于您如何使用 Polkadot-JS API 创建 WebSocket 实例。
   仅使用 Polkadot-JS API，您将创建如下所示的实例：

   ```javascript
   // Import
   import { ApiPromise, WsProvider } from "@polkadot/api";

   // Construct
   const wsProvider = new WsProvider("wss://rpc.polkadot.io");
   const api = await ApiPromise.create({ provider: wsProvider });
   ```

   对于 Substrate Connect，您将 WebSocket (`WsProvider`) 提供程序替换为 Substrate Connect (`ScProvider`) 提供程序，并且您不是指定 WebSocket URL 客户端地址，而是指定要连接到的网络的链规范 (`WellKnownChain.polkadot`)。

9. 通过运行以下命令安装任何剩余的依赖项：

   ```bash
   yarn
   ```

10. 通过运行以下命令启动 Web 应用程序：

```bash
yarn dev
```

如果您在启动本地服务器时遇到编译器错误，则您可能缺少当前`yarn`配置中未考虑到的依赖项。
如果缺少依赖项，则可以通过运行类似于以下命令的命令来添加包：

```bash
yarn add -D buffer
```

11. 验证浏览器是否打开了 URL `http://localhost:3001/`。

12. 打开浏览器的浏览器控制台。

导航到并打开浏览器控制台的方式因您使用的浏览器和操作系统而异。
例如，在 Chrome 上，选择**更多工具**、**开发者工具**，然后点击**控制台**。

13. 验证`smoldot`进程是否已初始化，然后是来自 Polkadot 的传入区块的哈希。

例如，控制台应显示类似于以下内容的日志条目：

```console
[substrate-connect-extension] [smoldot] Smoldot v0.7.7
[substrate-connect-extension] [smoldot] Chain initialization complete for polkadot. Name: "Polkadot". Genesis hash: 0x91b1…90c3. State root hash: 0x29d0d972cd27cbc511e9589fcb7a4506d5eb6a9e8df205f00472e5ab354a4e17. Network identity: 12D3KooWRse9u6Z9ukP4C92YCCH2gXziNm8ThRch2owaaFh9H6D1. Chain specification or database starting at: 0x7f52…8902 (#14614672)
...
💡 New block #14614893 has hash ⚡️ 0x18f8086952aa5f8f1f8a36ea05af462f6bb26615b481145f7c5daa24ebc0c4cd
💡 New block #14614894 has hash ⚡️ 0x92ca6fd51bc7a2fc5991441e9736bcccf3be45cee6fc88d40d145fc4211ba477
💡 New block #14614894 has hash ⚡️ 0x2353ce49f06206c6dd9882200666fa7d51fc43c1cc6a61cca81ce9fa543409cb
```

此简单的 Web 应用程序仅连接到 Polkadot 以检索区块哈希。
此应用程序的主要目的是演示如何连接到链，而无需使用集中式网络入口点，例如特定 RPC 节点的 URL。
但是，您可以扩展此应用程序以执行更多操作，因为在您将`WsProvider`替换为`ScProvider`后，您可以简单地使用现有的[Polkadot-JS API](https://polkadot.js.org/docs/)来编写应用程序的代码。

14. 按 Control-c 停止`smoldot`轻客户端节点。

## 连接到自定义链规范

连接到自定义链规范或可公开访问的平行链类似于连接到众所周知的链之一。
代码的主要区别在于您必须显式标识 Substrate Connect 要使用的链规范。

要连接到 Statemint：

1. 从[cumulus 存储库](https://github.com/paritytech/cumulus/blob/445f9277ab55b4d930ced4fbbb38d27c617c6658/parachains/chain-specs/statemint.json)下载自定义链规范文件。

2. 将下载的链规范复制到您在[连接到众所周知的链](#connect-to-a-well-known-chain)中创建的`empty-webapp`目录。

3. 在代码编辑器中打开`index.ts`文件。
4. 将内容替换为以下代码。

   ```typescript
   import { ApiPromise } from "@polkadot/api";
   import { ScProvider } from "@polkadot/rpc-provider/substrate-connect";
   import * as Sc from "@substrate/connect";
   import jsonParachainSpec from "./statemint.json";

   window.onload = () => {
     void (async () => {
       try {
         const relayProvider = new ScProvider(Sc, Sc.WellKnownChain.polkadot);
         const parachainSpec = JSON.stringify(jsonParachainSpec);
         const provider = new ScProvider(Sc, parachainSpec, relayProvider);

         await provider.connect();
         const api = await ApiPromise.create({ provider });
         await api.rpc.chain.subscribeNewHeads((lastHeader: { number: unknown; hash: unknown }) => {
           console.log(`💡 New block #${lastHeader.number} has hash ⚡️ ${lastHeader.hash}`);
         });
       } catch (error) {
         console.error(<Error>error);
       }
     })();
   };
   ```

   正如您所看到的，此代码有一些重要的区别。

   - `statemint.json`链规范文件被导入到`jsonParachainSpec`对象中。
   - 链规范被转换为 JSON 编码的字符串并存储在`parachainSpec`变量中，以便可以与 Web 服务器交换。
   - 为`polkadot`中继链创建`ScProvider`提供程序，但将其用作创建和连接到平行链提供程序的参数。

Substrate Connect 需要此信息来确定平行链与其通信的中继链。

6. 通过运行以下命令启动 Web 应用程序：

   ```bash
   yarn dev
   ```

7. 验证浏览器是否打开了 URL `http://localhost:3001/`。

8. 打开浏览器的浏览器控制台。

9. 验证`smoldot`进程是否已初始化，然后是来自 Polkadot 的传入区块的哈希。

   例如，控制台应显示类似于以下内容的日志条目：

   ```console
   [substrate-connect-extension] [smoldot] Parachain initialization complete for statemint. Name: "Statemint". Genesis hash: 0x68d5…de2f. State root hash: 0xc1ef26b567de07159e4ecd415fbbb0340c56a09c4d72c82516d0f3bc2b782c80. Network identity: 12D3KooWNicu1ZCX6ZNUC96B4TQSiet2NkoQc7SfitxWWE4fQgpK. Relay chain: polkadot (id: 1000)
   ...
   💡 New block #3393027 has hash ⚡️ 0x2401313496be4b1792d704f83b684be6bd2188a618581d30b3addb3648c4ad3a
   [substrate-connect-extension] [smoldot] Smoldot v0.7.7. Current memory usage: 222 MiB. Average download: 63.1 kiB/s. Average upload: 735 B/s.
   💡 New block #3393028 has hash ⚡️ 0x512af8ad5577f509f3f5123916ff2da6ca0f86df8099eafbc0bc001febec62dd
   ```

恭喜，您已创建了一个简单的 Web 应用程序，该应用程序使用浏览器内轻客户端连接到 Statemint 和 Polkadot。
请查看[此演示](https://github.com/paritytech/substrate-connect/tree/main/projects/demo)，了解如何从同一应用程序中添加对更多链的连接。
