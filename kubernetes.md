kubernetes组成部分：

etcd ：在k8s中存储建的作用

kube-apiserver ：对外暴露的k8s前端控制层。所有的组件的交互枢纽。

kube-controller-manager ：运行控制器，k8s后端控制层。处理集群中常规任务。通过apiserver维护控制每个节点。维护副本集的pod正常运行数量。

kube-scheduler ：调度器，检测新建没有分配节点的pod，根据每个节点的资源选择一个节点，分配给新建未分配节点的pod。

以上Master主要组成部分。

以下是节点node的组成部分。

kubelet ：节点主要组成部分，管理分配到节点上的pod，自动注册到apiserver，定期将节点的状态发送给apiserver。同时接收并执行apiserver传达的命令。

kube-proxy ：为Service服务提供网络转发服务，实现了内部pod到service和外部通过node port到内部service的访问。基于iptables实现service到pod负载均衡功能（老版本的负载均衡），v1.6版本使用iptables 的 kube-proxy 实际上是将集群扩展到5000个节点的瓶颈。 一个例子是，在5000节点集群中使用 NodePort 服务，如果我们有2000个服务并且每个服务有10个 pod，这将在每个工作节点上至少产生20000个 iptable 记录，这可能使内核非常繁忙。

ipvs：IPVS是LVS的核心组件，是一种四层负载均衡器。IPVS具有以下特点：  
           1与Iptables同样基于Netfilter，但使用的是hash表；  
           2支持TCP, UDP，SCTP协议，支持IPV4，IPV6；  
           3支持多种负载均衡策略：rr, wrr, lc, wlc, sh, dh, lblc…  
           4支持会话保持；

           LVS主要由两部分组成：  
          1ipvs\(ip virtual server\)：即ip虚拟服务，是工作在内核空间上的一段代码，主要是实现调度的代码，它是实现负载均衡的核心。  
          2ipvsadm： 工作在用户空间，负责为ipvs内核框架编写规则，用于定义谁是集群服务，谁是后端真实服务器。我们可以通过ipvsadm指令创建集群服务

clusterDNS：DNS服务，插件的形式，为集群中的服务提供DNS解析。

