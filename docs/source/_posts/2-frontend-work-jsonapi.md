---
title: 前端 | 本地json文件作为接口给前端提供数据
date: 2020-12-27
tags: 
- work
- Frontend
categories: notes
photos:
    - /blog/img/frontend.jpg
---

<br>
<!--more-->

> vue-cli 的话就直接 vue.config.js 配置开发环境

> 遇到的问题是后端给的接口为了安全并没有设置跨域（之后会放在一个服务器），在做本地测试的时候，又需要测试接口数据。此时就可以将接口的数据下载下来放在 json 文件里，再通过 json-server 发布至本地服务器默认 3000 端口，再在前端直接通过该端口接口访问数据。

```json
// 安装
npm i json-server -g

{
  "list": [
    {
      "id": 11,
      "products": ["胡萝卜", "白菜", "土豆", "豆腐", "青椒"]
    },
    {
      "id": 22,
      "products": ["香蕉", "苹果", "葡萄", "哈密瓜", "西瓜"]
    },
    {
      "id": 33,
      "products": ["牛肉", "羊肉", "鸡肉", "鱼肉", "鸭肉"]
    }
  ]
}

// 在 json 的目录下，启动 json-server 查看数据文件
json-server --watch db.json

// 再通过 http://localhost:3000/list 访问

// 启动配置 一般默认 3000 端口发布
Options:
  -c, --config                   Path to config file
                                                   [default: "json-server.json"]
  -p, --port                     Set port                        [default: 3000]
  -H, --host                     Set host                 [default: "localhost"]
  -w, --watch                    Watch file(s)                         [boolean]
  -r, --routes                   Path to routes file
  -m, --middlewares              Paths to middleware files               [array]
  -s, --static                   Set static files directory
      --read-only, --ro          Allow only GET requests               [boolean]
      --no-cors, --nc            Disable Cross-Origin Resource Sharing [boolean]
      --no-gzip, --ng            Disable GZIP Content-Encoding         [boolean]
  -S, --snapshots                Set snapshots directory          [default: "."]
  -d, --delay                    Add delay to responses (ms)
  -i, --id                       Set database id property (e.g. _id)
                                                                 [default: "id"]
      --foreignKeySuffix, --fks  Set foreign key suffix (e.g. _id as in post_id)
                                                                 [default: "Id"]
  -q, --quiet                    Suppress log messages from output     [boolean]
  -h, --help                     Show help                             [boolean]
  -v, --version                  Show version number                   [boolean]
```

