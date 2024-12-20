---
title: 准备部署
description: 
keywords:
  - 平行链
  - 中继链
  - 节点类型
  - 部署目标
---

为了准备部署，您需要更仔细地查看您的网络架构，包括如何以及在何处部署不同的节点、您正在部署的基础设施类型以及您将用于管理节点操作的命令行选项。
本节中的主题重点介绍了从本地开发和测试环境迁移到生产环境的一些关键注意事项。

## 节点角色和职责

正如[网络和节点](/learn/node-and-network-types/)中所述，您可以使用 Substrate 构建可以作为单链或平行链、私有网络或自定义中继链部署的区块链。
您还了解到，节点可以根据您的链和项目要求承担不同的角色。

在准备部署时，务必牢记不同类型的节点的使用方式，以确保它们配置了执行其职责所需的适当硬件和软件。
下表总结了最常见的节点类型和职责：

| 节点类型      | 它做什么                                                                                                                                                                                                               |
| :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 验证器节点 | 通过质押 DOT 来保护中继链，处理来自平行链协调器节点的验证证明，并与其他验证器一起对共识进行投票。                                                                  |
| 协调器节点  | 通过收集平行链交易并为验证器生成状态转换证明来维护平行链。                                                                                                       |
| 引导节点      | 提供一个静态地址和点对点 (`libp2p`) 公钥，用于将节点引导到网络的分布式哈希表上并查找对等节点。                                                        |
| RPC 节点       | 通过 HTTP 或 WebSocket 端口公开中继链或平行链的 RPC 接口，以便用户可以读取区块链状态并提交交易。通常在负载均衡器后面有多个 RPC 节点。 |
| 存档节点   | 从创世区块开始维护所有区块，并为每个区块提供完整的可用状态。                                                                                                                        |
| 完整节点      | 与中继链或平行链同步以存储最新的区块状态和旧区块的区块头。                                                                                                    |

### 角色特定的命令行选项

用于启动节点的命令行设置通常反映您正在启动的节点类型。
例如，如果您正在启动验证器节点，则应始终至少指定 `--validator` 命令行选项以启用区块验证。
您还应确保使用命令行或通过调用 `author_rotateKeys` RPC 方法来注入节点密钥。

对于协调器节点，您应始终至少指定 `--collator` 命令行选项以启用平行链整理。

引导节点使用静态密钥文件来确保点对点网络的公共地址始终相同。
您应该将私有节点密钥存储在文件中，并使用 `--node-key-file` 命令行选项指定文件路径。

对于 RPC 节点，您应该指定以下命令行选项以允许最多 5000 个公共 RPC 或 WebSocket 连接：

`--unsafe-ws-external`
`--rpc-methods Safe`
`--rpc-cors ‘*’ `
`--ws-max-connections 5000`

如果您希望节点成为存档节点，则必须指定 `-–pruning=archive` 命令行选项以防止节点丢弃任何区块状态。

## 平行链特定的语法

对于平行链节点，您必须指定两组命令行选项。
第一组命令行选项适用于平行链。
第二组命令行选项适用于与平行链并行运行的本地中继链。
例如：

```
./statemine $PARACHAIN_OPTIONS -- $RELAYCHAIN_OPTIONS
```

例如，以下命令说明了如何启动 Statemine 协调器节点：

```bash
./statemine  --chain statemine --in-peers 25 --out-peers 25 --db-cache 512 --pruning=1000 --unsafe-pruning -- --chain kusama -db-cache 512 --pruning=1000 --wasm-execution Compiled
```

## 常见部署目标

您可以通过多种不同的方式部署定义网络拓扑的节点，不同的部署场景具有不同的硬件和软件要求。
下表提供了常见部署目标的非详尽列表：

| 类型               | 描述                                                                                                                                                                                                                     |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 物理服务器    | 物理硬件通常部署为位于数据中心的机架式服务器。                                                                                                                                      |
| 虚拟机    | 虚拟硬件通常部署为托管在云提供商平台或自托管现场的虚拟映像。                                                                                                           |
| Kubernetes 集群 | Kubernetes 是一个容器编排引擎，可用于托管您的区块链实例。仅当您已经拥有 Kubernetes 的先前经验（尤其是在生产环境中）时，才建议使用此选项。 |
| 本地容器    | 本地容器引擎（例如 `containerd`、Docker 和 Pod Manager）提供比 Kubernetes 更简单的容器化服务，可用于创建和部署节点映像。                                               |
