---
layout: post
title: 'Kubenetes环境搭建'
date: 2019-10-22
author: Tinson
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Kubenetes Docker
---

> 原文链接：https://www.jianshu.com/p/e5c056baa8ab

大家都知道，Docker 与 Kubernetes 握手言和了，今天我也来体验一下。毕竟我们使用 Google 的东西没那么容易，希望这篇技术笔记能帮大家节省一点点时间，知道坑在哪里。我把 Mac、Windows 平台分开描述，根据你所在的平台选择浏览。

## Mac 平台

### 步骤1：准备工作

1. 我们需要下载并安装 Docker（参见《Docker 安装笔记》）。 不同的版本对 kubernetes 支持的版本不一样，请在安装后查看具体的版本号。  

- Docker Version 18.06.1-ce-mac73 (26764) 》kubernetes 1.10.3
- Docker 2.0.0.0 》kubernetes 1.10.3
- Docker Version 2.0.0.3 (31259) 》kubernetes 1.10.11  

2. 从github上直接下载指定版本的 kubectl，根据你的操作系统，下载指定平台版本，比如 macOS 的：  

- [kubernetes 1.10.3](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#v1103) 下载地址： [kubernetes-client-darwin-amd64.tar.gz](https://dl.k8s.io/v1.10.3/kubernetes-client-darwin-amd64.tar.gz)
- [kubernetes 1.10.11](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#v11011) 下载地址：[kubernetes-client-darwin-amd64.tar.gz](https://dl.k8s.io/v1.10.11/kubernetes-client-darwin-amd64.tar.gz)  

下载后解压到某个目录下。  

打开终端命令行，cd进入这个目录，执行以下脚本，将其变更为可执行命令，同时移动到系统特定目录下。  

```shell
chmod +x kubectl && mv kubectl /usr/local/bin/kubectl
```  

我们可以看下 kubectl 的版本号：  

```shell
kubectl version
```  

结果如下：
<img src="/assets/img/kube-1.webp">  

### 步骤2：手动拉取 Kubernetes 需要的镜像
根据目前我用的版本 Version 2.0.0.3 (31259) 集成 Kubernetes: v1.10.11。我要想把 Kubernetes 启动起来，需要手动下载 Kubernetes 组件的镜像。然后，进入 Docker 图标下拉菜单 “Preferences” > “Kubernetes”，启用它。  

<img src="/assets/img/kube-2.webp">  

因为在阿里云上，有同步镜像的组件，我们就不需要翻到官网下载了。借鉴网上找到脚本 k8s-deploy，进行改良一下，加入了 Dashboard 组件进去。大家如果只使用 kubectl 来控制 Kubernetes 的话，可以自己将这部分去掉。对于新手来说，可能有个网页界面，看着舒服些。
不过需要注意的是，因为 Dashboard 的版本是单独演进的，要了解最新版本是多少，需要查看 kubernetes-dashboard.yaml 文件。  

```shell
...
image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
...
```
现在，创建一个脚本文件：docker-k8s-images.sh  

```shell
#!/bin/bash

set -e 
# Check version in https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
# Search "Running kubeadm without an internet connection"
# For running kubeadm without an internet connection you have to pre-pull the required master images for the version of choice:
KUBE_VERSION=v1.10.11
KUBE_DASHBOARD_VERSION=v1.10.1
KUBE_PAUSE_VERSION=3.1
ETCD_VERSION=3.1.12
DNS_VERSION=1.14.8
GCR_URL=k8s.gcr.io
ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers

images=(kube-proxy-amd64:${KUBE_VERSION}
kube-scheduler-amd64:${KUBE_VERSION}
kube-controller-manager-amd64:${KUBE_VERSION}
kube-apiserver-amd64:${KUBE_VERSION}
pause-amd64:${KUBE_PAUSE_VERSION}
etcd-amd64:${ETCD_VERSION}
k8s-dns-sidecar-amd64:${DNS_VERSION}
k8s-dns-kube-dns-amd64:${DNS_VERSION}
k8s-dns-dnsmasq-nanny-amd64:${DNS_VERSION}
kubernetes-dashboard-amd64:${KUBE_DASHBOARD_VERSION}) 

for imageName in ${images[@]} ; do
docker pull $ALIYUN_URL/$imageName
docker tag $ALIYUN_URL/$imageName $GCR_URL/$imageName
docker rmi $ALIYUN_URL/$imageName
done

docker images
```
备注：如果你查看到的 Dashboard 有新版本了，修改一下脚本中的  
KUBE_DASHBOARD_VERSION。  
然后，我们运行一下这个脚本：  
  
```shell
./docker-k8s-images.sh
```  

看到最终的运行结果：  

<img src="/assets/img/kube-3.webp">  

这时候我们再回到 Docker，就可以看到 Kubernetes 已正常启动了。  

<img src="/assets/img/kube-4.webp">  

针对老版本 Kunbernetes 1.10.3 版本，请更改相关配置：  
```shell
KUBE_VERSION=v1.10.3
KUBE_DASHBOARD_VERSION=v1.10.0
KUBE_PAUSE_VERSION=3.1
ETCD_VERSION=3.1.12
DNS_VERSION=1.14.8
```  

### 步骤3：启动 Kubernetes Dashboard（可选）
接下来，我们要想启动 Kubernetes Dashboard，还得在集群中部署一下 kubernetes-dashboard.yaml。  

```shell
kubectl create -f https://github.com/kubernetes/dashboard/tree/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```  

部署成功后，我们进行启动 proxy。  

```
kubectl proxy
```  

Starting to serve on 127.0.0.1:8001  

这时候，打开浏览器，访问 [Kubernetes Dashboard](http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)  

<img src="/assets/img/kube-5.webp">  

通过以下脚本，填写 kubeconfig 的 Token 信息（如果不操作这一步，就会提示 config 信息不全）。  

```
#!/bin/bash
TOKEN=$(kubectl -n kube-system describe secret default| awk '$1=="token:"{print $2}')
kubectl config set-credentials docker-for-desktop --token="${TOKEN}"
```    

选择 kubeconfig 文件，使用“shift + command + .”打开 $HOME 下隐藏目录文件 ./kube/config，点击“登录”，就可以认证成功，进入首页了。  

<img src="/assets/img/kube-6.webp">  

## Windows 平台
### 步骤1：准备工作
1. 我们需要下载并安装 Docker（参见《Docker 安装笔记》）。
从github上直接下载指定版本的 kubectl。  

2. 下载地址：[https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#v1103](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#v1103)
根据你的操作系统，下载指定平台版本，比如 Windows 的下载 [kubernetes-client-windows-amd64.tar.gz](https://dl.k8s.io/v1.10.3/kubernetes-client-windows-amd64.tar.gz)
下载后解压 kubectl.exe 文件到 C:\Windows\System32 目录下。

### 步骤2：手动拉取 Kubernetes 需要的镜像
根据目前我用的版本 Version 18.06.1-ce-win73 (19507) 集成 Kubernetes: v1.10.3。我要想把 Kubernetes 启动起来，需要手动下载 Kubernetes 组件的镜像。  

因为在阿里云上，有同步镜像的组件，我们就不需要翻到官网下载了。借鉴网上找到脚本 k8s-deploy，进行改良一下，加入了 Dashboard 组件进去。大家如果只使用 kubectl 来控制 Kubernetes 的话，可以自己将这部分去掉。对于新手来说，可能有个网页界面，看着舒服些。
不过需要注意的是，因为 Dashboard 的版本是单独演进的，要了解最新版本是多少，需要查看 kubernetes-dashboard.yaml 文件。  
  
```
...
image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
...
```

对于 Windows 平台，暂没编辑好的脚本，可采用最原始的方式，创建一个批处理文件：docker-k8s-images.bat  

```
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy-amd64:v1.10.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy-amd64:v1.10.3 k8s.gcr.io/kube-proxy-amd64:v1.10.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy-amd64:v1.10.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler-amd64:v1.10.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler-amd64:v1.10.3 k8s.gcr.io/kube-scheduler-amd64:v1.10.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler-amd64:v1.10.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager-amd64:v1.10.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager-amd64:v1.10.3 k8s.gcr.io/kube-controller-manager-amd64:v1.10.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager-amd64:v1.10.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver-amd64:v1.10.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver-amd64:v1.10.3 k8s.gcr.io/kube-apiserver-amd64:v1.10.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver-amd64:v1.10.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1 k8s.gcr.io/pause-amd64:3.1
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd-amd64:3.1.12
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd-amd64:3.1.12 k8s.gcr.io/etcd-amd64:3.1.12
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/etcd-amd64:3.1.12

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64:1.14.8
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64:1.14.8 k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.8
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64:1.14.8

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-kube-dns-amd64:1.14.8
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-kube-dns-amd64:1.14.8 k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.8
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-kube-dns-amd64:1.14.8

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.8
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.8 k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.8
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.8

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.0 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.0
```  

这时候我们再回到 Docker，就可以看到 Kubernetes 已正常启动了。

### 步骤3：启动 Kubernetes Dashboard（可选）
接下来，我们要想启动 Kubernetes Dashboard，还得在集群中部署一下 kubernetes-dashboard.yaml。  

```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```  

部署成功后，我们进行启动 proxy。

```
kubectl proxy
```  

Starting to serve on 127.0.0.1:8001

这时候，打开浏览器，访问 [Kubernetes Dashboard](http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

（待完善 kubeconfig 部分，可以先参考 Mac 平台的领悟一下）

恭喜您，成功启动 Kubernetes 的学习之路了