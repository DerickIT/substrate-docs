---
title: 开发人员工具
description:
keywords:
---

由于您将使用 Rust 编写代码，因此请确保您的开发环境具有适合使用 Rust 工作的扩展和插件。
当您开始使用 Substrate 节点模板时，您会看到它包含一组专门为运行时开发设计的核心功能和工具。
但是，还有许多其他可用的专用工具，您可以安装这些工具来补充和扩展您的开发环境或处理特定任务。

在您开始开发基于 Substrate 的区块链时，您可能会发现以下一些工具很有用：

- [Polkadot-JS API](https://polkadot.js.org/docs/api)

  Polkadot-JS API 提供了一组方法库，使您能够使用 JavaScript 查询和交互任何基于 Substrate 的链。
  您可以将 `@polkadot/api` 包添加到任何 JavaScript 或 TypeScript 工作环境中。

  API 公开的多数接口都是通过连接到正在运行的节点动态生成的。
  由于节点的配置决定了哪些接口被公开，因此您可以使用 API 与实现不同功能的自定义链进行交互。
  要使用 API，您必须确定要连接到的链的 URL。
  连接到链的节点后，API 会收集有关链状态及其功能的信息，然后根据收集到的有关该特定链的信息来填充 API 方法。

- [前端模板](https://github.com/substrate-developer-hub/substrate-front-end-template)
  Substrate 前端模板提供了一个预定义的前端应用程序，您可以使用它以最少的配置连接到 Substrate 节点后端。
  该模板使您能够开始尝试 Substrate 节点的基本功能，而无需构建自己的自定义用户界面。
  该模板使用 Create React App 启动项目和 Polkadot-JS API 构建。
- [提交交易命令行界面](https://github.com/paritytech/subxt)

  `subxt-cli` 是一个命令行程序，您可以使用它通过连接到正在运行的节点来下载基于 Substrate 的链的完整配置信息（[元数据](/reference/glossary/#metadata)）。
  与 Polkadot-JS API 类似，您可以使用 `subxt-cli` 程序下载的元数据公开了有关 Substrate 链的信息，使您能够与该链进行交互。
  您还可以使用 `subxt-cli` 程序以人类可读的格式公开有关链的信息。

- [sidecar](https://github.com/paritytech/substrate-api-sidecar)

  @substrate/api-sidecar 包是一个 RESTful 服务，您可以使用它连接到并交互使用 [FRAME](/reference/glossary/#frame/) 开发人员框架构建的 Substrate 节点。
  有关服务支持的端点的信息，请参阅 [Substrate API Sidecar](https://paritytech.github.io/substrate-api-sidecar/dist/)。

您可能还想探索 [Awesome Substrate](https://github.com/substrate-developer-hub/awesome-substrate) 中列出的资源和社区项目。

有关一些最常用的工具的概述，请参阅 [命令行工具](/reference/command-line-tools/)。

## 下一步去哪里

- [命令行工具](/reference/command-line-tools/)
- [node-template](/reference/command-line-tools/node-template/)
- [subkey](/reference/command-line-tools/subkey/)
- [try-runtime](/reference/command-line-tools/try-runtime/)
