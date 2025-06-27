# 前言

# 本书与技术简介

本书介绍了 Prometheus，第二个从 **云原生计算基金会** (**CNCF**) 毕业的项目，将帮助你巩固监控的核心基础知识，以及确保所需基础设施可见性的可用方法。它通过实践示例、使用测试环境和图表的方式，以易于消化的方式传达知识。

本书的内容设计旨在确保涵盖所有重要的 Prometheus 堆栈概念。我们在编写过程中的主要目标是以我们过去的自己为目标，确保他们可以在本书中找到关于这项技术的所有必要知识。

从运行一个 Prometheus 服务器，到可用的扩展选项，从创建和测试告警规则，到模板化 Slack 通知；以及从有用的仪表盘，到自动化目标发现；许多其他主题将被解释，以确保在使用 Prometheus 作为基石的基础设施监控中建立完整的知识体系。

# 本书适合的人群

如果你是软件开发人员、云计算专家、站点可靠性工程师、DevOps 爱好者或系统管理员，想要搭建一个可靠的监控和告警系统来维持基础设施的安全性和性能，那么本书适合你。基本的网络和基础设施监控知识将帮助你理解本书中涉及的概念。

# 本书内容概述

第一章，*监控基础知识*，奠定了本书中多个关键概念的基础。本章还探讨了 Prometheus 在度量收集方面的方式，以及为何一些有争议的决策对其架构设计至关重要。

第二章，*Prometheus 生态系统概述*，提供了整个 Prometheus 生态系统的高级概述，介绍了各个组件的职能，以及它们如何在逻辑上协同工作。

第三章，*设置测试环境*，介绍了如何使用本书提供的测试环境的基础知识，以及如何操作这些环境来验证不同的配置。

第四章，*Prometheus 度量基础*，探讨了度量，这是 Prometheus 的核心资源。正确理解这些度量对于充分利用、管理甚至扩展 Prometheus 堆栈至关重要。

第五章，*运行 Prometheus 服务器*，聚焦于 Prometheus 服务器，提供了常见的使用模式和虚拟机与容器的完整设置过程场景。

第六章，*导出器与集成*，介绍了一些最有用的导出器，并提供了如何使用它们的示例。

第七章，*Prometheus 查询语言–PromQL*，深入讲解了强大而灵活的 Prometheus 查询语言，利用其多维数据模型，支持临时聚合和时间序列的组合。

第八章，*故障排除与验证*，提供了有关如何快速检测和修复问题的实用指南。还介绍了暴露关键信息的有用端点，并探讨了`promtool`，即 Prometheus 的命令行接口和验证工具。

第九章，*定义告警和记录规则*，介绍了记录和告警规则的使用与测试，并提供了相关的示例。

第十章，*发现与创建 Grafana 仪表板*，深入探讨了 Prometheus 栈中的可视化组件，涵盖了内置控制台功能，同时也探索了 Grafana，及如何构建、共享和重用仪表板。

第十一章，*理解与扩展 Alertmanager*，介绍了栈中的告警组件，展示了如何将其与多种告警提供者集成，并讲解了如何正确设置集群以实现高可用性和告警去重。

第十二章，*选择合适的服务发现*，探讨了多种服务发现集成方式，并提供了构建自定义集成所需的要求和知识。

第十三章，*扩展和联邦 Prometheus*，探讨了 Prometheus 栈的扩展问题，并介绍了分片和全局视图等概念，同时提供了背景信息并对其进行了说明。

第十四章，*将长期存储与 Prometheus 集成*，涵盖了 Prometheus 读取和写入端点的概念。接着深入探讨了外部和长期度量存储的考虑因素。最后，通过 Thanos 提供了一个端到端的示例。

# 为了最大程度地利用本书

基本的监控、网络和容器知识是有用的，但不是必需的。

所有测试环境均使用 macOS 和 Linux 进行验证，并强制使用特定的软件版本以提高兼容性，因此这些是最适合确保你不会遇到问题的环境。第三章，*设置测试环境*，提供了与此主题相关的所有技术信息。

下载软件依赖项需要良好的互联网连接，同时需要现代浏览器来确保访问书中呈现的所有 Web 界面。

# 下载示例代码文件

你可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果你从其他地方购买了本书，可以访问[www.packt.com/support](http://www.packt.com/support)，并注册让文件直接发送到你的邮箱。

你可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入书籍名称，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的解压或提取工具：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，链接为[`github.com/PacktPublishing/Hands-On-Infrastructure-Monitoring-with-Prometheus`](https://github.com/PacktPublishing/Hands-On-Infrastructure-Monitoring-with-Prometheus)。如果代码有更新，它将在现有的 GitHub 仓库中进行更新。

我们还在我们的丰富书籍和视频目录中提供了其他代码包，访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色版本。你可以在此下载：[`www.packtpub.com/sites/default/files/downloads/9781789612349_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789612349_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账户。例如：“现在，你可以运行`vagrant status`。”

代码块按以下方式设置：

```
 annotations:
     description: "Node exporter {{ .Labels.instance }} is down."
     link: "https://example.com"
```

当我们希望特别提醒你注意代码块中的某一部分时，相关的行或项目会以粗体显示：

```
 annotations:
     description: "Node exporter {{ .Labels.instance }} is down."
     link: "https://example.com"
```

任何命令行输入或输出都按以下方式编写：

```
vagrant up
```

**粗体**：表示新术语、重要词汇或在屏幕上看到的文字。例如，菜单或对话框中的文字在文本中会这样显示。以下是一个示例：“你可以通过进入顶部栏中的状态 | 规则找到此页面。”

警告或重要说明如下所示。

提示和技巧如下所示。

# 联系我们

我们非常欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中注明书名，并通过`customercare@packtpub.com`联系我们。

**勘误表**：尽管我们已尽力确保内容的准确性，但难免会有错误。如果你在本书中发现错误，我们将非常感激你向我们报告。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择你的书籍，点击勘误表提交链接，输入相关细节。

**盗版**：如果您在互联网上发现我们作品的任何非法复制版本，我们将非常感激您提供相关的地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣撰写或参与书籍的创作，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下您的评价。阅读并使用本书后，为什么不在您购买书籍的站点上留下评价呢？潜在读者可以根据您的公正意见做出购买决策，我们在 Packt 能够了解您对我们产品的看法，我们的作者也可以看到您对他们书籍的反馈。谢谢！

欲了解有关 Packt 的更多信息，请访问[packt.com](http://www.packt.com/)。
