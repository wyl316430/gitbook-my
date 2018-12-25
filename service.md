service通过selector和pod建立关联。

k8s会根据service关联到pod的podIP信息组合成一个endpoint。

若service定义中没有selector字段，service被创建时，endpoint controller不会自动创建endpoint。



## service负载分发策略

service 负载分发策略有两种：

```
RoundRobin：轮询模式，即轮询将请求转发到后端的各个pod上（默认模式）；

SessionAffinity：基于客户端IP地址进行会话保持的模式，第一次客户端访问后端某个pod，之后的请求都转发到这个pod上。

```

## k8s服务发现方式

DNS：可以通过cluster add-on的方式轻松的创建KubeDNS来对集群内的Service进行服务发现————这也是k8s官方强烈推荐的方式。为了让Pod中的容器可以使用kube-dns来解析域名，k8s会修改容器的/etc/resolv.conf配置。

## k8s服务发现原理

#### endpoint

endpoint是k8s集群中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址。

service配置selector，endpoint controller才会自动创建对应的endpoint对象；否则，不会生成endpoint对象.

#### endpoint controller

endpoint controller是k8s集群控制器的其中一个组件，其功能如下：

负责生成和维护所有endpoint对象的控制器

负责监听service和对应pod的变化

监听到service被删除，则删除和该service同名的endpoint对象

监听到新的service被创建，则根据新建service信息获取相关pod列表，然后创建对应endpoint对象

监听到service被更新，则根据更新后的service信息获取相关pod列表，然后更新对应endpoint对象

监听到pod事件，则更新对应的service的endpoint对象，将podIp记录到endpoint中

# 负载均衡

## kube-proxy

kube-proxy负责service的实现，即实现了k8s内部从pod到service和外部从node port到service的访问。

kube-proxy采用iptables的方式配置负载均衡，基于iptables的kube-proxy的主要职责包括两大块：一块是侦听service更新事件，并更新service相关的iptables规则，一块是侦听endpoint更新事件，更新endpoint相关的iptables规则（如 KUBE-SVC-链中的规则），然后将包请求转入endpoint对应的Pod。如果某个service尚没有Pod创建，那么针对此service的请求将会被drop掉。

