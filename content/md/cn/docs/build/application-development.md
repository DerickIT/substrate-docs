---
title: 应用程序开发
description: 本文探讨了如何在构建应用程序接口时使用 Substrate 元数据和 RPC 库。
keywords:
  - 前端
  - 应用程序
  - dApp
---

作为区块链开发者，你可能不会直接参与前端应用程序的构建。
但是，大多数在区块链上运行的应用程序都需要某种形式的前端或面向用户的客户端，以允许用户或其他程序访问和修改区块链存储的数据。
例如，你可能会开发一个基于浏览器的、移动的或桌面的应用程序，允许用户提交交易、发布文章、查看他们的资产或跟踪之前的活动。
该应用程序的后端在你的区块链的运行时逻辑中配置，但正是前端客户端使运行时功能对你的用户可用。

为了使你的自定义链对其他人有用，你需要提供某种类型的客户端应用程序，允许用户查看、交互或更新区块链跟踪的信息。
在本文中，你将学习如何公开有关运行时信息，以便客户端应用程序可以使用它，查看公开信息的示例，并探索使用此信息的工具和库。

## 将运行时信息公开为元数据

要与 Substrate 节点或存储在区块链中的信息进行交互，你需要知道如何连接到链以及如何访问运行时向外部世界公开的功能。
通常，这种交互涉及远程过程调用以请求你感兴趣的检索或更新的信息。
但是，作为应用程序开发者，你通常需要了解更多关于运行时逻辑的信息，包括以下详细信息：

- 应用程序连接到的运行时版本。
- 运行时支持的应用程序编程接口。
- 为特定运行时实现的模块。
- 为特定运行时定义的所有函数及其类型签名。
- 为特定运行时定义的所有自定义类型。
- 运行时公开供用户设置的所有参数。

由于 Substrate 是模块化的，并提供了一个可组合的区块链构建框架，因此没有预定义的属性模式。
相反，每个运行时都配置有自己的一组属性，并且这些属性（包括函数和类型）会随着升级而随着时间的推移而改变。
为了捕获特定于运行时的所有信息，Substrate 使你能够生成运行时**元数据**模式。
运行时的元数据描述了为特定版本的运行时定义的所有模块和类型。
对于每个模块，元数据都包含有关其存储项、函数、事件、错误和常量的信息。
元数据还包括运行时中包含的任何自定义类型的类型定义。

因为它提供了运行时的完整清单，所以元数据是使客户端应用程序能够与节点交互、解析响应和格式化消息有效负载的关键。

## 生成元数据

为了最大限度地减少通过网络传输数据所需的带宽，元数据模式使用 [SCALE 编解码库](/reference/scale-codec/) 进行编码。
当你使用 [`scale-info`](https://docs.rs/scale-info/latest/scale_info/) crate 编译节点时，此编码会自动为你完成。

在高层次上，生成元数据涉及以下步骤：

- 运行时逻辑中的模块公开所有需要在元数据中编码的可调用函数、类型、参数和文档。

- `scale-info` crate 收集运行时中模块的类型信息，并构建一个特定运行时中存在的模块的注册表以及注册表中每个模块的相关类型。
  类型信息足够详细，可以对每种类型进行编码和解码。

- [`frame-metadata`](https://github.com/paritytech/frame-metadata) crate 基于 `scale-info` crate 提供的注册表来描述运行时的结构。

- Substrate 节点提供 RPC 方法 `state_getMetadata` 以返回当前运行时中所有类型的完整描述，作为 SCALE 编码字节的十六进制编码向量。

下图简要概述了编译运行时逻辑时如何生成元数据，然后通过使用 RPC 请求连接到节点来访问元数据。

![编译运行时生成元数据](/media/images/docs/metadata.png)

## 获取运行时的元数据

你可以通过多种方式获取运行时的元数据。
例如，你可以执行以下任何操作：

- 使用 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/rpc) 连接到区块链或节点，并选择 **state** 端点和 **getMetadata** 方法以 JSON 格式返回元数据。
- 使用命令行 `polkadot-js-api` 调用 `state_getMetadata` RPC 方法以将元数据作为 SCALE 编码字节的十六进制编码向量返回。
- 使用 `subxt metadata` 命令下载 JSON、十六进制或原始字节中的元数据。
- 使用 `sidecar` API 和 `/runtime/metadata` 端点连接到节点并检索 JSON 格式的元数据。

元数据提供的类型信息使应用程序能够与具有不同运行时版本的节点以及公开不同调用、事件、类型和存储项的链进行通信。
元数据还允许库生成与给定 Substrate 节点通信所需的大部分代码，使像 `subxt` 这样的库能够生成特定于目标链的前端接口。

## 客户端应用程序和元数据

客户端应用程序使用元数据与节点交互、解析响应和格式化发送到节点的消息有效负载。
要使用元数据，客户端应用程序必须使用 [SCALE 编解码库](/reference/scale-codec/) 来编码和解码 RPC 有效负载。
因为元数据公开了预期如何解码每种类型，所以应用程序可以在没有手动编码和解码的情况下发送、检索和处理应用程序信息。

## 元数据格式

尽管可以使用 `frame-metadata` 和 [`parity-scale-codec`](https://github.com/paritytech/parity-scale-codec) 库解码 SCALE 编码字节，但还有其他工具（例如 `subxt` 和 Polkadot-JS API）可以将原始数据转换为人类可读的 JSON 格式。

`state_getMetadata` RPC 调用返回的元数据中包含的类型和类型定义取决于运行时的元数据版本。
通常，元数据包含以下信息：

- 一个常量，用于标识包含元数据的文件。
- 运行时中使用的元数据格式的版本。
- 运行时中使用的所有类型以及由 `scale-info` crate 生成的类型定义。
- 运行时中包含的所有模块的模块信息，其顺序与在 `construct_runtime` 宏中定义的顺序相同。

以下示例说明了元数据的简化和带注释的部分，该元数据已解码并转换为 JSON：

```json
[
  1635018093,
  {
    "V14": {
      "types": {
        "types": [
          {
            // 类型的索引
          }
        ]
      },
      "pallets": [
        {
          // 模块的索引以及每个模块中每个模块公开的元数据
        }
      ],
      "extrinsic": {
        "ty": 126, // 定义外部函数格式的类型索引标识符
        "version": 4, // 用于编码和解码外部函数的事务版本
        "signed_extensions": [
          {
            // 已签名扩展的索引
          }
        ]
      },
      "ty": 141 // 系统模块的类型 ID
    }
  }
]
```

常量 `1635018093` 是一个魔术数字，用于标识该文件为元数据文件。
其余元数据已分为 `types`、`pallets` 和 `extrinsic` 部分。
`types` 部分包含类型的索引，以及每个类型的类型签名的信息。
`pallets` 部分包含有关运行时中每个模块的信息。
`extrinsic` 部分描述了运行时正在使用的类型标识符和事务格式版本。
不同的外部函数版本可能具有不同的格式，尤其是在考虑 [签名事务](/learn/transaction-types) 时。

### 模块

以下是 `pallets` 数组中单个元素的简化和带注释的示例：

```json
{
  "name": "Sudo",        // 模块名称
  "storage": {           // 模块的存储信息
      "prefix": "Sudo",  // 模块存储项的数据库前缀
      "entries": [
        {
          "name": "Key",
          "modifier": "Optional",
          "ty": {
             "Plain": 0
          },
          "default": [
             0
          ],
          "docs": [
             "Sudo 密钥的 `AccountId`。"
          ]
        }
      ]
  },
  "calls": {       // 模块调用类型
      "ty": 117    // types 部分中的类型标识符
  },
  "event": {       // 模块事件类型
      "ty": 42     // types 部分中的类型标识符
  },
  "constants": [], // 模块常量
  "error": {       // 模块错误类型
      "ty": 124    // types 部分中的类型标识符
          },
  "index": 8       // 运行时中模块的索引标识符
},
```

每个元素都包含它所代表的模块的名称以及有关其存储、调用、事件和错误的信息。
你可以通过查看类型索引标识符来查找有关调用、事件和错误定义的详细信息。
每个项目的类型索引标识符是用于访问该项目类型信息的 `u32` 整数。
例如，Sudo 模块中调用的类型索引标识符是 117。
如果你查看 `types` 部分中该类型标识符的信息，它将提供有关可用调用（包括每个调用的文档）的信息。

例如，以下是 Sudo 模块调用的简化摘录：

```json
    {
      "id": 117,
      "type": {
          "path": [
              "pallet_sudo",
              "pallet",
              "Call"
          ],
          "params": [
            {
              "name": "T",
              "type": null
            }
          ],
          "def": {
              "variant": {
                  "variants": [
                    {
                      "name": "sudo",
                      "fields": [
                        {
                          "name": "call",
                          "type": 114,
                          "typeName": "Box<<T as Config>::RuntimeCall>"
                        }
                  ],
                      "index": 0,
                      "docs": [
                        "验证 sudo 密钥并使用 `Root` 原点调度函数调用。",
                      ]
                    },
                    {
                      "name": "sudo_unchecked_weight",
                      "fields": [
                        {
                          "name": "call",
                          "type": 114,
                          "typeName": "Box<<T as Config>::RuntimeCall>"
                        },
                        {
                          "name": "weight",
                          "type": 8,
                          "typeName": "Weight"
                        }
                      ],
                      "index": 1,
                      "docs": [
                        "验证 sudo 密钥并使用 `Root` 原点调度函数调用。",
                      ]
                    },
                    {
                      "name": "set_key",
                      "fields": [
                        {
                          "name": "new",
                          "type": 103,
                          "typeName": "AccountIdLookupOf<T>"
                        }
                      ],
                      "index": 2,
                      "docs": [
                        "验证当前 sudo 密钥并将给定的 AccountId (`new`) 设置为新的 sudo",
                      ]
                    },
                    {
                      "name": "sudo_as",
                      "fields": [
                        {
                          "name": "who",
                          "type": 103,
                          "typeName": "AccountIdLookupOf<T>"
                        },
                        {
                          "name": "call",
                          "type": 114,
                          "typeName": "Box<<T as Config>::RuntimeCall>"
                        }
                      ],
                      "index": 3,
                      "docs": [
                        "验证 sudo 密钥并使用来自给定帐户的 `Signed` 原点调度函数调用。",
                      ]
                    }
                  ]
                }
              },
            },
```

对于每个字段，你可以访问以下内容的类型信息和元数据：

- 存储元数据提供使应用程序能够获取特定存储项的信息所需的信息。
- 调用元数据包括由 `#[pallet]` 宏定义的运行时调用的信息，包括调用名称、参数和文档。
- 事件元数据提供由 `#[pallet::event]` 宏生成的元数据，包括每个模块事件的名称、参数和文档。
- 常量元数据提供由 `#[pallet::constant]` 宏生成的元数据，包括常量的名称、类型和十六进制编码值。
- 错误元数据提供由 `#[pallet::error]` 宏生成的元数据，包括每个模块错误的名称和文档。

你应该注意，类型标识符会不时发生变化。
你应该避免在应用程序中依赖类型标识符。

### 外部函数

外部函数元数据由运行时生成，并提供有关事务格式的有用信息。
解码后，元数据包含事务版本和已签名扩展的列表。
例如：

```json
    "extrinsic": {
        "ty": 126,
        "version": 4,
        "signed_extensions": [
          {
            "identifier": "CheckNonZeroSender",
            "ty": 132,
            "additional_signed": 41
          },
          {
            "identifier": "CheckSpecVersion",
            "ty": 133,
            "additional_signed": 4
          },
          {
            "identifier": "CheckTxVersion",
            "ty": 134,
            "additional_signed": 4
          },
          {
            "identifier": "CheckGenesis",
            "ty": 135,
            "additional_signed": 11
          },
          {
            "identifier": "CheckMortality",
            "ty": 136,
            "additional_signed": 11
          },
          {
            "identifier": "CheckNonce",
            "ty": 138,
            "additional_signed": 41
          },
          {
            "identifier": "CheckWeight",
            "ty": 139,
            "additional_signed": 41
          },
          {
            "identifier": "ChargeTransactionPayment",
            "ty": 140,
            "additional_signed": 41
          }
        ]
      },
      "ty": 141
    }
  }
]
```

类型系统是复合的。
每个类型标识符都包含对特定类型或提供有关关联基元类型信息的另一个类型标识符的引用。
例如，你可以编码 `BitVec<Order, Store>` 类型，但要正确解码它，你必须知道用于 `Order` 和 `Store` 类型的类型。
要查找 `Order` 和 `Store` 的类型信息，你可以使用解码的 JSON 中的路径来查找它们的类型标识符。

## RPC API

Substrate 附带以下 API 来与节点交互：

- [`AuthorApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/author/trait.AuthorApiServer.html)：一个 API，用于向完整节点发出调用，包括创作外部函数和验证会话密钥。
- [`ChainApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/chain/trait.ChainApiServer.html)：一个 API，用于检索区块头和最终性信息。
- [`OffchainApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/offchain/trait.OffchainApiServer.html)：一个 API，用于为离链工作程序发出 RPC 调用。
- [`StateApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/state/trait.StateApiServer.html)：一个 API，用于查询有关链上状态的信息，例如运行时版本、存储项和证明。
- [`SystemApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/system/trait.SystemApiServer.html)：一个 API，用于检索有关网络状态的信息，例如已连接的对等点和节点角色。

## 连接到节点

应用程序通常通过打开的 HTTP 或 WebSocket 端口使用 JSON-RPC 方法连接到 Substrate 节点。
大多数应用程序使用 WebSocket 端口，因为单个连接可用于节点之间的大量消息。
使用 HTTP 连接，应用程序一次只能发送和接收一个消息的响应。
使用 HTTP 连接到节点的最常见原因是如果你想使用离链工作程序来获取数据。
有关使用离链工作程序的更多信息，请参阅 [离链操作](/learn/offchain-operations)。

作为使用 RPC 连接的替代方法，你可以使用 [Substrate Connect](https://substrate.io/developers/substrate-connect/) 和轻客户端节点来连接到基于 Substrate 的区块链。
Substrate Connect 在浏览器中运行，并允许应用程序创建自己的轻客户端节点并直接连接到公开的 JSON-RPC 端点。
集成 Substrate Connect 的应用程序依赖于浏览器本地内存来与轻客户端节点建立连接。

## 构建前端应用程序

以下库使用 [JSON-RPC API](https://github.com/paritytech/jsonrpsee) 使应用程序能够与 Substrate 节点交互：

| 名称                                                             | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 语言       |
| :--------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- |
| [Chain API](https://github.com/paritytech/capi)                  | 提供了一个 TypeScript 工具包，用于创建与基于 Substrate 的链的交互。该工具包包括 FRAME 实用程序、功能性效果系统和流畅的 API，以便在不影响性能或安全性的情况下，方便最终用户进行多步骤、多链交互。                                                                                                                                                                                                                                          | TypeScript |
| [Polkadot JS API](https://polkadot.js.org/docs/api)              | 提供了一个 Javascript 库，用于构建能够在与基于 Substrate 的链交互时动态适应节点变化（例如区块浏览器或与链无关的服务）的应用程序。你可以将此库与流行的前端框架（如 React）结合使用。                                                                                                                                                                                                                             | Javascript |
| [Polkadot JS 扩展](https://polkadot.js.org/docs/extension/) | 提供了一个 API，用于与使用 Polkadot JS API 构建的浏览器扩展和提供程序进行交互。                                                                                                                                                                                                                                                                                                                                                                                                                         | Javascript |
| [Substrate Connect](/learn/light-clients-in-substrate-connect/)  | 提供了一个库和一个浏览器扩展，用于构建使用浏览器内轻客户端节点直接连接到基于 Substrate 的链的应用程序。Substrate Connect 使你能够构建连接到多个链的应用程序，如果他们使用你的应用程序与多个链交互，则为最终用户提供单一体验。                                                                                                                                                                      | Javascript |
| [`subxt`](https://github.com/paritytech/subxt/)                  | 提供了一个 Rust 库，该库根据目标链的元数据生成一个静态类型的 Rust 接口，用于与节点的 RPC API 交互。`subxt`（提交外部函数）库使你能够构建更低级别的应用程序（例如非浏览器图形用户界面、特定于链的 CLI 或需要节点和生成的接口之间类型安全通信的用户界面应用程序），以防止用户构建具有错误输入的事务或提交不存在的调用。 | Rust       |
| [`txwrapper`](https://github.com/paritytech/txwrapper)           | 提供了一个 Javascript 库，用于离线生成已签名的 Substrate 事务。此库使你能够编写脚本以离线生成已签名的交易，稍后可以将其提交到节点。此功能对于测试和解码交易特别有用。                                                                                                                                                                                                                             | Javascript |

有关 JSON-RPC API 和最新接口规范的更多信息，请参阅 [JSON-RPC 规范](https://paritytech.github.io/json-rpc-interface-spec/)。

## 接下来的步骤

- [Substrate Connect](https://github.com/paritytech/substrate-connect)
- [安装前端模板](/tutorials/build-a-blockchain/build-local-blockchain/#install-the-front-end-template)
- [生成元数据二维码](https://github.com/paritytech/metadata-portal)
- [获取向后兼容的元数据 (desub)](https://github.com/paritytech/desub)
