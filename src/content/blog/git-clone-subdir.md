---
author: magnum-zx
pubDatetime: 2023-06-02T15:28:00.000+08:00
title: git clone 远程仓库的子目录
postSlug: git-clone-subdir
featured: true
ogImage: ""
tags:
  - git
description: ""
---

# Git clone 远程仓库下的子目录

## Overview

在 Github 上存在很多 `big repository`，如果一次性把整个仓库 clone 下来看起来是有点多余，而理想情况应该是只 clone 仓库下某个文件或目录。一些 Chrome 插件（如 [GitZip](https://chrome.google.com/webstore/detail/gitzip-for-github/ffabmkklhbepgcgfonabamgnfafbdlkn)）也可以做到打包下载某个目录文件夹。但是，本文主要通过 git 的 `Spare Checkout` 特性来实现。

## Spare Checkout

git 有个"Spare Checkout"的功能，在 checkout 的时候，只跟踪符合配置文件.git/info/sparse-checkout 里面写入的模式的文件，别的文件可见，跟不存在一样。

为了启用"Sparse Checkout"功能，需要将 git 选项 core.sparsecheckout 为 true:

```
git config core.sparsecheckout true
```

## 具体操作

在 `terminal` 中如下操作

```
$ mkdir dirname # 创建一个目录
$ cd dirname
$ git init # 初始化
$ git remote add origin <repo> # 增加远端的仓库地址
$ git config core.sparsecheckout true # 设置Sparse Checkout 为true
$ echo "subdir" >> .git/info/sparse-checkout # 将想要clone的子目录（相对根目录）的路径写入配置文件
$ git pull origin <branch> # pull仓库代码
```

- `dirname` - 本地仓库目录（随你取）
- `<repo>` - 远程仓库地址，如 https://github.com/vuejs/vue.git
- `subdir` - 远程仓库下想要 clone 的子目录，如 `vue` 仓库下的 `examples/classic`
- `<branch>` - clone 分支，如 `vue` 仓库的 `main` 分支
