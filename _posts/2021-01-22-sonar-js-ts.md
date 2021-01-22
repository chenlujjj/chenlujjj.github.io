---
layout: single
title:  "使用SonarQube扫描JS/TS代码的经验总结"
date:   2021-01-22 22:00:00 +0800
categories: devops
tags: [sonarqube, javascript, typescript, jenkins]
---


**依赖**

需要运行sonar-scanner的机器上安装有node。node 可执行文件要在系统环境变量 $PATH 中，否则要在扫描时通过 `sonar.nodejs.executable` 属性指定该文件的绝对路径。

**执行扫描**

最简单的情况下，设置好 `sonar.projectKey`和`sonar.projectName`属性（通过`sonar-project.properties`文件定义或者通过命令行指定），然后执行sonar-scanner命令即可。

**Troubleshooting**

*执行扫描时报错 Missing Typescript dependency*

解决方法：在扫描前安装好依赖：`npm install typescript`。安装时可以指定源来加速。

*npm install 时报错 gyp ERR*

解决方法：对于 debian 系统，通过安装 build-essential 包即可，`apt-get install build-essential`

**和Jenkins集成时碰到的问题**

由于扫描需要node，我理所当然地想到Jenkins应该有类似于Java/Maven插件的tool在pipeline运行时提供node环境。搜索一下，确实有NodeJS插件。迅速安装好该插件试试。

问题来了，如果使用插件中默认的安装node方法，从官网nodejs.org安装，由于GFW，下载不了。

于是尝试另一种方法：通过解压压缩包安装。找了一个阿里镜像站提供的[下载链接](https://npm.taobao.org/mirrors/node/v14.15.4/node-v14.15.4-linux-x64.tar.xz)，下载很快，但是在Jenkins机器上却无法下载，报错“java.io.IOException: incorrect header check”。一番搜索后，没找到方法解决。

还有一种方式是通过运行命令安装。官方提供了在debian机器上安装node的[标准命](https://github.com/nodesource/distributions/blob/master/README.md#debinstall)，但是在Jenkins机器上执行又出现了权限问题。这个权限问题应该可以解，不过觉得可能会投入过多时间，暂且搁置。

最后的解决方法是：先将node的安装压缩包下载到本地，再上传到artifactory上，选择“解压压缩包安装”，填入artifactory制品的下载链接。

另一个问题是，即使在Jenkinsfile中声明的node的tool，在stage中执行诸如`node -v`, `npm -v`的命令时，会提示找不到命令。按道理这是不符合预期的：tool应该将node，npm放入`$PATH中`呀。没办法，只好自行在Jenkinsfile中设置环境变量了：`environment { PATH="$PATH:/dir/of/node/binary/" }`。

**参考**

* https://docs.sonarqube.org/latest/analysis/languages/javascript/
* https://github.com/SonarSource/SonarJS

