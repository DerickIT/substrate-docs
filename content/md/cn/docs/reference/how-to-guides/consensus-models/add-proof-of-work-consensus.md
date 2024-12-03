---
title: 配置链以使用工作量证明
description: 
keywords:
  - 节点
  - 客户端
  - 共识
  - 工作量证明
---

`basic-pow` 节点演示了如何将自定义共识引擎添加到基于 Substrate 的区块链中。
在此示例中，节点使用最小的工作量证明共识引擎来就区块链达成一致。
本指南介绍了使用共识引擎的一些核心原则。

## 使用案例

- 启动使用工作量证明共识引擎的链。
- 将链从基于权限的共识引擎升级到工作量证明共识引擎。

## 步骤预览

1. 定义一个使用工作量证明共识的全节点。
1. 为数据提供者创建导入队列。
1. 定义 `proposer` 和 `worker` 函数。
1. 定义轻客户端服务。

## 使用 sc_consensus_pow 和 sc_service 定义全节点

在 `src/service.rs` 中，创建一个名为 `new_full` 的函数，该函数定义了 [`PartialComponents`](https://paritytech.github.io/substrate/master/sc_service/struct.PartialComponents.html) 和 [`PowBlockImport`](https://paritytech.github.io/substrate/master/sc_consensus_pow/struct.PowBlockImport.html)：

```rust
let pow_block_import = sc_consensus_pow::PowBlockImport::new(
  client.clone(),
  client.clone(),
  sha3pow::MinimalSha3Algorithm,
  0,                              // check inherents starting at block 0
  select_chain.clone(),
  inherent_data_providers.clone(),
  can_author_with,
);

let import_queue = sc_consensus_pow::import_queue(
  Box::new(pow_block_import.clone()),
  None,
  sha3pow::MinimalSha3Algorithm,  // provide it with references to our client
  inherent_data_providers.clone(),
  &task_manager.spawn_handle(),
  config.prometheus_registry(),
)?;
```

## 创建导入队列

通过在定义 POW 系统提供者的函数中使用 [`InherentDataProviders`](https://paritytech.github.io/substrate/master/sc_consensus_aura/struct.InherentDataProvider.html) 来定义节点的固有数据：

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

let (_worker, worker_task) = sc_consensus_pow::start_mining_worker(
    Box::new(pow_block_import),
    client,
    select_chain,
    MinimalSha3Algorithm,
    proposer,
    network.clone(),
    None,
    inherent_data_providers,
    // time to wait for a new block before starting to mine a new one
    Duration::from_secs(10),
    // how long to take to actually build the block (i.e. executing extrinsics)
    Duration::from_secs(10),
    can_author_with,
);
```

让任务管理器生成它：

```rust
task_manager
    .spawn_essential_handle()
    .spawn_blocking("pow", worker_task);
```

## 示例

- [基本 POW 示例节点](https://github.com/substrate-developer-hub/substrate-how-to-guides/tree/main/example-code/consensus-nodes/POW)

## 资源

- [部分组件](https://paritytech.github.io/substrate/master/sc_service/struct.PartialComponents.html)
- [Pow 块导入](https://paritytech.github.io/substrate/master/sc_consensus_pow/struct.PowBlockImport.html)
- [创建固有数据提供者](https://paritytech.github.io/substrate/master/sp_inherents/trait.CreateInherentDataProviders.html)
- [Pow 算法](https://paritytech.github.io/substrate/master/sc_consensus_pow/trait.PowAlgorithm.html)
