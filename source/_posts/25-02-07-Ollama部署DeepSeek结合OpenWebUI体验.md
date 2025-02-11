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

#### 准备工作

以下步骤也包括让你的电脑安装以下软件：
- Windows或Linux系统系统
- Docker
- Ollama


<iframe style="width:100%; min-height:500px;" src="//player.bilibili.com/player.html?isOutside=true&aid=113961034843590&bvid=BV1rjNbepETo&cid=28257225920&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>



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

![20250211210718](https://s2.loli.net/2025/02/11/M9UIwO3TvVo2P5Q.png)

#### 使用Docker安装和部署OpenWebUI

OpenWebUI是一个开源的网页版LLM，支持多种模型，包括DeepSeek。以下是在Windows系统上使用Docker安装和部署OpenWebUI的步骤：

1. **了解安装方式**：安装OpenWebUI的过程相对复杂，这里借助Docker来快速完成安装和部署。关于使用Docker的详细过程，本文暂不赘述，可在本教程之外另行学习。

2. **运行容器**：下载Docker镜像后，通过以下命令运行容器。在命令中，关键是要指定Ollama的端口和地址。

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

![20250211210609](https://s2.loli.net/2025/02/11/kfEr8hbYgqUvSAJ.png)

OpenWebUI提供了一个简单的网页界面，可用于与LLM进行对话。以下是使用OpenWebUI进行网页对话的步骤：

1. **初始化账号**：在浏览器中输入对应的网址，首次使用时需要进行账号初始化。这里是本地注册，无需联网操作。

2. **开始对话**：完成注册后，在Windows系统上即可通过网页与DeepSeek进行对话。在对话过程中，可在后台查看前端对话日志，方便跟踪和排查问题。

整个本地化部署过程操作简单，对于技术小白来说也不难上手。 