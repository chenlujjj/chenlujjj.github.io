---
layout: single
title:  "[笔记] Linux 命令基础"
date:   2020-06-17 23:00:00 +0800
categories: os
tags: [os, linux]
toc: true
toc_sticky: true
toc_icon: "fish"
---

《趣谈 Linux 操作系统》专栏第4讲笔记，记录下之前不太熟悉的命令。



**浏览文件**

```shell
> ls -l
total 72
-rw-r--r--   1 chenluxin  staff   398B Jun 17 00:00 404.html
-rw-r--r--   1 chenluxin  staff   1.4K Jun 17 00:21 Gemfile
-rw-r--r--   1 chenluxin  staff   6.8K Jun 17 00:00 Gemfile.lock
-rw-r--r--   1 chenluxin  staff   8.6K Jun 17 00:00 _config.yml
drwxr-xr-x   3 chenluxin  staff    96B Jun 17 00:00 _data
drwxr-xr-x   7 chenluxin  staff   224B Jun 17 00:00 _pages
drwxr-xr-x  11 chenluxin  staff   352B Jun 17 23:14 _posts
drwxr-xr-x   3 chenluxin  staff    96B Jun 17 00:00 _sass
drwxr-xr-x   4 chenluxin  staff   128B Jun 17 00:00 assets
-rw-r--r--   1 chenluxin  staff    41B Jun 17 00:00 index.html
-rw-r--r--   1 chenluxin  staff    91B Jun 17 00:00 todo
```

上述输出中：

* 第一个字段的第一个字符是文件类型。如果是“-”，表示普通文件；如果是 d，就表示目录。剩下9个字符是权限位，分别代表文件所属的用户权限、文件所属的组权限以及其他用户的权限，每组3个权限为读，写，执行。
* 第二个字段是**硬链接（hard link）数目**
* 第三个字段是所属用户
* 第四个字段是所属组
* 第五个是文件大小
* 第六个是文件被修改的日期
* 最后是文件名



**systemd 命令**

* `systemctl start`：启动服务
* `systemctl enable`：设置服务开机启动。之所以成为服务并且能够开机启动，是因为在 `/usr/lib/systemd/system` 目录下会创建一个 `XXX.service` 的配置文件，里面定义了如何启动、如何关闭。​



:warning: :construction:**问题和 TODO**

1. 什么是硬链接？
2. 文件夹的大小代表什么含义？
3. `/etc/profile`，`/etc/bashrc`，`/etc/bashrc_xxx`，`$HOME/.bashrc` 的区别和用法？
4. `systemd` 深入学习