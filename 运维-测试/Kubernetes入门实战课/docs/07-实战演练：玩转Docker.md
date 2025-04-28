你好，我是Chrono。

学到今天的这次课，我们的“入门篇”就算是告一段落了，有这些容器知识作为基础，很快我们就要正式开始学习Kubernetes。不过在那之前，来对前面的课程做一个回顾和实践，把基础再夯实一下。

要提醒你的是，Docker相关的内容很多很广，在入门篇中，我只从中挑选出了一些最基本最有用的介绍给你。而且在我看来，我们不需要完全了解Docker的所有功能，我也不建议你对Docker的内部架构细节和具体的命令行参数做过多的了解，太浪费精力，只要会用够用，需要的时候能够查找官方手册就行。

毕竟我们这门课程的目标是Kubernetes，而Docker只不过是众多容器运行时（Container Runtime）中最出名的一款而已。当然，如果你当前的工作是与Docker深度绑定，那就另当别论了。

好下面我先把容器技术做一个简要的总结，然后演示两个实战项目：使用Docker部署Registry和WordPress。

## 容器技术要点回顾

容器技术是后端应用领域的一项重大创新，它彻底变革了应用的开发、交付与部署方式，是“云原生”的根本（[01讲](https://time.geekbang.org/column/article/528619)）。

容器基于Linux底层的namespace、cgroup、chroot等功能，虽然它们很早就出现了，但直到Docker“横空出世”，把它们整合在一起，容器才真正走近了大众的视野，逐渐为广大开发者所熟知（[02讲](https://time.geekbang.org/column/article/528640)）。

容器技术中有三个核心概念：**容器（Container）**、**镜像（Image）**，以及**镜像仓库（Registry）**（[03讲](https://time.geekbang.org/column/article/528651)）。

![图片](https://static001.geekbang.org/resource/image/c8/fe/c8116066bdbf295a7c9fc25b87755dfe.jpg?wh=1920x1048)

从本质上来说，容器属于虚拟化技术的一种，和虚拟机（Virtual Machine）很类似，都能够分拆系统资源，隔离应用进程，但容器更加轻量级，运行效率更高，比虚拟机更适合云计算的需求。

镜像是容器的静态形式，它把应用程序连同依赖的操作系统、配置文件、环境变量等等都打包到了一起，因而能够在任何系统上运行，免除了很多部署运维和平台迁移的麻烦。

镜像内部由多个层（Layer）组成，每一层都是一组文件，多个层会使用Union FS技术合并成一个文件系统供容器使用。这种细粒度结构的好处是相同的层可以共享、复用，节约磁盘存储和网络传输的成本，也让构建镜像的工作变得更加容易（[04讲](https://time.geekbang.org/column/article/528660)）。

为了方便管理镜像，就出现了镜像仓库，它集中存放各种容器化的应用，用户可以任意上传下载，是分发镜像的最佳方式（[05讲](https://time.geekbang.org/column/article/528677)）。

目前最知名的公开镜像仓库是Docker Hub，其他的还有quay.io、gcr.io，我们可以在这些网站上找到许多高质量镜像，集成到我们自己的应用系统中。

容器技术有很多具体的实现，Docker是最初也是最流行的容器技术，它的主要形态是运行在Linux上的“Docker Engine”。我们日常使用的 `docker` 命令其实只是一个前端工具，它必须与后台服务“Docker daemon”通信才能实现各种功能。

操作容器的常用命令有 `docker ps`、`docker run`、`docker exec`、`docker stop` 等；操作镜像的常用命令有 `docker images`、`docker rmi`、`docker build`、`docker tag` 等；操作镜像仓库的常用命令有 `docker pull`、`docker push` 等。

好简单地回顾了容器技术，下面我们就来综合运用在“入门篇”所学到的各个知识点，开始实战演练，玩转Docker。

## 搭建私有镜像仓库

在第5节课讲Docker Hub的时候曾经说过，在离线环境里，我们可以自己搭建私有仓库。但因为镜像仓库是网络服务的形式，当时还没有学到容器网络相关的知识，所以只有到了现在，我们具备了比较完整的Docker知识体系，才能够搭建私有仓库。

私有镜像仓库有很多现成的解决方案，今天我只选择最简单的Docker Registry，而功能更完善的CNCF Harbor留到后续学习Kubernetes时再介绍。

你可以在Docker Hub网站上搜索“registry”，找到它的官方页面（[https://registry.hub.docker.com/\_/registry/](https://registry.hub.docker.com/_/registry/)）：

![图片](https://static001.geekbang.org/resource/image/cb/26/cbab37b3a4f2a621ab81564bcdba1326.png?wh=1408x712)

Docker Registry的网页上有很详细的说明，包括下载命令、用法等，我们可以完全照着它来操作。

首先，你需要使用 `docker pull` 命令拉取镜像：

```plain
docker pull registry
```

然后，我们需要做一个端口映射，对外暴露端口，这样Docker Registry才能提供服务。它的容器内端口是5000，简单起见，我们在外面也使用同样的5000端口，所以运行命令就是 `docker run -d -p 5000:5000 registry` ：

```plain
docker run -d -p 5000:5000 registry
```

启动Docker Registry之后，你可以使用 `docker ps` 查看它的运行状态，可以看到它确实把本机的5000端口映射到了容器内的5000端口。

![图片](https://static001.geekbang.org/resource/image/18/01/183e8953bb544c1dbf42022bfacef101.png?wh=1920x123)

接下来，我们就要使用 `docker tag` 命令给镜像打标签再上传了。因为上传的目标不是默认的Docker Hub，而是本地的私有仓库，所以镜像的名字前面还必须再加上仓库的地址（域名或者IP地址都行），形式上和HTTP的URL非常像。

比如在这里，我就把“nginx:alpine”改成了“127.0.0.1:5000/nginx:alpine”：

```plain
docker tag nginx:alpine 127.0.0.1:5000/nginx:alpine
```

现在，这个镜像有了一个附加仓库地址的完整名字，就可以用 `docker push` 推上去了：

```plain
docker push 127.0.0.1:5000/nginx:alpine
```

![图片](https://static001.geekbang.org/resource/image/c8/7f/c851b97a722de6fe2a144ec75c69e27f.png?wh=1920x444)

为了验证是否已经成功推送，我们可以把刚才打标签的镜像删掉，再重新下载：

```plain
docker rmi  127.0.0.1:5000/nginx:alpine
docker pull 127.0.0.1:5000/nginx:alpine
```

![图片](https://static001.geekbang.org/resource/image/54/6e/54a288df48531205236f024ecd91816e.png?wh=1920x272)

这里 `docker pull` 确实完成了镜像下载任务，不过因为原来的层原本就已经存在，所以不会有实际的下载动作，只会创建一个新的镜像标签。

Docker Registry虽然没有图形界面，但提供了RESTful API，也可以发送HTTP请求来查看仓库里的镜像，具体的端点信息可以参考官方文档（[https://docs.docker.com/registry/spec/api/](https://docs.docker.com/registry/spec/api/)），下面的这两条curl命令就分别获取了镜像列表和Nginx镜像的标签列表：

```plain
curl 127.1:5000/v2/_catalog
curl 127.1:5000/v2/nginx/tags/list
```

![图片](https://static001.geekbang.org/resource/image/c4/e8/c4715261ef7e131b53af31931c1ecee8.png?wh=1294x246)

可以看到，因为应用被封装到了镜像里，所以我们只用简单的一两条命令就完成了私有仓库的搭建工作，完全不需要复杂的软件安装、环境设置、调试测试等繁琐的操作，这在容器技术出现之前简直是不可想象的。

## 搭建WordPress网站

Docker Registry应用比较简单，只用单个容器就运行了一个完整的服务，下面我们再来搭建一个有点复杂的WordPress网站。

网站需要用到三个容器：WordPress、MariaDB、Nginx，它们都是非常流行的开源项目，在Docker Hub网站上有官方镜像，网页上的说明也很详细，所以具体的搜索过程我就略过了，直接使用 `docker pull` 拉取它们的镜像：

```plain
docker pull wordpress:5
docker pull mariadb:10
docker pull nginx:alpine
```

我画了一个简单的网络架构图，你可以直观感受一下它们之间的关系：

![图片](https://static001.geekbang.org/resource/image/59/ca/59dfbe961bcd233b83e1c1ec064e2eca.png?wh=1920x643)

这个系统可以说是比较典型的网站了。MariaDB作为后面的关系型数据库，端口号是3306；WordPress是中间的应用服务器，使用MariaDB来存储数据，它的端口是80；Nginx是前面的反向代理，它对外暴露80端口，然后把请求转发给WordPress。

我们先来运行MariaDB。根据说明文档，需要配置“MARIADB\_DATABASE”等几个环境变量，用 `--env` 参数来指定启动时的数据库、用户名和密码，这里我指定数据库是“db”，用户名是“wp”，密码是“123”，管理员密码（root password）也是“123”。

下面就是启动MariaDB的 `docker run` 命令：

```plain
docker run -d --rm \
    --env MARIADB_DATABASE=db \
    --env MARIADB_USER=wp \
    --env MARIADB_PASSWORD=123 \
    --env MARIADB_ROOT_PASSWORD=123 \
    mariadb:10
```

启动之后，我们还可以使用 `docker exec` 命令，执行数据库的客户端工具“mysql”，验证数据库是否正常运行：

```plain
docker exec -it 9ac mysql -u wp -p
```

输入刚才设定的用户名“wp”和密码“123”之后，我们就连接上了MariaDB，可以使用 `show databases;` 和 `show tables;` 等命令来查看数据库里的内容。当然，现在肯定是空的。

![图片](https://static001.geekbang.org/resource/image/ef/5e/ef28b4482ef243c215b09dae6889695e.png?wh=1920x1282)

因为Docker的bridge网络模式的默认网段是“172.17.0.0/16”，宿主机固定是“172.17.0.1”，而且IP地址是顺序分配的，所以如果之前没有其他容器在运行的话，MariaDB容器的IP地址应该就是“172.17.0.2”，这可以通过 `docker inspect` 命令来验证：

```plain
docker inspect 9ac |grep IPAddress
```

![图片](https://static001.geekbang.org/resource/image/f3/e6/f371e89b97a0410acb628ac2889427e6.png?wh=1224x240)

现在数据库服务已经正常，该运行应用服务器WordPress了，它也要用 `--env` 参数来指定一些环境变量才能连接到MariaDB，注意“WORDPRESS\_DB\_HOST”必须是MariaDB的IP地址，否则会无法连接数据库：

```plain
docker run -d --rm \
    --env WORDPRESS_DB_HOST=172.17.0.2 \
    --env WORDPRESS_DB_USER=wp \
    --env WORDPRESS_DB_PASSWORD=123 \
    --env WORDPRESS_DB_NAME=db \
    wordpress:5
```

WordPress容器在启动的时候并没有使用 `-p` 参数映射端口号，所以外界是不能直接访问的，我们需要在前面配一个Nginx反向代理，把请求转发给WordPress的80端口。

配置Nginx反向代理必须要知道WordPress的IP地址，同样可以用 `docker inspect` 命令查看，如果没有什么意外的话它应该是“172.17.0.3”，所以我们就能够写出如下的配置文件（Nginx的用法可参考其他资料，这里就不展开讲了）：

```plain
server {
  listen 80;
  default_type text/html;

  location / {
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_pass http://172.17.0.3;
  }
}
```

有了这个配置文件，最关键的一步就来了，我们需要用 `-p` 参数把本机的端口映射到Nginx容器内部的80端口，再用 `-v` 参数把配置文件挂载到Nginx的“conf.d”目录下。这样，Nginx就会使用刚才编写好的配置文件，在80端口上监听HTTP请求，再转发到WordPress应用：

```plain
docker run -d --rm \
    -p 80:80 \
    -v `pwd`/wp.conf:/etc/nginx/conf.d/default.conf \
    nginx:alpine
```

三个容器都启动之后，我们再用 `docker ps` 来看看它们的状态：

![图片](https://static001.geekbang.org/resource/image/35/15/353f0f3aed1d1b540312548c29cb6c15.png?wh=1920x198)

可以看到，WordPress和MariaDB虽然使用了80和3306端口，但被容器隔离，外界不可见，只有Nginx有端口映射，能够从外界的80端口收发数据，网络状态和我们的架构图是一致的。

现在整个系统就已经在容器环境里运行好了，我们来打开浏览器，输入本机的“127.0.0.1”或者是虚拟机的IP地址（我这里是“[http://192.168.10.208](http://192.168.10.208)”），就可以看到WordPress的界面：

![图片](https://static001.geekbang.org/resource/image/a6/31/a63084a8dd95d0034ba72dcb60613531.png?wh=1224x1156)

在创建基本的用户、初始化网站之后，我们可以再登录MariaDB，看看是否已经有了一些数据：

![图片](https://static001.geekbang.org/resource/image/a2/1e/a22dcabe805471e304a74c715e7fb51e.png?wh=1920x1749)

可以看到，WordPress已经在数据库里新建了很多的表，这就证明我们的容器化的WordPress网站搭建成功。

## 小结

好了，今天我们简单地回顾了一下容器技术，这里有一份思维导图，是对前面所有容器知识要点的总结，你可以对照着用来复习。

![图片](https://static001.geekbang.org/resource/image/79/16/79f8c75e018e0a82eff432786110ef16.jpg?wh=1920x2142)

我们还使用Docker实际搭建了两个服务：Registry镜像仓库和WordPress网站。

通过这两个项目的实战演练，你应该能够感受到容器化对后端开发带来的巨大改变，它简化了应用的打包、分发和部署，简单的几条命令就可以完成之前需要编写大量脚本才能完成的任务，对于开发、运维来绝对是一个“福音”。

不过，在感受容器便利的同时，你有没有注意到它还是存在一些遗憾呢？比如说：

- 我们还是要手动运行一些命令来启动应用，然后再人工确认运行状态。
- 运行多个容器组成的应用比较麻烦，需要人工干预（如检查IP地址）才能维护网络通信。
- 现有的网络模式功能只适合单机，多台服务器上运行应用、负载均衡该怎么做？
- 如果要增加应用数量该怎么办？这时容器技术完全帮不上忙。

其实，如果我们仔细整理这些运行容器的 `docker run` 命令，写成脚本，再加上一些Shell、Python编程来实现自动化，也许就能够得到一个勉强可用的解决方案。

这个方案已经超越了容器技术本身，是在更高的层次上规划容器的运行次序、网络连接、数据持久化等应用要素，也就是现在我们常说的“**容器编排**”（Container Orchestration）的雏形，也正是后面要学习的Kubernetes的主要出发点。

## 课下作业

最后是课下作业时间，给你留两个思考题：

1. 学完了“入门篇”，和刚开始相比，你对容器技术有了哪些更深入的思考和理解？
2. 你觉得容器编排应该解决哪些方面的问题？

欢迎积极留言讨论，如果有收获，也欢迎你转发给身边的朋友一起学习。

下节课是视频课，我会用视频直观演示我们前面学过的操作，我们下节课见。  
![](https://static001.geekbang.org/resource/image/43/fa/43ccbfc6b629eed231ffec8d6eaf99fa.jpg?wh=1920x2051)
<div><strong>精选留言（15）</strong></div><ul>
<li><span>lesserror</span> 👍（17） 💬（3）<p>之前对docker的了解很杂乱，知识点很细碎、分散，没有一个整体、清晰的认知。

看过中文互联网上面别人的一些教程，要么照本宣科，要么浅尝辄止。

老师的课程虽然没有做到知识点的面面俱到，当然也不可能做到。但是，算是整体上帮我又重新梳理了一遍docker的整体架构，让我对其认识更加清晰了一些。</p>2022-07-06</li><br/><li><span>pyhhou</span> 👍（10） 💬（1）<p>思考题：
1. 相较于之前只知道容器是用来环境隔离，看完入门篇后，对容器技术有了一个比较宏观和基本的了解，列出来如下：
 1）知道了什么是镜像，以及镜像和容器的关系
 2）知道了 DockerHub 这样的镜像仓库
 3）明白了容器和虚拟机的不同
 4）懂得如何通过 Dockerfile 来构建自己的镜像
 5）理解了 Docker 的整体内部框架 docker client -&gt; docker daemon -&gt; registry
 6) 知道了，也实际操作了一些常用的镜像以及容器相关的指令
。。。

感觉学习到的这些东西可以覆盖工作中大多数的场景了，但是这些知识只能说是运用于小规模的东西。想要把容器技术玩的得心应手，还需了解一些容器应用的最佳实践，和一些工程化的理念和工具

2. 感觉容器编排主要应用于大规模集成应用。可以类比分布式系统，入门篇中讲的知识用在单机应用上是没有问题的，但是规模一旦变大到系统层面，就会出现一些问题，比如如何保证数据一致性？如何保证负载均衡？如何尽可能减少网络故障所带来的影响？如何能保证数据（容器）的持久化等等。。。这些问题需要运用容器编排来解决


另外想请教老师 2 个问题

文章一开始提到容器运行时（Container Runtime）这个概念，该如何理解？这是和容器绑定的一门技术吗？

还有就是，我看你在 curl 指令中直接将本地 IP 127.0.0.1 简写成 127.1，是说 curl 中允许这样的简写，还是说这本身就是一个惯例？

谢谢老师 🙏</p>2022-07-10</li><br/><li><span>朱雯</span> 👍（6） 💬（1）<p>

q1: 容器编排技术是有价值的，我之前以为价值不大，只是改变启动和使用方式，增加一些命令。
q2： 容器编排解决的问题是：一些非自动化，而是需要强人工干预的东西，比如网络交互需要知道对方ip地址的情况，虽然可以写自动化脚本，但这个并不通用，所以是一套通用的自动化方案。另外多台机器，自动创建负载均衡，创建路由的配置问题。这些是编排的范围。
</p>2022-07-06</li><br/><li><span>mj4ever</span> 👍（5） 💬（1）<p>老师的教程中，ng → wp → db，相互之间是通过容器的 IP 地址来访问，尝试以下两种方法，可以不指定 IP 地址，通过容器名：

1、启动容器时加入了自定义的网络 my_network，类型是 bridge；其原理是容器之间的互联是通过 Docker DNS Server；代码如下
docker run -d --rm --name db1 \
    --network my_network \
    --env MARIADB_DATABASE=db \
    --env MARIADB_USER=wp \
    --env MARIADB_PASSWORD=123 \
    --env MARIADB_ROOT_PASSWORD=123 \
    mariadb:10

docker run -d --rm --name wp1 \
    --network my_network \
    --env WORDPRESS_DB_HOST=db1 \
    --env WORDPRESS_DB_USER=wp \
    --env WORDPRESS_DB_PASSWORD=123 \
    --env WORDPRESS_DB_NAME=db \
    wordpress:5

vi wp.conf
server {
  listen 80;
  default_type text&#47;html;

  location &#47; {
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_pass http:&#47;&#47;wp1;
  }
}

docker run -d --rm --name ng1 \
    --network my_network \
    -p 80:80 \
    -v `pwd`&#47;wp.conf:&#47;etc&#47;nginx&#47;conf.d&#47;default.conf \
    nginx:alpine

2、启动 WordPress wp1 时，link 到 db1，即--link db1:db1，  启动 Nginx ng1 时，link 到 wp1，即--link wp1:wp1；其原理是容器之间的互联是通过容器里的 &#47;etc&#47;hosts；代码如下
docker run -d --rm --name db1 \
    --env MARIADB_DATABASE=db \
    --env MARIADB_USER=wp \
    --env MARIADB_PASSWORD=123 \
    --env MARIADB_ROOT_PASSWORD=123 \
    mariadb:10

docker run -d --rm --name wp1 \
    --link db1:db1 \
    --env WORDPRESS_DB_HOST=db1 \
    --env WORDPRESS_DB_USER=wp \
    --env WORDPRESS_DB_PASSWORD=123 \
    --env WORDPRESS_DB_NAME=db \
    wordpress:5

vi wp.conf
server {
  listen 80;
  default_type text&#47;html;

  location &#47; {
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_pass http:&#47;&#47;wp1;
  }
}

docker run -d --rm --name ng1 \
    --link wp1:wp1 \
    -p 80:80 \
    -v `pwd`&#47;wp.conf:&#47;etc&#47;nginx&#47;conf.d&#47;default.conf \
    nginx:alpine</p>2022-09-17</li><br/><li><span>柳成荫</span> 👍（5） 💬（1）<p>1. 刚开始学容器的时候觉得容器就是一个小的虚拟机，部署一套应用应该可以把中间件和应用都部署到同一个容器中，每个容器都应该对外暴露端口才能被访问，现在觉得有些应用可以不用暴露端口，反而更加安全
2. 容器编排应该会解决容器启动、维护的麻烦，应用集群等问题
请教一个问题，部署一个java应用，jdk应该安装在宿主机还是应用的容器里面呢？</p>2022-07-06</li><br/><li><span>henry</span> 👍（4） 💬（1）<p>2022&#47;08&#47;16，docker pull mariadb:10，会有问题，docker run 时报错：[ERROR] [Entrypoint]: mariadbd failed while attempting to check config，Can&#39;t initialize timers.

docker pull mariadb:10.8.2  解决问题，参考如下：
https:&#47;&#47;github.com&#47;MariaDB&#47;mariadb-docker&#47;issues&#47;434</p>2022-08-16</li><br/><li><span>A-Bot</span> 👍（4） 💬（2）<p>
docker run -d --rm \
    -p 80:80 \
    -v `pwd`&#47;wp.conf:&#47;etc&#47;nginx&#47;conf.d&#47;default.conf \
    nginx:alpine
老师，这个命令中 -v 后面跟的 &#39;pwd&#39; 什么意思？</p>2022-07-25</li><br/><li><span>lesserror</span> 👍（4） 💬（1）<p>老师，有几个小问题：

Q1：k8s应该算是容易编排技术吧？如果学会了k8s的日常操作，关于docker的使用是不是就可以减少了。了解一个大概就好了，很多操作应该逐渐偏向对k8s的操作？

Q2: 对于容器化的应用来说，如果想从外部访问对应的服务，是不是必须要做端口映射这一步？宿主机的端口需要唯一性，容器应用的端口随意指定，即使多个容器应用有相同的端口。</p>2022-07-06</li><br/><li><span>蔡晓慧</span> 👍（2） 💬（1）<p>1.之前没有容器技术的时候，部署应用各种环境会存在各种的问题，尤其是我司有C++的项目，需要安装依赖，光调试环境就搞得很头疼，现在有了容器技术，打包成镜像，随处可用，很方便；
2.感觉容器编排就是为了大型应用服务的。我们目前有十几个镜像，是用docker-compose用来做项目交付，应对一般场景足够用。但很多公司要求HA，这时候感觉上k8s感觉好一点，可扩展性好，快速扩容，我们也在往这个方向发展，所以自己有空来学习学习。</p>2023-03-02</li><br/><li><span>张申傲</span> 👍（2） 💬（1）<p>从头到尾跟到了现在，再加上本节课的实战，深切地感受到了容器化对于开发和运维方式的重塑。老师的课程深入浅出，受益匪浅~</p>2022-09-13</li><br/><li><span>peter</span> 👍（2） 💬（1）<p>请教老师几个问题：
Q1：镜像的“层”复用的问题：
--- 下载两个镜像A和B，这两个镜像都有一个层“M”，那么，这个层“M”在两个镜像中各存在一份，另外，docker会将此层“M”在宿主机上单独存一份，即在宿主机上，层“M”会存在三份，是这样吗？
--- 镜像A和B运行的时候，A的容器中有层“M”，B的容器中也有层“M”，是吗？
--- 镜像A在同一台宿主机上可以运行多个容器吗？如果可以，比如运行3个容器，那么，每个容器都有层“M”，对吗？
Q2：用rmi删除镜像后，镜像不存在了，但其包含的层还存在宿主机上，对吗？
这个问题和Q1有点关联，比如下载镜像A，其中含有层“M”，用rmi删除镜像后，镜像不存在了，其包含的层“M”不存在了，但宿主机上其实还有一份层“M”，对吗？
Q3：wordpress例子中，为什么nginx可以访问WP？
Wp没有对外暴露端口，而nginx对于WP来说就是外部访问者啊，应该不能访问才对啊。
Q4：小贴士的第一项中，挂载用法问题： 
 --- 挂载方法：  -v   &#47;home&#47;zhangsan   &#47;var&#47;lib&#47;registry， 其中&#47;home&#47;zhangsan是宿主机上的目录，是这样用吗？  
--- 挂载后，是把镜像本身放到&#47;home&#47;zhangsan下面吗？ 还是说，镜像不放在&#47;home&#47;zhangsan下面，但会把镜像用到的数据放到&#47;home&#47;zhangsan下面？</p>2022-07-06</li><br/><li><span>可可</span> 👍（1） 💬（1）<p>当我对wp.conf文件做了修改之后，执行nginx -t成功，但执行nginx -s reload却提示nginx 29#29: signal process started，发现修改并未生效。请问老师和其他同学遇到过这种情况吗？
我的解决办法是只能删除nginx容器后重新创建，这时候wp.conf就是生效的。但总不可能每次修改配置文件都重新创建nginx容器吧，寻求答案中……</p>2022-10-19</li><br/><li><span>aoe</span> 👍（1） 💬（1）<p>小贴士总能带来惊喜</p>2022-07-13</li><br/><li><span>Geek_b537b2</span> 👍（1） 💬（1）<p>老师请问下使用Docker Registry搭建本地镜像仓库后用docker pull拉取镜像怎么不是去共有仓库拉取而是默认去本地私有仓库拉取，这中间是不是自动配置的镜像源地址</p>2022-07-10</li><br/><li><span>Geek_18dfaf</span> 👍（1） 💬（1）<p>什么时候更新下一课</p>2022-07-06</li><br/>
</ul>