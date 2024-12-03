---
title: node-template
description: 命令行工具 node-template 的参考信息。
keywords:
---

`node-template` 程序提供了一个带有 FRAME 系统模块和用于处理常见区块链功能操作的额外模块子集的工作 Substrate 节点。
凭借其功能模块的基线，`node-template` 作为构建您自己的区块链和开发自定义运行时的入门工具包。
您可以使用 `node-template` 程序启动 Substrate 节点并执行 [子命令](#子命令) 中列出的任务。

## 基本命令用法

运行 `node-template` 命令的基本语法是：

```shell
node-template [子命令] [标志] [选项]
```

根据您指定的子命令，可能适用或需要其他参数、选项和标志。
要查看特定 `node-template` 子命令的用法信息，请指定子命令和 `--help` 标志。
例如，要查看 `node-template key` 的用法信息，您可以运行以下命令：

```shell
node-template key --help
```

### 标志

您可以在 `node-template` 命令中使用以下可选标志。

| 标志   | 描述
| ------ | -----------
| `--alice`  | 将预定义的 `Alice` 帐户的会话密钥添加到本地密钥库中。此标志等效于使用 `--name alice --validator` 作为命令行选项运行节点。
| `--allow-private-ipv4` | 允许节点连接到私有 IPv4 地址。如果节点的链规范被识别为 `local` 或您使用 `--dev` 标志在开发模式下启动节点，则此标志默认启用。
| `--bob` | 将预定义的 `Bob` 帐户的会话密钥添加到本地密钥库中。此标志等效于使用 `--name bob --validator` 作为命令行选项运行节点。
| `--charlie` | 将预定义的 `Charlie` 帐户的会话密钥添加到本地密钥库中。此标志等效于使用 `--name charlie --validator` 作为命令行选项运行节点。
| `--dave` | 将预定义的 `Dave` 帐户的会话密钥添加到本地密钥库中。此标志等效于使用 `--name dave --validator` 作为命令行选项运行节点。
| `--dev` | 在全新状态下以开发模式启动节点。如果您使用此标志运行节点，则不会持久化任何状态。
| `--disable-log-color` | 禁用在日志消息中使用颜色。
| `--disable-log-reloading` | 禁用日志过滤器更新和重新加载。默认情况下，动态日志过滤已启用。但是，该功能会影响性能。如果您使用此标志启动节点，则 `system_addLogFilter` 和 `system_resetLogFilter` 远程过程调用无效。
| `--discover-local`| 启用本地网络上的对等发现。默认情况下，如果您使用 `--dev` 标志启动节点，或者链规范是 `Local` 或 `Development`，则此标志为 `true`，否则为 `false`。
| `--eve` | 将预定义的 `Eve` 帐户的会话密钥添加到本地密钥库中。此标志等效于使用 `--name eve --validator` 作为命令行选项运行节点。
| `--ferdie` | 将预定义的 `Ferdie` 帐户的会话密钥添加到本地密钥库中。此标志等效于使用 `--name ferdie --validator` 作为命令行选项运行节点。
| `--force-authoring` | 即使节点处于离线状态，也启用区块创作。
| `-h`, `--help` | 显示用法信息。
| `--ipfs-server` | 加入 IPFS 网络并通过 bitswap 协议提供交易。
| `--kademlia-disjoint-query-paths` | 要求迭代式 Kademlia 分布式哈希表 (DHT) 查询使用不相交的路径。此选项在存在潜在的对抗性节点的情况下提高了弹性。有关高级设计及其安全改进的更多信息，请参阅 S/Kademlia 论文。
| `--no-grandpa` | 如果节点以验证器模式运行，则禁用 GRANDPA 投票器。如果节点未以验证器身份运行，则该选项将禁用 GRANDPA 观察器。
| `--no-mdns` | 禁用 mDNS 发现。默认情况下，网络使用 mDNS 在本地网络上发现其他节点。此选项禁用发现，如果您使用 `--dev` 选项启动节点，则会自动应用。
| `--no-private-ipv4` | 阻止连接到私有 IPv4 地址，除非该地址已使用 `--reserved-nodes` 或 `--bootnodes` 选项传递。此设置默认情况下对在其链规范中标记为“live”的链启用。
| `--no-prometheus` | 禁用公开 Prometheus 端点以接收指标。默认情况下，指标会导出到 Prometheus 端点。
| `--no-telemetry` | 禁用连接到 Substrate 遥测服务器。默认情况下，遥测对全局链启用。
| `--one` | 提供一个快捷方式，用于指定 `--name One --validator` 以将 `One` 的会话密钥添加到密钥库中。
| `--password-interactive` | 允许您在终端 shell 中以交互方式指定连接到密钥库的密码。
| `--prometheus-external` | 在所有接口上公开 Prometheus 导出器。默认值为本地。
| `--reserved-only` | 指定是否仅与保留节点同步链。此选项还禁用自动对等发现。TCP 连接可能仍然与非保留节点建立。特别是，如果您是验证器，您的节点可能仍然连接到其他验证器节点和验证器节点，无论它们是否被定义为保留节点。
| `--rpc-external` | 监听所有 RPC 接口。默认情况下，节点仅监听本地 RPC 调用。如果您设置此命令行选项，请记住，并非所有 RPC 方法都适合公开暴露。使用 RPC 代理服务器来过滤掉危险的方法。有关不应公开暴露的 RPC 方法的更多信息，请参阅 [远程过程调用](/build/remote-procedure-calls/)。如果您了解风险，请使用 `--unsafe-rpc-external` 来抑制警告。
| `--storage-chain` | 启用存储链模式。如果您设置此选项，每个交易将分别存储在交易数据库列中，并且仅通过哈希在区块主体列中引用。
| `--tmp` | 运行一个临时节点。此选项创建一个临时目录来存储区块链配置，包括节点数据库、节点密钥和密钥库。
| `--two` | 提供一个快捷方式，用于指定 `--name Two --validator` 以将 `Two` 的会话密钥添加到密钥库中。
| `--unsafe-pruning` | 强制节点以不安全的修剪设置启动。在以验证器身份运行时，强烈建议禁用状态修剪（即存档），这是默认设置。除非设置此选项，否则节点将拒绝以验证器身份启动，如果启用了修剪。
| `--unsafe-rpc-external` | 监听所有 RPC 接口。此选项与 `--rpc-external` 相同。
| `--unsafe-ws-external` | 监听所有 Websocket 接口。此选项与 `--ws-external` 相同，但不会向您发出警告。
| `--validator` | 以授权角色启动节点，并使其能够积极参与任何它可以参与的共识任务（例如，取决于本地密钥的可用性）。
| `-V`, `--version` | 显示版本信息。
| `--ws-external` | 监听所有 Websocket 接口。默认情况下，节点仅在本地监听。请记住，并非所有 RPC 方法都适合公开暴露。您可以使用 RPC 代理服务器来过滤掉危险的方法。如果您了解风险，请使用 `--unsafe-ws-external` 来抑制警告。

### 选项

您可以在 `node-template` 命令中使用以下选项。

| 选项 | 描述
| ------ | -----------
| `-d`, `--base-path <path>` | 指定一个自定义基本路径。
| `--bootnodes <node-identifier>...` | 指定用于对等通信的引导节点标识符列表。
| `--chain <chain-specification>` | 指定要使用的链规范。您可以使用预定义的链规范名称设置此选项，例如 `dev`、`local` 或 `staging`，或者您可以指定包含链规范的文件的路径，例如使用 `build-spec` 子命令生成的链规范。
| `--database <database>` | 选择要使用的数据库后端。有效值为 `rocksdb`、`paritydb-experimental` 或 `auto`。
| `--db-cache <MiB>` | 限制数据库缓存可以使用多少内存。
| `--offchain-worker <execution>` | 确定何时执行离线工作进程。默认情况下，离线工作进程仅对正在创作新区块的节点启用，并且离线工作进程在区块验证期间执行。有效值为 `Always`、`Never` 或 `WhenValidating`。
| `--execution <strategy>` | 确定所有执行上下文使用的执行策略。有效值为 `Native`、`Wasm`、`Both` 或 `NativeElseWasm`。
| `--execution-block-construction <strategy>` | 指定调用运行时以构建区块时使用的执行类型。有效值为 `Native`、`Wasm`、`Both` 或 `NativeElseWasm`。
| `--execution-import-block <strategy>` | 指定调用运行时以导入区块（包括本地创作的区块）时使用的执行类型。有效值为 `Native`、`Wasm`、`Both` 或 `NativeElseWasm`。
| `--execution-offchain-worker <strategy>` | 指定调用运行时以使用离线工作进程时使用的执行类型。有效值为 `Native`、`Wasm`、`Both` 或 `NativeElseWasm`。
| `--execution-other <strategy>` | 指定在不进行同步、导入或构建区块的情况下调用运行时时使用的执行类型。有效值为 `Native`、`Wasm`、`Both` 或 `NativeElseWasm`。
| `--execution-syncing <strategy>` | 指定在将区块作为初始同步的一部分导入运行时时使用的执行类型。有效值为 `Native`、`Wasm`、`Both` 或 `NativeElseWasm`。
| `--in-peers <count>` | 指定要接受的最大传入连接数。默认值为 25 个对等点。
| `--enable-offchain-indexing <database>` | 启用离线索引 API。离线索引 API 使运行时能够在区块导入期间直接写入离线工作进程数据库。
| `--ipc-path <path>` | 指定将进程间通信 (IPC) 发送到远程过程调用 (RPC) 服务器的路径。
| `--keep-blocks <count>` | 指定要保留在数据库中的已完成区块数。默认值为保留所有区块。
| `--keystore-path <path>` | 指定自定义密钥库的路径。
| `--keystore-uri <keystore-uri>` | 指定要连接到的自定义 URI 以获取密钥库服务。
| `--listen-addr <listen-address>... ` | 指定节点要监听的地址。默认情况下，如果您使用 `--validator` 选项启动节点，则使用地址 `/ip4/0.0.0.0/tcp/<port>` 和 `/ip6/[::]/tcp/<port>`。否则，将使用 `/ip4/0.0.0.0/tcp/<port>/ws` 和 `/ip6/[::]/tcp/<port>/ws` 地址。
| `-l`, `--log <log-pattern>...` | 设置自定义日志记录过滤器。要使用的语法是 `<log-target>=<level>`，例如 `-lsync=debug`。从最不详细到最详细的有效日志级别分别是 `error`、`warn`、`info`、`debug` 和 `trace`。默认情况下，所有目标都记录 `info` 级别的消息。您可以使用 `-l<level>` 设置全局日志级别。
| `--max-parallel-downloads <count>` | 指定要从其并行请求相同区块的最大对等点数量。此选项允许节点从多个对等点下载已宣布的区块。您可以减少计数以减少流量，但可能会增加延迟。默认值为 5 个并行下载。
| `--max-runtime-instances <max-runtime-instances>` | 指定每个运行时的实例缓存的最大大小。默认值为 8，大于 256 的值将被忽略。
| `--name <name>` | 指定此节点的人类可读名称。如果启用，节点名称将报告给遥测服务器。
| `--node-key <key>` | 指定用于 `libp2p` 网络的密钥。该值是一个字符串，根据 `--node-key-type` 进行解析。例如，如果节点密钥类型为 `ed25519`，则节点密钥将被解析为十六进制编码的 Ed25519 32 字节密钥（64 个十六进制字符）。此选项的值优先于 `--node-key-file`。请注意，作为命令行参数提供的密钥很容易暴露。您应该仅将此选项用于开发和测试。要使用外部管理的密钥，请使用 `--node-key-file` 选项。
| `--node-key-file <file>` | 指定包含节点用于 `libp2p` 网络的密钥的文件。文件的內容根据 `--node-key-type` 进行解析。例如，如果节点密钥类型为 `ed25519`，则该文件必须包含未编码的 32 字节或十六进制编码的 Ed25519 密钥。如果该文件不存在，则使用您使用 `--node-key-type` 选项指定的类型创建具有新生成的密钥的文件。
| `--node-key-type <type>` | 指定用于对等 (`libp2p`) 网络的密钥类型。您可以在命令行上使用 `--node-key` 选项指定密钥，使用 `--node-key-file` 选项从文件读取密钥，或从 `--base-dir` 选项指定的基本目录内的链特定 `config` 目录中指定的文件读取密钥。如果此文件不存在，则使用所选类型创建具有新生成的密钥的文件。节点的密钥决定了公钥（对等标识符），该公钥用于使用 `libp2p` 库与节点通信。默认类型为 Ed25519。
| `--out-peers <count>` | 指定要维护的最大传出连接数。默认值为 25。
| `--password <password>` | 指定用于密钥库的密码。
| `--password-filename <path>` | 指定包含用于访问密钥库的密码的文件的路径。
| `--pool-kbytes <count>` | 指定存储在交易池中的所有交易的最大千字节数。默认值为 20480 KB。
| `--pool-limit <count>` | 指定交易池中可以存在的最大交易数。默认值为 8192 个交易。
| `--port <port>` | 指定用于对等通信的 TCP 端口。
| `--prometheus-port <port>` | 指定用于 Prometheus 导出器服务的 TCP 端口。
| `--pruning <pruning-mode>` | 指定要保留的最大区块状态数或 `archive` 以保留所有区块状态。如果节点以验证器身份运行，则默认值为保留所有区块状态。如果节点未以验证器身份运行，则仅保留最后 256 个区块的状态。
| `--public-addr <public-address>...` | 指定其他节点可以用来连接到节点的公共地址。您可以使用此选项通过代理连接到节点。
| `--reserved-nodes <address>...` | 指定保留节点地址列表。
| `--rpc-cors <origins>` | 指定允许访问 HTTP 和 WS RPC 服务器的浏览器来源。您可以使用 `protocol://domain` 语法、`null` 或 `all` 将此选项指定为以逗号分隔的来源列表。`all` 值禁用来源验证。默认情况下，允许 `localhost` 和 `https://polkadot.js.org` 来源。如果您使用 `--dev` 选项启动节点，则默认情况下允许所有来源。
| `--rpc-http-threads <count>` | 指定 RPC HTTP 服务器线程池的大小。
| `--rpc-max-payload <rpc-max-payload>` | 设置请求和响应（HTTP 和 Web 套接字）的最大 RPC 有效负载大小（以兆字节为单位）。默认值为 15 MiB。
| `--rpc-methods <method-set>` | 指定要公开的 RPC 方法。有效值为 `Unsafe` 以公开每个 RPC 方法，`Safe` 仅公开 RPC 方法的安全子集，拒绝不安全的 RPC 方法，或 `Auto` 以公开 `Safe` RPC 方法（如果 RPC 在外部提供服务），例如，如果您使用 `--rpc-external` 或 `--rpc-external` 运行节点，或者公开 `Unsafe` RPC 方法（如果 RPC 未在外部提供服务）。默认值为 `Auto`。
| `--rpc-port <port>` | 指定用于 HTTP RPC 服务器的 TCP 端口。
| `--state-cache-size <bytes>` | 指定状态缓存大小。默认值为 67108864 字节。
| `--sync <sync-mode>` | 指定区块链同步模式有效值为 `Full` 以下载和验证完整的区块链历史记录，`Fast` 以仅下载区块和最新状态，或 `FastUnsafe` 以下载最新状态但跳过下载状态证明。默认值为 `Full`。
| `--telemetry-url <url verbosity>...` | 指定要连接到的遥测服务器的 URL。您可以多次传递此标志以指定多个遥测端点。详细程度级别范围为 0-9，其中 0 表示最不详细。使用以下格式指定 URL，然后详细程度选项为 `--telemetry-url 'wss://foo/bar 0'`。
| `--tracing-receiver <receiver>` | 指定接收器以处理跟踪消息。默认值为 Log。
| `--tracing-targets <targets>` | 设置自定义分析过滤器。语法与日志记录相同：`<target>=<level>`。
| `--wasm-execution <method>` | 指定执行 Wasm 运行时代码的方法。有效值为 `interpreted` 或 `compiled`。默认值为 `Compiled`。
| `--wasm-runtime-overrides <path>` | 指定本地 WASM 运行时存储的路径。当版本匹配时，这些运行时会覆盖链上运行时。
| `--ws-max-connections <count>` | 指定 WS RPC 服务器连接的最大数量。
| `--ws-port <port>` | 指定用于 WebSockets RPC 服务器的 TCP 端口。

### 子命令

您可以在 `node-template` 命令中使用以下子命令。
有关参考信息和说明使用这些子命令的示例，请选择相应的命令。

| 命令 | 描述
| ------- | -----------
| `benchmark` | 对运行时模块进行基准测试。
| `build-spec` | 构建链规范。
| `check-block` | 验证区块。
| `export-blocks` | 导出区块。
| `export-state` | 将给定区块的状态导出到链规范中。
| `help` | 显示 `node-template` 的用法信息或任何 `node-template` 子命令的命令行用法信息摘要。
| `import-blocks` | 导入区块。
| `key` | 提供本地密钥管理实用程序。
| `purge-chain` | 删除所有链数据。
| `revert` | 将链还原到以前的状态。

## benchmark

使用 `node-template benchmark` 命令分析在运行时模块中配置的外部调用中执行交易所需的资源。
您可以分析特定模块中的单个外部调用，或分析所有模块中的所有外部调用。
使用 `benchmark` 子命令，您可以使用其他命令行选项来测试不同的执行场景并比较结果。

请注意，您必须使用启用了基准测试的节点进行编译才能使用 `node-template benchmark` 的所有子命令。
要使用启用了基准测试功能的节点进行编译，请运行以下命令：

```shell
cargo build --package node-template --release --features runtime-benchmarks
```

#### 基本命令用法

```shell
node-template benchmark [子命令] [标志] [选项]
```

根据您指定的子命令，可能适用或需要其他参数、选项和标志。
要查看特定 `benchmark` 子命令的用法信息，请指定子命令和 `--help` 标志。
例如，要查看 `benchmark pallet` 的用法信息，您可以运行以下命令：

```shell
node-template benchmark pallet --help
```
#### 子命令

您可以在 `node-template benchmark` 命令中使用以下子命令。

| 命令 | 描述
| ------- | -----------
| `block` | 对历史区块的执行时间进行基准测试。
| `help` | 显示 `node-template benchmark` 或指定子命令的用法信息。
| `overhead` | 对每个区块和每个外部的执行开销进行基准测试。
| `pallet` | 对 FRAME 模块的外部权重进行基准测试。
| `storage` | 对链快照的存储速度进行基准测试。

#### 标志

您可以在 `node-template benchmark` 命令中使用以下可选标志。

| 标志   | 描述
| ------ | -----------
| `-h`, `--help` | 显示用法信息。
| `-V`, `--version` | 显示版本信息。

#### 选项

您可以在 `node-template benchmark` 子命令中结合使用所有常见的 node-template 命令行选项。
例如，您可以使用 `--base-path <path>` 指定区块链数据的自定义目录，并使用 `--chain <chain-specification>` 指定要与任何 `benchmark` 子命令一起使用的链规范。

但是，还有许多专门用于执行基准测试任务的命令行选项。
例如，`node-template benchmark block` 子命令支持 `--from` 和 `--to` 命令行选项，用于指定要分析的区块。

由于对 FRAME 模块进行基准测试代表了最常见的基准测试任务，因此 `node-template benchmark pallet` 子命令支持最多的任务特定命令行选项。
例如，您可以在 `node-template benchmark pallet` 子命令中使用以下选项。

| 选项 | 描述
| ------ | -----------
| `--external-repeat` <count> | 指定为客户端重复执行基准测试的次数。请注意，此选项可能会给出较慢的结果，但会最大限度地提高 Wasm 内存。默认值为一次执行。
| `--extra` | 显示并运行额外的基准测试，否则这些基准测试对于权重构建来说是不必要的。
| `-e`, `--extrinsic <extrinsic>` | 指定要基准测试的模块中的单个函数，或 `*` 以基准测试模块中的所有函数调用。
| `--header <header>` | 将头文件添加到您的基准测试输出中。
| `--heap-pages <heap-pages>` | 在运行基准测试时设置堆页面。如果未设置，则使用节点的默认值。
| `--high <highest-range-values>...` | 指示每个组件范围的最高值。
| `json-input <json-input-file>` | 指定包含先前生成的基准测试结果的 JSON 文件的路径。此选项使您能够重新使用使用 `--json-file` 生成的基准测试原始结果来重新运行基准测试分析并重新生成模块的权重，而无需实际重新运行基准测试测试。
| `--list` | 列出所有当前定义的基准测试，而不运行它们。
| `--low <lowest-range-values>...` | 指示每个组件范围的最低值。
| `--no-median-slopes` | 禁用中位数斜率线性回归分析。
| `--no-min-squares` | 禁用最小二乘线性回归分析。
| `--no-storage-info` | 禁用在分析输出中显示存储信息。这与存储信息是否出现在 *输出文件* 中无关。为此目的使用 Handlebar 模板。
| `--no-verify` | 在运行基准测试时禁用验证逻辑。
| `--output <output>` | 将基准测试输出到给定路径的 Rust 文件中。
| `--output-analysis <analysis-type> ` | 指定要在基准测试输出中使用的分析函数。有效值为 `min-squares`、`median-slopes` 或 `max`。默认值为 `min-squares` 分析。有关基准测试分析的更多信息，请参阅 [基准测试](/test/benchmark/)。
| `-p`, `--pallet <pallet>` | 指定要基准测试的 FRAME 模块，或 `*` 以基准测试所有模块。如果您对所有模块进行基准测试，则还必须指定 `--extrinsic *` 以基准测试所有外部调用。
| `--record-proof` | 估计验证证明 (PoV) 大小。
| `-r`, `--repeat <repeat>` | 指定从 WebAssembly 二进制文件中重复执行基准测试的次数。默认值为一次执行。
| `-s`, `--steps <steps>` | 指定在变量组件中要采样的样本数。默认值为一个样本。
| `--template <template>` | 指定用于输出基准测试结果的 Handlebars 模板文件的路径。

有关不同基准测试子命令和相关命令行选项的示例，请参阅 [基准测试示例](#基准测试示例)。

#### 基准测试示例

在您使用启用了基准测试的运行时编译后，您可以运行类似以下命令来对所有配置了运行时基准测试的模块中的所有函数调用进行基准测试：

```shell
./target/release/node-template benchmark pallet \
    --chain dev \
    --extrinsic "*" \
    --pallet "*" \
    --output pallets/weights.rs
```

使用此命令，每个函数调用都会使用单个值执行一次，并将结果权重记录在 `weights.rs` 文件中。

根据您要基准测试的函数，您可以添加 `--steps` 和 `--repeat` 命令行选项以使用不同的值多次执行调用。
例如，以下命令执行 `pallet_template` 中的 `do_something` 函数，并调用该函数 20 次以获取 10 个数据点：

```shell
./target/release/node-template benchmark pallet \
    --chain dev \
    --extrinsic do_something \
    --pallet pallet_template \
    --steps 10 \
    --repeat 20 \
    --list
```

使用 `--list` 选项，该命令将显示以下输出：

```shell
pallet, benchmark
pallet_template, do_something
```

使用 `--steps` 和 `--repeat` 命令行选项，该命令将显示以下基准测试结果：

```shell
Pallet: "pallet_template", Extrinsic: "do_something", Lowest values: [], Highest values: [], Steps: 10, Repeat: 20
Raw Storage Info
========
Storage: TemplateModule Something (r:0 w:1)

Median Slopes Analysis
========
-- Extrinsic Time --

Model:
Time ~=       12
    + s        0
              µs

Reads = 0 + (0 * s)
Writes = 1 + (0 * s)

Min Squares Analysis
========
-- Extrinsic Time --

Data points distribution:
    s   mean µs  sigma µs       %
    0      12.4     0.489    3.9%
   10      12.1       0.3    2.4%
   20      12.1       0.3    2.4%
   30      11.8       0.4    3.3%
   40        12         0    0.0%
   50      11.9       0.3    2.5%
   60        12         0    0.0%
   70      12.2       0.4    3.2%
   80        12         0    0.0%
   90        12         0    0.0%
  100        12         0    0.0%

Quality and confidence:
param     error
s             0

Model:
Time ~=    12.13
    + s        0
              µs

Reads = 0 + (0 * s)
Writes = 1 + (0 * s)
```

要测量每个区块和每个外部的平均、中位数、最小值和最大值执行时间，您可以运行 `node-template benchmark overhead` 子命令：

```shell
./target/release/node-template benchmark overhead
```

该命令将显示类似以下内容的输出：

```shell
Running 10 warmups...    
Executing block 100 times    
Per-block execution overhead [ns]:
	Total: 81779519
	Min: 695173, Max: 1522917
	Average: 817795, Median: 763360, Stddev: 160759.2
	Percentiles 99th, 95th, 75th: 1443343, 1263935, 814356    
	Writing weights to "block_weights.rs"    
Running 10 warmups...    
Executing block 100 times    
Building block, this takes some time...    
Extrinsics per block: 12000    
Running 10 warmups...    
Executing block 100 times
Per-extrinsic execution overhead [ns]:
	Total: 8955542
	Min: 85950, Max: 97448
	Average: 89555, Median: 88348, Stddev: 2713.05
	Percentiles 99th, 95th, 75th: 97247, 95040, 91733    
	Writing weights to "extrinsic_weights.rs"
```

默认情况下，该命令执行基准测试 100 次，生成结果，并将输出写入 `block_weights.rs` 和 `extrinsics_weights.rs` 文件。
您可以使用命令行选项通过添加单位或将平均执行时间乘以某个因子来调整计算的权重。

要测量 Substrate 开发链规范的存储执行时间，您可以运行以下命令：

```shell
./target/release/node-template benchmark storage --state-version 1
```

该命令将显示类似以下内容的输出：

```shell
Warmup round 1/1    
Preparing keys from block BlockId::Number(0)    
Reading 36 keys    
Time summary [ns]:
	Total: 478367
	Min: 8889, Max: 32258
	Average: 13287, Median: 12580, Stddev: 4189.73
	Percentiles 99th, 95th, 75th: 32258, 21863, 14563
Value size summary:
	Total: 157217
	Min: 1, Max: 155944
	Average: 4367, Median: 16, Stddev: 25621.19
	Percentiles 99th, 95th, 75th: 155944, 82, 80    
Warmup round 1/1    
Preparing keys from block BlockId::Number(0)    
Writing 36 keys    
Time summary [ns]:
	Total: 1877401
	Min: 13476, Max: 665226
	Average: 52150, Median: 25740, Stddev: 111805.24
	Percentiles 99th, 95th, 75th: 665226, 252697, 30427
Value size summary:
	Total: 157217
	Min: 1, Max: 155944
	Average: 4367, Median: 16, Stddev: 25621.19
	Percentiles 99th, 95th, 75th: 155944, 82, 80    
Writing weights to "rocksdb_weights.rs"
```

要获取 `paritydb` 数据库而不是默认的 `rocksdb` 数据库的基准测试信息，请使用 `--db paritydb` 命令行选项。
要获取 Polkadot 或任何其他真实链快照的存储基准测试信息，请使用命令行选项 `--state-version 0`。
有关使用基准测试存储子命令的更多信息，请参阅 [基准测试存储命令](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/utils/frame/benchmarking-cli/src/storage)。

有关如何在运行时添加基准测试的更多信息，请参阅 [基准测试](/test/benchmark/) 和 [添加基准测试](/reference/how-to-guides/weights/add-benchmarks/)。

## build-spec

使用 `node-template build-spec` 命令为您的运行时创建链规范文件。

#### 基本命令用法

```shell
node-template build-spec [标志] [选项]
```

#### 标志

您可以在 `node-template build-spec` 命令中使用以下可选标志。

| 标志   | 描述
| ------ | -----------
| `--detailed-log-output` | 启用详细日志输出，包括日志目标名称、日志级别和线程名称。如果您启用高于 `info` 的任何日志级别，则会自动使用此选项。
| `--dev` | 以开发模式启动节点。使用此标志还会默认启用 `--chain=dev`、`--force-authoring`、`--rpc-cors=all`、`--alice` 和 `--tmp` 标志。
| `--disable-default-bootnode` | 禁用将默认引导节点添加到规范中。默认情况下，当不存在引导节点时，将 `/ip4/127.0.0.1/tcp/30333/p2p/NODE_PEER_ID` 引导节点添加到规范中。
| `--disable-log-color` | 禁用日志颜色输出。
| `--enable-log-reloading` | 允许动态更新和重新加载日志过滤器。请注意，此选项会显着降低性能。设置此选项不会影响 `system_addLogFilter` 和 `system_resetLogFilter` RPC 方法。
| `-h`, `--help` | 显示用法信息。
| `--raw` |  将链规范格式化为原始创世存储输出。
| `-V`, `--version` | 显示版本信息。

#### 选项

您可以在 `node-template build-spec` 命令中使用以下命令行选项。

| 选项   | 描述
| -------- | -----------
| `-d`, `--base-path <path>` | 指定一个自定义基本路径。
| `--chain <chain-specification>` | 指定要使用的链规范。您可以使用预定义的链规范名称设置此选项，例如 `dev`、`local` 或 `staging`，或者您可以指定包含链规范的文件的路径，例如使用 `build-spec` 子
