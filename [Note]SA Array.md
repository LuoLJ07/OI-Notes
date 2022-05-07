>[OI WIKI](https://oi-wiki.org/string/sa/)

## SA
### 思想

核心思想挺 friendly 的。\
倍增+排序（基数排序&计数排序）。\
时间复杂度 `O(nlogn)`。\
当然还有 `O(n)` 的毒瘤写法。
### Code
`SA[i]:` 表示排名为 `i` 的后缀在原串中的位置\
`rk[i]:` 表示第 `i` 个后缀的排名
```cpp
int m,sa[N+5],rk[N+5],c[N+5];
int x[N+5],y[N+5];
void getSA(const char *s,int n) {
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
	U(i,1,n)rk[sa[i]]=i;
}

```
### 解析
```cpp
U(i,n-k+1,n) y[++num]=i;U(i,1,n)if(sa[i]>k)y[++num]=sa[i]-k;
```
此时 `y[i]` 存储的是以第二关键字排序，排名为 `i` 的后缀在原串中的起始位置。\
若该后缀在原串中的位置不存在第二关键字，即 `i + k > n`, 显然排在最前面。\
接下来我们根据原先的 `SA` 推导剩下数的排名，\
如果一个排名为 `i` 的后缀可以成为第二关键字，即满足 `sa[i]>k`，那么就将它依次排在后面\
注意到此时的起始位置为 `sa[i]-k`。\
这样的做法事实上是为了避免二次计数排序，从而减小常数。

```cpp
U(i,1,m)c[i]=0;U(i,1,n)++c[x[i]];U(i,2,m)c[i]+=c[i-1];
D(i,n,1)sa[c[x[y[i]]]--]=y[i],y[i]=0;
```
**计数排序**的过程。具体地，先将桶 `c` 清空，然后按照第一关键字插入桶中，然后进行一个前缀和的求。
然后按照第二关键字的排序后的顺序，逆着进行求解。

## Height

### 定义
`height[i]=lcp(sa[i],sa[i-1])`，
即**排名**为 `i` 的后缀和**排名**为 `i+1` 的后缀的最长公共前缀长度。
### 引理
`height[rk[i]] >= height[rk[i-1]]-1`
### 证明
若第 `i` 个后缀为 `cAC`\
则第 `i+1 ` 个后缀为 `AC`\
（注：`cA` 表示 `height[rk[i]]` 对应的字符串）\
则易得第 `rk[i]-1` 个后缀为 `cAB`\
那么一定有字符串 `AB` 在 `AC` 前。\
易证。

### Code
```cpp
void getH(const char *s,int n) {
	for(int i(1),j(0);i<=n;++i){
		if(rk[i]==1) continue;
		j-=!!j;
		while(s[i+k]==s[sa[rk[i]-1]+k])++k;
		height[rk[i]]=k;
	}
}
```
