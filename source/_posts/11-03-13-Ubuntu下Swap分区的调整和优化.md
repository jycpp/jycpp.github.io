---
title: Ubuntu 下 Swap 分区的调整和优化
date: 2011-03-13 17:27:20
comments: true
tags:
- Ubuntu
- Swap分区
- swappiness
- 内存优化
- 系统性能
- dd命令
- mkswap
- swapfile
- free命令
- fstab
categories:
- 软件开发
---



在Ubuntu系统中，`swappiness`的值与如何使用Swap分区密切相关。当`swappiness = 0`时，系统会最大限度地使用物理内存，之后才会使用Swap空间；而当`swappiness = 100`时，系统会积极使用Swap分区，并及时将内存中的数据转移到Swap空间。Ubuntu的默认设置下，这个值为60，不过建议将其修改为10，这样能优化系统性能。具体操作步骤如下：

1. **查看系统中的`swappiness`值**：
在终端中输入以下命令：
```bash
cat /proc/sys/vm/swappiness
```
正常情况下，你会看到显示的值为60。
2. **修改`swappiness`值为10**：
首先，在终端中执行临时修改命令：
```bash
sudo sysctl vm.swappiness=10
```
但需要注意的是，这种修改只是临时性的，重启系统后会恢复默认的60。因此，还需要进行如下操作：
```bash
gksudo gedit /etc/sysctl.conf
```
在打开的文件末尾添加一行：
```bash
vm.swappiness=10
```
保存文件并重启系统，这样设置就会生效。你可能会发现系统运行速度有所提升。另外，你也可以使用其他编辑器（如`kate`、`vi`、`vim`、`nano`等）进行修改，只需将上述命令中的`gedit`替换成相应编辑器名称即可。这里使用`gedit`是因为大多数人使用的是Gnome桌面。

除了调整`swappiness`值，有时还需要调整Swap分区的大小。比如，当系统中的Swap分区大小不符合某些软件（如`Oracle-xe-client`）的安装要求时，就需要进行调整。以下是调整Swap分区大小的具体步骤：


1. **查看系统内Swap分区大小**：

在终端中输入命令：

```bash
free -m
```
示例输出如下：
```
          total used free shared buffers cache
          Mem: 1002 964 38 0 21 410
          -/+ buffers/cache: 532 470
          Swap: 951 32 929
```
从输出中可以看到，当前Swap分区大小为951M。
2. **创建一个Swap文件**：
首先创建一个用于存放Swap文件的目录，然后进入该目录：
```bash
mkdir swap
cd swap
```
接着使用`dd`命令创建Swap文件，`count`的值决定了Swap文件的大小。例如，执行以下命令创建一个大小为100000块（每块1024字节，即约100MB）的Swap文件：
```bash
sudo dd if=/dev/zero of=swapfile bs=1024 count=100000
```
执行命令后，会出现类似以下提示：
```
记录了 100000+0 的读入
记录了 100000+0 的写出
102400000 字节 (102 MB) 已复制，0.74704 秒，137 MB/秒
```
3. **把生成的文件转换成Swap文件**：
在终端中执行以下命令：
```bash
sudo mkswap swapfile
```
执行结果示例如下：
```
Setting up swapspace version 1, size = 102395 kB
no label, UUID=09fde987-5567-498a-a60b-477e302a988b
```
4. **激活Swap文件**：
使用以下命令激活刚刚创建的Swap文件：
```bash
sudo swapon swapfile
```
激活后，再次使用`free -m`命令查看系统内存和Swap分区情况，示例输出如下：
```
          total used free shared buffers cached
          Mem: 1002 967 34 0 22 410
          -/+ buffers/cache: 534 467
          Swap: 1053 32 1021
```
可以看到，Swap分区大小已增加到1053M，说明添加成功。

**扩展内容**：
如果需要卸载这个Swap文件，可以进入建立的Swap文件目录，执行以下命令：
```bash
sudo swapoff swapfile
```
如果希望系统一直保持这个Swap文件，可以将其写入`/etc/fstab`文件，在文件中添加一行：
```bash
swapfilepath swap swap defaults 0 0
```
其中，`swapfilepath`需要替换为实际的Swap文件路径。 

