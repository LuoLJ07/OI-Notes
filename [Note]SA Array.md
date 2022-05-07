## 核心思想
倍增+排序（基数排序）
## Code
`SA[i]:` 表示排名为 `i` 的后缀在原串中的位置\
`rk[i]:` 表示第 `i` 个后缀的排名
```cpp
char s[N+5];int n; // n=strlen(s+1)
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
