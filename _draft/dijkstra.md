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
那么，一定有 $\delta(s, y) <= \delta(s, x) + w(x, y)$ = x.d + w(w, y) = y.d <=。

一定有 delta(x, d) + w(x, y) = delta(y.d)

y.d <= x.d + w(x, y) = delta(x, d) + w(x, y) = delta(y.d) 

ling
y.d >= delta(y, d)


===
这是因为 $\delta(s, y) <= \delta(s, x) + w(x, y)$, 若等号不成立，则存在另一条边
s～x2-y，使得等号成立，这与x的选取矛盾。
从而 step1成立。

step2: 
证明 $y = u$      
若不成立，则有 $y.d < u.d, y \in Q$, 与u是Q中 $\delta(u, s)$ 取最小值的顶点矛盾，
从而得证。

1/2 共同证明了 dijkstra 算法的正确性。


### 杀鸡用牛刀
使用 dijkstra 解决 leetcode 问题

https://leetcode-cn.com/problems/word-ladder/submissions/

d

```text
class Solution {
public:
    static bool compare(vector<int> a, vector<int> b){
      return a[1]<b[1];
    }

    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
    // using Dijkstra 
    if(find(wordList.begin(), wordList.end(), endWord)==wordList.end()){
      return 0;
    }
    // if(find(wordList.begin(), wordList.end(), beginWord)==wordList.end()){
    //   return 0;
    // }
    wordList.emplace_back(beginWord);
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
    
    unordered_set<int> S;
    unordered_set<int> Q;  // V-S
    vector<int> d(n_words, n_words+1);
    typedef pair<int, int> PII; 
    priority_queue<PII, vector<PII>, greater<PII> > heap;
    
    S.insert(start_idx);
    

    for(int j=0;j<edge[start_idx].size();j++){      
      d[edge[start_idx][j]] = 1;
      d[start_idx]=0;
    }

    for(int i=0;i<n_words-1;i++){
      Q.insert(i);
      // heap.push({d[i], i});
    }
    
    while(Q.size()>0){
      // cout<<"Q:";
      // for(auto it=Q.begin();it!=Q.end();it++){
      //   cout<<*it;
      // }
      // cout<<endl;
      
      // cout<<"S:";
      // for(auto it=S.begin();it!=S.end();it++){
      //   cout<<*it;
      // }
      // cout<<endl;

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
      // cout<<"find u_idx:"<<u_idx<<":"<<endl;

      if (u_idx==-1){
        break;
      }
      // exit(1);

      S.insert(u_idx);
      Q.erase(u_idx);      
      
      // RELAX for edge
      // vector<int> adj = edge[u_idx];
      int u_size = edge[u_idx].size();
      for (int i=0;i<u_size;i++){
        int v_idx = edge[u_idx][i];
        if (d[v_idx]>d[u_idx]+1){
          d[v_idx] = d[u_idx] + 1;
          // cout<<"updata:"<<v_idx<<"|"<<d[v_idx]<<endl;          
        }
      }

      if(find(S.begin(), S.end(), end_idx)!=S.end()){
        break;
      }
    }

    if (d[end_idx]>n_words-1){      
      return 0;
    }
    return d[end_idx] + 1;
    }

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
};
```
使用优先队列（pri queue进行 优化）

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
