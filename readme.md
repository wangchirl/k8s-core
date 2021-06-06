### Kubernetes 基础

### 一、学习目标

1. 理解 Pod，掌握 Pod 的生命周期
2. 各种控制器类型的特点及使用定义方式
3. 掌握 svc 原理及其构建方式
4. 掌握多种存储类型的特点及其使用场景
5. 掌握调度器原理，根据要求把 Pod 定义到想要的节点运行
6. kubernetes 高可用集群搭建，kubeadm方式，二进制方式，**rancher方式**
7. 集群安全，认证-鉴权-访问控制原理及其流程
8. 掌握 Helm 原理及其使用

### 一、Kubernetes 基本概念

#### 1、Kubernetes 简介

##### 1. kubernetes 前世今生及特征

- 参考谷歌的 borg 设计而来，Go 语言开发
- 轻量级，消耗资源小
- 开源
- 弹性伸缩
- 负载均衡 IPVS
- 滚动更新
- 版本回退

##### 2. kubernetes 概述

- k8s 是谷歌在2014年开源的容器化集群管理系统
- 使用 k8s 进行容器化应用部署
- 使用 k8s 利于应用扩展
- k8s 目标实施让部署容器化应用更加简洁和高效



#### 2、K8S 集群架构组件

![](./Kubernetes.png)

![](./k8s-master-node.jpg)



##### 1、Master 组件（主控节点）

- **==apiserver==**

  集群统一入口，以 restful 方式，交给 Etcd 存储，所有服务的统一入口

- **==scheduler==**

  节点调度，选择合适的 node 节点应用部署

- **==controller-manager==**

  处理集群中常规后台任务，一个资源对应一个控制器，维持副本期望数目

- **==etcd==**

  兼具一致性和高可用性的键值数据库，用于保存 Kubernetes  集群相关的数据



##### 2、Node 组件（工作节点）

- **==kubelet==**

  master 派到 node 节点代表，管理本机容器，保证容器都运行在 Pod 中，直接与容器引擎交互实现容器的生命周期管理

- **==kube-proxy==**

  提供网络代理，负载均衡等操作，负责写入规则至 iptables、ipvs 实现服务映射访问

- **==Container Runtime==**（容器运行环境）

  容器运行环境是负责运行容器的软件，支持的容器运行环境有： [Docker](http://www.docker.com/)、 [containerd](https://containerd.io/)、[cri-o](https://cri-o.io/)、 [rktlet](https://github.com/kubernetes-incubator/rktlet) 以及任何实现 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md)



##### 3、其他组件

- coreNDS

  可以为集群中的 svc 创建一个域名 ip 的对应关系解析

- dashboard

  给 k8s 集群提供一个 B/S 架构访问体系

- **ingress controller**

  官方只能实现四层代理，ingress 可以实现七层代理

- federation

  提供一个可以跨集群中心多 k8s 统一管理功能

- **Prometheus**

  提供 k8s 集群的监控能力

- **ELK**

  提供 k8s 集群日志统一分析平台



### 二、kubernetes 核心概念

#### 0、资源清单

##### 1. 资源清单格式

```yaml
apiVersion: group/apiversion # 版本信息
kind:   # 资源类型
metadata:  # 资源元数据
  name:  # 资源名称
  namespace: # 名称空间，默认default
  labels:  # 资源标签 - 用于被筛选
  annotations:
spec:   # kubernetes 对象描述
status:  # 当前状态，本字段由 kubernetes 自身维护
```

```shell
# 获取 apiversion 版本信息
kubectl api-versions
# 获取资源的 apiversion 版本信息
kubectl explain pod Ingress
# 获取帮助文档
kubectl --help
kubectl get --help
```

##### 2. 资源清单常用字段解释

| 参数名                                          | 字段类型 | 说明                                                         |
| ----------------------------------------------- | -------- | ------------------------------------------------------------ |
| ==apiVersion==                                  | String   | 这里是指 k8s api 的版本，目前基本是 v1，使用 kubectl api-version 查看 |
| ==kind==                                        | String   | 资源类型和角色，Pod，Deployment等                            |
| metadata                                        | Object   | 元数据对象，固定值                                           |
| ==metadata.name==                               | String   | 元数据对象名称，用户自定义                                   |
| ==metadata.namespace==                          | String   | 元数据对象命名空间，默认default，可以自定义                  |
| Spec                                            | Object   | 详细定义对象，固定值 spec                                    |
| spec.containers[]                               | List     | spec对象的容器列表定义                                       |
| ==spec.containers[].name==                      | String   | 容器名称                                                     |
| ==spec.containers[].image==                     | String   | 容器镜像名称                                                 |
| ==spec.containers[].imagePullPolicy==           | String   | 镜像拉取策略，Always、Never、IfNotPresent                    |
| ==spec.containers[].command[]==                 | List     | 容器启动命令                                                 |
| ==spec.containers[].args[]==                    | List     | 容器启动参数                                                 |
| spec.containers[].workingDir                    | String   | 指定容器的工作目录                                           |
| ==spec.containers[].volumeMounts[]==            | List     | 指定容器内部的存储卷配置                                     |
| ==spec.containers[].volumeMounts[].name==       | String   | 指定可以被容器挂载的存储卷的名称                             |
| ==spec.containers[].volumeMounts[].mountPath==  | String   | 指定可以被容器挂载的存储卷的路径                             |
| ==spec.containers[].volumeMounts[].readOnly==   | String   | 设置存储卷路径的读写模式，true或false，默认false读写模式     |
| ==spec.containers[].ports[]==                   | List     | 指定容器需要用到的端口列表                                   |
| ==spec.containers[].ports[].name==              | String   | 指定端口名称                                                 |
| ==spec.containers[].ports[].containerPort==     | String   | 指定容器需要监听的端口号                                     |
| ==spec.containers[].ports[].hostPort==          | String   | 指定容器所在宿主机监听的端口号，默认跟 containerPort相同，注意多个指定了 hostPort 的端口号不能重复 |
| ==spec.containers[].ports[].protocol==          | String   | 指定端口协议，支持 TCP 和UDP，默认 TCP                       |
| ==spec.containers[].env[]==                     | List     | 指定容器运行前需设置的环境变量列表                           |
| ==spec.containers[].env[].name==                | String   | 指定环境变量名称                                             |
| ==spec.containers[].env[].value==               | String   | 指定环境变量值                                               |
| spec.containers[].resources                     | Object   | 指定资源限制和资源请求的值                                   |
| spec.containers[].resources.limits              | Object   | 指定容器运行时资源的运行上限                                 |
| ==spec.containers[].resources.limits.cpu==      | String   | 指定CPU的最大限制，单位为 core 数                            |
| ==spec.containers[].resources.limits.memory==   | String   | 指定内存的最大限制，单位为 MIB、GiB                          |
| spec.containers[].resources.requests            | Object   | 指定容器启动和调度时的资源限制                               |
| ==spec.containers[].resources.requests.cpu==    | String   | 容器启动和调度时CPU限制                                      |
| ==spec.containers[].resources.requests.memory== | String   | 容器启动和调度内存限制                                       |
| ==spec.restartPolicy==                          | String   | Pod 重启策略，Always、OnFailure、Never                       |
| ==spec.nodeSelector==                           | Object   | 定义Node的Label 过滤标签，key-value格式                      |
| spec.imagePullSecrets                           | Object   | 定义拉取镜像时使用secret名称，以 name:secretkey格式指定      |
| spec.hostNetwork                                | Boolean  | 定义是否使用主机网络模式，默认false，设置true表示使用宿主机网络，不使用docker网桥，同时无法在同一台宿主机上启动第二个副本 |



#### 1、[Pod](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/)

##### 1. Pod 基本概念

- 最小部署的单元
- 包含多个容器（一组容器的集合，一个或多个容器）
- 一个 pod 中的容器共享网络命名空间
- pod 是短暂的

##### 2. Pod 存在意义

- 创建容器使用 docker，一个 docker 对应一个容器，一个容器有进程，一个容器运行一个应用程序

- Pod 是多进程设计，运行多个应用程序

  ==一个 Pod 有多个容器，一个容器里面运行一个应用程序==

- Pod 存在为了亲密性应用

  - 两个应用之间进行交换
  - 网络之间调用
  - 两个应用需要频繁调用

##### 3. Pod 实现机制

- 共享网络

  ==共享网络：通过 Pause 容器，把其他业务容器加入到 Pause 容器里面，让所有业务容器在同一个名称空间中，可以实现网络共享==

- 共享存储

  ==共享存储：引入数据券概念 Volumn，使用数据券仅限持久化存储==

##### 4. Pod 镜像拉取策略

| 拉取策略         | 说明                                               |
| ---------------- | -------------------------------------------------- |
| ==IfNotPresent== | 默认值，镜像在宿主机不存在才拉取                   |
| ==Always==       | 每次创建 Pod 都会重新拉取一次镜像                  |
| ==Never==        | Pod 永远不会主动拉取镜像，需要开发人员手动拉取镜像 |

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx:1.14
    imagePullPolicy: Always
```



##### 5. Pod 资源限制（resources）

| 资源限制     | 说明         |
| ------------ | ------------ |
| ==requests== | 调度资源限制 |
| ==limits==   | 最大资源限制 |

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fronted
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```



##### 6. Pod 重启策略（restartPolicy）

| 重启策略      | 说明                                          |
| ------------- | --------------------------------------------- |
| ==Always==    | 当容器终止退出后，总数重启容器，默认策略      |
| ==OnFailure== | 当容器异常退出（退出状态码非0）时，才重启容器 |
| ==Never==     | 当容器终止退出，从不重启容器                  |

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-test
spec:
  containers:
  - name: busybox
    image: busybox:1.28.4
    args:
    - /bin/sh
    - -c
    - sleep 36000
  restartPolicy: Never  
```



##### 7. Pod 健康检查

| 探针               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| ==livenessProbe==  | 如果检查失败，将杀死容器，根据 Pod 的 重启策略 restartPolicy 来操作，如果容器不提供存活探针， 则默认状态为 `Success` |
| ==readinessProbe== | 如果检查失败，kubernetes 会把 Pod 从 service endpoints 中剔除，如果容器不提供就绪态探针，则默认状态为 `Success` |
| ==startupProbe==   | 指示容器中的应用是否已经启动。==如果提供了启动探针，则所有其他探针都会被禁用==，直到此探针成功为止，如果启动探测失败，`kubelet` 将杀死容器，而容器依其 [重启策略](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)进行重启。 如果容器没有提供启动探测，则默认状态为 `Success` |

- Probe  探针支持的三种检查方法：

  | 方法          | 说明                                          |
  | ------------- | --------------------------------------------- |
  | ==httpGet==   | 发送 HTTP 请求，返回 200-400 范围状态码为成功 |
  | ==exec==      | 执行 shell 命令返回状态码为0为成功            |
  | ==tcpSocket== | 发起 TCP Socket 建立成功                      |

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    # Pod 名称
    name: readiness-check
    # 命名空间
    namespace: default
  spec:
    containers:
    # 容器名称
    - name: readiness-container
      # 容器镜像
      image: nginx
      # 镜像拉取策略
      imagePullPolicy: IfNotPresent
      #command: ["/bin.sh","-c","touch /tmp/live; sleep 60; rm -rf /tmp/live; sleep 3600"]
      # 就绪探测
      readinessProbe:
        # 1. httpGet 方式
        httpGet:
          port: 80
          path: /index.html
        initialDelaySeconds: 1
        periodSeconds: 3
        # 2. exec 方式
        #exec:
          #command: ["test","-e","/tmp/live"]
        # 3. tcpSocket 方式  
        #tcpSocket:
          #port: 80
  ```

- `lifecycle` 启动、退出动作

  - ==postStart==

  - ==preStop==

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: lifecycle
    spec:
      containers:
      - name: lifecycle-container
        image: nginx
        # lifecycle
        lifecycle:
          # postStart
          postStart:
            exec:
              command: ["/bin/sh","-c","echo hello start > /usr/local/message"]
          # preStop
          preStop:
            exec:
              command: ["/bin/sh","-c","echo hello stop > /usr/local/message"]
    ```

    查看
    
    ```sh
    kubectl exec -it lifecycle -c nginx-container cat /usr/local/message
    ```
    
    

##### 8. [Init 容器](https://kubernetes.io/zh/docs/concepts/workloads/pods/init-containers/)

- 是一种特殊容器

- 在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 内的应用容器启动之前运行

- Init 容器与普通的容器非常像

  - 它们总是运行到完成

  - 每个都必须在下一个启动之前成功完成

  - Init 容器失败，kubelet 会不断地重启直到成功，除非 Pod 的重启策略restartPolicy 为 Never

  - Init 容器不支持 `lifecycle`、`livenessProbe`、`readinessProbe` 和 `startupProbe`

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      # 应用容器
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      # Init 容器  
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
      - name: init-mydb
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
        
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: myservice
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: mydb
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9377
    ```

##### 9. Pod 分类

| 类型                 | 说明                                            |
| -------------------- | ----------------------------------------------- |
| ==自主式 Pod==       | Pod 退出了，此类型的 Pod 不会被创建             |
| ==控制器管理的 Pod== | 在控制器的生命周期里，始终要维持 Pod 的副本数目 |

##### 10. Pod 的状态

| 状态值        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| ==Pending==   | Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间 |
| ==Running==   | Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态 |
| ==Succeeded== | Pod 中的所有容器都已成功终止，并且不会再重启                 |
| ==Failed==    | Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止 |
| ==Unknown==   | 因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败 |

容器状态

- Waiting 等待
- Running 运行中
- Terminated 已终止

##### 11. 容器生命周期

![1621332868219](.\1621332868219.png)

##### 12. 创建 Pod 的流程

![1621406317478](.\1621406317478.png)

##### 13. 影响 Pod 调度配置（Scheduler）

###### 1.  [Pod 资源限制影响](https://kubernetes.io/zh/docs/concepts/policy/resource-quotas/)

> 根据 requests 找到足够 node 节点进行调度

```yaml
....
 resources:
   # requests 资源限制
      requests:
        memery: "64Mi"
        cpu: "250m"
```

###### 2. [节点选择器标签影响](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/assign-pod-node/)

- 首先对节点创建标签

  ```sh
  # 节点创建标签 kubectl label nodes <node-name> <label-key>=<label-value>
  kubectl label nodes node01 env_role=dev
  # 查看节点标签
  kubectl get nodes --show-labels
  ```

- 添加 nodeSelector 字段到 Pod 配置中

  ```yaml
  ...
  spec:
    # 节点选择器标签
    nodeSelector:
      env_role: dev
    containers:
    - name: nginx
      image: nginx:1.15
  ```

###### 3.  [节点亲和性影响](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/assign-pod-node/)

>  ==节点亲和性 nodeAffinity 和前面的 nodeSelector 基本一样==，根据节点上标签约束来决定 Pod 调度到哪些节点上
>
> - 硬亲和性：约束条件必须满足 ==requiredDuringSchedulingIgnoredDuringExecution==
> - 软亲和性：尝试满足，不保证 ==preferredDuringSchedulingIgnoredDuringExecution==
>
> 支持常用操作符：
>
> ==In、NotIn、Exists、Gt、Lt、DoesNotExists==

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: nginx:1.14
```

###### 4. [污点和污点容忍影响](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/taint-and-toleration/)

> 基本介绍：
>
> - ==nodeSelector 和 nodeAffinity== ：Pod 调度到某些节点上，==是Pod 属性==，调度时候实现
>
> - ==Taint 污点==：节点不做普通分配调度，==是节点属性==
>
> 场景：
>
> - 专用节点
> - 配置特点硬件节点
> - 基于 Taint 驱逐
>
> 操作演示：
>
> - 查看节点污点情况
>
>   ```sh
>   kubectl describe node k8smaster | grep Taint
>   ```
>
>   > 污点值有三个：
>   >
>   > 1. NoSchedule：一定不被调度
>   > 2. PreferNoSchedule：尽量不被调度
>   > 3. NoExecute：不会调度，并且还会驱逐Node已有的 Pod
>
> - 为节点添加污点
>
>   ```sh
>   # kubectl taint node [node] key=value:污点值
>   kubectl taint node k8smaster env_role=dev:NoSchedule
>   ```
>
> - 删除污点
>
>   ```sh
>   kubectl taint node k8smaster env_role=dev:NoSchedule-
>   ```
>
> - 污点容忍
>
>   ```yaml
>   apiVersion: v1
>   kind: Pod
>   metadata:
>     name: nginx
>     labels:
>       env: test
>   spec:
>     containers:
>  - name: nginx
>       image: nginx
>       imagePullPolicy: IfNotPresent
>     # 污点容忍  
>     tolerations:
>  - key: "example-key"
>       operator: "Exists"
>       effect: "NoSchedule"
>   ```





#### 2、Controller

##### 1. Controller 简介

###### 1. 是什么？

 集群上管理和运行容器的对象

###### 2. Pod 和 Controller 关系

- Pod 是通过 Controller 实现应用的运维，比如伸缩，滚动升级等
- Pod 和 Controller 之间通过 label 标签建立关系 => selector

###### 3. Deployment 控制器应用场景

- 部署无状态应用
- 管理 Pod 和 ReplicaSet
- 部署、滚动升级等功能 => web服务、微服务

###### 4. 基本操作

```sh
# 导出 yaml 文件
kubectl create deployment web --image=nginx --dry-run -o yaml > web.xml
# 使用 yaml 部署应用
kubectl apply -f web.yaml
# 对外发布（暴露对外端口号）
kubectl expose deployment web --port=80 --type=NodePort --target-port=80 --name=web1 -o yaml > web1.xml
kubectl apply -f web1.yaml
# 查看pod 和 svc 状态
kubectl get pods,svc
# 应用升级
kubectl set image deployment web nginx=nginx:1.15
# 查看升级状态
kubectl rollout status deployment web
# 查看升级版本
kubectl rollout history deployment web
# 回滚到上一个版本
kubectl rollout undo deployment web
# 回滚到指定版本
kubectl rollout undo deployment web --to-revision=2
# 弹性伸缩
kubectl scale deployment web --replicas=10
```



##### 2. 控制器类型

| 类型                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| **ReplicationController**      | RC在新版本已被ReplicaSet取代，用来确保用户定义的副本数，即容器异常退出会自动创建新的Pod来替代 |
| ==ReplicaSet==                 | 和 RC 没有本质区别， ==RS 支持集合式的 selector==            |
| ==Deployment==                 | 为 Pod 和 RS 提供了一个声明式定义方法，定义Deployment 来创建 Pod 和 RS，滚动升级和回滚应用，扩缩容，暂停/继续 Deployment |
| ==DaemonSet==                  | 确保全部 Node 上运行一个 Pod 副本，有 Node 加入集群时为该 Node新增一个 Pod，移除 Node 时回收 Pod，日志收集，监控等 |
| ==StatefulSet==                | 为 Pod 提供唯一的标识，保证部署和 scale 的顺序，==解决有状态服务的问题== |
| ==Job/CronJob==                | Job 负责批处理任务，只执行一次，CronJob 管理基于时间的 Job，在给定时间点只运行一次，周期性的运行 |
| ==Horizontal Pod Autoscaling== | HPA 使得 Pod 水平自动缩放，削峰填谷，提高系统资源利用率      |

##### 3. RS 与 Deployment 的关系

![1621384029373](./1621384029373.png)

##### 4. 控制器 yaml 示例

###### ① [ReplicationController](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicationcontroller/)

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  # 副本数
  replicas: 3
  # 标签选择符，不配置默认和 .template.metadata.labels保持一致
  # 若配置则和 .template.metadata.labels 进行匹配，如果不匹配则API拒绝
  # 与 RS 的区别是 selector 这里只支持单值，RS 支持集合
  selector:
    app: nginx
  # Pod 模板  
  template:
    metadata:
      name: nginx
      # Pod 标签
      labels:
        app: nginx
    spec:
      # 容器
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

[基本操作](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicationcontroller/)

```shell
# 运行示例任务，可以本地创建 yaml 文件
kubectl apply -f https://k8s.io/examples/controllers/replication.yaml
# 检查 ReplicationController 的状态
kubectl describe replicationcontrollers/nginx
# 删除 RC 及其 Pod
kubectl delete rc nginx
# 只删除 RC
kubectl delete rc nginx --cascade=false
# 修改 Pod 标签可以将 Pod 从 RC 中隔离
```



###### ② [ReplicaSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  # RS 标签
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  # selector.matchLabels - 与 ReplicationController 的 .spec.selector 的作用相同
  # matchExpressions 允许构建更加复杂的选择器
  # 可以通过指定 key、value 列表以及将 key 和 value 列表关联起来的 operator
  # RS 标签选择符与 .template.metadata.labels 进行匹配
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      # Pod 标签
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.15
```

[基本操作](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/)

```sh
# 创建 rs
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
# 查看 rs
kubectl get rs
# 查看 rs 状态
kubectl describe rs/frontend
# 查看启动的 pod
kubectl get pods
# 以 yaml 格式查看 Pod 的
kubectl get pods frontend-b2zdv -o yaml
```



###### ③ [<font color='red'>Deployment</font>](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  # deployment 标签
  labels:
    app: nginx
spec:
  # Pod 副本数,默认 1
  replicas: 3
  # deployment 标签选择符，选择与之匹配的 Pod
  selector:
    matchLabels:
      app: nginx
  # Pod 模板    
  template:
    metadata:
      # Pod 标签
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

[基本操作](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)

```sh
# 创建 Deployment
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml --record
# 查看 Deployment
kubectl get deployments
# 查看 Deployment 上线状态
kubectl rollout status deployment/nginx-deployment
# 查看 Deployment 创建的 ReplicaSet
kubectl get rs
# 查看每个 Pod 自动生成的标签
kubectl get pods --show-labels
# 更新 Deployment
kubectl --record deployment.apps/nginx-deployment set image \
   deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
# 或者
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
# 或者 edit Deployment 的 .spec.template.spec.containers[0].image 从 nginx:1.14.2 更改至 nginx:1.16.1
kubectl edit deployment.v1.apps/nginx-deployment
# 获取 Deployment 的更多信息
kubectl describe deployments
# 检查 Deployment 修订历史
kubectl rollout history deployment.v1.apps/nginx-deployment
# 查看修订历史的详细信息
kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
# 撤消当前上线并回滚到前一个的修订版本
kubectl rollout undo deployment.v1.apps/nginx-deployment
# 回滚到特定修订版本
kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
# 缩放 Deployment
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
# Pod 的水平自动缩放 HPA
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
# 暂停、恢复 Deployment
kubectl rollout pause deployment.v1.apps/nginx-deployment
kubectl rollout resume deployment.v1.apps/nginx-deployment
# 观察上线状态
kubectl get rs -w
# 以 yaml 格式查看 Deployment
kubectl get deployment nginx-deployment -o yaml
# kubectl rollout 命令的退出状态为 1（表明发生了错误）
kubectl rollout
echo $?
```



###### ④ [DaemonSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  # 标签选择符
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  # pod 模板    
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      # 容忍度
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        # 资源限制
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        # 挂载卷    
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      # 挂载卷定义
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

[基本操作](https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/)

```sh
# 创建 daemonSet
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
# 查看
kubectl get daemonset -n kube-system
kubectl describe ds fluentd-elasticsearch -n kube-system
```



###### ⑤ [<font color=red>StatefulSet</font>](https://kubernetes.io/zh/docs/concepts/workloads/controllers/statefulset/)

> 用来管理有状态应用的工作负载 API 对象
>
> 为 Pod 提供持久存储和持久标识符
>
> StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID
>
> 无论怎么调度，每个 Pod 都有一个永久不变的 ID
>
> StatefulSet 当前需要 [无头服务](https://kubernetes.io/zh/docs/concepts/services-networking/service/#headless-services) 来负责 Pod 的网络标识
>
> ==deployment 和 statefulSet 区别：statefulSet 的每一个 Pod 有唯一标识==
>
> 唯一标识格式：主机名称.service名称.名称空间.svc.cluster.local

参考 [mysql部署](https://kubernetes.io/zh/docs/tasks/run-application/run-single-instance-stateful-application/)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 1 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx:1.15
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html      
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: www
spec:
  storageClassName: snow
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

[基本操作](https://kubernetes.io/zh/docs/tutorials/stateful-application/basic-stateful-set/)

```sh
# 创建
kubectl apply -f web.yaml
# 查看 service
kubectl get service nginx
# 查看 statefulSet
kubectl get statefulset web
# 查看pod
kubectl get pods -w -l app=nginx
# 删除 StatefulSet 中所有的 Pod
kubectl delete pod -l app=nginx
# 查看持久卷
kubectl get pvc -l app=nginx
# 扩容
kubectl scale sts web --replicas=5
# 缩容
kubectl patch sts web -p '{"spec":{"replicas":3}}'
```



###### ⑥ [Job](https://kubernetes.io/zh/docs/concepts/workloads/controllers/job/)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

[基本操作](https://kubernetes.io/zh/docs/concepts/workloads/controllers/job/)

```sh
# 运行 job
kubectl apply -f https://kubernetes.io/examples/controllers/job.yaml
# 检查 Job 的状态
kubectl describe jobs/pi
# 查看其中一个 Pod 的标准输出
kubectl logs $pods
```



###### ⑦ [CronJob](https://kubernetes.io/zh/docs/concepts/workloads/controllers/cron-jobs/)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  # cron 表达式
  schedule: "*/1 * * * *"
  # job 模板
  jobTemplate:
    spec:
      # pod 模板
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

[基本操作](https://kubernetes.io/zh/docs/tasks/job/automated-tasks-with-cron-jobs/)

```sh
# 运行 CronJob
kubectl create -f https://k8s.io/examples/application/job/cronjob.yaml
# 获取 CronJob 状态
kubectl get cronjob hello
# 监视这个任务
kubectl get jobs --watch
# 查看 Pod 日志
kubectl logs $pods
# 删除 CronJob
kubectl delete cronjob hello
```



#### 3、Service

##### 1、Service 简介

###### 1. Service 存在的意义

- 防止 Pod 失联（服务发现）
- 定义一组 Pod 访问策略（负载均衡），一般根据 label 和 selector 标签与 Pod 建立关联
- 只提供 4 层负载均衡能力，Ingress 提供 7 层

###### 2. Pod 和 Service 关系

 Service 定义了一种抽象，是一个 Pod 的逻辑分组，一种访问 Pod 的策略，这一组 Pod 能够被 Service 访问到，通过 label selector 与 Pod 建立关联，Service 对外暴露服务，提供 Pod 服务代理

![1621415050281](.\1621415050281.png)

###### 3. Service 常用类型

- ==ClusterIp==  ：默认类型，自动分配一个仅 Cluster 内部可以访问的虚拟 IP
- ==NodePort== ：在 ClusterIP 基础上为 Service 在每台机器上绑定一个端口，这样就可以通过 NodePort 来访问该服务
- LoadBalancer ：在 NodePort 基础上，借助 cloud provider 创建一个外部负载均衡器，并将请求转发到 NodePort
- ExternalName ：把集群外部的服务引入到集群内部来，在集群内部直接使用，kubernetes1.7 及以上版本支持

###### 4. 代理模式分类

- userspace  造成 api-server 访问压力过大
- iptables
- ipvs ：高版本默认，比 iptables 支持更多的负载均衡算法



##### 2、[Service yaml 示例](https://kubernetes.io/zh/docs/concepts/services-networking/connect-applications-service/)

```yaml
# Service 定义
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  # Service 类型，默认ClusterIp
  type: NodePort
  # 端口
  ports:
    # 对外端口 8080
  - port: 8080
    # 转发端口 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  # 标签选择器，匹配 Pod 标签  
  selector:
    run: my-nginx
---
# 定义 控制器
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 1
  # Pod 模板
  template:
    metadata:
      # Pod 标签
      labels:
        run: my-nginx
    spec:
      # 挂载卷定义
      volumes:
      - name: secret-volume
        secret:
          secretName: nginxsecret
      - name: configmap-volume
        configMap:
          name: nginxconfigmap
      containers:
      - name: nginxhttps
        image: bprashanth/nginxhttps:1.0
        ports:
        - containerPort: 443
        - containerPort: 80
        # 挂载卷使用
        volumeMounts:
        - mountPath: /etc/nginx/ssl
          name: secret-volume
        - mountPath: /etc/nginx/conf.d
          name: configmap-volume
```

```sh
# 创建 Service，Deployment，RS，Pod
kubectl create -f ./nginx-secure-app.yaml
# 查看 svc
kubectl get svc my-nginx
# 查看 svc 详情
kubectl describe svc my-nginx
```



#### 4、[Ingress](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)

##### 1. 是什么？

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#ingress-v1beta1-networking-k8s-io) 公开了从集群外部到集群内[服务](https://kubernetes.io/zh/docs/concepts/services-networking/service/)的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制，Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP，Ingress 可以提供负载均衡、SSL 和基于名称的虚拟托管

##### 2. Service 实现及问题

1. 把端口号对外暴露，通过 ip + 端口号进行访问：使用 Service 的 NodePort 类型实现

2. NodePort 缺陷

   在==每个节点上都会开启端口==，在访问时候通过任何节点，通过节点 ip + 暴露端口号实现访问

   意味着==每个端口号只能使用一次，一个端口号对应一个应用==

   实际访问都是使用域名，根据不同域名跳转到不同端口服务中

##### 3. Ingress 和 Pod 的关系

​	Pod 和 Ingress 通过 Service 关联

​	Ingress 作为统一入口，由 Service 关联一组 Pod

##### 4. Ingress 工作流程

![ingress](ingress.png)

##### 5. Ingress 的使用

1. 创建 pod

   ```sh
   # 创建 nginx 应用，对外暴露端口使用 NodePort
   kubectl create deployment web --image=nginx
   # 查看
   kubectl get pods
   # 对外暴露端口
   kubectl expose deployment web --port=80 --target-port=80 --type=NodePort
   # 查看
   kubectl get svc
   ```

2. 部署 Ingress Controller

   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   
   ---
   
   kind: ConfigMap
   apiVersion: v1
   metadata:
     name: nginx-configuration
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   
   ---
   kind: ConfigMap
   apiVersion: v1
   metadata:
     name: tcp-services
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   
   ---
   kind: ConfigMap
   apiVersion: v1
   metadata:
     name: udp-services
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: nginx-ingress-serviceaccount
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   
   ---
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: ClusterRole
   metadata:
     name: nginx-ingress-clusterrole
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   rules:
     - apiGroups:
         - ""
       resources:
         - configmaps
         - endpoints
         - nodes
         - pods
         - secrets
       verbs:
         - list
         - watch
     - apiGroups:
         - ""
       resources:
         - nodes
       verbs:
         - get
     - apiGroups:
         - ""
       resources:
         - services
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - ""
       resources:
         - events
       verbs:
         - create
         - patch
     - apiGroups:
         - "extensions"
         - "networking.k8s.io"
       resources:
         - ingresses
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - "extensions"
         - "networking.k8s.io"
       resources:
         - ingresses/status
       verbs:
         - update
   
   ---
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: Role
   metadata:
     name: nginx-ingress-role
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   rules:
     - apiGroups:
         - ""
       resources:
         - configmaps
         - pods
         - secrets
         - namespaces
       verbs:
         - get
     - apiGroups:
         - ""
       resources:
         - configmaps
       resourceNames:
         # Defaults to "<election-id>-<ingress-class>"
         # Here: "<ingress-controller-leader>-<nginx>"
         # This has to be adapted if you change either parameter
         # when launching the nginx-ingress-controller.
         - "ingress-controller-leader-nginx"
       verbs:
         - get
         - update
     - apiGroups:
         - ""
       resources:
         - configmaps
       verbs:
         - create
     - apiGroups:
         - ""
       resources:
         - endpoints
       verbs:
         - get
   
   ---
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: RoleBinding
   metadata:
     name: nginx-ingress-role-nisa-binding
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: nginx-ingress-role
   subjects:
     - kind: ServiceAccount
       name: nginx-ingress-serviceaccount
       namespace: ingress-nginx
   
   ---
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: ClusterRoleBinding
   metadata:
     name: nginx-ingress-clusterrole-nisa-binding
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: nginx-ingress-clusterrole
   subjects:
     - kind: ServiceAccount
       name: nginx-ingress-serviceaccount
       namespace: ingress-nginx
   
   ---
   
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-ingress-controller
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   spec:
     replicas: 1
     selector:
       matchLabels:
         app.kubernetes.io/name: ingress-nginx
         app.kubernetes.io/part-of: ingress-nginx
     template:
       metadata:
         labels:
           app.kubernetes.io/name: ingress-nginx
           app.kubernetes.io/part-of: ingress-nginx
         annotations:
           prometheus.io/port: "10254"
           prometheus.io/scrape: "true"
       spec:
         hostNetwork: true
         # wait up to five minutes for the drain of connections
         terminationGracePeriodSeconds: 300
         serviceAccountName: nginx-ingress-serviceaccount
         nodeSelector:
           kubernetes.io/os: linux
         containers:
           - name: nginx-ingress-controller
             image: lizhenliang/nginx-ingress-controller:0.30.0
             args:
               - /nginx-ingress-controller
               - --configmap=$(POD_NAMESPACE)/nginx-configuration
               - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
               - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
               - --publish-service=$(POD_NAMESPACE)/ingress-nginx
               - --annotations-prefix=nginx.ingress.kubernetes.io
             securityContext:
               allowPrivilegeEscalation: true
               capabilities:
                 drop:
                   - ALL
                 add:
                   - NET_BIND_SERVICE
               # www-data -> 101
               runAsUser: 101
             env:
               - name: POD_NAME
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.name
               - name: POD_NAMESPACE
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.namespace
             ports:
               - name: http
                 containerPort: 80
                 protocol: TCP
               - name: https
                 containerPort: 443
                 protocol: TCP
             livenessProbe:
               failureThreshold: 3
               httpGet:
                 path: /healthz
                 port: 10254
                 scheme: HTTP
               initialDelaySeconds: 10
               periodSeconds: 10
               successThreshold: 1
               timeoutSeconds: 10
             readinessProbe:
               failureThreshold: 3
               httpGet:
                 path: /healthz
                 port: 10254
                 scheme: HTTP
               periodSeconds: 10
               successThreshold: 1
               timeoutSeconds: 10
             lifecycle:
               preStop:
                 exec:
                   command:
                     - /wait-shutdown
   
   ---
   
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: ingress-nginx
     namespace: ingress-nginx
     labels:
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
   spec:
     limits:
     - min:
         memory: 90Mi
         cpu: 100m
       type: Container
   ```

   ```sh
   # 查看 ingress controller 状态
   kubectl get pods -n ingress-nginx
   ```

3. 创建 Ingress 规则

   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-myapp
     namespace: default
     annotations:
       kubernetes.io/ingess.class: "nginx"
   spec:
     rules:
     - host: shadow.com
       http:
          paths:
          - path:
            backend:
              serviceName: web  ##注此处必须要和后端pod的service的名称一致，否则会报503错误
              servicePort: 80     ##注此处必须要和后端pod的service的端口一致，否则会报503错误
   ```

   ```sh
   # 运行
   kubectl apply -f ingress-rule.yaml
   # 查看 pod
   kubectl get pods -n ingress-nginx -o wide
   # 在分配的节点查看占用的端口
   ps -ef | grep 80
   ps -ef | grep 443
   # 查看 ingress
   kubectl get ingress
   # 查看 svc
   kubectl get svc
   ```

4. 本地主机 hosts 文件配置配置

   ```sh
   10.0.0.225  shadow.com
   ```

5. 访问服务

   ```sh
   shadow.com
   ```

  

#### 5、ConfigMap

##### 1. ConfigMap 简介

ConfigMap 功能在Kubernetes1.2 版本中引入，许多应用程序会从配置文件、命令行参数

或环境变量中读取配置信息。ConfigMap API 给我们提供了向容器中注入配置信息的机
制，ConfigMap 可以被用来保存单个属性，也可以用来保存整个配置文件或者JSON 二进
制大对象

##### 2. ConfigMap 存在的意义

 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 可以将其用作==环境变量==、==命令行参数==或者==存储卷中的配置文件==，存储不加密数据到 etcd

##### 3. ConfigMap 的使用

- 创建配置文件

  ```sh
  cd /usr/local/k8s/cm
  vim redis.properties
  ```

  输入以下内容

  ```properties
  redis.host=127.0.0.1
  redis.port=6379
  redis.password=123456
  ```

- 使用文件创建 ConfigMap [--from-file]

  ```sh
  kubectl create configmap redis-config --from-file=/usr/local/k8s/cm
  # 查看
  kubectl get cm
  kubectl describe cm
  ```

  - --from-file 指定目录下的所有文件都会被用在ConfigMap里面创建一个键值对，键的名字就是文件名，值就是文件的内容

  ```sh
  # 指定一个配置文件创建 ConfigMap
  kubectl create configmap game-config-2 --from-file=/usr/local/configmap/redis.properties
  # 查看
  kubectl get configmaps game-config-2 -o yaml
  ```

  - --from-file 可以使用多次，指定多个文件

- 使用字面值创建 ConfigMap [--from-literal]

  ```sh
  kubectl create configmap my-config --from-literal=my.name=shadow  --from-literal=my.addr=张家界
  # 查看
  kubectl get configmaps my-config -o yaml
  ```

- Pod 中使用 ConfigMap

  - 数据卷 Volume 方式 cm.yaml

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      containers:
        - name: busybox
          image: busybox
          command: [ "/bin/sh","-c","cat /etc/config/redis.properties" ]
          volumeMounts:
          - name: config-volume
            mountPath: /etc/config
      volumes:
        - name: config-volume
          configMap:
            name: redis-config
      restartPolicy: Never
    ```
    
    ```sh
    # 执行
    kubectl apply -f cm.yaml
    # 查看
    kubectl logs mypod
    # 删除
    kubectl delete -f cm.yaml
    ```
    
  - 变量方式  myconfig.yaml
  
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: myconfig
      namespace: default
    data:
      special.level: info
      special.type: hello
    ```
    
  ```sh
    # 执行
  kubectl apply -f myconfig.yaml
    # 查看
    kubectl get cm
    ```
    
    cm-var.yaml
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      containers:
        - name: busybox
          image: busybox
          command: [ "/bin/sh", "-c", "echo $(LEVEL) $(TYPE)" ]
          env:
            - name: LEVEL
              valueFrom:
                configMapKeyRef:
                  name: myconfig
                  key: special.level
            - name: TYPE
              valueFrom:
                configMapKeyRef:
                  name: myconfig
                  key: special.type
      restartPolicy: Never
    ```
    
    ```sh
    # 执行
    kubectl apply -f cm-var.yaml
    # 查看
    kubectl get cm
    kubectl logs mypod
    ```
  
- 执行命令

  ```sh
  kubectl apply -f xxx.yaml
  # 查看
  kubectl get cm
  kubectl describe cm xxx
  kubectl get pods
  kubectl logs xxx
  ```

  

#### 6、Secret

##### 1. Secret 存在的意义

 ConfigMap 解决了配置明文存储的问题，Secret 解决了密码，token，秘钥等敏感信息数据的配置存储问题。Secret可以以Volume或者环境变量的方式使用

##### 2. Secret 类型

- Service Account

  用来访问 Kubernetes API ，由 Kubernetes 自动创建，并且会自动挂载到 Pod的/run/secrets/kubernetes.io/serviceaccont 目录中

- **==Opaque==**

  base64 编码格式的Secret，用来存储密码，秘钥等

- kubernetes.io/dockerconfigjson

  用来存储私钥 docker registry 的认证信息

##### 3. Secret 的使用

- Service Account

  ```sh
  # 创建 deployment
  kubectl run nginx --image=nginx
  # 查看 pod
  kubectl get pods
  # 查看 pod 内部文件
  kubectl exec nginx-xxx ls /run/secrets/kubernetes.io/serviceaccount
  ```

- Opaque

  Opaque 类型的数据是一个 map 类型，要求 value 是 base64 编码格式

  ```sh
  # 得到 base64 编码内容
  echo -n "admin" | base64
  # YWRtaW4=
  echo -n "Ptmjygb@1002" | base64
  # UHRtanlnYkAxMDAy
  ```

  - 挂载券的方式使用 Secret sec-vol.yaml

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: mysecret
  type: Opaque
  data:
    username: YWRtaW4=
    password: UHRtanlnYkAxMDAy
    
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: secret-test
    labels:
      name: secret-test
  spec:
    containers:
    - name: nginx
      image: nginx:1.15
      volumeMounts:
      - name: secrets
        mountPath: "/etc/secrets"
        readOnly: true
    volumes:
    - name: secrets
      secret:
        secretName: mysecret
  ```

  ```sh
  # 执行
  kubectl apply -f sec-vol.yaml
  # 查看
  kubectl get pods
  kubectl exec -it secret-test bash
  >ls /etc/secrets
  >cd /etc/secrets
  >cat username
  >cat password
  >exit
  kubectl delete -f sec-vol.yaml
  ```

  - 环境变量的方式使用 Secret sec-var.yaml

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: mysecret
  type: Opaque
  data:
    username: YWRtaW4=
    password: UHRtanlnYkAxMDAy
  immutable: true
  
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: secret-env-pod
  spec:
    containers:
    - name: redis
      image: redis
      env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
    restartPolicy: Never      
  ```

  ```sh
  # 执行
  kubectl appy -f sec-var.yaml
  # 查看
  kubectl get pods
  kubectl exec -it secret-env-pod bash
  >echo $SECRET_USERNAME
  >echo $SECRET_PASSWORD
  >exit
  # 删除
  kubectl delete -f sec-var.yaml
  ```

- kubernetes.io/dockerconfigjson

  使用 Kubernetes 创建 docker registry 认证的 secret

  ```sh
  kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER --docker-name=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL secret "myregistrykey"
  ```

  Pod 中使用 imagePullSecrets 指定

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: foo
  spec:
    containers:
    - name: foo
      image: nginx
    # 引用创建的 myregistrykey  
    imagePullSecrets:
    - name: myregistrykey
  ```



### 四、Helm

##### 1、Helm 及其存在的意义

​	Helm 是一个 Kubernetes 的包管理工具，可以很方便的将打包好的 yaml 部署到 kubernetes上

- 使用 helm 可以整体的管理 yaml
- 实现 yaml 高效复用
- 使用 helm 应用级别的版本管理

##### 2、Helm 的三个概念

- helm

  是一个命令行客户端工具，主要用于 chart 的创建、打包、发布和管理

- Chart

  把 yaml 打包，是yaml 的集合，k8s资源文件描述的结集合

- Release

  基于chart部署实体，应用级别的版本管理

##### 3、helm V3 版本

- V3 版本删除 Tiller

- release 可以在不同的命名空间重用

- 将chart推送到docker仓库中

  ![v3]()

##### 4、[helm 安装](https://helm.sh/)

- 下载 helm 安装压缩文件，上次 linux 服务器

  ```sh
  helm-v3.0.0-linux-amd64.tar.gz
  ```

- 解压缩 helm 安装包，移动 helm 到 /usr/bin 下

  ```sh
  tar -zxvf helm-v3.0.0-linux-amd64.tar.gz
  # 重命名 
  mv linux-amd64 helm-v3
  # 移动 helm 命令
  mv helm-v3/helm /usr/bin
  ```

- 配置 helm 仓库

  - 添加仓库

    ```sh
    # helm repo add 仓库名称 仓库地址
    helm repo add micsoft http://mirror.azure.cn/kubernetes/charts
    helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
    # 更新仓库
    helm repo update
    # 查看仓库
    helm repo list
    ```

  - 删除仓库

    ```sh
    helm repo remove aliyun
    ```

##### 5、helm 使用

- 使用命令搜索应用

  ```sh
  # helm search repo 名称
  helm search repo weave
  ```

- 根据搜索内容选择安装

  ```sh
  # helm install 安装后名称 搜索之后应用名称
  helm install ui micsoft/weave-scope
  # 查看安装之后状态
  helm list
  # helm status 安装后名称
  helm status ui
  # 查看 pods
  kubectl get pods,svc
  ```

- 修改 svc 的 yaml 文件，对外暴露端口

  ```sh
  # kubectl edit svc svc名称
  kubectl edit svc ui-weave-scope
  # 修改 svc 的 type=NodePort
  # 再次查看 对外暴露端口号
  kubectl get svc
  # 访问服务
  10.0.0.225:对外端口号
  ```

##### 6、自己创建 chart

- 使用命令创建 chart

  ```sh
  # helm create chart名称
  helm create mychart
  ```

  - charts

  - Chart.yaml

    当前 chart 属性配置信息

  - templates

    编写的 yaml 文件放到这个目录

  - vaues.yaml

    yaml 文件中科院使用的全局变量

- templates 文件夹中创建 yaml

  - deployment.yaml

    ```sh
    kubectl create deployment web01 --image=nginx:1.15 --dry-run -o yaml > deployment.yaml
    ```

  - 创建 pod

    ```sh
    kubectl apply -f deployment.yaml
    ```

  - service.yaml

    ```sh
    kubectl expose deployment web01 --port=80 --target-port=80 --type=NodePort --dry-run -o yaml > service.yaml
    ```

  - 删除创建的 pod

    ```sh
    kubectl delete deployment web01
    ```

- 回到mychart上层目录进行安装

  ```sh
  helm install web01 mychart/
  ```

- 查看 pod svc

  ```sh
  kubectl get pods,svc
  ```

- 应用升级

  ```sh
  # 修改 yaml 后应用升级
  helm upgrade web01 mychart/
  ```

##### 7、高效的 yaml 复用

​	通过 values.yaml 传递参数，动态渲染模板，yaml 内容动态生成

- 主要的几个可变的地方
  - image
  - tag
  - label
  - port
  - replicas

- values.yaml 文件定义 yaml 文件全局变量

  ```sh
  vim values.yaml
  # 输入以下内容
  replicas: 1
  image: nginx
  tag: 1.15
  label: nginx
  port: 80
  ```

- 在具体的 yaml 文件，获取定义变量值

  通过表达式形式使用全局变量 **=={{.Values.变量名称}}==** 或 **=={{.Release.Name}} (动态生成名称)==**

  - 修改 deployment

    ```sh
    cd templates
    vim deployment.yaml
    ```

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: web01
      name: {{.Release.Name}}-deploy
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: {{.Values.label}}
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: {{.Values.label}}
        spec:
          containers:
          - image: {{.Values.image}}:{{.Values.tag}}
            name: nginx
            resources: {}
    status: {}
    ```

  - 修改 service.yaml

    ```sh
    vim service.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: web01
      name: {{.Release.Name}}-svc
    spec:
      ports:
      - port: {{.Values.port}}
        protocol: TCP
        targetPort: 80
      selector:
        app: {{.Values.label}}
      type: NodePort
    status:
      loadBalancer: {}
    ```

- 执行

  ```sh
  # 尝试运行
  helm install --dry-run web02 mychart/
  # 运行
  helm install web02 mychart/
  ```

  

### 五、持久化存储

##### 1、nfs 网络存储

- 在 nfs 服务器上安装 nfs 服务

  ```sh
  yum install -y nfs-utils
  ```

- 设置挂载目录

  ```sh
  vim /etc/exports
  # 输入以下挂载内容
  /data/nfs *(rw,no_root_squash)
  ```

- 挂载路径需要创建出来

  ```sh
  mkdir -p /data/nfs
  ```

- 在 k8s node 节点上安装 nfs

  ```sh
  yum install -y nfs-utils
  ```

- 在 nfs 服务器上启动 nfs 服务

  ```sh
  systemctl start nfs
  # 检查
  ps -ef | grep nfs
  ```

- k8s 中使用

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-dep1
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx
          volumeMounts:
          - name: wwwroot
            mountPath: /usr/share/nginx/html
          ports:
          - containerPort: 80
        volumes:
          - name: wwwroot
            nfs:
              server: 10.0.0.226
              path: /data/nfs
  ```

  ```sh
  kubectl apply -f nfs.yaml
  # 查看pod
  kubectl get pods
  # 进入pod
  kubectl exec -it nginx-dep1-c9cddc7d4-nzcdt bash
  # 查看目录
  ls /usr/share/nginx/html
  ```

  ```sh
  # 在 nfs 服务器上创建 index.html
  cd /data/nfs
  vim index.html
  # 输入内容
  hello nfs
  ```

  ```sh
  # 在 pod 中再次查看目录，发现已经有index.html 文件
  ls /usr/share/nginx/html
  ```

- 对外暴露端口

  ```sh
  kubectl expose deployment nginx-dep1 --port=80 --target-port=80 --type=NodePort
  # 查看 svc
  kubectl get svc
  # 浏览器访问IP 端口
  http://IP:port 
  ```

##### 2、PV 与 PVC

- PV 

  持久化存储，对存储资源镜像抽象，对外提供可以调用的地方（生产者）

- PVC

  用于调用，不需要关心内部实现细节（消费者）

- 实现流程

  - 定义 PV

    - 存储容量 resources

    - 匹配模式 accessModes

  - 定义 PVC

    绑定 PV

##### 3、实现案例

- pvc.yaml

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-dep1
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx
          volumeMounts:
          - name: wwwroot
            mountPath: /usr/share/nginx/html
          ports:
          - containerPort: 80
        volumes:
        - name: wwwroot
          persistentVolumeClaim:
            claimName: my-pvc
  
  ---
  
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pvc
  spec:
    # 匹配模式
    accessModes:
      - ReadWriteMany
    resources:
      # 容量限制
      requests:
        storage: 5Gi
  ```

  ```sh
  # 执行
  kubectl apply -f pvc.yaml
  # 查看
  kubectl get pods
  ```

- pv.yaml

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: my-pv
  spec:
    capacity:
      storage: 5Gi
    accessModes:
      - ReadWriteMany
    nfs:
      path: /data/nfs
      server: 10.0.0.226
  ```

  ```sh
  # 执行
  kubectl apply -f pv.yaml
  # 查看
  kubectl get pods
  ```

  ```sh
  # 修改 index.html 内容
  echo "this is pv and pvc" >> /data/nfs/index.html
  # 访问服务
  http://IP:port
  ```

  

### 六、集群安全

### 七、集群监控

#### 1、监控指标

##### 1. 集群监控

- 节点资源利用率
- 节点树
- 运行 pods

##### 2. Pod 监控

- 容器指标
- 应用程序

#### 2、监控平台

##### 1. Prometheus

- 开源的
- 监控、告警、数据库
- 以 HTTP 协议周期性抓取被监控组件状态
- 不需要复杂的集成过程，使用http接口接入

##### 2. Grafana

- 开源的，数据分析和可视化工具
- 支持多种数据源

#### 3、处理流程图



#### 4、监控平台搭建

##### 1. 部署 Prometheus

- node-exporter.yaml

  ```yaml
  ---
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: node-exporter
    namespace: kube-system
    labels:
      k8s-app: node-exporter
  spec:
    selector:
      matchLabels:
        k8s-app: node-exporter
    template:
      metadata:
        labels:
          k8s-app: node-exporter
      spec:
        containers:
        - image: prom/node-exporter
          name: node-exporter
          ports:
          - containerPort: 9100
            protocol: TCP
            name: http
  ---
  apiVersion: v1
  kind: Service
  metadata:
    labels:
      k8s-app: node-exporter
    name: node-exporter
    namespace: kube-system
  spec:
    ports:
    - name: http
      port: 9100
      nodePort: 31672
      protocol: TCP
    type: NodePort
    selector:
      k8s-app: node-exporter
  ```

- rbac-setup.yaml

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: prometheus
  rules:
  - apiGroups: [""]
    resources:
    - nodes
    - nodes/proxy
    - services
    - endpoints
    - pods
    verbs: ["get", "list", "watch"]
  - apiGroups:
    - extensions
    resources:
    - ingresses
    verbs: ["get", "list", "watch"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: prometheus
    namespace: kube-system
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: prometheus
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: prometheus
  subjects:
  - kind: ServiceAccount
    name: prometheus
    namespace: kube-system
  ```

- configmap.yaml

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: prometheus-config
    namespace: kube-system
  data:
    prometheus.yml: |
      global:
        scrape_interval:     15s
        evaluation_interval: 15s
      scrape_configs:
  
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
  
      - job_name: 'kubernetes-nodes'
        kubernetes_sd_configs:
        - role: node
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics
  
      - job_name: 'kubernetes-cadvisor'
        kubernetes_sd_configs:
        - role: node
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
  
      - job_name: 'kubernetes-service-endpoints'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
          action: replace
          target_label: __scheme__
          regex: (https?)
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          target_label: __address__
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: kubernetes_name
  
      - job_name: 'kubernetes-services'
        kubernetes_sd_configs:
        - role: service
        metrics_path: /probe
        params:
          module: [http_2xx]
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
          action: keep
          regex: true
        - source_labels: [__address__]
          target_label: __param_target
        - target_label: __address__
          replacement: blackbox-exporter.example.com:9115
        - source_labels: [__param_target]
          target_label: instance
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          target_label: kubernetes_name
  
      - job_name: 'kubernetes-ingresses'
        kubernetes_sd_configs:
        - role: ingress
        relabel_configs:
        - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
          regex: (.+);(.+);(.+)
          replacement: ${1}://${2}${3}
          target_label: __param_target
        - target_label: __address__
          replacement: blackbox-exporter.example.com:9115
        - source_labels: [__param_target]
          target_label: instance
        - action: labelmap
          regex: __meta_kubernetes_ingress_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_ingress_name]
          target_label: kubernetes_name
  
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name
  ```

- prometheus.deploy.yml

  ```yaml
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      name: prometheus-deployment
    name: prometheus
    namespace: kube-system
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: prometheus
    template:
      metadata:
        labels:
          app: prometheus
      spec:
        containers:
        - image: prom/prometheus:v2.0.0
          name: prometheus
          command:
          - "/bin/prometheus"
          args:
          - "--config.file=/etc/prometheus/prometheus.yml"
          - "--storage.tsdb.path=/prometheus"
          - "--storage.tsdb.retention=24h"
          ports:
          - containerPort: 9090
            protocol: TCP
          volumeMounts:
          - mountPath: "/prometheus"
            name: data
          - mountPath: "/etc/prometheus"
            name: config-volume
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 500m
              memory: 2500Mi
        serviceAccountName: prometheus    
        volumes:
        - name: data
          emptyDir: {}
        - name: config-volume
          configMap:
            name: prometheus-config   
  ```

- prometheus.svc.yml

  ```yaml
  ---
  kind: Service
  apiVersion: v1
  metadata:
    labels:
      app: prometheus
    name: prometheus
    namespace: kube-system
  spec:
    type: NodePort
    ports:
    - port: 9090
      targetPort: 9090
      nodePort: 30003
    selector:
      app: prometheus
  ```

  ```sh
  # 查看
  kubectl get pods -n kube-system
  ```

##### 2. 部署 Grafana

- grafana-deploy.yaml

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: grafana-core
    namespace: kube-system
    labels:
      app: grafana
      component: core
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: grafana
        component: core
    template:
      metadata:
        labels:
          app: grafana
          component: core
      spec:
        containers:
        - image: grafana/grafana:4.2.0
          name: grafana-core
          imagePullPolicy: IfNotPresent
          # env:
          resources:
            # keep request = limit to keep this container in guaranteed class
            limits:
              cpu: 100m
              memory: 100Mi
            requests:
              cpu: 100m
              memory: 100Mi
          env:
            # The following env variables set up basic auth twith the default admin user and admin password.
            - name: GF_AUTH_BASIC_ENABLED
              value: "true"
            - name: GF_AUTH_ANONYMOUS_ENABLED
              value: "false"
            # - name: GF_AUTH_ANONYMOUS_ORG_ROLE
            #   value: Admin
            # does not really work, because of template variables in exported dashboards:
            # - name: GF_DASHBOARDS_JSON_ENABLED
            #   value: "true"
          readinessProbe:
            httpGet:
              path: /login
              port: 3000
            # initialDelaySeconds: 30
            # timeoutSeconds: 1
          volumeMounts:
          - name: grafana-persistent-storage
            mountPath: /var
        volumes:
        - name: grafana-persistent-storage
          emptyDir: {}
  ```

- grafana-svc.yaml

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: grafana
    namespace: kube-system
    labels:
      app: grafana
      component: core
  spec:
    type: NodePort
    ports:
      - port: 3000
    selector:
      app: grafana
      component: core
  ```

- grafana-ing.yaml

  ```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
     name: grafana
     namespace: kube-system
  spec:
     rules:
     - host: k8s.grafana
       http:
         paths:
         - path: /
           backend:
            serviceName: grafana
            servicePort: 3000
  ```

  ```sh
  # 查看
  kubectl get pods -n kube-system
  ```

##### 3. 打开 Grafana ，配置数据源，设置显示模板

- 访问服务

  ```sh
  kubectl get svc
  # 记住 prometheus 的 内部CLUSTER_IP 和端口默认9090
  # 用于配置 grafana 的 数据源
  # 访问 grafana 的对外暴露端口，访问服务
  ```

- 登录

  ```sh
  admin/admin
  ```

- 添加数据源

  ```sh
  # 选择 prometheus
  # 注意 ip 是 ClusterIP
  ```

- 设置显示模板

  ```sh
  import -> 315 -> db
  ```

  



### 八、高可用集群搭建

### 九、java 项目镜像制作

##### 1、编写 java 项目

```java
demojenkins
```

##### 2、执行 maven 命令打包

```sh
mvn clean package
# 得到jar包 demojenkins.jar
```

##### 3、编写 Dockerfile 文件

```sh
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD ./target/demojenkins.jar demojenkins.jar
ENTRYPOINT ["java","-jar","/demojenkins.jar", "&"]
```

##### 4、制作镜像

```sh
docker build -t java-demo:v1 .
```

- -t 表示 打 tag
- . 表示Dockerfile文件的位置在当前位置

##### 5、查看镜像

```sh
docker images
```

##### 6、运行容器

```sh
docker run -d -p 8111:8111 java-demo:v1 -t
```

##### 7、阿里云镜像服务

- 申请个人版
- 创建命名空间
- 创建镜像仓库
- [访问凭证设置固定密码](https://cr.console.aliyun.com/cn-hangzhou/instance/credentials)

##### 8、上传镜像

- 服务器登陆阿里云服务

  ```sh
  docker login --username=weww**** registry.cn-hangzhou.aliyuncs.com
  ```

- 镜像 tag 

  ```sh
  docker tag 7c0cbef59328 registry.cn-hangzhou.aliyuncs.com/weww***/java-pro-01:[镜像版本号]
  ```

- 推送镜像到阿里云服务

  ```sh
  docker push registry.cn-hangzhou.aliyuncs.com/weww***/java-pro-01:[镜像版本号]
  ```

- 其他服务器拉去镜像测试

  ```sh
  # 先登录阿里云服务
  docker login --username=weww**** registry.cn-hangzhou.aliyuncs.com
  # 拉取镜像
  docker pull registry.cn-hangzhou.aliyuncs.com/weww***t/java-pro-01:[镜像版本号]
  # 查看镜像
  docker images
  ```

- 启动容器

  ```sh
  docker run -d --name javademo -p8111:8111 -t
  ```

- 查看容器启动情况

  ```sh
  docker ps
  ```

##### 9、k8s 操作

- deloyment

  ```sh
  kubectl create deployment javademo1 --image=registry.cn-hangzhou.aliyuncs.com/weww***/java-pro-01:1.0.0 --dry-run -o yaml > javademo.yaml
  # 查看文件
  cat javademo.yaml
  # 应用
  kubectl apply -f javademo.yaml
  # 查看 pods
  kubectl get pods -o wide
  # 扩容
  kubectl scale deployment javademo1 --replicas=3
  ```

- service 暴露服务

  ```sh
  kubectl expose deployment javademo1 --port=8111 --target-port=8111 --type=NodePort
  # 查看 svc
  kubectl get pods,svc -o wide
  ```

- 服务访问

  ```sh
  curl 10.0.0.225:随机端口/user
  ```

  

