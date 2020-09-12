---
layout: single
title:  "配置工作环境"
date:   2020-09-12 23:15:00 +0800
categories: work
tags: [setup]
---

当拿到一台新的 mbp 时，要做哪些安装和配置来 setup 一个适合自己的舒适的工作环境呢？
我尝试在这里记录一下，并可能不时更新，为了方便之后的自己。

# Homebrew 源配置
```shell
# 替换 Homebrew
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 替换 Homebrew Core
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# 替换 Homebrew Cask
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# 替换 Homebrew-bottles
# 对于 bash 用户：
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
# 对于 zsh 用户：
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

# 安装 macOS 常用软件

* [hidden](https://github.com/dwarvesf/hidden)：隐藏菜单栏中部分图标，看起来清爽
* [lepton](https://github.com/hackjutsu/Lepton)：管理 github gist 的 GUI 工具
* [dash](https://kapeli.com/dash)：离线 API 文档
* [Alfred](https://www.alfredapp.com/)：效率工具



# 常用命令行工具

* jq


# git 配置
## 全局 gitignore
```shell
touch ~/.gitignore

# 填写如下内容
.DS_Store
.vscode

# 设置为 global 级别的 ignore
git config --global core.excludesfile ~/.gitignore
```

# vim 配置


# tmux 配置

# python 配置
## pip.conf

