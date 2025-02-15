---
title: AWS-Lambda开发实战
date: 2022-07-16 20:38:44
comments: true
tags:
- AWS Lambda
- AWS CDK
- Python
- Node.js
- Visual Studio Code
- 开发环境配置
- Lambda函数编写
- 项目部署
- 调试测试
- Serverless应用
categories:
- 软件开发
- 云计算
---


AWS中的Lambda函数是一种无服务器计算服务，它允许开发者在无需管理服务器的情况下运行代码。Lambda函数可以响应事件、处理数据、执行计算任务等，对标腾讯云的SCF和阿里云的FaaS，都是一种云函数，其运行环境依赖于云服务器，提供自动的负载均衡、弹性扩展和容错能力。

本文通过Python和nodejs两个环境下对Lambda函数的开发与集成，介绍了基于Python环境的AWS CDK开发与Lambda函数集成的详细步骤。


## 基于Python环境的AWS CDK开发与Lambda函数集成

在 Visual Studio Code 中基于 AWS CDK 和 Lambda 并使用 Python 环境进行开发的详细步骤：

### 1. 安装必要软件与工具
- **安装 Visual Studio Code（VS Code）**：如果尚未安装，前往[VS Code 官方网站](https://code.visualstudio.com/)下载对应操作系统的安装包进行安装。

- **安装 Python**：确保你的系统中已经安装了 Python 3.x 版本。你可以从[Python 官方网站](https://www.python.org/downloads/)下载安装包并进行安装。安装完成后，可以通过在命令行中输入 `python --version` 来确认安装的版本。

- **安装 AWS CLI（命令行界面）**：AWS CLI 用于与 AWS 服务进行交互和配置。前往[AWS CLI 官方页面](https://aws.amazon.com/cli/)，按照指导文档进行安装。安装后，需要使用 `aws configure` 命令配置你的 AWS 访问密钥（从 AWS 管理控制台获取）、秘密访问密钥、默认区域以及默认输出格式等信息，以便能够与 AWS 服务通信。

- **安装 AWS CDK**：在命令行中，使用 Python 的包管理器 `pip` 来安装 AWS CDK，运行以下命令：
```bash
pip install aws-cdk.core
```
这将安装 AWS CDK 的核心模块。根据实际需求，你可能还需要安装其他特定语言相关的 CDK 模块，比如针对 Python 开发的 `aws-cdk.aws-lambda` 等，命令示例如下：
```bash
pip install aws-cdk.aws-lambda
```

### 2. 在 VS Code 中配置开发环境
- **安装 VS Code 扩展**：
    - **Python 扩展**：在 VS Code 的扩展面板（可通过左侧边栏的方块图标打开）中搜索“Python”，安装由微软官方提供的 Python 扩展，它能提供代码智能提示、语法检查、调试支持等功能，方便 Python 代码开发。
    - **AWS Toolkit for Visual Studio Code**：同样在扩展面板搜索并安装此扩展，它能够帮助你更便捷地在 VS Code 中与 AWS 服务交互，例如直接在 VS Code 中部署 CDK 应用、管理 Lambda 函数等。

- **创建项目文件夹并初始化 CDK 项目**：
在本地文件系统中创建一个新的文件夹作为项目目录，例如命名为 `my-cdk-project`。然后在命令行中切换到该文件夹目录下，运行以下命令初始化 CDK 项目：
```bash
cdk init app --language python
```
这将创建一个基于 Python 语言的基本 CDK 项目结构，包含了必要的文件和文件夹，比如 `app.py`（主应用文件，用于定义 CDK 栈等内容）、`cdk.json`（配置文件，用于指定 CDK 相关的构建和部署命令等）、`requirements.txt`（记录项目的 Python 依赖包）等。

### 3. 编写 CDK 代码来定义 AWS 资源与 Lambda 函数集成
- **定义 Lambda 函数资源**：
在 `app.py` 文件中，导入必要的 CDK 和 Lambda 相关模块，示例代码如下：
```python
from aws_cdk import core
from aws_cdk import aws_lambda

class MyCdkStack(core.Stack):
    def __init__(self, scope: core.Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        # 创建一个 Lambda 函数
        my_lambda_function = aws_lambda.Function(
            self,
            "MyLambdaFunction",
            function_name="my-lambda-function",
            runtime=aws_lambda.Runtime.PYTHON_3_8,  # 选择 Python 运行时版本，可根据实际调整
            handler="lambda_function.lambda_handler",  # 指向 Lambda 函数的入口点，后续需创建对应的 handler 函数
            code=aws_lambda.Code.from_asset("lambda")  # 假设 Lambda 函数代码位于项目目录下的 'lambda' 文件夹中
        )

# 创建 CDK 应用实例并添加栈
app = core.App()
MyCdkStack(app, "MyCdkStack")
app.synth()
```
上述代码在 CDK 栈中定义了一个 Lambda 函数，指定了函数名称、运行时版本、入口点以及代码所在位置等关键信息。这里假设 Lambda 函数的代码存放在项目目录下名为 `lambda` 的文件夹中。

- **编写 Lambda 函数代码**：
在项目目录下创建 `lambda` 文件夹（与上述代码中指定的 `code=aws_lambda.Code.from_asset("lambda")` 对应），然后在该文件夹内创建一个名为 `lambda_function.py` 的 Python 文件（对应代码中的 `handler="lambda_function.lambda_handler"`），示例的 Lambda 函数代码如下：
```python
def lambda_handler(event, context):
    print("Received event:", event)
    return {
        "statusCode": 200,
        "body": "Hello from Lambda!"
    }
```
这个简单的 Lambda 函数只是接收事件参数，打印出来，并返回一个包含状态码和简单消息的响应。你可以根据实际的业务需求替换这个函数的逻辑，比如进行数据处理、调用其他 AWS 服务等。

### 4. 构建和部署项目
- **安装项目依赖**：
在项目目录下，运行以下命令安装 `requirements.txt` 文件中列出的依赖包（初始化 CDK 项目时会自动生成基本的依赖，后续添加新依赖也需更新该文件并重新安装）：
```bash
pip install -r requirements.txt
```

- **合成 CDK 模板并部署**：
在命令行中切换到项目目录下，运行以下命令合成 CDK 应用，它会根据你的 Python 代码生成 CloudFormation 模板：
```bash
cdk synth
```
如果没有报错，说明 CDK 代码语法等方面没有问题，接着可以运行以下命令进行部署（首次部署可能需要一些时间，因为 AWS 会创建相应的资源）：
```bash
cdk deploy
```
在部署过程中，会提示你确认要创建的资源信息以及可能产生的费用等情况，按照提示输入 `y` 继续部署操作，部署成功后，你的 Lambda 函数以及相关配置的 AWS 资源就会在 AWS 环境中创建完成，可以通过 AWS 管理控制台等方式查看和使用它们。

### 5. 调试与测试
- **调试 Lambda 函数**：
利用 VS Code 的调试功能，你可以在 `lambda_function.py` 文件中设置断点，然后在 AWS 管理控制台触发 Lambda 函数（例如通过模拟事件触发），或者配置本地调试环境（通过一些工具如 AWS SAM 等，可模拟 Lambda 函数的本地运行环境），来对 Lambda 函数进行调试，查看变量值、执行流程等情况，便于排查问题和优化代码逻辑。

- **测试应用整体功能**：
根据你构建的整个应用的逻辑，通过相应的触发方式（比如如果 Lambda 函数与 API 网关关联，通过发送 API 请求来触发；如果与 S3 存储桶事件关联，通过上传文件到相应存储桶来触发等）来测试应用是否按照预期工作，检查返回结果、资源的交互情况等是否符合设计要求。

通过以上步骤，你就可以在 Visual Studio Code 中基于 AWS CDK 和 Lambda 并使用 Python 环境进行开发、部署以及测试相关的 Serverless 应用了。在实际开发过程中，可能还需要根据具体的业务场景和项目需求进一步扩展和优化代码及配置。 


## 基于nodejs环境的AWS CDK开发与Lambda函数集成

在 Visual Studio Code 中使用 Node.js 实现基于 AWS CDK 创建项目的详细步骤，示例中同样会创建包含 Lambda 函数以及相关资源（如 API 网关用于触发 Lambda 函数）的项目结构：

### 步骤一：准备工作
1. **安装必备软件**
    - 确保已安装 Visual Studio Code（VS Code），可从[VS Code 官方网站](https://code.visualstudio.com/)下载对应操作系统的安装包进行安装。
    - 安装 Node.js，前往[Node.js 官方网站](https://nodejs.org/)下载并安装长期支持版本（LTS），安装完成后，可在命令行输入 `node -v` 来确认 Node.js 的版本信息，同时 `npm`（Node.js 的包管理工具）也会一并安装，输入 `npm -v` 可查看其版本。
    - 安装 AWS CLI（命令行界面），参考[AWS CLI 官方页面](https://aws.amazon.com/cli/)的指引进行安装，安装后通过 `aws configure` 命令配置 AWS 访问密钥、秘密访问密钥、默认区域以及默认输出格式等信息，以便后续与 AWS 服务交互。

2. **安装 VS Code 扩展**
    - **Node.js 扩展**：打开 VS Code，点击左侧边栏的扩展图标（方块形状），在搜索框中输入“Node.js”，找到官方的 Node.js 扩展并安装，它可以为 Node.js 代码开发提供语法高亮、智能提示、调试支持等功能。
    - **AWS Toolkit for Visual Studio Code**：同样在扩展搜索框中搜索该名称并安装，方便后续在 VS Code 中操作 AWS 服务，比如部署 CDK 项目、管理 Lambda 函数等。

### 步骤二：创建并初始化项目
1. **创建项目文件夹**
在本地文件系统中选择合适位置创建一个新文件夹作为项目的根目录，例如命名为 `my-cdk-nodejs-demo`。

2. **初始化 CDK 项目**
打开命令行终端（在 Windows 下可使用 `cmd` 或 `PowerShell`，在 macOS 和 Linux 中使用终端应用），切换到新创建的 `my-cdk-nodejs-demo` 文件夹路径下，运行以下命令初始化一个基于 Node.js 的 AWS CDK 项目：
```bash
cdk init app --language typescript
```
这里选择使用 TypeScript，它是 JavaScript 的超集，能提供更好的类型检查和代码组织能力，当然你也可以选择纯 JavaScript（将 `--language` 参数改为 `javascript`），不过后续步骤会以 TypeScript 为例。上述命令执行后，会在 `my-cdk-nodejs-demo` 文件夹内生成一些必要的文件和文件夹，构成基本的 CDK 项目结构，主要包括：
    - `bin` 文件夹：里面存放项目的入口文件，例如 `my-cdk-nodejs-demo.ts`（文件名与项目文件夹名相关），它用于创建 CDK 应用实例并添加定义好的栈（Stack）到应用中。
    - `lib` 文件夹：用于存放定义 CDK 栈以及各种 AWS 资源的核心代码文件，后续主要在这里编写创建资源的代码逻辑。
    - `cdk.json`：配置文件，用于指定 CDK 应用在构建、合成以及部署等阶段的相关命令和参数，比如指定构建工具（默认通常是 `tsc` 用于 TypeScript 项目编译）、构建命令等内容。
    - `package.json`：Node.js 项目的标准配置文件，记录项目的基本信息、依赖关系以及可执行脚本等内容，其中 `scripts` 部分会定义一些常用的命令，如 `build`（编译代码）、`watch`（监听代码变化并自动编译）等。
    - `tsconfig.json`（针对 TypeScript 项目）：TypeScript 的配置文件，用于配置编译选项，如目标 JavaScript 版本、模块解析方式、是否包含严格类型检查等参数。

### 步骤三：编写 CDK 代码定义 AWS 资源
1. **编辑项目入口文件（`bin/my-cdk-nodejs-demo.ts`）**
打开 VS Code，通过“文件”菜单选择“打开文件夹”，然后选中 `my-cdk-nodejs-demo` 文件夹打开项目。找到 `bin/my-cdk-nodejs-demo.ts` 文件并打开进行编辑，示例代码如下：

```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { MyCdkStack } from '../lib/my-cdk-nodejs-demo-stack';

const app = new cdk.App();
new MyCdkStack(app, 'MyCdkStack');
app.synth();
```

这段代码首先引入了必要的模块，然后创建了一个 CDK 应用实例 `app`，接着将自定义的 `MyCdkStack`（后续在 `lib` 文件夹中定义）添加到应用中，最后调用 `app.synth()` 方法来合成 CloudFormation 模板，这是后续部署资源的基础。

2. **编写 CDK 栈代码定义资源（`lib/my-cdk-nodejs-demo-stack.ts`）**
在 `lib` 文件夹下找到或创建 `my-cdk-nodejs-demo-stack.ts` 文件（文件名与入口文件中的类名对应），编辑内容如下：

```typescript
import * as cdk from '@aws-cdk/core';
import * as lambda from '@aws-cdk/aws-lambda';
import * as apigateway from '@aws-cdk/aws-apigateway';

export class MyCdkStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // 创建 Lambda 函数
    const myLambdaFunction = new lambda.Function(this, 'MyLambdaFunction', {
      functionName: 'my_demo_lambda_function',
      runtime: lambda.Runtime.NODEJS_14_X, // 选择 Node.js 运行时版本，可按需调整
      handler: 'lambda_handler.handler',
      code: lambda.Code.fromAsset('lambda'),
    });

    // 创建 API 网关并关联 Lambda 函数
    const api = new apigateway.RestApi(this, 'MyApiGateway', {
      rest_api_name: 'my_demo_api',
    });

    const integration = new apigateway.LambdaIntegration(myLambdaFunction);

    const apiRoot = api.root.addResource('hello');
    apiRoot.addMethod('GET', integration);
  }
}
```

在上述代码中：
    - 首先导入了 AWS CDK 相关的核心模块以及用于 Lambda 函数和 API 网关的具体模块。
    - 定义了 `MyCdkStack` 类继承自 `cdk.Stack`，在其构造函数中：
        - 创建了一个名为 `MyLambdaFunction` 的 Lambda 函数，指定了函数名、Node.js 运行时版本（这里选择了 `NODEJS_14_X`，你可根据实际情况更改）、函数入口点（后续需创建对应的处理函数）以及代码所在位置（假设代码存放在项目根目录下名为 `lambda` 的文件夹中）。
        - 接着创建了一个名为 `MyApiGateway` 的 API 网关，然后将之前创建的 Lambda 函数通过 `LambdaIntegration` 关联起来，并且在 API 网关的根路径下添加了名为 `hello` 的资源，配置了 `GET` 方法来触发对应的 Lambda 函数。

3. **编写 Lambda 函数代码**
在项目根目录下创建 `lambda` 文件夹，然后在 `lambda` 文件夹内创建一个名为 `lambda_handler.js`（如果是纯 JavaScript 项目） 或 `lambda_handler.ts`（如果是 TypeScript 项目，这里以 TypeScript 为例）的文件，内容如下：

```typescript
export const handler = async (event: any, context: any) => {
  return {
    statusCode: 200,
    body: 'Hello from the Lambda function triggered by API Gateway!',
  };
};
```

这个简单的 Lambda 函数处理程序接收事件和上下文参数，返回一个包含状态码和消息的响应，表示成功响应 API 网关的请求并返回相应内容。你可以根据实际业务需求对这个函数进行扩展，比如添加数据库查询、数据处理等更复杂的逻辑，若是使用 TypeScript，注意进行相应的类型定义和检查。

### 步骤四：安装项目依赖并构建部署项目
1. **安装依赖包**
在 VS Code 的终端（确保终端所在目录为项目根目录 `my-cdk-nodejs-demo`）中，运行以下命令安装项目所需的依赖包，这些包用于支持 CDK 以及相关 AWS 服务模块的使用：

```bash
npm install
```

此命令会读取 `package.json` 文件中的 `dependencies` 和 `devDependencies` 等字段，安装对应的 Node.js 包，包括 `@aws-cdk/core`、`@aws-cdk/aws-lambda`、`@aws-cdk/aws-apigateway` 等 CDK 相关模块以及其他可能的依赖。

2. **编译 TypeScript 代码（如果使用 TypeScript）**
如果项目使用的是 TypeScript，需要先将其编译为 JavaScript 才能进行后续部署操作，运行以下命令进行编译：

```bash
npm run build
```

这个命令会根据 `tsconfig.json` 配置文件中的设置，将 `.ts` 文件编译为 `.js` 文件，并输出到相应的目录（通常是 `lib` 文件夹下对应的位置）。如果是纯 JavaScript 项目，则可跳过此步骤。

3. **合成 CDK 模板**
运行以下命令合成 CDK 应用，它会依据编写的代码生成对应的 CloudFormation 模板：

```bash
cdk synth
```

若命令执行顺利且无报错信息，表明 CDK 代码的语法和结构符合要求，可继续下一步的部署操作。

4. **部署项目**
运行以下命令进行项目部署，将在 AWS 中创建之前在 CDK 代码中定义的各类资源（如 Lambda 函数、API 网关等）：

```bash
cdk deploy
```

在部署过程中，终端会提示你确认要创建的资源信息以及可能涉及的费用情况（针对会产生费用的资源），按照提示输入 `y` 并回车，即可启动部署流程。部署所需时间会因资源创建的复杂程度而有所不同，完成部署后，你就能通过 AWS 管理控制台或者相应的 API 来访问和测试所创建的资源了。

通过上述完整的步骤，你便可以在 Visual Studio Code 中利用 Node.js（或 TypeScript）成功创建并部署一个基于 AWS CDK 的项目，构建包含 Lambda 函数和 API 网关等资源的 AWS 资源配置，后续还可根据具体业务场景进一步拓展和优化项目功能。 

