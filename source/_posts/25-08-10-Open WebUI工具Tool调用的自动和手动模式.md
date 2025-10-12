---
title: Open WebUI工具Tool调用的自动和手动模式
date: 2025-08-10 14:35:22
comments: true
tags:
- Open WebUI
- 大模型
- API接口
- 工具注册
- 智能体
- 天气查询
categories:
- 软件开发
- 大语言模型
---


基于上文《Open WebUI开发高级接口参数应用》的知识，我们可以进一步了解Open WebUI的工具调用模式。

你的理解非常正确！在实际使用中，Open WebUI确实会**自动完成函数调用的全流程**（模型生成指令→调用工具后端→整理结果），无需程序员手动解析`model_resp`和调用工具API。之前的示例展示手动步骤，是为了拆解底层逻辑，让你看清“黑盒”内部的工作原理。

下面明确两种模式的区别，并补充**自动调用工具的具体配置和代码**，帮你理清实际应用中的流程：


### 1 Open WebUI的两种工具调用模式
#### 1.1 自动模式（生产环境常用）
- **核心特点**：Open WebUI内置了工具调用的“中间层”，会自动处理：  
  - 模型生成的`function_call`指令解析；  
  - 调用工具后端API（如天气服务）；  
  - 将工具返回的结果回传给模型，生成最终回答。  
- **程序员无需干预**：只需确保工具函数已正确注册（包含后端地址），并在API请求中启用工具调用开关。

#### 1.2 手动模式（调试/定制化场景）
- **核心特点**：程序员手动解析模型生成的`function_call`，自行调用工具后端，再将结果返回给模型。  
- **用途**：用于调试工具调用逻辑、定制特殊的调用规则（如添加权限校验、日志记录），或在Open WebUI未支持自动模式的旧版本中使用。  


### 2 自动模式：让Open WebUI自动完成工具调用（实际使用方式）
要启用自动模式，只需在Open WebUI中完成**工具注册配置**，并在API请求中添加`tools: true`参数（或在UI中启用工具）。以下是具体步骤：


#### 步骤1：注册工具时配置“自动调用”
在Open WebUI中注册`get_weather`函数时，需确保：  
- 工具的后端地址（`servers.url`）正确且可被Open WebUI访问（如`http://host.docker.internal:5000`，避免跨域问题）；  
- 在工具详情页开启“Auto-invoke”（自动调用）开关（不同版本可能叫“Enable Auto Call”）。  

此时，Open WebUI会记住工具的后端地址，并自动处理调用逻辑。


#### 步骤2：发送API请求时启用工具调用
只需在请求中添加`tools: true`（告知Open WebUI“允许自动调用工具”），无需手动处理`function_call`：

```python
import requests
import uuid

API_URL = "http://localhost:3000/api/chat/completions"
API_KEY = "你的API密钥"
headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

# 用户提问："上海的天气如何？"
payload = {
    "model": "llama-3.2",  # 支持函数调用的模型
    "messages": [
        {
            "id": uuid.uuid4().hex,
            "role": "user",
            "content": "上海的天气如何？",
            "parentId": None
        }
    ],
    "tools": true,  # 关键：启用自动工具调用
    "stream": False
}

# 发送请求后，Open WebUI会自动完成全流程
response = requests.post(API_URL, json=payload, headers=headers)
final_answer = response.json()["messages"][0]["content"]
print(f"自动调用工具后的回答：{final_answer}")
# 输出：上海当前天气为晴，气温25℃。
```


#### 自动模式的底层逻辑（Open WebUI内部处理）
当`tools: true`时，Open WebUI会在内部执行以下步骤（无需程序员参与）：  
1. 模型生成`function_call`指令（`{"name": "get_weather", "parameters": {"city": "上海"}}`）；  
2. Open WebUI的中间层解析指令，调用`http://localhost:5000/get_weather`（工具注册时配置的地址）；  
3. 接收天气服务返回的`{"temperature": 25, "condition": "晴"}`；  
4. 将结果回传给模型，生成自然语言回答；  
5. 返回最终结果给用户。  


### 3 为什么需要了解手动模式？
手动模式虽然不常用，但能帮你：  
1. **调试问题**：当自动调用失败时（如工具后端超时、参数错误），可手动解析`function_call`排查问题；  
2. **定制化需求**：例如在调用工具前添加权限校验（如“仅VIP用户可查询天气”），或记录调用日志；  
3. **理解原理**：看清模型、工具、后端之间的交互链路，遇到复杂场景时能更灵活地扩展。  


### 4 总结
- **实际使用**：优先用自动模式，通过`tools: true`和正确注册工具，让Open WebUI自动完成调用；  
- **原理学习**：手动模式用于拆解流程，理解`function_call`的生成、工具调用、结果反馈的全链路。  

两种模式的核心差异在于“谁来处理`function_call`指令”——自动模式由Open WebUI处理，手动模式由程序员处理。掌握这一点，就能灵活应对不同场景了。


