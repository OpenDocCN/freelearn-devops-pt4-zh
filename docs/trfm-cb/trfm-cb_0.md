前言

**基础设施即代码**，通常称为 **IaC**，是一种 DevOps 文化的支柱实践。IaC 涉及用代码编写你期望的架构配置。除了其他优点外，IaC 还允许基础设施部署的自动化，减少或消除手动干预的需求，从而降低配置错误的风险，并且通过模块化和可扩展的代码创建模板并标准化基础设施。

在所有 DevOps 工具中，有许多支持基础设施即代码（IaC）。其中之一是 **Terraform**，由 HashiCorp 开发，今天非常流行，因为它不仅是开源和跨平台的，还具有以下优点：

+   它允许你预览将应用到基础设施的更改。

+   它支持操作的并行化，并考虑到依赖关系的管理。

+   它拥有众多的提供者。

在这本关于 Terraform 的书中，我们将首先讨论 Terraform 在本地开发中的安装，Terraform 配置的编写，以及动态构建多个环境。

然后，我们将学习如何使用 Terraform **命令行界面** (**CLI**) 以及如何共享和使用 Terraform 模块。

一旦理解了 Terraform 中的配置编写和命令使用，我们将讨论 Terraform 在构建基础设施时的实际应用，特别是在领先的云服务提供商 Azure 上的应用。

最后，我们将通过探讨 Terraform 的高级使用方法来结束本书内容，包括 Terraform 测试，将 Terraform 集成到 **持续集成/持续部署** (**CI/CD**) 流水线中，以及使用 Terraform Cloud，它是 Terraform 为团队和公司提供的协作平台。

本书将通过若干最佳实践食谱，指导你如何编写 Terraform 配置和命令，并涵盖 Terraform 与其他工具的集成食谱，如 Terragrunt、kitchen-terraform 和 Azure Pipelines。

本书中描述的大多数 Terraform 配置都基于 Azure 提供者作为示例，但你可以将这些实践应用于所有其他 Terraform 提供者。

在编写这本以食谱格式呈现的书籍时，我希望分享我通过与客户和公司多年的合作，积累的实际 Terraform 场景经验。

# 第一章：本书适用对象

本书面向那些已经了解 IaC 和 DevOps 文化基础，并且对 Terraform 有一定基础知识的读者。本书不要求具备开发或操作系统方面的经验。

# 本书内容概览

第一章，*设置 Terraform 环境*，详细介绍了手动安装 Terraform、使用脚本安装或通过 Docker 容器安装的不同方式，同时还详细介绍了 Terraform 迁移配置过程。

第二章，*编写 Terraform 配置*，涉及为提供程序编写 Terraform 配置，变量、输出、外部数据、内置函数、条件表达式、文件操作以及使用 Terraform 执行本地程序。

第三章，*使用 Terraform 构建动态环境*，讨论了通过使用循环、映射和集合深入编写 Terraform 配置，构建动态环境。

第四章，*使用 Terraform CLI*，解释了如何使用 Terraform 的 CLI 来清理代码、销毁 Terraform 提供的资源、使用工作空间、导入现有资源、生成依赖关系图以及调试 Terraform 执行。

第五章，*使用模块共享 Terraform 配置*，涵盖了 Terraform 模块的创建、使用和共享。还展示了模块测试实践以及实现 Terraform 模块的 CI/CD 流水线。

第六章，*使用 Terraform 配置 Azure 基础设施*，展示了如何在云服务提供商 Azure 的实际场景中使用 Terraform。涵盖了身份验证、远程后端、ARM 模板、Azure CLI 执行以及为现有基础设施生成 Terraform 配置等主题。

第七章，*深入探索 Terraform*，讨论了与 Terraform 更深入相关的主题，如执行 Terraform 配置测试、零停机时间部署、使用 Terragrunt 的 Terraform 包装器，以及实现 CI/CD 流水线来执行 Terraform 配置。

第八章，*使用 Terraform Cloud 改善协作*，解释了如何在团队中使用 Terraform Cloud 运行 Terraform，通过私有注册表共享 Terraform 模块、使用远程后端存储 Terraform 状态、远程运行 Terraform 以及使用 Sentinel 编写合规性测试。

# 为了充分利用本书

以下是本书的软件/硬件先决条件列表：

| **书中涉及的软件/硬件** | **操作系统要求** |
| --- | --- |
| Terraform 版本 >= 12.0 | 任何操作系统 |
| Terraform Cloud | 无 |
| Azure | 任何操作系统 |
| PowerShell 脚本 | Windows/Linux |
| Shell 脚本 | Linux |
| Azure CLI | 任何操作系统 |
| Azure DevOps | 无 |
| GitHub | 无 |
| Git | 任何操作系统 |
| Ruby 版本 2.6 | 任何操作系统 |
| Docker | 任何操作系统 |

**如果您使用的是本书的电子版，我们建议您自己输入代码，或者通过 GitHub 仓库访问代码（下节会提供链接）。这样可以帮助您避免复制粘贴代码时可能出现的错误。**

## 下载示例代码文件

您可以从您的[www.packt.com](http://www.packt.com)帐户下载本书的示例代码文件。如果您从其他地方购买了本书，可以访问[www.packtpub.com/support](https://www.packtpub.com/support)，并注册以将文件直接发送到您的邮箱。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“Support”标签。

1.  点击“Code Downloads”。

1.  在“Search”框中输入书名并按照屏幕上的指示操作。

一旦文件下载完成，请确保使用最新版本的工具解压或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Terraform-Cookbook`](https://github.com/PacktPublishing/Terraform-Cookbook)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供来自我们丰富书籍和视频目录的其他代码包，您可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看。快来看看吧！

## Code in Action

本书的《Code in Action》视频可以在[`bit.ly/3h7cNME`](https://bit.ly/3h7cNME)观看。

## 下载彩色图像

我们还提供了一份包含本书中截图/图示彩色图像的 PDF 文件。您可以在此下载：[`static.packt-cdn.com/downloads/9781800207554_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781800207554_ColorImages.pdf)。

## 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。这里有一个例子：“请注意，`current_version`属性包含最新的 Terraform 版本，它是一个值。”

代码块的格式如下：

```
TERRAFORM_VERSION=$(curl -s https://checkpoint-api.hashicorp.com/v1/check/terraform | jq -r .current_version)

....
```

当我们希望将您的注意力集中在代码块的特定部分时，相关的行或项目将以粗体显示：

```
FROMgolang:latest
ENVTERRAFORM_VERSION=0.12.29
RUNapt-get update && apt-get install unzip \
    && curl -Os
```

任何命令行输入或输出的格式如下：

```
docker exec tfapp terraform init
docker exec tfapp terraform plan
docker exec tfapp terraform apply --auto-approve
```

**粗体**：表示一个新术语、一个重要词汇或您在屏幕上看到的词。例如，菜单或对话框中的词语在文本中将这样显示。这里有一个例子：“通过点击扩展的安装按钮来执行此操作。”

警告或重要提示以这样的形式出现。

提示和技巧以这样的形式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`与我们联系。

**勘误**：虽然我们已经尽力确保内容的准确性，但错误还是有可能发生。如果你在本书中发现错误，我们将不胜感激，若你能报告给我们。请访问 [www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择你的书籍，点击勘误提交表单链接，并填写相关细节。

**盗版**：如果你在互联网上发现任何我们作品的非法复制版本，我们将不胜感激，如果你能提供该内容的地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上相关链接。

**如果你有兴趣成为一名作者**：如果你在某个领域有专业知识，并且有兴趣写书或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

## 评审

请留下评论。在阅读并使用本书后，何不在你购买书籍的网站上留下评论呢？潜在读者可以参考你的公正意见来做出购买决策，我们在 Packt 可以了解你对我们产品的看法，我们的作者也能看到你对他们书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
