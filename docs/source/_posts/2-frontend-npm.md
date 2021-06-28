---
title: 前端 | NPM 包管理工具
date: 2020-5-29
tags: 
- npm
- Frontend
categories: notes
photos:
    - /blog/img/npm.jpg
---

<br>
<!--more-->

# 安装

```bash
# 安装 node 环境
# 在 https://nodejs.org 下载安装

# 安装后查看 npm 版本
npm -v

# 安装后查看 node 版本
node -v

# 更新最新稳定版本 node 至全局
npm install npm@latest -g
```

# 常用操作

```bash
# 项目目录初始化
npm init -y

# 安装包
# 或者在 package.json 配置依赖
npm i jquery
npm i jquery@3.2.1  # 安装旧版本
npm i vue
npm i bootstrap

# 卸载包
npm uninstall normalize.css

# 安装至开发环境
npm i webpack --save-dev
```

# npm 指令速查

**常用**
- `npm init` 初始化项目，其实就是创建一个 package.json 文件。
- `npm install` 安装所有项目依赖。
- `npm help xxx` 查看xxx命令的帮助信息。

**搜索** npm search （快捷方式：find, s）
- xxx 搜索xxx 如：`npm search jquery`。

**安装** npm install （快捷方式：i）
- xxx 搜索并安装xxx（局部）。安装多个依赖可用空格分割，如 `npm i jquery bootstrap`。
- xxx -g 搜索并安装xxx（全局）。安装多个同上。
- xxx -D 安装并将依赖信息写在 package.json 中的 devDependencies 中。
- 快捷方式 i 均可，如 `npm i jquery`。
- xxx@版本号 指定需要安装的版本号，若不指定将安装最新的稳定版本。

**卸载** `npm uninstall` （快捷方式：rm, r）
- xxx 卸载xxx。多个依赖可用空格分割。
- xxx -D 卸载xxx，并将依赖信息从 package.json 中的 devDependencies 中清除。

**列出已安装依赖** `npm list` （快捷方式：ls）
- 默认列出局部依赖。
- `npm list -g` 列出已安装的全局依赖。

**检查过期依赖** `npm outdated`

**更新依赖** `npm update` （快捷方式：up）
- xxx 局部更新xxx。
- xxx -g 全局更新xxx。

**查看依赖安装路径** `npm root` （也就是node_modules的路径）
- 默认查看局部安装路径。
- -g 查看全局安装路径。

**查看模块的注册信息** `npm view` 
- xxx versions 列出xxx的所有版本， 如：`npm view jquery versions`。
- xxx dependencies 列出xxx的所有依赖， 如：`npm view gulp dependencies`。

# 参考资料

[npm 基础](https://github.com/wangdoc/node-tutorial/blob/master/docs/npm/basic.md)
[npm 相关资料](https://github.com/wangdoc/node-tutorial/tree/master/docs/npm)