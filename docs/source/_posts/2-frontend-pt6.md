---
title: 前端 | Vue + Express + MongoDB 制作电商应用
date: 2020-7-16 8:20:33
tags: 
- Vue
- Express
- Frontend
- MongoDB
photos:
    - /blog/img/vue.jpg
categories:
- practice
---
<br>
<!-- more -->

# page

[http://www.lixupeng.cn:8082](http://www.lixupeng.cn:8082/)

# Vue

1. 新版的 @vue/cli 与老版的 vue-cli 在初始化项目（`vue init webpack my-project`）上有区别，可以安装 @vue/cli-init，但安装后不管用。于是直接 `vue create my-project`，选择 eslint，vue-router

2. 相比原教程，新版的 vue-router 使用 view 文件夹将 components 中页面级别的组件单独提出来。

3. 使用 eslint 检测真的很严格，子对象后要有逗号，语句后要有封号，简写函数不能和括号有空格（data() {}），.vue 页最后要留一空行，标签与其他语句至多空一行，换行的规则要是 LF，windows 的 CRLF 不行，这个问题看有的回答说 改 eslintrc.js，不管用，直接在 vscode 右下角改了

4. Vue 在写 v-for 循环语法时，使用了一个额外的标签 template 来包裹我们需要渲染的 HTML 元素

5. 当前组件 list 通过计算属性从 this.$store.state 获取 vuex.Store 实例中存储的数组数据 products，可以在当前组件利用 v-for 得到数组元素 product，再通过 v-for 中遍历输出的子组件 item 的属性绑定 bind:product，这样就可以在子组件 item 中，通过 props 来获取 product 的值

# Express

1. 使用 yarn 安装 express，express 命令不能执行，但 yarn 文件夹下 bin 内有 express，因此将 yarn 文件夹添加到环境变量下，再执行下 `yarn global install`，可能之前 yarn 没加入环境变量，导致它 bin 下的都运行不了

2. express 默认使用 jade 预编译，想用 ejs 可以后期安装，或者创建项目时，`express --ejs my-project`

3. 设置跨源，直接给 express 中间件设置请求信息的处理方法；或者安装 cors 模块来配置

4. 在导入模块时，查看模块源码都是 export = xxx，因此在引入时需要注意，为整体导入，可以使用 const ... = require 或 import * as ... from 或 import ... = require（ts 官方推荐，但仅适用于 ts 文件）

5. express 作为单独的后端应用发布，所以前端使用的后端接口应为对应服务器的 ip，而不能是 localhost 或 127.0.0.0，比如 vue 项目发布后，在局域网访问 192.168.1.7:8080，此时如果 actions 里使用的后端接口是 localhost，电脑当然没问题，后端项目就发布在本地 3000 端口上，但此时局域网下手机访问则不行，因为手机本地没用后端项目，所以在接口此时应该用 192.168.1.7:3000


```js
// CORS config here
app.all('/*', function(req, res, next) {
  // CORS headers
  res.header('Access-Control-Allow-Credentials', true); //允许后端发送cookie
  res.header("Access-Control-Allow-Origin", "*"); // restrict it to the required domain
  res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS');
  // Set custom headers for CORS
  res.header('Access-Control-Allow-Headers', 'Content-type,Accept,X-Access-Token,X-Key');
  // 
  req.method === 'OPTIONS' ? res.status(204).end() : next()
});
```

4. 编写 ERST API，相当于 MVC 模式里的 controller，设定 express 路由方法，方法参数为路由地址和具体对象（model）的方法，分别编写每个数据模型的操控方法，MC 的关联。此外测试 API 可以使用 VS Code 插件 REST Client，直接编写 `POST/GET url HTTP/1.1`，下方指定 json 数据类型就可直接请求了。






# MongoDB


1. 需要在 connect 方法添加第二个参数申明，不然会警告当前服务器发现和监视引擎已弃用，将在将来的版本中删除

```js
const mongoose = require('mongoose');

// connect mongodb
mongoose.connect(`mongodb://localhost:27017/test`, {
  useNewUrlParser: true,
  useUnifiedTopology: true     // 在第二个参数添加设置
}).then(res => {
  console.log('mongodb connected')
});
```


# 环境部署

```bash
# 生产环境部署
# 安装 nginx
apt-get install nginx

service nginx status # Failed to read PID from file /run/nginx.pid 

# nginx 启动需要时间，而 systemd 在 nginx 完成启动前就去读取 pid file
# 造成读取 pid 失败
mkdir -p /etc/systemd/system/nginx.service.d
printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" >/etc/systemd/system/nginx.service.d/override.conf
systemctl daemon-reload
systemctl restart nginx.service

# 此时 nginx 正常运行

# 把前后端资源都放在服务器上
# 打包前端项目 
yarn build

# 运行后端项目
yarn start

# 生产环境有 nginx 就可
# 将打包后的 dist 放在指定目录，在 nginx 配置里说明项目根目录即可，
# nginx 此时便起 web 服务器的作为



# 开发环境也可以部署在服务器上，等于 linux 操作系统搞开发，
# 只不过发布的测试环境互联网就能看到
# 将 vue 开发环境依赖安装齐全，发布后访问 8080 端口即可
# 只不过 linux 上安装依赖这一块会碰到很多问题，yarn npm 安装的问题

# 安装 node npm
apt-get update

apt-get install nodejs
apt-get install nodejs-legacy
apt-get install npm

node -v # 8.10.0
npm -v # 3.5.2

# 更新 node npm
npm install -g n
n stable

command -v npm #/usr/bin/npm
# 最新版的 npm 的位置在 /usr/local/bin/npm，且已经正常运行
# Bash 的缓存依然显示的是旧版的 /usr/bin/npm
# 清理 bash 缓存
hash -d npm

command -v npm # /usr/local/bin/npm

node -v # v12.18.3
npm -v # 3.5.2


# 安装 mongodb
apt-get install mongodb

systemctl status mongodb # 可看到已启动

mongo # 进入 mongo shell


# 安装 @vue/cli
yarn global add @vue/cli
yarn global add @vue/cli-service


# 部署
git clone https://github.com/arginsen/Vue-Express.git


# 在后端 data 下给 mongodb 导入数据
mongoimport -d test -c manufacturers manufacturers.json
mongoimport -d test -c products products.json


# 分别安装前后端的依赖
yarn install

# 运行项目 命令可自定义 也可以直接写个 shell 脚本
# 后端
yarn start
# 前端
yarn serve
```


# 参考链接

[node 设置 cors，解决跨源](https://segmentfault.com/a/1190000022512695)
[node 跨域 cors 模块](https://www.jianshu.com/p/f650dfad5574)
[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
[typescript export](https://ts.xcatliu.com/basics/declaration-files.html#export-%3D)
[VS Code 上调试并共享你的 REST API 调用](https://mp.weixin.qq.com/s/0tXWPUtnx0bUB1kfU2i_kg)
[ESlint 规则配置](https://my.oschina.net/u/4408277/blog/3344005)
[ESlint 规则](https://cloud.tencent.com/developer/section/1135651)
[Ubuntu 18.04 安装的 NPM](https://www.jianshu.com/p/1ca21344a4b2)
