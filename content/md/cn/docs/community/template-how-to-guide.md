---
title: 模板 - 操作指南
description:
keywords:
  - 模板
  - 贡献
  - 风格指南
---

我们建议您使用以下模板来构建您想作为“操作指南”主题提交的文章。
您可以直接从 [此处](/assets/contribute-templates/how-to-template.md) 下载 Markdown 操作指南模板的副本。
下载模板后，重命名文件并将每个部分的描述替换为相关内容。

## 前置信息

您的 _操作指南_ 应以由在第一行键入三个破折号分隔的前置信息部分开头。
前置信息包含以下字段：

`---`

title: 保持标题简短

description: 编写一个描述性句子来总结文章的内容。

keywords:

`---`

只有 title 字段是必需的，您可以指定其他元数据字段。
您通过在最后一个前置信息条目后面的行上键入三个破折号来关闭前置信息部分。

### [指南标题]

指南标题应总结文章的目标。
对于“操作指南”，标题应完成“如何…？”句子。
例如，如果指南的目标是说明“如何铸造代币供应？”，您可以像这样在前置信息中设置标题：

`title: 铸造代币供应`

通常，您应保持标题简短，以便于扫描关键字。

### [指南描述]

指南描述是可选的，但如果您包含它，请使用一个句子来传达标题未传达的有关内容的任何其他信息。
例如：

`description: 说明如何铸造由单个帐户拥有的代币供应。`

### [指南关键字]

关键字是可选的，但如果您包含它们，请缩进两个空格，然后在每行使用破折号和单个关键字。
有关示例，请参阅 [操作指南模板](/assets/contribute-templates/how-to-template.md)。

## 导言

文章的第一段应简要概述文章的内容以及此信息对读者有何用处。
概述部分不需要 **概述** 标题，它可能不止一段。

每篇文章的开头部分为后续内容奠定了基础，应回答显而易见的问题，以便读者可以决定内容是否与他们相关。
仅通过阅读本节，读者就应该知道——他们是否应该继续，或者内容与他们无关，他们应该继续其他内容。

对于您的概述，请尝试回答以下问题：

- 这篇文章是关于什么的？

- 遵循该过程或技术要实现的目的或目标是什么？

- 为什么有人会想要使用此过程或技术？例如，是否有适用的特定用例场景？

- 某人何时会使用此过程？例如，这是否是一次性活动或重复活动？它是一种模式还是一个独特的案例？

- 该过程或技术在何处适用？

- 谁会使用此过程或技术？例如，是否需要特殊技能？是否适用特定权限或限制？

概述部分也是链接到其他资源（包括其他指南）的好地方。
作为内容创建者，您希望读者相信该指南对他们有用。

### 用例

本节是可选的，因为如果您的文章严格关注单个用例，则指南标题可能就足够了。

如果您的文章有多个实际应用，请使用本节简要描述每个应用。

如果唯一的用例是文章标题的重复或在概述部分中已充分涵盖，请跳过本节。

如果您的文章只有一个用例，但它需要比标题提供的更多解释，请添加本节和一个或多个句子以提供其他解释。例如：

- 本指南说明了为支付费用实现第二种货币。如果您想在您的运行时中支持多种货币，本指南提供了您可以应用于您想要支持的其他货币的实用建议和详细步骤。

- 本指南向您展示如何执行从 `Vec<u32>` 到 SomeStruct 的运行时迁移

如果您的文章有多个用例，请使用项目符号列表。

## 开始之前

本节是可选的，但 **建议** 使用。
使用“开始之前”标题，并使用节正文描述适用于您文章的任何先决条件。

本节应回答以下问题：

- 在阅读本文之前，某人应该 **拥有** 什么？

- 在阅读本文之前，某人应该 **了解** 什么？

- 在阅读本文之前，某人应该 **做什么**？

## 程序步骤

仅当文章有一组步骤来实现单个目标时，才使用 **步骤** 作为标题。
例如，如果文章严格关注单个用例，并且更具描述性的标题只会重复文章标题，则使用步骤。

对于更复杂的过程和技术，请使用清晰简洁的标题来描述过程或技术的每个部分。

每个步骤都应以动作为主导。
在大多数情况下，每个步骤都以动词开头，并以句点结尾。
步骤后面的段落应描述读者应该预期的结果或结果。
如果您觉得某个步骤需要任何其他信息，请链接到该信息，而不是在步骤中嵌入太多无关的细节。

代码片段可以帮助说明步骤，但不应压倒“如何执行此操作”（而不是“我该做什么”）的重点。

请记住，大多数步骤都有结果，读者希望在他们完成过程的过程中确认他们已采取了正确的操作。

## 示例

本节是可选的，但 **建议** 使用。
您可以使用本节提供指向一个或多个基于代码的示例的链接，这些示例对您的文章进行了实际应用。
本节应至少包含一个指向存储库的引用，该存储库以工作示例的形式公开本指南涵盖的内容。
您可以使用 Substrate 中的现有代码库或任何公开可用的存储库中的新代码进行引用。
例如，如果您有一个存储库，您在其中测试了您正在编写的过程，请在本节中包含指向它的链接。

## 相关资源

本节是可选的。
如果您包含它，请添加指向类似指南、其他开发者中心资源或相关材料的项目符号列表链接。
例如，您可以添加指向其他操作指南、教程或 Rust 文档的链接。