---
title: 使用Docker搭建Prometheus+Grafana运维监控可视化
date: 2024-06-10 17:27:20
comments: true
tags:
- 电子邮件
- 邮件协议
- SMTP
- 邮件服务器
categories:
- 电子邮件
- SMTP
---

# **1.项目**

今天和大家分享一个运维监控可视化工具——Prometheus（普罗米修斯），通过部署Prometheus+Grafana来监控Linux主机，实现运维监控可视化，目前 Prometheus 已经广泛用于 Kubernetes 集群的监控系统中

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

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=YTRhMzg5NGU4MmJiZTI0ZTYzNGU0MTZhZmU2MDMzNzBfM0tWcThDbHljbVU3S2JzSjkwZDFlc1FhT3Z5clJseG1fVG9rZW46S2kxaWJEQWFxb01aVVd4UnhYMGNyYmFSbmhkXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=OTVmNjE4NDJhYzk0N2E2ZmRkY2EzY2FmMjQ5ZGNkZWNfaEVOaTZrS0N5ZUNHaVJqMXBMcldoeGhEQTBoT3lleTJfVG9rZW46TG5JVWJuTnUxbzVjSjV4UWI2amN3R25UbmFmXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

# **2.相关地址**

官方GitHub地址： <https://github.com/prometheus/prometheus>

官网地址：<https://prometheus.io/>

# **3.搭建环境**

* **服务器**：使用的是莱卡云的境外香港云服务器。莱卡云服务器促销活动性价比会更高。查看官网购买链接：[https://www.lcayun.com](https://www.lcayun.com/)

* **资源配置**：2核2G 30G硬盘
  建议服务器内存1G以上，由于国内服务器访问海外源较慢，这边为了方便演示直接使用海外服务器搭建，如国内项目建议使用国内服务器。

* **服务器系统**：Debian-11

* 【必需】**安装Docker**：安装好 Docker、Docker-compose

* 【非必需】域名一枚，可用于解析到服务器上使用域名访问

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=MmRlMzk3NmQzZjg2NDdlZDdmMTE1ZWYxZTBiNTgwYzNfR2NONWVmOFQxQ0xzSU92VDJIUXZuQ05RTE9jZ3RKRGhfVG9rZW46T2FMZGJBZnhob2VRZnF4TVVpeGNpaUhibktlXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

# **4.搭建视频**

哔哩哔哩：<https://www.bilibili.com/video/BV1itcwexEbk/>

# **5.搭建方式**

## **5.1 安装docker和docker-compose**

安装教程：[服务器上安装docker和docker-compose教程](https://blog.lcayun.com/3204.html)

## **5.2 安装Grafana**

Grafana下载地址：<https://grafana.com/grafana/download>

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGY2ZmU2ZjlmMTI0Njc3ZTI1MGE0NTQ5ZDM5MmE5ODVfVklTVm1ZS2NLSG9oanJWSHQ0UkJPNXMyZENqV2tRS2ZfVG9rZW46V3pwYWJxckR3b1lSOUx4QjV6SmNFOFVpblhnXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

我们这里使用docker搭建

**复制

```plain&#x20;text
docker run -d --name=grafana -p 3000:3000 grafana/grafana-enterprise
```

docker拉取完就安装完成了

访问Grafana，浏览器输入IP:端口，如：服务器IP:3000

默认账号密码都为：admin

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmY1OGQxM2Y0YTc5NWE4M2FmNzUxNDAxMGEzYmVjMmRfcEw5RWN2Z1VyeDUwRlNNRmc0aWNLV1JIRzd6bWJ6YWRfVG9rZW46Q0lzV2JFbmtJbzc5azV4QXVkdWNWQmhvbnVlXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

第一次登录需要修改密码，这个按照自己需求修改登录密码

进去之后首页

## **5.3 安装Prometheus**

依次下载**Prometheus**镜像包



```plain&#x20;text
docker pull prom/node-exporter
docker pull prom/prometheus
```

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=NDllZDcwMGRmYWZjNWFlYzQ0ZmUxNmNiZjIzM2YxNWJfV3VtYks3VXBKeGRZR1FzUlNKMHV4bVIwMWtlMHJRdUxfVG9rZW46UUI1amJ1bzl3b0o2UjR4d0lOR2NubnFlbnVXXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

运行node-exporter的docker



```plain&#x20;text
docker run -d -p 9100:9100  -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro"  --net="host"  prom/node-exporter
```

访问 http://服务器:9100/metrics 看看是否有数据

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTg4MDZkZTAxMzdhZjVlNjU4MWQ5NDEyZGE3YWJhZmJfU29jTHZlRjRTTU1mQ3l6RFMyYUxuRlhrWkdsN1ppeUdfVG9rZW46S2QzbWJLRXZib0tLeGx4RzBHS2NuSVo2bkJmXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

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



![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=NTk3YmZlYTFlN2RhMGIyMzA4Nzk3YzVkZDk4OTU4MDdfbHpDbXNqb3ptWmZmVFRla2RUN0RXTUk4bDBGUlY1U3RfVG9rZW46S1NoSmJDV3VJb3M5T0R4OGtueGNSMDdEbllnXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

显示绿色UP则为运行正常

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=MWRmZGUyMWVmNWVjZTliMWQ0ZTJkMDhlNjc3YThiY2NfalNqR0RCVzJwU2hiYXFtNWdCamJYUlppYzR6NkZlN2lfVG9rZW46V0EyRGJ1c211b1d1QXB4TXBFdmNZOVFYblZlXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

## **5.5 Grafana添加数据源**



回到 http://服务器IP:3000/ Grafana添加prometheus数据源



![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=OGM4MzlkNmRiZWQ2MDQ2YTU0MzlhZTYwOWJlYWJhMmNfVjY3UWdrNkdFenNxWGhoREhPVDhHa2dsTVhCRTYxakdfVG9rZW46U2JrY2JQTW8wb3VlRDd4RGV1Q2M5a1NabmdiXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

选择prometheus

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWU1OWYxMWJjNWQ4NTJjOWFlY2I1YWFhZjc1NzliMzRfQks5Z1k0ZDJrOU5VajJDYTdHMldWd2t1YzlVR0xoeHNfVG9rZW46SHVXM2JJQ1RUb1dJUnZ4VVlIdGNKSWh5bnBkXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

此行输入 http://服务器IP:9090 后滑下去直接点击save保存

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=MWRlMWVhMjQzMTdkZGI3YmNiOTNhNDUwNTAyMzg5ZGFfVXBKNXNWc0sxUXdqdkQ3M1RJbWFIUEhCVWhHcWZoQ2ZfVG9rZW46VWtNMmJabWpLb1A5eVd4cGtSOGM0NHZKbjhmXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

## **5.6 Grafana添加面板**

回到首页点击DASHBOARDS

选择导入

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=YmU0ZjNlMDcwY2YzOTVhYTdmZTEzNmE5YzI4MjcxYWFfeEVmc2FkTEQ5bUVVdlM3eWliU3FhUmY3Z1pvWXhRUE9fVG9rZW46WlBVc2JhSTBMb3pJanB4WlhlcmNzS2x6bjdmXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

访问官方面板下载地址：<https://grafana.com/grafana/dashboards>

此处输入项目ID

![](https://yanyubao.feishu.cn/space/api/box/stream/download/asynccode/?code=NmE0NjU2Nzc0YzI1NWVkYTU0YzgzYzMwY2ZmMGU2ZWJfNmdvUW9BMUhjbHc0U1pKc2hsVlJLelpheEJNZ2FOWGJfVG9rZW46RjlSc2JsRldDbzZpdW54QmZ3SGN6UE5Xbk9iXzE3MzkxOTAxNjk6MTczOTE5Mzc2OV9WNA)

比如选中项目之后可以复制这个项目的ID填上去

填好ID后我们需要选择prometheus，然后点击import导入即可

这样面板信息就出来了

# **6.结尾**

Prometheus是个非常好用的开源项目，在本文中，我们介绍了什么是Prometheus，如何安装Prometheus，以及使用Prometheus的Pull(拉取)模式来采集Linux服务器资源，并在Grafana进行展现。
