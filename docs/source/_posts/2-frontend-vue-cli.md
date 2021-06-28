---
title: 前端 | Vue 学习笔记Ⅱ：@Vue/cli
date: 2020-6-3
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

# 概述

> Vue CLI 是一个基于 Vue.js 进行快速开发的完整系统


**提供**：
- 通过 @vue/cli 实现的交互式的项目脚手架。
- 通过 @vue/cli + @vue/cli-service-global 实现的零配置原型开发。
- 一个运行时依赖 (@vue/cli-service)，该依赖：
    - 可升级；
    - 基于 webpack 构建，并带有合理的默认配置；
    - 可以通过项目内的配置文件进行配置；
    - 可以通过插件进行扩展。
- 一个丰富的官方插件集合，集成了前端生态中最好的工具。
- 一套完全图形化的创建和管理 Vue.js 项目的用户界面。


## 组件

### CLI

> CLI (@vue/cli) 是一个全局安装的 npm 包，提供了终端里的 vue 命令。它可以通过 `vue create` 快速搭建一个新项目，或者直接通过 vue serve 构建新想法的原型。也可以通过 vue ui（一套图形化界面）管理所有项目。

### CLI 服务

> CLI 服务 (@vue/cli-service) 是一个开发环境依赖。它是一个 npm 包，局部安装在每个 @vue/cli 创建的项目中

CLI 服务是构建于 webpack 和 webpack-dev-server 之上的。它包含了：
- 加载其它 CLI 插件的核心服务；
- 一个针对绝大部分应用优化过的内部的 webpack 配置；
- 项目内部的 vue-cli-service 命令，提供 serve、build 和 inspect 命令

### CLI 插件

> CLI 插件是向 Vue 项目提供可选功能的 npm 包，例如 Babel/TypeScript 转译、ESLint 集成、单元测试和 end-to-end 测试等。Vue CLI 插件的名字以 @vue/cli-plugin- (内建插件) 或 vue-cli-plugin- (社区插件) 开头。在项目内部运行 vue-cli-service 命令时，它会自动解析并加载 package.json 中列出的所有 CLI 插件。


# 安装


```bash
# 安装
npm install -g @vue/cli
# OR
yarn global add @vue/cli


# 卸载
npm uninstall vue-cli -g


# 使用 vue 命令
vue -V   # @vue/cli 4.4.6
```


# 基础命令

> 使用 `vue serve` 和 `vue build` 命令对单个 *.vue 文件进行快速原型开发
> *快速原型开发需要先安装一个全局的扩展 `@vue/cli-service-global`

## vue serve

> 为 .js 或 .vue 文件启动一个服务器。`vue serve` 使用了和 `vue create` 创建的项目相同的默认设置 (webpack、Babel、PostCSS 和 ESLint)。它会在当前目录自动推导**入口文件**————入口可以是 main.js、index.js、App.vue 或 app.vue 中的一个。你也可以显式地指定入口文件：`vue serve MyComponent.vue`

```bash
Usage: serve [options] [entry]

在开发环境模式下零配置为 .js 或 .vue 文件启动一个服务器

Options:

  -o, --open  打开浏览器
  -c, --copy  将本地 URL 复制到剪切板
  -h, --help  输出用法信息


# 创建一个 App.vue 文件
<template>
  <h1>Hello!</h1>
</template>


# 在这个 App.vue 文件所在的目录下运行
vue-cli-service serve
```

## vue build

> 在生产环境模式下零配置构建一个 .js 或 .vue 文件，也可以将目标文件构建成一个生产环境的包并用来部署，也具有将组件构建成为一个库或一个 Web Components 组件的能力

```bash
Usage: build [options] [entry]

在生产环境模式下零配置构建一个 .js 或 .vue 文件


Options:

  -t, --target <target>  构建目标 (app | lib | wc | wc-async, 默认值：app)
  -n, --name <name>      库的名字或 Web Components 组件的名字 (默认值：入口文件名)
  -d, --dest <dir>       输出目录 (默认值：dist)
  -h, --help             输出用法信息


vue-cli-service build MyComponent.vue
```


## vue create

```bash
# 创建一个新项目
vue create hello-world


# vue create --help
用法：create [options] <app-name>

创建一个由 `vue-cli-service` 提供支持的新项目


选项：

  -p, --preset <presetName>       忽略提示符并使用已保存的或远程的预设选项
  -d, --default                   忽略提示符并使用默认预设选项
  -i, --inlinePreset <json>       忽略提示符并使用内联的 JSON 字符串预设选项
  -m, --packageManager <command>  在安装依赖时使用指定的 npm 客户端
  -r, --registry <url>            在安装依赖时使用指定的 npm registry
  -g, --git [message]             强制 / 跳过 git 初始化，并可选的指定初始化提交信息
  -n, --no-git                    跳过 git 初始化
  -f, --force                     覆写目标目录可能存在的配置
  -c, --clone                     使用 git clone 获取远程预设选项
  -x, --proxy                     使用指定的代理创建项目
  -b, --bare                      创建项目时省略默认组件中的新手指导信息
  -h, --help                      输出使用帮助信息


# 打开预览项目
yarn serve
# 等同于
vue-cli-service serve
```

## vue ui

> 通过 vue ui 命令以图形化界面创建和管理项目


# 插件

> Vue CLI 使用了一套基于插件的架构，插件可以修改 webpack 的内部配置，也可以向 vue-cli-service 注入命令。在项目创建的过程中，绝大部分列出的特性都是通过插件来实现的。
> 
> 每个 CLI 插件都会包含一个 (用来创建文件的) 生成器和一个 (用来调整 webpack 核心配置和注入命令的) 运行时插件。

```bash
# 创建一个新项目的时候，有些插件会根据你选择的特性被预安装好
vue create xxx

# 在一个已经被创建好的项目中安装插件
vue add eslint
# 使用 vue add 安装调用 Vue CLI 插件
# 将 @vue/eslint 解析为完整的包名 @vue/cli-plugin-eslint，
# 然后从 npm 安装它，调用它的生成器。


# 在项目里直接访问插件 API 而不需要创建一个完整的插件
# 在 package.json 文件中使用 vuePlugins.service 
{
  "vuePlugins": {
    "service": ["my-commands.js"]
  }
}
```

# Preset

> 一个 Vue CLI preset 是一个包含创建新项目所需预定义选项和插件的 JSON 对象，让用户无需在命令提示中选择它们

```bash
# 在 vue create 过程中保存的 preset 会建立在 home 目录下的配置文件(~/.vuerc)
# 可以通过直接编辑这个文件来调整、添加、删除保存好的 preset。
{
  "useConfigFiles": true,
  "cssPreprocessor": "sass",
  "plugins": {
    "@vue/cli-plugin-babel": {},
    "@vue/cli-plugin-eslint": {
      "config": "airbnb",
      "lintOn": ["save", "commit"]
    },
    "@vue/cli-plugin-router": {},
    "@vue/cli-plugin-vuex": {}
  }
}

# 为集成工具添加配置
# useConfigFiles 为 true 时，configs 的值将会被合并到 vue.config.js 
{
  "useConfigFiles": true,
  "plugins": {...},
  "configs": {
    "vue": {...},
    "postcss": {...},
    "eslintConfig": {...},
    "jest": {...}
  }
}


# 显式地指定用到的插件的版本
# 未指定时则会在源中选择最新的版本
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      "version": "^3.0.0",
      // ... 该插件的其它选项
    }
  }
}


# 允许插件的命令提示
# 使用 preset 使得项目创建过程中的命令提示已经声明，因此不会提示
# 需要声明的插件，可以通过单独注入命令提示
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      // 让用户选取他们自己的 ESLint config
      "prompts": true
    }
  }
}


# 通过发布 git repo 将一个 preset 分享给其他开发者
# # preset.json: 包含 preset 数据的主要文件（必需）。
# # generator.js: 一个可以注入或是修改项目中文件的 Generator。
# # prompts.js 一个可以通过命令行对话为 generator 收集选项的 prompts 文件。

# 创建项目时从 GitHub repo 使用 preset
vue create --preset username/repo my-project

# 测试时在本地加载 Preset
# ./my-preset 应当是一个包含 preset.json 的文件夹
vue create --preset ./my-preset my-project

# 或者，直接使用当前工作目录下的 json 文件：
vue create --preset my-preset.json my-project
```

# CLI 服务

## vue-cli-service serve

> 启动一个开发服务器 (基于 webpack-dev-server) 并附带开箱即用的模块热重载 (Hot-Module-Replacement)。

```bash
用法：vue-cli-service serve [options] [entry]

选项：

  --open    在服务器启动时打开浏览器
  --copy    在服务器启动时将 URL 复制到剪切版
  --mode    指定环境模式 (默认值：development)
  --host    指定 host (默认值：0.0.0.0)
  --port    指定 port (默认值：8080)
  --https   使用 https (默认值：false)
```

## vue-cli-service build

> 在 dist/ 目录产生一个可用于生产环境的包，带有 JS/CSS/HTML 的压缩，和为更好的缓存而做的自动的 vendor chunk splitting。它的 chunk manifest 会内联在 HTML 里。

```bash
用法：vue-cli-service build [options] [entry|pattern]

选项：

  --mode        指定环境模式 (默认值：production)
  --dest        指定输出目录 (默认值：dist)
  --modern      面向现代浏览器带自动回退地构建应用
  --target      app | lib | wc | wc-async (默认值：app)
  --name        库或 Web Components 模式下的名字 (默认值：package.json 中的 "name" 字段或入口文件名)
  --no-clean    在构建项目之前不清除目标目录
  --report      生成 report.html 以帮助分析包内容
  --report-json 生成 report.json 以帮助分析包内容
  --watch       监听文件变化
```

## vue-cli-service inspect

> 审查一个 Vue CLI 项目的 webpack config

```bash
用法：vue-cli-service inspect [options] [...paths]

选项：

  --mode    指定环境模式 (默认值：development)
  
```

## 查看所有的可用命令

> 除了上述三个，有些 CLI 插件会向 vue-cli-service 注入额外的命令。例如 @vue/cli-plugin-eslint 会注入 `vue-cli-service lint` 命令

```bash
# 查看所有注入的命令
npx vue-cli-service help

# 每个命令可用的选项
npx vue-cli-service help [command]
```

