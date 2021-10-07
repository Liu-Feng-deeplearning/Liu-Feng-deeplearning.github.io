## Dijkstra 

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

$R_t$ 是一个例子

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
其中，x与y相邻，x



### 杀鸡用牛刀
使用 dijkstra 解决 leetcode 问题

https://leetcode-cn.com/problems/word-ladder/submissions/
```text
class Solution {
public:    
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
    // using Dijkstra 
    if(find(wordList.begin(), wordList.end(), endWord)==wordList.end()){
      return 0;
    }
    
    wordList.emplace_back(beginWord);
    int n_words = wordList.size();
    int n = beginWord.size();
    vector<vector<int>> edge(n_words, vector<int>());  // construct graph
    for (int i=0;i<n_words;i++){
      for (int j=i+1;j<n_words;j++){                
        if (diff_word(wordList[i], wordList[j], n)){
          edge[i].emplace_back(j);          
          edge[j].emplace_back(i);
        }
      }      
    }
    // Test graph    
    // for (int i=0;i<n_words;i++){
    //   cout << i<< ":";
    //   for (int j=0;j<edge[i].size();j++){
    //     cout << edge[i][j];
    //   }
    //   cout << endl;
    // }

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
    // cout<<"start and end:"<<start_idx<<" " << end_idx << endl;
    
    // unordered_set<int> S;
    // unordered_set<int> Q;  // V-S
    vector<int> d(n_words, n_words+1);    
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>> > heap;
    vector<int> used(n_words, 0);
    used[start_idx] = 1;
    int used_total = 1;
    // S.insert(start_idx);
    
    d[start_idx]=0;
    for(int j=0;j<edge[start_idx].size();j++){    
      int start_adj_idx = edge[start_idx][j];
      d[start_adj_idx] = 1;
      heap.push({d[start_adj_idx], start_adj_idx});
    }

    // for(int i=0;i<n_words-1;i++){
    //   // Q.insert(i);
    //   heap.push({d[i], i});
    // }
    
    while(heap.size()>0){                
      // Extract-MIN(Q)      
      pair<int, int> q_pair = heap.top();
      heap.pop();
      int u_idx = q_pair.second;            
      if(used[u_idx]==1){  // only for Q: G-S
        continue;
      }
      // cout<<"find u_idx:"<<u_idx<<":"<<d[u_idx]<<endl;
      
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
    return d[end_idx]+1; //  +1- begin_in_dict ;
    }

    bool diff_word(string& a, string& b, int n){      
      int dif = 0;
      for(int i=0;i<n;i++){
        if (a[i]!=b[i]){
          dif++;
          if (dif==2) return false;
        }
      }
      return true;
    }
};

```
