---
title: 状态转换和存储
description: 描述运行时中处理的状态更改如何使用 Trie 数据结构和键值数据库进行存储和管理。
keywords:
  - 状态转换
  - 存储
  - Trie
  - 数据库
---

在 Substrate 中，运行时负责处理所有链上状态更改。
这些更改通过交易提交，并通过运行时中的逻辑进行处理。
例如，如果用户发送一笔交易来转移代币，则运行时将验证交易，并更新链上状态以反映代币所有权的更改。

为了确保链上状态的完整性和一致性，Substrate 使用了两种关键技术：Trie 数据结构和键值数据库。

## Trie 数据结构

Trie 是一种树形数据结构，用于存储键值对。
在 Substrate 中，Trie 用于存储区块链状态。
每个节点都包含一个键，并且每个节点都包含一个指向子节点的指针。
Trie 的根节点表示区块链状态的根。

Trie 的主要优势在于其空间效率和快速查找能力。
Trie 的空间效率源于其共享前缀的能力。
例如，如果两个键共享相同的第一个字符，则它们将共享 Trie 中的第一个节点。
Trie 的快速查找能力源于其结构，它允许您通过遍历树来快速查找特定键。

在 Substrate 中，Trie 用于存储区块链状态的以下部分：

- **存储根**：存储根是 Trie 的根节点，它表示区块链状态的根。
- **外部函数 Trie 根**：外部函数 Trie 根是 Trie 的根节点，它表示所有外部函数的根。

## 键值数据库

键值数据库是一种用于存储键值对的数据库。
在 Substrate 中，键值数据库用于存储区块链状态的以下部分：

- **存储**：存储是键值数据库，它存储区块链状态的所有数据。
- **外部函数**：外部函数是键值数据库，它存储所有外部函数的代码。

## 状态转换

当交易提交到区块链时，运行时将处理交易并更新链上状态。
此过程称为状态转换。
状态转换涉及以下步骤：

1. **验证交易**：运行时将验证交易以确保其有效。
2. **执行交易**：如果交易有效，则运行时将执行交易并更新链上状态。
3. **提交状态更改**：运行时将提交状态更改到 Trie 和键值数据库。

## 存储管理

Substrate 使用 Trie 和键值数据库来管理链上状态。
Trie 用于存储区块链状态的根，而键值数据库用于存储所有数据。
这种方法提供了以下优势：

- **空间效率**：Trie 和键值数据库都非常空间高效，因为它们可以共享前缀并仅存储必要的数据。
- **快速查找**：Trie 和键值数据库都允许您快速查找特定键。
- **一致性**：Trie 和键值数据库都确保链上状态的一致性。

## 下一步去哪里

现在您已经了解了 Substrate 如何使用 Trie 数据结构和键值数据库来存储和管理链上状态，您可以探索以下资源以了解更多信息。

- [Substrate 文档](https://substrate.dev/docs/)
- [Substrate 存储](https://substrate.dev/docs/en/knowledge-base/runtime/storage)
- [Substrate 交易](https://substrate.dev/docs/en/knowledge-base/runtime/transactions)
- [Substrate 共识](https://substrate.dev/docs/en/knowledge-base/runtime/consensus)