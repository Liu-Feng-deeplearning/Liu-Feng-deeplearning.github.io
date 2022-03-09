TTS 前端中 NLP 知识

一个基本的 tts 结构，一般包括了前端和后端。tts 前端是原始文本到文本特征的模块。
和后端模型相比，前端模型可能更像一个纯 NLP 问题，大多集中在文本本身的处理上。
例如文本正则/发音预测/韵律分析等。

虽然目前神经网络方案大行其道，但受限于训练数据和推理性能，传统的 nlp 方法并未完全代替。

这篇文章会重点介绍一下偏"传统"的 nlp 算法，看下怎样轻量级的处理类似的问题。

## 文本正则化

## 分词

中文这种天然没有词边界信息的语言，分词几乎是所有 NLP 问题的基础。
对于分词这一基础问题，大家研究了许多年，具体的分类和一些基础知识简介，可以参考 
https://lujiaying.github.io/posts/2018/01/Chinese-word-segmentation/

这里介绍实际工程中一种常用的方案: **基于 Uni-gram 和 Pos-Gram 的词图搜索方法**

该方案的输入，包括如下数据

- Word Uni-Gram 词典，包括词性和count

```text
word1, count1, pos1
word2, count2, pos2
...
```

- Pos-3Gram 词典(arpa format)，可选

```text
pos1, pos2, pos3, count1
pos1, pos3, pos4, count2
...
```

### 构建词图
对每个输入对句子，根据词典构建对应词图(lattice)。

每个字为一个顶点(node)，每个句子增加一个额外的开始和结束顶点(S and E)，使用边(arc)连接相邻两个词对应结尾字的顶点。一个样例如下所示:

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-03-15-lattice.png?raw=true" width="800" /></div>

每个从 S 到 E 的路径，代表了一种可能的分词。例如，红色 arc 代表了一种分词:

```text
今/天天/气/很好
```

词图结构建成后，我们给每条路径加上对应的权重。例如，根据每个词出现的频率高低，赋予权重。
(实际中常用该词出现的次数，除以所有词出现总次数再取对数作为权重值) 

这样权重越大的路径，出现几率则越高。例如:

```text
今天/天气/很好 weight = -(0.1 + 0.1 + 0.1) = -0.3
今/天天/气很好 weight = -(0.2 + 0.1 +0.2 + 0.1) = -0.3
``` 

这表明第一一个选择可能是一个更优的结果。

可以看出，**词图其实是一个 DAG (有向无环图)。**

### 搜索词图

这样，我们的任务变成了从所有S到E中，搜索一条权重最大的边。
怎样来高效的搜索呢？

对于DAG，一个显而易见的性质是，对于一条权重最优路径，其每个子路径都是权重最优。
因此我们可以使用贪心算法进行求解，即 Viterbi 算法。简单的介绍一下搜索的过程：

想象每个结点上有一个令牌(token)，该 token 上记录了以下信息：
当前最大权重(weight)，当前最大权重对应路径的上一个token所在id(pre_node_id)。
从起始点开始，依次更新token信息。一个大概的更新算法如下:

```text
token_id = 0
while (token_id != end_id):
  max_weight = 0
  for arc in all_arc:  // 遍历所有指向该 node 的 arc
    if weight(arc) > max_weight:
      max_weight = weight(arc)
      pre_node_id = get_start_node(arc)
  token(token_id) = {max_weight, pre_node_id}
  token_id ++ 
```

当我们搜索到最后的时候，可以通过每个 token 上的 pre_node_id 反向回溯，从而得到最优路径，以及其权值。

对于更精细的使用场景，可以考虑使用 ngram3 概率来计算每条边对应的 weight，算法如下:

```text
token_id = 0
while (token_id != end_id):
  node = node(token_id)
  max_weight = 0
  for arc in all_arc(node):  // 遍历所有指向该 node 的 arc
    pre_node = get_start_node(arc)
    all_pre_node_arc = all_arc(pre_node)
    for pre_arc in all_pre_node_arc:
      weight = compute_bigram(arc, pre_arc)
      ...
        
  token(token_id) = {max_weight, pre_node}
  token_id ++ 
```

当然，实际工程中，由于大规模语料对应的3gram模型对内存占用较大。使用一种更为折衷的方，采用
pos-3gram，由于不同词性的数目一般只有十几到几十个，因此，pos-3gram 会比 word-3gram小很多。
同时，分词时加入词性信息也会一定程度上提高准确率。因此，采用了如下的权重方案: 

```text
weight =  weigth(uni-gram) + weigth(pos-3gram) * alpha
```

 
### 韵律预测

一般tts 方案中，我们使用四级韵律预测。例如
```text
今天天气很好。-> 今天#2天气#1很好。#4
```
其中，数字越大表示停顿时间越长。有一些情况下，韵律标签比较容易预测。例如每句话的结尾，一定是 #4

对于有标点的情况 。！？标点后面一定是 #4, "，"末尾一定是 #3 等等。

这里一个工程里常用的做法是使用决策树。

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-03-15-tree.jpeg?raw=true" width="300" /></div>

决策树相对来说，比较适合于输入特征维度不高，且输入特征和输出有比较线性相关性的数据。

WagonTree 本质是一颗二叉树。
一般的设计结构包括 node/leave，其中 每个 node 上有一个 question，根据问题的答案流转
到对应的子结点（这里本质相当于一个 if/else 语句），直到最终到底叶子结点。每个叶子结点对应一个标签。

整个 wagonTree 的流程

est文档： https://github.com/festvox/speech_tools/blob/master/doc/estwagon.md

**WagonTree 的构建**

构建对应对特征向量，可以使用 当前词/前一词/后一词 的词性/词长度/是否为标点等特征。
通过对训练数据进行分析，构建CART树模型。可以通过Gini指数或Entropy等指标来构建一颗树。
该树最终可以写成类似如下的形式: 

```text
Question-Set
node_id fea value node_left node_rigth
... 

leave_id value
...
 
```
对于每个结点

```text
if feature[feat] == value:
  go to left node
else:
  go to right node 
```

如果使用类似的数据结构存储数据，可以按行依次将问题集存储到树当中。

```text
node{
int fea_num;
int vale;
int left_node;
int right_node;
int is_leave; 
}
```


**WagonTree 的推理**

整个推理过程如下

```text
node_id = 0 
while(true)
  if node(node_id).is_leaf:
    return node(node_id).label
  if node(node_id)==label:
    node_id = left_node
  else:
    node_id = right_node    
```

根据此过程可以看出，推理速度和树深度正相关。
因此，在性能相同的情况下，应该尽可能使得整个树深度最小。




### 中文

1. mark the english parts with /nx， mark the digits with /m

### 构建 klexicon
klexicon format: [word, pos, pron, count]
1. read all data
2. sorted by count, pos and pinyin
3.

### Wagon Tree for Prosody

https://github.com/festvox/speech_tools/blob/master/doc/estwagon.md