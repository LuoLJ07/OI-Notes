## 核心思想
倍增+排序（基数排序）
## Code
`SA[i]:` 表示排名为 `i` 的后缀在原串中的位置\
`rk[i]:` 表示第 `i` 个后缀的排名
```cpp
#include<bits/stdc++.h>
#define int long long
#define U(i,l,r) for(int i(l),END##i(r);i<=END##i;++i)
#define D(i,r,l) for(int i(r),END##i(l);i>=END##i;--i)
using namespace std;
namespace FOI {
	const int SZ(1<<23);unsigned char ibuf[SZ],*is(ibuf),*it(ibuf),obuf[SZ],*os(obuf);
	const int pw[]={1,10,100,1000,10000,100000,1000000,10000000,100000000,1000000000,10000000000,100000000000,1000000000000,10000000000000,100000000000000,1000000000000000,10000000000000000,100000000000000000,1000000000000000000};
	#define gc() ((is==it)&&(it=ibuf+fread(is=ibuf,1,SZ,stdin)),is==it?EOF:*is++)
	#define putdown (fwrite(obuf,1,os-obuf,stdout),os=obuf)
	#define pc(c) ((os==obuf+SZ)&&(putdown),*os++=c)
	#define judge(c) (c>='a'&&c<='z'||c>='A'&&c<='Z'||c>='0'&&c<='9')
	inline int ri(){char c;bool f(1);while(!isdigit(c=gc()))f=c!='-';int x(c^48);while(isdigit(c=gc()))x=x*10+(c^48);return f?x:-x;}
	inline void rs(char *s) {char c;while(c=gc(),!judge(c));while(judge(c))*s++=c,c=gc();s='\0';}
	inline void pi(int x){if(x<0)x=-x,pc('-');if(x>9)pi(x/10);pc(x%10^48);}
	inline void ps(char *s) {while(*s!='\0')pc(*s++);}
	inline double rf(){char c;bool f(1);while(!isdigit(c=gc()))f=c!='-';double x(c^48);while(isdigit(c=gc()))x=x*10+(c^48);if(c=='.')for(int tmp(0);isdigit(c=gc());)x+=1.00*(c^48)/pw[++tmp];return f?x:-x;}
	inline void pf(double x,int y){pi(x);if(y)pc('.'),pi((int)(x*pw[y])-((int)x*pw[y]));}
}; //namespace FOI
using namespace FOI;void Main();signed main(){Main(),putdown;return 0;}
const int N(1e6);
char s[N+5];int n;
int m,sa[N+5],rk[N+5],c[256];
int x[N+5],y[N+5];
void getSA() {
	m=255;U(i,1,n) ++c[x[i]=s[i]];U(i,1,m) c[i]+=c[i-1];D(i,n,1) sa[c[x[i]]--]=i;
	for(int k(1);k<=n;k<<=1) {
		int num(0);
		U(i,n-k+1,n) y[++num]=i;U(i,1,n)if(sa[i]>k)y[++num]=sa[i]-k;
		U(i,1,m)c[i]=0;U(i,1,n)++c[x[i]];U(i,2,m)c[i]+=c[i-1];
		D(i,n,1)sa[c[x[y[i]]]--]=y[i],y[i]=0;
		swap(x,y),x[sa[1]]=1,num=1;
		U(i,2,n)x[sa[i]]=num+=!(y[sa[i]]==y[sa[i-1]]&&(y[sa[i]+k]==y[sa[i-1]+k]));
		if(num==n)break;m=num;
	}
}
void Main() {
	rs(s+1),n=strlen(s+1);
	getSA();
	U(i,1,n) pi(sa[i]),pc(' ');
}

```
## 简要解析
```cpp
U(i,n-k+1,n) y[++num]=i;U(i,1,n)if(sa[i]>k)y[++num]=sa[i]-k;
```
此时 `y[i]` 存储的是以第二关键字排序，排名为 `i` 的后缀在原串中的起始位置。\
若该后缀在原串中的位置不存在第二关键字，即 `i + k > n`, 显然排在最前面。\
接下来我们根据原先的 `SA` 推导剩下数的排名，\
如果一个排名为 `i` 的后缀可以成为第二关键字，即满足 `sa[i]>k`，那么就将它依次排在后面\
注意到此时的起始位置为 `sa[i]-k`。

```cpp
U(i,1,m)c[i]=0;U(i,1,n)++c[x[i]];U(i,2,m)c[i]+=c[i-1];
D(i,n,1)sa[c[x[y[i]]]--]=y[i],y[i]=0;
```
**计数排序**的过程。具体地，先将桶 `c` 清空，然后按照第一关键字插入桶中，然后进行一个前缀和的求。
然后按照第二关键字的排序后的顺序，逆着进行求解。
