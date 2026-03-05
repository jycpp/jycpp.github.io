---
title: Gitlab中这烦人的Prometheus
date: 2024-08-05 14:30:00
comments: true
tags:
- GitLab
- Prometheus
- 监控
- Docker
- 磁盘空间
- 配置
- 运维
- 数据保留
- 容器
- 性能优化
categories:
- 软件开发
- 运维管理
---


## 一、问题由来


我们在使用 GitLab 时，发现 Prometheus 服务在 `/var/opt/gitlab/prometheus/data` 目录下占用了大量空间，这可能是由于 GitLab 配置中的 `prometheus['enable'] = true` 被注释掉了，导致 Prometheus 服务默认启用并产生数据。

因为我们的Gitlab是运行在Docker容器中的，所以这个问题隐蔽性较强，如果不注意很难被发现。因为最近总是出现服务器磁盘空间被占满的情况，通过“du -sm”分析磁盘空间使用，逐级查询，发现集中在这两个目录下：

- /var/lib/docker/containers
- /var/lib/docker/overlay2

从名字不难看出，这两个目录是Docker容器的存储目录，其中包含了所有容器的文件系统，第一个是Docker用来存储容器文件的，第二个是Docker用来存储镜像文件的。

结合最近出问题的总是Gitlab这个镜像，初步判断占用30多个GB的可能是Gitlab的容器，于是通过命令行“docker container exec -it xxxxx /bin/bash”进入容器，通过“du -sm”分析磁盘空间使用，发现占用空间主要集中在/var/opt/gitlab/prometheus/data目录下，这是Prometheus服务的存储目录。


## 二、原因分析和解决方案

查询Gitlab的配置文件gitlab.rb，发现prometheus['enable'] = true被注释掉了，而在 GitLab 中，即使 `prometheus['enable'] = true` 被注释掉，Prometheus 仍可能默认启用并产生数据，这是因为 GitLab 的默认配置行为导致的。


### **为什么 Prometheus 仍在运行？**
GitLab 的 Omnibus 包默认会启用大部分监控组件（包括 Prometheus），**除非明确禁用**。当 `prometheus['enable'] = true` 被注释时，GitLab 会使用其**内置的默认值**（通常为 `true`），而不是根据注释行来判断。


### **如何正确禁用 Prometheus？**
要完全禁用 Prometheus 及其数据收集，需在 `gitlab.rb` 中**取消注释并明确设置为 `false`**：

```ruby
# 完全禁用 Prometheus 服务
prometheus['enable'] = false

# 同时禁用相关的监控 exporter
node_exporter['enable'] = false
postgres_exporter['enable'] = false
redis_exporter['enable'] = false
gitlab_exporter['enable'] = false
```

修改后，重新配置并重启 GitLab：
```bash
gitlab-ctl reconfigure
gitlab-ctl restart
```


### **验证 Prometheus 是否已禁用**
检查服务状态：
```bash
gitlab-ctl status prometheus
# 输出应为：down (process is not running)
```

确认端口未被监听：
```bash
ss -tulpn | grep 9090  # Prometheus 默认端口
# 若禁用成功，无输出
# 如果没有ss命令，可以使用netstat命令：
netstat -ant
# 需要注意，在bash下的netstat命令可能支持的参数没有那么。
```


### **清理已有数据**
禁用后，删除现有 Prometheus 数据以释放空间：
```bash
# 停止所有 GitLab 服务
gitlab-ctl stop

# 删除 Prometheus 数据目录
rm -rf /var/opt/gitlab/prometheus/data

# 重新启动 GitLab
gitlab-ctl start
```


### **为什么默认启用 Prometheus？**
GitLab 团队推荐保留监控组件，以便：
1. 提供系统性能指标
2. 支持 GitLab 内置的监控仪表盘
3. 为自动扩展提供数据支持

如果你的环境不需要这些功能（如测试环境或资源受限的服务器），可安全禁用。


### **其他节省空间的方法**
如果仍需保留监控但想减少磁盘使用，可调整保留策略：
```ruby
# 减少数据保留时间（如 30 天 -> 7 天）
prometheus['storage_retention'] = "7d"

# 增加采样间隔（默认 15s -> 60s）
prometheus['scrape_interval'] = "60s"
```

通过以上配置，既能控制磁盘使用，又能保留必要的监控数据。




## 三、处理Prometheus占用空间的其他方法

在 GitLab 容器中，Prometheus 数据目录 `/var/opt/gitlab/prometheus/data` 占用空间过大是常见问题，还可以通过以下几种方法解决：

### **1. 调整 Prometheus 数据保留策略**

通过修改 GitLab 配置，减少数据保留时间和采样频率：

```bash
# 进入 GitLab 容器
docker exec -it gitlab bash

# 编辑 GitLab 配置
vi /etc/gitlab/gitlab.rb
```

添加/修改以下配置：

```ruby
prometheus['storage_retention'] = "7d"  # 默认 15d，改为 7 天
prometheus['scrape_interval'] = "60s"   # 增加采样间隔（默认 15s）
prometheus['evaluation_interval'] = "60s"
```

保存后重新配置并重启 GitLab：

```bash
gitlab-ctl reconfigure
gitlab-ctl restart
```


### **2. 手动清理旧数据**

停止 Prometheus 服务，删除旧数据：
```bash
gitlab-ctl stop prometheus

# 删除旧数据（保留最近 7 天）
find /var/opt/gitlab/prometheus/data -type d -mtime +7 -exec rm -rf {} \;

gitlab-ctl start prometheus
```


### **3. 迁移数据到外部存储**

将 Prometheus 数据目录挂载到外部大容量存储：

```bash
# 创建外部存储目录
mkdir -p /mnt/gitlab-prometheus-data

# 停止 GitLab
docker stop gitlab

# 修改 Docker 运行命令，添加数据卷挂载
docker run \
  -v /mnt/gitlab-prometheus-data:/var/opt/gitlab/prometheus/data \
  # 其他原有参数...
  gitlab/gitlab-ce:latest
```


### **4. 禁用不必要的监控**

如果不需要详细监控数据，可以禁用部分服务：

```ruby
# /etc/gitlab/gitlab.rb
prometheus['enable'] = false  # 完全禁用 Prometheus
# 或仅禁用特定监控
node_exporter['enable'] = false
postgres_exporter['enable'] = false
redis_exporter['enable'] = false
```


### **5. 定期执行数据压缩**
Prometheus 支持自动压缩旧数据，但可通过调整参数优化：

```ruby
# /etc/gitlab/gitlab.rb
prometheus['tsdb_retention_compaction'] = "15d"  # 压缩超过 15 天的数据
```


### **6. 升级到最新版本**

新版本的 GitLab 通常会优化 Prometheus 存储效率，建议升级到最新稳定版。


### **7. 使用监控数据汇总工具**

对于长期趋势分析，考虑使用 `promtool` 或第三方工具导出并汇总历史数据，然后删除原始数据。


### **注意事项**

- **数据备份**：操作前务必备份 `/var/opt/gitlab` 目录。
- **性能影响**：减少采样频率可能影响短期监控精度。
- **持久化存储**：使用外部存储时，确保目录权限正确（通常为 `git:git`）。

通过以上方法，可有效控制 Prometheus 数据占用空间，同时保持必要的监控功能。


## 四、禁用Prometheus服务

在 GitLab 中，`prometheus_monitoring['enable']` 和 `prometheus['enable']` 是两个不同的配置项，它们的作用和影响范围有所不同：


### **1. `prometheus['enable']`（主开关）**
- **作用**：完全控制 Prometheus 服务本身的启停。
- **影响**：
  - 启用时，Prometheus 会收集 GitLab 所有组件的监控数据。
  - 禁用时，Prometheus 服务停止，不再收集数据，相关目录（如 `/var/opt/gitlab/prometheus/data`）不再增长。
- **配置示例**：
  ```ruby
  prometheus['enable'] = false  # 彻底禁用 Prometheus
  ```


### **2. `prometheus_monitoring['enable']`（客户端开关）**
- **作用**：控制 GitLab 应用本身是否向 Prometheus 暴露监控指标。
- **影响**：
  - 启用时，GitLab 各组件（如 Rails、Sidekiq）会向 Prometheus 提供指标数据。
  - 禁用时，GitLab 不再暴露自身指标，但 **Prometheus 服务仍会运行**，可能继续收集其他来源的数据（如 Node Exporter）。
- **配置示例**：
  ```ruby
  prometheus_monitoring['enable'] = false  # 仅禁用 GitLab 自身的监控指标
  ```


### **3. 两者的关系**
- **若仅禁用 `prometheus_monitoring['enable']`**：
  - Prometheus 服务仍在运行，会继续占用内存和磁盘空间。
  - `/var/opt/gitlab/prometheus/data` 目录仍会增长（除非同时禁用其他 exporters）。

- **若禁用 `prometheus['enable']`**：
  - Prometheus 服务停止，所有监控数据收集终止。
  - 磁盘空间不再增长，已存在的数据可手动删除。


### **4. 如何正确释放磁盘空间？**
若需彻底释放 `/var/opt/gitlab/prometheus/data` 的空间，**必须禁用 `prometheus['enable']`**：

```ruby
# /etc/gitlab/gitlab.rb
prometheus['enable'] = false  # 停止 Prometheus 服务
prometheus_monitoring['enable'] = false  # 可选：禁用 GitLab 指标暴露

# 若不需要其他 exporters，也可禁用
node_exporter['enable'] = false
postgres_exporter['enable'] = false
redis_exporter['enable'] = false
```

然后执行：
```bash
gitlab-ctl reconfigure
gitlab-ctl restart
rm -rf /var/opt/gitlab/prometheus/data  # 删除现有数据
```


### **总结**
- **`prometheus_monitoring['enable'] = false`**：仅停止 GitLab 应用向 Prometheus 提供指标，但 Prometheus 仍在运行，无法解决磁盘占用问题。
- **`prometheus['enable'] = false`**：彻底停止 Prometheus 服务，是释放 `/var/opt/gitlab/prometheus/data` 空间的必要条件。


