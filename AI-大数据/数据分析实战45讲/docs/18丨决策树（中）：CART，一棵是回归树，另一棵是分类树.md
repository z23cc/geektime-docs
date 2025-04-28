上节课我们讲了决策树，基于信息度量的不同方式，我们可以把决策树分为ID3算法、C4.5算法和CART算法。今天我来带你学习CART算法。CART算法，英文全称叫做Classification And Regression Tree，中文叫做分类回归树。ID3和C4.5算法可以生成二叉树或多叉树，而CART只支持二叉树。同时CART决策树比较特殊，既可以作分类树，又可以作回归树。

那么你首先需要了解的是，什么是分类树，什么是回归树呢？

我用下面的训练数据举个例子，你能看到不同职业的人，他们的年龄不同，学习时间也不同。如果我构造了一棵决策树，想要基于数据判断这个人的职业身份，这个就属于分类树，因为是从几个分类中来做选择。如果是给定了数据，想要预测这个人的年龄，那就属于回归树。

![](https://static001.geekbang.org/resource/image/af/cf/af89317aa55ac3b9f068b0f370fcb9cf.png?wh=624%2A259)  
分类树可以处理离散数据，也就是数据种类有限的数据，它输出的是样本的类别，而回归树可以对连续型的数值进行预测，也就是数据在某个区间内都有取值的可能，它输出的是一个数值。

## CART分类树的工作流程

通过上一讲，我们知道决策树的核心就是寻找纯净的划分，因此引入了纯度的概念。在属性选择上，我们是通过统计“不纯度”来做判断的，ID3是基于信息增益做判断，C4.5在ID3的基础上做了改进，提出了信息增益率的概念。实际上CART分类树与C4.5算法类似，只是属性选择的指标采用的是基尼系数。

你可能在经济学中听过说基尼系数，它是用来衡量一个国家收入差距的常用指标。当基尼系数大于0.4的时候，说明财富差异悬殊。基尼系数在0.2-0.4之间说明分配合理，财富差距不大。

基尼系数本身反应了样本的不确定度。当基尼系数越小的时候，说明样本之间的差异性小，不确定程度低。分类的过程本身是一个不确定度降低的过程，即纯度的提升过程。所以CART算法在构造分类树的时候，会选择基尼系数最小的属性作为属性的划分。

我们接下来详解了解一下基尼系数。基尼系数不好懂，你最好跟着例子一起手动计算下。

假设t为节点，那么该节点的GINI系数的计算公式为：

![](https://static001.geekbang.org/resource/image/f9/89/f9bb4cce5b895499cabc714eb372b089.png?wh=463%2A94)  
这里p(Ck|t)表示节点t属于类别Ck的概率，节点t的基尼系数为1减去各类别Ck概率平方和。

通过下面这个例子，我们计算一下两个集合的基尼系数分别为多少：

集合1：6个都去打篮球；

集合2：3个去打篮球，3个不去打篮球。

针对集合1，所有人都去打篮球，所以p(Ck|t)=1，因此GINI(t)=1-1=0。

针对集合2，有一半人去打篮球，而另一半不去打篮球，所以，p(C1|t)=0.5，p(C2|t)=0.5，GINI(t)=1-（0.5\*0.5+0.5\*0.5）=0.5。

通过两个基尼系数你可以看出，集合1的基尼系数最小，也证明样本最稳定，而集合2的样本不稳定性更大。

在CART算法中，基于基尼系数对特征属性进行二元分裂，假设属性A将节点D划分成了D1和D2，如下图所示：

![](https://static001.geekbang.org/resource/image/69/9a/69a90a43146898150a0de0811c6fef9a.jpg?wh=1854%2A887)  
节点D的基尼系数等于子节点D1和D2的归一化基尼系数之和，用公式表示为：

![](https://static001.geekbang.org/resource/image/10/1e/107fed838cb75df62eb149499db20c1e.png?wh=571%2A88)  
归一化基尼系数代表的是每个子节点的基尼系数乘以该节点占整体父亲节点D中的比例。

上面我们已经计算了集合D1和集合D2的GINI系数，得到：  
![](https://static001.geekbang.org/resource/image/aa/0c/aa423c65b32bded13212b7e20fb65a0c.png?wh=354%2A91)  
![](https://static001.geekbang.org/resource/image/09/77/092a0ea87aabc5da482ff8a992691b77.png?wh=366%2A80)

所以在属性A的划分下，节点D的基尼系数为：

![](https://static001.geekbang.org/resource/image/3c/f8/3c08d5cd66a8ea098c397e14f1469ff8.png?wh=635%2A83)

节点D被属性A划分后的基尼系数越大，样本集合的不确定性越大，也就是不纯度越高。

## 如何使用CART算法来创建分类树

通过上面的讲解你可以知道，CART分类树实际上是基于基尼系数来做属性划分的。在Python的sklearn中，如果我们想要创建CART分类树，可以直接使用DecisionTreeClassifier这个类。创建这个类的时候，默认情况下criterion这个参数等于gini，也就是按照基尼系数来选择属性划分，即默认采用的是CART分类树。

下面，我们来用CART分类树，给iris数据集构造一棵分类决策树。iris这个数据集，我在Python可视化中讲到过，实际上在sklearn中也自带了这个数据集。基于iris数据集，构造CART分类树的代码如下：

```
# encoding=utf-8
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_iris
# 准备数据集
iris=load_iris()
# 获取特征集和分类标识
features = iris.data
labels = iris.target
# 随机抽取33%的数据作为测试集，其余为训练集
train_features, test_features, train_labels, test_labels = train_test_split(features, labels, test_size=0.33, random_state=0)
# 创建CART分类树
clf = DecisionTreeClassifier(criterion='gini')
# 拟合构造CART分类树
clf = clf.fit(train_features, train_labels)
# 用CART分类树做预测
test_predict = clf.predict(test_features)
# 预测结果与测试集结果作比对
score = accuracy_score(test_labels, test_predict)
print("CART分类树准确率 %.4lf" % score)
```

运行结果：

```
CART分类树准确率 0.9600
```

如果我们把决策树画出来，可以得到下面的图示：

![](https://static001.geekbang.org/resource/image/c1/40/c1e2f9e4a299789bb6cc23afc6fd3140.png?wh=736%2A755)  
首先train\_test\_split可以帮助我们把数据集抽取一部分作为测试集，这样我们就可以得到训练集和测试集。

使用clf = DecisionTreeClassifier(criterion=‘gini’)初始化一棵CART分类树。这样你就可以对CART分类树进行训练。

使用clf.fit(train\_features, train\_labels)函数，将训练集的特征值和分类标识作为参数进行拟合，得到CART分类树。

使用clf.predict(test\_features)函数进行预测，传入测试集的特征值，可以得到测试结果test\_predict。

最后使用accuracy\_score(test\_labels, test\_predict)函数，传入测试集的预测结果与实际的结果作为参数，得到准确率score。

我们能看到sklearn帮我们做了CART分类树的使用封装，使用起来还是很方便的。

**CART回归树的工作流程**

CART回归树划分数据集的过程和分类树的过程是一样的，只是回归树得到的预测结果是连续值，而且评判“不纯度”的指标不同。在CART分类树中采用的是基尼系数作为标准，那么在CART回归树中，如何评价“不纯度”呢？实际上我们要根据样本的混乱程度，也就是样本的离散程度来评价“不纯度”。

样本的离散程度具体的计算方式是，先计算所有样本的均值，然后计算每个样本值到均值的差值。我们假设x为样本的个体，均值为u。为了统计样本的离散程度，我们可以取差值的绝对值，或者方差。

其中差值的绝对值为样本值减去样本均值的绝对值：

![](https://static001.geekbang.org/resource/image/6f/97/6f9677a70b1edff85e9e467f3e52bd97.png?wh=208%2A125)  
方差为每个样本值减去样本均值的平方和除以样本个数：

![](https://static001.geekbang.org/resource/image/04/c1/045fd5afb7b53f17a8accd6f337f63c1.png?wh=245%2A87)  
所以这两种节点划分的标准，分别对应着两种目标函数最优化的标准，即用最小绝对偏差（LAD），或者使用最小二乘偏差（LSD）。这两种方式都可以让我们找到节点划分的方法，通常使用最小二乘偏差的情况更常见一些。

我们可以通过一个例子来看下如何创建一棵CART回归树来做预测。

## 如何使用CART回归树做预测

这里我们使用到sklearn自带的波士顿房价数据集，该数据集给出了影响房价的一些指标，比如犯罪率，房产税等，最后给出了房价。

根据这些指标，我们使用CART回归树对波士顿房价进行预测，代码如下：

```
# encoding=utf-8
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
from sklearn.datasets import load_boston
from sklearn.metrics import r2_score,mean_absolute_error,mean_squared_error
from sklearn.tree import DecisionTreeRegressor
# 准备数据集
boston=load_boston()
# 探索数据
print(boston.feature_names)
# 获取特征集和房价
features = boston.data
prices = boston.target
# 随机抽取33%的数据作为测试集，其余为训练集
train_features, test_features, train_price, test_price = train_test_split(features, prices, test_size=0.33)
# 创建CART回归树
dtr=DecisionTreeRegressor()
# 拟合构造CART回归树
dtr.fit(train_features, train_price)
# 预测测试集中的房价
predict_price = dtr.predict(test_features)
# 测试集的结果评价
print('回归树二乘偏差均值:', mean_squared_error(test_price, predict_price))
print('回归树绝对值偏差均值:', mean_absolute_error(test_price, predict_price)) 
```

运行结果（每次运行结果可能会有不同）：

```
['CRIM' 'ZN' 'INDUS' 'CHAS' 'NOX' 'RM' 'AGE' 'DIS' 'RAD' 'TAX' 'PTRATIO' 'B' 'LSTAT']
回归树二乘偏差均值: 23.80784431137724
回归树绝对值偏差均值: 3.040119760479042
```

如果把回归树画出来，可以得到下面的图示（波士顿房价数据集的指标有些多，所以树比较大）：

![](https://static001.geekbang.org/resource/image/65/61/65a3855aed648b32994b808296a40b61.png?wh=1861%2A368)

你可以在[这里](https://pan.baidu.com/s/1RKD6-IwAzL--cL0jt4GPiQ)下载完整PDF文件。

我们来看下这个例子，首先加载了波士顿房价数据集，得到特征集和房价。然后通过train\_test\_split帮助我们把数据集抽取一部分作为测试集，其余作为训练集。

使用dtr=DecisionTreeRegressor()初始化一棵CART回归树。

使用dtr.fit(train\_features, train\_price)函数，将训练集的特征值和结果作为参数进行拟合，得到CART回归树。

使用dtr.predict(test\_features)函数进行预测，传入测试集的特征值，可以得到预测结果predict\_price。

最后我们可以求得这棵回归树的二乘偏差均值，以及绝对值偏差均值。

我们能看到CART回归树的使用和分类树类似，只是最后求得的预测值是个连续值。

## CART决策树的剪枝

CART决策树的剪枝主要采用的是CCP方法，它是一种后剪枝的方法，英文全称叫做cost-complexity prune，中文叫做代价复杂度。这种剪枝方式用到一个指标叫做节点的表面误差率增益值，以此作为剪枝前后误差的定义。用公式表示则是：

![](https://static001.geekbang.org/resource/image/6b/95/6b9735123d45e58f0b0afc7c3f68cd95.png?wh=327%2A140)  
其中Tt代表以t为根节点的子树，C(Tt)表示节点t的子树没被裁剪时子树Tt的误差，C(t)表示节点t的子树被剪枝后节点t的误差，|Tt|代子树Tt的叶子数，剪枝后，T的叶子数减少了|Tt|-1。

所以节点的表面误差率增益值等于节点t的子树被剪枝后的误差变化除以剪掉的叶子数量。

因为我们希望剪枝前后误差最小，所以我们要寻找的就是最小α值对应的节点，把它剪掉。这时候生成了第一个子树。重复上面的过程，继续剪枝，直到最后只剩下根节点，即为最后一个子树。

得到了剪枝后的子树集合后，我们需要用验证集对所有子树的误差计算一遍。可以通过计算每个子树的基尼指数或者平方误差，取误差最小的那个树，得到我们想要的结果。

## 总结

今天我给你讲了CART决策树，它是一棵决策二叉树，既可以做分类树，也可以做回归树。你需要记住的是，作为分类树，CART采用基尼系数作为节点划分的依据，得到的是离散的结果，也就是分类结果；作为回归树，CART可以采用最小绝对偏差（LAD），或者最小二乘偏差（LSD）作为节点划分的依据，得到的是连续值，即回归预测结果。

最后我们来整理下三种决策树之间在属性选择标准上的差异：

- ID3算法，基于信息增益做判断；
- C4.5算法，基于信息增益率做判断；
- CART算法，分类树是基于基尼系数做判断。回归树是基于偏差做判断。

实际上这三个指标也是计算“不纯度”的三种计算方式。

在工具使用上，我们可以使用sklearn中的DecisionTreeClassifier创建CART分类树，通过DecisionTreeRegressor创建CART回归树。

你可以用代码自己跑一遍我在文稿中举到的例子。

![](https://static001.geekbang.org/resource/image/5c/84/5cfe1151f88befc1178eca3252890f84.png?wh=820%2A472)  
最后给你留两道思考题吧，你能说下ID3，C4.5，以及CART分类树在做节点划分时的区别吗？第二个问题是，sklearn中有个手写数字数据集，调用的方法是load\_digits()，你能否创建一个CART分类树，对手写数字数据集做分类？另外选取一部分测试集，统计下分类树的准确率？

欢迎你在评论下面留言，与我分享你的答案。也欢迎点击“请朋友读”，把这篇文章分享给你的朋友或者同事，一起交流。
<div><strong>精选留言（15）</strong></div><ul>
<li><span>rainman</span> 👍（39） 💬（1）<p>对于 CART 回归树的可视化，可以先在电脑上安装 graphviz；然后 pip install graphviz，这是安装python的库，需要依赖前面安装的 graphviz。可视化代码如下：

----
from sklearn.tree import export_graphviz
import graphviz

# 参数是回归树模型名称，不输出文件。
dot_data = export_graphviz(dtr, out_file=None)
graph = graphviz.Source(dot_data)
# render 方法会在同级目录下生成 Boston PDF文件，内容就是回归树。
graph.render(&#39;Boston&#39;)
----

具体内容可以去 sklearn(https:&#47;&#47;scikit-learn.org&#47;stable&#47;modules&#47;generated&#47;sklearn.tree.export_graphviz.html)
和 graphviz(https:&#47;&#47;graphviz.readthedocs.io&#47;en&#47;stable&#47;) 看看。</p>2019-02-15</li><br/><li><span>小熊猫</span> 👍（16） 💬（1）<p>ID3：以信息增益作为判断标准，计算每个特征的信息增益，选取信息增益最大的特征，但是容易选取到取值较多的特征
C4.5：以信息增益比作为判断标准，计算每个特征的信息增益比，选取信息增益比最大的特征
CART：分类树以基尼系数为标准，选取基尼系数小的的特征
            回归树以均方误差或绝对值误差为标准，选取均方误差或绝对值误差最小的特征

练习题：
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn import tree
from sklearn.metrics import accuracy_score
import graphviz 

# 准备手写数字数据集
digits = datasets.load_digits()
# 获取特征和标识
features = digits.data
labels = digits.target
# 选取数据集的33%为测试集，其余为训练集
train_features, test_features, train_labels, test_labels = train_test_split(features, labels, test_size=0.33)
# 创建CART分类树
clf = tree.DecisionTreeClassifier()
# 拟合构造CART分类树
clf.fit(train_features, train_labels)
# 预测测试集结果
test_predict = clf.predict(test_features)
# 测试集结果评价
print(&#39;CART分类树准确率:&#39;, accuracy_score(test_labels, test_predict))
# 画决策树
dot_data = tree.export_graphviz(clf, out_file=None)
graph = graphviz.Source(dot_data)
graph.render(&#39;CART&#47;&#47;CART_practice_digits&#39;)

CART分类树准确率: 0.8636363636363636</p>2019-02-15</li><br/><li><span>jake</span> 👍（7） 💬（2）<p>
首先想问一个问题 就是在讲到基尼系数那里 有一个图那里的例子 什么D: 9个打篮球 3个不打篮球那里
那里的D的基尼系数用到了子节点归一化基尼系数之和这个方法求 请问D的基尼系数不能直接用 上面那个公式 也就是&quot;1 - [p(ck|t)]^2&quot;那个公式计算吗 我用这个公式计算出D的基尼系数为 1 - (9&#47;12 * 9&#47;12 + 3&#47;12 * 3&#47;12) = 6&#47;16。 我也想问一下上面那个同学提的这个问题</p>2019-02-24</li><br/><li><span>xfoolin</span> 👍（4） 💬（1）<p>ID3 是通过信息增益，选取信息增益最大的特征；C4.5 是通过信息增益率，选取，CART 是通过基尼系数，选取基尼系数最小的特征。


from sklearn.model_selection import train_test_split#训练集和测试集
from sklearn.metrics import mean_squared_error#二乘偏差均值
from sklearn.metrics import mean_absolute_error#绝对值偏差均值
from sklearn.datasets import load_digits#引入 digits 数据集
from sklearn.metrics import accuracy_score#测试结果的准确性
from sklearn import tree
import graphviz
#准备数据集
digits = load_digits()
#获取数据集的特征集和分类标识
features = digits.data
labels = digits.target
#随机抽取 33% 的数据作为测试集，其余为训练集
train_features,test_features,train_labels,test_labels = \
train_test_split(features,labels,test_size = 0.33,random_state = 0)
#创建 CART 分类树
clf = tree.DecisionTreeClassifier(criterion = &#39;gini&#39;)
#拟合构造分类树
clf = clf.fit(train_features,train_labels)
#用 CART 分类树做预测
test_predict = clf.predict(test_features)
#预测结果与测试集作对比
score = accuracy_score(test_labels,test_predict)
#输出准确率
print(&#39;准确率 %.4f&#39;%score)
dot_data = tree.export_graphviz(clf,out_file = None)
graph = graphviz.Source(dot_data)
#输出分类树图示
graph.view()
</p>2019-02-19</li><br/><li><span>Lee</span> 👍（3） 💬（1）<p># encoding=utf-8
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_digits

# 准备数据集
digits=load_digits()
# 获取特征集和分类标识
features = digits.data
labels = digits.target
# 随机抽取 33% 的数据作为测试集，其余为训练集
train_features, test_features, train_labels, test_labels = train_test_split(features, labels, test_size=0.33, random_state=0)
# 创建 CART 分类树
clf = DecisionTreeClassifier(criterion=&#39;gini&#39;)
# 拟合构造 CART 分类树
clf = clf.fit(train_features, train_labels)
# 用 CART 分类树做预测
test_predict = clf.predict(test_features)
# 预测结果与测试集结果作比对
score = accuracy_score(test_labels, test_predict)
print(&quot;CART 分类树准确率 %.4lf&quot; % score)

CART 分类树准确率 0.8620</p>2019-01-24</li><br/><li><span>雨先生的晴天</span> 👍（2） 💬（1）<p>scikit learn package 确实非常好用，很简洁。推荐大家也去官网看一看，请问一下怎样可以把decision tree 可视化呀？ </p>2019-01-27</li><br/><li><span>Mi compaero de armas</span> 👍（1） 💬（1）<p>老师您好，请问采用CART算法时，如果离散型属性的值不止两种还能使用CART算法吗</p>2019-11-19</li><br/><li><span>Ronnyz</span> 👍（1） 💬（1）<p>第二问
```
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_digits

digits = load_digits()

# print(digits)

features=digits.data
labels = digits.target

train_features,test_features,train_labels,test_labels=train_test_split(features,labels,test_size =0.33,random_state=0)

clf = DecisionTreeClassifier(criterion=&#39;gini&#39;)
clf=clf.fit(train_features,train_labels)

predict_labels=clf.predict(test_features)

print(&#39;分类准确度：&#39;,accuracy_score(test_labels,predict_labels))
```
分类准确度： 0.8619528619528619</p>2019-11-11</li><br/><li><span>许宇宝</span> 👍（1） 💬（1）<p>老师，看了两遍还是不明白分类的这个决策树是依据什么画出来的？</p>2019-07-18</li><br/><li><span>羊小看</span> 👍（1） 💬（1）<p>1、为什么CRAT的基尼系数比C4.5的信息增益率好呢？既然sklearn库默认用的基尼系数，应该是这个好一些吧？
2、from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.tree import DecisionTreeClassifier
#from sklearn.datasets import load_iris
from sklearn.datasets import load_digits

# 准备数据集
#iris=load_iris()
digits=load_digits()
# 获取特征集和分类标识
#features = iris.data
#labels = iris.target
features = digits.data
labels = digits.target
# 随机抽取 33% 的数据作为测试集，其余为训练集
train_features, test_features, train_labels, test_labels = train_test_split(features, labels, test_size=0.33, random_state=0)
# 创建 CART 分类树
clf = DecisionTreeClassifier(criterion=&#39;gini&#39;)
# 拟合构造 CART 分类树
clf = clf.fit(train_features, train_labels)
# 用 CART 分类树做预测
test_predict = clf.predict(test_features)
# 预测结果与测试集结果作比对
score = accuracy_score(test_labels, test_predict)
print(&quot;CART 分类树准确率 %.4lf&quot; % score)

CART 分类树准确率 0.8636</p>2019-05-10</li><br/><li><span>圆圆的大食客</span> 👍（1） 💬（1）<p>from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_digits
#准备数据集
digits=load_digits()
#获取特征集和分类标识
features = digits.data
labels = digits.target
#随机抽取33%的数据作为测试集，其余为训练集
train_features, test_features, train_labels, test_labels = train_test_split(features, labels, test_size=0.33, random_state=0)
#创建CART分类树
clf = DecisionTreeClassifier(criterion=&#39;gini&#39;)
#拟合构造CART分类树
clf = clf.fit(train_features, train_labels)
#用CART分类树做预测
test_predict = clf.predict(test_features)
#预测结果与测试集结果作对比 
score = accuracy_score(test_labels, test_predict)
print (&quot;CART分类树准确率%.4lf&quot;% score)</p>2019-02-02</li><br/><li><span>Chino</span> 👍（1） 💬（1）<p>首先想问一个问题 就是在讲到基尼系数那里 有一个图那里的例子 什么D: 9个打篮球 3个不打篮球那里
那里的D的基尼系数用到了子节点归一化基尼系数之和这个方法求 请问D的基尼系数不能直接用 上面那个公式 也就是&quot;1 - [p(ck|t)]^2&quot;那个公式计算吗  我用这个公式计算出D的基尼系数为 1 - (9&#47;12 * 9&#47;12 + 3&#47;12 * 3&#47;12) = 6&#47;16

# ID3,C4.5,CART在做节点划分的区别
# 我认为是三者的共同之处就是得出每个类别的某个属性值 然后根据这个属性值
# 来选取哪个类型当节点 因此不同之处就是这个属性值.
# ID3 根据 信息增益 判断 哪个节点的信息增益最大就当节点
# C4.5 根据 信息增益率 判断 跟ID3相似
# CART分类树 根据 基尼系数 判断 越小代表越稳定

from sklearn.datasets import load_digits
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split


digit = load_digits()

# 把数据集中的数据和结果拿出来
features = digit.data
target = digit.target

# 把数据集中的数据分33%当作测试数据 其他用来训练
train_features,test_features,train_target,test_target = train_test_split(features,target,test_size=0.33)

# 定义CART分类树
clf = DecisionTreeClassifier()

# 把训练数据弄到分类数中 构造树
clf = clf.fit(train_features,train_target)

# 把测试数据放进树中得出预测结果
predict_target = clf.predict(test_features)

# 对比预测出来的数据和实际结果
score = accuracy_score(predict_target,test_target)

print(score)

</p>2019-02-01</li><br/><li><span>Melons</span> 👍（0） 💬（1）<p>如果按照第一个公式，数据集D的基尼系数计算方法为1-（9&#47;12 * 9&#47;12 + 3&#47;12 * 3&#47;12）=3&#47;8；请问为什么不能这样算呢？</p>2021-01-14</li><br/><li><span>lemonlxn</span> 👍（0） 💬（1）<p>使用CART树，做回归或者分类，样本数据类型 有无规定，只能是 分类 或 数值 数据类型，还是两者都可以存在？</p>2020-09-21</li><br/><li><span>freedomwl</span> 👍（0） 💬（3）<p>老师，波士顿房价代码每次运行结果不一样是因为训练集和测试集每运行一次都会重新随机选取的原因吗？</p>2020-07-11</li><br/>
</ul>