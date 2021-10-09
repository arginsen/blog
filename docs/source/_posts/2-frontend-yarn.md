---
title: 前端 | yarn 包管理工具
date: 2020-6-29
tags: 
- yarn
- Frontend
categories: notes
photos:
    - /blog/img/yarn.jpg
---

<br>
<!--more-->

# 基本用法

```bash 
# 使用 npm 安装至全局
npm install -g yarn


# 全局参数 global
# 如果要全局执行一个命令，必须加上global参数
yarn global add create-react-app --prefix /usr/local


# 安装模块
# 如果 yarn.lock 文件存在，则优先按照该文件指定的版本安装
yarn install
# 在本地 node_modules 目录安装 package.json 里列出的所有依赖
yarn install / yarn   
# 重新拉取所有包，即使之前已经安装的
yarn install --force 
# 为 node_modules 目录指定另一位置，代替默认的 ./node_modules 
yarn install --modules-folder <path> 
# 不读取或生成 yarn.lock 文件 
yarn install --no-lockfile 
# 只安装 dependence下的包，不安装 devDependencies 的包
yarn install --production / --prod 


# 新增安装模块
# 默认将新增模块加入 package.json 文件的 dependencies 字段
yarn add package-name
# 用 --dev 或 -D 安装包到 devDependencies
yarn add <package...> [--dev/-D] 
# 用 --peer 或者 -P 安装包到 peerDependencies
yarn add <package...> [--peer/-P]  
# 用 --optional 或者 -O 安装包到 optionalDependencies
yarn add <package...> [--optional/-O] 
# 用 --exact 或者 -E 会安装包的精确版本。
# 默认是安装包的主要版本里的最新版本。
yarn add <package...> [--exact/-E]
# 用 --tilde 或者 -T 来安装包的次要版本里的最新版。
# 默认是安装包的主要版本里的最新版本。
# 比如说，yarn add foo@1.2.3 --tilde 会接受 1.2.9，但不接受 1.3.0
yarn add <package...> [--tilde/-T]  


# 管理配置文件
# 查看配置 key 的值
yarn config get <key> 
# 查看当前的配置
yarn config list 
# 从配置中删除配置 key
yarn config delete <key> 
# 设置配置项 key 的值为 value
yarn config set <key> <value> [-g|--global] 


# 查询当前工作文件夹所有的依赖
yarn list


# 查看包信息
yarn info <package> [<field>]  


# 从依赖里移除名包
# 同时更新 package.json 和 yarn.lock 文件
yarn remove <package...>  


# 更新模块版本，依据 package.json 里指定的版本范围，不指定则升级到 latest
yarn upgrade d3-scale@1.0.2


# 执行用户自定义的脚本
yarn <script> [<args>] 


# 安装某个模块的原因，也可分析目录
yarn why package-name
yarn why node_modules/once


# 依照 package.json 文件，生成 yarn.lock 文件
yarn generate-lock-entry
```


# registry

>registry 是模块仓库提供了一个查询服务，一般称作源。yarn 官方镜像源的查询服务网址是https://registry.yarnpkg.com
>
>网址后面跟上模块名，就会得到一个该模块所有版本的信息的 JSON 对象。比如访问 https://registry.npmjs.org/vue，就会看到 vue 模块所有版本的信息。
>
>在模块名后面，还可以加上版本号或者标签，用来查询某个具体版本的信息。https://registry.yarnpkg.com/vue/2.6.10，其返回的 JSON 对象里面，dist.shasum 属性相当于 hash 值，在 lock 和缓存时会使用到，dist.tarball 属性是该版本压缩包的地址，安装时从此获取

```json
dist: {
  "shasum": "a72b1a42a4d82a721ea438d1b6bf55e66195c637",
  "tarball":"https://registry.npmjs.org/vue/-/vue-2.6.10.tgz"
},
```

```bash
# 查看当前使用的镜像源
yarn config get registry

# 修改镜像源为淘宝源
yarn config set registry https://registry.npm.taobao.org/
```

# 依赖版本

> yarn 的包遵守 semver，即语义化版本。SemVer 是一套语义化版本控制的约定，定义的格式为
>
> X.Y.Z（主版本号.次版本号.修订号）：
>  X.主版本号：进行不向下兼容的修改时，递增主版本号
>  Y.次版本号: 做了向下兼容的新增功能或修改
>  Z.修订号：做了向下兼容的问题修复


> 字符 ^ 表明不会修改版本号中的**第一个非零数字**，^3.1.4 里的 3 或者 ^0.4.2 里的 4。版本号中缺少的部分将被 0 填充，且在匹配时这些位置允许改变。
> 使用 `yarn add [package-name]` 命令安装依赖，默认使用的是 ^ 范围。
> 需要注意的是，如果一个比较器包含有预发布标签的版本，它将只匹配有相同 major.minor.patch 的版本。 例如 >=3.1.4-beta.2，可以匹配 3.1.4-beta.3，但不会匹配 3.1.5-beta.3 版本。

# 依赖类型

依赖 | 定义
---- | ---
dependences    | 代码运行时所需要的依赖，比如 vue，vue-router。
devDependences | 开发依赖，就是那些只在开发过程中需要，而运行时不需要的依赖，比如 babel，webpack。
peerDependences| 同伴依赖，它用来告知宿主环境需要什么依赖以及依赖的版本范围。如果宿主环境没有对应版本的依赖，在安装依赖时会报出警告。
optionalDependencies| 可选依赖，这种依赖即便安装失败，Yarn 也会认为整个依赖安装过程是成功的。可选依赖适用于那些即便没有成功安装可选依赖，也有后备方案的情况。
bundledDependencies | 打包依赖，在发布包时，这个数组里的包都会被打包打包到最终的发布包里，需要注意 bundledDependencies 中的包必须是在devDependencies或dependencies声明过的。


# 缓存

> yarn 会将安装过的包缓存下来，这样再次安装相同包的时候，就不需要再去下载，而是直接从缓存文件中直接 copy 进来。

```bash
# 查看 yarn 的全局缓存目录
yarn cache dir 

# yarn 会将不通版本解压后的包存放在不同目录下，目录命名如下
# shasum 即当前版本在源获得的 json 中的 dist.shasum
npm-[package name]-[version]-[shasum]

# 列出已缓存的每个包
yarn cache list    

# 列出匹配指定模式的已缓存的包
yarn cache list --pattern <pattern>  
```

# yarn.lock

> yarn.lock 中会准确的存储每个依赖的具体版本信息，以保证在不同机器安装可以得到相同的结果。
> Yarn 在安装期间，只会使用当前项目的 yarn.lock 文件（即 顶级 yarn.lock 文件），会忽略任何依赖里面的 yarn.lock 文件。在顶级 yarn.lock 中包含需要锁定的整个依赖树里全部包版本的所有信息。
> yarn.lock 文件是在安装期间，由 yarn 自动生成的，并且由 yarn 来管理，不应该手动去更改，更不应该删除 yarn.lock 文件，且要提交到版本控制系统中，以免因为不同机器安装的包版本不一致引发问题。

```json
"@babel/code-frame@7.0.0-beta.54":
  version "@7.0.0-beta.54"          // 版本
  resolved "url#dist.shasum"        // 包地址
  dependencies:                     // 当前包的依赖包
    "@babel/highlight" "^7.0.0"  
```

# yarn install 过程

> 首次执行 yarn install 安装，会按照 package.json 中的语义化版本，去向 registry 进行查询，并获取到符合版本规则的最新的依赖包进行下载，并构建构建依赖关系树。 比如在 package.json 中指定 vue 的版本为 ^2.0.0，就会获取符合 2.x.x 的最高版本的包。然后自动生成 yarn.lock 文件，并生成缓存。

> 之后再执行 yarn install，会对比 package.json 中依赖版本范围和 yarn.lock 中版本号是否匹配。
>  1、版本号匹配，会根据 yarn.lock 中的 resolved 字段去查看缓存， 如果有缓存，直接copy，没有缓存则按照 resolved 字段的url去下载包。
>  2、版本号不匹配，根据 package.json 中的版本范围去 registry 查询，下载符合版本规则最新的包，并更新至 yarn.lock 中。


# yarn 自定义命令

> 可以通过配置 package.json 中的 scripts 脚本命令来自定义命令，执行安装的一些插件。如当我们需要删除 vue 项目，node_modules 文件夹过大，删除很费时，这时我们可以安装一个插件 rimraf，但安装到全局时并不能直接执行，这时借用 yarn 来执行就是一个很好的选择。

```json
// 一般全局的 package.json 在 C:\Users\user\AppData\Local\Yarn，npm 亦然
{
  "scripts": {
    "rimraf": "node_modules/.bin/rimraf"
  },
  "dependencies": {
    "@vue/cli-init": "^4.4.6",
    "@vue/cli-service-global": "^4.4.6",
    "rimraf": "^3.0.2"
  }
}

// 删除命令
yarn rimraf node_modules
```



# 参考资料


[前端工程师应该知道的yarn知识](https://mp.weixin.qq.com/s/xFEI8ZIRBTAXHKUMBPwW_Q)