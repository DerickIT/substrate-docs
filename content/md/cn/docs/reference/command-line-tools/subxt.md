---
title: subxt
description: 提供使用 subxt 程序生成和管理密钥的命令行参考信息。
keywords:
---

`subxt` 程序是一个密钥生成和管理实用程序，包含在 Substrate 存储库中。
您可以使用 `subxt` 程序执行以下任务：

- 生成和检查密码安全的公钥和私钥对。
- 从秘密短语和原始种子恢复密钥。
- 对消息进行签名和验证签名。
- 对编码的交易进行签名和验证签名。
- 推导出分层确定性子密钥对。

## 签名方案

`subxt` 程序目前支持以下签名方案：

- [sr25519](https://wiki.polkadot.network/docs/en/learn-cryptography): Ristretto 群上的 Schorr 签名。
- [ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519): Curve25519 上的 SHA-512（SHA-2）。
- [secp256k1](https://en.bitcoin.it/wiki/Secp256k1): secp256k1 上的 ECDSA 签名。

在基于 Substrate 的网络中，`sr25519` 编码的密钥用于生成 SS58 地址作为与区块链交互的公钥。

## 安装

您可以使用 `cargo` 下载、安装和编译 `subxt`，而无需克隆完整的 Substrate 存储库。
但是，您必须在将 `subxt` 作为独立二进制文件安装之前将 Substrate 构建依赖项添加到您的环境中。
为了确保依赖项可用，您可以从 Substrate 存储库的克隆中构建 `subxt` 二进制文件。

要安装和编译 `subxt` 程序：

1. 如果需要，请打开一个终端 shell。

1. 如果需要，请验证您是否已安装 Rust 编译器和工具链。

1. 如果需要，请通过运行以下命令克隆 Polkadot SDK 存储库：

   ```bash
   git clone https://github.com/paritytech/polkadot-sdk.git
   ```

1. 通过运行以下命令更改到 Substrate 存储库的根目录：

   ```bash
   cd polkadot-sdk
   ```

1. 通过运行以下命令使用 `nightly` 工具链编译 `subxt` 程序：

   ```bash
   cargo +nightly build --package subxt --release
   ```

   由于涉及的软件包数量众多，编译程序可能需要几分钟。

1. 通过运行以下命令更改回 Polkadot SDK 存储库的根目录：

   ```bash
   cd ..
   ```

1. 验证 `subxt` 程序是否已准备好使用，并通过运行以下命令查看可用选项的信息：

   ```bash
   ./target/release/subxt --help
   ```

## 分层确定性密钥

`subxt` 程序支持分层确定性密钥。
分层确定性 (HD) 密钥使您能够使用父种子在分层树结构中推导出子密钥对。
在此分层结构中，从父级推导出的每个子级都有自己的密钥对。
推导出的密钥也可以用于推导出其他子密钥对，类似于文件系统如何在分层目录结构中具有嵌套目录。
有关如何推导出分层确定性密钥的背景信息，请参阅分层确定性钱包的 [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 规范。

有关使用 subxt 命令推导出分层确定性密钥的信息，请参阅 [使用推导出的密钥](#使用推导出的密钥)。

## 基本命令用法

运行 `subxt` 命令的基本语法是：

```text
subxt [子命令] [标志]
```

根据您指定的子命令，可能适用或需要其他参数、选项和标志。
要查看特定 `subxt` 子命令的用法信息，请指定子命令和 `--help` 标志。
例如，要查看 `subxt inspect` 的用法信息，您可以运行以下命令：

```bash
subxt inspect --help
```

### 标志

您可以在 `subxt` 命令中使用以下可选标志。

| 标志          | 描述                   |
| ------------- | ----------------------------- |
| -h, --help    | 显示用法信息。   |
| -V, --version | 显示版本信息。 |

### 子命令

您可以在 `subxt` 命令中使用以下子命令。
有关参考信息和说明使用 subxt 子命令的示例，请选择相应的命令。

| 命令                                          | 描述                                                                                                                     |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| [`generate`](#subxt-generate)                   | 生成一个随机的帐户密钥。                                                                                                 |
| [`generate-node-key`](#subxt-generate-node-key) | 生成一个随机的节点 `libp2p` 密钥。您可以将密钥保存到文件或将其显示为标准输出 (`stdout`)。 |
| [`help`](#subxt-help)                           | 显示 `subxt` 或指定子命令的用法信息。                                                          |
| [`inspect`](#subxt-inspect)                     | 显示您指定的秘密 URI 的公钥和 SS58 地址。                                                        |
| [`inspect-node-key`](#subxt-inspect-node-key)   | 显示与您指定的文件名中的秘密节点密钥相对应的对等点 ID。                                    |
| [`sign`](#subxt-sign)                           | 使用您指定的密钥对消息进行签名。                                                                                |
| [`vanity`](#subxt-vanity)                       | 生成一个提供虚荣地址的种子。                                                                                |
| [`verify`](#subxt-verify)                       | 验证消息的签名对于您指定的公钥或密钥是否有效。                                         |

### 输出

根据您指定的子命令，subxt 程序的输出将显示以下部分或全部信息：

| 此字段        | 包含                                                                                                                                                                                                                                                    |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 秘密短语     | 一系列英文单词，以人类友好的方式对密钥进行编码。这组单词（也称为助记词或种子短语）可用于在提供正确的一组单词并按正确顺序提供时恢复密钥。 |
| 秘密种子       | 恢复密钥对所需的最小信息。秘密种子有时也称为私钥或原始种子。所有其他信息都是从此值计算出来的。                                                                   |
| 公钥（十六进制）  | 以十六进制格式表示的密码密钥对的公钥部分。                                                                                                                                                                                        |
| 公钥（SS58） | 以 SS58 编码表示的密码密钥对的公钥部分。                                                                                                                                                                                             |
| 帐户 ID        | 以十六进制格式表示的公钥的别名。                                                                                                                                                                                                          |
| SS58 地址      | 基于公钥的 SS58 编码的公共地址。                                                                                                                                                                                                     |

### 示例

要显示 `subxt` 程序的版本信息，请运行以下命令：

```bash
subxt --version
```

要显示 `verify` 子命令的用法信息，请运行以下命令：

```bash
subxt verify --help
```

## subxt generate

使用 `subxt generate` 命令生成公钥和私钥以及帐户地址。
您可以使用命令行选项使用不同的签名方案生成密钥，或者使用更多或更少的单词生成助记词。

#### 基本用法

```text
subxt generate [标志] [选项]
```

#### 标志

您可以在 `subxt generate` 命令中使用以下可选标志。

| 标志 | 描述 |
| ---- | ----------- |
| `-h`, `--help` | 显示用法信息。|
| `--password-interactive` | 允许您在终端中以交互方式输入访问密钥库的密码。 |
| `-V`, `--version`| 显示版本信息。 |

#### 选项

您可以在 `subxt generate` 命令中使用以下命令行选项。

| 选项 | 描述 |
| ------ | ----------- |
| `--keystore-path <path>` | 指定自定义密钥库路径。 |
| `--keystore-uri <keystore-uri>` | 指定要连接到的自定义 URI 以获取密钥库服务 |
| `-n`, `--network <network>` | 指定要使用的网络地址格式。例如，`kusama` 或 `polkadot`。有关支持的网络的完整列表，请参阅在线用法信息。 |
| `--output-type <format>` | 指定要使用的输出格式。有效值为 Json 和 Text。默认输出格式为 Text。 |
| `--password <password>` | 指定密钥库使用的密码。此选项使您能够将额外的密钥附加到种子。 |
| `--password-filename <path>` | 指定包含密钥库使用的密码的文件的名称。 |
| `--scheme <scheme>` | 指定要生成的密钥的密码方案。有效值为 `Ecdsa`、`Ed25519`、`Sr25519`。默认方案为 `Sr25519`。 |
| `-w`, `--words <words>` | 指定要生成的密钥的秘密短语中的单词数。有效值为 12、15、18、21、24。默认情况下，秘密短语包含 12 个单词。 |

#### 示例

要生成一个使用 sr25519 签名方案的新密钥对，请运行以下命令：

```bash
subxt generate
```

该命令将显示类似以下内容的输出，其中包含一个 12 个单词的秘密短语：

```bash
Secret phrase:       bread tongue spell stadium clean grief coin rent spend total practice document
  Secret seed:       0xd5836897dc77e6c87e5cc268abaaa9c661bcf19aea9f0f50a1e149d21ce31eb7
  Public key (hex):  0xb6a8b4b6bf796991065035093d3265e314c3fe89e75ccb623985e57b0c2e0c30
  Account ID:        0xb6a8b4b6bf796991065035093d3265e314c3fe89e75ccb623985e57b0c2e0c30
  Public key (SS58): 5GCCgshTQCfGkXy6kAkFDW1TZXAdsbCNZJ9Uz2c7ViBnwcVg
  SS58 Address:      5GCCgshTQCfGkXy6kAkFDW1TZXAdsbCNZJ9Uz2c7ViBnwcVg
```

`subxt` 程序根据其使用网络所需的格式对与公钥/私钥对关联的地址进行不同的编码。
如果您想在 Kusama 和 Polkadot 网络上使用 **相同的私钥**，则可以使用 `--network` 选项为 Kusama 和 Polkadot 网络生成单独的地址格式。
公钥是相同的，但地址格式是特定于网络的。
要为特定网络生成密钥对，请运行类似以下命令：

```bash
subxt generate --network picasso
```

该命令将显示与输出相同的字段，但使用您指定的网络的地址格式。

要生成一个使用 `ed25519` 签名方案和 24 个单词的秘密短语的更安全的密钥对，用于 `moonriver` 网络，您将运行以下命令：

```bash
subxt generate --scheme ed25519 --words 24 --network moonriver
```

该命令将显示与输出相同的字段，但使用 Ed25519 签名方案、24 个单词的秘密短语和 `moonriver` 网络的地址格式。

```text
Secret phrase:       cloth elevator sadness twice arctic adjust axis vendor grant angle face section key safe under fee fine garage pupil hotel museum valve popular motor
  Secret seed:       0x5fa5923c1d6753fa30f268ffd363efb730ca0db906f55bc17efe65cd24f92097
  Public key (hex):  0x3f1da4d35489e3d84739de1490f51b567ad2a62793cca1357e624fbfa534fc85
  Account ID:        0x3f1da4d35489e3d84739de1490f51b567ad2a62793cca1357e624fbfa534fc85
  Public key (SS58): VkFLVqcighJnssSbL4LDSGy5ShJQZvYhm7G8K8W1ZXt96Z5VG
  SS58 Address:      VkFLVqcighJnssSbL4LDSGy5ShJQZvYhm7G8K8W1ZXt96Z5VG
```

要生成一个受密码保护的密钥，请使用 `--password <password>` 选项运行 `subxt generate` 命令。
例如：

```bash
subxt generate --password "pencil laptop kitchen cutter"
```

在您生成需要密码的密钥后，您可以通过在命令行中包含 `--password` 选项和密码字符串，或者在秘密短语末尾添加三个斜杠 (`///`) 来检索它。
请记住，务必保护密码、秘密短语和秘密种子，并在安全的位置备份它们。

## subxt generate-node-key

使用 `subxt generate-node-key` 命令为 Substrate 节点之间的对等 (`libp2p`) 通信生成随机的公钥和私钥。
公钥是对等点标识符，用于在链规范文件中或作为命令行参数来标识参与区块链网络的节点。
在大多数情况下，您将使用命令行选项运行此命令，以将私钥保存到文件。

#### 基本用法

```text
subxt generate-node-key [标志] [选项]
```

#### 标志

您可以在 `subxt generate-node-key` 命令中使用以下可选标志。

| 标志              | 描述                   |
| ----------------- | ----------------------------- |
| `-h`, `--help`    | 显示用法信息。   |
| `-V`, `--version` | 显示版本信息。 |

#### 选项

您可以在 `subxt generate-node-key` 命令中使用以下命令行选项。

| 选项  | 描述 |
| ------- | ----------- |
| `--file <file-name>` | 指定要用于保存为本地节点生成的密钥的文件位置。如果您未指定此选项，则生成的密钥将显示为标准输出 (`stdout`)。 |

#### 示例

要为对等通信生成一个随机密钥对并将私钥保存到文件，请运行类似以下命令：

```bash
subxt generate-node-key --file ../generated-node-key
```

此命令将在终端中显示节点密钥的对等点标识符，并将私钥保存到 `generated-node-key` 文件中。
在此示例中，保存的密钥位于父目录中，而不是当前工作目录中。

```text
12D3KooWHALHfL7dDBiGTt4JTEAvCbDWts8zHwvcPvJXDF9fxue7
```

## subxt help

使用 `subxt help` 命令显示 `subxt` 或指定子命令的用法消息。

#### 基本用法

```text
subxt help [子命令]
```

#### 示例

要显示 `verify` 子命令的用法信息，请运行以下命令：

```bash
subxt help verify
```

## subxt inspect

使用 `subxt inspect` 命令重新计算指定密钥或助记词的公钥和公共地址。

#### 基本用法

```text
subxt inspect [标志] [选项] uri
```

#### 标志

您可以在 `subxt inspect` 命令中使用以下可选标志。

| 标志 | 描述 |
| ---- | ----------- |
| `-h`, `--help` | 显示用法信息。 |
| `--password-interactive` | 允许您在终端中以交互方式输入访问密钥库的密码。 |
| `--public` | 指示您指定要检查的 `uri` 是一个十六进制编码的公钥。 |
| `-V`, `--version` | 显示版本信息。 |

#### 选项

您可以在 `subxt inspect` 命令中使用以下命令行选项。

| 选项 | 描述 |
| ------ | ----------- |
| `--keystore-path <path>` | 指定自定义密钥库路径。 |
| `--keystore-uri <keystore-uri>` | 指定要连接到的自定义 URI 以获取密钥库服务。 |
| `-n`, `--network <network>` | 指定要使用的网络地址格式。例如，`kusama` 或 `polkadot`。有关支持的网络的完整列表，请参阅在线用法信息。 |
| `--output-type <format>` | 指定要使用的输出格式。有效值为 Json 和 Text。默认输出格式为 Text。 |
| `--password <password>` | 指定密钥库使用的密码。此选项使您能够将额外的密钥附加到种子。 |
| `--password-filename <path>` | 指定包含密钥库使用的密码的文件的名称。 |
| `--scheme <scheme>` | 指定要检查的密钥的密码方案。有效值为 `Ecdsa`、`Ed25519`、`Sr25519`。默认方案为 `Sr25519`。 |

#### 参数

您必须在 `subxt inspect` 命令中指定以下必需参数。

| 参数 | 描述 |
| -------- | ----------- |
| `uri` | 指定要检查的密钥 URI。您可以使用其秘密短语、秘密种子（带有派生路径和密码）、SS58 地址、公钥或十六进制编码的公钥来指定密钥。如果您使用十六进制编码的公钥指定 `uri`，则还必须在命令行中包含 `--public` 标志。如果您为 `uri` 指定了一个文件名，则该文件的内容将用作 URI。 |

#### 示例

要检查从助记词推导出的公钥，您可以运行类似以下命令：

```bash
subxt inspect "caution juice atom organ advance problem want pledge someone senior holiday very"
```

该命令将显示类似以下内容的输出：

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  Public key (hex):  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58): 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  Account ID:        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 Address:      5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

要检查从秘密种子推导出的公钥，您可以运行类似以下命令：

```bash
subxt inspect 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```

如果您将秘密短语或秘密种子存储在文本文件中（例如，`my-secret-key`），则可以在命令行上指定文件名以传递文件的内容并显示与该秘密短语或秘密种子关联的公钥。
例如，您可以运行类似以下命令：

```bash
subxt inspect my-secret-key
```

要使用十六进制编码的公钥检查公钥，您可以运行类似以下命令：

```bash
subxt inspect --public 0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
```

在这种情况下，该命令仅显示类似以下内容的公共信息：

```text
Network ID/version: substrate
  Public key (hex):   0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Account ID:         0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58):  5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  SS58 Address:       5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

`subxt` 程序根据其使用网络所需的格式对与公钥/私钥对关联的地址进行不同的编码。
如果您在 Kusama 和 Polkadot 网络上使用 **相同的私钥**，则可以使用 `--network` 选项检查特定网络使用的地址。
公钥是相同的，但地址格式是特定于网络的。
要检查特定网络的密钥对，请运行类似以下命令：

```bash
subxt inspect --network kusama "caution juice atom organ advance problem want pledge someone senior holiday very"
```

在命令输出中，秘密短语、秘密种子和公钥是相同的，但 Kusama 网络的地址是：

```text
  SS58 Address:      HRkCrbmke2XeabJ5fxJdgXWpBRPkXWfWHY8eTeCKwDdf4k6
```

要检查同一私钥在 Polkadot 网络上的地址，您将运行类似以下命令：

```bash
subxt inspect --network polkadot "caution juice atom organ advance problem want pledge someone senior holiday very"
```

在命令输出中，秘密短语、秘密种子和公钥与 Kusama 网络相同，但 Polkadot 网络的地址是：

```text
  SS58 Address:      15rRgsWxz4H5LTnNGcCFsszfXD8oeAFd8QRsR6MbQE2f6XFF
```

要通过指定 `--password` 选项和密码来检查受密码保护的密钥，您可以运行类似以下命令：

```bash
subxt inspect "caution juice atom organ advance problem want pledge someone senior holiday very" --password "pencil laptop kitchen cutter"
```

如果您在命令行中指定了 `--password` 选项和密码，则命令输出 _不会_ 显示使用的密码。

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xdfc5d5d5235a37fdc907ee1cb720299f96aeb02f9c7c2fcad7ee8c7bfbd2a4db
  Public key (hex):  0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  Public key (SS58): 5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
  Account ID:        0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  SS58 Address:      5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
```

您也可以通过在秘密短语中添加 `///` 和密码来检查受密码保护的密钥。
例如，您可以运行类似以下命令：

```bash
subxt inspect "caution juice atom organ advance problem want pledge someone senior holiday very///pencil laptop kitchen cutter"
```

在这种情况下，命令输出将显示使用的密码。
例如：

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very///pencil laptop kitchen cutter` is account:
  Secret seed:       0xdfc5d5d5235a37fdc907ee1cb720299f96aeb02f9c7c2fcad7ee8c7bfbd2a4db
  Public key (hex):  0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  Public key (SS58): 5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
  Account ID:        0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  SS58 Address:      5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
```

## subxt inspect-node-key

使用 `subxt inspect-node-key` 命令显示与指定文件名中的节点密钥相对应的节点的对等点标识符。
在使用此命令之前，您应该已经使用过 [`subxt generate-node-key`](#subxt-generate-node-key) 命令并将密钥保存到文件。

#### 基本用法

```text
subxt inspect-node-key [标志] [选项] --file <file-name>
```

#### 标志

您可以在 `subxt inspect-node-key` 命令中使用以下可选标志。

| 标志              | 描述                   |
| ----------------- | ----------------------------- |
| `-h`, `--help`    | 显示用法信息。   |
| `-V`, `--version` | 显示版本信息。 |

#### 选项

您可以在 `subxt inspect-node-key` 命令中使用以下命令行选项。

| 选项 | 描述 |
| ------ | ----------- |
| `-n, --network <network>` | 指定要使用的网络地址格式。例如，`kusama` 或 `polkadot`。有关支持的网络的完整列表，请参阅在线用法信息。 |

#### 参数

您必须在 `subxt inspect-node-key` 命令中指定以下必需参数。

| 参数 | 描述 |
| -------- | ----------- |
| `--file <file-name>` | 指定包含为与节点的对等通信生成的密钥的文件。 |

## subxt sign

使用 `subxt sign` 命令通过将消息作为标准输入 (`stdin`) 传递来对消息进行签名。
您可以使用您的秘密种子或秘密短语对消息进行签名。

#### 基本用法

```text
subxt sign [标志] [选项]
```

#### 标志

您可以在 `subxt sign` 命令中使用以下可选标志。

| 标志 | 描述 |
| ---- | ----------- |
| `-h`, `--help` | 显示用法信息。 |
| `--hex` | 指示您作为标准输入指定的 message 是一个十六进制编码的消息。 |
| `--password-interactive` | 允许您在终端中以交互方式输入访问密钥库的密码。 |
| `-V`, `--version` | 显示版本信息。 |

#### 选项

您可以在 `subxt sign` 命令中使用以下命令行选项。

| 选项 | 描述 |
| ------ | ----------- |
| `--keystore-path <path>` | 指定自定义密钥库路径。 |
| `--keystore-uri <keystore-uri>` | 指定要连接到的自定义 URI 以获取密钥库服务。 |
| `--message <network>` | 指定要签名的消息字符串。 |
| `--password <password>` | 指定密钥库使用的密码。此选项使您能够将额外的密钥附加到种子。 |
| `--password-filename <path>` | 指定包含密钥库使用的密码的文件的名称。 |
| `--scheme <scheme>` | 指定密钥的密码签名方案。有效值为 `Ecdsa`、`Ed25519`、`Sr25519`。默认方案为 `Sr25519`。 |
| `--suri <secret-seed>` | 指定要用于对消息进行签名的密钥 URI。您可以使用其秘密短语、秘密种子（带有派生路径和密码）来指定密钥。如果您为 `--suri` 选项指定了一个文件名，则该文件的内容将用作 URI。如果您省略此选项，则会提示您输入 URI。 |

#### 示例

以下示例使用 `echo` 命令将测试消息作为输入传递到 `subxt sign` 命令。
要在终端中对文本消息进行签名，您可以运行类似以下命令：

```bash
echo "test message" | subxt sign --suri 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```

命令输出将显示消息的签名。
例如：

```text
f052504de653a5617c46eeb1daa73e2dbbf625b6bf8f16d9d8de6767bc40d91dfbd38c13207f8a03594221c9f68c00a158eb3120311b80ab2da563b82a995b86
```

要对十六进制编码的消息进行签名，请运行类似以下命令：

```bash
subxt sign --hex --message 68656c6c6f2c20776f726c64 --suri 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```

命令输出将显示消息的签名。
例如：

```text
9ae07defc0ddb752651836c25ac643fbdf9d45ba180ec6d09e4423ff6446487a52b609d69c06bd1c3ec09b3d06a43f019bacba12dc5a5697291c5e9faab13288
```

## subxt vanity

使用 `subxt vanity` 命令创建一个包含指定字符串模式的地址。
此命令不会为自定义地址生成秘密短语。

#### 基本用法

```text
subxt vanity [标志] [选项] --pattern <pattern>
```

#### 标志

您可以在 `subxt vanity` 命令中使用以下可选标志。

| 标志              | 描述                   |
| ----------------- | ----------------------------- |
| `-h`, `--help`    | 显示用法信息。   |
| `-V`, `--version` | 显示版本信息。 |

#### 选项

您可以在 `subxt vanity` 命令中使用以下命令行选项。

| 选项 | 描述 |
| ------ | ----------- |
| `-n, --network <network>` | 指定要使用的网络地址格式。例如，`kusama` 或 `polkadot`。有关支持的网络的完整列表，请参阅在线用法信息。 |
| `--output-type <format>`  | 指定要使用的输出格式。有效值为 Json 和 Text。默认输出格式为 Text。 |
| `--scheme <scheme>` | 指定密钥的密码签名方案。有效值为 `Ecdsa`、`Ed25519`、`Sr25519`。默认方案为 `Sr25519`。 |

#### 参数

您必须在 `subxt vanity` 命令中指定以下必需参数。

| 参数 | 描述 |
| -------- | ----------- |
| `--pattern <pattern>` | 指定要包含在生成的地址中的字符串。 |

#### 示例

根据您指定的模式，`subxt vanity` 命令可能需要一些时间来搜索密钥库并生成包含自定义字符串的地址。
通常，您应该为 `--pattern` 使用尽可能少的字符，并使用 `--network` 选项指定要使用自定义地址的网络。

要生成一个包含特定字符串的地址，您可以运行类似以下命令：

```bash
subxt vanity --network kusama --pattern DUNE
```

该命令将显示类似以下内容的输出：

```text
Generating key containing pattern 'DUNE'
100000 keys searched; best is 187/237 complete
200000 keys searched; best is 189/237 complete
300000 keys searched; best is 221/237 complete
400000 keys searched; best is 221/237 complete
500000 keys searched; best is 221/237 complete
600000 keys searched; best is 221/237 complete
best: 237 == top: 237
Secret Key URI `0x82737756075d15409053afd19a6b29ae2abeed96a3487d71d2af9b3eff19cbfa` is account:
  Secret seed:       0x82737756075d15409053afd19a
