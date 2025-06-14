
**知识归纳在这里**

https://yanyubao.feishu.cn/docx/MprrdxevnoRGE6xVJrAcsj7EnAd
250222.总结hexo制作github pages


使用的模板的操作说明：
【Hexo，基于nodejs，似乎可以，模板Fluid】
https://hexo.fluid-dev.com/docs/start/


**主要命令**

```bash
hexo clean # 清除缓存文件 (db.json) 和已生成的静态文件 (public)。
hexo generate # 生成静态文件。
hexo server # 启动服务器。

# 启动的默认网址是：
# http://localhost:4000/


本地启动用于调试服务
hexo server -l
hexo server -l --watch

将飞书上复制过来的Markdown文件的图片，转存到sm.ms
python.exe .\test_markdown_picgo.py 

清空缓存
hexo clean    

生成静态文件，到public目录
hexo generate --debug

**新电脑上安装环境的命令**
#查看Nodejs的版本
node -v
#目前是 16.20.0

#安装hexo
npm install -g hexo-cli

#安装依赖库
npm i

```



