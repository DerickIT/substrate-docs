---
title: 在 Linux 上安装
description:
keywords:
---

在 Linux 上安装 Rust 最简单的方法是使用 [Rustup](https://rustup.rs/)。
Rustup 是一个命令行工具，它使您可以轻松地安装、管理和更新 Rust 编译器和工具链。
要安装 Rustup，请打开终端并运行以下命令：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

该命令将下载 Rustup 安装程序并将其运行。
安装程序将提示您选择要安装的 Rust 版本。
建议选择最新的稳定版本。
安装完成后，您需要关闭并重新打开终端以使更改生效。
您现在可以使用 `rustc` 命令来验证 Rust 编译器是否已安装。

```bash
rustc --version
```

该命令应显示已安装的 Rust 编译器的版本。
<environment_details>
# VSCode Visible Files
../../../response_1/0
../../../response_1/0
content/md/cn/docs/install/linux.md

# VSCode Open Tabs
content/md/cn/docs/build/application-development.md
content/md/cn/docs/community/article-bounties.md
content/md/cn/docs/community/content-style-guide.md
content/md/cn/docs/community/contributor-guidelines.md
content/md/cn/docs/community/index.md
content/md/cn/docs/community/template-how-to-guide.md
content/md/cn/docs/deploy/deploy-on-a-test-network.md
content/md/cn/docs/deploy/index.md
content/md/cn/docs/deploy/keys-and-network-operations.md
content/md/cn/docs/deploy/prepare-to-deploy.md
content/md/cn/docs/deploy/deployment-options.md
content/md/cn/docs/design/decide-what-to-build.md
content/md/cn/docs/design/index.md
content/md/cn/docs/examples/quickstart/index.html
content/md/cn/docs/install/developer-tools.md
content/md/cn/docs/install/index.md
content/md/cn/docs/design/substrate-empowers-builders.md
content/md/cn/docs/build/chain-spec.md
content/md/cn/docs/build/custom-pallets.md
content/md/cn/docs/build/events-and-errors.md
content/md/cn/docs/build/genesis-configuration.md
content/md/cn/docs/build/index.md
content/md/cn/docs/build/libraries.md
content/md/cn/docs/build/pallet-coupling.md
content/md/cn/docs/build/randomness.md
content/md/cn/docs/build/remote-procedure-calls.md
content/md/cn/docs/build/origins.md
content/md/cn/docs/build/runtime-storage.md
content/md/cn/docs/build/smart-contracts-strategy.md
content/md/cn/docs/build/troubleshoot-your-code.md
content/md/cn/docs/build/tx-weights-fees.md
content/md/cn/docs/build/build-a-deterministic-runtime.md
content/md/cn/docs/build/build-process.md
</environment_details>
