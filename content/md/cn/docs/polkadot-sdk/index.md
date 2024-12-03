---
title: Polkadot SDK
description:
keywords:
  - polkadot-sdk
---


## Substrate 到 Polkadot SDK

在 [2023 年 5 月](https://forum.polkadot.network/t/psa-parity-is-currently-working-on-merging-the-polkadot-stack-repositories-into-one-single-repository/2883)，
Parity Technologies 开始将包含
[`substrate`](https://github.com/paritytech/substrate)、
[`polkadot`](https://github.com/paritytech/polkadot) 和
[`cumulus`](https://github.com/paritytech/cumulus) 的三个存储库（以前在开发和品牌方面是独立的）合并到一个名为
[`polkadot-sdk`](https://github.com/paritytech/polkadot-sdk) 的新单一存储库中。 因此，[波卡和 Kusama 中继链及其系统链的运行时](https://github.com/polkadot-fellows/runtimes) 被转移到由 [波卡研究员](polkadot-fellows.github.io/dashboard/) 维护。

这种转变催生了一种新的愿景，在这种愿景中，这些工具被视为是 `polkadot-sdk` 的一部分，同时仍然独立地发挥作用。

在此过渡中最受影响的是 **Substrate**，并且此影响的一部分是此网站的逐步弃用。 此网站上的内容不再维护，并将逐步迁移到新的位置。 请将 [以下资源](#alternative-resources) 视为主要信息来源。

> 您仍然可以在导航栏中访问此网站的旧内容，但请注意某些内容可能已过时。

请注意，**这不会影响 Substrate 作为框架的开发**。 Substrate 作为框架，作为 `polkadot-sdk` 的一部分，仍在全力维护，并保留其用于构建 [独立区块链](https://github.com/paritytech/polkadot-sdk-solochain-template) 或 [波卡驱动的平行链](https://github.com/paritytech/polkadot-sdk-parachain-template) 的全部能力。

## 替代资源

在下面，您可以找到许多替代资源来了解更多关于 Polkadot SDK 的信息：

- [波卡 Wiki](https://wiki.polkadot.network/docs/build-guide)
- [波卡开发者](https://github.com/polkadot-developers/)
- [波卡区块链学院](https://polkadot.com/blockchain-academy)
- [Polkadot SDK Rust 文档](https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/index.html)
- Papermoon 编写的波卡生态系统文档（正在进行中，请查看 [此处](https://forum.polkadot.network/t/decentralized-futures-papermoon-first-updates/9265) 的进度）
