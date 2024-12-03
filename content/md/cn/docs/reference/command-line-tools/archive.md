---
title: archive
description: 命令行工具 archive 的参考信息。
keywords:
---

`archive` 程序用于索引基于 Substrate 的链的所有区块、状态和交易数据，并将索引数据存储在关系型 SQL 数据库中。
`archive` 程序创建的数据库镜像了正在运行的 Substrate 区块链中的所有数据。
在您存档数据后，您可以使用数据库工具查询和检索有关区块链状态的 SQL 数据库中的信息。
有关您可能希望针对 Substrate 存档数据库运行的查询示例，请参阅 [有用查询](https://github.com/paritytech/substrate-archive/wiki/Useful-Queries)。

## 开始之前

在使用 `archive` 为基于 Substrate 的链创建数据库之前，您需要使用所需文件准备您的环境：

- 您必须在运行 Substrate 节点的计算机上安装 PostgreSQL。

  您可以从 PostgreSQL [下载](https://www.postgresql.org/download/) 页面下载适用于不同平台的 PostgreSQL 软件包。

  根据您的平台，您可能能够使用本地包管理器安装 PostgreSQL。
  例如，您可以在 macOS 计算机上通过在终端中运行 `brew install postgresql` 来安装 PostgreSQL 软件包。

- 您必须在安装了 PostgreSQL 的计算机上安装 RabbitMQ 或 Docker Compose。

  根据您的平台，安装 RabbitMQ 或 Docker 的说明和系统要求可能会有所不同。
  有关使用 [RabbitMQ](https://www.rabbitmq.com/) 或 [Docker](https://www.docker.com/) 的信息，请参阅 [设置](https://github.com/paritytech/substrate-archive/wiki/1-Setup) `substrate-archive` wiki 页面。

- 您的 Substrate 链必须使用 RocksDB 作为其后端数据库。

## 安装和配置

要安装 `substrate-archive-cli` 程序：

1. 在您的计算机上打开一个终端 shell。

1. 通过运行以下命令克隆 `substrate-archive` 存储库：

   ```
   git clone https://github.com/paritytech/substrate-archive.git
   ```

1. 通过运行以下命令更改到 `substrate-archive` 存储库的根目录：

   ```
   cd substrate-archive
   ```

1. 在 Substrate 节点上启动 PostgreSQL 数据库（`postgres`）和 Postgre 管理进程（`pgadmin`）。

   如果您有 Docker Compose，则可以通过运行 `docker-compose up -d` 命令自动启动服务。

1. 启动您的 Substrate 节点，并将 `pruning` 设置为 archive。

   例如：

   ```
   ./target/release/solo-template-node --pruning=archive
   ```

2. 查看当前的数据库：
   `psql -U postgres -hlocalhost -p6432`

3. 运行 `DATABASE_URL=postgres://postgres:123@localhost:6432/local_chain_db sqlx` 数据库创建在 `substrate-archive/src` 中创建数据库。

4. 设置 `CHAIN_DATA_DB="<your_path>"`。

5. 设置您的 `archive.conf` 文件：

   - 确保将您的基本路径设置为主数据库
   - 告诉它 RocksDB 在哪里。使用 CHAIN_DATA_DB 设置状态
   - 次级数据库是一种优化
   - postgres url（如果在生产环境中，则设置为变量）

6. （可选）设置日志记录和调试。

7. 运行节点模板。确保您在 `--release --dev base-path=/tmp/dir --pruning=archive` 中运行它

8. 使用您的节点模板进行交易。

9. 为您的目标链启动 `substrate-archive` 节点：
   `cargo run --release -- -c archive-conf.toml --chain=polkadot`

11. 打开 Web 浏览器并登录到 Postgres 管理控制台。

   - 默认 URL：localhost:16543
   - 默认用户名：pgadmin4@pgadmin.org
   - 默认密码：admin


12. 查看参考以开始创建您的查询。
