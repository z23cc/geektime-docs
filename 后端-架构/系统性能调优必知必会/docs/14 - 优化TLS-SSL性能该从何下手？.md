你好，我是陶辉。

从这一讲开始，我们进入应用层协议的处理。

信息安全在当下越来越重要，绝大多数站点访问时都使用https://替代了http://，这就是在用TLS/SSL协议（下文简称为TLS协议）来保障应用层消息的安全。但另一方面，你会发现很多图片类门户网站，还在使用http://，这是因为TLS协议在对信息加解密的同时，必然会降低性能和用户体验，这些站点在权衡后选择了性能优先。

实际上，TLS协议由一系列加密算法及规范组成，这些算法的安全性和性能各不相同，甚至与你的系统硬件相关。比如当主机的CPU支持AES-NI指令集时，选择AES对称加密算法便可以大幅提升性能。然而，要想选择合适的算法，需要了解算法所用到的一些数学知识，而很多同学由于忽视了数学原理便难以正确地配置TLS算法。

同时，TLS协议优化时也需要了解网络和软件工程知识，比如我们可以在网络的不同位置缓存密钥来优化性能。而且，TLS协议还可以优化其他应用层协议的性能，比如从HTTP/1升级到HTTP/2协议便可以通过TLS协议减少1个RTT的时间。

优化TLS性能究竟该从何下手呢？在我看来主要有两个方向，一是对称加密算法的性能优化，二是如何高效地协商密钥。下面我们来详细看看优化细节。

## 如何提升对称加密算法的性能？

如果你用Wireshark等工具对HTTPS请求抓包分析，会发现在TCP传输层之上的消息全是乱码，这是因为TCP之上的TLS层，把HTTP请求用对称加密算法重新进行了编码。**当然，用Chrome浏览器配合Wireshark可以解密消息，帮助你分析TLS协议的细节**（具体操作方法可参考[《Web协议详解与抓包实战》第51课](https://time.geekbang.org/course/detail/175-104932)）。

现代对称加密算法的特点是，即使把加密流程向全社会公开，攻击者也从公网上截获到密文，但只要他没有拿到密钥，就无法从密文中反推出原始明文。如何同步密钥我们稍后在谈，先来看如何优化对称加密算法。

目前主流的对称加密算法叫做AES（Advanced Encryption Standard），它在性能和安全上表现都很优秀。而且，它不只在访问网站时最为常用，甚至你日常使用的WINRAR等压缩软件也在使用AES算法（见[官方FAQ](https://www.win-rar.com/encryption-faq.html?&L=0)）。**因此，AES是我们的首选对称加密算法，**下面来看看AES算法该如何优化。

**AES只支持3种不同的密钥长度，分别是128位、192位和256位，它们的安全性依次升高，运算时间也更长。**比如，当密钥为128比特位时，需要经过十轮操作，其中每轮要用移位法、替换法、异或操作等对明文做4次变换。而当密钥是192位时，则要经过12轮操作，密钥为256比特位时，则要经过14轮操作，如下图所示。

[![](https://static001.geekbang.org/resource/image/8a/28/8ae363f2b0b8cb722533b596b9201428.png?wh=734%2A999 "AES128的10轮加密流程[br]此图由Ahmed Ghanim Wadday上传于www.researchgate.net")](http://www.researchgate.net)

密钥越长，虽然性能略有下降，但安全性提升很多。比如早先的DES算法只有56位密钥，在1999年便被破解。**在TLS1.2及更早的版本中，仍然允许通讯双方使用DES算法，这是非常不安全的行为，你应该在服务器上限制DES算法套件的使用**（Nginx上限制加密套件的方法，参见《Nginx 核心知识100讲》[第96课](https://time.geekbang.org/course/detail/138-75878) 和[第131课](https://time.geekbang.org/course/detail/138-79618)）。也正因为密钥长度对安全性的巨大影响，美国政府才不允许出口256位密钥的AES算法。

只有数百比特的密钥，到底该如何对任意长度的明文加密呢？主流对称算法会将原始明文分成等长的多组明文，再分别用密钥生成密文，最后把它们拼接在一起形成最终密文。而AES算法是按照128比特（16字节）对明文进行分组的（最后一组不足128位时会填充0或者随机数）。为了防止分组后密文出现明显的规律，造成攻击者容易根据概率破解出原文，我们就需要对每组的密钥做一些变换，**这种分组后变换密钥的算法就叫做分组密码工作模式（下文简称为分组模式），它是影响AES性能的另一个因素。**

[![](https://static001.geekbang.org/resource/image/46/d1/460d594465b9eb9a04426c6ee35da4d1.png?wh=1294%2A539 "优秀的分组密码工作模式[br]更难以从密文中发现规律，图参见wiki")](https://zh.wikipedia.org/wiki/%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F)

比如，CBC分组模式中，只有第1组明文加密完成后，才能对第2组加密，因为第2组加密时会用到第1组生成的密文。因此，CBC必然无法并行计算。在材料科学出现瓶颈、单核频率不再提升的当下，CPU都在向多核方向发展，而CBC分组模式无法使用多核的并行计算能力，性能受到很大影响。**所以，通常我们应选择可以并行计算的GCM分组模式，这也是当下互联网中最常见的AES分组算法。**

由于AES算法中的替换法、行移位等流程对CPU指令并不友好，所以Intel在2008年推出了支持[AES-NI指令集](https://zh.wikipedia.org/wiki/AES%E6%8C%87%E4%BB%A4%E9%9B%86)的CPU，能够将AES算法的执行速度从每字节消耗28个时钟周期（参见[这里](https://www.cryptopp.com/benchmarks-p4.html)），降低至3.5个时钟周期（参见[这里](https://groups.google.com/forum/#!msg/cryptopp-users/5x-vu0KwFRk/CO8UIzwgiKYJ)）。在Linux上你可以用下面这行命令查看CPU是否支持AES-NI指令集：

```
# sort -u /proc/crypto | grep module |grep aes
module       : aesni_intel
```

**因此，如果CPU支持AES-NI特性，那么应选择AES算法，否则可以选择[CHACHA20](https://tools.ietf.org/html/rfc7539) 对称加密算法，它主要使用ARX操作（add-rotate-xor），CPU执行起来更快。**

说完对称加密算法的优化，我们再来看加密时的密钥是如何传递的。

## 如何更快地协商出密钥？

无论对称加密算法有多么安全，一旦密钥被泄露，信息安全就是一纸空谈。所以，TLS建立会话的第1个步骤是在握手阶段协商出密钥。

早期解决密钥传递的是[RSA](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29) 密钥协商算法。当你部署TLS证书到服务器上时，证书文件中包含一对公私钥（参见[非对称加密](https://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)），其中，公钥会在握手阶段传递给客户端。在RSA密钥协商算法中，客户端会生成随机密钥（事实上是生成密钥的种子参数），并使用服务器的公钥加密后再传给服务器。根据非对称加密算法，公钥加密的消息仅能通过私钥解密，这样服务器解密后，双方就得到了相同的密钥，再用它加密应用消息。

**RSA密钥协商算法的最大问题是不支持前向保密**（[Forward Secrecy](https://zh.wikipedia.org/wiki/%E5%89%8D%E5%90%91%E4%BF%9D%E5%AF%86)），一旦服务器的私钥泄露，过去被攻击者截获的所有TLS通讯密文都会被破解。解决前向保密的是[DH（Diffie–Hellman）密钥协商算法](https://zh.wikipedia.org/wiki/%E8%BF%AA%E8%8F%B2-%E8%B5%AB%E7%88%BE%E6%9B%BC%E5%AF%86%E9%91%B0%E4%BA%A4%E6%8F%9B)。

我们简单看下DH算法的工作流程。通讯双方各自独立生成随机的数字作为私钥，而后依据公开的算法计算出各自的公钥，并通过未加密的TLS握手发给对方。接着，根据对方的公钥和自己的私钥，双方各自独立运算后能够获得相同的数字，这就可以作为后续对称加密时使用的密钥。**即使攻击者截获到明文传递的公钥，查询到公开的DH计算公式后，在不知道私钥的情况下，也是无法计算出密钥的。**这样，DH算法就可以在握手阶段生成随机的新密钥，实现前向保密。

![](https://static001.geekbang.org/resource/image/9f/1d/9f5ab0e7f64497c825a927782f58f31d.png?wh=1488%2A340)

DH算法的计算速度很慢，如上图所示，计算公钥以及最终的密钥时，需要做大量的乘法运算，而且为了保障安全性，这些数字的位数都很长。为了提升DH密钥交换算法的性能，诞生了当下广为使用的[ECDH密钥交换算法](https://zh.wikipedia.org/wiki/%E6%A9%A2%E5%9C%93%E6%9B%B2%E7%B7%9A%E8%BF%AA%E8%8F%B2-%E8%B5%AB%E7%88%BE%E6%9B%BC%E9%87%91%E9%91%B0%E4%BA%A4%E6%8F%9B)，**ECDH在DH算法的基础上利用[ECC椭圆曲线](https://zh.wikipedia.org/wiki/%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF)特性，可以用更少的计算量计算出公钥以及最终的密钥。**

依据解析几何，椭圆曲线实际对应一个函数，而不同的曲线便有不同的函数表达式，目前不被任何已知专利覆盖的最快椭圆曲线是[X25519曲线](https://en.wikipedia.org/wiki/Curve25519)，它的表达式是y2 = x3 + 486662x2 + x。因此，当通讯双方协商使用X25519曲线用于ECDH算法时，只需要传递X25519这个字符串即可。在Nginx上，你可以使用ssl\_ecdh\_curve指令配置想使用的曲线：

```
ssl_ecdh_curve X25519:secp384r1;
```

选择密钥协商算法是通过ssl\_ciphers指令完成的：

```
ssl_ciphers 'EECDH+ECDSA+AES128+SHA:RSA+AES128+SHA';
```

可见，ssl\_ciphers可以同时配置对称加密算法及密钥强度等信息。注意，当ssl\_prefer\_server\_ciphers设置为on时，ssl\_ciphers指定的多个算法是有优先顺序的，**我们应当把性能最快且最安全的算法放在最前面。**

提升密钥协商速度的另一个思路，是减少密钥协商的次数，主要包括以下3种方式。

首先，最为简单有效的方式是在一个TLS会话中传输多组请求，对于HTTP协议而言就是使用长连接，在请求中加入Connection: keep-alive头部便可以做到。

其次，客户端与服务器在首次会话结束后缓存下session密钥，并用唯一的session ID作为标识。这样，下一次握手时，客户端只要把session ID传给服务器，且服务器在缓存中找到密钥后（为了提升安全性，缓存会定期失效），双方就可以加密通讯了。这种方式的问题在于，当N台服务器通过负载均衡提供TLS服务时，客户端命中上次访问过的服务器的概率只有1/N，所以大概率它们还得再次协商密钥。

session ticket方案可以解决上述问题，它把服务器缓存密钥，改为由服务器把密钥加密后作为ticket票据发给客户端，由客户端缓存密文。其中，集群中每台服务器对session加密的密钥必须相同，这样，客户端携带ticket密文访问任意一台服务器时，都能通过解密ticket，获取到密钥。

当然，使用session缓存或者session ticket既没有前向安全性，应对[重放攻击](https://en.wikipedia.org/wiki/Replay_attack)也更加困难。提升TLS握手性能的更好方式，是把TLS协议升级到1.3版本。

## 为什么应当尽快升级到TLS1.3？

TLS1.3（参见[RFC8446](https://tools.ietf.org/html/rfc8446)）对性能的最大提升，在于它把TLS握手时间从2个RTT降为1个RTT。

在TLS1.2的握手中，先要通过Client Hello和Server Hello消息协商出后续使用的加密算法，再互相交换公钥并计算出最终密钥。**TLS1.3中把Hello消息和公钥交换合并为一步，这就减少了一半的握手时间**，如下图所示：

[![](https://static001.geekbang.org/resource/image/49/20/4924f22447eaf0cc443aac9b2d483020.png?wh=1832%2A1290 "TLS1.3相对TLS1.2，减少了1个RTT的握手时间[br]图片来自www.ssl2buy.com")](https://www.ssl2buy.com/wiki/tls-1-3-protocol-released-move-ahead-to-advanced-security-and-privacy)

那TLS1.3握手为什么只需要1个RTT就可以完成呢？因为TLS1.3支持的密钥协商算法大幅度减少了，这样，客户端尽可以把常用DH算法的公钥计算出来，并与协商加密算法的HELLO消息一起发送给服务器，服务器也作同样处理，这样仅用1个RTT就可以协商出密钥。

而且，TLS1.3仅支持目前最安全的几个算法，比如openssl中仅支持下面5种安全套件：

- TLS\_AES\_256\_GCM\_SHA384
- TLS\_CHACHA20\_POLY1305\_SHA256
- TLS\_AES\_128\_GCM\_SHA256
- TLS\_AES\_128\_CCM\_8\_SHA256
- TLS\_AES\_128\_CCM\_SHA256

相较起来，TLS1.2支持各种古老的算法，中间人可以利用[降级攻击](https://en.wikipedia.org/wiki/Downgrade_attack)，在握手阶段把加密算法替换为不安全的算法，从而轻松地破解密文。如前文提到过的DES算法，由于密钥位数只有56位，很容易破解。

因此，**无论从性能还是安全角度上，你都应该尽快把TLS版本升级到1.3。**你可以用[这个网址](https://www.ssllabs.com/ssltest/index.html)测试当前站点是否支持TLS1.3。

![](https://static001.geekbang.org/resource/image/a8/57/a816a361d7f47303cbfeb10035a96d57.png?wh=1051%2A794)

如果不支持，还可以参见[每日一课《TLS1.3原理及在Nginx上的应用》](https://time.geekbang.org/dailylesson/detail/100028440)，升级Nginx到TLS1.3版本。

## 小结

这一讲，我们介绍了TLS协议的优化方法。

应用消息是通过对称加密算法编码的，而目前AES还是最安全的对称加密算法。不同的分组模式也会影响AES算法的性能，而GCM模式能够充分利用多核CPU的并行计算能力，所以AES\_GCM是我们的首选。当你的CPU支持AES-NI指令集时，AES算法的执行会非常快，否则，可以考虑对CPU更友好的CHACHA20算法。

再来看对称加密算法的密钥是如何传递的，它决定着TLS系统的安全，也对HTTP小对象的传输速度有很大影响。DH密钥协商算法速度并不快，因此目前主要使用基于椭圆曲线的ECDH密钥协商算法，其中，不被任何专利覆盖的X25519椭圆曲线速度最快。为了减少密钥协商次数，我们应当尽量通过长连接来复用会话。在TLS1.2及早期版本中，session缓存和session ticket也能减少密钥协商时的计算量，但它们既没有前向安全性，也更难防御重放攻击，所以为了进一步提升性能，应当尽快升级到TLS1.3。

TLS1.3将握手时间从2个RTT降为1个RTT，而且它限制了目前已经不再安全的算法，这样中间人就难以用降级攻击来破解密钥。

密码学的演进越来越快，加密与破解总是在道高一尺、魔高一丈的交替循环中发展，当下安全的算法未必在一年后仍然安全。而且，当量子计算机真正诞生后，它强大的并行计算能力可以轻松地暴力破解当下还算安全的算法。然而，这种划时代的新技术出现时总会有一个时间窗口，而在窗口内也会涌现出能够防御住量子破解的新算法。所以，我们应时常关注密码学的进展，更换更安全、性能也更优秀的新算法。

## 思考题

最后，留给你一道思考题，TLS体系中还有许多性能优化点，比如在服务器上部署[OSCP Stapling](https://zh.wikipedia.org/wiki/OCSP%E8%A3%85%E8%AE%A2)（用于更快地发现过期证书）也可以提升网站的访问性能，你还用过哪些方式优化TLS的性能呢？欢迎你在留言区与我探讨。

感谢阅读，如果你觉得这节课对你有一些启发，也欢迎把它分享给你的朋友。
<div><strong>精选留言（14）</strong></div><ul>
<li><span>明翼</span> 👍（4） 💬（1）<p>老师我在另一本书上看DH算法不是前向安全算法动态DH算法或ECDH算法才是前向安全的</p>2020-08-03</li><br/><li><span>Bourne</span> 👍（3） 💬（1）<p>当然，用 Chrome 浏览器配合 Wireshark 可以解密消息，帮助你分析 TLS 协议的细节（具体操作方法可参考《Web 协议详解与抓包实战》第 51 课）。严重怀疑老师在给自己打广告哇，点过去一看，干货这么多，又买了，哈哈哈。不过话说回来，老师的课程质量真的很高，《Nginx核心知识100讲》还没学完，到100节左右了（其实有150讲），学到了很多，从Nginx小白到写Nginx配置信手拈来（吹个牛逼），谢谢老师的付出。</p>2020-08-07</li><br/><li><span>妥协</span> 👍（2） 💬（1）<p>所以TLS1.3可以放重放攻击吗</p>2020-08-07</li><br/><li><span>Geek_007</span> 👍（1） 💬（1）<p>陶辉老师你好，我有一个问题一直很困惑，希望能解答一下，感谢。TLS的record size 是16KB，Nginx 上有 ssl_buffer_size 参数，这个参数也会设置死 record size。如果没有像cloudflare一样做了动态tls record 的话 ，就是说TLS的record size是固定的16KB，但是假设TLS建联后每次只发1k的数据，不够16k。那么对端该如何处理呢？是等待？还是说发送发现发送的数据不够16K，就填充？如果等待会造成时延问题，如果填充会造成发送的数据总是过大。所以不知道这个 record size 为16 字节怎么理解。哦，对了，大部分Nginx都没有做动态TLS record size 吧，我是参考 https:&#47;&#47;blog.cloudflare.com&#47;optimizing-tls-over-tcp-to-reduce-latency&#47; 这篇文章才有这个疑惑的。希望讨论老师有空闲之余解答一下。</p>2020-07-07</li><br/><li><span>冬风向左吹</span> 👍（1） 💬（1）<p>请问老师，我在nginx配置中，不管ssl_certificate和ssl_certificate_key是否配置ecc证书，抓包查看服务器的server hello响应中的Cipher Suite字段都是TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256，这是正常的吗？</p>2020-05-30</li><br/><li><span>董永刚</span> 👍（0） 💬（1）<p>TCP三次握手过程中传递公钥，会被黑客拦截到吗，被拦截后就可以获取到公钥信息？</p>2021-04-19</li><br/><li><span>J.Smile</span> 👍（0） 💬（1）<p>读了两遍，其义自见。谓读得熟！come on!</p>2020-07-03</li><br/><li><span>Geek_007</span> 👍（18） 💬（0）<p>TLS1.3 感觉其实也不是优化特别大，TLS1.2 有False Start，也能做到1RTT，至于0RTT，现在主流的CDN应该也还有很多厂家不支持PSK，所以0RTT的效果也不一定好。
课后题：
1、ECC证书应该算是一种优化，因为证书更小，加解密更快。（不过好像因为客户端公钥太长，对客户端不友好，尤其是移动端）
2、算法分离、使用硬件加速卡代理计算。可以极大减轻接入层压力。
3、去掉证书链中的根证书（证书链太长可能会导致多一个RTT，毕竟TCP还在慢启动）
4、还是保持连接吧、减少连接次数。
5、客户端的session ticket 持久化，之前都是提到的服务端的session ticket，但是客户端如果是浏览器或者APP，因为重新打开等操作导致重新建立链接，应该会清空保存的会话票据，服务端即便有票据也没有用了。（个人理解，不正确的话请老师指正）
</p>2020-05-29</li><br/><li><span>我来也</span> 👍（6） 💬（4）<p>最近就遇到过tls协议版本的问题，不过我们是在开倒车。😂

最近升级到k8s后，默认的ingress nignx只支持tls1.2以上的版本，导致某些安卓5.x版本上的app无法与服务器正常通讯，为了兼容这部分用户，只能强制把tls支持的最低版本号调回到1.0。</p>2020-05-29</li><br/><li><span>斯蒂芬.赵</span> 👍（0） 💬（0）<p>老想想问一下再nginx里配置tls选项时候，设置了选项 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  这个选项是从网上随便复制的，目前部分河南的用户安卓手机访问我们的app，报网络错误的问题，我们后台看安卓的上报日志显示：connect reset ，是因为客户端不支持这个加密套件导致的么？这个选项需要设置么？有啥影响?</p>2023-04-23</li><br/><li><span>追风筝的人</span> 👍（0） 💬（0）<p>TLS1.3（参见RFC8446）对性能的最大提升，在于它把 TLS 握手时间从 2 个 RTT 降为 1 个 RTT。</p>2022-01-28</li><br/><li><span>追风筝的人</span> 👍（0） 💬（0）<p>TLS1.3 中把 Hello 消息和公钥交换合并为一步，这就减少了一半的握手时间</p>2022-01-28</li><br/><li><span>Denuth</span> 👍（0） 💬（0）<p>老师求介绍ssl硬件加速方面的实战</p>2020-05-31</li><br/><li><span>毛立平</span> 👍（0） 💬（2）<p>OSCP-&gt;OCSP?  最近碰到过letsencrypt在国内访问ocsp的问题导致ios端延迟3s的问题。 https:&#47;&#47;mp.weixin.qq.com&#47;s&#47;z_QsomzE3jBtwi8VdVDdEA</p>2020-05-29</li><br/>
</ul>