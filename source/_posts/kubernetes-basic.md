---
title: kubernetes 基础概念
date: 2019-02-01 19:18:36
tags:
categories: kubernetes
---

## 术语

### master/node

Kubenates 主机可以分为master 和node 两种节点, master 节点作为管理者一般为三个,作高可用。node 节点即worker 节点，作为容器的运行者。

master 节点包含组件：
- API server 
- Scheduler 调度器
- Controller-manager 控制器

node 节点包含：
- kubelet 用于和master 节点的API server 交互
- docker 运行容器

### pod

K8s的最小调度单位是pod而不是容器，一个pod可以包含多个容器。同一个pod的容器之间共享一个network namespace。一般情况下，一个pod 应只包含一个容器，除非多个容器间有强关联关系。

**label** 是一种kv 格式的数据，用于识别不同的pod。label selector 选择即master 上的label 选择器。

### Pod 控制器

Pod 我们人为的将其分为两类，自主式pod和控制器管理的pod

K8s中有很多种控制器以适应不同的应用场景
- ReplicationController
- ReplicaSet
- Deployment 管理无状态的pod
- statefulSet 管理有状态的pod
- DaemonSet
- Job 和 CronJob 任务和定时任务

其中Deployment 控制器还支持2级控制器HPA，即自动水平伸缩控制器。

## kubenates 网络

同一个pod内的的多个容器通信：lo
不同pod的通信：使用overlay，两个pod之间直接通信，所以k8s集群中pod的ip地址不能重复。
pod与service通信

## 服务注册与发现

#### etcd
k8s使用etcd作为服务发现数据库。

#### kubeproxy
运行在node上的守护进程，当service 或service 背后的pod改变，即目标地址发生改变时，kubeproxy 会反应到API server。

## CA

## K8S 名称空间

用于隔离每个Pod。