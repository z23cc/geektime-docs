你好，我是Tyler！

在专栏的第一节课中，我们就探讨过模型参数、训练数据和人工智能技术发展之间的密切关系。在深入研究 GPT 系列模型时，我们遇到一个关键概念——Scaling Law。

为了帮助大家更好地理解今天的内容，让我们回顾一下 Scaling Law 的核心要点。

Scaling Law 描述了模型性能与参数数量、训练数据量和计算资源之间的关系。它表明，**随着模型规模的增大、训练数据的增多以及计算资源的增加，模型的性能会相应提升。**

从参数数量来看，更大规模的参数意味着模型能够捕捉到更复杂的模式和特征。在自然语言处理任务中，像 GPT 系列这样的大型语言模型通过海量的参数，能更好地理解语言的细微差别和复杂的语义关系，从而在文本生成、机器翻译等任务上表现出色。

对于训练数据而言，质量和数量同样重要。大量高质量的数据能让模型能学习到更广泛的语言规律和常识。以 GPT 模型为例，它是在庞大的文本数据集上进行训练的，涵盖了各类主题和风格的文本，从而能够生成符合语言习惯的多样化内容。当训练数据量增加时，模型能接触到更多语言变体和知识领域。

今天我们先聚焦于第一个变量来说，即参数规模。

从理论和实践来看，扩增模型参数数量通常可以带来更好的模型表现，但现在随着深度学习的不断进步，大模型的规模在过去几年呈指数级膨胀，**当参数量级升至数千亿乃至万亿，推理阶段就变成一大瓶颈。**

## 混合专家模型的优势

在在线推理场景下，响应速度至关重要，我们往往希望一次性将所有参数加载进同一张显卡里，避免频繁的数据交换导致时延上升，用户体验下滑。

然而，当模型规模变得庞大到一定程度，显存和算力的消耗都将直线上升。即使通过数据并行或张量并行可以解决训练阶段的部分压力，也难以在推理阶段保持同样的便利：频繁的跨卡数据交互会大幅拉长推理时延，成本也会迅速飙升。

更何况，一旦商业化落地，日益增长的用户请求量会进一步扩大算力开支。

这时候，**如果依旧沿用传统稠密模型“所有参数全程激活、整块加载”的思路，就会出现效率低、耗费高、难扩容的困境。**

在这一背景下，**“分而治之”的混合专家模型便开始受到广泛关注。**

它的思路是：与其让一个超大的稠密网络对所有 token 做统一计算，不如把网络拆分为多个“专家模块”，通过门控机制在推理过程中只启用与该 token 最相关的模块。这样就能实现“稀疏激活”：大模型整体依然保持巨量参数，但每一次推理需要真正访问和计算的参数规模被极大缩减。

**DeepSeek V3 正是这一思路的典型应用。**通过稀疏化计算，它能够在有限的计算资源下继续享受 Scaling Law 带来的增长优势，推动开源生态加速发展，使更多公司有能力构建能与 OpenAI 竞争的大模型。

不难看出， DeepSeek R1 能取得突破，其背后的“核心大脑”——DeepSeek V3 功不可没。

并且，DeepSeek V3 进一步优化了“稀疏激活”这一机制，接下来我们深入看看 DeepSeek V3 使用的架构。

## DeepSeek V3 使用的 MoE 架构

在 DeepSeek V3 的每一层，都存在多个专家节点，其中包含两大类：路由专家（图中蓝色）和共享专家（图中绿色）。  
![](https://static001.geekbang.org/resource/image/ba/a1/ba8db4bd1fedc6d763abc0ab0f6732a1.png?wh=1314x1060 "DeepSeekV3 架构图，图片来源 DeepSeek-V3 Technical Report")

路由专家的主要功能是根据输入 token 的特征来选择激活的参数。

具体而言，当一个 token 进入该层时，会先经过一个门控机制（Gate），通过计算与不同路由专家的分数，选出得分最高的若干个专家进行计算，这种方式也被称作 Top-K 路由。

由于每次只激活部分路由专家，模型虽然规模庞大，但单次推理不必激活所有参数，单卡显存上限要求和计算开销因此大幅降低。

之所以能够实现“专才导向”的稀疏激活，是因为不同路由专家通常针对各自擅长的领域或功能进行了优化，比如自然语言理解、数学推理、代码生成等。

当 token 被分配到合适的专家节点后，会在该专家完成相应的计算，随后再与全局网络进行融合。这样，每个 token 只需依托最适合它的专家，而非从头到尾激活全量参数，就达到了稠密模型推理类似的目的。

与此同时，DeepSeek V3 还设置了一部分共享专家 (Shared Experts)，以获取和整合不同上下文或任务场景下的共通知识。这些共享专家在模型内部相当于“公共模块”，帮助减少那些原本可能重复出现在路由专家中的冗余参数。

**通过把共享知识集中到这些共享专家中，既能提升模型对广泛背景信息的适应能力，也能进一步节省路由专家间的重复开销。**

如果我们把传统稠密模型比作“所有人都排队到同一个大厅去办理各种业务”，MoE 架构就像是给大厅划分出若干独立的“专业柜台”，并设立了一个“门控调度中心”指导每位来访者去最合适的柜台办理业务。

这样既能提高办理效率，也节省了资源，使模型能够在相同或有限的硬件条件下容纳更多参数，继续享受大规模带来的性能提升。

## 负载均衡与计算效率

**当模型被拆分为多个专家节点后，如何合理分配计算负载成为新的挑战。**

如果路由机制设计不合理，可能会导致部分专家被过度调用，而其他专家长期处于闲置状态，导致计算资源分布不均衡。

比如，在推理过程中，如果多个 token 都被路由到相同的专家节点，而其他专家几乎没有任务，那么这些热门专家的计算压力会急剧上升，导致推理速度下降，而未被调用的专家则形成资源浪费。

更严重的是，如果某些专家在训练过程中也长期处于低使用状态，它们的参数可能得不到充分优化，影响整体模型的泛化能力和稳定性。因此，**如何确保专家节点的合理利用，避免计算负载的极端倾斜，是混合专家模型落地应用时必须解决的问题。**

DeepSeek V3 通过在门控网络中引入可学习的偏置项，使训练过程能够自适应地调整专家的选择概率，优化 token 在推理阶段的分配方式。

具体来说，在模型训练时，门控网络会学习每个专家的工作负载，并通过引入轻微的偏置，让某些使用率较低的专家获得更多机会被选中参与计算，从而避免所有计算任务集中到少数几个专家上。

这种方法不仅能够平衡各个专家的计算压力，还能在长期训练过程中，让所有专家节点都能获得充分的参数优化，确保它们在实际推理任务中都能发挥作用。

## 小结

学到这里，我们做个总结吧。DeepSeek V3 这一系列优化策略的核心在于“稀疏激活”机制，它不仅减少了单次推理所需的计算量，还赋予了大模型更强的适应性和可扩展性。

对于深度学习研究者和产业应用而言，这意味着即使在单卡显存受限的情况下，也可以部署更大规模的模型，同时保持较高的推理效率。

相比传统稠密模型，混合专家模型的动态路由机制使计算资源的使用更加灵活，使得开发者可以在有限的计算资源下，仍然能够运行参数量更大的模型，不必因计算成本或硬件限制而受制于推理效率的下降。

混合专家模型通过门控网络实现动态路由，使每个输入 token 仅调用最相关的专家进行计算，减少了推理时的资源消耗。这种“稀疏激活”机制不仅降低了显存占用，还提升了特定任务的适应能力和泛化效果。

DeepSeek V3 进一步优化了这一机制，在门控网络中引入可学习的偏置项，避免计算负载集中在少数专家上，实现更均衡的计算资源分配，提升整体推理效率。

DeepSeek R1 能在行业内表现突出，很大程度上得益于 DeepSeek V3 在推理阶段的高效调度与灵活架构。

好了，今天的分享就到这里。希望通过这次讲解，你对混合专家模型的背景动机、核心原理以及实际应用有了更清晰的认识。

接下来，我会继续围绕着 Scaling Law，解析 DeepSeek V3 在训练数据和训练方法上的优化策略，探讨如何利用大模型推理框架进一步提升在线推理效率，并深入剖析 DeepSeek R1 如何突破传统大模型的封闭形态，让更多开发者受益于这一技术变革。

## 思考题

1. 为什么要在大模型中采用动态路由机制而不是简单地继续堆叠参数？
2. 当某些专家因为任务量集中而反复被调用时，系统会面临哪些具体挑战，该如何利用负载均衡来缓解？

带着这些问题深入研究技术细节或尝试在项目中实践，有助于你更好地理解混合专家模型的内在价值和实现要领。