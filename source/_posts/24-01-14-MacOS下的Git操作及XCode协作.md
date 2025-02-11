---
title: MacOS下的Git操作及XCode协作
author: [您的名字]
date: 2024-01-14
version: 1.0
comments: true
tags:
- MacOS
- Git操作
- XCode协作
- 版本控制
- Gitee
- 项目管理
- 代码提交
categories:
- 编程开发
- MacOS
---




## 1、项目Git管理



### 1.1 Gitee的同步（本地打包的项目）



在gitee.com上建立Git项目，路径为：

![](https://s2.loli.net/2025/02/11/Tf71SACOehVx6bW.png)





在xcode中选择从Git clone，保存本地路径为：

![](https://s2.loli.net/2025/02/11/oc4rYIPUZhf96XK.png)



在Windows电脑上获取Git上的副本，保存路径为：





### 1.2 git常用命令



**入门指令：**

在目录下创建git：

git init

删除git管理的数据库：

ls -a 查询隐藏的.git目录后，rm -rf .git

添加所有文件到git（仅仅是添加，还没有commit）：

git add .

添加指定文件或文件夹到git

git add aaaaaa



#### **配置git的用户名和email：**

```javascript
git config --global user.name "Your Name" 
git config --global user.email "your.email@example.com"

git config user.name "Your Name"
git config user.email "your.email@example.com"

git config --list

# 你也可以直接编辑 Git 的配置文件。全局配置文件通常位于 ~/.gitconfig，
# 而仓库特定的配置文件位于仓库的 .git/config 文件中。
[user]
    name = Your Name
    email = your.email@example.com

```



![](https://s2.loli.net/2025/02/11/v31hFl7JdoKaDCu.png)

配置之后，XCode的git工具中就会出现这个区域：

![](https://s2.loli.net/2025/02/11/oHyjfwmGXrZ9iEd.png)







### 1.3 **将git关联gitee.com上的项目：**



创建项目之后，会提示如下：

![](https://s2.loli.net/2025/02/11/v4NaAT5tXYESw1h.png)

其中的全局设置username和email应该已经设置过了。



```plain&#x20;text
添加remote：
git remote add origin https://gitee.com/longmix/uni-plugin-ios.git

完成第一次提交
进入你已经初始化好的或者克隆项目的目录，然后执行：
git pull origin master

<这里需要修改/添加文件，否则与原文件相比就没有变动>

git add .
git commit -m "第一次提交"

git push origin master


从remote 克隆项目：
git clone https://gitee.com/longmix/uni-plugin-ios.git
```





![](https://s2.loli.net/2025/02/11/RKcZyPwmtohGAzu.png)



查看本地的config配置，看是否有user.name和user.email



添加远程服务器



查询远程服务器







![](https://s2.loli.net/2025/02/11/ATpurezCJXMUdmE.png)



如果直接用“git push origin master”，会报错误，这是因为本地没有master这个分支，而存在的是“main”这个分支。





#### 增加目录到git



![](https://s2.loli.net/2025/02/11/nHcK9JvjWAxgaIM.png)





增加一个目录到git，并commit



#### 修改XCode的项目名称：

![](https://s2.loli.net/2025/02/11/nlWLqjzEgf9Z4AM.png)


```
git mv HBuilder-Hello.xcodeproj ebi-ios-app.xcodeproj



jetyan@jetdeMac ebi-ios-app % pwd

/Users/jetyan/dev/HBuilderX.3.99.2023122611.SDK/SDK/ebi-ios-app

```

这时候，打开XCode，发现最近项目的列表中名称也跟踪同步改变了。





### 1.4 git基础知识



> **Git 可以大概分为三个区**
>
> Git 本地数据管理，大概可以分为三个区，工作区,暂存区和版本库。
>
> **工作区（Working Directory）**
>
> 是我们直接编辑的地方，例如 Android Studio 打开的项目，记事本打开的文本等，肉眼可见，直接操作。
>
> **暂存区（Stage 或 Index）**
>
> 数据暂时存放的区域，可在工作区和版本库之间进行数据的友好交流。
>
> **版本库（commit History）**
>
> 存放已经提交的数据，push 的时候，就是把这个区的数据 push 到远程仓库了。


![](https://s2.loli.net/2025/02/11/Nptx4FX8E2SUn7f.png)

![](https://s2.loli.net/2025/02/11/gYMyPLQiEau2lk9.png)





### 1.5 gitignore忽略文件并删除已经提交的忽略文件



1、将文件或文件夹添加到 `.gitignore` 文件中。

2、从暂存区中删除文件或文件夹，git rm --cached path/to/your/file\_or\_directory （这个命令只会将文件从 Git 的暂存区中删除，而不会删除你的本地文件。）【如果删除的是目录，带上参数 -r】

3、**提交更改**：现在，你可以提交这些更改。这次提交会从版本库中移除这些文件或文件夹，但它们仍然会保留在你的本地文件系统中。git commit -m "Remove files or directories from version control and add to .gitignore"



```bash
jetyan@jetdeMac ebi-ios-app % ls -a
.                        .gitignore                MyReplayKitExtension
..                        EbiUniPlugin                README.en.md
.DS_Store                HBuilder                README.md
.git                        HBuilder-Hello                ebi-ios-app.xcodeproj

jetyan@jetdeMac ebi-ios-app % vi .gitignore 

jetyan@jetdeMac ebi-ios-app % git rm --cached HBuilder-Hello/Pandora/apps/__UNI__3A5A0E2
fatal: not removing 'HBuilder-Hello/Pandora/apps/__UNI__3A5A0E2' recursively without -r

jetyan@jetdeMac ebi-ios-app % git rm -r --cached HBuilder-Hello/Pandora/apps/__UNI__3A5A0E2
rm 'HBuilder-Hello/Pandora/apps/__UNI__3A5A0E2/www/__uniappchooselocation.js'
rm 'HBuilder-Hello/Pandora/apps/__UNI__3A5A0E2/www/__uniapperror.png'
rm 'HBuilder-Hello/Pandora/apps/__UNI__3A5A0E2/www/__uniappes6.js'
rm 'HBuilder-Hello/Pandora/apps/__UNI__3A5A0E2/www/__uniappopenlocation.js'

jetyan@jetdeMac ebi-ios-app % git rm --cached HBuilder-Hello/Pandora/.DS_Store 
rm 'HBuilder-Hello/Pandora/.DS_Store'

jetyan@jetdeMac ebi-ios-app % git rm --cached HBuilder-Hello/Pandora/apps/.DS_Store
rm 'HBuilder-Hello/Pandora/apps/.DS_Store'

jetyan@jetdeMac ebi-ios-app % vi .gitignore                                                
jetyan@jetdeMac ebi-ios-app % git rm --cached ebi-ios-app.xcodeproj/project.xcworkspace/xcuserdata/jetyan.xcuserdatad/UserInterfaceState.xcuserstate
rm 'ebi-ios-app.xcodeproj/project.xcworkspace/xcuserdata/jetyan.xcuserdatad/UserInterfaceState.xcuserstate'
jetyan@jetdeMac ebi-ios-app % 
jetyan@jetdeMac ebi-ios-app % 
jetyan@jetdeMac ebi-ios-app % git rm --cached HBuilder-Hello/.DS_Store                                                     
rm 'HBuilder-Hello/.DS_Store'
jetyan@jetdeMac ebi-ios-app % 
jetyan@jetdeMac ebi-ios-app % git rm --cached .DS_Store               
rm '.DS_Store'
jetyan@jetdeMac ebi-ios-app % vi .gitignore



```











## 2、HBuilder相关文档

这个记录了最初尝试的基座打包：

[ 0630.UniPlugin-uniapp插件开发笔记](https://yanyubao.feishu.cn/docx/NWkQdVVbRo7h2RxxyDLcDr2In1d)

主要的参考文档在：

【iOS的本地打包】

https://nativesupport.dcloud.net.cn/AppDocs/usesdk/ios.html

【iOS原生插件开发】

https://nativesupport.dcloud.net.cn/NativePlugin/course/ios.html





## macOS操作技巧


### 调换Ctrl和Command键



在系统设置中操作。调换后的操作更符合使用习惯。



![](https://s2.loli.net/2025/02/11/l1B5n2vuK6MIzCd.png)



![](https://s2.loli.net/2025/02/11/nN3MShKY1rzgGfv.png)





### 输入法的中英文切换

macOS自带的输入法虽然不好用，但是不占用多余资源，切换中英文用Caps Lock键。

顿号用的是回车键上方那个键。

切记不要使用Windows键盘上的Home和End，会出现意想不到的结果。

快速定位光标到行尾：Command + 向右的箭头，如果要同时选中文字，则同时按住Shift键。



![](https://s2.loli.net/2025/02/11/K4EBteCYfAp5cQ8.png)



![](https://s2.loli.net/2025/02/11/7rw18jIK4FTxXVf.png)





### 快速复制文件路径

右键点击某个文件，然后按住Option键，找到复制文件路径的功能。



![](https://s2.loli.net/2025/02/11/JDRWI3L7CSHBtkf.png)



![](https://s2.loli.net/2025/02/11/Poi5lEpIaHFTUu1.png)





### 快速清除右键多余菜单

场景：删除了BBEdit这个应用之后，右键依然有相关菜单，命令行运行：

```javascript
/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister -kill -r -domain local -domain system -domain user
```



