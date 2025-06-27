# *第十一章*：与 Datadog 集成

在上一章中，您了解了一些重要的监控标准以及它们如何在 Datadog 中实现，目的是扩展 Datadog 监控平台的功能。到目前为止，在本书的这一部分，我们一直只关注于扩展专注于 Datadog 的组织的监控能力。与 Datadog 的集成是双向的——除了向 Datadog 填充与监控相关的数据以便与各种 Datadog 功能一起使用外，Datadog 中的信息也可以被其他内部应用程序利用。

要在应用程序之间推出此类通用集成，应该提供丰富的 API 集合。我们已经看到，Datadog REST API 是一个全面的编程接口，其他应用程序可以利用它来访问 Datadog 平台，发布和提取信息。我们还介绍了 DogStatsD 作为一种向 Datadog 平台发布信息的方法，并且我们将进一步了解如何从其他应用程序使用该接口。我们还将回顾其他方法，这些方法主要是社区驱动的努力，并非官方与 Datadog 产品套件一起发布，但在推出自定义集成时非常有用。

在本章中，您将学习关于 Datadog 集成的常用库，并具体涵盖以下主题：

+   使用客户端库

+   评估社区项目

+   开发集成

# 技术要求

要实现本章中的示例，您需要安装以下工具的环境：

+   Datadog 代理

+   `pip3`

+   **Python 2.7**（可选）

+   Datadog 开发工具包

+   Git 客户端

# 使用客户端库

在本节中，我们将看两类不同的客户端库——第一类是 Datadog REST API 的封装库，第二类是本地的 **DogStatsD** 客户端库。这两类库都适用于流行的编程语言，如 **C++**、**Java**、**Python**、**Java** 和 **Go**。

Datadog 为大多数编程语言提供了这两类库。尽管许多社区库在官方 Datadog 网站上有列出，但我们只关注那些正在积极维护的库。

## 基于 REST API 的客户端库

基本的 Datadog 客户端库是 REST API 集合，而特定编程语言的库本质上是 REST API 上的封装，便于 API 的使用。在本节中，我们将探讨一些重要的特定编程语言的客户端库，并且在相关的地方，我们也会展示示例代码。

### Datadog Python 库

我们已经在 *第九章**《使用 Datadog API》中看到过这个库的实际应用。这款官方库支持 REST API 和 DogStatsD，可以通过编程方式与 Datadog 进行交互。

该库可以通过 Python 安装工具 `pip` 安装到您的 Python 环境中，安装方法如下：

```
$ pip install datadog
```

代码可以在 GitHub 上找到，网址是 [`github.com/DataDog/datadogpy`](https://github.com/DataDog/datadogpy)，并且该库也可以从源代码安装。要执行此操作，克隆代码库到本地环境中，并按如下方式运行设置程序：

```
$ git clone https://github.com/DataDog/datadogpy.git
$ cd datadogpy
$ python setup.py install
```

安装后，可以通过导入 `API` 模块在程序中执行 REST API 特定的调用，也可以通过将 `statsd` 模块导入 Python 环境来执行 DogStatsD 特定的调用。

### Datadog 的 Python API 客户端

这是一个官方的 Python 库，映射了所有公共 Datadog REST API 的集合。它可以在 GitHub 上找到，网址是 [`github.com/DataDog/datadog-api-client-python`](https://github.com/DataDog/datadog-api-client-python)。使用 `pip`，可以按如下方式在兼容的 Python 环境中安装该库：

```
$ pip3 install datadog-api-client
```

目前，该客户端仅与 **Python 3.6** 及以上版本兼容。

### Datadog 的 Java 客户端

在开发企业级应用程序时，Java 是最流行的编程语言之一。因此，如果 Java 应用程序需要直接与 Datadog 集成，这个官方 Java 客户端库（它是 Datadog REST API 的封装）是默认选择。

与该客户端库相关的代码库可以在 GitHub 上找到，网址是 [`github.com/DataDog/datadog-api-client-java`](https://github.com/DataDog/datadog-api-client-java)。请查阅那里提供的文档，以了解如何使用 **Maven** 和 **Gradle** 等构建工具，在 Java 开发环境中构建和安装此 Java 库。

### Datadog 的 Go API 客户端

Go 是一种现代的编译型语言，它具有快速的执行速度，但又具备像 Python 这样的解释型语言的灵活性。虽然它是一种通用编程语言，但在系统编程中非常流行，并且被广泛用于在 DevOps 领域构建**命令行接口**（**CLI**）工具。例如，最新版本的 Datadog Agent 部分就是用 Go 开发的。

Datadog REST API 的官方 Go 客户端库在 GitHub 上维护，网址是 [`github.com/DataDog/datadog-api-client-go`](https://github.com/DataDog/datadog-api-client-go)。构建和安装该库的详细信息也可以在该位置找到。

### Datadog 的 Node.js 客户端

Node.js 平台用于在服务器端运行 JavaScript，且在开发基于 Web 浏览器的用户界面时非常流行。Datadog 并没有提供官方的客户端库来支持这个平台，但可以使用由 *Brett Langdon* 提供的 `node-dogapi` 代码库，GitHub 上的链接为 [`github.com/brettlangdon/node-dogapi`](https://github.com/brettlangdon/node-dogapi)。该代码库最近没有更新，其与 Datadog API 的兼容性需要验证，以确保符合预期的使用要求。

### WebService-DataDog – 一个 Perl 客户端

在 Python 广泛使用之前，**Perl** 曾是构建系统工具的默认脚本语言，尤其是在处理日志和文本数据方面。这个由 Jennifer Pinkham 维护的 Perl 客户端库最近没有更新，但对于 Perl 爱好者来说，仍然值得一试。代码库可在 GitHub 上找到，链接为 [`github.com/jpinkham/webservice-datadog`](https://github.com/jpinkham/webservice-datadog)，并附有如何安装相关 Perl 模块的步骤说明。

### Ruby 客户端用于 Datadog API

**Ruby** 是一种脚本语言，主要用于构建 Web 应用程序，特别是结合 **Ruby on Rails** 开发框架使用。然而，它和 Python、PHP、Perl 一样，都是通用编程语言。Datadog 为 Ruby 提供了一个官方客户端库，这是在 Datadog REST API 之上的抽象层。

代码库可以在 GitHub 上找到，链接为 [`github.com/DataDog/dogapi-rb`](https://github.com/DataDog/dogapi-rb)，附有安装库的步骤和如何在 Ruby 中使用 Datadog API 的代码示例。

## DogStatsD 客户端库

如在 *第十章* 中提到的，*与监控标准一起工作*，DogStatsD 是监控标准 StatsD 的一种实现。因此，任何通用的 StatsD 实现都可以与 Datadog Agent 提供的 DogStatsD 接口一起使用。这里回顾的社区库利用了这一特性，从而为目标编程语言提供了一个封装。

### C++ DataDog StatsD 客户端

使用这个库，指标和事件可以通过 StatsD 接口发布到 Datadog 后端。代码库可以在 GitHub 上找到，链接为 [`github.com/BoardiesITSolutions/cpp-datadogstatsd`](https://github.com/BoardiesITSolutions/cpp-datadogstatsd)。

代码可以被构建成适用于目标操作系统的共享库，通常是 Linux 发行版。一些 Windows 平台也得到支持。需要发布指标数据和事件的自定义应用程序可以动态链接到这个共享库。

### Java DogStatsD 客户端

对于**Java**，Datadog 提供了一个官方的 DogStatsD 客户端库，地址为[`github.com/DataDog/java-dogstatsd-client`](https://github.com/DataDog/java-dogstatsd-client)在 GitHub 上。它比标准的 StatsD 库支持更多功能，标准 StatsD 库仅限于发布度量数据。使用 Java DogStatsD 客户端，你还可以维护事件和服务检查。

可以通过 Maven 导入特定版本的客户端 JAR 文件到你的项目中，具体配置如下所示：

```
<dependency>
    <groupId>com.datadoghq</groupId>
    <artifactId>java-dogstatsd-client</artifactId>
    <version>2.11.0</version>
</dependency>
```

可以从 Java 应用程序调用 StatsD API，并在添加了前述配置到 Maven 设置之后构建。

### C#的 DogStatsD 客户端

**C#**是像 Java 和**C++**一样的通用编程语言，是最初由微软推广的**.NET**应用开发框架的一部分。它在 Windows 平台和多个 Linux 发行版上得到支持。此库由 Datadog 维护，源代码仓库可以在 GitHub 上找到，地址为[`github.com/DataDog/dogstatsd-csharp-client`](https://github.com/DataDog/dogstatsd-csharp-client)。

这个流行的客户端库可以通过**NuGet**上的软件包或通过 GitHub 上的源代码进行安装。与其他官方 DogStatsD 客户端库一样，除了标准的度量指标支持外，它还支持事件和服务检查。安装和库使用的详细信息可以在 GitHub 上的代码仓库中找到。

### datadog-go

**datadog-go**是一个为 Go 编程语言提供的 DogStatsD 客户端库，由 Datadog 在 GitHub 上的[`github.com/DataDog/datadog-go`](https://github.com/DataDog/datadog-go)进行维护。与其他官方的 DogStatsD 客户端库一样，它除了支持度量指标外，还支持事件和服务检查。

该库正式支持**Go**版本**1.12**及以上。安装和使用该库的详细信息可以在 GitHub 上的代码仓库中找到。

### dogstatsd-ruby

**dogstatsd-ruby**是一个为 Ruby 编程语言提供的 DogStatsD 客户端库，由 Datadog 在 GitHub 上的[`github.com/DataDog/dogstatsd-ruby`](https://github.com/DataDog/dogstatsd-ruby)进行维护。与其他官方的 DogStatsD 客户端库一样，它也支持事件和服务检查，除了度量指标外。

该库的安装和使用的详细信息可以在 GitHub 上的代码仓库中找到。完整的 API 文档可以在[`www.rubydoc.info/github/DataDog/dogstatsd-ruby/master/Datadog/Statsd`](https://www.rubydoc.info/github/DataDog/dogstatsd-ruby/master/Datadog/Statsd)查阅。

### 社区版 DogStatsD 客户端库

虽然 Datadog 为流行的编程语言提供了由 Datadog 维护的 DogStatsD 客户端库，但与基于 REST API 的客户端库一样，社区也积极支持其他语言，如 Node.js 和 Perl。通常，基于社区的库是通用 StatsD 库的封装，并且仅支持指标。以下是一些值得注意的库：

+   **Node.js 的 Host-shots 客户端库**：此库可以在 GitHub 上的 [`github.com/brightcove/hot-shots`](https://github.com/brightcove/hot-shots) 找到。这是一个通用的客户端库，支持其他提供 StatsD 接口的监控工具。

+   **NodeDogStatsD**：这是另一个 Node.js 客户端库，代码仓库和文档可以在 GitHub 上的 [`github.com/mrbar42/node-dogstatsd`](https://github.com/mrbar42/node-dogstatsd) 找到。

+   **DataDog DogStatsD – 用于 DogStatsD 的 Perl 模块**：使用此模块，可以从 Perl 程序向 Datadog 发布指标数据。代码和文档可以在 GitHub 上的 [`github.com/binary-com/dogstatsd-perl`](https://github.com/binary-com/dogstatsd-perl) 找到。

要查看完整且最新的客户端库列表，请访问官方编译的 [`docs.datadoghq.com/developers/libraries/`](https://docs.datadoghq.com/developers/libraries/)。

在本节中，你已经了解了两组可以与主要编程语言一起使用的客户端库，以访问 Datadog 资源。这些客户端库对于从头开始构建 Datadog 特定功能（例如程序或脚本）非常有用，作为与 Datadog SaaS 后端集成的一部分。在下一节中，我们将介绍一些提供良好集成功能或构建块的工具，用于开发这些集成。

# 评估社区项目

还有一些由其他公司和社区团体开发的工具，可以让你在 Datadog 相关的集成和自动化方面更加轻松。在本节中，我们将介绍一些在这一类别中非常有用的工具和框架。

## Brightcove 的 dog-watcher

在一个大规模环境中，若有多个仪表板和监控项基于 Datadog 构建用于操作使用，维护它们可能会迅速变成一项重大工作。这个 Node.js 实用工具可以用来备份 Datadog 的仪表板和监控项，以 JSON 格式保存到 Git 仓库中。这样的备份也在重新创建相似资源到同一账户或其他地方时非常有用。

该工具需要作为 Node.js 服务运行。代码和配置运行的详细信息可以在 GitHub 上的 [`github.com/brightcove/dog-watcher`](https://github.com/brightcove/dog-watcher) 找到。它可以安排定期备份，或者在跟踪的 Datadog 资源发生变化时备份。

## kennel

**kennel** 是一个用 Ruby 开发的工具，可以作为代码来管理 Datadog 监控、仪表盘和 **服务水平对象**（**SLOs**）。将各种基础设施资源作为代码进行管理是 DevOps 的一个原则，这个工具在实现这一目标中非常有用。该工具的代码和详细文档可以在 GitHub 上找到，地址是 [`github.com/grosser/kennel`](https://github.com/grosser/kennel)。

## 使用 Terraform 管理监控

**Terraform** 是一个通用工具，用于搭建和维护 **基础设施即代码**（**IaC**）。它可以通过在 Terraform 配置语言中定义监控来管理 Datadog 监控。

Terraform 通过将资源的当前状态与代码中的定义进行匹配，来维护它所管理的资源的状态。如果资源不存在，它将会被创建。

可以使用 Terraform 管理多种 Datadog 资源和集成，其中一些重要的资源如下：

+   用户

+   角色

+   指标

+   监控

+   仪表盘

+   停机时间

+   与公共云平台的集成

关于这些资源的标准 Terraform 文档可以在 [`registry.terraform.io/providers/DataDog/datadog/latest/docs`](https://registry.terraform.io/providers/DataDog/datadog/latest/docs) 中找到。

## Ansible 模块和集成

**Ansible** 是一个配置管理工具，由于其通用性，除了核心配置管理功能外，它在 DevOps 工程师中非常受欢迎。在基础设施管理方面，它与 Terraform 非常相似，直接支持 Datadog 资源。

一般来说，Ansible 提供了模块来支持某种类型的基础设施资源。目前，已经有 Ansible 模块可以发布事件和管理监控。通过这些模块，可以构建 Ansible playbooks 来管理 Datadog 账户中的事件和监控。

Datadog 还提供了一个官方的 Ansible 集成。它可以通过回调机制跟踪 Ansible playbook 的执行。然而，就公开到 Datadog 平台的信息而言，这个集成并不十分有用。

Datadog 提供了许多应用程序和公共云平台及服务的集成。如果您选择的第三方工具或需要通过 Datadog 监控的内部应用没有现成的集成，您可以开发一个自定义集成。我们将在下一节学习如何开发和部署自定义集成的基础知识。

# 开发集成

在 *第八章*，*与平台组件集成*，您学习了如何配置集成。Datadog 提供了许多与第三方应用程序集成的官方集成，这些应用程序被用于构建运行软件应用程序的云平台。使用官方集成的最大好处是，经过最少的配置后，该集成特定的指标将可以在仪表板、监视器和其他 Datadog 资源中使用。

Datadog 允许您构建自定义集成，使其像官方集成一样工作。这需要 DevOps 专业知识，特别是在 Python 编程方面的技能，而且学习 Datadog 提供的构建与 Datadog Agent 兼容的集成程序并不容易。然而，出于以下原因，构建集成可能是有意义的：

+   **为内部应用程序构建集成**：尽管是内部使用，但该应用程序可能会在生产环境中大规模部署，Datadog 集成有助于标准化该应用程序的监控需求。

+   **为第三方应用程序构建集成**：监控需求与上一使用案例中描述的内部应用程序类似，但尚无官方或社区级的集成，或者需求没有得到满足。

+   **为面向外部使用的应用程序提供监控支持**：您可能有一个面向外部用户的应用程序作为第三方软件，向 Datadog 提供集成可能是该应用程序监控支持策略的一部分。

在本节中，您将学习从头开始构建集成的步骤。构建一个完整的集成超出了本章的范围，但您将通过一些实践操作了解一般步骤。

## 先决条件

在本地开发环境中需要进行一些设置，以便开发、测试、构建、打包和部署集成：

+   安装 Python 3.8 及以上版本，Python 2.7 为可选项。

+   开发者工具包。可以使用 `pip3` 工具安装，方法如下：

    ```
    $ pip3 install "datadog-checks-dev[cli]"
    ```

    这将安装许多内容到本地 Python 3 环境中，如果一切顺利，输出将如下所示：

    ```
    Successfully installed PyYAML-5.3.1 Pygments-2.8.0 appdirs-1.4.4 atomicwrites-1.4.0 attrs-20.3.0 bcrypt-3.2.0 bleach-3.3.0 cached-property-1.5.2 certifi-2020.12.5 cffi-1.14.5 chardet-4.0.0 colorama-0.4.4 coverage-5.5 cryptography-3.4.6 datadog-checks-dev-9.0.0 distlib-0.3.1 distro-1.5.0 docker-4.4.4 docker-compose-1.28.5 dockerpty-0.4.1 docopt-0.6.2 docutils-0.16 filelock-3.0.12 idna-2.10 in-toto-1.0.1 iniconfig-1.1.1 iso8601-0.1.14 jsonschema-3.2.0 keyring-22.3.0 markdown-3.3.4 mock-4.0.3 packaging-20.9 paramiko-2.7.2 pathspec-0.8.1 pip-tools-5.5.0 pkginfo-1.7.0 pluggy-0.13.1 psutil-5.8.0 py-1.10.0 py-cpuinfo-7.0.0 pycparser-2.20 pygal-2.4.0 pygaljs-1.0.2 pynacl-1.4.0 pyparsing-2.4.7 pyperclip-1.8.2 pyrsistent-0.17.3 pytest-6.2.2 pytest-benchmark-3.2.3 pytest-cov-2.11.1 pytest-mock-3.5.1 python-dateutil-2.8.1 python-dotenv-0.15.0 readme-renderer-29.0 requests-2.25.1 requests-toolbelt-0.9.1 rfc3986-1.4.0 securesystemslib-0.20.0 semver-2.13.0 tenacity-6.3.1 texttable-1.6.3 toml-0.10.2 tox-3.22.0 tqdm-4.58.0 twine-3.3.0 urllib3-1.26.3 virtualenv-20.4.2 webencodings-0.5.1 websocket-client-0.57.0
    ```

+   Docker，用于运行单元测试和集成测试。

+   如果集成需要通过部署 Datadog Agent 来进行测试，请确保已安装该 Agent。请注意，集成包可以部署到任何地方，因此 Datadog Agent 不需要在开发和测试集成兼容性的位置本地运行。

这里提供的命令和输出已在类 Unix 系统（如 Linux 或 Mac）上验证。也可以在 Windows 上进行开发；有关平台特定的操作，请参见官方文档。

## 设置工具链

`integrations-extras` 代码仓库需要从 GitHub 克隆到本地环境，以便为开发和构建集成提供所有框架。按照以下步骤进行设置：

1.  在你的主目录中创建一个开发工作目录：

    ```
    $ mkdir dd
    ```

1.  克隆 `integrations-extras` 仓库：

    ```
    $ cd dd
    $ git clone https://github.com/DataDog/integrations-extras.git
    ```

1.  将 `integrations-extras` 设置为默认仓库：

    ```
    $ cd integrations-extras
    $ ddev config set repo extras
    ```

接下来，我们将为集成设置一个专用文件夹。

## 创建集成文件夹

对于新集成，开发工具包可以创建整个目录结构，并填充模板文件。你可以尝试进行演练，以查看将要创建的目录结构和文件。

为了更好地描述这些步骤，假设你有一个名为 *CityWeather* 的应用，它随时提供特定城市的天气相关信息。集成的目标是将这些天气信息的一部分发布到 Datadog。

创建目录的演练可以按如下方式进行：

```
$ ddev create -n CityWeather
```

输出将显示框架中目录和文件的层次列表；为了简洁起见，这里没有提供。

可以通过运行相同的命令而不带 `-n` 选项来创建目录结构。尽管使用的名称包含混合字符，但顶级目录名称将全部小写。因此，你可以按如下方式进入新创建的目录：

```
$ cd cityweather
```

在该目录中，你将找到以下文件和目录：

+   `README.md`：用于在 Git 中记录新集成的 README 文件。提供的模板具有正确的头部和格式，以确保文档标准化。

+   `manifest.json`：描述集成和文件位置的清单文件。

+   `tests`：用于配置和维护单元测试和集成测试的目录。

+   `metadata.csv`：维护所有收集到的度量指标列表。

+   `tox.ini`：现在，`tox` 用于运行测试。确保该配置文件中指定的 Python 版本与本地可用的 Python 版本匹配。例如，如果使用的是 Python 2.7 和 Python 3.9，该文件的内容将如下所示，并根据需要进行更改：

    ```
    [tox]
    minversion = 2.0
    skip_missing_interpreters = true
    basepython = py39
    envlist = py{27,39}
    [testenv]
    ensure_default_envdir = true
    envdir =
        py27: {toxworkdir}/py27
        py39: {toxworkdir}/py39
    dd_check_style = true
    usedevelop = true
    platform = linux|darwin|win32
    deps =
        datadog-checks-base[deps]>=6.6.0
        -rrequirements-dev.txt
    passenv =
        DOCKER*
        COMPOSE*
    commands =
        pip install -r requirements.in
        pytest -v {posargs}
    ```

现在，本地已经具备了开发集成的工具，接下来让我们尝试使用仓库中现有的集成来执行其余步骤。请注意，在这个仓库中大约有 100 个额外的集成，你可以通过构建相应的相关包来使用它们，接下来你将学到这个技巧。

为了测试和部署的实践目的，我们选择一个 **Zabbix** 集成。这是一个流行的本地监控工具，广泛应用于数据中心和云平台。许多正在迁移到 Datadog 的公司可能需要处理现有的 Zabbix 安装，更实用的策略是与 Zabbix 集成，而不是试图替代它，重点是为监控任何新基础设施推出 Datadog。在这种情况下，Datadog 和 Zabbix（或其他本地监控应用）将并行运行。

## 运行测试

对于集成开发的代码，可以借助 Docker 在本地运行单元测试和集成测试。对于 Zabbix 集成，Zabbix 将作为 Docker 上的微服务本地运行，并且测试将针对该实例运行。Zabbix 部署的详细信息提供在 `docker-compose.yml` 文件中。

按照以下步骤部署 Zabbix 并测试集成：

1.  切换到 Git 仓库的顶层目录。

1.  切换到 Zabbix 集成的目录：

    ```
    $ cd zabbix
    ```

1.  查找 `docker-compose.yml` 文件：

    ```
    $ cat tests/compose/docker-compose.yml 
    ```

    这里没有提供输出结果，以简洁为主。

1.  运行测试：

    ```
    $ ddev test zabbix
    ```

1.  输出信息较为详细，最后会以类似以下的消息结束，表示测试成功：

    ```
      py39: commands succeeded
      style: commands succeeded
      congratulations :)
    Passed!
    ```

接下来，让我们看看如何为集成构建配置文件。

## 构建配置文件

集成提供的示例配置文件 `conf.yaml.example` 是从 `assets/configuration/spec.yaml` 中的模板文件生成的。更改模板文件后，可以按如下方式生成示例配置文件：

```
$ ddev validate config --sync zabbix
Validating default configuration files...
Writing config file to `/Users/thomastheakanath/dd/integrations-extras/zabbix/datadog_checks/zabbix/data/conf.yaml.example`
All 2 configuration files are valid!
```

接下来，让我们看看如何将集成代码打包以便安装。

## 构建包

要部署集成，需要将其打包成 wheel 格式，这是一种用于打包和分发 Python 程序的格式。wheel 文件仅包含集成功能所需的文件，并不会包含构建集成所用的大多数源文件。可以按如下方式构建集成：

```
$ ddev release build zabbix
Building `zabbix`...
'build/lib' does not exist -- can't clean it
'build/bdist.macosx-10.13-x86_64' does not exist -- can't clean it
'build/scripts-3.9' does not exist -- can't clean it
warning: no previously-included files matching '__pycache__' found anywhere in distribution
Build done, artifact(s) in: /Users/thomastheakanath/dd/integrations-extras/zabbix/dist
Success!
```

wheel 文件可以在构建命令输出中提到的位置找到：

```
$ ls dist/*.whl
dist/datadog_zabbix-1.0.0-py2.py3-none-any.whl
```

接下来，让我们看看如何使用刚构建的包安装集成。

## 部署集成

如果 Datadog Agent 在本地运行，可以通过指向上一步中构建的 wheel 来安装它。wheel 文件必须复制到计划安装的其他位置。一旦 wheel 文件在本地可用，可以按如下方式安装：

```
$ sudo –u dd-agent datadog-agent integration install -w dist/datadog_zabbix-1.0.0-py2.py3-none-any.whl
For your security, only use this to install wheels containing an Agent integration and coming from a known source. The Agent cannot perform any verification on local wheels.
Processing  ./dist/datadog_zabbix-1.0.0-py2.py3-none-any.whl
Installing collected packages: datadog-zabbix
Successfully installed datadog-zabbix-1.0.0
Successfully copied configuration file conf.yaml.example
Successfully completed the installation of datadog-zabbix
```

这基本上会使集成可以供 Datadog Agent 使用，并作为官方集成与 Datadog Agent 一起启用。如果你检查 Datadog Agent 主目录下的 `conf.d` 目录，你可以看到 `zabbix.d` 列出如下：

```
$ ls zabbix.d
conf.yaml.example
```

与其他标准集成一样，要启用它，需要根据提供的示例文件创建`conf.yaml`，并重启 Datadog Agent 服务。

构建 Datadog 集成的完整流程在[`docs.datadoghq.com/developers/integrations/new_check_howto`](https://docs.datadoghq.com/developers/integrations/new_check_howto)上有官方文档。请参考该文档以获取更详细的细节和更新。现在，让我们来看看与你刚才看到的主题相关的最佳实践。

# 最佳实践

客户端库的可用性以及构建自定义集成的选项为你提供了很大的灵活性，可以将 Datadog 与其他应用程序甚至批处理作业集成。然而，在开始使用这些选项之一或多个进行自动化或自定义时，你需要先了解一些最佳实践：

+   如果你能选择编程语言，建议选择更受支持和流行的语言，例如用于脚本的 Python 和用于企业应用的 Java。如果需要集成的应用主要运行在 Microsoft Windows 平台上，选择 C#是明智的。

+   选择一个官方维护的客户端库。这是显而易见的——你需要依赖一个能够跟上 Datadog 平台和 REST API 增强功能的库。

+   计划使用 Terraform 将 Datadog 资源管理为代码。Ansible 也可以提供帮助，但目前它对 Datadog 的支持有限。

+   如果你构建了一个集成，并且它有外部使用，请将其发布到 Datadog 的**integrations-extras** GitHub 仓库。其他人的使用可以帮助获得有价值的反馈和修复，使其更加健壮和实用。

遵循这些最佳实践和相关模式将帮助你选择正确的方法来实现集成需求。

# 总结

到现在为止，你应该已经了解了 Datadog 中针对各种使用场景提供的所有重要集成功能；让我们回顾一下本章具体讨论的内容。

针对流行编程语言的许多 Datadog 客户端库是可用的，既有官方维护的，也有社区层级的。有两种类型的客户端库——一种提供对 Datadog REST API 的语言封装，另一种通过与 StatsD 兼容的 DogStatsD 服务提供 Datadog 接口支持。此外，GitHub 上也有社区级的 Datadog 集成项目。

这里没有讨论其他类型的 Datadog 客户端库，例如**APM**和**分布式追踪库**，支持无服务器计算资源（如**AWS Lambda**）的库，以及专门针对 Datadog 日志管理功能的客户端库。这些库的使用方式与核心 API 和 DogStatsD 库的使用方式没有不同，如果你有相关的使用场景，应该查看这些库。

本章完成了本书的*第三部分*，*扩展 Datadog*。在本书的下一部分，你将学习 Datadog 实现的更高级的监控概念和功能。我们之前简要地看了微服务监控，接下来的章节中，你将深入了解微服务监控，特别是如何在 Kubernetes 编排的环境中进行监控。
