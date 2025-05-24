---
title: 在GitLab和GitHub之间同步Git版本仓库的代码
date: 2021-11-18 14:30:45
comments: true
tags:
- GitLab
- GitHub
- 同步代码
- 备份项目
- 远程仓库
- 局域网
categories:
- 软件开发
- 运维管理
---


在GitLab和GitHub之间同步代码的意义，首先是实现备份代码：虽然Git版本仓库本身就提供了完整的版本记录，但是通过将代码同步到GitLab，你可以确保在GitHub上的项目因任何原因被删除时，你仍然有一份副本。另一方面，通过本地部署GitLab，可以提供了更强大的协作功能，除了分支管理、代码评审等，以及帮助团队成员更好地协同工作之外，还可以提供更安全的基于局域网的代码托管环境。

将GitHub上的项目导入到GitLab可以帮助你备份代码，并确保在GitHub上的项目因任何原因被删除时，你仍然有一份副本。以下是详细的步骤：

## 步骤一：在GitLab上导入GitHub项目

新建项目：在GitLab上，点击“新建项目”按钮。

导入项目：选择“导入项目”，然后选择“Repo by URL”。

输入GitHub项目地址：在URL栏中输入你要导入的GitHub项目地址。

设置项目信息（可选）：你可以设置项目名称、描述等信息。

创建项目：点击“Create”按钮完成导入。

## 步骤二：将GitLab和GitHub仓库关联

克隆GitLab项目：首先，将GitLab上的项目克隆到本地。

```bash
git clone git@gitlab.com:你的用户名/项目名.git
cd 项目名
```

添加GitHub远程库：在本地仓库中添加GitHub远程库。

```bash
git remote add github https://github.com/你的用户名/项目名.git
```

查看远程库：确认远程库添加成功。

```bash
git remote -v
```

从GitHub拉取代码：更新GitHub的仓库，会新建一个分支为远程库别名/分支名。

```bash
git pull github master
```

推送到GitLab：将更新后的代码推送到GitLab私有库。

```bash
git push origin master
```

通过以上步骤，你可以确保GitLab和GitHub上的代码保持一致。每次更新GitHub上的代码后，只需重复步骤4和步骤5即可同步到GitLab。

## 总结

主要解决两个问题：
- 如何从GitHub导入项目到GitLab
- GitLab和GitHub的同步方法


