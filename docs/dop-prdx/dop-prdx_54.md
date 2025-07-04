## 第五十五章：介绍 Wian Vos

Wian Vos 是一位经验丰富的 DevOps/云计算顾问，在信息技术和服务行业有着丰富的工作经验。他擅长 PaaS、敏捷方法论、DevOps 和云技术。你可以通过 Twitter 上的 `@wianvos` 关注他。

**Viktor Farcic**：嗨，Wian！在我们深入讨论 DevOps 之前，你能告诉我们一点关于你自己的情况吗？

**Wian Vos**：我目前是全球最大开源公司之一——位于阿姆斯特丹的 Red Hat 的解决方案架构师。在 DevOps 这个术语出现之前，我就已经从事相关工作，2005 年以来一直参与基础设施自动化，2013 年以来投身于容器化的推进。在我的职业生涯中，我曾在荷兰的 ING 银行、Rabobank 以及一些其他较小的政府机构工作。最近，我在 Zaandam 的 CINQ ICT 担任 DevOps 管理顾问，之前我还在波士顿的 XebiaLabs 工作了两年。

## 定义 DevOps

**Viktor Farcic**：我想从我在这本书中问过其他所有人同样的问题开始：什么是 DevOps？每个人给出的答案都不一样。就我个人而言，我不明白为什么每个人的定义都不一样，但希望我们能在讨论中涉及这个话题。

"DevOps 在不同的时间点意味着不同的东西。"

—Wian Vos

**Wian Vos**：实际上，这正是我所预期的。我认为 DevOps 在不同的时间点意味着不同的东西。当这个术语最初被提出时，它基本上是基于 DevOps 宣言，推动一种新的工作方式，当时我觉得这个想法是有意义的。但后来，DevOps 变得流行了，正如所有流行的事物一样，大公司纷纷跟风——包括我目前的雇主，Red Hat——并将其变成了一个营销术语。

那么，DevOps 对我来说是什么？DevOps 是一种管理 IT 业务文化的范式。如果你从最纯粹的词义来看，它是一种将开发和运维放在同一个团队中，共同朝着相同的商业目标努力的方式。我从事 DevOps 已经九年了，但我从未加入过那些传说中的团队，也从未见过这些团队真正有效地运作。但我做过的是与 DevOps 类似的实践和工具密切相关。通过我的经验，我发现 DevOps 基本上就是关于文化和工作方式。

**Viktor Farcic**：如果你从未见过 DevOps 的实际运作，我必须承认，我也很少见过它真正发挥作用。这是因为公司失败了吗？还是因为这些公司根本没有尝试过按照应有的方式实施 DevOps？

**Wian Vos**：如果你想在公司实现 DevOps，你会面临一些障碍。首先是开发和运维之间的实际关系。当你面对一个新的初创公司或正在实施全新应用的公司时，这些关系不算大问题。为什么？因为你可以把一个团队聚在一起，他们可以各自做自己的事。

但是如果你看看传统公司基本上的组织方式，你会发现开发和运维之间总是存在着传统的分裂，而这基本上是因为它们各自有不同的视角。一方面，开发致力于追求稳定性，另一方面，运维则由业务推动，追求变革。在传统公司中将这两者结合起来，而不是在初创公司环境下实现，是非常困难的。

**Viktor Farcic**：如果是这样，你认为公司在想要在组织中实现 DevOps 时，面临的最大问题是什么？

**Wian Vos**：我一直认为 DevOps 公司是那些从底层投入技术的公司。它不仅仅是创建一个团队，更重要的是团队之间相互倾听，技术变革应该从底层决定，而不是由上层传达下来。

你问的是公司今天面临的最大问题。实施真正的 DevOps 时，我见过的最大挑战之一就是人员流动。为什么？因为这些团队——开发和运维——已经互相对抗了多年，而现在你又让经理去把这些人组织成一个团队。然后，旧的经理被替换成一个新的经理，他有不同的方法。

"我一直认为 DevOps 公司是那些从底层投入技术的公司。它不仅仅是创建一个团队，更重要的是团队之间相互倾听。"

—Wian Vos

如果不是为了那些有具体人数的员工，经理到底是为了什么？假设我是一个经理，有 20 个人，现在我要裁掉 10 个人。那么我现在是什么？嗯，我已经是曾经的半个经理了。我知道这听起来很严厉，甚至有些不尊重在座的经理们。从 DevOps 的角度来看，我遇到过好的经理。但是要让这种方式发挥作用，需要一种非常开放且非常特别的公司文化。

我认为 DevOps 是过去 10 年中我们所见到的巨大的开源技术推动的一个重要催化剂。但是在实践中，它是可怕的。嗯，说它可怕其实不准确，它只是根本无法实现。

## 什么是真正的敏捷…以及 Kubernetes 的重要性

**Viktor Farcic**：那么我再问一个后续问题。你见过多少公司是真正敏捷的？

**Wian Vos**：我见过很多真正敏捷的开发团队。我在 Xebia 工作了足够长的时间，足以真正了解敏捷是什么，在哪里实现，以及该寻找什么。

但要真正做到敏捷需要毅力，而这在今天的企业环境中是很难找到的。这不仅仅是两周站在一个白板前，贴满了便签。它远不止这些；它是一种思维方式。这并不是进入那个“我现在要这个功能，因为我有钱”的思维陷阱。而是更多地说：“好吧，让我们来规划这个，将它放入待办事项列表，并进行分类。”

我没有见过很多真正敏捷的公司，但我见过一些尝试变得敏捷的公司，在此之前，还有一些公司在尝试精益——从某种意义上来说，这是好的，因为这些公司努力将敏捷和精益融入其中，这最终会对它们的公司文化产生积极影响。但这并不意味着它们就变得敏捷、精益或 DevOps。如果你经营的是一个持续业务的运营商店，敏捷是最难做到的。

**Viktor Farcic**：那么，公司的文化在达到一定规模后就不再改变了吗？

**Wian Vos**：我不是这个意思。我想说的是，DevOps 涉及的技术本身不是问题。问题在于人，而一旦涉及到人，他们是非常难以改变的，尤其是在企业文化中。但是，记住，我从来没有在初创公司工作过。所以，我没有那种初创公司的经验。

**Viktor Farcic**：我想初创公司是另一个层面的事情，因为它们小，理论上可以做任何事情。它们没有什么负担需要去除。

但这是我在拜访公司时遇到的一个问题，尤其是当我试图向他们解释技术的全貌时。以微服务为例，它们是某种文化和某种环境下的产物，像其他任何技术或过程一样，最终它们变成了工具。

**Wian Vos**：或者是一个过程，或者是一个流行词。

**Viktor Farcic**：但如果你没有真正将其实施，或者只是简单地采用了工具，那么你会处于一个非常糟糕的境地，就像微服务的情况一样。没有自给自足的团队，能有微服务吗？没有改变文化，能有自给自足的团队吗？

至少根据我的经验，这种做法会失败得非常惨。但回到工具上，我们都曾为软件供应商工作过，根据我的经验，我有这样的印象：世界上每一个软件供应商现在都将他们的工具重新品牌化为 DevOps。在今天的会议上，所有的内容都是 DevOps。对我而言，我过去 10 年使用的所有工具都可以算作 DevOps。

**Wian Vos**：我不想说这本身就是一个问题，因为通常被购买的工具就像是不存在的 DevOps 神奇子弹。但将这些工具贴上 DevOps 标签确实有助于高层管理人员的采纳。如果某样东西被贴上 DevOps 标签，那么作为工程师或开发者的你就更有可能去使用它。

“……通常，购买的工具就像是那些不存在的 DevOps 神奇子弹。”

—维安·沃斯

如果你问我，“它们基本上从我们这里拿走了 DevOps 并且发展了下去，这是坏事吗？”我的回答是，我不知道。我确实反对那种观点——如果你有足够多酷炫的新工具和开源工具，那么你就可以称自己的公司为 DevOps 公司。这个我真的是受够了。我不知道这是好事还是坏事，因为它确实带来了许多酷炫的东西让我们去玩。

**维克托·法尔西奇**：接下来，我们聊聊 Kubernetes。现在 Kubernetes 是唯一统治一切的吗？

**维安·沃斯**：在接下来的两年里，这可能是对的。在我成为 Puppet 工程师之前的十年里，我在 Puppet 和 XebiaLabs 做了各种各样的工作，建设了平台即服务（PaaS）相关的东西，那时候很容易做出预测。早在 2001 年，大家就很容易说我们会在接下来的三到四年里使用 WebSphere ND，甚至可能在更长的时间内。

所以，在这个世纪的第一个十年里，很容易预测你未来五年会做什么，应该在哪里投资，应该专注于哪些领域。但是自 2009 年，甚至 2010 年以来，我就毫无头绪了。首先是平台即服务（PaaS）和资源配置。然后是容器化，或者说——我不知道你是否记得——不可变基础设施（immutable infrastructure），有 Foundry、Heroku 等那种酷炫的东西。

然后是 Docker 的出现，那简直是，*啊！* 但我们不能忘记，那个技术自 1990 年代末期就已经存在了。Docker 只是让它变得可用，而且在当时，Docker 是最酷的公司：每个人都听说过它，大家都想和它合作。但突然间，Kubernetes 横空出世，这有点搞笑，因为它和 Docker 几乎同龄，现在 Kubernetes 无处不在。每个人都在标准化它，大家都在使用它，大家都必须用它，大家都想和它一起工作。我认为这是过去十年里我见过最具说服力的技术，尤其是因为我们需要理清楚这个公共云的局面，而 Kubernetes 完美契合这一点。或许在三四年后，我们会对它感到厌倦，因为它本身就是个庞然大物。它很复杂，有时也很难搞定。它由 Google 和我们共同控制。

**维克托·法尔西奇**：我觉得关于 Kubernetes 最有意思的是，我不记得在我职业生涯中，最后一次见到一个软件、平台、应用——无论你怎么称呼它——被全球所有软件供应商全面采用是什么时候。即便是那些通常会等到其他人都采用新技术之后才跟进的传统软件供应商，也都支持 Kubernetes，这点我真的是没想到。

**维安·沃斯**：在 1970 年代，主机计算曾非常流行。但说实话，我认为上一次类似的情况发生在 Java 应用服务器的时候。所以，我不得不同意你的看法，这确实是一个朝着 Kubernetes 大规模发展的趋势。不过，大多数人其实并没有真正理解 Kubernetes 是什么，因为大部分人谈论 Kubernetes 时，都是在谈容器工作负载调度，这在一定程度上能够覆盖负载问题。

但如果你看它的真正优势以及为什么它会获胜，你会发现它为任何云平台提供了一个通用接口。通过实现一个 Kubernetes 集群，你可以在 AWS、Google Cloud、Microsoft Azure、电力云或任何云平台上几乎透明地部署工作负载，只要你不选择他们提供的本地容器解决方案。

## 看云计算的未来

**维克托·法尔奇克**：但这样不就成了对那些云供应商的威胁吗？如果一切都那么透明，我可以很容易地从一个云服务切换到另一个。

**维安·沃斯**：我认为这对我们作为消费者来说是个好局面。但这也促使这些供应商更激烈地争夺我们的业务。在过去五到六年里，如果你和 IT 界的任何大佬聊，话题无非就是云。在这段时间里，我们经历了前所未有的经济繁荣。所以，我在想，一旦经济开始衰退，会发生什么：人们会再次削减成本吗？如果是，那会怎样？我们会回到硬件时代吗？

**维克托·法尔奇克**：但云计算比本地部署更贵吗？

**维安·沃斯**：是的。

**维克托·法尔奇克**：我有一个理论，虽然我可能错了，那就是当你计算每个 CPU 的价格时，如果我们把管理本地基础设施的成百上千人算在内，我实际上认为当你考虑到人工成本时，云计算并没有那么贵。

**维安·沃斯**：只要你的云基础设施足够小，你可能是对的。但如果你看大规模的云实现，假设你在谈论成千上万的节点分布在多个云平台上，它们依然需要很多经过认证的，非常昂贵的专业人员来运行。我可以告诉你，拥有 AWS/Google Cloud 认证的人，远比那些一辈子只会操作 Cisco 交换机的人要贵得多。

**维克托·法尔奇克**：这确实很有道理。

**维安·沃斯**：所以，你可能用一个云专家的钱，聘请两个专家。

**维克托·法尔奇克**：我曾和一个人谈过，他的公司最初是在本地部署的，然后他们转向了云。最终，这家公司又回到了本地部署，理由是“哦，我们终于在云端学到了如何做好事情，所以现在我们终于知道自己需要在本地做什么了。”

**Wian Vos**：我认为这是一个正确的陈述。因为，基本上——我只对 AWS 非常熟悉——如果你看看 AWS 的工作原理，它基本上为你提供了与自己数据中心相同的所有组件。不同之处在于，它还为你提供了控制权，作为工程师或开发人员，你无需与网络人员就各种问题进行无休止的讨论——这个防火墙设置，或者那个防火墙设置——也不需要提交变更请求来来回回地沟通。不，你只需要坐在那里，做，改——好，完成。

你仍然需要知道你在做什么。并不是说亚马逊有一个神奇的网络设备，能够自动生成连接。它并不是那样工作的。因此，从这个角度看，我认为将你的基础设施搬到 AWS 是很好的，因为它澄清了你一直做错的很多事情。我认为这对所有情况都可持续吗？可能不行。

**Viktor Farcic**：但这对你的客户设定了某些期望。如果你是一个基础设施团队，而其他所有人都在使用 Azure、AWS、Google Cloud Platform 等，而你们还在本地，那么你们需要提升自己的水平，不是吗？

**Wian Vos**：是的，这就是我们将在接下来的三四年中看到的事情。公司将尝试弄清楚，最终，如何做到这种混合模式，既有你始终需要的 60%的生产能力，保存在本地。你将采用灵活的容量，这样你就能快速将产品推向市场。我真的认为这就是我们所需要的最佳状态。

**Viktor Farcic**：事后看来，你可能无法回答这个问题，因为你为一家公司工作，但我今天看到的一个问题是困惑。假设我们为一家公司工作，并且最终决定选择 Kubernetes。然后我们面临的问题是如何从 57 种流行版本和 500 种不太流行的版本中选择一种，这让我们面临了一个问题：“那我们现在该怎么办？”

**Wian Vos**：我只能给你一个建议：选择我们的——开玩笑的。话虽如此，我认为 Kubernetes 发展得太快，不适合在企业中选择自建（DIY）用于生产。我并不是说如果你经营一家初创公司，就不应该选择 DIY Kubernetes，因为 Kubernetes 的功能推送非常棒，说实话，如果我在上一个工作中有机会，我可能也会选择 DIY。但那只是自负而已。实际上，企业认为他们能够自己做 DIY Kubernetes，这是非常傲慢的。整个过程是一个大型项目，发展速度快得前所未见。

“我认为 Kubernetes 发展得太快，不适合在企业中选择自建（DIY）用于生产。”

—Wian Vos

基本上，在开源中，Kubernetes 的提交几乎比实际的 Linux 内核还要多。因此，我肯定会选择发行版，因为发行版为你解决了许多安全性问题，或者可能选择托管服务，其中你可以获得真正的 Kubernetes——而不仅仅是别人池中的保留命名空间，而是一个真正的 Kubernetes 服务。

**Viktor Farcic**：你提到了提交的数量和诸如此类的事情。对我来说，这很令人困惑，但它也表明 Kubernetes 需要放慢速度，以便人们能够真正理解它。因为现在，即使是选择 Ingress 网络也是一周的工作量。

**Wian Vos**：你提到这点真有趣。今天早上我正好讨论了 Kubernetes Ingress 的整个问题！但是是的，我非常同意。仅仅选择你的网络插件、边缘路由器之类的东西，很容易因为你做出的一些选择而自食其果。

**Viktor Farcic**：我认为这就是为什么每当有人告诉我要自己在 Kubernetes 上动手的时候，我的问题就简单地是，“为什么？”

**Wian Vos**：确实！你为什么要这样做呢？

**Viktor Farcic**：你不会花费别人花在整合上的同样时间，即使是和 Kubernetes 这样的人们一起度过了他们一生的人，比如 Kelsey，例如 Mike Powers。这有点像，“我不知道该选择什么，因为今天早上刚出现了一个新东西。”

**Wian Vos**：这确实就是这样。它是一个大家伙。事实上，这在早期的 2000 年代与 Linux 非常相似。如果你看看当时实际可行的 Linux 发行版数量，那简直是奇怪。有那么多不同的风味和你可以做的事情，但最终都归结为现在的两三个主要 Linux 发行版。所以，我认为 Kubernetes 会像 Linux 一样发展。

我觉得当前的生态系统很好，因为它带来了竞争，这也带来了变化。但我认为 Kubernetes 肯定会长久存在，因为我们的数据中心变得更加复杂。我知道我在与之前说的话相矛盾，但这就像是数据中心的核心，可能会在未来十年或二十年内持续存在。但生态系统会更少，选择也会减少。

## 企业的问题

Spotify 并没有进行自己的 DIY Kubernetes，这让我思考。因为工程上来说，Spotify 是最杰出的公司之一。如果你看看他们在做什么，他们推出的东西，以及他们遇到的少量故障，他们一定是在做一些正确的事情。如果像他们这样的公司说，“不，我们不会自己动手做 Kubernetes”，那对其他人来说应该是一个信号，“好吧，如果班里最聪明的孩子不去做，我应该做这个吗？”

**Viktor Farcic**：但这不正是企业普遍存在的大问题吗？每个企业都认为自己比所有聪明人都聪明，认为自己是与众不同的。这个我经常听到：“我们将自己推出去，就像十年前我们推出自己云服务时失败的方式一样，唯一的区别是这一次，我们一定会成功！”

**Wian Vos**：我同意。在我目前的职位上，我为许多企业提供有关 DevOps 和容器化的咨询。是的，特别是在技术水平较低的企业中，很多人做了资源配置的工作，带来了这些变化，并使公司进入了不同的轨道。

大家都在想 Kubernetes 就像是实现 Puppet 或 Jenkins 一样。但你必须把 Kubernetes 当作一种不同的东西来看待，否则它会突然跳出来咬你。我不是在吓唬你，也不是说你不应该尝试它。因为，最终，它是有趣的，且是一次很好的体验。它能帮助你更好地理解 Kubernetes 是如何运作的，希望最终你会得出这样的结论：如果你足够聪明，你就不会自己去做这件事。

**Viktor Farcic**：好吧，那么假设我们得出了结论，不打算自己做这件事。我们将选择一个现有的平台——那接下来怎么办？我们是不是把老旧的数据库放进 Docker 镜像，然后在 Kubernetes 中部署？

“DevOps 中最大的问题永远是持久化数据。”

—Wian Vos

**Wian Vos**：哦，天哪！这是个棘手的问题。我想你知道，首先，DevOps 中最大的问题永远是持久化数据，也就是数据库中的数据，其次是传统数据库并不是为了良好配合而设计的。大型企业中的典型数据库非常昂贵，更不用说管理起来是多么的糟糕。所以，我会建议你从公司里先把这些东西剔除出去。

**Viktor Farcic**：好的，这一点我们达成共识。但对我来说，最让我感兴趣的是，我有种感觉，企业完全没有意识到，在 Kubernetes 之外，它们需要做多少额外的工作才能让事情在里面成功，即使是他们自己应用的情况也是如此。

**Wian Vos**：这不仅仅是实现问题，也不仅仅是构建 Kubernetes 集群。是第二天和第三天的操作会让你头疼。

**Viktor Farcic**：这确实非常真实。

**Wian Vos**：再次强调，并不是说你必须将你做的所有事情都迁移到 Kubernetes 平台。将数据库运行在 Kubernetes 之外完全没问题。问问自己这个问题：你真的需要数据库集群如此高的灵活性吗？你可能需要，但并不是每个人都需要。那么，你真的需要 Kubernetes 吗？我不知道。

**Viktor Farcic**：但对我来说，这是一个很有意思的问题，因为我不得不问自己，几年后，Kubernetes 还能算是一个选择吗？是的，我们确实可以选择不使用 Kubernetes。但如果其他所有厂商都在 Kubernetes 上发布新版本，那这真的是一个选择吗？

**Wian Vos**：我认为它应该是一个选择，因为如果它不再是一个选择，那么创新就死了，如果真是那样，我们将不得不为操作系统创造一个全新的生态系统，而这永远不会再发生。但在整个 Kubernetes 场景以及它周围发生的事情中，确实有一些有趣的变化。例如，我们收到了大量关于在裸机上运行 Kubernetes 的问题，我认为这是未来三个月左右的一个大趋势。因为，为什么要在 Kubernetes 上做虚拟化呢？我不知道——告诉我，为什么你想在虚拟机（VM）上运行 kubelet，而不是直接在裸机上运行？

**Viktor Farcic**：没有什么特别的理由。但另一方面，有些人使用虚拟机，因为他们仍然不知道自己在做什么。

**Wian Vos**：是的，这非常正确。

**Viktor Farcic**：我把它看作是一种演变，一旦你真正明白你在做什么，你就会把虚拟化层去掉——但不会在这之前。

**Wian Vos**：也许吧。我认为其中一个最大的问题是，我们有一代 IT 人才进入这个领域，他们从未在虚拟机以外的地方工作过。

## 大规模 DevOps 杀手

但是，如果你看看 Kubernetes 和你在做的事情，实际上在上面加一个虚拟化程序是没有任何意义的。因为对 Java 虚拟机（JVM）来说，它本质上是操作系统和应用程序之间的虚拟化层，而你是在容器中运行它。可以说，它就像是一个虚拟的隔离层。这不是虚拟化，但你明白我的意思。然后，如果再在下面加一个虚拟化层，就会导致你把所有的东西从 CPU 和内存中拉得很远。

**Viktor Farcic**：这可能是对的，但那服务器无状态呢？那是下一个趋势吗？

**Wian Vos**：我认为无状态就是 DevOps 的最大杀手。

**Viktor Farcic**：什么意思？

**Wian Vos**：我认为这基本上和我们在 1990 年代末以及 1970 年代时所做的是一样的。作为开发者，你不希望被操作团队正在做的事情困扰，因此，我们有六到七年的时间，我们大家都在假装共同协作。但现在出现了一个新事物，实际上是一个旧事物，你只需要将代码放进去，设置一个路由，就可以开始了！基本上，它将操作团队的工作抽象化了，因为总还是得有人负责那个无状态系统。

**Viktor Farcic**：那时，毕竟还是有服务器的。

**Wian Vos**：是的，这是真的。但记住，还是需要有人来安装和维护它，因为，当然了，无服务器系统永远不会出问题——就像 IT 中的其他任何东西永远不会出问题一样，对吧？我认为这是终结 DevOps 的范式。

这也是我认为无服务器是 DevOps 的“大杀手”的原因。

**Viktor Farcic**：现在我是在谈论原则层面的东西，而不是具体的实现：我困惑的是，无服务器和 Kubernetes 到底有何真正不同？

**Wian Vos**：其实并没有真正的不同；这就是关键。我确实想区分一下无服务器范式和实际的无服务器平台。Kubernetes 只是一个强大的无服务器推动者，而无服务器范式则是这样说的：“好了，我是一个开发者，我有代码，我把它放到这里，任务就完成了。”我认为，真正的无服务器平台自从平台即服务（Platform-as-a-Service，PaaS）时代就已经存在。如果你有一个实施得很好的平台即服务，那对于应用开发者来说，使用它简直是显而易见的。

我认为有一个大范式从 DevOps 的阴影中走了出来，并且实际上是有效的：站点可靠性工程（SRE）。拥有一个优秀的 SRE 团队，为你提供一个开发者可以直接使用的平台，真是太棒了。但这算是无服务器吗？我不确定。在 SRE 模型中，你仍然需要一位 SRE 工程师来帮助你将代码集成到平台中。现在，如果你将足够的功能抽象化，开发者就不再需要那位工程师了。那么，嘿，没问题，一个无服务器平台——你不再需要担心服务器了。但不要忘记，背后依然有服务器，而且背后有一个 SRE 团队实际管理着这些东西并在其上进行创新。

所以，对于你作为开发者来说，这变成了无服务器。但随之而来的是，开发者和工程师之间的互动再次丧失了，我认为这将再次阻碍创新，因为没有人会从不和别人交流中变得更好。

**Viktor Farcic**：没错。当你最初说 DevOps 死了的时候，我很困惑。但现在，我同意了。

**Wian Vos**：如果没有别的，DevOps 就是应用开发人员和构建平台的工程师之间的沟通。我曾经写过一篇博客文章说过这一点，结果得到了非常糟糕的反馈。所以，让我再明确一下。我不是说仅仅是那种沟通，但我确实认为它是一个非常重要的组成部分。

## DevOps 工程师的角色

**Viktor Farcic**：但如果 DevOps 的重要组成部分是开发人员和运维人员之间的沟通，那么 DevOps 工程师的角色到底是什么？当我看到职位描述时，我觉得 DevOps 工程师的职位目前似乎比其他任何职位都要重要。

**Wian Vos**：基本上，这只是为了混淆招聘。

**Viktor Farcic**：还有 DevOps 部门！

**维安·沃斯**：想象一下，假设你和我要成立一家公司。我们需要一个 DevOps 团队，因为我们有一个迫切的愿望要推出这个超棒的应用程序。然而，在我们的情况下，我们只能雇佣五个人。那么，当我们组建 DevOps 团队时，我们面临的问题是，“我们要雇佣谁？”以及“我们要雇佣什么？”

我们会雇佣 DevOps 工程师吗？不会。在那个团队里，我们需要最优秀的应用开发人员，最优秀的测试人员，也许还有一位优秀的基础设施专家以及前端/后端开发人员。我希望 DevOps 团队里的成员是具有特定角色且能够良好协作的人员。

当 DevOps 成为软件公司的一个营销术语时，招聘也赶上了这一潮流。红帽公司正在组建一个 DevOps 团队，所以我们现在需要一个 DevOps 工程师，招聘团队表示他们会为你找到一个 DevOps 工程师。但正如你所说，对于市场上的许多人来说，这依然是一个非常有吸引力的职位，因为职位名里包含了 DevOps 这个词。在那个人的脑海中，他们不再做工程工作；他们现在做的是 DevOps。

**维克托·法尔奇克**：在这一点上，我必须感谢敏捷方法。你永远不会看到一位敏捷工程师。

**维安·沃斯**：不，他们没有 DevOps 工程师，但他们确实有敏捷教练，这其实是另一种说法，就是不穿领带的经理。不过，公平地说，敏捷教练的视角确实与众不同。具体来说，更偏向于辅导和支持，而不是推动和让别人负责。如果你看敏捷和项目管理，敏捷是胡萝卜，而项目管理则是大棒。它们是不同的方式，而我可以告诉你，胡萝卜总是更有效。

**维克托·法尔奇克**：那么，你的工作就是拜访公司，向他们展示隧道尽头的光明，目标是帮助他们改进。我很好奇，当你拜访公司时，最讨厌的是什么？实现你目标的主要障碍是什么？

**维安·沃斯**：我认为问题几乎总是出在人身上。我曾经遇到过的最大问题，以及花费我最多时间和精力的事情，就是在实施 DevOps、平台即服务（PaaS）等新型基础设施或新型应用程序使能技术时，许多公司仍然有一批老旧的架构师。这些人实际上是与新技术和平台打交道的，如果他们看到新平台的好处，他们很快就能在新的平台或无服务器平台上运行他们的应用程序（不管你怎么称呼它）。虽然外面有一些很好的架构师，但公司中的架构师则是另一回事，特别是在政府部门。在政府部门，如果你有一个计划，而且这是一个好计划，它只有在那儿发明的才算是好计划。

比如，在一个政府部门，我们基本上在没有架构批准的情况下就开始构建新平台，然后在项目进行到四个月时才去争取架构批准。幸运的是，我们的赞助人位高权重，董事会成员中有足够影响力的人，能够推动这个项目。如果他们没有这样做，整个平台就会被架构师们取消——架构师们可能会说，“是的，但是有一个小细节我们不喜欢”，之类的。就像是，“好吧，我们没有考虑到这个，所以这一定不好。”

还有另一个政府机构，我在那也做了同样的事情。我们让 CTO 告诉我们：“好了，我只希望你们把这个做出来。我不在乎你们怎么做，但要尽最大努力做好，事后再告诉架构师们，完成后把文档交给我。”有很大的可能性，我们实现了一个新特性，三个月后，一个架构师会过来说：“你们没告诉我这个功能已经实现。” 是的，好吧，但我们三个月前就实现了这个功能，而且我们构建的平台非常成功。我的意思是，如果有开发人员来找我们说，“嘿，能不能改这个或那个？”，我们可以在一两次发布中做出更改，发布大概在三周后。但是如果你必须经历整个老派的企业架构过程，那么你就完了；你就没戏了。

**Viktor Farcic**：是的。我在规划方面也有同样的问题。

**Wian Vos**：顺便说一下，我曾在几次场合被称为架构师。对我来说，区分一个好架构师和一个普通架构师的标准是，你绝不能设计任何你自己至少无法实现 80%的东西。

**Viktor Farcic**：这些架构师中有多少人实际在实施东西？我的意思是，我遇到的大多数架构师，他们的工具是微软办公软件——他们在写 Word 文档——而一个成功的架构师是那种能够写超过 200 页文档的人。

**Wian Vos**：去年，我参加了 CLOUDBUSTING，这是一场由 Software Circus 组织的迷你会议，Software Circus 是我们在荷兰的一个聚会小组。在这次会议上，我听到了一位讲者分享了几个很好的例子，说明当你让传统的 IT 架构师进入公司时，建筑失误是如何发生的。因为你可以写出一份出色的架构文档，交给实际构建的人员，结果发现它根本行不通。特别是在今天的软件世界里，认为你能预见一切是非常自大的。尤其是当你从未亲自构建过它时——你根本不知道什么有效。

“我最喜欢的架构风格是进化式架构。我们需要供应工具，那就选三个工具，先试用一周，真正给它们来一次全面的测试，看看哪些有效。”

—Wian Vos

我最喜欢的架构风格是进化式架构。我们需要进行预配置，所以我们可以选择三个工具，测试一周，真正地把它们用到极限，看看哪个有效。一周结束时，你可以说：“好，这个可以用，但另外两个不行。所以我们就基于这个进行创新。可是，我们要如何实现它？让我们尝试三到四种不同的方式，看看哪个最好，然后在那个基础上创新。”

所以，我真的认为，架构过程不应成为障碍，而且你需要在团队中有架构师。如果你正在运营一个为客户构建平台的 SRE 团队，确保具备架构技能——更重要的是具备架构责任感的人——在这个团队中，与大家一起构建它。就像一个领导工程师一样，有权与团队其他成员一起做决策。

**Viktor Farcic**：没错。我唯一想补充的是，他们也需要感同身受。我相信如果一个人无法感受到那些决策背后的痛苦，他或她就不应该做任何决策。通常，架构只是，“给你一个图表，帮助你去实现它。”

**Wian Vos**：没错，但话说回来，最初 DevOps 的核心就是这个。你将一部分业务责任交给一个团队，由他们去构建和运行。所以，如果一个应用程序开发人员写出了有问题的代码，它无法正常运行，那么凌晨两点钟被叫醒的不会是运维人员，而是实际的应用程序开发人员，因为这样，他们会更有动力去构建一个能正常运行的东西。

**Viktor Farcic**：太棒了。我不想占用你太多时间，但你有什么总结性的评论吗？

**Wian Vos**：我只想说这一点：在整个 DevOps 讨论中没有绝对的对与错。这更多的是关于我认为 DevOps 已经变成了一种文化推动力，我认为让实际使用和构建平台的人选择他们知道适合公司的东西是非常重要的。但同时，他们也需要被允许在公司内分享他们的知识。

## 庆祝你的失败

我知道这有点感性，但实际上，了解的一个关键点是也要庆祝你的失败。如果你没有表达自己失败的事实，并探索为什么以及怎样失败，你就错过了一个学习的机会。一旦你开始庆祝失败，人们会不再害怕失败。我还认为，最具创新精神的工程师是那些敢于创新的工程师。

**Viktor Farcic**：我们只需要说服管理层，在失败时不要解雇员工。

**Wian Vos**：没错！这是作为经理在 DevOps 中需要进行的最重要的思维转变之一。

**Viktor Farcic**：但这不就是在接受不可避免的失败吗？是在说你知道自己最终会失败吗？

**Wian Vos**：没错，但如果你不接受失败，你的团队会替你掩盖问题，他们什么也学不到。或者即使失败的人能学到些什么，其他人也什么都学不到。我想这就是 GitLab 大概一年前做的——admin 55——那件事。

**Viktor Farcic**：完全正确！在那之后，我对他们的尊敬无法用语言表达。GitLab 是我的英雄，完全是因为他们处理失败的方式。

**Wian Vos**：他们说，“嘿，我们搞砸了。来，看看我们怎么做的。”

**Viktor Farcic**：是的，我记得。我在看他们的视频直播，他们修复问题时就像在看一部拉丁美洲肥皂剧。太棒了。

**Wian Vos**：那太棒了，我认为这应该是大多数公司愿意做的事情。但现在，我们离这个目标还有很长的路要走。

**Viktor Farcic**：非常长。至少当我访问公司时，我感觉自己还没有资格像那样表现。

**Wian Vos**：嗯，我得说，在荷兰情况已经在变好。正如我之前所说，我还在美国工作过两年，那里的情况完全不同。

**Viktor Farcic**：我觉得这里是个很好的停顿点。感谢你的时间，我真的很享受这个过程。

**Wian Vos**：没问题，非常感谢。
