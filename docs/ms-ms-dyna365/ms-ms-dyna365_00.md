# 序言

本书适用于需要在 Microsoft Dynamics **企业资源规划**（**ERP**）之上创建应用程序和定制的 Dynamics 365 Business Central 开发人员。

本书首先解释了 Microsoft Dynamics 365 Business Central 平台（重点介绍**软件即服务**（**SaaS**）平台）以及现代开发环境（Visual Studio Code、AL 语言和其他工具，如 Docker）。然后，本书涵盖了所有开发者需要了解的扩展和自定义 ERP 的主题，重点是如何编写更好的代码，遵循最佳实践，如何调试和部署扩展，以及如何编写自动化测试。

本书还涵盖了与集成、云服务和无服务器处理相关的高级主题（包括使用 Dynamics 365 Business Central 与 Office 365 应用程序，如 Power BI、Flow 和 PowerApps 的集成；与机器学习功能的集成；以及如何将 DevOps 技术应用到开发过程中）。

本书的最后一个主题是*架构*，分析了将现有解决方案迁移到基于扩展的新开发模型的最佳方法。

# 本书的读者对象

本书面向具有业务流程和 C/AL 编程高级知识的 Microsoft Dynamics NAV/Dynamics 365 Business Central 开发人员和解决方案架构师。了解 Web 服务编程（API）和 C# 会是加分项。

# 本书涵盖的内容

第一章，*Microsoft Dynamics 365 Business Central 概述*，提供了 Dynamics 365 Business Central 架构（云端和本地）的技术介绍，并介绍了新的开发平台。

第二章，*掌握现代开发环境*，概述了适用于 Dynamics 365 Business Central 的现代开发环境，并提供了如何在开发扩展时高效使用 Visual Studio Code 的技巧和窍门。

第三章，*基于在线和容器的沙盒*，详细介绍了如何为开发创建 Dynamics 365 Business Central 沙盒环境，如何使用在线沙盒进行测试，以及如何使用 Docker 改善开发过程。

第四章，*扩展开发基础*，提供了新的扩展模型的概述，解释了与过去的不同、新的对象类型，以及如何使用 AL 创建对象。

第五章，*为 Dynamics 365 Business Central 开发定制化解决方案*，指导你使用 Visual Studio Code 和 AL 语言从实际业务案例出发，开发完整的 Dynamics 365 Business Central 解决方案。在本章中，你将看到如何创建一个完整的解决方案，以及如何为客户定制解决方案，而不修改基础代码。

第六章，*高级 AL 开发*，讲解了开发扩展时需要了解的高级编程主题，如文件管理、使用.NET 对象、使用 AL 调用 Web 服务、处理 XML 和 JSON、处理媒体文件、处理通知、异步编程等。

第七章，*使用 AL 进行报表开发*，介绍了如何在新的扩展模型中处理报表（新报表的创建和现有报表的定制）。

第八章，*安装和升级扩展*，教你如何处理 Dynamics 365 Business Central 扩展的安装和升级逻辑。

第九章，*调试*，教你如何调试扩展以及如何检查代码和变量。

第十章，*使用 AL 进行自动化测试开发*，详细介绍了如何为 Dynamics 365 Business Central 扩展编写和执行自动化测试。

第十一章，*与 Business Central 的源代码管理和 DevOps*，讲解了在为 Dynamics 365 Business Central 开发解决方案时，如何高效地处理源代码管理技术，并概述了如何处理**持续集成/持续交付**（**CI/CD**）和 DevOps 技术，适用于你的扩展项目。

第十二章，*Dynamics 365 Business Central API*，介绍了 Dynamics 365 Business Central 的 API 框架。它展示了如何使用现有的 API，如何创建新的 API 来扩展平台，以及一些高级 API 主题，如绑定操作和 Webhooks。

第十三章，*使用 Business Central 和 Azure 实现无服务器业务流程*，描述了如何将 Azure Functions 与 Dynamics 365 Business Central 结合使用，在云中执行.NET 代码并实现无服务器处理解决方案。

第十四章，*使用 Azure Functions 进行监控、扩展和 CI/CD*，教你如何监控 Azure Functions，如何处理可扩展性以提高性能，以及如何使用 Azure DevOps 实现 CI/CD 过程。

第十五章，*Business Central 与 Power Platform 的集成*，展示了将 Dynamics 365 Business Central 与 Dynamics 365 Power Platform 结合使用的方式。我们将概览 Power Platform 应用，并通过与 Dynamics 365 Business Central 配合使用 Power Platform 的一些实际应用。

第十六章，*将机器学习集成到 Dynamics 365 Business Central*，概述了如何将机器学习功能集成到 Dynamics 365 Business Central 扩展中。

第十七章，*将现有 ISV 解决方案迁移到新的扩展模型*，讲解了如何将一个现有的 ISV 解决方案（使用旧的 C/AL 代码编写）迁移到新的扩展模型。内容详细讲解了架构方面的内容，并提供了如何正确开始该过程的建议。

第十八章，*AL 开发者的实用工具*，介绍了一些对开发经验非常有帮助的第三方工具。

# 获取本书最大收益

你应该熟悉使用 C/AL 编程语言进行 Dynamics NAV 或 Dynamics 365 Business Central 的开发，并且能够使用 PowerShell 和 Azure 门户等工具。了解云计算概念将大大帮助你理解某些话题。

本书将指导你如何创建完整的解决方案并解决任务。如果你希望从本书中获得最大收益，至少在试用环境中跟随提供的示例进行操作是必须的。

# 下载示例代码文件

你可以从你的帐户在 [www.packt.com](http://www.packt.com) 下载本书的示例代码文件。如果你是从其他地方购买本书的，可以访问 [www.packtpub.com/support](https://www.packtpub.com/support) 注册并让我们直接通过电子邮件将文件发送给你。

你可以按照以下步骤下载代码文件：

1.  登录或注册到 [www.packt.com](http://www.packt.com)。

1.  选择 Support 标签。

1.  点击 Code Downloads。

1.  在 Search 框中输入书名并按照屏幕上的指示操作。

下载文件后，请确保使用最新版本的解压工具解压文件夹：

+   Windows 版的 WinRAR/7-Zip

+   Zipeg/iZip/UnRarX for Mac

+   Linux 版的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，地址是 [`github.com/PacktPublishing/Mastering-Microsoft-Dynamics-365-Business-Central`](https://github.com/PacktPublishing/Mastering-Microsoft-Dynamics-365-Business-Central)。如果代码有更新，将会在现有的 GitHub 仓库中进行更新。

我们还有来自丰富书籍和视频目录的其他代码包，可以在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 查找。快去看看吧！

# 下载彩色图片

我们还提供一份包含本书中使用的截图/图表的彩色图像的 PDF 文件，您可以在这里下载：[`static.packt-cdn.com/downloads/9781789951257_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781789951257_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词语、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账户名。举个例子：“这是一个 `docker run` 命令的输出，其中该镜像在本地不可用。”

一段代码如下所示：

```
mcr.microsoft.com/businesscentral/onprem:1910-cu1-au-ltsc2019
```

所有命令行输入或输出都以以下方式书写：

```
docker network create -d transparent transpNet
```

**粗体**：表示新术语、重要单词或屏幕上出现的词语。例如，菜单或对话框中的词语在文本中是这样显示的。举个例子：“只需浏览 [`dynamics.microsoft.com/en-us/business-central/overview/`](https://dynamics.microsoft.com/en-us/business-central/overview/)，然后在所需的许可模块下点击 'Find a partner'。”

警告或重要提示以此方式出现。

提示和技巧以此方式出现。

# 与我们联系

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`与我们联系。

**勘误表**：虽然我们已尽力确保内容的准确性，但错误难免。如果您在本书中发现错误，我们将非常感激您能向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击 “Errata 提交表单” 链接，并输入详细信息。

**盗版**：如果您在互联网上发现任何形式的非法复制我们作品的情况，我们将非常感激您能提供相关网址或网站名称。请通过 `copyright@packt.com` 联系我们并附上相关内容的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣写书或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。阅读并使用本书后，为什么不在您购买书籍的网站上留下评论呢？潜在读者可以通过您的公正意见做出购买决策，我们可以了解您对我们产品的看法，作者也能看到您对其书籍的反馈。谢谢！

欲了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
