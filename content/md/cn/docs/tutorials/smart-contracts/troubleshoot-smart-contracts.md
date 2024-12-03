---
title: 对智能合约进行故障排除
description: 对编写和部署智能合约时可能遇到的常见问题进行故障排除。
keywords:
---

本节描述了在基于 Substrate 的区块链上编写和部署智能合约时可能遇到的一些常见问题以及如何解决这些问题。

## 意外的纪元更改

如果您在没有正确停止节点的情况下中断运行中的节点（例如，关闭终端或计算机进入睡眠模式），则您可能会看到以下错误：

```bash
ClientImport("Unexpected epoch change")
```

如果您看到此错误，请使用以下命令重新启动节点：

```bash
 substrate-contracts-node --log info,runtime::contracts=debug 2>&1
```

此命令将清除所有正在运行的节点状态。
重新启动节点后，请重复您在节点关闭之前执行的任何步骤。
例如，重新部署您先前上传的任何合约。
