---
title: 触发迁移
desciption: 使用 Polkadot-JS Apps 触发存储迁移
keywords:
  - 存储迁移
  - 运行时
  - 高级
---

本简要指南介绍了使用 [Polkadot-JS Apps](https://polkadot.js.org/apps/) 触发运行时迁移的步骤。它假设迁移代码已编写完毕，并且新运行时已编译完毕。


## 添加自定义类型

将新的存储类型放入 `types.json` 中，您需要使用 UI 来触发迁移。我们的新 JSON 类型如下：

```rust
{
  "Nickname": {
    "first": "BoundedVec<u8>",
    "last": "Option<BoundedVec<u8>>"
  }
}
```
在 Polkadot-js Apps UI 中，转到设置 > 开发人员以从 types.json 添加自定义类型。您可以直接上传文件，也可以将类型直接粘贴到 UI 中。保存以添加它们。

## 增加 spec_version

增加 `spec_version` 以指定新的运行时版本。
在 runtime/src/lib.rs 中找到 `runtime_version` 宏。
```rust
 #[sp_version::runtime_version]
 pub const VERSION: RuntimeVersion = RuntimeVersion {
    //--snip
    spec_version: 100,
    //--snip
 }
```
增加 `spec_version` 以指定新的运行时版本。
```rust
    spec_version: 101,
```

## 上传运行时

使用 `cargo build --release` 重新编译您的运行时，这将生成更适合提交到区块链网络的较小构建工件。
WebAssembly 构建工件位于 target/release/wbuild/node-template-runtime 目录中。例如，您应该看到以下 WebAssembly 工件：
```
    node_template_runtime.compact.compressed.wasm
    node_template_runtime.compact.wasm
    node_template_runtime.wasm
```
您现在拥有一个描述修改后的运行时逻辑的 WebAssembly 工件。要完成升级，您需要提交一个事务来更新节点以使用升级后的运行时：

转到 `开发者 > 外部函数`，使用管理帐户，选择 `sudo` pallet 和 `sudoUncheckedWeight(call, weight)` 函数。
选择 `system` 和 `setCode(code)` 作为要进行的调用
点击文件上传，然后选择或拖放您为更新的运行时生成的紧凑型和压缩型 WebAssembly 文件（`node_template_runtime.compact.compressed.wasm`）（例如，`./target/release/wbuild/node-template-runtime/node_template_runtime.compact.wasm`）。

提交事务，并在事务包含在区块中后，您将能够看到节点模板版本号指示运行时版本现在为 101。

如果您使用旧存储创建了 Nickname，您现在将能够查询链状态并查看此存储已更新为新的结构，并且您现在将能够使用名字和姓氏创建 Nickname。
