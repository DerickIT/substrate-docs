---
title: 部署选项和工具
description: 
keywords:
---

正如您在[常见部署目标](/deploy/prepare-to-deploy#common-deployment-targets)中看到的那样，有几种常见的部署场景。
您如何管理所提供的服务的节点在很大程度上取决于您是在物理硬件上部署、使用云提供商还是配置 Kubernetes 集群。
本节介绍了一些在不同环境中部署和管理节点的最常见选项和工具。

## Linux 服务器

如果您在使用 Linux 操作系统发行版的物理机或虚拟机上部署，则通常使用 `systemd` 来管理大多数服务。
您可以使用 `systemd` 来确保进程已启用并正在运行，设置服务重启策略，为主机运行的用户帐户指定策略，以及配置系统参数以限制内存使用和其他属性。

以下示例说明了如何为使用与 Alice 关联的帐户和本地用户名 `polkadot` 运行本地开发链的节点配置 `systemd` 文件：

```text
[Unit]
Description=Polkadot 验证器

[Service]
User=polkadot
ExecStart=/home/polkadot/polkadot  --dev --alice
Restart=always
RestartSec=90

[Install]
WantedBy=multi-user.target
```

出于演示目的，此文件名为 `polkadot.service` 并放置在 `/etc/systemd/system` 目录中。
然后，您可以通过运行以下命令启用该服务：

```bash
systemctl enable polkadot
```

要启用后启动服务，可以运行以下命令：

```bash
systemctl start polkadot
```

### 使用环境变量

如果您想从 `systemd` 配置中删除某些设置（例如，`--dev` 和 `--alice` 命令行选项），则可以在**环境变量**文件中配置这些设置。
使用环境变量文件，您可以为每个服务器配置适当的设置（在其自己的服务器特定文件中），但仍然可以使用 `systemd` 命令管理服务。
在此示例中，您将在 `/etc/default/polkadot` 中为本地主机创建一个新的环境变量文件，如下所示：

```text
START_OPTIONS="--dev --alice"
```

然后，您将修改 `systemd` 服务，使其如下所示：

```text
[Unit]
Description=Polkadot 验证器

[Service]
User=polkadot
EnvironmentFile=/etc/default/polkadot
ExecStart=/home/polkadot/polkadot  $START_OPTIONS
Restart=always
RestartSec=90

[Install]
WantedBy=multi-user.target
```

您可以将此技术与多个变量一起使用，以将配置详细信息从节点主机上运行的 `systemd` 文件中抽象出来。

### 本地日志记录

默认情况下，`systemd` 服务将输出写入本地 `syslog` 文件，通常为 `/var/log/syslog` 或 `/var/log/messages`。
您还可以使用 `journalctl` 命令查看此输出。
例如，要查看 `polkadot` 进程的最新输出，可以运行以下命令：

```bash
journalctl -u polkadot -f
```

要删除两天前之前的日志，可以运行类似以下的命令：

```bash
journalctl -u polkadot --vacuum-time=2d
```

要仅保留过去 1G 的数据，请运行：

```bash
journalctl --vacuum-size=1G
```

### 远程日志记录

如果您的部署包含许多主机，则可以使用 Loki 或 Elasticsearch 聚合来自多个来源的数据。

#### Loki

要记录到远程 `loki` 实例：

1. 在每台服务器上安装 `promtail` 服务器软件包。
2. 创建指定服务器和客户端信息以启用每台服务器将日志发送到远程主机的配置文件。

   例如：

   ```yaml
   # promtail 服务器配置
   server:
     http_listen_port: 9080
     grpc_listen_port: 0
     log_level: info
   positions:
     filename: /var/lib/promtail/positions.yaml

   # loki 服务器
   clients:
     - url: http://myloki.mycompany.com/loki/api/v1/push
       backoff_config:
         min_period: 1m
         max_period: 1h
         max_retries: 10000
   scrape_configs:
     - job_name: journald
       journal:
         max_age: 1m
         path: /var/log/journal
         labels:
           job: journald
       pipeline_stages:
         - match:
           selector: '{job="journald"}'
           stages:
             - multiline:
                 firstline: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3}'
                 max_lines: 2500
             - regex:
                 expression: '(?P<date>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3})\s+(?P<level>(TRACE|DEBUG|INFO|WARN|ERROR))\s+(?P<worker>([^\s]+))\s+(?P<target>[\w-]+):?:?(?P<subtarget>[\w-]+)?:[\s]?(?P<chaintype>\[[\w-]+\]+)?[\s]?(?P<message>.+)'
             - labels:
                 level:
                 target:
                 subtarget:
   ```

#### Elasticsearch

要记录到远程 Elasticsearch 集群：

1. 安装 `logstash` 软件包。
2. 创建指定服务器和客户端信息以启用每台服务器将日志发送到远程主机的配置文件。
   一个示例配置如下所示：

   ```text
   nput {
     journald {
       path      => "/var/log/journal"
       seekto => "tail"
       thisboot => true
       filter    => {
           "_SYSTEMD_UNIT" => "polkadot.service"
       }
       type => "systemd"
     }
   }

   filter {
     date {
       match => ["timestamp", "YYYY-mm-dd HH:MM:ss.SSS"]
       target => "@timestamp"
     }
     mutate {
       rename => [ "MESSAGE", "message" ]
       remove_field => [ "cursor", "PRIORITY", "SYSLOG_FACILITY", "SYSLOG_IDENTIFIER", "_BOOT_ID", "_CAP_EFFECTIVE", "_CMDLINE", "_COMM", "_EXE", "_GID", "_HOSTNAME", "_MACHINE_ID", "_PID", "_SELINUX_CONTEXT", "_STREAM_ID", "_SYSTEMD_CGROUP", "_SYSTEMD_INVOCATION_ID", "_SYSTEMD_SLICE", "_SYSTEMD_UNIT", "_TRANSPORT", "_UID" ]
     }
     if ([message] =~ ".*TRACE .*") { drop{ } }
     grok {
        match => { "message" => "%{NOTSPACE:thread} %{LOGLEVEL:log-level} %{NOTSPACE:namespace} %{GREEDYDATA:message}" }
     }
   }

   output {
      elasticsearch {
        hosts => ["https://myelasticsearch.mycompany.com:9243"]
        user => "username"
        password => "password"
        index => "logstash-polkadot-%{+YYYY.MM.dd}"
      }
   }
   ```

### 记录命令行选项

启动节点时，可以使用命令行选项指定要记录活动的日志级别和目标组件。
默认情况下，所有目标组件都设置为 `info` 日志级别。
您可以使用 `--log` 或 `-l` 命令行选项调整各个组件的日志级别。
例如，要更改 afg 和 sync 组件的日志级别：

```bash
--log afg=trace,sync=debug
```

要将所有组件的日志级别更改为 `debug`：

```bash
-ldebug.
```

从最低到最高详细度的有效日志级别为 `error`、`warn`、`info`、`debug` 和 `trace`。

日志记录的有效目标为：

```text
afg
aura
babe
beefy
db
gossip
header
peerset
pow
rpc
runtime
runtime::contracts
sc_offchain
slots
state-db
state_tracing
sub-libp2p
sync
telemetry
tracing
trie
txpool
```

## 云配置

有多种在云提供商上配置节点的选项。
一些用于使用云资源部署的工具是特定于提供商的，而一些工具是与提供商无关的。

以下特定于提供商的工具最常用于在 AWS、Microsoft Azure 或 Google Cloud 上部署：

- [Amazon Cloud Formation](https://aws.amazon.com/cloudformation/)
- [Azure 资源管理器](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview)
- [Google Cloud 部署管理器](https://cloud.google.com/deployment-manager/docs)

这些特定于提供商的部署工具易于使用并提供重要的资源，包括示例代码、文档和支持。
但是，如果您使用多个提供商（每个提供商都有自己的脚本格式和配置要求），即使对基础设施进行基本更改也可能需要更改每个提供商的多个代码部分才能执行相同操作。

作为特定于提供商的工具的替代方案，[Terraform](https://www.terraform.io/) 提供了一种更通用的基础设施配置解决方案。
使用 Terraform，您可以一次指定更改并在多个提供商中应用该更改。

### Terraform

Terraform 使用[HashiCorp 配置语言 (HCL)](https://developer.hashicorp.com/terraform/language) 支持超过 2000 种不同的云资源[提供商](https://registry.terraform.io/browse/providers)，包括 AWS、Azure 和 GCP 这三个主要提供商。

该配置语言使您能够抽象配置详细信息，并使用相同的代码用于开发、测试和生产环境，而无论您使用哪个提供商，并通过源代码版本控制管理对基础设施的所有更改。
Terraform 还使您能够使用通用语言将独立资源合并到您的基础设施中。
例如，您可以使用单个配置文件部署 RPC 节点以及前端负载均衡器。

准备主机进行部署后，可以使用 Terraform 从映像预配置主机所需的软件或运行预配置基本映像的脚本。

您可以在[polkadot-validator-setup](https://github.com/w3f/polkadot-validator-setup)中找到使用 Terraform 与多个提供商的示例。

### Ansible

部署基本主机后，必须使用所需的软件组件、配置文件和系统设置对其进行配置。
除了 Terraform 或云提供商工具之外，Ansible 还提供了一种灵活的方式，您可以通过它来自动化基础设施部署。

Ansible 使用**剧本**来编排、配置、管理和部署系统组件。
通过结合使用剧本和**角色**，您可以为一组节点实现特定的配置或行为。

部署区块链节点时，Ansible 使您能够定义一个**清单**，该清单描述主机以及如何根据其角色对主机进行分组（例如，在识别主机为验证器、协调器或 rpc 节点的组中）。
然后，您可以调用一个剧本将清单中的主机和组与要在每个主机上执行的角色链接起来。

您可以在[ansible-galaxy](https://github.com/paritytech/ansible-galaxy)和[节点角色](https://github.com/paritytech/ansible-galaxy/tree/main/roles/node)中找到使用 ansible 的示例。

## Kubernetes

只有在您拥有管理 Kubernetes 配置的先前经验时，才应在 Kubernetes 集群上部署。
在 Kubernetes 集群中管理基于 Substrate 的节点的主要工具是**Helm 图表**，您可以使用它来部署节点，以及**测试网管理器**，您可以使用它来部署和维护 Kubernetes 集群中的测试网络。
请注意，在使用这些工具之前，您必须能够访问 Kubernetes 集群、本地副本的 `kubectl` 和已安装的 Helm。

### Helm 图表

[Parity Helm 图表](https://github.com/paritytech/helm-charts) 是一个 Helm 图表集合，用于定义、安装、管理和升级 Substrate 和 Polkadot 组件。
在集合中，[节点](https://github.com/paritytech/helm-charts/tree/main/charts/node) 图表用于部署 Substrate 或 Polkadot 节点二进制文件。
图表的所有参数都在[节点图表 README.md](https://github.com/paritytech/helm-charts/tree/main/charts/node)中进行了说明。

需要注意的最重要的参数是：

| 选项                  | 描述                                                   |
| :---------------------- | :------------------------------------------------------------ |
| node.chain              | 要连接的网络。                                        |
| node.command            | 要使用的二进制文件。                                                |
| node.flags              | 要与容器中的二进制文件一起使用的命令行选项。 |
| node.customChainspecUrl | 自定义链规范 URL。                               |

还有一个示例 `values.yml` 配置文件，您可以从该文件开始工作。

以下示例说明了如何在 Kubernetes 中使用两个验证器和两个完整节点部署 `rococo-local` 测试网络链。

要使用 Helm 图表部署 `rococo-local` 链：

1. 验证您是否可以访问 Kubernetes 集群并安装了 Helm 客户端。
2. 通过运行以下命令将 Parity 图表存储库添加到 Helm：

   ```bash
   helm repo add parity https://paritytech.github.io/helm-charts/
   ```

3. 通过运行以下命令安装节点图表：

   ```bash
   helm install polkadot-node parity/node
   ```

4. 通过运行以下命令使用 Alice 帐户和自定义节点密钥部署验证器节点：

   ```bash
   helm install rococo-alice parity/node --set node.role="validator" \
      --set node.customNodeKey="91cb59d86820419075b08e3043cd802ba3506388d8b161d2d4acd203af5194c1" \
      --set node.chain=rococo-local \
      --set node.perNodeServices.relayP2pService.enabled=true \
      --set node.perNodeServices.relayP2pService.port=30333 \
      --set node.flags="--alice --rpc-external --ws-external --rpc-cors all --rpc-methods=unsafe"
   ```

   此命令将节点 `alice` 部署为有状态服务，并使用示例自定义节点密钥以及用作所有其他主机的引导节点的服务。

5. 通过运行以下命令使用 Bob 帐户和 `alice` 作为引导节点部署验证器节点：

   ```bash
   helm install rococo-bob parity/node --set node.role="validator" \
      --set node.chain=rococo-local \
      --set node.flags="--bob --bootnodes '/dns4/rococo-alice-node-0-relay-chain-p2p/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC'"
   ```

   两个验证器都运行后，链应该开始生成区块。

6. 通过运行以下命令部署两个完整节点：

   ```bash
   helm install rococo-pool parity/node --set node.chain=rococo-local \
      --set node.replicas=2 \
      --set node.flags="--bootnodes '/dns4/rococo-alice-node-0-relay-chain-p2p/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC'"
   ```

   完成这些步骤后，您将拥有一个具有两个验证器和两个完整节点的工作 `rococo-local` 测试链。

以下工具（从最简单到最先进）对于管理 Helm 版本很有用。

- [Helmfile](https://github.com/roboll/helmfile)
- [Terraform Helm 提供程序](https://registry.terraform.io/providers/hashicorp/helm/latest/docs)
- [Flux CD](https://fluxcd.io/)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)

## Docker

如果您将节点作为虚拟机部署在网络中，则可以使用 Docker 映像为不同类型的节点准备节点配置。
例如，您可以为验证器节点和 RPC 提供程序节点准备 Docker 映像，然后部署每种类型的多个节点，而无需为每个节点配置单独的虚拟机。
然后，外部节点操作员可以使用您提供的 Docker 映像在需要时随时随地部署新节点。

### 示例 Dockerfile

以下示例 Dockerfile 说明了以安全的方式构建 Docker 映像的最佳实践，从而最大程度地减少攻击面。
此示例类似于用于创建官方 Polkadot 映像的 Dockerfile 的版本。
您还可以查阅 Docker 文档以获取有关[编写 Dockerfile 的最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)的更多信息。

```dockerfile
# 这是节点模板的示例构建阶段。在这里，我们在临时映像中创建二进制文件。

# 这是构建 Substrate 节点的基本映像
FROM docker.io/paritytech/ci-linux:production as builder

WORKDIR /node-template
COPY . .
RUN cargo build --locked --release

# 这是第 2 阶段：一个非常小的映像，我们在这里复制二进制文件。
FROM docker.io/library/ubuntu:20.04
LABEL description="Substrate 节点模板的多阶段 Docker 映像" \
  image.type="builder" \
  image.authors="you@email.com" \
  image.vendor="Substrate 开发者中心" \
  image.description="Substrate 节点模板的多阶段 Docker 映像" \
  image.source="https://github.com/substrate-developer-hub/substrate-node-template" \
  image.documentation="https://github.com/substrate-developer-hub/substrate-node-template"

# 复制节点二进制文件。
COPY --from=builder /node-template/target/release/node-template /usr/local/bin

RUN useradd -m -u 1000 -U -s /bin/sh -d /node-dev node-dev && \
  mkdir -p /chain-data /node-dev/.local/share && \
  chown -R node-dev:node-dev /chain-data && \
  ln -s /chain-data /node-dev/.local/share/node-template && \
  # 清理并最大程度地减少攻击面
  rm -rf /usr/bin /usr/sbin && \
  # 检查可执行文件是否在此容器中工作
  /usr/local/bin/node-template --version

USER node-dev

EXPOSE 30333 9933 9944 9615
VOLUME ["/chain-data"]

ENTRYPOINT ["/usr/local/bin/node-template"]
```

### 自动化构建管道

以下示例[GitHub 操作](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/.github/workflows/release.yml) 将 Docker 映像构建并发布到 DockerHub。
在大多数情况下，您使用手动工作流或发布新版本时触发此操作。

请注意，您必须按照[加密机密](https://docs.github.com/en/actions/security-guides/encrypted-secrets)中的说明将机密添加到您的 GitHub 存储库或组织中，以安全地发布映像。
您还需要将 DockerHub 帐户的凭据保存在您的 GitHub 机密中。
如果您改为想要使用其他映像存储库（例如，GitHub 映像注册表），则可以修改“构建和推送 Docker 映像”步骤。

```yaml
# 您需要将以下机密添加到您的 GitHub 存储库或组织中才能使其工作
# - DOCKER_USERNAME：DockerHub 帐户的用户名。例如，parity
# - DOCKER_TOKEN：DockerHub 的访问令牌，请参阅https://docs.docker.com/docker-hub/access-tokens/。例如，VVVVVVVVV-WWWW-XXXXXX-YYYY-ZZZZZZZZZ
# 以下在下面设置为环境变量
# - DOCKER_REPO：DockerHub 存储库的唯一名称。例如，parity/polkadot

name: 构建和发布 Docker 映像

# 控制操作何时运行。
on:
  # 触发主分支上的推送事件的工作流
  # push:
  # branches: [ main ]

  # 允许您从“操作”选项卡手动运行此工作流
  workflow_dispatch:

# 设置环境变量（可以覆盖）以用于 Docker 存储库
env:
  DOCKER_REPO: parity/polkadot

# 工作流运行由一个或多个作业组成，这些作业可以顺序或并行运行
jobs:
  build:
    # 作业将在其上运行的运行程序类型
    runs-on: ubuntu-22.04

    # 步骤表示将作为作业的一部分执行的任务序列
    steps:
      # 在 $GITHUB_WORKSPACE 下检出您的存储库，以便您的作业可以访问它
      - name: 检出存储库
        uses: actions/checkout@v2.5.0

      # 使用存储在存储库机密中的凭据登录 Docker 集线器
      - name: 登录 Docker 集线器
        uses: docker/login-action@v2.1.0
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      # 获取提交短哈希，用作修订版
      - name: 计算修订版哈希
        id: rev
        run: echo "value=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      # 构建和推送 2 个映像，一个带有版本标签，另一个带有最新标签
      - name: 构建和推送 Docker 映像
        uses: docker/build-push-action@v3.2.0
        with:
          context: .
          push: true
          tags: ${{ env.DOCKER_REPO }}:v${{ steps.rev.outputs.value }}, ${{ secrets.DOCKER_REPO }}:latest
