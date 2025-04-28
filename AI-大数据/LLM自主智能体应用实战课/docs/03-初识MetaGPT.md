你好，我是李锟。

这节课我们正式进入实战环节，开始学习本课程第一个 Autonomous Agent 开发框架——**MetaGPT，也是目前此类开发框架中最为成熟的一个。**

目前 Python 是 AI + LLM 应用开发的首选编程语言，我们这套课程中使用的编程语言是 Python，选择的开发框架也是 Python 开发框架。其他编程语言例如 Java、C#、JavaScript、Go 等语言的开发者，暂时无法兼顾，只能说抱歉了。

## 安装 Python、Node.js 和 Git

在 Linux 主机上需要确保有符合要求的 Python 版本。在 Ubuntu 上安装 Python 的方法很多，你可以自行搜索。理想情况下，Ubuntu 自带的 Python 版本即可满足要求。从兼容性的角度，Python 的版本我推荐使用官方的 3.10 版。

虽然 Python 最新的官方版本已经是 3.13 版，但仍然有很多 AI、LLM 开发库无法在 3.12 以上版本中使用。所以以后对于要学习的每个开发框架，除非我明确说明可使用 Python 3.12 以上版本，否则都使用 3.10 或 3.11 版。使用 Conda 安装的 Python 也是可以的，只要对应的 Python 版本满足上述要求。

在 Python 环境中，需要提前安装好 poetry，因为我们将使用 poetry 这个强大的工具来创建和管理 Python 虚拟环境。安装方法：

```plain
pip3 install poetry
```

注意：在安装完 poetry 之后，还要确保将 poetry 命令所在目录加入到 PATH 环境变量中，以便以后可以在命令行直接调用。

还需要在 Linux 主机上安装好 Node.js 和一些 MetaGPT 会用到的基于 Node.js 的工具。

```plain
sudo apt update
sudo apt install nodejs build-essential
sudo npm install -g @mermaid-js/mermaid-cli
npx puppeteer browsers install chrome-headless-shell
```

在 Linux 主机上安装好 Git。

```plain
sudo apt install git
```

## Python 项目初始化

首先我们需要初始化一个 Python 项目，在 Linux 主机的终端窗口执行以下命令：

```plain
mkdir -p ~/work/learn_metagpt
cd ~/work/learn_metagpt
touch README.md
poetry init  
# 创建 poetry 虚拟环境，当出现以下提示
Compatible Python versions [^3.10]:  时，输入 ">=3.9, <3.12" (注意别加引号)
```

因为众所周知的原因，我建议使用国内的 Python 库镜像服务器，例如上海交大的镜像服务器：

```plain
poetry source add --priority=primary mirrors https://mirror.sjtu.edu.cn/pypi/web/simple
```

如果上海交大镜像服务器的访问速度很慢，也可以使用其他镜像服务器，你可以自己搜索其他镜像服务器的地址。

然后安装一些常用的 Python 库：

```plain
poetry add pysocks socksio 
```

MetaGPT 项目的创始人是吴承霖老师，于 2023 年 6 月开源。

项目的源代码在：[https://github.com/geekan/MetaGPT](https://github.com/geekan/MetaGPT)

官方文档在：[https://docs.deepwisdom.ai/main/zh/guide/get\_started/introduction.html](https://docs.deepwisdom.ai/main/zh/guide/get_started/introduction.html)

因为 MetaGPT 是由中国开发者创建并主导的，所以有很好的中文文档。

这里我就不花时间赘述 MetaGPT 的历史了，你可以自己搜索阅读网上的资料。闲话少述，我们直接进入主题，在项目中安装 MetaGPT，运行一些官方提供的例子。

## 安装 MetaGPT 并运行第一个例子

MetaGPT 的安装方法主要有两种：Python 库安装和项目源代码安装 (最新的开发版本)。因为即将发布的 MetaGPT 1.0 版与当前的 0.8.x 稳定版相比变化很大，所以我推荐基于项目最新的源代码 (见下面的命令) 安装，以便学习到最新的知识，能够长期有效。

执行以下命令安装 MetaGPT 及其依赖的 Python 库：

```plain
cd ~/work
git clone https://github.com/geekan/MetaGPT.git
cd learn_metagpt
# poetry add --editable "../MetaGPT"
poetry install --no-root && poetry run pip install -e "../MetaGPT" --config-settings editable_mode=compat
```

还记得在上一课的结尾，我们在 Linux 主机上使用 ollama 部署了一个开源 LLM——阿里巴巴的 qwen2.5。确保以下命令：

```plain
ollama run qwen2.5 
```

能够正常运行。然后我们使用这个开源 LLM 来运行 MetaGPT 官方提供的例子。

运行第一个 MetaGPT 例子前，需要给项目添加一个配置文件。建议你把配置文件放在 ~/.metagpt 目录下。

```plain
mkdir ~/.metagpt
vi ~/.metagpt/config2.yaml 
```

输入以下内容：

```plain
llm:
  api_type: 'ollama'
  base_url: 'http://127.0.0.1:11434/api'
  model: 'qwen2.5'
  temperature: 0


CALC_USAGE: True
repair_llm_output: true
```

保存配置文件 config2.yaml，然后执行以下命令，看看会发生什么神奇的事情。

```plain
poetry run metagpt "write a cli blackjack game"
```

屏幕上出现了长串输出的文本和代码，滚动了很长时间。请耐心等待一段时间，运行期间可以去做些其他事情，或者喝杯咖啡偷个懒 :) 。

如果在输出中没有报告任何错误，说明命令执行是成功的。输出中有一些警告：

```plain
Model qwen2.5 not found in TOKEN_COSTS
```

对于当前的 MetaGPT 0.8.x 稳定版，因为在系统中没有配置通过 ollama 支持的开源 LLM 的 token 费率，出现这个警告是正常的，可以简单忽略。在 MetaGPT 最新的开发版中已经解决了这个问题。

上述这个简单的命令究竟做了些什么？我向 MetaGPT 提出了一个要求：编写一个可在命令行运行的 21 点 (BlackJack) 游戏 。但是我并没有告诉 MetaGPT 21 点游戏究竟是啥，它通过底层 LLM 的理解能力和搜索工具自己搞清楚了这件事，并且写出了可以运行的 Python 代码（默认编程语言是 Python，也可以生成其他语言的代码）。

如果需要保存 metagpt 命令输出的内容，可以将屏幕输出重定向到一个文件。

```plain
poetry run metagpt "write a cli blackjack game" > blackjack.txt
```

运行完成后，可以使用编辑器打开 blackjack.txt，看看 MetaGPT 所生成的代码。有兴趣的话，还可以亲自把这些代码跑一下。

与 Autonomous Agent 的初次邂逅感觉如何？是不是与以前使用过的各种 ChatBot 的体验迥异？我们可以直观体会到，**Autonomous Agent 与 ChatBot 的主要区别就是自主性和自动化程度。**MetaGPT 完成我提出的编程要求，确实花了较长时间，但是如果自动生成的源代码大多数是可用的，带来的价值远远超出了所花费的时间。

那么生成的这些代码是否真的可用呢？里面会不会有 bug？这个问题跟我们所使用的基础 LLM 的能力有关，不同的 LLM 在生成代码方面的能力差距会很大。我们在这里所使用的 qwen2.5 在生成代码方面，是开源 LLM 中的佼佼者。代码质量的这个问题目前并不是重点，我们暂时先放下。

我们可以把默认情况下的 MetaGPT 看作是一个标准的**软件开发团队**，其中有架构师、程序员、测试工程师等等不同的角色。团队的产出物是可编译、可运行并通过了测试的软件源代码。但是使用 MetaGPT 不仅能组建一个软件开发团队，MetaGPT 事实上能组建各种不同类型的工作团队。在下一课我们将要开发一个能够陪伴我们玩 24 点游戏的智能体应用，我们并不需要它给我们编一套 24 点游戏的 Python 代码，而是希望它像一个人类一样能直接陪伴我们玩这个游戏。

## MetaGPT官方文档导读

MetaGPT 有很棒的官方文档，在我们课程中重复介绍免费的官方文档是毫无意义的。所以我会补充官方文档中没有提及的内容。前面我也说过，本课程可以看作是一个武林秘籍的目录，而不是取代这些武林秘籍。要深入掌握一个开发框架，**通读官方文档**是不能免去的功课，因此我在这里对官方文档做一个导读。

MetaGPT 官方文档中，初学者首先需要阅读最前面这三个部分。

- 概念简述：[https://docs.deepwisdom.ai/main/zh/guide/tutorials/concepts.html](https://docs.deepwisdom.ai/main/zh/guide/tutorials/concepts.html)
- 智能体入门：[https://docs.deepwisdom.ai/main/zh/guide/tutorials/agent\_101.html](https://docs.deepwisdom.ai/main/zh/guide/tutorials/agent_101.html)
- 多智能体入门：[https://docs.deepwisdom.ai/main/zh/guide/tutorials/multi\_agent\_101.html](https://docs.deepwisdom.ai/main/zh/guide/tutorials/multi_agent_101.html)

首先需要理解关于多 Agent 应用的一些核心概念。你不要急于运行示例代码，先认真阅读一下上述概念简述，这里我仅引用该文档里的一小部分内容：

> 学术界和工业界对术语“**智能体**” (Agent) 提出了各种定义。大致来说，一个智能体应具备类似人类的思考和规划能力，拥有记忆甚至情感，并具备一定的技能以便与环境、智能体和人类进行交互。
> 
> 在 MetaGPT 看来，可以将智能体想象成环境中的数字人，其中  
> **智能体 = 大语言模型（LLM） + 观察 + 思考 + 行动 + 记忆**
> 
> **多智能体** (MultiAgent) 系统可以视为一个智能体社会，其中  
> **多智能体 = 智能体 + 环境 + 标准流程（SOP） + 通信 + 经济**

在概念简述文档中，对上述两个公式中右侧的各个名词也给出了详细的解释。这些概念是 MetaGPT 设计的基础，我们理解了这些概念，再去理解其他 Autonomous Agent 开发框架的设计也会容易很多。现代 AI 研究方法的理论基础——**深度学习**其实是基于**仿生学**，也就是通过计算机软件模拟人类的脑科学。MetaGPT 对于 Agent 和多 Agent 的理解，同样也是基于仿生学。我们可以把一个 Agent 理解为一个像人类一样有智能的实体，可以通过自然语言来与之交互。多个 Agent 可以组合在一起，通过自然语言彼此沟通，并开展复杂的协作。

在官方文档中给出了一个简单的多 Agent 协作的图示：

![图片](https://static001.geekbang.org/resource/image/53/f3/53b0571e78ce5ac5940aecce879340f3.png?wh=1920x1061)

MetaGPT 最初设计和实现的典型场景是一个软件开发团队，所以这个图示是一个极简的软件开发团队。虽然这只是一个简单的图示，我们可以从中了解到很多信息：

- 每个 Agent 都有自己的角色 (也可以理解为岗位)。在这个图示中，Alice 作为产品经理编写需求文档；Bob 作为项目经理，分配任务并对生成的代码做出评价；Charlie 作为程序员，根据需求和任务描述编写 Python 代码。每个 Agent 仅完成自己这个角色需要完成的任务，各司其职，通过协作完成复杂的工作。这就是所谓的 **Role Playing**(角色扮演)，更好地支持 Role Playing 正是目前各个基础 LLM 的一个重要的改进方向。
- 同一个团队中的多个 Agent 共享一个环境。它们通过从环境接收和向环境发送各种事件来开展通信和协作。
- 环境中的事件有多种类型，最重要的事件是**执行Action**。执行每个Action (一个事件) 的输出结果会累积起来，作为接收这个事件的 Agent 将要执行的后续 Action 的输入上下文。每个 Agent 可订阅环境中自己感兴趣的任何 Action，并且基于这些 Action 的输出结果开展思考，确定自己需要执行的后续 Action。
- 执行 Action 时可以根据需要调用各种外部工具 ，完成某个任务并且返回执行结果。外部工具可以做需要的任何事情，包括访问搜索引擎、查询数据库、在订票网站上订飞机票、启动设备生产一批零件等等。
- 每个 Agent 都有自己的记忆，能够记住自己之前与其他智能体开展的对话、接收到的事件、采取的行动。并且根据记忆做出适合的决策，执行适合的 Action。

通过阅读 MetaGPT 官方文档的前三个部分，还可以了解到，在一个多 Agent 应用中，每一个 Agent 既可以由程序 (机器人) 来担任，也可以由人类用户来担任。听起来非常有趣是吧？

MetaGPT 的设计目标之一就是让人类用户与各种智能体完美协作。我们很快就会体会到，这样的设计非常灵活和强大。因为在一些复杂的协作中，当基础 LLM 尚不够强大时，由人类来承担关键角色，做出决策和评判是很重要的。LLM 同样也不是银弹，但是它是人类很好的辅助工具。在**百分之百自动化**难以实现时，能够大幅提高工作效率、降低工作成本的**半自动化**工作流同样也是值得追求的。

## 第二个例子：人类参与的多 Agent 应用

第一个例子是通过 metagpt 命令运行的，我们没有看到源代码，会感觉隔靴搔痒。接下来我们来看一个更简单的例子，一个支持人类用户参与的多 Agent 应用例子，示例代码在课程代码文件 build\_customized\_multi\_agents.py 中。

在这个示例代码中有三个 Role 的子类 (即前面图示中的 Agent)：一个程序员 SimpleCoder、一个测试工程师 SimpleTester、一个评论员 SimpleReviewer。由 SimpleReviewer 确定生成的代码质量是否可以接受以及工作是否成功完成。示例代码中每个 Agent 的名字和角色划分，与前面的图示有些差别，注意别搞混了。代码略微有点长，就不占用篇幅了，你可以自己用编辑器打开代码文件查看。

使用以下命令运行这个例子：

```plain
poetry run python build_customized_multi_agents.py --idea "write a function that calculates the product of a list" --add_human=True
```

在上述命令中，我提出的任务要求是，编写一个 Python 函数来计算列表中的产品数量。这里有一个参数 “add\_human=True”，默认为 False。如果设置为 True，则由人类用户来充当评论员，对生成的代码做出评价。执行上述命令时，在输出生成的代码之后，会在命令行等待用户的输入，可以简单输入“It’s OK”以结束程序执行。

从这个例子的代码可以看出，在一个典型的 MetaGPT 应用中，包含了三个层次：Team (团队)、Role (角色) 和 Action (行动)。

- 一个 Team 可包含 (雇佣) 多个 Role。在这个例子中，Team 包含了三个 Role：SimpleCoder、SimpleTester、SimpleReviewer。
- 每个 Role 可包含 (执行) 一个或多个 Action。SimpleCoder 包含了 SimpleWriteCode、SimpleTester 包含了 SimpleWriteTest、SimpleReviewer 包含了 SimpleWriteReview。

当传入“–add\_human=True”选项时，在初始化 SimpleReviewer 对象时会传入一个“is\_human=True”参数，这样 SimpleReviewer 就不再执行 SimpleWriteReview 这个 Action 来获得对代码的评价，而是在命令行等待人类用户输入代码的评价。

这个简单的例子比较容易理解。大多数神奇的事情是由开发框架实现的，我们在这里只需要理解高层的抽象即可。每个 Role 对象都是事件驱动的，在这里的事件就是各个 Action。在运行之前每个 Role 对象首先注册自己可执行的 Action，以及自己感兴趣的 Action。然后这些不同的 Action 按照串行和并行方式组合在一起，构成一个复杂的工作流。

从这个例子的代码再结合对之前图示的讲解，我们还可以了解到以下细节：

- 每个 Action 对象的 run() 函数的输出，会累积起来，作为接收这个事件的 Role 对象后续执行的 Action 对象 run() 函数的输入参数 context。
- 如果系统的工作流是单一串行的，在 Role 对象的初始化函数中，只需要注册自己可以执行的 Action，和订阅自己感兴趣的 Action。例如 SimpleReviewer 对象的初始化函数：

```plain
     def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.set_actions([SimpleWriteReview])
        self._watch([SimpleWriteTest])
```

需要注意的是，这里的提示词 (Prompt) 默认都是英文的。这也是目前大多数 Autonomous Agent 开发框架的要求。这对于 Autonomous Agent 来说不是一个大问题。因为这类应用并不需要使用者手写大量的提示词。如果使用者不熟悉英文，找一个翻译工具翻译一下也非常简单。

## 总结时刻

在这一课中，我们使用 poetry 创建了 Python 项目，在项目中安装了 MetaGPT。通过运行第一个例子验证 MetaGPT 可正常工作。然后我们阅读了 MetaGPT 官方文档的前三个部分，理解了 MetaGPT 的核心概念。最后运行了第二个例子，并研究了第二个例子的源代码。运行两个例子均使用优秀的国产开源 LLM——阿里巴巴的 qwen2.5。

MetaGPT 最初的设计就是为了支持多 Agent 协作的，堪称目前最完善的多 Agent 开发框架。吴承霖老师说他在 2023 年上半年看过大多数开源 LLM 应用开发框架（包括 LangChain），没有一个能支持多 Agent 协作，无奈之下只好自己动手，开启了一段传奇经历。

在下一课中，我们开始使用 MetaGPT 实现我们真正的需求——一个陪伴我们玩24点游戏的智能体应用。

课程代码链接：[https://gitee.com/mozilla88/autonomous\_agent/tree/master/lesson\_03](https://gitee.com/mozilla88/autonomous_agent/tree/master/lesson_03)

## 思考题

在 MetaGPT 应用中不同的 Agent (Role 的子类实例) 是如何协作的？

提示：通过在 build\_customized\_multi\_agents.py 源代码中各个 Role 和 Action 对象的函数内添加 print 或 logger 语句，搞清楚它们之间是如何协作的。

期待你的分享。如果今天的内容对你有所帮助，也期待你转发给你的同事或者朋友，大家一起学习，共同进步。我们下节课再见！
<div><strong>精选留言（11）</strong></div><ul>
<li><span>月狼葱葱</span> 👍（1） 💬（1）<p>我使用win11，在执行poetry install --no-root &amp;&amp; poetry run pip install -e &quot;..&#47;MetaGPT&quot; --config-settings editable_mode=compat，会报错： ERROR: Failed building wheel for volcengine-python-sdk，原因是文件路径过长：Windows 系统默认对文件路径长度有限制，可能会导致某些文件无法正确创建。，用win的伙伴可以参考解决方案 https:&#47;&#47;github.com&#47;geekan&#47;MetaGPT&#47;issues&#47;1677</p>2025-02-05</li><br/><li><span>种花家</span> 👍（1） 💬（1）<p>sudo npm install -g @mermaid-js&#47;mermaid-cli 之前加上 sudo apt install npm
然后检查 node -v
npm -v</p>2025-01-08</li><br/><li><span>蝈大虾</span> 👍（0） 💬（2）<p>如何配置metagpt调用通过vllm部署的qwen模型?


### 测试设置config.yaml配置如下：
llm:
  api_type: &quot;ollama&quot; 
  model: &quot;&#47;models&#47;Qwen2.5-72B-Instruct&quot;
  base_url: &quot;http:&#47;&#47;localhost:8000&#47;v1&quot;
  api_key: &quot;EMPTY&quot;
  temperature: 0

执行metagpt &quot;Create a 2048 game&quot;报错。

注：使用dashscope的qwen72b可以成功。

###vllm服务命令：
vllm serve &#47;models&#47;Qwen2.5-72B-Instruct \
              --host 0.0.0.0 \
              --port 8000 \
              --dtype bfloat16 \
              --tensor_parallel_size 2 \
              --max-num-seqs 1 \
              --enforce-eager \
              --gpu_memory_utilization 0.95 \
              --max_model_len 16384 \
              --max_seq_len_to_capture 16384 \
              --max-num-batched-tokens 16384 \
              --swap-space 8 \
              --disable-log-requests \
              --enable-auto-tool-choice \
              --tool-call-parser hermes
			  


## 该服务经测试正常调用。
from openai import OpenAI
client = OpenAI(api_key=&quot;EMPTY&quot;, base_url=&quot;http:&#47;&#47;localhost:8000&#47;v1&quot;)
model_type=&quot;&#47;models&#47;Qwen2.5-72B-Instruct&quot;
...
response = client.chat.completions.create(
    model=model_type,
    messages=messages,
    temperature=0,
    stream=True
)
...</p>2025-02-19</li><br/><li><span>yangchao</span> 👍（0） 💬（1）<p>思考题
在 build_customized_multi_agents.py 中，不同的 Agent（即 Role 的子类实例）通过消息传递和动作执行来进行协作，具体就是每个角色都有特定的职责和可以执行的操作。角色之间通过_watch机制建立联系，形成工作流。</p>2025-02-19</li><br/><li><span>欠债太多</span> 👍（0） 💬（1）<p>老师，如果推荐的配置机器没有，使用mac air替代是否可行，现在机器的配置是Apple M3 16g内存？会有哪些影响
</p>2025-02-13</li><br/><li><span>蓝天</span> 👍（0） 💬（1）<p>Linux 5.4.0-117-generic
npm ERR! argv &quot;&#47;usr&#47;bin&#47;node&quot; &quot;&#47;usr&#47;bin&#47;npm&quot; &quot;install&quot; &quot;-g&quot; &quot;@mermaid-js&#47;mermaid-cli&quot;
npm ERR! node v8.10.0
npm ERR! npm  v3.5.2
npm ERR! code EMISSINGARG

npm ERR! typeerror Error: Missing required argument #1
npm ERR! typeerror     at andLogAndFinish (&#47;usr&#47;share&#47;npm&#47;lib&#47;fetch-package-metadata.js:31:3)
npm ERR! typeerror     at fetchPackageMetadata (&#47;usr&#47;share&#47;npm&#47;lib&#47;fetch-package-metadata.js:51:22)
npm ERR! typeerror     at resolveWithNewModule (&#47;usr&#47;share&#47;npm&#47;lib&#47;install&#47;deps.js:456:12)
npm ERR! typeerror     at &#47;usr&#47;share&#47;npm&#47;lib&#47;install&#47;deps.js:190:5
npm ERR! typeerror     at &#47;usr&#47;share&#47;npm&#47;node_modules&#47;slide&#47;lib&#47;async-map.js:52:35
npm ERR! typeerror     at Array.forEach (&lt;anonymous&gt;)
npm ERR! typeerror     at &#47;usr&#47;share&#47;npm&#47;node_modules&#47;slide&#47;lib&#47;async-map.js:52:11
npm ERR! typeerror     at Array.forEach (&lt;anonymous&gt;)
npm ERR! typeerror     at asyncMap (&#47;usr&#47;share&#47;npm&#47;node_modules&#47;slide&#47;lib&#47;async-map.js:51:8)
npm ERR! typeerror     at exports.loadRequestedDeps (&#47;usr&#47;share&#47;npm&#47;lib&#47;install&#47;deps.js:188:3)
npm ERR! typeerror This is an error with npm itself. Please report this error at:
npm ERR! typeerror     &lt;http:&#47;&#47;github.com&#47;npm&#47;npm&#47;issues&gt;
</p>2025-02-11</li><br/><li><span>zshanjun</span> 👍（0） 💬（2）<p>运行报错，跑不通。老师帮忙看看

poetry run metagpt &quot;write a cli blackjack game&quot;
2025-02-10 17:24:03.997 | INFO     | metagpt.team:invest:93 - Investment: $3.0.
2025-02-10 17:24:03.998 | INFO     | metagpt.roles.role:_act:403 - Alice(Product Manager): to do PrepareDocuments(PrepareDocuments)
2025-02-10 17:24:04.062 | INFO     | metagpt.utils.file_repository:save:57 - save to: &#47;Users&#47;zhangsj190&#47;ai&#47;MetaGPT&#47;workspace&#47;20250210172403&#47;docs&#47;requirement.txt
2025-02-10 17:24:04.062 | INFO     | metagpt.roles.role:_act:403 - Alice(Product Manager): to do WritePRD(WritePRD)
2025-02-10 17:24:04.062 | INFO     | metagpt.actions.write_prd:run:86 - New requirement detected: write a cli blackjack game
2025-02-10 17:24:04.091 | ERROR    | metagpt.utils.common:log_it:554 - Finished call to &#39;metagpt.actions.action_node.ActionNode._aask_v1&#39; after 0.028(s), this was the 1st time calling it. exp: &#39;dict&#39; object has no attribute &#39;lower&#39;
2025-02-10 17:24:05.011 | ERROR    | metagpt.utils.common:log_it:554 - Finished call to &#39;metagpt.actions.action_node.ActionNode._aask_v1&#39; after 0.948(s), this was the 2nd time calling it. exp: &#39;dict&#39; object has no attribute &#39;lower&#39;
2025-02-10 17:24:15.394 | WARNING  | metagpt.utils.common:wrapper:673 - There is a exception in role&#39;s execution, in order to resume, we delete the newest role communication message in the role&#39;s memory.
2025-02-10 17:24:15.402 | ERROR    | metagpt.utils.common:wrapper:655 - Exception occurs, start to serialize the project, exp:
Traceback (most recent call last):
  File &quot;&#47;Users&#47;zhangsj190&#47;Library&#47;Caches&#47;pypoetry&#47;virtualenvs&#47;learn-metagpt-eQkDn61w-py3.10&#47;lib&#47;python3.10&#47;site-packages&#47;tenacity&#47;_asyncio.py&quot;, line 50, in __call__
    result = await fn(*args, **kwargs)
  File &quot;&#47;Users&#47;zhangsj190&#47;ai&#47;MetaGPT&#47;metagpt&#47;actions&#47;action_node.py&quot;, line 442, in _aask_v1
    parsed_data = llm_output_postprocess(
AttributeError: &#39;dict&#39; object has no attribute &#39;lower&#39;



</p>2025-02-10</li><br/><li><span>月狼葱葱</span> 👍（0） 💬（1）<p>win11用户无法使用 命令
mkdir ~&#47;.metagpt
vi ~&#47;.metagpt&#47;config2.yaml 
假设你的用户名是 username，转换命令如下
mkdir C:\Users\username\.metagpt
notepad C:\Users\username\.metagpt\config2.yaml</p>2025-02-05</li><br/><li><span>花前不枉此生壹梦</span> 👍（0） 💬（1）<p>poetry add pysocks socksio 

The currently activated Python version 3.12.3 is not supported by the project (&gt;=3.9, &lt;3.12).
Trying to find and use a compatible version. 

Poetry was unable to find a compatible version. If you have one, you can explicitly use it via the &quot;env use&quot; command.

执行的时候报错，应该怎么搞
</p>2025-01-15</li><br/><li><span>Geek_9708db</span> 👍（0） 💬（2）<p>用qwen:14b运行报错JSON parse的问题，应该是解析出了问题；换成llama3:latest好用了。这个怎么解决呢，总觉得如果不能支持大部分模型就不太好用呀</p>2025-01-10</li><br/><li><span>木法沙</span> 👍（2） 💬（0）<p>➜  learn_metagpt python3 main.py
Welcome to Blackjack!
Player&#39;s hand: (&#39;Hearts&#39;, 3)
Player&#39;s hand: (&#39;Spades&#39;, &#39;Jack&#39;)
Do you want to hit or stand? (h&#47;s): h
Player&#39;s hand: (&#39;Hearts&#39;, 3)
Player&#39;s hand: (&#39;Spades&#39;, &#39;Jack&#39;)
Bust! You lose this round.
➜  learn_metagpt python3 main.py
Welcome to Blackjack!
Player&#39;s hand: (&#39;Spades&#39;, 8)
Player&#39;s hand: (&#39;Spades&#39;, 7)
Do you want to hit or stand? (h&#47;s): s
Dealer busts! You win this round!</p>2025-01-08</li><br/>
</ul>