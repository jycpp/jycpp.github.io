---
title: Jenkins持续集成CI与持续部署CD过程说明
date: 2023-02-07
comments: true
tags:
- Jenkins
- 持续集成
- 持续部署
- 自动化服务器
- 软件开发周期
- 软件质量
categories:
- 软件开发
- 运维管理
---

Jenkins 是一个开源自动化服务器，广泛用于持续集成（Continuous Integration, CI）和持续部署（Continuous Deployment, CD）。通过使用 Jenkins，开发团队可以自动化构建、测试和部署应用程序，从而加快软件开发周期，提高软件质量。



## 1 持续集成（Continuous Integration）



持续集成是一种软件开发实践，其中团队频繁地将代码更改合并到共享存储库中。每次代码更改后，自动构建和测试过程运行，以确保这些更改不会破坏整个应用程序。



### 1.1 安装 Jenkins：



在服务器上安装 Jenkins。你可以从 Jenkins官网 下载安装包或使用包管理器（如 apt, yum）进行安装。



![Jenkins在CI/CD中的作用](https://s2.loli.net/2025/02/15/xGEwk3Minhfazev.png)



### 1.2 配置 Jenkins：



安装完成后，访问 Jenkins 的 Web 界面（通常是 http://\<your-server>:8080），并根据提示完成初始设置。



![](https://s2.loli.net/2025/02/15/natE974e1LRG5Qg.png)



### 1.3 创建 Jenkins 项目：



在 Jenkins 仪表盘上，点击“新建”来创建一个新的项目。

选择合适的项目类型（例如，自由风格项目或使用流水线项目）。



![](https://s2.loli.net/2025/02/15/5jOvoVAiQBIXZmK.png)



### 1.4 配置源代码管理：



指定你的代码仓库（如 Git），并输入仓库的 URL。



![](https://s2.loli.net/2025/02/15/gd6LGpnTeCHmMrO.png)



### 1.5 配置构建触发器：



设置构建触发器，例如轮询 SCM（定期检查 SCM 的更改），或使用 Webhook 来实时触发构建。



![](https://s2.loli.net/2025/02/15/fmajU8S6KWhip4N.png)



### 1.6 添加构建步骤：



这是Jenkins设置的最重要的步骤，配置构建脚本（如 Maven、Gradle 或 shell 脚本），用于编译和构建你的应用程序。

![](https://s2.loli.net/2025/02/15/q7J9ugbOECPr6My.png)



### 1.7 添加测试步骤：



配置测试脚本，例如使用 JUnit 或 TestNG 进行单元测试和集成测试。



### 1.8 配置构建后操作：



配置构建后操作，如将构建结果归档、发送通知等。



## 2 持续部署（Continuous Deployment）



持续部署是一种扩展了持续集成的实践，其中每次成功的代码更改都会自动部署到生产环境或其他目标环境。



### 2.1 配置部署环境：



在 Jenkins 中配置部署环境，例如使用 SSH、Docker 或 Kubernetes 等技术将应用程序部署到目标服务器或容器中。



### 2.2 添加部署步骤：



在 Jenkins 的构建配置中添加部署脚本或使用 Jenkins 插件（如 Publish Over SSH, Kubernetes Plugin 等）来自动化部署过程。



### 2.3 配置环境变量和凭据：



确保 Jenkins 有权访问部署目标，并正确配置任何必要的环境变量和凭据（如 SSH 密钥、Docker 镜像仓库凭据等）。



### 2.4 使用蓝绿部署或金丝雀发布：



为了减少风险，可以使用蓝绿部署或金丝雀发布策略，在生产环境中逐步引入新版本的应用程序。



### 2.5 监控和回滚：



设置监控来跟踪部署后的应用程序性能，并准备好在出现问题时快速回滚到先前的稳定版本。



## 3 示例：使用 Jenkins Pipeline 实现 CI/CD



Jenkins Pipeline 提供了一种声明式和脚本式的方式来定义项目的自动化流程。以下是一个简单的 Jenkinsfile 示例，用于实现 CI/CD：



```json
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/your-project.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'your-server', transfers: [sshTransfer(sourceFiles: 'target/*.jar', removePrefix: 'target/', remoteDirectory: '/path/to/deploy', remoteDirectorySDF: false, usePromotionTimestamp: false, flatten: false, cleanRemote: true, failOnError: false, verbose: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', override: true, excludes: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, promote: false, continueOnError: false)])
            }
        }
    }
}
```



这个示例中，我们定义了四个阶段：检出代码、构建、测试和部署。通过这种方式，可以轻松地实现持续集成和持续部署的自动化。

