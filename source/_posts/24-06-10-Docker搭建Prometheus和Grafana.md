---
title: 使用Docker搭建Prometheus+Grafana运维监控可视化
date: 2024-06-10 17:27:20
index_img:  https://s2.loli.net/2025/02/10/lKNpJUo5QkOt4gL.png
index_img_text: 使用Docker搭建Prometheus+Grafana运维监控可视化
comments: true
tags:
- Prometheus
- Grafana
- Kubernetes
- Alertmanager
- Exporters
- Docker
- 运维监控
- 可视化
- 监控系统
categories:
- 软件开发
- 运维管理
---

# **1.项目**

今天和大家分享一个运维监控可视化工具 —— Prometheus（普罗米修斯），通过部署Prometheus+Grafana来监控Linux主机，实现运维监控可视化，目前 Prometheus 已经广泛用于 Kubernetes 集群的监控系统中

## **1.1.项目介绍**

Prometheus的架构由四个主要组件组成：

1. **Prometheus Server** ：Prometheus Server是Prometheus的核心组件，主要负责从各个目标（target）中收集指标（metrics）数据，并对这些数据进行存储、聚合和查询。

2. **Client Libraries** ：Prometheus提供了多种客户端库，用于在应用程序中嵌入Prometheus的指标收集功能。

3. **Exporters** ：Exporters是用于将第三方系统的监控数据导出为Prometheus格式的组件。Prometheus支持多种Exporters，例如Node Exporter、MySQL Exporter、HAProxy Exporter等。

4. **Alertmanager**：Alertmanager是Prometheus的告警组件，用于根据用户定义的规则对监控数据进行告警。

同时Prometheus有以下优点

1. 灵活的数据模型：Prometheus采用的是key-value对的形式存储指标数据，每个指标都可以包含多个标签（labels），这样可以更加灵活地描述指标数据

2. 高效的存储和查询：Prometheus使用自己的时间序列数据库，可以高效地存储和查询大量的指标数据。

3. 强大的可视化和告警功能：Prometheus提供了Web界面和API，可以方便地展示和查询监控数据。

4. 可扩展性强：Prometheus的架构非常灵活，可以根据需要选择合适的组件进行配置。

5. CNCF的成员项目：Prometheus作为CNCF的项目之一，得到了广泛的关注和支持，并且得到了来自全球各地的贡献者的积极参与和开发。

**1.1.项目展示**

![](https://s2.loli.net/2025/02/10/lKNpJUo5QkOt4gL.png)

![](https://s2.loli.net/2025/02/10/LJOMjmXTrGCHVIW.png)

# **2.相关地址**

官方GitHub地址： <https://github.com/prometheus/prometheus>

官网地址：<https://prometheus.io/>

# **3.搭建环境**

* **服务器**：建议在云服务器上搭建，特别是涉及到Docker镜像，国内服务器访问国外源较慢，建议使用海外服务器。如果使用国内云服务器，建议使用国内镜像源，否则可能会出现镜像下载失败的情况。各个大厂的服务器基本都有自己的镜像源，具体可以参考各个大厂的文档。

* **资源配置**：2核2G 30G硬盘
  建议服务器内存1G以上，由于国内服务器访问海外源较慢，这边为了方便演示直接使用海外服务器搭建，如国内项目建议使用国内服务器。

* **服务器系统**：我们这里使用的Linux发型版本是 Debian-11。

* 【必需】**安装Docker**：安装好 Docker、Docker-compose

* 【非必需】域名一枚，可用于解析到服务器上使用域名访问

* 【非必需】**安装Nginx**：安装好 Nginx，用于反向代理

* 【非必需】**安装SSL证书**：安装好 SSL 证书，用于 HTTPS 访问

# **4.搭建视频**

本次操作没有录制视频，有需要的可以私信我，以后会录制视频。

# **5.搭建方式**

## **5.1 安装docker和docker-compose**

安装教程：[服务器上安装docker和docker-compose教程](https://blog.lcayun.com/3204.html)

## **5.2 安装Grafana**

Grafana下载地址：<https://grafana.com/grafana/download>

![](https://s2.loli.net/2025/02/10/9SHciGwudy1BWmI.png)

我们这里使用docker搭建

**复制

```plain&#x20;text
docker run -d --name=grafana -p 3000:3000 grafana/grafana-enterprise
```

docker拉取完就安装完成了

访问Grafana，浏览器输入IP:端口，如：服务器IP:3000

默认账号密码为：admin / admin

![](https://s2.loli.net/2025/02/10/gn64ofbGWtxBH1q.png)

第一次登录需要修改密码，这个按照自己需求修改登录密码

进去之后首页

## **5.3 安装Prometheus**

依次下载**Prometheus**镜像包



```plain&#x20;text
docker pull prom/node-exporter
docker pull prom/prometheus
```

![](https://s2.loli.net/2025/02/10/76MwdBQY2b5gfJp.png)

运行node-exporter的docker



```plain&#x20;text
docker run -d -p 9100:9100  -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro"  --net="host"  prom/node-exporter
```

访问 http://服务器:9100/metrics 看看是否有数据

![](https://s2.loli.net/2025/02/10/sSgZedbBUnjwtxp.png)

## **5.4 启动prometheus**

在opt目录下建立promethes文件夹



```plain&#x20;text
mkdir /opt/prometheus
```

打开/opt/prometheus/的目录



```plain&#x20;text
cd /opt/prometheus/
```

使用vim编辑prometheus.yml文件



```plain&#x20;text
vim prometheus.yml
```

输入内容如下（注意：localhost记得改为自己的服务器IP）



```bash
global:
抓取间隔，60秒向目标抓取一次数据
  scrape_interval: 60s
  evaluation_interval: 60s

这里表示抓取对象的配置
scrape_configs:
  - job_name: 'prometheus'
  重写了全局抓取间隔时间，由60秒重写成30秒
    scrape_interval: 30s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'linux'
    static_configs:
      - targets: ['localhost:9100']
```



输入修改完IP按esc键 输入 :wq 保存退出

启动Prometheus



```plain&#x20;text
docker run -d  -p 9090:9090  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```



输入完打开 http://服务器IP:9090/query 显示以下界面



![](https://s2.loli.net/2025/02/10/tAynahvBGboLkOJ.png)

显示绿色UP则为运行正常

![](https://s2.loli.net/2025/02/10/LzGxgRoCS9hmuf6.png)

## **5.5 Grafana添加数据源**



回到 http://服务器IP:3000/ Grafana添加prometheus数据源



![](https://s2.loli.net/2025/02/10/KLnmtr5ZpzDUFMk.png)

选择prometheus

![](https://s2.loli.net/2025/02/10/XoWlscDiUpk5wEv.png)

此行输入 http://服务器IP:9090 后滑下去直接点击save保存

![](https://s2.loli.net/2025/02/10/cAm2uU1qjK5O4bF.png)

## **5.6 Grafana添加面板**

回到首页点击DASHBOARDS

选择导入

![](https://s2.loli.net/2025/02/10/36o4dVn5zplvQCs.png)

访问官方面板下载地址：<https://grafana.com/grafana/dashboards>

此处输入项目ID

![](https://s2.loli.net/2025/02/10/tF9fjI5BET3Jpyn.png)

比如选中项目之后可以复制这个项目的ID填上去

填好ID后我们需要选择prometheus，然后点击import导入即可

这样面板信息就出来了

# **6.结尾**

Prometheus是个非常好用的开源项目，在本文中，我们介绍了什么是Prometheus，如何安装Prometheus，以及使用Prometheus的Pull(拉取)模式来采集Linux服务器资源，并在Grafana进行展现。
