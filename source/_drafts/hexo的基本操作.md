
### **知识归纳在这里**


250222.总结hexo制作github pages

https://yanyubao.feishu.cn/docx/MprrdxevnoRGE6xVJrAcsj7EnAd




使用的模板的操作说明：
【Hexo，基于nodejs，似乎可以，模板Fluid】
https://hexo.fluid-dev.com/docs/start/


### **主要命令**

```bash
hexo clean # 清除缓存文件 (db.json) 和已生成的静态文件 (public)。
hexo generate # 生成静态文件。
hexo server # 启动服务器。

# 启动的默认网址是：
# http://localhost:4000/


# 本地启动用于调试服务
hexo server -l
hexo server -l --watch

# 将飞书上复制过来的Markdown文件的图片，转存到sm.ms
# 先启用插件 Cloud Document Converter，将飞书文档转为markdown
python.exe .\test_markdown_picgo.py 

# 清空缓存
hexo clean    

# 生成静态文件，到public目录
hexo generate --debug


```

### 安装hexo

```bash
# 查看Nodejs的版本
node -v
#目前是 16.20.0

#安装hexo
npm install -g hexo-cli

#安装依赖库
npm i
```



### **CICD过程**

*[2026.01.07.]*

项目可以自动部署到GitHub pages，是因为项目的目录下有文件：
C:\dev_github\jycpp.github.io\.github\workflows\pages.yml
这里定义了具体部署的过程。

### **备份图床**

*[2026.02.01.]*

代码： C:\dev\yanyubao_python\abot_app\test_markdown_backup_sm_ms.py

数据： C:\dev\yanyubao_python\abot_app\sm_ms_images.db

图片： C:\dev\yanyubao_python\abot_app\sm_ms_images

### **404文件的跳转**

*[2026.03.05.]*

修改404.ejs文件，完善跳转到主页的功能。
1. 404.ejs文件是在layout.ejs文件中定义的，用于处理404错误，通过标签“<%- body %>”将404.ejs文件的内容插入到layout.ejs文件中。
2. page404 是在 _config.yml 文件中定义的，用于指定404错误时的跳转页面。
3. 404.html文件应该位与public目录下，与index.html文件在同一级目录，并由apache或者nginx在网站配置中指定类似这样的配置“ErrorDocument 404 /404.html”。
4. 本项目没有直接在source目录下定义404.html文件，而是在pages.js文件中动态生成的，目的是为了适应多语言，具体代码如下：

```javascript
// generate 404 page
if (!fs.existsSync(path.join(hexo.source_dir, '404.html'))) {
  hexo.extend.generator.register('_404', function(locals) {
    if (this.theme.config.page404.enable !== false) {
      return {
        path  : '404.html',
        data  : locals.theme,
        layout: '404'
      };
    }
  });
}
```

以下是网址替换规则：

```plaintext
例如：

https://jycpp.github.io/26-01-03-%E7%90%86%E8%A7%A3CPP%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84%E4%BD%8D%E4%B8%8E%E8%BF%90%E7%AE%97%E7%AC%A6.html


跳转到：

https://jycpp.github.io/26/26-01-03-%E7%90%86%E8%A7%A3CPP%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84%E4%BD%8D%E4%B8%8E%E8%BF%90%E7%AE%97%E7%AC%A6.html


而：

http://localhost:4000/25-10-18-%E6%98%A5%E7%A7%8B%E6%97%A0%E4%B9%89%E6%88%98%E4%BD%86%E6%88%98%E6%88%98%E4%B9%89%E5%BD%93%E5%85%88.html


跳转到：

http://localhost:4000/25/25-10-18-%E6%98%A5%E7%A7%8B%E6%97%A0%E4%B9%89%E6%88%98%E4%BD%86%E6%88%98%E6%88%98%E4%B9%89%E5%BD%93%E5%85%88.html


即在绝对路径前面加上对应的数字和“/”。​
```



