---
title: Ollama部署DeepSeek结合OpenWebUI体验大语言模型
date: 2025-02-07
index_img: https://s2.loli.net/2025/02/11/M9UIwO3TvVo2P5Q.png
comments: true
tags:
- Ollama
- DeepSeek
- OpenWebUI
- 大语言模型
- 本地部署
- Docker
- Windows
- Linux
categories:
- 人工智能AIGC
- 大语言模型
---


在本文中，将详细介绍如何在本地部署DeepSeek，并借助OpenWebUI来体验大语言模型（LLM）。此过程主要会用到Ollama和Docker这两个工具。

### 准备工作

以下步骤也包括让你的电脑安装以下软件：
- Windows或Linux系统系统
- Docker
- Ollama


<iframe style="width:100%; min-height:500px;" src="//player.bilibili.com/player.html?isOutside=true&aid=113961034843590&bvid=BV1rjNbepETo&cid=28257225920&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>


### Ollama的操作

#### 安装Ollama并运行DeepSeek模型

Ollama是一个开源的本地运行的LLM，支持多种模型，包括DeepSeek。以下是在Windows系统上安装和运行DeepSeek的步骤：

1. **下载Ollama**：可以在Ollama的官网下载Windows版本，该软件大小约800M，安装速度通常较快。安装完成后，在电脑右下角会出现Ollama的图标。

2. **下载DeepSeek模型**：前往Ollama的官网找到DeepSeek的页面，由于是演示用途，选择下载最小的模型。下载命令如下：

```bash
ollama run deepseek-r1:1.5b
```

将此命令复制后，在Windows的cmd或powershell中输入。下载大模型所需时间较长，主要取决于网络环境，尤其是连接国际互联网的速度。

3. **启动DeepSeek模型**：模型下载完成后，可通过相同命令启动DeepSeek。再次运行时，无需重新下载模型。启动命令同样为：

```bash
ollama run deepseek-r1:1.5b
```

此时，可进行简单的英文和中文对话测试，DeepSeek在处理这两种语言的对话时表现良好。

![命令行的Ollama](https://s2.loli.net/2025/02/11/M9UIwO3TvVo2P5Q.png)


### Ollama使用问题问题：

#### WebUI访问不了Ollama

默认只可以本地访问：
http://localhost:11434  或者 http://127.0.0.1:11434

而局域网无法访问：
http://192.168.0.83:11434

需要设置 OLLAMA_HOST 变量，内容为： 0.0.0.0:11434
然后重启Ollama。


![设置Ollama的局域网访问](https://s2.loli.net/2025/06/17/ghM7Qe8NR3rWwaI.png)

以下是设置前后访问的效果：

![Ollama的11434端口](https://s2.loli.net/2025/06/17/5g4Y7UnIapqZWLr.png)

![Ollama的11434端口](https://s2.loli.net/2025/06/17/HUKRjA1l2ymvogs.png)


#### 通过API访问Ollama

```bash
curl http://192.168.0.81:11434/api/generate -d '{
  "model": "deepseek-r1:1.5b",
  "prompt": "天空为什么是蓝色的？"
}'
```


### Open WebUI 的操作


#### 使用Docker安装和部署OpenWebUI

OpenWebUI是一个开源的网页版LLM，支持多种模型，包括DeepSeek。以下是在Windows系统上使用Docker安装和部署OpenWebUI的步骤：

1. **了解安装方式**：安装OpenWebUI的过程相对复杂，这里借助Docker来快速完成安装和部署。关于使用Docker的详细过程，本文暂不赘述，可在本教程之外另行学习。

2. **运行容器**：下载Docker镜像后，通过以下命令运行容器。在命令中，关键是要**指定Ollama的端口和地址**。我们这里指定为： http://192.168.0.81:11434

```bash
docker run -d \
-p 3000:8080 \
-v /opt/ollama/open-webui:/app/backend/data \
-e HF_ENDPOINT=https://hf-mirror.com \
-e OLLAMA_BASE_URL=http://192.168.0.81:11434 \
-e DEFAULT_MODELS=deepseek-r1:1.5b \
--name open-webui \
--restart always \
ghcr.io/open-webui/open-webui:main
```

上述命令运行后，Web服务会在端口3000启动。

#### 使用OpenWebUI进行网页对话

![Open WebUI 的操作界面](https://s2.loli.net/2025/02/11/kfEr8hbYgqUvSAJ.png)

OpenWebUI提供了一个简单的网页界面，可用于与LLM进行对话。以下是使用OpenWebUI进行网页对话的步骤：

1. **初始化账号**：在浏览器中输入对应的网址，首次使用时需要进行账号初始化。这里是本地注册，无需联网操作。

2. **开始对话**：完成注册后，在Windows系统上即可通过网页与DeepSeek进行对话。在对话过程中，可在后台查看前端对话日志，方便跟踪和排查问题。

整个本地化部署过程操作简单，对于技术小白来说也不难上手。 


#### Open WebUI的API和开发者token

Open WebUI 提供了 API 接口，用于与模型进行交互。

Open WebUI API 类似Ollam，这里不赘述。

以下是直接使用 Ollama API 进行对话的示例：

```bash
import requests

# 定义 API 接口 URL
api_url = "http://192.168.0.81:11434/api/generate"  # 替换为你的 Open WebUI 地址

# 定义请求参数
params = {
    "model": "deepseek-r1:1.5b",  # 模型名称
    "prompt": "天空为什么是蓝色的？",  # 对话内容
    "max_new_tokens": 50,  # 生成的最大新令牌数
    "temperature": 0.7,  # 温度参数，控制随机性
    "top_p": 0.9,  #  nucleus sampling 参数，控制生成文本的多样性
    "top_k": 40,  # top-k sampling 参数，控制生成文本的多样性
    "stop": ["\n"],  # 停止生成的标记列表
    "stream": True,  # 启用流式传输，实时获取生成内容
    "developer_token": "your_developer_token",  # 开发者 token
}

# 发送 POST 请求
response = requests.post(api_url, json=params, stream=True)  # 启用流式传输

# 处理流式响应
for line in response.iter_lines():
    if line:
        data = json.loads(line.decode('utf-8')[6:])  # 解析 JSON 数据
        if 'response' in data:
            print(data['response'], end='', flush=True)  # 实时打印生成内容
```


#### 基本Open WebUI开发智能体

具体的操作可以在Workspace（工作空间）找到，这里提供对于模型、知识库、提示词和工具的管理，其中“工具”是基于Python做开发的。

设置好工具后，可以在“智能体”中添加，添加后即可在Open WebUI中使用。


### 补充：Ollama的基础知识

以下是 Ollama 的常用命令分类整理，涵盖模型管理、服务控制、日志查看等核心操作，结合多个来源综合整理：

---

#### ⚙️ 1、模型管理命令
| **命令** | **功能** | **示例** |
|----------|----------|----------|
| `ollama pull <模型名>` | 下载模型 | `ollama pull llama3` |
| `ollama list` | 查看已安装模型列表 | `ollama list` |
| `ollama rm <模型名>` | 删除模型（⚠️ 可能残留文件需手动清理） | `ollama rm deepseek-r1:14b` |
| `ollama run <模型名>` | 运行模型（首次自动下载） | `ollama run llama3 --gpu` |
| `ollama cp <源模型> <新模型>` | 复制模型 | `ollama cp llama3 my-llama3` |
| `ollama show <模型名>` | 查看模型详细信息 | `ollama show qwen2.5` |

> 💡 **注意**：  
> - 彻底删除模型文件需手动清理目录（默认路径）：  
>   - **Windows**: `%USERPROFILE%\.ollama\models\`  
>   - **Linux/macOS**: `~/.ollama/models/`  
> - GPU 加速需添加 `--gpu` 参数（如 `ollama run deepseek-r1:7b --gpu`）。

---

#### 🖥️ 2、服务控制命令
| **命令** | **功能** | **示例** |
|----------|----------|----------|
| `ollama serve` | 启动后台服务（默认端口 `11434`） | `ollama serve` |
| `ollama stop` | 停止运行中的模型 | `ollama stop` |
| `ollama ps` | 查看正在运行的模型 | `ollama ps` |
| `OLLAMA_HOST=127.0.0.1:11434` | 限制服务仅本地访问（安全加固） | 前置环境变量使用 |

---

#### 📝 3、日志查看命令
| **系统** | **命令/路径** | **说明** |
|----------|---------------|----------|
| **macOS** | `cat ~/.ollama/logs/server.log` | 直接输出日志内容 |
| **Linux** | `journalctl -u ollama` | Systemd 系统日志 |
| | `cat /var/log/ollama/server.log` | 直接查看日志文件 |
| **Windows** | 资源管理器输入 `%LOCALAPPDATA%\Ollama` | 打开日志目录查看 `server.log` |
| **通用** | `ollama logs` | 查看实时运行日志（部分版本支持） |

> 🔍 **调试技巧**：  
> - 启用详细日志：设置环境变量 `OLLAMA_DEBUG="1"`（Windows: PowerShell）  
> - 实时跟踪日志：Linux/macOS 用 `tail -f ~/.ollama/logs/server.log`。

---

#### 🛠️ 4、系统与配置管理
| **命令** | **功能** | **示例** |
|----------|----------|----------|
| `ollama version` | 查看版本信息 | `ollama version` |
| `ollama help` | 查看帮助文档 | `ollama help` |
| `export OLLAMA_MODELS=/new/path` | 修改模型存储路径（避免占系统盘） | 写入 Shell 配置文件生效 |
| `ollama cleanup` | 清理缓存（部分版本支持） | `ollama cleanup` |

---

#### 💎 5、总结建议
1. **高频场景**：  
   - 下载运行：`ollama run <模型>`（自动下载+启动交互）  
   - 释放空间：先 `ollama rm`，再手动删除残留文件  
   - 多模型切换：`ollama list` 查看后选择运行。
2. **安全提示**：  
   - 避免公网暴露服务：启动前设置 `OLLAMA_HOST=127.0.0.1`  
   - 定期更新版本：`ollama update`（部分系统需重装）。
3. **扩展应用**：  
   - API 调用：服务启动后访问 `http://localhost:11434/v1/completions` 类 OpenAI 接口  
   - 自定义模型：通过 `Modelfile` 创建（`ollama create -f ./Modelfile`）。

> 更多命令详见 [Ollama 官方文档](https://ollama.com) 或社区教程（如模型性能测试 `ollama perf`）。







