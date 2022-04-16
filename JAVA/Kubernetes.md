# 基本概念

## Kubernetes 作用

![image-20220413105014492](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220413105014492.png)

工程的部署经历了上述的3个发展过程：

1. **传统的部署时代：**在同一台机器上运行多个程序，各个程序之间没有资源边界，这可能会出现资源分配严重不均的情况。另外，由于共用系统资源，在一个程序崩溃后，可能会导致其他程序无法正常运行。
2. **虚拟化的部署时代：**会在单个物理服务器的 CPU 上运行多个虚拟机（VM），应用程序在 VM 中隔离运行，起到一定程度的保护作用。
3. **容器化的部署时代：**容器类似于 VM，但是相比较来说，它更轻量级。同时，由于它与基础架构分离，因此有着更好的移植性。

在生产环境中，我们会有很多个容器，而且还需要处理机器的故障问题，非人力能够解决，所以就需要一个自动化管理容器的工具。 Kubernetes 为我们提供了一个**可弹性运行 ** **分布式系统**的框架。 Kubernetes 会满足你的扩展要求、故障转移、部署模式等。



## Kubernetes 特性

- **服务发现和负载均衡**
  Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

- **存储编排**
  Kubernetes 会自动挂载你选择的存储系统，例如本地存储、公共云提供商等。

- **自动部署和回滚**
  你可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态 更改为期望状态。例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。

- **自动装箱**
  Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。 当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源。

- **自我修复**
  Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的 运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。

- **密钥与配置管理**
  Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

- **扩展性**

  无需更改上游源码即可扩展你的 Kubernetes 集群。

## Kubernetes 组件

![image-20220413110504250](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220413110504250.png)

### 控制平面组件（Control Plane Components）

控制平面的组件对集群做出全局决策(比如调度)，以及检测和响应集群事件（例如，当不满足部署的 `replicas` 字段时，启动新的 [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)）。

控制平面组件可以在集群中的任何节点上运行。 然而，为了简单起见，设置脚本通常会在同一个计算机上启动所有控制平面组件， 并且不会在此计算机上运行用户容器。 请参阅[使用 kubeadm 构建高可用性集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/) 中关于跨多机器控制平面设置的示例。

#### kube-apiserver

API 服务器是 Kubernetes [控制面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件公开了 Kubernetes API。API 服务器是 Kubernetes 控制面的前端。

Kubernetes API 服务器的主要实现是 [kube-apiserver](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/)。 kube-apiserver 设计上考虑了水平伸缩，也就是说，它可通过部署多个实例进行伸缩。 你可以运行 kube-apiserver 的多个实例，并在这些实例之间平衡流量。

#### etcd 

etcd 是兼具一致性和高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。

您的 Kubernetes 集群的 etcd 数据库通常需要有个备份计划。

要了解 etcd 更深层次的信息，请参考 [etcd 文档](https://etcd.io/docs/)。

#### kube-scheduler

控制平面组件，负责监视新创建的、未指定运行节点（node）的 Pods，并选择节点让 Pod 在上面运行。

调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。

#### kube-controller-manager

运行控制器进程的控制平面组件。

从逻辑上讲，每个[控制器](https://kubernetes.io/zh/docs/concepts/architecture/controller/)都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。

这些控制器包括:

- 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应
- 任务控制器（Job controller）: 监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)
- 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌

#### cloud-controller-manager

云控制器管理器是指嵌入特定云的控制逻辑的 [控制平面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane)组件。 云控制器管理器使得你可以将你的集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来。

`cloud-controller-manager` 仅运行特定于云平台的控制回路。 如果你在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部署的环境中不需要云控制器管理器。

与 `kube-controller-manager` 类似，`cloud-controller-manager` 将若干逻辑上独立的 控制回路组合到同一个可执行文件中，供你以同一进程的方式运行。 你可以对其执行水平扩容（运行不止一个副本）以提升性能或者增强容错能力。

下面的控制器都包含对云平台驱动的依赖：

- 节点控制器（Node Controller）: 用于在节点终止响应后检查云提供商以确定节点是否已被删除
- 路由控制器（Route Controller）: 用于在底层云基础架构中设置路由
- 服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器

### Node组件

节点组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行环境。

#### kubelet

一个在集群中每个节点（node）上运行的代理。 它保证容器都 运行在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 中。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器。

#### kube-proxy 

[kube-proxy](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-proxy/) 是集群中每个节点上运行的网络代理， 实现 Kubernetes [服务（Service）](https://kubernetes.io/zh/docs/concepts/services-networking/service/) 概念的一部分。

kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

如果操作系统提供了数据包过滤层并可用的话，kube-proxy 会通过它来实现网络规则。否则， kube-proxy 仅转发流量本身。

![image-20220413112856623](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220413112856623.png)

# 安装Kubernetes



# Pods

## 基本概念

在 Kubernetes 中，Pod 代表的是集群上处于运行状态的**一组容器**，是可以在 K8s 中创建和管理的、最小的可部署的计算单元。

K8s 中的 Pod 主要有两种用法：

- **运行单个容器的 Pod。** 这是最常见的 K8s 用例，这这种情况下，Pod 就类似于 Docker 中的 容器。
- **运行多个容器的 Pod。** 将多个需要协同工作且共享资源的容器容器打包成一个 Pod。

> k8s 中增加了 Pod 的概念，除了在运行多个容器时，两者的概念不同。还因为除了 Docker 外，k8s 还支持了很多的容器运行时的软件，Docker 只是其中一种。

<img src="https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220414173759855.png" alt="image-20220414173759855" style="zoom:67%;" />

# 工作负载资源

![image-20220416101306528](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220416101306528.png)

## Deployments

### 创建 Deployment

**方法一：**

```bash
kubectl create deployment nginx-deployment --image=nginx:1.14.2 --replicas=3
```

**方法二：**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment   #deployment 的名称
  labels:
    app: nginx
spec:
  replicas: 3   # pod副本数
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



```bash
kubectl apply -f nginx-deployment.yaml
```

### 查看 Deployment

可以通过运行 `kubectl get deployments` 检查 Deployment 是否已创建。输出会类似：

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
```

- `NAME` 列出了集群中 Deployment 的名称。
- `READY` 显示应用程序的可用的“副本”数。显示的模式是“就绪个数/期望个数”。
- `UP-TO-DATE` 显示为了达到期望状态已经更新的副本数。
- `AVAILABLE` 显示应用可供用户使用的副本数。
- `AGE` 显示应用程序运行的时间。



### 更新 Deployment

如果要更新 Deployment，我们可以通过**命令行**或者**修改 yaml 文件** 达到预期。

例如，要将 nginx-deplopment 使用  `nginx:1.16.1` 镜像，而不是 `nginx:1.14.2` 镜像。

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```



```bash
# 将 .spec.template.spec.containers[0].image 从 nginx:1.14.2 更改至 nginx:1.16.1，并对 Deployment 执行 edit 操作
kubectl edit deployment/nginx-deployment
```



要查看修改后 Deployment 的上线状态，运行：

```bash
kubectl rollout status deployment/nginx-deployment

# 输出会类似于：
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
# 或
deployment "nginx-deployment" successfully rolled out
```

获取 Deployment 的详细信息：

```bash
kubectl describe deployments
```



> 1. Deployment 可确保在更新时仅关闭一定数量的 Pod。默认情况下，它确保至少所需 Pods 75% 处于运行状态。
> 2. 仅当 Deployment Pod 模板（即 `.spec.template`）发生改变时，例如模板的标签或容器镜像被更新， 才会触发 Deployment 上线。其他更新（如对 Deployment 执行扩缩容的操作）不会触发上线动作。

### 扩缩放 Deployment

可以通过**命令行**或者**修改 yaml 文件** 的方式达到扩缩放 Deployent 的目的。

```bash
kubectl scale deployment/nginx-deployment --replicas=10
```



```bash
# 修改 yaml文件中的 .spec.replicas，并对 Deployment 执行 edit 操作
kubectl edit deployment/nginx-deployment
```



如果集群启用了[Pod 的水平自动缩放](https://kubernetes.io/zh/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)， 还可以为 Deployment 设置自动缩放器，并基于现有 Pod 的 CPU 利用率选择要运行的 Pod 个数下限和上限。

```bash
kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

### 回滚 Deployment

```bash
#历史记录
kubectl rollout history deployment/my-dep

#查看某个历史详情
kubectl rollout history deployment/my-dep --revision=2

#回滚(回到上次)
kubectl rollout undo deployment/my-dep

#回滚(回到指定版本)
kubectl rollout undo deployment/my-dep --to-revision=2
```



### 相关常用命令

```bash
# 获取 Deployment 详细信息
kubectl describe deployments
# 查看 Deployment 上线状态
kubectl rollout status deployment/deployment名称
# 查看所以 Deployment 的信息
kubectl get deployments
```



## StatefulSets

### 使用场景

StatefulSet 用来管理某 Pod 集合的部署和扩缩， 并为这些 Pod 提供**持久存储和持久标识符**。

和 Deployment 类似， StatefulSet 管理基于相同容器规约的一组 Pod。但和 Deployment 不同的是， StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID。这些 Pod 是基于相同的规约来创建的， 但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。

在应用程序有以下一个或多个需求时，可以考虑使用 StatefulSets：

- 稳定的、唯一的网络标识符。
- 稳定的、持久的存储。
- 有序的、优雅的部署和缩放。
- 有序的、自动的滚动更新。

## DaemonSet

### 使用场景

DaemonSet 确保全部节点上运行一个 Pod 的副本。 当有节点加入集群时， 会为他们新增一个 Pod 。 当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

DaemonSet 的一些典型用法：

- 在每个节点上运行集群守护进程
- 在每个节点上运行日志收集守护进程
- 在每个节点上运行监控守护进程



## Jobs



## CronJob



## 参考

- https://kubernetes.io/zh/docs/concepts/workloads/controllers/

