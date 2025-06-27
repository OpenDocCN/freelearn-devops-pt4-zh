# *第十章*：《故障排除和监控容器》

运行容器可能会被误认为是 DevOps 团队的最终目标，但实际上，这只是漫长旅程的第一步。系统管理员应该确保他们的系统正常运行，以保持服务的持续运行；同样，DevOps 团队应确保他们的容器正常工作。

在容器管理活动中，拥有正确的故障排除技术知识确实能帮助减少对最终服务的影响，减少停机时间。谈到问题和故障排除，一个好的做法是持续监控容器，以便轻松发现任何问题或错误，加快恢复速度。

本章将涵盖以下主要内容：

+   故障排除正在运行的容器

+   通过健康检查监控容器

+   检查我们的容器构建结果

+   使用`nsenter`进行高级故障排除

# 技术要求

在继续介绍章节信息和示例之前，需要一台已经安装并正常运行 Podman 的机器。正如在*第三章*《运行第一个容器》中所述，本书中的所有示例都在 Fedora 34 或更高版本的系统上执行，但也可以在你选择的操作系统上复现。

对*第四章*《管理运行中的容器》和*第五章*《实现容器数据存储》中的内容有一个良好的理解，将有助于轻松掌握与容器注册中心相关的概念。

# 故障排除正在运行的容器

故障排除容器是一个重要的实践，我们需要通过经验解决常见问题并调查在容器层或容器内部运行的应用程序中可能遇到的任何 bug。

从*第三章*《运行第一个容器》开始，我们已开始使用基本的 Podman 命令来运行和检查主机系统上的容器。我们看到了如何使用`podman logs`命令收集日志，我们还学习了如何使用`podman inspect`命令提供的信息。最后，我们还应考虑查看有用的`podman system df`命令的输出，该命令会报告容器和镜像的存储使用情况，还会显示有用的`podman system info`命令的输出，显示有关运行 Podman 的主机的有用信息。

通常，我们应始终考虑运行中的容器只是主机系统上的一个进程，因此我们始终可以使用所有用于故障排除底层操作系统及其可用资源的工具和命令。

故障排除容器的最佳实践可以采用自上而下的方法，首先分析应用层，然后转到容器层，最终到达基础主机系统。

在容器级别，我们可能遇到的许多问题已被 Podman 项目团队在项目页面上总结成一个综合列表。我们将在接下来的章节中介绍一些更有用的问题。

## 使用存储卷时权限被拒绝

我们在 RHEL、Fedora 或任何使用 SELinux 安全子系统的 Linux 发行版上，可能遇到的一个非常常见的问题与存储权限有关。以下描述的错误是在 SELinux 设置为 `Enforcing` 模式时触发的，这也是建议的做法，以完全保障 SELinux 强制访问安全特性。

我们可以尝试在 Fedora 工作站上测试这一点，首先创建一个目录，然后尝试将其用作容器中的卷：

```
$ mkdir ~/mycontent
$ podman run -v ~/mycontent:/content fedora \ 
touch /content/file
touch: cannot touch '/content/file': Permission denied
```

正如我们所见，`touch` 命令报告了 `Permission denied` 错误，因为实际上，它无法在文件系统中写入。

正如我们在*第五章*《实现容器数据存储》中详细看到的，SELinux 会递归地为文件和目录应用标签，以定义它们的上下文。这些标签通常存储为扩展文件系统属性。SELinux 使用上下文来管理策略并定义哪些进程可以访问特定的资源。

我们刚刚运行的容器有自己的 Linux 命名空间和一个与 Fedora 工作站上的本地用户完全不同的 SELinux 标签，这也是我们之前实际遇到错误的原因。

如果没有正确的标签，SELinux 系统会阻止容器中运行的进程访问内容。这是因为如果没有通过命令选项明确请求，Podman 不会更改操作系统设置的标签。

要让 Podman 更改容器的标签，我们可以在卷挂载时使用 `:z` 或 `:Z` 两个后缀中的任意一个。这些选项告诉 Podman 对卷上的文件对象重新标记。

`:z` 选项用于指示 Podman 两个容器共享一个存储卷。因此，在这种情况下，Podman 会为内容打上共享内容标签，允许两个或更多容器在该卷上读取/写入内容。

`:Z` 选项用于指示 Podman 将卷的内容标记为私有的、非共享的标签，仅当前容器可以使用。

该命令的结果将类似于以下内容：

```
$ podman run -v ~/mycontent:/content:Z fedora \ 
touch /content/file
```

正如我们所见，命令没有报告任何错误；它执行成功了。

## 在无 root 容器中使用 ping 命令的问题

在某些强化的 Linux 系统上，`ping` 命令的执行可能仅限于一组受限用户。这可能导致容器中使用 `ping` 命令失败。

正如我们在 *第三章* 中看到的，*运行第一个容器*，启动容器时，基础操作系统会将一个与容器内使用的用户 ID 不同的用户 ID 关联到容器上。与容器关联的用户 ID 可能会超出允许使用 `ping` 命令的用户 ID 范围。

在 Fedora 工作站的安装中，默认配置将允许任何容器运行 `ping` 命令而不受限制。为了管理 `ping` 命令的使用限制，Fedora 使用 `ping_group_range` 内核参数，该参数定义了允许执行 `ping` 命令的系统组。

如果我们查看一个刚安装的 Fedora 工作站，默认范围如下：

```
$ cat /proc/sys/net/ipv4/ping_group_range
0      2147483647
```

所以，对于一个全新的 Fedora 系统，不必担心。但如果范围比这个小呢？

好的，我们通过一个简单的命令来更改允许的范围来测试这种行为。在这个示例中，我们将限制范围并看到 `ping` 命令实际会失败：

```
$ sudo sysctl -w "net.ipv4.ping_group_range=0 0"
```

以防范围小于前面输出中报告的范围，我们可以通过将一个包含 `net.ipv4.ping_group_range=0 0` 的文件添加到 `/etc/sysctl.d` 来使其持久化。

对 `ping` 组范围所做的更改将影响映射到容器内运行 `ping` 命令的用户权限。

让我们开始使用 Buildah 构建一个基于 Fedora 的镜像，并包含 `iputils` 包（默认未包含）：

```
$ container=$(buildah from docker.io/library/fedora) && \
  buildah run $container -- dnf install -y iputils && \
  buildah commit $container ping_example
```

我们可以通过在容器内运行以下命令来进行测试：

```
$ podman run --rm ping_example ping -W10 -c1 redhat.com
PING redhat.com (209.132.183.105): 56 data bytes
--- redhat.com ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
```

在一个限制范围的系统上执行该命令时，因 `ping` 命令无法通过原始套接字发送数据包，因此会产生 100% 的数据包丢失。

该示例演示了 `ping_group_range` 限制如何影响无根容器内 `ping` 的执行。通过将范围设置为足够大的值以包含用户私有组 GID（或用户的某个次要组），`ping` 命令将能够正确发送 ICMP 数据包。

重要提示

在继续进行下一个示例之前，请不要忘记恢复原始的 `ping_group_range`。在 Fedora 中，默认配置可以通过 `sudo sysctl -w "net.ipv4.ping_group_range=0 2147483647"` 命令恢复，并通过删除在练习过程中应用于 `/etc/sysctl.d` 下的任何持久配置来恢复。对于我们通过 Dockerfile 构建的基础容器镜像，可能需要添加一个新的用户，并为其分配较大的 UID/GID。这将导致创建一个较大且稀疏的 `/var/log/lastlog` 文件，并可能导致构建永远挂起。这个问题与 Go 语言相关，因为 Go 语言不正确地支持稀疏文件，导致在容器镜像中创建了这个巨大的文件。

需要知道的事项

`/var/log/lastlog` 文件是一个二进制稀疏文件，包含有关用户上次登录系统的时间的信息。通过 `ls -l` 查看时，稀疏文件的显示大小大于实际磁盘使用量。稀疏文件通过将表示空块的元数据写入磁盘，而不是将空闲空间存储在块中，从而更加高效地使用文件系统空间。这将节省更多的磁盘空间。

如果我们需要为基础容器镜像添加一个 UID 数字较高的新用户，最好的方法是将 `--no-log-init` 选项附加到 Dockerfile 命令中，如下所示：

```
RUN useradd --no-log-init -u 99999000 -g users myuser
```

此选项指示 `useradd` 命令停止创建 lastlog 文件，从而解决我们可能遇到的问题。

正如本节前几段所提到的，Podman 团队创建了一个很长但不全面的常见问题列表。如果遇到任何问题，我们强烈建议查看该列表：[`github.com/containers/podman/blob/main/troubleshooting.md`](https://github.com/containers/podman/blob/main/troubleshooting.md)。

排查问题可能会有些棘手，但第一步始终是识别问题。因此，监控工具可以在出现问题时尽早发出警报。接下来我们将看看如何使用健康检查监控容器。

# 监控具有健康检查的容器

从 1.2 版本开始，Podman 支持为容器添加健康检查选项。在本节中，我们将深入探讨这些健康检查以及如何使用它们。

健康检查是 Podman 的一个功能，它可以帮助判断容器中运行的进程的健康状况或准备状态。它可以像检查容器进程是否在运行一样简单，也可以更复杂，比如使用网络连接等方式验证容器及其应用是否响应。

健康检查由五个核心组件组成。第一个是主要元素，它指示 Podman 执行特定的检查；其他组件用于配置健康检查的调度。让我们详细了解这些元素：

+   `0`）或失败（其他退出代码）。

如果我们的容器提供一个 web 服务器，例如，我们的健康检查命令可以是一个非常简单的命令，比如 `curl` 命令，尝试连接到 web 服务器的端口，确保它是响应的。

+   **重试次数**：此选项定义了 Podman 必须执行的连续失败命令数量，容器才会被标记为不健康。如果某个命令成功执行，Podman 会重置重试计数器。

+   **间隔时间**：此选项定义了 Podman 执行健康检查命令的时间间隔。

找到合适的间隔时间可能非常困难，并且需要一些反复试验。如果我们设置为一个较小的值，那么系统可能会花费大量时间进行健康检查。但如果我们设置为较大的值，可能会出现超时问题。这个值可以使用广泛使用的时间格式来定义：`30s`或`1h5m`。

+   **启动周期**：这描述了健康检查将由 Podman 启动的时间，在此期间，Podman 将忽略健康检查失败。

我们可以将其视为一个宽限期，用于让我们的应用成功启动并开始正确地响应任何客户端请求以及健康检查。

+   **超时**：这定义了健康检查本身必须完成的时间段，超过该时间段即视为检查失败。

让我们看一个实际的例子，假设我们要为一个容器定义一个健康检查，并手动运行该健康检查：

```
$ podman run -dt --name healthtest1 --healthcheck-command \
'CMD-SHELL curl http://localhost || exit 1' \
--healthcheck-interval=0 quay.io/libpod/alpine_nginx:latest
Trying to pull quay.io/libpod/alpine_nginx:latest...
Getting image source signatures
Copying blob ac35fae19c6c done  
Copying blob 4c0d98bf9879 done  
Copying blob 5b0fccc9c35f done  
Copying config 7397e078c6 done  
Writing manifest to image destination
Storing signatures
1faae6c46839b9076f68bee467f9d56751db6ab45dd149f249b0790e05 c55b58
$ podman healthcheck run healthtest1
$ echo $?
0
```

从之前的代码块中可以看到，我们刚刚启动了一个名为`checktest1`的全新容器，定义了一个`healthcheck-command`，该命令将在目标容器内的`localhost`地址上运行`curl`命令。容器启动后，我们手动运行了`healthcheck`并验证退出代码为`0`，意味着检查成功完成，并且我们的容器是健康的。在之前的示例中，我们还使用了`--healthcheck-interval=0`选项，实际上禁用了运行间隔并将健康检查设置为手动。

Podman 使用`cron`来调度健康检查，但这些应该手动设置。

让我们通过创建一个具有间隔时间的健康检查来检查系统与 systemd 的自动集成是如何工作的：

```
$ podman run -dt --name healthtest2 --healthcheck-command 'CMD-SHELL curl http://localhost || exit 1' --healthcheck-interval=10s quay.io/libpod/alpine_nginx:latest
70e7d3f0b4363759fc66ae4903625e5f451d3af6795a96586bc1328c1b149 ce5
$ podman ps
CONTAINER ID  IMAGE                               COMMAND               CREATED        STATUS                      PORTS       NAMES
70e7d3f0b436  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  7 seconds ago  Up 7 seconds ago (healthy)              healthtest2
```

从之前的代码块中可以看到，我们刚刚启动了一个名为`checktest2`的全新容器，定义了与前面示例相同的`healthcheck-command`，但现在指定了`--healthcheck-interval=10s`选项，实际上将检查任务每 10 秒调度一次。

在执行`podman run`命令之后，我们还运行了`podman ps`命令，实际上检查健康检查是否正常工作，正如我们在输出中看到的那样，我们为新容器显示了`healthy`状态。

但这个集成是如何工作的呢？让我们获取容器 ID，并在以下目录中搜索它：

```
$ ls /run/user/$UID/systemd/transient/70e*
/run/user/1000/systemd/transient/70e7d3f0b4363759fc66ae4903625 e5f451d3af6795a96586bc1328c1b149ce5.service
/run/user/1000/systemd/transient/70e7d3f0b4363759fc66ae4903625 e5f451d3af6795a96586bc1328c1b149ce5.timer
```

之前代码块中显示的目录包含了当前用户使用的所有 systemd 资源。特别地，我们查看了`transient`目录，它存放了当前用户的临时单元文件。

当我们启动一个带有健康检查和调度间隔的容器时，Podman 将执行一个临时设置的 systemd 服务和计时器单元文件。这意味着这些单元文件不是永久性的，并且在重启时可能会丢失。

让我们检查一下这些文件中定义的内容：

```
$ cat /run/user/$UID/systemd/transient/70e7d3f0b4363759fc66a e4903625e5f451d3af6795a96586bc1328c1b149ce5.service
# This is a transient unit file, created programmatically via the systemd API. Do not edit.
[Unit]
Description=/usr/bin/podman healthcheck run 70e7d3f0b4363759 fc66ae4903625e5f451d3af6795a96586bc1328c1b149ce5

[Service]
Environment="PATH=/home/alex/.local/bin:/home/alex/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/var/lib/snapd/snap/bin"
ExecStart=
ExecStart="/usr/bin/podman" "healthcheck" "run" "70e7d3f0b 4363759fc66ae4903625e5f451d3af6795a96586bc1328c1b149ce5"
$ cat /run/user/$UID/systemd/transient/70e7d3f0b4363759fc66 ae4903625e5f451d3af6795a96586bc1328c1b149ce5.timer
# This is a transient unit file, created programmatically via the systemd API. Do not edit.
[Unit]
Description=/usr/bin/podman healthcheck run 70e7d3f0b4363759 fc66ae4903625e5f451d3af6795a96586bc1328c1b149ce5

[Timer]
OnUnitInactiveSec=10s
AccuracySec=1s
RemainAfterElapse=no
```

从之前的代码块中可以看到，服务单元文件包含了 Podman 健康检查命令，而计时器单元文件定义了调度间隔。

最后，如果我们想快速识别健康或不健康的容器，可以使用以下命令快速输出它们：

```
$ podman ps -a --filter health=healthy
CONTAINER ID  IMAGE                               COMMAND               CREATED         STATUS                                 PORTS       NAMES
1faae6c46839  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  36 minutes ago  Exited (137) 19 minutes ago (healthy)              healthtest1
70e7d3f0b436  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  13 minutes ago  Up 13 minutes ago (healthy)                        healthtest2
```

在这个示例中，我们使用了 `--filter health=healthy` 选项，通过 `podman ps` 命令仅显示健康的容器。

我们在前面的章节中学会了如何排查和监控容器，但容器构建过程又该如何处理呢？让我们在接下来的章节中更深入地了解容器构建检查。

# 检查容器构建结果

在前面的章节中，我们详细讨论了容器构建过程，并学习了如何使用 Dockerfile/Containerfile 或 Buildah 原生命令创建自定义镜像。我们还展示了第二种方法如何帮助更好地控制构建工作流程。

本节帮助提供了一些最佳实践，用于检查构建结果并理解可能相关的问题。

## 排查来自 Dockerfile 的构建问题

当使用 Podman 或 Buildah 根据 Dockerfile/Containerfile 运行构建时，构建过程会将所有指令的输出和相关错误打印到终端标准输出。对于所有 `RUN` 指令，执行命令时产生的错误会被传播并打印出来以供调试。

现在让我们尝试测试一些潜在的构建问题。这不是一个详尽无遗的错误清单；其目的是提供一种分析根本原因的方法。

第一个示例展示了一个最小构建，其中由于执行的命令出现错误，`RUN` 指令失败。`RUN` 指令中的错误可能涵盖广泛的情况，但一般的经验法则是：执行的命令返回一个退出码，如果该退出码非零，构建失败，错误和退出状态会被打印出来。

在下一个示例中，我们使用 `yum` 命令安装 `httpd` 包，但我们故意在包名中打了一个错别字，以产生一个错误。下面是 Dockerfile 的记录：

Chapter10/RUN_command_error/Dockerfile

```
FROM registry.access.redhat.com/ubi8
# Update image and install httpd
RUN yum install -y htpd && yum clean all –y
# Expose the default httpd port 80
EXPOSE 80
# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

如果我们尝试执行该命令，将会遇到一个错误，错误由 `yum` 命令无法找到缺失的 `httpd` 包生成：

```
$ buildah build -t custom_httpd .
STEP 1/4: FROM registry.access.redhat.com/ubi8
STEP 2/4: RUN yum install -y htpd && yum clean all –y
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered with an entitlement server. You can use subscription-manager to register.
Red Hat Universal Base Image 8 (RPMs) - BaseOS  3.9 MB/s | 796kB     00:00    
Red Hat Universal Base Image 8 (RPMs) - AppStre 6.2 MB/s | 2.6 MB     00:00    
Red Hat Universal Base Image 8 (RPMs) - CodeRea 171 kB/s |  16 kB     00:00    
No match for argument: htpd
Error: Unable to find a match: htpd
error building at STEP "RUN yum install -y htpd && yum clean all -y": error while running runtime: exit status 1
ERRO[0004] exit status 1                   
```

前两行打印出由 `yum` 命令生成的错误信息，类似于标准命令行环境中的输出。

接下来，Buildah（同样，Podman）会产生一条信息，告知我们产生错误的步骤。此信息由 `imagebuildah` 包中的阶段执行器管理，正如其名称所示，阶段执行器处理构建阶段的执行及其状态。源代码可以在 Buildah 的 GitHub 仓库中查看：[`github.com/containers/buildah/blob/main/imagebuildah/stage_executor.go`](https://github.com/containers/buildah/blob/main/imagebuildah/stage_executor.go)。

该信息包含 Dockerfile 指令和生成的错误，以及退出状态。

最后一行包括`ERRO[0004]`错误代码和与`buildah`命令执行相关的最终退出状态`1`。

使用`RUN`指令包含出错的命令，并修复或排查该命令错误。

构建过程中的另一个常见失败原因是缺少父镜像。这可能与拼写错误的仓库名称、缺失的标签或无法访问的镜像仓库相关。

下一个示例展示了前一个 Dockerfile 的另一种变体，其中镜像仓库名称拼写错误，因此在远程仓库中找不到该镜像：

Chapter10/FROM_repo_not_found/Dockerfile

```
FROM registry.access.redhat.com/ubi_8
# Update image and install httpd
RUN yum install -y httpd && yum clean all –y
# Expose the default httpd port 80
EXPOSE 80
# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

当从这个 Dockerfile 进行构建时，我们将遇到由于缺少镜像仓库而导致的错误，如下一个示例所示：

```
$ buildah build -t custom_httpd .
STEP 1/4: FROM registry.access.redhat.com/ubi_8
Trying to pull registry.access.redhat.com/ubi_8:latest...
error creating build container: initializing source docker://registry.access.redhat.com/ubi_8:latest: reading manifest latest in registry.access.redhat.com/ubi_8: name unknown: Repo not found
ERRO[0001] exit status 125                 
```

最后一行产生了不同的错误代码`ERRO[0001]`和退出状态`125`。这是一个非常容易排查的错误，只需将有效的仓库传递给`FROM`指令即可。

**解决方案**：修正仓库名称并重新启动构建过程。或者，验证目标仓库是否包含所需的仓库。

如果我们拼写错误了镜像标签，会发生什么呢？下一个 Dockerfile 片段显示了官方 Fedora 镜像的无效标签：

Chapter10/FROM_tag_not_found/Dockerfile

```
FROM docker.io/library/fedora:sometag
# Update image and install httpd
RUN dnf install -y httpd && dnf clean all –y
# Expose the default httpd port 80
EXPOSE 80
# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

这一次，当我们构建镜像时，仓库会产生 404 错误，因为它无法找到`sometag`标签的关联清单：

```
$ buildah build -t custom_httpd .
STEP 1/4: FROM docker.io/library/fedora:sometag
Trying to pull docker.io/library/fedora:sometag...
error creating build container: initializing source docker://fedora:sometag: reading manifest sometag in docker.io/library/fedora: manifest unknown: manifest unknown
ERRO[0001] exit status 125
```

缺少标签会产生`ERRO[0001]`错误，退出状态再次设置为`125`。

使用`skopeo list-tags`命令可以查找给定仓库中所有可用的标签。

有时，`FROM`指令捕获到的错误是由于尝试访问一个私有仓库而没有进行身份验证。这是一个非常常见的错误，只需要在进行构建操作之前，在目标仓库进行身份验证即可。

在下一个示例中，我们有一个使用来自通用私有仓库的镜像的 Dockerfile，该私有仓库通过 Docker Registry v2 APIs 运行：

Chapter10/FROM_auth_error/Dockerfile

```
FROM local-registry.example.com/ubi8
# Update image and install httpd
RUN yum install -y httpd && yum clean all –y
# Expose the default httpd port 80
EXPOSE 80
# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

让我们尝试构建镜像并查看会发生什么：

```
$ buildah build -t test3 .
STEP 1/4: FROM local-registry.example.com/ubi8
Trying to pull local-registry.example.com/ubi8:latest...
error creating build container: initializing source docker://local-registry.example.com/ubi8:latest: reading manifest latest in local-registry.example.com/ubi8: unauthorized: authentication required
ERRO[0000] exit status 125
```

在这个用例中，错误非常明显。我们没有权限从目标仓库拉取镜像，因此我们需要使用有效的认证令牌进行身份验证才能访问它。

使用`podman login`或`buildah login`命令登录仓库以获取令牌，或者提供包含有效令牌的身份验证文件。

到目前为止，我们已经检查了通过 Dockerfile 构建时产生的错误。接下来，让我们看看在使用 Buildah 命令行指令时，发生错误的情况。

## 使用 Buildah 原生命令排查构建问题

在运行 Buildah 命令时，常见的做法是将它们放入 shell 脚本或管道中。

在这个示例中，我们将使用 Bash 作为解释器。默认情况下，Bash 会一直执行脚本，直到结束，无论中间是否有错误。如果脚本中的 Buildah 指令失败，这种行为可能会产生意外的错误。因此，最佳实践是在脚本开头添加以下命令：

```
set -euo pipefail
```

结果配置是一种安全网，一旦遇到错误，它就会阻止脚本的执行，并避免常见的错误，比如未设置的变量。

`set`命令是一个 Bash 内部指令，用于配置脚本执行时的 Shell 环境。此指令中的`-e`选项告诉 Shell 在管道或单个命令失败时立即退出，`–o pipefail`选项则告诉 Shell 在失败管道的最右侧命令产生非零退出码时退出，并返回该命令的错误码。`-u`选项告诉 Shell 在参数扩展过程中将未设置的变量和参数视为错误。这能保护我们免受未设置变量未展开的影响。

下一个脚本嵌入了在 Fedora 镜像上构建一个简单`httpd`服务器的逻辑：

```
#!/bin/bash
set -euo pipefail
# Trying to pull a non-existing tag of Fedora official image
container=$(buildah from docker.io/library/fedora:non-existing-tag)
buildah run $container -- dnf install -y httpd; dnf clean all –y
buildah config --cmd "httpd -DFOREGROUND" $container
buildah config --port 80 $container
buildah commit $container custom-httpd
buildah tag custom-httpd registry.example.com/custom-httpd:v0.0.1
```

图像标签故意设置错误。让我们看看脚本执行的结果：

```
$ ./custom-httpd.sh 
Trying to pull docker.io/library/fedora:non-existing-tag...
initializing source docker://fedora:non-existing-tag: reading manifest non-existing-tag in docker.io/library/fedora: manifest unknown: manifest unknown
ERRO[0001] exit status 125
```

构建会产生一个`manifest unknown`错误，错误代码为`ERRO[0001]`，退出状态为`125`，就像使用 Dockerfile 时的类似尝试。

从这个输出中，我们还可以了解到，Buildah（以及使用 Buildah 库进行构建实现的 Podman）生成的消息与使用 Dockerfile/Containerfile 进行标准构建时的消息相同，唯一的例外是没有提到构建步骤，这是显而易见的，因为我们在脚本中运行的是自由命令。

使用`skopeo list-tags`查找给定仓库中所有可用的标签。

在这一部分中，我们学习了如何分析和排除构建错误，但当错误发生在容器内的运行时，并且我们没有合适的工具来进行容器内故障排除时该怎么办？为此，我们有一个本地 Linux 工具，可以视为命名空间的真正瑞士军刀：`nsenter`。

# 使用 nsenter 进行高级故障排除

我们从一句戏剧性的句子开始：在运行时排除故障有时可能是复杂的。

此外，理解和排除容器内的运行时问题需要理解容器在 GNU/Linux 中的工作原理。我们在*第一章*《容器技术简介》中解释了这些概念。

有时，故障排除非常简单，正如前面几部分所述，使用基本命令，如`podman logs`、`podman inspect`和`podman exec`，再加上定制的健康检查，可以帮助我们获得完成分析所需的信息。

如今，镜像通常尽可能小。那么当我们需要更多专业的故障排除工具，而它们在镜像中不可用时怎么办？你可以考虑在容器内执行一个 Shell 进程，并安装缺失的工具，但有时（这也是一个日益增长的安全模式），容器镜像中没有包管理器，有时甚至没有`curl`或`wget`命令！

我们可能会感到有些迷茫，但我们必须记住，容器是运行在专用命名空间和 cgroups 中的进程。如果我们有一个工具，可以在保持访问主机工具的同时，在一个或多个命名空间内执行命令，那会怎么样？这个工具就叫做`nsenter`（可以使用`man nsenter`访问手册页）。它与任何容器引擎或运行时无关，提供了一种简单的方法来在一个或多个进程的未共享命名空间内执行命令（即容器的主进程）。

在深入真实示例之前，让我们通过使用`--help`选项来运行`nsenter`，讨论一下它的主要选项和参数：

```
$ nsenter --help 
Usage:
 nsenter [options] [<program> [<argument>...]]
Run a program with namespaces of other processes.
Options:
-a, --all              enter all namespaces
-t, --target <pid>     target process to get namespaces from
-m, --mount[=<file>]   enter mount namespace
-u, --uts[=<file>]     enter UTS namespace (hostname etc)
-i, --ipc[=<file>]     enter System V IPC namespace
-n, --net[=<file>]     enter network namespace
-p, --pid[=<file>]     enter pid namespace
-C, --cgroup[=<file>]  enter cgroup namespace
-U, --user[=<file>]    enter user namespace
-T, --time[=<file>]    enter time namespace
-S, --setuid <uid>     set uid in entered namespace
-G, --setgid <gid>     set gid in entered namespace
     --preserve-credentials do not touch uids or gids
-r, --root[=<dir>]     set the root directory
-w, --wd[=<dir>]       set the working directory
-F, --no-fork          do not fork before exec'ing <program>
-Z, --follow-context   set SELinux context according to --target PID
-h, --help             display this help
-V, --version          display version
For more details see nsenter(1).
```

从此命令的输出中，可以轻松发现选项的数量与可用命名空间的数量相同。

感谢`nsenter`，我们可以捕获容器主进程的 PID，然后在相关的命名空间内执行命令（包括启动一个 Shell）。

要提取容器的主 PID，可以使用以下命令：

```
$ podman inspect <Container_Name> --format '{{ .State.Pid }}'
```

输出结果可以插入到变量中以便于访问：

```
$ CNT_PID=$(podman inspect <Container_Name> \
  --format '{{ .State.Pid }}')
```

提示

与进程相关的所有命名空间都表示在`/proc/[pid]/ns`目录中。该目录包含一系列符号链接，映射到命名空间类型及其对应的 inode 编号。

以下命令显示与容器执行的进程相关的命名空间：`ls –al /proc/$CNT_PID/ns`。

我们将通过一个实际的例子学习如何使用`nsenter`。在接下来的子章节中，我们将尝试对一个数据库客户端应用进行网络故障排除，该应用返回 HTTP 内部服务器错误，但应用日志中没有任何有用信息。

## 使用 nsenter 对数据库客户端进行故障排除

在处理 alpha 应用时，遇到日志未正确实现或日志消息处理较差的情况并不少见。

以下示例是一个 Web 应用，从 Postgres 数据库中提取字段，并打印出包含所有出现项的 JSON 对象。应用日志的详细信息被故意设为最少，且没有产生连接或查询错误。

为了节省空间，书中不会打印应用的源代码；但是，源代码可以通过以下 URL 进行查看：[`github.com/PacktPublishing/Podman-for-DevOps/tree/main/Chapter10/students`](https://github.com/PacktPublishing/Podman-for-DevOps/tree/main/Chapter10/students)。

文件夹中还包含一个 SQL 脚本，用于填充示例数据库。该应用是通过以下 Dockerfile 构建的：

Chapter10/students/Dockerfile

```
FROM docker.io/library/golang AS builder
# Copy files for build
RUN mkdir -p /go/src/students/models
COPY go.mod main.go /go/src/students
COPY models/main.go /go/src/students/models
# Set the working directory
WORKDIR /go/src/students
# Download dependencies
RUN go get -d -v ./...
# Install the package
RUN go build -v 
# Runtime image
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest as bin
COPY --from=builder /go/src/students /usr/local/bin
COPY entrypoint.sh /
EXPOSE 8080
ENTRYPOINT ["/entrypoint.sh"]
```

一如既往，我们将使用 Buildah 构建容器：

```
$ buildah build -t students .
```

容器接受一组自定义标志来定义数据库、主机、端口和凭据。要查看帮助信息，只需运行以下命令：

```
$ podman run students students -help
%!(EXTRA string=students)  
-database string
      Default application database (default "students")
-host string     Default host running the database (default "localhost")
-password string      Default database password (default "password"
-port string    Default database port (default "5432")
-username string       Default database username (default "admin")
```

我们已获悉数据库正在`pghost.example.com`主机上的`5432`端口运行，用户名为`students`，密码为`Podman_R0cks#`。

下一条命令使用自定义参数运行`students`网页应用程序：

```
$ podman run --rm -d -p 8080:8080 \
   --name students_app students \
   students -host pghost.example.com \
   -port 5432 \
   -username students \
   -password Podman_R0cks#
```

容器成功启动，打印的唯一日志消息如下：

```
$ podman logs students_app
2021/12/27 21:51:31 Connecting to host pghost.example.com:5432, database students
```

现在是时候测试应用程序并查看运行查询时会发生什么了：

```
$ curl localhost:8080/students
Internal Server Error
```

应用程序可能需要一些时间来响应，但过了一会儿，它会打印出一个内部服务器错误（`500`）HTTP 消息。我们将在接下来的段落中找到原因。日志没有什么用处，因为除了第一次启动消息之外没有其他输出。此外，容器是使用 UBI 最小镜像构建的，预装二进制文件的占用空间很小，且没有故障排除的工具。我们可以使用`nsenter`来检查容器的行为，特别是从网络角度，通过将当前的 Shell 程序附加到容器网络命名空间，同时保持对主机二进制文件的访问。

现在我们可以找出主进程的 PID，并将其值存入一个变量中：

```
$ CNT_PID=$(podman inspect students_app --format '{{ .State.Pid }}')
```

以下示例在容器网络命名空间中运行 Bash，同时保留其他所有主机命名空间（请注意使用`sudo`命令以提升权限运行）：

```
$ sudo nsenter -t $CNT_PID -n /bin/bash
```

重要说明

可以直接从`nsenter`运行任何主机二进制文件。如下命令是完全合法的：`$ sudo nsenter -t $CNT_PID -n ip addr show`。

为了证明我们确实在执行附加到容器网络命名空间的 Shell，我们可以启动`ip addr show`命令：

```
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: tap0: <BROADCAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether fa:0b:50:ed:9d:37 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.100/24 brd 10.0.2.255 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fe80::f80b:50ff:feed:9d37/64 scope link 
       valid_lft forever preferred_lft forever
# ip route
default via 10.0.2.2 dev tap0 
10.0.2.0/24 dev tap0 proto kernel scope link src 10.0.2.100
```

第一条命令，`ip addr show`，打印出 IP 配置，显示一个连接到主机的基本`tap0`接口和环回接口。

第二条命令，`ip route`，显示容器网络命名空间内的默认路由表。

我们可以使用`ss`工具查看活动连接，`ss`工具已经在我们的 Fedora 主机上可用：

```
# ss –atunp
Netid State     Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
tcp   TIME-WAIT 0      0         10.0.2.100:50728   10.0.2.100:8080
tcp   LISTEN    0      128                *:8080             *:*    usersL"("studen"s",pid=402788,fd=3))
```

我们立刻发现应用程序与数据库主机之间没有建立连接，这告诉我们问题可能与路由、防火墙规则或名称解析相关，这些因素阻止了我们正确访问主机。

下一步是尝试使用`psql`客户端工具手动连接到数据库，该工具来自`postgresql`rpm 包：

```
# psql -h pghost.example.com
psql: error: could not translate host name "pghost.example.com" to address: Name or service not known
```

这个消息很清楚：主机无法通过 DNS 服务解析，导致应用程序失败。为了最终确认这一点，我们可以运行`dig`命令，它返回一个`NXDOMAIN`错误，这是 DNS 服务器典型的消息，表示域无法解析并不存在：

```
# dig pghost.example.com
; <<>> DiG 9.16.23-RH <<>> pghost.example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 40669
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;pghost.example.com.        IN    A
;; Query time: 0 msec
;; SERVER: 192.168.200.1#53(192.168.200.1)
;; WHEN: Mon Dec 27 23:26:47 CET 2021
;; MSG SIZE  rcvd: 47
```

在与开发团队确认后，我们发现数据库名称中缺少了一个连字符并拼写错误，正确的名称是 `pg-host.example.com`。现在，我们可以通过使用正确的名称运行容器来修复问题。

我们现在期望再次启动查询时看到正确的结果：

```
$ curl localhost:8080/students
{"Id":10149,"FirstName":"Frank","MiddleName":"Vincent","LastName":"Zappa","Class":"3A","Course":"Composition"}
```

在这个示例中，我们集中讨论了网络命名空间故障排除，但通过简单地添加相关标志，我们也可以将当前的 shell 程序附加到多个命名空间。

我们还可以通过添加 `-a` 选项来模拟 `podman exec`：

```
$ sudo nsenter -t $CNT_PID -a /bin/bash
```

该命令将进程附加到所有未共享的命名空间，包括挂载命名空间，从而提供与容器内进程看到的相同的文件系统树视图。

# 总结

在本章中，我们重点讨论了容器故障排除，旨在提供一套最佳实践和工具，用于在构建时或运行时查找并修复容器内的问题。

我们首先展示了在容器执行和构建阶段中的一些常见用例及其相关解决方案。

随后，我们介绍了健康检查的概念，并说明了如何在容器上实现稳健的探针以监控其状态，同时展示了其背后的架构概念。

在第三节中，我们学习了一系列与构建相关的常见错误场景，并展示了如何快速解决它们。

在最后一节中，我们介绍了 `nsenter` 命令，并模拟了一个需要网络故障排除的 Web 前端应用，以找出内部服务器错误的原因。通过这个示例，我们学会了如何在容器命名空间内部进行高级故障排除。

在下一章中，我们将讨论容器安全性，这是一个至关重要的概念，值得特别关注。我们将学习如何通过一系列最佳实践来保障容器安全，了解无根容器和有根容器之间的区别，并学习如何签名容器镜像以使其公开可用。

# 深入阅读

+   Podman 故障排除指南：[`github.com/containers/podman/blob/main/troubleshooting.md`](https://github.com/containers/podman/blob/main/troubleshooting.md)
