---
title: kubernetes 基础概念
date: 2019-02-01 19:18:36
tags:
---

## kubenates 网络

同一个pod内的的多个容器通信：lo
不同pod的通信：使用overlay，两个pod之间直接通信，所以k8s集群中pod的ip地址不能重复。
pod与service通信

## 服务注册与发现

#### etcd
k8s使用etcd作为服务发现数据库。

#### kubeproxy
运行在node上的守护进程，当service 或service 背后的pod改变，即目标地址发生改变时，kubeproxy 会反应到API server。

## CA

## K8S 名称空间

用于隔离每个Pod。