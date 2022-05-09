> [A Nice Blog](https://www.luogu.com.cn/blog/daniu/wqs-er-fen)\
> [A Nice Blog](https://www.cnblogs.com/HocRiser/p/9834069.html)

## 适用问题
求恰好选 `k` 个某类物品，物品价值和的最大或最小值。

## 核心思想

画 `(i,F(i))` 的图像，发现是凸的，也就是**斜率是单调递增或单调递减**的。\
求它的极值，容易想到**二分斜率，求与凸包的切点**，然后根据情况放缩范围。\
事实上就是通过**二分斜率去掉了选 `k` 个物品的限制**。

## 实际应用

### Luogu P2619 [国家集训队]Tree I

#### 题目描述
[题目传送门](https://www.luogu.com.cn/problem/P2619)
#### 思路简述
+ 经典问法，觉得 wqs二分 可行。
+ 发现如果我们每多加一个白边的限制，那么 MST 的边权和必然成不下降状态，**满足凸性质**。
+ 可以考虑 wqs 二分，**二分每条白边加上一个值**，kruskal 进行 check 即可。
+ 此时给每条白边加上的值也就是切线的斜率。
#### 代码展示
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
int n,m,need;
namespace Graph{
	int cb,cw;
	struct Edge{
		int a,b,c,d;
	}bl[N+5],wh[N+5],e[N+5];
	bool cmp(Edge A,Edge B){
		return A.c<B.c;
	}
	void msort() {
		for(int i(1),j(1),k(1);i<=cb||j<=cw;++k) 
			if(j>cw||i<=cb&&bl[i].c<wh[j].c) e[k]=bl[i++];
			else e[k]=wh[j++];
	}
} ;
using namespace Graph;
namespace DSU {
	int f[N+5];
	int getf(int x) {
		return x==f[x]?x:f[x]=getf(f[x]);
	}
	bool judge(int x,int y) {
		return getf(x)==getf(y);
	}
	void merge(int x,int y) {
		f[getf(x)]=getf(y);
	}
	void init() {
		U(i,1,n) f[i]=i;
	}
}; // namespace DSU
int ans,res;
bool check(int x) {
	U(i,1,cw) wh[i].c+=x;
	msort(),res=0,DSU::init();
	int cnt(0);
	U(i,1,m) 
		if(!DSU::judge(e[i].a,e[i].b)) {
			DSU::merge(e[i].a,e[i].b);
			res+=e[i].c,cnt+=!e[i].d;
		}
	U(i,1,cw) wh[i].c-=x;
	return cnt>=need;
}
signed main() {
	n=qr(),m=qr(),need=qr();
	U(i,1,m) {
		int s(qr()+1),t(qr()+1),c(qr()),col(qr());
		if(col)bl[++cb]=(Edge){s,t,c,col};
		else wh[++cw]=(Edge){s,t,c,col};
	}
	stable_sort(bl+1,bl+1+cb,cmp);
	stable_sort(wh+1,wh+1+cw,cmp);
	int L(-101),R(101);
	while(L<=R){
		int mid(L+R>>1);
		if(check(mid)) L=mid+1,ans=res-need*mid;
		else R=mid-1;
	}
	printf("%d",ans);
}

```
#### 多倍经验
[Luogu P5633](https://www.luogu.com.cn/problem/P5633)
### P3620 [APIO/CTSC 2007] 数据备份
[题目传送门](https://www.luogu.com.cn/problem/P3620)
### 思路简述
+ 又是经典问法
+ 显然每添一个限制，会增加且增量变大，**满足凸性质**。
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
int n,k,s[N+5];
struct Func{
	int x,y;//值，个数 
}f[N+5][2];
bool comp(Func a,Func b) {
	return a.x<b.x||(a.x==b.x&&a.y>b.y);
}
Func min(Func a,Func b) {
	if(comp(a,b)) return a;return b;
}
int ans;
bool check(int kk) {
	f[1][1]=(Func){(int)1e9,(int)1e9};
	f[1][0]=(Func){0,0};
	U(i,2,n){
		f[i][1]=(Func){f[i-1][0].x+(s[i]-s[i-1]+kk),f[i-1][0].y+1};
		f[i][0]=min(f[i-1][1],f[i-1][0]);
	}
	Func res(min(f[n][0],f[n][1]));
	if(res.y>=k) {
		ans=res.x-kk*k;
		return true;
	}
	return false;
}
signed main() {
	n=qr(),k=qr();
	U(i,1,n) s[i]=qr();
	int L(-1e10),R(0);
	while(L<=R){
		int mid(L+R>>1);
		if(check(mid))L=mid+1;
		else R=mid-1;
	}
	printf("%d",ans);
}
```
