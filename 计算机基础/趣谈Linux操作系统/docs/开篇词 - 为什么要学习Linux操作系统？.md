你好，我是你的老朋友刘超。在“[趣谈网络协议](https://time.geekbang.org/column/intro/85?utm_term=zeusMX7NJ&utm_source=app&utm_medium=geektime&utm_campaign=85-end&utm_content=caozuoxitongkaipianci)”结课半年之后，我又给你带来了一个新的基础课程，“趣谈Linux操作系统”。

在“趣谈网络协议”的开篇词中，我表达了作为一个合格的IT工程师，在职业生涯中学习基础知识的重要性。如果说当时，我对这件事只是一种感性认识，在专栏推出之后，我的想法有了一些变化。

我通过留言区和同学们进行了很多互动，也和其他做基础知识专栏的作者有了不少交流，我发现，**无论是从个人的职业发展角度，还是从公司招聘候选人的角度来看，扎实的基础知识是很多人的诉求**。这让我更加坚信，我应该在“趣谈”基础知识这条道路上走下去。

目前极客时间的专栏，覆盖了网络、算法、数学、数据库、编程语言等各个方面，而操作系统也是基础中非常重要的一环。尤其我作为一名云架构师，Linux操作系统的基础知识更是必不可少的。在实践中收获了很多心得之后，我希望在极客时间继续跟你分享。

你可能会说，**我们大学里上过操作系统的课，而且每天都在用操作系统，为什么还要专门学一遍呢**？尽管我的操作系统课成绩不错，但是在大学的时候，我和你的看法一样，我觉得这门课没有什么用，现在回想起来可能有这样几个原因。

第一，大学里普遍使用的操作系统是Windows，老师大多也用Windows。Windows的优势是界面友好，很容易上手，于是我们就养成了要配置东西了就去菜单找，用鼠标点点的习惯，似乎会攒电脑、装系统、配软件就能搞定一切问题。

第二，一种操作系统对应的是一系列的软件生态，而大学里很多课程都是围绕Windows软件生态展开的。例如学C++用的是Vistual Studio，学数据库用的是SQL Server，做网站用的是IIS等等。

第三，大学里的操作系统课往往都是纯讲理论，讲了很多原理，但是压根儿没法和平时用的Windows系统的行为关联起来，也根本弄不清操作系统在底层到底是怎么做的。

直到毕业之后，我加入EMC，第一个项目就是基于Linux开发分布式文件系统。你能想象，只能对着一个黑框敲命令时，我心中的崩溃吗？我那时真的觉得，我大学的操作系统算是白学了。于是，我痛定思痛，开启了学习Linux的征程。

一旦开始学，我发现，Linux对于编程世界来说，简直就像一扇门。尽管门里的知识浩如烟海，每一本书都厚如砖头，但我发现这条路上任何一片景色都精彩无比。

## 打开Linux操作系统这扇门，你才是合格的软件工程师

根据2018年W3Techs的数据统计，对于服务器端，Unix-Like OS占的比例近70%，其中Linux可以称得上是中流砥柱。随着移动互联网的发展，客户端基本上以Android和iOS为主。Android是基于Linux内核的，因而客户端也进入了Linux阵营。可以说，**在编程世界中，Linux就是主流，不会Linux你就会格格不入。**

那些火得不行的技术，什么云计算、虚拟化、容器、大数据、人工智能，几乎都是基于Linux技术的。那些牛得不行的系统，团购、电商、打车、快递，都是部署在服务端，也几乎都是基于Linux技术的。

所以说，如果你想进大公司，想学新技术，Linux一定是一道绕不过去的坎。只有进入Linux操作系统这扇门，你才能成为合格的软件工程师。

## 研究Linux内核代码，你能学到数据结构与设计模式的落地实践

Linux最大的优点就是开源。作为程序员，有了代码，啥都好办了。只要有足够的耐心，我们就可以一层一层看下去，看内核调度函数，看内存分配过程。理论理解起来不容易，但是一行行的“if-else”却不会产生歧义。

在Linux内核里，你会看到数据结构和算法的经典使用案例；你甚至还会看到并发情况下的保护这种复杂场景；在实践中遇到问题的时候，你可以直接参考内核中的实现。

例如，平时看起来最简单的文件操作，通过阅读Linux代码，你能学到从应用层、系统调用层、进程文件操作抽象层、虚拟文件系统层、具体文件系统层、缓存层、设备I/O层的完美分层机制，尤其是虚拟文件系统对于接入多种类型文件系统的抽象设计，在很多复杂的系统里面，这个思想都能用得上。

再如，当你写代码的时候，大部分情况下都可以使用现成的数据结构和算法库，但是有些场景对于内存的使用需要限制到很小，对于搜索的时间需要限制到很小的时候，我们需要定制化一些数据结构，这个时候内核里面这些实现就很有参考意义了。

## 了解Linux操作系统生态，能让你事半功倍地学会新技术

Linux是一个生态，里面丰富多彩。很多大牛都是基于Linux来开发各种各样的软件。可以这么说，只要你能想象到的技术领域，几乎都能在里面找到Linux的身影。

数据库MySQL、PostgreSQL，消息队列RabbitMQ、Kafka，大数据Hadoop、Spark，虚拟化KVM、Openvswitch，容器Kubernetes、Docker，这些软件都会默认提供Linux下的安装、使用、运维手册，都会默认先适配Linux。

因此，在Linux环境下，很容易能够找到现成的工具，这不仅会让你的工作事半功倍，还能让你有亲密接触大牛思想的机会，这对于你个人的技术进步和职业发展都非常有益。

如果不进入Linux世界，你恐怕很难享受到开源软件如此多的红利。

考虑到以上这些，在设计“趣谈Linux操作系统”专栏的时候，我主要秉承两大原则，希望能够帮你打开Linux操作系统这扇门。

第一个原则仍然是“**趣谈**”。我希望通过故事化的方式，将枯燥的基础知识结合某个场景，给你生动、具象地讲述出来，帮你加深理解、巩固记忆、夯实基础。

操作系统是干什么的呢？我们都知道，一台物理机上有很多硬件，最重要的就是CPU、内存、硬盘、网络。同时，一台物理机上也要跑很多程序，这些资源应该给谁用呢？当然是大家轮着用，谁也别独占，谁也别饿着。为了完成资源分配这件事，操作系统承担了一个“大管家”的作用。它将硬件资源分配给不同的用户程序使用，并且在适当的时间将这些资源拿回来，再分配给其他的用户进程。

鉴于操作系统这个“大管家”的角色，我设计了一个故事，将各个知识点串起来，来帮助你理解和记忆。

假设，我们现在就是在做一家外包公司，我们的目标是把这家公司做上市。其中，操作系统就是这家外包公司的老板。我们把这家公司的发展阶段分为这样几个时期：

- **初创期**：这个老板基于开放的营商环境（x86体系结构），创办一家外包公司（系统的启动）。因为一开始没有其他员工，老板需要亲自接项目（实模式）。
- **发展期**：公司慢慢做大，项目越接越多（保护模式、多进程），为了管理各个外包项目，建立了项目管理体系（进程管理）、会议室管理体系（内存管理）、文档资料管理系统（文件系统）、售前售后体系（输入输出设备管理）。
- **壮大期**：公司越来越牛，开始促进内部项目的合作（进程间通信）和外部公司合作（网络通信）。
- **集团化**：公司的业务越来越多，会成立多家子公司（虚拟化），或者鼓励内部创业（容器化），这个时候公司就变成了集团。大管家的调度能力不再局限于一家公司，而是集团公司（Linux集群），从而成功上市（从单机操作系统到数据中心操作系统）。

![](https://static001.geekbang.org/resource/image/80/5d/80a4502300dfa51c8520001c013cee5d.jpeg?wh=2366%2A6284)

第二个原则就是**图解**。Linux操作系统中的概念非常多，数据结构也很多，流程也复杂，一般人在学习的过程中很容易迷路。所谓“一图胜千言”，我希望能够通过图的方式，将这些复杂的概念、数据结构、流程表现出来，争取用一张图串起一篇文章的知识点。最终，整个专栏下来，你如果能把这些图都掌握了，你的知识就会形成体系和连接。在此基础上再进行深入学习，就会如鱼得水、易如反掌。

![](https://static001.geekbang.org/resource/image/bf/02/bf0bcbea6a24bc5084bc0d4ffca7c502.jpeg?wh=4816%2A2602)

例如，这张图就表示了文件操作在各个层的数据结构的关联。只要你学完之后，能对着这张图将它们之间的关系讲清楚，对于文件系统的部分，你就会了然于心了。

一段新的征途即将开始，今天就是“开学典礼”。从今天开始，在接下来的四个月时间里，我会带你一步一步进入Linux操作系统的大门，让基础变成你技术生涯的左膀右臂。

在开始正式学习之前，我也想听你讲讲，之前你在学习和工作过程中，遇到过哪些操作系统相关的问题，有哪些困惑，又有哪些经验，也可以谈谈你对新学期的期许。

欢迎在留言区和我分享。

[![unpreview](https://static001.geekbang.org/resource/image/19/bc/19bc90ffcf4b1fba4938727e5bc0ecbc.jpg?wh=1029%2A315)](time://mall?url=https%3A%2F%2Fshop18793264.youzan.com%2Fv2%2Fgoods%2F1y7qqgp3ghd2g%3Fdc_ps%3D2347114008676525065.200001)

Linux知识地图2.0典藏版，现货发售2000份，把5米长的图谱装进背包，1分钟定位80%的高频问题。
<div><strong>精选留言（15）</strong></div><ul>
<li><span>无名</span> 👍（45） 💬（4）<p>我想问一下这个图是使用什么工具绘制的？</p>2019-03-26</li><br/><li><span>W.jyao</span> 👍（12） 💬（1）<p>麻烦问下老师，几天一更？</p>2019-03-25</li><br/><li><span>Loens迟暮</span> 👍（22） 💬（2）<p>也没有编程基础，是不是不适合学？</p>2019-03-25</li><br/><li><span>Stubborn</span> 👍（10） 💬（2）<p>我是一个没有计算机基础的学生，这个对我来讲，容易上手嘛？老师</p>2019-03-25</li><br/><li><span>恒</span> 👍（53） 💬（6）<p>刘老师，看您这个专栏需要什么基础呢？</p>2019-03-25</li><br/><li><span>fumeck.com🍋🌴summer sky</span> 👍（24） 💬（2）<p>网络基础、数据结构与算法、操作系统这些基础，真的很重要。。所有的抽象都是基于这些进行封装的，所以学懂他们理解概念与理论很重要。所有的技术与编程语言都是基于此，后面学习其他新技术都是事半功倍。一定要跟上</p>2019-03-25</li><br/><li><span>X</span> 👍（22） 💬（1）<p>作为服务器开发，对刘超老师的话深有同感。因为对Linux缺乏一个系统的学习，遇到很多问题都都只能傻眼，问别人、百度谷歌一下吧，又发现其实是最简单的原理问题。买过《深入理解操作系统》，一大厚本没看得下去。。买过《Linux内核分析》一类的书，满满的内核C语言实现，确实又太“硬核”了一些。。通过学《Linux性能优化专栏》，对CPU上下文切换、平均负载、各种I&#47;O有了进一步的理解，不过，还迫切需要知道进程调度、进程相关的知识，以及最最头疼的网络知识，socket通信和网络包的具体实现，急需拯救！</p>2019-03-25</li><br/><li><span>期末考试</span> 👍（14） 💬（1）<p>我是一名java开发者，昨天遇到一个问题，一个线程跑着跑着就没了。我目前感触是，我在Linux上只会看spring等应用的日志，我觉得线程没了肯定是报错了，但是我的日志就是没有找到报错。后来和运维询问，大概是说内存不足导致的。我的问题是，像内存不足的报错，我能在操作系统的什么地方看到系统级别的日志吗？</p>2019-03-31</li><br/><li><span>星光</span> 👍（11） 💬（1）<p>记忆深刻的是考试前几天把操作系统中的页面置换算法和银行家算法的流程写满了一黑板，考完试就忘光了。现在工作是在Linux系统上做开发，希望可以跟着老师认真系统学习下！</p>2019-03-25</li><br/><li><span>Collapsar</span> 👍（9） 💬（1）<p>没搞懂主线程和进程的关系</p>2019-03-25</li><br/><li><span>珠闪闪</span> 👍（8） 💬（2）<p>刘老师，能想到用外包公司上市这个例子将linux操作系统串起来，真的太厉害了，能看出来刘老师对操作系统的深刻理解。希望跟着老师能对操作系统有更深入的理解，从而不再被开发和分析方案中因为操作系统知识缺失而羁绊，浪费宝贵时间查找资料。而且老师真心赞的是给订阅过《趣谈网络协议》的同学减免10元，真心点赞，鸡冻～</p>2019-03-27</li><br/><li><span>Gojustforfun</span> 👍（8） 💬（1）<p>我很期待，最近在研究Goroutine及其调度。遇到瓶颈，回头补基础找遍经典OS教材也只有进程、线程理论层面的介绍，缺少协程介绍。即使是线程，也没有描述用户级线程到底是如何映射到内核级线程，如何委托内核级线程执行等等一系列问题，希望和老师及同学们讨教｡◕‿◕｡</p>2019-03-25</li><br/><li><span>MJ</span> 👍（7） 💬（1）<p>开始之前，老师推荐几本伴随专栏学习的书籍吧。毕竟更新有点慢啊</p>2019-03-25</li><br/><li><span>云学</span> 👍（6） 💬（1）<p>我以前买过老师的网络专栏，受益匪浅，这次也不犹豫，有个小建议，故事比喻成分不宜太多，还是直白的把知识讲透彻，因为工作中面对的就是硬知识。</p>2019-03-27</li><br/><li><span>风声是棵树</span> 👍（6） 💬（1）<p>老师，作为网工，对编程了解的不多，这门课能看懂吗？</p>2019-03-25</li><br/>
</ul>