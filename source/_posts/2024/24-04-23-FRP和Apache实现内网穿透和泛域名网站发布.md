---
title: FRP和Apache实现内网穿透和泛域名网站发布
date: 2024-04-23 12:04:15
comments: true
tags:
- FRP
- Apache
- 内网穿透
- 泛域名网站发布
- 公网服务器
- 内网服务器
- 域名解析
- 代理配置
- 端口冲突
- 安全设置
categories:
- 项目管理
---

本文结合 Apache、FRP 实现内网穿透，并在 Apache 端设置代理代码，同时携带 HTTP 消息报头中的域名信息，通过 FRP 转发到穿透后的网站服务器的具体步骤。对于无论是否有固定IP的客户来说，这样的解决方案都非常有用，因为对于动态IP来说，好多情况下，使用的是SDN（Software Define Network），很难做到在路由器端做端口穿透；而即使你有固定IP，到运营商那里做网站备案，也是很麻烦的事情，因为你需要开发80和443端口，这跟在云服务器提供商那里比起来，例如阿里云、腾讯云、百度云、华为云、AWS等，电信运营商的网站备案流程还不成熟。

![20250220120328](https://s2.loli.net/2025/02/20/RqsChF3KYaVobZI.png)

### 整体思路
- 公网服务器部署 FRP 服务端，监听特定端口。
- 内网服务器部署 FRP 客户端，将内网服务端口映射到公网服务器。
- Apache 服务器（可以是公网服务器或其他具备公网访问条件的服务器）配置代理，将请求转发到 FRP 服务端端口，并携带原始域名信息。

### 具体步骤

#### 1. 公网服务器部署 FRP 服务端
- **下载并解压 FRP**
```bash
wget https://github.com/fatedier/frp/releases/download/v0.48.0/frp_0.48.0_linux_amd64.tar.gz
tar -zxvf frp_0.48.0_linux_amd64.tar.gz
cd frp_0.48.0_linux_amd64
```
- **配置 `frps.ini`**
```ini
[common]
bind_port = 7000  # FRP 服务端监听的端口，客户端会连接此端口
dashboard_port = 7500  # 仪表盘端口，可通过浏览器查看 FRP 状态
dashboard_user = admin  # 仪表盘用户名
dashboard_pwd = admin  # 仪表盘密码
token = your_token  # 用于客户端和服务端通信的认证令牌，自定义
```
- **启动 FRP 服务端**
```bash
./frps -c frps.ini
```
可使用 `nohup` 让其在后台持续运行：
```bash
nohup ./frps -c frps.ini &
```

#### 2. 内网服务器部署 FRP 客户端
- **下载并解压 FRP**
操作同公网服务器。
- **配置 `frpc.ini`**
```ini
[common]
server_addr = your_public_ip  # 公网服务器的 IP 地址
server_port = 7000  # 对应公网服务器的 bind_port
token = your_token  # 与服务端的 token 保持一致

[web]
type = http
local_ip = 127.0.0.1  # 内网网站服务器的 IP
local_port = 80  # 内网网站服务器监听的端口
custom_domains = your_domain.com  # 自定义域名，需要解析到公网服务器 IP
```
- **启动 FRP 客户端**
```bash
./frpc -c frpc.ini
```
使用 `nohup` 让其在后台持续运行：
```bash
nohup ./frpc -c frpc.ini &
```

#### 3. 配置 Apache 代理
- **启用 Apache 代理模块**
在 Apache 服务器上，执行以下命令启用必要的代理模块：
```bash
sudo a2enmod proxy proxy_http
```
- **创建 Apache 虚拟主机配置文件**
```bash
sudo nano /etc/apache2/sites-available/your_domain.conf
```
在文件中添加以下内容：
```apache
<VirtualHost *:80>
    ServerName your_domain.com  # 你的域名

    # 开启代理请求的日志记录
    LogLevel debug proxy:debug

    # 代理配置
    ProxyPreserveHost On  # 保留原始请求的 Host 头信息
    ProxyPass / http://your_public_ip:7000/  # 将请求转发到 FRP 服务端端口
    ProxyPassReverse / http://your_public_ip:7000/  # 处理反向代理响应

    # 日志配置
    ErrorLog ${APACHE_LOG_DIR}/your_domain_error.log
    CustomLog ${APACHE_LOG_DIR}/your_domain_access.log combined
</VirtualHost>
```
**说明**：
    - `ProxyPreserveHost On`：此配置会保留客户端请求中的 `Host` 头信息，确保该信息能被正确转发到内网服务器。
    - `ProxyPass` 和 `ProxyPassReverse`：将所有请求转发到 FRP 服务端监听的端口。

- **启用虚拟主机配置**
```bash
sudo a2ensite your_domain.conf
```
- **重新加载 Apache 配置**
```bash
sudo systemctl reload apache2
```

#### 4. 域名解析
将域名 `your_domain.com` 解析到 Apache 服务器的公网 IP 地址。可在域名解析服务提供商的管理界面添加一条 A 记录，将域名指向 Apache 服务器的 IP。

### 测试访问
在浏览器中输入配置的域名 `your_domain.com`，如果所有配置无误，请求会经过 Apache 代理，携带原始域名信息通过 FRP 转发到内网的网站服务器，你应该能够看到内网网站的内容。

### 注意事项
- **端口冲突**：确保公网服务器和 Apache 服务器上的端口未被其他服务占用。
- **安全设置**：可考虑为 Apache 配置 SSL/TLS 加密，保障数据传输安全。同时，合理设置防火墙规则，仅开放必要的端口。 

