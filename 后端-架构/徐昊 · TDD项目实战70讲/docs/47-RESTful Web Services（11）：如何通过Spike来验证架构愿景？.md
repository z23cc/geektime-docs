你好，我是徐昊。今天我们继续使用TDD的方式实现RESTful Web Services。

## 回顾架构愿景与任务列表

目前，我们的任务列表是这样的：

- RuntimeDelegate
  
  - 为MediaType提供HeaderDelegate
  - 为CacheControl提供HeaderDelegate
  - 为Cookie提供HeaderDelegates
  - 为EntityTag提供HeaderDelegate
  - 为Link提供HeaderDelegate
  - 为NewCookie提供HeaderDelegate
  - 为Date提供HeaderDelegate
  - 提供OutboundResponseBuilder
- OutboundResponseBuilder
  
  - 可按照不同的Status生成Resposne
- OutboundResponse
- ResourceDispatcher
  
  - 将Resource Method的返回值包装为Response对象
- Providers
  
  - 可获取MessageBodyWriter
  - 可获取ExceptionMapper
- Runtimes
  
  - 可获取ResourceDispatcher
  - 可获取Providers
- MessageBodyWriter
- ExceptionMapper
  
  - 需要提供默认的ExceptionMapper

## 通过Spike验证架构愿景

上节课我们讨论到，当使用伦敦学派时，能不能直接进入经典模式继续开发，对于如何继续拆分任务起到了关键作用。在目前的任务列表中，ResourceDispatcher显然难以直接进入经典模式（也不是不可以，如果你对重构足够自信的话，已经算是恰当的粒度了），那么我们可以围绕ResourceDispatcher继续构建架构愿景，澄清调用栈，然后采用伦敦学派继续开发。

![](https://static001.geekbang.org/resource/image/a2/cb/a2e9416a553cea8a5ab079a716eb30cb.jpg?wh=2072x1215)

如上图所示，是一个非常简单的架构构想：

- 将所有的RootResource的Path转化为正则表达式的Pattern；
- ResourceRouter拿到HttpServletRequest之后，尝试与Pattern匹配；
- 匹配到的RootResource通过Context实例化；
- 调用实例化后的RootResource，处理请求，过程中把中间信息存入UriInfo；
- ResourceRouter拿到结果后，转化为Response对象返回。

接下来我们需要通过Spike验证架构愿景：

## 思考题

在进入下节课之前，希望你能认真思考如下两个问题。

1. 要如何调整架构愿景？
2. 进入项目三的学习后，你有什么有意思的收获吗？

欢迎把你的想法分享在留言区，也欢迎把你的项目代码分享出来。相信经过你的思考与实操，学习效果会更好！
<div><strong>精选留言（3）</strong></div><ul>
<li><span>临风</span> 👍（1） 💬（0）<p>我觉得RootResource下的每个method对应的path可以以每个&#39;&#47;&#39;为分隔，一个segment为一个node，按图的方式进行保存，match的时候就可以按图的方式进行遍历，查找出满足条件的RootResource和方法。</p>2022-12-15</li><br/><li><span>范特西</span> 👍（0） 💬（0）<p>进入项目三的学习后，你有什么有意思的收获吗？

总的来说就是当我们面对一个需求时，刚开始我们可以按照一个小需求的方式去开发，代码可以快速成型并且满足需求需要。但是问题又来了，可能刚开始我们只是考虑得少，以后这个需求可能会变得非常复杂，我们怎么可以在最开始就识别到这些点，并且充分考虑如何设计我们的抽象层而不是依赖于后面一次次“大规模”重构呢？

很期待后面老师的讲解！</p>2024-07-28</li><br/><li><span>Michael</span> 👍（0） 💬（0）<p>老师能讲一下对于Context这种设计么？比如Spring的ApplicationContext, 我们什么情况下会想到可以抽象出一个Context对象？以及为什么是Context去做实例化而不是其他的去做实例化呢？这里面有些什么考量么？</p>2022-07-08</li><br/>
</ul>