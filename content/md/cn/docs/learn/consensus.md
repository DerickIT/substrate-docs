---
title: 共识
description: 描述 Substrate 提供的共识模型。
keywords:
---

所有区块链都需要某种类型的共识机制来就区块链状态达成一致。由于 Substrate 为构建区块链提供了模块化框架，因此它支持几种不同的节点达成共识的模型。
通常，不同的共识模型具有不同的权衡，因此选择要用于链的共识类型是一个重要的考虑因素。
Substrate 默认支持的共识模型需要最少的配置，但如果需要，也可以构建自定义共识模型。

## 两阶段共识

与某些区块链不同，Substrate 将达成共识的要求分为两个独立的阶段：

- **区块生成**是节点用于创建新区块的过程。

- **区块最终确定**是用于处理分叉并选择**规范**链的过程。

## 区块生成

在达成共识之前，区块链网络中的一些节点必须能够生成新区块。
区块链如何决定有权生成区块的节点取决于您正在使用的共识模型。
例如，在集中式网络中，单个节点可能会生成所有区块。
在没有任何可信节点的完全去中心化网络中，必须在每个区块高度选择区块生成者。

对于基于 Substrate 的区块链，您可以选择以下区块生成算法之一或创建您自己的算法：

- 基于权限的循环调度（[Aura](/reference/glossary/#authority-round-aura)）。
- 区块扩展的盲分配（[BABE](/reference/glossary/#blind-assignment-of-blockchain-extension-babe)）基于时隙的调度。
- 基于工作量证明计算的调度。

Aura 和 BABE 共识模型要求您有一组已知的**验证者节点**，这些节点被允许生成区块。
在这两种共识模型中，时间被划分为离散的时隙。
在每个时隙中，只有部分验证者可以生成区块。
在 Aura 共识模型中，可以生成区块的验证者以循环方式轮换。
在 BABE 共识模型中，验证者是根据可验证随机函数 (VRF) 选择的，而不是轮换选择方法。

在工作量证明共识模型中，如果节点解决了计算密集型问题，则任何节点都可以在任何时间生成区块。
解决问题需要 CPU 时间，因此节点只能根据其计算资源的比例生成区块。
Substrate 提供了一个工作量证明区块生成引擎。

## 最终确定和分叉

作为基本要素，区块包含一个区块头和[交易](/learn/transaction-types)。
每个区块头都包含对其父区块的引用，因此您可以追溯到其创世区块。
当两个区块引用同一个父区块时，就会发生分叉。
区块最终确定是一种解决分叉的机制，以便只存在规范链。

分叉选择规则是一种算法，用于选择应扩展的最佳链。
Substrate 通过[`SelectChain`](https://paritytech.github.io/substrate/master/sp_consensus/trait.SelectChain.html) 特性公开此分叉选择规则。
您可以使用该特性编写自定义分叉选择规则，或使用[GRANDPA](https://github.com/w3f/consensus/blob/master/pdf/grandpa.pdf)，这是 Polkadot 和类似链中使用的最终确定机制。

在 GRANDPA 协议中，最长链规则简单地说，最佳链是最长链。
Substrate 使用[`LongestChain`](https://paritytech.github.io/substrate/master/sc_consensus/struct.LongestChain.html) 结构提供此链选择规则。
GRANDPA 在其投票机制中使用最长链规则。

![最长链规则](/media/images/docs/consensus-longest.png)

最长链规则的一个缺点是，攻击者可以创建一个超过网络速度的区块链，并有效地劫持主链。
贪婪最重观察子树 (GHOST) 规则指出，从创世区块开始，每个分叉都通过选择在其上构建了最多区块的最重分支来解决。

![GHOST 规则](/media/images/docs/consensus-ghost.png)

在此图中，最重的链是已在其上累积了最多构建区块的分叉。
如果您正在使用 GHOST 规则进行链选择，即使此分叉的区块少于最长链分叉，它也会被选为主链。

## 确定性最终确定

用户自然希望知道交易何时最终确定并由某些事件发出信号，例如交付收据或签署文件。
但是，使用到目前为止描述的区块生成和分叉选择规则，交易永远不会完全最终确定。
始终存在更长或更重的链可能撤销交易的可能性。
但是，在特定区块上构建的区块越多，它被撤销的可能性就越小。
这样，区块生成以及适当的分叉选择规则提供了**概率最终确定性**。

如果您的区块链需要确定性最终确定性，您可以向区块链逻辑添加最终确定机制。
例如，您可以让固定权限集的成员投**最终确定性**票。
当某个区块获得足够的投票时，该区块就被视为最终确定。
在大多数区块链中，此阈值为三分之二。
已最终确定的区块无法在没有外部协调（例如硬分叉）的情况下被撤销。

在某些共识模型中，区块生成和区块最终确定性是组合在一起的，并且在区块 `N` 最终确定之前，无法生成新的区块 `N+1`。
如您所见，在 Substrate 中，这两个过程彼此隔离。
通过将区块生成与区块最终确定分离，Substrate 使您能够将任何区块生成算法与概率最终确定性一起使用，或将其与最终确定机制结合使用以实现确定性最终确定性。

如果您的区块链使用最终确定机制，则必须修改分叉选择规则以考虑最终确定投票的结果。
例如，节点将选择包含最近最终确定的区块的最长链，而不是选择最长链周期。

## 默认共识模型

尽管您可以实现自己的共识机制，但[Substrate 节点模板](https://github.com/substrate-developer-hub/substrate-node-template) 默认包含用于区块生成的 Aura 和用于最终确定的 GRANDPA。
Substrate 还提供了 BABE 和工作量证明共识模型的实现。

### Aura

[Aura](https://paritytech.github.io/substrate/master/sc_consensus_aura/index.html) 提供了一种基于时隙的区块生成机制。
在 Aura 中，一组已知的权限轮流生成区块。

### BABE

[BABE](https://paritytech.github.io/substrate/master/sc_consensus_babe/index.html) 提供基于时隙的区块生成，具有一组已知的验证者，通常用于权益证明区块链。
与 Aura 不同，时隙分配基于可验证随机函数 (VRF) 的评估。
每个验证者都会为一个_纪元_分配一个权重。
此纪元被划分为时隙，验证者在每个时隙评估其 VRF。
对于验证者的 VRF 输出低于其权重的每个时隙，它都被允许生成区块。

由于多个验证者可能能够在同一时隙生成区块，因此即使在良好的网络条件下，BABE 中的分叉也比 Aura 中更常见。

Substrate 的 BABE 实现还具有一种回退机制，用于在给定时隙中未选择任何权限时。
这些辅助时隙分配允许 BABE 实现恒定的区块时间。

### 工作量证明

[工作量证明](https://paritytech.github.io/substrate/master/sc_consensus_pow/index.html) 区块生成不是基于时隙的，也不需要一组已知的权限。
在工作量证明中，任何人都可以在任何时间生成区块，只要他们能够解决一个计算挑战性问题（通常是哈希原像搜索）。
此问题的难度可以进行调整以提供统计目标区块时间。

### GRANDPA

[GRANDPA](https://paritytech.github.io/substrate/master/sc_consensus_grandpa/index.html) 提供区块最终确定性。
它具有一组已知的加权权限，如 BABE。
但是，GRANDPA 不会生成区块。
它只是侦听有关由区块生成节点生成的区块的闲聊。
GRANDPA 验证者对_链_进行投票，而不是对_区块_进行投票。
GRANDPA 验证者对他们认为最佳的区块进行投票，并且他们的投票会传递应用于所有先前的区块。
在三分之二的 GRANDPA 权限对特定区块投票后，该区块被视为最终确定。

所有确定性最终确定算法，包括 GRANDPA，都需要至少 `2f + 1` 个无故障节点，其中 `f` 是故障或恶意节点的数量。
在开创性论文[在存在故障的情况下达成一致](https://lamport.azurewebsites.net/pubs/reaching.pdf)或[维基百科：拜占庭故障](https://en.wikipedia.org/wiki/Byzantine_fault)中，了解此阈值的来源以及为什么它是理想的。

并非所有共识协议都定义一个单一的规范链。
当两个具有相同父区块的区块没有冲突的状态更改时，某些协议会验证[有向无环图](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG)。
有关此类示例，请参阅[AlephBFT](https://github.com/aleph-zero-foundation/aleph-node)。

## 下一步去哪里

- [BABE 研究](https://research.web3.foundation/Polkadot/protocols/block-production/Babe)
- [GRANDPA 研究](https://research.web3.foundation/Polkadot/protocols/finality)
