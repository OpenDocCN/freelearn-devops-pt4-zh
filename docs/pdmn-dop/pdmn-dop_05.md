# *第四章*：管理运行中的容器

在前一章中，我们学习了如何设置环境以便使用 Podman 运行容器，包括主要发行版的二进制安装、系统配置文件以及第一个示例容器的运行，以验证我们的设置是否正确。本章将提供容器执行的更详细概述，介绍如何管理和检查正在运行的容器，以及如何将容器分组到 Pod 中。本章对于获得正确的知识和技能，以便开始作为容器技术的系统管理员经验非常重要。

在本章中，我们将涵盖以下主要内容：

+   管理容器镜像

+   与运行中的容器进行操作

+   检查容器信息

+   捕获容器日志

+   在运行的容器中执行进程

+   在 Pod 中运行容器

# 技术要求

在继续本章及其练习之前，需要一台运行 Podman 实例的机器。如*第三章*所述，*运行第一个容器*，本书中的所有示例都在 Fedora 34 系统上执行，但也可以在您选择的**操作系统**（**OS**）上复制。

最后，充分理解前几章中涵盖的主题有助于轻松掌握有关**开放容器倡议**（**OCI**）镜像和容器执行的概念。

# 管理容器镜像

在本节中，我们将看到如何在本地系统中查找并拉取（下载）镜像，以及如何检查其内容。当容器第一次创建并运行时，Podman 会自动拉取相关镜像。然而，能够提前拉取和检查镜像具有一些宝贵的优势，第一个优势是，当镜像已经在机器的本地存储中时，容器执行速度更快。

如前几章所述，容器是一种将进程隔离在具有独立命名空间和资源分配的沙箱环境中的方式。

容器中挂载的文件系统由在*第二章*中描述的 OCI 镜像提供，*比较 Podman 和 Docker*。

OCI 镜像由专门的服务称为**容器镜像仓库**进行存储和分发。容器镜像仓库存储镜像和元数据，并暴露简单的**表述性状态转移**（**REST**）**应用程序编程接口**（**API**）服务，以便用户推送和拉取镜像。

基本上有两种类型的镜像仓库：公共和私有。公共仓库作为公共服务可以访问（有或没有认证）。主要的公共仓库，如`docker.io`、`gcr.io`或`quay.io`，也被用作大型开源项目的镜像库。

私有注册表是在组织内部部署和管理的，可以更专注于安全性和内容过滤。目前的主要容器注册表项目已在**云原生计算基金会**（**CNCF**）下毕业（[`landscape.cncf.io/card-mode?category=container-registry&grouping=category`](https://landscape.cncf.io/card-mode?category=container-registry&grouping=category)），并提供管理多租户、身份验证、**基于角色的访问控制**（**RBAC**）等高级企业功能，以及图像漏洞扫描和图像签名。

在*第九章*中，*推送图像到容器注册表*，我们将提供更多关于与容器注册表交互的细节和示例。

大部分公共和私有注册表暴露 Docker Registry HTTP API V2（[`docs.docker.com/registry/spec/api/`](https://docs.docker.com/registry/spec/api/)）。使用`curl`命令或设计自己的定制客户端。

Podman 提供了一个**命令行界面**（**CLI**），用于与公共和私有容器注册表进行交互，管理在需要注册表身份验证时的登录，按字符串模式搜索图像仓库，以及处理本地缓存的图像。

## 搜索图像

我们将学习的第一个用于在多个注册表中搜索图像的命令是`podman search`命令。以下示例展示了如何搜索 nginx 图像：

```
# podman search nginx 
```

上述命令将生成一个输出，其中包含来自所有白名单注册表的多个条目（参见*第三章*，*运行第一个容器*部分，*准备你的环境* | *自定义容器注册表的搜索列表*）。输出会有点笨重，包含来自未知和不可靠仓库的多个条目。

一般来说，`podman search`命令接受以下模式：

```
podman search [options] TERM
```

在这里，`TERM`是搜索参数。搜索结果的输出包含以下字段：

+   `INDEX`：索引该图像的注册表

+   `NAME`：图像的完整名称，包括注册表名称和相关的命名空间

+   `DESCRIPTION`：图像角色的简短描述

+   `STARS`：用户评分的星级数（仅在支持此功能的注册表中可用，如`docker.io`）

+   `OFFICIAL`：一个布尔值，用于指定图像是否为官方图像

+   `AUTOMATED`：如果图像是自动化的，该字段设置为`OK`

    重要说明

    永远不要信任未知的仓库，总是优先选择官方图像。拉取来自小众项目的图像时，尝试在运行之前了解图像的内容。记住，攻击者可能会在容器内部隐藏恶意代码。

    即使是受信任的仓库，也有可能在某些情况下被攻破。在企业场景中，实施图像签名验证以避免图像篡改。

可以对搜索应用过滤器并细化输出。例如，为了精确搜索并仅打印官方镜像，我们可以添加以下过滤选项，仅打印带有`is-official`标志的镜像：

```
# podman search nginx --filter=is-official
```

此命令将打印一行指向`docker.io/library/nginx:latest`。这个官方镜像由 nginx 社区维护，可以更放心地使用。

用户可以调整命令的输出格式。以下示例演示了如何仅打印镜像注册表和镜像名称：

```
# podman search fedora  \
  --filter is-official \
  --format "table {{.Index}} {{.Name}}"

INDEX       NAME
docker.io   docker.io/library/fedora
```

输出的镜像名称有一个标准的命名模式，值得详细描述。标准格式如下所示：

```
<registry>[:<port>]/[<namespace>/]<name>:<tag>
```

让我们详细描述前面提到的字段，如下所示：

+   `registry`：这包含存储镜像的注册表。在我们的示例中，nginx 镜像存储在`docker.io`公共注册表中。可选地，也可以为注册表指定自定义端口号。默认情况下，注册表会暴露`5000` **传输控制协议** (**TCP**) 端口。

+   `namespace`：此字段提供了一个层次结构，有助于区分镜像的上下文和提供者。命名空间可以代表父组织、仓库所有者的用户名或镜像角色。

+   `name`：这包含存储所有标签的私有/公共镜像仓库的名称。通常将其称为应用程序名称（即 nginx）。

+   `tag`：每个存储在注册表中的镜像都有一个唯一的标签，映射到`:latest`标签时，镜像名称中可以省略该标签。

通用搜索默认隐藏镜像标签。要显示给定仓库的所有可用标签，我们可以对给定的镜像名称使用`–list-tags`选项，如下所示：

```
# podman search quay.io/prometheus/prometheus --list-tags
NAME                           TAGquay.io/prometheus/prometheus  v2.5.0
quay.io/prometheus/prometheus  v2.6.0-rc.0
quay.io/prometheus/prometheus  v2.6.0-rc.1
quay.io/prometheus/prometheus  v2.6.0
quay.io/prometheus/prometheus  v2.6.1
quay.io/prometheus/prometheus  v2.7.0-rc.0
quay.io/prometheus/prometheus  v2.7.0-rc.1
quay.io/prometheus/prometheus  v2.7.0-rc.2
quay.io/prometheus/prometheus  v2.7.0
quay.io/prometheus/prometheus  v2.7.1
[...output omitted...]
```

这个选项对于在注册表中查找特定镜像标签非常有用，通常与应用程序/运行时的发布版本相关联。

重要提示

使用`:latest`标签可能会导致镜像版本控制问题，因为它不是描述性的标签。而且，通常期望它指向最新的镜像版本。不幸的是，这并不总是正确的，因为一个未标记的镜像可能会保留`latest`标签，而最新推送的镜像可能具有不同的标签。是否正确应用标签由仓库维护者决定。如果仓库使用语义版本控制，最好的选择是拉取最新版本标签。

## 拉取和查看镜像

一旦我们找到了所需的镜像，就可以使用`podman pull`命令下载，如下所示：

```
# podman pull docker.io/library/nginx:latest
```

注意运行 Podman 命令的根用户。在这种情况下，我们作为 root 用户拉取镜像，其层和元数据存储在`/var/lib/containers/storage`路径中。

我们可以通过在标准用户的 shell 中执行相同的命令，以标准用户身份运行该命令，如下所示：

```
$ podman pull docker.io/library/nginx:latest
```

在这种情况下，镜像将被下载到用户的主目录下，路径为`$HOME/.local/share/containers/storage/`，并可用于运行无根容器。

用户可以使用`podman images`命令检查所有本地缓存的镜像，如此处所示：

```
# podman images
REPOSITORY                  TAG         IMAGE ID      CREATED        SIZE
docker.io/library/nginx     latest      ad4c705f24d3  2 weeks ago    138 MB
docker.io/library/fedora    latest      dce66322d647  2 months ago   184 MB
[...omitted output...]
```

输出显示了镜像的仓库名称、标签、镜像**标识符**（**ID**）、创建日期和镜像大小。这对于保持本地存储中的镜像的最新视图，并了解哪些镜像已过时非常有用。

`podman images`命令还支持许多选项（可以通过执行`man podman-images`命令获取完整列表）。其中一个更有趣的选项是`–sort`，它可以用于按大小、日期、ID、仓库或标签对镜像进行排序。例如，我们可以按创建日期排序，找出最过时的镜像，如下所示：

```
# podman images --sort=created
```

另一个非常有用的选项是`–all`（或`–a`）和`–quiet`（或`–q`）选项。它们可以组合使用，打印出所有本地存储的镜像的镜像 ID，包括中间层镜像。该命令的输出将类似于以下示例：

```
# podman images -qa
ad4c705f24d3
a56f85702a94
b5c5125e3fee
4d7fc5917f3e
625707533167
f881f1aa4d65
96ab2a326180
```

在系统中列出并显示已经拉取的镜像并不是最有趣的部分！让我们在下一节中探讨如何检查镜像的配置和内容。

## 检查镜像的配置和内容

要检查拉取镜像的配置，`podman image inspect`（或更短的`podman inspect`）命令可以帮助我们，如此处所示：

```
# podman inspect docker.io/library/nginx:latest
```

打印的输出将是一个**JavaScript 对象表示法**（**JSON**）格式的对象，包含镜像配置、架构、层、标签、注释和镜像构建历史。

图像历史显示了每一层的创建历史，对于在没有 Dockerfile 或 Containerfile 的情况下，理解图像是如何构建的非常有用。

由于输出是一个 JSON 对象，我们可以提取单个字段来收集特定数据，或者将它们用作其他命令的输入参数。

以下示例打印出创建容器时执行的命令：

```
# podman inspect docker.io/library/nginx:latest \
--format "{{ .Config.Cmd }}"
[nginx -g daemon off;]
```

请注意，格式化输出是通过 Go 模板进行管理的。

有时，镜像的检查必须超越简单的配置检查。在某些情况下，我们需要检查镜像的文件系统内容。为了实现这一点，Podman 提供了有用的`podman image mount`命令。

以下示例挂载该镜像并打印其挂载路径：

```
# podman image mount docker.io/library/nginx
/var/lib/containers/storage/overlay/ba9d21492c3939befbecd5ec32f6f1b9d564ccf8b1b279e0fb5c186e8b7 967f2/merged 
```

如果我们在提供的路径中运行简单的`ls`命令，我们将看到由其各种合并层组成的镜像文件系统，如下所示：

```
# ls -al /var/lib/containers/storage/overlay/ba9d21492c3939befbecd5ec32f6f1b9d564ccf8b1b279e0fb5c186e8b7 967f2/merged
total 92
dr-xr-xr-x. 1 root root 4096 Sep 25 22:30 .
drwx------. 5 root root 4096 Sep 25 22:53 ..
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 bin
drwxr-xr-x. 2 root root 4096 Jun 13 12:30 boot
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 dev
drwxr-xr-x. 1 root root 4096 Sep  9 20:26 docker-entrypoint.d
-rwxrwxr-x. 1 root root 1202 Sep  9 20:25 docker-entrypoint.sh
drwxr-xr-x. 1 root root 4096 Sep  9 20:26 etc
drwxr-xr-x. 2 root root 4096 Jun 13 12:30 home
drwxr-xr-x. 1 root root 4096 Sep  9 20:26 lib
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 lib64
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 media
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 mnt
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 opt
drwxr-xr-x. 2 root root 4096 Jun 13 12:30 proc
drwx------. 2 root root 4096 Sep  2 02:00 root
drwxr-xr-x. 3 root root 4096 Sep  2 02:00 run
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 sbin
drwxr-xr-x. 2 root root 4096 Sep  2 02:00 srv
drwxr-xr-x. 2 root root 4096 Jun 13 12:30 sys
drwxrwxrwt. 1 root root 4096 Sep  9 20:26 tmp
drwxr-xr-x. 1 root root 4096 Sep  2 02:00 usr
drwxr-xr-x. 1 root root 4096 Sep  2 02:00 var
```

要卸载镜像，只需运行`podman image unmount`命令，如下所示：

```
# podman image unmount docker.io/library/nginx
```

以 rootless 模式挂载镜像有点不同，因为此执行模式仅支持手动挂载，`mount`/`unmount`命令将无法工作。一个解决方法是先运行`podman unshare`命令，它将在一个新的命名空间中执行一个新的 shell 进程，其中当前的`podman mount`命令可以正常工作。让我们在这里看一个示例：

```
$ podman unshare
# podman image mount docker.io/library/nginx:latest \
/home/<username>/.local/share/containers/storage/overlay/ba9d21492c3939befbecd5ec32f6f1b9d564ccf8b1b279e0fb5c186e8b7967 f2/merged
```

请注意，挂载点现在位于`<username>`的主目录中。

要卸载，只需运行 podman `unmount`命令，如下所示：

```
# podman image unmount docker.io/library/nginx:latest
ad4c705f24d392b982b2f0747704b1c5162e45674294d5640cca7076eba2 865d
# exit
```

`exit`命令用于退出临时的未共享命名空间。

## 删除镜像

要删除本地存储的镜像，我们可以使用`podman rmi`命令。以下示例删除之前拉取的 nginx 镜像：

```
# podman rmi docker.io/library/nginx:latest
Untagged: docker.io/library/nginx:latest
Deleted: ad4c705f24d392b982b2f0747704b1c5162e45674294d5640cca7 076eba2865d
```

同样的命令在 rootless 模式下也有效，当标准用户在其主目录本地存储中执行时。

要删除所有缓存的镜像，请使用以下示例，该示例依赖于 Shell 命令扩展来获取完整的镜像 ID 列表：

```
# podman rmi $(podman images -qa)
```

注意行首的井号符号，这告诉我们命令是以 root 身份执行的。

下一个命令删除常规用户本地缓存中的所有镜像（注意行首的美元符号）：

```
$ podman rmi $(podman images -qa)
```

重要提示

`podman rmi`命令无法删除当前在运行容器中使用的镜像。首先，停止使用被阻塞镜像的容器，然后再运行该命令。

Podman 还提供了一种更简单的方式来清理悬挂或未使用的镜像——`podman image prune`命令。它不会删除正在使用中的容器的镜像，因此如果你有正在运行或已停止的容器，对应的容器镜像将不会被删除。

以下示例删除所有未使用的镜像，且无需确认：

```
$ sudo podman image prune -af
```

相同的命令适用于 rootless 模式，只删除用户主目录本地存储中的镜像，如下代码片段所示：

```
$ podman image prune -af
```

通过这个，我们已经学习了如何管理我们机器上的容器镜像。接下来，让我们学习如何处理和检查正在运行的容器。

# 与正在运行的容器的操作

在*第二章*，*比较 Podman 和 Docker*中，我们在*运行你的第一个容器*部分学习了如何通过基本示例运行容器，涉及在 Fedora 容器内执行 Bash 进程和`httpd`服务器，这对于学习如何将容器暴露到外部也很有帮助。

接下来，我们将探索一组用于监控和检查正在运行的容器的命令，并获得有关它们行为的洞察。

## 查看和处理容器状态

让我们先运行一个简单的容器，并将其暴露在`8080`端口，使其能够从外部访问，如下所示：

```
$ podman run -d -p 8080:80 docker.io/library/nginx
```

上述示例在 rootless 模式下运行，但相同的操作也可以作为 root 用户应用，只需在命令前添加`sudo`。在这种情况下，实际上不需要以这种方式执行容器。

重要提示

Rootless 容器提供了额外的安全优势。如果恶意进程突破了容器的隔离，可能利用主机上的漏洞，那么它最多会获得启动 rootless 容器的用户的权限。

现在我们的容器已经启动并运行，并准备好提供服务，我们可以通过在本地主机上运行`curl`命令来测试它，应该会产生如下的**超文本标记语言** (**HTML**) 默认输出：

```
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and 
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

显然，一个没有内容可提供的空 nginx 服务器是没有用的，但我们将在接下来的章节中学习如何通过使用卷或构建自定义镜像来提供自定义内容。

我们可以使用的第一个命令来检查容器状态的是`podman ps`。这个命令简单地打印出运行中容器的有用信息，并且可以自定义和排序输出。让我们在主机上运行此命令并查看输出，如下所示：

```
$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS                 NAMES
d8bbd5da64d0  docker.io/library/nginx:latest  nginx -g daemon o...  13 minutes ago  Up 13 minutes ago  0.0.0.0:8080->80/tcp  unruffled_saha
```

输出提供了一些关于正在运行容器的有趣信息，具体如下：

+   `CONTAINER ID`：每个新容器都会获得一个唯一的十六进制 ID。完整 ID 的长度为 64 个字符，`podman ps`输出中显示的是其中 12 个字符的简短版本。

+   `IMAGE`：容器使用的镜像。

+   `COMMAND`：容器内执行的命令。

+   `CREATED`：容器的创建日期。

+   `STATUS`：当前容器的状态。

+   `PORTS`：容器中打开的网络端口。当应用端口映射时，我们可以看到一个或多个主机`ip:port`对映射到容器端口，并带有箭头符号。例如，`0.0.0.0:8080->80/tcp`表示主机端口`8080/tcp`暴露在所有监听接口上，并映射到容器端口`80/tcp`。

+   `NAMES`：容器名称。这个名称可以由用户指定，也可以由容器引擎随机生成。

    提示

    注意输出最后一列中的随机生成名称。Podman 延续了 Docker 的*传统*，使用名称左侧的形容词和右侧的著名科学家或黑客来生成随机名称。实际上，Podman 仍然使用相同的`github.com/docker/docker/namesgenerator` Docker 包，该包包含在项目的 vendor 目录中。

要获取正在运行和已停止容器的完整列表，我们可以在命令中添加`–a`选项。为了演示这一点，我们首先介绍`podman stop`命令。该命令将容器状态更改为停止状态，并向容器内运行的进程发送`SIGTERM`信号。如果容器没有响应，则会在 10 秒的超时后发送`SIGKILL`信号。

让我们尝试停止之前的容器，并通过执行以下代码检查其状态：

```
$ podman stop d8bbd5da64d0  
$ podman ps
```

这次，`podman ps`输出为空。这是因为容器的状态是停止的。要获取正在运行和已停止容器的完整列表，请运行以下命令：

```
$ podman ps –a
CONTAINER ID  IMAGE  COMMAND  CREATED   STATUS   PORT   NAMES
d8bbd5da64d0  docker.io/library/nginx:latest  nginx -g daemon o...  About a minute ago  Exited (0) About a minute ago  0.0.0.0:8080->80/tcp  unruffled_saha
```

注意容器的状态，它显示容器已退出，并且退出代码为`0`。

停止的容器可以通过运行`podman start`命令恢复，如下所示：

```
$ podman start d8bbd5da64d0  
```

这个命令只是重新启动我们之前停止的容器。

如果我们现在再次检查容器状态，我们会看到它已经启动并正在运行，如下所示：

```
$ podman ps
CONTAINER ID  IMAGE  COMMAND  CREATED   STATUS   PORT   NAMES
d8bbd5da64d0  docker.io/library/nginx:latest  nginx -g daemon o...  8 minutes ago  Up 1 second ago  0.0.0.0:8080->80/tcp  unruffled_saha
```

Podman 在容器处于停止状态时会保留容器配置、存储和元数据。无论如何，当我们恢复容器时，容器内部会启动一个新进程。

更多选项，请参见相关的`man podman-start`。

如果我们只需要重启一个正在运行的容器，可以使用`podman restart`命令，如下所示：

```
$ podman restart <Container_ID_or_Name>
```

这个命令的作用是立即重启容器内的进程，并赋予其新的**进程 ID**（**PID**）。

`podman start`命令还可以用于启动之前创建但未运行的容器。要创建一个容器而不启动它，可以使用`podman create`命令。以下示例创建一个容器，但不启动它：

```
$ podman create -p 8080:80 docker.io/library/nginx
```

要启动它，请运行`podman start`命令，后跟已创建容器的 ID 或名称，如下所示：

```
$ podman start <Container_ID_or_Name>
```

这个命令在准备一个不运行的环境或挂载容器文件系统时非常有用，如下面的例子所示：

```
$ podman unshare
$ podman container mount <Container_ID_or_Name>
/home/<username>/.local/share/containers/storage/overlay/bf9d8df299436d80dece200a23e1b8b957f987a254a656ef94cdc5666982 3b5c/merged
```

现在让我们介绍一个非常常用的命令：`podman rm`。顾名思义，它用于从主机中删除容器。默认情况下，它会删除已停止的容器，但可以通过`–f`选项强制删除正在运行的容器。

使用之前示例中的容器，如果我们再次停止它并发出`podman rm`命令，如下面的代码片段所示，所有容器的存储、配置和元数据都会被丢弃：

```
$ podman stop d8bbd5da64d0
$ podman rm d8bbd5da64d0
```

如果我们现在再次运行`podman ps`命令，即使使用了`–a`选项，我们将获得一个空列表，如下所示：

```
$ podman ps –a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

欲了解更多详细信息，请查看命令手册页（`man podman-rm`）。

有时，这就像处理镜像一样，使用`–q`选项仅打印容器 ID 是很有用的。这个选项与`–a`选项结合使用时，可以打印主机上所有已停止和正在运行的容器列表。让我们在这里尝试另一个例子：

```
$ for i in {1..5}; do podman run -d docker.io/library/nginx; done
```

有趣的是，我们已经使用了一个 shell 循环来启动五个相同的容器，这次没有任何端口映射—仅是纯粹的 nginx 容器。我们可以通过以下命令检查它们的 ID：

```
$ podman ps –qa
b38ebfed5921
6204efc6d6b2
762967d87657
269f1affb699
1161072ec559
```

如何快速停止并删除所有正在运行的容器？我们可以利用 shell 扩展将其与其他命令结合，达到预期结果。Shell 扩展是一个强大的工具，它在圆括号内运行命令，并将输出字符串作为参数传递给外部命令，如下面的代码片段所示：

```
$ podman stop $(podman ps -qa)
$ podman rm $(podman ps -qa)
```

这两个命令停止了所有正在运行的容器，通过它们的 ID 进行标识，并将它们从主机上删除。

`podman ps`命令允许用户通过应用特定的过滤器来精细化输出。所有适用的过滤器的完整列表可以在`podman-ps`手册页中找到。一个简单而有用的应用是状态过滤器，它使用户能够仅打印处于特定状态的容器。可能的状态有`created`、`exited`、`paused`、`running`和`unknown`。

以下示例仅打印处于`exited`状态的容器：

```
$ podman ps --filter status=exited
```

同样，我们可以利用 shell 扩展的强大功能，仅删除已退出的容器，如下所示：

```
$ podman rm $(podman ps -qa --filter status=exited)
```

通过这里展示的更易记住的`podman container prune`命令，也可以获得类似的结果，该命令会移除（清理）主机上所有停止的容器：

```
$ podman container prune
```

排序是列出容器时生成有序输出的另一个有用选项。以下示例展示了如何按容器 ID 进行排序：

```
$ podman ps -q --sort id
```

`podman ps`命令支持使用 Go 模板进行格式化，以生成自定义输出。下一个示例仅打印容器 ID 和容器内执行的命令：

```
$ podman ps -a --format "{{.ID}}  {{.Command}}" --no-trunc 
```

另外，请注意，`--no-trunc`选项已添加，以避免截断命令输出。虽然这不是强制性的，但在容器内部执行长命令时，它是非常有用的。

如果我们仅希望提取正在运行的容器内部进程的主机 PID，可以运行以下示例：

```
$ podman ps --format "{{ .Pid }}"
```

如果我们需要查找关于隔离命名空间的信息，`podman ps`可以打印正在运行容器的克隆命名空间的详细信息。这是进行高级故障排除和检查的一个有用起点。你可以看到这里执行的命令：

```
$ podman ps --namespace
CONTAINER ID  NAMES                 PID         CGROUPNS    IPC         MNT         NET         PIDNS       USERNS      UTS
f2666ed4a46a  unruffled_hofstadter  437764      4026533088  4026533086  4026533083  4026532948  4026533087  4026532973  4026533085
```

本小节介绍了许多常见操作，用于控制和查看容器的状态。在下一节中，我们将学习如何暂停和恢复正在运行的容器。

## 暂停和恢复容器

本小节介绍了`podman pause`和`podman unpause`命令。尽管这是一个与容器状态处理相关的部分，但了解 Podman 和容器运行时如何利用**控制组**（**cgroups**）来实现特定目的，仍然很有趣。

简单来说，`pause`和`unpause`命令的目的是暂停和恢复正在运行容器中的进程。现在，读者可能需要澄清`pause`和`stop`命令之间的区别。

当`podman stop`命令仅向容器中的父进程发送`SIGTERM`/`SIGKILL`信号时，`podman pause`命令则使用 cgroups 暂停进程而不终止它。当容器被恢复时，相同的进程将透明地继续运行。

提示

`pause`/`unpause`的低级逻辑是在容器运行时实现的——对于最为好奇的人，这是写作时在`crun`中的实现：

https://github.com/containers/crun/blob/7ef74c9330033cb884507c28fd8c267861486633/src/libcrun/cgroup.c#L1894-L1936

以下示例演示了`podman pause`和`podman unpause`命令。首先，让我们启动一个 Fedora 容器，该容器每 2 秒在一个无限循环中打印日期和时间字符串，如下所示：

```
$ podman run --name timer docker.io/library/fedora bash -c "while true; do echo $(date); sleep 2; done"
```

我们故意让容器在一个窗口中保持运行，并打开一个新的窗口/标签页来管理其状态。在发出`pause`命令之前，让我们通过执行以下代码检查 PID：

```
$ podman ps --format "{{ .Pid }}" --filter name=timer
816807
```

现在，让我们使用以下命令暂停正在运行的容器：

```
$ podman pause timer
```

如果我们回到`timer`容器，我们会看到输出刚刚被暂停，但容器并没有退出。此处看到的`unpause`操作将使其恢复：

```
$ podman unpause timer
```

在 `unpause` 操作之后，timer 容器将再次开始打印日期输出。查看此处的 PID，正如预期的那样，什么都没有改变：

```
$ podman ps --format "{{ .Pid }}" --filter name=timer
816807
```

我们可以检查暂停/未暂停容器的 cgroups 状态。在第三个标签页中，打开一个具有 root shell 的终端，并在替换为正确的容器 ID 后访问 `cgroupfs` 控制器层次结构，如下所示：

```
$ sudo –i
$ cd /sys/fs/cgroup/user.slice/user-1000.slice/user@1000.service/user.slice/libpod-<CONTAINER_ID>.scope/container
```

现在，查看 `cgroup.freeze` 文件的内容。该文件包含一个布尔值，其状态会随着我们暂停/恢复容器的操作从 0 变为 1，反之亦然。尝试再次暂停和恢复容器以测试变化。

清理提示

由于 echo 循环是通过 `bash -c` 命令发出的，因此我们需要向进程发送 `SIGKILL` 信号。为此，我们可以停止容器并等待 10 秒超时，或者直接运行 `podman kill` 命令，如下所示：

`$ podman kill timer`

在本小节中，我们详细介绍了用于监视和修改容器状态的最常见命令。现在我们可以继续检查正在运行的容器内的进程。

## 检查容器内的进程

当容器正在运行时，容器内的进程在命名空间级别上被隔离，但用户仍然对正在运行的进程拥有完全控制权，并且可以检查它们的行为。进程检查有很多复杂的层次，但 Podman 提供了可以加速此任务的工具。

我们从 `podman top` 命令开始：它提供了容器内进程的完整视图。以下示例显示了在 nginx 容器内运行的进程：

```
$ podman top  f2666ed4a46a
USER        PID         PPID        %CPU        ELAPSED          TTY         TIME        COMMAND
root        1           0           0.000       3m26.540290427s  ?           0s          nginx: master process nginx -g daemon off; 
nginx       26          1           0.000       3m26.540547429s  ?           0s          nginx: worker process 
nginx       27          1           0.000       3m26.540788803s  ?           0s          nginx: worker process 
nginx       28          1           0.000       3m26.540914386s  ?           0s          nginx: worker process 
nginx       29          1           0.000       3m26.541040023s  ?           0s          nginx: worker process 
nginx       30          1           0.000       3m26.541161213s  ?           0s          nginx: worker process 
nginx       31          1           0.000       3m26.541297546s  ?           0s          nginx: worker process 
nginx       32          1           0.000       3m26.54141773s   ?           0s          nginx: worker process 
nginx       33          1           0.000       3m26.541564289s  ?           0s          nginx: worker process 
nginx       34          1           0.000       3m26.541685475s  ?           0s          nginx: worker process 
nginx       35          1           0.000       3m26.541808977s  ?           0s          nginx: worker process 
nginx       36          1           0.000       3m26.541932099s  ?           0s          nginx: worker process 
nginx       37          1           0.000       3m26.54205111s   ?           0s          nginx: worker process
```

结果非常类似于 `ps` 命令的输出，而不是由 Linux `top` 命令生成的交互式输出。

可以对输出应用自定义格式。以下示例只打印 PID、命令和参数：

```
$ podman top f2666ed4a46a pid comm args
PID         COMMAND     COMMAND
1           nginx       nginx: master process nginx -g daemon off; 
26          nginx       nginx: worker process 
27          nginx       nginx: worker process 
28          nginx       nginx: worker process 
29          nginx       nginx: worker process 
30          nginx       nginx: worker process 
31          nginx       nginx: worker process 
32          nginx       nginx: worker process 
33          nginx       nginx: worker process 
34          nginx       nginx: worker process 
35          nginx       nginx: worker process 
36          nginx       nginx: worker process 
37          nginx       nginx: worker process
```

我们可能需要更详细地检查容器进程。正如我们在*第一章*《容器技术简介》中讨论的那样，一旦启动一个全新的容器，它将从 0 开始分配 PID，同时在后台，容器引擎将该容器的 PID 与主机上的真实 PID 映射。因此，我们可以使用 `podman ps --namespace` 命令的输出，提取给定容器在主机上的原始 PID。有了这些信息，我们就可以进行高级分析。以下示例展示了如何附加 `strace` 命令，该命令用于检查进程的 **系统调用** (**syscalls**)，到容器内运行的进程：

```
$ sudo strace –p <PID>
```

`strace` 命令的使用细节超出了本书的范围。有关更高级的示例以及命令选项的更深入解释，请参见 `man strace`。

另一个可以轻松应用于容器内运行的进程的有用命令是 `pidstat`。一旦我们获得了 PID，就可以通过以下方式检查资源使用情况：

```
$ pidstat –p <PID> [<interval> <count>]
```

最后的整数依次表示命令执行间隔和必须打印使用情况统计的次数。详见`man pidstat`以获取更多使用选项。

当容器中的进程变得无响应时，可以使用`podman kill`命令处理其突然终止。默认情况下，它向容器内部的进程发送`SIGKILL`信号。以下示例创建一个`httpd`容器，然后将其终止：

```
$ podman run --name custom-webserver -d docker.io/library/httpd
$ podman kill custom-webserver
```

我们可以使用`--signal`选项选择性地发送自定义信号（如`SIGTERM`或`SIGHUP`）。请注意，终止的容器不会从主机中移除，而是继续存在、停止并处于退出状态。

在*第十章*，*故障排除和监控容器*中，我们将再次处理容器故障排除，并学习如何使用`nsenter`等高级工具来检查容器进程。现在我们继续进行基本容器统计命令，这些命令对于监视系统中所有运行容器的整体资源使用非常有用。

## 监控容器统计信息

当多个容器在同一主机上运行时，监控使用`podman stats`命令至关重要，如下所示：

```
$ podman stats
```

如果没有任何选项，该命令将打开类似 top 的自刷新窗口，显示所有运行容器的统计信息。默认打印的值列于此处：

+   `ID`：运行中的容器 ID

+   `NAME`：运行中的容器名称

+   `CPU %`：总 CPU 使用率百分比

+   `MEM USAGE / LIMIT`：相对于给定限制（由系统能力或 cgroups 驱动的限制）的内存使用

+   `MEM %`：内存使用率百分比

+   `NET IO`：网络输入/输出（I/O）操作

+   `BLOCK IO`：磁盘 I/O 操作

+   `PIDS`：容器内的 PID 数量

+   `CPU TIME`：总消耗的 CPU 时间

+   `AVG CPU %`：平均 CPU 使用率百分比

如果需要重定向，可以使用`--no-stream`选项避免流式自刷新输出，如下所示：

```
$ podman stats --no-stream
```

无论如何，这种类型的静态输出并不适合解析或摄入。更好的方法是应用 JSON 或 Go 模板格式化程序。以下示例以 JSON 格式打印统计信息：

```
$ podman stats --format=json
[
 {
  "id": "e263f68bbb83",
  "name": "infallible_sinoussi",
  "cpu_time": "33.518ms",
  "cpu_percent": "2.05%",
  "avg_cpu": "2.05%",
  "mem_usage": "19.3MB / 33.38GB",
  "mem_percent": "0.06%",
  "net_io": "-- / --",
  "block_io": "-- / --",
  "pids": "13"
 }
]
```

类似地，可以使用 Go 模板定制输出字段。以下示例仅打印容器 ID、CPU 使用率、总内存使用量（以字节为单位）和进程 ID：

```
$ podman stats -a --no-stream --format "{{ .ID }} {{ .CPUPerc }} {{ .MemUsageBytes }} {{ .PIDs }}"
```

在本节中，我们学习了如何监视运行中的容器及其隔离的进程。下一节将展示如何检查容器配置以进行分析和故障排除。

# 检查容器信息

运行中的容器暴露一组可供使用的配置数据和元数据。Podman 实现了`podman inspect`命令以打印所有容器配置和运行时信息。在其最简单形式下，我们只需传递容器 ID 或名称，如下所示：

```
$ podman inspect <Container_ID_or_Name>
```

该命令打印出包含所有容器配置的 JSON 输出。为了节省空间，我们将在这里列出一些最显著的字段：

+   `Path`：容器入口点路径。当我们分析 Dockerfile 时，我们将进一步探讨入口点。

+   `Args`：传递给入口点的参数。

+   `State`：容器的当前状态，包括关键的执行 PID、公共 PID、OCI 版本和健康检查状态等信息。

+   `Image`：用于运行容器的镜像 ID。

+   `Name`：容器名称。

+   `MountLabel`：容器的挂载标签，适用于**增强型安全 Linux**（**SELinux**）。

+   `ProcessLabel`：容器进程标签，适用于 SELinux。

+   `EffectiveCaps`：应用于容器的有效能力。

+   `GraphDriver`：存储驱动类型（默认是 `overlayfs`）以及覆盖的上层、下层和合并目录的列表。

+   `Mounts`：容器中的实际绑定挂载。

+   `NetworkSettings`：容器的整体网络设置，包括其内部**互联网协议**（**IP**）地址、暴露的端口和端口映射。

+   `Config`：容器的运行时配置，包括环境变量、主机名、命令、工作目录、标签和注释。

+   `HostConfig`：主机配置，包括 cgroups 配额、网络模式和能力。

这是一大堆信息，大多数时候这些信息对于我们的需求来说过于冗长。当我们需要提取特定字段时，可以使用 `--format` 选项仅打印所选字段。以下示例仅打印容器内执行进程的主机绑定 PID：

```
$ podman inspect <ID or Name> --format "{{ .State.Pid }}"
```

结果采用 Go 模板格式。这使得我们可以灵活地自定义输出字符串。

`podman inspect` 命令对于理解容器引擎的行为以及在故障排除过程中获取有用信息也非常有帮助。

例如，当启动容器时，我们可以了解到 `resolv.conf` 文件是从在 `{{ .ResolvConfPath }}` 键中定义的路径挂载到容器内的。当容器在 rootless 模式下执行时，目标路径是 `/run/user/<UID>/containers/overlay-containers/<Container_ID>/userdata/resolv.conf`，在 rootful 模式下则是 `/var/run/containers/storage/overlay-containers/<Container_ID>/userdata/resolv.conf`。

另一个有趣的信息是 `overlayfs` 管理的所有合并层的列表。让我们尝试运行一个新的容器，这次使用 rootful 模式，并查找有关合并层的信息，操作如下：

```
# podman run --name logger -d docker.io/library/fedora bash -c "while true; do echo test >> /tmp/test.log; sleep 5; done"
```

该容器运行一个简单的循环，每 5 秒将一个字符串写入文本文件。现在，让我们运行一个 `podman inspect` 命令，了解 `MergedDir` 的信息，这个目录是所有层通过 `overlayfs` 合并的地方。代码示例如下：

```
# podman inspect logger --format "{{ .GraphDriver.Data.MergedDir  }}"
/var/lib/containers/storage/overlay/27d89046485db7c775b108a80072eafdf9aa63d14ee1205946d746 23fc195314/merged
```

在这个目录内，我们可以找到 `/tmp/test.log` 文件，具体如下所示：

```
# cat /var/lib/containers/storage/overlay/27d89046485db7c775b108a80072eafdf9aa63d14ee1205946d746 23fc195314/merged/tmp/test.log
test
test
test
test
test
[...]
```

我们可以深入探讨——`LowerDir`目录包含基础镜像层的列表，如下代码片段所示：

```
# podman inspect logger \
--format "{{ .GraphDriver.Data.LowerDir}}"
 /var/lib/containers/storage/overlay/4c85102d65a59c6d478bfe6bc0bf32e8c79d9772689f62451c7196 380675d4af/diff
```

在这个示例中，基础镜像由一个层组成。我们会在这里找到日志文件吗？让我们来看看：

```
# cat /var/lib/containers/storage/overlay/4c85102d65a59c6d478bfe6bc0bf32e8c79d9772689f62451c7196 380675d4af/diff/tmp/test.log
cat: /var/lib/containers/storage/overlay/4c85102d65a59c6d478bfe6bc0bf32e8c79d9772689f62451c7196 380675d4af/diff/tmp/test.log: No such file or directory
```

我们在这一层中缺少日志文件。这是因为`LowerDir`目录是只读的，代表了只读镜像层。它与`UpperDir`目录合并，后者是容器的读写层。通过`podman inspect`，我们可以找出它所在的位置，如下所示：

```
# podman inspect logger --format "{{ .GraphDriver.Data.UpperDir }}"
/var/lib/containers/storage/overlay/27d89046485db7c775b108a80072eafdf9aa63d14ee1205946d746 23fc195314/diff
```

输出目录将仅包含自容器启动以来写入的一堆文件和目录，其中包括`/tmp/test.log`文件，如下所示：

```
# cat /var/lib/containers/storage/overlay/27d89046485db7c775b108a80072eafdf9aa63d14ee1205946d746 23fc195314/diff/tmp/test.log
test
test
test
test
test
[...]
```

我们现在可以通过运行以下命令来停止并删除日志容器：

```
# podman stop logger && podman rm logger
```

这个示例是为了预示将在*第五章*中讨论的容器存储话题，*为容器的数据实现存储*。`overlayfs`机制，包含下层、上层和合并目录的概念，将被更详细地分析。

在本节中，我们学习了如何检查运行中的容器并收集运行时信息和配置。下一节将涵盖捕获容器日志的最佳实践。

# 捕获容器日志

正如本章前面所描述的，容器由一个或多个可能会失败的进程组成，这些进程会打印错误并在日志文件中描述其当前状态。那么，这些日志存储在哪里呢？

当然，容器中的进程也可以将日志消息写入临时文件系统中的某个文件，而该文件系统是容器引擎为它提供的（如果有的话）。但是，读写文件系统或运行中的容器中的任何权限限制又该怎么办呢？

容器暴露相关日志的最佳实践实际上是利用标准流的使用：`STDOUT`）和`STDERR`）。

需要知道的是

标准流是与操作系统中正在运行的进程互联的通信通道。当程序通过交互式 Shell 运行时，这些流会直接连接到用户的终端，以便在终端和进程之间传输输入、输出和错误信息，反之亦然。

根据我们运行全新容器时所使用的选项，Podman 将适当操作，将`STDIN`、`STDOUT`和`STDERR`标准流连接到本地文件，用于存储日志。

在*第三章*中，*运行第一个容器*，我们看到如何在后台运行容器，并从运行中的容器中分离。我们使用`-d`选项通过`podman run`命令以*分离模式*启动容器，如下所示：

```
$ podman run -d -i -t registry.fedoraproject.org/f29/httpd
```

使用之前的命令，我们指示 Podman 以分离模式（`-d`）启动容器，并将伪终端附加到`STDIN`流（`-t`），即使还没有附加终端，也保持标准输入流打开（`-i`）。

Podman 的标准行为是附加到`STDOUT`和`STDERR`流，并将任何容器发布的数据存储在主机文件系统中的日志文件中。

如果我们作为*root*用户使用 Podman 工作，可以通过执行以下步骤查看主机系统上的日志文件：

1.  首先，我们需要启动容器并记录 Podman 返回的 ID，或者请求 Podman 列出容器并记录它们的 ID。实现此操作的代码如下所示：

    ```
    # podman run -d -i -t registry.fedoraproject.org/f29/httpd
    c6afe22eac7c22c35a303d5fed45bc1b6442a4cec4a9060f392362bc
    4cecb25d
    # .
    CONTAINER ID                                                      IMAGE                                        COMMAND             CREATED         STATUS             PORTS       NAMES
    c6afe22eac7c22c35a303d5fed45bc1b6442a4cec4a9060f392362bc4 cecb25d  registry.fedoraproject.org/f29/httpd:latest  /usr/bin/run-httpd  27 minutes ago  Up 27 minutes ago              gifted_allen
    ```

1.  之后，我们可以查看`/var/lib/containers/storage/overlay-containers/`目录，并搜索与我们容器 ID 匹配的文件夹，具体如下：

    ```
    # cd /var/lib/containers/storage/overlay-containers/c6afe22eac7c22c35a303d5fed45bc1b6442a4cec4a9060f392362bc4 cecb25d/
    ```

1.  最后，我们可以通过查看`userdata`目录下名为`ctr.log`的文件来检查正在运行的容器的日志，具体如下：

    ```
    # cat userdata/ctr.log 
    2021-09-27T15:42:46.925288013+00:00 stdout P => sourcing 10-set-mpm.sh ...
    2021-09-27T15:42:46.925604590+00:00 stdout F 
    2021-09-27T15:42:46.926882725+00:00 stdout P => sourcing 20-copy-config.sh ...
    2021-09-27T15:42:46.926920142+00:00 stdout F 
    2021-09-27T15:42:46.929405654+00:00 stdout P => sourcing 40-ssl-certs.sh ...
    2021-09-27T15:42:46.929460531+00:00 stdout F 
    2021-09-27T15:42:46.987174441+00:00 stdout P AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.9\. Set the 'ServerName' directive globally to suppress this message
    2021-09-27T15:42:46.987242961+00:00 stdout F 
    2021-09-27T15:42:46.996989350+00:00 stdout F [Mon Sep 27 15:42:46.996748 2021] [ssl:warn] [pid 1:tid 139708367605120] AH01882: Init: this version of mod_ssl was compiled against a newer library (OpenSSL 1.1.1b FIPS  26 Feb 2019, version currently loaded is OpenSSL 1.1.1 FIPS  11 Sep 2018) - may result in undefined or erroneous behavior
    ...
    2021-09-27T15:42:47.101066096+00:00 stdout F [Mon Sep 27 15:42:47.099445 2021] [core:notice] [pid 1:tid 139708367605120] AH00094: Command line: 'httpd -D FOREGROUND'
    ```

我们刚刚发现了 Podman 保存所有容器日志的秘密位置！

请注意，我们刚才介绍的过程只有在`containers.conf`文件中的`log_driver`字段设置为`k8s-file`值时才能正常工作。例如，在 Fedora Linux 发行版从版本 35 开始，维护者决定将`k8s-file`更改为`journald`。在这种情况下，您可以直接使用`journalctl`命令行工具查找日志。

如果您想查看默认的`log_driver`字段，可以查看以下路径：

```
# grep log_driver /usr/share/containers/containers.conf
```

这是否意味着每次需要分析容器的日志时，都需要执行整个复杂的过程？当然不是！

Podman 有一个内建的`podman logs`命令，可以轻松发现、抓取并打印最新的容器日志。考虑到前面的例子，我们可以通过执行以下命令轻松检查正在运行的容器的日志：

```
# podman logs c6afe22eac7c22c35a303d5fed45bc1b6442a4cec4a9060f 392362bc4cecb25d
=> sourcing 10-set-mpm.sh ...
=> sourcing 20-copy-config.sh ...
=> sourcing 40-ssl-certs.sh ...
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.9\. Set the 'ServerName' directive globally to suppress this message
[Mon Sep 27 15:42:46.996748 2021] [ssl:warn] [pid 1:tid 13970 8367605120] AH01882: Init: this version of mod_ssl was compiled  against a newer library (OpenSSL 1.1.1b FIPS  26 Feb 2019, version currently loaded is OpenSSL 1.1.1 FIPS  11 Sep 2018) - may result in undefined or erroneous behavior
...
[Mon Sep 27 15:42:47.099445 2021] [core:notice] [pid 1:tid 139708367605120] AH00094: Command line: 'httpd -D FOREGROUND'
```

我们还可以获取正在运行的容器的短 ID，并将此 ID 传递给`podman logs`命令，具体如下：

```
# podman ps
CONTAINER ID  IMAGE                                        COMMAND               CREATED         STATUS             PORTS       NAMES
c6afe22eac7c  registry.fedoraproject.org/f29/httpd:latest  /usr/bin/run-http...  40 minutes ago  Up 40 minutes ago              gifted_allen
# podman logs --tail 2 c6afe22eac7c
[Mon Sep 27 15:42:47.099403 2021] [mpm_event:notice] [pid 1:tid 139708367605120] AH00489: Apache/2.4.39 (Fedora) OpenSSL/1.1.1 configured -- resuming normal operations
[Mon Sep 27 15:42:47.099445 2021] [core:notice] [pid 1:tid 1397 08367605120] AH00094: Command line: 'httpd -D FOREGROUND'
```

在之前的命令中，我们还使用了`podman logs`命令的一个很棒的选项：`--tail`选项，它允许我们仅输出容器日志中最新需要的行。在我们的例子中，我们请求了最新的两行。

正如我们在本节前面所看到的，Podman 将容器日志保存到主机文件系统中。默认情况下，这些文件没有大小限制，因此可能会发生，对于长时间运行并产生大量日志的容器，这些文件可能会变得非常大。

因此，我们通常谈论日志和日志文件时，一个重要的配置参数可以通过 Podman 全局配置文件来帮助减少日志文件的大小，该文件位于以下位置：`/etc/containers/containers.conf`。

如果这个配置文件丢失，你可以轻松创建一个新的文件，插入以下几行来应用配置：

```
# vim /etc/containers/containers.conf
[containers]
log_size_max=10000000
```

通过之前的配置，我们将所有未来运行容器的日志文件限制为 10 **兆字节**（**MB**）。如果你有一些正在运行的容器，必须重启它们以应用此新配置。

现在我们准备好进入下一部分，在那里我们将发现另一个有用的命令。

# 在运行中的容器内执行进程

在*第二章*的 *Podman 无守护进程架构* 部分中，我们讨论了 Podman（与任何其他容器引擎一样）利用 Linux 命名空间功能来正确地将运行中的容器相互隔离，并且与操作系统主机隔离。

因此，仅仅因为 Podman 为每个运行中的容器创建了一个全新的命名空间，我们就不应该感到惊讶，我们可以附加到同一个正在运行的容器的 Linux 命名空间，像在完整操作环境中一样执行其他进程。

Podman 允许我们通过 `podman exec` 命令在运行中的容器内执行一个进程。

一旦执行，此命令将内部查找目标运行容器所附加的正确 Linux 命名空间。在找到该命名空间后，Podman 将执行传递给 `podman exec` 命令的相应进程，并将其附加到目标 Linux 命名空间。最终的进程将处于与原始进程伴随的相同环境中，并能够与其交互。

为了理解这个在实践中的工作原理，我们可以考虑以下示例：我们将首先运行一个容器，然后在现有进程旁边执行一个新进程：

```
# podman run -d -i -t registry.fedoraproject.org/f29/httpd
47fae73e4811a56d799f258c85bc50262901bec2f9a9cab19c01af89713 a1248
# podman exec -ti 47fae73e4811a56d799f258c85bc50262901bec2f9a9cab19c01af89713 a1248 /bin/bash
bash-4.4$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START    TIME COMMAND
default        1  0.6  0.6  20292 13664 pts/0    Ss+  13:37    0:00 httpd -D FOREGROUND
...
```

如你从之前的命令中看到的那样，我们获取了 Podman 提供的容器 ID，当容器启动后，我们将其作为参数传递给 `podman exec` 命令。

`podman exec` 命令对于故障排除、测试和与现有容器一起工作非常有用。在前面的示例中，我们附加了一个交互式终端，运行了 Bash 控制台，并启动了 `ps` 命令来检查当前 Linux 命名空间中可用的运行进程。

`podman exec` 命令有许多可用选项，类似于 `podman run` 命令提供的选项。如你从前面的示例中看到的那样，我们使用了一个选项来获取附加到 `STDIN` 流的伪终端（`-t`），即使没有终端附加时也保持标准输入流开启（`-i`）。

要了解更多可用选项，我们可以查看相应命令的手册，如下所示：

```
# man podman exec
```

我们在容器管理的旅程中不断前进，在下一节中，我们将进一步了解 Podman 提供的一些功能，这些功能可以在 Kubernetes 容器编排世界中启用容器化工作负载。

# 在 pod 中运行容器

如我们在*第二章*的*Docker 与 Podman 的主要区别*部分中提到的，*比较 Podman 和 Docker*，Podman 提供了轻松开始采用事实上的容器编排器 Kubernetes（有时也称为`k8s`）的一些基本概念的能力。

pod 的概念是随着 Kubernetes 引入的，它代表了 Kubernetes 集群中的最小执行单元。使用 Podman，用户可以轻松创建空的 pod，然后在其中运行容器。

将两个或更多容器组合到一个 pod 中可以带来许多好处，如下所示：

+   共享相同的网络命名空间，包括 IP 地址

+   共享相同的存储卷以存储持久化数据

+   共享相同的配置

此外，将两个或更多容器放在同一个 pod 中，实际上会使它们共享相同的**进程间通信**（**IPC**）Linux 命名空间。这对于需要通过共享内存进行互相通信的应用程序非常有用。

创建 pod 并开始使用它的最简单方法是使用以下命令：

```
# podman pod create --name myhttp
3950703adb04c6bca7f83619ea28c650f9db37fd0060c1e263cf7ea34 dbc8dad
# podman pod ps
POD ID        NAME        STATUS      CREATED        INFRA ID       # OF CONTAINERS
3950703adb04  myhttp      Created     6 seconds ago  1bdc82 e77ba2  1
```

如前所示，我们创建了一个名为`myhttp`的新 pod，然后检查了主机系统上 pod 的状态：只有一个 pod 处于`created`状态。

我们现在可以如下启动 pod 并查看会发生什么：

```
# podman pod start myhttp
3950703adb04c6bca7f83619ea28c650f9db37fd0060c1e263cf7ea34 dbc8dad
# podman pod ps
POD ID        NAME        STATUS      CREATED             INFRA ID      # OF CONTAINERS
3950703adb04  myhttp      Running     About a minute ago  1bdc82e77ba2  1
```

pod 现在正在运行，但 Podman 实际上在运行什么呢？我们创建了一个没有容器的空 pod！让我们通过执行`podman ps`命令来看一下正在运行的容器，如下所示：

```
# podman ps
CONTAINER ID  IMAGE                 COMMAND     CREATED             STATUS            PORTS       NAMES
1bdc82e77ba2  k8s.gcr.io/pause:3.5              About a minute ago  Up 6 seconds ago              3950703adb04-infra
```

`podman ps`命令显示了一个名为`pause`的镜像正在运行的容器。Podman 默认将此容器作为`infra`容器运行。这种类型的容器什么也不做——它仅仅持有命名空间并允许容器引擎连接到 pod 内的任何其他正在运行的容器。

在揭开这个特殊容器在我们 pods 中的角色后，我们现在可以简要地了解启动一个多容器 pod 所需的步骤。

首先，让我们从在之前示例中创建的现有 pod 内部运行一个新容器开始，如下所示：

```
# podman run --pod myhttp -d -i -t registry.fedoraproject.org/f29/httpd
Cb75e65f10f6dc37c799a3150c1b9675e74d66d8e298a8d19eadfa125d ffdc53
```

然后，我们可以检查现有 pod 是否更新了它所包含的容器数量，如下所示的代码片段所示：

```
# podman pod ps
POD ID        NAME        STATUS      CREATED         INFRA ID       # OF CONTAINERS
3950703adb04  myhttp      Running     21 minutes ago  1bdc82e77ba2  2
```

最后，我们可以要求 Podman 列出正在运行的容器及其相关的 pod 名称，如下所示：

```
# podman ps -p
CONTAINER ID  IMAGE                                        COMMAND               CREATED         STATUS             PORTS       NAMES                POD ID        PODNAME
1bdc82e77ba2  k8s.gcr.io/pause:3.5                                               22 minutes ago  Up 20 minutes ago              3950703adb04-infra   3950703adb04  myhttp
cb75e65f10f6  registry.fedoraproject.org/f29/httpd:latest  /usr/bin/run-http...  4 minutes ago   Up 4 minutes ago               determined_driscoll  3950703adb04  myhttp
```

如你所见，运行的两个容器都与名为`myhttp`的 pod 相关联！

重要提示

请考虑在完成本章中的所有示例后定期清理实验环境。这可以帮助你节省资源，并避免在进行下一章示例时出现任何错误。为此，你可以参考本书 GitHub 仓库中 `AdditionalMaterial` 文件夹中的代码：https://github.com/PacktPublishing/Podman-for-DevOps/tree/main/AdditionalMaterial。

采用相同的方式，我们可以将越来越多的容器添加到同一个 pod 中，让它们共享之前描述的所有数据。

请注意，将容器放置在同一个 pod 中在某些情况下可能是有益的，但这对于容器技术来说是一个反模式。事实上，正如之前所提到的，Kubernetes 将**pod**视为在分布式节点上运行的最小计算单元。这意味着，一旦你将两个或更多容器组合在同一个 pod 中，它们将在同一节点上一起执行，而编排器无法将它们的工作负载平衡或分配到多台机器上。

我们将在接下来的章节中进一步探索 Podman 的功能，这些功能能帮助你通过 Kubernetes 进入容器编排的世界！

# 总结

在本章中，我们开始积累管理容器的经验，从容器镜像开始，然后使用运行中的容器。一旦我们的容器启动运行，我们还探讨了在 Podman 中可用的各种命令，用于检查日志、检查容器的状态和故障排除。监控和管理运行中的容器是任何容器管理员非常重要的操作。最后，我们还简要了解了 Podman 中与 Kubernetes 相关的概念，这些概念让我们可以将两个或更多容器组合在同一个 Linux 命名空间中。我们刚刚讲解的所有概念和示例将帮助我们开始作为容器技术的系统管理员的经验。

我们现在准备在下一章中探讨另一个重要主题：管理我们的容器存储！
