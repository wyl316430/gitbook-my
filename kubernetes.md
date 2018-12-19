kubernetes组成部分：

etcd ：在k8s中存储建的作用

kube-apiserver ：对外暴露的k8s前端控制层。所有的组件的交互枢纽。

kube-controller-manager ：运行控制器，k8s后端控制层。处理集群中常规任务。通过apiserver维护控制每个节点。维护副本集的pod正常运行数量。

kube-scheduler ：调度器，检测新建没有分配节点的pod，根据每个节点的资源选择一个节点，分配给新建未分配节点的pod。

以上Master主要组成部分。

以下是节点node的组成部分。

kubelet ：节点主要组成部分，管理分配到节点上的pod，自动注册到apiserver，定期将节点的状态发送给apiserver。同时接收并执行apiserver传达的命令。

kube-proxy ：自动发现服务，服务的端口转发。

ipvs：节点的负载均衡。

clusterDNS：DNS服务，插件的形式，为集群中的服务提供DNS解析。

