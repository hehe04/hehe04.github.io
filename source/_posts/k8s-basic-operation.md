---
title: k8s-basic-operation
date: 2019-02-03 20:54:36
tags:
categories: kubernetes
---

## 什么是Pod

pod 是k8s的最小调度单位，一个pod中可以包含多个container，同一个pod中的container共享同一个网络名称空间（network namespace），即它们之间可以使用lo 通信。

## 常用命令

`kubectl get pods` 列出所有pods

`kubectl get pods -o wide` 列出所有pods,并包含网络和节点信息

`kubectl exec [Pod name]` 在pod中执行一个shell命令（同docker exec类似）

`kubectl exec -it [Pod name] sh` 在pod中以交互模式运行shell（同docker exec类似），当Pod中有多个容器时，默认进入第一个容器。可以使用 `-c` 选项指定容器。

`kubectl describe` 显示组件的详细信息，例如显示pod的详细信息

```shell
kubectl describe pods nginx
```
`kubectl create` 创建一个资源，例如通过配置文件创建pod

```shell
kubectl create -c pod_nginx.yml
```
配置文件示例：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

`kubectl delete -f pod_nginx.yml ` 通过配置文件删除资源

`kubectl port-forward` 将一个端口转发到k8s集群内的一个pod上

```shell 
# 将本地8080端口转发到nginx pod的80端口，从而可以通过127.0.0.1:8080访问nginx
kubectl port-forward nginx 8080:80 
```
