---
layout: post
title: 'Dockerfile构建debian-nginx-php多版本镜像'
date: 2019-07-29
author: Tinson
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Dockerfile PHP镜像
---

## Debian8.4+Nginx
### image
tinson/debian8.4-nginx
 
### size
260MB
 
### dockerfile
```
FROM debian:8.4
MAINTAINER Tinson

# update debian source
RUN echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list && \
	echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list && \
	apt-get update

# install common tool
RUN apt-get install -y curl vim cron

# install Nginx‘s PGP signing key
RUN curl -o nginx_signing.key http://nginx.org/keys/nginx_signing.key && \
	apt-key add nginx_signing.key

# add nginx source
RUN echo "deb http://nginx.org/packages/mainline/debian/ jessie nginx" >> /etc/apt/sources.list.d/nginx.list && \
	echo "deb-src http://nginx.org/packages/mainline/debian/ jessie nginx" >> /etc/apt/sources.list.d/nginx.list && \
	apt-get update

# install nignx	
RUN apt-get install -y nginx

# repair
RUN echo "y" |cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
	rm /etc/nginx/conf.d/default.conf && \
	sed -i 's/user  nginx/user  www-data/g'  /etc/nginx/nginx.conf && \
	sed -i '11,$d' /etc/crontab

EXPOSE 80

CMD ["/bin/bash"]
```

 


## Debian8.4+Nginx+PHP5.6
### image
tinson/debian8.4-nginx-php5.6
 
### size
310MB
 
### dockerfile
```
FROM tinson/debian8.4-nginx
MAINTAINER Tinson

# install php5.6
RUN apt-get install -y php5 php5-cli php5-fpm php5-common php5-mysql php5-mcrypt php5-gd php5-memcache php5-memcached php5-redis php5-msgpack && \
	ln -s /etc/init.d/php5-fpm /etc/init.d/php-fpm && \
	sed -i 's/listen = \/var\/run\/php5-fpm.sock/listen = \/var\/run\/php-fpm.sock/g'  /etc/php5/fpm/pool.d/www.conf
```
 
 
 


## Debian8.4+Nginx+PHP7source
### image
tinson/debian8.4-nginx-php7source
 
### size
284MB
 
### dockerfile
```
FROM tinson/debian8.4-nginx
MAINTAINER Tinson

# add php7.x source
RUN curl -s -o /etc/apt/trusted.gpg.d/php.gpg https://mirror.xtom.com.hk/sury/php/apt.gpg && \
	echo "deb https://mirror.xtom.com.hk/sury/php/ jessie main" | tee /etc/apt/sources.list.d/php.list && \
	rm -f /etc/apt/sources.list.d/nginx.list && \
	apt-get install -y apt-transport-https && \
	apt-get update
```
 
 


## Debian8.4+Nginx+PHP7.0
### image
tinson/debian8.4-nginx-php7.0
 
### size
352MB
 
### dockerfile
```
FROM tinson/debian8.4-nginx-php7source
MAINTAINER Tinson
 
RUN apt-get update

# install php7.0 and related general library
RUN apt-get install -y php7.0 php7.0-cli php7.0-fpm php7.0-common php7.0-mysql php7.0-mcrypt php7.0-gd php7.0-curl php7.0-msgpack php7.0-soap php7.0-xml php7.0-opcache php7.0-memcached php7.0-redis php7.0-mbstring php7.0-bcmath php7.0-zip && \
	ln -s /etc/init.d/php7.0-fpm /etc/init.d/php-fpm && \
	sed -i 's/listen = \/run\/php\/php7.0-fpm.sock/listen = \/var\/run\/php-fpm.sock/g'  /etc/php/7.0/fpm/pool.d/www.conf
```
 
 
 


## Debian8.4+Nginx+PHP7.1
### image
tinson/debian8.4-nginx-php7.1
 
### size
353MB
 
### dockerfile
```
FROM tinson/debian8.4-nginx-php7source
MAINTAINER Tinson
 
RUN apt-get update

# install php7.1 and related general library
RUN apt-get install -y php7.1 php7.1-cli php7.1-fpm php7.1-common php7.1-mysql php7.1-gd php7.1-curl php7.1-msgpack php7.1-soap php7.1-xml php7.1-opcache php7.1-mcrypt php7.1-memcached php7.1-redis php7.1-mbstring php7.1-bcmath php7.1-zip && \
	ln -s /etc/init.d/php7.1-fpm /etc/init.d/php-fpm && \
	sed -i 's/listen = \/run\/php\/php7.1-fpm.sock/listen = \/var\/run\/php-fpm.sock/g'  /etc/php/7.1/fpm/pool.d/www.conf
```
 
 
 


## Debian8.4+Nginx+PHP7.2
### image
tinson/debian8.4-nginx-php7.2
 
### size
355MB
 
### dockerfile
```
FROM tinson/debian8.4-nginx-php7source
MAINTAINER Tinson
 
RUN apt-get update

# install php7.2 and related general library
RUN apt-get install -y php7.2 php7.2-cli php7.2-fpm php7.2-common php7.2-mysql php7.2-gd php7.2-curl php7.2-soap php7.2-msgpack php7.2-xml php7.2-opcache php7.2-memcached php7.2-redis php7.2-mbstring php7.2-bcmath php7.2-zip && \
	ln -s /etc/init.d/php7.2-fpm /etc/init.d/php-fpm && \
	sed -i 's/listen = \/run\/php\/php7.2-fpm.sock/listen = \/var\/run\/php-fpm.sock/g'  /etc/php/7.2/fpm/pool.d/www.conf
```
 
 
 


## Debian8.4+Nginx+PHP7.3
### image
tinson/debian8.4-nginx-php7.3
 
### size
355MB
 
### dockerfile
```
FROM tinson/debian8.4-nginx-php7source
MAINTAINER Tinson
 
RUN apt-get update

# install php7.3 and related general library
RUN apt-get install -y php7.3 php7.3-cli php7.3-fpm php7.3-common php7.3-mysql php7.3-gd php7.3-curl php7.3-msgpack php7.3-soap php7.3-xml php7.3-opcache php7.3-memcached php7.3-redis php7.3-mbstring php7.3-bcmath php7.3-zip && \
	ln -s /etc/init.d/php7.3-fpm /etc/init.d/php-fpm && \
	sed -i 's/listen = \/run\/php\/php7.3-fpm.sock/listen = \/var\/run\/php-fpm.sock/g'  /etc/php/7.3/fpm/pool.d/www.conf
```
 
 
 


## CentOS6.6+Nginx+PHP5.6
### image
tinson/centos-nginx-php5.6
 
### size
480MB
 
### dockerfile
```
FROM centos:6.6
MAINTAINER Tinson

# 调整时区
RUN /bin/cp -r /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 修复curl ssl问题
RUN yum -y update nss

# 配置nginx
RUN rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm && yum install -y nginx && rm -fr /etc/nginx/conf.d/*

# 配置PHP 
RUN rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm && rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm
RUN yum -y install php56w php56w-common php56w-fpm php56w-cli php56w-gd php56w-mcrypt php56w-mbstring php56w-mysql php56w-pdo php56w-soap php56w-pecl-memcached php56w-pecl-redis

# 可选删除安装文件
RUN yum clean all

EXPOSE 80

CMD ["/bin/bash"]
```




## Debain8.4+Go1.12.5+nodev12.3.1
### image
zouzhiyong0513/debian8.4-golang

### size
648MB
 
### dockerfile
```
FROM debian:8.4
MAINTAINER Jimmy

# update debian source
RUN echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list && \
	echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list && \
	apt-get update

# install common tool
RUN apt-get install -y curl vim cron xz-utils

# repair
RUN echo "y" |cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
	sed -i '11,$d' /etc/crontab

# install golang
RUN cd ~ && \
	mkdir -m 777 go && \
	cd go && \
    curl -o go1.12.5.tar.gz https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz && \
	tar -xzvf go1.12.5.tar.gz && \
	rm -rf go1.12.5.tar.gz && \
	mv ~/go/go ~/go/go1.12.5 && \
	rm -rf ~/go/go && \
	mkdir -m 777 -p ~/go/bin ~/go/pkg ~/go/src && \
	echo "export PATH=/root/go/go/bin:$PATH" >> ~/.bashrc && \
	echo "export GOPATH=/root/go" >> ~/.bashrc && \
	echo "export GOBIN=/root/go/bin" >> ~/.bashrc && \
	/bin/bash -c "source /root/.bashrc"

# reset PATH
ENV PATH "/root/go/go1.12.5/bin:$PATH"

# install nodejs
RUN cd ~ && \
	mkdir -m 777 nodejs && \
	cd nodejs && \
	curl -o node-v12.3.1-linux-x64.tar.xz https://nodejs.org/dist/v12.3.1/node-v12.3.1-linux-x64.tar.xz && \
	tar -xJvf node-v12.3.1-linux-x64.tar.xz && \
	rm -rf node-v12.3.1-linux-x64.tar.xz && \
	echo "export PATH=/root/nodejs/node-v12.3.1-linux-x64/bin:$PATH" >> ~/.bashrc && \
	/bin/bash -c "source /root/.bashrc"

# reset PATH
ENV PATH "/root/nodejs/node-v12.3.1-linux-x64/bin:$PATH"

# install yarn
RUN /bin/bash -c "npm install -g yarn"

EXPOSE 80

CMD ["/bin/bash"]
```
