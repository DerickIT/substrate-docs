---
title: 使用 REST 端点获取链数据
description:
keywords:
  - 节点
  - 客户端
  - 工具
---

本操作指南说明了如何使用 `sidecar` 服务提供的 REST 端点与 Substrate 区块链节点进行交互。

要查找已完成拍卖的获胜者，我们需要知道拍卖结束的区块号。由于 [Sidecar](https://github.com/paritytech/substrate-api-sidecar) 是一个无状态 API，并且拍卖信息存储在拍卖的最终区块中，因此一旦拍卖结束，我们就需要区块号来进行历史查询以检索存储在其中的事件和数据。

## 目标

使用 `sidecar` 查找已完成 Parachain 拍卖的获胜者

## 用例

使用 REST 服务与 Substrate 区块链节点进行交互。

## 步骤

### 1. 利用 /experimental/paras/auctions/current 端点

我们将在数据库中跟踪和存储 `finishEnd`、`auctionIndex` 和 `leasePeriods`：

- `finishEnd`：这是拍卖的最后一个区块。存储它允许您查询拍卖结束的区块。从该区块中，您可以提取租赁获胜相关的事件。（要查询区块：GET /blocks/{finishEnd}。）

- `auctionIndex`：拍卖的唯一标识符。

- `leasePeriods`：可用于特定 `auctionIndex` 的可用租赁期索引。

### 2. 使用 Sidecar 查找拍卖获胜者

通过存储 `finishEnd` 块并查看其中的 `Leased` 事件，我们可以看到拍卖获胜者是谁以及他们获得了哪些租赁期。

根据需要格式化数据，例如：

```js
auctionIndex: {
    leasePeriods: [
        "11", "12", "13", "14"
    ],
    finishEnd: '200'
}
```

### 3. 查询 /blocks/:blockId 端点

此步骤查询 `finishEnd` 字段中指定的区块高度的所有区块，并检索 `on_initialize` 中的所有事件。示例响应如下：

```js
{
    authorId: ....,
    extrinsics:....
    ...
    on_initialize: {
        events: [
            {
                "method": {
                    "pallet": "slots",
                    "method": "Leased"
                },
                "data": [
                    '1000', // ParaId
                    '5HpG9w8EBLe5XCrbczpwq5TSXvedjrBGCwqxK1iQ7qUsSWFc', // AccountId
                    '1', // LeasePeriod（租赁期的开始）
                    '4', // LeasePeriod（租赁期的计数）
                    '10000', // Balance（保留的额外余额）
                    '1000000000', // Balance（总余额）
                ]
            },
            {
                "method": {
                    "pallet": "auctions",
                    "method": "AuctionClosed"
                },
                "data": [
                    ...
                ]
            }
        ]
    }
}
```

### 4. 比较您的数据

现在您已经拥有了在该拍卖中赢得时隙的所有 `paraIds`，您可以将其与与 `auctionIndex` 相关的数据进行比较。
将活动拍卖期间可用的 `leasePeriods` 与已赢得并在 `Leased` 事件中标明的 `leasePeriods` 进行比较（如果有多个获胜者，则可能有多个）将为您提供该拍卖的所有获胜者。

## 示例

- 更多指南 [此处](https://github.com/paritytech/substrate-api-sidecar/tree/master/guides)

## 资源

- [Sidecar 文档](https://github.com/paritytech/substrate-api-sidecar)
- 可用的 [Substrate Sidecar API 端点](https://paritytech.github.io/substrate-api-sidecar/dist/)
