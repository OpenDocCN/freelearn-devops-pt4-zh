# 序言

Vagrant 是一个开源工具，允许你通过编程创建和管理虚拟环境。Vagrant 的主要目标是创建可以在全球各地的团队之间共享的开发环境。它解决了“在我机器上运行正常”的问题，任何拥有 Vagrantfile 配置的人都可以创建一个与原始机器完全相同的副本。

Vagrant 由 Mitchell Hashimoto 和 HashiCorp 创建并维护，持续获得支持和更新。自 2010 年创建以来，它已经逐步壮大，成为一款不断进步的软件。

# 这本书适合谁

在本书中，我们将涵盖 Vagrant 的多个方面。本书适合那些对 Vagrant 几乎没有或完全没有经验的初学者。我们将介绍如何安装 Vagrant 以及启动所需的所有基础知识。

本书也适用于那些希望更好地理解和使用 Vagrant 的高级用户。我们将涵盖可用的命令、网络配置、多机器以及如何使用配置管理工具如 Chef 和 Ansible 进行资源配置。

无论你处于什么水平，本书都会教你一些新东西，或者帮助你巩固知识并提供一些技巧和窍门。

# 本书所涉及的内容

第一章，*介绍*，是通向 Vagrant 世界的一个极好的入门。它将帮助你打下知识基础，引导你继续阅读本书。你将了解 Vagrant 是什么，Vagrant 的好处，什么是 VirtualBox，以及什么是 DevOps。你还将学习 Vagrant 如何融入 DevOps 生态，如何作为一个 DevOps 工具使用，并且了解其他一些软件。

第二章，*安装 VirtualBox 和 Vagrant*，Windows、macOS 和 Linux，带你动手操作，教授你如何安装 VirtualBox 和 Vagrant。我们将覆盖三大主流操作系统：Windows、macOS 和 Linux。你将学习如何浏览两个网站（[`www.virtualbox.org`](https://www.virtualbox.org) 和 [`www.vagrantup.com`](https://www.vagrantup.com)），下载、安装并验证软件的安装情况。

第三章，*命令行界面 - Vagrant 命令*，教你 Vagrant 提供的各种有用命令。你将学习所有可用的命令和子命令。你还将了解 Vagrant 命令的结构，如何使用`help`命令获取更多信息，以及每个命令的简要描述。通过本章的学习，你将能够自信地通过命令行管理 Vagrant。

第四章，*发现 Vagrant 盒子 - Vagrant Cloud*，涵盖了 Vagrant 盒子的所有方面。我们将学习如何管理它们：安装、删除和版本管理。我们还将创建一个具有构建 Vagrant 环境最小要求的基础盒子。在这一章中，我们还将介绍 Vagrant Cloud 及其为你提供的内容。Vagrant Cloud 是一个可以搜索的 Vagrant 盒子索引，所有盒子都可以直接下载。我们将讨论如何使用 Vagrant Cloud 网站、如何搜索特定盒子以及如何安装该盒子。

第五章，*使用 Vagrantfile 配置 Vagrant*，探讨了 Vagrantfile，它允许你轻松自定义你的 Vagrant 虚拟机。Vagrantfile 提供了多种不同的配置选项，例如网络、文件夹同步、多机器选项、配置管理和特定提供者设置。你还将学习 Vagrantfile 的语法和格式，并了解如何在创建后验证它。

第六章，*Vagrant 中的网络配置*，解释了 Vagrant 中如何轻松配置网络，并可以用来创建一些强大的设置。在本章中，你将学习三种关键的网络配置选项：端口转发、公共网络和私有网络。你将通过示例学习如何使用每一个选项，并查看每个选项的优点。

第七章，*多机器*，介绍了多机器选项，该选项允许你创建多个 Vagrant 虚拟机，并使用单个 Vagrantfile 对它们进行管理和配置。你将创建一个模拟现实世界场景的多机器环境。你将创建一台运行 Web 服务器的机器和一台运行数据库的机器。这些机器将通过网络配置进行通信。这将为你打下坚实的基础，帮助你开始使用多机器选项创建强大的环境。

第八章，*探索 Vagrant 插件和文件同步*，讲解了尽管 Vagrant 提供了许多功能，但有时你可能需要一些额外的功能。在本章中，你将了解 Vagrant 插件的所有内容。你将看到如何轻松安装和使用 Vagrant 插件。这里还有许多命令和子命令需要学习。在这一章中，你还将学习如何使用 Vagrant 同步文件，并了解不同的配置选项。

第九章，*Shell 脚本 - 配置 Vagrant 环境*，讲解了在 Vagrant 中的配置过程，这是 Vagrant 另一个强大的功能，使你能够轻松配置 Vagrant 虚拟机。本章作为配置介绍，将帮助你了解更多关于配置管理工具、Shell 配置和文件配置的内容。当使用这些配置类型时，还有多种配置选项可供学习。

第十章，*Ansible - 使用 Ansible 配置 Vagrant 环境*，教你如何使用 Ansible 和 Ansible 剧本来配置 Vagrant 环境。你还将简要了解如何在 Vagrant 虚拟机上安装 Ansible，然后学习如何在主机上使用 Ansible 来配置 Vagrant 虚拟机。

第十一章，*Chef - 使用 Chef 配置 Vagrant 环境*，教你如何使用 Chef 和 Chef 食谱来配置 Vagrant 环境。你将学习如何使用基本选项 Chef Solo 和高级选项 Chef Client 来配置机器。

第十二章，*Docker - 使用 Docker 配置 Vagrant 环境*，深入探讨了如何使用 Docker 配置 Vagrant 环境。我们将学习如何从 Docker Hub 搜索并拉取镜像，然后将其作为容器运行。我们还将了解使用 Docker 作为 Vagrant 配置工具时 Docker 接受的不同选项。

第十三章，*Puppet - 使用 Puppet 配置 Vagrant 环境*，探讨了如何使用 Puppet 配置 Vagrant 环境。你将学习 Vagrant 提供的两个主要选项：Puppet Apply 和 Puppet Agent。使用 Puppet Agent，你将看到如何连接到 Puppet 主控并从中获取指令。

第十四章，*Salt - 使用 Salt 配置 Vagrant 环境*，讲解了如何使用 Salt 配置 Vagrant 环境。你还将了解 Salt 状态，它允许我们指定在配置过程中应该添加哪些包和服务。

# 为了最大限度地发挥本书的作用

本书适合初学者和高级用户。它将教你如何安装所需的软件。如果你已经安装了这些软件，请检查你安装的版本，因为可能与你所使用的版本存在差异。你可能需要升级软件。你将需要：

+   VirtualBox 版本：5.2.10

+   Vagrant 版本：2.0.4

+   Ubuntu 虚拟机（来自 Vagrant cloud）版本：ubuntu/xenial64 20180510.0.0

值得多读几遍每一章，确保不会遗漏任何信息。如果需要更多信息或澄清，官方 Vagrant 网站的文档非常棒。

# 下载示例代码文件

你可以从你的帐户下载本书的示例代码文件，网址为[www.packtpub.com](http://www.packtpub.com)。如果你在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)，注册后文件将直接发送到你的邮箱。

你可以按照以下步骤下载代码文件：

1.  登录或注册到[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址是[`github.com/PacktPublishing/Hands-On-DevOps-with-Vagrant`](https://github.com/PacktPublishing/Hands-On-DevOps-with-Vagrant)。如果代码有任何更新，它将会在现有的 GitHub 仓库中更新。

我们的丰富图书和视频目录中还提供了其他的代码包，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

# 下载彩色图像

我们还提供了一份 PDF 文件，里面包含了本书中使用的截图/图表的彩色图像。你可以下载它：[`www.packtpub.com/sites/default/files/downloads/9781789138054_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789138054_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。例如："该元数据通常存储为 JSON 文档。文件名将是`metadata.json`。"

一段代码如下所示：

```
Vagrant.configure("2") do |config|
     config.vm.box = "base"
 end
```

所有命令行输入或输出都如下所示：

```
config.vm.network "public_network", ip: "192.168.1.123"
```

**粗体**：表示一个新术语、重要单词或屏幕上显示的单词。例如，菜单或对话框中的单词在文本中会以这种方式出现。示例："这里列出了受支持的操作系统，但我们需要点击 macOS 上的`Latest Releases`部分。"

警告或重要提示会以这种方式显示。

提示和技巧以这种方式显示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请通过电子邮件`feedback@packtpub.com`与我们联系，并在邮件主题中注明书名。如果你对本书的任何内容有疑问，请通过`questions@packtpub.com`发送邮件与我们联系。

**勘误**：尽管我们已尽力确保内容的准确性，但仍可能发生错误。如果你在本书中发现错误，请通过访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)并选择你的书籍，点击“勘误提交表格”链接并输入详细信息来报告错误，我们将非常感激。

**盗版**：如果你在互联网上发现任何我们作品的非法复制品，无论其形式如何，我们将非常感激你提供相关的地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上该材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域具有专业知识，并且有意写书或参与书籍的编写，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦你阅读并使用了本书，为什么不在你购买它的站点上留下评论呢？潜在的读者可以看到并参考你的公正意见，从而做出购买决策，我们 Packt 也能了解你对我们产品的看法，而我们的作者则能看到你对他们书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packtpub.com](https://www.packtpub.com/)。
