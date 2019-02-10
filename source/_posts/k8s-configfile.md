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
