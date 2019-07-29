---
layout: post
title: 'Docker-Composer搭建本地测试环境'
date: 2019-07-29
author: Tinson
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: GoLand debug Go
---

## Docker开发环境
推荐使用Docker方式搭建开发环境，项目负责人需在项目根目录下配置docker-composer.yml，参考案例如下：  
```
version: "3"

services:
   frameworks.dev:
     image: tinson/centos6.8-php7.0
     expose:
       - 80
     depends_on:
       - memcache.dev
       - redis.dev
     volumes:
       - ".:/www/frameworks"
     restart: always
     environment:
       VIRTUAL_HOST: api.frameworks-dev.hetiansu.com,admin.frameworks-dev.hetiansu.com
       YII_ENV: "dev"
     container_name: frameworks.dev
     entrypoint: "/www/frameworks/assets/sh/entrypoint.sh"
   memcache.dev:
     image: memcached
     restart: always
     container_name: memcache.frameworks.dev
   redis.dev:
     image: redis
     restart: always
     container_name: redis.frameworks.dev

networks:
  default:
    external:
      name: nginx-proxy
```

### 搭建Docker环境

* 安装Docker环境
	* MAC 
		* 官网：`https://www.docker.com/docker-mac`
		* 链接: `https://pan.baidu.com/s/1vM819JjWaXT2e8btT4qq6Q 密码: ayz3`
	* WINDOWS 
		* 官网：`https://www.docker.com/docker-windows`
		* 链接：`链接: https://pan.baidu.com/s/1WajtmjcdjS-0E3HPEEzpjg 密码: 8cd3`


* 启动Nginx-Proxy代理
	* 创建本地开发环境docker Network
	```
	$ docker network create nginx-proxy
	```
	* 启动Nginx-Proxy镜像，该镜像将监控上一步创建的nginx-proxy network内的所有新VIRTUHOST添加，并自动增加反向代理
	```
	$ docker run -d -p 80:80 --name nginx-proxy --net nginx-proxy --restart always -v /var/run/docker.sock:/tmp/docker.sock jwilder/nginx-proxy
	```

### 启动项目服务
<pre>
$ git clone {项目git地址} {本地项目目录}   # 拷贝代码仓库到本地目录
$ cd {本地项目目录}         # 进入本地目录
$ docker-compose up --force-recreate --remove-orphans -d
$ docker ps       # 查看执行的容器
$ docker exec -it {container_name} sh # 登入进容器
</pre>

### 配置本地解析
给项目相关域名配置本地host
127.0.0.1 admin.frameworks-dev.hetiansu.com api.frameworks-dev.hetiansu.com

### 附加说明
如果本地已经有搭建了Web服务器，上述启动Nginx-Proxy会有80端口冲突问题，可修改端口映射规则，命令如下
```
$ docker run -d -p 20001:80  --name nginx-proxy --net 	nginx-proxy --restart always -v /var/run/docker.sock:/tmp/docker.sock jwilder/nginx-proxy
```

另外需在Web服务配置代理转发，以下为Nginx的配置模板
<pre>
server {
	listen 80;
	server_name admin.frameworks-dev.hetiansu.com api.frameworks-dev.hetiansu.com;
	location / {
			proxy_redirect off;
        	proxy_set_header Host $host;
        	proxy_set_header X-Real-IP $remote_addr;
        	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        	proxy_pass   http://127.0.0.1:20001;
	}
}
</pre>

### docker相关命令
一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose，不再需要使用shell脚本来启动容器。在配置文件中，所有的容器通过services来定义，然后使用docker-compose脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器。完整的命令列表如下：  
<pre>
        build 构建或重建服务
        help 命令帮助
        kill 杀掉容器
        logs 显示容器的输出内容
        port 打印绑定的开放端口
        ps 显示容器
        pull 拉取服务镜像
        restart 重启服务
        rm 删除停止的容器
        run 运行一个一次性命令
        scale 设置服务的容器数目
        start 开启服务
        stop 停止服务
        up 创建并启动容器
</pre>

#### 常用docker命令
<pre>
创建容器: docker-compose up --force-recreate --remove-orphans  前台启动(-d 后台启动)
进入容器: docker exec -it ${appName}.dev sh
开启容器: docker-compose start
停止容器: docker-compose stop
删除容器: docker-compose down
</pre>
