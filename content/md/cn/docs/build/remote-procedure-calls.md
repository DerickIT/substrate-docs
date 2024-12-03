---
title: 远程过程调用
description: 描述如何使用远程过程调用和 RPC 方法与 Substrate 节点交互。
keywords:
  - rpc
  - 前端
---

<div class="warning">
  请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/custom_runtime_api_rpc/index.html">Rust 文档</a> 获取有关此主题的最新文档。
</div>

远程过程调用或 RPC 方法是外部程序（例如浏览器或前端应用程序）与 Substrate 节点通信的一种方式。
通常，这些方法使 RPC 客户端能够连接到 RPC 服务器端点以请求某种类型的服务。
例如，你可能使用 RPC 方法来读取存储的值、提交事务或请求有关节点连接到的链的信息。

访问 Substrate 节点的默认 [JSON-RPC 方法](https://polkadot.js.org/docs/substrate/rpc/) 的最便捷方式是通过 [Polkadot-JS API](https://polkadot.js.org/docs/api/)。

## 安全和不安全的 RPC 方法

务必注意，RPC 方法可以访问核心节点操作（包括共识和存储），并且还可以作为公共接口公开，以允许外部用户向区块链提交事务或从中检索信息。
因此，为了区块链的安全，务必考虑不同的 RPC 方法公开的内容以及是否应将其限制为在本地节点上运行或公开提供。

### 公共 RPC 接口

Substrate 节点提供以下命令行选项，允许你公开 RPC 接口：

```bash
--ws-external
--rpc-external
--unsafe-ws-external
--unsafe-rpc-external
```

默认情况下，如果你尝试公开 RPC 接口并同时运行验证者节点，则节点将拒绝启动。
`--unsafe-*` 标志允许你取消此安全措施。
公开 RPC 接口可能会打开大量的攻击面，必须仔细审查。

有很多 RPC 方法可用于控制节点的行为，但你应该避免公开这些方法。
例如，你不应公开以下 RPC 方法：

- [`submit_extrinsic`](https://paritytech.github.io/substrate/master/sc_rpc_api/author/trait.AuthorApiClient.html) - 允许将事务提交到本地池。
- [`insert_key`](https://paritytech.github.io/substrate/master/sc_rpc_api/author/trait.AuthorApiClient.html) - 允许将私钥插入到本地密钥库中。
- [`rotate_keys`](https://paritytech.github.io/substrate/master/sc_rpc_api/author/trait.AuthorApiClient.html) - 会话密钥轮换。
- [`remove_extrinsic`](https://paritytech.github.io/substrate/master/substrate_rpc_client/trait.AuthorApi.html#method.remove_extrinsic) - 从池中删除并禁止外部函数。
- [`add_reserved_peer`](https://paritytech.github.io/substrate/master/sc_rpc_api/system/trait.SystemApiClient.html) - 添加保留节点。
- [`remove_reserved_peer`](https://paritytech.github.io/substrate/master/sc_rpc_api/system/trait.SystemApiClient.html) - 删除保留节点。

你还应该避免公开执行时间可能很长的 RPC 方法，这可能会阻止客户端同步。
例如，你应该避免使用以下 RPC 方法：

- [`storage_keys_paged`](https://paritytech.github.io/substrate/master/sc_rpc_api/state/trait.StateApiClient.html) - 获取状态中具有特定前缀的所有密钥以及分页支持。
- [`storage_pairs`](https://paritytech.github.io/substrate/master/sc_rpc_api/state/trait.StateApiClient.html) - 获取状态中具有特定前缀的所有密钥及其值。

这些 RPC 是通过使用 `#[rpc(name = "rpc_method")]` 宏声明的，其中 `rpc_method` 是函数的名称，例如 `submit_extrinsic`。

如果请求来自不受信任的用户，则务必过滤掉此类调用。
实现此目的的方法是通过能够检查调用并仅传递允许的 API 调用集的 [JSON-RPC](/reference/glossary#json-rpc) 代理。

## 用于 remote_externalities 的 RPC

Substrate 还提供了一些专门的 RPC 方法来调用节点的 [`remote_externalities`](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/utils/frame/remote-externalities/src/lib.rs#L347-#L746)。
这些用于远程外部函数的专门方法使你能够向 Substrate 节点发出一次性 RPC 调用，以获取有关块和头的信息。
这些调用返回的信息对于使用 [`try-runtime`](/reference/command-line-tools/try-runtime/) 等工具进行测试很有用。

## 端点

在本地启动 Substrate 节点时，默认情况下只有一个端点可用：

- HTTP 和 WebSocket 端点：`ws://localhost:9944/`

大多数 Substrate 前端库和工具都使用该端点与区块链交互。
例如，如果你使用 Polkadot-JS 应用程序连接到本地节点或公共链，则通常连接到 HTTP 和 WebSocket 端点。
WebSocket 连接允许前端应用程序和响应请求的后端节点之间进行双向通信。
但是，你还可以通过使用 `curl` 命令连接到端点来单独调用 RPC 方法，而无需保持打开的通信通道。
例如，你可以使用 curl 命令获取系统信息或订阅链以接收有关块状态发生特定类型更改时的通知。

要使用端点调用 RPC 方法：

1. 打开终端 shell 并更改到 Substrate 节点模板的根目录。

2. 通过运行以下命令在开发模式下在本地启动节点：

   ```bash
   ./target/release/node-template --dev
   ```

3. 通过运行以下命令连接到本地节点并调用 `rpc_methods` 端点：

   ```bash
   curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "rpc_methods"}' http://localhost:9944/
   ```

   此命令将返回为本地节点公开的 JSON-RPC 方法列表。

4. 使用适当的方法名称调用其他方法。

   例如，你可以运行以下命令来获取有关本地节点的版本信息：

   ```bash
   curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_version"}' http://localhost:9944/
   ```

   在大多数情况下，直接连接到 RPC 端点会返回 JSON 格式的结果。
   例如：

   ```bash
   {"jsonrpc":"2.0","result":"4.0.0-dev-de262935ede","id":1}
   ```

为了使返回值更易于阅读，你可以使用 SCALE 编解码器对其进行解码。
有关编码和解码信息的更多信息，请参阅 [类型编码 (SCALE)](/reference/scale-codec/)。

每个存储项都有一个与其关联的相对存储密钥，该密钥用于[查询存储](/main-docs/build/runtime-storage#querying-storage)。
这就是 RPC 端点知道在哪里查找的方式。

## 示例

### state_getMetadata

要获取本地节点的元数据，你可以运行以下命令：

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getMetadata"}' http://localhost:9944/
```

此命令将以十六进制编码字节而不是人类可读的格式返回元数据。
此 RPC 请求的 JavaScript 代码如下所示：

```javascript
function get_metadata_request(endpoint) {
  let request = new Request(endpoint, {
    method: "POST",
    body: JSON.stringify({
      id: 1,
      jsonrpc: "2.0",
      method: "state_getMetadata",
    }),
    headers: { "Content-Type": "application/json" },
  });
  return request;
}
```

简单的文本解码：

```javascript
function decode_metadata(metadata) {
  return new TextDecoder().decode(util.hexToU8a(metadata));
}
```

### `state_getStorage`

RPC 请求：

```json
Request:   {"id":1,"jsonrpc":"2.0","method":"state_getStorage",["{storage_key}"]}
```

其中 `storage_key` 是由模块、函数和密钥（可选）的名称生成的参数：

```javascript
function get_runtime_storage_parameter_with_key(module_name, function_name, key) {
  // 我们对运行时开发者可以控制的字符串使用 xxhash 128
  let module_hash = util_crypto.xxhashAsU8a(module_name, 128);
  let function_hash = util_crypto.xxhashAsU8a(function_name, 128);

  // 我们对最终用户可以控制的字符串使用 blake2 256
  let key_hash = util_crypto.blake2AsU8a(keyToBytes(key));

  // 特殊语法用于连接 Uint8Array
  let final_key = new Uint8Array([...module_hash, ...function_hash, ...key_hash]);

  // 返回十六进制字符串
  return util.u8aToHex(final_key);
}
```

## 接下来的步骤

- [JSON-RPC 的 Rust 实现](https://github.com/paritytech/jsonrpc)
- [类型编码 (SCALE)](/reference/scale-codec)
- [运行时存储](/main-docs/build/runtime-storage/)
