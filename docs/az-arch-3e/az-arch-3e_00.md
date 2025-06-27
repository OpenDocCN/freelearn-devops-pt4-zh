# 序言

## 关于本书

本节简要介绍了作者、书籍的覆盖内容、开始时所需的技术技能以及使用 Azure 架构解决方案所需的硬件和软件要求。

## 关于《Azure for Architects》第三版

由于其对高可用性、可扩展性、安全性、性能和灾难恢复的支持，Azure 已广泛应用于轻松创建和部署各种类型的应用程序。更新了最新发展的《*Azure for Architects*》第三版帮助你掌握设计无服务器架构的核心概念，包括容器、Kubernetes 部署和大数据解决方案。你将学习如何架构诸如无服务器功能等解决方案，发现容器和 Kubernetes 的部署模式，并探索使用 Spark 和 Databricks 进行大规模大数据处理。随着深入，你将使用 Azure DevOps 实现 DevOps，利用 Azure Cognitive Services 工作智能解决方案，并将安全性、高可用性和可扩展性集成到每个解决方案中。最后，你将深入了解 Azure 安全概念，如 OAuth、OpenConnect 和托管身份。

到本书结束时，你将有信心基于容器和无服务器功能设计智能的 Azure 解决方案。

### 作者简介

**Ritesh Modi** 是微软前高级技术布道者。他因对微软产品、服务和社区的贡献被认定为微软区域总监。他是一位云架构师、出版作者、演讲者和领导者，因其在数据中心、Azure、Kubernetes、区块链、认知服务、DevOps、人工智能和自动化方面的贡献而广受欢迎。他是八本书的作者。

Ritesh 曾在众多国内外会议上发表演讲，并且是 MSDN 杂志的出版作者。他在为客户构建和部署企业解决方案方面有超过十年的经验，并且拥有超过 25 项技术认证。他的爱好包括写书、和女儿玩耍、看电影以及学习新技术。他目前居住在印度海得拉巴。你可以在 Twitter 上关注他，账号是 **@automationnext**。

**Jack Lee** 是一位高级 Azure 认证顾问和 Azure 实践负责人，对软件开发、云计算和 DevOps 创新充满热情。Jack 因其对技术社区的贡献而被评为微软 MVP。他曾在多个用户组和会议上发表演讲，包括微软加拿大的全球 Azure 启动营。Jack 是一位经验丰富的导师和黑客松评审员，同时也是一个专注于 Azure、DevOps 和软件开发的用户组的主席。他是《*Cloud Analytics with Microsoft Azure*》一书的合著者，出版方为 Packt Publishing。你可以在 Twitter 上关注 Jack，账号是 **@jlee_consulting**。

**Rithin Skaria** 是一名开源倡导者，拥有超过 7 年的在 Azure、AWS 和 OpenStack 上管理开源工作负载的经验。他目前在微软工作，并参与了微软内多项开源社区活动。他是微软认证培训师、Linux 基金会认证工程师和管理员、Kubernetes 应用开发者和管理员，以及认证 OpenStack 管理员。在 Azure 方面，他拥有四项认证（解决方案架构、Azure 管理、DevOps 和安全），同时他也获得了 Office 365 管理认证。他在多个开源部署中扮演了重要角色，并负责这些工作负载向云平台的管理和迁移。他还是*Linux Administration on Azure*一书的合著者，书籍由 Packt 出版。通过 LinkedIn 可以联系他，账号为**@rithin-skaria**。

### 审稿人简介

**Melony Qin** 是一位从事 STEM 领域的女性。目前，她在微软担任项目经理，并且是**计算机协会**（**ACM**）和**项目管理协会**（**PMI**）的成员。她曾在微软 Azure 平台上贡献于无服务器计算、大数据处理、DevOps、人工智能、机器学习和物联网（IoT）等领域。她拥有所有 Azure 认证（包括应用与基础设施轨道和数据与 AI 轨道），以及**认证 Kubernetes 管理员**（**CKA**）和**认证 Kubernetes 应用开发者**（**CKAD**）证书，目前主要专注于在社区中为**开源软件**（**OSS**）、DevOps、Kubernetes、无服务器、大数据分析和物联网等领域做出贡献。她是两本书的作者和合著者，分别是*Microsoft Azure Infrastructure*和*The Kubernetes Workshop*，这两本书均由 Packt 出版。她可以通过 Twitter 与人联系，账号为**@MelonyQ**。

**Sanjeev Kumar** 是微软 Azure 上 SAP 云解决方案架构师。他目前居住在瑞士苏黎世，拥有超过 19 年的 SAP 技术经验。在过去的 8 年里，他一直在公共云技术领域工作，其中最近 2 年专注于微软 Azure。

在担任 SAP on Azure 云架构顾问的角色时，Sanjeev Kumar 与许多世界顶级的金融服务和制造公司合作。他的关注领域包括云架构和设计，帮助客户将 SAP 系统迁移到 Azure，并采用 Azure 最佳实践进行 SAP 部署，特别是在实施基础设施即代码（Infrastructure as Code）和 DevOps 方面。他还在使用 Docker 和 Azure Kubernetes Service 进行容器化和微服务、使用 Apache Kafka 进行流数据处理以及使用 Node.js 进行全栈应用开发等方面有丰富经验。他参与了涉及 IaaS、PaaS 和 SaaS 的各种产品开发工作。他还对人工智能、机器学习以及大规模数据处理和分析等新兴领域感兴趣。他在 LinkedIn 上撰写有关 SAP on Azure、DevOps 和基础设施即代码的文章，你可以在**@sanjeevkumarprofile**找到他。

### 学习目标

在本书结束时，你将能够：

+   了解 Azure 云平台的组件

+   使用云设计模式

+   使用企业安全指南进行 Azure 部署

+   设计并实现无服务器和集成解决方案

+   在 Azure 上构建高效的数据解决方案

+   了解 Azure 上的容器服务

### 目标读者

如果你是云架构师、DevOps 工程师或开发人员，想要了解 Azure 云平台的关键架构方面，本书适合你。对 Azure 云平台的基本理解将帮助你更有效地掌握本书中的概念。

### 方法

本书通过逐步讲解关键概念、实际示例和自我评估问题，涵盖了每个主题。通过理论和实际操作经验的平衡，本书将帮助你理解架构师在现实世界中的工作方式。

### 硬件要求

为了获得最佳体验，我们建议以下配置：

+   最少 4GB RAM

+   最少 32GB 的空闲内存

### 软件要求

+   Visual Studio 2019

+   Docker for Windows 最新版本

+   AZ PowerShell 模块 1.7 及以上版本

+   Azure CLI 最新版本

+   Azure 订阅

+   Windows Server 2016/2019

+   Windows 10 最新版本 - 64 位

### 约定

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL 和用户输入如下所示：

DSC 配置仍未被 Azure Automation 识别。它在某些本地机器上可用，应该上传到 Azure Automation DSC 配置中。Azure Automation 提供了`Import-AzureRMAutomationDscConfiguration` cmdlet 用于将配置导入到 Azure Automation：

```
Import-AzureRmAutomationDscConfiguration -SourcePath "C:\DSC\AA\DSCfiles\ConfigureSiteOnIIS.ps1" -ResourceGroupName "omsauto" -AutomationAccountName "datacenterautomation" -Published -Verbose
```

### 下载资源

本书的代码包也托管在 GitHub 上，地址为：[`github.com/PacktPublishing/Azure-for-Architects-Third-Edition`](https://github.com/PacktPublishing/Azure-for-Architects-Third-Edition)。

我们还有其他代码包，来自我们丰富的书籍和视频目录，您可以在[`github.com/PacktPublishing`](https://github.com/PacktPublishing)找到。快去看看吧！
