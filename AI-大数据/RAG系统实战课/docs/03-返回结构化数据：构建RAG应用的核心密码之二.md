你好，我是叶伟民。

上一节课我们所看到的对话模式返回的结果是人类语言（assistant的部分）。这会带来一个问题，就是程序不能识别这种语言，程序只能够识别结构化数据。

那什么是结构化数据呢？布尔值、整数、浮点数、数组、json等格式的数据就是结构化数据。人类语言就是非结构化数据。

除了这个问题之外，还有一个问题是AI只能返回一个结果，不能返回多个结果。而我们的MIS系统，特别是查询部分的代码，如果需要按照多个条件查询，肯定是要输入多个参数的。那么我们如何处理这种情况呢？

其实这两个问题都涉及同一个基础概念，就是让AI返回结构化数据，这也是我们今天课程的主题。

## 让AI返回结构化数据

我们由易到难，从最简单的返回布尔值开始。

### 返回布尔值

当我们需要询问大模型一个问题是对是错，例如询问大模型：**老婆饼和老婆是不是同一类东西？** 那么大模型很可能会回答：不是，老婆饼和老婆不是同一类东西。

很显然，程序（例如我们的MIS系统）是无法识别以上回答的。因此我们就需要大模型返回一个布尔值结果，这样程序才能理解。

那么如何实现呢？我们可以这样设置对话模式里面user角色的content值。

```python
messages=[
  {"role": "user", "content": f"""
  请以布尔值格式返回答案：老婆饼和老婆是不是同一类东西？  
  """},
  ]
```

如果大模型运行正常，应该会返回 `false`，显然，程序是可以识别这个结果的。

我们将会在**第十七节课**里用到这个知识点。

### 返回整数

返回一个布尔值只能用来做判断题，但是如果一个问题不是判断题呢？

例如当我们询问大模型：**老婆饼是什么类型？** 那么大模型很可能会回答：老婆饼是食物。

除了前面返回布尔值一节提到的程序无法处理这个问题之外，还会有另外一个问题。就是如果我们的MIS系统没有食物这种类型，只有点心、水果、菜肴这三种类型，那么该怎么办？

我们可以给出一系列选项，然后让大模型回答正确选项。这时候比较适合让程序返回整数类型。

具体如何实现呢？我们可以这样设置对话模式里面user角色的content值。

```python
messages=[
  {"role": "user", "content": f"""
  请以整数格式返回正确选项：老婆饼是什么类型？
  
1. 点心
2. 水果
3. 菜肴
  """},
  ]
```

如果大模型运行正常，应该会返回 `1`，这样程序不但能够理解结果，而且可以保证结果是程序所需要的。我们将会在\*\*[第五节课](https://time.geekbang.org/column/article/806979)\*\*中用到这个知识点。

### 返回浮点数

现在我们可以通过返回布尔值来做判断题，通过返回整数来做选择题。但是如果问题是一个开放性的问题呢？

例如询问大模型：**姚明有多高？** 那么大模型很可能会回答：姚明有226cm。

然而我们的程序需要的是多少米，这时候就需要返回一个浮点数结果，而不是前面的整数结果了。

实现方式也和前面类似，只需要将对话模式里面user角色的content值修改一下。

```python
messages=[
  {"role": "user", "content": f"""
  请以浮点数格式返回按米计算的答案：姚明有多高？  
  """},
  ]
```

如果大模型运行正常，应该会返回 `2.26`。

### 返回数组

我们知道，Python的函数和C#的方法都是支持可变参数的。当我们的程序需要接受可变参数来调用对应的函数或方法时，我们就需要大模型返回数组了。

结合例子，你更容易理解这一点。例如，我们有这样一个Python函数。

```python
def greet_many(*names):
    for name in names:
        print(f"Hello, {name}!")
```

那么我们就需要将对话模式里面user角色的content值设置为如下内容。

```python
messages=[
  {"role": "user", "content": f"""
  请以数组的格式返回答案：请列出唐宋八大家的姓名  
  """},
  ]
```

如果大模型运行正常，应该会返回类似下面这样的结果。

```python
["韩愈","柳宗元","欧阳修","苏洵","苏轼","苏辙","王安石","曾巩"]
```

经过这样的处理，结果就能被程序理解了。

### 返回json

当我们的程序（例如MIS系统），特别是查询部分的代码，如果需要按多个条件查询，肯定是要输入多个参数的。比如这样一个场景——我们需要查询A公司到账款项，而程序需要接受一个SQL查询语句。

这种情况应该如何处理呢？

我们可以让大模型返回一个json结果。例如将对话模式里面user角色的content值设置为如下内容。

```python
messages=[
  {"role": "user", "content": f"""
  请以json的格式返回答案：我们需要查询A公司到账款项。
  
  json格式为：
  {
    'name':'A公司',
    'type':'到账款项',
  }  
  """},
  ]
```

如果大模型运行正常，应该会返回类似后面的结果。

```python
{
  'name':'A公司',
  'type':'到账款项',
 }  
```

经过这样的处理，结果就可以被程序正常处理了么？你可以暂停思考一下，这样解决是否足够完美。

细心的同学会发现，上述回答中的 **‘type’:‘到账款项’** 会有问题，因为数据库里面不一定有**到账款项**这个type。那么如何解决这个问题呢？

## 组合拳

其中一个方法是使用“组合拳”，把前面提到的返回整数结果的方法整合进来。

那么就需要将对话模式user角色的content值设置为如下内容。

```python
messages=[
  {"role": "user", "content": f"""
  请以json的格式返回答案：我们需要查询A公司到账款项。
  
  json格式为：
  {
    'name':'A公司',
    'type':1,
  }
  其中type对应的选项和序号是：
  1. 到账款项
  2. 剩余款项
  请返回对应的序号。
  """},
  ]
```

如果大模型运行正常，应该会返回类似后面的结果。

```python
{
  'name':'A公司',
  'type':1,
 }  
```

然而这种方法对大模型的能力要求很高。如果我们因为价格原因只能用低版本的大模型（一分价钱一分货嘛），这样的处理可能无法满足要求。那只能换一种要求低一点的方法了，就是分步处理。

## 分步处理

我们可以分两步进行。第一步是沿用返回json一节的结果。然后将第一步的结果作为第二步的输入。接着将第二步的输入结果整合进第一步的结果，得到最终结果。

采用这种方法，我们第二步对话模式里面user角色的content值需要这样设置：

```python
messages=[
  {"role": "user", "content": f"""
  请以整数格式返回正确选项：到账款项
  
  1. 到账款项
  2. 剩余款项
  """},
  ]
```

如果大模型运行正常，应该会返回如下类似结果。

```python
1  
```

然后我们用这个结果替换掉第一步结果中的type。于是我们就得到了最终结果。

```python
{
  'name':'A公司',
  'type':1,
 } 
```

## 一劳永逸、通用的方法

懒惰是人类的天性，程序员更是需要懒惰，因为程序员的职责就是制造自动化工具来减轻人类的劳动。那么有没有一种一劳永逸的通用方法，可以更方便地返回程序需要的结构化数据呢？

有的，聪明的同学已经发现了，使用返回json的方案就可以一劳永逸地替代前面所有方案。例如返回布尔值可以用返回以下json格式替代。

```python
{'result':false}
```

返回整数、浮点数可以用返回以下json格式替代。

```python
{'result':1}
```

返回数组可以用返回以下json格式替代。

```python
{'result':["韩愈","柳宗元","欧阳修","苏洵","苏轼","苏辙","王安石","曾巩"]}
```

通过这种方式，我们就能一劳永逸地解决所有问题。

## 返回辅助信息

有了这个基础，我们还可以添加辅助信息。通过辅助信息我们可以理解大模型为什么会输出这样的结果。例如，我们可以要求大模型输出以下json结果。

```python
{'result':false,'理由':'不是，老婆饼和老婆不是同一类东西'}
```

然后我们可以把以上结果用日志记录下来，这样在我们诊断问题的时候就十分方便了。我们能清晰地知道程序结果和大模型实际想法之间的关系，以及输出是否正确。

## 提供示例

然而在实际应用中，即便按照前面的方法操作，有时候，大模型可能还是无法正常输出结构化数据。

这时候怎么办呢？我们可以给大模型一个示例，让大模型照葫芦画瓢去输出。提供示例的对话模式如下。

```python
messages=[
  {"role": "user", "content": f"""
  请根据用户的输入返回json格式结果：
  
  示例1：
  用户：客户北京极客邦有限公司的款项到账了多少？
  系统：
  {{'模块':1,'客户名称':'北京极客邦有限公司'}}

  用户：{用户输入}
  系统：
  """},
  ]
```

其中第5行到第8行就是我们提供的示例。通过这种方法，将会大大提升大模型输出结构化数据的概率。

细心的同学可能发现，我说的是大大提升了概率，并没有说百分百能够保证。那是为什么呢？又如何解决这个问题呢？我们下一节课继续探讨。

最后，我想说明一下，今天这节课侧重概念梳理，所以为了理解起来方便，用到的示例代码都做了简化。如果直接沿用，大概率大模型还是无法正常输出。生产环境下的代码是需要根据大模型的能力以及实际场景做调整的，实际代码量会多很多。我们会在后续动手实战的章节中看到实际代码。

## 小结

好了，今天这一讲到这里就结束了，最后我们来回顾一下。这一讲我们学会了两件事情。

第一件事情是对话模式返回的结果是人类语言，并且只有一个输出。所以我们需要AI将这个唯一的输出转换为程序可以识别的结构化数据。而这种结构化数据最好的格式是json。

第二件事情是AI跟人类一样，有时候需要给他一个样本，他才能够按照这个样本正确输出。

至此，我们第一个实战案例所需的基础概念都已经讲完，我们下一节课开始动手实战环节。

## 思考题

如果给了大模型一个示例，它还是无法正确输出。这时候有什么办法解决呢？

欢迎你在留言区和我交流互动，如果这节课对你有启发，也推荐分享给身边更多朋友。
<div><strong>精选留言（7）</strong></div><ul>
<li><span>张申傲</span> 👍（11） 💬（1）<p>第3讲打卡~

思考题：老师课程中的示例，基本都是在Prompt去做优化，指导LLM生成特定的结构化数据，这种方式在大多数情况下没有问题，但是由于LLM生成的方式本质上是基于概率的，会存在一定的随机性，不一定会完全符合指定的结构。

为了更好地实现格式化输出，可以使用LLM的工具调用(tool-calling&#47;functiom-calling)功能，即在调用LLM的API时，传入可选择的工具列表，并详细描述工具调用参数的格式。这样LLM推理出需要调用工具时，就会生成工具调用的参数，这个参数是完全满足指定的结构(如json)的。通过这种方式，可以让LLM更加稳定地生成结构化数据。目前大多数主流的LLM都支持工具调用功能了。

我在实际的项目中实现过一个基于自然语言的SQL查询代理，就可以利用LLM的tool-calling功能，生成结构化的SQL，并实际执行获取查询结果。感兴趣的同学可以参考我的文章：https:&#47;&#47;blog.csdn.net&#47;weixin_34452850&#47;article&#47;details&#47;141677371 </p>2024-09-06</li><br/><li><span>grok</span> 👍（3） 💬（1）<p>老师请问这个json schema是否有帮助？
“Introducing Structured Outputs in the API” -- https:&#47;&#47;openai.com&#47;index&#47;introducing-structured-outputs-in-the-api&#47;</p>2024-09-06</li><br/><li><span>明辰</span> 👍（2） 💬（1）<p>老师我有一个问题请教
我们面对复杂问题的时候，prompt可能会写很长（尤其是增加了例子的情况），那么到线上应用的时候，也是保留完整的prompt嘛，这样把例子放到prompt里是否会增加不必要的成本</p>2024-09-12</li><br/><li><span>Luo</span> 👍（0） 💬（1）<p>我一般在提示词里使用下面的描述：
&quot;&quot;&quot;
输出的格式要求如下：
The output should be a markdown code snippet formatted in the following schema, including the leading and trailing &quot;```json&quot; and &quot;```&quot;:
```json
{
    &quot;content&quot;: &quot;设计的阅读题内容&quot;, &#47;&#47;使用汉语,控制在250个汉字左右
    &quot;theme&quot;: &quot;&quot;,
    &quot;choices&quot;: [{  &#47;&#47;出的3选择题内容
        &quot;question&quot;: &quot;题目内容&quot; &#47;&#47;根据阅读题内容出的单选题, 使用汉字，控制在100个汉字以内
        &quot;options&quot;: [
            &quot;A_option&quot;: &quot;A选项内容&quot;, &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;B_option&quot;: &quot;B选项内容&quot; &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;C_option&quot;: &quot;C选项内容&quot; &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;D_option&quot;: &quot;D选项内容&quot; &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;correct_option&quot;: &quot;该题目的答案选项&quot; &#47;&#47;该题目的正确选项,内容为&#39;A&#39;,&#39;B&#39;,&#39;C&#39;,&#39;D&#39;中的一个
            &quot;correct_option_reason&quot;: &quot;解释答案的原因&quot; &#47;&#47;使用汉语,控制在100个汉字以内

        ]
    }],
    &quot;difficulty&quot;: &quot;设计的阅读理解的难度&quot; &#47;&#47;分&quot;容易&quot;、&quot;中等&quot;、&quot;难&quot;
}
&quot;&quot;&quot;
返回的JSON格式大概是这样的：
&quot;&quot;&quot;
{
	&quot;content&quot;: “content1”,
	&quot;theme&quot;: &quot;聚会和休闲活动&quot;,
	&quot;choices&quot;: [{
			&quot;question&quot;: &quot;小明和朋友们在哪里举行了野餐活动？&quot;,
			&quot;options&quot;: {
				&quot;A_option&quot;: &quot;电影院&quot;,
				&quot;B_option&quot;: &quot;公园&quot;,
				&quot;C_option&quot;: &quot;湖边&quot;,
				&quot;D_option&quot;: &quot;家里&quot;,
				&quot;correct_option&quot;: &quot;B&quot;,
				&quot;correct_option_reason&quot;: &quot;根据阅读内容，小明和朋友们是在公园里举行野餐。&quot;
			}
		},
		{
			&quot;question&quot;: &quot;他们野餐时享受了哪些活动？&quot;,
			&quot;options&quot;: {
				&quot;A_option&quot;: &quot;划船&quot;,
				&quot;B_option&quot;: &quot;看电影&quot;,
				&quot;C_option&quot;: &quot;品尝美食和聊天&quot;,
				&quot;D_option&quot;: &quot;参加音乐会&quot;,
				&quot;correct_option&quot;: &quot;C&quot;,
				&quot;correct_option_reason&quot;: &quot;阅读内容提到他们在野餐时品尝美食并聊天。&quot;
			}
		},
		{
			&quot;question&quot;: &quot;小明和朋友们的聚会在一天中的哪个时间段结束？&quot;,
			&quot;options&quot;: {
				&quot;A_option&quot;: &quot;中午&quot;,
				&quot;B_option&quot;: &quot;傍晚&quot;,
				&quot;C_option&quot;: &quot;晚上&quot;,
				&quot;D_option&quot;: &quot;早晨&quot;,
				&quot;correct_option&quot;: &quot;B&quot;,
				&quot;correct_option_reason&quot;: &quot;根据阅读内容，他们傍晚时分去电影院观看电影，说明聚会在傍晚结束。&quot;
			}
		}
	],
	&quot;difficulty&quot;: &quot;中等&quot;
}
&quot;&quot;&quot;</p>2024-10-15</li><br/><li><span>爬行的蜗牛</span> 👍（0） 💬（1）<p>更改 prompt</p>2024-09-07</li><br/><li><span>Luo</span> 👍（1） 💬（0）<p>我一般在提示词里使用下面的描述：
&quot;&quot;&quot;
输出的格式要求如下：
The output should be a markdown code snippet formatted in the following schema, including the leading and trailing &quot;```json&quot; and &quot;```&quot;:
```json
{
    &quot;content&quot;: &quot;设计的阅读题内容&quot;, &#47;&#47;使用汉语,控制在250个汉字左右
    &quot;theme&quot;: &quot;&quot;,
    &quot;choices&quot;: [{  &#47;&#47;出的3选择题内容
        &quot;question&quot;: &quot;题目内容&quot; &#47;&#47;根据阅读题内容出的单选题, 使用汉字，控制在100个汉字以内
        &quot;options&quot;: [
            &quot;A_option&quot;: &quot;A选项内容&quot;, &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;B_option&quot;: &quot;B选项内容&quot; &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;C_option&quot;: &quot;C选项内容&quot; &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;D_option&quot;: &quot;D选项内容&quot; &#47;&#47;使用汉语,控制在20个汉字以内
            &quot;correct_option&quot;: &quot;该题目的答案选项&quot; &#47;&#47;该题目的正确选项,内容为&#39;A&#39;,&#39;B&#39;,&#39;C&#39;,&#39;D&#39;中的一个
            &quot;correct_option_reason&quot;: &quot;解释答案的原因&quot; &#47;&#47;使用汉语,控制在100个汉字以内

        ]
    }],
    &quot;difficulty&quot;: &quot;设计的阅读理解的难度&quot; &#47;&#47;分&quot;容易&quot;、&quot;中等&quot;、&quot;难&quot;
}
&quot;&quot;&quot;
返回的JSON格式大概是这样的：
&quot;&quot;&quot;
{
	&quot;content&quot;: “content1”,
	&quot;theme&quot;: &quot;聚会和休闲活动&quot;,
	&quot;choices&quot;: [{
			&quot;question&quot;: &quot;小明和朋友们在哪里举行了野餐活动？&quot;,
			&quot;options&quot;: {
				&quot;A_option&quot;: &quot;电影院&quot;,
				&quot;B_option&quot;: &quot;公园&quot;,
				&quot;C_option&quot;: &quot;湖边&quot;,
				&quot;D_option&quot;: &quot;家里&quot;,
				&quot;correct_option&quot;: &quot;B&quot;,
				&quot;correct_option_reason&quot;: &quot;根据阅读内容，小明和朋友们是在公园里举行野餐。&quot;
			}
		},
		{
			&quot;question&quot;: &quot;他们野餐时享受了哪些活动？&quot;,
			&quot;options&quot;: {
				&quot;A_option&quot;: &quot;划船&quot;,
				&quot;B_option&quot;: &quot;看电影&quot;,
				&quot;C_option&quot;: &quot;品尝美食和聊天&quot;,
				&quot;D_option&quot;: &quot;参加音乐会&quot;,
				&quot;correct_option&quot;: &quot;C&quot;,
				&quot;correct_option_reason&quot;: &quot;阅读内容提到他们在野餐时品尝美食并聊天。&quot;
			}
		},
		{
			&quot;question&quot;: &quot;小明和朋友们的聚会在一天中的哪个时间段结束？&quot;,
			&quot;options&quot;: {
				&quot;A_option&quot;: &quot;中午&quot;,
				&quot;B_option&quot;: &quot;傍晚&quot;,
				&quot;C_option&quot;: &quot;晚上&quot;,
				&quot;D_option&quot;: &quot;早晨&quot;,
				&quot;correct_option&quot;: &quot;B&quot;,
				&quot;correct_option_reason&quot;: &quot;根据阅读内容，他们傍晚时分去电影院观看电影，说明聚会在傍晚结束。&quot;
			}
		}
	],
	&quot;difficulty&quot;: &quot;中等&quot;
}
&quot;&quot;&quot;</p>2024-10-15</li><br/><li><span>Geek_fbf3a3</span> 👍（0） 💬（0）<p>学习到了，比如：LLM的工具调用(tool-calling&#47;functiom-calling)功能，以为数据结构化是个很简单的问题，没想到还有那么多的门道</p>2024-11-05</li><br/>
</ul>