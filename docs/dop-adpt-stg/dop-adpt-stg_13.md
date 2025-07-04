# 第十章：理解 DevOps 的技术栈

将工具加入到你的 DevOps 投资中是确保你的采纳从好变得更好的关键。如今市面上有许多 DevOps 工具。理解今天和明天应实施哪些工具集可能是一个挑战。在本章中，我们将探讨 DevOps 中主要工具的优缺点。

到本章结束时，你可以期待理解不同的 DevOps 工具家族，并了解工具如何在 DevOps 中发挥作用。你还将理解 DevOps 工具的好处和障碍。

在本章中，我们将讨论以下主要内容：

+   DevOps 工具的家族有哪些？

+   工具如何帮助推动 DevOps 的采纳？

+   理解 DevOps 工具的好处

+   理解 DevOps 工具的障碍

# DevOps 工具的家族有哪些？

我们所说的 DevOps 生态系统有许多不同的类别，工具被划分到这些类别中。一些工具是为了非常特定的目的设计、开发和营销的。也有一些行业特定的工具解决了独特的问题。

当然，你还会遇到一些工具，虽然它们特定于某一类别，但也适用于许多行业，有些工具是提供跨整个生态系统服务的工具套件。

我们可以使用一个传统的图表来描述 DevOps 循环，以讨论不同的类别。我喜欢使用以下这些类别，它们与传统图表中的类别非常接近，但有一些细微的差异：

+   协作

+   构建

+   测试

+   部署

+   运行

你可以在下面的图表中看到这一点的可视化表示：

![图 9.1 – 工具链阶段的可视化表示](img/B17192_09_01.jpg)

图 9.1 – 工具链阶段的可视化表示

让我们更详细地看一下这些，了解哪些类型的工具属于生态系统中的每个部分。

## 协作

我将协作加入到这个列表中，这个内容实际上在大多数传统列表中找不到，因为协作在 DevOps 中的重要性。所以到目前为止，我们已经从文化视角以及人员和流程的角度看过协作，但工具在协作中也非常重要。

你可以在组织内部围绕协作建立出色的流程、人员和文化，但最终，如果没有合适的工具，你仍然无法走得很远，规模化也会成为一个真正的问题。

当你想到协作时，很容易想到 Zoom、Microsoft Teams、Skype 等大型平台，但工具集远比这要广泛。协作也涉及到知识共享。

像 Read the Docs、GitHub Pages 和 Apiary 这样的工具都是文档工具；它们也被归类为协作工具。

知识共享在任何组织中都很重要，尤其是在那些高度协作并朝着 DevOps 实践努力的团队中，知识尤为重要。关于你产品的每一条知识都应该被记录下来并集中存储，以便在任何人需要时能够找到。

提示

可以将知识管理视为避免知识孤岛的一种方式。没有这些孤岛的情况下，你能够更好地扩展，并确保每个人都有平等的机会学习新技能，并了解更多关于产品的信息。

知识还包括文档。在我看来，如果没有相关的文档，不能认为某个工作已完成。这些文档可以是面向公众的，也可以仅供内部团队使用。不管哪种方式，文档都非常重要。

## 构建

构建工具使你能够将你开发的内容转化为可以在其他地方部署的成果。这从源代码控制工具开始。目前最常见的工具是**Git**。它有多种版本，其中最流行的是 GitHub，尽管 Git 技术也存在于其他不同的源代码控制产品中。

工具还支持持续集成。这一实践涉及将你的工件从源代码控制库提取，并通过一个叫做管道的自动化工作流进行处理。在管道中，许多任务会被完成，管道的结果是一个具体的工件，之后你可以将其部署到其他地方。

构建工具不仅与软件相关，你还可以使用一些工具来构建基础设施，这就是所谓的“基础设施即代码”。如果你使用的是容器，也会找到一些帮助你构建它们的工具。

如果你的应用程序涉及使用数据库，你还会找到数据库工具，帮助你管理数据库的架构和结构。

## 测试

测试是 DevOps 工具中最广泛的术语之一。在这里，你可以找到用于执行各种测试要求的工具。测试过程可以包括从开发者进行代码单元测试到用户验收测试，再到自动化浏览器测试的工具，这些都可以用于 Web 应用程序。

此外，测试还可以包括针对 OWASP Top 10 基准的安全扫描、静态和动态代码分析以发现漏洞，以及负载测试，以确保你的应用程序在负载下表现良好。

## 部署

部署的过程是将你构建和测试过的应用程序工件部署到需要去的地方。这可能是一个云平台，如 Microsoft Azure、Amazon Web Services 或 Google Cloud，也可能是一个移动应用商店，甚至是一个本地数据中心。

如果你将应用部署到云平台或应用商店，那么你很可能会使用本地工具来部署到这些环境。这些工具可能仅仅会将部署执行到这些特定的环境，而不会做其他操作。

如果你正在为其他开发人员编写可共享的应用程序库并部署到包管理库，你也可能会使用支持这种场景的工具。

如果你正在使用容器，那么你也很可能在使用制品管理工具；这些工具也属于部署范畴。此时，可以访问让你将容器指向公共或私有容器注册表的工具。

最后，即便你正在配置和部署虚拟机或其他类型的传统基础设施，你可能也会使用负责基线配置管理或基础设施配置的工具。这些工具在管理企业级基础设施时能够节省大量时间。

## 运行

一旦你部署了应用程序，就进入了所谓的运行阶段。在这一阶段，运营团队使用工具来管理应用程序。开发人员可能在此阶段也会使用一些工具来帮助监控应用程序性能和捕获异常。

运行阶段中的一些工具可能是你所使用平台中本地内建的；也有可能是额外的产品和工具。例如，你可能会使用云平台中的本地监控功能，然后使用应用性能监控工具来同时监控基础设施性能和应用程序性能。

根据经验，这是许多组织在工具使用上存在不足的一个领域；然而，正确的运行工具对于确保你拥有关于基础设施和应用程序性能的正确技术反馈至关重要，这样你才能做出基于数据的决策。

现在我们了解了 DevOps 工具链中不同类型的工具家庭，接下来让我们看看工具如何帮助 DevOps 的采纳。

# 工具如何帮助 DevOps 的采纳？

DevOps 利用其与敏捷开发的关系，进而创造一种促进协作和价值流的文化。这是通过将精益、约束理论和丰田生产系统等经过验证的原则与实践与敏捷开发结合实现的。

为了实现这一目标，DevOps 要求组织在团队内采纳文化变革，并采纳诸如自动化、版本控制和持续集成与交付等技术原则。与制造业类似，正确工具的整合对于充分实现 DevOps 内部技术实践的好处至关重要。

但是，有一点需要提醒：DevOps 不仅仅是使用工具——它是关于我们迄今为止学到的所有知识的结合，以及与工具的互动，从而充分实现 DevOps 的好处。

这里有一套可以帮助选择适合你组织工具的指导原则：

+   选择促进协作的工具。

+   使用能增强沟通的工具。

+   偏向有 API 的工具。

+   始终鼓励学习。

+   避免使用环境特定的工具。

现在让我们更详细地看一下这些指导原则，帮助更好地理解如何应用它们。

## 选择能促进协作的工具。

团队之间能够有效协作对于 DevOps 的成功至关重要。虽然有很多专门用于协作的工具，可能让你认为应该为此购买一个专用工具，但在 DevOps 工具链中，你可以使用许多不同的工具来增强团队之间的协作。

一个很好的例子是版本控制，它实际上是 DevOps 方法中的关键元素。当你在组织中鼓励更多人使用版本控制工具时，要考虑你的工具选择的影响。你和你信任的基础团队成员可能习惯了仅使用命令行工具，但其他人呢？对于那些更习惯使用用户界面的人来说，使用仅支持命令行的工具可能会成为一种障碍。

重要提示

在这个例子中，命令行工具用于版本控制，虽然是 DevOps 工具链的一部分，但对人们来说，尤其是非开发人员来说，它是陌生的。

在这种情况下，命令行工具的受众有限，因此协作机会非常少。然而，如果你采用像 GitLab、GitHub、BitBucket 或 Azure DevOps 这样的版本控制平台，你就可以利用关于文件更改和提交的讨论；这也是一种协作形式。

这有助于与拥有不同技能的人进行协作，并鼓励更多人学习如何根据自己的需求使用平台，从而促进协作。

我们在这里讨论的方法同样适用于工具链中的其他部分，而不仅仅是版本控制。它不仅仅影响新的工具；这也可以应用于你组织中现有的工具。

考虑权限对协作的影响。多次我曾与运维团队合作，他们拒绝向开发人员提供他们认为是自己工具的访问权限，认为其他人不应该有访问权限。如果你想提高协作，就应该向开发人员开放这些工具——结果会是两个团队之间的更好协作。工具本身没有改变，但权限改变了。

## 使用能增强沟通的工具。

根据我的经验，使用 DevOps 方法构建现代软件平台的组织中，最常见也是最大的问题之一，是团队职责与其工具之间的不匹配。

有时，组织会使用多个工具来完成一项任务，而实际上一个工具就足够了。反之亦然：有时组织只有一个工具，但当团队需要使用独立工具时，反而会产生问题。

团队间互动和沟通效果的最大影响因素之一，就是共享工具的使用。共享工具有助于团队之间的协作，但如果你需要明确责任边界，使用不同的工具可能是最好的选择。在*第二章*《业务效益、团队拓扑和 DevOps 的陷阱》中，我们讨论了业务效益、团队拓扑结构和 DevOps 的陷阱。可以利用此章节来帮助你了解哪种模型适合你的组织。

如果你希望开发与运营团队之间建立紧密的合作关系，那么使用独立的工单系统只会导致这两个团队之间的沟通不畅。为了帮助这两个团队更加高效，你应该选择一个能满足这两个团队需求的工具。

提示

在考虑选择工具时，首先要考虑团队关系，然后再为整个组织选择工具。

关键是要确保你查看整个组织，部署共享工具以便团队协作，并且在需要时，不要害怕使用独立的工具。

## 倾向选择具有 API 的工具

基于服务的架构和 API 驱动的应用程序是云原生或云就绪系统的基石。能够自定义且高度自动化的工具是一个重要的优势。具备全面功能的 API，尤其是基于 HTTP 的 API，是必须具备的。

从 DevOps 角度来看，使用 API 可以将你正在使用的多种工具连接在一起，作为你流程的一部分。符合这些标准的工具非常重要，因为当你需要将现有工具更换为新工具时，更换将一切连接在一起的“管道”会变得非常容易。

使用这种方法将工具链接在一起非常简单，但你需要小心这个过程中未记录的脚本。驱动软件交付和运营过程的工具应该像生产工具一样对待，这意味着要具备正确的文档编写和发布这些工具的能力。

在成熟度较低的组织中，常常犯下没有提供必要的操作支持就实施新工具的错误，导致这些工具无法发挥作用并有效地运作。

总结来说，你应该通过结合多个 API 驱动的工具来获得新能力。

## 始终鼓励学习

当你查看 DevOps 工具链中的工具时，你会发现其中一些相当复杂，尤其是在新人使用这些工具时。工具越复杂，你不应期待每个人都能迅速采用它们。

也可能发生相反的情况，当一个工具复杂且难以使用时，人们可能会固守己见，不愿使用它。这就是为什么你需要考虑为人们提供新的工具培训机会的原因。

引入新工具需要你评估组织内的广泛技能，然后为团队制定一条通往更高效工作方式的路线图。给人们提供按自己节奏学习的机会至关重要，因此，考虑拥有多种界面的工具，例如用户界面、命令行和 API，可以让每个人都能学习。

采用 DevOps 的过程是从手动操作到自动化的过渡；不是每个人都会在同一时间处于相同的位置，所以给人们提供能力，尤其是学习的空间，将使你有更大的机会让人们成功采用新的工具和方法。

把这看作是一个逐步演化的过程：通过引入令人害怕的工具来避免恐惧，并让人们有机会按照自己的节奏学习。就像敏捷方法通过短周期迭代实现渐进式改进一样，也应该以同样的方式对待你的工具，偏好那些小幅度的进步，而非未来状态和大爆炸式的方法。

## 避免使用特定环境的工具

随着 DevOps 的采用，交付的速度和频率会增加。这意味着，要想取得成功，你需要增加交付和运营过程中的反馈循环。尽可能多的技术人员应该学习尽可能多的生产工作原理，以便反过来构建更可靠、更具韧性的产品。系统的变更也应在部署到生产环境之前进行测试。

任何仅在生产环境中运行的工具都会引发问题，因为这会阻止人们学习，因为生产环境被视为一个特殊案例，而不是你应用程序运行的另一个环境。

为了尽可能提高效率，你应该选择能够在所有环境中工作的工具；在某些情况下，这甚至包括开发人员的本地环境。要留心那些按环境收费的部署或安装工具，尽量寻找提供站点范围许可证的工具，以便在可能的情况下降低成本。

也要考虑 DevOps 的自动化方法；优秀的 DevOps 工具应该能够在每个环境中自动设置。避免使用那些需要手动部署的工具——这些工具在 DevOps 中不是好的选择。

当你在每个环境中使用相同的工具时，你实际上在增加团队之间的互动，并增加学习的机会。将工具限制在生产环境中会将人们排除在学习机会之外。

总结来说，涉及特定环境工具时，必须避免使用它们。这会破坏学习的反馈循环，并使得你的持续集成和交付变得困难。

现在我们了解了工具如何帮助你采用 DevOps 方法，让我们来看看 DevOps 工具的好处。

# 了解 DevOps 工具的好处

根据 Puppet 发布的 *《DevOps 状态报告（2017）》* ([`puppet.com/blog/2017-state-devops-report-here`](https://puppet.com/blog/2017-state-devops-report-here))，*“高效且准确地开发和交付软件是所有组织的关键差异化因素和价值驱动因素。”* 尽管这份报告可能已经有几年历史，但其中的内容在我们谈论为什么实施 DevOps 时依然非常贴切。

报告发现，DevOps 组织在效率、满意度、质量以及组织目标实现方面，超过目标的可能性是其他组织的两倍以上。这些目标为成功的 DevOps 组织提供了重要的洞察力。那么它们是如何做到的呢？

以这些关键点为基准，整合正确的工具以成功应用 DevOps 技术实践，将使你实现以下好处：

+   增加代码和部署速度

+   缩短新产品和功能的上市时间

+   降低新版本发布的失败率

+   改进恢复平均时间

+   提高可靠性指标

+   提高协作与生产力

+   消除高水平的进行中工作和技术债务

我们已经通过本章前面的一些例子探讨了增强协作和生产力的好处；这些好处之间的一个共同主题是协作和沟通，但现在让我们用工具作为例子，更详细地看看这些好处。

## 提高代码和部署的速度

如果没有合适的工具，是无法准确衡量部署和代码速度的。你需要结合版本控制软件、任务管理工具和管道环境中的信息，才能获得关于这些数据的洞察。

拥有这些工具并能够准确报告速度是很重要的：它有助于你更好地进行团队规划，并理解你在一个迭代中能够完成多少工作。

从长期来看，理解这一点也有助于你做出关于何时增聘团队成员的决策，尤其是当你将这些信息与技术债务和进行中的工作相结合时。

## 缩短新产品和功能的上市时间

沟通和协作确实是这个过程的核心，尤其是在开始阶段。当你拥有能够促进团队间良好协作和沟通的好工具时，你可以更快地将你的创意从构想到生产。

再加上持续集成和部署工具的使用，一旦你的想法被开发并测试完成，它就可以在短时间内部署，经过各种测试周期。

仅这一点就成为许多组织，尤其是软件行业中的大企业，巨大的业务驱动因素。如果你能比竞争对手更快地将你的想法推向市场，这将给你带来竞争优势，最终让你的客户有更多理由选择你的产品而非他们的。

## 新发布版本故障率的降低

当你对管道进行投资，并使其能够一致地处理部署时，便需要关注反馈循环的必要性。在这种情况下，反馈循环是围绕提高构建和发布管道的质量展开的。

通过引入对管道的监控，并利用现有的 CI 和 CD 工具中内置的报告，你可以提取出显示管道执行成功的指标。

当故障发生时，将其视为一个学习机会，提取为什么会发生故障的信息，并找出你可以做些什么来提高质量，以确保问题不再发生。通过收集的信息，这个循环有助于降低发布的故障率，帮助你开发出更高质量的管道。

## 改进平均恢复时间

没有人喜欢应用程序停机，无论基础设施设计得多么优秀，应用程序开发得多么出色。你很可能在某个时刻会遇到停机问题。

单体环境或遗留工作环境通常对应用程序停机及其影响有着严苛的看法。今天，许多实践 DevOps 的组织不再仅仅关注**服务水平协议（SLAs）**，而是转向衡量**平均恢复时间（MTTR）**，即衡量从故障中恢复服务的平均时间。

在这个领域，站点可靠性工具至关重要，它能够帮助你尽可能多地获取有关停机的信息。这包括对你的基础设施和应用程序的 360 度监控，包括性能和异常；用户旅程；以及合成事务监控应用程序性能、可用性和安全性的方方面面。

重要提示

你还应该不要低估日志文件的重要性，虽然它们经常被忽视。

日志文件为你的故障排除能力提供了另一个维度。我经常看到开发人员和运维团队抓耳挠腮，因为他们没有适当的日志来诊断应用程序停机。

当你拥有所有这些信息，并且具备良好的沟通、协作和文档时，你更有可能尽快找出问题的根源。

测量应用程序端点的可用性同样让你能够衡量 MTTR。你可以测量从故障到恢复服务之间的时间。在每次故障之后，重要的是回顾收集到的信息，不仅要看看如何避免再次发生故障，还要思考如何进行流程或其他方面的变更，以减少下次恢复服务所需的时间。

## 可靠性指标的改进

紧随 MTTR 之后的是可靠性指标的改进。使用我们之前讨论过的许多相同原则，你还可以提高你应用程序各个方面的可靠性。

大多数站点可靠性和仪表工具都能够生成一个仪表板或评分卡，帮助你展示你的可靠性水平。如前所述，使用这些数据实际推进并提高可靠性是将组织在成熟度上区分开来的关键。

## 消除高水平的进行中任务和技术债务

在 DevOps 环境中，生产力的最大杀手之一是工作中的任务数量和某些组织中存在的高水平**技术债务**。我使用过的所有待办事项管理工具都可以让你通过仪表板或看板等方式标出每个团队成员的进行中的任务数量。

确保这里的工作量在可接受范围内。当然，什么是可接受的，通常因团队而异，当然也因个人、他们的技能以及他们能够完成的工作而异。

找到平衡点可能很难，但当人们有太多“进行中”项目时，生产力就会受到影响。我的经验告诉我，每个个体在进行中项目不超过三个时，保持高效。当数量超过这个点时，他们的生产力将开始下滑。

另一个生产力杀手是组织中的技术债务水平。技术债务，有时也叫设计债务，反映了应用程序中额外返工的成本。

例如，如果你今天用一种简单的方法设计了应用程序的某个部分，而不是采用更好的但更耗时的方法，那就是技术债务。我们通常会将技术债务加入待办事项中，稍后再回顾并解决。

为了了解你的技术债务水平，你需要一种可靠的方式来记录它。在一些工具中，你可以创建一个特定的待办事项模板来记录它，或者简单地为用户故事或缺陷添加标签——你可以决定如何记录它。

然后，你可以运行查询并将数据添加到仪表板中，以突出显示你的技术债务。当它达到你认为不可接受的水平时，你可以启动一个技术债务冲刺，旨在减少技术债务的数量。

在本节中，我们探讨了 DevOps 工具链中工具的好处，并看到不同的工具如何为我们的 DevOps 价值带来益处。接下来，我们将看看在成功部署工具时存在的一些障碍。

# 理解 DevOps 工具的障碍

到目前为止，我们已经讨论了 DevOps 工具链中工具的所有优点。那么，使用这么多不同工具带来的障碍是什么呢？以下是一些阻碍这些工具采用的障碍：

+   DevOps 定义的缺乏

+   缺乏对工具的知识

+   工具评估

+   市场上工具的数量

+   工具集成的缺乏

现在让我们更详细地探讨这些问题，更好地理解采用工具时面临的障碍。

## DevOps 结果定义的缺乏

不用多说，如果没有对 DevOps 的清晰定义以及它对你组织的意义，你将面临工具选型的困难。这会导致组织因竞争对手部署了工具或其他原始原因而部署工具。

拥有这些定义也能帮助你了解工具发展路线图可能是什么样子，并且如何使用我们在本章中讨论的所有建议，合理地弥补现有的工具空缺。

不仅你的组织需要对 DevOps 有明确的定义，而且这一点需要在整个组织内保持一致。从高层到基层的每个人都需要认同这一点，才能确保你在 DevOps 采用过程中每个环节的成功；这其中包括工具的使用。

## 工具知识不足

缺乏对可用工具的知识也是成功的障碍之一。这个问题可能以几种方式表现出来；然而，其中一些对整体进展的危害更大。

首先，知识的缺乏可能会妨碍你选择适合任务的正确工具。虽然没有人能在整个工具链中，甚至在某个特定领域成为专家，但拥有广泛的知识将帮助你根据从供应商那里获得的信息做出合适的评估。

其次，知识的缺乏可能会让你陷入困境——没有对工具的广泛了解，你可能会发现自己无法理解这些工具如何满足你的具体需求。这种延迟会影响你的实施进度。

第三，最后，当你缺乏工具知识时，最糟糕的后果就是让你的决策陷入瘫痪。与第一点中做出错误决策，或第二点中缺乏理解不同，这更多的是误解工具的真正目标。

## 工具评估

就像从任何供应商购买软件一样，你的组织肯定希望确保你能从任何演示或试用中获得最大收益。这是一个重大投资，你需要确保选择一个能满足需求、符合要求的工具。

考虑工具的成本与其提供的价值之间的关系。你必须确保购买的投资回报符合预期。

如果你正在评估多个工具，确保对它们的评估是公平的。对做相同事情的工具进行不同的评估是没有意义的。这不仅无法给每个候选工具展示其优点，还能确保决策过程中没有偏见。

每个人对不同的技术栈和工具都有偏好，但你需要确保选择适合组织的工具，而不是仅仅因为你过去使用过某个工具就继续使用它。

## 市场上工具的数量

当然，在你开始评估工具之前，你必须穿越各种 DevOps 工具链类别中庞大的工具海洋。

工具的数量庞大，**Cloud Native Computing Foundation (CNCF)** 甚至创建了一个网站工具，详细列出了其领域内的工具，并提供了多个过滤器帮助你限制搜索范围。这就是 *CNCF Cloud Native Landscape* ([`landscape.cncf.io`](https://landscape.cncf.io))。不要误会：这非常有用，但它也突显了我们在选择工具时面临的问题。

除了 CNCF 列出的工具外，当然还有一些不在 CNCF 生态系统中的其他工具。这份庞大的工具清单进一步加剧了你在选择可用工具时可能感受到的选择困难。

你可以通过查看独立的工具信息来源来突破厂商提供的营销内容。查看使用过这些产品的社区帖子，看看他们的使用案例，并利用这些信息帮助你筛选工具。

## 工具集成不足

在本章前面，我们讨论了 DevOps 工具中集成的重要性。使用缺乏足够集成的工具会限制你的能力，而这种限制的影响取决于其限制程度。

例如，如果持续集成和部署工具缺乏足够的集成，而平台又没有提供广泛的其他服务，这将成为一个问题。在查看这些工具时，我们应该寻找与待办事项管理工具、部署平台和票务系统（如果这是一个需求）的集成机会。

我们还必须考虑与提供安全服务的工具的集成，以便它们能为我们的整体目标提供价值。

缺乏集成会为你的工具创建一个黑盒，你无法与其交互，也无法从中提取可能有价值的数据。

# 摘要

在本章中，我们探讨了 DevOps 工具链中的工具。我们了解了不同类型的 DevOps 工具，并研究了工具如何帮助您的组织采用 DevOps。接着，我们分析了 DevOps 工具的好处，并试图理解可能阻碍高质量工具采用的障碍。

接下来，我们将探讨如何制定工具实施的策略。我们将讨论工具的架构和安全要求，以及如何制定培训计划来帮助团队，并将其与定义 DevOps 工具的负责人和流程结合起来。

# 问题

现在让我们回顾一下本章中所学到的一些内容：

1.  哪些工具会对整个 DevOps 工具链产生影响？

    a. 构建工具

    b. 测试工具

    c. 协作工具

    d. 运行工具

1.  为什么在采用工具时 API 如此重要？

    a. 其实不重要——我们不需要 API。

    b. 它们提供了自动化和集成的机会。

    c. DevOps 工程师需要了解 API。

    d. 所有优秀的工具本身都会配备 API。
