---
title: srtool
description: 描述用于构建 Substrate WebAssembly 运行时的 Docker 容器和命令行界面。
keywords:
---

Substrate 运行时工具箱 (`srtool`) 的核心组件是一个 Docker 容器，它使您能够以确定性方式构建 Substrate WebAssembly 运行时。
通过使用此工具，您可以确保相同的源代码始终如一地生成相同的 WebAssembly blob。
您还可以使用该工具检查和审计任何基于 Substrate 的链的运行时，并将构建 WebAssembly 运行时集成到您的 CI/CD 管道中。

## 使用 Docker 容器

因为 `srtool` 是一个 Docker 容器，所以您必须在构建环境中使用它才能使用它。
但是，您不需要了解有关使用 Docker 的任何知识来使用 `srtool` 构建链，因为您可以使用 `srtool-cli` 命令行界面来使用 Docker 镜像。

`srtiool-cli` 包是一个用 Rust 编写的命令行实用程序，它在您的计算机上安装一个名为 `srtool` 的可执行程序。
该程序简化了您与 `srtool` Docker 容器的交互。
随着时间的推移，围绕 `srtool` Docker 镜像的工具已扩展到包括以下工具和辅助程序：

- [srtool-cli](https://github.com/chevdor/srtool-cli) 提供了一个命令行界面来拉取 `srtool` Docker 镜像，获取有关镜像和用于与其交互的工具的信息，以及使用 `srtool` Docker 容器构建运行时。

- [subwasm](https://github.com/chevdor/subwasm) 提供了用于处理使用 `srtool` 构建的元数据和 WebAssembly 运行时的命令行选项。
  `subwasm` 程序也用于在 `srtool` 镜像中执行任务。

- [srtool-actions](https://github.com/chevdor/srtool-actions) 提供了 Github 操作，将使用 `srtool` 镜像生成的构建与您的 GitHub CI/CD 管道集成。

- [srtool-app](https://gitlab.com/chevdor/srtool-app) 为使用 `srtool` Docker 镜像构建运行时提供了一个简单的图形用户界面。

## 安装 srtool-cli

`srtool` 命令行界面使您能够使用 `srtool` Docker 镜像构建 WebAssembly 运行时。
您可以通过运行以下命令安装 `srtool` 命令行界面：

```bash
cargo install --git https://github.com/chevdor/srtool-cli
```

### 基本命令用法

运行 `srtool` 命令的基本语法是：

```text
srtool [选项] [子命令]
```

### 选项

您可以在 `srtool` 命令中使用以下命令行选项。

| 选项                      | 描述
| :---------------------------| :---------------------
| `-h`,&nbsp;`--help`           | 显示用法信息。
| `i`,&nbsp;`--image`&nbsp;`<image>` | 指定一个备用镜像。请务必指定与默认 `paritytech/srtool` 镜像兼容的镜像。您应该注意，指定不同的镜像可能不会产生与 `paritytech/srtool` 镜像产生的相同确定性结果。
| `-n`,&nbsp;`--no-cache`&nbsp; | 指定您不想使用本地缓存中的标签值。
| `-V`,&nbsp;`--version`&nbsp; | 显示版本信息。

### 子命令

您可以在 `srtool` 命令中使用以下子命令。

| 命令                    | 描述
|:-------------------------- |:-----------
| `build` | 启动一个新的 `srtool` 容器来构建您的运行时。
| `help` | 显示 `srtool` 或指定子命令的用法信息。
| `info` | 显示有关 `srtool` 容器和您的存储库的信息。
| `pull` | 拉取 `srtool` 镜像，而不运行其他任何操作。
| `version` | 显示 `srtool` 容器的版本信息。如果您想要 `srtool-cli` 可执行文件的版本信息，请使用 `--version`。

### 示例

要获取 `srtool` Docker 镜像的版本信息，请运行以下命令：

```bash
srtool version
```

该命令将显示类似以下内容的输出：

```text
{
  "name": "srtool",
  "version": "0.9.21",
  "rustc": "1.62.0",
  "subwasm": "0.18.0",
  "tera": "0.2.1",
  "toml": "0.2.1"
}
```

要获取 `srtool-cli` 可执行文件的版本信息，请运行以下命令：

```bash
srtool --version
```

该命令将显示类似以下内容的输出：

```text
srtool-cli 0.8.0
```

## srtool build

使用 `srtool build` 命令启动一个新的 `srtool` 容器来构建您指定的包的运行时。
默认情况下，`srtool build` 命令假设运行时的 `Cargo.toml` 文件位于名为链的 [`runtime`](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/runtime) 子目录中。
例如，`srtool build` 命令默认使用以下位置：

- runtime/kusama
- runtime/polkadot
- runtime/rococo
- runtime/westend

如果您的运行时的 `Cargo.toml` 文件位于其他位置，则可以将路径指定为命令行选项。

### 基本用法

运行 `srtool build` 命令的基本语法是：

```text
srtool build [选项] --package <package> [--runtime-dir <path>] [项目路径]
```

### 参数

默认情况下，`srtool build` 在当前工作目录中运行。如果您的项目未位于当前工作目录中，则可以指定项目位置的路径。

| 参数       | 描述
| :------------- | :-----------
| `项目路径` | 指定您要为其构建运行时的区块链项目的路径。

### 选项

您可以在 `srtool build` 命令中使用以下命令行选项。

| 选项            | 描述
| :---------------- | :-----------
| `-a`,&nbsp;`--app` | 在构建过程中启用标准输出和 JSON 输出的混合。此选项推荐用于 CI。JSON 输出作为构建结束时的单行提供。
| `--build-opts`&nbsp;`<BUILD_OPTS>` | 允许您将自定义选项直接传递给 `cargo` 构建过程。如果您指定此命令行选项，请注意，构建 Kusama 或 Polkadot 的所有自动选项都不会传递给构建过程。使用 `--build-opts` 命令行选项时，您必须显式设置所需的构建选项。通常，此选项很少需要。此选项等效于设置 `BUILD_OPTS` 环境变量。
| `--default-features`&nbsp;`<default-features>` | 允许您更改运行时的默认功能列表，而无需禁用自动功能检测。此选项等效于设置 `DEFAULT_FEATURES` 环境变量。如果您设置了 `BUILD_OPTS`，则此命令行选项无效。
| `-h`,&nbsp;`--help` | 显示用法信息。
| `i`,&nbsp;`--image`&nbsp;`<image>` | 指定一个备用镜像。请务必指定与默认 `paritytech/srtool` 镜像兼容的镜像。您应该注意，指定不同的镜像可能不会产生与 `paritytech/srtool` 镜像产生的相同确定性结果。
| `-j`,&nbsp;`--json` | 启用 JSON 输出。
| `--no-cache` | 禁用所有缓存。如果您指定此选项，则 `srtool` 镜像将不会访问 Cargo 主缓存以获取构建依赖项。通常，此选项很少使用，因为使用缓存没有已知问题。
| `-p`,&nbsp;`--package`&nbsp;`<package>` | 指定要构建的运行时包的名称。您指定的名称应与运行时的 `Cargo.toml` 文件中定义的名称相同，例如 kusama-runtime、polkadot-runtime 等等。此选项等效于设置 `PACKAGE` 环境变量。
| `--profile`&nbsp;`<profile>` | 指定用于构建运行时的配置文件，构建运行时的默认配置文件始终为 `release`。您可以使用此命令行选项覆盖默认值。此选项等效于设置 `PROFILE` 环境变量。
| `-r`,&nbsp;`--runtime-dir`&nbsp;`<runtime>` | 指定运行时的 `Cargo.toml` 文件的位置。如果您的运行时不在标准位置，则可以使用此命令行选项指定正确的位置。此选项等效于设置 `RUNTIME_DIR` 环境变量。
| `-V`,&nbsp;`--version` | 显示版本信息。

### 示例

要从 `cumulus` 存储库构建 Westend 运行时，其中运行时的 `Cargo.toml` 的路径为 [parachains/runtimes/assets/asset-hub-westend](https://github.com/paritytech/cumulus/tree/master/parachains/runtimes/assets/asset-hub-westend)，您将运行以下命令：

```bash
srtool build --app --package westmint-runtime --runtime-dir parachains/runtimes/assets/westmint
```

第一次运行 `srtool build` 命令时，需要一些时间才能完成。
在运行时编译时，有关其进度的消息将显示为标准输出。
因为此示例使用 `--app` 命令行选项，所以 JSON 输出将在构建结束时以单行显示，类似于以下截断的输出：

```text
...
   Compiling cumulus-primitives-parachain-inherent v0.1.0 (/build/primitives/parachain-inherent)
   Compiling cumulus-pallet-parachain-system v0.1.0 (/build/pallets/parachain-system)
    Finished release [optimized] target(s) in 112m 11s
✨ Your Substrate WASM Runtime is ready! ✨
{"gen":"srtool v0.9.21","src":"git","version":"1.0.0","commit":"bd41e3f11887ea2f55fc37be71ff652923388e03","tag":"v0.9.220-rc2","branch":"master","rustc":"rustc 1.62.0 (a8314ef7d 2022-06-27)","pkg":"westmint-runtime","tmsp":"2022-08-22T21:12:18Z","size":"707937","prop":"0x6b8e93443b6660a16f67a6cd34d415af463e2285eda3fd02b9fe052c1ad2ceb9"
... }}}}
```

## srtool help

使用 `srtool help` 命令显示 `srtool` 或指定子命令的用法消息。

### 基本用法

```text
srtool help [子命令]
```

### 示例

要显示构建子命令的用法信息，请运行以下命令：

```bash
subkey help build
```

## srtool info

使用 `srtool info` 命令显示有关 `srtool` 容器和您的存储库的信息。
默认情况下，`srtool info` 命令假设运行时的 `Cargo.toml` 文件位于名为链的 [`runtime`](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/runtime) 子目录中。
例如，`srtool info` 命令默认使用以下位置：

- runtime/kusama
- runtime/polkadot
- runtime/rococo
- runtime/westend

如果您的运行时的 `Cargo.toml` 文件位于其他位置，则可以将路径指定为命令行选项。

### 基本用法

运行 `srtool info` 命令的基本语法是：

```text
srtool info [选项] --package <package> [--runtime-dir <path>] [项目路径]
```

### 参数

默认情况下，`srtool info` 在当前工作目录中运行。
如果您的项目未位于当前工作目录中，则可以指定项目位置的路径。

| 参数       | 描述
| :-------------- | :-----------
| `项目路径` | 如果项目未位于当前工作目录中，则指定区块链项目的路径。 |

### 选项

您可以在 `srtool info` 命令中使用以下命令行选项。

| 选项                | 描述
| :-------------------- | -----------
| `-h`,&nbsp;`--help` | 显示用法信息。
| `i`,&nbsp;`--image`&nbsp;`<image>` | 指定一个备用镜像。请务必指定与默认 `paritytech/srtool` 镜像兼容的镜像。您应该注意，指定不同的镜像可能不会产生与 `paritytech/srtool` 镜像产生的相同确定性结果。
| `-p`,&nbsp;`--package`&nbsp;`<package>` | 指定要构建的运行时包的名称。您指定的名称应与运行时的 `Cargo.toml` 文件中定义的名称相同，例如 kusama-runtime、polkadot-runtime 等等。此选项等效于设置 `PACKAGE` 环境变量。
| `-r`,&nbsp;`--runtime-dir`&nbsp;`<runtime>` | 指定运行时的 `Cargo.toml` 文件的位置。如果您的运行时不在标准位置，则可以使用此命令行选项指定正确的位置。此选项等效于设置 `RUNTIME_DIR` 环境变量。
| `-V`,&nbsp;`--version` | 显示版本信息。

### 示例

要显示有关 `srtool` 容器和本地 node-template 存储库的信息，您可能需要运行类似以下命令：

```bash
srtool info --package node-template-runtime --runtime-dir runtime
```

此命令将显示类似以下内容的输出：

```text
{
  "generator": {
    "name": "srtool",
    "version": "0.9.21"
  },
  "src": "git",
  "version": "4.0.0-dev",
  "git": {
    "commit": "6a8b2b12371395979099d2c79ccc1860531b0449",
    "tag": "",
    "branch": "my-release-branch"
  },
  "rustc": "rustc 1.62.0 (a8314ef7d 2022-06-27)",
  "pkg": "polkadot-runtime",
  "profile": "release"
}
```

## srtool pull

使用 `srtool pull` 命令检查并下载最新版本的 `srtool` Docker 镜像。

### 基本用法

运行 `srtool pull` 命令的基本语法是：

```text
srtool pull [选项]
```

### 选项

您可以在 `srtool pull` 命令中使用以下命令行选项。

| 选项              | 描述
| :-------------------| :-----------
| `-h`,&nbsp;`--help`  | 显示用法信息。
| `i`,&nbsp;`--image`&nbsp;`<image>` | 指定一个备用镜像。请务必指定与默认 `paritytech/srtool` 镜像兼容的镜像。您应该注意，指定不同的镜像可能不会产生与 `paritytech/srtool` 镜像产生的相同确定性结果。
| `-V`,&nbsp;`--version` | 显示版本信息。

### 示例

要检查 `srtool` 容器和 Docker 镜像的新版本，您可能需要运行类似以下命令：

```bash
srtool pull
```

此命令将检查 Docker Hub 以获取最新版本的 `paritytech/srtool` 镜像，并开始下载和提取软件。
例如

```text
Found 1.62.0, we will be using paritytech/srtool:1.62.0 for the build
1.62.0: Pulling from paritytech/srtool
405f018f9d1d: Pull complete 
c49473e7f7b3: Pull complete 
7edf98d07029: Pull complete 
85a50724a6fa: Pull complete 
87fb1e3dee5b: Downloading   19.4MB/170.4MB
469075c5d317: Download complete 
533bfa44b64a: Download complete 
...
```

当所有任务完成后，该命令将显示类似以下内容的输出：

```text
Digest: sha256:d5353a63d8fccbef5666e28a8fa0b302d71d4f53cabeb760fe213f3d7df4b8b6
Status: Downloaded newer image for paritytech/srtool:1.62.0
docker.io/paritytech/srtool:1.62.0
```

如果您已经安装了最新版本，则该命令将显示类似以下内容的输出：

```text
Found 1.62.0, we will be using paritytech/srtool:1.62.0 for the build
1.62.0: Pulling from paritytech/srtool
Digest: sha256:d5353a63d8fccbef5666e28a8fa0b302d71d4f53cabeb760fe213f3d7df4b8b6
Status: Image is up to date for paritytech/srtool:1.62.0
docker.io/paritytech/srtool:1.62.0
```

## srtool version

使用 `srtool version` 命令显示 `srtool` 容器的版本信息。如果您想要 `srtool-cli` 可执行文件的版本，请使用 `--version`。

### 基本用法

运行 `srtool version` 命令的基本语法是：

```text
srtool version [选项]
```

### 选项

您可以在 `srtool version` 命令中使用以下命令行选项。

| 选项           | 描述
| :----------------| :-------------
| `-h`,&nbsp;`--help` | 显示用法信息。
| `i`,&nbsp;`--image`&nbsp;`<image>` | 指定一个备用镜像。请务必指定与默认 `paritytech/srtool` 镜像兼容的镜像。您应该注意，指定不同的镜像可能不会产生与 `paritytech/srtool` 镜像产生的相同确定性结果。
| `-V`,&nbsp;`--version` | 显示版本信息。

### 示例

要显示有关 srtool 容器的信息，请运行以下命令：

```bash
srtool version       
```

该命令将显示类似以下内容的输出：

```text
{
  "name": "srtool",
  "version": "0.9.21",
  "rustc": "1.62.0",
  "subwasm": "0.18.0",
  "tera": "0.2.1",
  "toml": "0.2.1"
}
```

要显示 `srtool` 命令行界面的版本信息而不是容器，您可以运行以下命令：

```bash
srtool version --version
```

该命令将显示类似以下内容的输出：

```text
srtool-version 0.8.0
