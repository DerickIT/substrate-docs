---
title: 创建混合节点
description: 
keywords:
  - 共识
  - 混合节点
  - 混合共识
---

本指南演示了如何创建混合节点，该节点结合了两种不同的共识引擎，例如 Aura 和 BABE。
混合节点允许您在单个节点中利用两种共识引擎的优势，例如，Aura 的高吞吐量和 BABE 的安全性。

## 使用案例

- 创建一个使用 Aura 和 BABE 的混合共识引擎的链。
- 将链从基于权限的共识引擎升级到混合共识引擎。

## 步骤预览

1. 定义一个使用 Aura 和 BABE 的混合节点。
1. 为数据提供者创建导入队列。
1. 定义 `proposer` 和 `worker` 函数。
1. 定义轻客户端服务。

## 使用 sc_consensus_aura 和 sc_consensus_babe 定义混合节点

在 `src/service.rs` 中，创建一个名为 `new_full` 的函数，该函数定义了 [`PartialComponents`](https://paritytech.github.io/substrate/master/sc_service/struct.PartialComponents.html) 和 [`AuraBlockImport`](https://paritytech.github.io/substrate/master/sc_consensus_aura/struct.AuraBlockImport.html)：

```rust
let aura_block_import = sc_consensus_aura::AuraBlockImport::new(
  client.clone(),
  client.clone(),
  select_chain.clone(),
  inherent_data_providers.clone(),
  can_author_with,
);

let babe_block_import = sc_consensus_babe::BabeBlockImport::new(
  client.clone(),
  client.clone(),
  select_chain.clone(),
  inherent_data_providers.clone(),
  can_author_with,
);

let import_queue = sc_consensus_aura::import_queue(
  Box::new(aura_block_import.clone()),
  None,
  inherent_data_providers.clone(),
  &task_manager.spawn_handle(),
  config.prometheus_registry(),
)?;

let babe_import_queue = sc_consensus_babe::import_queue(
  Box::new(babe_block_import.clone()),
  None,
  inherent_data_providers.clone(),
  &task_manager.spawn_handle(),
  config.prometheus_registry(),
)?;
```

## 创建导入队列

通过在定义 Aura 和 BABE 系统提供者的函数中使用 [`InherentDataProviders`](https://paritytech.github.io/substrate/master/sc_consensus_aura/struct.InherentDataProvider.html) 来定义节点的固有数据：

```rust
pub fn build_inherent_data_providers() -> Result<InherentDataProviders, ServiceError> {
  let providers = InherentDataProviders::new();

  providers
    .register_provider(sp_timestamp::InherentDataProvider)
    .map_err(Into::into)
    .map_err(sp_consensus::error::Error::InherentData)?;

    Ok(providers)
}
```

## 定义 proposer 和 worker

在 `new_full` 函数中，定义 `proposer`：

```rust
let proposer = sc_basic_authorship::ProposerFactory::new(
    task_manager.spawn_handle(),
    client.clone(),
    transaction_pool,
    prometheus_registry.as_ref(),
);

let (_worker, worker_task) = sc_consensus_aura::start_aura_worker(
    aura_block_import,
    client,
    select_chain,
    proposer,
    network.clone(),
    None,
    inherent_data_providers,
    can_author_with,
);

let (_worker, worker_task) = sc_consensus_babe::start_babe_worker(
    babe_block_import,
    client,
    select_chain,
    proposer,
    network.clone(),
    None,
    inherent_data_providers,
    can_author_with,
);
```

让任务管理器生成它：

```rust
task_manager
    .spawn_essential_handle()
    .spawn_blocking("aura", worker_task);

task_manager
    .spawn_essential_handle()
    .spawn_blocking("babe", worker_task);
```

## 示例

- [混合共识示例节点](https://github.com/substrate-developer-hub/substrate-how-to-guides/tree/main/example-code/consensus-nodes/hybrid)

## 资源

- [部分组件](https://paritytech.github.io/substrate/master/sc_service/struct.PartialComponents.html)
- [Aura 块导入](https://paritytech.github.io/substrate/master/sc_consensus_aura/struct.AuraBlockImport.html)
- [BABE 块导入](https://paritytech.github.io/substrate/master/sc_consensus_babe/struct.BabeBlockImport.html)
- [创建固有数据提供者](https://paritytech.github.io/substrate/master/sp_inherents/trait.CreateInherentDataProviders.html)
- [Aura 算法](https://paritytech.github.io/substrate/master/sc_consensus_aura/trait.AuraAlgorithm.html)
- [BABE 算法](https://paritytech.github.io/substrate/master/sc_consensus_babe/trait.BabeAlgorithm.html)
