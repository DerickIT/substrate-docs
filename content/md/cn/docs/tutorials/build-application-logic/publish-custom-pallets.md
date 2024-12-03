---
title: 发布自定义模块
description: 建议如何发布自定义模块和crates，以使它们可供更广泛的社区使用。
keywords:
---

作为区块链构建者或平行链开发者，您可以访问大型的专用模块库，您可以将这些模块组合起来以自定义运行时，以满足您特定的项目目标。
要大致了解当前可用的预定义模块数量，请浏览 Substrate 的[FRAME 存储库](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame)。
可用的模块选择及其解决的用例正在不断发展，其中包括许多来自社区成员的贡献。

当您开始编写自己的自定义模块来执行特定于应用程序的逻辑时，您可能会发现您的自定义逻辑会使其他团队或整个生态系统受益。
如果您有要与社区共享的自定义模块，则可以在公共论坛（例如 GitHub 或 crates.io）上发布它们。

本教程总结了发布自定义模块并将其作为开源项目供其他开发者使用的步骤。

## 在 GitHub 上发布

要在 GitHub 上发布您的模块：

1. [创建 GitHub 存储库](https://help.github.com/en/articles/create-a-repo)。

   为存储库可见性选择**公共**。

1. [将模块的所有源代码](https://help.github.com/en/articles/pushing-to-a-remote)推送到远程存储库。

1. 添加一个[README](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)文件，其中描述了模块的功能和用法。

1. 根据需要向存储库添加 LICENSE、CODE-OF-CONDUCT、CONTRIBUTING 或其他文件。

发布模块后，其他开发者可以通过在 `Cargo.toml` 文件中包含它来将其导入到他们的运行时中，如下所示：

1. 打开终端 shell 并更改到节点模板的根目录。

1. 在文本编辑器中打开 `runtime/Cargo.toml` 配置文件。

1. 找到[依赖项]部分。

1. 添加类似于以下内容的行：

   ```toml
   your-pallet-name = { version = "1.0.0", default-features = false, git = "https://github.com/<your-organization-name>/<your-pallet-repo-name>", branch = "<default-or-specific-branch-name" }
   ```

   如果您希望开发者使用特定的标签或提交来导入模块，则应在 README 中包含此信息。

## 在 crates.io 上发布

Rust 社区维护[crates.io](https://crates.io/)网站，允许无许可地发布任何 Rust 模块。
您可以按照他们的指南学习该过程，该指南介绍了如何在 crates.io 上[发布](https://doc.rust-lang.org/cargo/reference/publishing.html)。

发布模块后，其他开发者可以通过在 `Cargo.toml` 文件中包含它来将其导入到他们的运行时中，如下所示：

1. 打开终端 shell 并更改到节点模板的根目录。

1. 在文本编辑器中打开 `runtime/Cargo.toml` 配置文件。

1. 找到[依赖项]部分。

1. 添加类似于以下内容的行：

   ```toml
   your-pallet-name = { version = "<compatible-version>", default-features = false }
   ```

   如果您在 crates.io 上发布模块，开发者无需指定目标目的地。
   默认情况下，cargo 会在 crates.io 注册表中搜索指定的包。

## 下一步

- [运行时开发](/learn/runtime-development/)
- [自定义模块](/build/custom-pallets/)
- [在自定义模块中使用宏](/tutorials/build-application-logic/use-macros-in-a-custom-pallet/)
- [方法：导入模块](/reference/how-to-guides/basics/import-a-pallet/)。
