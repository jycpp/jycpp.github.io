---
title: postfix设置默认的缺省收件邮箱
date: 2022-04-13 23:52:23
comments: true
tags:
- Postfix
- 邮件转发
- 虚拟别名映射
- 数据库配置
- MySQL
- 邮件服务器
- 主配置文件
- 虚拟邮箱
- 域名转发
- Dovecot
categories:
- 电子邮件
- 邮件服务器配置
---

搭建邮件服务器，postfix依然是最好的选择，组合dovecot+mysql+postfix可以实现邮件的接收，存储，转发等功能。

在postfix的Web控制台，除了可以管理多个域名邮箱之外，每个域名下还可以设置虚拟邮箱用户（Alias account），这些虚拟邮箱用户可以通过邮箱地址直接接收邮件，而没有发送邮件的功能，好处在于不需要在Web控制台中创建邮箱，对于只是接收验证码之类的场景费用有用。

我们是希望在Web控制台中，每个域名下都可以设置一个默认邮箱地址，当用户发送邮件到该域名时，如果该邮箱地址不存在，则会将邮件转发到默认邮箱地址。这样可以避免大量设置虚拟邮箱用户，提高管理效率。

同时，考虑到postfix支持多域名邮箱，我们还希望postfix按照不同的域名，转发到各自对应的邮箱。


## 配置Postfix转发虚拟邮箱

要实现以上过程，需要在Postfix的配置文件中进行以下设置：

### 配置Postfix主配置文件

打开`/etc/postfix/main.cf`文件。添加或修改以下配置项：

```
virtual_alias_domains = domain1.com, domain2.com  # 列出需要进行转发的域名，多个域名用逗号分隔
virtual_alias_maps = hash:/etc/postfix/virtual  # 指定虚拟别名映射文件的路径
```

### 创建虚拟别名映射文件

创建`/etc/postfix/virtual`文件。在文件中按照以下格式添加转发规则：

```
@domain1.com destination1@example.com  # 将发往domain1.com中不存在账号的邮件转发到destination1@example.com
@domain2.com destination2@example.com  # 将发往domain2.com中不存在账号的邮件转发到destination2@example.com
```

### 生成映射文件数据库

执行命令`postmap hash:/etc/postfix/virtual`，生成虚拟别名映射文件的数据库。

### 重启Postfix服务

执行命令`systemctl restart postfix`，使配置生效。

以上步骤中，要确保`main.cf`文件中的`virtual_alias_domains`配置项准确列出了需要处理的域名，`virtual_alias_maps`指定的映射文件路径正确，并且在`/etc/postfix/virtual`文件中为每个域名配置了正确的转发规则。


## 为什么需要修改main.cf文件？

在Postfix的配置文件`main.cf`中，有一个配置项`virtual_alias_maps`，用于指定虚拟别名映射文件的路径。

当你配置了`virtual_alias_maps`后，Postfix会读取该文件中的规则，将发往指定域名的邮件地址转换为对应的邮箱地址。

如果不修改`main.cf`文件，Postfix默认会使用`/etc/postfix/virtual`作为虚拟别名映射文件的路径。

这个过程并非绝对不能通过配置MySQL数据库表来实现，只是通常需要同时修改`main.cf`文件来让Postfix与MySQL数据库进行交互，从而实现基于数据库配置的邮件转发等功能。以下是相关原因说明：

- **修改main.cf文件的必要性**

    - **启用数据库查询功能**：Postfix默认不会主动连接和查询MySQL数据库。要让Postfix能够利用MySQL数据库中的数据来处理邮件，必须在`main.cf`文件中进行配置，以告知Postfix使用数据库以及如何连接数据库。例如，需要设置`virtual_mailbox_domains`参数来指定虚拟邮箱域名的来源为MySQL数据库，还需配置`virtual_mailbox_maps`和`virtual_alias_maps`等参数，让Postfix知道从数据库的哪些表和字段中获取用户邮箱和别名信息。
    
    - **配置邮件处理行为**：`main.cf`文件用于设置Postfix的各种运行参数和邮件处理策略。即使邮件转发规则存储在MySQL数据库中，也需要在`main.cf`中配置相关参数，来确定Postfix如何根据数据库中的规则进行邮件转发。比如，设置`smtpd_recipient_restrictions`参数，可限制收件人地址，并根据数据库中的信息来判断是否接受或转发邮件。

- **使用MySQL数据库表的优势**

    - **灵活管理转发规则**：将邮件转发规则存储在MySQL数据库表中，可以通过数据库管理工具方便地添加、修改和删除转发规则，无需直接编辑配置文件。对于大型邮件系统，当需要频繁调整转发策略或管理大量域名和邮箱时，使用数据库表进行管理更加高效和便捷。

    - **支持动态配置**：数据库表可以根据实际需求实时更新，Postfix在运行过程中能够及时读取到最新的转发规则，实现动态配置邮件转发。例如，当有新的域名需要添加转发规则，或者某个邮箱的转发目标发生变化时，只需在数据库中进行相应修改，而无需重启Postfix服务。

要通过MySQL数据库表实现邮件按不同域名转发到对应邮箱，需在`main.cf`文件中正确配置Postfix与MySQL数据库的连接和交互参数，同时在数据库中合理设计表结构来存储转发规则等信息。这样Postfix才能依据数据库中的配置信息，准确地处理邮件的转发。
