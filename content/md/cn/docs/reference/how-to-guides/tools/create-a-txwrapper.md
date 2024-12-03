---
title: 为链创建 txwrapper
description: 扩展链用户的离线签名选项。
keywords:
  - 运行时
  - 工具
---

创建 `txwrapper` 包将扩展链用户的离线签名选项。
这对于需要使用隔离设备（例如）促进交易签名、构建和/或解码的安全意识用户非常重要。
这包括（但不限于）托管人、交易所和冷存储用户。

在为自己的链构建 `txwrapper` 之前，请查看 [`txwrapper-examples`](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-examples/README.md)。

确保您理解 Polkadot 示例，并查看最终用户应使用的 `txwrapper-core` 方法（请参阅 `decode`、`construct.{signingPayload, signedTx, txHash}`）。
您的包将重新导出这些方法，因此请务必了解您将创建的公共 API。

## 目标

构建链的 `txwrapper` 包的公共 API。

## 用例

方便现有 `txwrapper` 用户轻松集成新的 txwrapper。

## 步骤

### 1. 使用 `txwrapper-template` 创建一个仓库

将 [`txwrapper-template`](https://github.com/paritytech/txwrapper-core/tree/main/packages/txwrapper-template) 目录复制到您的工作仓库中。

该模板提供了几乎可以发布到 `NPM` 的 typescript 包的基础知识。导出显示了一些与使用至少 `balances`、`proxy` 和 `utility pallets` 的基于 FRAME 的链相关的有用方法。

请注意，[`txwrapper-core\`](https://github.com/paritytech/txwrapper-core) 在顶层重新导出，以便用户可以访问其工具。

### 2. 更新 package.json

修改以下字段以反映您的链信息：

- name
- author
- description
- repository
- bugs
- homepage
- private（标记为 false）

此外，添加以下字段以授予发布权限：

```js
  "publishConfig": {
    "access": "public"
  },
```

### 3. 选择要重新导出的相关方法

您需要选择您希望 `txwrapper` 公开的 pallet 方法。
建议选择可能由离线存储的密钥签名的那些方法。

如果您只需要 Substrate 或 ORML pallet 的方法，请查看 [txwrapper-substrate](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-substrate/README.md) 和 [txwrapper-orml](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-orml/README.md) 以查看这些方法是否已定义。

### 4. 创建 getRegistry 方法

您的 txwrapper 需要导出 `getRegistry` 方法，以便用户可以使用最新的链类型获取 Polkadot-js `TypeRegistry`。

通过一些小的修改，下面的 `foo` 示例可以应用于与 Polkadot-js 类型兼容的任何基于 `FRAME` 的链：

```js
// src/index.ts

import { typesBundleForPolkadot } from '@foo-network/type-definitions';
import { OverrideBundleType } from '@polkadot/types/types';
import {
  getRegistryBase,
  GetRegistryOptsCore,
  getSpecTypes,
  TypeRegistry,
} from '@substrate/txwrapper-core';

// 为了方便用户，我们可以为他们提供硬编码的链属性，
// 因为这些属性很少改变。
/**
 * txwrapper-foo 支持的网络的 `ChainProperties`。这些通常由 `system_properties` 调用返回，
 * 但由于它们变化不大，因此硬编码它们是相当安全的。
 */
const KNOWN_CHAIN_PROPERTIES = {
  foo: {
    ss58Format: 3,
    tokenDecimals: 18,
    tokenSymbol: 'FOO',
  },
  bar: {
    ss58Format: 42,
    tokenDecimals: 18,
    tokenSymbol: 'FOO',
  },
};

// 我们覆盖 `GetRegistryOptsCore` 的 `specName` 属性以获得更窄的类型特异性，
// 有望为用户创造更好的体验。
/**
 * `getRegistry` 函数的选项。
 */
export interface GetRegistryOpts extends GetRegistryOptsCore {
  specName: keyof typeof KNOWN_CHAIN_PROPERTIES;
}

/**
 * 获取 txwrapper-foo 支持的网络的类型注册表。
 *
 * @param GetRegistryOptions 当前运行时的 specName、chainName、specVersion 和 metadataRpc
 */
export function getRegistry({
  specName,
  chainName,
  specVersion,
  metadataRpc,
  properties,
}: GetRegistryOpts): TypeRegistry {
  const registry = new TypeRegistry();
  registry.setKnownTypes({
    // 如果您的类型不是以 `OverrideBundleType` 格式打包的，则可以
    // 使用 `RegisteredTypes` 支持的任何格式指定类型：
    // https://github.com/polkadot-js/api/blob/4ff9b51af2c49294c676cc80abc6476565c70b11/packages/types/src/types/registry.ts#L59
    typesBundle: (typesBundleForPolkadot as unknown) as OverrideBundleType,
  });

  return getRegistryBase({
    chainProperties: properties || KNOWN_CHAIN_PROPERTIES[specName],
    specTypes: getSpecTypes(registry, chainName, specName, specVersion),
    metadataRpc,
  });
}
```

并添加相关的导出：

```js
// src/methods/currencies/index.ts

// 导出方法，有效地在 `currencies` 命名空间下可用
export * from "./transfer";
// src/methods

// 导出 `methods` 中的所有内容，包括 `currencies` 命名空间，使其能够
// 通过 `methods.currencies.transfer` 访问该方法
export * as methods from "./methods";
```

### 5. 创建一个可工作的示例

一个好的示例可以减轻用户的摩擦并减少维护人员的工作量。
创建一个端到端示例，以便用户清楚地了解为您的链生成离线交易的完整流程。

1. 将 `template-example.ts` 重命名为适合您链的名称，并更新文件中标记为 TODO 的所有部分。
2. 更新示例/README.md 中标记为 TODO 的部分。
3. 确保您可以使用链的开发节点运行该示例。

### 6. 发布您的包

一旦您确保版本控制有意义并且[包在本地工作](https://docs.npmjs.com/cli/v6/commands/npm-pack)后，请参考[本指南](https://docs.npmjs.com/cli/v6/commands/npm-publish)了解如何将您的包发布到 `NPM`。

## 示例

- [模板示例](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-template/examples/template-example.ts)

## 资源

- 如何使用 [`tx-wrapper-polkadot`](https://github.com/paritytech/txwrapper-core/blob/main/packages/)
- 使用 `jest` 进行序列化/反序列化[单元测试](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-orml/src/methods/currencies/transfer.spec.ts)
