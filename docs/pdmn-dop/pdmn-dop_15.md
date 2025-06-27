# 第十二章：实现容器网络概念

容器网络隔离利用网络命名空间为每个容器提供独立的网络栈。如果没有容器运行时，跨多个命名空间管理网络接口会变得非常复杂。Podman 提供了灵活的网络管理，允许用户自定义容器如何与外部容器以及同一主机内部的其他容器进行通信。

在本章中，我们将学习管理容器网络的常见配置实践，以及无根容器和有根容器之间的区别。

在本章中，我们将涵盖以下主要话题：

+   容器网络与 Podman 配置

+   互联两个或更多容器

+   将容器暴露到我们底层主机之外

+   无根容器网络行为

# 技术要求

要完成本章内容，你需要一台已安装并能正常运行 Podman 的机器。正如我们在*第三章*《运行第一个容器》中提到的，书中的所有示例都可以在 Fedora 34 或更高版本的系统上执行，但也可以在你选择的**操作系统**（**OS**）上重现。本章中的示例将涉及 Podman v3.4.z 和 Podman v4.0.0，因为它们提供了不同的网络实现。

对于我们在*第四章*《管理运行中的容器》、*第五章*《实现容器数据存储》和*第九章*《推送镜像到容器注册表》中所覆盖的内容有良好的理解，将帮助你掌握我们将在本章中讨论的容器网络话题。

你还需要具备对基础网络概念的良好理解，以便理解路由、IP 协议、DNS 和防火墙等话题。

# 容器网络与 Podman 配置

在本节中，我们将讨论 Podman 的网络实现以及如何配置网络。Podman 4.0.0 对网络栈进行了重要更改。然而，Podman 3 在社区中仍然广泛使用。因此，我们将同时介绍这两种实现。

Podman 3 利用**容器网络接口**（**CNI**）来管理主机上创建的本地网络。CNI 提供了一组标准的规范和库，用于在容器环境中创建和配置基于插件的网络接口。

CNI 规范是为 Kubernetes 创建的，提供了一种网络配置格式，容器运行时使用该格式来设置定义的插件，并且插件二进制文件与运行时之间有一个执行协议。这种基于插件的方法的一个重要优点是，供应商和社区可以开发符合 CNI 规范的第三方插件。

Podman 4 的网络堆栈基于一个全新的项目 **Netavark**，这是一个完全用 Rust 编写的容器原生网络实现，旨在与 Podman 配合使用。Rust 是一种非常适合开发系统和网络组件的编程语言，因其高效的内存管理和高性能，类似于 C 语言。Netavark 提供了对双栈网络（IPv4/IPv6）和容器间 DNS 解析的更好支持，并与 Podman 项目的开发路线图更加紧密地结合。

重要提示

从 Podman 3 升级到 Podman 4 的用户将继续默认使用 CNI 并保留他们之前的配置。新的 Podman 4 安装将默认使用 Netavark。用户可以通过升级 `/usr/share/containers/containers.conf` 文件中的 `network_backend` 字段，恢复使用 CNI 网络后端。

在下一小节中，我们将重点介绍 Podman 3 用来编排容器网络的 CNI 配置。

## CNI 配置快速入门

一个典型的 CNI 配置文件定义了插件列表及其相关配置。以下示例展示了在 Fedora 上新安装的 Podman 的默认 CNI 配置：

Chapter12/podman_cni_conf.json

```
  "cniVersion": "0.4.0",
  "name": "podman",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "cni-podman0",
      "isGateway": true,
      "ipMasq": true,
      "hairpinMode": true,
      "ipam": {
        "type": "host-local",
        "routes": [{ "dst": "0.0.0.0/0" }],
        "ranges": [
          [
            {
              "subnet": "10.88.0.0/16",
              "gateway": "10.88.0.1"
            }
          ]
        ]
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    },
    {
      "type": "firewall"
    },
    {
      "type": "tuning"
    }
  ]
}
```

正如我们所见，这个文件中的 `plugins` 列表包含了一组由运行时用来编排容器网络的插件。

CNI 社区维护着一个参考插件库，这些插件可以被容器运行时使用。CNI 参考插件分为 **接口创建**、**IP 地址管理**（**IPAM**）和 **Meta** 插件。接口创建插件可以使用 IPAM 和 Meta 插件。

以下是一个非详尽的列表，描述了最常用的接口创建插件：

+   `bridge`：这个插件在主机上为网络创建一个专用的 Linux 桥接。容器接口连接到管理的桥接上，以便相互通信并与外部系统进行通信。这个插件目前被 Podman 和 `podman network` CLI 工具支持，是 Podman 安装时或创建新网络时配置的默认接口创建插件。

+   `ipvlan`：这个插件允许你将一个 IPVLAN 接口连接到容器。IPVLAN 解决方案是传统 Linux 桥接网络方案的一种替代方法，其中一个父接口在多个子接口之间共享，每个子接口都有一个 IP 地址。这个插件目前被 Podman 支持，但如果需要，你仍然可以手动创建和编辑 CNI 配置文件。

+   `macvlan`：这个插件允许配置 MACVLAN，它是一种类似于 IPVLAN 的方法，主要区别在于：在这种配置中，每个容器的子接口都会获取一个 MAC 地址。这个插件目前被 Podman 和 `podman network` CLI 工具支持。

+   `host-device`：此插件允许您直接将现有接口传递到容器中。目前 Podman 不支持此功能。

CNI IPAM 插件与容器内的 IP 地址管理相关联。只有三个参考 IPAM 插件：

+   `dhcp`：此插件允许您在主机上执行一个守护程序，管理运行容器的 `dhcp` 租约。这还意味着主机网络上已经运行了一个 `dhcp` 服务器。

+   `host-local`：此插件用于使用定义的地址范围为容器分配 IP 地址。分配的数据存储在主机文件系统中。它非常适合本地容器执行，并且是 Podman 在网络桥中使用的默认 IPAM 插件。

+   `static`：这是一个基本插件，管理分配给容器的静态地址列表。

NI Meta 插件用于配置主机中的特定行为，例如调整、防火墙规则和端口映射，并作为与创建接口插件一起链式执行的元插件。当前在参考插件仓库中维护的 Meta 插件如下：

+   `portmap`：此插件用于管理容器与主机之间的端口映射。它使用主机防火墙（`iptables`）应用配置，并负责创建 **Source NAT** (**SNAT**) 和 **Destination NAT** (**DNAT**) 规则。在 Podman 中，默认情况下启用此功能。

+   `firewall`：此插件配置防火墙规则以允许容器的入站和出站流量。在 Podman 中，默认情况下启用此功能。

+   `tuning`：此插件用于自定义系统调整（使用 `sysctl` 参数）和网络命名空间中的接口属性。在 Podman 中，默认情况下启用此功能。

+   `bandwidth`：此插件可用于使用 Linux 交通控制子系统在容器上配置流量速率限制。

+   `sbr`：此插件用于配置 `/usr/libexec/cni` 文件夹，并由 `containernetworking-plugins` 包提供，安装为 Podman 依赖项。

回到 CNI 配置示例，我们可以看到默认的 Podman 配置使用 `bridge` 插件和 `host-local` IP 地址管理，并且 `portmap`、`tuning` 和 `firewall` 插件与之链接在一起。

在为 Podman 创建的默认网络中，用于容器网络的子网是 `10.88.0.0/16`，桥接器称为 `cni-podman0`，作为容器的默认网关在 `10.88.0.1`，这意味着容器的所有出站流量都会被定向到桥接器的接口。

重要说明

此配置仅适用于完整根容器。在本章后面，我们将了解到 Podman 为克服用户权限有限而使用了不同的网络方法来处理无根容器。我们将看到这种方法在主机接口和 IP 地址管理方面存在许多限制。

现在，让我们看看当创建一个新的 rootfull 容器时，主机上会发生什么。

## Podman CNI 演练

在本小节中，我们将研究使用 CNI 作为网络后端时，创建新容器时发生的最特殊的网络事件。

重要提示

本小节中的所有示例均作为 `root` 用户执行。在查看网络接口和防火墙规则时，请确保清理掉现有的运行容器，以便能更清晰地看到相关配置。

我们将尝试使用 Nginx 容器，并将其默认的内部端口 `80/tcp` 映射到主机端口 `8080/tcp`。

在我们开始之前，我们需要验证当前主机的 IP 配置：

```
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group
default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:a9:ce:df brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.121.189/24 brd 192.168.121.255 scope global dynamic noprefixroute eth0
       valid_lft 3054sec preferred_lft 3054sec
    inet6 fe80::2fb:9732:a0d9:ac70/64 scope link noprefixroute        valid_lft forever preferred_lft forever
3: cni-podman0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether de:52:45:ae:1a:7f brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global cni-podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::dc52:45ff:feae:1a7f/64 scope link 
       valid_lft forever preferred_lft forever
```

除了主机的主要接口 `eth0`，我们还可以看到一个名为 `cni-podman0` 的桥接接口，地址为 `10.88.0.1/16`。另外，注意到该桥接的状态被设置为 `DOWN`。

重要

如果用于测试的主机是全新安装，且 Podman 从未执行过，则不会列出 `cni-podman0` 桥接接口。这并不是问题——它会在首次创建 rootfull 容器时被创建。

如果主机上没有其他容器在运行，我们应该看不到任何接口附加到虚拟桥接上。为了验证这一点，我们将使用 `bridge link show` 命令，期望其输出为空：

```
# bridge link show cni-podman0
```

查看防火墙规则时，我们不应在 `filter` 和 `nat` 表中看到与容器相关的规则：

```
# iptables -L
# iptables -L -t nat 
```

重要提示

前面命令的输出为了简洁起见已被省略，但值得注意的是，`filter` 表应已包含两个与 CNI 相关的链，分别是 `CNI-ADMIN` 和 `CNI-FORWARD`。

最后，我们想检查 `cni-podman0` 接口的路由规则：

```
# ip route show dev cni-podman0 
10.88.0.0/16 proto kernel scope link src 10.88.0.1 linkdown
```

该命令表示，所有发送到 `10.88.0.0/16` 网络的流量都会经过 `cni-podman0` 接口。

让我们运行 Nginx 容器，看看网络接口、路由和防火墙配置会发生什么：

```
# podman run -d -p 8080:80 \
  --name net_example docker.io/library/nginx
```

第一个也是最有趣的事件是一个新的网络接口的创建，正如 `ip addr show` 命令的输出所示：

```
# ip addr show
[...omitted output...]
3: cni-podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether de:52:45:ae:1a:7f brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global cni-podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::dc52:45ff:feae:1a7f/64 scope link 
       valid_lft forever preferred_lft forever
5: vethcf8b2132@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni-podman0 state UP group default 
    link/ether b6:4c:1d:06:39:5a brd ff:ff:ff:ff:ff:ff link-netns cni-df380fb0-b8a6-4f39-0d19-99a0535c2f2d
    inet6 fe80::90e3:98ff:fe6a:acff/64 scope link 
       valid_lft forever preferred_lft forever
```

这个新接口是 `man 4 veth`（虚拟以太网设备的一部分），它们充当本地隧道。Veth 对是 Linux 内核的虚拟接口，不依赖于容器运行时，可以应用于超出容器执行的使用场景。

veth 对的有趣之处在于它们可以跨多个网络命名空间生成，并且发送到对端的一个包会立即被接收。

`vethcf8b2132@if2` 接口与一个位于名为 `cni-df380fb0-b8a6-4f39-0d19-99a0535c2f2d` 网络命名空间中的设备相连。由于 Linux 提供了使用 `ip netns` 命令检查网络命名空间的选项，我们可以检查该命名空间是否存在并检查其网络堆栈：

```
# ip netns
cni-df380fb0-b8a6-4f39-0d19-99a0535c2f2d (id: 0)
```

提示

当创建新的网络命名空间时，会在`/var/run/netns/`下创建一个同名的文件。这个文件也有与`/proc/<PID>/ns/net`下符号链接指向的相同 inode 编号。当打开该文件时，返回的文件描述符将允许访问该命名空间。

上述命令确认了网络命名空间存在。现在，我们想检查在其中定义的网络接口：

```
# ip netns exec cni-df380fb0-b8a6-4f39-0d19-99a0535c2f2d ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether fa:c9:6e:5c:db:ad brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.88.0.3/16 brd 10.88.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f8c9:6eff:fe5c:dbad/64 scope link 
       valid_lft forever preferred_lft forever 
```

这里，我们执行了一个`ip addr show`命令，该命令嵌套在`ip netns exec`命令中。输出显示了我们 veth 对端的接口。这也给了我们一些有价值的信息：容器的 IPv4 地址，设置为`10.88.0.3`。

提示

如果你感兴趣，当使用 Podman 的默认网络并且使用`host-local` IPAM 插件时，容器的 IP 配置会被保存在`/var/lib/cni/networks/podman`文件夹中。在这里，会创建一个以分配的 IP 地址命名的文件，并写入容器生成的 ID。

如果创建了新的网络并且容器使用了该网络，它的配置将保存在`/var/lib/cni/networks/<NETWORK_NAME>`文件夹中。

我们还可以检查容器的路由表：

```
# ip netns exec cni-df380fb0-b8a6-4f39-0d19-99a0535c2f2d ip route
default via 10.88.0.1 dev eth0 
10.88.0.0/16 dev eth0 proto kernel scope link src 10.88.0.3
```

所有指向外部网络的出站流量都将通过`10.88.0.1`地址，它已分配给`cni-podman0`桥接器。

当创建新的容器时，`firewall`和`portmapper` CNI 插件会在主机的过滤器和 NAT 表中应用必要的规则。在以下代码中，我们可以看到已应用到容器 IP 地址的`nat`表规则，其中应用了 SNAT、DNAT 和伪装规则：

```
# iptables -L -t nat -n | grep -B4 10.88.0.3
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
CNI-HOSTPORT-MASQ  all  --  0.0.0.0/0            0.0.0.0/0            /* CNI portfwd requiring masquerade */
CNI-fb51a7bfa5365a8a89e764fd  all  --  10.88.0.3            0.0.0.0/0            /* name: "podman" id: "a5054cca3436a7bc4dbf78fe4b901ceef0569ced24181d2e7b118232123a5f e3" */
--
Chain CNI-DN-fb51a7bfa5365a8a89e76 (1 references)
target     prot opt source               destination         
CNI-HOSTPORT-SETMARK  tcp  --  10.88.0.0/16         0.0.0.0/0            tcp dpt:8080
CNI-HOSTPORT-SETMARK  tcp  --  127.0.0.1            0.0.0.0/0            tcp dpt:8080
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:10.88.0.3:80
```

更粗的那行显示了一个名为`CNI-DN-fb51a7bfa5365a8a89e76`的自定义链中的 DNAT 规则。这个规则表示，所有目标是主机上的`8080/tcp`端口的 TCP 数据包应该被重定向到`10.88.0.3:80`端口，这是容器暴露的网络套接字。这个规则与我们在容器创建时传递的`–p 8080:80`选项相匹配。

那么，容器是如何与外界通信的呢？让我们再次检查`cni-podman0`桥接器，看看是否有显著的变化：

```
# bridge link show cni-podman0
5: vethcf8b2132@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master cni-podman0 state forwarding priority 32 cost 2
```

上述接口连接到虚拟桥接器，该桥接器也恰好分配了一个 IP 地址（`10.88.0.1`），作为所有容器的默认网关。

让我们尝试追踪一个来自容器到著名主机`1.1.1.1`（Cloudflare 公共 DNS）的 ICMP 包的路径。为此，我们必须使用`ip netns exec`命令从容器网络命名空间中运行`traceroute`工具：

```
# ip netns exec cni-df380fb0-b8a6-4f39-0d19-99a0535c2f2d traceroute -I 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
1  _gateway (10.88.0.1)  0.071 ms  0.025 ms  0.003 ms
2  192.168.121.1 (192.168.121.1)  0.206 ms  0.195 ms  0.189 ms
3  192.168.1.1 (192.168.1.1)  5.326 ms  5.323 ms  5.319 ms
4  192.168.50.6 (192.168.50.6)  17.598 ms  17.595 ms  17.825 ms
5  192.168.50.5 (192.168.50.5)  17.821 ms  17.888 ms  17.882 ms
6  10.177.21.173 (10.177.21.173)  17.998 ms  17.772 ms  24.777 ms
7  185.210.48.42 (185.210.48.42)  25.963 ms  7.604 ms  7.702 ms
8  185.210.48.43 (185.210.48.43)  7.906 ms  10.344 ms  10.984 ms
9  185.210.48.77 (185.210.48.77)  12.212 ms  12.030 ms  12.983 ms
10  1.1.1.1 (1.1.1.1)  12.524 ms  12.160 ms  12.649 ms
```

重要提示

默认情况下，主机上可能已安装 traceroute 程序。要在 Fedora 上安装它，请运行`sudo dnf install traceroute`命令。

上述输出显示了一系列`10.88.0.1`，接着是主机的网络堆栈。

第二跳是主机的默认网关（`192.168.121.1`），它被分配给虚拟化主机中的虚拟桥接，并连接到我们实验室的主机虚拟机。

第三跳是分配给物理路由器的私有网络默认网关（`192.168.1.1`），该路由器连接到实验室的虚拟化主机网络。

这展示了所有流量都通过`cni-podman0`桥接接口。

我们可以创建多个网络，无论是使用 Podman 原生命令，还是通过我们喜欢的编辑器直接管理 JSON 文件。

现在我们已经探索了 CNI 的实现和配置细节，让我们来看看 Podman 4 中的新 Netavark 实现。

## Netavark 配置快速入门

Podman 4.0.0 版本引入了 Netavark 作为默认的网络后端。Netavark 的优势如下：

+   支持双栈 IPv4/IPv6

+   支持使用**aardvark-dns**伴随项目进行 DNS 原生解析

+   支持 rootless 容器

+   支持不同的防火墙实现，包括 iptables、firewalld 和 nftables

Netavark 使用的配置文件与为 CNI 所示的文件没有太大区别。Netavark 仍然使用 JSON 格式来配置网络；文件存储在`/etc/containers/networks`路径下，适用于 rootfull 容器；对于 rootless 容器，存储路径为`~/.local/share/containers/storage/networks`。

以下配置文件展示了一个在 Netavark 下创建和管理的示例网络：

```
[
     {
          "name": "netavark-example",
          "id": "d98700453f78ea2fdfe4a1f77eae9e121f3cbf4b6160dab89edf9ce23c b924d7",
          "driver": "bridge",
          "network_interface": "podman1",
          "created": "2022-02-17T21:37:59.873639361Z",
          "subnets": [
               {
                    "subnet": "10.89.4.0/24",
                    "gateway": "10.89.4.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": true,
          "ipam_options": {
               "driver": "host-local"
          }
     }
]
```

第一个显著的元素是配置文件的体积比 CNI 配置文件更紧凑。以下字段被定义：

+   `name`：网络的名称。

+   `id`：唯一的网络 ID。

+   `driver`：指定正在使用的网络驱动程序类型。默认值是`bridge`。Netavark 还支持 MACVLAN 驱动程序。

+   `network_interface`：这是与网络相关联的网络接口的名称。如果配置的驱动程序是`bridge`，那么这将是 Linux 桥接的名称。在前面的示例中，创建了一个名为`podman1`的桥接。

+   `created`：网络创建的时间戳。

+   `subnets`：提供子网和网关对象的列表。子网是自动分配的。不过，当你使用 Podman 创建新网络时，用户可以提供自定义 CIDR。Netavark 允许你在网络上管理多个子网和网关。

+   `ipv6_enabled`：可以使用此布尔值启用或禁用 Netavark 的 IPv6 原生支持。

+   `internal`：这个布尔值用于配置仅供内部使用的网络，并阻止外部路由。

+   `dns_enabled`：这个布尔值启用网络的 DNS 解析，由`aardvark-dns`守护进程提供服务。

+   `ipam_options`：此对象定义了一系列`ipam`参数。在前面的示例中，唯一的选项是 IPAM 驱动程序的类型，`host-local`，它的行为类似于 CNI 的 host-local 插件。

默认的 Podman 4 网络，名为 `podman`，实现了桥接驱动程序（桥接的名称是 `podman0`）。在这里，DNS 支持被禁用，类似于默认 CNI 配置的情况。

Netavark 也是一个可执行二进制文件，默认安装在 `/usr/libexec/podman/netavark` 路径下。它具有简单的 `setup` 和 `teardown` 命令，将网络配置应用到给定的网络命名空间（参见 `man netavark`）。

现在，让我们看看使用 Netavark 创建一个新容器的效果。

## Podman Netavark 使用指南

和 CNI 一样，Netavark 管理容器网络命名空间和主机网络命名空间中的网络配置创建，包括创建 veth 对和配置文件中定义的 Linux 桥接。

在默认的 Podman 网络中创建第一个容器之前，不会创建任何桥接，主机接口是唯一可用的，另外还有环回接口：

```
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:9a:ea:f4 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.121.15/24 brd 192.168.121.255 scope global dynamic noprefixroute eth0
       valid_lft 3293sec preferred_lft 3293sec
    inet6 fe80::d0fb:c0d1:159e:2d54/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

让我们运行一个新的 Nginx 容器，看看会发生什么：

```
# podman run -d -p 8080:80 \
  --name nginx-netavark 
  docker.io/library/nginx
```

当容器启动时，`podman0` 桥接和一个 veth 接口会出现：

```
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:9a:ea:f4 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.121.15/24 brd 192.168.121.255 scope global dynamic noprefixroute eth0
       valid_lft 3140sec preferred_lft 3140sec
    inet6 fe80::d0fb:c0d1:159e:2d54/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: veth2772d0ea@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman0 state UP group default qlen 1000
    link/ether fa:a3:31:63:21:60 brd ff:ff:ff:ff:ff:ff link-netns netns-61a5f9f9-9dff-7488-3922-165cdc6cd320
    inet6 fe80::f8a3:31ff:fe63:2160/64 scope link 
       valid_lft forever preferred_lft forever
8: podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ea:b4:9d:dd:2c:d1 brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::24ec:30ff:fe1a:2ca8/64 scope link 
       valid_lft forever preferred_lft forever
```

在网络命名空间、版本管理之间的上下文混合、防火墙规则或路由方面，最终用户没有与之前提供的 CNI 使用指南相比发生特别的变化。

再次为 `nginx-netavark` 容器创建了一个主机上的网络命名空间。让我们检查网络命名空间的内容：

```
# ip netns exec netns-61a5f9f9-9dff-7488-3922-165cdc6cd320 ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000    link/ether ae:9b:7f:07:3f:16 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.88.0.4/16 brd 10.88.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::ac9b:7fff:fe07:3f16/64 scope link 
       valid_lft forever preferred_lft forever
```

再次强调，可以找到分配给容器的内部 IP 地址。

如果容器以 rootless 模式运行，桥接和 veth 对将会在一个 rootless 网络命名空间中创建。

重要提示

在 Podman 4 中，可以使用 `podman unshare --rootless-netns` 命令检查 rootless 网络命名空间。

使用 Podman 3 和 CNI 的用户可以使用 `--rootless-cni` 选项来获得相同的结果。

在下一个子节中，我们将学习如何使用 Podman 提供的 CLI 工具来管理和定制容器网络。

## 使用 Podman 管理网络

`podman network` 命令提供了管理容器网络所需的工具。以下是可用的子命令：

+   `create`：创建一个新的网络

+   `connect`：连接到指定的网络

+   `disconnect`：断开与网络的连接

+   `exists`：检查网络是否存在

+   `inspect`：转储网络的 CNI 配置

+   `prune`：删除未使用的网络

+   `reload`：重新加载容器防火墙规则

+   `rm`：删除指定的网络

在本节中，你将学习如何创建一个新的网络并将容器连接到它。对于 Podman 3，所有生成的 CNI 配置文件都会写入主机上的 `/etc/cni/net.d` 文件夹。

对于 Podman 4，所有生成的 rootfull 网络的 Netavark 配置文件都会写入 `/etc/containers/networks`，而 rootless 网络的配置文件则写入 `~/.local/share/containers/storage/networks`。

以下命令创建一个名为 `example1` 的新网络：

```
# podman network create \
  --driver bridge \
  --gateway "10.89.0.1" \
  --subnet "10.89.0.0/16" example1
```

在这里，我们提供了子网和网关信息，以及与 CNI 接口创建插件对应的驱动类型。生成的网络配置根据网络后端类型写入上述路径，并可通过`podman network inspect`命令进行检查。

以下输出显示了一个 CNI 网络后端的配置：

```
# podman network inspect example1
[
    {
        "cniVersion": "0.4.0",
        "name": "example1",
        "plugins": [
            {
                "bridge": "cni-podman1",
                "hairpinMode": true,
                "ipMasq": true,
                "ipam": {
                    "ranges": [
                        [
                            {
                                "gateway": "10.89.0.1",
                                "subnet": "10.89.0.0/16"
                            }
                        ]
                    ],
                    "routes": [
                        {
                            "dst": "0.0.0.0/0"
                        }
                    ],
                    "type": "host-local"
                },
                "isGateway": true,
                "type": "bridge"
            },
            {
                "capabilities": {
                    "portMappings": true
                },
                "type": "portmap"
            },
            {
                "backend": "",
                "type": "firewall"
            },
            {
                "type": "tuning"
            },
            {
                "capabilities": {
                    "aliases": true
                },
                "domainName": "dns.podman",
                "type": "dnsname"
            }
        ]
    }
]
```

新的网络 CNI 配置显示会为此网络创建一个名为`cni-podman1`的桥接，并且容器将从`10.89.0.0/16`子网中分配 IP 地址。

配置的其他字段与默认配置非常相似，除了`dnsname`插件（项目的仓库：[`github.com/containers/dnsname`](https://github.com/containers/dnsname)），它用于启用容器内的名称解析。这个功能在容器间通信中提供了优势，我们将在下一小节中介绍。

以下输出显示了为 Netavark 网络后端生成的配置：

```
# podman network inspect example1
[
     {
          "name": "example1",
          "id": "a8ca04a41ef303e3247097b86d9048750e5f1aa819ec573b0e5f78e3cc8a 971b",
          "driver": "bridge",
          "network_interface": "podman1",
          "created": "2022-02-18T17:56:28.451701452Z",
          "subnets": [
               {
                    "subnet": "10.89.0.0/16",
                    "gateway": "10.89.0.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": true,
          "ipam_options": {
               "driver": "host-local"
          }
     }
]
```

请注意，Netavark 的桥接命名约定略有不同，因为它使用`podmanN`模式，*N >= 0*。

要列出所有现有的网络，我们可以使用`podman network ls`命令：

```
# podman network ls
NETWORK ID   NAME     VERSION  PLUGINS
2f259bab93aa podman   0.4.0    bridge,portmap,firewall,tuning
228b48a56dbc example1 0.4.0    bridge,portmap,firewall,tuning,dnsname
```

上面的输出显示了每个活动网络的名称、ID、CNI 版本和活动插件。

在 Podman 4 中，输出稍微简洁一些，因为没有 CNI 插件要显示：

```
# podman network ls
NETWORK ID    NAME        DRIVER
a8ca04a41ef3  example1    bridge
2f259bab93aa  podman      bridge
```

现在，是时候启动一个连接到新网络的容器了。以下代码创建一个附加到`example1`网络的 PostgreSQL 数据库：

```
# podman run -d -p 5432:5432 \
  --network example1 \
  -e POSTGRES_PASSWORD=password \
  --name postgres \
  docker.io/library/postgres
533792e9522fc65371fa6d694526400a3a01f29e6de9b2024e84895f354e d2bb
```

新的容器从`10.89.0.0/16`子网接收了一个地址，如`podman inspect`命令所示：

```
# podman inspect postgres --format '{{.NetworkSettings.Networks.example1.IPAddress}}'
10.89.0.3
```

当我们使用 CNI 网络后端时，可以通过查看新的`/var/lib/cni/networks/example1`文件夹内容来再次检查这些信息：

```
# ls -al /var/lib/cni/networks/example1/
total 20
drwxr-xr-x. 2 root root 4096 Jan 23 17:26 .
drwxr-xr-x. 5 root root 4096 Jan 23 16:22 ..
-rw-r--r--. 1 root root   70 Jan 23 16:26 10.89.0.3
-rw-r--r--. 1 root root    9 Jan 23 16:57 last_reserved_ip.0
-rwxr-x---. 1 root root    0 Jan 23 16:22 lock
```

查看`10.89.0.3`文件的内容时，我们发现了以下内容：

```
# cat /var/lib/cni/networks/example1/10.89.0.3
533792e9522fc65371fa6d694526400a3a01f29e6de9b2024e84895f354 ed2bb
```

该文件保存了我们`postgres`容器的 ID，用于跟踪与分配的 IP 地址之间的映射。如前所述，这一行为由`host-local`插件管理，这是 Podman 网络的默认 IPAM 选择。

重要提示

Netavark 网络后端跟踪`/run/containers/networks/ipam.db`文件中的 IPAM 配置，适用于 rootfull 容器。

我们还可以看到，已创建了一个新的 Linux 桥接（注意使用`cni-`前缀，这用于 CNI 网络后端）：

```
# ip addr show cni-podman1
8: cni-podman1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 56:ed:1d:a9:53:54 brd ff:ff:ff:ff:ff:ff
    inet 10.89.0.1/16 brd 10.89.255.255 scope global cni-podman1
       valid_lft forever preferred_lft forever
    inet6 fe80::54ed:1dff:fea9:5354/64 scope link 
       valid_lft forever preferred_lft forever
```

新设备连接到了 PostgreSQL 容器的一个 veth 对端：

```
# bridge link show
10: vethf03ed735@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master cni-podman1 state forwarding priority 32 cost 2 
20: veth23ee4990@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master cni-podman0 state forwarding priority 32 cost 2
```

在这里，我们可以看到`vethf03ed735@eth0`连接到了`cni-podman1`桥接。该接口有如下配置：

```
# ip addr show vethf03ed735
10: vethf03ed735@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni-podman1 state UP group default 
    link/ether 86:d1:8c:c9:8c:2b brd ff:ff:ff:ff:ff:ff link-netns cni-77bfb1c0-af07-1170-4cc8-eb56d15511ac
    inet6 fe80::f889:17ff:fe83:4da2/64 scope link 
       valid_lft forever preferred_lft forever
```

上面的输出还显示了 veth 对端的另一端位于容器的网络命名空间中——即`cni-77bfb1c0-af07-1170-4cc8-eb56d15511ac`。我们可以检查容器的网络配置并确认从新子网分配的 IP 地址：

```
# ip netns exec cni-77bfb1c0-af07-1170-4cc8-eb56d15511ac ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ba:91:9e:77:30:a1 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.89.0.3/16 brd 10.89.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b891:9eff:fe77:30a1/64 scope link 
       valid_lft forever preferred_lft forever
```

重要提示

在 Podman 4 中，Netavark 后端的网络命名空间命名模式是 `netns-<UID>`。

可以在不停止和重启容器的情况下，将运行中的容器连接到另一个网络。通过这种方式，容器将保留一个连接到原始网络的接口，同时会创建一个连接到新网络的第二个接口。这个功能在反向代理等使用场景中非常有用，可以通过 `podman network connect` 命令实现。让我们尝试运行一个新的 `net_example` 容器：

```
# podman run -d -p 8080:80 --name net_example docker.io/library/nginx 
# podman network connect example1 net_example
```

为了验证容器是否已经连接到新网络，我们可以运行 `podman inspect` 命令并查看网络信息：

```
# podman inspect net_example
[...omitted output...]
            "Networks": {
                "example1": {
                    "EndpointID": "",
                    "Gateway": "10.89.0.1",
                    "IPAddress": "10.89.0.10",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "fa:41:66:0a:25:45",
                    "NetworkID": "example1",
                    "DriverOpts": null,
                    "IPAMConfig": null,
                    "Links": null
                },
                "podman": {
                    "EndpointID": "",
                    "Gateway": "10.88.0.1",
                    "IPAddress": "10.88.0.7",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "ba:cd:eb:8d:19:b5",
                    "NetworkID": "podman",
                    "DriverOpts": null,
                    "IPAMConfig": null,
                    "Links": null
                }
            }
[…omitted output...]
```

在这里，我们可以看到容器现在已经有两个接口，分别连接到 `podman` 和 `example1` 网络，并且从每个网络的子网中分配了 IP 地址。

要将容器从网络中断开，我们可以使用 `podman network disconnect` 命令：

```
# podman network disconnect example1 net_example
```

当网络不再需要且已与运行中的容器断开连接时，我们可以使用 `podman network rm` 命令删除它：

```
# podman network rm example1
example1
```

命令的输出显示了被移除的网络列表。在这里，网络的 CNI 配置从主机的 `/etc/cni/net.d` 目录中移除。

重要提示

如果网络上有关联的容器，无论这些容器是正在运行还是已停止，前一个命令会因 `Error: "example1" has associated containers with it` 而失败。为了绕过这个问题，请在使用命令之前移除或断开这些关联容器。

`podman network rm` 命令在我们需要删除特定网络时非常有用。要删除所有未使用的网络，`podman network prune` 命令是一个更好的选择：

```
# podman network prune
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
example2
db_network
```

在本节中，我们学习了 CNI 规范以及 Podman 如何利用其接口简化容器网络设置。在多层或微服务场景中，我们需要让容器之间能够通信。在下一节中，我们将学习如何管理容器之间的通信。

# 连接两个或更多容器

运用我们在上一节学到的知识，我们应该知道，两个或更多在同一网络中创建的容器，可以在同一子网上互相通信，而无需外部路由。

同时，属于不同网络的两个或更多容器将能够通过它们各自的网络路由数据包，互相到达不同的子网。

为了演示这一点，让我们在相同的默认网络中创建几个 `busybox` 容器：

```
# podman run -d --name endpoint1 \
  --cap-add=net_admin,net_raw busybox /bin/sleep 10000
# podman run -d --name endpoint2 \
  --cap-add=net_admin,net_raw busybox /bin/sleep 10000
```

在我们的实验环境中，两个容器的地址分别是 `10.88.0.14` (`endpoint1`) 和 `10.88.0.15` (`endpoint2`)。这两个地址可能会变化，可以使用前面介绍的 `podman inspect` 或 `nsenter` 命令收集这些地址。

关于能力的定制，我们添加了 `CAP_NET_ADMIN` 和 `CAP_NET_RAW` 能力，以便容器能够无缝运行 `ping` 或 `traceroute` 等命令。

让我们尝试从 `endpoint1` 到 `endpoint2` 运行 `traceroute` 命令，以查看数据包的路径：

```
# podman exec -it endpoint1 traceroute 10.88.0.14
traceroute to 10.88.0.14 (10.88.0.14), 30 hops max, 46 byte packets
1  10.88.0.14 (10.88.0.14)  0.013 ms  0.004 ms  0.002 ms
```

正如我们所见，数据包停留在内部网络并直接到达节点，没有额外的跳数。

现在，让我们创建一个新的网络 `net1`，并将一个名为 `endpoint3` 的容器连接到该网络：

```
# podman network create --driver bridge --gateway "10.90.0.1" --subnet "10.90.0.0/16" net1
# podman run -d --name endpoint3 --network=net1 --cap-add=net_admin,net_raw busybox /bin/sleep 10000
```

我们实验中的容器获得了一个 IP 地址 `10.90.0.2`。让我们看看从 `endpoint1` 到 `endpoint3` 的网络路径：

```
# podman exec -it endpoint1 traceroute 10.90.0.2
traceroute to 10.90.0.2 (10.90.0.2), 30 hops max, 46 byte packets
1  host.containers.internal (10.88.0.1)  0.003 ms  0.001 ms  0.006 ms
2  10.90.0.2 (10.90.0.2)  0.001 ms  0.002 ms  0.002 ms
```

这一次，数据包穿越了 `endpoint1` 容器的默认网关（`10.88.0.1`）并到达了 `endpoint3` 容器，该数据包从主机路由到关联的 `net1` Linux 桥接。

在同一主机上的容器之间的连接非常容易管理和理解。然而，我们仍然缺少一个重要方面，即容器间通信：DNS 解析。

让我们学习如何通过 Podman 网络来利用这一功能。

## 容器 DNS 解析

尽管 DNS 解析有许多配置细节，但它是一个非常简单的概念：一个服务会被查询以提供与给定主机名关联的 IP 地址。虽然 DNS 服务器可以提供的信息远比这丰富，但在这个示例中，我们将重点关注简单的 IP 解析。

例如，假设有一个场景，其中一个名为 `webapp` 的容器上的 Web 应用程序需要对另一个名为 `db` 的容器上的数据库进行读写访问。DNS 解析使得 `webapp` 能在联系 `db` 之前查询其 IP 地址。

之前，我们学习过 Podman 的默认网络不提供 DNS 解析，而新创建的用户网络默认启用 DNS 解析。在 CNI 网络后端，`dnsname` 插件会自动配置一个 `dnsmasq` 服务，当容器连接到网络时，该服务会启动并提供 DNS 解析。在 Netavark 网络后端，DNS 解析由 `aarvark-dns` 提供。

为了测试此功能，我们将重新使用在 *第十章* 中展示的 **students** Web 应用程序，*容器故障排除和监控*，因为它提供了一个合适的客户端-服务器示例，包含一个最小的 REST 服务和基于 PostgreSQL 的数据库后端。

信息

源代码可以在本书的 GitHub 仓库中找到，网址是 [`github.com/PacktPublishing/Podman-for-DevOps/tree/main/Chapter10/students`](https://github.com/PacktPublishing/Podman-for-DevOps/tree/main/Chapter10/students)。

在这个示例中，Web 应用程序通过 HTTP GET 打开一个查询 PostgreSQL 数据库的请求，结果会以 JSON 格式输出。为了演示，我们将在同一个网络上运行数据库和 Web 应用程序。

首先，我们必须创建 PostgreSQL 数据库 pod，同时提供一个通用的用户名和密码：

```
# podman run -d \
   --network net1 --name db \
   -e POSTGRES_USER=admin \
   -e POSTGRES_PASSWORD=password \
   -e POSTGRES_DB=students \
   postgres
```

接下来，我们必须将 `students` 文件夹中的 SQL 转储数据恢复到数据库中：

```
# cd Chapter10/students
# cat db.sql | podman exec -i db psql -U admin students
```

如果你之前没有在章节中构建过它，你需要构建`students`容器镜像并在主机上运行它：

```
# buildah build -t students .
# podman run -d \
   --network net1 \
   -p 8080:8080 \
   --name webapp \
   students \
   students -host db -port 5432 \
   -username admin -password
```

请注意命令中的高亮部分：`students`应用程序接受`-host`、`-port`、`-username`和`-password`选项，用于自定义数据库的端点和凭证。

我们没有在主机字段中提供任何 IP 地址。相反，使用了 Postgres 容器名称`db`和默认的`5432`端口来标识数据库。

同时注意，`db`容器创建时没有进行任何端口映射：我们预计直接通过`net1`容器网络访问数据库，这两个容器都在该网络中创建。

让我们尝试调用`students`应用程序的 API，看看会发生什么：

```
# curl localhost:8080/students {"Id":10149,"FirstName":"Frank","MiddleName":"Vincent","LastName":"Zappa","Class":"3A","Course":"Composition"}
```

查询成功，这意味着应用程序成功地查询了数据库。那么，究竟是如何发生的呢？它是如何仅凭容器名称就解析出容器的 IP 地址的？在下一节中，我们将探讨在 CNI 和 Netavark 网络后端上的不同表现。

### 在 CNI 网络后端上的 DNS 解析

在带有 CNI 后端的 Podman 3 或 Podman 4 中，`dnsname`插件在`net1`网络中启用，并且会启动一个专门的`dnsmasq`服务，负责将容器名称解析为其分配的 IP 地址。我们先从查找容器的 IP 地址开始：

```
# podman inspect db --format '{{.NetworkSettings.Networks.net1.IPAddress}}'
10.90.0.2
# podman inspect webapp --format '{{.NetworkSettings.Networks.net1.IPAddress}}'
10.90.0.3
```

我们要查找系统中运行的`dnsmasq`进程：

```
# ps aux | grep dnsmasq
root        2703  0.0  0.0  26436  2384 ?        S    16:16   0:00 /usr/sbin/dnsmasq -u root --conf-file=/run/containers/cni/dnsname/net1/dnsmasq.conf
root        5577  0.0  0.0   6140   832 pts/0    S+   22:00   0:00 grep --color=auto dnsmasq
```

上面的输出显示了一个`dnsmasq`进程的实例，它的配置文件位于`/run/containers/cni/dnsname/net1/`目录下。我们来检查一下它的内容：

```
# ls -al /run/containers/cni/dnsname/net1/
total 12
drwx------. 2 root root 120 Jan 25 16:16 .
drwx------. 3 root root  60 Jan 25 16:16 ..
-rw-r--r--. 1 root root  30 Jan 25 16:28 addnhosts
-rwx------. 1 root root 356 Jan 25 16:16 dnsmasq.conf
-rwxr-x---. 1 root root   0 Jan 25 16:16 lock
-rw-r--r--. 1 root root   5 Jan 25 16:16 pidfile
```

`/run/containers/cni/dnsname/net1/dnsmasq.conf`定义了`dnsmasq`的配置：

```
# cat /run/containers/cni/dnsname/net1/dnsmasq.conf 
## WARNING: THIS IS AN AUTOGENERATED FILE
## AND SHOULD NOT BE EDITED MANUALLY AS IT
## LIKELY TO AUTOMATICALLY BE REPLACED.
strict-order
local=/dns.podman/
domain=dns.podman
expand-hosts
pid-file=/run/containers/cni/dnsname/net1/pidfile
except-interface=lo
bind-dynamic
no-hosts
interface=cni-podman1
addn-hosts=/run/containers/cni/dnsname/net1/addnhosts
```

该进程在`cni-podman1`接口（`net1`网络桥接，IP 地址为`10.90.0.1`）上监听，并且是`dns.podman`域的权威服务器。主机的记录保存在`/run/containers/cni/dnsname/net1/addnhosts`文件中，文件内容如下：

```
# cat /run/containers/cni/dnsname/net1/addnhosts 
10.90.0.2  db
10.90.0.3  webapp
```

当`net1`网络中的容器尝试 DNS 解析时，它会使用`/etc/resolv.conf`文件来查找应该将查询指向哪个 DNS 服务器。`webapp`容器中的该文件内容如下：

```
# podman exec -it webapp cat /etc/resolv.conf
search dns.podman
nameserver 10.90.0.1
```

这表明容器与`10.90.0.1`地址（这也是容器的默认网关和`cni-podman1`桥接网络）进行了联系，以查询主机名解析。

搜索域允许进程查找`db.dns.podman`，并通过 DNS 服务正确解析。CNI 网络配置的搜索域可以通过编辑`/etc/cni/net.d/`下的相关配置文件来定制。`net1`配置中`dnsname`插件的默认配置如下：

```
{
         "type": "dnsname",
         "domainName": "dns.podman",
         "capabilities": {
            "aliases": true
         }
      }
```

当你更新 `domainName` 字段为新值时，变化不会立即生效。要重新生成更新后的 `dnsmasq.conf`，必须停止网络中的所有容器，以便 `dnsname` 插件清理当前的网络配置。当容器重新启动时，`dnsmasq` 配置将相应地重新生成。

### 在 Netavark 网络后端上进行 DNS 解析

如果前面的示例在使用 Netavark 网络后端的 Podman 4 上执行，`aardvark-dns` 守护进程将负责类似于 `dnsmasq` 的容器解析。

`aardvark-dns` 项目是 Netavark 的一个伴生项目，使用 Rust 编写。它是一个轻量级的权威 DNS 服务，可以同时支持 IPv4 A 记录和 IPv6 AAAA 记录。

当创建一个启用 DNS 解析的新网络时，系统将创建一个新的 `aardvark-dns` 进程，如以下代码所示：

```
# ps aux | grep aardvark-dns
root        9115  0.0  0.0 344732  2584 pts/0    Sl   20:15   0:00 /usr/libexec/podman/aardvark-dns --config /run/containers/networks/aardvark-dns -p 53 run
root       10831  0.0  0.0   6400  2044 pts/0    S+   23:36   0:00 grep --color=auto aardvark-dns
```

该进程在主机网络命名空间的 `53/udp` 端口上监听根容器的请求，在 rootless 网络命名空间的 `53/udp` 端口上监听无根容器的请求。

`ps` 命令的输出还显示了默认配置路径——`/run/containers/networks/aardvark-dns` 目录——这是 `aardvark-dns` 进程存储解析配置的地方，配置文件按相关网络命名。例如，对于 `net1` 网络，我们将找到类似以下内容：

```
# cat /run/containers/networks/aardvark-dns/net1
10.90.0.1
dc7fff2ef78e99a2a1a3ea6e29bfb961fc07cd6cf71200d50761e25df30 11636 10.90.0.2  db,dc7fff2ef78e
10c7bbb7006c9b253f9ebe1103234a9af41dced8f12a6d94b7fc46a9a97 5d8cc 10.90.0.2  webapp,10c7bbb7006c
```

该文件存储了每个容器的 IPv4 地址（如果有 IPv6 地址，也会存储）。在这里，我们可以看到容器的名称和短 ID 被解析为 IPv4 地址。

第一行告诉我们 `aardvark-dns` 正在监听传入请求的地址。它再次对应于网络的默认网关地址。

在同一网络中连接容器可以实现不同服务之间的快速和简单通信，尤其是在不同的网络命名空间中运行的服务。然而，也有一些使用案例要求容器共享相同的网络命名空间。Podman 提供了一种解决方案，轻松实现这一目标：Pod。

## 在 Pod 内部运行容器

Pod 的概念来源于 Kubernetes 架构。根据官方上游文档，“*一个 Pod ... 是一个包含一个或多个容器的组，共享存储和网络资源，并有一个运行容器的规范*。”

Pod 也是 Kubernetes 调度中最小的可部署单元。Pod 中的所有容器共享相同的网络、UTC、IPC 和（可选的）PID 命名空间。这意味着，在不同容器上运行的所有服务可以彼此通过 **localhost** 进行引用，而外部容器则继续联系 Pod 的 IP 地址。一个 Pod 接收一个 IP 地址，这个地址在所有容器之间共享。

有许多使用案例，其中一个非常常见的是边车容器：在这种情况下，一个反向代理或 OAuth 代理与主容器一起运行，以提供身份验证或服务网格功能。

Podman 提供了用于操作 Pod 的基本工具，通过 `podman pod` 命令。以下示例展示了如何创建一个包含两个容器的基本 Pod，并演示了 Pod 内容器之间共享网络命名空间的情况。

重要提示

为了理解以下示例，请停止并移除所有正在运行的容器和 Pod，然后从一个干净的环境开始。

`podman pod create` 从头开始初始化一个新的空 Pod：

```
# podman pod create --name example_pod
```

重要提示

当创建一个新的空 Pod 时，Podman 还会创建一个 `infra` 容器，用于在启动 Pod 时初始化命名空间。此容器基于 Podman 3 的 `k8s.gcr.io/pause` 镜像，对于 Podman 4 则基于本地构建的 `podman-pause` 镜像。

现在，我们可以在 Pod 中创建两个基本的 `busybox` 容器：

```
# podman create --name c1 --pod example_pod busybox sh -c 'sleep 10000'
# podman create --name c2 --pod example_pod busybox sh -c 'sleep 10000'
```

最后，我们可以使用 `podman pod start` 命令启动 Pod（及其关联的容器）：

```
# podman pod start example_pod
```

这里，我们有一个正在运行的 Pod，其中包含两个容器（加一个 infra 容器）。要验证其状态，可以使用 `podman pod ps` 命令：

```
# podman pod ps
POD ID        NAME         STATUS      CREATED        INFRA ID      # OF CONTAINERS
8f89f37b8f3b  example_pod  Degraded    8 minutes ago  95589171284a  4
```

使用 `podman pod top` 命令，我们可以看到 Pod 中每个容器所消耗的资源：

```
# podman pod top example_pod
USER        PID         PPID        %CPU        ELAPSED          TTY         TIME        COMMAND
root        1           0           0.000       10.576973703s  ?           0s          sleep 1000 
0           1           0           0.000       10.577293395s  ?           0s          /catatonit -P 
root        1           0           0.000       9.577587032s   ?           0s          sleep 1000 
```

创建 Pod 后，我们可以检查网络的行为。首先，我们会看到系统中只创建了一个网络命名空间：

```
# ip netns
netns-17b9bb67-5ce6-d533-ecf0-9d7f339e6ebd (id: 0)
```

让我们检查一下此命名空间及其相关网络堆栈的 IP 配置：

```
# ip netns exec netns-17b9bb67-5ce6-d533-ecf0-9d7f339e6ebd ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether a6:1b:bc:8e:65:1e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.88.0.3/16 brd 10.88.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a41b:bcff:fe8e:651e/64 scope link 
       valid_lft forever preferred_lft forever
```

为了验证 `c1` 和 `c2` 容器共享相同的网络命名空间，并且它们使用 IP 地址 `10.88.0.3` 运行，我们可以使用 `podman exec` 命令，在容器内运行相同的 `ip addr show` 命令：

```
# podman exec -it c1 ip addr show
# podman exec -it c2 ip addr show
```

这两个容器预计将返回与 `netns-17b9bb67-5ce6-d533-ecf0-9d7f339e6ebd` 网络命名空间相同的输出。

示例 Pod 可以通过 `podman pod stop` 和 `podman pod rm` 命令分别停止和移除：

```
# podman pod stop example_pod
# podman pod rm example_pod
```

我们将在*第十四章*，*与 systemd 和 Kubernetes 交互*中更详细地讨论 Pod，届时我们还将讨论名称解析和多 Pod 编排。

在本节中，我们专注于在同一主机或 Pod 内，两个或更多容器之间的通信，而不管涉及的网络数量和类型。然而，容器是一个可以运行通常由外部世界访问的服务的平台。因此，在下一节中，我们将探讨暴露容器到其主机外部的最佳实践，并使其服务能够被其他客户端/消费者访问。

# 将容器暴露到我们底层主机外部

在企业公司或社区项目中采用容器可能是一件困难的事情，需要时间。因此，在我们的采用过程中，可能并没有所有所需的服务都作为容器运行。这就是为什么将容器暴露到我们的底层主机外部，可能是一个不错的解决方案，能够将容器中的服务与传统世界中的服务互联。

正如我们在本章稍早的部分简要看到的，Podman 使用两种不同的网络堆栈，具体取决于容器：无根或有根。

尽管底层机制略有不同，具体取决于你使用的是无根容器还是有根容器，Podman 用于暴露网络端口的命令行选项对这两种容器类型是相同的。

好知道

请注意，我们在本节中将要看到的示例将作为根用户执行。这是必要的，因为本节的主要目的是向你展示一些可能在暴露容器服务到外部时必需的防火墙配置。

暴露容器从端口发布活动开始。我们将在下一节学习这是什么。

## 端口发布

端口发布包括指示 Podman 在容器端口和某些随机或自定义主机端口之间创建临时映射。

指示 Podman 发布端口的选项非常简单——它包括将`-p`或`--publish`选项添加到`run`命令中。让我们看看它是如何工作的：

```
-p=ip:hostPort:containerPort
```

之前的选项将容器的端口或端口范围发布到主机。当我们为`hostPort`或`containerPort`指定范围时，两个范围中的数字必须相等。

我们甚至可以省略`ip`。在这种情况下，端口将绑定到底层主机的所有 IP。如果我们没有设置主机端口，容器的端口将随机分配一个主机端口。

让我们看一个端口发布选项的示例：

```
# podman run -dt -p 80:80/tcp docker.io/library/httpd
Trying to pull docker.io/library/httpd:latest...
Getting image source signatures
Copying blob 41c22baa66ec done  
Copying blob dcc4698797c8 done  
Copying blob d982c879c57e done  
Copying blob a2abf6c4d29d done  
Copying blob 67283bbdd4a0 done  
Copying config dabbfbe0c5 done  
Writing manifest to image destination
Storing signatures
ea23dbbeac2ea4cb6d215796e225c0e7c7cf2a979862838ef4299d410c90 ad44
```

如你所见，我们告诉 Podman 从`httpd`基础镜像启动一个容器。然后，我们分配了一个伪终端（`-t`），并在分离模式（`-d`）下设置端口映射，将底层主机的端口`80`绑定到容器的端口`80`。

现在，我们可以使用`podman port`命令查看实际的映射：

```
# podman ps
CONTAINER ID  IMAGE                           COMMAND           CREATED        STATUS            PORTS               NAMES
ea23dbbeac2e  docker.io/library/httpd:latest  httpd-foreground  3 minutes ago  Up 3 minutes ago  0.0.0.0:80->80/tcp  ecstatic_chaplygin
# podman port ea23dbbeac2e
80/tcp -> 0.0.0.0:80
```

首先，我们请求了正在运行的容器列表，然后将正确的容器 ID 传递给`podman port`命令。我们可以这样检查映射是否正常工作：

```
# curl localhost:80
<html><body><h1>It works!</h1></body></html>
```

在这里，我们从主机系统执行了`curl`命令，并且成功了——容器中运行的`httpd`进程刚好回应了我们。

如果我们有多个端口，并且不关心它们在底层主机系统上的分配，我们可以轻松地利用`–P`或`--publish-all`选项，将容器镜像暴露的所有端口发布到主机接口上的随机端口。Podman 会通过容器镜像的元数据查找暴露的端口。这些端口通常在 Dockerfile 或 Containerfile 中使用`EXPOSE`指令定义，如下所示：

```
EXPOSE 80/tcp
EXPOSE 80/udp
```

使用前面的关键字，我们可以指示容器引擎运行最终容器时暴露并使用哪些网络端口。

然而，我们可以利用一种简单但不安全的替代方法，如下一节所示。

## 附加主机网络

要将容器服务暴露给外界，我们可以将整个主机网络附加到运行中的容器。正如你所想，这种方法可能会导致未经授权使用主机资源，因此不建议使用，应该谨慎操作。

正如我们预期的那样，将主机网络附加到运行中的容器是相当简单的。通过使用正确的 Podman 选项，我们可以轻松地消除任何网络隔离：

```
# podman run --network=host -dt docker.io/library/httpd
2cb80369e53761601a41a4c004a485139de280c3738d1b7131c241f4001 f78a6
```

在这里，我们使用了`--network`选项并指定了`host`值。这告诉 Podman，我们希望让容器附加到主机网络。

在运行之前的命令后，我们可以检查运行中的容器是否已经绑定到主机系统的网络接口，因为它可以访问所有接口：

```
# netstat -nap|grep ::80
tcp6       0      0 :::80                   :::*                    LISTEN      37304/httpd
# curl localhost:80
<html><body><h1>It works!</h1></body></html>
```

在这里，我们从主机系统执行了一个`curl`命令，并且它成功了——容器中运行的`httpd`进程向我们做出了响应。

暴露容器到底层主机外部的过程并没有就此停止。在下一节中，我们将学习如何完成这项工作。

## 主机防火墙配置

无论我们选择利用端口发布（Port Publishing）还是将主机网络附加到容器，暴露容器到底层主机外部的过程并不会就此停止——我们已经达到了主机机器的基础操作系统。在大多数情况下，我们还需要允许传入的连接流入主机的底层机器，这将与系统防火墙进行交互。

以下示例展示了一种非详尽的方式来与基础操作系统防火墙进行交互。如果我们使用的是 Fedora 操作系统或其他任何使用 Firewalld 作为防火墙守护进程管理器的 Linux 发行版，我们可以通过运行以下命令来允许端口`80`上的传入连接：

```
# firewall-cmd --add-port=80/tcp
success
# firewall-cmd --runtime-to-permanent
success
```

第一个命令编辑了实时系统规则，而第二个命令则以永久方式存储运行时规则，这些规则在系统重启或服务重启后仍然有效。

知识点提醒

Firewalld 是一个防火墙服务守护进程，它为我们提供了一种简便而快速的方式来定制系统防火墙。Firewalld 是动态的，这意味着它可以在不重启防火墙守护进程的情况下创建、修改和删除防火墙规则。

正如我们所看到的，暴露容器服务的过程相当简单，但应在一定的意识和注意下进行：将网络端口开放给外部时，必须小心谨慎。

# 无根容器网络行为

正如我们在前面的章节中看到的，Podman 依赖于 CNI 插件或 Netavark 来运行作为 root 的容器，并且有权限修改主机网络命名空间中的网络配置。对于无根容器，Podman 使用 `slirp4netns` 项目，该项目允许您在不需要 root 权限的情况下创建容器网络配置；网络接口是在无根网络命名空间中创建的，标准用户在该命名空间中拥有足够的权限。这种方法允许您透明且灵活地管理无根容器的网络。

在前面的章节中，我们看到了如何通过 veth 对将容器网络命名空间连接到一个桥接网络。能够在主机网络命名空间中创建 veth 对需要 root 权限，而标准用户是不允许拥有这些权限的。

在最简单的场景下，`slirp4netns` 旨在通过允许创建一个连接到用户模式网络命名空间的 tap 设备来克服这些权限限制。这个 tap 设备是在无根（rootless）网络命名空间中创建的。

对于每一个新的无根容器，主机上都会执行一个新的 `slirp4netns` 进程。该进程为容器创建一个网络命名空间，并创建一个 `tap0` 设备，并将其配置为 `10.0.2.100/24` 地址（来自默认的 slirp4netns `10.0.2.0/24` 子网）。这防止了两个容器在同一网络中直接通信，因为会存在 IP 地址重叠的问题。

以下示例演示了一个无根 `busybox` 容器的网络行为：

```
$ podman run -i busybox sh -c 'ip addr show tap0'
2: tap0: <BROADCAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 2a:c7:86:66:e9:20 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.100/24 brd 10.0.2.255 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fd00::28c7:86ff:fe66:e920/64 scope global dynamic mngtmpaddr 
       valid_lft 86117sec preferred_lft 14117sec
    inet6 fe80::28c7:86ff:fe66:e920/64 scope link 
       valid_lft forever preferred_lft forever 
```

可以检查无根网络命名空间并找到相应的 `tap0` 设备：

```
$ podman unshare --rootless-netns ip addr show tap0
2: tap0: <BROADCAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 1a:eb:82:6a:82:8d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.100/24 brd 10.0.2.255 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fd00::18eb:82ff:fe6a:828d/64 scope global dynamic mngtmpaddr 
       valid_lft 86311sec preferred_lft 14311sec
    inet6 fe80::18eb:82ff:fe6a:828d/64 scope link 
       valid_lft forever preferred_lft forever
```

由于无根容器没有独立的 IP 地址，我们有两种方式让两个或多个容器相互通信：

+   最简单的方法可能是将所有容器放在同一个 Pod 中，这样容器就可以通过 localhost 接口进行通信，无需打开任何端口。

+   第二种方法是将容器附加到一个自定义网络，并让它的接口在无根网络命名空间中进行管理。

+   如果我们希望保持所有容器独立，我们可以使用端口映射技术发布所有必要的端口，然后通过这些端口让容器之间进行通信。

使用 Podman 4 网络后端，让我们快速关注第二种场景，其中两个 Pod 附加到一个无根网络。首先，我们需要创建网络并附加几个测试容器：

```
$ podman network create rootless-net
$ podman run -d --net rootless-net --name endpoint1 --cap-add=net_admin,net_raw busybox /bin/sleep 10000
$ podman run -d --net rootless-net --name endpoint2 --cap-add=net_admin,net_raw busybox /bin/sleep 10000
```

让我们尝试从 `endpoint1` 容器 ping `endpoint2` 容器：

```
$ podman exec -it endpoint1 ping -c1 endpoint1
PING endpoint1 (10.89.1.2): 56 data bytes
64 bytes from 10.89.1.2: seq=0 ttl=64 time=0.023 ms 
--- endpoint1 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.023/0.023/0.023 ms
```

这两个容器可以在公共网络上通信，并拥有不同的 IPv4 地址。为了验证这一点，我们可以检查无根容器的 `aardvark-dns` 配置内容：

```
$ cat /run/user/1000/containers/networks/aardvark-dns/rootless-net 
10.89.1.1
fe27f8d653384fc191d5c580d18d874d480a7e8ef74c2626ae21b118eedb f1e6 10.89.1.2  endpoint1,fe27f8d65338
19a4307516ce1ece32ce58753e70da5e5abf9cf70feea7b981917ae399ef 934d 10.89.1.3  endpoint2,19a4307516ce
```

最后，让我们展示自定义网络如何绕过`tap0`接口，并允许在 rootless 网络命名空间中创建专用的 veth 对和桥接。以下命令将显示`rootless-net`网络的 Linux 桥接和两个附加的 veth 对：

```
$ podman unshare --rootless-netns ip link | grep 'podman'
3: podman2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
4: vethdca7cdc6@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman2 state UP mode DEFAULT group default qlen 1000
5: veth912bd229@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman2 state UP mode DEFAULT group default qlen 1000
```

重要提示

如果你在 CNI 网络后端运行此代码，请使用`podman unshare –rootless-cni`命令。

根 less 容器的另一个限制与`ping`命令有关。通常，在 Linux 发行版中，标准的非 root 用户缺乏`CAP_NET_RAW`安全功能。这会阻止`ping`命令的执行，因为它需要发送/接收 ICMP 数据包。如果我们希望在 rootless 容器中使用`ping`命令，可以通过`sysctl`命令启用缺失的安全功能：

```
# sysctl -w "net.ipv4.ping_group_range=0 2000000"
```

请注意，这可能会允许任何在这些组中由用户执行的进程发送 ping 数据包。

最后，在使用 rootless 容器时，我们还需要考虑到端口发布技术只能用于`1024`以上的端口。这是因为，在 Linux 操作系统中，所有`1024`以下的端口是特权端口，标准的非 root 用户无法使用。

# 总结

本章中，我们学习了如何利用容器网络隔离，通过网络命名空间为每个正在运行的容器提供网络隔离。这些操作看起来复杂，但幸运的是，在容器运行时的帮助下，这些步骤几乎是自动化的。我们学习了如何使用 Podman 管理容器网络，以及如何连接两个或更多容器。最后，我们学习了如何将容器的网络端口暴露到基础主机外部，以及在 rootless 容器网络中可能遇到的限制。

在下一章中，我们将发现 Docker 和 Podman 之间的主要区别。这对于高级用户非常有用，同时也有助于新手理解通过比较这两个容器引擎可以期待什么。

# 深入阅读

想了解更多本章中涉及的主题，请查阅以下资源：

+   容器网络接口：[`github.com/containernetworking/cni`](https://github.com/containernetworking/cni)

+   GitHub 上的 Netavark 项目：[`github.com/containers/netavark`](https://github.com/containers/netavark)

+   GitHub 上的`Aardvark-dns`项目：[`github.com/containers/aardvark-dns`](https://github.com/containers/aardvark-dns)

+   CNI 参考插件： [`www.cni.dev/plugins/current/`](https://www.cni.dev/plugins/current/)

+   CNI 第三方插件：[`github.com/containernetworking/cni#3rd-party-plugins`](https://github.com/containernetworking/cni#3rd-party-plugins)

+   Kubernetes Pod 定义：[`kubernetes.io/docs/concepts/workloads/pods/`](https://kubernetes.io/docs/concepts/workloads/pods/)

+   `Slirp4netns`项目仓库：[`github.com/rootless-containers/slirp4netns`](https://github.com/rootless-containers/slirp4netns)
