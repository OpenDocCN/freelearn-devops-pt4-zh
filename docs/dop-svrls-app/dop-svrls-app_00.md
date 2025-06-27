# 前言

无服务器开发为开发者提供了专注于开发本身的自由，无需担心服务器端的事务。本书旨在通过向您展示如何有效地将 DevOps 原则应用于无服务器应用程序，简化您的无服务器部署体验。在每一步过程中，您将接触到有效构建完整持续集成和持续交付管道及日志管理的最佳实践。

# 本书适用对象

*无服务器应用程序的 DevOps* 适用于 DevOps 工程师、架构师或任何想了解无服务器世界中 DevOps 思想的人。您将学习如何在无服务器开发中使用 DevOps，并应用持续集成、持续交付、测试、日志管理和监控。

# 本书内容

第一章，*介绍无服务器*，用简单的语言描述了什么是无服务器，涵盖了其优缺点。它讨论了不同的无服务器服务提供商，并介绍了他们提供的无服务器服务。

第二章，*理解无服务器框架*，讨论了不同的无服务器部署框架，并深入探讨了在本书大多数章节中使用的 Serverless Framework。

第三章，*将 DevOps 应用于 AWS Lambda 应用程序*，深入探讨了 AWS Lambda 与 DevOps 相关的内容。它通过多个实践教程，讲解了如何使用 Serverless Framework 和 Jenkins 构建持续集成与持续部署管道，以及如何进行监控和日志管理。还涵盖了如何为 AWS Lambda 设置金丝雀发布和蓝绿发布。

第四章，*与 Azure Functions 一起使用 DevOps*，讲解了 Azure Functions。首先介绍如何创建和部署 Azure Functions，然后讲解如何通过 Jenkins 设置持续集成与持续部署管道。还涵盖了监控和日志管理，并讨论了与 Azure Functions 一起使用 DevOps 的最佳实践。

第五章，*将 DevOps 与 IBM OpenWhisk 集成*，介绍了 OpenWhisk，并讲解了如何使用 Serverless Framework 和 Jenkins 设置部署管道。还涵盖了如何在 IBM Cloud 上监控和动态展示 OpenWhisk。

第六章，*与 Google Functions 一起使用 DevOps*，首先介绍了 Google Functions。然后通过多个教程，从创建函数到设置自动化部署管道，逐步讲解。它还讲述了如何使用 Google Stackdriver 进行监控和日志管理。

第七章，*为 Kubeless 添加 DevOps 元素*，解释了 Kubeless 是一个基于 Kubernetes 的开源无服务器架构。本章介绍了 Kubeless，并解释了如何为 Kubeless 设置持续集成和持续部署。

第八章，*无服务器与 DevOps 的最佳实践与未来*，讨论了我们在开发过程中如何处理应用程序的性能问题，这引出了最佳实践。因此，本章概述了无服务器和 DevOps 的最佳实践，以便我们能够轻松地进行开发和部署。

第九章，*用例与基础知识*，介绍了一些常见的无服务器用例，并且讲解了能够提高开发和部署效率的一些必要知识点。

第十章，*无服务器的 DevOps 趋势*，介绍了无服务器如何塑造 DevOps，以及 DevOps 在采用无服务器后需要如何调整其轨迹。

# 要充分利用本书

本书的主要内容是阐明无服务器及其不同的服务提供商。它还讲解了如何采用自动化方式为无服务器函数设置 DevOps。

你需要了解 DevOps、持续集成和持续部署，并且应该具备一些流行的 DevOps 工具的知识，如 Jenkins、ELK（Elasticsearch、Logstash 和 Kibana）。如果你了解不同的云服务提供商，如 AWS、Azure 和 Google，那将是一个加分项。

我的教程大多数基于 MacBook 和 Docker 容器构建，因此我建议使用某种形式的 Linux 来进行这些教程。

# 下载示例代码文件

你可以从你的帐户在 [www.packt.com](http://www.packt.com) 下载本书的示例代码文件。如果你在其他地方购买了本书，你可以访问 [`www.packt.com/support`](http://www.packt.com/support)，并注册以直接通过电子邮件获取文件。

你可以按照以下步骤下载代码文件：

1.  请在 [www.packt.com](http://www.packt.com) 上登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址是 [`github.com/PacktPublishing/DevOps-for-Serverless-Applications`](https://github.com/PacktPublishing/DevOps-for-Serverless-Applications)。如果代码有更新，将会在现有的 GitHub 仓库中更新。

我们还从我们丰富的书籍和视频目录中提供其他代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看！快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在[这里下载：https://www.packtpub.com/sites/default/files/downloads/9781788623445_ColorImages.pdf](https://www.packtpub.com/sites/default/files/downloads/9781788623445_ColorImages.pdf)。

# 使用的约定

本书使用了许多文本约定。

`CodeInText`：表示文本中的代码字词，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL，用户输入和 Twitter 句柄。这是一个示例："在语句中，我们将参数传递给 CLI，并且`stage`值填充在`serverless.yml`文件中。"

一个代码块设置如下：

```
# serverless.yml
service: myService
provider:
  name: aws
  runtime: nodejs6.10
  memorySize: 512 # will be inherited by all functions
```

任何命令行输入或输出都按以下格式编写：

```
$ pip install zappa
```

**粗体**：表示一个新术语，重要词或屏幕上看到的字词。例如，菜单或对话框中的字词以这种方式出现在文本中。这是一个例子："在左侧边栏点击`Users`，然后点击`Add User`按钮，并添加用户名`adm-serverless`。"

警告或重要提示会出现在这样的地方。

提示和技巧会出现在这样的地方。

# 联系我们

我们的读者的反馈总是受欢迎的。

**一般反馈**：电邮至`customercare@packtpub.com`并在您的消息主题中提到书名。如果您对本书的任何方面有疑问，请发送电子邮件至`customercare@packtpub.com`与我们联系。

**勘误**：尽管我们尽最大努力确保内容的准确性，但错误确实偶尔发生。如果您在本书中发现错误，请向我们报告。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何形式的非法副本，请提供地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为什么不在购买它的网站上留下评论呢？潜在的读者可以看到并使用您的公正意见来做出购买决策，我们在 Packt 能理解您对我们产品的看法，而我们的作者也能看到您对他们书籍的反馈。谢谢！

欲了解更多关于 Packt 的信息，请访问[packt.com](http://packt.com)。
