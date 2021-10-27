---
layout: post
title: 算法导论之图论
categories: 算法与数据结构
description: 
keywords: 算法与数据结构
---

最近在二刷算法导论，特别关注平时工作能用到的部分。
总结和整理如下，梳理脉络并加深记忆。
本文摘录一些和图论相关的内容。

后续陆续更新。

## Dijkstra 算法

迪杰斯特拉算法， 本质上依然是贪心算法。

### 问题定义与解答
对于单源最短路径问题，当图内无负自环情况下，dijkstra 算法是其中一种较优的实现。基本算法为

$x = \emptyset$

```text
// DIJKSTRA method (G, w, s)

INITIALIZE(G, s)
S = {empty set}
Q = G.V
while Q != {empty set}	
    u = EXTRACT-MIN(Q)
    S = S + {u}     // update S and Q
    Q = Q - {u}     // 这行是我加的，原文没有
    for each vertex in G.adj[u]:  // 遍历 u 的所有邻边 
        RELAX(u, v, w)
```

算法内包括几个核心步骤
- INITIALIZE: 初始化，对每个顶点元素 s，有 s.d = $infinity$ 
- EXTRACT_MIN: 从剩余的顶点中，选择距离最短的。（重要，否则不能保证算法的正确性）
- update: S and Q(always = G - S)
- RELAX: delta(s, v) = min(delta(s, u)+ w(u, v), delta(s, v))

对于稀疏图，内层的for循环消耗时间极少，时间瓶颈在于Extract-min 函数。
可以使用优先队列进行优化，下给出实例。

Note: 对于稠密图，优化可能并不会更快。

### 证明

算法导论的核心就是各种算法的证明。总结整理一个简化的版本如下:

我们证明Q中每个u移到s中时候，u.d = delta(s, u).

反证，初始时刻显然成立。u是第一个不满足 u.d = delta(s, u) 条件的顶点，则比如存在 y 属于 S
满足
s-x-y-u 是最短路线。
x与y相邻，$x \in S$, $y \in Q, u \in Q$, 一定有 y = u，否则与u是最短路径的条件矛盾。

step1: 
证明 $\delta(s, y) = y.d$
首先，因为u是第一个不满足此性质的点，所以$x.d=\delta(x, s)$
x是y的前驱节点，则有 s～x 也是最短路径，$\delta(s, x) = x.d$, 从而有
，
那么，一定有 $y.d <= x.d + w(x, y) = delta(x, d) + w(x, y) = delta(y.d)$, 
（relax操作保证了第一个不等式成立）
最后一个等号成立的原因是，最短路径上的子路径也一定是最短的。

另一方面
$y.d >= delta(y, d)$

从而 step1成立。

step2: 
证明 $y = u$      
若不成立，则有 $y.d < u.d, y \in Q$, 与u是Q中 $\delta(u, s)$ 取最小值的顶点矛盾，
从而得证。

1/2 共同证明了 dijkstra 算法的正确性。


### 杀鸡用牛刀
使用 dijkstra 解决 leetcode 问题

https://leetcode-cn.com/problems/word-ladder/submissions/

**127. 单词接龙**   

字典 wordList 中从单词 beginWord 和 endWord 的 转换序列 是一个按下述规格形成的序列：

序列中第一个单词是 beginWord 。
序列中最后一个单词是 endWord 。
每次转换只能改变一个字母。
转换过程中的中间单词必须是字典 wordList 中的单词。
给你两个单词 beginWord 和 endWord 和一个字典 wordList ，找到从 beginWord 到 endWord 的 最短转换序列 中的 单词数目 。如果不存在这样的转换序列，返回 0。

输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
输出：5
解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5。

解答：

这当然是一个并不复杂的题目，bfs + 哈希即可。每次迭代将可以转换的元素加入一个hash序列，判断是否
输出词是否在hash序列内即可。双向搜索还能更快一点。

为了熟悉 dijkstra 算法，特意找只鸡用牛刀试试。

建立一个无向无环图，可以转换的两个词之间有边相连（权重为1），其余词之前无边（权重为∞）。
下大概写下过程来帮助更好的理解。

#### 建图

```text     
if(find(wordList.begin(), wordList.end(), endWord)==wordList.end()){
  return 0;  // 特殊case判断，终点词不在图内
}
    
wordList.emplace_back(beginWord);  // 把起始词加入wordlist，这样处理起来会更简单
int n_words = wordList.size();
vector<vector<int>> edge(n_words, vector<int>());  // construct graph
for (int i=0;i<n_words;i++){
  for (int j=i+1;j<n_words;j++){        
    // cout<<i<<"|"<<j<<"|"<<diff_word(wordList[i], wordList[j])<<endl;
    if (diff_word(wordList[i], wordList[j])){
      edge[i].emplace_back(j);          
      edge[j].emplace_back(i);
    }
  }      
}

// 确定图的起始节点和终止节点     
int start_idx = -1;
for(int i=0;i<n_words;i++){
  if (wordList[i]==beginWord){
    start_idx = i;
  }
}
int end_idx = -1;
for(int i=0;i<n_words;i++){
  if (wordList[i]==endWord){
    end_idx = i;
  }
}
    
unordered_set<int> S; 
unordered_set<int> Q;  // V-S
vector<int> d(n_words, n_words+1);  // for distance 
... 
...

bool diff_word(string a, string b){
  int n = a.size();
  int dif = 0;
  for(int i=0;i<n;i++){
    if (a[i]!=b[i]){
      dif++;
      if (dif==2) return false;
    }
  }
  return true;
}
```

#### 初始化

初始点

```text
S.insert(start_idx);    
for(int j=0;j<edge[start_idx].size();j++){      
  d[edge[start_idx][j]] = 1;
  d[start_idx]=0;
}

for(int i=0;i<n_words-1;i++){
  Q.insert(i);
  // heap.push({d[i], i});
}
```

#### 朴素实现

```text    
while(Q.size()>0){  
  // u = Extract-MIN(Q)
  int u_idx = -1; int min_val = n_words+1;
  for(auto it=Q.begin();it!=Q.end();it++){
    int idx_q = *it; // todo        
    if (d[idx_q]<min_val){
        min_val = d[idx_q];
        u_idx = idx_q;
        // break;            
    }
  }
  
  if (u_idx==-1 || find(S.begin(), S.end(), end_idx)!=S.end()){
    break;  // 剪枝
  }  
  
  // Update
  S.insert(u_idx);
  Q.erase(u_idx);      
  
  // RELAX for edge  
  int u_size = edge[u_idx].size();
  for (int i=0;i<u_size;i++){
    int v_idx = edge[u_idx][i];
    if (d[v_idx]>d[u_idx]+1){
      d[v_idx] = d[u_idx] + 1;                
    }
  } 

  if (d[end_idx]>n_words-1){      
    return 0; // 未联通
  } else {
    return d[end_idx] + 1;
  }    
};
```
使用优先队列（priority queue）进行 优化

```text
// 使用 vector used 代替 S, pq 代替 Q    
vector<int> d(n_words, n_words+1);    
priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>> > heap; // 大根堆
vector<int> used(n_words, 0);
used[start_idx] = 1;
int used_total = 1;

d[start_idx]=0;
for(int j=0;j<edge[start_idx].size();j++){    
  int start_adj_idx = edge[start_idx][j];
  d[start_adj_idx] = 1;
  heap.push({d[start_adj_idx], start_adj_idx});
}

while(heap.size()>0){                
  // Extract-MIN(Q)      
  pair<int, int> q_pair = heap.top();
  heap.pop();
  int u_idx = q_pair.second;            
  if(used[u_idx]==1){  // only for Q: G-S
    continue;
  }
、  
  // update for S
  used[u_idx]=1;      
  used_total++;
  
  if(used[end_idx]==1){
    break;  // 剪枝
  }
  
  // RELAX for edge
  int u_size = edge[u_idx].size();
  for (int i=0;i<u_size;i++){
    int v_idx = edge[u_idx][i];
    if (d[v_idx]>d[u_idx]+1){
      d[v_idx] = d[u_idx] + 1;                    
    }
    heap.push({d[v_idx], v_idx});     // 有重复计算   
  }            
}

if (d[end_idx]>n_words-1){      
  return 0;
}
return d[end_idx] + 1; 
}
```