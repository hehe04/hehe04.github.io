---
title: k8s使用hostPath Volume 收集容器app 的日志
date: 2019-01-03 16:11:13
tags:
categories: kubernetes
---

app 日志的重要性大家都应该知道，当我们需要收集运行在容器内的app 所产生的日志时，很自然的会联想到side car 这种方式。但是，这种方式对原始的pod 依然有入侵，并没有完全解耦，并且，每一个pod 都需要部署一个日志收集组件，对于宝贵的服务器资源来说，也是一种浪费。

还有一种方式，就是将容器内的日志目录挂载在宿主机上，这样我们每个宿主机只需要安装一个日志收集组件，并监控宿主机上的日志文件就可以了。当然，我们可以使用Daemonset 的方式部署日志收集组件，保证集群内每个节点都会有一个组件，并且也不需要额外的监控。

这种方式也不是没有任何问题，这里有个小坑，当节点数量小于副本集数量时，也就是说在同一个节点上会有一个pod 的多个副本在运行，那么此时app 日志会写入到宿主机上同一个文件内。如果需要为不同的pod 建立不同的文件夹要怎么做呢？

这里有一个小技巧，跟大家分享，我们可以使用 **软链接 + HOSTNAME** 的方式。使用shell 脚本为每个日志文件目录建立一个以HOSTNAME 命名的软链接目录，宿主机只需要监控这个软链接目录就可以实现不同pod 日志写入不同的目录内。

```shell

mkdir -p /home/mount/${HOSTNAME} /home/work/logs && ln -s /home/mount/${HOSTNAME} /home/work/logs/applogs
```

完整示例：

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostpathtest
  labels:
    app: hostpathtest
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hostpathtest
  template:
    metadata:
      labels:
        app: hostpathtest
    spec:
      containers:
      - name: hostpathtest
        image: busybox
        command: ['sh','-c']
        args:
        -  "mkdir -p /home/mount/${HOSTNAME} /home/work/logs && ln -s /home/mount/${HOSTNAME} /home/work/logs/applogs && echo $(date)   ${HOSTNAME} >> /home/work/logs/applogs/nginx.log && sleep 10000000"
        volumeMounts:
        - mountPath: /home/mount
          name: test-volume
      volumes:
      - name: test-volume
        hostPath:
          path: /home/data
          type: DirectoryOrCreate
```

