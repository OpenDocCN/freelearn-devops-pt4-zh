# *第十一章*：保护容器安全

安全性正成为当今最热的话题。全球各地的企业和公司都在大量投资于安全实践和工具，以帮助保护其系统免受内部或外部攻击。

正如我们在*第一章*《容器技术简介》中简要看到的，容器及其宿主系统可以看作是执行并保持目标应用程序运行的一种媒介。安全性应该应用于服务架构的所有层面，从基础设施到目标应用程序代码，在穿越虚拟化或容器化层时，都应当得到保障。

在本章中，我们将探讨一些最佳实践和工具，帮助提高容器化层的整体安全性。特别是，我们将讨论以下主要主题：

+   使用 Podman 运行无根容器

+   不要以 UID 0 运行容器

+   签名我们的容器镜像

+   自定义 Linux 内核能力

+   SELinux 与容器的交互

# 技术要求

要完成本章的示例，你需要一台已经安装好 Podman 的机器。正如我们在*第三章*《运行第一个容器》中提到的，本书中的所有示例都在 Fedora 34 或更高版本的系统上执行，但你也可以在你选择的操作系统上复现这些示例。

对*第四章*《管理正在运行的容器》，*第五章*《为容器的数据实现存储》，以及*第九章*《将镜像推送到容器注册表》所涵盖的主题有较好的理解，将帮助你更好地理解我们将在这里讨论的容器安全相关内容。

# 使用 Podman 运行无根容器

正如我们在*第四章*《管理正在运行的容器》中简要看到的，Podman 使得没有管理权限的标准用户也能在 Linux 主机上运行容器。这些容器通常被称为“无根容器”。

无根容器具有许多优势，包括以下几点：

+   它们为容器引擎、运行时或编排器被攻破的情况下，增加了一层额外的安全保护，能够阻止攻击者尝试获取主机的根权限。

+   它们允许多个没有特权的用户在同一主机上运行容器，从而充分利用高性能计算环境。

让我们思考任何 Linux 系统处理传统进程服务的方法。通常，软件包维护者会为调度和运行目标进程创建一个专用用户。如果我们尝试通过默认的软件包管理器在我们喜欢的 Linux 发行版上安装 Apache Web 服务器，那么我们会发现，安装的服务将通过一个名为“apache”的专用用户运行。

这种方法已经是多年的最佳实践，因为从安全角度来看，授权较少的权限可以提高安全性。

使用相同的方法，但采用无根容器，这样我们就可以在不需要额外权限提升的情况下运行容器进程。此外，Podman 是无守护进程的，因此它只会创建一个子进程。

在 Podman 中运行 rootless 容器非常简单，正如我们在前几章看到的，书中的许多示例都可以作为标准无特权用户运行。现在，让我们了解一下 rootless 容器执行背后的原理。

## Podman 瑞士军刀——subuid 和 subgid

现代 Linux 发行版使用的 `shadow-utils` 包版本依赖于两个文件：`/etc/subuid` 和 `/etc/subgid`。这些文件用于确定哪些 UIDs 和 GIDs 可以用于映射用户命名空间。

每个用户的默认分配是 65536 个 UID 和 65536 个 GID。

我们可以运行以下简单命令来检查 rootless 容器中 subuid 和 subgid 分配的工作原理：

```
$ id
uid=1000(alex) gid=1000(alex) groups=1000(alex),10(wheel) 
$ podman run alpine cat /proc/self/uid_map /proc/self/gid_map
Resolved "alpine" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob 59bf1c3509f3 done  
Copying config c059bfaa84 done  
Writing manifest to image destination
Storing signatures
         0       1000          1
         1     100000      65536
         0       1000          1
         1     100000      65536
```

如我们所见，两个文件都指示它们开始将 UID 和 GID 0 映射到我们刚刚使用来运行容器的当前用户的 UID/GID；即 `1000`。之后，它们将 UID 和 GID 1 从 `100000` 开始映射，并最终到达 `165536`。这是通过将起始点 `100000` 与默认范围 `65536` 相加计算得出的。

使用 rootless 容器并不是我们可以为容器环境实施的唯一最佳实践。在接下来的部分中，我们将了解为什么不应该使用 UID 0 运行容器。

# 不要使用 UID 0 运行容器

容器运行时可以被指示在容器内部执行与最初创建容器的用户 ID 不同的用户 ID 下的进程，类似于我们在 rootless 容器中看到的情况。以非 root 用户运行容器进程有助于安全性。例如，在容器中使用无特权用户可以限制容器内外的攻击面。

默认情况下，Dockerfile 和 Containerfile 可能会将默认用户设置为 root（即 UID=0）。为了避免这种情况，我们可以在这些构建文件中利用 `USER` 指令——例如，`USER 1001`——来指示 Buildah 或其他容器构建工具使用特定用户（UID 1001）来构建和运行容器镜像。

如果我们希望强制使用特定的 UID，需要调整我们计划与正在运行的容器一起使用的任何文件、文件夹或挂载点的权限。

现在，让我们学习如何调整现有镜像，使其可以以标准用户身份运行。

我们可以利用 DockerHub 上的一些预构建镜像，或者选择一个官方的 Nginx 容器镜像。首先，我们需要创建一个基本的 `nginx` 配置文件：

```
$ cat hello-podman.conf 
server {
    listen 80;

    location / {
        default_type text/plain;
        expires -1;
        return 200 'Hello Podman user!\nServer address: $server_addr:$server_port\n';
    }
}
```

`nginx` 配置文件非常简单：我们定义了监听端口（80）以及请求到达服务器时返回的内容消息。

然后，我们可以创建一个简单的 Dockerfile 来利用官方的 Nginx 容器镜像：

```
$ cat Dockerfile 
FROM docker.io/library/nginx:mainline-alpine
RUN rm /etc/nginx/conf.d/*
ADD hello-podman.conf /etc/nginx/conf.d/
```

Dockerfile 包含三个指令：

+   `FROM`：用于选择官方的 Nginx 镜像

+   `RUN`：用于清理配置目录中的任何默认配置示例

+   `ADD`：用于复制我们刚创建的配置文件

现在，让我们使用 Buildah 构建容器镜像：

```
$ buildah bud -t nginx-root:latest -f .
STEP 1/3: FROM docker.io/library/nginx:mainline-alpine
STEP 2/3: RUN rm /etc/nginx/conf.d/*
STEP 3/3: ADD hello-podman.conf /etc/nginx/conf.d/
COMMIT nginx-root:latest
Getting image source signatures
Copying blob 8d3ac3489996 done 
...
Copying config 21c5f7d8d7 done  
Writing manifest to image destination
Storing signatures
--> 21c5f7d8d70
Successfully tagged localhost/nginx-root:latest
21c5f7d8d709e7cfdf764a14fd6e95fb4611b2cde52b57aa46d43262a 6489f41
```

一旦构建了镜像，命名为 `nginx-root`。现在，我们准备运行我们的容器：

```
$ podman run --name myrootnginx -p 127.0.0.1::80 -d nginx-root 
364ec7f5979a5059ba841715484b7238db3313c78c5c577629364aa46b6d 9bdc
```

这里，我们使用了 `–p` 选项来发布端口并使其可以从主机访问。让我们找出在主机系统中随机选择的本地端口：

```
$ podman port myrootnginx 80
127.0.0.1:38029
```

最后，让我们启动我们的容器化 Web 服务器：

```
$ curl localhost:38029
Hello Podman user!
Server address: 10.0.2.100:80
```

容器终于启动了，但到底是哪个用户在使用我们的容器？让我们来看看：

```
$ podman ps | grep root
364ec7f5979a  localhost/nginx-root:latest  nginx -g daemon o...  55 minutes ago  Up 55 minutes ago  0.0.0.0:38029->80/tcp      myrootnginx
$ podman exec 364ec7f5979a id
uid=0(root) gid=0(root)
```

正如预期的那样，容器正在以 root 用户身份运行！

现在，让我们进行一些编辑以更改用户。首先，我们需要更改 Nginx 服务器配置中的监听端口：

```
$ cat hello-podman.conf 
server {
    listen 8080;

    location / {
        default_type text/plain;
        expires -1;
        return 200 'Hello Podman user!\nServer address: $server_addr:$server_port\n';
    }
}
```

在这里，我们将监听端口从 (`80`) 替换为 `8080`；我们不能在非特权用户下使用低于 `1024` 的端口。

接着，我们需要编辑我们的 Dockerfile：

```
$ cat Dockerfile 
FROM docker.io/library/nginx:mainline-alpine
RUN rm /etc/nginx/conf.d/*
ADD hello-podman.conf /etc/nginx/conf.d/

RUN chmod -R a+w /var/cache/nginx/ \
        && touch /var/run/nginx.pid \
        && chmod a+w /var/run/nginx.pid
EXPOSE 8080
USER nginx
```

如你所见，我们修复了 Nginx 服务器上主文件和文件夹的权限，暴露了新的 `8080` 端口，并将默认用户设置为 Nginx 用户。

现在，我们准备构建一个全新的容器镜像。我们将其命名为 `nginx-user`：

```
$ buildah bud -t nginx-user:latest -f .
STEP 1/6: FROM docker.io/library/nginx:mainline-alpine
STEP 2/6: RUN rm /etc/nginx/conf.d/*
STEP 3/6: ADD hello-podman.conf /etc/nginx/conf.d/
STEP 4/6: RUN chmod -R a+w /var/cache/nginx/         && touch /var/run/nginx.pid         && chmod a+w /var/run/nginx.pid 
STEP 5/6: EXPOSE 8080
STEP 6/6: USER nginx
COMMIT nginx-user:latest
Getting image source signatures
Copying blob 8d3ac3489996 done  
... 
Copying config 7628852470 done  
Writing manifest to image destination
Storing signatures
--> 76288524704
Successfully tagged localhost/nginx-user:latest
762885247041fd233c7b66029020c4da8e1e254288e1443b356cbee4d73 adf3e
```

现在，我们可以运行容器了：

```
$ podman run --name myusernginx -p 127.0.0.1::8080 -d nginx-user 
299e0fb727f339d87dd7ea67eac419905b10e36181dc1ca7e35dc7d0a 9316243
```

找到关联的随机主机端口并检查 Web 服务器是否正常工作：

```
$ podman port myusernginx 8080
127.0.0.1:42209
$ curl 127.0.0.1:42209
Hello Podman user!
Server address: 10.0.2.100:8080
```

最后，让我们查看是否改变了在容器中运行目标进程的用户：

```
$ podman ps | grep user
299e0fb727f3  localhost/nginx-user:latest  nginx -g daemon o...  38 minutes ago  Up 38 minutes ago  127.0.0.1:42209->8080/tcp  myusernginx
$ podman exec 299e0fb727f3 id
uid=101(nginx) gid=101(nginx) groups=101(nginx)
```

如你所见，我们的容器以非特权用户身份运行，这正是我们所期望的。

如果你想查看一个现成的示例，请访问本书的 GitHub 仓库：[`github.com/PacktPublishing/Podman-for-DevOps`](https://github.com/PacktPublishing/Podman-for-DevOps)。

不幸的是，安全不仅仅关乎权限和用户——我们还需要关注基础镜像及其来源，并检查容器镜像的签名。我们将在下一节中学习这个。

# 签名我们的容器镜像

当我们处理从外部注册表拉取的镜像时，我们会有一些安全顾虑，特别是与潜在的攻击手段相关的安全问题（见 *进一步阅读* 部分中的 [*1*]），尤其是伪装技术，它帮助攻击者操控镜像组件使其看起来是合法的。这也可能是由于 **中间人攻击**（**MITM**）在传输过程中被攻击者实施。

为了防止在管理容器时出现某些类型的攻击，最佳解决方案是使用分离的镜像签名来信任镜像提供者并确保其可靠性。

**GNU 隐私保护工具**（**GPG**）是 OpenPGP 标准的一个自由实现，可以与 Podman 一起使用，用于签名镜像并在拉取后检查它们的有效签名。

当拉取镜像时，Podman 可以验证签名的有效性，并拒绝没有有效签名的镜像。

现在，让我们学习如何实现一个基本的镜像签名工作流。

## 使用 GPG 和 Podman 签名镜像

在本节中，我们将创建一个基本的 GPG 密钥对，并配置 Podman 来推送和签署镜像，同时将签名存储在一个临时存储库中。为了清晰起见，我们将使用基本的 Docker Registry V2 容器镜像来运行一个注册中心，而不做任何自定义。

在测试镜像拉取和签名验证工作流之前，我们将公开一个基本的 web 服务器以发布分离的签名。

要使用 GPG 创建镜像签名，我们需要创建一个有效的 GPG 密钥对，或者使用现有的密钥对。因此，我们将简要回顾 GPG 密钥对，以帮助你理解镜像签名是如何工作的。

密钥对由私钥和公钥组成。公钥可以广泛共享，而私钥则保持私密，绝不与任何人共享。接收者的公钥可以被发送者用于签署文件或消息。这样，只有私钥的拥有者（即接收者）才能解密消息。

我们可以轻松地将这一概念转化为容器镜像：将镜像推送到远程注册中心的镜像所有者可以使用密钥对进行签名，并将分离的签名存储在一个对用户公开可访问的存储库（从现在起称为*sigstore*）中。在这里，签名与镜像本身是分开的——注册中心会存储镜像的二进制数据，而 sigstore 则保存并公开镜像的签名。

拉取镜像的用户将能够使用之前共享的公钥来验证镜像的签名。

现在，让我们回到创建 GPG 密钥对的过程。我们将使用以下命令创建一个简单的密钥对：

```
$ gpg --full-gen-key
```

上述命令会要求你回答一系列问题，并提供一个密码短语来帮助生成密钥对。默认情况下，密钥对将存储在 `$HOME/.gnupg` 文件夹中。

密钥对的输出应类似于以下内容：

```
$ gpg --list-keys
/home/vagrant/.gnupg/pubring.kbx
pub   rsa3072 2022-01-05 [SC]
      2EA4850C32D29DA22B7659FEC38D92C0F18764AC
uid           [ultimate] Foo Bar foobar@example.com
sub   rsa3072 2022-01-05 [E]
```

也可以导出生成的密钥对。以下命令将公钥导出到一个文件：

```
$ gpg --armor --export foobar@example.com > pubkey.pem
```

该命令在稍后定义镜像签名的验证时会很有用。

以下命令可用于导出私钥：

```
$ gpg --armor \
  --export-secret-keys foobar@example.com > privkey.pem
```

在两个示例中，都使用了 `--armor` 选项将密钥导出为 **隐私增强邮件**（**PEM**）格式。

一旦密钥对生成完成，我们可以创建一个基本的注册中心来托管我们的容器镜像。为此，我们将重新使用 *第九章* 中的基本示例，*推送镜像到容器注册中心*，并以 root 身份运行以下命令：

```
# mkdir /var/lib/registry
# podman run -d \
   --name local_registry \
   -p 5000:5000 \
   -v /var/lib/registry:/var/lib/registry:z \
   --restart=always registry:2
```

现在，我们有一个没有身份验证的本地注册中心，可以用来推送测试镜像。正如我们之前提到的，注册中心并不知道镜像的分离签名。

Podman 必须能够在 staging sigstore 上写入签名。`/etc/containers/registries.d/default.yaml` 文件中已经有默认配置，内容如下：

```
default-docker:
#  sigstore: file:///var/lib/containers/sigstore
  sigstore-staging: file:///var/lib/containers/sigstore
```

`sigstore-staging` 路径是 Podman 写入镜像签名的地方；它必须写入一个可写的文件夹。可以自定义此路径或保持默认配置不变。

如果我们想创建多个与用户相关的 sigstore，可以创建 `$HOME/.config/containers/registries.d/default.yaml` 文件，并在用户的主目录中定义一个自定义的 `sigstore-staging` 路径，遵循之前示例中展示的相同语法。这将允许用户以无根模式运行 Podman，并成功写入他们的 sigstore。

重要

不建议通过允许一般写权限来共享默认 `sigstore` 给所有用户。因为这样主机上的每个用户都能对现有签名进行写操作。

由于我们想使用默认的 sigstore，同时在用户的主目录下使用默认的 GPG 密钥对，我们将通过提升权限使用 `sudo` 运行 Podman，这是本书方法的一个例外。

以下示例展示了一个使用 UBI 8 构建的自定义 `httpd` 镜像的 Dockerfile：

Chapter11/image_signature/Dockerfile

```
FROM registry.access.redhat.com/ubi8
# Update image and install httpd
RUN yum install -y httpd && yum clean all –y
# Expose the default httpd port 80
EXPOSE 80
# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

要构建镜像，我们可以运行以下命令：

```
$ cd Chapter11/image_signature 
$ sudo podman build -t custom_httpd .
```

现在，我们可以用本地注册表名称标记镜像：

```
$ sudo podman tag custom_httpd localhost:5000/custom_httpd
```

最后，是时候将镜像推送到临时注册表，并使用生成的密钥对进行签名。`--sign-by` 选项允许用户传递由用户邮箱标识的有效密钥对：

```
$ sudo GNUPGHOME=$HOME/.gnupg podman \
   push --tls-verify=false \
   --sign-by foobar@example.com \
   localhost:5000/custom_httpd
Getting image source signatures
Copying blob 3ba8c926eef9 done  
Copying blob a59107c02e1f done  
Copying blob 352ba846236b done  
Copying config 569b015109 done  
Writing manifest to image destination
Signing manifest
Storing signatures
```

前面的代码成功地将镜像 blob 推送到注册表，并存储了镜像签名。注意 `GNUPGHOME` 变量，它在命令开始时被传递，用于定义 Podman 访问的 GPG 密钥存储路径。

警告

远程 Podman 客户端不支持 `--sign-by` 选项。

为了验证镜像是否已正确签名，并且其签名是否被保存在 sigstore 中，我们可以检查 `/var/lib/containers/sigstore` 的内容：

```
$ ls -al /var/lib/containers/sigstore/
drwxr-xr-x. 6 root    root    4096 Jan  5 18:58  .
drwxr-xr-x. 5 root    root    4096 Jan  5 13:29  ..
drwxr-xr-x. 2 root    root    4096 Jan  5 18:58 'custom_httpd@sha256=573c1eb93857c0169a606f1820271b143ac5073456f844255c3c7a9e 308bf639'
```

如你所见，新目录中包含了镜像签名文件：

```
$ ls -al /var/lib/containers/sigstore/'custom_httpd@sha256=573c1eb93857c0169a606f1820271b143ac5073456f844255c3c7a9 e308bf639'
total 12
drwxr-xr-x. 2 root root 4096 Jan  5 18:58 .
drwxr-xr-x. 6 root root 4096 Jan  5 18:58 ..
-rw-r--r--. 1 root root  730 Jan  5 18:58 signature-1
```

这样，我们就成功推送并签署了镜像，使其在未来使用中更加安全。接下来，让我们学习如何配置 Podman 来检索签名镜像。

## 配置 Podman 拉取签名镜像

为了成功拉取签名镜像，Podman 必须能够从 sigstore 获取签名，并且能够访问公钥以验证签名。

在这里，我们处理的是分离的签名，并且我们已经知道注册表不包含任何有关镜像签名的信息。因此，我们需要通过一个公开可访问的 sigstore 让用户可以访问它：一个 Web 服务器（如 Nginx、Apache httpd 等）将是一个不错的选择。

由于签名主机将与用于测试镜像拉取的主机相同，我们将运行一个 Apache httpd 服务器，将 sigstore 暂存文件夹暴露为服务器文档根目录。在实际场景中，我们会将签名移动到专用的 Web 服务器上。

在这个示例中，我们将使用标准的 `docker.io/library/httpd` 镜像，并以 root 权限运行容器，以便访问 sigstore 文件夹：

```
# podman run -d -p 8080:80 \
  --name sigstore_server \
  -v /var/lib/containers/sigstore:/usr/local/apache2/htdocs:z \
  docker.io/library/httpd
```

现在，Web 服务器已在 `http://localhost:8080` 上可用，Podman 可以使用它来获取镜像签名。

现在，让我们配置 Podman 以进行镜像拉取。首先，我们必须配置默认的镜像 sigstore。我们已经定义了 Podman 用来写入签名的暂存 sigstore，现在，我们需要定义用于读取镜像签名的 sigstore。

为此，我们必须再次编辑 `/etc/containers/registries.d/default.yaml` 文件，并添加对运行在 `http://localhost:8080` 的默认 sigstore Web 服务器的引用：

```
default-docker:
  sigstore: http://localhost:8080
  sigstore-staging: file:///var/lib/containers/sigstore
```

前面的代码配置了 Podman 用于所有镜像的 sigstore。然而，通过填写文件的*docker*字段，我们可以为特定的镜像仓库添加更多的 sigstore。以下代码配置了公共 Red Hat 仓库的 sigstore：

```
docker:
  registry.access.redhat.com:
    sigstore: https://access.redhat.com/webassets/docker/content/sigstore
```

在测试镜像拉取之前，我们必须实现 Podman 用于验证签名的公钥。这个公钥必须存储在拉取镜像的主机上，并且属于用于签名镜像的密钥对。

用于定义公钥路径的配置文件是 `/etc/containers/policy.json`。

以下代码显示了具有自定义配置的 `/etc/containers/policy.json` 文件，适用于注册表 `localhost:5000`：

```
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports": {
        "docker": {
            "localhost:5000": [
                {
                    "type": "signedBy",
                    "keyType": "GPGKeys",
                    "keyPath": "/tmp/pubkey.gpg"
                }
            ]
        },
        "docker-daemon": {
            "": [
                {
                    "type": "insecureAcceptAnything"
                }
            ]
        }
    }
}
```

要验证从 `localhost:5000` 拉取的镜像的签名，我们可以使用存储在 `keyPath` 字段定义的路径中的公钥。公钥必须存在于定义的路径中，并且可以被 Podman 读取。

如果我们需要从本节开头生成的示例密钥对中提取公钥，可以使用以下 GPG 命令：

```
$ gpg --armor --export foobar@example.com > /tmp/pubkey.gpg
```

现在，我们准备好测试镜像拉取并验证其签名了：

```
$ podman pull --tls-verify=false localhost:5000/custom_httpd
Getting image source signatures
Checking if image destination supports signatures
Copying blob 23fdb56daf15 skipped: already exists  
Copying blob d4f13fad8263 skipped: already exists  
Copying blob 96b0fdd0552f done  
Copying config 569b015109 done  
Writing manifest to image destination
Storing signatures
569b015109d457ae5fabb969fd0dc3cce10a3e6683ab60dc10505fc2d68 e769f
```

在使用提供的公钥进行签名验证后，镜像成功拉取到本地存储中。

现在，让我们看看当 Podman 无法正确验证签名时，它的表现如何。

## 测试签名验证失败

如果我们使 sigstore 无法访问，会怎么样？如果 Podman 无法验证签名，它还会成功拉取镜像吗？让我们尝试停止暴露 sigstore 的本地 httpd 服务器：

```
# podman stop sigstore_server
```

在再次拉取镜像之前，让我们删除之前缓存的镜像，以避免误报：

```
$ podman rmi localhost:5000/custom_httpd
```

现在，我们可以尝试再次拉取镜像：

```
$ podman pull --tls-verify=false localhost:5000/custom_httpd
Trying to pull localhost:5000/custom_httpd:latest...
WARN[0000] failed, retrying in 1s ... (1/3). Error: Source image rejected: Get "http://localhost:8080/custom_httpd@sha256=573c1eb93857c0169a606f1820271b143ac5073456f844255c3c7a9 e308bf639/signature-1": dial tcp [::1]:8080: connect: connection refused 
WARN[0001] failed, retrying in 1s ... (2/3). Error: Source image rejected: Get "http://localhost:8080/custom_httpd@sha256=573c1eb93857c0169a606f1820271b143ac5073456f844255c3c7a9 e308bf639/signature-1": dial tcp [::1]:8080: connect: connection refused 
WARN[0002] failed, retrying in 1s ... (3/3). Error: Source image rejected: Get "http://localhost:8080/custom_httpd@sha256=573c1eb93857c0169a606f1820271b143ac5073456f844255c3c7a9 e308bf639/signature-1": dial tcp [::1]:8080: connect: connection refused 
Error: Source image rejected: Get "http://localhost:8080/custom_httpd@sha256=573c1eb93857c0169a606f1820271b143ac5073456f 844255c3c7a9e308bf639/signature-1": dial tcp [::1]:8080: connect: connection refused
```

前面的错误表明 Podman 尝试连接暴露 sigstore 的 Web 服务器时失败了。此错误阻止了整个镜像拉取过程。

当我们用来验证签名的公共密钥无效或不属于用于签署镜像的密钥对时，会发生不同的错误。为了测试这一点，让我们用另一个密钥对中的公共密钥替换当前的公共密钥——在本示例中，我们使用的是从 `/etc/pki/rpm-gpg` 目录获取的公共 Fedora 34 RPM-GPG 密钥（也可以使用任何其他公共密钥）：

```
$ mv /tmp/pubkey.gpg /tmp/pubkey.gpg.bak 
$ cp /etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-34-x86_64 \
     /tmp/pubkey.gpg
```

之前停止的 httpd 服务器必须重新启动；我们要使签名可用，并专注于错误的公共密钥问题：

```
# podman start sigstore_server
```

现在，我们可以再次拉取镜像并检查生成的错误：

```
$ podman pull --tls-verify=false localhost:5000/custom_httpd
Trying to pull localhost:5000/custom_httpd:latest...
Error: Source image rejected: Invalid GPG signature: gpgme.Signature{Summary:128, Fingerprint:"2EA4850C32D29DA22B7659FEC38D92C0F18764AC", Status:gpgme.Error{err:0x9}, Timestamp:time.Time{wall:0x0, ext:63777026489, loc:(*time.Location)(0x560e17e5d680)}, ExpTimestamp:time.Time{wall:0x0, ext:62135596800, loc:(*time.Location)(0x560e17e5d680)}, WrongKeyUsage:false, PKATrust:0x0, ChainModel:false, Validity:0, ValidityReason:error(nil), PubkeyAlgo:1, HashAlgo:8}
```

在这里，Podman 产生了一个错误，这个错误是由于无效的 GPG 签名造成的，这是正确的，因为正在使用的公共密钥不属于正确的密钥对。

重要

在继续进行以下示例之前，请不要忘记恢复有效的公共密钥。

Podman 可以管理多个注册表和 sigstore，并且还提供专用命令来帮助您自定义安全策略，正如我们在下一小节中将看到的那样。

## 使用 Podman 镜像信任命令管理密钥

可以编辑 `/etc/containers/policy.json` 文件并修改其 JSON 对象，以添加或删除专用注册表的配置。然而，手动编辑容易出错，且不易自动化。

另外，我们可以使用 `podman image trust` 命令来转储或修改当前配置。

以下代码展示了如何使用 `podman image trust show` 命令打印当前的配置：

```
$ 
default         accept                                         
localhost:5000  signedBy                foobar@example.com  http://localhost:8080
                insecureAcceptAnything                         http://localhost:8080
```

也可以配置新的信任。例如，我们可以添加 Red Hat 公共 GPG 密钥，以检查 UBI 镜像的签名。

首先，我们需要下载 Red Hat 公共密钥：

```
$ sudo wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat \ 
  https://www.redhat.com/security/data/fd431d51.txt
```

注意

Red Hat 的产品签名密钥，包括本示例中使用的那个，可以在 [`access.redhat.com/security/team/key`](https://access.redhat.com/security/team/key) 找到。

下载密钥后，我们必须使用 `podman image trust set` 命令为从 *registry.access.redhat.com* 拉取的 UBI 8 镜像配置镜像信任：

```
$ sudo podman image trust set -f /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat registry.access.redhat.com/ubi8
```

运行上述命令后，`/etc/containers/policy.json` 文件将发生如下变化：

```
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports": {
        "docker": {
            "localhost:5000": [
                {
                    "type": "signedBy",
                    "keyType": "GPGKeys",
                    "keyPath": "/tmp/pubkey.gpg"
                }
            ],
            "registry.access.redhat.com/ubi8": [
                {
                    "type": "signedBy",
                    "keyType": "GPGKeys",
                    "keyPath": "/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat"
                }
            ]
        },
        "docker-daemon": {
            "": [
                {
                    "type": "insecureAcceptAnything"
                }
            ]
        }
    }
```

请注意，文件中已经添加了与 *registry.access.redhat.com/ubi8* 相关的条目，以及用于验证镜像签名的公共密钥。

为了完成配置，我们需要将 Red Hat sigstore 配置添加到 `/etc/containers/registries.d/default.yaml` 配置文件中：

```
docker:
  registry.access.redhat.com:
    sigstore: https://access.redhat.com/webassets/docker/content/sigstore
```

提示

可以在 `/etc/containers/registries.d` 文件夹中为不同的提供商创建自定义注册表配置文件。例如，前面的示例可以在专用的 `/etc/containers/registries.d/redhat.yaml` 文件中定义。这使得您可以轻松维护和版本化注册表 sigstore 配置。

从现在开始，每次从 *registry.access.redhat.com* 拉取 UBI8 镜像时，它的签名将从 Red Hat sigstore 中拉取，并使用提供的公共密钥进行验证。

到目前为止，我们已经查看了与 Podman 相关的密钥管理示例，但也可以使用 Skopeo 来管理签名验证。在下一个小节中，我们将查看一些基本示例。

## 使用 Skopeo 管理签名

当我们从有效的传输中拉取镜像时，可以使用 Skopeo 验证镜像签名。

以下示例使用 `skopeo copy` 命令将镜像从我们的注册表拉取到本地存储。此命令与使用 `podman pull` 命令具有相同效果，但允许对源和目标传输进行更多控制：

```
$ skopeo copy --src-tls-verify=false \
  docker://localhost:5000/custom_httpd \
  containers-storage:localhost:5000/custom_httpd
```

Skopeo 不需要进一步配置，因为先前修改过的配置文件已经定义了 sigstore 和公钥路径。

我们还可以使用 Skopeo 在将镜像复制到传输之前对其进行签名：

```
$ sudo GNUPGHOME=$HOME/.gnupg skopeo copy \
   --dest-tls-verify=false \
   --sign-by foobar@example.com \
   containers-storage:localhost:5000/custom_httpd \
   docker://localhost:5000/custom_httpd
```

再次强调，Podman 使用的配置文件对于 Skopeo 仍然有效，后者使用相同的 sigstore 来写入签名，并使用相同的 GPG 存储来检索生成签名所需的密钥。

在这一部分，我们了解了如何验证镜像签名并避免潜在的中间人攻击。在接下来的部分，我们将重点介绍如何通过自定义 Linux 内核能力来执行容器运行时。

# 自定义 Linux 内核能力

能力是 Linux 内核 2.2 中引入的特性，目的是将提升的特权拆分成单一单元，这些单元可以被任意分配给进程或线程。

我们可以通过为非特权进程分配一组特定的限制性能力，而不是以有效 UID 0 运行一个完全特权的进程。通过对进程执行的安全上下文进行更细粒度的控制，这种方法有助于缓解潜在的攻击手段。

在讨论容器的能力之前，我们先回顾一下它们在 Linux 系统中是如何工作的，这样我们就能理解它们的内部逻辑。

## 能力快速入门指南

能力是通过使用扩展属性（参见`man xattr`）与文件可执行文件关联的，并且会被通过`execve()`系统调用执行的进程自动继承。

可用能力的列表相当庞大，并且仍在增长；它包括线程可以执行的非常具体的操作。一些基本示例如下：

+   **CAP_CHOWN**：此能力允许线程修改文件的 UID 和 GID。

+   **CAP_KILL**：此能力允许你绕过权限检查，向进程发送信号。

+   `mknod()` 系统调用。

+   **CAP_NET_ADMIN**：此能力允许你对系统的网络配置执行各种特权操作，包括更改接口配置、启用/禁用接口的混杂模式、编辑路由表和启用/禁用多播。

+   **CAP_NET_RAW**：该功能允许线程使用 RAW 和 PACKET 套接字。程序如 ping 可以使用此功能发送 ICMP 数据包，而无需提升权限。

+   `chroot()` 系统调用和通过 `setns()` 系统调用更改挂载命名空间。

+   **CAP_DAC_OVERRIDE**：该功能允许绕过 **自主访问控制**（**DAC**）检查，从而允许文件的读取、写入和执行操作。

更多详细信息以及可用功能的详细列表，请参阅相关的手册页（`man capabilities`）。

要将某个功能分配给可执行文件，我们可以使用 `setcap` 命令，如以下示例所示，其中 `CAP_NET_ADMIN` 和 `CAP_NET_RAW` 被授权给 `/usr/bin/ping` 可执行文件：

```
$ sudo setcap 'cap_net_admin,cap_net_raw+p' /usr/bin/ping
```

前述命令中的 '*+p*' 标志表示这些功能已设置为 *Permitted*。

要检查文件的功能，我们可以使用 `getcap` 命令：

```
$ getcap /usr/bin/ping
/usr/bin/ping cap_net_admin,cap_net_raw=p
```

有关这些工具的更多详细信息，请参阅 `man getcap` 和 `man setcap`。

我们可以通过查看 `/proc/<PID>/status` 文件来检查运行中的进程的活动功能。在以下代码中，我们在设置 `CAP_NET_ADMIN` 和 `CAP_NET_RAW` 功能后启动 `ping` 命令。我们希望将进程置于后台并检查其当前功能：

```
$ ping example.com > /dev/null 2>&1 &
$ grep 'Cap.*' /proc/$(pgrep ping)/status
CapInh: 0000000000000000
CapPrm: 0000000000003000
CapEff: 0000000000000000
CapBnd: 000000ffffffffff
CapAmb: 0000000000000000
```

在这里，我们的兴趣在于评估 `CapPrm` 字段中的位图，该字段表示允许的功能。为了获得更易读的值，我们可以使用 `capsh` 命令解码位图的十六进制值：

```
$ capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```

结果与 `getcap` 命令在 `/usr/bin/ping` 文件中的输出相似，表明执行该命令将文件的允许功能传播到了其进程实例。

要查看用于设置位图的常量及其对应的功能列表，请参阅以下内核头文件：[`github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h`](https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h)。

提示

像 RHEL 和 CentOS 这样的发行版使用上述配置，允许 ping 命令发送 ICMP 数据包，并且所有用户都可以访问，而无需将其作为特权进程执行，且不使用 *setuid 0*。这种做法不安全，攻击者可以利用可执行文件中的漏洞或错误来提升权限并获得系统控制。

Fedora 在 31 版本中引入了一种新的、更安全的方法，该方法基于使用 `net.ipv4.ping_group_range` Linux 内核参数。通过设置涵盖所有系统组的广泛范围，该参数允许用户发送 ICMP 数据包，而无需启用 `CAP_NET_ADMIN` 和 `CAP_NET_RAW` 功能。

更多详细信息，请参阅 Fedora 项目的以下 Wiki 页面：[`fedoraproject.org/wiki/Changes/EnableSysctlPingGroupRange`](https://fedoraproject.org/wiki/Changes/EnableSysctlPingGroupRange)。

现在我们已经提供了关于 Linux 内核能力的高层次描述，让我们学习这些能力是如何应用到容器中的。

## 容器中的能力

能力可以应用于容器内，以便执行特定的操作。默认情况下，Podman 使用一组在`/usr/share/containers/containers.conf`文件中定义的 Linux 内核能力来运行容器。在撰写本文时，以下能力在该文件中已启用：

```
default_capabilities = [
    "CHOWN",
    "DAC_OVERRIDE",
    "FOWNER",
    "FSETID",
    "KILL",
    "NET_BIND_SERVICE",
    "SETFCAP",
    "SETGID",
    "SETPCAP",
    "SETUID",
    "SYS_CHROOT"
]
```

我们可以运行一个简单的测试来验证这些能力是否已有效应用于容器内运行的进程。对于这个测试，我们将使用官方的 Nginx 镜像：

```
$ podman run -d --name cap_test docker.io/library/nginx
$ podman exec -it cap_test sh -c 'grep Cap /proc/1/status'
CapInh: 00000000800405fb
CapPrm: 00000000800405fb
CapEff: 00000000800405fb
CapBnd: 00000000800405fb
CapAmb: 0000000000000000
```

在这里，我们提取了来自父进程 Nginx（在容器内以 PID 1 运行）的当前能力。现在，我们可以使用`capsh`工具检查位图：

```
$ capsh --decode=00000000800405fb
0x00000000800405fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_setfcap
```

上述能力列表与默认 Podman 配置中定义的列表相同。请注意，这些能力在无根模式和有根模式下都会应用。

注意

如果你感兴趣，容器化进程的能力是由容器运行时设置的，该运行时可以是`runc`或`crun`，具体取决于发行版。

现在我们知道了能力是如何在容器内配置和应用的，让我们学习如何定制容器的能力。

## 定制容器的能力

我们可以在运行时或静态地添加或删除能力。

要静态更改默认能力，我们只需编辑`/usr/share/containers/containers.conf`文件中的*default_capabilities*字段，并根据需要添加或删除它们。

要在运行时修改能力，我们可以使用`–cap-add`和`–cap-drop`选项，这两个选项都由`podman run`命令提供。

以下代码从容器中移除`CAP_DAC_OVERRIDE`能力：

```
$ podman run -d --name cap_test2 --cap-drop=DAC_OVERRIDE docker.io/library/nginx
```

如果我们再次查看能力位图，我们会看到它们已相应更新：

```
$ podman exec cap_test2 sh -c 'grep Cap /proc/1/status'
CapInh: 00000000800405f9
CapPrm: 00000000800405f9
CapEff: 00000000800405f9
CapBnd: 00000000800405f9
CapAmb: 0000000000000000
$ capsh --decode=00000000800405f9
0x00000000800405f9=cap_chown,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_setfcap
```

可以多次传递`--cap-add`和`--cap-drop`选项：

```
$ podman run -d --name cap_test3 \
   --cap-drop=KILL \
   --cap-drop=DAC_OVERRIDE \
   --cap-add=NET_RAW \
   --cap-add=NET_ADMIN \
   docker.io/library/nginx
```

在处理能力时，我们必须小心删除默认能力。以下代码展示了在删除`CAP_CHOWN`能力时，Nginx 容器中的错误：

```
$ podman run --name cap_test4 \
  --cap-drop=CHOWN \
  docker.io/library/nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/01/06 23:19:39 [emerg] 1#1: chown("/var/cache/nginx/client_temp", 101) failed (1: Operation not permitted)
nginx: [emerg] chown("/var/cache/nginx/client_temp", 101) failed (1: Operation not permitted)
```

这里容器失败了。从输出中，我们可以看到 Nginx 进程无法显示`/var/cache/nginx/client_temp`目录。这是因为`CAP_CHOWN`能力被移除所导致的直接后果。

并非所有能力都可以应用于无根容器。例如，如果我们尝试将`CAP_MKNOD`能力应用于无根容器，内核将不允许在无根容器内创建特殊文件：

```
$ podman run -it --cap-add=MKNOD \
  docker.io/library/busybox /bin/sh
/ # mkdir -p /test/dev
/ # mknod -m 666 /test/dev/urandom c 1 8
mknod: /test/dev/urandom: Operation not permitted
```

相反，如果我们以提升的 root 权限运行容器，则可以成功分配该能力：

```
# podman run -it --cap-add=MKNOD \
  docker.io/library/busybox /bin/sh
/ # mkdir -p /test/dev
/ # mknod -m 666 /test/dev/urandom c 1 8
/ # stat /test/dev/urandom
File: /test/dev/urandom
  Size: 0          Blocks: 0          IO Block: 4096   character special file
Device: 31h/49d Inode: 530019      Links: 1     Device type: 1,8
Access: (0666/crw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-01-06 23:50:06.056650747 +0000
Modify: 2022-01-06 23:50:06.056650747 +0000
Change: 2022-01-06 23:50:06.056650747 +0000
```

注意

通常，向容器添加能力意味着扩大潜在的攻击面，恶意攻击者可能利用这一点。如果不必要，保持默认能力并在分析潜在副作用后删除不需要的能力是一种良好的做法。

在这一部分，我们学习了如何管理容器中的能力。然而，能力并不是在保护容器时需要考虑的唯一安全方面。正如我们在下一部分将要学习的，SELinux 在确保容器隔离方面起着至关重要的作用。

# SELinux 与容器的交互

在这一部分，我们将讨论 SELinux 策略，并介绍 **Udica**，这是一个用于为容器生成 SELinux 配置文件的工具。

SELinux 直接在内核空间中工作，并遵循最小特权模型来管理对象隔离，该模型包含一系列 **策略**，用于强制执行或例外处理。为了定义这些对象，SELinux 使用定义 **类型** 的标签。默认情况下，SELinux 处于 **强制** 模式，拒绝对资源的访问，并根据策略定义一系列例外。要禁用强制模式，可以将 SELinux 设置为 **宽容** 模式，在该模式下，违规行为仅会被审计，而不会被阻止。

安全警报

正如我们之前提到的，切换 SELinux 到宽容模式或完全禁用它是 *不推荐的做法*，因为这会让你面临潜在的安全威胁。与其这样做，用户应该创建自定义策略来管理必要的例外。

默认情况下，SELinux 使用 **定向** 策略类型，试图通过一组预定义的策略来针对并限制特定的对象类型（进程、文件、设备等）。

SELinux 允许多种类型的访问控制。它们可以总结如下：

+   **类型强制**（**TE**）：这根据进程和文件类型控制对资源的访问。这是 SELinux 访问控制的主要应用场景。

+   **基于角色的访问控制**（**RBAC**）：这通过 SELinux 用户（可以映射到真实系统用户）及其相关的 SELinux 角色来控制对资源的访问。

+   **多级安全**（**MLS**）：这为所有具有相同敏感性级别的进程授予对资源的读/写访问权限。

+   **多类别安全**（**MCS**）：这通过 **类别** 来控制访问，类别是应用于资源的纯文本标签。类别用于创建对象的隔离区，同时与其他 SELinux 标签一起使用。只有属于同一类别的进程才能访问某一特定资源。在 *第五章*中，*为容器的数据实现存储*，我们讨论了 MCS 以及如何将类别映射到容器访问过的资源。

通过类型强制，系统文件被赋予称为**类型**的标签，而进程则被赋予称为**域**的标签。属于某个域的进程可以被允许访问属于特定类型的文件，这种访问可以由 SELinux 进行审核。

例如，根据 SELinux，标记有`httpd_t`域的 Apache `httpd`进程可以访问具有`httpd_sys_content_t`标签的文件或目录。

SELinux 类型策略基于以下模式：

```
POLICY DOMAIN TYPE:CLASS OPERATION;
```

在这里，`POLICY`是策略类型（例如，`allow`、`allowxperm`、`auditallow`、`neverallow`、`dontaudit`等），`DOMAIN`是进程域，`TYPE`是资源类型上下文，`CLASS`是对象类别（例如，`file`、`dir`、`lnk_file`、`chr_file`、`blk_file`、`sock_file`或`fifo_file`），而`OPERATION`是由策略处理的操作列表（例如，`open`、`read`、`use`、`lock`、`getattr`或`revc`）。

以下示例展示了一个基本的`allow`规则：

```
allow myapp_t myapp_log_t:file { read_file_perms append_file_perms };
```

在此示例中，运行在`myapp_t`域中的进程被允许访问`myapp_log_t`类型的文件，并执行`read_file_perms`和`append_file_perms`操作。

SELinux 以模块化的方式管理策略，允许您动态加载和卸载策略模块，无需每次重新编译整个策略集。策略可以使用`semodule`实用程序加载和卸载，如以下示例所示，展示了加载自定义策略的示例：

```
# semodule -i custompolicy.pp
```

`semodule`实用程序也可以用于查看所有加载的策略：

```
# semodule -l
```

在 Fedora、CentOS、RHEL 以及衍生分发版上，当前的二进制策略被安装在`/etc/selinux/targeted/policy`目录下，文件名为`polixy.XX`，其中`XX`表示策略版本。

在相同的分发版本上，容器策略定义在`container-selinux`软件包中，该软件包包含已编译的 SELinux 模块。如果您希望更详细地查看，软件包的源代码可在 GitHub 上获取：[`github.com/containers/container-selinux`](https://github.com/containers/container-selinux)。

通过查看存储库的内容，我们将找到开发任何模块所需的三个最重要的策略源文件：

+   `container.fc`：此文件定义了与模块中定义的类型绑定的文件和目录。

+   `container.te`：此文件定义了策略规则、属性和别名。

+   `container.if`：此文件定义了模块接口。它包含一组由模块公开的宏函数。

在容器内运行的进程被标记为`container_t`域。它具有对被标记为`container_file_t`类型上下文的资源的读写访问权限，并且对被标记为`container_share_t`类型上下文的资源具有读取和执行访问权限。

当容器执行时，`podman` 进程、容器运行时以及 `conmon` 进程将以 `container_runtime_t` 域类型运行，并且只允许执行那些仅能过渡到特定类型的进程。这些类型被分组在 `container_domain` 属性中，并且可以使用 `seinfo` 工具（在 Fedora 上与 `setools-console` 包一起安装）进行检查，如以下代码所示：

```
$ seinfo -a container_domain -x
Type Attributes: 1
   attribute container_domain;
container_engine_t
container_init_t
container_kvm_t
container_logreader_t
container_t
container_userns_t
spc_t
```

`container_domain` 属性在 `container-policy` 仓库中的 `container.te` 源文件里通过 **attribute** 关键字声明：

```
attribute container_domain;
attribute container_user_domain;
attribute container_net_domain;
```

上述属性通过 `typeattribute` 声明映射到 `container_t` 类型：

```
typeattribute container_t container_domain, container_net_domain, container_user_domain;
```

使用这种方法，SELinux 确保了容器之间以及容器与主机之间的进程隔离。通过这种方式，逃离容器的进程（可能是利用漏洞）无法访问主机或其他容器中的资源。

当容器被创建时，镜像的只读层（即形成 OverlayFS 的 LowerDirs 集合）会被标记为 `container_ro_file_t` 类型，这会防止容器在这些目录中进行写操作。同时，MergedDir（LowerDirs 和 UpperDir 的总和）是可写的，并被标记为 `container_file_t`。

为了证明这一点，让我们运行 `c1` 和 `c2` MCS 类别：

```
# podman run -d --name selinux_test1 --security-opt label=level:s0:c1,c2 nginx
```

现在，我们可以在主机文件系统中找到所有标记为 `container_file_t:s0:c1,c2` 的文件：

```
# find /var/lib/containers/storage/overlay -type f -context '*container_file_t:s0:c1,c2*' -printf '%-50Z%p\n'
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1 d7737f80056677b/merged/lib/x86_64-linux-gnu/libreadline.so.8.1
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1 d7737f80056677b/merged/lib/x86_64-linux-gnu/libhistory.so.8.1
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1 d7737f80056677b/merged/lib/x86_64-linux-gnu/libexpat.so.1.6.12
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1 d7737f80056677b/merged/lib/udev/rules.d/96-e2scrub.rules
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1 d7737f80056677b/merged/lib/terminfo/r/rxvt-unicode-256color
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1 d7737f80056677b/merged/lib/terminfo/r/rxvt-unicode
[…output omitted...]
```

如预期的那样，`container_file_t` 标签，关联了 `c1` 和 `c2` 类别，被应用于 MergedDir 容器下的所有文件。

同时，我们可以演示容器的 LowerDirs 被标记为 `container_ro_file_t`。首先，我们需要提取容器的 LowerDirs 列表：

```
# podman inspect selinux_test1 \
  --format '{{.GraphDriver.Data.LowerDir}}'
/var/lib/containers/storage/overlay/9566cbcf1773eac59951c14c52156a6164db1b0d8026d015 e193774029db18a5/diff:/var/lib/containers/storage/overlay/24de59cced7931bbcc0c4a34d4369c15119a0b8b180f98a0434 fa76a6dfcd490/diff:/var/lib/containers/storage/overlay/1bb84245b98b7e861c91ed4319972ed3287bdd2ef02a8657c696 a76621854f3b/diff:/var/lib/containers/storage/overlay/97f26271fef21bda129ac431b5f0faa03ae0b2b50bda6af 969315308fc16735b/diff:/var/lib/containers/storage/overlay/768ef71c8c91e4df0aa1caf96764ceec999d7eb0aa584 e241246815c1fa85435/diff:/var/lib/containers/storage/overlay/2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073 b41ef9727da6c851f/diff
```

最右侧的目录代表容器的最底层，通常是镜像的基础文件系统树。让我们检查这个目录的类型上下文：

```
# ls -alZ /var/lib/containers/storage/overlay/2edcec3590a4ec7f40 cf0743c15d78fb39d8326bc029073b41ef9727da6c851f/diff
total 84
dr-xr-xr-x. 21 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Jan  5 23:16 .
drwx------.  6 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Jan  5 23:16 ..
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 bin
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 11 17:25 boot
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 dev
drwxr-xr-x. 30 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 etc
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 11 17:25 home
drwxr-xr-x.  8 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 lib
[...omitted output...]
```

上述输出还展示了另一个有趣的方面：由于 LowerDir 层在使用相同镜像的多个容器之间共享，我们不会在这里找到任何已应用的 MCS 类别。

容器无法读写未标记为 `container_file_t` 的文件或目录。之前，我们看到通过对挂载卷应用 `:z` 后缀，或者在运行容器之前手动重新标记它们，都是可以重新标记这些文件的。

然而，重新标记像 `/home` 或 `/var/logs` 这样的关键目录是一个非常糟糕的主意，因为许多其他非容器化的进程将无法再访问它们。

唯一的解决方案是手动创建自定义策略来覆盖默认行为。然而，这对于日常使用和生产环境来说，管理起来过于复杂。

幸运的是，我们可以通过一个工具来解决这个限制，该工具为我们的容器生成自定义的 SELinux 安全配置文件：**Udica**。

## 引入 Udica

Udica 是一个开源项目（[`github.com/containers/udica`](https://github.com/containers/udica)），由 Lukas Vrabec 创建，他是 SELinux 的倡导者，并且是 Red Hat SELinux 与安全专项工程团队的负责人。

Udica 旨在克服前面描述的严格策略限制，通过为容器生成 SELinux 配置文件，使它们能够访问通常会被`container_t`域阻止的资源。

在 Fedora 上安装 Udica，只需运行以下命令：

```
$ sudo dnf install -y udica setools-console container-selinux
```

在其他发行版中，可以通过运行以下命令从源代码安装 Udica：

```
$ sudo dnf install -y setools-console git container-selinux
$ git clone 
$ cd udica && sudo python3 ./setup.py install
```

为了演示 Udica 的工作原理，我们将创建一个容器，它会写入主机的`/var/log`目录，该目录在容器创建时会被绑定挂载。默认情况下，具有`container_t`域的进程无法写入标记为`var_log_t`类型的目录。

以下脚本已在容器内执行，它是一个无限循环，写入由当前日期和计数器组成的日志行：

Chapter11/custom_logger/logger.sh

```
#!/bin/bash
set -euo pipefail
trap "echo Exited; exit;" SIGINT SIGTERM
# Run an endless loop writing a simple log entry with date
count=1
while true; do
echo "$(date +%y/%m/%d_%H:%M:%S) - Line #$count" | tee -a /var/log/custom.log
  count=$((count+1))
  sleep 2
done
```

上述脚本使用了`set -euo pipefail`选项，以便在发生错误时立即退出，并使用`tee`工具，将标准输出和`/var/log/custom.log`文件追加模式同时写入。`count`变量在每个循环周期递增。

这个容器的 Dockerfile 保持简洁——它只复制日志记录脚本并在容器启动时执行：

Chapter11/custom_logger/Dockerfile

```
FROM docker.io/library/fedora
# Copy the logger.sh script
COPY logger.sh /
# Exec the logger.sh script
CMD ["/logger.sh"]
```

重要提示

`logger.sh`脚本必须在构建前执行，以便在容器启动时正确启动。

容器镜像是以`custom_logger`为名称构建的：

```
# cd /Chapter11/custom_logger
# buildah build -t custom_logger .
```

现在，是时候测试容器并查看其行为了。`/var/log`目录以`rw`权限与容器的`/var/log`进行绑定挂载，而不改变其类型上下文。我们应该保持执行在前台，以便查看即时输出：

```
# podman run -v /var/log:/var/log:rw \
  --name custom_logger1 custom_logger
tee: /var/log/custom.log: Permission denied
22/01/08_09:09:33 - Custom log event #1
```

正如预期的那样，脚本未能写入目标文件。我们可以通过将目录类型上下文更改为`container_file_t`来修复此问题，但正如我们之前所学，这并不是一个好主意，因为它会阻止其他进程写入它们的日志。

另外，我们可以使用 Udica 为容器生成一个自定义的 SELinux 安全配置文件。在以下代码中，容器规格被导出到一个`container.json`文件，然后由 Udica 解析生成一个名为*custom_logger*的自定义配置文件：

```
# podman inspect custom_logger1 > container.json
# udica -j container.json custom_logger
Policy custom_logger created!
Please load these modules using: 
# semodule -i custom_logger.cil /usr/share/udica/templates/{base_container.cil,log_container.cil}
Restart the container with: "--security-opt label=type:custom_logger.process" parameter
```

一旦配置文件生成，Udica 会输出配置容器的指令。首先，我们需要使用`semodule`工具加载新的自定义策略。生成的文件位于`/usr/share/udica/templates/base_container.cil`和`/usr/share/udica/templates/log_container.cil`，其规则会继承到自定义容器策略文件中。

让我们使用推荐的命令加载模块：

```
# semodule -i custom_logger.cil /usr/share/udica/templates/{base_container.cil,log_container.cil}
```

在加载 SELinux 模块后，我们准备好运行带有自定义`custom_logger.process`标签的容器，将其作为参数传递给 Podman 的`--security-opt`选项。其他容器选项保持不变，除了名称已更新为`custom_logger2`，以便与先前的实例区分开来：

```
# podman run -v /var/log:/var/log:rw \
  --name custom_logger2 \
  --security-opt label=type:custom_logger.process \
  custom_logger
22/01/08_09:05:19 - Line #1
22/01/08_09:05:21 - Line #2
22/01/08_09:05:23 - Line #3
22/01/08_09:05:25 - Line #5
[...Omitted output...]
```

这次，脚本成功地写入了`/var/log/custom.log`文件，这得益于使用 Udica 生成的自定义配置文件。

请注意，容器进程不是在`container_t`域中运行，而是使用新的`custom_logger.process`超集，该超集包含在默认规则之上添加的额外规则。

我们可以通过在主机上运行以下命令来确认这一点：

```
# ps auxZ | grep 'custom_logger.process'
unconfined_u:system_r:container_runtime_t:s0-s0:c0.c1023 root 26546 0.1  0.6 1365088 53768 pts/0 Sl+ 09:16   0:00 podman run -v /var/log:/var/log:rw --security-opt label=type:custom_logger.process custom_logger system_u:system_r:custom_logger.process:s0:c159,c258 root 26633 0.0  0.0 4180 3136 ? Ss 09:16   0:00 /bin/bash /logger.sh
system_u:system_r:custom_logger.process:s0:c159,c258 root 26881 0.0  0.0 2640 1104 ? S 09:18   0:00 sleep 2 
```

Udica 通过解析 JSON 规范文件并查找容器挂载点、端口和能力来创建自定义策略。我们来看一下从示例中生成的`custom_logger.cil`文件的内容：

```
(block custom_logger
    (blockinherit container)
    (allow process process ( capability ( chown dac_override fowner fsetid kill net_bind_service setfcap setgid setpcap setuid sys_chroot ))) 
    (blockinherit log_rw_container)
```

CIL 语言语法超出了本书的范围，但我们仍然可以注意到一些有趣的内容：

+   *custom_logger*配置文件由`block`语句定义。

+   `allow`规则启用了容器的默认能力。

+   策略通过`blockinherit`语句继承了`container`和`log_rw_container`模块。

生成的 CIL 文件继承了在可用的 Udica 模板中已定义的模块，每个模块都专注于特定的操作。在 Fedora 中，这些模板通过`container-selinux`包安装，并可在`/usr/share/udica/templates/`文件夹中找到：

```
# ls -1 /usr/share/udica/templates/
base_container.cil
config_container.cil
home_container.cil
log_container.cil
net_container.cil
tmp_container.cil
tty_container.cil
virt_container.cil
x_container.cil
```

可用的模板为常见场景提供了实现，例如访问日志目录或用户主目录，甚至是打开网络端口。在这些模板中，`base_container.cil`模板始终由所有 Udica 生成的策略包含，作为生成自定义策略所用的基础构建块。

根据从规范文件派生的容器行为，其他模板被包含进来。例如，策略继承了来自`log_container.cil`模板的`log_rw_container`模块，以便让自定义日志容器访问`/var/log`目录。

Udica 是一个非常好的工具，能够解决容器隔离问题，帮助管理员通过克服手动编写规则的复杂性来解决 SELinux 限制的使用场景。

生成的安全配置文件也可以在 GitHub 仓库中进行版本控制，并在不同主机上的类似容器之间复用。

# 总结

在本章中，我们学习了如何开发和应用技术来提高基于容器的服务架构的整体安全性。我们了解了如何通过利用无根容器和避免 UID 0 来减少服务的攻击面。然后，我们学习了如何对容器镜像进行签名和信任，以避免中间人攻击。最后，我们深入探讨了容器工具的内部工作，查看了 Linux 内核的能力和 SELinux 子系统，这些可以帮助我们精细调整运行容器的各种安全方面。

经过深入的安全性分析后，我们已经准备好进入下一个章节，在那里我们将对容器的网络进行高级探讨。

# 进一步阅读

若想了解更多本章涉及的主题，请查看以下资源：

+   MITRE ATT&CK 容器矩阵: [`attack.mitre.org/matrices/enterprise/containers/`](https://attack.mitre.org/matrices/enterprise/containers/)

+   GNU 隐私保护工具: [`gnupg.org/`](https://gnupg.org/)

+   RFC4880 – OpenPGP 标准: [`www.rfc-editor.org/info/rfc4880`](https://www.rfc-editor.org/info/rfc4880)

+   Podman 镜像签名教程: [`github.com/containers/podman/blob/main/docs/tutorials/image_signing.md`](https://github.com/containers/podman/blob/main/docs/tutorials/image_signing.md)

+   Lukas Vrabec 的博客: [`lukas-vrabec.com/`](https://lukas-vrabec.com/)

+   CIL 介绍及设计原则: [`github.com/SELinuxProject/cl/wiki`](https://github.com/SELinuxProject/cl/wiki)

+   Red Hat 博客上的 Udica 介绍: [`www.redhat.com/en/blog/generate-selinux-policies-containers-with-udica`](https://www.redhat.com/en/blog/generate-selinux-policies-containers-with-udica)
