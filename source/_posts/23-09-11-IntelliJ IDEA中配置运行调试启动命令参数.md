---
title: IntelliJ IDEA中配置运行调试启动命令参数
date: 2023-09-11 14:35:20
comments: true
tags:
- IntelliJ IDEA
- Spring Boot
- Springboot
- IDEA编辑配置
- IDEA参数
- IDEA调试
- IDEA运行
- VM Options
- Program arguments
- Nacos
categories:
- 软件开发
- 微服务
---


## 一、操作过程记录


在 IntelliJ IDEA 中运行或调试 Spring Boot 应用时，可以通过以下方式为启动命令添加参数（如 `--nacos_host=192.168.1.7`）：

### 1. 配置 Run/Debug Configurations 添加 VM Options 或 Program arguments



![运行>>编辑配置](https://s2.loli.net/2025/05/17/L8Xxq5Ibar1imUR.png)


#### 步骤如下：
1. 打开顶部菜单栏：**Run > Edit Configurations...**
2. 选择你的 Spring Boot 启动配置（通常是一个 Application 类）。
3. 在 **Configuration** 标签下找到：
   - **VM options**：用于设置 JVM 参数（以 `-D` 开头）
   - **Program arguments**：用于设置应用程序参数（即你希望通过 `--xxx=yyy` 传递的参数）

#### 示例：
```text
--nacos_host=192.168.1.7 --nacos_port=8848
```

将上述内容粘贴到 **Program arguments** 输入框中。


![设置命令行参数](https://s2.loli.net/2025/05/17/EsG1D7mfzLSkwYd.png)

---

### 2. 使用 Environment Variables（环境变量）方式替代参数传入

如果希望通过环境变量定义（比如部署和开发分离），也可以这样写：

```text
-Dnacos_host=192.168.1.7 -Dnacos_port=8848
```


然后将其粘贴到 **VM options** 中。

*如上图的第二个方框所示*

---

### 3. 保存并运行
点击 **Apply** 和 **OK**，然后使用 Run 或 Debug 按钮启动应用即可生效。

---

✅ **验证方法**：
- 可以在代码中注入 `Environment` 并打印出 `nacos_host` 和 `nacos_port` 的值。
- 或者查看启动日志，Spring Boot 会在启动时打印出使用的配置项。


```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class NacosConfigPrinter {

    @Autowired
    private Environment environment;

    @PostConstruct
    public void printNacosConfig() {
        String nacosHost = environment.getProperty("nacos_host");
        String nacosPort = environment.getProperty("nacos_port");

        System.out.println("nacos_host: " + nacosHost);
        System.out.println("nacos_port: " + nacosPort);
    }
}
```


如有需要可以进一步配置 Spring Profiles 来区分 dev/test 环境。


## 二、补充Springboot基础知识

之前有网友专门私发Email询问以上示例代码的含义，示例代码中的 NacosConfigPrinter 如何在Springboot项目中被调用呢？

在 Spring Boot 项目中，`NacosConfigPrinter` 是一个被 `@Component` 注解标记的组件类，会被 Spring 容器自动扫描并注册为 Bean。它通过 `@PostConstruct` 注解确保在 Bean 初始化完成后执行打印逻辑。

### 调用流程说明：

1. **Spring 容器启动**
   - 在 Spring Boot 应用启动时，会自动扫描 `@Component`、`@Service` 等注解的类。
   - `NacosConfigPrinter` 类会被注册为一个单例 Bean。

2. **依赖注入完成**
   - `@Autowired` 注解的 `Environment` 属性会被正确注入。

3. **`@PostConstruct` 方法自动调用**
   - 在 Bean 初始化完成后，Spring 会自动调用标注了 `@PostConstruct` 的方法（即 `printNacosConfig()`）。
   - 此时会从配置中读取 `nacos_host` 和 `nacos_port` 并打印到控制台。

---

### 验证方式：

你可以在 Spring Boot 启动日志中看到如下输出（假设你在本地设置了参数）：

```text
nacos_host: 192.168.1.7
nacos_port: 8848
```


> ✅ 注意：如果未设置参数，则输出可能为默认值：
```text
nacos_host: nacos-host
nacos_port: 8848
```


---

### 如何确保该类被调用？

- **确保类在组件扫描路径下**  
  如果你的 Spring Boot 主启动类上有 `@SpringBootApplication` 注解，它会默认扫描同包及其子包下的组件。
  - 如果 `NacosConfigPrinter` 所在包不在主类的子包下，需要显式配置组件扫描路径：
    ```java
    @SpringBootApplication
    @ComponentScan(basePackages = "com.yourcompany.yourpackage")
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
    ```


- **确保类不是被 new 出来的**  
  不要使用 `new NacosConfigPrinter()` 创建实例，这样会导致 `@Autowired` 注入失败。应由 Spring 管理生命周期。

---

### 总结：

| 步骤 | 内容 |
|------|------|
| 1 | `@Component` 让 Spring 管理该类 |
| 2 | `@Autowired` 注入 `Environment` |
| 3 | `@PostConstruct` 触发打印逻辑 |
| 4 | Spring Boot 启动时自动执行 |

只要满足组件扫描路径和依赖注入条件，`NacosConfigPrinter` 就会被 Spring 自动调用并打印配置信息。


**扩展：使用 @Value 注解直接注入属性**

也可以通过 @Value 直接将属性注入到字段中：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class NacosConfigPrinter {

    @Value("${nacos_host}")
    private String nacosHost;

    @Value("${nacos_port}")
    private String nacosPort;

    @PostConstruct
    public void printNacosConfig() {
        System.out.println("nacos_host: " + nacosHost);
        System.out.println("nacos_port: " + nacosPort);
    }
}

```


