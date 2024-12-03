---
title: 准备工作环境
description:
tutorial: 1
---

为了充分利用本研讨会，您需要一个可用的开发环境。
为了确保您拥有所需的一切，请查看此预检清单。
如果您发现缺少某些内容，请点击相应的链接。

研讨会中的说明假设您在本地环境中工作。

## 支持的操作系统

要设置本地开发环境，您必须拥有以下受支持的操作系统之一：

- [ ] Linux 发行版
- [ ] macOS
- [ ] Windows 子系统 Linux

## Rust 编程语言和工具链

Substrate 是使用 Rust 构建的，Rust 是一种现代的类型安全编程语言。
Rust 编译器最大限度地减少了错误进入代码的可能性，并生成可在大多数操作系统和 WebAssembly 目标上运行的二进制文件。

- [ ] Rust 已在本地安装。

  如果您不确定，请打开终端并运行`rustup show`。
  如果您的计算机上未安装 Rust，请按照[安装](/install)中针对您操作系统的说明进行操作。
  安装说明中的最后一步是验证默认节点模板是否已编译。

- [ ] Substrate 节点已在本地编译。

如果您不熟悉 Rust，请记住，本研讨会并非关于_学习_Rust。
但是，有关一些重要概念的简要介绍，请参阅[绕行：学习用于 Substrate 的 Rust](/tutorials/collectibles-workshop/detours/learn-rust/)。

## 代码编辑器

您需要一个编辑器来修改文件。
理想情况下，您应该选择一个集成开发环境 (IDE)，它提供语法高亮显示、自动代码完成和调试功能。
如果您没有首选的 IDE，Visual Studio Code 是一个不错的选择。

- [ ] [Visual Studio Code](https://code.visualstudio.com/download)

<!--其他常见的代码编辑器包括以下内容：

- [Sublime Text](https://www.sublimetext.com/)
- [Vim](https://www.vim.org/)
- [Atom](https://atom.io/)
-->

## 浏览器

要与区块链交互并在构建 Substrate 可收藏应用程序时测试您的工作，您需要一个可以连接到 Substrate 节点的基于浏览器的应用程序。
对于本研讨会，如果您有 Chrome 或基于 Chromium 的浏览器，则可以从[Polkadot/Substrate 门户](https://polkadot.js.org/apps/)连接到节点。

- [ ] [Google Chrome](https://www.google.com/chrome/)或基于 Chromium 的浏览器，例如[Brave](https://brave.com/download/)、[Microsoft Edge](https://www.microsoft.com/en-us/edge?ep=79&form=MA13KE&es=23)、[Opera](https://www.opera.com/download)或[Vivaldi](https://vivaldi.com/download/)。

如果您使用的是更严格的浏览器（例如 Firefox），您可能会发现 Polkadot/Substrate 门户与节点之间的连接由于安全或隐私原因而被阻止。

如果您的浏览器阻止连接，请克隆[polkadot-js/apps](https://github.com/polkadot-js/apps)存储库并在本地运行它。
有关在本地运行 Polkadot/Substrate 门户的帮助，请参阅[绕行：设置 Polkadot/Substrate 门户](/tutorials/collectibles-workshop/detours/set-up-app-locally/)。

## 前端库

要使 Substrate 可收藏物品成为面向用户的应用程序，您需要用于构建至少基本用户界面的工具。

- [ ] [Node.js](https://nodejs.org/en/download/)和包管理器`npm`
- [ ] [Yarn 包管理器](https://yarnpkg.com/)
- [ ] [TypeScript](https://www.typescriptlang.org/)
- [ ] 基本的 UI/UX 框架，例如 React、Vue、Bootstrap 或 Angular。

有关选择前端库的帮助，请参阅[绕行：选择前端工具](/tutorials/collectibles-workshop/detours/select-ui-tools/)。

## 节点模板

要完成本研讨会，您必须能够访问 Substrate 节点。
如果您正在设置本地开发环境，则可以按照[快速入门](/quickstart/)中的说明下载和编译开发者中心的[substrate-node-template](https://github.com/substrate-developer-hub/substrate-node-template/tags/)。

`substrate-node-template`存储库提供的是主 Substrate `node-template`二进制文件的快照，其中包含开始使用功能性节点和核心功能集所需的一切。
