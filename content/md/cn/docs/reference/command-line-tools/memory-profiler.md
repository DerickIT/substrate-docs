---
title: memory-profiler
description: 命令行工具 memory-profiler 的参考信息。
keywords:
---

内存分析使您能够了解基于 Substrate 的客户端中区块链应用程序随时间的内存分配和行为。
它在内存分配的上下文中识别方法调用，并将此信息与已分配对象的数目结合起来。
此外，分析可用于分析内存泄漏、识别内存消耗发生的位置、定义临时分配以及调查应用程序内的过度内存碎片。

我们推荐的分析器是 [koute 的内存分析器](https://github.com/koute/memory-profiler)。

## 安装

您可以从 [发布](https://github.com/koute/memory-profiler/releases) 下载分析器的预编译二进制版本。
我们测试的最后一个版本是 0.6.1，但任何更新的版本也可能有效。

以下是如何从命令行下载和解压缩它：

```bash
$ curl -L https://github.com/koute/memory-profiler/releases/download/0.6.1/memory-profiler-x86_64-unknown-linux-gnu.tgz -o memory-profiler-x86_64-unknown-linux-gnu.tgz
$ tar -xf memory-profiler-x86_64-unknown-linux-gnu.tgz
```

这将导致解压缩三个文件。我们只对其中的两个感兴趣：

- `libmemory_profiler.so` - 这是我们将挂接到 Substrate 的内存分析器本身
- `memory-profiler-cli` - 这是我们稍后将用于分析分析数据的程序

您也可以 [从源代码编译分析器](https://github.com/koute/memory-profiler#building)。

首先，您需要确保已安装以下内容：

- GCC 工具链
- Rust nightly（我们测试了版本 `nightly-2021-06-08`）
- [Yarn](https://yarnpkg.com) 包管理器（用于构建 GUI）

然后您应该能够像这样构建分析器：

```bash
$ git clone https://github.com/koute/memory-profiler
$ cd memory-profiler
$ cargo build --release -p memory-profiler
$ cargo build --release -p memory-profiler-cli
```

您将在 `target/release/libmemory_profiler.so` 和 `target/release/memory-profiler-cli` 中找到我们需要的二进制文件。

## 从命令行启动带有分析的节点

如果您是从命令行手动启动 Substrate，那么将分析器连接到它
归结为设置一些额外的环境变量，然后像往常一样正常启动它。

首先，我们希望启用日志记录，告诉分析器它应该在哪里输出其日志，它应该在哪里收集其分析数据，以及可选地告诉它剔除临时分配：

```bash
$ export MEMORY_PROFILER_LOG=info
$ export MEMORY_PROFILER_LOGFILE=profiling_%e_%t.log
$ export MEMORY_PROFILER_OUTPUT=profiling_%e_%t.dat

# 可选，取决于您想要分析的确切方面
# 以及您将要分析多长时间。
$ export MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS=1
```

然后我们可以启动连接了分析器的 Substrate：

```bash
$ LD_PRELOAD=/path/to/libmemory_profiler.so ./target/release/substrate
```

设置 `LD_PRELOAD` 环境变量将指示 Linux 的动态链接器在 Substrate 启动之前将其注入
内存分析器，这允许分析器挂接到
系统的内存分配例程并跟踪 Substrate 执行的每个内存分配。

## 远程启动带有分析的节点

如果您远程运行基于 Substrate 的节点，您可能正在使用 `systemd` 来管理它。
以下是如何在这种情况下设置分析的方法。

我们假设您已经下载了分析器的预编译二进制文件或从
源代码编译它，并且您已将其放在当前目录中。

首先，我们希望将内存分析器复制到全局可访问的位置并设置一个位置
它可以在其中写入其日志并收集分析数据。

```bash
$ sudo mkdir -p /opt/memory-profiler/bin
$ sudo cp libmemory_profiler.so /opt/memory-profiler/bin/
$ sudo mkdir /opt/memory-profiler/logs
$ sudo chmod 0777 /opt/memory-profiler/logs
```

然后我们希望设置一个包含所有环境变量的文件来配置分析器本身：

```bash
$ echo "MEMORY_PROFILER_OUTPUT=/opt/memory-profiler/logs/profiling_%e_%t_%p.dat" | sudo tee /opt/memory-profiler/env
$ echo "MEMORY_PROFILER_LOGFILE=/opt/memory-profiler/logs/profiling_%e_%t_%p.txt" | sudo tee -a /opt/memory-profiler/env
$ echo "MEMORY_PROFILER_LOG=info" | sudo tee -a /opt/memory-profiler/env
$ echo "MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS=1" | sudo tee -a /opt/memory-profiler/env
$ echo "LD_PRELOAD=/opt/memory-profiler/bin/libmemory_profiler.so" | sudo tee -a /opt/memory-profiler/env
```

现在您需要打开节点的 `systemd` 单元文件并在 `[Service]` 部分添加以下内容：

```text
[Service]
EnvironmentFile=/opt/memory-profiler/env
```

如果已存在 `[Service]` 部分，请勿添加另一个 `[Service]` 部分；只需将 `EnvironmentFile` 键添加到其中即可。
如果您已经有一个 `EnvironmentFile` 键，请不要替换它；只需添加第二个即可，`systemd` 将应用两者。

现在您可以重新加载您的 `systemd` 守护程序：

```bash
$ sudo systemctl daemon-reload
```

然后重新启动您的服务以开始分析：

```bash
$ sudo systemctl restart kusama
```

分析数据将收集在 `/opt/memory-profiler/logs` 中。如果您想禁用
内存分析器只需删除您添加到单元文件中的 `EnvironmentFile` 键，然后再次重新启动
服务。

## 配置分析器

还有 [其他环境变量可以设置来配置分析器](https://github.com/koute/memory-profiler#environment-variables-used-by-libmemory_profilerso)，
尽管除了我们已经显示的之外，在正常情况下更改它们应该是不必要的。

一个需要额外考虑的配置旋钮是 `MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS`，
它控制分析器何时收集短暂的分配。

默认情况下，分析器将收集被分析应用程序进行的 **每个** 分配。
这是一个 _大量_ 的数据，并且可能达到每秒 _兆字节_ 的数量级。如果您
只想在短时间内进行分析，或者如果您特别关心诊断
临时分配，但这在您想要让分析器运行更长时间时会变得有问题。

这就是 `MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS` 选项的用武之地。当您通过将其设置为 `1` 来打开它时
分析器将省略所有真正短暂的分配，并且不会将其写入
磁盘。这显着减少了生成的数据量，通常达到每秒 _千字节_ 的范围，这使得可以将分析运行数天
时间。

## 分析

现在您已经收集了分析数据，您现在可以分析它了。

假设您同时拥有 `memory-profiler-cli` 和您收集的 `.dat` 文件
在同一个目录中，您可以加载它的 GUI：

```bash
$ ./memory-profiler-cli server *.dat
```

这可能需要一段时间，具体取决于您的确切硬件以及您尝试加载的数据量。
最终您应该会看到类似以下内容被打印出来：

```text
[2020-05-06T08:59:20Z INFO  cli_core::loader] Loaded data in 315s 820
[2020-05-06T08:59:20Z INFO  actix_server::builder] Starting 8 workers
[2020-05-06T08:59:20Z INFO  actix_server::builder] Starting server on 127.0.0.1:8080
```

现在您可以打开您的 Web 浏览器并访问 `http://localhost:8080/` 上的 GUI。

![来自 Web UI 的图表](/media/images/docs/reference/mem-profiling-memory-graph.png)

还有 [一个 REST API](https://github.com/koute/memory-profiler#rest-api-exposed-by-memory-profiler-cli-server)
如果您想将数据导出到其他格式或以编程方式检查它，则可以访问它。

## 其他提示

- 建议始终检查分析器生成的日志，并查看是否存在任何 `WRN` 或 `ERR` 日志。

- 根据您运行的 Linux 发行版，您可能会在分析器的日志中看到以下错误或警告：

  ```text
  The perf_event_open syscall failed for PID 0: Operation not permitted (os error 1)`
  ```

  这通常是无害的；最多这只会导致在分析时 CPU 使用率更高。
  您可以通过执行以下操作来避免它：

  ```bash
  $ echo "-1" | sudo tee /proc/sys/kernel/perf_event_paranoid
  ```

  但请注意，这可能有一些安全隐患。
  查看 `man perf_event_open` 以了解更多详细信息。

- 在分析过程中，必须将整个数据文件加载到内存中。
  如果您没有足够的 RAM 并尝试加载一个大文件，分析器可能会耗尽内存并崩溃。
