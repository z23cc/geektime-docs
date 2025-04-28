你好，我是三桥。

在之前的课程当中，我们设计了一套基于最少字段原则的全链路日志模型。你还记得数据模型中提到的fpId字段吗？

对，它就是指纹ID。在我们深入探讨指纹ID之前，先来看看前端同学在实际项目遇到的一种情况。这次，我们以极客时间的网站为例。

通常，一个Web网站总会有陌生的访客通过浏览器访问。我注意到极客官网会对陌生访客自动弹出一个提供双方意向匹配度的产品功能，给用户推荐合适的内容。

![图片](https://static001.geekbang.org/resource/image/11/03/1160479530cb678897632c70cdf36b03.png?wh=1284x1330)

咱们来看看这个业务场景，用户在选好信息选项后，直接提交并跳转至登录页面。登录成功后，页面会自动跳转至个人主页。

我们假设，你进入个人中心页面时发现很多地方都是空白的，没任何数据显示，包括推荐购买课程的列表。

在这种情况下，我们的前端同学只能针对发生这个问题的用户检查接口是否存在问题。虽然检查结果显示接口没有问题，但我们还是无法追踪用户登录前的流程状态，因为我们无法关联任何未登录用户的交互日志。

想一下，如果我们能在用户访问我们的服务时提供有一个唯一的标识，并将这个标识和用户的行为日志关联起来，那么是不是无论用户是否登录，我们都可以追踪到用户的行为轨迹了？

## 指纹ID

我们每个人的指纹都是独一无二的，浏览器指纹也是同样道理。由于浏览器提供了很多有价值的特性给前端同学通过代码获取，例如UserAgent、分辨率、色彩深度、系统平台、语言、触摸屏、地理位置、语言支持特性、图像特性等等。**所以，我们只要收集这些具有较高辨析度的信息，并进行一定的计算处理，就能生成一个能唯一标识当前浏览器的值，也就是我们所说的指纹ID。**

既然指纹ID是利用浏览器的一些特性来生成一个独一无二的标识，那么，我们就可以利用这个标识，把用户在登录前和登录后的行为日志关联起来，追踪到用户的行为轨迹。有时候，你在不同网站上看到的那些曾经访问过的产品的广告，就是利用指纹特性记录用户行为之后推荐给你的。

如果作为用户，你可能会觉得被网站通过指纹技术追踪自己的行为踪迹，或多或少侵犯了自己的隐私。但在这里，我们的目的是追踪前端应用的用户行为，是为了帮助用户提升体验，快速解决问题。一定要记住，我们的出发点并不是获取用户的隐私。

## 指纹生成方案的选择

市场上有很多生成指纹ID的解决方案。在前端技术领域中，最知名的JavaScript库就是 **FingerprintJS**，它有两个版本，分别是开源版本和商用版本。据官方介绍，开源版本的识别率达到40~60%，而商用版本的识别率则高到99.5%。

实际上，尽管FingerprintJS的指纹识别技术非常出色，但在前端全链路中使用就有点儿大材小用了。为了一个字段，没有必要引入一个复杂的JavaScript库。

那么，有没有其它轻量级的指纹生成解决方案？

答案是有的，这个方案就是**帆布指纹识别技术**。它是一种利用帆布纹理特征进行身份验证的方法。具体实现的思路，就是利用HTML5的画布的Canvas特性，通过Canvas生成的图片，然后转换成哈希码，从而形成用户指纹。

为什么选择Canvas作为我们指纹生成方案呢？

你想想看，我们追踪用户链路日志的前提是**在一个浏览器场景下的行为追踪**。在这节课开始时提到的例子中，我们并不需要在多个浏览器之间识别同一用户。恰好，使用Canvas生成的画布在不同设备和浏览器之间是存在细微的差异的，只有在同设备和浏览器下生成的指纹几乎不变。

## 浏览器指纹的实现

接下来，我们将尝试利用Canvas画布特性，实现生成独特指纹ID的通用方法。

既然我们要使用Canvas特性，那么有一个前提条件就是浏览器必须支持Canvas标签。首先，我们需要创建一个Canvas元素，并生成上下文Context。

```javascript
const canvas = document.createElement('canvas'); const ctx = canvas.getContext("2d");
```

第二步，我们在Canvas画布上填充矩形和文字，并设置字体、颜色、位置等属性。

```javascript
const txt = 'geekbang'
ctx.textBaseline ="top"
ctx.font = "14px 'Arial'"

ctx.fillStyle = "#f60"
// 先画一个60x20矩形内容
ctx.fillRect(125, 1, 60, 20)
// 把字填充到矩形内
ctx.fillStyle = "#069"
ctx.fillText(txt, 2, 15);
```

然后，我们采用社区提供的转换方案，将填充的矩形和文字的画布转换成Base64字符串。接下来，我们使用atob函数对Base64字符串进行编码，最后截取一部分字符，将其转换成十六进制字符串。

```javascript
const b64 = canvas.toDataURL().replace("data:image/png;base64,","");
const bin = atob(b64);
const crc = bin2hex(bin.slice(-16,-12));
```

通过刚才的方案生成指纹ID后，就能产生同一设备下、同一个浏览器的唯一标识。

但要注意，这个代码不是通用的，因为画布是通过输出“geekbang”字符串实现的。为了让每个业务都有自己独特的指纹ID识别逻辑，我们还需要对代码进行简单的封装。

你可以和我一起尝试一下封装优化。由于每个业务都有各自独立性，我们将填充字符串、字体、色彩、位置的部分提取出来，作为参数传入。

例如，我定义了以下四个指纹参数类型。

```javascript
type FingerprintOptions = {
  font?: string
  reactStyle?: string | CanvasGradient | CanvasPattern
  contentStyle?: string | CanvasGradient | CanvasPattern
  textBaseline?: CanvasTextBaseline
}
```

接下来，我把生成指纹的逻辑封装在了一个函数中，并把它命名为getFingerprintId函数。

```typescript
export const getFingerprintId = (content: string, options?: FingerprintOptions) => {
  if (!content) {
    console.error("content is empty");
    return null;
  }
  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext("2d");
  // 如果不存在，则返回空值，说明不支持Canvas指纹
  if (!ctx) return null;

  const txt = content || 'geekbang';
  ctx.textBaseline = options && options.textBaseline ? options.textBaseline : "top";
  ctx.font = options && options.font ? options.font : "14px 'Arial'";

  ctx.fillStyle = options && options.reactStyle ? options.reactStyle : "#f60";
  // 先画一个60x20矩形内容
  ctx.fillRect(125, 1, 60, 20);

  ctx.fillStyle = options && options.contentStyle ? options.contentStyle : "#069";
  // 把字填充到矩形内
  ctx.fillText(txt, 2, 15);

  const b64 = canvas.toDataURL().replace("data:image/png;base64,","");
  const bin = atob(b64);
  const crc = bin2hex(bin.slice(-16,-12));
  return crc;
}
```

getFingerprintId函数接收了两个参数。第一个是必选项，传入业务名称参数，也就是我们在画布上填充的内容。第二个参数是可选项，传入样式参数，用于业务自定义参数，以实现与不同项目的差异化。

至此，我们实现了一套获取浏览器指纹的通用方法。拥有了指纹ID，我们就能将这节课开始时提到的登录前后的用户交互逻辑关联起来。通过登录前的用户全链路日志，我们就能了解用户在登录前的状态了。

下图展示了指纹ID关联全链路日志的流程。

![图片](https://static001.geekbang.org/resource/image/96/e8/964f3917def73be24cd600502cc726e8.png?wh=2014x872)

实际上，使用Canvas特性生成的指纹ID也是有缺点的。它需要依赖canvas画布能力，万一浏览器不支持Canvas就无法生成指纹ID，例如小程序环境。

小程序是一个特殊的容器环境，并非是一个真实的浏览器容器，其提供的canvas能力差异性较大，不能用作指纹ID生成的方案。所以，如果小程序环境要获取类似指纹作用的唯一值，我的建议是把小程序设置为静默登录后获取openid。

## 指纹ID的局限性

说到这，你可能会觉得指纹ID简直太重要了，关联用户在同一浏览器下的每个过程的行为轨迹，定位问题，能解决各种疑难杂症。但是，任何解决方案都是有自己的局限性的。

首先，我们再强调一下，浏览器指纹ID可能侵犯用户的隐私，当多款前端产品使用的是同一套指纹逻辑，产品之间的指纹ID就可以实现用户信息的共享。这也是你经常在不同的软件里看到推荐的广告都很像的原因之一。

其次，就咱们这节课使用的**帆布指纹识别技术**来说，不同设备和浏览器生成的指纹ID是存在差异的。那反过来说，同一用户在不同浏览器使用前端功能的时候，我们就不能关联分析两个浏览器的状态。假设，你在苹果手机上使用微信和Safari浏览器访问同一个前端页面，这时就会出现两个不同指纹ID，这样就没法把两者的链路日志关联起来。

## 总结

本节课，我们重点探讨了把指纹ID关联到全链路日志的方法。这节课实现的Canvas指纹方案逻辑相对简单，不能保证100%不会重复指纹ID，但已足够协助前端同学快速定位问题。

在大公司里，指纹ID的实现方案大同小异。他们的前端产品用户量级更大，更需要增加更多的影响元素来减少重复率。

例如，在前端场景，我们还可以增加浏览器语言、屏幕色彩、屏幕分辨率、操作系统、CPU信息等。结合Canvas指纹，可以进一步增加指纹ID的量级，减少重复率。

对于海外的前端工程师，由于产品调性以及个人喜好，他们更倾向使用诸如Fingerprint这样的订阅制产品。这类商业产品提供的指纹ID更精准，接近100%无重复，非常受欢迎。只不过唯一的缺点是需要付费，且价格不菲。

## 思考题

在这节课的最后，我给你一个思考题。我们之前提到，为了提高指纹识别ID的精度并降低重复率，需要增加更多的影响因素。

首先，你可以尝试增加更多的影响因素，修改上面的通用函数逻辑。

其次，观察一下，增加影响因素后指纹ID是否有明显的差异，以及是否有重复的问题？

欢迎你在留言区和我交流。如果觉得有所收获，也可以把课程分享给更多的朋友一起学习。我们下节课见！
<div><strong>精选留言（7）</strong></div><ul>
<li><span>苏果果</span> 👍（0） 💬（0）<p>06源码：
https:&#47;&#47;github.com&#47;sankyutang&#47;fontend-trace-geekbang-course&#47;blob&#47;main&#47;trace-sdk&#47;src&#47;core&#47;fingerprint.ts

完整源码入口：
https:&#47;&#47;github.com&#47;sankyutang&#47;fontend-trace-geekbang-course</p>2024-07-18</li><br/><li><span>嘿~那个谁</span> 👍（0） 💬（1）<p>课程中的内容没有课间和源码吗？</p>2024-07-11</li><br/><li><span>JuneRain</span> 👍（0） 💬（5）<p>没太理解这个指纹ID为什么要利用 canvas 来生成，直接利用指定字符串 &quot;geekbang&quot; 然后转成 base64 格式再截取不也一样的效果？</p>2024-05-13</li><br/><li><span>westfall</span> 👍（0） 💬（5）<p>有了指纹ID，还需要 traceId 吗？</p>2024-04-30</li><br/><li><span>Aaaaaaaaaaayou</span> 👍（1） 💬（0）<p>同一个型号的手机加同一个版本的浏览器生成出来的画布也会不同么？</p>2024-09-14</li><br/><li><span>天天</span> 👍（0） 💬（0）<p>canvas这个原理确实没讲清楚，我猜就算是同一个型号的手机，同个版本的浏览器，用canvas画一个图案出来也不一样</p>2024-11-14</li><br/><li><span>雪舞</span> 👍（0） 💬（1）<p>bin2hex 这个是自定义函数吗？作用是干嘛的？函数定义是怎样的？</p>2024-05-17</li><br/>
</ul>