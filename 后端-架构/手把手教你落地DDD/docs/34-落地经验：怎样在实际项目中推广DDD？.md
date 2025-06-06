你好，我是钟敬。

前面我们学习的主要是DDD本身的技能。我们以此为基础，进一步精进，就可以向DDD专家迈进了。然而，即使你已经成为DDD专家，也不意味着你可以自然而然地在组织里顺利推广DDD。

这节课，我把我在不同企业中推广 DDD 的一些心得分享出来，相信会对你有所帮助。

## 怎样寻找 DDD 的切入点

在准备推广 DDD 的时候，你的公司一般已经有了自己的一套业务、系统、团队和开发流程。那么，怎么把 DDD 切入到现有的研发体系中呢？

### DDD的切入场景

我们讨论3种最常见的切入 DDD 的场景。

第一种是**新建系统**。也就是说现在刚好有一个新项目，可能是要开发一个全新的系统，也可能是为现有系统新增或重写一个比较大的模块。这时候，领导希望保证质量，降低风险，觉得需要方法学的支持，因此要引入 DDD。

第二种是**改造现有系统**。常见的情况是，某个对公司很有价值的系统，已经维护了很多年，系统架构和质量日益腐化，很难维护，不能满足快速变化的业务需求。这时候希望引入 DDD 对系统进行比较大的重构，使系统重新“焕发青春”。另一种情况是企业要进行服务化改造，把一些现有的单体应用拆分成微服务。

第三种是**改进现有研发流程**。公司未必想专门花一大笔预算新建或者改造系统，但领导已经意识到目前的研发流程有种种不足，如果再“野蛮生长”下去，会有很大的隐患。因此希望通过引入 DDD 等方法，提高研发水平和效能。

这三种情况有时会相互交叉，比如说，不论新建还是改造系统，只要引入了DDD，必然也会带来研发流程的改进。

不论哪种情况，推广 DDD 都应该是“大处着眼，小处着手”。所谓大处着眼，就是认识到，从长远来看引入 DDD对公司的整体收益，并基于这种理念做宏观的长期规划。所谓小处着手，指的是，不能指望在短期内靠“运动式”地推广，就能让DDD大范围普及，而是要先选择若干个范围适中的项目和团队作为试点，取得经验后，再扩大战果、逐步推广。

### 试点项目和团队的选择

那么选择试点项目和团队有什么诀窍呢？

试点的选择，一定要把项目（需求因素和技术因素）和开发这个项目的团队（人的因素）一起考虑，不能孤立地考虑其中一个方面。所以这里把“项目”和“团队”一起来讲。

综合考虑“项目”和“团队”两方面，我们可以从这几点关键因素来衡量：有价值、有痛点、有意愿和有时间。

所谓**有价值**，是指站在企业的角度，这个系统对达成企业的战略目标有较大的意义；或者从业务角度，这个系统能够为公司带来比较大的收益。包括 DDD 在内的任何技术改进过程，都需要一定的成本。只有应用到价值较大系统，才能带来足够的回报。

有些组织为了降低引入 DDD 的风险，会选择一些价值很小的琐碎项目“试试”看。这往往造成投入不足，难以坚持，最后销声匿迹，反而浪费了投入的成本。当然，风险也是要考虑的，尤其是改造核心业务系统。这时候，可以在价值较大的系统中选择一个风险和规模适中的模块做尝试。

所谓**有痛点**，指的是公司管理层或者开发团队确实遇到了难以解决的困难，需求寻求方法学的帮助。如果目前的开发方法挺顺利的，没有感受到明显的问题，只是“为了引入而引入 DDD”，那么往往会动力不足。

“从痛点出发”是引入技术改进的重要思路。有了痛点，才容易制定改进目标，有的放矢，也更容易制定衡量改进效果的标准。痛点识别是否准确，往往影响着 DDD 的落地效果。

所谓**有意愿**，指的是开发团队确实愿意学习新技能。其中，项目经理、开发组长、技术骨干等角色往往起着决定性的作用。

很多伙伴常常困扰于这个问题：如果团队没有改进的意愿怎么办？

其实，从概率的角度，当我们引入一个新技能的时候，总有 20% 的人一开始就很想尝试；有 20% 的人一开始就比较抗拒；中间 60% 的人是“随大流”，先观望一下，看别人做得好了，自己再做。所以，在开始推广的时候，我们要想办法识别出前 20% 最有意愿的团队，先做出榜样，再带动其他人，这样就可以了。

最后，**有时间**也很重要。引入任何新技术，总会有些成本。包括学习成本、试错成本等等。关键是看产出是否大于成本。这些成本初期往往体现为对开发时间的占用。如果团队天天 996 ，根本没有做任何技术改进的时间，那么也很难成功。当然，并不是要团队把所有手头上的事情都暂停，投入 100% 的时间学习DDD。而是要有一个合理的投入比例。

接下来，我们看看从前面说的三种场景切入，要注意什么。

## 新建系统要注意什么

由于新建系统的历史包袱比较小，引入 DDD 往往比重构系统容易。

但是在开发过程中一定要避免“瀑布型思维”。这种思维假设在项目的前期，就可以把需求理解得很透彻，模型建立得很合理，然后在中后期，把模型交给开发人员编码就万事大吉了。

尽管凡是有“敏捷思维”的人，都知道这种假设并不成立，但我还是发现这种错误的认知非常普遍。

对于比较大的项目，前期的总体规划和设计还是必要的。但要意识到，这时候产生的模型还是方向性的、粗粒度的，之后开发过程中要随着对问题理解的深入，不断演进。

除了从需求到模型，再由模型到编码的方向以外，在编码过程中发现模型的问题，反过来修正模型也很重要。另外，刚开始引入 DDD的时候，架构师往往还不成熟，建模时就更容易出现错误，这时候更需要这种演进式的改进。

## 改造现有系统要注意什么

对于引入 DDD 来说，改造现有系统比新建系统更常见。这是因为，多数公司在初创的时候，往往是野蛮生长，并不注重系统的内在质量。发展到一定阶段，粗放式开发的弊病才暴露出来，也就有了改进系统需求。这时候引入 DDD，往往就是为了治理已经存在的系统。

### 改造现有系统的步骤

改造现有系统的基本步骤大体上有 4 步。

![](https://static001.geekbang.org/resource/image/ef/98/ef24687df2e317e9332e5a86dd4de398.jpg?wh=3447x1267)

**第一步是反推领域模型**。新建系统的时候是从需求到模型，可以叫做正推。而由于现有系统已经存在了，所以我们做的第一步，反而是从系统现状中“反推”出当前的领域模型，目的是客观地反映出系统当前的领域知识和逻辑。这时候的模型往往有不少问题，比如不能正确反映领域知识、存在矛盾、冗余等。

反推领域模型，我们可以从数据库、用户界面、代码等方面入手。一般是先看数据库，因为数据库里常常已经凝聚了80%的业务知识。如果数据库看不明白，再看界面，如果还不明白，就只能翻代码了。之所以把代码放在最后，是因为阅读代码比较耗时，尤其是现有系统的代码往往很难理解。

虽然课程是按照“正推”来讲的，但是我们也重点强调了需求、模型和实现的一致性。如果你把这个原理吃透，那么反推领域模型也就不在话下了。

反推出的领域模型可以为进一步的改进建立“基线”。在反推的过程中所发现的问题，也可以作为下一步建立目标领域模型的输入。

**第二步是建立目标领域模型**。根据当前系统的痛点、问题以及业务需求，就可以建立目标领域模型，作为改进的方向。建立目标领域模型，一定要有明确的“时间点”。也就是说，这个目标是 1 年的、3个月的、还是 2 周的。脱离时间谈目标是没有意义的。越远的目标应该越宏观，越粗粒度；越近的目标应该越具体，越细致。过于长远的目标模型往往难以落地，所以要合理地设置目标时间。

**第三步是设计演进路线**。有了当前模型和目标模型，就可以分析两者之间的差距。跨越这个差距的过程就是改进的过程。设计演进路线最大的问题就是怎么保证可行性。一般要把改进过程化整为零，迭代实施，并且还要兼顾日常的业务需求，后面我们还会提到这个问题。

**第四步是迭代实施**。最好基于敏捷软件开发方法，小步快跑地实施。在这个过程中，必然会对之前建立的目标领域模型进行反馈，不断改进。同时还要不断评估开发现状，保证不偏离目标。

### 选择精益切片

上面这 4 步是改造现有系统的总体思路，但真正实施的时候，不能一开始就针对整个系统进行。

就拿反推领域模型来说，由于现有系统往往规模庞大逻辑复杂，如果针对整个系统反推模型，必然旷日持久。再加上建立目标领域模型的时间，可能半年就过去了。这段时间只有模型，没有任何落地的代码，也就看不到任何实际效果。于是人们往往失去耐心， DDD 无疾而终。

另一方面，在开始引入 DDD 的时候，架构师、领域专家、开发人员其实还没有真的掌握相关技能。只有落地到代码，形成闭环，才会有真切的感受，真正学懂 DDD。所以，就算我们愿意花半年时间来建模，由于没有掌握领域建模的精髓，也很难保证模型的正确性，更加无法落地到代码了。

所以，建议的做法是，首先选择系统中一个相对独立的小模块，然后按照前面的 4 步，尽快落地到代码并上线，建立最小闭环。通过这个过程，初步掌握 DDD落地技能并取得实际效果。同时，这么做也能培养人才，积累经验，建立必要的开发流程。完成之后，再选择下一个切片，逐步扩大范围，并深化 DDD 的技能。

这个相对独立的模块往往称为“精益切片”。精益切片的难度、范围、风险要适中，最好在 3 个月内形成最小闭环。

## 改进研发流程要注意什么

再谈谈怎样改进研发流程。这一步，既可以在新建或改造系统的过程中完成，也可以单独进行。

我们首先要看看公司目前是否已经有成熟的研发流程。如果没有，那么就把引入 DDD 作为规范研发流程的契机。

如果已经有了，则要设法把 DDD 融入到现有流程，具体做法就是规定在流程的哪些节点、引入哪些 DDD 活动、由什么人执行、产生哪些交付物。我为你梳理了几个常见的节点。

第一个节点是**创建需求**。对于 DDD 比较成熟的团队，在产品经理、BA等角色创建需求的时候，最好就和架构师一起进行分析，初步建立领域模型、词汇表、业务规则表等交付物。

第二个节点是**需求梳理**。也就是在开始编码之前，需求方要给开发人员讲解需求的时候。如果之前已经有了初步的领域模型，这时候可以同步进行评审和优化。如果之前没有，就可以在这时由需求方和开发人员共同建模。

第三个节点是**系统设计**。复杂的需求要有专门的设计阶段。这时候要注意设计方案是否与领域模型吻合，同时也进一步检验领域模型的合理性。

第四个节点是**编写代码**。要求开发人员一定要根据模型来编写代码和设计数据库。如果发现模型有不合理的地方，也要及时反馈。

第五个节点是**代码评审**。在评审过程中，也要检视代码是否和模型一致，及时处理发现的问题。

第六个节点是**上线**。正式上线后，要对领域模型做好归档，进行必要的版本控制。

最后，有些公司每年对研发流程还有**内部审计环节**。在这个过程中，可以让审计人员观察 DDD 是否执行到位。比如说，如果发现领域模型有 3 个月没有更新了，那么多半说明 DDD已经名存实亡了。

![](https://static001.geekbang.org/resource/image/31/c2/31a51c3ceeab56d2ee72d333310837c2.jpg?wh=3403x1764)

DDD不是银弹，起码要经过几个月到半年的时间才能看到效果。很多团队还没有看到效果，就坚持不下去了。而将 DDD 融入研发流程，形成纪律，则可以保证团队坚持下去，直到产生明显的成效。

## 合理的资源和时间安排

再谈一下资源和时间的安排问题。做技术改进，常见的问题就是和开发业务需求争抢资源。有些团队为了重构系统，一上来就提出要冻结需求 3 个月，业务方往往很难答应。

那么怎样才能兼顾技术改进和业务需求呢？

这里首先要说明一个原理。就是运用演进式架构、重构等技巧，可以把原来看起来不能拆分的系统改造工作，拆成一个个相对独立的小任务。每个任务大概几天就可以完成，并且可以在不影响系统功能的前提下上线。上线后，尽管还不能达到总体的改进目标，但已经向目标迈进了一小步。

掌握了这个技巧，就可以和需求方说明“磨刀不误砍柴工”的道理，从而争取到一定的技术改进时间。比如说，每个迭代争取 30% 的时间改造系统。然后，把拆分出的小任务分散在各个迭代中，就可以和业务需求并行处理了。

当然，投入资源和时间越多，达成改进目标就越快。投入时间的比例，要根据企业的战略目标，由各相关干系人协商决定。关键是，我们建立了一种受控的、可以兼顾业务和技术的方式来推进技术改进工作。

## “低配版”的DDD

再谈一个有意思的问题——怎样在推广过程中降低 DDD 的难度。

DDD 的知识点还是比较多的，而且其中有一些理解起来有一定难度。如果在推广过程中，一下子就让所有人掌握所有知识点，往往会造成很多误解，导致动作走形，影响推广效果。

所以，一开始可以聚焦在 DDD 最核心的问题上，暂时省略其他要点，推行一个“低配版”的 DDD。等到大家掌握了基本技能，需要更深层次的运用时，再引入其他知识点。

那么在开始的时候，哪些可以省略，哪些不能省略呢？我梳理了一张表，供你参考。

![](https://static001.geekbang.org/resource/image/e2/f2/e243fd9367dc83a134f413539a670cf2.jpg?wh=2920x1675)

## 配套开发技能

最后，为了保证 DDD 顺利落地，团队还要加强一些基本的开发技能，包括**整洁代码、面向对象设计、代码重构、自动化测试，持续集成、持续交付，以及进一步的DevOps**等等。

当然，并不是说不掌握这些技能，就不能开始 DDD。事实上，常见的情况是，在推进 DDD 的过程中，同步提升相关技能，起到相得益彰的效果。

## 总结

推广DDD，首先要找到合适的切入点，一般有新建系统、改造现有系统、改进研发流程三种情况。推广时一般是先从试点团队开始，符合有价值、有痛点、有意愿、有时间等条件。

引入DDD新建系统时，要避免瀑布式思维，在开发过程中不断演进模型。

改造现有系统时，一般包括反推当前领域模型、建立目标领域模型、设计演进路线和迭代实施四步。改造时不要一开始就对整个系统建模，而是找到一个规模适当的精益切片，短期内实现需求、模型、实现的最小闭环。在这个过程中提高技能后，再逐步扩大范围。

改进研发流程，则要把 DDD 的活动和产出物融入到流程的各个节点中去，形成纪律，保证 DDD 的持续推进。

此外，我们还要合理安排资源和时间，兼顾技术改进和业务需求的开发。这是通过将大型的技术改造任务拆分成小任务，分散到各个迭代中实现的。

在开始推 DDD的时候，为了降低学习负担，可以只抓最基本的领域建模技术和统一语言，忽略其他知识点，采用 “低配版”的DDD。日后在逐步加强其他技能。

最后，想成功推广 DDD，还需要其他技术的配合，例如整洁代码、重构、自动化测试、CI/CD 等。

## 思考题

最后给你留两道思考题。

1.你在实际项目中推广技术改进（不局限于DDD）曾经遇到过哪些困难？  
2.你在推广或学习新技术的时候，有什么成功经验？

好，今天的课程结束了，有什么问题欢迎在评论区留言。
<div><strong>精选留言（6）</strong></div><ul>
<li><span>6点无痛早起学习的和尚</span> 👍（12） 💬（1）<p>热乎着呢，追更了
思考题：
1. 困难就是：给 leader 讲新的技术改进，但是呢，当然自己也没有 100% 掌握（可能是掌握了 70%啥的），给 leader 讲了一圈，leader 又不去做技术调研&#47;细细研究我的技术改进 idea，就完全靠自己的过往经验（吃老本）否定我的技术改进 idea，否定我的 idea，但是他又说不出来具体理由，完全就是靠过往经验，这样也造成了我的内心不服气，谁也不服气

针对这个困难，一方面我自己也可以改进，
1. 把技术改进掌握 100%，再跟他谈
2. 针对他提出来否定的理由，找到一些解决方案丢给他，说：你提的这些否定是可以解决的
3. 如果做了以上这些，leader 还是否定，说实话，我虽然也是不服气，但是依然还是得听leader的，那就不改进了。但是我内心可能还是不服气，有点抵触。

针对学了 DDD，准备在团队推进，我会先尝试推进领域模型、领域逻辑写在领域对象里，通过应用服务编排领域逻辑，先尝试落地这个编码的一些点。</p>2023-03-02</li><br/><li><span>赵晏龙</span> 👍（5） 💬（5）<p>总的来说吧，其实就是从各方面的利益出发。
1、对于开发人员，你要告诉他，这是为了你将来少加班在努力；
2、对于产品，你要告诉他，东西不分析清楚，后期修改你也来跟着加班吧。
3、对于销售，你要告诉他，东西出不来，你也拿不到提成，反而扣绩效。
4、对于业务方，你要告诉他，这个绝对不会影响交付进度，反而会促进快速交付。
5、对于老板，你要告诉他，这对于公司来说是在节约成本。

当然，各方面用何种说辞，那是你的语言技巧问题。</p>2023-03-17</li><br/><li><span>plimlips</span> 👍（3） 💬（1）<p>DDD，分析模式，设计模式，微服务架构，这些如果能进大学课堂，就好推广了</p>2023-03-06</li><br/><li><span>赵晏龙</span> 👍（2） 💬（1）<p>不得不说，你这个低配版DDD落地的总结实在是太好了，光是这一课就值回票价了。关于重点，没有真实的架构经验其实很难抓得住的。我的落地经验几乎和你的总结是完全一致的。</p>2023-03-17</li><br/><li><span>Geek_36e6d4</span> 👍（0） 💬（1）<p>改造现有系统中的：
第三步，一般要把改进过程化整为零，请问如何化整为零？
第四步，最好基于敏捷软件开发方法，小步快跑地实施，请问如何小步快跑？能有具体步骤例子吗？经常听到敏捷，不知道具体的实施怎么做？

这两个问题请教下钟敬老师下，谢谢。</p>2023-04-19</li><br/><li><span>aoe</span> 👍（2） 💬（0）<p>先来一个低配版的 DDD 这个建议非常好！虽然门槛低了，但也是 DDD ！</p>2023-03-02</li><br/>
</ul>