---
title: 分析nodejs中的express和http-server
date: 2021-06-16 13:57:00
comments: true
tags:
- Node.js
- express
- http-server
- Web服务
- 静态资源
- Sass编译
- npm脚本
- 配置对比
- Vite
- 项目开发
categories:
- 软件开发
- 网站建设
---



## http-server 启动web服务

我们做vue项目开发的时候，一般是通过vue-cli创建项目，然后通过vue-cli-service serve启动项目。

但是，在nodejs项目中，我们一般是通过npm run start启动项目，这个时候，我们需要在package.json中配置启动脚本。

一般情况下，package.json的配置如下：

```json
{
  "scripts": {
    "start": "node server.js",  // 启动nodejs服务
    "dev": "vite",
    "build": "npm run build:prod",
    "build:prod": "vue-tsc --noEmit && vite build --mode production",
    "build:sit": "vue-tsc --noEmit && vite build --mode production.sit",
    "build:uat": "vue-tsc --noEmit && vite build --mode production.uat",
    "serve": "npm run build && vite preview"
  }
}
```

其中负责创建Web服务的是node命令，他会执行一个JavaScript文件并创建临时的web服务，然后将当前目录下的文件作为静态文件提供给浏览器。

vite也是一样的原理，执行vite命令的时候，vite会自动查找当前目录下的index.html文件，然后将当前目录下的文件作为静态文件提供给浏览器。具体的工作过程如下：

1. 入口文件识别
Vite会优先查找项目根目录的index.html作为应用入口（可通过vite.config.js的root配置修改根目录）。这个HTML文件中的

```html
<script type="module">
```

标签会被识别为项目的JS入口点。

2. 模块处理
对于HTML中引入的.js/.ts/.vue等模块文件，Vite会启动ES模块服务器（基于原生ES模块导入），按需编译并热更新这些文件（HMR），而不是直接读取磁盘文件。

3. 静态资源服务
静态资源（图片/字体/JSON等）的处理分为两种情况：

  - 通过import或require引入的资源：会被Vite作为模块处理（可能重命名、压缩），路径会被自动解析；
  - 放置在public目录下的资源：会直接复制到输出目录（或开发服务器根路径），保持原文件名，可通过绝对路径直接访问（如/image/logo.png）。


而有一些场景，比如Sass文件的编译，HTML不需要编译，可以直接使用，我们需要在本地预览项目，这个时候，如何不依赖于Apache、Nginx等外部服务器启动web服务n呢？

这种情况下，如果需要在本地预览项目（即启动一个临时web服务器托管静态文件），需要额外安装工具并配置脚本。例如：

安装轻量级静态服务器工具：

```bash
npm install -g http-server
```

在package.json中添加启动脚本：

```package.json
{
  "scripts": {
    "start": "http-server ./ -p 8081"  // 在8081端口启动服务器，托管当前目录
  }
}
```

启动服务：

```bash
npm run start  // 会创建并启动web服务，可通过http://localhost:8081访问项目
```

总结：sass:build是纯构建命令，启动web服务需要额外配置。


## 补充冷知识：编译Sass文件

要将项目中的Sass文件编译为生产环境可用的静态CSS文件，需要先安装Sass编译器（推荐使用Dart Sass，官方维护）：

```bash
# 全局安装（需先安装Node.js）
npm install -g sass
```

根据项目NGINX配置和代码上下文，推测Sass相关目录结构如下：

```plaintext
apc-pc-web/
├─ sass/          # 源Sass文件目录（存放.scss/.sass文件）
├─ css/           # 编译后CSS输出目录（需提前创建）
└─ script/        # JS文件（当前提供的web_public.js所在目录）
```

在项目根目录执行以下命令，将Sass文件编译为CSS：

```bash
sass --style=compressed sass/:css/
```

- --style=compressed：指定输出样式为压缩模式（生产环境推荐）
- sass/:：源目录（所有.scss/.sass文件）
- css/：输出目录（保持与源文件相同的子目录结构）


开发过程中实时监听Sass文件修改并自动编译（输出未压缩的易读CSS）：

```bash
sass --watch --style=expanded sass/:css/
```

- --watch：监听文件变化自动重新编译
- --style=expanded：输出未压缩的易读格式（开发环境推荐）


编译完成后，css/目录下会生成与Sass文件同名的CSS文件（如sass/main.scss → css/main.css）。

只需将整个项目目录（含css/、html/、script/等）部署到NGINX配置的路径/data/website/sass-web/下即可，NGINX会自动托管/css/目录的静态文件。


在项目根目录创建package.json（若不存在），添加编译脚本：

```json
{
  "name": "apc-pc-web",
  "version": "1.0.0",
  "scripts": {
    "sass:build": "sass --style=compressed sass/:css/",  // 生产构建
    "sass:watch": "sass --watch --style=expanded sass/:css/"  // 开发监听
  }
}
```

之后可通过以下命令执行：

```bash
# 生产构建
npm run sass:build

# 开发监听
npm run sass:watch
```





## node+express 启动web服务


nginx 配置如下，在这个项目中，html文件都在 “/html”这个目录下，不是根目录，而加载css和script资源的时候，又需要读取根目录下的子目录。



```nginx
           location / {
             alias /data/website/sass-web/html/;
             index index.html;
           }

           location /script/ {
             alias /data/website/sass-web/script/;
           }
           location /framework/ {
             alias /data/website/sass-web/framework/;
           }
           location /css/ {
             alias /data/website/sass-web/css/;
           }
           location /image/ {
             alias /data/website/sass-web/image/;
           }
           
           location /sass/ {
             alias /data/website/sass-web/sass/;
           }
           location /lang/ {
             alias /data/website/sass-web/lang/;
           }           
           location /proper/ {
             alias /data/website/sass-web/proper/;
           }
```



这个时候，http-server就无法实现这样复杂的配置了，因此需要用express创建自定义静态服务器。

```bash
npm install express --save-dev  # 安装Express框架
```

接下来，我们就可以基于express创建一个自定义静态服务器，用于托管静态文件。加速名称为express-server.js。


```js
const express = require('express');
const app = express();
const port = 8081;

// 1. 优先挂载具体路径的中间件（如/script/、/css/等）
const staticDirs = ['script', 'framework', 'css', 'image', 'sass', 'lang', 'proper'];
staticDirs.forEach(dir => {
  app.use(`/${dir}/`, express.static(dir));  // 匹配/script/、/css/等具体路径
});

// 2. 最后挂载根路径中间件（处理HTML文件）
app.use('/', express.static('html', {
  index: 'index.html'  // 默认加载html/index.html
}));

app.listen(port, () => {
  console.log(`本地服务已启动，访问 http://localhost:${port}`);
});
```

启动这个文件，可以通过以下命令行：

```bash
node express-server.js  # 替代之前的http-server命令
```

配置在package.json中：

```json
{
  "scripts": {
    "start:express": "node express-server.js"  // 启动nodejs服务
  }
}
```

### express实现动态路由

再扩展一点知识，通过express创建的静态服务器，可以支持动态路由，例如：

```js
// 动态路由示例：/user/:id
app.get('/user/:id', (req, res) => {
  const userId = req.params.id;  // 获取动态参数
  res.send(`User ID: ${userId}`);  // 响应动态内容
});

// 动态路由示例：/product/:category/:id
app.get('/product/:category/:id', (req, res) => {
  const category = req.params.category;  // 获取动态参数
  const productId = req.params.id;  // 获取动态参数
  res.send(`Category: ${category}, Product ID: ${productId}`);  // 响应动态内容 
})

```

### nodemon监控js文件变化并重启服务

在Node.js开发中，监控JS文件变化并自动重启服务，推荐使用 nodemon 工具。虽然最新版本的nodejs（Version 18及以上）支持参数“--watch”，但在某些情况下，可能还是需要时间去验证的。

全局安装nodemon：

```bash
npm install -g nodemon
```

如果仅仅在项目中使用，也可以在项目目录下安装：

```bash
npm install nodemon --save-dev
```

在package.json中添加启动脚本：

```json
{
  "scripts": {
    "start:dev": "nodemon server.js"  // 启动nodejs服务
  }
}
```

首次启动：与 node express-server.js 效果一致，正常运行服务。当 express-server.js 或其依赖的JS文件（如路由、中间件）发生修改并保存时，nodemon 会自动终止当前进程并重启服务。

如果指定具体监控的文件或目录，可以通过在项目的根目录下增加配置文件 nodemon.json 来实现。

```json
{
  "watch": ["express-server.js", "src/**/*.js", "common/"],  // 监控src目录下的所有js文件，以及 common下的所有文件
  "ext": "js",  // 仅监控的文件扩展名
  "ignore": ["src/**/*.spec.js"],  // 忽略的文件
  "exec": "node server.js"  // 启动命令
}
```


如果要知道nodemon的具体参数，可以通过以下命令查看：

```bash
nodemon --help
```


