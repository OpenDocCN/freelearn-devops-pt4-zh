# 第七章: 与现有应用构建流程的集成

学会了如何使用 Podman 和 Buildah 创建自定义容器镜像后，我们现在可以集中精力处理一些特殊的用例，从而使我们的构建工作流更加高效和可移植。例如，在企业环境中，小尺寸镜像是非常常见的需求，出于性能和安全性的考虑。我们将探讨如何通过将构建过程拆分为不同阶段来实现这一目标。

本章还将尝试揭示一些场景，其中 Buildah 不是直接在开发者机器上运行，而是由容器编排器驱动，或者嵌入在预期调用其库或**命令行界面**(**CLI**)的自定义应用程序中。

本章将覆盖以下主要主题：

+   多阶段容器构建

+   在容器中运行 Buildah

+   将 Buildah 与自定义构建工具集成

# 技术要求

在继续本章内容之前，需要一台已安装并正常运行 Podman 的机器。如*第三章*《运行第一个容器》中所述，本书中的所有示例都是在 Fedora 34 或更高版本的系统上执行的，但也可以在读者选择的操作系统上重现。

对于*第六章*《了解 Buildah —— 从头开始构建容器》中的内容有一个良好的理解，将有助于轻松掌握构建相关的概念，无论是使用本地 Buildah 命令还是从 Dockerfile 中构建。

# 多阶段容器构建

到目前为止，我们已经学习了如何使用 Podman 和 Buildah 通过 Dockerfile 或本地 Buildah 命令来创建构建，从而释放潜在的高级构建技术。

还有一个我们尚未讨论的重要点——镜像的大小。

在创建新镜像时，我们应该始终关注其最终大小，这是总层数和其中变更的文件数量的结果。

小尺寸的最小镜像具有一个很大的优势，即能够更快地从注册表中拉取。然而，较大的镜像将占用主机本地存储中大量宝贵的磁盘空间。

我们已经展示了一些最佳实践的例子，以保持镜像紧凑小巧，例如从头开始构建、清理包管理器缓存，以及将 `RUN`、`COPY` 和 `ADD` 指令的数量减少到最少必要量。然而，当我们需要从源代码构建应用并创建包含最终制品的最终镜像时，情况会如何？

假设我们需要构建一个容器化的 Go 应用程序——我们应该从一个包含 Go 运行时的基础镜像开始，复制源代码，并通过一系列中间步骤进行编译以生成最终的二进制文件，最重要的是在镜像缓存中下载所有必要的 Go 包。在构建结束时，我们应该清理所有源代码和下载的依赖项，并将最终的二进制文件（Go 中静态链接）放在工作目录中。一切正常，但最终的镜像仍然包含基础镜像中的 Go 运行时，而这些在编译过程结束后已经不再需要。

当 Docker 被引入并且 Dockerfile 逐渐流行时，DevOps 团队通过不同的方法解决了这个问题，他们努力保持镜像的最小化。例如，**二进制构建**是一种将外部编译的最终产物注入到构建镜像中的方法。这种方法解决了镜像大小的问题，但消除了运行时/编译器镜像提供的标准化构建环境的优势。

更好的方法是通过容器之间共享卷，并让最终的容器镜像从第一个构建镜像中获取编译后的产物。

为了提供一种标准化的方法，Docker 和随后的 OCI 规范引入了 `FROM` 指令的概念，允许后续镜像从前一个镜像中抓取内容。

在接下来的子章节中，我们将探讨如何使用 Dockerfile/Containerfile 和 Buildah 的原生命令实现这一结果。

## 使用 Dockerfile 进行多阶段构建

多阶段构建的第一种方法是在单个 Dockerfile/Containerfile 中创建多个阶段，每个阶段以 `FROM` 指令开始。

构建阶段可以使用 `--from` 选项从前一个阶段复制文件和文件夹，以指定源阶段。

下列示例展示了如何为 Go 应用程序创建一个最小的多阶段构建，第一个阶段作为纯粹的构建上下文，第二个阶段将最终产物复制到一个最小的镜像中：

Chapter07/http_hello_world/Dockerfile

```
# Builder image
FROM docker.io/library/golang
# Copy files for build
COPY go.mod /go/src/hello-world/
COPY main.go /go/src/hello-world/
# Set the working directory
WORKDIR /go/src/hello-world
# Download dependencies
RUN go get -d -v ./...
# Install the package
RUN go build -v 
# Runtime image
FROM registry.access.redhat.com/ubi8/ubi-micro:latest
COPY --from=0 /go/src/hello-world/hello-world /
EXPOSE 8080
CMD ["/hello-world"]
```

第一个阶段将源 `main.go` 文件和 `go.mod` 文件复制过来，以管理 Go 模块依赖关系。下载依赖包（`go get -d -v ./...`）后，最终应用程序将被构建（`go build -v ./...`）。

第二个阶段抓取最终的产物（`/go/src/hello-world/hello-world`）并将其复制到新镜像的根目录下。为了指定源文件应从第一个阶段复制，使用 `--from=0` 语法。

在第一阶段，我们使用了官方的 `docker.io`/`library`/`golang` 镜像，其中包含 Go 编程语言的最新版本。在第二阶段，我们使用了 **ubi-micro** 镜像，这是来自 Red Hat 的一个最小化镜像，具有减少的体积，针对微服务和静态链接二进制文件进行了优化。通用基础镜像将在*第八章*中更详细地介绍，*选择容器基础镜像*。

以下列出的 Go 应用程序是一个基础的 web 服务器，它监听 `8080/tcp` 端口，并在收到 `GET /` 请求时打印一个包含 *"Hello World!"* 消息的 HTML 页面：

重要说明

本书的目的并不要求能够编写或理解 Go 编程语言。然而，了解语言的基本语法和逻辑将非常有用，因为大部分与容器相关的软件（如 Podman、Docker、Buildah、Skopeo、Kubernetes 和 OpenShift）都是用 Go 编写的。

Chapter07/http_hello_world/main.go

```
package main
import (
       "log"
   "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
     log.Printf("%s %s %s\n", r.RemoteAddr, r.Method, r.URL)
     w.Header().Set("Content-Type", "text/html")
     w.Write([]byte("<html>\n<body>\n"))
     w.Write([]byte("<p>Hello World!</p>\n"))
     w.Write([]byte("</body>\n</html>\n"))
}
func main() {
     http.HandleFunc("/", handler)
     log.Println("Starting http server")
     log.Fatal(http.ListenAndServe(":8080", nil))
}
```

应用程序可以使用 Podman 或 Buildah 构建。在此示例中，我们选择使用 Buildah 来构建应用程序：

```
$ cd http_hello_world
$ buildah build -t hello-world .
```

最后，我们可以检查结果镜像的大小：

```
$ buildah images --format '{{.Name}} {{.Size}}' \     localhost/hello-world
localhost/hello-world   45 MB
```

最终的镜像大小只有 45 MB！

我们可以通过使用 `AS` 关键字为基础镜像添加自定义名称，从而改进我们的 Dockerfile。以下示例是按照此方法重新制作的 Dockerfile，关键元素用粗体显示：

```
# Builder image
FROM docker.io/library/golang AS builder
# Copy files for build
COPY go.mod /go/src/hello-world/
COPY main.go /go/src/hello-world/
# Set the working directory
WORKDIR /go/src/hello-world
# Download dependencies
RUN go get -d -v ./...
# Install the package
RUN go build -v ./...
# Runtime image
FROM registry.access.redhat.com/ubi8/ubi-micro:latest AS srv
COPY --from=builder /go/src/hello-world/hello-world /
EXPOSE 8080
CMD ["/hello-world"]
```

在前面的示例中，构建器镜像的名称设置为 `builder`，而最终的镜像命名为 `srv`。有趣的是，`COPY` 指令现在可以使用 `--from=builder` 选项指定构建器并使用自定义名称。

Dockerfile/Containerfile 构建是最常见的方法，但在实现自定义构建工作流程时仍缺乏一定的灵活性。对于那些特殊的使用案例，Buildah 原生命令为我们提供了帮助。

## 使用 Buildah 原生命令的多阶段构建

如前所述，多阶段构建功能是一种很好的方法，可以生成小体积且减少攻击面图像。为了在构建过程中提供更大的灵活性，Buildah 原生命令派上了用场。正如我们在*第六章*中提到的，*遇见 Buildah——从零开始构建容器*，Buildah 提供了一系列命令，复制了 Dockerfile 指令的行为，从而在将这些命令包含在脚本或自动化过程中时，提供了更大的构建过程控制。

在多阶段构建中也适用相同的概念，我们还可以在阶段之间应用额外的步骤。例如，我们可以挂载构建容器的覆盖文件系统，并提取构建的工件以发布备用包，所有这些都可以在构建最终运行时镜像之前完成。

以下示例通过将之前的 Dockerfile 指令转换为本地 Buildah 命令，在一个简单的 shell 脚本中构建相同的 `hello-world` Go 应用程序：

```
#!/bin/bash
# Define builder and runtime images
BUILDER=docker.io/library/golang
RUNTIME=registry.access.redhat.com/ubi8/ubi-micro:latest
# Create builder container
container1=$(buildah from $BUILDER)
# Copy files from host
if [ -f go.mod ]; then 
    buildah copy $container1 'go.mod' '/go/src/hello-world/'
else
    exit 1
fi
if [ -f main.go ]; then 
    buildah copy $container1 'main.go' '/go/src/hello-world/'
else 
    exit 1
fi
# Configure and start build
buildah config --workingdir /go/src/hello-world $container1
buildah run $container1 go get -d -v ./...
buildah run $container1 go build -v ./...
# Create runtime container
container2=$(buildah from $RUNTIME)
# Copy files from the builder container
buildah copy --chown=1001:1001 \
    --from=$container1 $container2 \
    '/go/src/hello-world/hello-world' '/'
# Configure exposed ports
buildah config --port 8080 $container2
# Configure default CMD
buildah config --cmd /hello-world $container2
# Configure default user
buildah config --user=1001 $container2
# Commit final image
buildah commit $container2 hello-world
# Remove build containers
buildah rm $container1 $container2
```

在前面的示例中，我们突出了创建两个工作容器的命令以及相关的 `container1` 和 `container2` 变量，这些变量存储了容器 ID。

同时，注意 `buildah copy` 命令，我们在其中通过 `--from` 选项定义了源容器，并使用 `--chown` 选项定义了复制资源的用户和组所有者。这种方法比基于 Dockerfile 的工作流更加灵活，因为我们可以用变量、条件和循环来丰富我们的脚本。

例如，我们在 Bash 脚本中测试了使用 `if` 条件来检查 `go.mod` 和 `main.go` 文件是否存在，然后再将它们复制到专用于构建的工作容器内。

现在让我们向脚本添加一个额外的功能。在以下示例中，我们通过为构建添加语义化版本控制，并在开始构建最终运行时镜像之前创建版本归档，进一步改进了之前的脚本：

重要提示

语义化版本控制的概念旨在提供一种清晰且标准化的方式来管理软件版本和依赖关系管理。它是一套标准规则，其目的是定义如何应用软件发布版本，并遵循 **X.Y.Z** 版本模式，其中 **X** 是主版本号，**Y** 是次版本号，**Z** 是补丁版本号。有关更多信息，请查看官方规范：[`semver.org/`](https://semver.org/)。

```
#!/bin/bash
# Define builder and runtime images
BUILDER=docker.io/library/golang
RUNTIME=registry.access.redhat.com/ubi8/ubi-micro:latest
RELEASE=1.0.0
# Create builder container
container1=$(buildah from $BUILDER)
# Copy files from host
if [ -f go.mod ]; then 
    buildah copy $container1 'go.mod' '/go/src/hello-world/'
else
    exit 1
fi
if [ -f main.go ]; then 
    buildah copy $container1 'main.go' '/go/src/hello-world/'
else 
    exit 1
fi
# Configure and start build
buildah config --workingdir /go/src/hello-world $container1
buildah run $container1 go get -d -v ./...
buildah run $container1 go build -v ./...
# Extract build artifact and create a version archive
buildah unshare --mount mnt=$container1 \
    sh -c 'cp $mnt/go/src/hello-world/hello-world .'
cat > README << EOF
Version $RELEASE release notes:
- Implement basic features
EOF
tar zcf hello-world-${RELEASE}.tar.gz hello-world README
rm -f hello-world README
# Create runtime container
container2=$(buildah from $RUNTIME)
# Copy files from the builder container
buildah copy --chown=1001:1001 \
    --from=$container1 $container2 \
    '/go/src/hello-world/hello-world' '/'
# Configure exposed ports
buildah config --port 8080 $container2
# Configure default CMD
buildah config --cmd /hello-world $container2
# Configure default user
buildah  config--user=1001 $container2
# Commit final image
buildah commit $container2 hello-world:$RELEASE
# Remove build containers
buildah rm $container1 $container2
```

脚本中的关键更改再次以粗体突出显示。首先，我们添加了一个 `RELEASE` 变量，用于跟踪应用程序的发布版本。然后，我们使用 `buildah unshare` 命令提取了构建产物，接着使用 `--mount` 选项传递了容器的挂载点。用户命名空间的 unshare 是必要的，以使脚本能够以无根（rootless）模式运行。

提取构建产物后，我们使用 `$RELEASE` 变量在归档文件名中创建了一个 gzipped 压缩文件，并删除了临时文件。

最后，我们开始构建运行时镜像，并使用 `$RELEASE` 变量再次提交，作为镜像标签。

在这一部分中，我们学习了如何使用 Buildah 运行多阶段构建，使用 Dockerfiles/Containerfiles 和本地命令。在下一部分，我们将学习如何在容器内隔离 Buildah 构建。

# 在容器内运行 Buildah

Podman 和 Buildah 遵循 fork/exec 方法，使得它们非常容易在容器内运行，包括无根容器场景。

有许多用例需要容器化构建。如今，其中一个最常见的应用场景是基于 **Kubernetes** 集群的应用构建工作流。

Kubernetes 基本上是一个容器编排器，管理通过控制平面调度容器，容器运行在与 **容器运行时接口** (**CRI**) 兼容的容器引擎上。它的设计允许在网络、存储和运行时的定制上具有极大的灵活性，推动了许多侧项目的蓬勃发展，这些项目现在正在 **云原生计算基金会** (**CNCF**) 内孵化或成熟。

**Vanilla** Kubernetes（即没有任何自定义或附加组件的基本社区版本）本身没有原生构建功能，但提供了实现此功能的适当框架。随着时间的推移，许多解决方案应运而生，试图解决这一需求。

例如，**OpenShift**（红帽的 Kubernetes 平台）在 Kubernetes 1.0 发布时就引入了自己的构建 API 和 *Source-to-Image* 工具包，用于直接在 OpenShift 集群上从源代码创建容器镜像。

另一个有趣的解决方案是谷歌的 **kaniko**，它是一个构建工具，可以在 Kubernetes 集群内创建容器镜像，并且每个构建步骤都在用户空间中运行。

除了使用已实现的解决方案外，我们还可以设计自己在 Kubernetes 编排下运行的 Buildah 容器。我们还可以利用无根设计实现安全的构建工作流。

可以在 Kubernetes 集群之上运行 CI/CD 流水线，并将容器化构建嵌入到流水线中。**Tekton Pipelines** 是一个非常有趣的 CNCF 项目，提供了一种云原生的方法来实现这一目标。Tekton 允许运行由 Kubernetes 自定义资源驱动的流水线——这些是扩展基本 API 集的特殊 API。

Tekton Pipelines 由许多不同的任务组成，用户可以创建自己的任务，也可以从 **Tekton Hub** ([`hub.tekton.dev/`](https://hub.tekton.dev/)) 中获取任务，这是一个免费的仓库，提供了许多可以立即使用的预制任务，包括来自 Buildah 的示例 ([`hub.tekton.dev/tekton/task/buildah`](https://hub.tekton.dev/tekton/task/buildah))。

前面的例子有助于理解为什么容器化构建如此重要。在本书中，我们将重点关注在容器中运行构建的细节，特别关注与安全相关的约束。

## 运行无根权限的 Buildah 容器并使用卷存储

在本小节中的例子将使用稳定的上游 `quay.io/buildah/stable` Buildah 镜像。此镜像已嵌入最新的稳定 Buildah 二进制文件。

让我们运行第一个示例，使用一个无根容器，它构建主机中 `~/build` 目录的内容，并将输出存储在名为 `storevol` 的本地卷中：

```
$ podman run --device /dev/fuse \
    -v ~/build:/build:z \
    -v storevol:/var/lib/containers quay.io/buildah/stable \
    buildah build -t build_test1 /build
```

这个例子引入了一些值得注意的特殊选项，具体如下：

+   `--device /dev/fuse` 选项会在容器中加载 fuse 内核模块，这是运行 fuse-overlay 命令所必需的

+   `-v ~/build:/build:z` 选项，它将 `/root/build` 目录绑定挂载到容器内部，并通过 `:z` 后缀为其分配适当的 SELinux 标签。

+   `-v storevol:/var/lib/containers` 选项，它创建了一个新的卷并挂载到默认的容器存储路径，所有层都在该位置创建。

当构建完成后，我们可以使用相同的卷运行一个新容器，检查或操作已构建的镜像：

```
$ podman run --rm -v storevol:/var/lib/containers quay.io/buildah/stable buildah images
REPOSITORY                  TAG      IMAGE ID       CREATED          SIZE
localhost/build_test1             latest   cd36bf58daff   12 minutes ago   283 
docker.io/library/fedora    latest   b080de8a4da3   4 days ago       159 MB
```

我们已经成功构建了一个镜像，其层存储在 `storevol` 卷中。要递归列出存储内容，我们可以使用 `podman volume inspect` 命令提取卷的挂载点：

```
$ ls -alR \
$(podman volume inspect storevol --format '{{.Mountpoint}}')
```

从现在开始，可以启动一个新的 Buildah 容器来对远程注册表进行身份验证，并标记和推送镜像。在下一个示例中，Buildah 会标记生成的镜像、对远程注册表进行身份验证，并最终推送镜像：

```
$ podman run --rm -v storevol:/var/lib/containers \
  quay.io/buildah/stable \
  sh -c 'buildah tag build_test1 \
    registry.example.com/build_test1 \
    && buildah login -u=<USERNAME> -p=<PASSWORD> \
    registry.example.com && \
    buildah push registry.example.com/build_test1'
```

当镜像成功推送后，可以安全地移除该卷：

```
# podman volume rm storevol
```

尽管这种方法工作得非常完美，但它有一些值得讨论的限制。

我们可以注意到的第一个限制是存储卷没有隔离，因此其他任何容器都可以访问其内容。为了解决这个问题，我们可以使用 SELinux 的 `:Z` 后缀，应用类别标签到该卷，并使其仅对正在运行的容器可访问。

然而，由于第二个容器默认会使用不同的类别标签，我们需要获取卷的类别标签，并使用 `--security-opt label=level:s0:<CAT1>,<CAT2>` 选项来运行第二个标记/推送容器。

或者，我们可以在一个容器中运行构建、标记和推送命令，如以下示例所示：

```
$ podman run --device /dev/fuse \
    -v ~/build:/build \
    -v secure_storevol:/var/lib/containers:Z \
    quay.io/buildah/stable \
    sh -c 'buildah build -t test2 /build && \
      buildah tag test2 registry.example.com/build_test2 && \
      buildah login -u=<USERNAME> \
      -p=<PASSWORD> \
      registry.example.com && \
      buildah push registry.example.com/build_test2'
```

重要提示

在前面的示例中，我们通过直接在命令中传递用户名和密码来使用 Buildah 登录。毋庸置疑，这远远不是一种可接受的安全做法。

我们可以将包含有效会话令牌的认证文件作为一个卷挂载到容器内部，而不是通过命令行传递敏感数据。

下一个示例将一个有效的 `auth.json` 文件挂载到构建容器中，文件存储在 `/run/user/<UID>` 的 tmpfs 中，然后将 `--authfile /auth.json` 选项传递给 `buildah push` 命令：

```
$ podman run --device /dev/fuse \
    -v ~/build:/build \ 
    -v /run/user/<UID>/containers/auth.json:/auth.json:z \
    -v secure_storevol:/var/lib/containers:Z \
    quay.io/buildah/stable \
    sh -c 'buildah build -t test3 /build && \
      buildah tag test3 registry.example.com/build_test3 && \
      buildah push --authfile /auth.json \
      registry.example.com/build_test3'
```

最后，我们有一个有效的示例，避免了在传递给容器的命令中暴露明文凭据。

为了提供有效的认证文件，我们需要从将运行容器化构建的主机进行认证，或者复制一个有效的认证文件。为了通过 Podman 进行认证，我们将使用以下命令：

```
$ podman login –u <USERNAME> -p <PASSWORD> <REGISTRY>
```

如果认证过程成功，获取的令牌将存储在 `/run/user/<UID>/containers/auth.json` 文件中，该文件存储了一个 JSON 编码的对象，其结构类似于以下示例：

```
{
      "auths": {
           "registry.example.com": {
                "auth": "<base64_encoded_token>"
               }
    }
}
```

安全警告！

如果挂载到容器内的身份验证文件包含多个针对不同注册表的身份验证记录，它们将暴露在构建容器中。这可能导致潜在的安全问题，因为容器可以使用文件中指定的令牌在这些注册表上进行身份验证。

我们刚刚描述的基于卷的方法与本地主机构建相比，在性能上有一些小的影响，但由于采用无根执行和跨主机标准化构建环境，它提供了更好的构建过程隔离，减少了攻击面。

现在，让我们来看看如何使用绑定挂载的存储来运行容器化的构建。

## 使用绑定挂载存储运行 Buildah 容器

在最高隔离的场景下，当 DevOps 团队遵循零信任方法时，每个构建容器应该拥有自己独立的存储，该存储在构建开始时创建，在构建完成后销毁。隔离可以通过 SELinux MCS 安全轻松实现。

为了测试这种方法，首先创建一个临时目录，用于存放构建层。我们还需要为名称生成一个随机后缀，以便在不发生冲突的情况下托管多个构建：

```
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)
# mkdir $BUILD_STORE
```

重要提示

上述示例和接下来的构建都是以 root 身份执行的。

现在，我们可以运行构建并将新目录绑定挂载到容器内的`/var/lib/containers`文件夹，并添加`:Z`后缀，以确保多类别的安全隔离：

```
# podman run --device /dev/fuse \
    -v ./build:/build:z \
    -v $BUILD_STORE:/var/lib/containers:Z \
    -v /run/containers/0/auth.json:/auth.json \ 
    quay.io/buildah/stable \
    bash -c 'set -euo pipefail; \
      buildah build -t registry.example.com/test4 /build; \
      buildah push --authfile /auth.json \
      registry.example.com/test4'
```

MCS 隔离确保与其他容器的隔离。每个构建容器将拥有自己的自定义存储，这意味着每次执行时都需要重新拉取基础镜像层，因为它们从未被缓存。

尽管在隔离方面是最安全的，但这种方法也因为在构建运行过程中持续拉取镜像而提供最慢的性能。

另一方面，较不安全的方法不期望任何存储隔离，所有构建容器都将默认主机存储挂载到`/var/lib/containers`下。这种方法提供更好的性能，因为它允许重用主机存储中的缓存层。

SELinux 不允许容器化的进程访问主机存储；因此，我们需要放宽 SELinux 安全限制，以使用`--security-opt label=disable`选项运行以下示例。

以下示例使用默认主机存储运行另一个构建：

```
# podman run --device /dev/fuse \
  -v ./build:/build:z 
  -v /var/lib/containers:/var/lib/containers \
  --security-opt label=disable \
  -v /run/containers/0/auth.json:/auth.json \
  quay.io/buildah/stable \
  bash -c 'set -euo pipefail; \
    buildah build -t registry.example.com/test5 /build; \
    buildah push --authfile /auth.json \
    registry.example.com/test5'
```

本示例中描述的方法与前一种方法相反——性能更好，但安全隔离较差。

两者之间的良好折中方法是使用一个次要的只读镜像存储来提供对缓存层的访问。Buildah 支持使用多个镜像存储，且`/etc/containers/storage.conf`文件*在 Buildah 稳定镜像内*已经配置了`/var/lib/shared`文件夹以实现此目的。

为了验证这一点，我们可以检查 `/etc/containers/storage.conf` 文件的内容，在其中定义了以下部分：

```
# AdditionalImageStores is used to pass paths to additional Read/Only image stores
# Must be comma separated list.
additionalimagestores = [
"/var/lib/shared",
]
```

这样做可以实现良好的隔离和更好的性能，因为来自主机的缓存镜像将已经可用于只读存储。只读存储可以预先填充以加快构建速度，或者可以从网络共享中挂载。

以下示例展示了这种方法，通过将只读存储绑定到容器并执行构建，利用重新拉取的镜像的优势：

```
# podman run --device /dev/fuse \
  -v ./build:/build:z \
  -v $BUILD_STORE:/var/lib/containers:Z \
  -v /var/lib/containers/storage:/var/lib/shared:ro \ 
  -v /run/containers/0/auth.json:/auth.json:z \
  quay.io/buildah/stable \
  bash -c 'set -euo pipefail; \
  buildah build -t registry.example.com/test6 /build; \
  buildah push --authfile /auth.json \
  registry.example.com/test6'
```

此子节中显示的示例也受到了 *Dan Walsh*（Buildah 和 Podman 项目的主要负责人之一）在 *Red Hat Developer* 博客上撰写的一篇出色技术文章的启发；请参阅 *进一步阅读* 部分以获取原始文章链接。让我们通过一个原生 Buildah 命令的示例来结束这一节。

## 在容器内运行原生 Buildah 命令

到目前为止，我们已经演示了使用 Dockerfiles/Containerfiles 的示例，但没有任何东西阻止我们运行容器化的原生 Buildah 命令。以下示例创建了一个基于 Fedora 基础映像构建的自定义 Python 镜像：

```
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)# mkdir $BUILD_STORE 
# podman run --device /dev/fuse \
  -e REGISTRY=<USER_DEFINED_REGISTRY:PORT> \
  --security-opt label=disable \
  -v $BUILD_STORE:/var/lib/containers:Z \
  -v /var/lib/containers/storage:/var/lib/shared:ro \
  -v /run/containers/0:/run/containers/0 \
  quay.io/buildah/stable \
  bash -c 'set -euo pipefail; \
    container=$(buildah from fedora); \
    buildah run $container dnf install -y python3 python3; \
    buildah commit $container $REGISTRY/python_demo; \
    buildah push –authfile \
    /run/containers/0/auth.json $REGISTRY/python_demo'
```

从性能和构建过程的角度来看，与之前的示例相比，没有任何变化。如前所述，这种方法在构建操作上提供了更多的灵活性。

如果要传递的命令太多，一个很好的解决方法是创建一个 shell 脚本，并通过专用卷将其注入到 Buildah 镜像中：

```
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)
# PATH_TO_SCRIPT=/path/to/script
# REGISTRY=<USER_DEFINED_REGISTRY:PORT>
# mkdir $BUILD_STORE 
# podman run --device /dev/fuse \
  -v $BUILD_STORE:/var/lib/containers:Z \
  -v /var/lib/containers/storage:/var/lib/shared:ro \
  -v /run/containers/0:/run/containers/0 \
  -v $PATH_TO_SCRIPT:/root:z \ 
  quay.io/buildah/stable /root/build.sh
```

`build.sh` 是包含所有构建自定义命令的 shell 脚本文件的名称。

在这一节中，我们学习了如何在容器中运行 Buildah，涵盖了卷挂载和绑定挂载。我们学习了如何运行无根构建容器，这些容器可以轻松集成到管道或 Kubernetes 集群中，以提供端到端的应用程序生命周期工作流程。这归功于 Buildah 的灵活性，出于同样的原因，将 Buildah 嵌入到自定义构建器中也非常容易，我们将在下一节中看到。

# 在自定义构建器中集成 Buildah

正如我们在本章的前一节中看到的那样，Buildah 是 Podman 容器生态系统的关键组件。Buildah 是一个动态和灵活的工具，可以适应不同的场景来构建全新的容器。它有多个选项和配置可用，但我们的探索还没有结束。

Podman 和围绕它开发的所有项目都考虑到了可扩展性，使得每个可编程接口都可以从外部世界重用。

例如，Podman 通过 `podman build` 命令继承了 Buildah 的能力，可以用相同的原则将 Buildah 接口及其引擎嵌入我们的自定义构建器中，用于构建全新的容器。

让我们看看如何在 Go 语言中构建自定义构建器；我们会发现过程相当简单，因为 Podman、Buildah 和这个生态系统中的许多其他项目实际上都是用 Go 语言编写的。

## 将 Buildah 包含到我们的 Go 构建工具中

作为第一步，我们需要准备我们的开发环境，下载并安装所有所需的工具和库来创建我们的自定义构建工具。

在 *第三章*，*运行第一个容器*，我们看到了一些 Podman 的安装方法。在接下来的章节中，我们将使用类似的过程，进行从头开始构建 Buildah 项目的初步步骤，下载其源文件并将其包含在我们的自定义构建器中。

首先，让我们确保在我们的开发主机系统上安装了所有所需的包：

```
# dnf install -y golang git go-md2man btrfs-progs-devel \ gpgme-devel device-mapper-devel
Last metadata expiration check: 0:43:05 ago on mar 9 nov 2021, 17:21:23.
Package git-2.33.1-1.fc35.x86_64 is already installed.
Dependencies resolved.
=============================================================================================================================================================================
Package                                                 Architecture                     Version                                    Repository                         Size
=============================================================================================================================================================================
Installing:
btrfs-progs-devel                                       x86_64                            5.14.2-1.fc35                              updates                            50 k
device-mapper-devel                                     x86_64                           1.02.175-6.fc35                            fedora                             45 k
golang                                                  x86_64                           1.16.8-2.fc35                              fedora                            608 k
golang-github-cpuguy83-md2man                           x86_64                           2.0.1-1.fc35                               fedora                            818 k
gpgme-devel                                             x86_64                           1.15.1-6.fc35                              updates                           163 k
Installing dependencies:
[... omitted output]
```

安装了 Go 语言核心库和一些其他开发工具后，我们已准备好为我们的项目创建目录结构并初始化它：

```
$ mkdir ~/custombuilder
$ cd ~/custombuilder
[custombuilder]$ export GOPATH=`pwd`
```

如前所述的示例所示，我们按照以下步骤进行了操作：

1.  创建了项目根目录

1.  定义了我们将要使用的 Go 语言根路径

我们现在准备创建我们的 Go 模块，它将通过几个简单的步骤来创建我们定制的容器镜像。

为了加快示例并避免任何书写错误，我们可以从本书的官方 GitHub 仓库下载我们将要用于本次测试的 Go 语言代码：

1.  访问 [`github.com/PacktPublishing/Podman-for-DevOps`](https://github.com/PacktPublishing/Podman-for-DevOps) 或运行以下命令：

    ```
    $ git clone https://github.com/PacktPublishing/Podman-for-DevOps 
    ```

1.  之后，将 `Chapter07/*` 目录中的文件复制到新创建的 `~/custombuilder/` 目录中。

到目前为止，你的目录中应该有以下文件：

```
$ cd ~/custombuilder/src/builder
$ ls -latotal 148
drwxrwxr-x. 1 alex alex     74 9 nov 15.22 .
drwxrwxr-x. 1 alex alex     14 9 nov 14.10 ..
-rw-rw-r--. 1 alex alex   1466 9 nov 14.10 custombuilder.go
-rw-rw-r--. 1 alex alex    161 9 nov 15.22 go.mod
-rw-rw-r--. 1 alex alex 135471 9 nov 15.22 go.sum
-rw-rw-r--. 1 alex alex    337 9 nov 14.17 script.js
```

此时，我们可以运行以下命令，让 Go 工具获取所有需要的依赖项，为执行模块做准备：

```
$ go mod tidy
go: finding module for package github.com/containers/storage/pkg/unshare
go: finding module for package github.com/containers/image/v5/storage
go: finding module for package github.com/containers/storage
go: finding module for package github.com/containers/image/v5/types
go: finding module for package github.com/containers/buildah/define
go: finding module for package github.com/containers/buildah
go: found github.com/containers/buildah in github.com/containers/buildah v1.23.1
go: found github.com/containers/buildah/define in github.com/containers/buildah v1.23.1
go: found github.com/containers/image/v5/storage in github.com/containers/image/v5 v5.16.1
go: found github.com/containers/image/v5/types in github.com/containers/image/v5 v5.16.1
go: found github.com/containers/storage in github.com/containers/storage v1.37.0
go: found github.com/containers/storage/pkg/unshare in github.com/containers/storage v1.37.0
```

工具分析了提供的 `custombuilder.go` 文件，找到了所有所需的库，并填充了 `go.mod` 文件。

重要提示

请注意，之前的命令将验证模块是否可用，如果不可用，工具将开始从互联网下载它。所以在此步骤中请耐心等待！

我们可以通过检查之前创建的目录结构来验证前面的命令是否下载了所有必需的包：

```
$ cd ~/custombuilder
[custombuilder]$ ls
pkg  src
[custombuilder]$ ls -la pkg/
total 0
drwxrwxr-x. 1 alex alex  28  9 nov 18.27 .
drwxrwxr-x. 1 alex alex  12  9 nov 18.18 ..
drwxrwxr-x. 1 alex alex  20  9 nov 18.27 linux_amd64
drwxrwxr-x. 1 alex alex 196  9 nov 18.27 mod
[custombuilder]$ ls -la pkg/mod/
total 0
drwxrwxr-x. 1 alex alex 196  9 nov 18.27 .
drwxrwxr-x. 1 alex alex  28  9 nov 18.27 ..
drwxrwxr-x. 1 alex alex  22  9 nov 18.18 cache
drwxrwxr-x. 1 alex alex 918  9 nov 18.27 github.com
drwxrwxr-x. 1 alex alex  24  9 nov 18.27 go.etcd.io
drwxrwxr-x. 1 alex alex   2  9 nov 18.27 golang.org
[... omitted output]
[custombuilder]$ ls -la pkg/mod/github.com/
[... omitted output]
drwxrwxr-x. 1 alex alex  98  9 nov 18.27  containerd
drwxrwxr-x. 1 alex alex  20  9 nov 18.27  containernetworking
drwxrwxr-x. 1 alex alex 184  9 nov 18.27  containers
drwxrwxr-x. 1 alex alex 110  9 nov 18.27  coreos
[... omitted output]
```

我们现在准备运行我们的自定义构建器模块，但在继续之前，让我们看一下 Go 源文件中包含的关键元素。

如果我们开始查看 `custombuilder.go` 文件，在定义包和要使用的库之后，我们定义了模块的主函数。

在主函数中，在定义的开始，我们插入了一个基本的代码块：

```
  if buildah.InitReexec() {
    return
  }
  unshare.MaybeReexecUsingUserNamespace(false)
```

这段代码使得能够使用 `unshare` 包，该包可通过 `github.com/containers/storage/pkg/unshare` 获取。

为了利用 Buildah 的构建功能，我们必须实例化 `buildah.Builder`。这个对象包含定义构建步骤、配置构建和最终运行构建的所有方法。

为了创建 `Builder`，我们需要一个叫做 `storage.Store` 的对象，它来自 `github.com/containers/storage` 包。这个元素负责存储中间的和最终的容器镜像。让我们来看一下我们正在讨论的代码块：

```
buildStoreOptions, err := storage.DefaultStoreOptions(unshare.IsRootless(), unshare.GetRootlessUID())
buildStore, err := storage.GetStore(buildStoreOptions)
```

从前面的示例中可以看到，我们获取了默认选项，并将其传递给 `storage` 模块来请求一个 `Store` 对象。

创建 `Builder` 所需的另一个元素是 `BuilderOptions` 对象。这个元素包含我们可能分配给 Buildah 的 `Builder` 的所有默认和自定义选项。让我们来看一下如何定义它：

```
builderOpts := buildah.BuilderOptions{
  FromImage:        "node:12-alpine", // Starting image
  Isolation:        define.IsolationChroot, // Isolation environment
  CommonBuildOpts:  &define.CommonBuildOptions{},
  ConfigureNetwork: define.NetworkDefault,
  SystemContext:    &types.SystemContext {},
}
```

在之前的代码块中，我们定义了一个 `BuilderOptions` 对象，其中包含以下内容：

+   我们将用来构建目标容器镜像的初始镜像：

    +   在这个例子中，我们选择了基于 Alpine Linux 发行版的 Node.js 镜像。这是因为，在我们的示例中，我们正在模拟构建一个 Node.js 应用程序的过程。

+   构建开始时采用的隔离模式。在这种情况下，我们将使用 chroot 隔离，它非常适合许多构建场景——隔离较少，但要求较低。

+   一些构建、网络和系统上下文的默认选项：

    +   `SystemContext` 对象定义了包含在配置文件中的信息作为参数。

现在我们已经拥有了所有必要的数据来实例化 `Builder`，让我们来做吧：

```
builder, err := buildah.NewBuilder(context.TODO(), buildStore, builderOpts)
```

如你所见，我们调用了 `NewBuilder` 函数，传入了在本节前面代码中创建的所有必需选项，以准备好 `Builder` 来创建我们的自定义容器镜像。

现在我们准备好给 `Builder` 指定所需的选项来创建自定义镜像，首先让我们将包含我们应用程序的 **JavaScript** 文件添加到容器镜像中，正是为了这个容器镜像我们正在进行创建：

```
err = builder.Add("/home/node/", false, buildah.AddAndCopyOptions{}, "script.js")
```

我们假设 JavaScript 主文件存储在我们正在编写并使用的 Go 模块旁边，并且我们将这个文件复制到 `/home/node` 目录中，这是基础容器镜像期望找到这种数据的默认路径。

我们将复制到容器镜像并用于此测试的 JavaScript 程序非常简单——让我们来查看一下：

```
var http = require("http");
http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello Podman and Buildah friends. This page is provided to you through a container running Node.js version: ");
  response.write(process.version);
  response.end();
}).listen(8080);
```

在不深入探讨 JavaScript 语言语法及其概念的情况下，我们可以注意到，从这个 JavaScript 文件来看，我们使用了 HTTP 库来监听 `8080` 端口上的传入请求，并用一个默认的欢迎信息来响应这些请求：`Hello Podman and Buildah friends. This page is provided to you through a container running Node.js`。我们还将 Node.js 的版本号附加到响应字符串中。

重要提示

请注意，JavaScript，也称为**JS**，是一种即时编译的高级编程语言。正如我们之前所说，我们既不会深入探讨 JavaScript 语言的定义，也不会深入 Node.js 这个最著名的运行时环境。

之后，我们配置了默认命令，以便为我们的自定义容器镜像运行：

```
builder.SetCmd([]string{"node", "/home/node/script.js"})
```

我们刚刚设置了执行 Node.js 执行运行时的命令，指向我们刚刚添加到容器镜像中的 JavaScript 程序。

为了提交我们所做的更改，我们需要获取正在处理的镜像引用。同时，我们还将定义`Builder`将创建的容器镜像名称：

```
imageRef, err := is.Transport.ParseStoreReference(buildStore, "podmanbook/nodejs-welcome")
```

现在，我们已经准备好提交更改并调用`Builder`的`commit`函数：

```
imageId, _, _, err := builder.Commit(context.TODO(), imageRef, define.CommitOptions{})
fmt.Printf("Image built! %s\n", imageId)
```

正如我们所看到的，我们只是请求`Builder`提交了更改，传递了我们之前获得的镜像引用，然后我们最终将其作为引用打印出来。

我们现在准备好运行我们的程序了！让我们执行它：

```
[builder]$ go run custombuilder.go 
Image built! e60fa98051522a51f4585e46829ad6a18df704dde774634dbc010baae440 4849
```

现在，我们可以测试刚刚构建的自定义容器镜像：

```
[builder]$ podman run -dt -p 8080:8080/tcp podmanbook/nodejs-welcome:latest
747805c1b59558a70c4a2f1a1d258913cae5ffc08cc026c74ad3ac21aab1 8974
[builder]$ curl localhost:8080
Hello Podman and Buildah friends. This page is provided to you through a container running Node.js version: v12.22.7
```

正如我们在前面的代码块中看到的，我们正在运行我们刚刚创建的容器镜像，并使用以下选项：

+   `-d`：分离模式，在后台运行容器

+   `-t`：分配一个新的伪终端

+   `-p`：将容器端口发布到主机系统

+   `podmanbook/nodejs-welcome:latest`：我们自定义容器镜像的名称

最后，我们使用`curl`命令行工具来请求并打印我们的 JavaScript 程序提供的 HTTP 响应，该程序被容器化到我们创建的自定义容器镜像中！

重要提示

本节描述的示例仅是 Buildah Go 模块为我们自定义镜像构建器启用的所有强大功能的简要概述。要了解更多有关各种功能、变量和代码文档的信息，您可以参考[`pkg.go.dev/github.com/containers/buildah`](https://pkg.go.dev/github.com/containers/buildah)上的文档。

正如我们在本节中看到的，Buildah 是一个非常灵活的工具，凭借其库，它可以在许多不同的场景中支持自定义构建器。

如果我们在互联网上搜索，可以找到许多关于 Buildah 支持创建自定义容器镜像的示例。让我们来看看其中的一些。

## Quarkus 原生可执行文件在容器中的运行

**Quarkus**被定义为一个 Kubernetes 原生 Java 栈，利用 OpenJDK（开放 Java 开发工具包）项目和 GraalVM 项目。GraalVM 是一个具有许多特殊功能的 Java 虚拟机，例如为快速启动和低内存占用编译 Java 应用程序。

重要提示

我们不会深入讨论 Quarkus、GraalVM 以及任何其他相关项目。我们将要深入研究的示例仅供参考。我们鼓励您通过浏览它们的网页和阅读相关文档来了解更多关于这些项目的信息。

如果我们查看 Quarkus 文档网页，我们很容易发现，在经过一个长时间的教程之后，我们可以学习如何构建一个 Quarkus 原生可执行文件，接着我们可以将该可执行文件打包并在容器镜像中执行。

Quarkus 文档中提供的步骤利用了带有特殊选项的 Maven 包装器。Maven 最初作为一个 Java 构建自动化工具诞生，但后来也扩展到了其他编程语言。如果我们快速查看这个命令，会注意到 Podman 的名字出现在其中：

```
$ ./mvnw package -Pnative -Dquarkus.native.container-build=true -Dquarkus.native.container-runtime=podman
```

这意味着 Maven 包装器程序将调用 Podman 构建来创建一个包含由 Quarkus 项目提供的预配置环境以及我们正在开发的二进制应用程序的容器镜像。

我们在选项中看到了 Podman 的名字。这是因为，正如我们在*第六章*中所看到的，*了解 Buildah——从零构建容器*，Podman 通过使用其库来借用 Buildah 的构建逻辑。

要进一步探索这个示例，我们可以查看[`quarkus.io/guides/building-native-image`](https://quarkus.io/guides/building-native-image)。

## Rust 语言的 Buildah 包装器

另一个通过 Buildah 库或 CLI 制作的酷工具示例是 Rust 编程语言的 Buildah 包装器。Rust 是一种类似 C++的编程语言，旨在提供性能和安全的并发。其主项目页面可以在以下网址找到：[`github.com/Dennis-Krasnov/Buildah-Rust`](https://github.com/Dennis-Krasnov/Buildah-Rust)。

这个 Buildah 包装器利用 Rust 包管理器**Cargo**来下载所需的依赖项，将其编译成一个包，并使其可分发。

重要提示

我们不会深入 Rust、Cargo 以及其他相关项目的细节。我们将深入探讨的示例仅供参考。我们鼓励你通过浏览这些项目的网页并阅读相关文档，来了解更多关于这些项目的知识。

项目主页中的示例非常简单，如下所示的代码块：

```
$ cd examples/
$ cargo run --example nginx
$ podman run --rm -it -p 8080:80 nginx_rust
```

第一个命令，在选择名为`examples`的目录后，执行一个简单的代码块，创建容器所需的代码，而第二个命令则通过 Buildah 本身测试刚刚由 Buildah 包装器创建的容器镜像。

我们可以查看在之前代码块中的第一个命令使用的 Rust 代码。第一个命令执行的是`nginx.rs`文件中的一小段代码：

```
use buildah_rs::container::Container;
fn main() {
    let mut container = Container::from("nginx:1.21");
    container.copy("html", "/usr/share/nginx/html").unwrap();
    container.commit("nginx_rust").unwrap();
}
```

如前所述，我们不会深入探讨代码语法或库本身；无论如何，代码相当简单，它只是导入了 Buildah 包装器库，从`nginx:1.21`创建容器镜像，最后将本地的`html`目录复制到容器镜像的目标路径。

要进一步探索这个示例，查看[`github.com/Dennis-Krasnov/Buildah-Rust`](https://github.com/Dennis-Krasnov/Buildah-Rust)。

本节到此结束。通过许多有用的示例，我们了解了如何在不同的场景中集成 Buildah，以支持我们项目的容器镜像自定义构建器。

# 总结

在本章中，我们学习了如何在一些高级场景中利用 Podman 的伙伴工具 Buildah 来支持我们的开发项目。

我们了解了如何使用 Buildah 进行多阶段容器镜像创建，这使得我们可以使用不同的 `FROM` 指令创建多个阶段的构建，随后拥有从之前阶段抓取内容的镜像。

然后，我们发现有许多使用场景需要容器化构建。如今，最常见的采用场景之一是应用构建工作流运行在 Kubernetes 集群之上。基于此，我们深入探讨了容器化 Buildah。

最后，通过大量有趣的示例，我们学习了如何集成 Buildah 来创建容器镜像的自定义构建器。正如本章所示，实际上有几种选择和方法可以使用 Podman 生态系统工具来构建容器镜像，而且大多数时候，我们通常是从基础镜像开始，定制和扩展前一个操作系统层，以适应我们的使用场景。

在下一章中，我们将进一步了解容器基础镜像，如何选择它们，以及在选择时需要注意的事项。

# 进一步阅读

+   CNCF 项目列表: [`landscape.cncf.io/`](https://landscape.cncf.io/)

+   *在容器中运行 Buildah 的最佳实践*: [`developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container`](https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container)

+   Buildah Go 模块文档: [`pkg.go.dev/github.com/containers/buildah`](https://pkg.go.dev/github.com/containers/buildah)

+   Quarkus 本地可执行文件: [`quarkus.io/guides/building-native-image`](https://quarkus.io/guides/building-native-image)

+   Buildah Rust 语言封装: [`github.com/Dennis-Krasnov/Buildah-Rust`](https://github.com/Dennis-Krasnov/Buildah-Rust)
