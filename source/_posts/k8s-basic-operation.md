---
title: k8s-basic-operation
date: 2019-02-03 20:54:36
tags:
categories: kubernetes
---

## 什么是Pod

pod 是k8s的最小调度单位，一个pod中可以包含多个container，同一个pod中的container共享同一个网络名称空间（network namespace），即它们之间可以使用lo 通信。

## kubectl 客户端工具

kubectl 是一个由官方提供的连接到master节点上api server 的客户端工具。以实现k8s各种资源（Pod、Service、deployment等等）的增删改查基本操作。

### 常用命令

`kubectl cluster-info` 查看整个集群的信息

`kubectl get pods` 列出所有pods

`kubectl get pods -o wide` 列出所有pods,并包含网络和节点信息

`kubectl exec [Pod name]` 在pod中执行一个shell命令（同docker exec类似）

`kubectl exec -it [Pod name] sh` 在pod中以交互模式运行shell（同docker exec类似），当Pod中有多个容器时，默认进入第一个容器。可以使用 `-c` 选项指定容器。

`kubectl describe` 显示组件的详细信息，例如显示pod的详细信息

```shell
kubectl describe pods nginx
```

`kubectl run` 创建并运行一个资源

```shell
kubectl run nginx-deploy --image=nginx:1.14-alpine --replicas=2
```

`kubectl port-forward` 将一个端口转发到k8s集群内的一个pod上

```shell 
# 将本地8080端口转发到nginx pod的80端口，从而可以通过127.0.0.1:8080访问nginx
kubectl port-forward nginx 8080:80 
```

`kubectl expose` 暴露一个可访问的service，可通过service名称+端口的方式访问内部pod。

```shell
kubectl expose deployment nginx-deploy --name=nginx-server --port=8080 --target-port=80
```


> 不指定service 的type 时，默认为ClusterIP , 该service 供集群 **内部Pod 客户端** 访问，在集群外部无法访问. 若需要外部访问，需要指定 `--type=NodePort` 
> NodePort Service 即微服务架构中的网关层，可以部署多个，用HA-Proxy 或F5 做负载均衡，以实现高可用。



