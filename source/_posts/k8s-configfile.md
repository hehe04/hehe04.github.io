---
title: kubernetes 资源清单配置文件
date: 2019-02-10 22:29:45
tags:
categories: kubernetes
---

## K8S 的核心资源对象

按功能分类：
- workload: Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob...
- 服务发现： Service, Ingress...
- 配置与存储： Volume, CSI
  - ConfigMap, Secret
  - DownwardAPI
- 集群级资源： Namespce, Node, Role, ClusterRole, RoleBinding, ClusterRoleBinding
- 元数据型资源: HPA, PodTemplate, LimitRange

## 配置清单简介
  
创建k8s资源时，可以使用配置清单来完成。api server 仅接受Json 格式的资源定义。用户定义时，使用yaml格式的配置清单。api server 会将其自动转换为Json格式的资源定义。

大部分资源的yaml 配置清单都包括以下字段：
- apiVersion: 定义了所使用的api 版本，value 的完整形式应该是group/version，可以使用kubectl api-versions查看所有支持的版本。此处忽略了group，则为core group.
- kind: 定义了资源的类别。
- metadata: 定义了资源的元数据信息。
  - name 资源名称
  - namespace 
  - annotations 注解
  - labels
- spec: 最重要的信息之一， 定义了用户所期望的资源状态。
- status 显示资源的当前状态，应该与spec 一致。

使用 kubectl get 命令查看资源时，可以指定 `-o yaml` 参数输出yaml格式的配置清单。

```shell
kubectl get [type] [name] -o yaml
```

输出：
```shell
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2019-02-10T08:14:09Z"
  generateName: nginx-deploy-67b97557d5-
  labels:
    pod-template-hash: "2365311381"
    run: nginx-deploy
  name: nginx-deploy-67b97557d5-5b8kt
  namespace: default
  ownerReferences:
  - apiVersion: extensions/v1beta1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-deploy-67b97557d5
    uid: d7aff9b7-2d0b-11e9-8d5d-0800270faddc
  resourceVersion: "19270"
  selfLink: /api/v1/namespaces/default/pods/nginx-deploy-67b97557d5-5b8kt
  uid: d7b4f37a-2d0b-11e9-8d5d-0800270faddc
spec:
  containers:
  - image: nginx:1.14-alpine
    imagePullPolicy: IfNotPresent
    name: nginx-deploy
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-zm9l2
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: minikube
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-zm9l2
    secret:
      defaultMode: 420
      secretName: default-token-zm9l2
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2019-02-10T08:14:09Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2019-02-10T08:14:27Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2019-02-10T08:14:09Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://508d71cd9b92c4630c2aa8677cd53264dabde784e2e808a0993f7dd17f58c1b2
    image: nginx:1.14-alpine
    imageID: docker-pullable://nginx@sha256:b96aeeb1687703c49096f4969358d44f8520b671da94848309a3ba5be5b4c632
    lastState: {}
    name: nginx-deploy
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2019-02-10T08:14:26Z"
  hostIP: 10.0.2.15
  phase: Running
  podIP: 172.17.0.5
  qosClass: BestEffort
  startTime: "2019-02-10T08:14:09Z"
```

kubectl explain 命令可查看相关节点的文档

```shell
kubectl explain pod
kubectl explain pod.spec
```
## kubectl 针对配置清单的基本操作

`kubectl create -f` 通过配置文件创建资源

```shell
kubectl create -f pod_nginx.yml
```

用户定义的配置文件示例：

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

## 配置清单中的一些重要字段

### spec

资源定义，最重要的信息之一， 定义了用户所期望的资源状态。

#### spec.restartPolicy 

Pod 的重启策略：可能的值有Always、Never、Onfailure

#### spec.containers.imagePullPolicy 

拉取镜像的策略，设置后，Pod 一旦创建该字段就不允许修改了
- Always
- Never
- IfNotPresent

>  如果本地使用的镜像版本为latest，该选项应该设置为Always 。因为本地所使用的latest 版本在服务器上并不一定就是latest 。

#### spec.containers.ports

暴露端口，与docker 不同的是，该参数仅仅起到一个描述信息的作用，并不能真正的起到暴露端口的作用。可以用name字段为指定的端口起一个名称（可选）。在service 中映射时，可以尝试使用该名称。

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - name: http
      containerPort: 80
    - name： https
      containerPort: 443
```

#### spec.containers.commands

用于指定pod 运行时所执行的命令，指定该字段，docker镜像Entrypoint 和Cmd 将被忽略。

#### spec.conttainers.args

运行命令所需要的参数，如果指定该字段，会覆盖docker镜像的cmd 字段，但不会覆盖Entrypoint 。变量引用格式`$(VAR_NAME)`. 使用$$ 来转义（escape）。

| Description                         | Docker field name | Kubernetes field name |
| :---------------------------------- | :---------------- | :-------------------- |
| The command run by the container    | Entrypoint        | command               |
| The arguments passed to the command | Cmd               | args                  |

### metadata

元数据信息

#### labels

每个资源都可以拥有多个标签，反之，一个标签也可以贴在多个资源上。每个标签都可以被资源选择器匹配，从而完成资源选择。

key：字母，数字，_,-,.
value: 可以为空，只能以字母或数字开头结尾，中间可以使用下划线等。

标签可以在创建时通过配置文件指定，也可以在资源创建后，通过管理工具来管理（添加、删除...）

```shell
# 为pod 打标签
kubectl label pod [pod_name] key=value
```

k8s 的标签选择器：
- 等值关系：=, ==, !=
- 集合关系：key in (value1,value2...)，key notin (value1,value2...)， key 和 !key

#### 节点标签

label 不仅可以用于资源上，也可以打在节点上。当节点拥有标签，在创建资源时，就可以指定资源应该运行在哪个节点上。