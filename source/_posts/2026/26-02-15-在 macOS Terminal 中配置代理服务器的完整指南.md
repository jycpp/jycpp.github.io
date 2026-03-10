---
title: 在 macOS Terminal 中配置代理服务器的完整指南
date: 2026-02-15 10:30:00
comments: true
tags:
- macOS
- Terminal
- 代理服务器
- SOCKS5
- HTTP 代理
- 环境变量
- Shell 配置
- Zsh
- Bash
- 网络配置
categories:
- 软件开发
- MacOS
---


在 macOS 系统中配置网络代理有多种方式，其中在 Terminal（终端）中设置代理是开发者常用的方法。本文将详细介绍三种不同层级的代理配置方案，从全局系统设置到临时的 Terminal 会话，再到永久的环境变量配置，帮助您根据实际需求选择最合适的方法。

## 一、全局系统代理设置（最简单但限制多）

### 配置步骤
1. 打开**系统设置**
2. 进入**网络** > **Wi-Fi** > 选择当前连接的 Wi-Fi（如“4DX”）
3. 点击**详细信息** > **代理**
4. 根据需要配置 **SOCKS 代理** 或 **HTTP/HTTPS 代理**
   - 地址：`127.0.0.1`
   - 端口：`10808`（V2Ray 默认）或 `7891`（ClashX 默认）

### 注意事项
- ✅ **优点**：配置一次，对所有应用程序（包括 Terminal）生效
- ⚠️ **限制**：部分命令行工具可能不遵循系统代理设置
- 🔧 **验证方法**：在 Terminal 中执行 `curl https://httpbin.org/ip`，查看返回的 IP 是否为代理服务器 IP

**如果全局设置对 Terminal 无效，请继续阅读下文的手动配置方法。**

## 二、Terminal 会话级代理设置（临时有效）

当您只需要在当前 Terminal 会话中使用代理时，可以通过环境变量快速配置。这种方法在关闭 Terminal 窗口后自动失效。

### 1. 根据代理客户端设置对应变量

**V2Ray 用户**（默认 SOCKS5 端口 10808）：
```bash
# 设置 SOCKS5 代理
export ALL_PROXY=socks5://127.0.0.1:10808
```

**ClashX 用户**（默认 SOCKS5 端口 7891）：
```bash
# 设置 SOCKS5 代理
export ALL_PROXY=socks5://127.0.0.1:7891
```

**HTTP 代理用户**（如有需要）：
```bash
# 设置 HTTP/HTTPS 代理
export HTTP_PROXY=http://127.0.0.1:10809
export HTTPS_PROXY=http://127.0.0.1:10809
```

### 2. 关键细节说明
- **变量名大小写敏感**：在 macOS 的 Terminal 中，`ALL_PROXY`（大写）是标准的环境变量名
- **兼容性设置**：为增加兼容性，可同时设置小写变量：
  ```bash
  export all_proxy=socks5://127.0.0.1:10808
  export ALL_PROXY=socks5://127.0.0.1:10808
  ```

### 3. 验证与诊断
设置完成后，请立即验证配置是否生效：

```bash
# 查看当前 IP 地址（如显示代理服务器 IP 则表示成功）
curl https://httpbin.org/ip

# 查看环境变量值
echo $ALL_PROXY
echo $all_proxy

# 测试网络连通性
curl -I https://www.google.com
```

### 4. 管理临时代理设置
```bash
# 删除代理设置（当前会话恢复直连）
unset ALL_PROXY
unset all_proxy

# 或设置为空值
export ALL_PROXY=
```

## 三、永久代理配置（对所有新 Terminal 会话生效）

如果您希望每次打开 Terminal 都自动应用代理设置，需要将配置添加到 shell 的配置文件中。

### 1. 确定当前使用的 Shell
```bash
# 查看当前 Shell 类型
echo $SHELL
```
- 输出 `/bin/zsh`：使用 Zsh（macOS Catalina 及之后版本的默认 shell）
- 输出 `/bin/bash`：使用 Bash

### 2. 编辑配置文件
根据您的 Shell 类型编辑对应的配置文件：

**Zsh 用户**：
```bash
# 编辑配置文件
nano ~/.zshrc

# 在文件末尾添加
export ALL_PROXY=socks5://127.0.0.1:10808
export all_proxy=socks5://127.0.0.1:10808
```

**Bash 用户**：
```bash
# 编辑配置文件
nano ~/.bash_profile

# 在文件末尾添加相同内容
export ALL_PROXY=socks5://127.0.0.1:10808
export all_proxy=socks5://127.0.0.1:10808
```

### 3. 应用配置
保存文件后，执行以下命令使配置立即生效：

```bash
# Zsh
source ~/.zshrc

# Bash
source ~/.bash_profile
```

## 四、高级技巧与故障排除

### 1. 灵活的代理开关函数
在配置文件中添加以下函数，可快速切换代理状态：

```bash
# 添加到 ~/.zshrc 或 ~/.bash_profile
function proxy_on() {
    export ALL_PROXY=socks5://127.0.0.1:10808
    export all_proxy=socks5://127.0.0.1:10808
    echo "✅ 代理已启用: $ALL_PROXY"
}

function proxy_off() {
    unset ALL_PROXY
    unset all_proxy
    echo "✅ 代理已禁用"
}

function proxy_status() {
    if [ -n "$ALL_PROXY" ]; then
        echo "🟢 代理状态: 启用 ($ALL_PROXY)"
    else
        echo "⚪️ 代理状态: 禁用"
    fi
}
```

使用方法：
```bash
proxy_on    # 启用代理
proxy_off   # 禁用代理
proxy_status # 查看状态
```

### 2. 端口冲突排查
如果代理不生效，请检查端口是否正确：

```bash
# 查看端口监听状态
lsof -i :10808
lsof -i :7891

# 测试端口连通性
curl --socks5 127.0.0.1:10808 https://httpbin.org/ip
```

### 3. 工具特异性配置
某些工具有自己的代理配置，需要单独设置：

**Git 代理**：
```bash
# 设置 Git 使用代理
git config --global http.proxy socks5://127.0.0.1:10808
git config --global https.proxy socks5://127.0.0.1:10808

# 取消 Git 代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

**npm 代理**：
```bash
# 设置 npm 代理
npm config set proxy http://127.0.0.1:10809
npm config set https-proxy http://127.0.0.1:10809
```

## 五、配置方案对比

| 配置方式 | 生效范围 | 持久性 | 适用场景 |
|---------|---------|--------|---------|
| 系统网络设置 | 全局所有应用 | 永久 | 希望所有网络流量都走代理 |
| Terminal 环境变量 | 当前 Terminal 会话 | 临时 | 临时需要代理的命令行操作 |
| Shell 配置文件 | 所有新 Terminal 会话 | 永久 | 经常需要在命令行中使用代理 |

## 六、常见问题解答

**Q：为什么设置了代理，但 `ping` 命令仍然不走代理？**
A：`ping` 使用 ICMP 协议，而 SOCKS5/HTTP 代理通常只支持 TCP/UDP 协议。这是正常现象。

**Q：如何为单个命令设置代理而不影响环境？**
A：使用命令前缀：
```bash
ALL_PROXY=socks5://127.0.0.1:10808 curl https://httpbin.org/ip
```

或者使用 `curl` 的 `-x` 选项：

```bash
curl -x socks5://127.0.0.1:10808 https://httpbin.org/ip
```

**Q：如何查看代理设置的端口已经开启？**
A：您可以使用 `lsof` 命令查看端口监听情况：
```bash
lsof -i :10808
```

或者使用 `netstat` 命令：
```bash
netstat -an | grep 10808
netstat -antp | grep 10808
```

**Q：使用curl查询当前IP地址的时候会返回什么？**
A：返回当前IP地址。以 httpbin。org/ip 为例，返回结果类似：
```json
{
    "origin": "192.168.1.1"
}
```

**Q：代理设置冲突怎么办？**
A：遵循优先级：命令行参数 > 环境变量 > 系统设置。使用 `unset` 清除环境变量可避免冲突。

## 总结

在 macOS Terminal 中配置代理是一个灵活且强大的功能。通过本文介绍的三种方法，您可以根据需要选择：
1. **系统全局设置**：适合希望一劳永逸的用户
2. **Terminal 会话变量**：适合临时、偶尔使用代理的场景
3. **Shell 配置文件**：适合经常在命令行中工作的开发者

建议从临时配置开始，确认代理工作正常后，再考虑将其添加到配置文件中。正确配置代理后，您可以在 Terminal 中更顺畅地使用 `git`、`npm`、`brew` 等需要访问境外资源的工具。


