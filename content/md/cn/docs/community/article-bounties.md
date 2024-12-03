---
title: 提交文章以获得奖励
slug: /v3/contribute/bounties
version: "3.0"
section: docs
category: style guide
keywords:
  - 贡献
  - 风格指南
  - 奖励
---

<Message
type="yellow"
title="信息"
text={`此部分仍在开发中。虽然我们计划很快推出官方的 Substrate 开发者中心奖励计划，但目前还没有。`}
/>

为了鼓励社区支持和对开发者生态系统的贡献，我们建立了一个奖励计划。
该奖励计划为内容开发者提供奖励——以 XXX 的形式——他们提交的文章扩展和改进了
Substrate 文档，涵盖了新的“操作指南”类型主题。

## 参与

要参与该计划：

1. 在 Substrate 开发者中心文档存储库的 [问题](https://github.com/substrate-developer-hub/substrate-docs/issues) 页面上，选择 **操作指南** 和 **新内容**
   标签以筛选显示的问题列表。

1. 选择您感兴趣的需要指南的问题。

例如：

- [如何从另一个模块调用函数](https://github.com/substrate-developer-hub/substrate-docs/issues/75)

- [如何使用基准测试来计算权重](https://github.com/substrate-developer-hub/substrate-docs/issues/88)

如果您想贡献的主题没有对应的 issue，请先创建一个 issue 描述您想涵盖的内容以及至少一个适用的用例。了解社区关心的问题对于帮助我们确定要涵盖的主题的优先级非常有价值。

1. 使用 [操作指南模板](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/contribute-templates/how-to-template.md) 来组织您主题的信息。

请确保您的文章包含以下 **必填** 部分。

- 概述
- 步骤
- 示例

**如果适用**，您的文章还应包含以下部分：

- 用例
- 开始之前
- 相关资源

提交之前，请验证您的内容是否符合以下要求：

- 遵循操作指南模板结构。
- 在指南标题或概述部分陈述明确的目标。
- 专注于实现既定目标。
- 包含指向有效代码或示例存储库的链接。
- 在相关情况下提供支持性参考。

1. 添加适当的标签来描述您的内容 [复杂度] 和 [类别]。

1. 在 [操作指南](https://github.com/substrate-developer-hub/substrate-docs/blob/main/content/md/en/docs/reference/how-to-guides/index.md) 存储库中创建一个分支。

1. 为您要贡献的文章创建一个拉取请求 (PR)。

1. 为您的文章的 PR 添加 **奖励提交** 标签。

1. 使用指向您文章的 PR 的链接更新您选择或创建的问题。

您提交的文章将在拉取请求审查中进行评估，并根据以下标准进行评判：

- 实用性。您的文章涵盖的材料不存在，并且至少提供了一个明确的用例。

- 结构。您已遵循 [操作指南模板](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/contribute-templates/how-to-template.md) 结构以及 [贡献者指南] 中描述的约定。

- 正确性和完整性。每个步骤都清晰、正确且完整地阐述。

- 可重复性。这些步骤始终如一地达到预期结果。
