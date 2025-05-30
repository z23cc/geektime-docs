互联网发展到现在，搜索引擎已经非常好用，基本上输入关键词，都能找到匹配的内容，质量还不错。但在1998年之前，搜索引擎的体验并不好。早期的搜索引擎，会遇到下面的两类问题：

1. 返回结果质量不高：搜索结果不考虑网页的质量，而是通过时间顺序进行检索；
2. 容易被人钻空子：搜索引擎是基于检索词进行检索的，页面中检索词出现的频次越高，匹配度越高，这样就会出现网页作弊的情况。有些网页为了增加搜索引擎的排名，故意增加某个检索词的频率。

基于这些缺陷，当时Google的创始人拉里·佩奇提出了PageRank算法，目的就是要找到优质的网页，这样Google的排序结果不仅能找到用户想要的内容，而且还会从众多网页中筛选出权重高的呈现给用户。

Google的两位创始人都是斯坦福大学的博士生，他们提出的PageRank算法受到了论文影响力因子的评价启发。当一篇论文被引用的次数越多，证明这篇论文的影响力越大。正是这个想法解决了当时网页检索质量不高的问题。

## PageRank的简化模型

我们先来看下PageRank是如何计算的。

我假设一共有4个网页A、B、C、D。它们之间的链接信息如图所示：

![](https://static001.geekbang.org/resource/image/81/36/814d53ff8d73113631482e71b7c53636.png?wh=1472%2A1007)  
这里有两个概念你需要了解一下。

出链指的是链接出去的链接。入链指的是链接进来的链接。比如图中A有2个入链，3个出链。

简单来说，一个网页的影响力=所有入链集合的页面的加权影响力之和，用公式表示为：

![](https://static001.geekbang.org/resource/image/70/0c/70104ab44fa1d9d690f99dc328d8af0c.png?wh=227%2A88)  
u为待评估的页面，$B\_{u}$ 为页面u的入链集合。针对入链集合中的任意页面v，它能给u带来的影响力是其自身的影响力PR(v)除以v页面的出链数量，即页面v把影响力PR(v)平均分配给了它的出链，这样统计所有能给u带来链接的页面v，得到的总和就是网页u的影响力，即为PR(u)。

所以你能看到，出链会给被链接的页面赋予影响力，当我们统计了一个网页链出去的数量，也就是统计了这个网页的跳转概率。

在这个例子中，你能看到A有三个出链分别链接到了B、C、D上。那么当用户访问A的时候，就有跳转到B、C或者D的可能性，跳转概率均为1/3。

B有两个出链，链接到了A和D上，跳转概率为1/2。

这样，我们可以得到A、B、C、D这四个网页的转移矩阵M：

![](https://static001.geekbang.org/resource/image/20/d4/204b0934f166d6945a90185aa2c95dd4.png?wh=209%2A116)  
我们假设A、B、C、D四个页面的初始影响力都是相同的，即：

![](https://static001.geekbang.org/resource/image/a8/b8/a8eb12b5242e082b5d2281300c326bb8.png?wh=180%2A160)  
当进行第一次转移之后，各页面的影响力$w\_{1}$变为：

![](https://static001.geekbang.org/resource/image/fc/8c/fcbcdd8e96384f855b4f7c842627ff8c.png?wh=385%2A112)  
然后我们再用转移矩阵乘以$w\_{1}$得到$w\_{2}$结果，直到第n次迭代后$w\_{n}$影响力不再发生变化，可以收敛到(0.3333，0.2222，0.2222，0.2222），也就是对应着A、B、C、D四个页面最终平衡状态下的影响力。

你能看出A页面相比于其他页面来说权重更大，也就是PR值更高。而B、C、D页面的PR值相等。

至此，我们模拟了一个简化的PageRank的计算过程，实际情况会比这个复杂，可能会面临两个问题：

1.等级泄露（Rank Leak）：如果一个网页没有出链，就像是一个黑洞一样，吸收了其他网页的影响力而不释放，最终会导致其他网页的PR值为0。

![](https://static001.geekbang.org/resource/image/77/62/77336108b0233638a35bfd7450438162.png?wh=1190%2A997)  
2.等级沉没（Rank Sink）：如果一个网页只有出链，没有入链（如下图所示），计算的过程迭代下来，会导致这个网页的PR值为0（也就是不存在公式中的V）。

![](https://static001.geekbang.org/resource/image/0d/e6/0d113854fb56116d79efe7f0e0374fe6.png?wh=1203%2A987)  
针对等级泄露和等级沉没的情况，我们需要灵活处理。

比如针对等级泄露的情况，我们可以把没有出链的节点，先从图中去掉，等计算完所有节点的PR值之后，再加上该节点进行计算。不过这种方法会导致新的等级泄露的节点的产生，所以工作量还是很大的。

有没有一种方法，可以同时解决等级泄露和等级沉没这两个问题呢？

## PageRank的随机浏览模型

为了解决简化模型中存在的等级泄露和等级沉没的问题，拉里·佩奇提出了PageRank的随机浏览模型。他假设了这样一个场景：用户并不都是按照跳转链接的方式来上网，还有一种可能是不论当前处于哪个页面，都有概率访问到其他任意的页面，比如说用户就是要直接输入网址访问其他页面，虽然这个概率比较小。

所以他定义了阻尼因子d，这个因子代表了用户按照跳转链接来上网的概率，通常可以取一个固定值0.85，而1-d=0.15则代表了用户不是通过跳转链接的方式来访问网页的，比如直接输入网址。

![](https://static001.geekbang.org/resource/image/5f/8f/5f40c980c2f728f12159058ea19a4d8f.png?wh=368%2A114)  
其中N为网页总数，这样我们又可以重新迭代网页的权重计算了，因为加入了阻尼因子d，一定程度上解决了等级泄露和等级沉没的问题。

通过数学定理（这里不进行讲解）也可以证明，最终PageRank随机浏览模型是可以收敛的，也就是可以得到一个稳定正常的PR值。

## PageRank在社交影响力评估中的应用

网页之间会形成一个网络，是我们的互联网，论文之间也存在着相互引用的关系，可以说我们所处的环境就是各种网络的集合。

只要是有网络的地方，就存在出链和入链，就会有PR权重的计算，也就可以运用我们今天讲的PageRank算法。

我们可以把PageRank算法延展到社交网络领域中。比如在微博上，如果我们想要计算某个人的影响力，该怎么做呢？

一个人的微博粉丝数并不一定等于他的实际影响力。如果按照PageRank算法，还需要看这些粉丝的质量如何。如果有很多明星或者大V关注，那么这个人的影响力一定很高。如果粉丝是通过购买僵尸粉得来的，那么即使粉丝数再多，影响力也不高。

同样，在工作场景中，比如说脉脉这个社交软件，它计算的就是个人在职场的影响力。如果你的工作关系是李开复、江南春这样的名人，那么你的职场影响力一定会很高。反之，如果你是个学生，在职场上被链入的关系比较少的话，职场影响力就会比较低。

同样，如果你想要看一个公司的经营能力，也可以看这家公司都和哪些公司有合作。如果它合作的都是世界500强企业，那么这个公司在行业内一定是领导者，如果这个公司的客户都是小客户，即使数量比较多，业内影响力也不一定大。

除非像淘宝一样，有海量的中小客户，最后大客户也会找上门来寻求合作。所以权重高的节点，往往会有一些权重同样很高的节点在进行合作。

## PageRank给我们带来的启发

PageRank可以说是Google搜索引擎重要的技术之一，在1998年帮助Google获得了搜索引擎的领先优势，现在PageRank已经比原来复杂很多，但它的思想依然能带给我们很多启发。

比如，如果你想要自己的媒体影响力有所提高，就尽量要混在大V圈中；如果想找到高职位的工作，就尽量结识公司高层，或者认识更多的猎头，因为猎头和很多高职位的人员都有链接关系。

同样，PageRank也可以帮我们识别链接农场。链接农场指的是网页为了链接而链接，填充了一些没有用的内容。这些页面相互链接或者指向了某一个网页，从而想要得到更高的权重。

## 总结

今天我给你讲了PageRank的算法原理，对简化的PageRank模型进行了模拟。针对简化模型中存在的等级泄露和等级沉没这两个问题，PageRank的随机浏览模型引入了阻尼因子d来解决。

同样，PageRank有很广的应用领域，在许多网络结构中都有应用，比如计算一个人的微博影响力等。它也告诉我们，在社交网络中，链接的质量非常重要。

![](https://static001.geekbang.org/resource/image/f9/7d/f936296fed70f27ba23064ec14a7e37d.png?wh=1727%2A924)  
学完今天的内容，你不妨说说PageRank的算法原理？另外在现实生活中，除了我在文中举到的几个例子，你还能说一些PageRank都有哪些应用场景吗？

欢迎在评论区与我分享你的答案，也欢迎点击“请朋友读”，把这篇文章分享给你的朋友或者同事。
<div><strong>精选留言（15）</strong></div><ul>
<li><span>滨滨</span> 👍（25） 💬（2）<p>pagerank算法就是通过你的邻居的影响力来评判你的影响力，当然然无法通过邻居来访问你，并不代表你没有影响力，因为可以直接访问你，所以引入阻尼因子的概念。现实生活中，顾客比较多的店铺质量比较好，但是要看看顾客是不是托。</p>2019-04-14</li><br/><li><span>third</span> 👍（17） 💬（1）<p>复习感悟：
1.PageRank算法，有点像
海纳百川有容乃大（网页影响力=所有入链集合的页面的加权影响力之和）
像我汇聚的东西，越多，我就越厉害。

2.随机访问模型
有点像下雨。
海洋除了有河流流经，还有雨水，但是下雨是随机的（网页影响力=阻尼影响力+所有入链集合页面的加权影响力之和）</p>2019-02-26</li><br/><li><span>third</span> 👍（9） 💬（1）<p>作业
1.原理
1）基础：网页影响力=所有入链集合的页面的加权影响力之和

2）拉里佩奇加入随机访问模型，即有概率用户通过输入网址访问
网页影响力=阻尼影响力+所有入链集合页面的加权影响力之和

2.应用场景：
评估某个新行业怎么样，通过计算涌入这个行业的人的智力和数量。
如果这个行业，正在有大量的聪明人涌入，说明这是一个正在上升的行业。

作业及问题
转移矩阵
第一列是A的出链的概率
A0B1&#47;3C1&#47;3D1&#47;3
第二列是B的的出链的概率
A1&#47;2B0C0D1&#47;2
第三列是C的出链概率
A1B0C0D0
第四列是D的出链概率
A0B1&#47;2C1&#47;2D0

等级泄露的转移矩阵应该是
M=[0 0 1&#47;2 0]   
     [1 0  0   0]
     [0 0 1&#47;2 0]
     [0 1 0    0]

还是
M=[0 0 0 1&#47;2]   
     [1 0 0  0 ]
     [0 0 0 1&#47;2]
     [0 1 0  0  ]

假设概率相同，都为1&#47;4
进行第一次转移之后，会发现，后面的
W1=[1&#47;8]
[1&#47;4]
[1&#47;8]
[1&#47;4]

总和已经小于1了，在不断转移的过程中，会使得所有PR为0

等级沉没的转移矩阵怎么写？
M=[0 0 1 0]   
     [1 0  0 0]
     [0 0 0 0]
     [0 1 0  0]
</p>2019-02-25</li><br/><li><span>李沛欣</span> 👍（6） 💬（1）<p>有人的地方，就有入世和出世
有网络和地方，就有入链和出链

入世的人，链接的大牛越多，越有影响力，
对网站而言，链接出去的网页越多，说明网站影响力越大，但是越多链接进来你这里的网页，也直接影响到网站的价值。

出链太多，如同出世一样，耗散内力，排名等级越来越低，最终江湖再见。
入链太多，就可能成为流量黑洞，如同涉世太深的人一样走火入魔。

谷歌创始人拉里佩奇则力图破解等级泄露和等级沉没困境，创造了随机浏览模型。</p>2019-03-01</li><br/><li><span>Ling</span> 👍（4） 💬（1）<p>其实提出阻尼系数，还是为了解决某些网站明明存在大量出链（入链），但是影响力却非常大的情形。比如说 www.hao123.com 一样的导航网页，这种网页就完全是导航页，存在极其多出链；还有各种搜索引擎，比如 www.baidu.com、www.google.com 这种网站，基本不存在出链，但是入链可能非常多。这两种网站的影响力其实非常大。</p>2019-11-21</li><br/><li><span>S.Mona</span> 👍（3） 💬（1）<p>PageRank和机器学习和数据分析的关系是怎样的？</p>2019-10-16</li><br/><li><span>白夜</span> 👍（3） 💬（1）<p>1.把影响力转化为每日使用时间考虑。
在感兴趣的人或事身上投入了相对多的时间。对其相关的人事物也会投入一定的时间。
那个人或事，被关注的越多，它的影响力&#47;受众也就越大。而每个人的时间有限，一般来说最多与150人保持联系，相当于最多有150个出链。
其中，一部分人，没人关注，只有出链没有入链，他们就需要社会最低限度的关照，这个就是社会福利（阻尼）。
2.矩阵以前学了一直不知道在哪里可以应用，今天学了用处感觉还蛮爽的。</p>2019-02-25</li><br/><li><span>Geek_hve78z</span> 👍（3） 💬（1）<p>一、学完今天的内容，你不妨说说 PageRank 的算法原理？
1、PageRank 的算法原理核心思想：
-如果一个网页被很多其他网页链接到的话说明这个网页比较重要，也就是PageRank值会相对较高；
-如果一个PageRank值很高的网页链接到一个其他的网页，那么被链接到的网页的PageRank值会相应地因此而提高。
2、公式
PR(u)=PR(v1)&#47;N1+PR(v2)&#47;N2+……
其中PR(u), PR(v1) 为页面影响力。N1, N2是v1, v2页面对应的出链总数。
3、算法过程
1）给每个页面分配相同的PR值，比如PR(u)=PR(v1)=PR(v2)=……=1
2）按照每个页面的PR影响力计算公式，给每个页面的PR值重新计算一遍
3）重复步骤2，迭代结束的收敛条件：比如上次迭代结果与本次迭代结果小于某个误差，我们结束程序运行；或者比如还可以设置最大循环次数。

二、你还能说一些 PageRank 都有哪些应用场景吗？
引用链接：https:&#47;&#47;36kr.com&#47;p&#47;214680.html
1、研究某段时间最重要的文学作品
2、研究与遗传有关的肿瘤基因
3、图书馆图书推荐
</p>2019-02-25</li><br/><li><span>梁智行</span> 👍（2） 💬（1）<p>用网络科学来理解算法就是，网页的影响力（中心度），体现在：很多人说这网页好（度中心度），说这网页好的网页也要好（特征向量中心度），就好像一个人牛不牛逼，首先他自己要很牛逼，然后很多人说他牛逼，最后说他很牛逼的人也要很牛逼。
</p>2020-05-05</li><br/><li><span>FeiFei</span> 👍（2） 💬（1）<p>PageRank原理：
通过聚合入链和出链的权重，来判断自身的排序。
因为可能没有入链或者外链，因此加入阻尼因子d，来将这种情况规避。</p>2019-08-27</li><br/><li><span>sunny</span> 👍（2） 💬（1）<p>这个计算PR权重的时候，是计算对象的每个入链的权重除以出链数量的之和，那从一开始计算的时候每个页面需要有个原始的权重值才行，这个原始权重是否就是1</p>2019-02-27</li><br/><li><span>Geek_34dbb7</span> 👍（1） 💬（1）<p>淘宝商品流量，某件商品流量越大，销量也会越好，但也要排除刷单</p>2020-05-17</li><br/><li><span>Geek_c9fa4e</span> 👍（1） 💬（1）<p>PageRank算法原理：
   一个网页的影响力=所有入链集合页面的加权影响力之和。
  简单来说，根据你周围的人来去判断你这个人得影响力。</p>2020-04-29</li><br/><li><span>Simon</span> 👍（1） 💬（1）<p>为什么Rank Leak会造成PR为0，怎么算的？</p>2020-04-08</li><br/><li><span>William～Zhang</span> 👍（1） 💬（1）<p>老师，在计算一个网页u的影响力的时候，用到v的影响力，这是怎么得到的？</p>2019-11-14</li><br/>
</ul>