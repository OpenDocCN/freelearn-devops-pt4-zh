# *第六章*：认识 Buildah – 从零构建容器

容器的巨大吸引力在于，它们允许我们将应用程序打包到不可变的镜像中，这些镜像可以在系统上部署并无缝运行。在本章中，我们将学习如何使用不同的技术和工具创建镜像。这包括了解镜像构建的底层原理以及如何从零开始创建镜像。

本章将涵盖以下主要主题：

+   使用 Podman 构建基本镜像

+   认识 Buildah，Podman 的构建伴侣工具

+   准备我们的环境

+   选择我们的构建策略

+   从零开始构建镜像

+   从 Dockerfile 构建镜像

# 技术要求

在继续本章之前，需要一台已安装 Podman 的工作机器。正如在*第三章*中所述，*运行第一个容器*，书中的所有示例都是在 Fedora 34 或更高版本的系统上执行的，但可以在读者选择的操作系统上重现。

理解*第四章*中涉及的内容，*管理运行中的容器*，对轻松理解与 **开放容器倡议**（**OCI**）镜像相关的概念非常有帮助。

# 使用 Podman 构建基本镜像

容器的 OCI 镜像是一组不可变的层，按复制写入逻辑堆叠在一起。当镜像构建时，所有层会按照精确的顺序创建，然后推送到容器注册表，容器注册表以 tar 格式存储我们的层及附加的镜像元数据。

正如我们在*第二章*的*OCI 镜像*部分中学到的，*比较 Podman 和 Docker*，这些清单对于正确地重新组合镜像层（镜像清单和镜像索引）以及将运行时配置传递给容器引擎（镜像配置）是必要的。

在继续学习使用 Podman 构建镜像的基本示例之前，我们需要了解镜像构建的一般工作原理，以便理解其背后那些简单但非常聪明的关键概念。

## 构建背后的原理

容器镜像可以通过不同的方式构建，但最常见的方法，可能也是容器巨大成功的关键之一，是基于 Dockerfile。

**Dockerfile**，顾名思义，是 Docker 构建的主要配置文件，它是构建过程中需要执行的操作的普通列表。

随着时间的推移，Dockerfile 成为了 OCI 镜像构建的标准，并且今天在许多使用场景中得到了采用。

重要提示

为了标准化并去除与品牌的关联，Containerfile 也被引入了；它们与 Dockerfile 具有完全相同的语法，并且 Podman 原生支持。在本书中，我们将交替使用 *Dockerfile* 和 *Containerfile* 这两个术语。

我们将在下一小节详细学习 Dockerfile 的语法。现在，先聚焦于一个概念——Dockerfile 是一组构建指令，构建工具将按顺序执行这些指令。我们来看这个例子：

```
FROM docker.io/library/fedora
RUN dnf install -y httpd && dnf clean all -y 
COPY index.html /var/www/html
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

这个基本的 Dockerfile 示例只有四个指令：

+   `FROM` 指令，定义将要使用的基础镜像

+   `RUN` 指令，在构建过程中执行一些操作（在此示例中，使用 `dnf` 包管理器安装软件包）

+   `COPY` 指令，用于将文件或目录从构建工作目录复制到镜像中

+   `CMD` 指令，定义容器启动时要执行的命令

当执行示例中的 `RUN` 和 `COPY` 操作时，会有新的层来缓存变更，这些变更被缓存到中间层，并通过临时容器表示。这是 Docker 中的一个原生特性，它的优点是当特定层没有请求变更时，可以在后续构建中重用缓存层。所有中间容器都会生成只读层，并通过叠加图驱动合并。

用户无需手动管理缓存层——引擎会通过创建临时容器、执行 Dockerfile 指令定义的操作，然后提交的方式自动实现必要的操作。通过对所有必要指令重复相同的逻辑，Podman 在基础镜像的层上创建一个包含额外层的新镜像。

可以将镜像层压缩成单一层，以避免对叠加层性能的负面影响。Podman 提供相同的功能，并允许用户选择是否缓存中间层。

并非所有 Dockerfile 指令都会改变文件系统，只有那些会改变文件系统的指令才会创建新的镜像层；所有其他指令，比如前面示例中的 `CMD` 指令，仅会生成一个包含元数据的空层，不会对叠加文件系统造成变化。

通常，唯一通过有效更改文件系统来创建新层的指令是 `RUN`、`COPY` 和 `ADD` 指令。Dockerfile 或 Containerfile 中的其他所有指令仅创建临时中间镜像，并不会影响最终镜像的文件系统。

这也是限制 Dockerfile 中 `RUN`、`COPY` 和 `ADD` 指令数量的一个好理由，因为镜像中包含过多层是不好的模式，并且会影响图驱动的性能。

我们可以检查镜像的历史以及对每个层应用的操作。以下示例展示了 `podman inspect` 命令的输出摘录，目标镜像是从之前的 Dockerfile 示例创建的潜在镜像：

```
$ podman inspect myhttpd
[...omitted output]
        "History": [
            {
                "created": "2021-04-01T17:59:37.09884046Z",
                "created_by": "/bin/sh -c #(nop)  LABEL maintainer=Clement Verna \u003ccverna@fedoraproject.org\u003e",
                "empty_layer": true
            },
            {
                "created": "2021-04-01T18:00:19.741002882Z",
                "created_by": "/bin/sh -c #(nop)  ENV DISTTAG=f34container FGC=f34 FBR=f34",
                "empty_layer": true
            },
            {
                "created": "2021-07-23T11:16:05.060688497Z",
                "created_by": "/bin/sh -c #(nop) ADD file:85d7 f2d8e4f31d81b27b8e18dfc5687b5dabfaafdb2408a3059e120e4c15307b in / "
            },
            {
                "created": "2021-07-23T11:16:05.833115975Z",
                "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/bash\"]",
                "empty_layer": true
            },
            {
                "created": "2021-10-24T21:27:18.783034844Z",
                "created_by": "/bin/sh -c dnf install -y httpd \u0026\u0026 dnf clean all -y  ",
                "comment": "FROM docker.io/library/fedora:latest"
            },
            {
                "created": "2021-10-24T21:27:21.095937071Z",
                "created_by": "/bin/sh -c #(nop) COPY file: 78c6e1dcd6f819581b54094fd38a3fd8f170a2cb768101e533c964e 04aacab2e in /var/www/html "
            },
            {
                "created": "2021-10-24T21:27:21.182063974Z",
                "created_by": "/bin/sh -c #(nop) CMD [\"/usr/sbin/httpd\", \"-DFOREGROUND\"]",
                "empty_layer": true
            }
        ]
[...omitted output]
```

查看镜像历史的最后三项，我们可以注意到 Dockerfile 中定义的确切指令，包括最后一个 CMD 指令，该指令不会创建新层，而是创建持久存在于镜像配置中的元数据。

以这种更深入了解镜像构建逻辑的角度来看，让我们在继续 Podman 构建示例之前，先探索一下最常用的 Dockerfile 指令。

## Dockerfile 和 Containerfile 指令

如前所述，Dockerfile 和 Containerfile 共享相同的语法。那些文件中的指令应该被视为（并且确实是）传递给容器引擎或构建工具的命令。本小节提供了最常用指令的概述。

所有 Dockerfile/Containerfile 指令遵循相同的模式：

```
# Comment
INSTRUCTION arguments
```

以下列表提供了最常用指令的非详尽列表：

+   `FROM <image>[:<tag>]`语法，用于确定正确的镜像来使用。

+   `RUN <command>`语法。调用的二进制文件或脚本必须存在于基础镜像或先前的层中。

如前所述，`RUN`指令会创建一个新的镜像层；因此，常见做法是将多个命令连接到同一个`RUN`指令中，以避免创建太多层。

这个示例将三个命令压缩到同一个`RUN`指令中：

```
RUN dnf upgrade -y && \
     dnf install httpd -y && \
     dnf clean all -y
```

+   `COPY <src>… <dest>`语法，并且它有一个非常有用的选项，允许我们定义目标用户和组，而不是稍后手动更改所有权——`--chown=<user>:<group>`。

+   `ADD <src>… <dest>`语法。此指令还支持从源自动提取 tar 文件并直接写入目标路径。

+   `podman run <image> <arguments>`）或来自`CMD`指令。

一个`ENTRYPOINT`镜像不能被命令行参数覆盖。支持的格式如下：

+   `ENTRYPOINT ["command", "param1", "paramN"]`（也称为*exec*格式）

+   `ENTRYPOINT command param1 paramN`（*shell*格式）

如果未设置，默认值为`bash -c`。当设置为默认值时，命令作为参数传递给`bash`进程。例如，如果在运行时或在 CMD 指令中传递`ps aux`命令，容器将执行`bash -c "ps aux"`。

一个常见做法是用一个自定义的**脚本**替换默认的`ENTRYPOINT`命令，该脚本具有相同的行为，并且提供更细粒度的运行时控制。

+   `ENTRYPOINT`指令。它可以是完整的命令或一组传递给自定义脚本或二进制文件的纯参数，设置为`ENTRYPOINT`。它支持的格式如下：

    +   `CMD ["command", "param1", "paramN"]`（*exec*格式）

    +   `CMD ["param1", "paramN"]`（*parameter*格式，用于将参数传递给自定义的`ENTRYPOINT`）

    +   `CMD command param1 paramN`（*shell*格式）

+   `LABEL <key1>=<value1> … <keyN>=<valueN>`语法。

+   `EXPOSE <port>/<protocol>`格式。

+   `ENV <key1>=<value1>… <keyN>=<valueN>`格式。

环境变量也可以在 `RUN` 指令中设置，其作用范围仅限于该指令本身。

+   `VOLUME ["/path/to/dir"]`

+   `VOLUME /path/to/dir`

另请参阅 *第五章* 中的 *将主机存储附加到容器* 部分，了解有关卷的更多详细信息。

+   `RUN`、`CMD` 和 `ENTRYPOINT` 指令。`GID` 值不是必须的。

支持的格式如下：

+   `USER <username>:[<groupname>]`

+   `USER <UID>:[<GID>]`

+   `WORKDIR /path/to/workdir` 格式。

+   `FROM` 指令。其目的是允许在子容器镜像上执行某些最终命令。

支持的格式如下：

+   `ONBUILD ADD . /opt/app`

+   `ONBUILD RUN /opt/bin/custom-build /opt/app/src`

现在我们已经学习了最常见的指令，接下来让我们深入了解第一个使用 Podman 构建的示例。

## 使用 Podman 运行构建

好消息——Podman 提供了与 Docker 相同的构建命令和语法。如果你从 Docker 切换过来，使用 Podman 构建镜像不会有学习曲线。从技术层面上讲，选择 Podman 作为构建工具有一个显著优势——Podman 可以在无根模式下构建容器，使用的是 fork/exec 模型。

这是与 Docker 构建相比的一步进展，后者需要与监听 Unix 套接字的守护进程进行通信才能运行构建。

让我们从基于第一个 *构建过程概述* 子部分中说明的 `httpd` Dockerfile 运行一个简单的构建开始。我们将使用以下 `podman build` 命令：

```
$ podman build -t myhttpd .
STEP 1/4: FROM docker.io/library/fedora
STEP 2/4: RUN dnf install -y httpd && dnf clean all -y  
[...omitted output]
--> 50a981094eb
STEP 3/4: COPY index.html /var/www/html
--> 73f8702c5e0
STEP 4/4: CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
COMMIT myhttpd
--> e773bfee6f2
Successfully tagged localhost/myhttpd:latest e773bfee6f289012b37285a9e559bc44962de3aeed001455231b5a8f2721b8f9
```

在前面的示例中，为了清晰和节省空间，省略了 `dnf install` 命令的输出。

命令按顺序执行指令，并在最终镜像提交和标记之前保留中间层。构建步骤被编号（`1/4`到`4/4`），其中一些步骤（如`RUN`和`COPY`）会产生非空的层，成为镜像的 lowerDirs 部分。

第一条 `FROM` 指令定义了基础镜像，如果主机上不存在，系统会自动拉取该镜像。

第二条指令是`RUN`，它执行`dnf`命令以安装 httpd 软件包，并在完成后清理系统。实际执行时，这一行会作为`"bash -c 'dnf install -y httpd && dnf clean all -y'"`执行。

第三条 `COPY` 指令只是将 `index.html` 文件复制到默认的 httpd 文档根目录。

最后，第四步定义了默认的容器 `CMD` 指令。由于没有设置 `ENTRYPOINT` 指令，这将转换为以下命令：

`"bash -c '/usr/sbin/httpd -DFOREGROUND'"`

下一个示例是一个自定义的 Dockerfile/Containerfile，用于构建一个自定义的 Web 服务器：

```
FROM docker.io/library/fedora
# Install required packages
RUN set -euo pipefail; \
    dnf upgrade -y; \
    dnf install httpd -y; \
    dnf clean all -y; \
    rm -rf /var/cache/dnf/*
# Custom webserver configs for rootless execution
RUN set -euo pipefail; \
    sed -i 's|Listen 80|Listen 8080|' \
           /etc/httpd/conf/httpd.conf; \
    sed -i 's|ErrorLog "logs/error_log"|ErrorLog /dev/stderr|' \
           /etc/httpd/conf/httpd.conf; \
    sed -i 's|CustomLog "logs/access_log" combined|CustomLog /dev/stdout combined|' \
           /etc/httpd/conf/httpd.conf; \
    chown 1001 /var/run/httpd

# Copy web content
COPY index.html /var/www/html
# Define content volume
VOLUME /var/www/html
# Copy container entrypoint.sh script
COPY entrypoint.sh /entrypoint.sh
# Declare exposed ports
EXPOSE 8080 
# Declare default user
USER 1001
ENTRYPOINT ["/entrypoint.sh"]
CMD ["httpd"]
```

这个示例是为了本书的目的设计的，用来说明一些特殊的元素：

+   使用包管理器安装的软件包应保持在最小范围。安装完运行 Web 服务器所需的`httpd`软件包后，清理缓存以节省层空间。

+   多个命令可以在单一的 `RUN` 指令中组合在一起。然而，如果其中一个命令失败，我们不希望继续构建。为了提供容错的 shell 执行，`set -euo pipefail` 命令被预先添加。此外，为了提高可读性，单个命令被使用 `\` 字符拆分成更多行，` \` 可以作为换行符或转义字符。

+   为了避免以 root 用户身份运行隔离的进程，实施了一系列变通方法，使得 httpd 进程以通用的 `1001` 用户身份运行。这些变通方法包括更新特定目录的文件权限和组所有权，预计这些目录将被非 root 用户访问。这是一项安全最佳实践，能减少容器的攻击面。

+   容器中的一个常见模式是将应用程序日志重定向到容器的 `stdout` 和 `stderr`。为了这个目的，常见的 httpd 日志流已经通过对 `/etc/httpd/conf/httpd.conf` 文件的正则表达式进行了修改。

+   Web 服务器端口通过 `EXPOSE` 指令声明为暴露端口。

+   `CMD` 指令是一个简单的 `httpd` 命令，没有其他参数。这是为了说明 `ENTRYPOINT` 如何与 `CMD` 参数交互。

容器的 `ENTRYPOINT` 指令通过一个自定义脚本进行了修改，该脚本为 `CMD` 指令的管理方式带来了更多灵活性。`entrypoint.sh` 文件会测试容器是否以 root 用户身份执行，并检查第一个 `CMD` 参数——如果参数是 `httpd`，则执行 `httpd -DFOREGROUND` 命令；否则，它允许你执行任何其他命令（例如一个 shell）。以下代码是 `entrypoint.sh` 脚本的内容：

```
#!/bin/sh
set -euo pipefail
if [ $UID != 0 ]; then
    echo "Running as user $UID"
fi
if [ "$1" == "httpd" ]; then
    echo "Starting custom httpd server"
    exec $1 -DFOREGROUND
else
    echo "Starting container with custom arguments"
    exec "$@"
fi
```

现在，让我们使用 `podman build` 命令来构建镜像：

```
$ podman build –t myhttpd .
```

新构建的镜像将可用在本地主机的缓存中：

```
$ podman images | grep myhttpd
localhost/myhttpd latest 6dc90348520c 2 minutes ago   248 MB
```

构建完成后，我们可以使用 `v1.0` 标签和最新标签：

```
$ podman tag localhost/myhttpd quay.io/<username>/myhttpd:v1.0
```

在标记之后，镜像将准备好推送到远程注册表。我们将在*第九章*中更详细地讨论与注册表的交互，*推送镜像到容器注册表*。

示例镜像将由五个层组成，包括基础 Fedora 镜像层。我们可以通过运行 `podman inspect` 命令检查新镜像的层数：

```
$ podman inspect myhttpd --format '{{ .RootFS.Layers }}'
[sha256:b6d0e02fe431db7d64d996f3dbf903153152a8f8b857cb4829 ab3c4a3e484a72
sha256:f41274a78d9917b0412d99c8b698b0094aa0de74ec8995c88e5 dbf1131494912
sha256:e57dde895085c50ea57db021bffce776ee33253b4b8cb0fe909b bbac45af0e8c
sha256:9989ee85603f534e7648c74c75aaca5981186b787d26e0cae0bc 7ee9eb54d40d
sha256:ca402716d23bd39f52d040a39d3aee242bf235f626258958b889 b40cdec88b43]
```

可以使用 `--layers=false` 选项将当前的构建层压缩为单一层。生成的镜像将只有两个层——基础 Fedora 层和压缩后的层。以下示例重新构建镜像，不缓存中间层：

```
$ podman build -t myhttpd --layers=false .
```

现在，让我们再次检查输出镜像：

```
$ podman inspect myhttpd --format '{{ .RootFS.Layers }}'
[sha256:b6d0e02fe431db7d64d996f3dbf903153152a8f8b857cb 4829ab3c4a3e484a72
sha256:6c279ab14837b30af9360bf337c7f9b967676a61831eee9 1012fa67083f5dcf1]
```

这一次，最终镜像只包含了两个预期的层。

减少层数有助于保持镜像在叠加层方面的简洁。该方法的缺点是，每次配置更改时，我们必须重新构建整个镜像，无法利用缓存的层。

在隔离方面，Podman 可以安全地在无根模式下构建镜像。实际上，这是一个非常重要的特性，因为构建时无需以具有特权的用户（如 root）身份运行。如果必须使用 root 用户进行构建，也是完全可行并得到支持的。以下示例以 root 用户身份运行构建：

```
# podman build -t myhttpd .
```

生成的镜像将仅保存在系统镜像缓存中，其层次结构将存储在 `/var/lib/containers/storage/` 目录下。

Podman 构建的灵活性与其配套工具**Buildah**密切相关，Buildah 是一个专门用于构建 OCI 镜像的工具，提供了更大的构建灵活性。在下一节中，我们将介绍 Buildah 的特性以及它如何管理镜像构建。

# 认识 Buildah，Podman 的构建配套工具

Podman 在使用 Dockerfile/Containerfile 进行常规构建时表现出色，并帮助团队保留之前实现的构建管道，而无需进行新的投资。

然而，当涉及到更专业的构建任务时，或者用户需要更多的构建流程控制并可以包含脚本逻辑时，Dockerfile/Containerfile 方法会暴露其局限性。社区一直在努力寻找能够克服 Dockerfile/Containerfile 固定工作流逻辑的替代构建方法。

开发 Podman 的同一个社区推出了 Buildah（发音为*build-ah*）项目，这是一个支持多种构建策略的 OCI 构建管理工具。使用 Buildah 创建的镜像完全可移植并与 Docker 兼容，所有引擎都符合 OCI 镜像和运行时规范。

Buildah 是一个开源项目，遵循 Apache 2.0 许可证发布。源代码可以在 GitHub 上找到，网址为：[`github.com/containers/buildah`](https://github.com/containers/buildah)。

Buildah 是 Podman 的补充工具，借用了其构建逻辑，通过将其库嵌入到 Podman 中，实现在 Dockerfile 和 Containerfile 上的基本构建功能。最终编译的 Podman 二进制文件是一个静态链接的 Go 单文件，其中嵌入了 Buildah 包来管理构建步骤。

Buildah 使用 *containers/image* 项目（[`github.com/containers/image`](https://github.com/containers/image)）来管理镜像的生命周期及其与注册表的交互，并使用 *containers/storage* 项目（[`github.com/containers/storage`](https://github.com/containers/storage)）来管理镜像和容器的文件系统层。

Buildah 的高级构建策略基于对传统 Dockerfile/Containerfile 构建和由原生 Buildah 命令驱动的构建的并行支持，这些命令能够复制 Dockerfile 指令。

通过在标准命令中复制 Dockerfile 指令，Buildah 成为一个可编写脚本的工具，能够与自定义逻辑和原生 shell 构造（如条件语句、循环或环境变量）进行插值。例如，Dockerfile 中的 `RUN` 指令可以用 `buildah run` 命令替代。

如果团队需要保留之前 Dockerfile 中实现的构建逻辑，Buildah 提供了 `buildah build`（或其别名 `buildah bud`）命令，该命令通过读取提供的 Dockerfile/Containerfile 来构建镜像。

Buildah 可以平稳地以无 root 模式运行以构建镜像；从安全角度来看，这是一个宝贵且需求量大的特性。构建过程中不需要 Unix 套接字。在本章开头，我们解释了构建始终基于容器；Buildah 也不例外，所有构建都在工作容器内执行，从基础镜像开始。

以下列表提供了 Buildah 中最常用命令的非详尽描述：

+   `buildah from [options] <image>` 语法。该命令的示例是 `$ buildah from fedora`。

+   Dockerfile 的 `RUN` 指令；它在工作容器内运行命令。此命令接受 `buildah run [options] [--] <container> <command>` 语法。`--`（双破折号）选项用于将潜在的选项与实际的容器命令分开。该命令的示例是 `buildah run <containerID> -- dnf install -y nginx`。

+   `buildah config [options] <container>` 格式。此命令的选项与不修改文件系统层但设置容器元数据的各种 Dockerfile 指令相关，例如设置 `entrypoint` 容器。该命令的示例是 `buildah config --entrypoint/entrypoint.sh <containerID>`。

+   Dockerfile 的 `ADD` 指令；它将文件、目录甚至 URL 添加到容器中。它支持 `buildah add [options] <container> <src> [[src …] <dst>` 语法，并允许在一个命令中复制多个文件。该命令的示例是 `buildah add <containerID> index.php /var/www.html`。

+   `COPY` 指令；它将文件、URL 和目录添加到容器中。它支持 `buildah copy [options] <container> <src> [[src …] <dst>` 语法。该命令的示例是 `buildah copy <containerID> entrypoint.sh /`。

+   `buildah copy [options] <container> <image_name>` 语法。通过此命令创建的容器镜像可以后续标记并推送到镜像仓库。该命令的示例是 `buildah commit <containerID> <myhttpd>`。

+   `buildah build [options] [context]` 语法及 `buildah bud` 命令别名。该命令的示例是 `buildah build –t <imageName> .`。

+   `buildah ls` 和 `buildah ps`。支持的语法是 `buildah containers [options]`。该命令的示例是 `buildah containers`。

+   `buildah delete` 命令是等效的。支持的语法是 `buildah rm <container>`。此命令只有一个选项，即 `–all, -a` 选项，用于移除所有工作容器。此命令的示例如 `buildah rm <containerID>`。

+   `buildah mount [containerID … ]`。当没有传递任何参数时，命令仅显示当前已挂载的容器。此命令的示例如 `buildahmount<containerID>`。

+   `buildah images [options] [image]`。支持像 JSON 这样的自定义输出格式。此命令的示例如 `buildah images --json`。

+   `buildah tag <name> <new-name>` 格式。此命令的示例如 `buildah tag myapp quay.io/packt/myapp:latest`。

+   `buildah push [options] <image> [destination]`。此命令的示例包括 `buildah push quay.io/packt/myapp:latest`，`buildah push <imageID> docker://<URL>/repository:tag`，以及 `buildah push <imageID> oci:</path/to/dir>:image:tag`。

+   `buildah pull [options] <image>`。此命令的示例包括 `buildah pull <imageName>`，`buildah pull docker://<URL>/repository:tag`，以及 `buildah pull dir:</path/to/dir>`。

所有之前描述的命令都有相应的 `man` 页面，遵循 `man buildah-<command>` 的模式。例如，要查看 `buildah run` 命令的文档细节，只需在终端输入 `man buildah-run`。

下一个示例展示了基本的 Buildah 功能。一个 Fedora 基础镜像被定制以运行 httpd 进程：

```
$ container=$(buildah from fedora)
$ buildah run $container -- dnf install -y httpd; dnf clean all 
$ buildah config --cmd "httpd -DFOREGROUND" $container
$ buildah config --port 80 $container
$ buildah commit $container myhttpd
$ buildah tag myhttpd registry.example.com/myhttpd:v0.0.1
```

上述命令将生成一个符合 OCI 标准、可移植的镜像，具有从 Dockerfile 构建的镜像的相同功能，所有这些都可以在几行代码中实现，可以包含在一个简单的脚本中。

现在我们将重点关注第一个命令：

```
$ container=$(buildah from fedora)
```

`buildah from` 命令从允许的注册表之一拉取一个 Fedora 镜像，并从中创建一个工作容器，返回容器的名称。我们将不只是将它打印在标准输出上，而是使用 shell 扩展语法捕获名称。从现在起，我们可以将 `$container` 变量传递给后续的命令，`$container` 保存着生成的容器名称。因此，构建命令将在此工作容器内执行。这是一种相当常见的模式，特别适合在脚本中自动化 Buildah 命令。

重要说明

在 Buildah 和 Podman 中，容器的概念之间存在细微的差异。两者都采用相同的技术来创建容器，但 Buildah 容器是短生命周期的实体，旨在被修改并提交，而 Podman 容器则应该运行长期的工作负载。

这种方法的灵活性和可嵌入性是非常显著的——Buildah 命令可以包含在任何地方，用户可以选择完全自动化的构建过程，也可以选择更具互动性的构建过程。

例如，Buildah 可以轻松地与 **Ansible**（开源自动化引擎）集成，利用原生连接插件自动化构建，插件可以与工作容器进行通信。

你可以选择将 Buildah 纳入 CI 流水线（如 **Jenkins**、**Tekton** 或 **GitLab CI/CD**），以便完全控制构建和集成任务。

Buildah 也包含在云原生社区的大型项目中，例如 **Shipwright** 项目 ([`github.com/shipwright-io/build`](https://github.com/shipwright-io/build))。

Shipwright 是一个可扩展的 Kubernetes 构建框架，提供了通过自定义资源定义和不同构建工具自定义镜像构建的灵活性。Buildah 是设计构建过程时可以选择的解决方案之一。

在接下来的子章节中，我们将看到更详细、更丰富的示例。现在我们已经了解了 Buildah 的功能和使用场景概述，让我们深入了解安装和环境准备步骤。

# 准备我们的环境

Buildah 可以在不同的发行版上使用，并且可以通过相应的包管理器进行安装。本节提供了一些主要发行版的安装示例，并非详尽无遗。为了清晰起见，需要重申的是，本书实验环境均基于 Fedora 34：

+   `dnf` 命令：

    ```
    $ sudo dnf -y install buildah
    ```

+   `apt-get` 命令：

    ```
    $ sudo apt-get update
    $ sudo apt-get -y install buildah
    ```

+   `yum` 命令：

    ```
    $ sudo yum install -y buildah
    ```

+   `yum module` 命令：

    ```
    $ sudo yum module enable -y container-tools:1.0
    $ sudo yum module install -y buildah
    ```

+   `rhel-7-server-extras-rpms` 仓库并使用 `yum` 安装：

    ```
    $ sudo subscription-manager repos --enable=rhel-7-server-extras-rpms
    $ sudo yum -y install buildah
    ```

+   `pacman` 命令：

    ```
    $ sudo pacman –S buildah
    ```

+   `apt-get` 命令：

    ```
    $ sudo apt-get -y update
    $ sudo apt-get -y install buildah
    ```

+   `emerge` 命令：

    ```
    $ sudo emerge app-emulation/libpod
    ```

+   **从源代码构建**：Buildah 也可以从源代码构建。为了本书的目的，我们将专注于简单的部署方法，但如果你有兴趣，下面的指南将对你尝试自己的构建有帮助：[`github.com/containers/buildah/blob/main/install.md#building-from-scratch`](https://github.com/containers/buildah/blob/main/install.md#building-from-scratch)。

最后，Buildah 可以作为容器部署，构建任务可以在其中使用嵌套方式执行。这个过程将在 *第七章*，*与现有应用构建流程集成* 中详细讲解。

安装 Buildah 到我们的主机后，我们可以继续验证安装情况。

## 验证安装

安装 Buildah 后，我们现在可以运行一些基本的测试命令来验证安装情况。

要查看主机本地存储中所有可用的镜像，可以使用以下命令：

```
$ buildah images
# buildah images
```

镜像列表将与通过 `podman images` 命令打印的列表相同，因为它们共享相同的本地存储。

还需注意，两个命令分别作为非特权用户和 root 用户执行，分别指向用户的无特权本地存储和系统范围的本地存储。

我们可以运行一个简单的测试构建来验证安装情况。这是一个很好的机会来测试一个基本的构建脚本，其唯一目的是验证 Buildah 是否能够完全运行完整的构建。

本书的目的（以及为了好玩），我们创建了以下简单的测试脚本，用来创建一个最小的 Python 3 镜像：

```
#!/bin/bash
BASE_IMAGE=alpine
TARGET_IMAGE=python3-minimal
if [ $UID != 0 ]; then
    echo "### Running build test as unprivileged user"
else
    echo "### Running build test as root"
fi
echo "### Testing container creation"
container=$(buildah from $BASE_IMAGE)
if [ $? -ne 0 ]; then
    echo "Error initializing working container"
fi
echo "### Testing run command"
buildah run $container apk add --update python3 py3-pip
if [ $? -ne 0 ]; then
    echo "Error on run build action"
fi
echo "### Testing image commit"
buildah commit $container $TARGET_IMAGE
if [ $? -ne 0 ]; then
    echo "Error committing final image"
fi
echo "### Removing working container"
buildah rm $container
if [ $? -ne 0 ]; then
    echo "Error removing working container"
fi
echo "### Build test completed successfully!"
exit 0
```

同一个测试脚本可以由非特权用户和 root 用户执行。

我们可以通过运行一个简单的容器来验证新构建的镜像，该容器执行一个 Python shell：

```
$ podman run -it python3-minimal /usr/bin/python3
Python 3.9.5 (default, May 12 2021, 20:44:22) 
[GCC 10.3.1 20210424] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

在成功测试我们的新 Buildah 安装后，让我们检查 Buildah 使用的主要配置文件。

## Buildah 配置文件

主要的 Buildah 配置文件与 Podman 使用的配置文件相同。它们可以被利用来自定义在构建过程中执行的工作容器的行为。

在 Fedora 上，这些配置文件是通过 `containers-common` 包安装的，我们已经在*准备你的环境*部分中涵盖了它们，见 *第三章*，*运行第一个容器*。

Buildah 使用的主要配置文件如下：

+   `/usr/share/containers/mounts.conf`：此配置文件定义了在 Buildah 工作容器内自动挂载的文件和目录。

+   `/etc/containers/registries.conf`：此配置文件的作用是管理可以访问的镜像仓库，用于镜像的搜索、拉取和推送。

+   `/usr/share/containers/policy.json`：此 JSON 配置文件定义了镜像签名验证行为。

+   `/usr/share/containers/seccomp.json`：此 JSON 配置文件定义了容器化进程允许和禁止的系统调用。

在本节中，我们了解了如何准备主机环境来运行 Buildah。在下一节中，我们将识别可以通过 Buildah 实现的可能构建策略。

# 选择我们的构建策略

基本上，我们可以使用 Buildah 的三种构建策略：

+   从现有基础镜像开始构建容器镜像

+   从零开始构建容器镜像

+   从 Dockerfile 开始构建容器镜像

我们已经在*遇见 Buildah，Podman 的伴侣*部分提供了一个从现有基础镜像构建策略的示例。由于从工作流的角度来看，这种策略与从零开始构建非常相似，因此我们将重点展示最后一种策略，它提供了极大的灵活性来创建小体积且安全的镜像。

在下一节中深入了解各种技术细节之前，让我们从高层次开始探索所有这些策略。

尽管我们可以在最流行的公共容器仓库中找到许多预构建的容器镜像，有时我们可能找不到特定的配置、设置或工具和服务组合用于我们的容器；这就是为什么容器镜像创建成为我们需要实践的一个非常重要的步骤。

此外，安全约束通常要求我们实现具有减少攻击面特性的镜像，因此，DevOps 团队必须知道如何自定义构建过程中的每一步，以实现这一结果。

牢记这些知识后，我们开始介绍第一种构建策略。

## 从现有基础镜像开始构建容器镜像

假设我们找到一个非常出色的预构建容器镜像，用于我们公司广泛使用的某个应用服务器。这个容器镜像的所有配置都很合适，我们可以将存储附加到正确的挂载点以持久化数据等等，但迟早我们会意识到，容器镜像中缺少我们用于故障排除的某些特定工具，或者缺少某些应该包含的库！

在另一种场景中，我们可能对预构建的镜像感到满意，但仍然需要向其中添加自定义内容——例如，客户的应用程序。

在这些情况下，解决方案会是什么呢？

在这个第一个用例中，我们可以扩展现有的容器镜像，添加内容并编辑现有的文件以满足我们的目的。在前面的基本示例中，Fedora 和 Alpine 镜像被定制化以满足不同的需求。这些镜像是通用的操作系统文件系统，没有特定的用途，但同样的概念也可以应用于更复杂的镜像。

在第二个用例中，我们可以定制一个镜像——例如，默认库 Httpd。我们可以安装 PHP 模块，然后添加我们应用程序的 PHP 文件，生成一个包含我们自定义内容的新镜像。

接下来的章节中，我们将看到如何扩展现有的容器镜像。

让我们继续讨论第二种策略。

## 从零开始构建容器镜像

前面的策略对于许多常见情况已经足够，在这些情况下我们可以找到一个预构建的镜像来开始工作，但有时我们希望容器化的特定用例、应用或服务可能并不常见或广泛使用。

想象一下有一个自定义的遗留应用程序，需要一些旧的库和工具，而这些库和工具在最新的 Linux 发行版中已经不再包含，或者可能已被更新的版本所替代。在这种情况下，您可能需要从一个空的容器镜像开始，并一点点地添加所有必要的内容，以便运行您的遗留应用程序。

我们在这一章中学到，实际上，我们总是会从某种初始容器镜像开始，所以这个策略和前面的那个基本是相同的。

让我们继续讨论第三种也是最后一种策略。

## 从 Dockerfile 开始构建容器镜像

在*第一章*《容器技术简介》中，我们讨论了容器技术的历史以及 Docker 如何在那个背景下获得动力。Podman 作为 Docker 帮助开发的伟大概念的替代进化项目诞生了。Docker 在其项目历史中所创造的伟大创新之一，毫无疑问，就是 Dockerfile。

从高层次来看，使用这种策略时，我们可以肯定，即使使用 Dockerfile，我们最终也会采用之前提到的某种构建策略。实际上，情况与我们之前的假设不谋而合，因为 Buildah 在后台会解析 Dockerfile，并构建我们之前简单介绍的容器，符合之前构建策略的要求。

总结一下，当选择默认的构建策略时，我们需要考虑是否有任何差异或优势？显然，这个问题没有最终答案。首先，我们应该始终查看容器社区，寻找一些可能帮助我们*构建*过程的预构建镜像；另一方面，我们总是可以回退到*从头构建*的过程。最后但同样重要的是，我们可以考虑使用 Dockerfile 来轻松地分发和共享我们的构建步骤，给我们的开发团队或更广泛的容器社区。

这就结束了我们快速的高层次介绍；现在我们可以继续实际的示例！

# 从头构建镜像

在进入本节的详细内容并学习如何从头构建容器镜像之前，让我们先做一些测试，以验证安装的 Buildah 是否正常工作。

首先，让我们检查我们的 Buildah 镜像缓存是否为空：

```
# buildah images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
# buildah containers -a
CONTAINER ID  BUILDER  IMAGE ID     IMAGE NAME                       CONTAINER NAME
```

重要提示

Podman 和 Buildah 共享相同的容器存储；因此，如果你之前运行过本章或本书中的其他示例，你会发现你的容器存储缓存并不是那么空！

正如我们在上一节中学到的，我们可以利用 Buildah 会输出刚创建的工作容器的名称这一事实，将其轻松存储在环境变量中，并在需要时使用它。让我们从头创建一个全新的容器：

```
# buildah from scratch
# buildah images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
# buildah containers
CONTAINER ID  BUILDER  IMAGE ID     IMAGE NAME                       CONTAINER NAME
af69b9547db9     *                  scratch                          working-container
```

如你所见，我们使用了特殊的`from scratch`关键字，它告诉 Buildah 创建一个没有任何数据的空容器。如果我们运行`buildah images`命令，我们会注意到这个特殊镜像没有被列出。

让我们检查容器是否真的为空：

```
# buildah run working-container bash
2021-10-26T20:15:49.000397390Z: executable file 'bash' not found in $PATH: No such file or directory
error running container: error from crun creating container for [bash]: : exit status 1
error while running runtime: exit status 1
```

在我们的空容器中没有找到可执行文件——真是一个惊喜！原因是工作容器是基于一个空的文件系统创建的。

让我们看看如何轻松填充这个空容器。在以下示例中，我们将直接与底层存储交互，使用主机系统的包管理器安装运行`bash` shell 所需的二进制文件和库，以构建我们的容器镜像。

首先，让我们指示 Buildah 挂载容器存储，并检查它的位置：

```
# buildah mount working-container
/var/lib/containers/storage/overlay/b5034cc80252b6f4af2155f 9e0a2a7e65b77dadec7217bd2442084b1f4449c1a/merged
```

了解一下

如果你在无根模式下开始构建，Buildah 会在不同的命名空间中运行挂载，因此，使用除 vfs 以外的驱动程序时，挂载的卷可能无法从主机访问。

太好了！现在我们已经找到了它，我们可以利用主机的包管理器将所有需要的包安装到这个`root`文件夹中，这将是我们容器镜像的`root`路径：

```
# scratchmount=$(buildah mount working-container)
# dnf install --installroot $scratchmount --releasever 34 bash coreutils --setopt install_weak_deps=false -y
```

重要提示

如果你在不同于版本 34 的 Fedora 版本上运行之前的命令（例如，版本 35），则需要导入 Fedora 34 的 GPG 公钥，或使用 `--nogpgcheck` 选项。

首先，我们将把非常长的目录路径保存在环境变量中，然后执行 `dnf` 包管理器，将刚刚获取的目录路径作为安装根目录，设置我们 Fedora 操作系统的发布版本，指定我们要安装的包（`bash` 和 `coreutils`），最后禁用弱依赖，接受所有对系统的更改。

命令应该以 `Complete!` 语句结束；完成后，让我们再次尝试之前在本节中看到的失败命令：

```
# buildah run working-container bash
bash-5.1# cat /etc/fedora-release
Fedora release 34 (Thirty Four)
```

成功了！我们刚刚在空容器中安装了一个 Bash shell。现在，让我们看看如何通过其他配置步骤完成镜像创建。首先，我们需要向最终的容器镜像中添加一个命令，一旦容器启动并运行时就会执行。因此，我们将创建一个 Bash 脚本文件，其中包含一些基本命令：

```
# cat command.sh 
#!/bin/bash
cat /etc/fedora-release
/usr/bin/date
```

我们创建了一个 Bash 脚本文件，打印容器的 Fedora 版本和系统日期。该文件在复制之前必须具有执行权限：

```
# chmod +x command.sh
```

现在，我们已经为基础容器存储填充了所有需要的基础包，我们可以卸载 `working-container` 存储并使用 `buildah copy` 命令将文件从主机注入到容器中：

```
# buildah unmount working-container
af69b9547db93a7dc09b96a39bf5f7bc614a7ebd29435205d358e09ac 99857bc
# buildah copy working-container ./command.sh /usr/bin
659a229354bdef3f9104208d5812c51a77b2377afa5ac819e3c3a1a2887eb9f7
```

`buildah copy` 命令使我们能够在不担心挂载它或在后台处理它的情况下，直接操作底层存储。

现在我们准备通过向容器镜像中添加一些元数据来完成它：

```
# buildah config --cmd /usr/bin/command.sh working-container
# buildah config --created-by "podman book example" working-container
# buildah config --label name=fedora-date working-container
```

我们从 `cmd` 选项开始，之后添加了一些描述性元数据。现在我们终于可以将 `working-container` 提交为镜像了！

```
# buildah commit working-container fedora-date
Getting image source signatures
Copying blob 939ac17066d4 done  
Copying config e24a2fafde done  
Writing manifest to image destination
Storing signatures
e24a2fafdeb5658992dcea9903f0640631ac444271ed716d7f749eea7a651487
```

让我们清理环境并检查主机中可用的容器镜像：

```
# buildah rm working-container
af69b9547db93a7dc09b96a39bf5f7bc614a7ebd29435205d358e09ac99857bc
```

我们现在可以检查刚刚创建的容器镜像的详细信息：

```
# podman images
REPOSITORY             TAG         IMAGE ID      CREATED             SIZE
localhost/fedora-date  latest      e24a2fafdeb5  About a minute ago  366 MB
# podman inspect localhost/fedora-date:latest
[...omitted output]        "Labels": {
            "io.buildah.version": "1.23.1",
            "name": "fedora-date"
        },
        "Annotations": {
            "org.opencontainers.image.base.digest": "",
            "org.opencontainers.image.base.name": ""
        },
        "ManifestType": "application/vnd.oci.image.manifest.v1+json",
        "User": "",
        "History": [
            {
                "created": "2021-10-26T21:16:48.777712056Z",
                "created_by": "podman book example"
            }
        ],
        "NamesHistory": [
            "localhost/fedora-date:latest"
        ]
    }
]
```

从之前的输出中可以看出，容器镜像有很多元数据，可以告诉我们许多细节。我们通过之前的命令设置了其中的一些，例如 `created_by`、`name` 和 `Cmd` 标签；其他标签则由 Buildah 自动填充。

最后，让我们用 Podman 运行我们全新的容器镜像！

```
# podman run -ti localhost/fedora-date:latest 
Fedora release 34 (Thirty Four)
Tue Oct 26 21:18:29 UTC 2021
```

这结束了我们从零开始创建容器镜像的过程。正如我们所看到的，这不是创建容器镜像的典型方法；在许多场景和各种用例中，从一个操作系统基础镜像（例如 `from fedora` 或 `from alpine`）开始，然后添加所需的包，使用这些镜像中可用的包管理器就足够了。

需要了解

一些 Linux 发行版还提供 **最小化** 版本的基础容器镜像（例如 **fedora-minimal**），它们减少了安装的包的数量以及目标容器镜像的大小。有关更多信息，请参考 [`www.docker.com/`](https://www.docker.com/) 和 [`quay.io/`](https://quay.io/)！

现在让我们检查如何使用 Buildah 从 Dockerfile 构建镜像。

# 从 Dockerfile 构建镜像

如我们在本章前面描述的那样，Dockerfile 可以是创建和共享容器镜像构建步骤的简便选项，因此在互联网上很容易找到许多源 Dockerfile。

这项活动的第一步是构建一个简单的 Dockerfile 来使用。让我们创建一个 Dockerfile 来创建一个容器化的 Web 服务器：

```
# mkdir webserver
# cd webserver/
[webserver]# vi Dockerfile 
[webserver]# cat Dockerfile
# Start from latest fedora container base image
FROM fedora:latest
MAINTAINER podman-book  # this should be an email
# Update the container base image
RUN echo "Updating all fedora packages"; dnf -y update; dnf -y clean all
# Install the httpd package
RUN echo "Installing httpd"; dnf -y install httpd
# Expose the http port 80
EXPOSE 80
# Set the default command to run once the container will be started
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

从前面的输出来看，我们首先创建了一个新目录，并在其中创建了一个名为 `Dockerfile` 的文本文件。然后，我们插入了在定义全新 Dockerfile 时常用的各种关键字和步骤；每个步骤和关键字上方都有专门的描述注释，因此该文件应该容易阅读。

让我们回顾一下，这些是我们全新 Dockerfile 中包含的步骤：

1.  从最新的 Fedora 容器基础镜像开始。

1.  更新容器基础镜像中的所有包。

1.  安装 `httpd` 包。

1.  暴露 HTTP 端口 `80`。

1.  设置容器启动后要运行的默认命令。

如本章前面所述，Buildah 提供了一个专用的 `buildah build` 命令来从 Dockerfile 开始构建。

让我们看看它是如何工作的：

```
[webserver]# buildah build -f Dockerfile -t myhttpdservice .
STEP 1/6: FROM fedora:latest
Resolved "fedora" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull registry.fedoraproject.org/fedora:latest...
Getting image source signatures
Copying blob 944c4b241113 done  
Copying config 191682d672 done  
Writing manifest to image destination
Storing signatures
STEP 2/6: MAINTAINER podman-book  # this should be an email
STEP 3/6: RUN echo "Updating all fedora packages"; dnf -y update; dnf -y clean all
Updating all fedora packages
Fedora 34 - x86_64                               16 MB/s |  74 MB     00:04  
...
STEP 4/6: RUN echo "Installing httpd"; dnf -y install httpd
Installing httpd
Fedora 34 - x86_64                               20 MB/s |  74 MB     00:03    
...
STEP 5/6: EXPOSE 80
STEP 6/6: CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
COMMIT myhttpdservice
Getting image source signatures
Copying blob 7500ce202ad6 skipped: already exists  
Copying blob 51b52d291273 done  
Copying config 14a2226710 done  
Writing manifest to image destination
Storing signatures
--> 14a2226710e
Successfully tagged localhost/myhttpdservice:latest
14a2226710e7e18d2e4b6478e09a9f55e60e0666dd8243322402ecf6fd1eaa0d
```

从之前的输出中我们可以看到，我们将以下选项传递给 `buildah build` 命令：

+   `-f`：定义 Dockerfile 的名称。默认的文件名是 `Dockerfile`，所以在我们的例子中，我们可以省略此选项，因为我们将文件命名为默认名称。

+   `-t`：定义我们正在构建的镜像的名称和标签。在我们的例子中，我们只定义了名称。镜像将默认被标记为`latest`。

+   最后，作为最后一个选项，我们需要设置 Buildah 工作的目录并查找 Dockerfile。在我们的例子中，我们传递了当前的 `.` 目录。

当然，这些并不是 Buildah 给我们配置构建的唯一选项；稍后我们将在本节中看到其中的一些选项。

回到我们刚刚执行的命令，从输出中可以看到，Dockerfile 中定义的所有步骤都按书写的顺序执行，并且以一个给定的分数编号打印，显示中间步骤与总步骤的比例。总共执行了六个步骤。

我们可以通过使用 `buildah images` 命令列出镜像，来检查命令的执行结果：

```
[webserver]# buildah images
REPOSITORY                                  TAG      IMAGE ID       CREATED          SIZE
localhost/myhttpdservice                    latest   14a2226710e7   2 minutes ago   497 MB
```

如我们所见，我们的容器镜像已经使用 `latest` 标签创建；让我们尝试运行它：

```
# podman run -d localhost/myhttpdservice:latest
133584ab526faaf7af958da590e14dd533256b60c10f08acba6c1209ca05a885
# podman logs 133584ab526faaf7af958da590e14dd533256b60c10f08acba6c1209ca05a885
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.4\. Set the 'ServerName' directive globally to suppress this message
# curl 10.88.0.4
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>Test Page for the HTTP Server on Fedora</title>
    <style type="text/css">
...
```

从输出中可以看到，我们刚刚以分离模式运行了容器；之后，我们检查了日志，以找出需要作为`curl`测试命令参数传递的 IP 地址。

我们刚刚以 root 用户在工作站上运行了容器，容器在 Podman 的容器网络接口上分配了一个内部 IP 地址。我们可以通过运行以下命令来检查该 IP 地址是否属于该网络：

```
# ip a show dev cni-podman0
14: cni-podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether c6:bc:ba:7c:d3:0c brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global cni-podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::c4bc:baff:fe7c:d30c/64 scope link 
       valid_lft forever preferred_lft forever
```

如我们所见，容器的 IP 地址来自前面`10.88.0.1/16`输出中的网络。

如我们所预期，`buildah build`命令有许多其他选项，在开发和创建全新容器镜像时可能非常有用。让我们来探讨其中一个值得一提的选项——`--layers`。

我们已经在本章之前学习了如何与 Podman 一起使用这个选项。从 Buildah 的 1.2 版本开始，开发团队添加了这个非常棒的选项，使我们能够启用或禁用层的缓存机制。默认配置将`--layers`选项设置为*false*，这意味着 Buildah 不会保留中间层，导致构建将所有更改合并为单一层。

也可以通过环境变量来设置层的管理——例如，要启用层缓存，可以运行`export BUILDAH_LAYERS=true`。

显然，这种选项的缺点是保留的层实际上会占用系统主机的存储空间，但另一方面，如果我们需要重新构建某个镜像，只更改最新的层而不重建整个镜像，我们就可以节省计算能力！

# 总结

在本章中，我们探讨了容器管理的一个基本主题——它们的创建。如果我们希望定制、保持更新并正确管理容器基础设施，这一步是必须的。我们了解到，Podman 通常与另一个名为 Buildah 的工具搭配使用，Buildah 可以帮助我们进行容器镜像构建的过程。这个工具有很多选项，像 Podman 一样，并且与 Podman 共享许多选项（包括存储！）。最后，我们介绍了 Buildah 提供的构建新容器镜像的不同策略，其中一个实际上是 Docker 生态系统继承的——Dockerfile。

本章仅是容器镜像构建主题的介绍；我们将在下一章中探索更高级的技术！

# 深入阅读

+   Buildah 项目教程：[`github.com/containers/buildah/tree/main/docs/tutorials`](https://github.com/containers/buildah/tree/main/docs/tutorials)

+   如何在容器内使用 Podman：[`www.redhat.com/sysadmin/podman-inside-container`](https://www.redhat.com/sysadmin/podman-inside-container)

+   如何构建小型容器镜像：[`www.redhat.com/sysadmin/tiny-containers`](https://www.redhat.com/sysadmin/tiny-containers)
