---
title: Rust API
description: 突出显示直接从 Substrate 源代码生成的 Rust API 文档的入口点。
keywords:
  - Rust 文档
  - crate
  - 编程
---

Rust 文档是 Substrate 代码库的唯一事实来源。
但是，如果您不确定要查找什么（例如，特定的 crate、特性或数据结构），则可能难以导航或知道最佳的起点。

如 [架构和 Rust 库](/learn/architecture/) 中所述，Substrate 库使用命名约定来指示库是否是 Substrate 核心客户端 (`sc_*`)、FRAME 和运行时 (`frame_*` 和 `pallet_*`) 或 Substrate 原语 (`sp_*`) 的一部分。

![外部节点和运行时的核心节点库](/media/images/docs/libraries.png)

如果您没有特定的 crate，请使用以下链接作为探索核心 Rust 库的入口点。

## Substrate 核心客户端库

使用以下链接来探索作为 Substrate 核心客户端 (`sc_*`) 一部分的 Substrate 库。

- [`sc_authority_discovery`](https://paritytech.github.io/substrate/master/sc_authority_discovery/index.html)
- [`sc_block_builder`](https://paritytech.github.io/substrate/master/sc_block_builder/index.html)
- [`sc_chain_spec`](https://paritytech.github.io/substrate/master/sc_chain_spec/index.html)
- [`sc_cli`](https://paritytech.github.io/substrate/master/sc_cli/index.html)
- [`sc_client_api`](https://paritytech.github.io/substrate/master/sc_client_api/index.html)
- [`sc_client_db`](https://paritytech.github.io/substrate/master/sc_client_db/index.html)
- [`sc_consensus`](https://paritytech.github.io/substrate/master/sc_consensus/index.html)
- [`sc_network`](https://paritytech.github.io/substrate/master/sc_network/index.html)
- [`sc_rpc`](https://paritytech.github.io/substrate/master/sc_rpc/index.html)
- [`sc_service`](https://paritytech.github.io/substrate/master/sc_service/index.html)
- [`sc_state_db`](https://paritytech.github.io/substrate/master/sc_state_db/index.html)
- [`sc_transaction_pool`](https://paritytech.github.io/substrate/master/sc_transaction_pool/index.html)

## FRAME 库

使用以下链接来探索 Substrate 运行时中使用的核心 FRAME 库（`frame_*` 和 `pallet_*`）。

- [`frame_benchmarking`](https://paritytech.github.io/substrate/master/frame_benchmarking/index.html)
- [`frame_executive`](https://paritytech.github.io/substrate/master/frame_executive/index.html)
- [`frame_remote_externalities`](https://paritytech.github.io/substrate/master/frame_remote_externalities/index.html)
- [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html)
- [`frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html)
- [`pallet_assets`](https://paritytech.github.io/substrate/master/pallet_assets/index.html)
- [`pallet_balances`](https://paritytech.github.io/substrate/master/pallet_balances/index.html)
- [`pallet_collective`](https://paritytech.github.io/substrate/master/pallet_collective/index.html)
- [`pallet_identity`](https://paritytech.github.io/substrate/master/pallet_identity/index.html)
- [`pallet_membership`](https://paritytech.github.io/substrate/master/pallet_membership/index.html)
- [`pallet_proxy`](https://paritytech.github.io/substrate/master/pallet_proxy/index.html)

## Substrate 原语库

使用以下链接来探索 Substrate 原语库 (`sp_*`)。

- [`sp_api`](https://paritytech.github.io/substrate/master/sp_api/index.html)
- [`sp_blockchain`](https://paritytech.github.io/substrate/master/sp_blockchain/index.html)
- [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html)
- [`sp_io`](https://paritytech.github.io/substrate/master/sp_io/index.html)
- [`sp_runtime`](https://paritytech.github.io/substrate/master/sp_runtime/index.html)
- [`sp_state_machine`](https://paritytech.github.io/substrate/master/sp_state_machine/index.html)
- [`sp_storage`](https://paritytech.github.io/substrate/master/sp_storage/index.html)

## 其他库

使用以下链接来探索其他库。

- [`kitchensink_runtime`](https://paritytech.github.io/substrate/master/kitchensink_runtime)
- [`node_primitives`](https://paritytech.github.io/substrate/master/node_primitives/index.html)
- [`node_rpc`](https://paritytech.github.io/substrate/master/node_rpc/index.html)
- [`node_template`](https://paritytech.github.io/substrate/master/node_template/index.html)
- [`node_template_runtime`](https://paritytech.github.io/substrate/master/node_template_runtime/index.html)
