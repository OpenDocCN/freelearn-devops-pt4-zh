# 第六章：测试设计

在查看了可测试性框架、测试工具、标准测试和库之后，我已经向你展示了平台和应用程序中可用的内容，允许你创建和执行自动化测试。这就是我们在本书这一部分要做的事情。但是，让我们退后一步，不要急于跳入代码编写。首先，我想介绍几个概念和设计模式，这些将帮助你更有效、更高效地构思你的测试。同时，这些概念将使你的测试不仅仅是技术性的练习，还将帮助你让整个团队参与进来。

我当然不会让你烦恼正式的测试文档和从上到下的八个阶段方法，从测试计划，到测试设计/用例规范，再到测试总结报告。这些远远超出了本书的范围，并且无论如何，这些也超出了大多数 Dynamics 365 Business Central 实施的日常实践。然而，事先花一些时间考虑设计，将为你的工作带来杠杆效应。

本章将涵盖以下主题：

+   没有设计，就没有测试

+   测试用例设计模式

+   测试数据设计模式

+   客户需求作为测试设计

如果我已经让你信息量过大，而你想直接开始编写代码，你可以跳到下一章。在那里，我们将练习到目前为止讨论的所有内容，以及本章后面内容。然而，如果你发现自己缺少背景信息，可以回到这一章，补充了解。

# 没有设计，就没有测试

**目标：理解为什么测试应该在编写和执行之前就被构思出来。**

我想我说得并不过分，大多数我们世界中的应用测试都可以归类为探索性测试。手动测试由经验丰富的测试人员进行，他们了解被测应用，并且有很好的理解和直觉，知道如何*击破它*。但是，这些测试没有明确的设计，也没有可重复、可共享、可重用的脚本。在这个世界里，我们通常不希望开发人员测试自己的代码，因为他们无论是有意还是无意，都知道如何使用软件并规避问题。他们的思维方式是*如何让它*（工作），而不是*如何击破它*。

但是，对于自动化测试，编写代码的将是开发人员。而且，往往是同一个开发人员负责应用程序的编码。所以，他们需要一个测试设计，设计出要编写的测试。这些测试将覆盖广泛的场景，包括晴天和雨天的场景，头 less 和 UI 测试。那单元测试和功能测试呢？

依我拙见，测试设计和其他交付物一样，是团队共同拥有的成果。它是一个团队合作的结果，大家共同达成一致的测试设计。这是产品负责人、测试人员、开发人员、功能顾问和关键用户之间的协议。如果*没有设计*，就*没有测试*。测试设计是一个帮助团队讨论测试工作、揭示思维漏洞并在工作中不断完善的工具。正如接下来所讨论的，它是一种将需求转化为测试和应用代码的有效方式。

完整的测试设计将描述应该执行的各种测试，如性能测试、应用测试和安全性测试；它们执行的条件；以及评判测试成功的标准。我们的测试设计将仅涉及应用测试，因为这是本书的重点：如何创建应用测试自动化。一旦我们的测试设计包含了完整的测试用例集，这些用例需要被详细描述，而这正是下一部分的内容。

如果你想了解更多关于正式测试文档的内容，以下的维基百科文章可能是一个起点：[`en.wikipedia.org/wiki/Software_test_documentation`](https://en.wikipedia.org/wiki/Software_test_documentation)。

# 理解测试用例设计模式

**目标：学习设计测试的基本模式。**

如果你有过软件测试的经验，可能知道每个测试都有类似的整体结构。在你执行测试操作之前，例如文档的过账，首先需要对数据进行*设置*。然后，进行操作的测试将被*执行*。最后，需要对操作的结果进行*验证*。在某些情况下，还会有第四个阶段，所谓的*拆解*，用于将待测试系统恢复到其之前的状态。

测试用例设计模式的四个阶段如下所示：

+   设置

+   练习

+   验证

+   拆解

关于四阶段设计模式的简短清晰描述，请参考以下链接：

[`robots.thoughtbot.com/four-phase-test`](http://robots.thoughtbot.com/four-phase-test)

这种设计模式通常是微软在 C/SIDE 测试编码初期使用的模式。比如以下这个测试功能示例，摘自代码单元 137295 - `SCM 库存杂项 III`，你将在大量旧的测试代码单元中遇到这种模式：

```
[Test] PstdSalesInvStatisticsWithSalesPrice()
// Verify Amount on Posted Sales Invoice Statistics
//        after posting Sales Order.

// Setup: Create Sales Order, define Sales Price on Customer
Initialize();
CreateSalesOrderWithSalesPriceOnCustomer(SalesLine, WorkDate());
LibraryVariableStorage.Enqueue(SalesLine."Line Amount");
          // Enqueue for SalesInvoiceStatisticsPageHandler.

// Exercise: Post Sales Order.
DocumentNo := PostSalesDocument(SalesLine, true);
          // TRUE for Invoice.

// Verify: Verify Amount on Posted Sales Invoice Statistics.
//         Verification done in SalesInvoiceStatisticsPageHandler
PostedSalesInvoice.OpenView;
PostedSalesInvoice.Filter.SetFilter("No.", DocumentNo);
PostedSalesInvoice.Statistics.Invoke();
```

# 验收测试驱动开发

现在，微软使用**验收测试驱动开发**（**ATDD**）设计模式。这是一个更完整的结构，并且更贴近客户，因为测试是从用户的角度来描述的。该模式通过以下所谓的标签来定义：

+   `FEATURE`：定义测试或测试用例集合正在测试的特性

+   `SCENARIO`：为单个测试定义所测试的场景

+   `GIVEN`：定义需要哪些数据设置；当数据设置更复杂时，一个测试用例可以有多个`GIVEN`标签

+   `WHEN`：定义测试中的动作；每个测试用例应仅有一个`WHEN`标签

+   `THEN`：定义动作的结果，或者更具体地说，定义结果的验证；如果适用多个结果，需要多个`THEN`标签

以下测试示例，取自测试代码单元 134141 - `ERM 银行对账`，展示了基于 ATDD 设计模式的测试：

```
[Test] VerifyDimSetIDOfCustLedgerEntryAfterPostingBankAccReconLine()
// [FEATURE] [Customer]
// [SCENARIO 169462] "Dimension set ID" of Cust. Ledger Entry
//                   should be equal "Dimension Set ID" of Bank
//                   Acc. Reconciliation Line after posting
Initialize();

// [GIVEN] Posted sales invoice for a customer
CreateAndPostSalesInvoice(
  CustomerNo,CustLedgerEntryNo,StatementAmount);

// [GIVEN] Default dimension for the customer
CreateDefaultDimension(CustomerNo,DATABASE::Customer);

// [GIVEN] Bank Acc. Reconciliation Line with "Dimension Set ID" =
//         "X" and "Account No." = the customer
CreateApplyBankAccReconcilationLine(
  BankAccReconciliation,BankAccReconciliationLine,
  BankAccReconciliationLine."Account Type"::Customer,
  CustomerNo,StatementAmount,LibraryERM.CreateBankAccountNo);
DimSetID :=
  ApplyBankAccReconcilationLine(
    BankAccReconciliationLine,
    CustLedgerEntryNo,
    BankAccReconciliationLine."Account Type"::Customer,
    '');

// [WHEN] Post Bank Acc. Reconcilation Line
LibraryERM.PostBankAccReconciliation(BankAccReconciliation);

// [THEN] "Cust. Ledger Entry"."Dimension Set ID" = "X"
VerifyCustLedgerEntry(
  CustomerNo,BankAccReconciliation."Statement No.", DimSetID);
```

在进行任何测试编码之前，测试用例设计应该已经构思好。在前面的例子中，这些内容应该在编写测试代码之前就交给开发人员：

```
[FEATURE] [Customer]
[SCENARIO 169462] "Dimension set ID" of Cust. Ledger Entry
                  should be equal "Dimension Set ID" of Bank
                  Acc. Reconcilation Line after posting
[GIVEN] Posted sales invoice for a customer
[GIVEN] Default dimension for the customer
[GIVEN] Bank Acc. Reconcilation Line with "Dimension Set ID" =
        "X" and "Account No." = the customer
[WHEN] Post Bank Acc. Reconcilation Line
[THEN] "Cust. Ledger Entry"."Dimension Set ID" = "X"
```

# 测试验证说明

在我的研讨会中，主要是开发人员参与。对于他们来说，在进行测试自动化时，必须克服的一个障碍就是验证部分。对于他们来说，数据设置（`GIVEN`部分）必须考虑到，更不用说在`WHEN`部分的测试动作了。然而，`THEN`部分是一个容易被忽视的环节，尤其是当他们需要自己设计`GIVEN`-`WHEN`-`THEN`时。有些人可能会问：如果代码成功执行，我为什么还需要验证呢？

因为你需要检查：

+   创建的数据是正确的数据，也就是预期数据

+   在正负测试的情况下抛出的错误是预期错误

+   确认处理的是预期的确认

充分的验证将确保你的测试经得起时间的考验。你可能想将下一句话作为海报挂在墙上：

没有验证的测试根本不算测试！

你可能会添加一些感叹号。

你可能注意到，ATDD 设计模式没有四阶段测试设计模式中的清理阶段。正如前面提到的，ATDD 是以用户为导向的，而清理阶段更多是技术性的操作。当然，如果需要，清理部分应该在测试结束时编写。

有关 ATDD 的更多信息，可以访问以下链接：

[`en.wikipedia.org/wiki/Acceptance_test%E2%80%93driven_development`](https://en.wikipedia.org/wiki/Acceptance_test%E2%80%93driven_development) 或者 [`docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-extension-advanced-example-test#describing-your-tests`](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-extension-advanced-example-test#describing-your-tests).

# 理解测试数据设置设计模式

**目标：学习设置测试数据的基本模式。**

当你进行手动测试时，你会发现大部分时间都消耗在了设置正确的数据上。作为一个真正的 IT 专家，你会想出尽可能高效的方式来完成这一点。你可能已经考虑过确保以下几点：

+   提供了基本的数据设置，这将成为你即将执行的所有测试的基础。

+   对于每个待测试的功能，额外的测试数据预先存在

+   测试特定数据将即时创建

通过这种方式，你为自己创建了一些模式，帮助你高效地完成测试数据的设置。这些就是我们所称的**测试数据设置**设计模式，或者**夹具**或**测试夹具**设计模式，每种模式都有其特定的名称：

+   第一个是我们所称的**预构建夹具**。这是在任何测试运行之前创建的测试数据。在 Dynamics 365 Business Central 的上下文中，这将是一个准备好的数据库，比如 Microsoft 提供的演示公司`CRONUS`。

+   第二个模式被称为**共享夹具**，或懒惰设置。这涉及到由一组测试共享的数据设置。在我们的 Dynamics 365 Business Central 环境中，这涉及到通用的主数据、补充数据和设置数据，比如客户和货币数据以及四舍五入精度，所有这些数据都是运行一组测试所必需的。

+   第三个也是最后一个模式是**新鲜夹具**，或新鲜设置。这涉及到单个测试特别需要的数据，比如一个空的位置、特定的销售价格或待发布的文档。

在自动化测试中，我们将使用这些模式，原因如下：

+   **高效的测试执行**：尽管自动化测试看起来似乎是光速执行，但随着时间的推移，建立测试资料的累积将增加总执行时间，这可能轻易地达到几个小时；自动化测试的运行时间越短，它就越能被频繁使用。

+   **有效的数据设置**：在设计测试用例时，数据需求和所需的阶段一目了然；这将加速测试编码的速度。

在这里阅读更多关于夹具模式的内容：

[`xunitpatterns.com/Fixture Setup Patterns.html`](http://xunitpatterns.com/Fixture%20Setup%20Patterns.html)

请注意，测试数据设置还有很多内容需要形式化。在接下来的章节中，我们的测试编码将使用更多前面提到的模式。

# 测试夹具、数据不可知和预构建夹具

如引言章节所述，自动化测试是*可重现的*、*快速的*和*客观的*。它们在执行时是可重现的，因为代码总是相同的。但这并不保证结果是可重现的。如果每次运行测试时，测试的输入——即数据设置——不同，那么测试的输出很可能也会不同。以下三点有助于确保你的测试是可重现的：

1.  使一个测试在相同的夹具上运行

1.  使一个测试遵循相同的代码执行路径

1.  使一个测试根据相同且充分的标准集来验证结果

为了对夹具进行全面控制，强烈建议让你的自动化测试每次运行时都重新创建所需的数据。换句话说，不要依赖于测试运行前系统中已经存在的数据。自动化测试应该对被测系统中存在的任何数据保持独立。因此，在 Dynamics 365 Business Central 中运行测试不应依赖数据库中现有的数据，无论是`CRONUS`、你自己的演示数据，还是客户特定的数据。是的，你可能需要客户特定的数据来重现已报告的问题，但一旦修复，并且测试自动化得到了更新，它应该能够实现数据独立性地运行。因此，在我们的任何测试中，我们都不会使用预构建的夹具模式。

如果你曾经运行过标准测试，你可能注意到其中有相当一部分测试并非数据独立的。它们高度依赖于`CRONUS`中的数据。你也可能注意到，这种情况适用于较旧的测试。目前，标准测试力求实现数据独立性。

# 测试夹具和测试隔离

为了使每组测试都使用相同的夹具，我们将利用测试运行器代码单元的测试隔离功能，正如在第二章的*可测试性框架*部分中讨论的那样，*支柱 4 - 测试运行器与测试隔离*部分所述。通过使用标准测试运行器的测试隔离值代码单元，并将一组连贯的测试放入同一个测试代码单元中，为整个测试代码单元设置一个通用的*清理*操作。这将确保在每个测试代码单元终止时，夹具被恢复到初始状态。如果测试运行器使用*函数*级别的测试隔离，它将在每个测试函数中添加一个通用的清理操作。

# 共享夹具实现

你可能已经观察到，在作为四阶段和 ATDD 模式示例使用的两个 Microsoft 测试函数中，每个测试都在场景描述后开始调用一个名为`Initialize`的函数。`Initialize`包含了共享夹具模式的标准实现（接下来我们将详细介绍的通用新夹具模式），其实现如下：

```
local Initialize()
// Generic Fresh Setup
LibraryTestInitialize.OnTestInitialize(<codeunit id>);

<generic fresh data initialization>

// Lazy Setup
if isInitialized then
  exit();

LibraryTestInitialize.OnBeforeTestSuiteInitialize(<codeunit id>);

<shared data initialization>

isInitialized := true;
Commit();
LibraryTestInitialize.OnAfterTestSuiteInitialize(<codeunit id>);
```

当在同一个测试代码单元中的每个测试函数开始时调用`Initialize`时，懒加载设置部分只会执行一次，因为只有在第一次调用`Initialize`时，布尔变量才会是`false`。请注意，`Initialize`还包含三个钩子，即事件发布者，以便通过将订阅者函数链接到它来扩展`Initialize`：

+   `OnTestInitialize`

+   `OnBeforeTestSuiteInitialize`

+   `OnAfterTestSuiteInitialize`

在第九章，*让 Business Central 标准测试在你的代码上工作*，我们将特别利用这些发布者。

`Initialize`中的懒加载设置部分就是 xUnit 模式中所称的 SuiteFixture 设置。

# 新的夹具实现

根据先前讨论的，可以在`Initialize`函数中设置通用的新装置（部分）。这是需要在每个测试开始时创建或清理的数据。特定于一个测试的新设置，即由`GIVEN`标签定义的实施测试函数中的内联设置。

`Initialize`中的通用新设置部分是 xUnit 模式称为隐式设置的一部分。测试特定的新设置称为内联设置。

# 以客户的期望作为测试设计

**目标：了解为什么要以测试设计的形式描述需求。**

过去的发展理念是阶段性的，每个阶段都会在下一个开始之前完成。就像瀑布一样，水从一个水平流向另一个水平。从需求收集到分析，再到设计、编码、测试，最后到运行和维护，每个阶段都有其截止日期，并且文档化的可交付成果会移交给下一个阶段。这种系统的一个主要缺点是对变化洞见的响应不足，导致需求的变动。另一个问题是产生的大量文档带来的显著开销。在最近的十年或两十年里，敏捷方法已经成为应对这些缺点的一般实践。

引入测试设计，作为你开发实践的额外文档，也许不是你一直在等待的，尽管我可以保证，你的开发实践会得到提升。但如果你的测试设计可以成为一种统一的文档呢？成为项目中每个学科的输入？每个层次共享同一真相？*如果你可以一石五鸟*？如果你可以以一种格式编写你的需求，作为所有实施任务的输入？

将需求定义为用户故事或用例是一种常见做法。但我个人认为它们的一个主要缺失是它们往往只定义了晴天路径，没有明确描述阴雨天的情景。在非典型输入下，你的功能应该如何表现？它应该如何报错？正如前面提到的，这是测试人员思维与开发者思维的分歧之处：如何使其出错而不是如何使其工作。这绝对是测试设计的一部分。那么，为什么不将测试设计提升为需求呢？或者从内而外：使用 ATDD 模式编写你的需求，就像编写测试设计一样。

这就是我们现在在我的主要雇主那里尝试的。这就是我在我的研讨会中提倡的，以及实施合作伙伴正在接受的。将每个愿望和每个功能拆分成像下面这样的测试列表，并将其作为我们的主要沟通工具：

1.  详细说明你的**客户期望**

1.  实施你的**应用程序代码**

1.  结构化执行你的**手动测试**

1.  编写你的**测试自动化**

1.  更新你解决方案的**文档**

通过这样做，你的测试自动化将是前期工作的逻辑结果。新的洞见和需求更新将反映在这个列表中，并相应地体现在你的测试自动化中。虽然你当前的需求文档可能未必总是与实现的最新版本保持同步，但当你将测试设计推向需求时，它们会同步更新，因为你的自动化测试必须反映最新版本的应用程序代码。这样一来，你的测试自动化就是你最新的文档。真是一举多得。

正如我们在接下来的章节中将要做的那样，我们将把需求指定为测试设计，最初在功能和场景级别，使用`FEATURE`和`SCENARIO`标签。接着，将使用`GIVEN`、`WHEN`和`THEN`标签进行详细的规范。在接下来的例子中，先来看看它是如何展示的，这是一个关于我们将在下一个章节中处理的`LookupValue`扩展的场景：

```
[FEATURE] LookupValue UT Sales Document
[SCENARIO #0006] Assign lookup value on sales quote document page
[GIVEN] A lookup value
[GIVEN] A sales quote document page
[WHEN] Set lookup value on sales quote document
[THEN] Sales quote has lookup value code field populate
```

完整的 ATDD 测试设计作为 Excel 表格`LookupValue`存储在 GitHub 上。

# 总结

测试自动化将从结构化的方法中获益，为此，我们引入了一套概念和测试模式，如 ATDD 模式和测试夹具模式。

现在，在下一章节，第五章，*从客户需求到测试自动化——基础篇*，我们将利用这些模式，最终实现测试代码。
