你好，我是倪朋飞。

上一讲我带你回顾了 eBPF 在 2022 年的旅程。过去一年，eBPF 不仅在 Linux 内核中新增了很多功能特性，其生态和实践也取得了突飞猛进的发展。特别是以 Cilium 为代表的云原生网络，自加入 CNCF 以来，不仅得到了主流公有云和广泛合作伙伴的支持，更是推动了 eBPF 在内核中的快速发展。想要把握 eBPF 最前沿的技术发展和实践应用，了解和学习 Cilium 自然是必不可少的一步。

那么，Cilium 到底是如何取得成功的？它又是如何驾驭 eBPF 并成功重构云原生网络的呢？今天，我就带你一起去揭开 Cilium 的神秘面纱。

## Cilium 概述

Cilium 是由 CNCF 托管的开源云原生网络解决方案，专注于为云原生应用带来可观测、高性能和卓越安全性的网络连接与管理能力。Cilium 利用 Linux 内核中的 eBPF 技术，动态地将可观测、高性能和强大安全性的控制逻辑嵌入内核，从而实现了高性能网络插件、多集群与多云管理、高效负载均衡、透明加密、广泛的网络安全功能，以及透明观测等诸多优秀特性。凭借 eBPF 在内核中完全透明运行的优势，Cilium 为用户提供的各种网络和安全功能不需要对应用程序进行任何更改，同时也无需在应用程序和网络之间添加任何代理。

Cilium 的起源可以追溯到 2014 年诞生的 eBPF。当时为了研究新的软件定义网络方案，Alexei Starovoitov 为 Linux 内核中的 BPF 带来了革命性的更新，将 BPF 扩展为一个通用的虚拟机，也就是 eBPF。eBPF 不仅扩展了寄存器的数量，引入了全新的 BPF 映射存储，还在 3.15 到 4.x 内核版本的演进过程中将原本单一的数据包过滤事件逐步扩展到了内核态函数、用户态函数、跟踪点、性能事件以及安全控制等。

eBPF 的出现引起了开源社区和企业的广泛关注，他们认识到 eBPF 可以为云原生应用提供更高效、更灵活、更安全的网络解决方案，其中就包括 Cilium 的创始人 Thomas Graf 和 Dan Wendlandt。于是，他们在 2015 年创建并开源了 Cilium 项目，正式将 eBPF 带入到云原生网络领域；并于 2017 年成立了 Isovalent 公司，开启了 Cilium 的企业化产品之路。

Cilium 刚诞生的几年里，它的发展并不顺利，碰到了非常多的挑战。

- 一方面，eBPF 的采纳需要非常高的技术门槛。不仅 Linux 内核需要采用最新的版本，eBPF 自身的功能还不够完善和稳定。而且内核中的 eBPF 也没有统一的接口，导致 Cilium 无法在不同的内核版本中实现一致的功能。
- 另一方面，Cilium 与 Kubernetes 的集成也不够完善，与 Calico、Flannel 等其他网络方案相比，无论在功能特性上，还是在产品接纳度上都有明显的差距。

尽管面临诸多挑战，但凭借开源社区的力量以及在 Linux 内核和 CNCF 生态的突出贡献，Cilium 逐步克服了这些困难，并在 2018 年成功发布首个正式版本。从此，Cilium 项目开始走上了快速发展的道路，不断地推出新的功能和特性，并同时推进 eBPF 在内核中的快速发展。特别是 2021 年加入 CNCF，**标志着 Cilium 正式开启了其云原生网络的大规模应用之路**。

时至今日，Cilium 项目已吸引了超过 500 位贡献者，并得到了众多主流云原生厂商和开源社区的广泛支持。如下图所示，Cilium 的生态已覆盖了从网络、安全到可观测性等各个领域。Isovalent，作为 Cilium 项目背后的公司，也已成为 eBPF 技术领域最重要的贡献者和推广者之一，借助 eBPF 基金会引领着 eBPF 的技术方向和发展趋势。

![图片](https://static001.geekbang.org/resource/image/68/23/68cf2f3a9a44a9fa2eb082bbd3597123.png?wh=5396x2772 "图片来自 Cilium Github [br]https://github.com/cilium")

说了这么多，你一定好奇 Cilium 是如何做到这一切的呢？接下来，我就带你一起去看看 Cilium 的工作原理。

## Cilium 工作原理

如下图所示，展示了 Cilium 的整体架构，它主要由四个关键组件构成：核心模块 Cilium、可观测性模块 Hubble、内核层 eBPF 程序模块以及存储模块。

![图片](https://static001.geekbang.org/resource/image/14/46/144dd316964fd77b610d1913958d1d46.png?wh=1619x1443 "图片来自 Cilium 官方文档 [br]https://docs.cilium.io/en/stable/overview/component-overview/")

具体来说：

- 核心模块 Cilium 是整个架构的基石，由在集群节点和服务器上运行的代理（Cilium Agent）、控制平面（Cilium Operator）、命令行管理界面（Cilium CLI）以及网络插件（Cilium CNI）等部分组成。
- 可观测性模块 Hubble 是一个分布式网络和安全观测平台，包括 Web 界面（Hubble UI）、数据采集服务（Hubble Server）、数据聚合服务（Hubble Relay）以及命令行管理界面（Hubble CLI）等组件。
- 内核层 eBPF 程序模块是 Cilium 的数据平面核心，通过利用内核中的各种挂载点和 eBPF 运行时，确保 Cilium 数据平面的高性能和高安全性。
- 存储模块负责存储集群状态，并在不同节点之间进行同步，支持 Kubernetes CRD 和 etcd。

今天我们就着重来看看与我们专栏直接相关的 eBPF 模块。在前面的课程中，我们已经讲到 eBPF 的工作原理以及各种类型 eBPF 程序的挂载点和使用方法。那么，Cilium 是如何利用 eBPF 来实现其网络功能的呢？接下来，我们就来看看 Cilium 的 eBPF 数据面。

## eBPF 数据面

Cilium 的 eBPF 程序按照其挂载位置和程序类型的不同，可以分为四类。

- 第一类，XDP 程序。XDP 程序位于网络驱动的最早阶段，负责在接收到数据包时触发 eBPF 程序的执行。由于 XDP 程序的执行速度极快，Cilium 利用它实现了高性能负载均衡，从而加速 LoadBalancer 和 NodePort 类型的服务。
- 第二类，TC 程序。TC 程序在内核协议栈初始化之后、L3 协议处理之前执行，支持接收和发送两个方向。因此，Cilium 使用 TC 程序来实现网络策略校验、连接跟踪、NAT 转换等功能。
- 第三类，套接字操作程序。套接字操作程序挂载在 cgroup 上并在 TCP 事件上运行。Cilium 将套接字操作程序挂载到根 cgroup，用以监控 TCP 状态转换和套接字信息维护等任务。
- 第四类，套接字发送/接收程序。套接字发送/接收程序挂载在 TCP 套接字上，并在发送 TCP 请求时运行。Cilium 利用这类程序加速本地数据路径的重定向。

在这几类程序的基础上，Cilium 构建了功能丰富的云原生网络特性，并与 Kubernetes 集成，实现了云原生网络的全生命周期管理。

接下来，我们将以两个不同的 Pod 之间的通信为例，详细介绍 Cilium 的数据平面路径。

如下图所示**，当两个 Pod 位于同一节点上**时，数据包的流程如下：

![图片](https://static001.geekbang.org/resource/image/e2/03/e2087460d2b32775b9a46afd33441b03.png?wh=1677x1271 "图片来自 Cilium 官方文档[br]https://docs.cilium.io/en/stable/network/ebpf/lifeofapacket/")

- 在 Pod 将数据包发送至套接字之后，套接字操作程序（bpf\_sockops）将对其进行拦截。当检测到目标地址位于同一节点时，它会直接通过重定向（bpf\_redir）跳过内核协议栈，将数据包发送到目标 Pod 的套接字中，进而传递给目标进程。
- 如果发送数据包的 Pod 设置了七层出口策略，数据包将首先转发至用户态代理（即 Envoy）以进行出口安全策略验证，然后再重定向至目标 Pod。
- 如果目标 Pod 同时配置了七层入口策略，数据包将再被转发至用户态代理（即 Envoy）进行入口安全策略验证，然后再重定向到目标 Pod。

从这个过程中我们可以看出，套接字操作程序（Sockops）使得 Cilium 能够绕过主机内核协议栈，从而实现 Pod 之间的直接通信。

然而，当两个 Pod 位于不同节点上时，它们的处理路径变得相对复杂。为了便于阐述，以下内容将分为源 Pod 出口路径和目标 Pod 入口路径两部分进行介绍。

首先，**源 Pod 出口路径**如下图所示：

![图片](https://static001.geekbang.org/resource/image/51/c9/51b3e1f28f9de999c2e1cd2d364df4c9.png?wh=1677x940 "图片来自 Cilium 官方文档[br]https://docs.cilium.io/en/stable/network/ebpf/lifeofapacket/")

从图中可以看到：

- Pod 发送数据包到套接字之后，套接字操作程序（sockops）对其目标地址进行检查，当检测到目的地址不在同一节点后，数据包继续发送到容器虚拟网卡对端设备。
- 挂载到容器虚拟网卡对端设备的 TC 程序（bpf\_lxc）进行网络策略校验、连接跟踪、DNAT 等处理，然后发送到主机 TC 程序（bpf\_host）进行南北向数据包处理。
- 如果发送数据包的 Pod 配置了七层出口策略，那么数据包还会被先转发到用户态代理（即 Envoy）进行安全策略验证，之后再发送到主机 TC 程序（bpf\_host）进行南北向数据包处理。
- 之后数据包进入内核协议栈进行路由选择后，经主机网卡发送到目的节点。

第二部分涉及网络数据包在**到达目标主机网卡后的入口路径**，如下图所示：

![](https://static001.geekbang.org/resource/image/47/dd/47fa1020d2624eba021b13673aabf7dd.png?wh=1688x821 "图片来自 Cilium 官方文档[br]https://docs.cilium.io/en/stable/network/ebpf/lifeofapacket/")

- 数据包进入目标主机网卡后，主机 cilium\_host TC 程序（bpf\_host/bpf\_overlay）进行南北向数据包处理（如果目的地址是 LoadBalancer 或 NodePort 类型 Service 地址，则先会进入主机网卡的 XDP 负载均衡处理程序）。
- 接着，cilium\_host TC 程序找到目的 Pod，将数据包重定向到它的虚拟网卡对端设备。
- 虚拟网卡对端设备挂载的 TC 程序（bpf\_lxc）会进一步对入口流量进行处理。
- 最后数据包经过虚拟网卡被发送到目的 Pod 内部的虚拟网卡，最终到达容器进程。

针对入口路径和出口路径，如果配置了加密策略，则在相应的 TC 程序前后还需进行加解密处理（bpf\_network）。如果配置了七层策略，数据包会先转发至用户态代理（即 Envoy），在通过安全策略验证后，才会发送至目标 Pod 的虚拟网卡对端设备。

正如我们课程之前提到的，内核态的 eBPF 程序与用户态进程进行交互需要通过 BPF 映射。Cilium Agent 与各种 eBPF 程序正是通过 BPF 映射存储了连接跟踪、NAT 转换、IP 地址、临接表等各种信息。从这些入口路径和出口路径的处理流程你可以看出，Cilium 使用 eBPF 简化了 Linux 内核原有的连接跟踪、网络安全策略、路由转发、负载均衡等功能，从而显著提升了网络处理的性能。作为一个开源项目，所有这些 eBPF 程序也都是开源的，可以在 [Cilium GitHub 仓库](https://github.com/cilium/cilium/tree/master/bpf)中找到，它们为学习 eBPF 提供了非常好的参考素材。

## 小结

在今天的课程中，我们共同回顾了 Cilium 的发展历程并探讨了其工作原理。作为 eBPF 在云原生网络领域应用的典范，Cilium 的核心理念在于用高效的 eBPF 程序替换 Linux 内核网络协议栈中的连接跟踪、NAT 转换、安全策略和路由转发等功能。这样一来，网络性能得以显著提升，同时还无需对现有应用程序进行任何修改。

值得赞誉的是，Cilium 项目不仅在云原生网络领域取得了成功，同时也衍生出一系列不断推动 eBPF 发展和应用的开源项目。例如，[cilium/ebpf](https://github.com/cilium/ebpf) 是使用 Go 语言开发和管理 eBPF 程序的优秀库，[pwru](https://github.com/cilium/pwru) 可以用于追踪 Linux 内核中的网络数据包处理流程，而 [tetragon](https://github.com/cilium/tetragon) 是一个专注于安全监测和策略执行的项目。

今天的这一讲就到这里了。我将继续关注 eBPF 的发展，并为你带来更多与 eBPF 相关的内容。预计下次的更新将于 6 月份推出。如你对我们未来课程内容有任何建议，请在评论区留言，期待你与我们共同完善和构建一个贴近实践的 eBPF 知识体系。

## 思考题

在这一讲的最后，我想邀请你来聊一聊：在最近一段时间学习和应用 eBPF 的过程中，你是否尝试过使用 Cilium 或其他类似的 eBPF 网络解决方案？如果有的话，它们协助你解决了哪些实际问题？相较于传统的解决方案，这些基于 eBPF 的方案具备哪些优点和不足之处？

欢迎在留言区和我讨论，也欢迎把这节课分享给你的同事、朋友。让我们一起在实战中演练，在交流中进步。
<div><strong>精选留言（2）</strong></div><ul>
<li><span>hudy_coco</span> 👍（1） 💬（1）<p>想知道的是，cilium有没有用ebpf实现进程流量限速、链接跟踪、防火墙与进程关联呢</p>2023-05-11</li><br/><li><span>mxmkeep</span> 👍（0） 💬（0）<p>希望下期老师能讲讲katran~</p>2023-05-05</li><br/>
</ul>