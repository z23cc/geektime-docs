你好，我是叶伟民。

上节课，我们学习了两种改进检索质量的方法——在用户交互层面提供精确信息，在业务逻辑层面提供精确信息。

但是仅仅靠这两种方法是不够的，那么我们如何去学习其他改进RAG质量的技术呢？授人以鱼不如授人以渔，这一节课我们就来学习一种很实用的方法，通过查看LangChain、LlamaIndex源码，学习改进RAG质量的技术。

## 我使用LangChain的经历

LangChain是最早的RAG框架之一，也是至今名气最响的RAG框架。你可以通过后面这个[链接](https://www.langchain.com/)进入到LangChain的官网。

我入门RAG的时候就使用了LangChain。但是正如开篇词里面所说到的，发现LangChain不支持微信小程序流式输出，于是这部分代码不再使用LangChain，改成自己实现。后来又发现当时的Langchain不支持百度文心，于是这部分代码也改成了自己实现。后来又发现LangChain不支持很多中国特有的场景……

不知不觉，我发现在我的RAG应用里面，LangChain占的比例极低。我也看过其他的国产RAG框架，也有各自的缺点，没有一个RAG框架可以满足我现实工作中20%的实际需求。这就是这门课程没有使用任何RAG框架的原因。

根据我的了解，其他人的情况也是如此，RAG项目做得越久，框架定制化的比例就越高。

那么LangChain、LlamaIndex这些RAG框架就没有用了吗？并非如此，这些框架有不少优秀的地方，虽然我们不会整体使用这些框架，但是我们可以借鉴他们优秀的地方，整合进我们自己的项目。

现在，我就讲述一个我自己的实际范例，希望同学们能够通过这个范例学习到如何将LlamaIndex的优点整合进自己的RAG项目。

## 将LlamaIndex优点整合进自己项目

在开始之前，我先为你简单介绍一下LlamaIndex这个框架。

相对于LangChain，LlamaIndex是后起之秀。LlamaIndex专注于构建和维护索引，它的设计初衷是提高信息检索的速度和效率，通过索引化处理，使得大模型能够更快地访问和处理数据。后面这个[链接](https://docs.llamaindex.ai)是LlamaIndex的官网。

将RAG框架的优点整合进自己项目分为四个步骤：

1. 明确问题与目标：清楚地知道自己项目所存在的问题以及自己想要什么。
2. 寻找工具或参考：分析官网文档找到可以解决这些问题的工具或者参考。
3. 找到对应源代码。
4. 整合进自己项目。

### 明确问题与目标

现在我们开始第一步，找到问题和确定目标。

当时的我，在项目里面使用了[第10节课](https://time.geekbang.org/column/article/810048)提到的按换行符分割文本。但是这种方法有时候粒度过大，这就导致了检索准确率比较低，而且输入大模型的token数量过多，也会拉高成本。因此我需要一种粒度适中的文本分割方法，能够兼顾检索准确率和控制成本需要。

现在我清楚地知道自己项目所存在的问题以及自己想要什么了。于是我带着问题和需求开始浏览各个RAG框架官网。这些RAG框架包括LangChain、Qanything、RAGFlow、FastGPT、LlamaIndex等等。最终我在LlamaIndex官网找到了我所需要的东西。

### 寻找工具或参考

当时的我发现LlamaIndex有这么一个概念——SentenceWindowNodeParser（句子窗口节点解析器）。

当时的LlamaIndex官网是这么描述这个概念的：以每五句话为一个窗口进行分割。结合官网给的示例，使用句子窗口节点解析器，我们会将第1句话到第5句话分成第一段文本，第2句话到第6句话分成第二段文本，第3句话到第7句话分成第三段文本，以此类推。

当我看到这段描述之后，我第一时间就意识到，这就是我想要的，因为它满足两个条件：

1. 文本长度粒度适中，为每五句话。
2. 采用滚动分割，因此检索粒度适中，检索准确率更好。

注意，这里我多次用了“当时”这个词，因为LlamaIndex变化十分快，目前的描述是每三句话做分割，也就是是1～3句话是第一段文本，2～4句话为第二段文本，3～5句话是第三段文本。

除了LlamaIndex，LangChain也在快速变化。这也是课程没有使用任何RAG框架的另一个原因。因为如果直接使用这些框架，这门课很快就会过时。但是将这些框架的优点整合进自己项目的方法就没有那么快过时了。这就是我说的授人以鱼，不如授人以渔。

找到了自己想要的概念或者工具之后，接下来我们要找到对应的源代码。

### 找到对应源代码

一般来说，对于一个概念或工具，都会在它的官网文档有对应的实现类。如果这个框架是开源的话，在官网会提供对应的源代码仓库地址。因此你在源代码仓库地址搜索整个实现类即可。

于是当意识到这就是我想要的东西之后，我就按照这个概念的实现类SentenceWindowNodeParser去搜索LlamaIndex在GitHub上的源代码仓库，果然找到了这个类的实现源代码。

找到对应源代码之后，接下来我们就来把对应源代码整合进自己的项目。

### 整合进自己项目

直接把对应源代码整合进自己的项目很简单，但是当我整合进去之后，我发现它并不能很好处理以下文本。

```plain
在一个风和日丽的午后，小猫咪在花园里追逐蝴蝶。它的爪子在草地上轻轻一蹬，便纵身跃起，像个小皮球一样在空中翻滚！

“喵呜——”它轻声叫唤，仿佛在与大自然交流。蝴蝶翩翩起舞，躲避着小猫的捕捉，却又似乎在引诱它继续游戏。

突然，一阵风吹过，带来了远处田野的芳香。小猫停止了追逐，抬头望向远方，似乎在思考着什么……

这时，一只小鸟从天而降，落在了小猫的旁边。它们互相打量着对方，好奇心驱使它们开始交流：“你是谁？”小鸟问道，“我是这片花园的主人。”小猫自信地回答。

它们的对话虽然简单，但却充满了友好的气氛。太阳渐渐西沉，金色的光芒洒在它们身上，为这温馨的一幕增添了几分美感。

突然，一声响亮的汽笛打破了宁静。一辆火车呼啸而过，留下一缕长长的白色蒸汽……

小猫和小鸟被这突如其来的声音吓了一跳，但很快它们又恢复了平静。它们知道，这只是生活中一个小小的插曲。

夜幕降临，星星点点的灯光亮起。小猫和小鸟各自回到了它们的家，结束了这个美好的下午。

在这个充满惊喜与温馨的一天里，它们收获了友谊与成长；而明天，又将是全新的一天，等待着它们去探索和发现。
```

仔细分析之后，发现问题出在当时的LlamaIndex的源码不能很好地支持中文标点符号，例如第1行结尾的叹号、第5行结尾的省略号、第7行结尾的问号。

因此我们还需要针对性地添加对应代码。

看到这你可能会问，为什么要这么麻烦，不直接调用LlamaIndex里面的这个类呢？

## 为什么不直接调用LlamaIndex里面的这个类呢？

不直接调用这类，主要有以下原因。

1. 目前的RAG框架更新太快，经常每一两天就更新一个版本，而且兼容性并没有想象中的好。这样一来，每次因为更好的功能或者性能更新框架版本的时候，原有的代码很可能就跑不通了。
2. 目前的RAG框架基本处于初级阶段，很多功能都不完善，特别是在支持中文方面。更要命的是，直接调用框架里面的类，还不能自定义修改功能。
3. 最后还有性能原因，直接调用框架里面的类会把很多不需要的依赖也导入进来，这会导致在处理高并发的时候，内存使用量多了很多，甚至有可能多两三倍。

基于以上原因，我建议不直接调用框架，而是把框架的源代码整合进自己的项目里。

除了LangChain、LlamaIndex等比较宽泛的RAG框架之外，互联网上还存在很多更细分的RAG框架和技术。下一节课我们将会继续学习探讨，最近大热的GraphRAG框架和其他RAG技术，敬请期待。

## 小结

好了，今天这一讲到这里就结束了，最后我们来回顾一下。这一讲我们学会了两件事情。

第一件事情是现有的RAG框架很难满足实际项目的需求。这就是这门课没有使用任何RAG框架的原因。

第二件事情是我们可以将RAG框架的优点整合进自己的项目。具体分成这几个步骤：先弄清楚知道自己项目所存在的问题以及自己想要什么，分析官网文档找到可以解决这些问题的工具或者参考，随后找到对应源代码，最终把代码整合进自己项目。

## 思考题

当一个RAG框架是用其他编程语言实现的，例如C#的时候，你如何取长补短？

欢迎你在留言区和我交流互动，如果这节课对你有启发，也推荐分享给身边更多朋友。
<div><strong>精选留言（2）</strong></div><ul>
<li><span>Jamysp</span> 👍（0） 💬（0）<p>通过大模型，将C#代码转换为自己熟悉的代码，比如python</p>2024-11-14</li><br/><li><span>kevin</span> 👍（0） 💬（0）<p>老师整合LlamaIndex代码能分享吗？</p>2024-10-24</li><br/>
</ul>