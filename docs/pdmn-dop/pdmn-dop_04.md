# *第三章*：运行第一个容器

在前几章中，我们讨论了容器的历史、其采用情况以及促使其传播的各种技术，同时也看到了 **Docker** 和 **Podman** 之间的主要区别。

现在，到了开始使用真实示例的时候：在本章中，我们将学习如何在你首选的 Linux 操作系统上安装并运行 Podman，以便我们可以启动我们的第一个容器。我们将了解各种安装方法，所有的先决条件，然后启动一个容器。

在本章中，我们将讨论以下主要主题：

+   选择操作系统和安装方法

+   准备环境

+   运行第一个容器

# 技术要求

拥有良好的 Linux 操作系统管理经验将有助于理解本章提供的关键概念。

我们将介绍在各种 Linux 发行版上安装新软件的主要步骤，因此，作为一名 Linux 系统管理员拥有一定经验，在解决安装过程中可能出现的问题时会有所帮助。

此外，前几章中解释的一些理论概念有助于你理解本章描述的过程。

# 选择操作系统和安装方法

Podman 在不同的发行版和操作系统中得到支持。它非常容易安装，并且各个发行版现在提供了自己维护的安装包，可以通过它们各自的包管理器进行安装。

在本节中，我们将介绍最常见的 GNU/Linux 发行版的不同安装步骤，以及在 macOS 和 Windows 上的安装步骤，尽管本书的重点是 Linux 环境。

作为附加话题，我们还将学习如何从源代码直接构建 Podman。

## 在 Linux 发行版和其他操作系统之间的选择

在不同的 GNU/Linux 发行版之间的选择是由用户的偏好和需求决定的，这些偏好和需求通常受到本书范围外的几个因素的影响。

目前，许多高级用户选择 Linux 发行版作为他们的主要操作系统。然而，尤其在开发者中，仍然有一大部分人坚持将 macOS 作为标准操作系统。微软的 Windows 仍然在桌面工作站和笔记本电脑中占据着最大的市场份额。

如今，我们拥有一个庞大的 Linux 发行版生态系统，这些发行版从 Debian、Fedora、Red Hat 企业版 Linux、Gentoo、Arch 和 openSUSE 等少数核心历史发行版中发展而来。像 **DistroWatch** 这样的专业网站（[`distrowatch.com`](https://distrowatch.com)）追踪着 Linux 和 BSD 发行版的众多版本。

尽管运行的是 Linux 内核，不同的发行版在用户空间行为的架构方法上存在差异，例如文件系统结构、库或用于交付软件发布的打包系统。

另一个显著的区别与安全性和强制访问控制子系统有关：例如，Fedora、CentOS、Red Hat Enterprise Linux 及其所有衍生版依赖于**SELinux**作为其强制访问控制子系统。另一方面，Debian、Ubuntu 及其衍生版则基于类似的解决方案——**AppArmor**。

Podman 与 SELinux 和 AppArmor 交互，以提供更好的容器隔离，但其底层接口是不同的。

重要提示

本书中的所有示例和源代码均使用**Fedora Workstation 34**作为参考操作系统编写和测试。

那些希望在实验室中尽可能复现本书环境的读者有不同的选择：

+   使用 Fedora 34 Vagrant Box ([`app.vagrantup.com/fedora/boxes/34-cloud-base`](https://app.vagrantup.com/fedora/boxes/34-cloud-base))。**Vagrant**是由**Hashicorp**开发的软件解决方案，用于创建快速、轻量级的虚拟机，特别适用于开发使用。有关 Vagrant 及如何在你选择的操作系统上使用它的详细信息，请参阅[`www.vagrantup.com/`](https://www.vagrantup.com/)。

+   直接下载云镜像([`alt.fedoraproject.org/cloud/`](https://alt.fedoraproject.org/cloud/))，并在公共/私人云上创建实例，或者直接在你选择的虚拟化平台上部署它。

+   手动安装 Fedora Workstation。在这种情况下，官方安装指南([`docs.fedoraproject.org/en-US/fedora/f34/install-guide/`](https://docs.fedoraproject.org/en-US/fedora/f34/install-guide/))提供了关于部署操作系统的详细说明。

在公共云上运行实例是无法在本地运行虚拟机的用户的最佳选择。

提供商如 Amazon Web Services、Google Cloud Platform、Microsoft Azure 和 DigitalOcean 也提供基于 Fedora 的即用云实例，且小型实例的月租价格非常低。

价格可能随着时间和不同等级而变化，跟踪这些变化超出了本书的范围。几乎所有提供商都提供免费的学习或基础使用计划，且小型/微型套餐的价格非常低。

容器是基于 Linux 的，不同的容器引擎和运行时与 Linux 内核和库交互以运行。Windows 最近引入了对本地容器的支持，其隔离方法与前面描述的 Linux 命名空间概念非常相似。然而，仅支持 Windows 基础镜像的本地运行，并不是所有容器引擎都支持本地执行。

对 macOS 来说，同样的考虑事项也适用：它的架构并非基于 Linux，而是基于一种名为 **XNU** 的混合 Mach/BSD 内核。因此，它不提供运行容器所需的 Linux 内核特性。

对于 Windows 和 macOS，都需要一个虚拟化层来抽象出 Linux 机器，以运行本地 Linux 容器。

Podman 提供了适用于 Windows 和 macOS 的远程客户端功能，使用户能够连接到本地或远程的 Linux 主机。

Windows 用户也可以受益于基于 **Windows Subsystem for Linux** (**WSL**) **2.0** 的替代方法，这是一种兼容层，通过 Hyper-V 虚拟化支持运行轻量级虚拟机，以提供 Linux 内核接口和 Linux 用户空间二进制文件。

以下章节将涵盖在最流行的 Linux 发行版、macOS 和 Windows 上安装 Podman 所需的步骤。

### 在 Fedora 上安装 Podman

Fedora 包由其广泛的社区维护，并使用 **DNF** 包管理器进行管理。要安装 Podman，请在终端中运行以下命令：

```
# dnf install –y podman 
```

该命令安装 Podman 并配置带有配置文件的环境（将在下一节中详细介绍）。它还安装了 `systemd` 单元，以提供额外的功能，如 REST API 服务或容器自动更新。

### 在 CentOS 上安装 Podman

Podman 可以安装在 CentOS 7、CentOS 8 和 CentOS Stream 上 ([`www.centos.org/`](https://www.centos.org/))。在 CentOS 7 上安装的用户必须启用 **Extras** 仓库，而在 CentOS 8 和 Stream 上安装的用户必须从已经启用的 **AppStream** 仓库中获取 Podman 包。

要安装 Podman，请在终端中运行以下命令：

```
# yum install –y podman
```

与 Fedora 中一样，这个命令安装 Podman 及其所有依赖项，包括配置文件和 `systemd` 单元文件。

### 在 RHEL 上安装 Podman

要在 **Red Hat Enterprise Linux** (**RHEL**) ([`www.redhat.com/en/technologies/linux-platforms/enterprise-linux`](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)) 上安装 Podman，用户应根据 RHEL 7 和 RHEL 8 执行不同的安装程序。

在 RHEL 7 上，用户必须启用额外的渠道，然后安装 Podman 包：

```
# subscription-manager repos \
--enable=rhel-7-server-extras-rpms
# yum -y install podman
```

在 RHEL 8 上，Podman 包可通过一个名为 **container-tools** 的专用模块获取。模块是可以通过独立发布周期组织的 RPM 包的自定义集合：

```
# yum module enable -y container-tools:rhel8
# yum module install -y container-tools:rhel8
```

`container-tools` 模块安装了 Podman 和另外两个有用的工具，稍后在本书中会详细介绍：

+   **Skopeo**，一个用于管理 OCI 镜像和注册表的工具

+   **Buildah**，一个用于从 Dockerfiles 或从零开始构建自定义 OCI 镜像的专业工具

### （不）在 Fedora CoreOS 和 Fedora Silverblue 上安装 Podman

本小节的标题有点开玩笑的成分。实际上，Podman 已经在这两个发行版上安装并成为运行容器化工作负载的关键工具。

**Fedora CoreOS** 和 **Fedora SilverBlue** 发行版是不可变的、原子化的操作系统，分别旨在用于服务器/云和桌面环境。

Fedora CoreOS ([`getfedora.org/en/coreos/`](https://getfedora.org/en/coreos/)) 是 Red Hat CoreOS 的上游版本，是运行 Red Hat OpenShift 和 **OpenShift Kubernetes Distribution** (**OKD**) 的操作系统，OKD 是用于构建 Red Hat OpenShift 的社区版本 Kubernetes 发行版。

Fedora Silverblue ([`silverblue.fedoraproject.org/`](https://silverblue.fedoraproject.org/)) 是一个面向桌面的不可变操作系统，旨在提供稳定和舒适的桌面用户体验，特别是为从事容器开发的开发者提供支持。

因此，在 Fedora CoreOS 和 Fedora Silverblue 上，只需打开终端并运行 Podman。

### 在 Debian 上安装 Podman

**Debian** ([`www.debian.org/`](https://www.debian.org/)) 自版本 11 起提供 Podman，该版本的代号为 Bullseye（以《玩具总动员 2 和 3》中的著名玩具马命名）。

Debian 使用 `apt-get` 包管理工具来安装和升级系统软件包。

要在 Debian 系统上安装 Podman，请在终端中运行以下命令：

```
# apt-get –y install podman
```

前述命令安装了 Podman 二进制文件及其依赖项，以及其配置文件、`systemd` 单元和手册页。

### 在 Ubuntu 上安装 Podman

基于 Debian 构建的 **Ubuntu** ([`ubuntu.com/`](https://ubuntu.com/)) 在软件包管理方面表现类似。要在 Ubuntu 20.10 或更高版本上安装 Podman，请运行以下命令：

```
# apt-get -y update
# apt-get -y install podman
```

这两个命令将更新系统软件包，然后安装 Podman 二进制文件及相关依赖项。

### 在 openSUSE 上安装 Podman

**openSUSE** 发行版 ([`www.opensuse.org/`](https://www.opensuse.org/)) 由 SUSE 支持，并提供两种不同的版本——滚动发布的 **Tumbleweed** 和 LTS 版本的 **Leap**。Podman 可以通过 openSUSE 仓库安装，使用以下命令：

```
# zypper install podman
```

Zypper 包管理器将下载并安装所有必要的软件包和依赖项。

### 在 Gentoo 上安装 Podman

**Gentoo** ([`www.gentoo.org/`](https://www.gentoo.org/)) 是一个巧妙的发行版，其特点是直接在目标机器上构建已安装的软件包，并提供可选的用户自定义功能。为此，它使用 **Portage** 包管理器，灵感来自 FreeBSD ports。

要在 Gentoo 上安装 Podman，请运行以下命令：

```
# emerge app-emulation/podman
```

`emerge` 工具将下载并自动在系统上构建 Podman 源代码。

### 在 Arch Linux 上安装 Podman

**Arch Linux** ([`archlinux.org/`](https://archlinux.org/)) 是一个滚动更新的 Linux 发行版，以高度可定制性而著称。它使用 **pacman** 包管理器从官方和用户自定义的仓库安装和更新软件包。

要在 Arch Linux 及其衍生发行版上安装 Podman，请在终端中运行以下命令：

```
# pacman –S podman
```

默认情况下，在 Arch Linux 上安装的 Podman 不支持无根容器。要启用该功能，请按照官方 Arch Wiki 指南操作：[`wiki.archlinux.org/title/Podman#Rootless_Podman`](https://wiki.archlinux.org/title/Podman#Rootless_Podman)。

### 在 Raspberry Pi OS 上安装 Podman

著名的 Raspberry Pi 单板计算机在开发者、创客和爱好者中取得了巨大的成功。

它运行的是 Raspberry Pi OS（[`www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit`](https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit)），该系统基于 Debian。

Podman 的 arm64 构建版本已发布，并且可以通过遵循前述在 Debian 发行版上的安装步骤进行安装。

### 在 macOS 上安装 Podman

使用 Apple 设备的用户可以安装并使用 Podman 作为远程客户端，而容器将在远程的 Linux 主机上执行。该 Linux 主机也可以是一个在 macOS 上执行并直接由 Podman 管理的虚拟机。

要通过 Homebrew 包管理器安装 Podman，请在终端中运行以下命令：

```
$ brew install podman
```

要初始化运行 Linux 虚拟机的环境，请运行以下命令：

```
$ podman machine init
$ podman machine start
```

用户也可以创建并连接到一个外部的 Linux 主机。

在 macOS 上，另一种创建快速、轻量级开发虚拟机的有效方法是 Vagrant。当 Vagrant 虚拟机创建完成后，用户可以手动或自动配置其他软件，如 Podman，并开始使用通过远程客户端配置的定制实例。

### 在 Windows 上安装 Podman

要作为远程客户端运行 Podman，只需从 GitHub 发布页面下载并安装最新版本（[`github.com/containers/podman/releases/`](https://github.com/containers/podman/releases/)）。将归档文件解压到合适的位置，并编辑 TOML 编码的 `containers.conf` 文件，配置远程 URI 以连接到 Linux 主机，或者传递其他选项。

以下代码片段显示了一个示例配置：

```
[engine]
remote_uri= " ssh://root@10.10.1.9:22/run/podman/podman.sock"
```

远程 Linux 主机会通过 `systemd` 单元管理的 UNIX 套接字暴露 Podman。我们将在本书后续章节中详细介绍这个话题。

要在 WSL 2.0 上运行 Podman，用户必须首先从 Microsoft Store 在 Windows 主机上安装一个 Linux 发行版。Microsoft 目录下有多种可用的发行版。

以下示例基于 Ubuntu 20.10：

```
# apt-get –y install podman
# mkdir -p /etc/containers
# echo -e "[registries.search]\nregistries = \
[‘docker.io’, ‘quay.io’]" | tee \ /etc/containers/registries.conf
```

上述命令安装了最新的 Podman 稳定版本，并配置了 `/etc/containers/registries.conf` 文件，以提供一个注册表白名单。

安装完成后，某些小的定制化操作是必需的，以使其适应 WSL 2.0 环境：

```
# cp /usr/share/containers/libpod.conf /etc/containers
# sed –i  ‘s/ cgroup_manager = "systemd"/ cgroup_manager = "cgroupfs"/g’ /etc/containers/libpod.conf
# sed –i ‘s/ events_logger = "journald"/ events_logger = "file"/g’ /etc/containers/libpod.conf
```

上述命令配置了日志记录和 CGroup 管理，以便在子系统中成功运行 rootful 容器。

### 从源代码构建 Podman

从源代码构建应用程序有许多优点：用户可以在构建之前检查和自定义代码，进行跨架构编译，或仅选择构建部分二进制文件。这也是深入了解项目结构和理解其演变的一个很好的学习机会。最后但同样重要的是，从源代码构建可以让用户获得最新的开发版本，带有炫酷的新特性，当然也包括一些 bugs。

以下步骤假设构建机器使用的是 Fedora 发行版。首先，我们需要安装编译 Podman 所需的必要依赖：

```
# dnf install -y \
  btrfs-progs-devel \
  conmon \
  containernetworking-plugins \
  containers-common \
  crun \
  device-mapper-devel \
  git \
  glib2-devel \
  glibc-devel \
  glibc-static \
  go \
  golang-github-cpuguy83-md2man \
  gpgme-devel \
  iptables \
  libassuan-devel \
  libgpg-error-devel \
  libseccomp-devel \
  libselinux-devel \
  make \
  pkgconfig
```

这个命令会花一些时间来安装所有的软件包及其级联的依赖关系。

安装完成后，选择一个工作目录并使用 `git` 命令克隆 Podman 仓库：

```
$ git clone https://github.com/containers/podman.git
```

此命令将在工作目录中克隆整个仓库。

切换到项目目录并开始构建：

```
$ cd podman
$ make package-install
```

`make package-install` 命令编译源代码，构建 RPM 文件，并将包本地安装。请记住，RPM 格式与 Fedora/CentOS/RHEL 发行版相关联，并由 `dnf` 和 `yum` 包管理器管理。

构建过程将需要几分钟才能完成。要测试软件包是否成功安装，只需运行以下代码：

```
$ podman version
Version:      4.0.0-dev
API Version:  4.0.0-dev
Go Version:   go1.16.6
Git Commit:   cffc747fccf38a91be5cd106d2e507afaaa23e14
Built:        Sat Aug  4 00:00:00 2018
OS/Arch:      linux/amd64
```

有时候，将二进制文件在专用的构建主机上构建，再通过包管理器或简单的归档文件将其部署到其他机器上是很有用的。若只需要构建二进制文件，请运行以下命令：

```
$ make
```

构建完成后，二进制文件将位于 `bin/` 文件夹中。要通过将编译后的二进制文件和配置文件直接复制到 Makefile 中定义的目标目录来本地安装，请运行以下命令：

```
$ make install
```

要创建类似于 `.tar.gz` 归档的二进制发布版本（该版本可在 GitHub 发布页面上获取），请运行以下命令：

```
$ make podman-release.tar.gz
```

`git` 命令。例如，要构建 v3.3.1，请使用以下命令：

```
$ git checkout v3.3.1
```

在这一部分中，我们学习了如何使用各自的包管理器在不同的发行版上安装 Podman 的二进制发布版本。我们还学习了如何在 macOS 和 Windows 上安装 Podman 远程客户端，并支持 Windows WSL 2.0 模式。最后，我们通过向你展示如何从源代码构建来结束这一部分。

在下一部分中，我们将学习如何通过准备系统环境来配置 Podman 进行首次运行。

# 准备你的环境

一旦安装了 Podman 包，Podman 就可以*开箱即用*了。然而，一些小的自定义配置可能对提高与外部注册表的互操作性或自定义运行时行为非常有用。

## 自定义容器注册表搜索列表

Podman 会从一系列受信任的容器注册表中搜索并下载镜像。`/etc/containers/registries.conf` 文件是一个 TOML 配置文件，可用于自定义允许搜索和使用作为镜像来源的白名单注册表，以及注册表镜像和未经 TLS 终止的不安全注册表。

在此配置文件中，`unqualified-search-registries` 键填充了一个未指定图像仓库和标签的不合格注册表数组。

在 Fedora 系统上，使用 Podman 的新安装中，此键具有以下内容：

```
unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io", "quay.io"]
```

用户可以向此数组添加或删除注册表，以让 Podman 从中搜索和拉取。

重要提示

在添加注册表时要非常谨慎，并且只使用可信的注册表，以避免拉取包含恶意代码的镜像。

默认列表足以搜索并运行所有的书本示例。那些已经运行私有注册表的用户可以尝试将它们添加到不合格搜索注册表数组中。

由于注册表既可以是私有的也可以是公共的，请记住，通常需要额外的身份验证才能访问私有注册表。可以使用 `podman login` 命令来实现这一点，这将在本书的后续部分介绍。

如果在用户的主目录中找到 `$HOME/.config/containers/registries.conf` 文件，则会覆盖 `/etc/containers/registries.conf` 文件。这样，同一系统上的不同用户将能够使用其自定义注册表白名单和镜像运行 Podman。

## 可选 - 启用基于套接字的服务

这是一个可选步骤，在没有特定需求的情况下，可以安全地跳过本节的内容。

正如前面提到的，Podman 是一种无守护进程的容器管理器，无需后台服务即可运行容器。但是，用户可能需要与 Podman 公开的 Libpod API 进行交互，特别是在从基于 Docker 的环境迁移时。

Podman 可以使用 UNIX 套接字（默认行为）或 TCP 套接字公开其 API。后者选项不太安全，因为它使 Podman 可以从外部访问，但在某些情况下是必需的，例如应由 Windows 或 macOS 工作站上的 Podman 客户端访问时。

重要提示

在将 API 服务在暴露于互联网的机器上使用 TCP 端点时要小心，因为该服务将全局可访问。

以下命令将在 UNIX 套接字上公开 Podman API：

```
$ sudo podman system service --time=0 \
  unix:///run/podman/podman.sock
```

运行此命令后，用户可以连接到 API 服务。

在终端窗口上运行此命令并不是一个方便的方法。而是最好的方法是使用 `man systemd.socket`）。

Socket 单位在 `systemd` 中是一种特殊类型的服务激活器：当请求到达套接字的预定义端点时，`systemd` 立即生成同名的服务。

当安装 Podman 时，会创建 `podman.socket` 和 `podman.service` 单元文件。`podman.socket` 具有以下内容：

```
# cat /usr/lib/systemd/system/podman.socket
[Unit]
Description=Podman API Socket
Documentation=man:podman-system-service(1) 
[Socket]
ListenStream=%t/podman/podman.sock
SocketMode=0660
[Install]
WantedBy=sockets.target
```

`ListenStream` 键保存套接字的相对路径，该路径扩展为 `/run/podman/podman.sock`：

podman.service 包含以下内容：

```
# cat /usr/lib/systemd/system/podman.service
[Unit]
Description=Podman API Service
Requires=podman.socket
After=podman.socket
Documentation=man:podman-system-service(1)
StartLimitIntervalSec=0
[Service]
Type=exec
KillMode=process
Environment=LOGGING="--log-level=info"
ExecStart=/usr/bin/podman $LOGGING system service
[Install]
WantedBy=multi-user.target
```

`ExecStart=` 字段指示由服务启动的命令，这与我们之前展示的 `podman system service` 命令相同。

`Requires=` 字段指示 `podman.service` 单元需要激活 `podman.socket`。

那么，当我们启用并启动 `podman.socket` 单元时会发生什么呢？`systemd` 处理该套接字并等待连接到套接字端点。当此事件发生时，它会立即启动 `podman.service` 单元。经过一段时间的空闲后，服务会再次停止。

要启用并启动套接字单元，请运行以下命令：

```
# systemctl enable --now podman.socket
```

我们可以使用一个简单的 `curl` 命令来测试结果：

```
# curl --unix-socket /run/podman/podman.sock \   
  http://d/v3.0.0/libpod/info
```

打印的输出将是一个 JSON 有效负载，其中包含容器引擎配置。

当我们点击 URL 时发生了什么？在后台，服务单元在发出连接时立即被套接字触发并启动。你们中的一些人可能注意到第一次执行命令时有轻微的延迟（大约 1/10 秒）。

在 5 秒钟的空闲时间后，`podman.service` 会再次停用。这是由于 `podman system service` 命令的默认行为，默认情况下它只运行 5 秒，除非传递 `–time` 选项来提供不同的超时（值为 0 表示永远运行）。

## 可选 – 自定义 Podman 的行为

Podman 的默认配置适用于大多数使用场景，但其配置非常灵活。以下配置文件可用于自定义其行为：

+   `containers.conf`：该 TOML 格式的文件包含 Podman 运行时配置，以及 `conmon` 和容器运行时二进制文件的搜索路径。默认情况下，它安装在 `/usr/share/containers/` 路径下，并可以通过 `/etc/containers/containers.conf` 和 `$HOME/.config/containers/containers.conf` 文件分别覆盖系统范围和用户范围的设置。

该文件可用于自定义引擎的行为。用户可以通过自定义设置（如日志记录、DNS 解析、环境变量、共享内存使用、Cgroup 管理等）来影响容器的创建及其生命周期。

有关完整设置列表，请查看随 Podman 包一起安装的相关 `man` 页面。(`man containers.conf`)

+   `storage.conf`：该 TOML 格式的文件用于自定义容器引擎使用的存储设置。特别地，该文件使你能够自定义默认的存储驱动程序以及容器存储的读/写目录（也称为图形根目录），这也是一个额外的驱动程序存储选项。默认情况下，驱动程序设置为 **overlay**。

该文件的默认路径是`/usr/share/containers/storage.conf`，可以在`/etc/containers/storage.conf`下找到或创建覆盖文件，用于系统范围的自定义配置。

影响无根容器的用户范围配置可以在`$XDG_CONFIG_HOME/containers/storage.conf`或`$HOME/.config/containers/storage.conf`下找到。

+   `mounts.conf`：此文件定义了在容器启动时应自动挂载的卷。这在自动传递容器内的密钥和证书等机密信息时非常有用。

它可以在`/usr/share/containers/mounts.conf`找到，并可以通过位于`/etc/containers/mounts.conf`的文件进行覆盖。

在无根模式下，覆盖文件可以放置在`$HOME/.config/containers/mounts.conf`下。

+   `seccomp.json`：这是一个 JSON 文件，允许用户自定义容器内进程可以执行的`syscalls`，并同时定义被阻止的`syscalls`。这个话题将在*第十一章*《容器安全》中再次讨论，届时会提供更深入的容器安全约束的理解。

该文件的默认路径是`/usr/share/containers/seccomp.json`。seccomp 手册页（`man seccomp`）提供了有关 seccomp 如何在 Linux 系统上工作的概述。

+   `policy.json`：这是一个 JSON 文件，用于定义 Podman 如何执行签名验证。该文件的默认路径是`/etc/containers/policy.json`，并可以通过用户范围的`$HOME/.config/containers/policy.json`进行覆盖。

该配置文件接受三种类型的策略：

+   **insecureAcceptAnything**：接受来自指定注册表的任何镜像。

+   **reject**：拒绝来自指定注册表的任何镜像。

+   **signedBy**：仅接受由特定已知实体签名的镜像。

默认配置是接受每个镜像（`insecureAcceptAnything`策略），但可以修改为只拉取可以通过签名验证的受信镜像。用户可以定义自定义 GPG 密钥来验证签名及其签名者的身份。有关可能的策略和配置示例的更多细节，请参阅相关的手册页（`man containers-policy.json`）。

在本节中，我们讨论了一些在 Podman 首次安装时需要了解的基本配置。下一节将介绍我们的第一个容器执行示例。

# 运行第一个容器

现在，终于可以运行我们的第一个容器了。

在上一节中，我们揭示了如何在我们喜欢的 Linux 发行版上安装 Podman，以及安装后包含的基本软件包内容。现在，我们可以开始使用我们的无守护进程容器引擎了。

在 Podman 中运行容器是通过`podman run`命令来处理的，该命令接受许多选项来控制刚刚运行的容器的行为、隔离性、通信、存储等。

运行全新容器的最简单和最短的 Podman 命令如下：

```
$ podman run <imageID>
```

我们必须将 `imageID` 字符串替换为我们想要运行的镜像名称/位置/标签。如果该镜像不在缓存中，或者我们之前没有下载过，Podman 将从相应的容器注册表为我们拉取镜像。

## 交互式和伪终端（pseudo-tty）

为了介绍这个命令及其选项，我们先从简单开始，执行以下命令：

```
$ podman run -i -t fedora /bin/bash 
Resolved "fedora" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull registry.fedoraproject.org/fedora:latest...
Getting image source signatures
Copying blob ecfb9899f4ce done  
Copying config 37e5619f4a done  
Writing manifest to image destination
Storing signatures
[root@ec444ad299ab /]#
```

让我们看看在执行上一个命令后，Podman 做了什么：

1.  它识别出镜像名称 `fedora` 作为最新 Fedora 容器镜像的别名。

1.  它随后意识到，由于这是第一次尝试运行该镜像，镜像在本地缓存中是缺失的。

1.  它从正确的注册表中拉取了镜像。它选择了 Fedora 项目注册表，因为它与注册表配置中包含的别名匹配。

1.  最终，它启动了容器，并为我们提供了一个交互式 shell，执行了我们请求的 Bash shell 程序。

上一个命令启动了一个交互式 shell，这得益于我们可以分析的两个选项，如下所示：

+   `--tty, -t`：使用此选项，Podman 会为容器分配一个 `man pty` 并将其附加到容器的标准输入。

+   `--interactive, -i`：使用此选项，Podman 会保持 `stdin` 打开，并准备附加到之前的伪终端。

如前几章所述，当创建容器时，容器内部的隔离进程将在可写的根文件系统上运行，这是层叠叠加的结果。

这允许任何进程写入文件，但不要忘记，这些文件会一直存在，直到容器停止运行，因为容器默认是短暂的。

现在，你可以执行任何命令并检查我们刚刚启动的控制台中的输出：

```
[root@ec444ad299ab /]# dnf install -y iputils iproute
Last metadata expiration check: 0:01:50 ago on Mon Sep 13 08:54:20 2021.
Dependencies resolved.
=============================================================================================================================================================================
Package                                           Architecture                      Version                                        Repository                          Size
=============================================================================================================================================================================
Installing:
iproute                                           x86_64                            5.10.0-2.fc34                                  fedora                             679 k
iputils                                           x86_64                            20210202-2.fc34                                fedora                             170 k
Installing dependencies:
...
[root@ec444ad299ab /]# ip r
default via 10.0.2.2 dev tap0 
10.0.2.0/24 dev tap0 proto kernel scope link src 10.0.2.100 
[root@ec444ad299ab /]# ping -c2 10.0.2.2
PING 10.0.2.2 (10.0.2.2) 56(84) bytes of data.
64 bytes from 10.0.2.2: icmp_seq=1 ttl=255 time=0.030 ms
64 bytes from 10.0.2.2: icmp_seq=2 ttl=255 time=0.200 ms

--- 10.0.2.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1034ms
rtt min/avg/max/mdev = 0.030/0.115/0.200/0.085 ms
```

如你所见，在上面的示例中，我们安装了两个软件包，用于检查容器的网络配置，然后执行 ping 测试，目标是分配给我们运行容器的虚拟网络的默认路由器。同样，如果我们停止该容器，任何更改都将丢失。

要退出这个交互式 shell，我们只需要按下*Ctrl* + *D* 或执行 `exit` 命令。这样，容器将被终止，因为我们请求执行的主进程（`/bin/bash`）将停止！

现在，让我们看看一些我们可以与 `podman run` 命令一起使用的其他有用选项。

## 从运行中的容器中分离

正如我们之前学到的，Podman 让我们有机会将交互式 shell 附加到正在运行的容器中。然而，我们很快会发现，这并不是运行容器的首选方式。

一旦容器启动，我们可以轻松地从容器中分离，即使我们是用附加交互式 `tty` 启动的：

```
$ podman run -i -t registry.fedoraproject.org/f29/httpd
Trying to pull registry.fedoraproject.org/f29/httpd:latest...
Getting image source signatures
Copying blob aaf5ad2e1aa3 done  
Copying blob 7692efc5f81c done  
Copying blob d77ff9f653ce done  
Copying config 25c76f9dcd done  
Writing manifest to image destination
Storing signatures
=> sourcing 10-set-mpm.sh ...
=> sourcing 20-copy-config.sh ...
=> sourcing 40-ssl-certs.sh ...
AH00558: httpd: Could not reliably determine the server’s fully qualified domain name, using 10.0.2.100\. Set the ‘ServerName’ directive globally to suppress this message
[Tue Sep 14 09:26:05.691906 2021] [ssl:warn] [pid 1:tid 140416655523200] AH01882: Init: this version of mod_ssl was compiled against a newer library (OpenSSL 1.1.1b FIPS  26 Feb 2019, version currently loaded is OpenSSL 1.1.1 FIPS  11 Sep 2018) - may result in undefined or erroneous behavior
[Tue Sep 14 09:26:05.692610 2021] [ssl:warn] [pid 1:tid 140416655523200] AH01909: 10.0.2.100:8443:0 server certificate does NOT include an ID which matches the server name
AH00558: httpd: Could not reliably determine the server’s fully qualified domain name, using 10.0.2.100\. Set the ‘ServerName’ directive globally to suppress this message
[Tue Sep 14 09:26:05.752028 2021] [ssl:warn] [pid 1:tid 140416655523200] AH01882: Init: this version of mod_ssl was compiled against a newer library (OpenSSL 1.1.1b FIPS  26 Feb 2019, version currently loaded is OpenSSL 1.1.1 FIPS  11 Sep 2018) - may result in undefined or erroneous behavior
[Tue Sep 14 09:26:05.752806 2021] [ssl:warn] [pid 1:tid 140416655523200] AH01909: 10.0.2.100:8443:0 server certificate does NOT include an ID which matches the server name
[Tue Sep 14 09:26:05.752933 2021] [lbmethod_heartbeat:notice] [pid 1:tid 140416655523200] AH02282: No slotmem from mod_heartmonitor
[Tue Sep 14 09:26:05.755334 2021] [mpm_event:notice] [pid 1:tid 140416655523200] AH00489: Apache/2.4.39 (Fedora) OpenSSL/1.1.1 configured -- resuming normal operations
[Tue Sep 14 09:26:05.755346 2021] [core:notice] [pid 1:tid 140416655523200] AH00094: Command line: ‘httpd -D FOREGROUND’
```

现在怎么办？要从运行中的容器中分离，我们只需按下以下特殊快捷键：*Ctrl* + *P*，*Ctrl* + *Q*。通过这个组合，我们将返回到我们的 shell 提示符，同时容器会继续运行。

为了恢复我们分离的容器的`tty`，我们必须获取正在运行的容器列表：

```
$ podman ps
CONTAINER ID  IMAGE                                        COMMAND               CREATED        STATUS            PORTS       NAMES
685a339917e7  registry.fedoraproject.org/f29/httpd:latest  /usr/bin/run-http...  3 minutes ago  Up 3 minutes ago              clever_zhukovsky
```

我们将在下一章中更详细地探讨此命令，但目前，只需记下`容器 ID`，然后执行以下命令以重新连接到之前的`tty`：

```
$ podman attach 685a339917e7
```

请注意，我们只需将`-d`选项添加到`podman run`命令中，就可以轻松启动一个容器并进入*分离*模式，如下所示：

```
$ podman run -d -i -t registry.fedoraproject.org/f29/httpd
```

在下一节中，我们将学习如何在特殊情况下使用分离选项。

## 网络端口发布

正如我们在前几章中提到的，Podman 像其他容器引擎一样，在容器运行时会附加一个虚拟网络，这个网络与原主机网络是隔离的。因此，如果我们想要轻松访问容器，甚至将它暴露到主机网络外部，我们需要指示 Podman 进行端口映射。

Podman 的`-p`选项将容器的端口发布到主机：

```
-p=ip:hostPort:containerPort
```

`hostPort`和`containerPort`都可以是端口范围，如果主机 IP 未设置或设置为`0.0.0.0`，则端口将绑定到主机的所有 IP 地址。

如果我们回顾上一节使用的命令，它变成了以下内容：

```
$ podman run -p 8080:8080 -d -i -t \ registry.fedoraproject.org/f29/httpd
```

现在，我们可以记录下分配给我们正在运行的容器的`容器 ID`：

```
$ podman ps
CONTAINER ID  IMAGE                                        COMMAND               CREATED         STATUS             PORTS                   NAMES
fc9d97642801  registry.fedoraproject.org/f29/httpd:latest  /usr/bin/run-http...  10 minutes ago  Up 10 minutes ago  0.0.0.0:8080->8080/tcp  confident_snyder
```

然后，我们可以查看我们刚才定义的端口映射：

```
$ podman port fc9d97642801
8080/tcp -> 0.0.0.0:8080
```

接下来，我们可以使用`curl`命令测试端口映射是否有效，`curl`是一个易于使用的 HTTP 客户端。或者，您也可以使用您喜欢的网页浏览器访问相同的 URL，如下所示：

```
$ curl –s 127.0.0.1:8080 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
4
6<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
5    <head>
0         <title>Test Page for the Apache HTTP Server on Fedora</title>
          <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
          <style type="text/css">
1               /*<![CDATA*/
0               body {
0                     background-color: #fff;
```

在本章结束前，让我们看一下其他一些有趣的选项，这些选项可能在运行时管理配置和容器行为时非常有用。

## 配置和环境变量

`podman run`命令提供了大量的选项，允许我们在运行时配置容器的行为——在撰写本书时，大约有 120 个选项。

例如，我们有一个选项可以更改正在运行的容器的时区，也就是`--tz`：

```
$ date
Tue Sep 14 17:44:59 CEST 2021
$ podman run --tz=Asia/Shanghai fedora date
Tue Sep 14 23:45:11 CST 2021
```

我们可以通过`--dns`选项更改我们新容器的 DNS：

```
$ podman run --dns=1.1.1.1 fedora cat /etc/resolv.conf
search lan
nameserver 1.1.1.1
```

我们还可以将一个主机添加到`/etc/hosts`文件中，以覆盖本地内部地址：

```
$ podman run --add-host=my.server.local:192.168.1.10 \
fedora cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.10 my.server.local
```

我们甚至可以添加一个 HTTP 代理，让容器使用代理进行 HTTP 请求。Podman 的默认行为是从主机传递许多环境变量，其中一些是`http_proxy`、`https_proxy`、`ftp_proxy`和`no_proxy`。

另一方面，我们也可以定义自定义的环境变量，借助`–env`选项将它们传递给容器：

```
$ podman run --env MYENV=podman fedora printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
container=oci
DISTTAG=f34container
FGC=f34
MYENV=podman
HOME=/root
HOSTNAME=93f2541180d2
```

使用环境变量并与我们的容器一起使用是传递配置参数给应用程序并从操作系统主机影响服务行为的最佳实践。正如我们在[*第一章*中看到的，*容器技术简介*，容器默认是不可变且短暂的。因此，出于这个原因，我们应该像前面的示例中那样利用环境变量，在运行时配置容器。

# 总结

在本章中，我们开始玩转 Podman 的基本命令，我们通过查看最有趣的选项学习了如何运行容器，现在我们准备进入下一个层次：容器管理。要在容器世界中作为系统管理员工作，我们必须理解并学习那些让我们检查和检查正在运行的容器化服务健康状况的管理命令；这正是我们在本章中看到的内容。

在下一章中，我们将深入学习容器管理，我们将学习如何使用 Podman 管理镜像和容器生命周期。我们将学习如何检查和提取正在运行的容器的日志，并且还将介绍 Pod，如何创建它们，以及如何在其中运行容器。

# 深入阅读

关于本章中涉及的主题的更多信息，请参考以下资源：

+   在 MacOS 上安装 Podman：[`podman.io/blogs/2021/09/06/podman-on-macs.html`](https://podman.io/blogs/2021/09/06/podman-on-macs.html)

+   在 Windows 上安装 Podman：[`www.redhat.com/sysadmin/podman-windows-wsl2`](https://www.redhat.com/sysadmin/podman-windows-wsl2)

+   管理容器注册表：[`www.redhat.com/sysadmin/manage-container-registries`](https://www.redhat.com/sysadmin/manage-container-registries)

+   Podman API 文档：[`docs.podman.io/en/latest/_static/api.html`](https://docs.podman.io/en/latest/_static/api.html)

+   systemd socket 手册：[`www.freedesktop.org/software/systemd/man/systemd.socket.html`](https://www.freedesktop.org/software/systemd/man/systemd.socket.html)

+   Podman 和 seccomp 配置文件：[`podman.io/blogs/2019/10/15/generate-seccomp-profiles.html`](https://podman.io/blogs/2019/10/15/generate-seccomp-profiles.html)
