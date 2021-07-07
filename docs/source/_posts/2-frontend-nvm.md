---
title: 前端 | nvm 包管理工具
date: 2021-6-28
tags: 
- nvm
- Frontend
categories: notes
photos:
    - /blog/img/nvm.jpeg
---

<br>
<!--more-->

# 安装

安装之前要清理掉旧的 `node` 环境，需要先安装有 `git`

## win

选择安装 `nvm-windows`
直接在GitHub上下载：`https://github.com/coreybutler/nvm-windows/releases`
按步骤安装

## mac/linux

`curl` 或者 `wget` 都可

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash

wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash
```

# 常用操作

```bash
# 显示 node 是运行在32位还是64位模式。指定32或64来覆盖默认体系结构。
nvm arch [32|64]： 

# 列出已经安装的 node.js 版本。可选的 available，显示可下载版本的部分列表。
nvm ls [available]

# 安装具体版本。可选 arch，指定安装32位或64位版本（默认为系统arch）
nvm install <version> [arch]

# 设置镜像
nvm node_mirror [url]
nvm npm_mirror [url]

# 可设置为淘宝镜像
nvm node_mirror http://npm.taobao.org/mirrors/node/
nvm npm_mirror https://npm.taobao.org/mirrors/npm/

# 卸载指定版本的 nodejs
nvm uninstall <version>

# 切换到使用指定的 nodejs版本。可以指定 32/64 位 [arch]，也可单指定 arch 切换
nvm use [version] [arch]

# 显示当前运行的 nvm 版本，可以简写为 nvm v
nvm version

```

# 参考

多版本nodejs管理工具nvm的介绍与使用：https://www.tensweets.com/article/5bafb7aef0cb0c04f86b23ed