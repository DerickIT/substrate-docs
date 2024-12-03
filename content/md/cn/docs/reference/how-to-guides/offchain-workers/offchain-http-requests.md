---
title: 进行链下 HTTP 请求
description: 说明如何使用链下工作者发出 HTTP 请求。
keywords:
  - 链下工作者
  - ocw
  - http
  - https
  - 请求
---

<div class="warning">
	 请参考 <a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_offchain_workers/index.html">Rust文档</a> 获取有关此主题的最新文档。
</div>

因为大多数区块链无法访问其自身网络之外的服务器上托管的数据，所以它们通常使用外部第三方服务（**预言机**）从网络外部的位置提取信息或将信息推送到网络外部的位置。
对于基于 Substrate 的区块链，**链下工作者** (OCW) 提供类似的功能，但具有能够访问链上状态的优势。

本指南说明如何使用链下工作者使用 GET 或 POST 方法发出 HTTP 请求。
在本指南中的示例中，您将看到如何从 `cryptocompare` API 检索比特币的价格以及如何使用链下工作者 API 提交数据。

您可能知道 Rust 提供了它自己的用于发出 HTTP 请求的库。
但是，链下工作者在其自身的 WebAssembly 执行环境（[no-std](https://docs.rust-embedded.org/book/intro/no-std.html) 环境）中运行，因此无法访问标准 Rust 库。
相反，Substrate 提供了您自己可以用来发出 HTTP 请求的库。

Substrate HTTP 库支持以下方法：

- GET
- POST
- PUT
- PATCH
- DELETE

## 设置截止时间并实例化 HTTP 请求

在大多数情况下，您希望限制链下工作者执行其操作所允许的时间。
对于此示例，您可以设置一个硬编码的截止时间为两秒钟以完成外部调用。
您也可以无限期地等待响应。
但是，无限期地等待可能会导致外部主机超时。

1. 创建一个 2 秒的截止时间。

   ```rust
   let deadline = sp_io::offchain::timestamp().add(Duration::from_millis(2_000));
   ```

1. 启动外部 HTTP GET 请求。

   ```rust
   let request = http::Request::get("https://min-api.cryptocompare.com/data/price?fsym=BTC&tsyms=USD");
   let pending = request.deadline(deadline).send().map_err(|_| http::Error::IoError)?;
   let response = pending.try_wait(deadline).map_err(|_| http::Error::DeadlineReached)??;
   ```

## 读取并提交响应

1. 检查响应状态代码。

   ```rust
   // 在继续读取响应之前，让我们检查状态代码。
   if response.code != 200 {
     log::warn!("意外的状态代码：{}", response.code);
     return Err(http::Error::Unknown)
   }
   ```

1. 读取响应。

   ```rust
   let body = response.body().collect::<Vec<u8>>();
   // 从正文创建 str 切片。
   let body_str = sp_std::str::from_utf8(&body).map_err(|_| {
     log::warn!("没有 UTF8 正文");
     http::Error::Unknown
   })?;
   ```

1. 使用 POST 请求将数据提交到 API。

   ```rust
    // 发送 POST 请求
   let request_body = Vec::new();
   let request = http::Request::post("https://reqres.in/api/users", vec![request_body.clone()])
     .add_header("x-api-key", "test_api_key")
     .add_header("content-type", "application/json");

   let pending = request
     .deadline(deadline)
     .body(vec![request_body.clone()])
     .send()
     .map_err(|_| http::Error::IoError)?;

   // 等待响应
   let response = pending
     .try_wait(deadline)
     .map_err(|_| http::Error::DeadlineReached)??;
   ```

## 示例

- [示例模块：链下工作者](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs)
- [演示：OCW 模块](https://github.com/jimmychu0807/substrate-offchain-worker-demo/blob/master/pallets/ocw/src/lib.rs#L363-#L401)
- [来源：Substrate 核心原语](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/primitives/runtime/src/offchain/http.rs#L63-L76)
