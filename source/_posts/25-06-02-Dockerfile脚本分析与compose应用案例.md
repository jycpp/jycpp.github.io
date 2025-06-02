---
title: Dockerfile脚本分析与compose应用案例
date: 2025-06-02 17:27:20
comments: true
tags:
- Dockerfile
- Docker Compose
- 镜像构建
- 容器部署
- 微服务
- SkyWalking
- JVM参数
- 时区设置
- 字体配置
- 环境变量
categories:
- 软件开发
- 运维管理
---


玩转Docker，如果不会写Dockerfile脚本，就像天天谈论汽车而没有驾照一样尴尬。我们以人人微服务开发框架提供的一个Dockerfile脚本为例，分析Dockerfile脚本的编写，并基于这个脚本，将Docker上传到私有镜像仓库后，通过Docker-compose启动多个容器，实现微服务的部署。


## Dockerfile脚步分析

```dockerfile
# 构建镜像，执行命令：【docker build -t renren_io:2.0 .】
FROM openjdk:8u212-jre
MAINTAINER Mark
# 时区问题
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone

# 字体
COPY fonts/simsun.ttc /usr/share/fonts/simsun.ttc

ADD apache-skywalking-apm-bin/agent/ /agent

ENTRYPOINT ["java", "-server", "-Xms512M", "-Xmx512M", "-Djava.security.egd=file:/dev/./urandom", "-Dfile.encoding=UTF-8", "-XX:+HeapDumpOnOutOfMemoryError", "-javaagent:/agent/skywalking-agent.jar", "-jar", "/app/app.jar" ]
```


**关键点：**

- `FROM openjdk:8u212-jre`：指定基础镜像，这里使用的是 OpenJDK 8 的 JRE 版本。
- `MAINTAINER Mark`：指定镜像的维护者信息。
- `RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`：设置容器的时区为上海。
- `RUN echo 'Asia/Shanghai' >/etc/timezone`：同上。
- `COPY fonts/simsun.ttc /usr/share/fonts/simsun.ttc`：将字体文件复制到容器中，不是ADD，只是复制，不会解压，不会下载。
  - 注意：这里的 `fonts/simsun.ttc` 是相对于 `dockerfile` 文件所在的目录的路径。
- `ADD apache-skywalking-apm-bin/agent/ /agent`：将 SkyWalking 代理添加到容器中，这里代理文件位于 跟dockerfile同级的 `apache-skywalking-apm-bin`的 `agent` 目录下。

- `ENTRYPOINT ["java", "-server", "-Xms512M", "-Xmx512M", "-Djava.security.egd=file:/dev/./urandom", "-Dfile.encoding=UTF-8", "-XX:+HeapDumpOnOutOfMemoryError", "-javaagent:/agent/skywalking-agent.jar", "-jar", "/app/app.jar" ]`：指定容器启动时执行的命令。
  - `-server`：启用 JVM 的服务器模式，适用于生产环境。
  - `-Xms512M`：设置 JVM 的初始堆内存大小为 512MB。
  - `-Xmx512M`：设置 JVM 的最大堆内存大小为 512MB。
  - `-Djava.security.egd=file:/dev/./urandom`：设置 JVM 的安全随机数生成器，以提高安全性。
  - `-Dfile.encoding=UTF-8`：设置文件编码为 UTF-8，确保文件读取和写入时的正确编码。
  - `-XX:+HeapDumpOnOutOfMemoryError`：当 JVM 发生内存溢出时，自动生成堆转储文件，用于分析内存泄漏。
  - `-javaagent:/agent/skywalking-agent.jar`：加载 SkyWalking 代理，用于收集和分析应用程序的性能数据。
  - `-jar /app/app.jar`：指定要运行的 JAR 文件，这里假设应用程序的 JAR 文件名为 `app.jar`，并位于 `/app` 目录下。

- ENTRYPOINT的参数较多，可以考虑将环境变量参数化：
```dockerfile
ENV JAVA_OPTS="-Xms512M -Xmx512M"
ENTRYPOINT ["java", "$JAVA_OPTS", "-javaagent:/agent/skywalking-agent.jar", "-jar", "/app/app.jar"]
```

**构建与使用**

```bash
# 构建镜像
docker build -t renren_io:2.0 .

# 运行容器
docker run -p 8080:8080 \
  -e SW_AGENT_NAME=renren-app \
  -e SW_AGENT_COLLECTOR_BACKEND_SERVICES=skywalking-oap:11800 \
  renren_io:2.0
```


## compose应用案例

与这个镜像呼应的/app/app.jar，是这个容器运行时候“-jar”参数指定的要运行的内容，我们可以在运行基于 renren_io 创建的镜像的时候设置。

以下compose脚步中，关键点是：通过“volumes”选项，将宿主机的 /usr/local/data/renren/renren-monitor.jar 文件挂载到容器的 /app/app.jar 路径下，从而实现启动容器的时候运行指定的 jar 文件的目的。


```yaml
version: '3.3'
services:
  renren-monitor:
    image: renren_io:2.0
    container_name: renren-monitor
    env_file:
      - common.env
    volumes:
      - /usr/local/data/renren/renren-monitor.jar:/app/app.jar
  renren-gateway:
    image: renren_io:2.0
    container_name: renren-gateway
    ports:
      - "8080:8080"
    links:
      - skywalking-oap
    environment:
      - SW_AGENT_NAME=renren-gateway
      - SW_AGENT_COLLECTOR_BACKEND_SERVICES=skywalking-oap:11800
    env_file:
      - common.env
    volumes:
      - /usr/local/data/renren-cloud/renren-gateway.jar:/app/app.jar
```

**关键点：**

- `image: renren_io:2.0`：指定要使用的镜像名称和标签。
- `container_name: renren-monitor`：指定容器的名称。
- `env_file: - common.env`：从 `common.env` 文件中加载环境变量，文件位于yaml脚步文件同级的目录下。
- `volumes: - /usr/local/data/renren/renren-monitor.jar:/app/app.jar`：将主机上的 `renren-monitor.jar` 文件挂载到容器的 `/app/app.jar` 路径下。
  - 注意：这里的路径 `/usr/local/data/renren/renren-monitor.jar` 是相对于宿主机的路径，而 `/app/app.jar` 是相对于容器的路径。
- `ports: - "8080:8080"`：将容器的 8080 端口映射到主机的 8080 端口，以便外部访问容器的应用。
- `links: - skywalking-oap`：指定容器之间的链接关系，这里将 `renren-gateway` 容器与 `skywalking-oap` 容器建立链接，与depends_on的区别是，前者是容器之间的链接关系，后者是容器启动的顺序关系。
- `environment: - SW_AGENT_NAME=renren-gateway - SW_AGENT_COLLECTOR_BACKEND_SERVICES=skywalking-oap:11800`：设置容器的环境变量，包括 SkyWalking 代理的名称和收集器的后端服务地址，是  renren_io:2.0 镜像要求的参数。
- skywalking-oap与这里要讨论的内容关系不大，不展开说了，可以参考skywalking官方文档，另外这个视频《20分钟上手新版Skywalking 9.x APM监控系统》讲得比较浅显易懂。

