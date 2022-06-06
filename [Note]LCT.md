>[OI WIKI](https://oi-wiki.org/ds/lct/)
## 思路简述
想通了就很简单了。\
将树划分成实链和虚链。\
每个实链使用一棵支持区间修改的 BST 维护（如 FHQtreap,LCT）\
每次操作就将对应的实链提取出来操作。
## 代码讲解
LCT 的写法网上已经有很多了，所以这里专门选取 FHQtreap 的写法。\
这里展示一个较优写法。\
[例题传送门](https://www.luogu.com.cn/problem/P3690)
```cpp
#include<bits/stdc++.h>
#define int long long
#define U(i,l,r) for(int i(l),END##i(r);i<=END##i;++i)
#define D(i,r,l) for(int i(r),END##i(l);i>=END##i;--i)
using namespace std;
inline int qr() {
    char c;bool f(1);
    while(!isdigit(c=getchar()))f=c!='-';
    int x(c^48);
    while(isdigit(c=getchar()))x=x*10+(c^48);
    return f?x:-x;
}
const int N(1e5+5);
int n,m,a[N+5];
namespace LCT {
    struct Node {
        int ls,rs,sz,rk,xs,rv,fa,ff,v;
    }nd[N+5];
    bool isroot(int x){
        return !nd[x].fa||nd[nd[x].fa].ls!=x&&nd[nd[x].fa].rs!=x;
    }
    void init() {
        srand(time(NULL));
        U(i,1,n) nd[i]=(Node) {0,0,1,rand(),a[i],0,0,0,a[i]};
    }
    void pushup(int rt) {
        nd[rt].xs=nd[nd[rt].ls].xs^nd[nd[rt].rs].xs^nd[rt].v;
        nd[rt].sz=nd[nd[rt].ls].sz+nd[nd[rt].rs].sz+1;
        if(nd[rt].ls) nd[nd[rt].ls].fa=rt;
        if(nd[rt].rs) nd[nd[rt].rs].fa=rt;
    }
    void update(int rt) {
        nd[rt].rv^=1,swap(nd[rt].ls,nd[rt].rs);
    }
    void pushdown(int rt) {
        if(nd[rt].rv) update(nd[rt].ls),update(nd[rt].rs),nd[rt].rv=0;
    }
    int t1,t2,top;
    bool st[N+5];
    int find1(int x) { 
        top=0;
        while(!isroot(x)) st[++top]=x==nd[nd[x].fa].ls,x=nd[x].fa;
        return x;
    }
    int find2(int x) {  
    x=find1(x);
        while(nd[x].ls) pushdown(x),x=nd[x].ls;
        return x;
    }
    int merge(int x,int y) {
        if(!x||!y) return x+y;
        pushdown(x),pushdown(y);
        if(nd[x].rk<nd[y].rk) return nd[x].rs=merge(nd[x].rs,y),pushup(x),x;
        return nd[y].ls=merge(x,nd[y].ls),pushup(y),y;
    }
    void split(int &x,int &y,int rt,bool tag=0) { 
        if(!top) return pushdown(rt),y=nd[rt].rs,nd[x=rt].rs=0,pushup(rt);
        bool tmp(st[top--]^tag);
        tag=nd[rt].rv,pushdown(rt);
        if(tmp) y=rt,split(x,nd[y].ls,nd[y].ls,tag);
        else x=rt,split(nd[x].rs,y,nd[x].rs,tag);
        pushup(rt);
    }
    int access(int x){
        int pre(0);
        while(x) {
            split(t1,t2,find1(x)),nd[find2(pre)].ff=0,pre=merge(t1,pre),nd[find2(t2)].ff=x,x=nd[find2(pre)].ff;
        }
        return pre;
    }
    int root(int x) {
        return find2(access(x));
    }   
    void makeroot(int x){
        update(access(x));
    }
    void link(int x,int y){ 
        if(root(x)!=root(y)) makeroot(x),nd[x].ff=y;
    }   
    void cut(int x,int y) { 
        makeroot(x),access(y),access(x),nd[y].ff=0;
    }
    int ask(int x,int y){
        return makeroot(x),nd[access(y)].xs;
    }
    void modify(int x,int y) { 
        makeroot(x),split(t1,t2,find1(x)),nd[x].v=y,merge(t1,t2); 
    }   
} ; // namespace LCT
using namespace LCT; 
signed main() {
    n=qr(),m=qr();
    U(i,1,n) a[i]=qr();
    init();
    while(m--) {
        int opt(qr()),x(qr()),y(qr());
        if(!opt) printf("%lld\n",ask(x,y));
        else if(opt==1) link(x,y);
        else if(opt==2) cut(x,y);
        else modify(x,y);
    }
    return 0;
}
```
1. `access(x)` 提取从 x 到根节点的 FHQtreap 并返回根节点：\
每次将 x 所在实链分成两部分，一部分是深度小于等于 x 的（即在路径上的），一部分是深度大于 x 的（即不在路径上的）。\
然后将它合并到当前作为答案的 FHQtreap 即可。
1. `find1(x)` 找到 x 所在的 FHQtreap 的根节点并记录路径。
1. `find2(x)` 找到 x 所在的 FHQtreap 的深度最小点。
3. `split(x)` 与普通版 FHQtreap 略有不同，是按照所记录的路径分裂的（当然按大小分裂也可以，但是这种写法常数略大）。
