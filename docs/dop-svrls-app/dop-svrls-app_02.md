# 了解无服务器框架

在上一章中，我们探讨了无服务器计算的世界，它是如何工作的，目的是什么，采用它的好处，各种服务提供商以及它们在提供服务方面的表现。我们还了解了采用无服务器架构的优缺点。本章的目标是教会我们不同的无服务器部署框架，以及它们最终如何帮助我们实现持续集成和持续交付。此外，我们还将了解框架提供的各种功能，并更详细地讨论无服务器框架，了解它在幕后做了什么。

在应用开发的世界里，开发应用的过程通常是相同的。开发者在本地机器上开发代码，然后将更改编译并推送到源代码管理仓库。测试人员随后测试并发布报告，而运维团队则负责将代码部署到不同的环境中并管理基础设施。

但有可能相同的代码在生产环境中会失败。为了让这段代码重新工作，开发者、测试人员和运维团队必须加班才能使生产环境恢复正常。在根本原因分析中，开发者会说他的代码在个人电脑上运行正常，测试人员会声称她已经测试了所有内容并提供支持这一事实的报告，而运维人员则会说他的工作只是部署代码。因此，我们面临的挑战如下：

+   确保每次部署时代码都能在生产环境中完美运行

+   加速部署周期

+   让团队一起合作，承担各自的责任

+   实现无摩擦的生产部署

解决这些问题的方法是采用 DevOps，自动化流程和团队协作。

"DevOps 是一套自动化软件开发和 IT 团队之间流程的实践，以便他们能够更快、更可靠地构建、测试和发布软件。"

- 在 Atlassian 上的 DevOps 定义

DevOps 依赖于工具、人员、流程和反馈循环的结合。但工具和流程是 DevOps 的前轮，在非生产和生产环境中推动更快的发布周期、持续集成、持续测试和持续部署方面发挥着至关重要的作用。

使用无服务器架构来实现 DevOps 要容易得多，因为我们不必担心底层基础设施。然而，我们仍然需要持续集成、监控、日志记录以及持续部署，以便代码顺利推向生产环境。由于无服务器架构仍处于初期阶段，现有的一些新开发工具和框架数量有限，但这些数量最终会增长。我们将关注更流行的工具或框架，最后集中讨论一个框架，深入了解其功能。

本书中我们将要探讨的所有工具都是开源框架。它们各自根据需求提供特定功能。我们将考虑四个流行的无服务器框架，并了解它们的特点。

# ClaudiaJS

**ClaudiaJS**是最早的部署框架和工具之一。它是开源许可证下的，且在撰写本文时，仅支持 AWS Lambda。ClaudiaJS 是一个 Node.js 库，帮助将 Node.js 项目部署到 AWS Lambda 和 API Gateway。它目前仅支持 Node.js 语言。ClaudiaJS 声明它不是一个框架，而是一个部署工具，因此开发者只需在代码中调用 ClaudiaJS，而无需改变代码结构。ClaudiaJS 建立在 AWS SDK 和 AWS CLI 之上。它标识了三种类型的 JavaScript 库：

+   命令行库

+   API 构建库

+   Bot 构建库

# 命令行库

第一个 JavaScript 库是一个命令行工具或库。该命令行工具帮助部署、更新、回滚、打包、调用或测试以及销毁 Lambda 函数，它还与 AWS API Gateway 无缝集成。它使用标准的 npm 打包惯例，这意味着你可以在不修改代码结构的情况下直接调用它。所以，ClaudiaJS 的命令行库提供的有趣功能如下：

+   `claudia create `: 这个命令将在 AWS 门户上创建一个函数和相关的安全角色。

+   `claudia update `: 这个命令将通过部署新版本的函数来更新功能，并更新相关的 API。

+   `claudia test-lambda`: 这个命令将执行 Lambda 函数。

+   `claudia set-version`: 这个命令将把 Lambda API 阶段指向最新的部署版本。

+   `claudia add-scheduled-event`: 这个命令可以为 Lambda 函数添加定期执行的计划事件，通过这个命令，我们可以保持 Lambda 函数处于“热”状态。

+   `claudia destroy`: 这个命令将销毁函数及其相关的 API 和安全角色。

# API 构建库

第二种类型的库是 API 构建库。它是 ClaudiaJS 库的扩展，帮助设置 AWS API Gateway 端点。它还帮助将多个 API 网关路由到单一的 Lambda 函数，并自动为这些端点启用 CORS。

# Bot 构建库

这个库是 ClaudiaJS 提供的最有趣的库之一。这个库可以帮助在几分钟内创建不同类型的机器人。它提供开箱即用的功能，能够与 Facebook Messenger、Telegram 和 Skype 集成。使用 ClaudiaJS 的机器人库，设置一个机器人相当简单。

总结来说，ClaudiaJS 在使用 AWS 云服务提供商时非常好用。它也支持 Node.js。以下信息框中提供的文档链接是最新的，文档详细解释了每一个 CLI 命令。提供了很多教程，涵盖从简单开发到高级任务。机器人库是 ClaudiaJS 提供的最棒的功能之一。然而，它不支持多个无服务器提供商，也不支持多语言。

更多关于 Claudia.JS 的信息可以在以下链接找到：

[`claudiajs.com`](https://claudiajs.com)

# Apex

**Apex**是另一个基于 Go 构建的无服务器框架，用于管理 AWS Lambda 函数。它是一个开源框架，使用 Terraform 来启动资源，使得执行速度更快。该框架提供的功能包括部署、测试函数、回滚部署、查看指标和跟踪日志。

尽管它不支持本地调用函数，但它支持多种语言，如 Node.js、Python、Java、Rust 和 Go。我们可以通过 Apex 创建各种环境。它有很好的文档，帮助你快速上手使用该框架。然而，Apex 目前只支持 AWS Lambda。

更多关于 Apex 的信息可以在以下链接找到：

[`apex.run/#function-hooks`](http://apex.run/#function-hooks)

# Zappa

如果你决定用 Python 编写函数，那么你可以使用 Zappa 来部署它们。**Zappa**是一个 CLI/命令行框架，并且是开源的。Zappa 目前支持 Python WSGI 应用，基本上是 Flask 和 Django 应用。它可以部署宏应用和微应用。Zappa 拥有各种功能，如能够将 API 等函数部署到 AWS Lambda 和 AWS API Gateway。它还可以配置 AWS 事件源。

一旦部署，我们还可以通过 Zappa 调用函数。它可以从 AWS 获取或跟踪日志。它还支持回滚到之前的版本。我们可以设置多阶段部署（通过**stage**，指的是多个环境部署，如`dev`、`qa`、`uat`和`prod`）。

Zappa 还具有一个很酷的功能，可以保持 Lambda 函数常驻。这可以提高性能并在一定程度上减少延迟。它允许我们安排部署，这意味着我们可以在一天的早些时候设置部署，以免干扰常规流量。它还具备撤销部署和从 CloudWatch 清除日志的能力。我们还可以使用它打包 Lambda 函数以供未来部署。部署后的状态也可以通过 Zappa 进行检查。Zappa 允许我们将 Lambda 函数部署到 AWS 的任何区域。我们来看看 Zappa 的一些功能：

+   **安装 Zappa：** 要安装 Zappa，你需要确保你本地的 Python 版本为 2.7 或更低，并且已经安装和配置了 PIP。你还需要确保你的 AWS 凭证已设置好（[`aws.amazon.com/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/`](https://aws.amazon.com/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/)）。我们需要 Python 和 pip 来在本地环境中安装 Zappa：

```
$ pip install zappa 
```

+   **Zappa init：** `init` 命令将设置部署配置。它应该会自动检测到 Flask/Django 应用，并在项目目录中创建一个名为 `zappa_settings.json` 的 JSON 文件：

```
$ zappa init 
```

+   **打包和部署：** 一旦设置好配置，我们可以使用以下命令打包和部署应用程序。默认情况下，它使用生产环境，但我们可以创建多个不同的环境：

```
 $ zappa deploy production
```

Zappa 是一个很棒的框架，但它也有一些缺点，比如不支持其他云服务提供商，如 Azure、Google 和 OpenWhisk。它仅支持基于 Python-WSGI 的应用程序，不支持其他语言，如 Node.js。

你可以在以下链接找到更多关于 Zappa 框架的信息：

[`www.zappa.io/`](https://www.zappa.io/)

# Serverless Framework

**Serverless Framework** 是构建无服务器架构最受欢迎的框架之一。它是一个开源命令行工具，在 GitHub 上大约有 23,000 个 stars。它还有一个企业版，帮助设置模板并提供支持。许多公司，如 EA、可口可乐、Expedia 和路透社，都在使用这个框架。它支持很多云服务提供商，如 AWS、Azure、Google、OpenWhisk、Kubeless、Oracle Fn 等。它有一个文档非常完善的用户指南，包含大量示例，帮助你快速上手。它支持多种语言，如 Node.js、Python、Java、Scala、C#、Go、F#、Groovy、Kotlin、PHP 和 Swift。

它支持无服务器架构的生命周期，可以进行构建、部署、更新和删除。它支持功能分组，便于在大型项目中管理代码、流程和资源，并且对 CD/CI 提供了相当好的支持。与其他框架相比，它拥有更强大的社区支持。它提供了大量插件来支持框架功能，并且有很多博客可以帮助我们建立使用框架的最佳实践。它还提供了支持论坛和 Slack 聊天室来解决问题。它支持很多功能，如部署函数和事件、调用函数、跟踪日志、集成测试以及为将来部署进行打包。让我们更详细地了解一下 Serverless Framework 的功能。

# 框架功能

Serverless Framework 中有许多功能，尽管它们会因云提供商而异。我将列出并描述一些更重要和更常见的功能。

# 服务和部署

服务是我们定义函数、触发这些函数的事件，以及为函数执行所需的任何基础设施资源的项目。它们被收集到一个文件中，这个文件叫做`serverless.yml`：

```
eg. 
 myServerlessService/
        serverless.yml
```

当我们开始使用 Serverless Framework 进行部署时，我们将使用一个单一的服务。但随着应用的增长，建议您使用多个服务，如以下代码所示：

```
 users/
        serverless.yml # Contains 4 functions  
 posts/
         serverless.yml # Contains 4 functions
```

拥有多个服务可以隔离要使用的基础设施资源。但它也有一个缺点，即目前每个服务都会在 API Gateway 上创建一个独立的 REST API。这是 API Gateway 的一个限制。但有一个解决方法，我们将在未来的章节中讨论。

要创建服务，我们必须使用`create`命令，并且必须传递您希望编写服务的运行时语言。我们还可以提供路径，如下例所示：

```
$ serverless create --template <runtimes> --path myService
```

Serverless Framework 的主要目的是将函数、事件和基础设施资源部署到远程云端，且无需太多麻烦，这是通过`deploy`插件完成的。这个`deploy`插件提供了各种功能。让我们来看其中的一些：

+   **部署到不同的阶段和区域：**

    `$ serverless deploy --stage production --region us-east-1`

+   **从服务中部署单个函数：**

    `$ serverless deploy function <function_name>`

+   **部署包到云端：**

    `$ serverless deploy --package <path to package>`

这个`deploy`插件的工作方式如下：

+   框架将目标 AWS Lambda 函数打包成`.zip`文件

+   框架获取已上传的函数`.zip`文件的哈希值，并将其与本地`.zip`文件的哈希值进行比较

+   如果两个哈希值相同，框架将终止

+   该`.zip`文件使用与之前函数相同的名称上传到您的 S3 存储桶，这就是它指向的 CloudFormation 堆栈

# 函数和事件

函数是定义在服务中的属性，它们在 `serverless.yml` 文件中定义，因此我们为函数命名并提供 handler 属性，指向函数文件，该文件可以是 Node.js 或 Python。我们可以在该属性中添加多个函数。函数可以继承提供者的属性，也可以在函数级别定义属性。这些函数属性会根据云提供商的不同而有所变化，如以下代码所示：

```
# serverless.yml
service: myService
provider:
  name: aws
  runtime: nodejs6.10
  memorySize: 512 # will be inherited by all functions
functions:
  usersAdd:
    handler: handler.userAdd
    description: optional description for your function
userModify:
    handler: handler.userModify
userDelete:
    handler: handler.userDelete
    memorySize: 256 # function specific
```

如果为每个函数创建一个单独的文件，可以将函数列为数组：

```
 # serverless.yml

 functions: 
     - ${file(../user-functions.yml)}
   - ${file(../post-functions.yml)}
 # user-functions.yml 
addUser: 
     handler: handler.user 
deleteUser: 
      handler: handler.user
```

可以在服务中的函数内添加环境对象属性，且它应该是键值对。同时，函数特定的环境变量会覆盖提供者特定的环境变量：

```
 # serverless.yml 
service: service-name 
provider: aws 

 functions: 
   hello: 
       handler: handler.hello 
       environment: 
           TABLE_NAME: tableName
```

事件是触发函数的操作，例如 S3 存储桶上传。Serverless 框架支持多种事件，但它们根据云提供商不同而有所变化。我们可以为单个函数定义多个事件，如以下代码所示：

```
events:
 - http:
    path: handler
    method: get
```

Serverless 框架为 AWS Lambda 提供的事件类型如下链接所示：

[`serverless.com/framework/docs/providers/aws/events/`](https://serverless.com/framework/docs/providers/aws/events/)

# 变量与插件

**变量**是在运行 Serverless 框架命令时，可以传递给 `serverless.yml` 配置值的值。它们需要通过 `${}` 括号传递引用值，但你可以在属性值中使用变量，而不是在属性键中。以下代码展示了如何在 `serverless.yml` 文件中添加这些变量：

```
provider:  
    name: aws 
     stage: ${opt:stage, 'dev'}
```

在以下语句中，我们将参数传递给 CLI，`stage` 值将被填充到 `serverless.yml` 文件中：

```
$ serverless deploy --stage qa
```

变量可以作为引用属性递归使用——也就是说，我们可以将多个值和变量来源结合起来，如以下环境变量所示：

```
 environment: SERV_SECRET: ${file(../config.${self:provider.stage}.json):CREDS} 
```

我们也可以引用环境变量（如以下代码所示），但是将敏感数据添加到环境变量中是不安全的，因为它们可能会通过构建日志或 Serverless CloudFormation 模板被访问：

```
functions: 
    myfunction: 
        name: ${env:FUNC_PREFIX}-myfunction 
        handler: handler.myfunction
```

我们来看看几个实现这一点的教程。确保你已安装并正常运行最新版本的 Serverless 框架：

1.  使用 Serverless AWS 模板创建一个简单的 `hello` 项目，然后在你喜欢的编辑器中打开该项目：

```
 $ serverless create --template aws-nodejs --path env-variable-service 
```

1.  替换 `serverless.yml` 文件和 handler，如以下代码所示。在这里，我们通过 `MY_VAR` 的环境变量名添加一个环境变量，并在 handler 中显示该环境变量的值：

```
  $ cat serverless.yml
service: env-variable-service
# You can pin your service to only deploy with a specific Serverless version
 # Check out our docs for more details
 # frameworkVersion: "=X.X.X"
provider:
   name: aws
   runtime: nodejs6.10
# you can define service wide environment variables here
   environment:
     MY_VAR: Lion
  functions:
     hello:
       handler: handler.hello
 $ less handler.js
'use strict';
module.exports.hello = (event, context, callback) => {
const response = {
  statusCode: 200,
  body: JSON.stringify({
    message: `my favourite animal is ${process.env.MY_VAR}`,
    input: event,
  }),
};
callback(null, response);
};
```

1.  让我们在本地调用并查看结果。如果你查看消息部分，你可以看到我们定义的环境变量的值，如以下代码所示：

```
 $ serverless invoke local -f hello
 {
 "statusCode": 200,
 "body": "{\"message\":\"my favourite animal is Lion\",\"input\":\"\"}"
 }

```

正如我们之前讨论的，对于非敏感数据，将变量添加到`serverless.yml`文件应该没问题，但我们如何将敏感数据（如数据库连接）添加到环境变量中呢？让我们看看实现这一目标所需的步骤：

1.  在`env-variable-service`文件夹中创建一个新文件，命名为**`serverless.env.yml`**。然后按照以下代码的示例添加相应的内容。在这里，我们根据环境创建一个密钥变量：

```
 $ less serverless.env.yml
 dev:
   MYSECRET_VAR: 'It is at secret den'

```

1.  让我们在`serverless.yml`文件中再添加一个环境变量，但这次它的值将从文件中获取，因此你需要添加高亮显示的那一行作为环境变量。这样，Serverless Framework 将读取文件并将其引用到特定的环境中：

```
# serverless.yml
 # you can define service wide environment variables here
environment:
   MY_VAR: Lion
   MYSECRET_VAR: ${file(./serverless.env.yml):dev.MYSECRET_VAR}
```

1.  让我们更改处理程序的响应，通过消息显示密钥。理想情况下，我们应该在屏幕上显示密钥，但为了本教程，我将手动进行。所以让我们将消息体替换为以下代码中显示的内容：

```
 # handler.js
message: `my favourite animal is ${process.env.MY_VAR} and ${process.env.MYSECRET_VAR}`,
```

1.  本地调用函数并查看输出：

```
 $ serverless invoke local -f hello
 {
 "statusCode": 200,
 "body": "{\"message\":\"my favourite animal is Lion and It is at secret den\",\"input\":\"\"}"
 }
```

最后，正如我之前所说，我们可以为每个部署阶段设置多个环境变量，如`dev`、`sit`、`uat`和`prod`。以下步骤展示了如何添加这些环境变量：

1.  让我们为`prod`添加一个环境变量到`serverless.env.yml`文件中。然后，我们可以在`serverless.yml`文件中动态使用它们，如下所示：

```
 $ less serverless.env.yml
dev:
   MYSECRET_VAR: 'It is at secret dev den'
prod:
   MYSECRET_VAR: 'It is at secret prod den'
```

1.  现在，我们需要对`serverless.yml`文件进行修改，动态地根据调用或部署函数时设置的阶段来获取环境变量，这就是将`MYSECRET_VAR`行替换为以下行：

```
 #serverless.yml
MYSECRET_VAR: ${file(./serverless.env.yml):${opt:stage}.MYSECRET_VAR}
```

1.  现在我们将本地调用该函数，并查看不同阶段的输出：

```
 $ serverless invoke local -f hello -s dev
 {
 "statusCode": 200,
 "body": "{\"message\":\"my favourite animal is Lion and It is at secret dev den\",\"input\":\"\"}"
 }

 $ serverless invoke local -f hello -s prod
 {
 "statusCode": 200,
 "body": "{\"message\":\"my favourite animal is Lion and It is at secret Prod den\",\"input\":\"\"}"
 }
```

我已经将前面的教程上传到以下 GitHub 仓库，你可以随时使用：

[`github.com/shzshi/env-variable-service.git`](https://github.com/shzshi/env-variable-service.git)

插件是提供扩展现有 Serverless Framework CLI 命令的自定义 JavaScript 代码。框架本身就是一组核心提供的插件。我们可以构建自己的自定义插件，Serverless Framework 也提供了该插件的文档。可以使用以下代码安装插件：

```
$ npm install --save custom-serverless-plugin
```

我们还需要在`serverless`服务中调用它们，使用以下代码：

```
serverless.yml file 
 plugins:
     - custom-serverless-plugin
```

可以通过以下链接查看现有的 Serverless 插件列表：

[`github.com/serverless/plugins`](https://github.com/serverless/plugins)

# 资源

当我们创建 Lambda 函数时，它们可能依赖于多种不同类型的基础设施资源，如 AWS、DynamoDB 或 AWS S3，因此我们可以在 `serverless.yml` 文件中定义这些资源并进行部署。当我们添加这些资源时，它们将被添加到 `serverless.yml` 文件中，并且在部署时，它们会被添加到 CloudFormation 堆栈中并在 `serverless deploy` 时执行。我们可以通过以下示例查看这些资源是如何定义的：

```
resources:
  Resources:
    NewResource:
       Type: AWS::S3::Bucket
       Properties:
         BucketName: my-s3-bucket
```

您可以参考以下链接，了解更多资源详情，并查看仅使用 AWS Lambda 时可用的资源：[`serverless.com/framework/docs/providers/aws/guide/resources/`](https://serverless.com/framework/docs/providers/aws/guide/resources/)

让我们看一个使用 AWS Lambda 的 Serverless Framework 简单示例。

您将需要以下先决条件：

+   需要创建一个免费的 AWS 账户。

+   本地机器上必须安装 Node.js 4.0 或更高版本。

+   AWS CLI 也可以安装，但这是可选的。

# 设置 AWS 访问密钥。

请按照以下步骤操作：

1.  登录到 AWS 账户并进入 IAM（身份与访问管理）页面。

1.  点击左侧栏的用户（Users），然后点击“添加用户”按钮并添加用户名 `adm-serverless`。接着启用编程访问（programmatic access），勾选复选框。然后点击下一步：权限按钮（Next:Permissions）。

1.  在此页面上，选择直接附加现有策略，搜索并选择 AdministratorAccess 复选框，然后点击下一步：审核。

1.  现在检查一切是否正常，然后点击“创建用户”。这将创建一个用户并显示我们的访问密钥 ID 和秘密访问密钥。暂时复制并存储这些密钥。

1.  现在我们已经获得了密钥，我们将其作为环境变量导出，以便框架能够访问并执行其所需的功能：

```
$ export AWS_ACCESS_KEY_ID=<access-key-id>
$ export AWS_SECRET_ACCESS_KEY=<secret-key-id>
```

# 安装 Serverless Framework。

请按照以下步骤操作：

1.  从 [`nodejs.org/en/download/`](https://nodejs.org/en/download/) 安装 Node.js 4.0 或更高版本。安装完成后，我们可以通过以下命令验证安装：

```
 $ node --version
```

1.  现在我们需要通过以下命令全局安装 Serverless Framework：

```
 $ npm install -g serverless
```

1.  安装成功后，我们可以通过以下命令验证安装。它将显示所有框架命令和文档：

```
$ serverless
```

1.  我们还可以通过以下命令查看已安装的 Serverless Framework 版本：

```
$ serverless --version
```

# Lambda 服务和函数部署。

在接下来的步骤中，我们将创建一个简单的 Node.js 服务和 Lambda 函数，然后部署和调用它们：

1.  使用 Serverless Framework 创建一个新的服务，选择 Node.js 模板。我们需要确保名称唯一，并可以选择性地添加服务路径。此命令将创建两个文件—`handler.js` 和 `serverless.yml`：

```
$ serverless create --template aws-nodejs --path my-serverless-service
Serverless: Generating boilerplate...
 Serverless: Generating boilerplate in "/Users/shashi/Documents/packt/chapter2/serverless/my-serverless-service"
 _______ __
 | _ .-----.----.--.--.-----.----| .-----.-----.-----.
 | |___| -__| _| | | -__| _| | -__|__ --|__ --|
 |____ |_____|__| \___/|_____|__| |__|_____|_____|_____|
 | | | The Serverless Application Framework
 | | serverless.com, v1.26.1
 -------'
Serverless: Successfully generated boilerplate for template: "aws-nodejs"
```

服务是框架的组织单元。它可以被视为一个项目文件。在这里，我们可以定义函数、触发这些函数的事件以及我们将使用的函数资源。这些内容都被放入一个文件`serverless.yml`中。在这个`.yaml`文件中，我们定义了名为`my-serverless-service`的服务。然后我们定义了提供者；正如我之前提到的，Serverless Framework 支持许多其他云服务提供商。我们可以在这个标签中列出提供者的详细信息，并且还可以提到运行时环境，在我们的案例中是 Node.js。运行时环境会根据我们编写函数所使用的语言而变化。我们可以定义部署环境或阶段—在我们的案例中是`dev`—以及相应的区域。接着，在`functions`部分，我们定义了函数名称—在我们的案例中是`hello`—它有一个名为`handler`的属性，这个处理程序将调用`handler.js`文件。我们还可以定义内存大小。接下来，我添加了一个 HTTP 事件，它除了配置 Lambda 函数外，还配置了 AWS API Gateway。它将为处理程序创建并提供一个端点。所以，使用一个脚本，我们可以配置 Lambda 函数和 API 端点。我们可以定义其他各种属性和参数；我们将在下一章中详细研究这些内容。

请确保`.yaml`文件正确缩进，否则它们将无法运行。你也可以使用我在以下 GitHub 链接中提供的文件：

[`github.com/shzshi/my-serverless-service.git`](https://github.com/shzshi/my-serverless-service.git)

```
$ cat serverless.yml 
# Welcome to Serverless!
#
# This file is the main config file for your service.
# It's very minimal at this point and uses default values.
# You can always add more config options for more control.
# We've included some commented out config examples here.
# Just uncomment any of them to get that config option.
#
# For full config options, check the docs:
# docs.serverless.com 
#
# Happy Coding!
service: my-serverless-service
# You can pin your service to only deploy with a specific Serverless version
# Check out our docs for more details
# frameworkVersion: "=X.X.X"
provider:
name: aws
runtime: nodejs6.10
stage: dev
region: us-east-1
functions:
hello:
handler: handler.hello
memorySize: 128
events:
- http:
path: handler
method: get
```

`handler.js`代码段是`Hello, World!`的 Node.js Lambda 函数，在`serverless.yml`文件中引用。它是一个相当简单的函数，执行时将显示消息`My Serverless World`，如下所示：

```
$ cat handler.js
'use strict';
module.exports.hello = (event, context, callback) => {
const response = {
statusCode: 200,
body: JSON.stringify({
message: 'My Serverless World',
input: event,
}),
};
callback(null, response);
// Use this code if you don't use the http event with the LAMBDA-PROXY integration
// callback(null, { message: 'Go Serverless v1.0! Your function executed successfully!', event });
};
```

# 本地调用

每次将代码推送到 AWS Lambda 并进行测试既昂贵又耗时。所以，通过 Serverless Framework，我们可以在本地调用或测试函数，然后将其部署到云端。我们可以将此作为持续部署管道的一部分，在这里我们可以为`dev`阶段部署设置本地调用，设置自动化测试，然后将它们进一步推进管道，进行远程部署和测试。以下命令用于本地调用函数：

```
$ serverless invoke local --function hello
 {
 "statusCode": 200,
 "body": "{\"message\":\"My Serverless World\",\"input\":\"\"}"
 }
```

# 本地部署和调用

既然我们现在能够成功地在本地调用和测试函数，接下来我们应该可以将其部署并进行远程测试。首先，我们需要确保已将访问密钥和秘密访问密钥作为环境变量获取并导出，代码如下所示：

```
$ export AWS_ACCESS_KEY_ID=<access-key-id>
$ export AWS_SECRET_ACCESS_KEY=<secret-key-id> 
```

现在我通过一个简单的 `deploy` 命令将功能和 API 部署到 AWS 云中。在后台，部署过程将创建一个 `.serverless` 文件夹，包含 `CloudFormation` JSON 模板。无服务器代码被打包成 `.zip` 文件以及一个无服务器状态 JSON 文件。如果我们查看 `create-stack` JSON 模板，Serverless Framework 将在 AWS 云上创建一个 S3 存储桶，并将功能和 API 包部署到该存储桶中，使用 `CloudFormation` 模板 JSON 文件。它还会以 JSON 文件的形式保持部署状态。成功部署后，将提供一个 API 端点，该端点与 AWS Lambda 功能绑定，并创建一个如提供商中提到的服务：

```
$ serverless deploy

 Serverless: Packaging service...
 Serverless: Excluding development dependencies...
 Serverless: Creating Stack...
 Serverless: Checking Stack create progress...
 .....
 Serverless: Stack create finished...
 Serverless: Uploading CloudFormation file to S3...
 Serverless: Uploading artifacts...
 Serverless: Uploading service .zip file to S3 (417 B)...
 Serverless: Validating template...
 Serverless: Updating Stack...
 Serverless: Checking Stack update progress...
 ..............................
 Serverless: Stack update finished...
 Service Information
 service: my-serverless-service
 stage: dev
 region: us-east-1
 stack: my-serverless-service-dev
 api keys:
 None
 endpoints:
 GET - https://kn6esoolgi.execute-api.us-east-1.amazonaws.com/dev/handler
 functions:
 hello: my-serverless-service-dev-hello
```

部署后，我们将进入 AWS 门户，查看该功能是否已部署。然后我们将通过门户调用它，并再次通过无服务器 CLI 调用远程功能，使用以下步骤：

1.  登录到 AWS 门户，然后选择已部署功能和 API 的正确区域。

1.  选择服务为 Lambda。Lambda 门户页面将显示已部署的功能名称。使用单选按钮选择你要测试的功能，然后转到标记为“操作”的下拉菜单并选择“测试”选项。窗口将弹出配置事件。你可以添加自己的事件或保持默认设置，然后点击“保存”。事件将被保存，页面将重定向到功能页面。现在点击“测试”按钮。功能将被执行，执行状态和结果将显示出来。

如前所述，我们也可以在本地调用远程功能。现在我们将查看远程 Lambda 功能的各种命令：

要调用该功能并获取日志，请使用以下命令：

```
 $ serverless invoke -f hello -l
 {
 "statusCode": 200,
 "body": "{\"message\":\"My Serverless World\",\"input\":{}}"
 }
 --------------------------------------------------------------------
 START RequestId: c4bdc7f8-60e1-11e8-9846-f5267af11144 Version: $LATEST
 END RequestId: c4bdc7f8-60e1-11e8-9846-f5267af11144
 REPORT RequestId: c4bdc7f8-60e1-11e8-9846-f5267af11144 Duration: 44.37 ms Billed Duration: 100 ms Memory Size: 128 MB Max Memory Used: 20 MB
```

若只需获取上次调用的日志，请输入以下代码。我们可以在一个单独的控制台中测试其功能：

```
$ serverless logs -f hello -t
START RequestId: 75139847-60dc-11e8-8c0b-4f4996ceffb3 Version: $LATEST
 END RequestId: 75139847-60dc-11e8-8c0b-4f4996ceffb3
 REPORT RequestId: 75139847-60dc-11e8-8c0b-4f4996ceffb3 Duration: 9.04 ms Billed Duration: 100 ms Memory Size: 128 MB Max Memory Used: 20 MB
START RequestId: 675574ec-60de-11e8-bbd5-8bf1c1e7f05c Version: $LATEST
 END RequestId: 675574ec-60de-11e8-bbd5-8bf1c1e7f05c
 REPORT RequestId: 675574ec-60de-11e8-bbd5-8bf1c1e7f05c Duration: 18.15 ms Billed Duration: 100 ms Memory Size: 128 MB Max Memory Used: 20 MB
```

该命令将从 Lambda 中撤销功能的部署，并移除 S3 存储桶中的包详情。一旦`serverless remove`成功运行（如以下代码所示），你可以登录 AWS 门户，检查 S3 存储桶和 Lambda 功能的具体区域——它应该已经被移除：

```
$ serverless remove
Serverless: Getting all objects in S3 bucket...
 Serverless: Removing objects in S3 bucket...
 Serverless: Removing Stack...
 Serverless: Checking Stack removal progress...
 .........
 Serverless: Stack removal finished...

```

# 总结

在本章中，我们学习了不同的框架，例如 ClaudiaJS、Zappa 和 Apex。我们还查看了一些使用它们的示例，但我们主要深入探讨了 Serverless Framework。在本书中的大部分教程中，我们将广泛使用 Serverless Framework，因为它比其他框架表现得更好，因为它具有更好的社区支持和对多个云提供商的支持。这意味着你不会像使用某些其他框架时那样被绑定到特定供应商。它还拥有大量的插件，支持不同类型的云服务，良好的博客支持，最后，还有一些非常适合轻松部署到不同云提供商的优良功能。
