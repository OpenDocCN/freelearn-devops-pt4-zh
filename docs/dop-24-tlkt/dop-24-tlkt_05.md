## 第五章：先决条件

每一章都会假设你已经有一个正在运行的 Kubernetes 集群。无论它是一个本地运行的单节点集群，还是一个完全可操作的类似生产环境的集群，都无关紧要。重要的是你至少有一个集群。

我们不会详细讨论如何创建 Kubernetes 集群。我相信你已经知道如何操作，并且在你的笔记本电脑上安装了 `kubectl`。如果不是这种情况，你可能需要阅读附录 A: 安装 kubectl 并使用 minikube 创建集群。虽然 minikube 非常适合运行本地单节点 Kubernetes 集群，但你可能希望在一个更接近生产环境的集群中尝试一些想法。我希望你已经在 AWS、GKE、DigitalOcean、本地或其他地方运行了一个“真正的” Kubernetes 集群。如果没有，而且你不知道如何创建一个集群，请阅读 *附录 B: 使用 Kubernetes Operations (kops)*。它会提供你准备、创建和销毁集群所需的足够信息。

尽管 *附录 A* 和 附录 B 解释了如何在本地和 AWS 上创建 Kubernetes 集群，你不必仅仅局限于在本地使用 minikube 或在 AWS 上使用 kops。我尽力提供了关于一些常用 Kubernetes 集群变种的指导。

在大多数情况下，相同的示例和命令将在所有测试过的组合中有效。当无法直接适用时，你将看到说明，解释如何在你偏好的 Kubernetes 和托管环境中完成相同的操作。即使你使用的是其他平台，你也不应该有问题调整命令和规格以适应你的平台。

每一章都会包含一小段要求，说明你的 Kubernetes 集群需要满足哪些条件。如果你对某些要求不确定，我准备了一些 Gists，列出了我用来创建集群的命令。由于每一章可能需要不同的集群组件和规模，用于设置集群的 Gists 可能会有所不同。请将它们作为指南，而不必严格按照这些命令执行。毕竟，本书假设你已经具备一些 Kubernetes 知识。如果你从未创建过 Kubernetes 集群，却又声称自己不是 Kubernetes 新手，那就有些说不过去了。

简而言之，先决条件是具备 Kubernetes 的实际操作经验，以及至少一个 Kubernetes 集群。
