---
title: kubernetes 集群环境搭建
date: 2019-02-01 22:43:15
tags: devops microservice
categories: kubernetes
---

### 两种部署方式

**守护进程：** k8s的所有组件都以系统级守护进程的方式运行。该方式部署难度高，所有的组件都必须手动搭建。编辑配置文件等都需要手动进行，部署方式繁琐而复杂。
也可以使用ansible 脚本的方式来启动。但是一旦守护进程down掉，必须手动启动。

**kubeadm：** 组件都以pod 的方式运行。master节点上的API server，controller-manager，Scheduler以及node节点上的kube-proxy都以pod的方式运行。这些pod与传统pod不同，都是静态pod。

### 使用kubeadm部署

两个重要子程序
- `kubeadm init` 用于初始化集群
- `kubeadm join` 用于将node 加入集群

`/etc/kubernetes/manifests` kubernetes会在此路径查找静态Pod的清单文件

Kubeadm会自动生成CA，API servery，client以及其它需要使用到的证书

### kubeadm 启动步骤

启动步骤大致分为：
1. master, node 安装kubelet, kubeadm, docker
2. master: kubeadm init
3. nodes: kubeadm join

详细启动步骤：

master 节点安装好kubelet, kubeadm, docker等组件后，将kubelet设置为开机启动。

```shell
sudo systemctl enable kubelet
```

> 此时kubelet是无法启动的，因为相关配置文件信息还没有设置好，可通过查看系统日志了解具体错误信息。

使用 kubeadm 初始化：

```shell
sudo kubeadm init  --pod-network-cidr 10.244.0.0/16 
```

> 如果提示 [Error swap]，可以使用 --ignore-preflight-errors 选项忽略相关错误。编辑/etc/sysconfig/kubelet 文件，KUBELET_EXTRA_ARGS="--fail-swap-on=false" 然后加入--ignore-preflight-errors=Swap 选项

出于某些不可描述的原因，可能无法拉取镜像。这里需要暂时设置一下docker 的代理。编辑docker.service文件

```shell
sudo vim /lib/systemd/system/docker.service
```

在service节点下添加环境变量
```shell
Environment="HTTPS_PROXY=http://127.0.0.1:1081"
```

重启docker守护进程：

```shell
sudo systemctl daemon-reload 
sudo systemctl restart docker.service
```

配置node节点：

