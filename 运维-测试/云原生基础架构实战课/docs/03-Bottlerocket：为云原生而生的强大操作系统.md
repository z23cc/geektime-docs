你好，我是潘野。

在做云原生训练营助教的时候，很多同学对“不可变基础设施”这一个概念不太理解，如果你在网络上搜索“不可变基础设施”，往往会到得这样的说法：

> 不可变基础设施是指一种基于代码和配置的基础设施管理方法，其中所有的更改和配置都是通过重新构建和替换来实现的，而不是直接修改现有的设施。这种方法的目标是确保系统的稳定性和一致性，减少人为错误和配置漂移，并提高系统的可靠性和可管理性。

很多同学读了之后一头雾水，产生了后面这些疑问。

- 我用puppet或者ansible不也是代码方式来管理基础设施，这样不是也能保证一致性吗？
- 如果更改配置都是通过替换实例的方式，这样做的成本是不是太高了？
- 没看出“不可变”的好处在哪里？

今天这节课，我就带你重新理解一下“不可变基础设施”，看看它究竟带来了什么样的管理优势。最后，我还会带你一起体验下，不可变基础设施中的新星bottlerocket如何使用。

## 运维之痛：给操作系统打补丁

通过前面的学习，我们知道了在虚拟机时代，的确可以通过Puppet、Saltstack、Ansible这类配置管理工具来实现IaC。即便到今天，这些工具也仍然在运维管理工作中非常流行。

以往比较常见的运维工作就是给虚拟机打安全补丁，或者是升级主机内核，这时我们的做法基本都是更改Puppet的配置文件，配置推送到目标机器，并且联系业务团队安排时间做主机重启。用这种方式管理的机器，我们叫做**可变基础架构**。

如果公司里机器数量是几百、几千台，通过自己开发的运维平台，投入三四个人力，花上半个月基本也可以搞定。但是如果你公司里的机器数量在十万台以上，你会发现必须安排几个运维同学，常年给操作系统打补丁。

在虚拟机时代，一个机器上一般只跑一个应用，为了提高资源利用率，最多也就跑三四个应用，那么通知用户、给机器打补丁，这些事可能还属于可控范围内。然而现在的容器环境中，一个主机上动辄几十到上百个容器，并且掺杂了很多不同的应用。这时候工作变得不太可控，三四个人可能半年也做不完一轮维护。

近些年安全管理在各个公司里都逐渐上升到公司战略层面，安全管理中最常见的一个事情就是给操作系统打安全补丁；而且自从Docker出现之后，Linux操作系统的内核版本演进变得快了很多，新的内核中包含了很多为容器设计的特性，比如最近非常热门的eBPF，它要求Kernel的版本不能低于4.1，而真正比较完善的是4.18这个版本，所以Kernel版本的跟进也成为了基础架构管理中的一个强需求。

那么在云原生时代，如果我们还是依靠原来的管理方式，通过Puppet等配置管理工具滚动地升级内核版本、重启主机，这不仅非常耗费人力，而且还可能会造成一些应用的中断，并不是一个很优雅的管理方式。

既然云原生时代，容器这种部署稳定可靠且易管理的方式已经深入人心，我们能不能像管理容器一样来管理虚拟机，来简化运维操作呢？

## 什么是不可变基础设施？

想要像管理容器一样管理虚拟机，我们需要先分析一下容器有哪些特点。

我们第一反应大概会包括这几个特点：能快速启动、轻量级、有版本控制等等。

所以对应到虚拟机上，我们就会有这样的期望。

- 虚拟机启动要越快越好，所以镜像要最小化。
- 虚拟机镜像是有版本控制的，当虚拟机需要打补丁的时候，我们需要做的是，打包编译新的镜像，然后用新镜像启动机器做直接替换。
- 利用Kubernetes自身的能力，保证pod平滑地漂移到新的节点上即可。一切的管理都由Kubernetes来负责，不再需要人工介入管理。

我们期望的这种管理方式其实就是不可变基础设施！

有了感性认知以后，我们就可以重新拆解不可变基础设施的定义了，这里有三个关键点需要掌握。

**关键点一，一种基于代码和配置的基础设施管理方法。**

我们打包一个容器镜像的过程是，写Dockerfile，编译镜像，推送进容器仓库。一般这个过程都是由公司内的CI/CD工具来完成。其实虚拟机的镜像管理也是类似的做法，利用Packer这样的打包工具，配合CI工具来生成虚拟机镜像，然后推送到虚拟机镜像管理的仓库。

**关键点二，所有的更改和配置都是通过重新构建和替换来实现的。**

当v1版本的虚拟机需要更新到v2版本的虚拟机，我们只要先启动v2版本的虚拟机，然后将v1版本虚拟机中的workload转移到新机器即可。我估计，熟悉Kubernetes的同学立刻就可以想到有哪些方法可以转移workload。

**关键点三，确保系统的稳定性和一致性，减少人为错误和配置漂移。**

运维的同学应该深有体会，面对几万台机器的管理，人为推错了配置这种情况并不少。另外，不健壮的配置脚本也可能引起环境配置不一致。而实施不可变基础设施的最大作用，就是避免这类情况出现。

好，前面我们花了不少篇幅讲解不可变基础设施的概念，为了让你加深对它的认识，我们来看看历史上曾经出现的不可变基础设施的技术。

## CoreOS/Atomic - 出师未捷

我们都知道Kubernetes的核心组件是etcd。最早发布etcd的这家公司叫CoreOS，公司的名字就是自家的一个操作系统CoreOS。CoreOS为容器专门设计操作系统的鼻祖，可能你没用过甚至没听说过它，所以我简单做个介绍。

CoreOS是在2013年出现的，CoreOS比Kubernetes还早出来一年，CoreOS就是为“不可变基础设施”量身打造的操作系统，它与传统的CentOS/Ubuntu这类系统不同，具有以下特点。

- CoreOS利用主动和被动双分区方案来更新 OS，使用分区作为一个单元，而不是一个包一个包的更新。这使得每次更新变得快速、可靠，而且很容易回滚。
- CoreOS 内置了服务发现工具etcd，容器管理工具rkt。
- CoreOS专门为云环境设定，提供了AWS、GCP、Azure等各大云厂商镜像。

作为Linux操作系统的扛把子Redhat，看到CoreOS的设计方式之后大为震惊，基于RHEL的系统也推出了专为容器设计的操作系统Atomic，eBay的容器平台使用的就是Atomic。

2018年的时候，云原生热潮席卷全球，Red Hat干脆直接收购了CoreOS，并且将CoreOS与Red Hat原有的Atomic操作系统项目合并，一并改名叫Fedora CoreOS。

这两个操作系统的特点，非常符合前面所说的不可变基础设施的最佳实践，但是两个项目最终合并成为了商业产品。那么在容器时代，还有没有其他可以不花钱的替代产品了？

有，这就是Amazon专门推出的、为容器服务的操作系统Bottlerocket！

## Bottlerocket - 旗开得胜

Bottlerocket是Amazon在2020年3月时发布的一套完全开源的操作系统，由现有开源组件（例如Linux内核）、约50个软件包以及专为Bottlerocket编写的新组件（主要以Rust及Go语言编写而成）共同组成。你可以在AWS网站上找到它们的[介绍](https://aws.amazon.com/cn/bottlerocket/)。

你可能仍然有顾虑，前面提到CoreOS被收购了，Atomic项目就终止了，那么Bottlerocket有没有这样的风险呢？

从现在的情况来看，Bottlerocket中Amazon自己编写的组件使用Apache 2.0许可或MIT许可，Linux Kernel等其他开源组件遵守原始开源许可，所以风险不大。同时Bottlerocket项目源自长期以来在Amazon大规模生产运行服务当中总结出的经验教训，结合了过去多年以来Amazon对于容器技术的提炼与沉淀，可以说Bottlerocket是现在容器操作系统的绝佳选择。

Bottlerocket和传统操作系统相比，引入了一些新的概念，这里我着重讲一下Bottlerocket的三大特点。

**第一，Bottlerocket 没有软件包管理器，一切应用程序用容器运行。**

很多同学不太理解什么是“没有包管理器”。传统的操作系统，比如Ubuntu或者centOS，它们的包管理器是RPM和DEB，基于RHEL系统的Atomic就是用的RPM包管理的模式。然而，在Bottlerocket没有任何的包管理工具，Bottlerocket系统里的组件是直接以二进制形式打包进镜像的。这样做可以减少很多无用组件，达到操作系统最小化的目的。

**第二，Bottlerocket是一个API驱动的操作系统。**

“API驱动”，这听起来也比较难理解。简单来说就是当操作系统启动之后，系统没有ssh，也没有传统Linux上必备的shell。如果我们想调试系统，需要启动一个带有 shell 的容器来调试、手动更新和更改主机上的配置。那么启动这个shell的容器，就是调用Bottlerocket的API来启动这个shell，在后面的章节我会演示如何使用它的API，这里你先有个印象就好。

**第三，Bottlerocket根文件系统是只读，不可修改，这是实现“不可变基础设施”的一个重要特性。**而非根分区系统也就是用户存数据的分区，可以读写，采用SELinux策略来保护非根文件系统中的文件。

## 快速体验

说了这么多Bottlerocket的特点，我们怎么快速上手呢？

我猜有同学已经通过搜索点开了Bottlerocket的 [GitHub链接](https://github.com/bottlerocket-os)，但是可能会发现GitHub里一些指引看不明白，比如说，当我们接触一个新的操作系统，第一步肯定是在虚拟机里尝试安装，但是你在Bottlerocket的GitHub上找不到“如何安装Bottlerocket”这样的文档。

这是因为Bottlerocket是为容器而生，所以整个的设计是围绕Kubernetes的，所以它的使用离不开kubernetes集群。如果想体验Bottlerocket，最快捷的办法是在AWS上启动一个EKS集群，然后选择Bottlerocket作为节点的操作系统。

当然，如果你要想把它用在物理机器或者在本地环境里跑起来，Bottlerocket也可以提供了相应的文档指引，你可以自行查阅Github上的[这个链接](https://github.com/bottlerocket-os/bottlerocket/blob/develop/PROVISIONING-METAL.md)了解更多内容。在Bottlerocket的Github上，Bottlerocket不但为Kubernetes 1.23之后的每个主版本都提供了镜像，同时还提供了AWS AMI镜像和VMware的镜像。

### 调试实操环节

最后，我们来简单体验一下怎么进入Bottlerocket的调试模式，并且尝试一些重启机器之类的操作，找找手感。

首先，我们在EKS集群里添加一个节点组，其中AMI的类型是Bottlerocket\_x86\_64，你可以参考后面的截图来配置。

![](https://static001.geekbang.org/resource/image/89/eb/893ebeb9acf387f84e29a245fc27ddeb.jpg?wh=1179x737)

添加完成之后等待片刻，我们就可以在集群里看到一个新的节点产生。之后用-owide参数就可以看到这个节点的操作系统是Bottlerocket。

```go
~>kubectl get node ip-192-168-81-115.ap-east-1.compute.internal -owide
NAME                                           STATUS   ROLES    AGE     VERSION               INTERNAL-IP      EXTERNAL-IP     OS-IMAGE                                KERNEL-VERSION   CONTAINER-RUNTIME
ip-192-168-81-115.ap-east-1.compute.internal   Ready    <none>   3m47s   v1.27.7-eks-1670f88   192.168.81.115   18.163.118.76   Bottlerocket OS 1.16.1 (aws-k8s-1.27)   5.15.136         containerd://1.6.24+bottlerocket
```

接下来，让我们在AWS的console上找到这个EC2节点，使用session manager这个工具连接这个节点。然后你会看到Bottlerocket的欢迎提示，同时给你提示了三个命令。

第一个命令是 `apiclient` ，这是用来与Bottlerocket API交互的工具。

第二个命令是 `enter-admin-container` ，它的作用是启动一个具有admin权限的容器，我们接下来的操作都会在admin容器中进行。

第三个命令是 `disable-admin-container`，该命令用来关闭具备admin权限的容器。

![](https://static001.geekbang.org/resource/image/2c/70/2ceca67abdfc3ccf734173a9e6cea170.jpg?wh=1990x1748)

敲下 `enter-admin-container`命令之后，我们即可进入admin容器。然后敲下apiclient reboot命令，此时你会发现提示机器重启了，集群中这个节点变成了NotReady的状态。等机器启动完成之后，这个节点会重新变成Ready状态。

![](https://static001.geekbang.org/resource/image/20/38/205e4106d9e8e91783760ddb7a5c3838.jpg?wh=1990x901)![](https://static001.geekbang.org/resource/image/bf/3f/bf32e24027b827yy4f3b7d107681293f.jpg?wh=1990x386)

短短几分钟我们就体验了Bottlerocket的启动，API交互的方法。如果你有兴趣，可以进入admin容器，运行apiclient -h来仔细研究一下apiclient这个命令能做哪些事情。下面这张图展示了该命令支持的参数，供你参考。

![](https://static001.geekbang.org/resource/image/cd/10/cdc3e00811de655d96d945e67f8ee010.jpg?wh=1990x1188)

## 总结

不可变基础设施是云原生时代一种构建和管理基础设施的方法，这种方法提供了可靠性、可伸缩性和可重复性。而Bottlerocket是一个轻量级，高度可靠性和安全性的Linux发行版，是对不可变基础设施的最佳实践。

今天，我们学习了不可变基础设施的概念，其中不可变基础设施概念的三个关键点你需要重点掌握。之后，我们还通过实操体会到Bottlerocket的一些用法，还有其他的一些特点，同学们可以课后自行查阅Bottlerocket的[官方文档](https://bottlerocket.dev/)。建议你课后对照课程讲解，动手多多练习，加深记忆。

后面的课程中，我们启动的集群会默认使用Bottlerocket作为kubernetes nodes，相信随着课程学习的深入，你也会对Bottlerocket更加熟悉。

## 思考题

这一讲，我提到容器环境中重启主机这有可能会造成一些应用的中断。你知道什么样的情况会造成应用中断么？我们又该如何预防？

欢迎你和我在评论区一起讨论。如果这一讲对你有帮助，别忘了分享给身边更多同事、朋友。
<div><strong>精选留言（5）</strong></div><ul>
<li><span>暴躁的蜗牛</span> 👍（3） 💬（1）<p>bottlerocket 是可以替代 容器的 Alpine Linux吗 感觉虽然都是操作系统 是否某些应用会指定 操作系统的版本 这样对 系统bottlerocket  有影响吗</p>2024-04-20</li><br/><li><span>alex run</span> 👍（3） 💬（1）<p>老师，能谈一下私有云落地bottlerocket的做法吗？ 然后我想问一下，将传统操作系统替换成bottlerocket是有必要做的一件事情吗？ 回报会高于投入吗？</p>2024-04-13</li><br/><li><span>三万棵树</span> 👍（0） 💬（2）<p>Bottlerocket 这个在国内的落地情况 想咨询您一下</p>2024-03-29</li><br/><li><span>二十三</span> 👍（0） 💬（0）<p>国内公有云貌似都还没有支持</p>2024-04-19</li><br/><li><span>cfanbo</span> 👍（0） 💬（0）<p>Bottlerocket 这个在国内用的多吗？</p>2024-03-29</li><br/>
</ul>