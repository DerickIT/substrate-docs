---
title: 密钥签名和网络操作
description: 
keywords:
---

在[帐户、地址和密钥](/learn/accounts-addresses-keys/)中，对公钥和私钥的讨论侧重于识别与用户与不同网络交互相关的帐户和地址。
但是，密钥和不同的密钥签名对于部署 Substrate 节点和执行特定的节点操作也至关重要。
本节回顾了加密方案以及不同节点组件在何处使用它们。

## 数字签名方案

大多数数字签名方案提供以下功能：

- **密钥生成**。签名方案必须提供一种方法，用于从所有可能的私钥集中生成随机私钥，以及一个相应的公钥，该公钥可用于验证私钥的真实性。
- **消息签名**。签名方案必须提供一种方法，用于为给定的消息和私钥生成签名。
- **签名验证**。签名方案必须提供一种方法，用于根据正在评估的消息、公钥和签名来接受或拒绝消息的真实性。

不同的签名方案使用不同的算法来执行这些操作。
无论使用何种数学方法，所有签名方案的设计都旨在实现两个主要结果：

- 使用相应的公钥验证为给定消息和私钥生成的签名的真实性。
- 可以合理地假设消息的完整性，因为在没有私钥的情况下生成有效签名在计算上是不可行的。

Substrate 基于链支持以下签名方案：

| 方案  | 描述                                                                                                                                                                                                                                                                                                                                                                                                                          |
| :------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ed25519 | Ed25519 签名方案基于 Edwards 曲线数字签名算法 (EdDSA) 签名方案（Schnorr 签名的变体），该方案使用 SHA-512 和 Curve25519。与 ECDSA 签名方案相比，此签名方案生成更安全的签名，并且签名消息的速度明显更快。                                                                                                             |
| sr25519 | Sr25519 签名方案是 Substrate 的默认签名方案。此签名方案基于 Schnorrkel 变体，该变体使用带有 Ristretto 点压缩的 Schnorr 签名。Sr25519 签名方案支持其他功能（例如分层确定性密钥派生、多签名签名和可验证随机函数），这些功能在区块链环境中特别有用。 |
| ecdsa   | 椭圆曲线数字签名算法 (ECDSA) 是数字签名算法 (DSA) 的一种变体，它使用 Secp256k1 椭圆曲线密码学。由于保护 Schnorr 签名的专利，此签名方案最初在比特币和以太坊中使用。使用 ECDSA 签名方案会使某些高级加密技术（例如阈值签名）变得复杂。                           |

## 会话密钥和类型

会话密钥是验证器用于签署与共识相关的消息的私有在线密钥。
会话密钥必须在线可用，以使验证器能够执行某些网络操作。

这些密钥不用于控制资金，并且仅应用于其预期目的。
它们可以定期更改。
要创建会话密钥，验证器节点操作员必须使用控制器帐户生成一个使用会话的公钥签名的证书。
该证书证明该密钥代表验证器的质押帐户和提名人。
创建会话密钥后，验证器节点操作员通过在链上发布交易中的会话证书来通知链此密钥代表控制器密钥。
在大多数情况下，节点操作员使用[会话](https://paritytech.github.io/substrate/master/pallet_session/index.html) 托盘来管理他们的会话密钥。

[`SessionKeys`](https://paritytech.github.io/substrate/master/sp_session/index.html)
特征是一个通用的、可索引的类型，您可以在运行时声明任意数量的会话密钥。
默认的 Substrate 节点模板使用四个会话密钥。
其他链可以根据链期望其验证器执行的操作拥有更多或更少的密钥。

在实践中，验证器将所有会话公钥组合到一个对象中，使用控制器帐户对公钥集进行签名，并提交交易以在链上注册密钥。
此链上注册将验证器_节点_与持有资金的_帐户_链接起来。
因此，与会话密钥对象关联的帐户可以根据节点的行为获得奖励或被削减。

运行时使用 `impl_opaque_keys!` 宏声明使用哪些会话密钥：

```rust
impl_opaque_keys! {
    pub struct SessionKeys {
        pub grandpa: Grandpa,
        pub babe: Babe,
        pub im_online: ImOnline,
        pub authority_discovery: AuthorityDiscovery,
    }
}
```

Polkadot 使用以下会话密钥：

| 名称                 | 类型    |
| :------------------- | :------ |
| 权威发现              | sr25519 |
| GRANDPA              | ed25519 |
| BABE                 | sr25519 |
| 我在线           | sr25519 |
| 平行链分配           | sr25519 |
| 平行链验证器        | ed25519 |

BABE 需要适合在可验证随机函数中以及用于数字签名的密钥。
Sr25519 密钥同时具有这两种功能，因此用于 BABE。

## 命令行界面

您可以使用 `polkadot keys` 或 `subkey` 命令来生成和检查密钥。

两个重要的子命令是：

- `generate` 用于创建新的随机帐户并将私钥打印到标准输出或将密钥保存到文件。
- `inspect` 用于通过传递秘密短语或种子来查看帐户的帐户和地址信息。

一些重要的选项是：

- `--network` 用于指定密钥将用于的网络（默认值为 `substrate`）。
- `--scheme` 用于指定密钥的签名方案（默认值为 `sr25519`）。

例如，您可以通过运行以下命令**生成** Polkadot 随机密钥：

```bash
polkadot key generate -n polkadot
```

该命令显示类似以下的输出：

```text
Secret phrase:       settle whisper usual blast device source region pumpkin ugly beyond promote cluster
  Network ID:        polkadot
  Secret seed:       0x2e6371e04b45f16cd5c2d66fc47c8ad7f2881215287c374abfa0e07fd003cb01
  Public key (hex):  0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Account ID:        0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Public key (SS58): 14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
  SS58 Address:      14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
```

您可以通过使用秘密短语运行类似以下的命令来**检查**密钥的帐户和地址信息：

```bash
./polkadot key inspect -n polkadot "settle whisper usual blast device source region pumpkin ugly beyond promote cluster"
```

该命令显示类似以下的输出：

```text
Secret phrase:       settle whisper usual blast device source region pumpkin ugly beyond promote cluster
  Network ID:        polkadot
  Secret seed:       0x2e6371e04b45f16cd5c2d66fc47c8ad7f2881215287c374abfa0e07fd003cb01
  Public key (hex):  0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Account ID:        0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Public key (SS58): 14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
  SS58 Address:      14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
```

要检查使用硬派生 `//Stash//0` 创建的密钥，您将运行类似以下的命令：

```bash
polkadot key inspect -n polkadot "settle whisper usual blast device source region pumpkin ugly beyond promote cluster//Stash//0"
```

该命令显示类似以下的输出：

```text
Secret Key URI `settle whisper usual blast device source region pumpkin ugly beyond promote cluster//Stash//0` is account:
  Network ID:        polkadot
 Secret seed:       0xe9437b365161e8228e8abd53d64e6b31058dcddcd0b96f895045ecc41579ee3e
  Public key (hex):  0xd8ed7b942f6e590b06e99951ac10e3312f65f01df5b3f250b70374fc2da1046d
  Account ID:        0xd8ed7b942f6e590b06e99951ac10e3312f65f01df5b3f250b70374fc2da1046d
  Public key (SS58): 15uRtdeE4MyMHV9LP1UHKqTx4f8Qa8uVZUpxWWw8VKSroucK
  SS58 Address:      15uRtdeE4MyMHV9LP1UHKqTx4f8Qa8uVZUpxWWw8VKSroucK
```

有关使用 subkey 命令和命令行选项的更多信息，请参阅[subkey](/reference/command-line-tools/subkey/)。
