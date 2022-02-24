# Kubernetes

## 综述

`Kubernetes`(下简称`k8s`)是源于`google`的容器编排系统，是目前世界上使用最广泛的容器编排事实标准。作为长期趋势，`k8s`使用`containerd`而不是`docker`作为容器运行时。`k8s`系统由多个节点`node`组成，可以由`kubectl get node`命令查看

```bash
[root@aminer-dc-060 ~]# kubectl get node
NAME            STATUS   ROLES                  AGE   VERSION
aminer-dc-053   Ready    <none>                 48d   v1.23.1
aminer-dc-054   Ready    <none>                 48d   v1.23.1
aminer-dc-060   Ready    control-plane,master   50d   v1.23.1
aminer-dc-061   Ready    <none>                 50d   v1.23.1
aminer-dc-062   Ready    <none>                 50d   v1.23.1
aminer-dc-066   Ready    <none>                 10d   v1.23.1
```

其中，一个节点是主节点`master`，也是控制平面`control-plane`，这个节点通常不调度基础服务外的任何服务。其他节点作为从节点，可以被调度器分配服务到节点上执行。

`k8s`作为一个分布式系统，其所有的接口通过`kube-apiserver`暴露，该服务作为`pod`运行在`master`节点的`kube-system`命名空间中，默认以`https`方式暴露`6443`端口供外部调用。`kubectl`通过`KUBECONFIG`环境变量指示的配置文件可以读取到集群的`api endpoint`和相应的鉴权信息，从而提供`CLI`服务。

## 对象

根据`k8s`的设计哲学，所有的操作都是`声明式`而非`命令式`的，这使得所有的`k8s`操作都是幂等的，在网络状况不好时也可以安全地进行重试而不用担心副作用。一个典型的`k8s`配置文件如下所示

```yaml
apiVersion: v1 # (1)
kind: PersistentVolumeClaim # (2)
metadata: # (3)
  name: ${CI_PROJECT_NAME}-pvc
spec: # (4)
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: nfs-client
```

- (1) `api`版本，`k8s`内部版本定义方式如下:
  
  - `alpha`：测试版，可能包含`bug`，可以不经通知删除
  - `beta`：经过很好地测试，默认可用，可能会发生改变但不会被删除
  - `stable`：该版本会命名为`vx`，作为稳定版本长期维护，如果变更使用新版本号
  
  由于自定义对象`CRD`的引入，后续`k8s`定义的`apiVersion`包含命名空间。可以使用命令`kubectl api-versions`查询全部可用`api`版本

- (2) 对象类型，可以使用命令`kubectl api-resources`查看，下面列出部分对象，可以知道该对象的`kind`、对应`apiVersion`，是否属于特定命名空间以及名称和缩写形式
  
  ```bash
  [root@aminer-dc-060 ~]# kubectl api-resources
  NAME                              SHORTNAMES        APIVERSION                                  NAMESPACED   KIND
  bindings                                            v1                                          true         Binding
  componentstatuses                 cs                v1                                          false        ComponentStatus
  configmaps                        cm                v1                                          true         ConfigMap
  endpoints                         ep                v1                                          true         Endpoints
  events                            ev                v1                                          true         Event
  limitranges                       limits            v1                                          true         LimitRange
  namespaces                        ns                v1                                          false        Namespace
  nodes                             no                v1                                          false        Node
  persistentvolumeclaims            pvc               v1                                          true         PersistentVolumeClaim
  persistentvolumes                 pv                v1                                          false        PersistentVolume
  pods                              po                v1                                          true         Pod
  ```

- (3) 元数据，`k8s`对象的通用配置项都由`metadata`定义，系统要求每个对象都具有`name`字段。

- (4) 配置项定义，具体对象有哪些配置和具体含义可以使用命令`kubectl explain <path>`查看，会返回`openAPI`格式的接口描述，部分结果如下
  
  ```bash
  [root@aminer-dc-060 ~]# kubectl explain pvc
  KIND:     PersistentVolumeClaim
  VERSION:  v1
  
  DESCRIPTION:
       PersistentVolumeClaim is a user's request for and claim to a persistent
       volume
  
  FIELDS:
     apiVersion   <string>
       APIVersion defines the versioned schema of this representation of an
       object. Servers should convert recognized schemas to the latest internal
       value, and may reject unrecognized values. More info:
       https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
  
     kind <string>
       Kind is a string value representing the REST resource this object
       represents. Servers may infer this from the endpoint the client submits
       requests to. Cannot be updated. In CamelCase. More info:
       https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
  
     metadata     <Object>
       Standard object's metadata. More info:
       https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
  
     spec <Object>
       Spec defines the desired characteristics of a volume requested by a pod
       author. More info:
       https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims
  
     status       <Object>
       Status represents the current information/status of a persistent volume
       claim. Read-only. More info:
       https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims
  ```
  
  [root@aminer-dc-060 ~]# kubectl explain pvc.spec
  KIND:     PersistentVolumeClaim
  VERSION:  v1
  
  RESOURCE: spec <Object>
  
  DESCRIPTION:
  
       Spec defines the desired characteristics of a volume requested by a pod
       author. More info:
       https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims
      
       PersistentVolumeClaimSpec describes the common attributes of storage
       devices and allows a Source for provider-specific attributes
  
  FIELDS:
     accessModes  <[]string>
  
       AccessModes contains the desired access modes the volume should have. More
       info:
       https://kubernetes.io/docs/concepts/storage/persistent-volumes#access-modes-1
  
     dataSource   <Object>
  
       This field can be used to specify either: * An existing VolumeSnapshot
       object (snapshot.storage.k8s.io/VolumeSnapshot) * An existing PVC
       (PersistentVolumeClaim) If the provisioner or an external controller can
       support the specified data source, it will create a new volume based on the
       contents of the specified data source. If the AnyVolumeDataSource feature
       gate is enabled, this field will always have the same contents as the
       DataSourceRef field.

```
通常使用`yaml`格式定义`k8s`对象配置，根据`yaml`语法，可以使用`---`分割不同文件，因此允许将多个对象配置写在一个配置文件中。命令

- `kubectl create -f <filename>`

- `kubectl apply -f <filename>`

- `kubectl delete -f <filename>`

分别对应对象的创建、修改和删除，如果多个配置有某个不符合规范，`k8s`会在该配置处停止。这意味着该配置前的变更被应用；该配置及之后的变更被搁置。

可以使用`kubectl get <kind>`来查看已声明的对象，特别地，使用`kubectl get all`来打印全部的常用基础对象，包括`pod`、`service`、`daemonset`、`deploymemt`、`replicaset`和`statefulset`

使用`kubectl describe <kind> <name>`查看对象详情

使用`kubectl edit <kind> <name>`对对象属性进行编辑，该命令会打开一个类似`vim`的程序，并在保存时对设置进行校验，只有通过校验配置才会被更新并关闭编辑器

使用`kubectl delete <kind> <name>`删除一个对象

对象可以由`kubectl create`直接创建，比如建立命名空间：`kubectl create ns <name>`

## Pod

`pod`是`k8s`调度的最小单位，默认情况下每个节点可执行的`pod`数量上限是`110`。

一个典型的`pod`定义如下

```yaml
apiVersion: v1
kind: Pod
metadata:
name: demo
labels: # (1)
  app: aminer-ssr-3
  version: "0.1.11"
annotations: # (2)
  prometheus.io/scrape: "true"
  prometheus.io/path: "/metrics"
  prometheus.io/port: "8000"
  co.elastic.logs/enabled: "true"
  co.elastic.logs/json.add_error_key: "true"
  co.elastic.logs/json.message_key: "msg"
spec: 
imagePullSecrets: # (3)
- name: data-middle-platform
containers: # (4)
- name: aminer-ssr-3
  image: docker.aminer.cn/zhangyi/aminer-ssr-3:0.1.11 # (5)
  imagePullPolicy: IfNotPresent # (6)
  env: # (7)
  - name: ENV
    value: test
  command: [ "sh" ] # (8)
  args: [ "run.sh" ] # (9)
  ports: # (10)
  - containerPort: 5050
  readinessProbe: # (11)
    tcpSocket:
      port: 5050
    periodSeconds: 10
  volumeMounts: # (12)
  - mountPath: /service/config/${ENV}
    name: config
  - mountPath: /service/data
    name: data
volumes: # (13)
- name: config
  configMap:
    name: aminer-ssr-3-config
- name: data
  persistentVolumeClaim:
    claimName: aminer-ssr-3-pvc
```

- (1) `pod`的`label`配置，`k8s`建议`pod`配备`version`标签，`label`常用于选择器`selector`

- (2) `annotations`注解，一些集群内的后台服务使用注解发现需要服务的`pod`，这里在`8000`端口开启了`prometheus`服务，访问`/metrics`路由，并且开启了`filebeat`的`json`日志捕获服务

- (3) 镜像拉取密钥，使用同命名空间下的`docker secret`，拉取镜像使用该密钥认证

- (4) 容器列表，一个`pod`可以包含多个容器，`sideCar`边车模式也通过容器注入`pod`

- (5) 镜像名称，指明该容器运行的镜像

- (6) 镜像拉取策略，`IfNotPresent`在镜像存在时直接使用，`Always`总是拉取新镜像

- (7) 容器环境变量配置，用于向容器内部传递初始设置

- (8) 启动命令，等价于`Dockerfile`中的`ENTRYPOINT`

- (9) 启动参数，等价于`Dockerfile`中的`CMD`

- (10) 暴露端口，等价于`Dockerfile`中的`EXPOSE`

- (11) 存活探针，支持多种探针配置，用于判断`pod`是否存活

- (12) 卷挂载，用于将卷挂载到指定挂载点

- (13) 卷声明，声明`pod`用到的存储卷

尽管`pod`是`k8s`编排的最小单位，通常不会直接使用`pod`部署应用。根据场景不同，`k8s`提供了多种高阶对象来描述一个具体的服务，这些对象的核心模板是一个`pod`的定义，提供了一些额外的配置，以表达更多语义并提供更多功能。接下来几节将分别介绍这些对象。

## ReplicaSet

`k8s`用`ReplicaSet`来部署无状态的服务。`无状态`意味着副本集所有的`pod`是等价的。`ReplicaSet`对象确保可以在`k8s`中拉起指定数量的`pod`。

典型配置如下

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: test
spec:
  replicas: 1 #(1)
  selector: # (2)
    matchLabels:
      app: aminer-ssr-3
  template: {...} # (3)
```

- (1) 副本数量，用于确定拉起几个副本

- (2) 选择器，`k8s`使用该选择器确定哪些副本属于该副本集

- (3) `pod`模板，类似于`pod`配置，详细描述了每个副本的属性

可以使用`kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50`对`rs`进行扩容或缩容。

通常不会直接使用`ReplicaSet`对象，而是使用`Deployment`对象进行无状态服务的部署。`Deployment`提供包括滚动更新在内的很多实用功能，这些功能是`Deployment`通过操作`ReplicaSet`实现的。

当删除`rs`时，垃圾回收器也会删除所有依赖的`pod`。

## Deployment

通常使用`Deployment`对象部署无状态服务，典型配置如下

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  revisionHistoryLimit: 1
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

该配置创建了名为`nginx-deployment`的部署，该部署创建了一个`ReplicaSet`，负责启动`3`个`pod`副本。

当服务更新时，修改该部署对应的配置，例如`kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record`，将服务镜像由`nginx:1.14.2`更新至`nginx:1.16.1`，会触发滚动更新

具体原理为：`deployment`为新配置建立新的`rs`，然后在保证更新时仅关闭一定数量`pod`的情况下，逐步将服务升级到新版本。部署首先建立一个新`pod`，在`ready`后关闭一个旧`pod`，重复这个过程直到完成滚动更新。在更新过程中，存在一段时间新老服务并存，且在全量时间内，服务是一直在线的。如果新`pod`无法到达`ready`状态，`deployment`不会强制更新，这时可以进行部署回滚以停止有问题的上线过程。

使用命令`kubectl rollout undo deployment nginx-deployment`回滚到之前版本

使用命令`kubectl scale deployment nginx-deployment --replicas=10`缩放`deployment`

使用命令`kubectl get deployment.apps/org-service -o yaml | kubectl replace --force -f -`重启服务

使用字段`revisionHistoryLimit`限定保留`history`的个数，默认为`10`

## StatefulSets

`statefulSet`用来管理有状态应用，为每个`pod`维护了一个由粘性的`id`，无论怎么调度，每个`pod`都由一个永久不变的`pod`。同样地，如果分配了持久卷，`statefulSet`也会保证

TODO 完善用例

## DaemonSet

`daemonSet`确保每个节点运行一个`pod`副本，常用于守护进程、日志收集和监控

TODO 完善用例

## Job/CronJob

TODO 任务和定时任务
