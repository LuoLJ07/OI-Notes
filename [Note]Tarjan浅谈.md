>[OI WIKI](https://oi-wiki.org/graph/scc/)
## 简单定义
时间戳（dfn[]）：每个节点第一次被访问的顺序
搜索树：深搜形成的一棵树
追溯值（low[]）：定义为subtree(x)中的节点和通过**一条**不在搜索树上的边能够到达subtree节点的编号

为了方便说明，引入如下定义：
1. 树边（tree edge）：搜索树上的边
2. 反祖边（back edge）：指向祖先结点的边。
3. 横叉边（cross edge）：搜索的时候遇到了一个已经访问过的结点，但是这个结点并不是当前结点的祖先。
4. 前向边（forward edge）：搜索的时候遇到子树中的结点，但非树边。

## 无向图中的运用

可以发现当图为无向图有如下重要性质：**搜索树不存在横叉边**。

### 求割边
>一条边 `(x,y)` 是割边，当且仅当搜索树上 `x` 的一个子节点 `y` 满足 `dfn[x]<low[y]`。

感性理解即为 `y` 被 `x` 更前的点所访问，所以此时 `(x,y)` 为一条割边（即桥）。

```cpp
int dfn[N+5],low[N+5],cntD;
bool bridge[M*2+5];
void tarjan(int u,int pre) {
    dfn[u]=low[u]=++cntD;
    for(int i(hd[u]);i;i=e[i].nxt) {
        int v(e[i].to);
        if(!dfn[v]) {
            tarjan(v,i);
            low[u]=min(low[u],low[v]);
            if(low[v]>dfn[u])
                bridge[i]=bridge[i^1]=1;
        }else if(i!=(pre^1))
            low[u]=min(low[u],dfn[v]);
    }
} 
```

### 求割点

进行以下的分类讨论：
1. 若 `x` 非搜索树的树根，那么它若有一个子节点 `y` 满足 `dfn[x]<=low[y]`，则它是割点。
2. 若 `x` 为搜索树的树根，如果有超过 `2` 个子节点满足上述条件，那么它也是割点。简单的说，就是有两个儿子。
```cpp
int dfn[N+5],low[N+5],cntD;
bool cut[N+5];
void tarjan(int u,int root) {
    dfn[u]=low[u]=++cntD;
    int cnt(0);
    for(int i(hd[u]);i;i=e[i].nxt) {
        int v(e[i].to);
        if(!dfn[v])  {
            tarjan(v,root);
            low[u]=min(low[u],low[v]);
            if(dfn[u]<=low[v]&&u!=root) cut[u]=1;
            cnt+=u==root;
        }else low[u]=min(low[u],dfn[v]);
    }
    if(cnt>=2)cut[u]=1;
}
```

### 点双连通分量
**点双内部无割点**，但可能包含外部的割点。
```cpp
int dfn[N+5],low[N+5],cntD,cnt;
vector<int>ss[N+5];
stack<int>st;
void tarjan(int u,int root) {
    dfn[u]=low[u]=++cntD;
    if(u==root&&!hd[u]) {
        ss[++cnt].push_back(u);
        return;
    }
    st.push(u);
    for(int i(hd[u]);i;i=e[i].nxt) {
        int v(e[i].to);
        if(!dfn[v])  {
            tarjan(v,root);
            low[u]=min(low[u],low[v]);
            if(low[v]>=dfn[u]) 
                for(ss[++cnt].push_back(u);;) {
                    int t(st.top());st.pop();
                    ss[cnt].push_back(t);
                    if(t==v) break;
                }
        }else low[u]=min(low[u],dfn[v]);
    }
}
```
### 边双连通分量
关于边双的缩点，即以每个e-dcc为节点，桥为边建图。
```cpp
int col[N+5], dcc;
void dfs(int u){
    col[u]=dcc;
    for (int i(hd[u]);i;i=e[i].nxt) 
        if (!col[e[i].to]&&!bridge[i])
            dfs(e[i].to);
}
```
### 基环树找环
容易发现，即找一个 $u$ 使得有一个 $v$ 与之相连且 `dfn[u]<dfn[v]`。~~简单讲就是搜索~~
### 求LCA
过于冷门且不太实用，还是离线算法。相较之下树剖更为优秀。
挖个坑溜了。

## 2-SAT
将一个点 `x` 拆成两个点 `x0`, `x1`，连边且Tarjan缩点。\
如果 `u → v`，表示若 `u` 则 `v`。\
容易发现，要选择 `u` 推不出 `v` ，必定有 `u` 缩点后所在的强连通分量的拓扑序比  `v` 所在的强连通分量来的大。
>**而tarjan得出的强连通分量的编号是拓扑逆序。**

容易得到 2-SAT 算法全流程。

## 有向图中的运用---求强连通分量
**如果结点u是某个强连通分量在搜索树中遇到的第一个结点，那么这个强连通分量的其余结点肯定是在搜索树中以u为根的子树中**
>证明：
>假设有个结点v在该强连通分量中但是不在以u为根的子树中，那么u到v的路径中肯定有一条离开子树的边。但是这样的边只可能是横叉边或者反祖边，然而这两条边都要求指向的结点已经被访问过了，这就和u是第一个访问的结点矛盾了。
--引自**OI-WIKI**
```cpp
int dfn[N+5],low[N+5],scc[N+5],cntD,num;
bool vis[N+5];
stack<int>st;
void tarjan(int u){
    dfn[u]=low[u]=++cntD,vis[u]=1,st.push(u);
    for(int i(hd[u]);i;i=e[i].nxt) {
      int v(e[i].to);
      if(!dfn[v]) tarjan(v),low[u]=min(low[v],low[u]);
      else if(vis[v]) low[u]=min(low[u],dfn[v]);
    }
    if(low[u]==dfn[u]) for(++num;;){
      int v(st.top());st.pop();
      scc[v]=num,vis[v]=0;
      if(v==u) break;
    }
}
```
