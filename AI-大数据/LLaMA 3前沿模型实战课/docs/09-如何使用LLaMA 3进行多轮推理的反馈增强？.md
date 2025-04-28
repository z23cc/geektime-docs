你好，我是Tyler！

今天，我们来聊聊如何通过反馈增强来提升LLaMA 3的多步推理能力。我会给你介绍几种反馈增强的技术，以及它们是如何有效提升智能体表现的。我们将重点讨论推理（Reasoning）、行动（Act）和反应（ReAct）这三者之间的相互作用。

## 反馈增强的形式

在探讨反馈增强之前，我们首先要理解多步推理的重要性。现代智能体系统中的推理不再是简单的单一响应，而是一个由反馈驱动的动态决策过程。通过将复杂任务分解为多个步骤，模型可以更深入地理解任务的不同方面，提升执行的效率与准确性。

### Reasoning（推理闭环）

首先我们回顾一下推理闭环，推理是LLaMA 3面对复杂任务时最关键的能力之一。我们之前提到的思维链 (CoT) 是一个非常有效的框架，用来实现分步骤的推理过程。

![图片](https://static001.geekbang.org/resource/image/77/92/77ab890e9ec4eb49f7c71ffffc30de92.png?wh=250x238 "图片来自ReAct: Synergizing Reasoning and Acting in Language Models")

通过将复杂问题分解成多个小步骤，思维链能够让模型更好地理解问题，并提升推理能力。比如说，假设模型遇到一个包含加法和减法的数学题。用思维链的方法，模型会把问题拆成：

1. 计算第一个加数和第二个加数的和。
2. 从上述结果中减去第三个数。

这样的推理方式让模型更清楚逻辑关系。比如在题目“5 + 3 - 2”中，模型首先会算出“5 + 3 = 8”，然后再算“8 - 2 = 6”。CoT明显提升了模型在复杂场景中的表现能力。

当然，尽管思维链可以提高推理能力，但在推理复杂场景时仍然会遇到挑战。比如说，处理多个变量和条件的组合推理需要更细致的推理步骤。这里，我们可以利用学习过的反馈增强和搜索增强技术，帮助模型更好地验证推理过程，持续调整推理路径，最终得到正确结论。

### Act（行动-观察闭环）

回顾完推理，我们来看一个新知识，那就是“行动”，它在多步推理中也扮演着至关重要的角色。行动是是模型实现现实目标的抓手。LLaMA 3可以通过调用外部工具执行多种任务，使其在不同情境中更加灵活与适应。

![图片](https://static001.geekbang.org/resource/image/ed/79/ed67af818fbe287fa390e695e6f7a679.png?wh=274x267 "图片来自ReAct: Synergizing Reasoning and Acting in Language Models")

比如，当模型被布置一个数据分析任务时，它可能会建议使用特定的数据分析工具。在这个过程中，模型不仅需要处理数据本身，还必须考虑与数据相关的上下文信息，例如数据来源、格式以及所需的分析方法。

不过，在这个过程中，模型可能难以明确环境信息与下一步行动之间的关系，导致行为频繁出错。为了解决这个问题，我们可以引入更多的推理思考，使模型能更好地做出决策。比如说，当模型在数据分析中发现数据不完整或模糊时，应该能够识别问题并提出补救措施，比如请求更多信息或选择其他分析工具。

### ReAct（推理-行动-观察）

ReAct方法则是把这两者结合起来的关键理念。ReAct结合了推理和行动的思想，要求模型在每次行动后观察结果，并基于观察结果进行新的推理和决策。

![图片](https://static001.geekbang.org/resource/image/f0/06/f00c542aea672334eac0323bfafbf306.png?wh=595x560 "图片来自ReAct: Synergizing Reasoning and Acting in Language Models")

LLaMA 3在多步推理中的流程大致可分为以下几个步骤：

- **推理（Thought）**：模型在执行每个行动时，基于已有知识和当前环境信息进行推理，以决定下一步的行动。
- **工具调用（Act）**：模型根据推理结果调用外部工具，并执行相关操作。
- **观察（Obs）**：模型在行动后观察结果，并分析这些结果与预期之间的差异。

通过这种循环反馈机制，模型能在实时任务中学习和调整。比如，当模型发现选用的工具未达到预期效果时，它可以快速识别并调整后续行动策略。这种灵活的适应能力使LLaMA 3能够在复杂任务中表现出更高的智能化，理解用户意图，并提供精准的帮助。

![图片](https://static001.geekbang.org/resource/image/59/e3/59f7d860c3577eac04069e73df7e0ae3.png?wh=818x699)

接下来，我将给你介绍如何开发一个经过提示语优化的ReAct智能体。

**第1步：安装依赖**

首先需要安装相关库，比如transformers和torch，用来加载LLaMA模型，以及其他必要的工具库。

```bash
pip install ollama
```

**第2步：加载 LLaMA 模型**

我们可以使用Transformers库来加载预训练的LLaMA模型，并进行微调或直接使用。下面是加载模型和分词器的代码：

```python
import ollama
import re
import httpx
```

**第3步：设计 ReAct 框架**

ReAct框架的核心是让模型生成“想法”和“行动”的交互式循环。这里设定一个简单逻辑：模型首先产生一系列推理“想法”，然后再根据这些“想法”生成相应的“行动”。

```python
class ChatBot:
    def __init__(self, system=""):
        self.messages = [{"role": "system", "content": system}] if system else []
    
    def __call__(self, message):
        self.messages.append({"role": "user", "content": message})
        response = ollama.chat(model="llama3", messages=self.messages)
        content = response['message']['content']
        self.messages.append({"role": "assistant", "content": content})
        return content

prompt = """
你将循环执行以下步骤：思考、行动、暂停、观察，并最终输出答案。
每次循环包含以下步骤：

1. **思考**：描述你对所提问题的思考过程。
2. **行动**：执行可用的操作，并返回“暂停”。
3. **观察**：记录操作执行后的结果。

可用的操作有：

- **计算**：
  例如：`calculate: 4 * 7 / 3`
  执行计算并返回结果，确保使用浮点数格式以获取小数。

- **维基百科**：
  例如：`wikipedia: 拉格朗日`
  从维基百科检索相关内容的摘要。

示例对话：

**问题**：拉格朗日和牛顿谁出生更早？
- **思考**：我应该在维基百科上查找“拉格朗日”。
- **行动**：`wikipedia: 拉格朗日`
- **暂停**

系统会返回以下内容：
- **观察**：约瑟夫·路易·拉格朗日（法语：Joseph-Louis Lagrange，1736年1月25日—1813年4月10日）。

- **思考**：我应该在维基百科上查找“牛顿”。
- **行动**：`wikipedia: 牛顿`
- **暂停**

系统会返回以下内容：
- **观察**：艾萨克·牛顿爵士 PRS MP（英语：Sir Isaac Newton，1642年12月25日—1727年3月20日）。

最终，你将输出：
- **答案**：牛顿出生更早。
""".strip()

# 优化后的正则表达式
action_re = re.compile(r'^行动:\s*`(\w+)`:\s*(.*)$', re.MULTILINE)
```

**第4步：智能体执行循环**

在实际使用中，智能体将根据用户输入生成推理“想法”，然后基于这些想法生成相应的“行动”。这个过程会在多个回合中反复执行，直到任务完成。

```python
def query(question, max_turns=5):
    bot = ChatBot(prompt)
    next_prompt = question
    for _ in range(max_turns):
        result = bot(next_prompt)
        print("AI响应：", result)
        
        # 提取行动
        actions = [action_re.match(line) for line in result.splitlines() if action_re.match(line)]
        if actions:
            action, action_input = actions[0].groups()
            if action in known_actions:
                print(f" -- 正在执行操作: {action} 参数: {action_input}")
                observation = known_actions[action](action_input)
                print("观察结果:", observation)
                next_prompt = f"观察：{observation}"
            else:
                raise Exception(f"未知操作: {action}: {action_input}")
        else:
            break

def wikipedia(query):
    response = httpx.get("https://en.wikipedia.org/w/api.php", params={
        "action": "query",
        "list": "search",
        "srsearch": query,
        "format": "json"
    }).json()
    return response["query"]["search"][0]["snippet"]

def calculate(expression):
    try:
        return eval(expression)
    except Exception as e:
        return f"计算错误: {e}"

# 定义可用操作
known_actions = {
    "wikipedia": wikipedia,
    "calculate": calculate
}

# 示例执行
if __name__ == "__main__":
    question = "万有引力常数、自自然常数和圆周率的乘积是什么？"
    query(question)
```

## 为什么 LLaMA 3 能做到这一点？

很多人可能以为，只要给模型一个 ReAct 提示词，它就能自动完成工作，似乎这种能力是天生的。其实，这是一种误解，LLaMA 3 之所以能像智能体那样工作，背后有很多设计和训练的巧妙安排。这就是我们今天要探讨的一个极其重要的问题，为什么可以用 LLaMA 3 来做智能体？ 为什么 LLaMA 3 能做到这一点？

### 多步推理与工具使用增强

在 LLaMA 3 的技术报告中，明确指出了它对多步推理的支持。报告提到，模型在训练过程中接触到了多种需要多步操作和多个工具的任务场景。这个过程通过两个关键步骤：合成数据生成和提示生成来实现。

#### 合成数据生成

Meta AI 团队为了教会 LLaMA 3 如何在任务中使用多步工具，首先生成了大量的合成数据。这些数据通过**复杂的用户提示**来实现，要求至少调用两个工具才能解决问题。合成数据可能涉及同一个工具的多次调用，也可能是不同工具的组合。这种多样化的数据为 LLaMA 3 提供了丰富的训练素材，模拟了用户在真实场景中可能遇到的复杂查询，使 LLaMA 3 能够理解和处理多步骤的问题。

#### 提示生成

Meta 团队在提示生成过程中，确保了 LLaMA 3 的训练数据中的提示词多样且具有挑战性，最大限度地提升了模型的学习能力。这些提示可能涉及复杂的计算、数据分析或信息检索等任务，覆盖了各种实际应用场景。通过这种方式，LLaMA 3 能适应并应对复杂且多变的任务需求，增强了模型的灵活性和适用性。

### 交错推理与动态调整

此外，LLaMA 3 会基于这些提示生成解决方案的训练数据，通常包括交错的推理步骤和工具调用。它的推理过程类似于 ReAct 框架，模型不仅需要选择合适的工具，还得在任务进行过程中灵活调整工具调用的顺序和方式。

也就是说，**LLaMA 3 能根据每一步的结果调整下一步的操作，而不是一味地按照固定流程**。这种能力不仅增强了模型在处理复杂任务时的逻辑一致性，也展现出极高的适应性。例如，在数据分析任务中，LLaMA 3 可能会先调用数据提取工具，然后基于提取的数据进行分析，最终生成用户需要的报告。每一步的推理都能根据上一步的反馈进行调整，确保任务的连贯性和准确性。

通过下面的图例，我们可以直观地看到 LLaMA 3 在多步推理中的表现：

![图片](https://static001.geekbang.org/resource/image/2c/2f/2ce4bcf799b2a0e01c58ca86e0bfc22f.png?wh=1352x806 "图：LLaMA 3 使用多步规划和工具推理的过程，展示了其如何动态适应任务需求")

这张图展示了 LLaMA 3 如何通过多次工具调用，根据前一步的结果动态调整接下来的操作。这种灵活应对复杂问题的能力，使其在多轮对话和实际应用场景中具备了智能体的表现。

这种能力不仅提升了模型的实用性，也为用户提供了更丰富的交互体验。因此，LLaMA 3 能在更广泛的场景中灵活使用各种工具，帮助用户解决实际问题，推动智能体技术的进一步发展。

## 小结

学到这里，我们做个总结吧。在今天的讨论中，我们深入探讨了LLaMA 3多步推理能力的提升，重点关注了反馈增强技术在实际应用中的重要性。基于反馈增强的闭环设计是智能体能够自治解决问题的关键。此外我们还学习了为何 LLaMA 3 能够在现实场景中无需定制就可以展现出出色的多步推理潜力。

这些知识是智能体技术的重要前置知识，希望大家能重点掌握。我们在后面的智能体设计中，会引入更多的闭环和工具，形成一个能够完成复杂智能行为的语言智能体。

## 思考题

请你思考一下，为什么无需任何调整就可以直接使用原生的 LLaMA 3 进行 ReAct 智能体的开发？欢迎你把你思考后的结果分享到留言区，也欢迎你把这节课的内容分享给其他朋友，我们下节课再见！
<div><strong>精选留言（2）</strong></div><ul>
<li><span>扭断翅膀的猪</span> 👍（1） 💬（0）<p>为什么无需任何调整就可以直接使用原生的 LLaMA 3 进行 ReAct 智能体的开发?

第一是大模型支持推理能力 Reasoning: The ability to generate structured reasoning traces (e.g., through Chain-of-Thought prompting).
第二是与外部工具交互的能力 Action: The ability to interact with external tools or environments to retrieve additional information or perform tasks.</p>2025-01-22</li><br/><li><span>方华Elton</span> 👍（0） 💬（0）<p>因为llama3在训练过程中训练了react场景的数据吗?通过数据工程，提高了模型推理，调用工具的能力，并且模型能够根据工具调用结果反馈判断是否结束推理。
</p>2024-12-18</li><br/>
</ul>