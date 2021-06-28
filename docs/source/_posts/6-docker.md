---
title: docker 学习笔记
date: 2020-8-15 8:20:33
tags: 
- Docker
photos:
    - /blog/img/docker.jpg
categories:
- notes
---
<br>
<!-- more -->


# 安装

```bash
# ubuntu 系统
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 开机启动
sudo systemctl enable docker
```


# 使用

```bash
# 查看容器运行状态
docker ps

# 查看所有容器 包括停止的
docker ps -a

# 停止容器
docker stop my-nginx

# 销毁容器
docker rm my-nginx

# 删除运行/暂停中的容器
docker rm -f my-nginx
# 强制删除所有
docker rm -f $(docker ps -aq)
```

## 运行一个 nginx 服务器

```bash
# 将本机 8080 端口映射到 docker 的 80
docker run -p 8080:80 nginx

# 后台运行 Nginx，--name 指定容器名，-d（--detach）后台运行
docker run -p 8080:80 --name my-nginx -d nginx
```

这个过程，Docker 实现了：
1. 检查本地是否有指定的 nginx:latest 镜像，如果没有，执行第 2 步，否则直接执行第 3 步
2. 本地没有指定镜像（Unable to find xxx locally），从 Docker Hub 下载到本地
3. 根据本地的 nginx:latest 镜像创建一个新的容器，并通过 `-p（--publish）`参数建立本机的 8080 端口与容器的 80 端口之间的映射，然后运行其中的程序
4. Nginx 服务器程序保持运行，容器也不会退出


## 运行一个 ubuntu 操作系统

```bash
# -i（--interactive，交互式模式）， -t（--tty，分配一个模拟终端） 
docker run -it --name dreamland ubuntu
```

# 容器生命周期

![](/blog/img/2020/docker/lifecycle.png)

## 自然结束

![](/blog/img/2020/docker/nature.png)

1. 通过 docker run 命令，直接创建（create）并启动（start）一个容器，进入到运行状态（Running）
2. 然后程序运行结束（例如输出 Hello World 之后，或者通过 Ctrl + C 使得程序终止），容器死亡（die）
3. 由于没有设置重启策略，所以直接进入到停止状态（Stopped）
4. 最后通过 docker rm 命令销毁容器，进入到被删除状态（Deleted）

## 强制结束

![](/blog/img/2020/docker/force.png)

1. 通过 docker run 命令，直接创建（create）并启动（start）一个容器，进入到运行状态（Running）
2. 然后通过 docker stop 命令杀死容器中的程序（die）并停止（stop）容器，最终进入到停止状态（Stopped）
3. 最后通过 docker rm 命令销毁容器，进入到被删除状态（Deleted）


# 容器化应用

容器化包括三个方面:
- 编写应用代码
- 构建镜像
- 创建和运行镜像


## 构建镜像

构建 Docker 镜像主要包括两种方式：
- 手动：根据现有的镜像创建并运行一个容器，进入其中进行修改，然后运行 docker commit 命令根据修改后的容器创建新的镜像
- 自动：创建 Dockerfile 文件，指定构建镜像的命令，然后通过 docker build 命令直接创建镜像

### 以一个前端服务来说

> 前端服务通过 nginx 发布在互联网

```bash
# 配置 nginx.conf 放在前端项目根目录
server {
    listen 80;
    root /www;
    index index.html;
    sendfile on;
    sendfile_max_chunk 1M;
    tcp_nopush on;
    gzip_static on;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

#### 配置 Dockerfile

> Dockerfile 实际上是默认名称，我们当然可以取一个别的名字，例如 myDockerfile，然后在构建镜像时 `docker build` 指定 -f（--file）参数即可

```bash
# Dockerfile 放在前端项目根目录
# 给应用的客户端配置 nginx web 服务器
FROM nginx:1.13

# 删除 Nginx 的默认配置
RUN rm /etc/nginx/conf.d/default.conf

# 添加自定义 Nginx 配置
COPY config/nginx.conf /etc/nginx/conf.d/

# 将前端静态文件拷贝到容器的 /www 目录下
COPY dist /www
```

#### 配置 .dockerignore

```bash
# .dockerignore 放在前端项目根目录
node_modules
```

### 构建并运行

```bash
# 在前端项目根目录构建
# docker 会默认读取 Dockerfile 的配置，
# -t（--tag，容器标签）给镜像打上标签，得到一个新镜像
docker build -t app-client .

# 运行容器 
# 端口映射规则为 8080:80，容器名称为 client，-d 设置为后台运行
docker run -p 8080:80 --name client -d app-client
```


# 容器架构

![](/blog/img/2020/docker/framework.png)


# 参考资料

[一杯茶的时间，上手 Docker](https://tuture.co/2020/01/01/442cc8d/)