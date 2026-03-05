---
title: Open WebUI函数Function的调用步骤
date: 2025-08-10 14:35:22
comments: true
tags:
- Open WebUI
- 工具调用
- 大模型
- 系统提示词
- API接口
- 智能体
- 天气查询
categories:
- 软件开发
- 大语言模型
---


基于上文《Open WebUI开发高级接口参数应用》的知识，本文对Function的定义和调用做更详细的笔记。


Open WebUI的Function调用流程可以分为以下几个步骤：

1. 模型判断是否需要调用函数
2. 模型生成函数调用指令
3. 后端执行函数
4. 函数返回结果
5. 模型整合结果，生成最终回答

在实际开发中，需要注意以下几点：
- 模型判断是否需要调用函数的依据是函数注册信息和用户输入。
- 模型生成的函数调用指令需要符合Open WebUI的格式要求，包括函数名、参数名和参数值。
- 后端根据模型生成的函数调用指令，调用注册的函数，并将函数返回的结果整合到最终回答中。


对于上文中涉及到的function call，我们对其调用步骤做拆解，这部分代码确实涉及多个环节（模型生成调用指令、后端执行函数、结果反馈），容易让人混淆。核心问题在于：**`function_call`不是预先定义的“函数”，而是模型根据用户输入和已注册的工具函数，动态生成的“调用指令”**。下面重新拆解流程，用更清晰的步骤和注释说明：


### 1 先明确核心前提
在模型调用工具函数前，必须先在Open WebUI中完成**函数注册**（这是模型能“知道”有这个函数的前提），步骤如下：  
1. 准备函数定义文件（如前面的`weather_api.json`，描述`get_weather`函数的参数、功能、后端地址）；  
2. 登录Open WebUI管理员账号 → 进入`Admin Panel → Tools → Functions` → 上传`weather_api.json`，完成注册。  

此时，模型已“认识”`get_weather`函数，知道它能查天气、需要`city`参数、调用地址是`http://localhost:5000/get_weather`。


### 2 重新拆解“模型调用函数”的完整流程（带清晰代码注释）
以“用户问‘上海天气’→ 模型调用`get_weather`→ 返回结果”为例，分4步拆解：


#### 2.1 步骤1：用户提问，触发模型判断是否需要调用函数
用户输入“上海的天气如何？”，代码向Open WebUI发送请求，模型会分析：“这个问题需要实时天气数据，我已注册了`get_weather`函数，应该调用它”。

```python
import requests
import uuid

# 基础配置
API_URL = "http://localhost:3000/api/chat/completions"
API_KEY = "你的API密钥"  # 从Open WebUI的「设置→账户」获取
headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

# 1. 用户发送问题
user_input = "上海的天气如何？"
user_msg_id = uuid.uuid4().hex  # 生成唯一消息ID

# 发送请求给Open WebUI，模型将判断是否需要调用函数
payload = {
    "model": "llama-3.2",  # 必须是支持函数调用的模型（如llama-3.2、gpt-4）
    "messages": [
        {
            "id": user_msg_id,
            "role": "user",  # 消息角色：用户
            "content": user_input,
            "parentId": None,  # 首次提问，无父消息
            "childrenIds": []
        }
    ],
    "stream": False  # 关闭流式，一次性获取结果
}

# 发送请求，获取模型的响应（此时模型可能返回函数调用指令）
response = requests.post(API_URL, json=payload, headers=headers)
model_resp = response.json()  # 模型的原始响应
```


#### 2.2 步骤2：模型生成`function_call`指令（核心！）
模型分析用户问题后，会生成一个“调用指令”（即`function_call`），告诉后端：“我需要调用`get_weather`函数，参数是`city=上海`”。  

此时`model_resp`的结构类似这样（模型动态生成的）：
```python
# 模型返回的响应示例（model_resp）
{
    "id": "模型生成的响应ID",
    "messages": [
        {
            "id": "msg_abc123",
            "role": "assistant",  # 模型角色：助手
            "content": "",  # 此时无直接回答，因为需要先调用工具
            "function_call": {  # 核心：模型生成的函数调用指令
                "name": "get_weather",  # 要调用的函数名（必须与注册的一致）
                "parameters": {"city": "上海"}  # 函数参数
            },
            "parentId": user_msg_id,  # 关联用户的问题
            "childrenIds": []
        }
    ]
}
```

代码中提取`function_call`的逻辑：
```python
# 2. 提取模型生成的函数调用指令
function_call = model_resp["messages"][0].get("function_call")
if not function_call:
    print("模型未生成函数调用，直接回答：", model_resp["messages"][0]["content"])
else:
    print(f"模型决定调用函数：{function_call['name']}，参数：{function_call['parameters']}")
    # 输出：模型决定调用函数：get_weather，参数：{'city': '上海'}
```


#### 2.3 步骤3：后端执行函数（调用实际的天气服务）
拿到`function_call`指令后，后端需要：  
- 确认调用的函数是`get_weather`；  
- 提取参数`city=上海`；  
- 发送请求到`weather_api.json`中定义的后端地址（`http://localhost:5000/get_weather`）。  

假设我们已用Flask搭建了天气服务后端（代码见前文），此时调用它：
```python
# 3. 执行函数：调用天气服务后端
if function_call["name"] == "get_weather":
    # 提取参数
    city = function_call["parameters"]["city"]
    # 调用天气服务（地址来自注册的weather_api.json）
    weather_backend_url = "http://localhost:5000/get_weather"
    weather_response = requests.post(
        weather_backend_url,
        json={"city": city}  # 传递参数
    )
    # 获取天气数据（假设返回：{"temperature": 25, "condition": "晴"}）
    weather_data = weather_response.json()
    print(f"天气服务返回结果：{weather_data}")
```


#### 2.4 步骤4：将函数结果返回给模型，生成最终回答
模型本身不会直接处理天气数据，需要将`weather_data`返回给模型，让它整理成自然语言回答：
```python
# 4. 将函数结果返回给模型，生成最终回答
function_result_msg = {
    "id": uuid.uuid4().hex,  # 函数结果消息的ID
    "role": "function",  # 角色：工具返回结果
    "content": str(weather_data),  # 传递天气数据（转为字符串）
    "parentId": model_resp["messages"][0]["id"],  # 关联模型的调用指令
    "childrenIds": []
}

# 构造新的请求，包含历史消息+函数结果
final_payload = {
    "model": "llama-3.2",
    "messages": [
        # 历史：用户的问题
        {
            "id": user_msg_id,
            "role": "user",
            "content": user_input,
            "parentId": None,
            "childrenIds": [model_resp["messages"][0]["id"]]
        },
        # 历史：模型的调用指令
        model_resp["messages"][0],
        # 新增：函数返回的结果
        function_result_msg
    ],
    "stream": False
}

# 发送请求，获取模型整理后的回答
final_response = requests.post(API_URL, json=final_payload, headers=headers)
final_answer = final_response.json()["messages"][0]["content"]
print(f"模型最终回答：{final_answer}")
# 输出：上海当前天气为晴，气温25℃。
```


### 3 为什么之前的代码看起来“混乱”？
因为它把4个步骤的代码合并了，没有明确拆分。核心逻辑其实是：  
`用户提问 → 模型生成调用指令（function_call）→ 后端执行函数 → 结果返回模型 → 模型生成回答`  

其中，**`function_call`是模型动态生成的指令**，不是预先定义的函数，它的作用是“告诉后端该调用哪个工具、传什么参数”。


### 4 一句话总结
1. 先在Open WebUI注册函数（模型才能“认识”它）；  
2. 用户提问时，模型判断需要调用函数，生成`function_call`指令；  
3. 后端按指令调用实际服务，拿到结果；  
4. 结果返回模型，模型整理成自然语言回答。  

这样拆解后，每个步骤的目的就清晰了，代码逻辑也更容易理解。


