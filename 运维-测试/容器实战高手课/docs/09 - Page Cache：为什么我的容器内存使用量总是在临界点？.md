你好，我是程远。

上一讲，我们讲了Memory Cgroup是如何控制一个容器的内存的。我们已经知道了，如果容器使用的物理内存超过了Memory Cgroup里的memory.limit\_in\_bytes值，那么容器中的进程会被OOM Killer杀死。

不过在一些容器的使用场景中，比如容器里的应用有很多文件读写，你会发现整个容器的内存使用量已经很接近Memory Cgroup的上限值了，但是在容器中我们接着再申请内存，还是可以申请出来，并且没有发生OOM。

这是怎么回事呢？今天这一讲我就来聊聊这个问题。

## 问题再现

我们可以用这里的[代码](https://github.com/chengyli/training/tree/main/memory/page_cache)做个容器镜像，然后用下面的这个脚本启动容器，并且设置容器Memory Cgroup里的内存上限值是100MB（104857600bytes）。

```shell
#!/bin/bash

docker stop page_cache;docker rm page_cache

if [ ! -f ./test.file ]
then
	dd if=/dev/zero of=./test.file bs=4096 count=30000
	echo "Please run start_container.sh again "
	exit 0
fi
echo 3 > /proc/sys/vm/drop_caches
sleep 10

docker run -d --init --name page_cache -v $(pwd):/mnt registry/page_cache_test:v1
CONTAINER_ID=$(sudo docker ps --format "{{.ID}}\t{{.Names}}" | grep -i page_cache | awk '{print $1}')

echo $CONTAINER_ID
CGROUP_CONTAINER_PATH=$(find /sys/fs/cgroup/memory/ -name "*$CONTAINER_ID*")
echo 104857600 > $CGROUP_CONTAINER_PATH/memory.limit_in_bytes
cat $CGROUP_CONTAINER_PATH/memory.limit_in_bytes
```

把容器启动起来后，我们查看一下容器的Memory Cgroup下的memory.limit\_in\_bytes和memory.usage\_in\_bytes这两个值。

如下图所示，我们可以看到容器内存的上限值设置为104857600bytes（100MB），而这时整个容器的已使用内存显示为104767488bytes，这个值已经非常接近上限值了。

我们把容器内存上限值和已使用的内存数值做个减法，104857600–104767488= 90112bytes，只差大概90KB左右的大小。

![](https://static001.geekbang.org/resource/image/11/2c/1192c3b6430dc1de84199e9da153502c.png?wh=1184%2A160)

但是，如果这时候我们继续启动一个程序，让这个程序申请并使用50MB的物理内存，就会发现这个程序还是可以运行成功，这时候容器并没有发生OOM的情况。

这时我们再去查看参数memory.usage\_in\_bytes，就会发现它的值变成了103186432bytes，比之前还少了一些。那这是怎么回事呢？

![](https://static001.geekbang.org/resource/image/79/7b/79ed17b066762a5385bf70758d4de87b.png?wh=1192%2A294)

## 知识详解：Linux系统有那些内存类型？

要解释刚才我们看到的容器里内存分配的现象，就需要先理解Linux操作系统里有哪几种内存的类型。

因为我们只有知道了内存的类型，才能明白每一种类型的内存，容器分别使用了多少。而且，对于不同类型的内存，一旦总内存增高到容器里内存最高限制的数值，相应的处理方式也不同。

## Linux内存类型

Linux的各个模块都需要内存，比如内核需要分配内存给页表，内核栈，还有slab，也就是内核各种数据结构的Cache Pool；用户态进程里的堆内存和栈的内存，共享库的内存，还有文件读写的Page Cache。

在这一讲里，我们讨论的Memory Cgroup里都不会对内核的内存做限制（比如页表，slab等）。所以我们今天主要讨论**与用户态相关的两个内存类型，RSS和Page Cache。**

### RSS

先看什么是RSS。RSS是Resident Set Size的缩写，简单来说它就是指进程真正申请到物理页面的内存大小。这是什么意思呢？

应用程序在申请内存的时候，比如说，调用malloc()来申请100MB的内存大小，malloc()返回成功了，这时候系统其实只是把100MB的虚拟地址空间分配给了进程，但是并没有把实际的物理内存页面分配给进程。

上一讲中，我给你讲过，当进程对这块内存地址开始做真正读写操作的时候，系统才会把实际需要的物理内存分配给进程。而这个过程中，进程真正得到的物理内存，就是这个RSS了。

比如下面的这段代码，我们先用malloc申请100MB的内存。

```
    p = malloc(100 * MB);
            if (p == NULL)
                    return 0;

```

然后，我们运行top命令查看这个程序在运行了malloc()之后的内存，我们可以看到这个程序的虚拟地址空间（VIRT）已经有了106728KB（～100MB)，但是实际的物理内存RSS（top命令里显示的是RES，就是Resident的简写，和RSS是一个意思）在这里只有688KB。

![](https://static001.geekbang.org/resource/image/fc/09/fce68702702bd94539357134ba32ab09.png?wh=1616%2A142)

接着我们在程序里等待30秒之后，我们再对这块申请的空间里写入20MB的数据。

```
            sleep(30);
            memset(p, 0x00, 20 * MB)
```

当我们用memset()函数对这块地址空间写入20MB的数据之后，我们再用top查看，这时候可以看到虚拟地址空间（VIRT）还是106728，不过物理内存RSS（RES）的值变成了21432（大小约为20MB）， 这里的单位都是KB。

![](https://static001.geekbang.org/resource/image/8c/ed/8ca5fda34f50166cbc48f6aab93479ed.png?wh=1600%2A146)

所以，通过刚才上面的小实验，我们可以验证RSS就是进程里真正获得的物理内存大小。

对于进程来说，RSS内存包含了进程的代码段内存，栈内存，堆内存，共享库的内存, 这些内存是进程运行所必须的。刚才我们通过malloc/memset得到的内存，就是属于堆内存。

具体的每一部分的RSS内存的大小，你可以查看/proc/\[pid]/smaps文件。

### Page Cache

每个进程除了各自独立分配到的RSS内存外，如果进程对磁盘上的文件做了读写操作，Linux还会分配内存，把磁盘上读写到的页面存放在内存中，这部分的内存就是Page Cache。

Page Cache的主要作用是提高磁盘文件的读写性能，因为系统调用read()和write()的缺省行为都会把读过或者写过的页面存放在Page Cache里。

还是用我们这一讲最开始的的例子：代码程序去读取100MB的文件，在读取文件前，系统中Page Cache的大小是388MB，读取后Page Cache的大小是506MB，增长了大约100MB左右，多出来的这100MB，正是我们读取文件的大小。

![](https://static001.geekbang.org/resource/image/19/df/19e5319f0ac8821986efb005de8235df.png?wh=1612%2A446)

在Linux系统里只要有空闲的内存，系统就会自动地把读写过的磁盘文件页面放入到Page Cache里。那么这些内存都被Page Cache占用了，一旦进程需要用到更多的物理内存，执行malloc()调用做申请时，就会发现剩余的物理内存不够了，那该怎么办呢？

这就要提到Linux的内存管理机制了。 **Linux的内存管理有一种内存页面回收机制（page frame reclaim），会根据系统里空闲物理内存是否低于某个阈值（wartermark），来决定是否启动内存的回收。**

内存回收的算法会根据不同类型的内存以及内存的最近最少用原则，就是LRU（Least Recently Used）算法决定哪些内存页面先被释放。因为Page Cache的内存页面只是起到Cache作用，自然是会被优先释放的。

所以，Page Cache是一种为了提高磁盘文件读写性能而利用空闲物理内存的机制。同时，内存管理中的页面回收机制，又能保证Cache所占用的页面可以及时释放，这样一来就不会影响程序对内存的真正需求了。

### RSS &amp; Page Cache in Memory Cgroup

学习了RSS和Page Cache的基本概念之后，我们下面来看不同类型的内存，特别是RSS和Page Cache是如何影响Memory Cgroup的工作的。

我们先从Linux的内核代码看一下，从mem\_cgroup\_charge\_statistics()这个函数里，我们可以看到Memory Cgroup也的确只是统计了RSS和Page Cache这两部分的内存。

RSS的内存，就是在当前Memory Cgroup控制组里所有进程的RSS的总和；而Page Cache这部分内存是控制组里的进程读写磁盘文件后，被放入到Page Cache里的物理内存。

![](https://static001.geekbang.org/resource/image/a5/3e/a55c7d3e74e17yy613d83e220f93223e.png?wh=1440%2A658)

Memory Cgroup控制组里RSS内存和Page Cache内存的和，正好是memory.usage\_in\_bytes的值。

当控制组里的进程需要申请新的物理内存，而且memory.usage\_in\_bytes里的值超过控制组里的内存上限值memory.limit\_in\_bytes，这时我们前面说的Linux的内存回收（page frame reclaim）就会被调用起来。

那么在这个控制组里的page cache的内存会根据新申请的内存大小释放一部分，这样我们还是能成功申请到新的物理内存，整个控制组里总的物理内存开销memory.usage\_in\_bytes 还是不会超过上限值memory.limit\_in\_bytes。

## 解决问题

明白了Memory Cgroup中内存类型的统计方法，我们再回过头看这一讲开头的问题，为什么memory.usage\_in\_bytes与memory.limit\_in\_bytes的值只相差了90KB，我们在容器中还是可以申请出50MB的物理内存？

我想你应该已经知道答案了，容器里肯定有大于50MB的内存是Page Cache，因为作为Page Cache的内存在系统需要新申请物理内存的时候（作为RSS）是可以被释放的。

知道了这个答案，那么我们怎么来验证呢？验证的方法也挺简单的，在Memory Cgroup中有一个参数memory.stat，可以显示在当前控制组里各种内存类型的实际的开销。

那我们还是拿这一讲的容器例子，再跑一遍代码，这次要查看一下memory.stat里的数据。

第一步，我们还是用同样的[脚本](https://github.com/chengyli/training/blob/main/memory/page_cache/start_container.sh)来启动容器，并且设置好容器的Memory Cgroup里的memory.limit\_in\_bytes值为100MB。

启动容器后，这次我们不仅要看memory.usage\_in\_bytes的值，还要看一下memory.stat。虽然memory.stat里的参数有不少，但我们目前只需要关注"cache"和"rss"这两个值。

我们可以看到，容器启动后，cache，也就是Page Cache占的内存是99508224bytes，大概是99MB，而RSS占的内存只有1826816bytes，也就是1MB多一点。

这就意味着，在这个容器的Memory Cgroup里大部分的内存都被用作了Page Cache，而这部分内存是可以被回收的。

![](https://static001.geekbang.org/resource/image/b5/fe/b5c30dec6f760fbafd81de264b323dfe.png?wh=1142%2A322)

那么我们再执行一下我们的[mem\_alloc程序](https://github.com/chengyli/training/blob/main/memory/page_cache/mem-alloc/mem_alloc.c)，申请50MB的物理内存。

我们可以再来查看一下memory.stat，这时候cache的内存值降到了46632960bytes，大概46MB，而rss的内存值到了54759424bytes，54MB左右吧。总的memory.usage\_in\_bytes值和之前相比，没有太多的变化。

![](https://static001.geekbang.org/resource/image/c6/8a/c6835495bd44817cb5c60069d491c38a.png?wh=1096%2A312)

从这里我们发现，Page Cache内存对我们判断容器实际内存使用率的影响，目前Page Cache完全就是Linux内核的一个自动的行为，只要读写磁盘文件，只要有空闲的内存，就会被用作Page Cache。

所以，判断容器真实的内存使用量，我们不能用Memory Cgroup里的memory.usage\_in\_bytes，而需要用memory.stat里的rss值。这个很像我们用free命令查看节点的可用内存，不能看"free"字段下的值，而要看除去Page Cache之后的"available"字段下的值。

## 重点总结

这一讲我想让你知道，每个容器的Memory Cgroup在统计每个控制组的内存使用时包含了两部分，RSS和Page Cache。

RSS是每个进程实际占用的物理内存，它包括了进程的代码段内存，进程运行时需要的堆和栈的内存，这部分内存是进程运行所必须的。

Page Cache是进程在运行中读写磁盘文件后，作为Cache而继续保留在内存中的，它的目的是**为了提高磁盘文件的读写性能。**

当节点的内存紧张或者Memory Cgroup控制组的内存达到上限的时候，Linux会对内存做回收操作，这个时候Page Cache的内存页面会被释放，这样空出来的内存就可以分配给新的内存申请。

正是Page Cache内存的这种Cache的特性，对于那些有频繁磁盘访问容器，我们往往会看到它的内存使用率一直接近容器内存的限制值（memory.limit\_in\_bytes）。但是这时候，我们并不需要担心它内存的不够， 我们在判断一个容器的内存使用状况的时候，可以把Page Cache这部分内存使用量忽略，而更多的考虑容器中RSS的内存使用量。

## 思考题

在容器里启动一个写磁盘文件的程序，写入100MB的数据，查看写入前和写入后，容器对应的Memory Cgroup里memory.usage\_in\_bytes的值以及memory.stat里的rss/cache值。

欢迎在留言区写下你的思考或疑问，我们一起交流探讨。如果这篇文章让你有所收获，也欢迎你分享给更多的朋友，一起学习进步。
<div><strong>精选留言（15）</strong></div><ul>
<li><span>莫名</span> 👍（74） 💬（6）<p>Momory Cgroup 应该包括了对内核内存的限制，老师给出的例子情况比较简单，基本没有使用 slab，可以试下在容器中打开海量小文件，内核内存 inode、dentry 等会被计算在内。

内存使用量计算公式（memory.kmem.usage_in_bytes 表示该 memcg 内核内存使用量）：
memory.usage_in_bytes = memory.stat[rss] + memory.stat[cache] + memory.kmem.usage_in_bytes

另外，Memory Cgroup OOM 不是真正依据内存使用量 memory.usage_in_bytes，而是依据 working set（使用量减去非活跃 file-backed 内存），working set 计算公式：
working_set = memory.usage_in_bytes - total_inactive_file</p>2020-12-07</li><br/><li><span>蒋悦</span> 👍（12） 💬（1）<p>您好，问一个操作系统相关的问题。根据我的理解，操作系统为了性能会在刷盘前将内容放在page cache中（如果可以申请的话），后续合适的时间刷盘。如果是这样的话，在一定条件下，可能还没刷盘，这个内存就需要释放给rss使用。这时必然就会先刷盘。这样会导致 系统 malloc 的停顿，对吗？如果是这样的话，另外一个问题就是 linux 是如何保证 磁盘的数据的 crash safe 的呢？</p>2020-12-04</li><br/><li><span>Alery</span> 👍（9） 💬（3）<p>首先，很感谢老师，从您这几节关于容器的文章学到了很多知识点，感觉自己以前了解的容器都是些皮毛，在学习的过程中发现容器的很多的问题都需要深入了解操作系统或者linux内核相关的知识，这块知识是我比较缺失的，除了继续打卡老师的文章，空闲时间还想系统的学习一下操作系统及内核相关的知识，老师可以推荐几本讲操作系统或者linux内核相关的书籍吗？</p>2020-12-05</li><br/><li><span>Geek3340</span> 👍（6） 💬（2）<p>page_cache是不是会被很多进程共享呢，比如同一个文件需要被多个进程读写，这样的话，page_cache会不会无法被释放呢？

另外，老师能不能讲解下，这里面的page_cache和free中的cache、buffer、shared还有buffer cache的区别呢？</p>2020-12-04</li><br/><li><span>缝合怪天下无敌</span> 👍（5） 💬（1）<p>请问老师：这边结合了课程内容以及评论中的一些补充想在确认一下以下几个问题：
1 Memory Cgroup OOM 的依据是working set吗？还是说rss，working set都会进行判断
2 这边看到有评论的大佬给了对于memory.usage_in_bytes 以及working_set ，但是对这两个间的关系有一些疑惑，想请问一下老师是否可以理解为working_set = memory.stat[rss] + memory.kmem.usage_in_bytes+常用的page cache 这样？</p>2021-02-04</li><br/><li><span>缝合怪天下无敌</span> 👍（2） 💬（3）<p>老师，这边在请教两个问题，
1 cgroup底下的memory.stat文件中inactive_file 是否在oom时能回收，因为这边看到代码里描述workset = memory.usage-inactive_file这个值获得的，而这边看到在评论区里有大佬提到Memory Cgroup OOM会以working set为标准
2 在查看某个容器的时候发现container_memory_working_set_bytes远小于container_memory_rss指标，原因是因为inactive_file的值非常大，因此这个时候内存监控的时候用container_memory_working_set_bytes似乎看不出来是否马上就要oom这样，因此比较困惑如果想要监控内存的使用率判断oom该使用哪个指标？
下面是查看cgroup的情况：
cat &#47;sys&#47;fs&#47;cgroup&#47;memory&#47;memory.stat 
....
rss 2666110976
...
inactive_anon 0
active_anon 172163072
inactive_file 2489147392
active_file 7667712
....
total_cache 2596864
total_rss 2666110976
total_rss_huge 276824064</p>2021-04-13</li><br/><li><span>笃定</span> 👍（2） 💬（1）<p>老师，如果新程序申请的内存大小是大于之前进程的page cache内存大小的；是不是就会发生oom？</p>2021-03-30</li><br/><li><span>上邪忘川</span> 👍（2） 💬（1）<p>看了这篇文章，对于linux内存的机制和free的使用有了很大的认识</p>2020-12-06</li><br/><li><span>流浪地球</span> 👍（2） 💬（1）<p>请问文中提到rss包括共享内存的大小，那pss呢，pss和rss的区别不是 是否包括共享内存的大小码？</p>2020-12-05</li><br/><li><span>仲玄</span> 👍（0） 💬（1）<p>老师,  page cache 如果都被回收了, 会不会没办法使用page cache 导致磁盘很慢?</p>2021-08-11</li><br/><li><span>Q～Q</span> 👍（0） 💬（1）<p>请问老师： 容器内file-backed过大导致容器oom这个案例有遇到过吗 我现在遇到过一个现象 容器内由于读取的是宿主机的proc&#47;meminfo 导致cache没办法回收 </p>2021-07-30</li><br/><li><span>BertGeek</span> 👍（0） 💬（1）<p>docker容器内存资源限制有了新的认知</p>2021-07-01</li><br/><li><span>13549804879</span> 👍（0） 💬（1）<p>老师，我看过一些liunx的书，看了很容易就忘记了。如何能做到与实践想结合，或者用代码演示kernel的代码。这样可能会容易理解。请问有什么相关的案例书籍推荐。</p>2021-05-13</li><br/><li><span>alpha</span> 👍（0） 💬（2）<p>老师，你好 最近我们在迁移docker时发现当应用第一次去使用到swap的内存时，应用会有几秒的响应延迟？这个问题如何解决？</p>2021-04-11</li><br/><li><span>刘浩</span> 👍（3） 💬（1）<p>我遇见过会因为page cache高OOM的··如果我运行一个定期清cache的就不会OOM，所以容器的cache高真的不会被杀进程吗</p>2022-09-23</li><br/>
</ul>