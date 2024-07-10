# 默认 VPC 使用

Kube-OVN 支持多租户隔离级别的 VPC 网络。不同 VPC 网络相互独立，可以分别配置 Subnet 网段，
路由策略，安全策略，出网网关，EIP 等配置。

> VPC 主要用于有多租户网络强隔离的场景，部分 Kubernetes 网络功能在多租户网络下存在冲突。
> 例如节点和 Pod 互访，NodePort 功能，基于网络访问的健康检查和 DNS 能力在多租户网络场景暂不支持。
> 为了方便常见 Kubernetes 的使用场景，Kube-OVN 默认 VPC 做了特殊设计，该 VPC 下的 Subnet
> 可以满足 Kubernetes 规范。用户自定义 VPC 支持本文档介绍的静态路由，EIP 和 NAT 网关等功能。
> 常见隔离需求可通过默认 VPC 下的网络策略和子网 ACL 实现，在使用自定义 VPC 前请明确是否需要
> VPC 级别的隔离，并了解自定义 VPC 下的限制。
> 在 Underlay 网络下，物理交换机负责数据面转发，VPC 无法对 Underlay 子网进行隔离。

![](../static/network-topology.png)

Kube-OVN 支持的 VPC 有两种：

- 默认 VPC（ovn-cluster）: Kube-OVN 一旦部署即创建，包括 ovn-default 和 join 两个 subnet，这两个子网用于实现 node 和 pod 互联。
- 自定义 VPC: 由用户管理。

只有默认 VPC 支持其内 pod 可以和 node 互通，因此，Pod 可以基于策略路由访问 node 能够访问的网络，比如 service，localdns，以及基于 node 默认路由访问公网。
只有默认 VPC 支持使用 OVN 提供的 eip 和 snat 功能，基于 Pod annotaion 配置。自定义 vpc 只能基于 ovn eip snat dnat 来使用该功能。
只有默认 VPC 支持使用集中式网关和分布式网关等功能，自定义 VPC 的网关实现和默认 VPC 的网关实现不一样。
默认 VPC 也可以使用 iptables nat gw pod，该功能需要用户自己配置路由将 VPC 流量转发到 nat gw pod，然后再转发到物理交换机网关。由于 VPC 策略路由比较多，一旦路由配置错误可能会影响其他功能。
