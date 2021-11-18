---
title: 面试 | 浏览器
date: 2020-8-29
tags: 
  - interview
  - Frontend
categories: notes
hide: true
photos:
    - /blog/img/interview.jpg
---


<br>
<!--more-->

# cookie、localStorage、sessionStorage

cookie 是网站用来标识用户身份而存储在用户本地终端上的数据，cookie 数据始终在同源的http请求中携带，会在浏览器和服务器间传递
localStorage、sessionStorage 仅在本地保存，不参与服务器通信


相同点：都保存在客户端。
不同点：有效期、大小、服务器端通信方面
cookie：可设置有效期，默认浏览器关闭失效。大小4K。参与服务器端通信，携带在请求头里。
localStorage：永久有效，除非手动删除。大小5M。不参与服务器端通信。
sessionStorage：当前会话期有效。大小5M。不参与服务器端通信。

# cookie 有哪些参数

setcookie()函数向客户端发送一个 HTTP cookie。

```js
setcookie(name, value, expire, path, domain, secure)
```

参数	描述
name	  必需。规定 cookie 的名称。
value	  必需。规定 cookie 的值。
expire	可选。规定 cookie 的有效期。
path	  可选。规定 cookie 的服务器路径。
domain	可选。规定 cookie 的域名。
secure	可选。规定是否通过安全的 HTTPS 连接来传输 cookie
httpOnly 是否能通过js脚本获取cookie

# 浏览器多个标签页面之间通信

1. cookie + setInterval

cookie 是浏览器本地存储机制，通过 `document.cookie = "msg=value"` , 注意 cookie 是 'key = value' 的形式存储的

跨标签接受数据时，通过 document.cookie 获取，若是有多个键值，可以将获得的字符串进行处理通过 JSON.parse 转换为对象，再通过 key 提取。通过把 cookie 里的等号换成冒号， 分号空格换成逗号空格，最后再加大括号

从发送消息页到接收页并不能实时更新，需要手动刷新。因此可以使用 setInterval 定时器解决

```js
function getValue(key){
  var cookies = '{"'+document.cookie.replace(/=/g,'":"').replace(/;\s+/g,'", "')+'"}';
  cookies = JSON.parse(cookies);
  return cookies[key];
}

setInterval(function(){
  recMsg.innerHTML = getValue("msg1");
},500)
```

缺点：
1) 容量小，每个域名最多设置 30-50 个 cookie，容量就 4k
2) 每次 http 请求会把当前域的 cookie 都发送到服务器，包括只在本地使用的，浪费带宽
3) setInteval 频率过高影响浏览器性能，过低又影响时效

2. localStorage

localStorage 存东西会自动触发整个浏览器的 storage 事件，别的标签窗口也都可以获取到之前存的东西，只需要监听 storage 事件就可以

```js
localStorage.setItem("msg", msg.value)

// get
function load(){
  recMsg.innerHTML = localStorage.getItem("msg1");
}

window.addEventListener("storage", function(){
  load();
})
```

3. websocket

websocket 用到了服务端，需将消息先发送到 WebSocketServer, WebSocketServer 再实时把消息发给其他使用页面，完成实时通信

server: 创建 websockect 对象实例，添加 connection 监听事件, 为每个客户端对象绑定 message 事件

send: 先建立 websocket 连接, 接着用该实例发送消息

receive: 先建立 websocket 连接, 接着绑定连接打开时的回调, 里边注册接收消息时的处理函数

```js
// server.js - 服务端
//获得WebSocketServer类型
var WebSocketServer = require('ws').Server;
//创建WebSocketServer对象实例，监听指定端口
var wss = new WebSocketServer({ port: 8080 });
//创建保存所有已连接到服务器的客户端对象的数组
var clients=[]; 
//为服务器添加connection事件监听，当有客户端连接到服务端时，立刻将客户端对象保存进数组中。
wss.on('connection', function(client) {
  console.log("一个客户端连接到服务器");
  // 如果没有这个client对象，说明是第一次连接，就加入到clients中
  if(clients.indexOf(client)===-1){
    clients.push(client);
    console.log("有"+clients.length+"个客户端在线");
    //为每个client对象绑定message事件，当某个客户端发来消息时，自动触发
    client.on('message', function (msg) {
      console.log("收到消息:"+msg);
      //遍历clients数组中每个其他客户端对象，并发送消息给其他客户端
      for(var c of clients){
        if(c!=client){
          c.send(msg);
        }
      }
    })
  }
});

// send - 客户端
//建立到服务端webSocket连接
var ws = new WebSocket("ws://localhost:8080");
ws.onclick = function(){
  if(msg.value.trim()!==""){
    // 将消息发到服务器
    ws.send(msg.value.trim());
  }
}

// receive
//建立到服务端webSocket连接
var ws = new WebSocket("ws://localhost:8080");
//当连接被打开时，注册接收消息的处理函数
ws.onopen = function(event) {
  //当有消息发过来时，就将消息放到显示元素上
  ws.onmessage=function(event) {
    recMsg.innerHTML=event.data;
  }
}
```

# websocket

websocket 是一种基于http协议的协议，由于http协议是非状态性的，客户端有请求才会响应，关闭连接后需要再此鉴别信息建立新连接，而websocket可以服务端主动通信，
websocket需要进行一次http连接，标记upgrade为websocket，也就是连接升级为websocket，服务端确认后就保持websocket连接；

websocket在服务端创建server，绑定connection事件，在回调中参数即为client，给client绑定message来对client收到的消息进行处理，再用send发送数据给客户端；而客户端通过创建websocket实例来连接到服务端的websocket服务器，使用onopen/onmessage接收服务端的数据,使用onclick/onsend发送数据给服务端

# 前端如何处理大量数据

使用分页,分表数据传输, 异步渲染, 可见区域局部先渲染, 

使用 web worker 来处理大量数据计算
在面对大量数据时, 可以使用 web worker 来创建独立于主线程的一个子线程来处理运算, 这样就不会造成页面卡死
使用 new work(url, options) 来读取网络 js 文件创建 worker 实例, 通过 postMessage 来向 worker 实例发送消息, 用 onmessage 来接收处理完的数据; 在 work.js 里通过 onmessage 接收页面传过来的数据, 进行计算, 完后 postMessage 发送处理完的数据; 这样形成一个通信

worker 接收的脚本文件必须与主线程的脚本文件同源, 且不能读取本地文件系统的文件;
worker 与主线程不在一个上下文环境, 无法读取到 dom 对象, 但可以获得 navigator location 对象, worker 必须与主线程通过消息的形式通信; worker 可以使用 xhr 发出 ajax 请求


# 输入 url 到页面加载发生了些什么

1. 解析域名：搜索本地浏览器 DNS 服务器，如没用则搜索系统的，再是路由器，运营商
2. TCP 连接：三次握手、四次挥手
3. 发送请求报文
4. 接受响应
5. 渲染页面：解析 HTML，生成 DOM 树；解析 CSS，生成 CSSOM 树；结合 DOM 和 CSSOM，渲染树；回流（layout），得到各元素几何信息；重绘（painting），得到绝对像素；展示（display），将像素发给 GPU，渲染至页面

# get 和 post 有哪些区别

区别 | GET | POST
---- | --- | ---
大小 | GET 传输一般 2K-8K，IE 限制 2K | 无限制
安全 | url 明文传输，不安全 | 通过 body 传输，不安全
编码方式 | url 编码 | 支持多种编码
TCP 数据包 | 1个数据包 | 2个数据包
浏览器记录 | 浏览器会记录 | 不会记录
浏览器后退 | 无影响 | 再次提交
浏览器收藏 | 可收藏 | 不可
浏览器缓存 | 可缓存 | 不可
使用方式 | 获取数据 | 提交保存数据

# 跨域问题 cors （字节、腾讯）

允许浏览器向跨域服务器发出 XMLHTTPRequest 请求。
所谓同源策略即是：协议、域名、端口三者相等

1. 浏览器端会自动向请求头部添加 origin 字段，标注请求来源
2. 在服务器端设置响应头的允许方法、头部、源等
3. 请求分为两种：简单请求和非简单请求；简单请求的头部可包含 GET/POST/HEAD；非简单请求头部有 PUT/DELETE

> 对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。

```json
// 简单请求
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

// 服务器返回的响应
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

> 非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。
> 过程：
> 非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段
> 浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

1. 预检请求
除了Origin字段，"预检"请求的头信息包括两个特殊字段。
Access-Control-Request-Method 该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。
Access-Control-Request-Headers 该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段

2. 预检请求的回应
Access-Control-Allow-Methods 该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。
Access-Control-Allow-Headers 如果浏览器请求包含该字段，则该字段是必需的。表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。
Access-Control-Allow-Credentials
Access-Control-Max-Age 可选，用来指定本次预检请求的有效期，单位为秒

3. 浏览器的正常请求和回应
与简单请求一样，直接发送cors请求

```json
// 预检请求
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header

// 预检请求的回应
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000

// 正常cors请求
PUT /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

// 响应
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
```

> 与 JSONP 相比，JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。

# 事件循环

同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；
异步任务指的是，不进入主线程、而进入任务队列（task queue）的任务，只有等主线程任务执行完毕，任务队列开始通知主线程，请求执行任务，该任务才会进入主线程执行。

当某个宏任务执行完后,会查看是否有微任务队列。如果有，先执行微任务队列中的所有任务；如果没有，在执行环境栈中会读取宏任务队列中排在最前的任务；执行宏任务的过程中，遇到微任务，依次加入微任务队列。栈空后，再次读取微任务队列里的任务，依次类推。

同步（Promise）>异步（微任务（process.nextTick ，Promises.then, Promise.catch ，resove,reject,MutationObserver)>宏任务（setTimeout，setInterval，setImmediate））

# 权限验证

vue 项目中，在登陆页面输入账号密码后，后端返回一个标识用户身份的 token 存在本地 cookie ，
用户登录成功后使用 router.beforeEach 拦截路由，判断是否获得 token，再用 token 获取用户的基本信息及权限 role ，
将 role 和路由表的每个页面需要的权限作比较，生成当前用户可访问的路由表，
调用 router.addRoutes 添加用户可访问的路由，
使用 vuex 管理路由表，根据 vuex 中可访问的路由渲染侧边栏组件

```js
// main.js
router.beforeEach((to, from, next) => {
  if (store.getters.token) { // 判断是否有token
    if (to.path === '/login') {
      next({ path: '/' });
    } else {
      if (store.getters.roles.length === 0) { // 判断当前用户是否已拉取完user_info信息
        store.dispatch('GetInfo').then(res => { // 拉取info
          const roles = res.data.role;
          store.dispatch('GenerateRoutes', { roles }).then(() => { // 生成可访问的路由表
            router.addRoutes(store.getters.addRouters) // 动态添加可访问路由表
            next({ ...to, replace: true }) // hack方法 确保addRoutes已完成 ,set the replace: true so the navigation will not leave a history record
          })
        }).catch(err => {
          console.log(err);
        });
      } else {
        next() //当有用户权限的时候，说明所有可访问路由已生成 如访问没权限的全面会自动进入404页面
      }
    }
  } else {
    if (whiteList.indexOf(to.path) !== -1) { // 在免登录白名单，直接进入
      next();
    } else {
      next('/login'); // 否则全部重定向到登录页
    }
  }
});
```

```js
// store/permission.js
function hasPermission(roles, route) {
  if (route.meta && route.meta.role) {
    return roles.some(role => route.meta.role.indexOf(role) >= 0)
  } else {
    return true
  }
}

function filterAsyncRoutes(routes, roles) {
  const res = []
  routes.forEach(route => {
    const tmp = { ...route }
    if (hasPermission(roles, tmp)) {
      if (tmp.children) {
        tmp.children = filterAsyncRoutes(tmp.children, roles)
      }
      res.push(tmp)
    }
  })
  return res
}

const permission = {
  state: {
    routers: constantRouterMap,
    addRouters: []
  },
  mutations: {
    SET_ROUTERS: (state, routers) => {
      state.addRouters = routers;
      state.routers = constantRouterMap.concat(routers);
    }
  },
  actions: {
    generateRoutes({ commit }, roles) {
      return new Promise(resolve => {
        let accessedRoutes
        if (roles.includes('admin')) {
          accessedRoutes = asyncRoutes || []
        } else {
          accessedRoutes = filterAsyncRoutes(asyncRoutes, roles)
        }
        commit('SET_ROUTES', accessedRoutes)
        resolve(accessedRoutes)
      })
    }
  }
};
```