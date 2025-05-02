---
title: Edge和Chrome插件：图片批量下载器
date: 2024-07-19 17:27:20
comments: true
tags:
- Chrome插件
- Edge插件
- 图片下载
- 右键菜单
- manifest.json
- background.js
- content.js
- 消息传递
- 浏览器插件
- 批量下载
- 图片收集
- 瀑布流布局
- 多语言支持
- 响应式设计
- 错误处理
categories:
- 软件开发
---


自从Fatkun这个图片批量下载插件发布以来，我一直在使用它来下载网页上的图片。它的功能非常强大，不仅可以自动收集网页上的图片，还可以通过滑动条来过滤图片的尺寸，满足不同的需求。但是最近这半年来，这个插件开始耍流氓了，强制设置浏览器主页，最近直接被Edge浏览器插件市场红色标记为“此扩展包含恶意软件”，于是决定自己动手写一个插件，下载页面上的所有图片，完美替代fatkun图片批量下载。使用方法：在网页上点击右键选择“下载页面上的所有图片”即可。


![chrome图片批量下载插件](https://s2.loli.net/2025/05/02/snHTDjB7QMdt6b2.jpg)


下载地址：
[Edge浏览器插件 - 图片批量下载器](https://microsoftedge.microsoft.com/addons/detail/%E5%9B%BE%E7%89%87%E6%89%B9%E9%87%8F%E4%B8%8B%E8%BD%BD%E5%99%A8/ibnlbjklnjigaoelkblpngbnmelahkhn?hl=zh-CN)


![Chrome插件-下载页面图片01](https://s2.loli.net/2025/03/19/UcZFvGDlJ4O8pEH.jpg)

![Chrome插件-下载页面图片02](https://s2.loli.net/2025/03/19/SpgyKqICJm4Oxun.jpg)


### 1 开发需要满足的功能


#### **一、核心功能**
1. **右键菜单触发**  
   - 右键点击网页任意位置，显示“下载页面上的所有图片”选项。
   - 支持通过 `manifest.json` 配置菜单项，避免重复创建 ID。

2. **图片资源收集**  
   - 自动分析当前页面所有图片资源，包括：
     - `<img>` 标签的 `src` 属性。
     - `<a>` 标签中以 `.jpg`、`.png`、`.jpeg`、`.gif` 结尾的 `href` 链接。
   - 过滤无效链接，确保仅收集有效图片资源。

3. **图片信息展示**  
   - 在新标签页中展示所有图片，每个图片包含：
     - **名称**：自动生成（支持 MD5 重命名）。
     - **大小**：占位符（后续可扩展 API 获取）。
     - **宽高**：通过 `naturalWidth` 和 `naturalHeight` 获取真实像素值。


#### **二、增强功能**
1. **文件名智能处理**  
   - 当文件名满足以下任一条件时，使用 MD5 生成文件名：
     - 长度超过 100 字符。
     - 包含 `?`、`&`、`=` 等特殊字符。
   - 无扩展名时自动追加 `.jpg`。

2. **过滤与筛选**  
   - 支持通过滑动条动态过滤图片宽高（最小宽度/最小高度）。
   - 实时更新列表，仅显示符合条件的图片。

3. **多选与批量下载**  
   - 左上角提供“全选”和“全不选”按钮。
   - 右上角“下载选中图片”按钮，支持将图片保存至以当前页面标题命名的文件夹。


#### **三、界面优化**
1. **瀑布流布局**  
   - 使用 CSS Grid 或多列布局实现瀑布流效果，每行固定列数（如 5 列）。
   - 图片容器宽度固定，高度自适应，避免变形。

2. **多语言支持**  
   - 通过 `_locales` 目录和 `messages.json` 实现界面语言自适应浏览器设置。
   - 支持英文、中文等多语言切换。

3. **响应式设计**  
   - 滑动条标签明确标注“最小宽度”和“最小高度”。
   - 按钮和输入框布局清晰，操作便捷。


#### **四、兼容性与稳定性**
1. **错误处理**  
   - 避免重复创建右键菜单项，使用标记机制确保唯一性。
   - 消息传递添加延迟或确认机制，确保目标上下文存在。

2. **资源加载优化**  
   - 通过 `Promise.all` 等待图片加载完成后再渲染，避免宽高获取不准确。
   - 使用 `crypto-js` 库计算 MD5 值，确保文件名安全。


#### **五、扩展与定制**
1. **可配置性**  
   - 通过 `manifest.json` 灵活配置权限、图标和菜单项。
   - 支持通过 `content.js` 扩展图片收集逻辑。

2. **扩展性**  
   - 预留接口支持更多图片格式（如 `.webp`）。
   - 可扩展图片大小获取功能（通过 `fetch` 获取响应头）。


#### **功能规划总结**
该插件通过右键菜单触发，实现了图片资源的智能收集、过滤、展示与批量下载。结合 MD5 文件名处理、瀑布流布局、多语言支持等特性，提供了高效且用户友好的图片下载体验，适用于需要快速获取网页图片的场景。


### 2 开发过程

#### 插件的目录结构

![20250319170657](https://s2.loli.net/2025/03/19/2sHRTaBjiSlzN7Q.png)


#### Chrome插件的项目特点

Chrome插件运行的配置文件是manifest.json，它定义了插件的名称、版本、权限、图标、背景脚本、内容脚本等信息。而background.js是插件的后台脚本，它负责插件的初始化、消息传递、事件监听等功能。content.js是插件的内容脚本，它负责在网页中注入JS代码，实现插件的功能。

#### 插件的消息传递

插件的消息传递是通过chrome.runtime.sendMessage()和chrome.runtime.onMessage.addListener()实现的。其中，chrome.runtime.sendMessage()用于发送消息，chrome.runtime.onMessage.addListener()用于监听消息。


#### 插件的权限

插件的权限是通过manifest.json中的permissions字段定义的。permissions字段定义了插件需要访问的权限，例如：

```json
"permissions": [
  "activeTab",
  "storage",
  "contextMenus"

]
```



#### 右键菜单触发

右键菜单触发是插件的核心功能，通过监听右键点击事件，在点击位置创建右键菜单项。

```js
// 监听右键点击事件
document.addEventListener('contextmenu', function (event) {
  // 创建右键菜单项
  createContextMenu(event);
});

// 创建右键菜单项
function createContextMenu(event) {
  // 检查是否已创建菜单项
  if (contextMenuCreated) {
    return;
  }

  // 创建菜单项
  const menuItem = document.createElement('div');
  menuItem.textContent = '下载页面上的所有图片';
  menuItem.addEventListener('click', function () {
    // 处理菜单项点击事件
    handleContextMenuClick(event);
  });

  // 添加菜单项到文档
  document.body.appendChild(menuItem);

  // 标记已创建菜单项
  contextMenuCreated = true;

  // 监听文档点击事件，关闭右键菜单
  document.addEventListener('click', function () {
    closeContextMenu();
  });

}
```


### 3 常见问题

**为什么采集到的页面上的图片数量不够？**

这个问题可能是由于页面上的图片资源没有完全加载完成，导致无法获取到所有图片资源。可以通过以下方法解决：

在Edge或者Chrome浏览器中，确保要下载的图片在浏览器的内容栏中出现过。在下载之前，等待页面上的图片资源加载完成。

**如何自适应浏览器的语言？**

这个问题可能是由于浏览器的语言设置没有正确加载导致的。可以通过以下方法解决：

在Edge或者Chrome浏览器中，点击浏览器右上角的三个点，选择“设置”。在设置中，选择“语言”，确保浏览器的语言设置正确。

在本项目的代码中，支持English和Chinese两种语言，暂时不支持其他语言。

**如何在Edge浏览器中安装插件？**

这个问题可能是由于Edge浏览器的版本问题导致的。可以通过以下方法解决：

在Edge浏览器中，点击浏览器右上角的三个点，选择“设置”。在设置中，选择“扩展程序”，确保“开发者模式”已经开启。