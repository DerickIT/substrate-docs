---
title: 地址格式
description: 提供有关基于 Substrate 的链中帐户的默认 SS58 地址格式的技术规范。
keywords:
  - 地址格式
  - 帐户格式
  - 网络特定帐户
  - SS58
---

默认的 Substrate 地址格式是 SS58。
SS58 编码地址格式基于比特币 Base-58-check 格式，但进行了一些修改，专门设计用于基于 Substrate 的链。
您可以为基于 Substrate 的链使用其他地址格式。
但是，SS58 地址格式提供了一个 base-58 编码值，该值可以在任何 Substrate 链上标识特定帐户。
由于不同的链可能有不同的帐户标识方式，因此 SS58 地址设计为可扩展的。

## 基本格式

您可以在 [Ss58Codec](https://paritytech.github.io/substrate/master/sp_core/crypto/trait.Ss58Codec.html) 中找到 SS58 地址格式的实现。

地址的基本格式可以描述为：

```text
base58encode ( concat ( <address-type>, <address>, <checksum> ) )
```

地址是连接的字节序列，由地址类型、编码地址和校验和组成，这些校验和被传递到 base-58 编码器中。
`base58encode` 函数的实现与比特币和 IPFS 规范中定义的完全相同，使用与这两个实现相同的字母表。
Base-58 字母表消除了打印时可能看起来模棱两可的字符，例如：

- 非字母数字字符（+ 和 /）
- 零 (0)
- 大写 I (I)
- 大写 O (O)
- 小写 L (l)

## 地址类型

SS58 地址格式中的 `address-type` 是一个或多个字节，用于描述其后地址字节的确切格式。

目前，有效值为：

- `00000000b..=00111111b`（0 到 63 包含）

  简单的帐户/地址/网络标识符。
  该字节可以直接解释为这样的标识符。

- `01000000b..=01111111b`（64 到 127 包含）

  完整的地址/地址/网络标识符。
  此字节的低 6 位应被视为 14 位标识符值的较高 6 位，低 8 位由下一个字节定义。
  这适用于所有高达 2\*\*14（16,383）的标识符。

- `10000000b..=11111111b`（128 到 255 包含）

  为将来的地址格式扩展保留。
  后者（42）地址旨在对支持固定长度地址的所有 Substrate 网络有效。
  但是，对于生产网络，网络特定版本可能更可取，以帮助避免网络之间的密钥重复以及重复使用可能导致的一些问题。默认情况下，Substrate 节点以地址类型 42 打印密钥
  但是，具有替代节点实现的基于 Substrate 的链（例如，Polkadot 生态系统中的节点）可以默认为其他地址类型。

## 地址长度（以字节为单位）

共有 16 种不同的地址格式，由包括校验和在内的总有效负载的字节长度标识。

| 总计 | 类型 | 原始帐户 | 校验和 |
| ----- | ---- | ----------- | -------- |
| 3     | 1    | 1           | 1        |
| 4     | 1    | 2           | 1        |
| 5     | 1    | 2           | 2        |
| 6     | 1    | 4           | 1        |
| 7     | 1    | 4           | 2        |
| 8     | 1    | 4           | 3        |
| 9     | 1    | 4           | 4        |
| 10    | 1    | 8           | 1        |
| 11    | 1    | 8           | 2        |
| 12    | 1    | 8           | 3        |
| 13    | 1    | 8           | 4        |
| 14    | 1    | 8           | 5        |
| 15    | 1    | 8           | 6        |
| 16    | 1    | 8           | 7        |
| 17    | 1    | 8           | 8        |
| 35    | 1    | 32          | 2        |

## 校验和类型

Substrate 中存在几种潜在的校验和策略，提供了不同的长度和寿命保证。
有两种类型的校验和原像（称为 SS58 和 AccountID）和许多不同的校验和长度（1 到 8 个字节）。

在 Substrate 的所有情况下，都使用 Blake2b-512（规范，Wiki）哈希函数（OID 1.3.6.1.4.1.1722.12.2.1.16）。
这些变体只是选择用作哈希函数输入的原像以及从其输出中获取的字节数。

使用的字节始终是最左边的字节。
要使用的输入是作为 base-58 函数输入的 SS58 字节序列的非校验和部分，例如 `concat( <address-type>, <address> )`。
将上下文前缀 0x53533538505245（字符串 SS58PRE）添加到输入中以获得最终的哈希原像。

使用更多校验和字节的优点仅仅是更多字节提供了更大程度的保护，以防止输入错误和索引更改，但代价是将文本地址加宽几个字符。
对于帐户 ID 格式，这微不足道，因此不提供 1 字节的替代方案。
对于较短的帐户索引格式，额外字节代表最终地址的更大一部分，因此将其留给堆栈更高层（但不一定是用户自己）来确定最适合其目的的最佳权衡。

## 地址类型和网络注册表

[SS58 注册表](https://github.com/paritytech/ss58-registry) 是所有地址类型标识符及其如何映射到基于 Substrate 的网络的规范列表。

## 编码地址和网络标识符

可以使用简单的地址格式来表达高达 64 的标识符。
对于简单的地址格式，网络标识符值的最低有效字节表示为编码地址的第一个字节。

对于 64 到 16,383 之间的标识符，必须使用完整的地址格式。

完整的地址编码需要特殊处理，因为作为小端序的 SCALE 编码需要使用前两位来表示 01 前缀。
为了编码网络标识符，完整的地址格式将前两个字节视为 16 位序列，并忽略该序列的前两位以解释 01 前缀。
其余的 14 位以小端序编码网络标识符值，并假设缺少的两个高位为零。
这有效地将低位字节分散在两个字节之间的边界上。

例如，14 位标识符 `0b00HHHHHH_MMLLLLLL` 在两个字节中表示为：

```text
0b01LLLLLL
0bHHHHHHMM
```

目前不支持 16384 及以上的标识符。

## 验证地址

您可以使用 `subkey inspect` 命令或通过 Polkadot-JS API 来验证某个值是否是有效的 SS58 地址。

### 使用 subkey

`subkey inspect` 命令的基本语法是：

```text
subkey inspect [flags] [options] uri
```

对于 `uri` 命令行参数，您可以指定秘密种子短语、十六进制编码的私钥或 SS58 地址。
如果输入是有效的地址，则 `subkey` 程序将显示相应的十六进制编码的公钥、帐户标识符和 SS58 地址。
例如，要检查从秘密种子短语派生的公钥，您可以运行类似于以下命令的命令：

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very"
```

该命令显示类似于以下内容的输出：

```text
秘密短语 `caution juice atom organ advance problem want pledge someone senior holiday very` 是帐户：
  秘密种子：       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  公钥（十六进制）：  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  公钥（SS58）： 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  帐户 ID：        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 地址：      5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

`subkey` 程序假设地址基于公钥/私钥对。
如果您检查地址，则该命令将返回 32 字节的帐户标识符。
但是，并非基于 Substrate 的网络中的所有地址都基于密钥。

根据您指定的命令行选项和您提供的输入，命令输出还可能显示已编码地址的网络。
例如：

```bash
subkey inspect "12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU"
```

该命令显示类似于以下内容的输出：

```text
公钥 URI `12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU` 是帐户：
  网络 ID/版本： polkadot
  公钥（十六进制）：   0x46ebddef8cd9bb167dc30878d7113b7e168e6f0646beffd77d69d39bad76b47a
  帐户 ID：         0x46ebddef8cd9bb167dc30878d7113b7e168e6f0646beffd77d69d39bad76b47a
  公钥（SS58）：  12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU
  SS58 地址：       12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU
```

### 使用 Polkadot-JS API

要在 JavaScript 或 TypeScript 项目中验证地址，您可以使用 Polkadot-JS API 中内置的函数。
例如：

```bash
// 导入 Polkadot.js API 依赖项。
const { decodeAddress, encodeAddress } = require('@polkadot/keyring')
const { hexToU8a, isHex } = require('@polkadot/util')

// 指定要测试的地址。
const address = '<addressToTest>'

// 检查地址。
const isValidSubstrateAddress = () => {
  try {
    encodeAddress(isHex(address) ? hexToU8a(address) : decodeAddress(address))

    return true
  } catch (error) {
    return false
  }
}

// 查询结果。
const isValid = isValidSubstrateAddress()
console.log(isValid)
```

如果函数返回 `true`，则您指定的地址是有效地址。

### 其他 SS58 实现

在其他几种语言和库中都实现了对编码和解码 Substrate SS58 地址的支持。

- Crystal：[`wyhaines/base58.cr`](https://github.com/wyhaines/base58.cr)
- Go：[`itering/subscan`](https://github.com/subscan-explorer/subscan-essentials/blob/master/util/ss58/ss58.go)
- Python：[`polkascan/py-scale-codec`](https://github.com/polkascan/py-scale-codec/blob/master/scalecodec/utils/ss58.py)
- Typescript：[`subsquid/squid-sdk`](https://github.com/subsquid/squid-sdk/tree/master/substrate/ss58-codec)
