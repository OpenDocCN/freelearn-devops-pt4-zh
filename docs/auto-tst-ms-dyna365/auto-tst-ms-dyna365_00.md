# 前言

随着最近的变化，尤其是 Microsoft Dynamics NAV 发展成 Dynamics 365 Business Central，你的开发实践需要变得更加规范。你的成功将比以往任何时候都更加依赖于你快速而一致地判断自己工作质量的能力。传统的手动测试方法已不再足够。因此，你需要学习自动化测试，以及如何高效地将其纳入日常工作中。在本书中，你将学习它如何从功能和技术上提升你的工作，并且希望你能够喜欢它。

# 本书的适用人群

本书面向开发人员、开发经理、测试人员、功能顾问以及使用 Microsoft Dynamics 365 Business Central 或 Dynamics NAV 的高级用户。

假设读者对 Dynamics 365 Business Central 作为一款应用程序及其如何扩展已有基本了解。尽管某些章节将专门讲解自动化测试的编写，但并不要求读者必须具备 Dynamics 365 Business Central 的编程能力。总体来说，能够从测试人员的角度进行思考无疑会给读者带来优势。

# 本书内容

第一章，*自动化测试简介*，将带你了解自动化测试：你为何需要使用它，它具体包含哪些内容，以及何时使用它。

第二章，*可测试性框架*，详细讲解了 Microsoft Dynamics 365 Business Central 如何支持自动化测试的执行，以及所谓的 *可测试性框架* 实际上是什么，通过描述其五大支柱进行阐述。

第三章，*测试工具与标准测试*，介绍了 Dynamics 365 Business Central 中的测试工具，帮助你运行测试。同时，我们将讨论 Microsoft 提供的标准测试和测试库。

第四章，*测试设计*，讨论了几个概念和设计模式，使你能够更有效率地设计测试。

第五章，*从客户需求到测试自动化 - 基础知识*，通过一个业务案例，教你如何从客户需求出发，实现自动化测试的实施，并允许你进行实践。在本章中，你将利用前几章中讨论的标准测试库和技术。本章中的示例将教你头 less 测试和 UI 测试的基础知识，以及如何处理正面和负面测试。

第六章，*从客户需求到测试自动化 - 下一阶段*，继续第五章，*从客户需求到测试自动化 - 基础知识*中的业务案例，并介绍了一些更高级的技巧：如何利用共享构造，如何对测试进行参数化，如何处理 UI 元素并将变量传递给这些所谓的 UI 处理器。

第七章，*从客户需求到测试自动化 - 以及更多内容*，包括两个额外示例，并继续使用前两章中的相同业务案例：如何进行报告测试以及如何为更复杂的场景设置测试。

第八章，*如何将测试自动化集成到日常开发实践中*，讨论了一些最佳实践，这些实践可能对您和您的团队在日常工作中启动和运行测试自动化有所帮助。

第九章，*使 Business Central 标准测试在您的代码中正常工作*，讨论了为什么要使用微软为 Dynamics 365 Business Central 提供的标准测试资料，以及当标准测试因您扩展标准应用程序而失败时如何修复错误。

附录 A，*测试驱动开发*，简要描述了**测试驱动开发**（**TDD**）是什么，并指出对您的日常开发实践也可能有价值的部分。

附录 B，*设置 VS Code 并使用 GitHub 项目*，关注 VS Code 和 AL 开发，以及在 GitHub 上的代码库中可以找到的代码示例。

# 为了从本书中获得最大收益

本书是《Dynamics 365 Business Central 测试自动化入门》。一方面，讨论了各种概念和术语，另一方面，我们还将通过编写测试代码进行实践。为了从本书中获得最大收益，您可能需要通过实现讨论过的代码示例来*实践所倡导的内容*。然而，由于本书未涉及如何针对 Business Central 编程，您可能首先需要阅读附录 B*，**设置 VS Code 并使用 GitHub 项目**中的提示。

如果你的学习方式是先了解原理、术语和概念，可以开始阅读第一章，《自动化测试简介》，然后逐步进入更实用的*第三部分：**为 Microsoft Dynamics 365 Business Central 设计和构建自动化测试*。如果你的学习方式更倾向于通过实践来学习，你可以大胆地直接深入阅读第五章，《从客户需求到测试自动化——基础》，第六章，《从客户需求到测试自动化——进阶》，以及第七章，《从客户需求到测试自动化——更多内容》，并稍后或在学习过程中再阅读不同的背景知识。

# 下载示例代码文件

你可以从你的账户在[www.packt.com](http://www.packt.com)下载本书的示例代码文件。如果你是在其他地方购买的本书，可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接发送到你的邮箱。

你可以通过以下步骤下载代码文件：

1.  登录或注册[www.packt.com](http://www.packt.com)。

1.  选择“SUPPORT”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Automated-Testing-in-Microsoft-Dynamics-365-Business-Central`](https://github.com/PacktPublishing/Automated-Testing-in-Microsoft-Dynamics-365-Business-Central)。如果代码有更新，将会在现有的 GitHub 仓库中进行更新。

# 下载彩色图像

我们还提供了一份 PDF 文件，包含了本书中使用的截图/图表的彩色版本。你可以在此下载：[`www.packtpub.com/sites/default/files/downloads/9781789804935_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781789804935_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账户。示例如下：“唯一需要采取的步骤是将以下代码添加到相关的`Initialize`函数中。”

代码块的表示方式如下：

```
fields
    {
        field(1; Code; Code[10]){}
        field(2; Description; Text[50]){}
    }
```

当我们希望特别指出代码块中的某部分时，相关的行或项目将以粗体显示：

```
[FEATURE] LookupValue Customer 
[SCENARIO #0001] Assign lookup value to customer
[GIVEN] A lookup value
[GIVEN] A customer
```

**粗体**：表示新术语、重要词汇或屏幕上出现的文字。例如，菜单或对话框中的文字会像这样出现在文本中。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要说明像这样呈现。

提示和技巧像这样呈现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中提到书名，并通过`customercare@packtpub.com`联系我们。

**勘误表**：尽管我们已尽力确保内容的准确性，但难免会出现错误。如果你在本书中发现了错误，我们将非常感激你向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择你的书籍，点击“勘误提交表单”链接，并输入相关信息。

**盗版**：如果你在互联网上发现我们作品的任何非法版本，我们将非常感激你能提供具体的地址或网站名称。请通过`copyright@packt.com`联系我们，并附上相关资料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有兴趣撰写或参与编写一本书，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评论。阅读并使用本书后，为什么不在你购买本书的站点上留下评价呢？潜在的读者可以通过你的公正评价做出购买决策，我们 Packt 也能了解你对我们产品的看法，我们的作者也能看到你对他们书籍的反馈。谢谢！

若需了解更多有关 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
