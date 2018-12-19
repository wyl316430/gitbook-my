Kubernetes系统在长时间运行后，Kubernetes Node会下载非常多的镜像，其中可能存在很多过期的镜像。同时因为运行大量的容器，容器推出后就变成死亡容器，将数据残留在宿主机上，这样一来，过期镜像和死亡容器都会占用大量的硬盘空间。如果磁盘空间被用光，可能会发生非常糟糕的情况，甚至会导致磁盘的损坏。为此kubelete会进行垃圾清理工作，即定期清理过期镜像和死亡容器。不推荐使用其它管理工具或手工进行容器和镜像的清理，因为kubelet需要通过容器来判断pod的运行状态，如果使用其它方式清除容器有可能影响kubelet的正常工作。

**二：镜像清理**

Kubernetes通过kubelet集成的cadvisor进行镜像的回收，有两个参数可以设置：--image-gc-high-threshold和--image-gc-low-threshold。当用于存储镜像的磁盘使用率达到百分之--image-gc-high-threshold时将触发镜像回收，删除最近最久未使用（LRU，Least Recently Used）的镜像直到磁盘使用率降为百分之--image-gc-low-threshold或无镜像可删为止。默认--image-gc-high-threshold为90，--image-gc-low-threshold为80。

**三：容器清理**

容器的回收有三个参数可设置：

1.--minimum-container-ttl-duration：死亡容器能够被删除的最小TTL，默认是1分钟

2.--maximum-dead-containers-per-container：每个Pod允许存在的最大死亡容器数目，默认是2

3.--maximum-dead-containers: 运行存在的最大死亡容器数目，默认值是100.

Kubelet定时执行容器清理，每次根据以上3个参数选择死亡容器删除，通常情况下优先删除创建时间最久的死亡容器。Kubelet不会删除非Kubelet管理的容器。

