---
title: 类型编码 (SCALE)
description:
keywords:
---

Substrate 使用轻量级且高效的编码和解码程序来优化如何在网络上传输和接收数据。
用于序列化和反序列化数据程序称为 SCALE 编解码器，其中 SCALE 是<u>简</u>单<u>串</u>联<u>聚</u>合<u>小</u>端序的缩写。

SCALE 编解码器是运行时和外部节点之间通信的关键组件。

它专为在资源受限的执行环境（如 Substrate [WebAssembly 运行时](/learn/runtime-development//)）中高效、无复制地编码和解码数据而设计。

SCALE 编解码器本身没有任何描述性信息。
它假设解码上下文具有有关编码数据的所有类型知识。
Parity 维护的前端库使用 [`parity-scale-codec`](https://github.com/paritytech/parity-scale-codec) crate（这是 SCALE 编解码器的 Rust 实现）来编码和解码 RPC 与运行时之间的交互。

SCALE 编解码器对 Substrate 和区块链系统有利，因为：

- 相对于 [serde](https://serde.rs/) 等通用序列化框架，它很轻量级，后者会添加大量样板代码，从而增加二进制文件的大小。
- 它不使用 Rust `libstd`，使其与编译到 Wasm 的 `no_std` 环境兼容，例如 Substrate 运行时。
- 它在 Rust 中内置了对使用以下方法为新类型派生编解码器逻辑的强大支持：`#[derive(Encode, Decode)]`。

定义 Substrate 中使用的编码方案而不是重用现有的 Rust 编解码器库非常重要，因为此编解码器需要在想要支持 Substrate 区块链之间互操作性的其他平台和语言上重新实现。

下表显示了 Parity SCALE 编解码器的 Rust 实现如何编码不同类型。

**SCALE 编解码器不同类型的示例**

| 类型                          | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | SCALE 解码值的示例                                                                                         | SCALE 编码值                  |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| 定宽整数          | 基本整数使用定宽小端序 (LE) 格式进行编码。                                                                                                                                                                                                                                                                                                                                                                                                            | `带符号 8 位整数 69`                                                                                           | `0x45`                               |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `无符号 16 位整数 42`                                                                                        | `0x2a00`                             |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `无符号 32 位整数 16777215`                                                                                  | `0xffffff00`                         |
| 紧凑/通用整数[^1]  | “紧凑”或通用整数编码足以对大整数（高达 2\*\*536）进行编码，并且比定宽版本更有效地编码大多数值。（但对于单字节值，定宽整数永远不会更差。）                                                                                                                                                                                                                                                                                                                           | `无符号整数 0`                                                                                                | `0x00`                               |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `无符号整数 1`                                                                                                | `0x04`                               |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `无符号整数 42`                                                                                               | `0xa8`                               |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `无符号整数 69`                                                                                               | `0x1501`                             |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `无符号整数 65535`                                                                                            | `0xfeff0300`                         |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `BigInt(100000000000000)`                                                                                           | `0x0b00407a10f35a`                   |
| 布尔值                       | 布尔值使用单个字节的最低有效位进行编码。                                                                                                                                                                                                                                                                                                                                                                                                         | `false`                                                                                                             | `0x00`                               |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `true`                                                                                                              | `0x01`                               |
| 结果 [^2]                  | 结果通常是枚举，指示某些操作是成功还是失败。                                                                                                                                                                                                                                                                                                                                                                    | `Ok(42)`                                                                                                            | `0x002a`                             |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `Err(false)`                                                                                                        | `0x0100`                             |
| 选项                       | 特定类型的零个或一个值。                                                                                                                                                                                                                                                                                                                                                                                                                                             | `Some`                                                                                                              | `0x01` 后跟编码值 |
|                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `None`                                                                                                              | `0x00` |
| 向量（列表、序列、集合） | 同类型值的集合进行编码，前面带有项数的_紧凑_编码，然后是依次连接的每个项的编码。                                                                                                                                                                                                                                                                                                                      | 无符号 16 位整数向量：`[4, 8, 15, 16, 23, 42]`                                                        | `0x18040008000f00100017002a00`       |
| 字符串                       | 字符串是包含有效 UTF8 序列的字节向量 (`Vec<u8>`)。                                                                                                                                                                                                                                                                                                                                                                                                           |                                                                                                                     |                                      |
| 元组                        | 固定大小的值序列，每个值可能具有不同的但预定的和固定的类型。这只是每个编码值的串联。                                                                                                                                                                                                                                                                                                                              | 紧凑无符号整数和布尔值的元组：`(3, false)`                                                         | `0x0c00`                             |
| 结构体                       | 对于结构体，值是有名称的，但这与编码无关（名称被忽略 - 只有顺序很重要）。所有容器都连续存储元素。元素的顺序不是固定的，取决于容器，解码时不可靠。这意味着将一些字节数组解码为强制执行顺序的指定结构体，然后对其进行重新编码，可能会导致与原始解码的字节数组不同的字节数组。 | 始终按升序排列字节元素的 `SortedVecAsc<u8>` 结构体：`SortedVecAsc::from([3, 5, 2, 8])` | `[3, 2, 5, 8]`                       |
| 枚举（标记联合）  | 固定数量的变体，每个变体都是互斥的，并且可能暗示进一步的值或一系列值。编码为第一个字节，标识值为该变体的索引。任何其他字节都用于编码变体暗示的任何数据。因此，最多支持 256 个变体。                                                                                                                                                                                          | `Int(42)` 和 `Bool(true)`，其中 `enum IntOrBool { Int(u8), Bool(bool),}`                                           | `0x002a` 和 `0x0101`                |

[^1]: 紧凑/通用整数使用两位最低有效位表示模式：

    - `0b00`：单字节模式；高六位是值的 LE 编码（仅对 0-63 的值有效）。
    - `0b01`：两字节模式：高六位和下一个字节是值的 LE 编码（仅对值 `64-(2**14-1)` 有效）。
    - `0b10`：四字节模式：高六位和接下来的三个字节是值的 LE 编码（仅对值 `(2**14)-(2**30-1)` 有效）。
    - `0b11`：大整数模式：高六位是后续字节数加四。
      该值包含在后续字节中，以 LE 编码。最后一个（最高有效）字节必须是非零值。仅对值 `(2**30)-(2**536-1)` 有效。

[^2]: 结果编码为：

    - 如果操作成功，则为 `0x00`，后跟编码值。
    - 如果操作失败，则为 `0x01`，后跟编码错误。

SCALE 编解码器已在其他语言中实现，包括：

- AssemblyScript：[`LimeChain/as-scale-codec`](https://github.com/LimeChain/as-scale-codec)


- C：[`MatthewDarnell/cScale`](https://github.com/MatthewDarnell/cScale)
- C++：[`soramitsu/scale-codec-cpp`](https://github.com/soramitsu/scale-codec-cpp)
- JavaScript：[`polkadot-js/api`](https://github.com/polkadot-js/api)
- Dart：[`leonardocustodio/polkadart`](https://github.com/leonardocustodio/polkadart)
- Haskell：[`airalab/hs-web3`](https://github.com/airalab/hs-web3/tree/master/packages/scale)
- Golang：[`itering/scale.go`](https://github.com/itering/scale.go)
- Java：[`emeraldpay/polkaj`](https://github.com/emeraldpay/polkaj)
- JavaScript：[`polkadot-js/api`](https://github.com/polkadot-js/api)
- Python：[`polkascan/py-scale-codec`](https://github.com/polkascan/py-scale-codec)
- Ruby：[`wuminzhe/scale_rb`](https://github.com/wuminzhe/scale_rb)
- TypeScript：[`parity-scale-codec-ts`](https://github.com/paritytech/parity-scale-codec-ts), [`scale-ts`](https://github.com/unstoppablejs/unstoppablejs/tree/main/packages/scale-ts#scale-ts), [`soramitsu/scale-codec-js-library`](https://github.com/soramitsu/scale-codec-js-library), [`subsquid/scale-codec`](https://github.com/subsquid/squid-sdk/tree/master/substrate/scale-codec)
