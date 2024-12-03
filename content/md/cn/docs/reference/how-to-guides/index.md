---
title: 操作指南快速参考
description:
keywords:
---

Substrate _操作指南_ 快速参考指南提供了实现特定目标的说明。
每份指南都解释了如何执行特定任务，并假设您已经熟悉 Substrate 和 Rust 编程。

## 基础知识

请参阅以下指南，了解运行时开发中的常见模式。

- [导入 pallet](/reference/how-to-guides/basics/import-a-pallet/)
- [配置运行时常量](/reference/how-to-guides/basics/configure-runtime-constants/)
- [配置创世状态](/reference/how-to-guides/basics/configure-genesis-state)
- [自定义链规范](/reference/how-to-guides/basics/customize-a-chain-specification)
- [使用辅助函数](/reference/how-to-guides/basics/use-helper-functions)

## Pallet 设计

请参阅以下指南，了解使用 FRAME 构建 pallet 的最佳实践。

- [实现可锁定货币](/reference/how-to-guides/pallet-design/implement-lockable-currency/)
- [整合随机性](/reference/how-to-guides/pallet-design/incorporate-randomness/)
- [配置众筹](/reference/how-to-guides/pallet-design/configure-crowdfunding/)
- [创建存储结构体 (struct)](/reference/how-to-guides/pallet-design/create-a-storage-structure/)
- [使用紧密 pallet 耦合](/reference/how-to-guides/pallet-design/use-tight-coupling/)
- [使用松散 pallet 耦合](/reference/how-to-guides/pallet-design/use-loose-coupling/)

## 权重

请参阅以下指南，了解有关基准测试和权重配置的帮助。

- [计算费用](/reference/how-to-guides/weights/calculate-fees/)
- [添加基准测试](/reference/how-to-guides/weights/add-benchmarks/)

## 测试

请参阅以下指南，了解有关测试 pallet 和运行时逻辑的帮助。

- [设置基本测试](/reference/how-to-guides/testing/set-up-basic-tests/)
- [测试转移函数](/reference/how-to-guides/testing/test-a-transfer-function/)

## 存储迁移

请参阅以下指南，了解有关存储迁移的帮助。

- [基本存储迁移](/reference/how-to-guides/storage-migrations/basic-storage-migration/)
- [触发迁移](/reference/how-to-guides/storage-migrations/trigger-migration/)

## 共识模型

请参阅以下指南，了解如何在运行时中实现共识机制。

- [创建混合节点](/reference/how-to-guides/consensus-models/create-a-hybrid-node/)
- [添加工作量证明共识](/reference/how-to-guides/consensus-models/add-proof-of-work-consensus/)

## Parachain

请参阅以下指南，了解有关使用 Substrate Parachain 的帮助。

- [转换单链](/reference/how-to-guides/parachains/convert-a-solo-chain/)
- [连接到中继链](/reference/how-to-guides/parachains/connect-to-a-relay-chain/)
- [选择 Collator](/reference/how-to-guides/parachains/select-collators/)
- [准备启动](/reference/how-to-guides/parachains/prepare-to-launch/)
- [升级 Parachain](/reference/how-to-guides/parachains/upgrade-a-parachain/)
- [拍卖和众筹](/reference/how-to-guides/parachains/auctions-and-crowdloans/)
- [添加 HRMP 通道](/reference/how-to-guides/parachains/add-hrmp-channels/)

## 工具

请参阅以下指南，了解有助于您在生产环境中管理 Substrate 链的附加工具。

- [使用 try-runtime](/reference/how-to-guides/tools/use-try-runtime/)
- [为链创建 txwrapper](/reference/how-to-guides/tools/create-a-txwrapper/)
- [使用 REST 端点获取链数据](/reference/how-to-guides/tools/use-sidecar/)
- [验证 Wasm 二进制文件](/reference/how-to-guides/tools/verify-wasm/)

## 链下工作者

请参阅以下指南，了解有关使用链下数据的帮助。

- [发出链下 HTTP 请求](/reference/how-to-guides/offchain-workers/offchain-http-requests/)
- [链下本地存储](/reference/how-to-guides/offchain-workers/offchain-local-storage/)
- [链下索引](/reference/how-to-guides/offchain-workers/offchain-indexing/)

<!--
- [计算权重](/reference/how-to-guides/basics/calc-weights/)
- [铸造基本代币](/reference/how-to-guides/basics/mint-basic-tokens/)
- [添加 contracts pallet](/reference/how-to-guides/pallet-design/add-contracts-pallet/)
-->
