# 设置 VS Code 并使用 GitHub 项目

本书讲述的是在 Microsoft Dynamics 365 Business Central 中编写自动化测试，而不是如何使用 VS Code 和 AL 语言开发扩展。假设你已经知道如何将 VS Code 与 AL 开发语言结合使用，并且了解 Dynamics 365 Business Central 作为平台和应用程序的工作原理。基于此，我们直接进入了 第二章，*可测试性框架* 的编码，没有解释我们使用的开发工具，并且在接下来的章节中继续这样做。

然而，在本附录中，我们会关注一些 VS Code 和 AL 开发，并介绍在 GitHub 仓库中找到的代码示例。

# VS Code 和 AL 开发

如果你是 AL 语言和 VS Code 扩展开发的新手，你可能需要先练习这一点。有许多资源可以参考，但一本内容详尽且不冗长的书是 Stefano Demiliani 和 Duilio Tacconi 编写的 *Dynamics 365 Business Central* *开发快速入门指南*。

这里有一个链接到 Packt 页面，你也可以在这里订购这本书：[`www.packtpub.com/business/dynamics-365-business-central-development-quick-start-guide`](https://www.packtpub.com/business/dynamics-365-business-central-development-quick-start-guide)

# VS Code 项目

在 *Dynamics 365 Business Central* *开发快速入门指南* 中，Stefano Demiliani 和 Duilio Tacconi 以实用的方式解释了如何在 VS Code 中设置一个新的扩展项目，开始使用 AL 编程，并将扩展部署到 Business Central。你在本书的 GitHub 仓库中找到的项目也是以相同的方式创建的：每个章节都有一个单独的文件夹，其中包含 AL 代码对象和 `app.json` 文件。如果你已经在开发扩展，你会注意到缺少了一个重要的资源：`launch.json`。

请注意，我们将在接下来的章节中更详细地讨论 GitHub 仓库的结构。

# launch.json

要能够将章节文件夹用作完整的 VS Code AL 扩展项目文件夹，你需要向其中添加一个 `launch.json` 文件。`launch.json` 文件通常存储在项目文件夹的 `.vscode` 子文件夹中，并包含有关将在其上启动扩展的 Business Central 安装信息。要获取一个包含 `launch.json` 文件的 `.vscode` 文件夹，请按照以下步骤操作：

1.  打开安装了 AL 语言扩展的 VS Code

1.  使用 `AL: GO!` 命令创建一个新的 VS Code AL 项目

1.  在这个项目中，配置 `launch.json` 文件，以便与即将执行测试编码的 Business Central 安装建立关系

1.  将这个 `.vscode` 文件夹复制到你想要处理的章节项目文件夹中

你的 `launch.json` 可能会像这样：

```
{
   "version": "0.2.0",
    "configurations": [
        {
            "type": "al",
            "request": "launch",
            "name": "Your own server",
            "server": "http://localhost",
            "serverInstance": "BC130",
            "authentication": "Windows",
            "port": 7049,
            "startupObjectId": 130401,
            "startupObjectType": "Page",
            "breakOnError": true,
            "launchBrowser": true
        }
    ]
}
```

与默认的 `launch.json` 文件相比，你会注意到我添加/更新了一些有用的关键词，例如 `startupObjectId`，启动页 130401，即 `测试工具` 页面。

你可以在*Dynamics 365 Business Central* *开发快速入门指南*中找到更多详细信息，作者是 Stefano Demiliani 和 Duilio Tacconi。

# app.json

`app.json` 文件，也叫做清单文件，定义了扩展的元描述，并已在 GitHub 上为每个章节文件夹提供。这里是第五章，第六章，第七章和第九章中仅列出相关部分的清单：

```
{
  "id": "e26890f8-fafe-49c6-8951-2c1457921f9b",
  "name": "LookupValue",
  "publisher": "fluxxus.nl",
  "brief": "LookupValue extension for book Automated Testing in
               Microsoft Dynamics 365 Business Central",
  "description": "LookupVale extension as basis to test examples
                     in chapters 5, 6, and 7 for book Automated
                     Testing in Microsoft Dynamics 365 Business
                     Central written by Luc van Vugt and published
                     by Packt",
  "version": "1.0.0.0",
  "platform": "14.0.0.0",
  "application": "14.0.0.0",
  "test": "14.0.0.0",
  "idRange": {
    "from": 50000,
    "to": 81099
  },
  "runtime": "2.4",
  "showMyCode": true
}
```

除了使用 GitHub 提供的 `app.json` 文件外，你还可以使用新创建的 VS Code AL 项目文件夹中的 `app.json` 文件，像之前创建的那样，以获得 `launch.json` 文件。如果你这么做，确保你的扩展具有不同的（且唯一的）名称和 ID，因为在编写本书时，我已经部署了 `LookupValue` 扩展无数次。

在 *如何创建每租户扩展的十大技巧与窍门* 博客的提示 #10 中（[`simplanova.com/top-tips-tricks-per-tenant-extensions/`](https://simplanova.com/top-tips-tricks-per-tenant-extensions/)），Dmitry Katson 解释了如果你的扩展不是唯一的会发生什么，以及如何使其唯一。

# GitHub 仓库

本书中使用的各种代码示例已经上传到一个专门的 GitHub 仓库中。可以通过以下链接访问该仓库：[`github.com/PacktPublishing/Automated-Testing-in-Microsoft-Dynamics-365-Business-Central`](https://github.com/PacktPublishing/Automated-Testing-in-Microsoft-Dynamics-365-Business-Central)。

# GitHub 仓库结构

在该仓库的主页上，你会找到以下文件夹：

+   `ATDD 场景`

+   第二章

+   `第五章 (LookupValue 扩展)`

+   `第六章 (LookupValue 扩展)`

+   `第七章 (LookupValue 扩展) - 重构并完成`

+   `第七章 (LookupValue 扩展)`

+   `第九章 (LookupValue 扩展)`

+   `LookupValue 扩展 (仅应用)`

+   `LookupValue 测试扩展 (仅测试)`

在本节中，我们将讨论各个文件夹的内容。请注意，GitHub 会按字母顺序排列它们。这与文件夹在书中使用的顺序无关。不过，我会详细阐述书中的顺序。

像其他任何 GitHub 仓库一样，我们的仓库也包含 `LICENSE` 和 `README.md` 文件。

# 第二章

该文件夹包含第二章的代码示例，*可测试性框架*，包括 `MyTestsExecutor` 页面和 `app.json` 文件，后者允许你将代码作为扩展部署。

# ATDD 场景

该文件夹包含以下两个 Excel 文件：

+   `清单.xlsx`

+   `LookupValue.xlsx`

`LookupValue`文件包含所有**验收测试驱动开发**（**ATDD**）场景的列表，按照`GIVEN`-`WHEN`-`THEN`格式详细描述了完整的客户需求，如第五章中介绍的*从客户需求到测试自动化——基础篇*，用于我们的`LookupValue`扩展。在第五章中已进一步详细说明，并延续至第七章。

`Clean sheet.xlsx`文件是一个现成的干净版本，用于编写你自己的 ATDD 场景。

# LookupValue 扩展（仅应用）

本文件夹包含`LookupValue`扩展的应用代码，包含我们在第五章、第六章、第七章和第九章中编写测试代码的应用代码，包含`app.json`文件，允许你将代码作为扩展进行部署。请注意，它已包含一个`Test Codeunits`文件夹，里面有第一个代码单元结构，构建于第五章。

# 第五章（LookupValue 扩展）

从`LookupValue 扩展（仅应用）`文件夹中的代码开始，本文件夹将第五章中的完整代码示例，*从客户需求到测试自动化——基础篇*，添加到扩展中。它还包含`app.json`文件，允许你将代码作为扩展进行部署。

# 第六章（LookupValue 扩展）

从`第五章（LookupValue 扩展）`文件夹中的代码开始，本文件夹添加了第六章的完整代码示例，*从客户需求到测试自动化——进阶篇*。它还包含`app.json`文件，允许你将代码作为扩展进行部署。

# 第七章（LookupValue 扩展）

从`第六章（LookupValue 扩展）`文件夹中的代码开始，本文件夹添加了第七章的完整代码示例，*从客户需求到测试自动化——以及更多内容*。它还包含`app.json`文件，允许你将代码作为扩展进行部署。

# 第七章（LookupValue 扩展）- 重构并完成

本文件夹包含整个`LookupValue`扩展的重构和完成后的应用与测试代码，包括允许你将代码作为扩展部署的`app.json`文件。

# 第九章（LookupValue 扩展）

从`第七章（LookupValue 扩展）- 重构并完成`文件夹中的代码开始，本文件夹添加了第九章的完整代码示例，*让 Business Central 标准测试在你的代码上工作*。它还包含`app.json`文件，允许你将代码作为扩展进行部署。

# LookupValue 测试扩展（仅测试）

如书中所述，在发布扩展时，最佳实践是将应用程序代码与测试代码分开。对于 `AppSource`，这甚至是一个要求。这意味着最终的测试扩展是依赖于真实扩展构建的。此文件夹包含一个单独的测试扩展，属于 `LookupValue Extension (仅应用)` 文件夹中的扩展。

# 关于 AL 代码的说明

现在，关于 GitHub 上的代码和书中的代码示例，最后再做几点说明。

# VS Code 与 C/SIDE

本书完全聚焦于以扩展的方式开发 Microsoft Dynamics 365 Business Central 功能的现代方法。因此，GitHub 上的代码和书中的代码示例都是用 AL 语言编写的。但所有呈现和使用的原则同样适用于在 C/SIDE 中开发自动化测试，C/SIDE 是 Microsoft Dynamics NAV 的开发环境。目前，Business Central 的标准应用和测试代码仍由 Microsoft 提供，且是以 C/SIDE 对象形式存在。

# 前缀或后缀

在构建你自己的扩展时，最佳实践是使用所谓的*前缀*或*后缀*来命名你的对象、字段和控件。我们选择不使用前缀/后缀，原因如下：

+   `LookupValue` 扩展不是一个注册过的扩展

+   使用前缀/后缀不会提高代码示例的可读性

关于使用*前缀*或*后缀*的更多信息，请参见：[`docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/compliance/apptest-prefix-suffix`](http://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/compliance/apptest-prefix-suffix)

# 自动换行

在书中添加代码示例时，始终面临如何整齐地格式化长行代码以保持代码可读性的问题。在本书中的代码示例中，采用了强制*自动换行*的方法，即使代码可能无法再编译。然而，GitHub 仓库中的所有代码在技术上是没有问题的。
