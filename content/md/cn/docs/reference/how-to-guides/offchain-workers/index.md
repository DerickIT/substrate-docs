---
title: 链下工作者
description: 说明如何使用链下工作者的快速参考指南。
keywords:
---

<div class="warning">
	 请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_offchain_workers/index.html">Rust文档</a> 获取有关此主题的最新文档。
</div>

“链下工作者”类别中的“操作指南”说明了链下操作的常见用例。

- [进行链下HTTP请求](/reference/how-to-guides/offchain-workers/offchain-http-requests/)
- [链下本地存储](/reference/how-to-guides/offchain-workers/offchain-local-storage/)
- [链下索引](/reference/how-to-guides/offchain-workers/offchain-indexing/)

需要注意的是，链下存储与链上存储是分开的。
您无法将链下工作者收集或处理的数据直接发送到链上存储。
要存储链下工作者收集或处理的任何数据（即修改链的状态），您必须启用链下工作者来发送修改链上存储系统的交易。
有关如何准备链下工作者以发送交易的示例，请参阅[添加链下工作者](/tutorials/build-application-logic/add-offchain-workers/)。
