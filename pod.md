个人对pod的理解，pod是kubernetes中的最小单元（官方），pod可由一个或者多个docker容器构成，pod主要由yaml/json创建，pod可水平扩展应用程序。pod可以单独创建在节点上。测试后发现，pod创建在节点后，当这个节点出现问题重启或者pod出现问题，pod会被删除。官方：Pod不会自愈。如果Pod运行的Node故障，或者是调度器本身故障，这个Pod就会被删除。同样的，如果Pod所在Node缺少资源或者Pod处于维护状态，Pod也会被驱逐。Kubernetes使用更高级的称为Controller的抽象层，来管理Pod实例。虽然可以直接使用Pod，但是在Kubernetes中通常是使用Controller来管理Pod的。

# **官方pod介绍：**

一个_吊舱_是在创建或部署Kubernetes对象模型Kubernetes-最小最简单的单元的基本构建块。Pod表示群集上正在运行的进程。

Pod封装了一个应用程序容器（或者，在某些情况下，多个容器），存储资源，一个唯一的网络IP以及控制容器应该如何运行的选项。Pod表示部署单元：_Kubernetes中的单个应用程序实例_，可能包含单个容器或少量紧密耦合且共享资源的容器。

> [Docker](https://www.docker.com/)是Kubernetes Pod中最常用的容器运行时，但Pods也支持其他容器运行时。

Kubernetes集群中的Pod可以以两种主要方式使用：

* **运行单个容器的Pod**
  。
  “one-container-per-Pod”模型是最常见的Kubernetes用例;
  在这种情况下，您可以将Pod视为单个容器的包装，而Kubernetes直接管理Pod而不是容器。
* **运行多个需要协同工作的容器的Pod**
  。
  Pod可能封装了一个由多个共址容器组成的应用程序，这些容器紧密耦合并需要共享资源。
  这些共处一地的容器可能形成一个统一的服务单元 - 一个容器从共享卷向公众提供文件，而一个单独的“sidecar”容器刷新或更新这些文件。
  Pod将这些容器和存储资源作为单个可管理实体包装在一起。

该[Kubernetes博客](http://kubernetes.io/blog)对波德用例一些额外的信息。有关更多信息，请参阅：

* [分布式系统工具包：复合容器的模式](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
* [容器设计模式](https://kubernetes.io/blog/2016/06/container-design-patterns)

每个Pod都用于运行给定应用程序的单个实例。如果要水平扩展应用程序（例如，运行多个实例），则应使用多个Pod，每个实例一个。在Kubernetes中，这通常被称为_复制_。复制Pod通常通过称为Controller的抽象创建和管理。有关更多信息，请参阅[Pod和控制器](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pods-and-controllers)。

### Pod如何管理多个容器 {#how-pods-manage-multiple-containers}

Pod旨在支持多个协作流程（作为容器），形成一个有凝聚力的服务单元。Pod中的容器将自动共存，并在群集中的同一物理或虚拟机上进行共同调度。容器可以共享资源和依赖关系，彼此通信，并协调它们何时以及如何终止。

请注意，将多个共址和共同管理的容器分组到一个Pod中是一个相对高级的用例。您应该仅在容器紧密耦合的特定实例中使用此模式。例如，您可能有一个容器充当共享卷中文件的Web服务器，以及一个单独的“sidecar”容器，用于从远程源更新这些文件，如下图所示：

![](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)![](/assets/import.png)

#### pod图 {#pod-diagram}

Pod为其组成容器提供两种共享资源：_网络_和_存储_。

#### 联网 {#networking}

为每个Pod分配一个唯一的IP地址。Pod中的每个容器都共享网络命名空间，包括IP地址和网络端口。_Pod内的_容器可以使用相互通信`localhost`。当Pod中的容器与Pod_外部的_实体通信时，它们必须协调它们如何使用共享网络资源（例如端口）。

#### 存储 {#storage}

Pod可以指定一组共享存储_卷_。Pod中的所有容器都可以访问共享卷，允许这些容器共享数据。如果需要重新启动其中一个容器，则卷还允许Pod中的持久数据存活。见[卷](https://kubernetes.io/docs/concepts/storage/volumes/)上Kubernetes如何实现一个豆荚里的共享存储更多的信息。

## 使用Pods {#working-with-pods}

你很少直接在Kubernetes - 甚至是单身Pod中创建单独的Pod。这是因为Pods被设计为相对短暂的一次性实体。当Pod（由您直接创建或由Controller间接创建）时，它将被安排在群集中的节点上运行。Pod保留在该节点上，直到进程终止，pod对象被删除，pod因资源不足而被_驱逐_，或者Node失败。

> **注意：**  
> 不应将重新启动Pod重新启动Pod中的容器。  
> Pod本身不会运行，但是容器运行的环境会持续存在，直到删除为止。

豆荚本身不能自我修复。如果将Pod调度到失败的节点，或者调度操作本身失败，则删除Pod;同样，由于缺乏资源或节点维护，Pod将无法在驱逐中存活。Kubernetes使用更高级别的抽象，称为_Controller_，它处理管理相对可处理的Pod实例的工作。因此，尽管可以直接使用Pod，但在Kubernetes中使用Controller管理pod更为常见。有关Kubernetes如何使用控制器实现Pod缩放和修复的更多信息，请参阅[Pods和控制器](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pods-and-controllers)。

### 荚和控制器 {#pods-and-controllers}

Controller可以为您创建和管理多个Pod，处理复制和部署，并在集群范围内提供自我修复功能。例如，如果节点发生故障，Controller可能会通过在不同节点上安排相同的替换来自动替换Pod。

## 什么是Pod {#what-is-a-pod}

一个_pod_（如在一个鲸鱼或豌豆荚中）是一组一个或多个容器（如Docker容器），具有共享存储/网络，以及如何运行容器的规范。pod的内容始终位于同一位置并共同调度，并在共享上下文中运行。pod模拟特定于应用程序的“逻辑主机” - 它包含一个或多个相对紧密耦合的应用程序容器 - 在预容器世界中，在同一物理或虚拟机上执行意味着在同一逻辑主机上执行。

虽然Kubernetes支持的容器运行时间多于Docker，但Docker是最常见的运行时，它有助于用Docker术语描述pod。

pod的共享上下文是一组Linux命名空间，cgroup，以及可能的隔离方面 - 与隔离Docker容器相同的东西。在pod的上下文中，各个应用程序可能会应用进一步的子隔离。

pod中的容器共享IP地址和端口空间，并且可以通过它们找到彼此`localhost`。它们还可以使用SystemV信号量或POSIX共享内存等标准进程间通信相互通信。不同pod中的容器具有不同的IP地址，并且在没有[特殊配置的](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)情况下无法通过IPC进行通信。这些容器通常通过Pod IP地址相互通信。

pod中的应用程序还可以访问共享卷，共享卷被定义为pod的一部分，可以安装到每个应用程序的文件系统中。

就[Docker](https://www.docker.com/)构造而言，pod被建模为一组具有共享命名空间和共享[卷](https://kubernetes.io/docs/concepts/storage/volumes/)的Docker容器。

与单个应用程序容器一样，pod被认为是相对短暂的（而不是持久的）实体。正如在[pod的生命周期中](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)所讨论的，创建[pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)，分配唯一ID（UID），并调度到它们保留的节点，直到终止（根据重启策略）或删除。如果节点终止，则在超时期限之后，将调度计划到该节点的Pod进行删除。给定的pod（由UID定义）不会“重新安排”到新节点;相反，它可以被相同的pod替换，如果需要，甚至可以使用相同的名称，但是使用新的UID（有关更多详细信息，请参阅[复制控制器](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)）。

当某些东西被认为具有与pod相同的生命周期时，例如卷，这意味着只要该pod（具有该UID）存在就存在。如果由于任何原因删除了该pod，即使创建了相同的替换，相关的东西（例如卷）也会被销毁并重新创建。

### 管理 {#management}

Pod是多个合作过程模式的模型，形成了一个有凝聚力的服务单元。它们通过提供比其组成应用程序集更高级别的抽象来简化应用程序部署和管理。Pod用作部署，水平扩展和复制的单元。对容器中的容器自动处理共置（共同调度），共享命运（例如终止），协调复制，资源共享和依赖关系管理。

### 资源共享和沟通 {#resource-sharing-and-communication}

Pod可以实现其成员之间的数据共享和通信。

pod中的应用程序都使用相同的网络命名空间（相同的IP和端口空间），因此可以相互“找到”并使用它们进行通信`localhost`。因此，pod中的应用程序必须协调它们对端口的使用。每个pod在平面共享网络空间中具有IP地址，该网络空间与网络中的其他物理计算机和pod完全通信。

主机名设置为pod中应用程序容器的pod名称。[关于网络的更多细节](https://kubernetes.io/docs/concepts/cluster-administration/networking/)。

除了定义在pod中运行的应用程序容器之外，pod还指定了一组共享存储卷。卷使数据能够在容器重新启动后继续存在，并在容器内的应用程序之间共享。

Pod可用于托管垂直集成的应用程序堆栈（例如LAMP），但其主要动机是支持共址，共同管理的帮助程序，例如：

* 内容管理系统，文件和数据加载器，本地缓存管理器等。
* 日志和检查点备份，压缩，旋转，快照等
* 数据变更观察者，日志零售商，日志和监控适配器，活动发布者等。
* 代理，网桥和适配器
* 控制器，管理器，配置器和更新器

## 终止豆荚 {#termination-of-pods}

通常，单个pod不用于运行同一应用程序的多个实例。

有关更长的说明，请参阅[分布式系统工具包：复合容器的模式](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)。

吊舱不应被视为耐用实体。它们将无法在调度故障，节点故障或其他驱逐（例如由于缺乏资源或节点维护的情况下）中存活。

通常，用户不需要直接创建pod。他们应该几乎总是使用控制器，即使是单身人士，例如，[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)。控制器提供集群范围的自我修复，以及复制和部署管理。像[StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset.md)这样的控制器也可以为有状态的pod提供支持。

使用集合API作为主要的面向用户的原语在集群调度系统中相对常见，包括[Borg](https://research.google.com/pubs/pub43438.html)，[Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html)，[Aurora](http://aurora.apache.org/documentation/latest/reference/configuration/#job-schema)和[Tupperware](http://www.slideshare.net/Docker/aravindnarayanan-facebook140613153626phpapp02-37588997)。

Pod作为基元公开，以便于：

* 调度程序和控制器可插拔性
* 支持pod级操作，无需通过控制器API“代理”它们
* pod寿命与控制器生命周期的分离，例如自举
* 控制器和服务的分离 - 端点控制器只是监视pod
* 具有集群级功能的Kubelet级功能的清晰组合 - Kubelet实际上是“pod控制器”
* 高可用性应用程序，它们将期望在终止之前更换pod，并且肯定在删除之前，例如在计划驱逐或图像预取的情况下。

因为pod表示集群中节点上的正在运行的进程，所以允许这些进程在不再需要时优雅地终止（与使用KILL信号猛烈杀死并且没有机会清理）非常重要。用户应该能够请求删除并知道进程何时终止，但也能够确保删除最终完成。当用户请求删除pod时，系统会在允许pod被强制终止之前记录预期的宽限期，并将TERM信号发送到每个容器中的主进程。宽限期到期后，KILL信号将发送到这些进程，然后从API服务器中删除该pod。如果在等待进程终止时重新启动Kubelet或容器管理器，

示例流程：

1. 用户发送删除Pod的命令，默认宽限期（30s）
2. API服务器中的Pod随着时间的推移而更新，其中Pod被视为“死”以及宽限期。
3. 在客户端命令中列出时，Pod显示为“终止”
4. （与3同时）当Kubelet发现Pod已被标记为终止，因为已经设置了2中的时间，它开始了pod关闭过程。
   1. 如果pod已定义了
      [preStop挂钩](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#hook-details)
      ，则会在pod中调用它。
      如果
      `preStop`
      挂钩在宽限期到期后仍在运行，则以小（2秒）延长的宽限期调用步骤2。
   2. Pod中的进程发送TERM信号。
5. （与3同时）Pod从端点列表中删除以进行维护，并且不再被视为复制控制器的运行pod集的一部分。
   缓慢关闭的窗格无法继续为流量提供服务，因为负载平衡器（如服务代理）会将其从旋转中移除。
6. 当宽限期到期时，仍然在Pod中运行的任何进程都将被SIGKILL杀死。
7. Kubelet将通过设置宽限期0（立即删除）完成删除API服务器上的Pod。
   Pod从API中消失，不再从客户端可见。

默认情况下，所有删除在30秒内都是正常的。该`kubectl delete`命令支持`--grace-period=<seconds>`允许用户覆盖默认值并指定其自己的值的选项。值`0`[force删除](https://kubernetes.io/docs/concepts/workloads/pods/pod/#force-deletion-of-pods)pod。在kubectl版本&gt; = 1.5时，必须指定一个额外的标志`--force`一起`--grace-period=0`，以执行力的缺失。

### 强制删除pod {#force-deletion-of-pods}

强制删除pod被定义为立即从群集状态和etcd删除pod。当执行强制删除时，apiserver不会等待来自kubelet的确认该pod已在其运行的节点上终止。它会立即删除API中的pod，以便可以使用相同的名称创建新的pod。在节点上，设置为立即终止的pod在被强制终止之前仍将被给予一个小的宽限期。

强制删除可能对某些吊舱有潜在危险，应谨慎执行。对于StatefulSet pod，请参阅任务文档以[从StatefulSet中删除](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/)Pod。

