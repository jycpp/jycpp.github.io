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


## 4、stages与steps的区别

在 Jenkins 的 **Pipeline as Code**（通常使用 **Jenkinsfile** 编写）中，`stages` 和 `steps` 是构建流程中两个不同层级的概念，它们的区别主要体现在 **逻辑分组** 和 **执行粒度** 上。以下是详细对比：

---

### **1. 核心区别**
| **概念**       | **作用**                                                                 | **层级**       | **典型内容**                                                                 |
|----------------|--------------------------------------------------------------------------|----------------|------------------------------------------------------------------------------|
| **stages**     | 将整个构建流程 **逻辑分组** 为多个阶段（如构建、测试、部署），每个阶段有明确的职责和目标。 | **高层级**（宏观） | 包含多个 `steps`，代表一个完整的、独立的构建环节（例如 "Build" 阶段包含编译代码的步骤）。 |
| **steps**      | 定义 **具体执行的操作**（如运行 shell 命令、调用插件、执行脚本等），是实际执行的代码单元。 | **低层级**（微观） | 包含具体的指令（例如 `sh 'mvn clean package'` 或 `echo 'Hello'`）。           |

---

### **2. 结构关系**
Jenkins Pipeline 的典型结构是 **`stages` 包含多个 `stage`，每个 `stage` 包含一个或多个 `steps`**：
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {          // 阶段1：构建
            steps {               // 该阶段的具体步骤
                sh 'mvn clean package'  // 步骤1：编译代码
                echo 'Build completed!' // 步骤2：打印日志
            }
        }
        stage('Test') {           // 阶段2：测试
            steps {
                sh 'mvn test'     // 步骤1：运行单元测试
            }
        }
        stage('Deploy') {         // 阶段3：部署
            steps {
                sh 'scp target/app.jar user@server:/opt/app' // 步骤1：部署到服务器
            }
        }
    }
}
```

---

### **3. 详细说明**
#### **(1) stages（阶段）**
- **作用**：将构建流程划分为逻辑上独立的阶段，每个阶段代表一个完整的任务单元（如编译、测试、部署）。  
- **特点**：
  - **可视化**：在 Jenkins 的 Blue Ocean 或经典 UI 中，`stages` 会以 **独立的卡片/块** 形式展示，便于区分不同阶段的执行状态（成功/失败）。
  - **并行/顺序控制**：通过 `parallel` 或 `stage` 的顺序排列，控制阶段的执行逻辑（例如并行测试多个模块）。
  - **必选结构**：在声明式 Pipeline（`pipeline { ... }`）中，`stages` 是 **必需的顶级块**（除非使用脚本式 Pipeline）。

- **常见阶段示例**：
  - `Build`：编译代码（如 Maven/Gradle/NPM）。
  - `Test`：运行单元测试、集成测试。
  - `Deploy`：部署到测试环境/生产环境。
  - `Notify`：发送通知（如邮件、Slack）。

#### **(2) steps（步骤）**
- **作用**：定义每个阶段内 **具体执行的操作**，是实际运行的代码指令。  
- **特点**：
  - **原子性**：每个 `step` 是一个独立的操作（如执行一条 shell 命令、调用一个 Jenkins 插件功能）。
  - **灵活性**：支持多种类型的步骤（Shell 命令、Groovy 脚本、插件提供的专用步骤等）。
  - **必选内容**：每个 `stage` 必须包含至少一个 `steps` 块（即使只有一个步骤）。

- **常见步骤示例**：
  - `sh 'command'`：在 Linux/macOS 上执行 Shell 命令。
  - `bat 'command'`：在 Windows 上执行批处理命令。
  - `echo 'message'`：打印日志信息。
  - `checkout scm`：拉取代码仓库的最新代码。
  - 插件步骤：如 `docker.build('image:tag')`（Docker 插件）、`npm('install')`（NodeJS 插件）。

---

### **4. 对比示例**

#### **场景**：构建一个 Java 应用并部署

##### **声明式 Pipeline（推荐）**

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm  // 从代码仓库拉取代码
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'  // 编译代码
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'  // 运行测试
            }
        }
        stage('Deploy') {
            steps {
                sh 'scp target/app.jar user@server:/opt/app'  // 部署到服务器
            }
        }
    }
}
```

- **说明**：  
  - `stages` 划分为 4 个逻辑阶段（拉取代码、构建、测试、部署）。  
  - 每个 `stage` 包含 1~2 个 `steps`（具体操作）。

##### **脚本式 Pipeline（灵活但复杂）**

```groovy
node {
    stage('Build') {
        sh 'mvn clean package'  // steps 直接写在 stage 块内（隐式 steps）
    }
    stage('Deploy') {
        echo 'Starting deployment...'
        sh 'scp target/app.jar user@server:/opt/app'
    }
}
```

- **说明**：  
  - 脚本式 Pipeline 没有显式的 `stages` 块，但通过 `stage('名称') { ... }` 隐式分组，内部的代码默认属于 `steps`。

---

### **5. 关键总结**
| **问题**                | **答案**                                                                 |
|-------------------------|--------------------------------------------------------------------------|
| **stages 是什么？**      | 将构建流程分为多个逻辑阶段（如构建、测试、部署），每个阶段有明确目标。       |
| **steps 是什么？**       | 阶段内具体执行的操作（如运行命令、调用插件），是实际执行的代码单元。         |
| **为什么需要 stages？**  | 提高可读性、可视化执行状态、支持阶段级控制（如并行、失败暂停）。             |
| **steps 可以单独存在吗？**| 在脚本式 Pipeline 中可以隐式存在（通过 `stage` 块），但声明式 Pipeline 必须显式包含在 `stages` 中。 |
| **常见步骤类型**         | Shell 命令（`sh`/`bat`）、日志输出（`echo`）、插件步骤（如 Docker/NPM 操作）。 |

---

### **6. 最佳实践**
1. **合理划分 stages**：  
   - 按职责分离（如构建、测试、部署分开），便于定位问题。
2. **每个 stage 包含明确的 steps**：  
   - 避免在一个 `stage` 中塞入过多无关操作。
3. **利用声明式 Pipeline**：  
   - 结构清晰，易于维护，适合大多数场景。
4. **调试 steps**：  
   - 通过 `echo` 或 `script { ... }` 块输出调试信息（例如 `echo "当前目录: ${pwd()}"`）。

通过理解 `stages` 和 `steps` 的区别，可以更高效地设计和维护 Jenkins Pipeline，确保构建流程的可读性和可维护性。


  