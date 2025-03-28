---
title: Mermaid绘制流程图让Markdown画起来
date: 2023-07-13 17:27:20
comments: true
tags:
- Mermaid
- Markdown
- 流程图
- 时序图
- 类图
- 状态图
- 甘特图
- 饼图
- 可视化
- JavaScript
categories:
- 软件开发
- 系统架构
---


# Mermaid 流程图：用 Markdown 绘制可视化逻辑

作为一名老程序员，我对于markdown格式的文件可谓情有独钟，因为它的简洁性和易读性让我能够快速地编写文档和笔记。然而，在编写技术文档时，我发现markdown的表达能力有限，特别是对于复杂的逻辑流程和图表展示。

为了解决这个问题，我开始寻找一种更强大的工具来支持markdown文档中的图表展示，而Mermaid就是这样的一个工具。但是mermaid的语法相对复杂，而且在不同的平台上的集成方式也存在差异，这给我带来了一些困扰。

随着ChatGPT的火爆，我发现似乎这完全不是问题。当我想让复杂的逻辑关系用mermaid表示出来的时候，我会让ChatGPT先绘制一个版本，然后我在这个版本的基础上修改，效率简直火箭般的提升。

所以，我决定将mermaid的学习和使用经验分享给大家，如果你能借助ChatGPT或者文心一言助力，无疑是生产力爆棚的节奏。


## 一、Mermaid 与 Markdown 的关系

Mermaid 是一种基于文本的可视化工具，通过简单的语法生成流程图、时序图、类图等。它与 Markdown 的结合实现了文档与图表的无缝融合：
- **语法集成**：Mermaid 代码通过 Markdown 的代码块（需要指出代码类型为mermaid）嵌入
- **轻量化**：无需复杂绘图工具，纯文本即可定义图表
- **跨平台**：支持 GitHub、GitLab、VSCode 等主流平台

**JavaScript 的作用**：

Mermaid 图表需要 JavaScript 渲染引擎支持，不同平台的集成方式差异：
| 场景         | 浏览器端                     | VSCode 端                   |
|--------------|------------------------------|----------------------------|
| 渲染方式     | 加载 CDN 脚本                | 使用扩展（如 Mermaid Previewer）|
| 交互性       | 支持动态缩放/导出            | 实时预览                    |
| 依赖管理     | 手动引入脚本                 | 自动处理依赖                |



## 二、集成 Mermaid 的方法

### 1. 浏览器端集成

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
  <style>
    .mermaid { background: white; }
  </style>
</head>
<body>
  <div class="mermaid">
    graph TD
      A[Start] --> B{Decision}
      B -->|Yes| C[End]
      B -->|No| D[Loop]
  </div>
  <script>mermaid.initialize({startOnLoad:true});</script>
</body>
</html>

```




### 2. VSCode 端集成
1. 安装扩展：
   - **Mermaid Previewer**（推荐）
   - **Markdown Preview Enhanced**
2. 快捷键预览：
   - `Ctrl+Shift+V`（Markdown 预览）
   - `Ctrl+K V`（分屏预览）



## 三、Mermaid 核心图表类型

### 1. 流程图（Flowchart）

```mermaid
graph TD
  A[用户登录] --> B{权限验证}
  B -->|管理员| C[显示后台]
  B -->|普通用户| D[显示前台]
  B -->|未通过| E((结束))
```

### 2. 时序图（Sequence Diagram）

```mermaid
sequenceDiagram
  客户端 ->> 服务器: 发送请求
  activate 服务器
  服务器 ->> 数据库: 查询数据
  数据库 -->> 服务器: 返回结果
  服务器 -->> 客户端: 返回响应
  deactivate 服务器
```

### 3. 类图（Class Diagram）

```mermaid
classDiagram
  class User {
    -id: int
    -name: string
    +getProfile(): array
  }
  class Order {
    -orderNo: string
    -amount: float
    +calculateTotal(): float
  }
  User --> Order
```

### 4. 状态图（State Diagram）

```mermaid
stateDiagram
  [*] --> 未支付
  未支付 --> 已支付: 支付成功
  已支付 --> 已发货: 确认发货
  已发货 --> 已完成: 确认收货
  已完成 --> [*]
```

### 5. 甘特图（Gantt Chart）

```mermaid
gantt
  title 项目开发计划
  dateFormat  YYYY-MM
  section 开发阶段
  需求分析       :a1, 2024-01, 30d
  系统设计       :a2, after a1, 20d
  section 测试阶段
  单元测试       :b1, after a2, 15d
  集成测试       :b2, after b1, 10d
```

### 6. 饼图（Pie Chart）

```mermaid
pie
  title 任务分配比例
  "开发" : 45
  "测试" : 30
  "文档" : 15
  "其他" : 10
```

## 四、图表类型详解

### 1. 流程图（Flowchart）
| 符号         | 描述           | 示例代码           | 显示效果          |
|--------------|----------------|--------------------|-------------------|
| `[节点内容]` | 矩形节点       | `A[用户登录]`      | ▢ 用户登录        |
| `((节点))`   | 圆形节点       | `B((结束))`        | ◯ 结束            |
| `[[节点]]`   | 外部系统节点   | `C[[支付网关]]`    | ▢▢ 支付网关       |
| `[(节点)]`   | 数据库节点     | `D[(订单表)]`      | ▢(订单表)         |
| `{节点}`     | 菱形判断节点   | `E{是否通过?}`     | ◇ 是否通过?       |

### 2. 时序图（Sequence Diagram）
```mermaid
sequenceDiagram
  参与者1 ->> 参与者2: 消息1
  参与者2 -->> 参与者1: 消息2
  参与者1 ->> 参与者3: 消息3
  激活 参与者3
  参与者3 --> 参与者2: 消息4
  非激活 参与者3
```

### 3. 类图（Class Diagram）
```mermaid
classDiagram
  class 订单 {
    -id: int
    -amount: float
    +calculateTax(): float
  }
  class 用户 {
    -name: string
    -email: string
  }
  订单 --> 用户
```

### 4. 状态图（State Diagram）
```mermaid
stateDiagram
  [*] --> 草稿
  草稿 --> 已发布: 发布
  已发布 --> 已归档: 归档
  已归档 --> [*]
  已发布 --> 已删除: 删除
```

### 5. 甘特图（Gantt Chart）
```mermaid
gantt
  title 项目进度计划
  dateFormat  YYYY-MM
  section 开发
  需求分析       :a1, 2024-01, 30d
  系统设计       :a2, after a1, 20d
  section 测试
  单元测试       :b1, after a2, 15d
  集成测试       :b2, after b1, 10d
```

### 6. 饼图（Pie Chart）
```mermaid
pie
  title 浏览器市场份额
  "Chrome" : 65
  "Firefox" : 15
  "Edge" : 10
  "Safari" : 5
  "Other" : 5
```

## 五、高级用法示例

### 1. 嵌套子图
```mermaid
graph TD
  subgraph 主流程
    A[开始] --> B{决策}
    B -->|分支1| C[子流程1]
    B -->|分支2| D[子流程2]
    subgraph 子流程1
      C1[步骤1] --> C2[步骤2]
    end
    subgraph 子流程2
      D1[步骤A] --> D2[步骤B]
    end
  end
```

### 2. 并行处理
```mermaid
graph LR
  A[任务1] --> B
  A --> C
  B --> D
  C --> D
  D --> E
```

### 3. 动态交互（浏览器端）

```javascript
mermaidAPI.render('graph', 'graph TD A[动态节点] --> B', function(svgCode) {
  document.getElementById('container').innerHTML = svgCode;
});
```

## 六、最佳实践建议

1. **保持简洁**：避免复杂嵌套，必要时拆分子图
2. **规范命名**：节点名称使用清晰的业务术语
3. **统一方向**：流程图保持一致的流向（如全部 LR 方向）
4. **交互增强**：在浏览器中使用 `mermaidAPI` 实现动态交互
5. **版本控制**：将 Mermaid 代码与文档一同纳入版本管理

通过这种图文结合的方式，技术文档可以实现：
- 逻辑可视化：复杂流程一目了然
- 维护便捷性：文本格式便于版本控制
- 跨平台兼容性：一次编写多处渲染
- 协作友好性：图表与代码同步更新

> **注意事项**：

> 1. 确保渲染引擎版本兼容（Mermaid 8+ 支持更多特性）
> 2. 长文本节点可使用 `<<br>>` 换行
> 3. 复杂图表建议使用 `subgraph` 分组
> 4. 浏览器中可通过 `mermaid.parse()` 方法动态渲染

