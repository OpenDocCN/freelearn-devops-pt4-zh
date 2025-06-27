# 序言

## 关于

本节简要介绍了作者、全书内容概览、入门所需的技能以及完成所有技术主题所需的硬件和软件。

## 关于《DevOps 文化与实践（OpenShift 篇）》

*DevOps 文化与实践（OpenShift 篇）* 涉及了许多不同的实际操作——包括一些与人相关的、一些与流程相关的、一些与技术相关的——以促进 DevOps 的成功实施，从而推动 OpenShift 在组织中的采用。它介绍了许多 DevOps 概念和工具，通过持续的发现、调整和交付循环，将文化和实践联系在一起，并以协作和软件工程为基础。

容器及容器化应用生命周期管理已成为行业标准，而 OpenShift 在基于 Kubernetes 的企业产品市场中占据领先地位。*DevOps 文化与实践（OpenShift 篇）* 提供了一条在组织内构建高效能产品团队的路线图。

本指南将精益、敏捷、设计思维、DevOps、文化、引导和实践技术实现结合在一本书中。通过结合真实案例、实践性案例研究、引导指南和技术实施细节，*DevOps 文化与实践（OpenShift 篇）* 为在 Red Hat 的 OpenShift 容器平台上构建 DevOps 文化提供了工具和方法。

### 关于作者

**蒂姆·比蒂**是 Red Hat Open Innovation Labs 的全球产品负责人及高级首席参与顾问。他在产品交付方面的职业生涯已有 20 年，担任过敏捷和精益转型教练——一位持续交付和设计思维的倡导者，他将人们凝聚在一起，构建有意义的产品和服务，同时帮助大型公司向业务敏捷转型。他和妻子及狗杰拉德（一只拉布拉多犬，生活中的另一只“实验室犬”）一起住在英国温彻斯特，30 岁时他从猫派转变成了狗派。

**迈克·赫本**是 Red Hat Open Innovation Labs 的全球首席架构师，帮助客户转变工作方式。他的大部分工作时间都在帮助客户和团队通过 OpenShift 转变交付应用的方式。他是《DevOps 与 OpenShift》一书的合著者，喜欢户外活动、家人、朋友、好咖啡和好啤酒。迈克喜欢大多数动物，但不喜欢澳大利亚的巨大毛蛛（猎蛛），通常他是猫派，除非是星期二，那天他是狗派。

**Noel O'Connor**是 Red Hat EMEA 解决方案实践的高级首席架构师，专注于云原生应用和集成架构。他曾与 Red Hat 的众多全球企业客户合作，遍及欧洲、中东和亚洲。他是《DevOps 与 OpenShift》一书的合著者，并不断尝试学习新事物，取得了不同程度的成功。Noel 更喜欢狗而非猫，但最终被团队其他成员否决了。

**Donal Spring**是 Red Hat Open Innovation Labs 的高级架构师。他在交付团队中袖手高卷，处理任何需要解决的问题——从指导和辅导团队成员、设定技术方向，到编写代码和测试。他热爱技术，喜欢动手探索新技术、框架和模式。周末时，他经常可以在个人项目中编程并自动化所有事务。猫或狗？他两者都喜欢 :)

### 关于插画师

**Ilaria Doria**是 Red Hat Open Innovation Labs 的参与领导和首席顾问。2013 年，她进入了敏捷领域，成为一名教练，帮助大型客户进行数字化转型。她的背景是终端用户体验和咨询，运用开放实践引领复杂的转型，并在大规模组织中推广敏捷。彩色便签和涂鸦一直是她生活的一部分，这也是她为本书提供所有插图并构建所有数字模板的原因。她绝对是一个狗的爱好者。

### 关于评审员

**Ben Silverman**目前是 Cincinnati Bell 技术服务公司全球客户团队的首席架构师。他还是《OpenStack for Architects》、《Mastering OpenStack》、《OpenStack – Design and Implement Cloud Infrastructure》一书的合著者，并且是《Learning OpenStack》（Packt 出版）的技术审阅者。

当 Ben 不在写书时，他活跃于 Open Infrastructure Superuser 编辑委员会，并且是 Open Infrastructure Foundation 文档团队（架构指南）的技术贡献者。他还领导着位于亚利桑那州凤凰城的 Open Infrastructure 用户小组。Ben 经常受邀在客户活动、聚会和特别供应商会议上发表关于云计算和 Kubernetes 的采用、实施、迁移以及文化影响的演讲。

### 学习目标

+   在你的组织中实施成功的 DevOps 实践，并进而实现 OpenShift

+   在持续交付的世界中处理职责分离

+   通过以应用为中心的视角理解自动化及其重要性

+   管理持续部署策略，如 A/B 测试、滚动发布、金丝雀发布和蓝绿发布

+   利用 OpenShift 的 Jenkins 功能执行持续集成流水线

+   管理并将配置与静态运行时软件分离

+   掌握沟通与协作，通过持续发现和持续交付在规模上交付卓越的软件产品

### 受众

本书适用于任何对 DevOps 实践（无论是 OpenShift 还是其他 Kubernetes 平台）感兴趣的读者。

本书为软件架构师、开发人员和基础设施运维工程师提供了对 OpenShift 的实用理解，教您如何高效使用 OpenShift 进行应用架构的有效部署，并与用户和利益相关者合作，交付具有业务影响的成果。

### 方法

本书将简洁的理论解释与实际案例相结合，帮助您发展成为一名 DevOps 实践者或倡导者。

### 硬件和软件要求

有五个章节深入探讨技术。*第六章*，*开放技术实践——起步，正确开始* 和 *第七章*，*开放技术实践——中期* 专注于启动技术环境。*第十四章*，*构建它*，*第十五章*，*运行它*，以及 *第十六章*，*拥有它* 讲解了将功能开发和运维到我们在 OpenShift 平台上运行的应用程序中。

我们建议所有读者，无论其技术水平如何，都深入探索本章节中解释的概念。您也可以选择亲自尝试一些技术实践。这些章节提供了相关的指导。

运行这些练习的 OpenShift 配置要求已在附录 A 中列出。

### 约定

文本中的代码字、数据库名称、文件夹名称、文件名以及文件扩展名如下所示：

我们将涵盖使用 Jest 对 PetBattle 用户界面进行组件测试的基础知识。用户界面由多个组件构成。您第一次进入应用程序时看到的组件是首页。对于首页组件，测试类被命名为 `home.component.spec.ts`：

```
describe('HomeComponent', () => {
  let component: HomeComponent;
  let fixture: ComponentFixture<HomeComponent>;
  beforeEach(async () => {...
  });
  beforeEach(() => {...
  });
  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

### 下载资源

所有技术资料都可以在本书的 GitHub 仓库中找到：[`github.com/PacktPublishing/DevOps-Culture-and-Practice-with-OpenShift/`](https://github.com/PacktPublishing/DevOps-Culture-and-Practice-with-OpenShift/)

所有视觉资料的高分辨率版本，包括照片、图表和数字化工件模板，均可在 [`github.com/PacktPublishing/DevOps-Culture-and-Practice-with-OpenShift/tree/master/figures`](https://github.com/PacktPublishing/DevOps-Culture-and-Practice-with-OpenShift/tree/master/figures) 上找到。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，您可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 中查看。快来看看吧！

我们意识到技术会随时间变化，API 也会不断演进。有关技术内容的最新变更，请查看上述本书的 GitHub 仓库。如果您在使用过程中遇到任何问题并希望直接与我们联系，请在该仓库中提一个问题。
