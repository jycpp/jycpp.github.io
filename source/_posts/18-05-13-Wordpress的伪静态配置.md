---
title: Wordpress的伪静态配置：Apache和Nginx两种平台的详细说明
date: 2018-05-13 17:27:20
comments: true
tags:
- WordPress
- 伪静态
- Apache
- Nginx
- mod_rewrite
- .htaccess
- 固定链接
- SEO
- URL重写
- try_files
categories:
- 软件开发
- 网站搭建
---


伪静态是一种通过重写URL，使动态网页在形式上表现为静态网页的技术，有助于提升网站的SEO效果和用户体验。在WordPress中实现伪静态，在Apache和Nginx两种平台上有不同的配置方式。下面将基于文档内容，并以WordPress为例，详细说明在这两种平台上实现伪静态的具体配置。

### Apache平台
1. **检测Apache是否支持mod_rewrite**：
    - 通过PHP的`phpinfo()`函数查看环境配置。在浏览器中访问包含`phpinfo()`函数的PHP文件（如在WordPress的主题文件或插件文件中添加`phpinfo()`函数并访问对应页面，不过在实际部署完成后建议删除该测试代码以保障安全），使用`Ctrl+F`查找到“Loaded Modules”，查看其中是否列出了“mod_rewrite”。若已包含，则说明Apache已经支持`mod_rewrite`，无需继续设置。
    - 若未开启“mod_rewrite”，打开Apache安装目录下的“/apache/conf/httpd.conf”文件。通过`Ctrl+F`查找到“LoadModule rewrite_module”，将前面的“#”号删除。如果没有查找到，则到“LoadModule”区域，在最后一行加入“LoadModule rewrite_module modules/mod_rewrite.so”（必须独占一行），完成后重启Apache服务器。
2. **让Apache服务器支持.htaccess**：
    - 打开Apache目录的“CONF”目录中的“httpd.conf”文件。
    - 查找以下内容：
```apache
Options FollowSymLinks
AllowOverride None
```
    - 将其修改为：
```apache
Options FollowSymLinks
AllowOverride All
```
3. **配置WordPress的.htaccess文件**：
    - 在WordPress网站的根目录下找到`.htaccess`文件（如果没有则创建一个，创建方法如文档中所述，用记事本打开，点击文件 - 另存为，在文件名窗口输入`.htaccess`，注意不要包含英文引号，然后点击保存）。
    - 对于WordPress，常见的伪静态规则如下：
```apache
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME}!-f
RewriteCond %{REQUEST_FILENAME}!-d
RewriteRule. /index.php [L]
```
    - 上述规则解释：
        - `RewriteEngine On`：开启重写引擎。
        - `RewriteBase /`：设置重写的基础目录为网站根目录。
        - `RewriteRule ^index\.php$ - [L]`：如果请求的是`index.php`，则不进行重写，直接处理。
        - `RewriteCond %{REQUEST_FILENAME}!-f`和`RewriteCond %{REQUEST_FILENAME}!-d`：这两个条件表示如果请求的文件或目录不存在。
        - `RewriteRule. /index.php [L]`：如果满足前面的条件，则将请求重定向到`index.php`进行处理。

### Nginx平台
1. **配置Nginx服务器**：
    - 找到Nginx的配置文件，通常位于`/etc/nginx/sites-available/`目录下（不同系统和安装方式可能有所不同），以WordPress所在的虚拟主机配置文件为例（假设为`your_domain.conf`）。
    - 在该配置文件的`location /`块中添加如下伪静态规则：
```nginx
location / {
    try_files $uri $uri/ /index.php?$args;
}
```
    - 上述规则解释：
        - `try_files`指令会按照顺序检查文件或目录是否存在。首先检查请求的`$uri`（即请求的文件路径）是否存在，如果存在则直接返回该文件；如果不存在，则检查`$uri/`（即请求的目录路径）是否存在，如果存在则返回该目录下的默认文件（通常是`index.html`或`index.php`等）；如果前两者都不存在，则将请求重定向到`/index.php?$args`，其中`$args`表示传递原请求中的参数。
2. **重启Nginx服务**：
    - 完成配置修改后，重启Nginx服务使配置生效。在大多数Linux系统中，可以使用以下命令重启Nginx：
```bash
sudo systemctl restart nginx
```

无论是在Apache还是Nginx平台上配置WordPress的伪静态，完成配置后都需要确保WordPress的“固定链接”设置正确。在WordPress后台，依次点击“设置” - “固定链接”，选择合适的固定链接结构（如“文章名”等），保存设置后，伪静态功能就会生效。这样，访问WordPress网站的文章或页面时，就会以伪静态的URL形式呈现，提升网站的访问体验和SEO效果。 
