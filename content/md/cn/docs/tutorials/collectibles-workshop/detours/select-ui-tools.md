---
title: “绕行：选择前端工具”
---

以下库使用[JSON-RPC API](https://github.com/paritytech/jsonrpsee)使应用程序能够与 Substrate 节点交互：

| 名称 | 说明 | 语言 |
| :---- | :----------- | :-------- |
| [Chain API](https://github.com/paritytech/capi) | 提供一个 TypeScript 工具包，用于创建与基于 Substrate 的链的交互。该工具包包括 FRAME 实用程序、功能性效果系统和流畅的 API，以便在不影响性能或安全性的情况下，为最终用户促进多步骤、多链交互。您可以将此工具包与流行的前端框架（如 React）结合使用。 | TypeScript |
| [Polkadot JS API](https://polkadot.js.org/docs/api) | 提供一个 Javascript 库，用于构建能够在与基于 Substrate 的链交互时动态适应节点更改（例如区块浏览器或与链无关的服务）的应用程序。您可以将此库与流行的前端框架（如 React）结合使用。 | Javascript |
| [Polkadot JS 扩展](https://polkadot.js.org/docs/extension/) | 提供一个 API，用于与使用 Polkadot JS API 构建的浏览器扩展和提供程序进行交互。 | Javascript |
| [Substrate Connect](/learn/light-clients-in-substrate-connect/) | 提供一个库和一个浏览器扩展，用于构建直接使用浏览器内轻客户端节点连接到基于 Substrate 的链的应用程序。Substrate Connect 使您可以构建连接到多个链的应用程序，如果最终用户使用您的应用程序与多个链进行交互，则为他们提供单一体验。 | Javascript |
