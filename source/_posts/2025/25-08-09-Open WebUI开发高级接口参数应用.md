---
title: Open WebUI开发高级接口参数应用
date: 2025-08-09 14:35:22
comments: true
tags:
- Open WebUI
- 大模型
- 缓存优化
- 函数调用
- system提示词
- API接口
- 智能体
- 工具调用
categories:
- 软件开发
- 大语言模型
---


Open WebUI开发过程中，掌握一些高级接口参数，可以帮助我们提升大模型调用性能和能力。本文主要总结以下知识点：

1. "model_status_cache"可以设置缓存时间，以提高首次对话时候模型的加载速度。
2. 消息包中messages 中通过 parentId 和 childrens id 来实现连续对话。
3. AI对话过程中，通过 call function 来调用各种工具，例如查询天气等。
4. 通过 model input 参数的 system 属性来设置系统级的提示词，相当于设置一个智能体的角色。


## 1 高级应用开发要点

### 1.1 模型加载与性能优化：缓存机制的应用

1. **模型状态缓存（`model_status_cache`）**
   为提升模型列表加载速度，Open WebUI支持通过配置`model_status_cache`减少对底层模型服务（如Ollama）的频繁查询。例如，在`config.json`中设置缓存有效期（如`ttl: 300`，即5分钟），可避免重复检查模型状态（如是否运行、元数据是否完整），尤其适用于本地部署的大模型（如70B级量化模型），显著降低首次对话的等待时间。

2. **数据库与Redis优化**
   - 模型列表、用户权限等高频查询数据可缓存至Redis，通过`redis`配置项（如`host: localhost`、`port: 6379`）实现，减少数据库压力。
   - 数据库需合理设计索引（如`models`表的`owner_id`、`status`字段），避免全表扫描，优化多表关联查询（如用户-角色-模型权限的联动校验）。


### 1.2 连续会话的实现：对话链与上下文维护

1. **`messages`数组与双向链表结构**
   连续会话依赖`messages`数组中的`id`、`parentId`和`childrenIds`字段构建对话链：
   - `parentId`指向当前消息的前一条消息ID，`childrenIds`存储后续消息ID，形成类似双向链表的结构，确保上下文连贯性。
   - 每次请求需携带完整历史消息（包含`user`和`assistant`角色的内容），模型基于全量上下文生成响应，避免服务端无状态导致的上下文丢失。

2. **流式响应与Token管理**
   通过`stream: true`参数启用流式响应，逐行接收模型输出（格式为`data: {"content": "部分内容"}`），提升交互实时性。同时需注意`max_tokens`参数控制单次响应长度，避免超过模型上下文限制（如GPT-4 Turbo的128k Token）。


### 1.3 工具调用与自定义函数：扩展模型能力边界

1. **函数调用的核心流程**
   模型可通过`function call`机制调用外部工具（如天气查询、数据库操作），需通过以下步骤实现：
   - **定义函数**：按OpenAPI 3.0规范编写JSON/YAML文档，描述函数名称（如`get_weather`）、参数（如`city`）、返回值（如温度、天气状况）及后端服务地址（`servers.url`）。
   - **注册函数**：在Open WebUI的`Admin Panel → Tools → Functions`中导入函数文档，配置触发条件（如关键词“天气”）和权限（如允许特定用户组调用）。
   - **执行与反馈**：用户提问触发模型生成函数调用指令（如`{"function": "get_weather", "parameters": {"city": "上海"}}`），后端解析并调用对应服务，将结果返回模型后整理为自然语言回答。

2. **典型应用场景**
   - 动态获取实时数据（如股票行情、新闻）；
   - 集成企业内部系统（如CRM查询、订单处理）；
   - 通过工具函数实现“固定提示词预设”（如`set_fixed_prompt`函数存储系统提示，后续对话自动注入）。


### 1.4 系统提示词与智能体：定义模型的“身份与规则”

1. **`system`属性的作用**
   `model input`中的`system`属性用于设置系统级提示词，定义模型的角色、行为规则（如“用简洁中文回答”“避免专业术语”），是区分不同“智能体”的核心标志之一。例如：
   - 若`system`设为“你是数学教师，需详细推导解题步骤”，模型会以“数学教师”身份响应；
   - 若`system`设为“你是电商客服，优先解决订单问题”，模型则表现为“客服智能体”。

2. **与智能体（Agents）的关联与差异**
   - **关联**：`system`提示词是智能体的“基础身份定义”，决定其核心行为逻辑。
   - **差异**：完整的智能体（如Coze平台的Agents）还包含工具集、记忆机制、工作流规则等（如“先调用工具获取数据，再生成报告”），而`system`仅为其中的角色配置部分。


### 1.5 API接口与权限管理：安全与可控的访问

1. **核心API端点**
   - 聊天补全：`POST /api/chat/completions`（兼容OpenAI格式，支持流式响应和RAG）；
   - 模型管理：`GET /api/models`（获取模型列表）、`POST /api/models/pull`（拉取新模型）；
   - 知识库操作：`POST /api/documents`（上传文档）、`POST /api/reindex`（重新生成索引）。

2. **认证与权限控制**
   - 通过`Authorization: Bearer YOUR_API_KEY`进行身份验证，API密钥在`设置 → 账户`中生成；
   - 管理员可在`Admin Panel → Users & Permissions`中为用户/角色分配权限（如允许访问的模型、工具调用权限），确保操作可控。


### 1.6 总结

Open WebUI通过缓存优化提升性能，依赖对话链维护连续会话，借助函数调用扩展工具能力，通过`system`提示词定义智能体角色，同时提供完整的API接口与权限管理，兼顾灵活性与安全性。这些特性使其既能作为轻量对话工具，也能扩展为企业级智能体平台，适配从简单问答到复杂业务集成的多样化需求。


## 2 示例代码

### 2.1 连续对话的实现（基于`messages`数组与对话链）
连续对话需通过`id`、`parentId`和`childrenIds`维护上下文，每次请求携带完整历史消息。

#### 2.1.1. 首次对话（初始化对话链）
```python
import requests
import uuid

API_URL = "http://localhost:3000/api/chat/completions"
API_KEY = "your_openwebui_api_key"  # 从Open WebUI的「设置→账户」获取

# 生成唯一消息ID（可使用uuid）
first_user_id = uuid.uuid4().hex

# 首次请求：发送用户消息，parentId为null
payload = {
    "model": "llama-3.2",  # 目标模型
    "messages": [
        {
            "id": first_user_id,
            "role": "user",
            "content": "什么是Open WebUI？",
            "parentId": None,  # 首次消息无父节点
            "childrenIds": []
        }
    ],
    "stream": False  # 非流式响应
}

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# 发送请求
response = requests.post(API_URL, json=payload, headers=headers)
first_resp = response.json()

# 提取助手响应（假设响应格式包含messages字段）
assistant_msg = first_resp["messages"][0]
print(f"助手首次回复：{assistant_msg['content']}")
```


#### 2.1.2. 延续对话（携带历史上下文）
```python
# 生成第二次用户消息ID
second_user_id = uuid.uuid4().hex

# 历史消息链：包含首次用户消息和助手响应
history_messages = [
    # 首次用户消息
    {
        "id": first_user_id,
        "role": "user",
        "content": "什么是Open WebUI？",
        "parentId": None,
        "childrenIds": [assistant_msg["id"]]  # 关联助手响应ID
    },
    # 首次助手响应
    assistant_msg,
    # 第二次用户消息（新消息）
    {
        "id": second_user_id,
        "role": "user",
        "content": "它支持自定义函数吗？",
        "parentId": assistant_msg["id"],  # 父节点为上一条助手响应
        "childrenIds": []
    }
]

# 第二次请求：携带完整历史
payload = {
    "model": "llama-3.2",
    "messages": history_messages,
    "stream": False
}

response = requests.post(API_URL, json=payload, headers=headers)
second_resp = response.json()
print(f"助手二次回复：{second_resp['messages'][0]['content']}")
```

**关键说明**：  
- 每次请求的`messages`必须包含完整历史（用户+助手消息），通过`parentId`和`childrenIds`关联；  
- 实际开发中需用`uuid`生成唯一`id`，确保对话链不冲突。


### 2.2 工具与函数调用（以天气查询为例）

需先在Open WebUI注册`get_weather`函数，再通过对话触发模型调用。

注册的操作是通过浏览器访问Open WebUI的`Admin Panel → Tools & Functions`，点击`Add Function`，填写函数名称、后端地址（天气服务API）、参数格式（JSON）。

在不同的WebUI的版本中，以上菜单可能会有区别。


#### 2.2.1 注册函数的OpenAPI文档（`weather_api.json`）

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "天气查询API",
    "version": "1.0.0"
  },
  "paths": {
    "/get_weather": {
      "post": {
        "operationId": "get_weather",
        "description": "查询城市实时天气",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "city": {
                    "type": "string",
                    "description": "城市名称（如北京、上海）"
                  }
                },
                "required": ["city"]
              }
            }
          }
        },
        "responses": {
          "200": {
            "description": "天气信息",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "temperature": {"type": "number"},
                    "condition": {"type": "string"}
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "servers": [{"url": "http://localhost:5000"}]  # 天气服务后端地址
}
```


以上代码中，`/get_weather`路径下的`post`方法定义了天气查询的接口，接收`city`参数，返回`temperature`和`condition`。实际的请求会被转发到天气服务后端（如`http://localhost:5000/get_weather`）。



#### 2.2.2 模型调用函数的对话示例

天气后端的具体代码实现的示例，我们用Python Flask框架编写一个简单的天气服务后端。

```python
# 用户提问："查询上海的天气"
user_msg_id = uuid.uuid4().hex
payload = {
    "model": "llama-3.2",  # 需支持函数调用的模型
    "messages": [
        {
            "id": user_msg_id,
            "role": "user",
            "content": "查询上海的天气",
            "parentId": None,
            "childrenIds": []
        }
    ],
    "functions": [  # 显式传入已注册的函数（部分版本需手动指定）
        {
            "name": "get_weather",
            "parameters": {"city": "上海"}
        }
    ],
    "stream": False
}

response = requests.post(API_URL, json=payload, headers=headers)
function_call = response.json()

# 假设模型返回函数调用指令
print("模型生成的函数调用：", function_call["function_call"])
# 输出示例：{"name": "get_weather", "parameters": {"city": "上海"}}


# 后端执行函数（调用天气服务）
import json
weather_backend_url = "http://localhost:5000/get_weather"
weather_response = requests.post(
    weather_backend_url,
    json={"city": "上海"}
)
weather_data = weather_response.json()  # 假设返回：{"temperature": 25, "condition": "晴"}


# 将函数结果返回给模型，生成自然语言回答
final_payload = {
    "model": "llama-3.2",
    "messages": [
        # 历史用户消息
        {
            "id": user_msg_id,
            "role": "user",
            "content": "查询上海的天气",
            "parentId": None,
            "childrenIds": [function_call["id"]]
        },
        # 函数调用结果（作为工具返回）
        {
            "id": function_call["id"],
            "role": "function",
            "content": json.dumps(weather_data),
            "parentId": user_msg_id,
            "childrenIds": []
        }
    ],
    "stream": False
}

final_response = requests.post(API_URL, json=final_payload, headers=headers)
print("最终回答：", final_response.json()["messages"][0]["content"])
# 输出示例："上海当前天气为晴，气温25℃。"
```

**关键说明**：  
- 模型通过分析用户输入生成`function_call`指令；  
- 后端需解析指令并调用实际服务，再将结果返回模型生成自然语言。


### 2.3 `system`参数的使用（定义智能体角色）
`system`参数用于设置全局规则，定义模型的“身份”和行为方式。

```python
# 示例1：设置"简洁回答"智能体
payload_simple = {
    "model": "llama-3.2",
    "system": "你是一个简洁回答助手，用不超过20字回应用户。",  # system提示词
    "messages": [{"role": "user", "content": "什么是Open WebUI？"}],
    "stream": False
}
response_simple = requests.post(API_URL, json=payload_simple, headers=headers)
print("简洁回答：", response_simple.json()["messages"][0]["content"])
# 输出示例："Open WebUI是开源的大模型交互平台。"


# 示例2：设置"详细解释"智能体
payload_detail = {
    "model": "llama-3.2",
    "system": "你是技术顾问，需详细解释技术概念，包含核心功能。",
    "messages": [{"role": "user", "content": "什么是Open WebUI？"}],
    "stream": False
}
response_detail = requests.post(API_URL, json=payload_detail, headers=headers)
print("详细回答：", response_detail.json()["messages"][0]["content"])
# 输出示例："Open WebUI是一款开源的大模型交互平台，支持多模型集成（如Ollama、OpenAI）、对话历史管理、工具函数调用等功能，可通过API实现自动化交互..."
```

**关键说明**：  
- `system`参数优先级高于用户消息，直接影响模型的回答风格和深度；  
- 结合工具函数时，`system`可定义调用工具的规则（如“优先调用天气API获取实时数据”）。


### 2.4 总结
以上示例直观展示了：  
- 连续对话通过`messages`数组的`id`关联维护上下文；  
- 工具调用需通过OpenAPI定义函数，由模型生成调用指令并执行；  
- `system`参数通过预设规则定义模型的“智能体角色”，控制回答风格。  

实际开发中需根据Open WebUI版本调整参数格式，并处理流式响应、错误重试等细节。

