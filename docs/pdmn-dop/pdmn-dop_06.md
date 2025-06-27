# 第五章：为容器的数据实现存储

在之前的章节中，我们探讨了如何使用 Podman 运行和管理容器，但我们很快会意识到，在本章中，这些操作在某些场景下并不有用，特别是当容器中的应用程序需要以持久方式存储数据时。容器默认是短暂的，这是它们的主要特性之一，正如我们在本书的第一章中描述的那样，因此我们需要一种方法，将持久存储附加到运行中的容器，以保存容器的重要数据。

本章我们将讨论以下主要内容：

+   为什么存储对容器很重要？

+   容器的存储特性

+   将文件复制进出容器

+   将主机存储附加到容器

# 技术要求

在继续本章的讲解和示例之前，需要一台安装了 Podman 的工作机器。如*第三章*中所述，*运行第一个容器*，本书中的所有示例都在 Fedora 34 系统或更高版本上执行，但也可以在读者选择的操作系统上重现。

最后，充分理解*第四章*中讨论的主题，*管理正在运行的容器*，对于能够轻松掌握 OCI 镜像和容器执行的概念非常有用。

# 为什么存储对容器很重要？

在继续本章并回答这个有趣的问题之前，我们需要区分容器的两种存储类型：

+   附加到正在运行的容器的外部存储，用于存储数据，使数据在容器重启时保持持久性

+   用于容器根文件系统和容器镜像的底层存储

讨论外部存储，如我们在*第一章*中描述的那样，*容器技术概述*，容器是无状态的、短暂的，并且通常具有只读文件系统。这是因为该技术背后的理论认为，容器应该用于创建可扩展的、分布式的应用程序，这些应用程序必须进行横向扩展，而不是纵向扩展。

横向扩展应用程序意味着，当我们需要为正在运行的服务增加额外资源时，我们不会增加单个运行容器的 CPU 或 RAM，而是启动一个全新的容器，新的容器将与现有容器一起处理传入的请求。这与公有云中采用的著名范式相同。原则上，容器应该是短暂的，因为任何现有容器镜像的额外副本应该随时运行，以增强现有的运行服务。

当然，存在一些例外情况，可能发生正在运行的容器无法水平扩展，或者容器在启动时或运行时需要共享配置、缓存或与其他相同容器镜像副本相关的任何数据。

让我们通过一个现实生活中的例子来理解这一点。使用共享汽车服务，每次前往城市中的目的地时都可以获得一辆新车，这是一种非常有用且智能的出行方式，既能避免停车费、油费和其他问题的困扰。然而，另一方面，这项服务并不允许你将物品存放在停放的车内。因此，使用共享汽车服务时，我们可以在进入车内时拿出物品，但在离开车之前，我们必须把物品收拾好。同样的道理也适用于容器，我们必须为容器附加存储，以允许容器写入数据，但当容器停止时，我们应该卸载存储，以便新的容器在需要时使用。

这里有一个更技术性的例子：假设我们有一个标准的三层应用程序，包含 Web 服务、后端服务和数据库服务。这个应用程序的每一层可能都需要存储，并以各种方式使用这些存储。Web 服务可能需要一个地方来保存缓存、存储渲染后的网页、运行时的一些自定义图像等等。后端服务需要一个地方来存储配置数据，以及与其他运行中的后端服务之间的同步数据（如果有的话）等等。数据库服务肯定需要一个地方来存储数据库数据。

存储通常与低级基础设施相关联，但在容器中，存储变得对开发人员同样重要，开发人员需要规划存储附加的位置，以及其应用程序所需的功能。

如果我们将话题扩展到容器编排，那么存储就承担了战略性角色，因为它应该像我们可能使用的 Kubernetes 编排器一样具备弹性和可行性。在这种情况下，容器存储应更像是软件定义的存储——能够以自助服务的方式为开发人员和容器提供存储资源。

尽管本书将讨论本地存储，但值得注意的是，对于 Kubernetes 编排器来说，这还不够，因为容器应该能够根据定义的可用性和扩展规则，从一个主机迁移到另一个主机。这就是软件定义存储可能成为解决方案的地方！

从前面的例子中我们可以推断出，外部存储在容器中至关重要。其使用可能会根据我们容器中运行的应用程序不同而有所不同，但它是必需的。同时，另一个关键角色是由底层容器存储驱动的，它负责处理容器的正确存储以及容器镜像根文件系统的存储。选择正确、稳定且性能良好的底层本地存储，将确保更好的容器管理。

所以，让我们首先探索一下容器存储的理论，然后讨论如何使用它。

# 容器存储功能

在进入实际的示例和用例之前，我们应该首先深入了解容器存储与 **容器存储接口**（**CSI**）之间的主要区别。

容器存储，之前被称为 *底层容器存储*，负责处理 **写时复制**（**COW**）文件系统上的容器镜像。容器镜像需要传输并移动，直到容器引擎被指示运行它们，因此我们需要一种方法来存储镜像，直到它被运行。这就是容器存储的作用。

一旦我们开始使用 Kubernetes 等调度器，CSI 负责提供容器所需的块存储或文件存储，用于容器写入数据。

本章的下一部分，我们将重点讨论容器存储及其配置。稍后，我们将讨论容器的外部存储以及 Podman 提供的将主机本地存储暴露给运行中的容器的选项。

Podman 引入的一项重要创新是 *containers/storage* 项目（[`github.com/containers/storage`](https://github.com/containers/storage)），它为共享一种访问主机容器存储的底层通用方法提供了一个很好的方式。随着 Docker 的出现，我们不得不通过 Docker 守护进程来与容器存储进行交互。由于没有其他直接与底层存储交互的方式，Docker 守护进程只会将其隐藏在用户和系统管理员面前。

通过 *containers/storage* 项目，我们现在可以轻松地同时使用多个工具来分析、管理或处理容器存储。

这个低级软件的配置对于 Podman 以及其他 Podman 配套工具至关重要，可以通过其配置文件 `/etc/containers/storage.conf` 进行检查或编辑。

看一下配置文件，我们可以轻松发现，我们可以改变许多选项，来决定容器如何与底层存储交互。让我们检查一下最重要的选项——存储驱动程序。

## 存储驱动程序

配置文件作为其第一个选项之一，提供了选择默认 **写时复制**（**COW**）容器存储驱动程序的机会。在本书写作时，当前版本的配置文件支持以下 COW 驱动程序：

+   overlay

+   vfs

+   devmapper

+   aufs

+   btrfs

+   zfs

这些驱动程序通常也被称为 **图形驱动程序**，因为它们大多数通过图形结构组织它们处理的层。

在 Fedora 34 或更高版本上使用 Podman 时，容器的存储配置文件默认使用 overlay 作为驱动程序。

另一个需要提到的重要点是，在本书写作时，overlay 文件系统有两个版本——版本 1 和版本 2。

原始的 Overlay 文件系统版本 1 最初由 Docker 容器引擎使用，但后来被版本 2 替代。这就是为什么 Podman 和容器的存储配置文件通常使用 `overlay` 这个名字，但实际使用的是新的版本 2。

在详细了解本章中的其他选项和实际示例之前，让我们进一步探讨这些 COW 文件系统驱动程序中的一个是如何工作的。

Overlay 联合文件系统从 Linux 内核版本 3.18 开始就已存在。它通常默认启用，并且在初始化挂载时动态激活。

这个文件系统背后的机制非常简单，但却非常强大——它允许将一个目录树叠加到另一个目录上，仅存储差异，但展示最新更新的、*合并*后的目录树。

通常，在容器世界里，我们会开始使用一个只读文件系统，添加一个或多个只读层，再次变为只读，直到运行中的容器将这些 *合并* 层作为其根文件系统。这时，将创建最后一个读写层，作为其他层的叠加。

让我们看看当我们用 Podman 拉取一个全新的容器镜像时，幕后发生了什么：

重要说明

如果你希望在测试机器上继续测试下面的示例，请确保删除任何正在运行的容器和容器镜像，这样可以更轻松地将镜像与 Podman 下载的层匹配。

```
# podman pull quay.io/centos7/httpd-24-centos7:latest
Trying to pull quay.io/centos7/httpd-24-centos7:latest...
Getting image source signatures
Copying blob 5f2e13673ac2 done  
Copying blob 8dd5a5013b51 done  
Copying blob b2cc5146c9c7 done  
Copying blob e17e89f32035 done  
Copying blob 1b6c93aa6be5 done  
Copying blob 6855d3fe68bc done  
Copying blob f974a2323b6c done  
Copying blob d620f14a5a76 done  
Copying config 3b964f33a2 done  
Writing manifest to image destination
Storing signatures
3b964f33a2bf66108d5333a541d376f63e0506aba8ddd4813f9d4e104 271d9f0
```

从之前命令的输出中，我们可以看到已经下载了多个层。这是因为我们拉取的容器镜像由多个层组成。

现在我们可以开始检查下载的层。首先，我们需要找到正确的目录，可以通过在配置文件中搜索来定位。或者，我们可以使用一种更简单的方法。Podman 有一个命令专门用于显示其运行配置和其他有用信息——`podman info`。让我们看看它是如何工作的：

```
# podman info | grep -A19 "store:"
store:
  configFile: /etc/containers/storage.conf
  containerStore:
    number: 0
    paused: 0
    running: 0
    stopped: 0
  graphDriverName: overlay
  graphOptions:
    overlay.mountopt: nodev,metacopy=on
  graphRoot: /var/lib/containers/storage
  graphStatus:
    Backing Filesystem: btrfs
    Native Overlay Diff: "false"
    Supports d_type: "true"
    Using metacopy: "true"
  imageStore:
    number: 1
  runRoot: /run/containers/storage
  volumePath: /var/lib/containers/storage/volumes
```

为了减少 `podman info` 命令的输出，我们使用了 `grep` 命令，仅匹配包含当前容器存储配置的 `store` 部分。

正如我们所看到的，使用的驱动程序是 `overlay`，要搜索层的根目录被报告为 `graphRoot` 目录：`/var/lib/containers/storage`；对于无根容器，相应的目录是 `$HOME/.local/share/containers/storage`。我们还可以看到其他路径，但这些路径将在本节后面讨论。`graph` 这个关键词来自我们之前介绍过的驱动类别。

让我们看看这个目录，看看里面到底有什么内容：

```
# cd /var/lib/containers/storage
# ls
libpod  mounts  overlay  overlay-containers  overlay-images  overlay-layers  storage.lock  tmp  userns.lock
```

我们可以看到几个可用的目录，它们的名称很直观。我们要寻找的是以下这些：

+   `overlay-images`：此目录包含下载的容器镜像的元数据。

+   `overlay-layers`：这里包含了每个容器镜像所有层的归档文件。

+   `overlay`：这是包含每个容器镜像解压层的目录。

让我们检查第一个目录`overlay-images`的内容：

```
# ls -l overlay-images/
total 8
drwx------. 1 root root  630 15 oct 18.36 3b964f33a2bf66108d5333a541d376f63e0506aba8ddd4813f9d4e10427 1d9f0
-rw-------. 1 root root 1613 15 oct 18.36 images.json
-rw-r--r--. 1 root root   64 15 oct 18.36 images.lock
```

正如我们所想，在这个目录中，我们可以找到我们拉取下来的唯一容器镜像的元数据，并且在那个 ID 非常长的目录中，我们将找到描述构成容器镜像层的清单文件。

现在让我们检查第二个目录`overlay-layers`的内容：

```
# ls -l overlay-layers/
total 1168
-rw-------. 1 root root   2109 15 oct 18.35 0099baae6cd3ca0ced38d658d7871548b32bd0e42118b788d818b76131ec 8e75.tar-split.gz
-rw-------. 1 root root 795206 15 oct 18.35 53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309 aee.tar-split.gz
-rw-------. 1 root root  52706 15 oct 18.35 6c26feaaa75c7bac1f1247acc06e73b46e8aaf2e741ad1b8bacd6774bffdf6 ba.tar-split.gz
-rw-------. 1 root root   1185 15 oct 18.35 74fa1495774e94d5cdb579f9bae4a16bd90616024a6f4b1ffd13344c367df1 f6.tar-split.gz
-rw-------. 1 root root 308144 15 oct 18.36 ae314017e4c2de17a7fb007294521bbe8ac1eeb004ac9fb57d1f1f03090f78 c9.tar-split.gz
-rw-------. 1 root root   1778 15 oct 18.36 beba3570ce7dd1ea38e8a1b919a377b6dc888b24833409eead446bff401d8f 6e.tar-split.gz
-rw-------. 1 root root    697 15 oct 18.36 e59e7d1e1874cc643bfe6f854a72a39f73f22743ab38eff78f91dc019cca91 f5.tar-split.gz
-rw-------. 1 root root   5555 15 oct 18.36 e5a13564f9c6e233da30a7fd86489234716cf80c317e52ff8261bf0cb34dc 7b4.tar-split.gz
-rw-------. 1 root root   3716 15 oct 18.36 layers.json
-rw-r--r--. 1 root root     64 15 oct 19.06 layers.lock
```

正如我们所看到的，我们刚刚找到了为容器镜像下载的所有层的归档文件，但它们被解压到哪里了呢？答案很简单——在第三个文件夹`overlay`中：

```
# ls -l overlay
total 0
drwx------. 1 root root  46 15 oct 18.35 0099baae6cd3ca0ced38d658d7871548b32bd0e42118b788d818b76131ec 8e75
drwx------. 1 root root  46 15 oct 18.35 53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e23230 9aee
drwx------. 1 root root  46 15 oct 18.35 6c26feaaa75c7bac1f1247acc06e73b46e8aaf2e741ad1b8bacd6774bffd f6ba
drwx------. 1 root root  46 15 oct 18.35 74fa1495774e94d5cdb579f9bae4a16bd90616024a6f4b1ffd13344c367d f1f6
drwx------. 1 root root  46 15 oct 18.35 ae314017e4c2de17a7fb007294521bbe8ac1eeb004ac9fb57d1f1f03090f 78c9
drwx------. 1 root root  46 15 oct 18.36 beba3570ce7dd1ea38e8a1b919a377b6dc888b24833409eead446bff401d 8f6e
drwx------. 1 root root  46 15 oct 18.36 e59e7d1e1874cc643bfe6f854a72a39f73f22743ab38eff78f91dc019cca 91f5
drwx------. 1 root root  46 15 oct 18.36 e5a13564f9c6e233da30a7fd86489234716cf80c317e52ff8261bf0cb34d c7b4
drwx------. 1 root root 416 15 oct 18.36 l
```

当查看最新目录内容时，可能会产生的第一个问题是，`l`（小写 L）目录的用途是什么？

为了回答这个问题，我们必须检查一个层目录的内容。我们可以从列表中的第一个开始：

```
# ls -la overlay/0099baae6cd3ca0ced38d658d7871548b32bd0e42118b788d818b 76131ec8e75/
total 8
drwx------. 1 root root   46 15 oct 18.35 .
drwx------. 1 root root 1026 15 oct 18.36 ..
dr-xr-xr-x. 1 root root   24 15 oct 18.35 diff
-rw-r--r--. 1 root root   26 15 oct 18.35 link
-rw-r--r--. 1 root root   86 15 oct 18.35 lower
drwx------. 1 root root    0 15 oct 18.35 merged
drwx------. 1 root root    0 15 oct 18.35 work
```

让我们理解这些文件和目录的用途：

+   `diff`：这个目录表示叠加层的上层，用于存储对该层的任何更改。

+   `lower`：这个文件报告所有下层的挂载情况，按从上到下的顺序排列。

+   `merged`：该目录是叠加层挂载到的目录。

+   `work`：该目录用于内部操作。

+   `link`：这个文件包含层的唯一字符串。

现在，回到我们的问题，`l`（小写 L）目录的用途是什么？

在`l`目录下，有指向每个层的`diff`目录的符号链接。符号链接在`lower`文件中引用了下层。让我们检查一下：

```
# ls -la overlay/l/
total 32
drwx------. 1 root root  416 15 oct 18.36 .
drwx------. 1 root root 1026 15 oct 18.36 ..
lrwxrwxrwx. 1 root root   72 15 oct 18.35 A4ZYMM4AK5NM6JYJA7 EK2DLTGA -> ../74fa1495774e94d5cdb579f9bae4a16bd90616024a6f4 b1ffd13344c367df1f6/diff
lrwxrwxrwx. 1 root root   72 15 oct 18.35 D2WVDYIWL6I77ZOIXR VQKCXNG2 -> ../ae314017e4c2de17a7fb007294521bbe8ac1eeb004ac9fb57d1f1f03090 f78c9/diff
lrwxrwxrwx. 1 root root   72 15 oct 18.36 G4KXMAOCE56TIB252 ZMWEFRFHU -> ../beba3570ce7dd1ea38e8a1b919a377b6dc888b24833409eead446bff401 d8f6e/diff
lrwxrwxrwx. 1 root root   72 15 oct 18.35 JHHF5QA7YSKDSKRSC HNADBVKDS -> ../53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e2 32309aee/diff
lrwxrwxrwx. 1 root root   72 15 oct 18.36 KNCK5EDUAQJDAIDWQ6 TWDFQF5B -> ../e59e7d1e1874cc643bfe6f854a72a39f73f22743ab38eff78f91dc019cca 91f5/diff
lrwxrwxrwx. 1 root root   72 15 oct 18.35 LQUM7XDVWHIJRLIWAL CFKSMJTT -> ../0099baae6cd3ca0ced38d658d7871548b32bd0e42118b788d818b76131 ec8e75/diff
lrwxrwxrwx. 1 root root   72 15 oct 18.35 V6OV3TLBBLTATIJDCTU 6N72XQ5 -> ../6c26feaaa75c7bac1f1247acc06e73b46e8aaf2e741ad1b8bacd6774bf fdf6ba/diff
lrwxrwxrwx. 1 root root   72 15 oct 18.36 ZMKJYKM2VJEAYQHCI7SU Q2R3QW -> ../e5a13564f9c6e233da30a7fd86489234716cf80c317e52ff8261bf0cb34dc7 b4/diff
```

为了再次确认我们刚刚学到的内容，让我们找到容器镜像的第一层，并检查是否有`lower`文件。

让我们检查我们容器镜像的清单文件：

```
# cat overlay-images/3b964f33a2bf66108d5333a541d376f63e0506ab a8ddd4813f9d4e104271d9f0/manifest | head -15
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1 +json",
      "size": 16212,
      "digest": "sha256:3b964f33a2bf66108d5333a541d376f63e0506aba8ddd4813f9d4 e104271d9f0"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 75867345,
         "digest": "sha256:b2cc5146c9c7855cb298ca8b77ecb153d37e3e5c69916ef42361 3a46a70c0503"
      },
```

然后，我们必须将压缩归档文件的校验和与我们下载的所有层的列表进行比较：

知识点

SHA-256 是一种算法，用于生成唯一的加密哈希值，可以用来验证文件的完整性（校验和）。

```
# cat overlay-layers/layers.json | jq | grep -B3 -A10 "sha256:b2cc5"
  {
    "id": "53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e23 2309aee",
    "created": "2021-10-15T16:35:49.782784856Z",
    "compressed-diff-digest": "sha256:b2cc5146c9c7855cb298ca8b77ecb153d37e3e5c69916ef423 613a46a70c0503",
    "compressed-size": 75867345,
    "diff-digest": "sha256:53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a448 3b51e232309aee",
    "diff-size": 211829760,
    "compression": 2,
    "uidset": [
      0,
      192
    ],
    "gidset": [
      0,
```

我们刚刚分析的文件`overlay-layers/layers.json`没有进行缩进。因此，我们使用`jq`工具来格式化它，使其更易于人类阅读。

知识点

如果你在系统上找不到`jq`工具，可以通过操作系统的默认包管理器安装它。例如，在 Fedora 上，你可以运行`dnf install jq`来安装。

正如你所看到的，我们刚刚找到了根层的 ID。现在，让我们看看它的内容：

```
# ls -l overlay/53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a448 3b51e232309aee/
total 4
dr-xr-xr-x. 1 root root 158 15 oct 18.35 diff
drwx------. 1 root root   0 15 oct 18.35 empty
-rw-r--r--. 1 root root  26 15 oct 18.35 link
drwx------. 1 root root   0 15 oct 18.35 merged
drwx------. 1 root root   0 15 oct 18.35 work
```

正如我们可以验证的那样，在层的目录中没有`lower`文件，因为这是我们容器镜像的第一层！

我们可能注意到的不同之处是有一个名为`empty`的目录。这是因为如果某一层没有父层，那么叠加系统将创建一个名为`empty`的虚拟下层目录，并跳过写入`lower`文件。

最后，作为我们实践示例的最后阶段，让我们运行容器并验证会创建一个新的`diff`层。我们预计这个层将只包含与下层的差异。

首先，我们运行我们刚刚分析的容器镜像：

```
# podman run -d quay.io/centos7/httpd-24-centos7
bd0eef7cd50760dd52c24550be51535bc11559e52eea7d782a1fa69 76524fa76
```

如您所见，我们通过`-d`选项在后台启动了容器，这样我们可以继续在系统主机上工作。之后，我们将在该 Pod 上执行一个新的 shell 来实际检查容器的根文件夹并在其中创建一个新文件：

```
# podman exec -ti bd0eef7cd50760dd52c24550be51535bc11559e52eea7d782a1fa69 76524fa76 /bin/bash
bash-4.2$ pwd
/opt/app-root/src
bash-4.2$ echo "this is my NOT persistent data" > tempfile.txt
bash-4.2$ ls
tempfile.txt
```

我们刚刚创建的这个新文件是临时性的，且只在容器的生命周期内存在。现在是时候找到我们主机系统上由覆盖驱动程序刚刚创建的`diff`层了。最简单的方法是分析正在运行的容器中使用的挂载点：

```
bash-4.2$ mount | head
overlay on / type overlay (rw,relatime,context="system_u:object_r:container_file_t:s0:c300,c861",lowerdir=/var/lib/containers/storage/overlay/l/ZMKJYKM2VJEAYQHCI7SUQ2R3QW:/var/lib/containers/storage/overlay/l/G4KXMAOCE56TIB252ZMWEFRFHU:/var/lib/containers/storage/overlay/l/KNCK5EDUAQJDAIDWQ6TWDF QF5B:/var/lib/containers/storage/overlay/l/D2WVDYIWL6I77ZOIX RVQKCXNG2:/var/lib/containers/storage/overlay/l/LQUM7XDVWHI JRLIWALCFKSMJTT:/var/lib/containers/storage/overlay/l/A4ZYMM4AK5NM6JYJA7EK2DLTGA:/var/lib/containers/storage/overlay/l/V6OV3TLBBLTATIJDCTU6N72XQ5:/var/lib/containers/storage/overlay/l/JHHF5QA7YSKDSKRSCHNADBVKDS,upperdir=/var/lib/containers/storage/overlay/b71e4bea5380ca233bf6b0c7a1c276179b841e263ee293e987c6cc54 af516f23/diff,workdir=/var/lib/containers/storage/overlay/b71e4bea5380ca233bf6b0c7a1c276179b841e263ee293e987c6cc54af 516f23/work,metacopy=on)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,context="system_u:object _r:container_file_t:s0:c300,c861",size=65536k,mode=755,inode64)
```

如您所见，列表中的第一个挂载点显示了一长串用冒号分隔的层路径。在这条长长的路径中，我们可以找到正在寻找的`upperdir`目录：

```
upperdir=/var/lib/containers/storage/overlay/b71e4bea5380ca233bf6b0c7a1c276179b841e263ee293e987c6cc54af5 16f23/diff
```

现在，我们可以检查这个目录的内容，并浏览可用的各种路径，以找到我们在前面命令中写入该文件的容器根目录：

```
# ls -la /var/lib/containers/storage/overlay/b71e4bea5380ca233bf6b0c7a1c276179b841e263ee293e987c6cc54af5 16f23/diff/opt/app-root/src/
total 12
drwxr-xr-x. 1 1001 root   58 16 oct 00.40 .
drwxr-xr-x. 1 1001 root   12 22 set 10.39 ..
-rw-------. 1 1001 root   81 16 oct 00.46 .bash_history
-rw-------. 1 1001 root 1024 16 oct 00.38 .rnd
-rw-r--r--. 1 1001 root   31 16 oct 00.39 tempfile.txt
# cat /var/lib/containers/storage/overlay/b71e4bea5380ca233bf6b0c7a1c276179b841e263ee293e987c6cc54af5 16f23/diff/opt/app-root/src/tempfile.txt 
this is my NOT persistent data
```

正如我们验证过的那样，数据存储在主机操作系统上，但它存储在一个临时层中，一旦容器被删除，它就会被删除！

现在，回到最初让我们走到这个覆盖存储驱动程序底层的小旅程的主题，我们在谈论的是`/etc/containers/storage.conf`。这个文件包含了所有关于*containers/storage*项目的配置，负责为在主机上访问容器存储提供共享的底层通用方法。

这个文件中可用的其他选项与存储驱动程序的定制以及更改内部存储目录的默认路径相关。

我们应该简要谈论的最后一点是`runroot`目录。在这个文件夹中，容器存储程序将存储容器生成的所有临时可写内容。

如果我们检查在前一个示例中启动容器时我们主机上的文件夹，我们会发现有一个以容器 ID 命名的文件夹，里面有多个已挂载到容器上的文件，这些文件替换了原始文件：

```
# ls -l /run/containers/storage/overlay-containers/bd0eef7cd50760dd52c24550be51535bc11559e52eea7d782a1fa69765 24fa76/userdata
total 20
-rw-r--r--. 1 root root   6 16 oct 00.38 conmon.pid
-rw-r--r--. 1 root root  12 16 oct 00.38 hostname
-rw-r--r--. 1 root root 230 16 oct 00.38 hosts
-rw-r--r--. 1 root root   0 16 oct 00.38 oci-log
-rwx------. 1 root root   6 16 oct 00.38 pidfile
-rw-r--r--. 1 root root  34 16 oct 00.38 resolv.conf
drwxr-xr-x. 3 root root  60 16 oct 00.38 run
```

如前所示，`runroot`路径下的容器文件夹包含了多个文件，这些文件已经直接挂载到容器中以进行定制。

总结一下，在之前的示例中，我们分析了容器镜像的结构，以及当我们从该镜像运行一个新容器时会发生什么。幕后技术令人惊叹，我们看到许多功能与操作系统提供的隔离能力相关。在这里，存储提供了其他重要功能，使得容器成为了我们现在所熟知的最伟大的技术。

# 复制文件进出容器

Podman 允许用户将文件进出运行中的容器。这一结果是通过使用`podman cp`命令实现的，该命令可以将文件和文件夹移动到容器内外。它的使用非常简单，接下来的示例将展示这一点。

首先，让我们启动一个新的 Alpine 容器：

```
$ podman run -d --name alpine_cp_test alpine sleep 1000
```

现在，让我们从容器中获取一个文件——我们选择了`/etc/os-release`文件，它提供了一些关于操作系统发行版及其版本 ID 的信息：

```
$ podman cp alpine_cp_test:/etc/os-release /tmp
```

文件已经被复制到主机的`/tmp`文件夹中，可以进行检查：

```
$ cat /tmp/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.14.2
PRETTY_NAME="Alpine Linux v3.14"
HOME_URL=https://alpinelinux.org/
BUG_REPORT_URL="https://bugs.alpinelinux.org/"
```

在相反的方向，我们可以将文件或文件夹从主机复制到正在运行的容器中：

```
$ podman cp /tmp/build_folder alpine_cp_test:/
```

这个示例将`/tmp/build_folder`文件夹及其所有内容复制到 Alpine 容器的根文件系统下。然后，我们可以使用`podman exec`命令结合`ls`工具来检查复制命令的结果。

## 与 overlayfs 交互

还有另一种从容器复制文件到主机的方法，那就是使用`podman mount`命令并直接与合并的覆盖层交互。

要挂载一个正在运行的无根容器的文件系统，我们首先需要运行`podman unshare`命令，这允许用户在修改后的用户命名空间内运行命令：

```
$ podman unshare
```

该命令在一个新用户命名空间中启动一个根 shell，并配置了*UID 0*和*GID 0*。现在可以运行`podman mount`命令，并获取挂载点的绝对路径：

```
# cd $(podman mount alpine_cp_test)
```

上述命令使用了 Shell 扩展，切换到`MergedDir`的路径，正如其名所示，它将`LowerDir`和`UpperDir`的内容合并，提供一个统一的视图，展示不同层的内容。从现在开始，我们可以复制文件到容器的根文件系统，也可以从容器复制文件。

之前的示例基于无根容器，但相同的逻辑适用于有根容器。让我们启动一个有根的 Nginx 容器：

```
$ sudo podman run -d \
  --name rootful_nginx docker.io/library/nginx 
```

要复制文件进出容器，我们需要在命令前加上`sudo`命令：

```
$ sudo podman cp \
  rootful_nginx:/usr/share/nginx/html/index.html /tmp
```

上述命令将默认的`index.html`页面复制到主机的`/tmp`目录。请记住，`sudo`会提升用户权限至 root，因此复制的文件将具有*UID 0*和*GID 0*的所有权。

从容器中复制文件和文件夹的做法对于故障排除特别有用。将它们复制到正在运行的容器内的相反操作，对于更新和测试机密或配置文件也很有用。在这种情况下，我们可以选择保存这些更改，具体方法将在下一小节中描述。

### 使用`podman commit`持久化更改

之前的示例并不是一种永久定制正在运行的容器的方法，因为容器的不可变性意味着持久性修改应该通过镜像重建来实现。

然而，如果我们需要保存更改并生成一个新的镜像而不重新构建，可以使用`podman commit`命令将容器中的更改保存到一个新的镜像中。

提交（commit）概念在 Docker 和 OCI 镜像构建中非常重要。事实上，我们可以将 Dockerfile 的不同步骤视为构建过程中应用的一系列提交。

以下示例展示了如何将复制到运行中的容器中的文件持久化，并生成一个新镜像。假设我们想要更新 Nginx 容器的默认 `index.html` 页面：

```
$ echo "Hello World!" > /tmp/index.html
$ podman run --name custom_nginx -d -p \  
  8080:80 docker.io/library/nginx 
$ podman cp /tmp/index.html \
  custom_nginx:/usr/share/nginx/html/
```

让我们测试应用的更改：

```
$ curl localhost:8080
Hello World!
```

现在我们希望将更改后的 `index.html` 文件持久化到一个新镜像中，从运行中的容器使用 `podman commit` 开始：

```
$ podman commit -p custom_nginx hello-world-nginx
```

前述命令通过有效地创建一个包含更新文件和文件夹的新镜像层来持久化更改。

之前的容器现在可以在测试新自定义镜像之前安全地停止和删除：

```
$ podman stop custom_nginx && podman rm custom_nginx
```

让我们测试一下新的自定义镜像，并检查更改后的 `index.html` 文件：

```
$ podman run -d -p 8080:80 --name hello_world \
  localhost/hello-world-nginx
$ curl localhost:8080
Hello World!
```

在本节中，我们学会了如何将文件复制到运行中的容器以及如何通过生成新镜像即时提交更改。

在下一节中，我们将学习如何通过引入**卷（volumes）**和**绑定挂载（bind mounts）**的概念将主机存储附加到容器。

# 将主机存储附加到容器

我们已经讨论过容器的不可变特性。从预构建镜像开始，当我们运行一个容器时，我们会在一层只读层的堆栈上实例化一个读/写层，采用写时复制（copy-on-write）的方法。

容器是基于有状态镜像的短暂对象。这意味着容器不应该存储数据在其中——如果容器崩溃或被删除，所有数据都会丢失。我们需要一种方法，将数据存储在一个独立的位置，并将其挂载到正在运行的容器中，在容器删除时保留数据，并准备好被新容器重用。

还有一个重要的警告不容忽视——**机密**和**配置文件**。当我们构建一个镜像时，可以将所需的所有文件和文件夹传递到镜像中。然而，将机密如证书或密钥封装到构建中并不是一个好做法。如果我们需要，例如，轮换一个证书，我们必须从头开始重新构建整个镜像。同样，改变一个位于镜像内的配置文件也意味着每次更改设置时都需要重新构建。

由于这些原因，OCI 规范支持**卷（volumes）**和**绑定挂载（bind mounts）**来管理附加到容器的存储。在接下来的章节中，我们将学习卷和绑定挂载是如何工作的，以及如何将它们附加到容器中。

## 管理和附加绑定挂载到容器

让我们从绑定挂载（bind mounts）开始，因为它们利用了 Linux 的原生功能。根据官方的 Linux 手册页，绑定挂载是*一种将文件系统层次结构的某部分重新挂载到其他地方的方法*。这意味着，通过使用绑定挂载，我们可以在主机的另一个挂载点下复制目录的视图。

在学习容器如何使用绑定挂载之前，先来看一个基本示例，我们将`/etc`目录绑定挂载到`/mnt`目录下：

```
$ sudo mount --bind /etc /mnt  
```

执行此命令后，我们将看到`/mnt`下的`/etc`目录的精确内容。要卸载，只需运行以下命令：

```
$ sudo umount /mnt
```

同样的概念可以应用于容器——Podman 可以将主机目录绑定挂载到容器内，并提供专门的 CLI 选项来简化挂载过程。

Podman 提供了两个可以用于绑定挂载的选项：`-v|--volume` 和 `–mount`。我们将更详细地介绍这两个选项。

### -v|--volume 选项

此选项使用紧凑的单字段参数来定义源主机目录和容器挂载点，模式为`/HOST_DIR:/CONTAINER_DIR`。以下示例将主机的`/host_files`目录挂载到容器内的`/mnt`挂载点：

```
$ podman run -v /host_files:/mnt docker.io/library/nginx
```

可以传递额外的参数来定义挂载行为；例如，将主机目录挂载为只读：

```
$ podman run –v /host_files:/mnt:ro \
   docker.io/library/nginx
```

使用`-v|--volume`选项的其他可行绑定挂载选项可以在运行命令的手册页（`man podman-run`）中找到。

### --mount 选项

此选项更加详细，因为它使用*key=value*语法来定义源和目标，以及挂载类型和额外的参数。该选项接受不同的挂载类型（绑定挂载、卷、tmpfs、镜像和 devpts），格式为`type=TYPE,source=HOST_DIR,destination=CONTAINER_DIR`。源和目标键可以分别用更短的`src`和`dst`替代。之前的示例可以重写为：

```
$ podman run \
  --mount type=bind,src=/host_files,dst=/mnt \
  docker.io/library/nginx
```

我们还可以通过添加额外的逗号传递额外的选项；例如，将主机目录挂载为只读：

```
$ podman run \
  --mount type=bind,src=/host_files,dst=/mnt,ro=true \
  docker.io/library/nginx
```

尽管绑定挂载非常简单易用，但它也有一些限制，在某些情况下可能会影响容器的生命周期。主机文件和目录必须在运行容器之前存在，并且必须根据需要设置权限，使其可读或可写。另一个需要注意的重要问题是，如果绑定挂载由文件或目录填充，绑定挂载会始终遮掩容器内的底层挂载点。**卷**是绑定挂载的有用替代方案，接下来会详细描述。

## 管理和附加卷到容器

卷是由容器引擎直接创建和管理的目录，并挂载到容器内的挂载点。它们为持久化容器生成的数据提供了一个很好的解决方案。

卷可以通过 `podman volume` 命令进行管理，使用该命令可以列出、检查、创建和删除系统中的卷。让我们从一个基本的示例开始，Podman 会在 Nginx 文档根目录上自动创建一个卷：

```
$ podman run -d -p 8080:80  --name nginx_volume1 -v /usr/share/nginx/html docker.io/library/nginx
```

这次，`–v`选项有一个只有一个项目的参数——文档根目录。在这种情况下，Podman 会自动创建一个卷并将其绑定挂载到目标挂载点。

为了证明新卷已创建，我们可以检查容器：

```
$ podman inspect nginx_volume1
[...omitted output...]
"Mounts": [
          {
                "Type": "volume",
                "Name": "2ed93716b7ad73706df5c6f56bda262920accec59e7b6642d36f938e936 d36d9",
                "Source": "/home/packt/.local/share/containers /storage/volumes/2ed93716b7ad73706df5c6f56bda262920accec59e7b6 642d36f93 8e936d36d9/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "",
                "Options": [
                    "nosuid",
                    "nodev",
                    "rbind"
                ],
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
[…omitted output]
```

在`Mounts`部分，我们列出了容器中挂载的对象。唯一的项目是一个`volume`类型的对象，具有生成的 UID 作为其`Name`，并且`Source`字段表示它在主机中的路径，而`Destination`字段则是容器内的挂载点。

我们可以通过`podman volume ls`命令再次检查卷是否存在：

```
$ podman volume ls
DRIVER VOLUME NAME
local  2ed93716b7ad73706df5c6f56bda262920accec59e7b6642d36f93 8e936d36d9
```

查看源路径时，我们会找到容器文档根目录中的默认文件：

```
$ ls -al 
/home/packt/.local/share/containers/storage/volumes/2ed93716b7ad73706df5c6f56bda262920accec59e7b6642d36f93 8e936d36d9/_data
total 16
drwxr-xr-x. 2 gbsalinetti gbsalinetti 4096 Sep  9 20:26 .
drwx------. 3 gbsalinetti gbsalinetti 4096 Oct 16 22:41 ..
-rw-r--r--. 1 gbsalinetti gbsalinetti  497 Sep  7 17:21 50x.html
-rw-r--r--. 1 gbsalinetti gbsalinetti  615 Sep  7 17:21 index.html
```

这演示了当创建一个空卷时，它会被填充到目标挂载点的内容中。当容器停止时，卷会保留下来并保存所有数据，在容器重新启动时可以被其他容器重新使用。

上述示例显示了一个具有生成 UID 的卷，但也可以选择附加卷的名称，如下例所示：

```
$ podman run -d -p 8080:80  --name nginx_volume2 -v nginx_vol:/usr/share/nginx/html docker.io/library/nginx
```

在上述示例中，Podman 创建了一个名为`nginx_vol`的新卷，并将其存储在默认的卷目录下。当创建命名卷时，Podman 无需生成 UID。

默认的卷目录在无根容器和有根容器中有不同的路径：

+   对于无根容器，默认的卷存储路径是`<USER_HOME>/.local/share/containers/storage/volumes`。

+   对于有根容器，默认的卷存储路径是`/var/lib/containers/storage/volumes`。

在这些路径中创建的卷在容器销毁后会被保留下来，并可以被其他容器重用。

要手动删除一个卷，可以使用`podman volume rm`命令：

```
$ podman volume rm nginx_vol
```

当处理多个卷时，`podman volume prune`命令会删除所有未使用的卷。以下示例会清理用户默认卷存储中的所有卷（即无根容器使用的存储）：

```
$ podman volume prune
```

下一个示例显示了如何使用`sudo`前缀删除有根容器使用的卷：

```
$ sudo podman volume prune
```

重要提示

不要忘记监控主机上累积的卷，因为它们会消耗磁盘空间，可能会被回收，并定期清理未使用的卷，以避免主机存储杂乱无章。

用户还可以在运行容器之前预先创建并填充卷。以下示例使用`podman create volume`命令创建挂载到 Nginx 文档根目录的卷，然后用一个测试的`index.html`文件填充它：

```
$ podman volume create custom_nginx
$ echo "Hello World!" >> $(podman volume inspect custom_nginx –format "{{ .Mountpoint }}")/index.html
```

现在，我们可以使用预填充的卷运行一个新的 Nginx 容器：

```
$ podman run -d -p 8080:80  --name nginx_volume3 -v custom_nginx:/usr/share/nginx/html docker.io/library/nginx
```

HTTP 测试显示了更新后的内容：

```
$ curl localhost:8080
Hello World!
```

这次，卷在一开始并非为空，且它用其中的内容覆盖了容器目标目录。

### 使用`--mount`选项挂载卷

与绑定挂载一样，我们可以自由选择使用`-v|--volume`和`--mount`选项。以下示例使用`--mount`标志运行 Nginx 容器：

```
$ podman run -d -p 8080:80  --name nginx_volume4 --mount type=volume,src=custom_nginx,dst=/usr/share/nginx/html docker.io/library/nginx
```

虽然`-v|--volume`选项简洁且广泛采用，但`--mount`选项的优点在于语法更清晰易懂，并且准确地说明了挂载类型。

### 卷驱动程序

之前的卷示例都基于同一个 `/usr/share/containers/containers.conf` 文件中的 `[engine.volume_plugins]` 部分，传递插件名称后跟文件或套接字路径。

本地卷驱动程序也可以用于挂载 `/data/db` 目录：

```
$ sudo podman volume create --driver local --opt type=nfs --opt o=addr=nfs-host.example.com,rw,context="system_u:object_r:container_file_t:s0" --opt device=:/opt/nfs-export nfs-volume
$ sudo podman run -d -v nfs-volume:/data/db docker.io/library/mongo
```

上述示例的前提是已经配置好 NFS 服务器，且该服务器能够被运行容器的主机访问。

### 构建中的卷

卷可以在镜像构建过程中预先定义。这让镜像维护者定义哪些容器目录会自动附加到卷上。为了理解这一概念，让我们检查一下这个最小的 Dockerfile：

```
FROM docker.io/library/nginx:latest
VOLUME /usr/share/nginx/html
```

对 `docker.io/library/nginx` 镜像唯一的修改是 **VOLUME** 指令，该指令定义了哪个目录应作为匿名卷在主机上外部挂载。这仅仅是元数据，卷仅会在容器从此镜像启动时创建。

如果我们构建镜像并运行一个基于示例 Dockerfile 的容器，我们可以看到一个自动创建的匿名卷：

```
$ podman build -t my_nginx .
$ podman run -d --name volumes_from_build my_nginx
$ podman inspect volumes_from_build --format "{{ .Mounts }}"
[{volume 4d6ac7edcb4f01add205523b7733d61ae4a5772786eacca68e49 72b20fd1180c /home/packt/.local/share/containers/storage/volumes/4d6ac7edcb4f01add205523b7733d61ae4a5772786eacca68e4972 b20fd1180c/_data /usr/share/nginx/html local  [nodev exec nosuid rbind] true rprivate}]
```

在没有明确创建卷选项的情况下，Podman 已经创建并挂载了容器卷。这种在构建时自动定义卷的做法，在所有需要持久化数据的容器中是常见的，比如数据库。

例如，`docker.io/library/mongo` 镜像已经配置好创建两个卷，一个用于 `/data/configdb`，另一个用于 `/data/db`。这种行为在最常见的数据库中也可以看到，包括 PostgreSQL、MariaDB 和 MySQL。

可以定义预先定义的匿名卷在容器启动时如何挂载。默认的 ID `--image-volume` 选项。以下示例启动一个 MongoDB 容器，并将其默认卷挂载为 tmpfs：

```
$ podman run -d --image-volume tmpfs docker.io/library/mongo
```

在 *第六章* 中，*认识 Buildah - 从零构建容器*，我们将更详细地介绍构建过程。我们现在用一个跨多个容器挂载卷的示例来结束这一小节。

### 跨容器挂载卷

卷的最大优点之一是它们的灵活性。例如，容器可以挂载来自已经运行的容器的卷，以共享相同的数据。为了实现这个结果，我们可以使用 `--volumes-from` 选项。以下示例启动一个 MongoDB 容器，并在 Fedora 容器上交叉挂载其卷：

```
$ podman run -d --name mongodb01 docker.io/library/mongo
$ podman run -it --volumes-from=mongodb01 docker.io/library/fedora
```

第二个容器启动一个交互式的 root shell，我们可以用来检查文件系统内容：

```
[root@c10420016687 /]# ls -al /data
total 20
drwxr-xr-t.  4 root root 4096 Oct 17 15:36 .
dr-xr-xr-x. 19 root root 4096 Oct 17 15:36 ..
drwxr-xr-x.  2  999  999 4096 Sep 20 22:20 configdb
drwxr-xr-x.  4  999  999 4096 Oct 17 15:36 db
```

正如预期的那样，我们可以在 Fedora 容器中找到挂载的 MongoDB 卷。如果我们停止甚至移除第一个 `mongodb01` 容器，卷依然会保持活跃并挂载在 Fedora 容器中。

到目前为止，我们已经看到了没有特定容器或挂载资源隔离的基本用例。如果主机启用了 SELinux 并且处于强制模式，必须应用一些额外的考虑。

## SELinux 对挂载的考虑

SELinux 会递归地为文件和目录应用标签以定义它们的上下文。这些标签通常存储为扩展文件系统属性。SELinux 使用上下文来管理策略，并定义哪些进程可以访问特定资源。

`ls` 命令用于查看资源的类型上下文：

```
$ ls -alZ /etc/passwd
-rw-r--r--. 1 root root system_u:object_r:passwd_file_t:s0 2965 Jul 28 21:00 /etc/passwd
```

在前面的示例中，`passwd_file_t` 标签定义了 `/etc/passwd` 文件的类型上下文。根据类型上下文，程序可以或不能在 SELinux 处于强制模式时访问该文件。

进程也有其类型上下文——容器运行时标签为 `container_t`，并对标记为 `container_file_t` 类型上下文的文件和目录具有读/写访问权限，对标记为 `container_share_t` 的资源具有读/执行权限。

默认情况下，其他主机目录是可访问的，其中 `/etc` 以只读方式访问，`/usr` 以读/执行方式访问。此外，`/var/lib/containers/overlay/` 下的资源被标记为 `container_share_t`。

如果我们尝试挂载一个没有正确标记的目录会发生什么？

Podman 仍然会执行容器，而不会因标签错误而报错，但挂载的目录或文件将无法从容器内运行的进程访问，而这些进程是以 `container_t` 类型标签运行的。以下示例尝试在不考虑标签约束的情况下，为 Nginx 容器挂载一个自定义的文档根目录：

```
$ mkdir ~/custom_docroot
$ echo "Hello World!" > ~/custom_docroot/index.html
$ podman run -d \
   --name custom_nginx \
  -p 8080:80 \
   -v ~/custom_docroot:/usr/share/nginx/html \
   docker.io/library/nginx
```

显然，一切顺利——容器正常启动，内部的进程也在运行，但如果我们尝试联系 Nginx 服务器，我们会看到以下错误：

```
$ curl localhost:8080
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.3</center>
</body>
</html>
```

`403 – Forbidden` 表明 Nginx 进程无法访问 `index.html` 页面。为了解决这个错误，我们有两个选择——将 SELinux 设置为 **宽松** 模式，或者重新标记挂载的资源。将 SELinux 设置为宽松模式时，它会继续跟踪违规情况，但不会阻止它们。无论如何，这不是一种好习惯，只有在我们无法正确排查访问问题，并需要将 SELinux 排除在外时，才应使用这种方式。以下命令将 SELinux 设置为宽松模式：

```
$ sudo setenforce 0
```

重要说明

宽松模式并不等于完全禁用 SELinux。在这种模式下，SELinux 仍然会记录 AVC 拒绝日志，但不会阻止操作。系统管理员可以在无需重启的情况下立即在宽松模式和强制模式之间切换。而禁用 SELinux 则意味着需要重启整个系统。

第二个优选方案是简单地重新标记我们需要挂载的资源。为了实现这个目标，我们可以使用 SELinux 的命令行工具。作为快捷方式，Podman 提供了一种更简单的方法——在卷挂载参数中应用 `:z` 和 `:Z` 后缀。这两个后缀之间的区别很微妙：

+   `:z` 后缀告诉 Podman 重新标记挂载的资源，以使所有容器都能够读写它。它适用于卷和绑定挂载。

+   `:Z`后缀告诉 Podman 重新标记挂载的资源，以便仅允许当前容器独占读取和写入。这对于卷和绑定挂载都适用。

为了测试差异，让我们再次尝试使用`:z`后缀运行容器，看看会发生什么：

```
$ podman run -d \
   --name custom_nginx \
  –p 8080:80 \
   -v ~/custom_docroot:/usr/share/nginx/html:z \
   docker.io/library/nginx
```

现在，HTTP 调用返回了预期的结果，因为进程能够访问`index.html`文件，并且没有被 SELinux 阻止：

```
$ curl localhost:8080
Hello World!
```

让我们看看自动应用于挂载目录的 SELinux 文件上下文：

```
$ ls -alZ ~/custom_docroot
total 20
drwxrwxr-x.  2 packt packt system_u:object_r:container_file_t:s0  4096 Oct 16 15:53 .
drwxrwxr-x. 74 packt packt unconfined_u:object_r:user_home_dir_t:s0 12288 Oct 16 16:32 ..
-rw-rw-r--.  1 packt packt system_u:object_r:container_file_t:s0  13 Oct 16 15:53 index.html
```

让我们关注`system_u:object_r:container_file_t:s0`标签。最终的`s0`字段表示一个`s0`敏感级别，它将能够以读写权限挂载资源。这也代表了一个安全问题，因为同一主机上的恶意容器可能会通过窃取或覆盖数据来攻击其他容器。

解决这个问题的方法称为**多类别安全**（**MCS**）。SELinux 使用 MCS 来配置额外的类别，这些类别是与其他 SELinux 标签一起应用于资源的明文标签。MCS 标签的对象仅能被分配相同类别的进程访问。

当容器启动时，容器内的进程会被标记为 MCS 类别，格式为**cXXX,cYYY**，其中 XXX 和 YYY 是随机选择的整数。

当传递`Z`（大写）时，Podman 会自动将 MCS 类别应用于挂载的资源。为了测试这一行为，让我们再次使用`:Z`后缀运行 Nginx 容器：

```
$ podman run -d \
   --name custom_nginx \
  –p 8080:80 \
   -v ~/custom_docroot:/usr/share/nginx/html:Z \
   docker.io/library/nginx
```

我们可以立即看到挂载的文件夹已经被重新标记为 MCS 类别：

```
$ ls -alZ ~/custom_docroot
total 20
drwxrwxr-x.  2 packt packt system_u:object_r:container_file_t:s0:c16,c898  4096 Oct 16 15:53 .
drwxrwxr-x. 74 packt packt unconfined_u:object_r:user_home_dir_t:s0       12288 Oct 16 21:12 ..
-rw-rw-r--.  1 packt packt system_u:object_r:container_file_t:s0:c16,c898    13 Oct 16 15:53 index.html
```

一个简单的测试将返回预期的`Hello World!`文本，证明容器内的进程被允许访问目标资源：

```
$ curl localhost:8080
Hello World!
```

如果我们使用相同的方法再次运行第二个容器，并对相同的绑定挂载应用`:Z`，会发生什么？

```
$ podman run -d \
   --name custom_nginx2 \
  –p 8081:80 \
   -v ~/custom_docroot:/usr/share/nginx/html:Z \
   docker.io/library/nginx
```

这次，我们在端口`8081`上运行 HTTP 测试，`HTTP GET`仍然能够正常工作：

```
$ curl localhost:8081
Hello World!
```

然而，如果我们再次测试映射到端口`8080`的容器，我们会得到一个意外的`403 Forbidden`消息：

```
$ curl localhost:8080
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.3</center>
</body>
</html>
```

不出所料，第二个容器使用了`:Z`后缀并重新标记了目录，赋予了新的 MCS 类别，使得第一个容器无法访问之前可用的内容。

重要提示

前面的示例是通过绑定挂载进行的，但同样适用于卷。使用这些技术时要小心，以避免不必要的重新标签化绑定挂载的系统或主目录。

在这一小节中，我们展示了 SELinux 在管理容器和资源隔离方面的强大功能。让我们以对其他类型存储的概述来结束这一章。

## 将其他类型的存储附加到容器

除了绑定挂载和卷之外，还可以将其他类型的存储附加到容器，具体来说，包括**tmpfs**、**image**和**devpts**。

### 附加 tmpfs 存储

有时，我们需要将存储附加到容器中，而这些存储不打算持久化（例如，缓存使用）。使用卷或绑定挂载会混乱主机本地磁盘（或如果使用不同的存储驱动则混乱任何其他后端）。在这些特定情况下，我们可以使用 **tmpfs** 卷。

tmpfs 是一个虚拟内存文件系统，这意味着它的所有内容都在主机的虚拟内存中创建。tmpfs 的一个优点是提供更快的 I/O，因为所有的读写操作大多数发生在 RAM 中。

要将 tmpfs 卷附加到容器中，我们可以使用 `--mount` 选项或 `--tmpfs` 选项。

`--mount` 标志的一个重要优点是，它在存储类型、来源、目标和额外挂载选项方面更加详细和具有表现力。以下示例运行一个附加了 tmpfs 卷的 `httpd` 容器：

```
$ podman run –d –p 8080:80 \ 
   --name tmpfs_example1 \
   --mount type=tmpfs,tmpfs-size=512M,destination=/tmp \
   docker.io/library/httpd
```

前面的命令创建了一个 512 MB 的 tmpfs 卷，并将其挂载到容器的 `/tmp` 文件夹中。我们可以通过在容器中运行 `mount` 命令来测试是否正确挂载：

```
$ podman exec -it tmpfs_example1 mount | grep '\/tmp'
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,relatime,context="system_u:object_r:container_file_t:s0:c375,c804",size=524288k,uid=1000,gid=1000,inode64
```

这表明 tmpfs 文件系统已在容器中正确挂载。停止容器时，tmpfs 会自动丢弃：

```
$ podman stop tmpfs_example1 
```

以下示例使用 `--tmpfs` 选项挂载一个 tmpfs 卷：

```
$ podman run –d –p 8080:80 \
   --name tmpfs_example2 \
   --tmpfs /tmp:rw,size= 524288k,mode=1777 \
   docker.io/library/httpd
```

这个示例与前一个示例的结果相同：一个运行中的容器，带有一个 512 MB 的 tmpfs 卷，挂载在 `/tmp` 目录上，具有读写模式和 `1777` 权限。

默认情况下，tmpfs 卷会以以下挂载选项挂载到容器中——**rw**、**noexec**、**nosuid** 和 **nodev**。

另一个有趣的特性是 SELinux 的自动 MCS 标签。它提供了文件系统的自动隔离，并防止任何其他容器访问内存中的数据。

### 附加镜像

OCI 镜像是提供启动容器的层和元数据的基础，但它们也可以在运行时附加到容器文件系统。这对于故障排除或附加外部镜像中的可用二进制文件非常有用。当 OCI 镜像被挂载到容器中时，会创建一个额外的叠加层。这意味着即使镜像以读写权限挂载，用户也不会修改原始镜像，而仅修改上层叠加层。

以下示例在 Alpine 容器中挂载一个具有读写权限的 `busybox` 镜像：

```
$ podman run -it \
   --mount type=image,src=docker.io/library/busybox,dst=/mnt,rw=true \
   alpine
```

重要提示

挂载的镜像必须已经缓存到主机中。Podman 只有在创建容器时镜像可用时才会拉取基础容器镜像，但它期望挂载的镜像已经存在。提前拉取这些镜像将解决此问题。

### 附加 devpts

这个选项对于将主机的 `/dev/` 附加到容器中非常有用，同时仍然可以创建一个终端。主机的 `/dev` 伪文件系统使容器能够直接访问机器的物理或虚拟设备。

要创建一个带有 `/dev` 文件系统并附加了 `devpts` 设备的容器，请运行以下命令：

```
$ sudo podman run -it \
  -v /dev/:/dev:rslave \
  --mount type=devpts,destination=/dev/pts \
  docker.io/library/fedora
```

要检查挂载选项的结果，我们需要在容器内使用一个额外的工具。为此，我们可以使用以下命令进行安装：

```
[root@034c8a61a4fc /]# dnf install -y toolbox
```

生成的容器在 `/dev/pts` 上挂载了一个额外的、非隔离的 `devpts` 设备：

```
# mount | grep '\/dev\/pts'
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,context="system_u:object_r:container_file_t:s0:c299,c741",gid=5,mode=620,ptmxmode=666)
```

上述输出是通过在容器内运行 `mount` 命令提取的。

# 总结

在本章中，我们完成了关于容器存储和 Podman 提供的操作功能的学习。本章的内容对于理解 Podman 如何管理临时数据和持久数据非常关键，并为用户提供了操作数据的最佳实践。

在第一部分，我们学习了容器存储的重要性，以及如何在单一主机和编排的多主机环境中正确管理它。

在第二部分，我们深入研究了容器存储特性和存储驱动，特别关注了 overlayfs。

在第三部分，我们学习了如何将文件复制到容器中或从容器中复制文件。我们还看到了如何将更改提交到新镜像。

第四部分描述了附加到容器的不同存储场景，涵盖了绑定挂载、卷、tmpfs、镜像和 devpts。本节也是讨论 SELinux 与存储管理交互的完美契机，展示了如何利用 SELinux 隔离同一主机上容器之间的存储资源。

在下一章，我们将学习一个对开发人员和运维团队都非常重要的话题，即如何使用 Podman 和 **Buildah**（一个高级专门的镜像构建工具）来构建 OCI 镜像。

# 进一步阅读

请参考以下资源以获取更多信息：

+   容器存储项目页面：[`github.com/containers/storage`](https://github.com/containers/storage)

+   容器标签：[`danwalsh.livejournal.com/81269.html`](https://danwalsh.livejournal.com/81269.html)

+   为什么你应该在 Linux 容器中使用多类别安全（Multi-Category Security）：[`www.redhat.com/en/blog/why-you-should-be-using-multi-category-security-your-linux-containers`](https://www.redhat.com/en/blog/why-you-should-be-using-multi-category-security-your-linux-containers)

+   Udica：生成 SELinux 策略：[`github.com/containers/udica`](https://github.com/containers/udica)

+   Overlay 源代码：[`github.com/containers/storage/blob/main/drivers/overlay/overlay.go`](https://github.com/containers/storage/blob/main/drivers/overlay/overlay.go)
