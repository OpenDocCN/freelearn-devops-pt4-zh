# 序言

随着 IT 产品创建和交付的速度在企业成功中的重要性不断提升，具备全面理解 DevOps 的 IT 专业人才需求逐年增加。在本书《实施 Azure DevOps 解决方案》中，你将学习到帮助实现这一目标的微软工具。此外，你所获得的知识将帮助你拥抱 DevOps，并改进持续交付价值给最终用户的方式。

# 本书适合的人群

本书面向有兴趣在 Azure 云上实施 DevOps 实践的软件开发人员和运维专家。应用程序开发人员和具备一定软件开发经验的 IT 专业人士也会发现本书非常有用。对 Azure DevOps 基础使用的熟悉程度将是一个额外的加分项。

此外，最棒的是，你还可以将本书作为 AZ-400 考试的参考资料之一，因为所涉及的主题与考试内容非常相似。

# 本书的内容

本书分为四个部分。各章节按推荐的阅读顺序排列，每一章都是在前一章的基础上进行扩展。章节的顺序是与在现实世界中实施不同方面的顺序相匹配的。然而，完成第一部分后，你也可以根据需要选择性阅读其他部分。

第一章，*DevOps 简介*，为你提供了对 DevOps 的理解，包括它是什么以及它不是什么。定义了 DevOps，并描述了它与敏捷工作方式的关系。本章最后介绍了 DevOps 实践和习惯，这些是 DevOps 文化的核心。

第二章，*一切从源代码管理开始*，介绍了不同类型的源代码管理系统及其比较。拉取请求和代码审查作为确保每个更改在成为源代码一部分之前都会经过审查的默认方式进行了介绍。本章最后回顾了与持续流动价值的工作方式高度契合的分支和合并策略。

第三章，*迁移到持续集成*，介绍了如何从源代码到构建工件，以便稍后部署。Azure Pipelines 作为实现这一目标的最重要方式进行了深入讨论。除了传统的可视化编辑器外，还介绍了 YAML 管道作为代码。

第四章，*持续部署*，讲述了如何使用 Azure Pipelines 来协调部署。再次介绍了经典的可视化发布编辑器。还介绍了多阶段的 YAML 管道，它作为一种方式，可以将部署过程描述为代码。本章最后介绍了不同的部署策略，用于减轻持续部署带来的风险，以平衡变更需求与稳定性。

第五章，*依赖管理*，介绍了包管理。包管理可以作为一种手段，将大型软件解决方案拆分为多个独立构建和测试的组件，然后再将其组合在一起。包还可以用来在不同的持续集成和部署产品之间分发构建工件。

第六章，*基础设施和配置即代码*，讲述了如何将创建、管理和配置基础设施的过程，从一个容易出错的手动任务转变为自动化的配置部署，这些配置被作为代码保存在源代码管理中。本章涵盖了 ARM 模板、Azure Blueprints、Azure App Configuration 服务以及一些相关工具。

第七章，*数据库管理*，深入探讨了如何在持续部署的背景下管理数据库架构。

第八章，*持续测试*，解释了如何将一切作为代码，持续部署新版本，如果新版本的质量不够高，则几乎没有价值。本章介绍了不同类型的测试，以及如何将它们集成到 Azure DevOps 管道中。

第九章，*安全与合规*，讲述了如何将安全性和合规性问题集成到 DevOps 实践中。你将学习如何将安全扫描和依赖扫描集成到管道中。引入了 Azure Policy 和 Security Center，用于防止不合规配置，并检测随着时间推移出现的新风险。

第十章，*应用监控*，是第一章讲述如何从先前部署的更改中学习。为此，使用 Azure Monitor 和 Application Insights 创建指标、仪表板和警报。

第十一章，*收集用户反馈*，同样涉及学习，但学习的对象是用户，而非系统。你将学习如何与用户互动，以推动你的产品路线图并最大化为用户创造的价值。假设驱动的开发方法被引入，作为一种减少低回报投资并发现需求高、价值大的功能的方式。

第十二章，*容器*，介绍了容器的相关话题。虽然 DevOps 和容器不是同义词，但容器有助于在那些可能无法实现 DevOps 原则的情况下，推动 DevOps 原则的采纳。

第十三章，*规划您的 Azure DevOps 组织*，深入讨论了有关设置 Azure DevOps 组织和项目的最终考虑事项。包括了许可和成本模型的内容，以及可追溯性的话题。另一个重要的主题是产品间迁移，以实现 DevOps 工具的标准化。

第十四章，AZ-*400 模拟考试*，为你提供了通过模拟考试测试你在本书中所学知识的机会。

# 为了充分利用本书的内容

你应该理解软件开发流程，并且具有作为软件开发人员或 IT 专业人员的应用程序开发和运维经验。无论你的背景是软件开发还是运维，了解软件交付流程及其相关工具的基本知识都非常重要。

如果你已经参加过模拟考试，或者在读完本书后打算参加正式考试，并发现自己在某些考试目标上有困难，可以使用下面的表格找到需要重新阅读的章节。

| **考试目标** | **相关章节** |
| --- | --- |
| 设计 DevOps 战略 | 第 1、8 和 13 章 |
| 实现 DevOps 开发流程 | 第 4、6 和 9 章 |
| 实现持续集成 | 第 2 和第三章 |
| 实现持续交付 | 第四章 |
| 实现依赖管理 | 第五章 |
| 实现应用程序基础设施 | 第 6、9 和 12 章 |
| 实现持续反馈 | 第 10 和第十一章 |

请记住，某些问题可能会涉及多个类别，而且本书的开发没有使用任何官方考试材料。

虽然本书的很多部分是理论内容，但如果你对这些概念没有实际操作经验，建议进行实践。记住，如果你打算参加 AZ-400 考试，这个考试是为有两到三年实际经验的从业人员设计的。为了完成书中的实践练习，你需要具备以下条件：

| 本书涵盖的**软件/硬件** | **操作系统要求** |
| --- | --- |
| Azure DevOps 服务 | 任何带有现代浏览器的设备 |
| Azure 门户 | 任何带有现代浏览器的设备 |
| Azure Powershell | Windows 10 |
| Azure CLI, Git 客户端 | Windows、Linux 或 MacOS |
| Visual Studio | Windows 或 MacOS |

为了获得更多的实际操作经验，每章末尾都会提供与练习或实验室相关的链接。这些练习大多数来自 Microsoft Learn，也可以在[`docs.microsoft.com/en-us/learn/browse/?term=devops`](https://docs.microsoft.com/en-us/learn/browse/?term=devops)查找。微软还发布了一个云端工作坊，让你可以实践本书第 1 至第六章中涉及的许多主题。该云工作坊可以在[`github.com/microsoft/MCW-Continuous-delivery-in-Azure-DevOps/blob/master/Hands-on%20lab/Before%20the%20HOL.md`](https://github.com/microsoft/MCW-Continuous-delivery-in-Azure-DevOps/blob/master/Hands-on%20lab/Before%20the%20HOL.md)找到。

# 下载彩色图像

我们还提供了一份包含本书中截图/图表的彩色图像的 PDF 文件。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789619690_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781789619690_ColorImages.pdf)

# 本书使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。这里有一个示例：“在命令提示符下，输入`hostname`并按下*Enter*键。”

代码块如下所示：

```
public class FoodClassifier : IFoodClassifier
{
                public FoodClassification Classify(Food food)
                {
                                // Unchanged classification algorithm
                } 
}
public class FoodClassifierToBeRemoved : IFoodClassifer
{
    public FoodClassification Classify(Food food)
                {
                                // Unchanged classification algorithm
                } 
}
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都如下所示：

```
git lfs install
git lfs track "*.mp4"
git add .gitattributes
```

**粗体**：表示一个新术语、一个重要词汇，或者是你在屏幕上看到的词语。例如，菜单或对话框中的词汇在文本中会这样显示。这里有一个例子：“从管理面板中选择‘系统信息’。”

警告或重要提示如下所示。

提示和技巧会这样显示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中注明书名，并通过`customercare@packtpub.com`联系我们。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但难免会出现错误。如果你在本书中发现了错误，我们将非常感激你能将其报告给我们。请访问[www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择你的书籍，点击“勘误提交表单”链接，并填写详细信息。

**盗版**：如果你在互联网上发现我们作品的任何非法复制品，我们将非常感激你能提供该材料的地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上该材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有兴趣写作或为一本书做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。阅读并使用本书后，为什么不在你购买它的网站上留下评论呢？潜在读者可以查看并根据你的公正意见做出购买决策，我们在 Packt 可以了解你对我们产品的看法，而我们的作者也能看到你对他们书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packt.com](http://www.packt.com/)。
