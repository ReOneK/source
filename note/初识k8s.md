### 1.一个运行的linux容器组成

- 一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图；
- 一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。



### 2."容器编排"迅速成为上层建筑的原因

- 开发者不需要关心容器运行时的差异。

- 真正承载着容器信息进行传递的，是**容器镜像**，而不是容器运行时。

- 通过容器镜像，它们可以和潜在用户（即，开发者）直接关联起来。（CI/CD、监控、安全、网络、存储等等）

  > 这样，容器就从一个开发者手里的小工具，一跃成为了云计算领域的绝对主角；而能够定义容器组织和管理规范的“容器编排”技术，则当仁不让地坐上了容器技术领域的“头把交椅”。



### 3.k8s项目主要解决的问题

![img](C:\Users\95951\Desktop\source\初识别k8s1.jpg)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



> Kubernetes 项目的架构，跟它的原型项目 Borg 非常类似，都由 Master 和 Node 两种节点组成，而这两种角色分别对应着控制节点和计算节点。其中，控制节点，即 Master 节点，由三个紧密协作的独立组件组合而成，它们分别是负责 API 服务的 kube-apiserver、负责调度的 kube-scheduler，以及负责容器编排的 kube-controller-manager。整个集群的持久化数据，则由 kube-apiserver 处理后保存在 Etcd 中。而计算节点上最核心的部分，则是一个叫作 kubelet 的组件。

- **在 Kubernetes 项目中，kubelet 主要负责同容器运行时（比如 Docker 项目）打交道。**

> 而这个交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接口，这个接口定义了容器运行时的各项核心操作，比如：启动一个容器需要的所有参数。这也是为何，Kubernetes 项目并不关心你部署的是什么容器运行时、使用的什么技术实现，只要你的这个容器运行时能够运行标准的容器镜像，它就可以通过实现 CRI 接入到 Kubernetes 项目当中。而具体的容器运行时，比如 Docker 项目，则一般通过 **OCI 这个容器运行时规范同底层的 Linux 操作系统进行交互**，即：**把 CRI 请求翻译成对 Linux 操作系统的调用（操作 Linux Namespace 和 Cgroups 等）。**

- **kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互**。

> 这个插件，是 Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件，也是基于 Kubernetes 项目进行机器学习训练、高性能作业支持等工作必须关注的功能。

- **kubelet 的另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储。**

  > 这两个插件与 kubelet 进行交互的接口，分别是 CNI（Container Networking Interface）和 CSI（Container Storage Interface）。



- **k8s着重解决的问题**

  - 运行在大规模集群中的各种任务之间，**应用与应用之间的关系**。这些关系的处理，才是作业编排和管理系统最困难的地方。
  - **应用运行的形态**是影响“如何容器化这个应用”的第二个重要因素。

- k8s的设计思想

  - 从更宏观的角度，以统一的方式来定义任务之间的各种关系
  - 且为将来支持更多种类的关系留有余地

- 在 Kubernetes 项目中，所推崇的使用方法

  : 

  - 通过一个“`编排对象`”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用；
  - 再为它定义一些“`服务对象`”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能。

> 这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）。



 **推荐阅读**：1.[架构原理 · Kubernetes指南![img](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://feisky.gitbooks.io/kubernetes/content/architecture/architecture.html](https://feisky.gitbooks.io/kubernetes/content/architecture/architecture.html)

​					2.[深入剖析 Kubernetes (geekbang.org)](https://time.geekbang.org/column/intro/100015201)