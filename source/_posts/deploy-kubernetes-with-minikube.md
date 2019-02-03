---
title: 通过minikube 部署kubernetes单节点集群
date: 2019-02-03 16:11:13
tags:
categories: kubernetes
---

minikube 是由kubernetes社区提供的一款工具，可以快速的创建一个k8s单节点集群环境。方便大家开发和测试。

官方提供了很详细的文档，但是因为众所周知的原因，在国内无法拉取相关的k8s组件镜像。即使使用科学上网，也有无数的坑。这里，我们直接使用阿里云提供的minikube镜像来安装。

安装minikube 之前，首先确保你的电脑已经安装了virtualbox。minikube 会创建一个virtualbox 虚拟机，并利用这个虚拟机部署k8s集群。

![yq.aliyun.com](https://yqfile.alicdn.com/c03a43e0731ca579d1844fb44269fd2fd257bfb3.jpeg)

### 安装 kubectl
首先，需要在本地机器上安装 [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。如果不能安装请考虑使用其他镜像安装或者参考我的上一篇文章，这里就不在赘述了。

{% post_link kubernetes-deployment 点击这里查看 %}

### 安装 minikube 

#### Linux
```shell
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.30.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
#### Mac OSX
```shell
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.25.2/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
#### Windows

下载 [minikube-windows-amd64.exe](http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.25.2/minikube-windows-amd64.exe?spm=a2c4e.11153940.blogcont221687.29.7dd54cecTB6Vxi&file=minikube-windows-amd64.exe) 文件，并重命名为 minikube.exe

### 启动 minikube

注意，一定要添加 `--registry-mirror` 选项，出于某些不可描述的原因，你懂的。

```shell
minikube start --registry-mirror=https://registry.docker-cn.com
```
可能需要花一些时间，因为要拉取kubelet、kube-proxy等一些组件的镜像，快慢取决于你的网络情况。

等待安装完成以后，就可以打开Kubernetes控制台,或者使用kubectl 命令行工具来操作kubernetes集群了

>这里有一个坑，如果你开启了代理或者vpn，例如设置了环境变量$https_proxy=http://proxy:port，在启动时可能会出现`timed out waiting to elevate kube-system RBAC privileges: ` 的错误，请关闭代理后重新启动即可。

```shell
minikube dashboard
```

