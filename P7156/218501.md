**题目大意：**

给 $N$ 个区间 $[l_i,r_i]$ ，问在所有区间的并中有多少个三元组 $(a,b,c)$ ，满足：

- $a,b,c$ 互不相同 .
- $a\oplus b,a\oplus c,b\oplus c\le K$ .

输出对 $10^9+7$ 取模。

数据范围：$1\le N\le 2\times 10^4,1\le K\le 10^9$ .

> 题目难度：NOI/NOI+/CTSC

**解析：**

~~没人写官方题解思路啊，那我写一手~~

首先对 $K+1$ 这样就把 $\le$ 转化成了 $<K$ 。定义 $P$ 为最小的 $2$ 的次幂使得 $P>K$ ，那么显然 $(a,b,c)$ 必须满足（否则会有 $>K$ 且不为 0 的位出现）：
$$
\lfloor\frac{a}{P}\rfloor=\lfloor\frac{b}{P}\rfloor=\lfloor\frac{c}{P}\rfloor
$$
 接下来分类讨论：

1. $\lfloor\frac{a}{P/2}\rfloor=\lfloor\frac{b}{P/2}\rfloor=\lfloor\frac{c}{P/2}\rfloor$

此时 $a\oplus b$ 在 $P/2$ 以上的位均为 $0$ ，显然满足要求，统计相等的数量并对答案加上 $\binom{cnt}{3}$ 即可。

2. 其他情况

由于 $P$ 以上的二进制位值都一样，抽屉原理得出必有两项 $a,b$ 满足 $\lfloor\frac{a}{P/2}\rfloor=\lfloor\frac{b}{P/2}\rfloor$ 。不妨考虑 $\lfloor\frac{a}{P/2}\rfloor=\lfloor\frac{b}{P/2}\rfloor>\lfloor\frac{c}{P/2}\rfloor$ 时的情况，由于显然有 $a\oplus b<P/2<K$ ，只用考虑对每个 $c$ 求所有满足 $c\oplus a<K$ 的 $a$ 。

具体地，有 $c\in[Pt+P/2,Pt+P)$ ，集合 $S$ 为 所有 $x\in[Pt,Pt+P/2)$ 且 $x\oplus c<K$ 的 $x$ ，对于该 $c$ 的答案就是 $\binom{|S|}{2}$ 。

下面考虑如何实现，首先我们将每个原区间按上述讨论拆为长度为 $P$ 的整块和不到 $P$ 的散块。对于长度为 $P$ 的块，答案为
$$
2\times\binom{P/2}{3}+P\times\binom{K-P/2}{2}
$$
接下来把散块离线处理，

```cpp
void solve(vector <pii> v){
    int p2=p/2;
    vector<pii>tot[2];
    for(auto& t:v){
        if(t.fi/p2<t.se/p2){
            tot[0].push_back({t.fi,p2-1});
            tot[1].push_back({0,t.se-p2});
        }else
            tot[t.fi/p2].push_back({t.fi%p2,t.se%p2});
    }
    rep(i,0,1){
        add(ans,c3(totlen(tot[i])));//第一类
        solve2(tot[i],tot[i^1],p2,0);//第二类
    }
}
```

第一类的答案很好统计，考虑如何处理第二类。可以将第二类划分为更小的两个区间进行计算，这样的分治最多进行 $O(\log K)$ 层，可以通过。

时间复杂度：$O(N\log K)$ .

```cpp
#include <bits/stdc++.h>
#define rep(i,a,b) for(int i=(a);i<=(b);++i)
#define pii pair<int,int>
#define fi first
#define se second
#define ll long long
using namespace std;
inline ll read(){
    ll x=0,f=1;char ch=getchar();
    while (!isdigit(ch)){if (ch=='-') f=-1;ch=getchar();}
    while (isdigit(ch)){x=x*10+ch-48;ch=getchar();}
    return x*f;
}
const int M=1e9+7,i6=166666668;
int n,k,p=1;
ll ans;
ll c2(ll x){return 1ll*x*(x-1)/2%M;}
ll c3(ll x){return 1ll*x*(x-1)%M*(x-2)%M*i6%M;}
inline void add(ll &a,const ll b){a=(a+b)%M;}
inline ll totlen(const vector<pii>&v){
    ll len=0;for(auto &t:v)len+=t.se-t.fi+1;
    return len;
}
void solve2(const vector<pii>&a,const vector<pii>&b,int block,int cur){
    //基本情况
    if(!(int)a.size())return;
    if(!(int)b.size()){
        ans=(ans+c2(cur)*totlen(a)%M)%M;
        return;
    }
    vector<pii> des{{0,block-1}};
    if (b == des) { // b = [0,block)
		cur =(cur+((block-1)&k))%M;
		ans=(ans+c2(cur)*totlen(a)%M)%M;
		return;
	}
	//继续往下分治
	vector<pii> A[2], B[2];
	auto ad = [&](vector<pii>& v,pii p) {
		p.fi=max(p.fi,0),p.se=min(p.se,block/2-1);
		if (p.fi > p.se) return;
		v.push_back(p);
	};
	for (auto& t: a) ad(A[0],t), ad(A[1],{t.fi-block/2,t.se-block/2});
	for (auto& t: b) ad(B[0],t), ad(B[1],{t.fi-block/2,t.se-block/2});
	if (k&(block/2)) {
		for(int i=0; i<2; ++i) solve2(A[i],B[i^1],block/2,(cur+totlen(B[i]))%M);
	} else {
		for(int i=0; i<2; ++i) solve2(A[i],B[i],block/2,cur);
	}
}
//处理a/P相等的散块
void solve(vector <pii> v){
    int p2=p/2;
    vector<pii>tot[2];
    for(auto& t:v){
        if(t.fi/p2<t.se/p2){
            tot[0].push_back({t.fi,p2-1});
            tot[1].push_back({0,t.se-p2});
        }else
            tot[t.fi/p2].push_back({t.fi%p2,t.se%p2});
    }
    rep(i,0,1){
        add(ans,c3(totlen(tot[i])));//第一类
        solve2(tot[i],tot[i^1],p2,0);//第二类
    }
}
int main(){
    n=read(),k=read();
    ++k;while(p<=k)p<<=1;
    k=k-p/2;
    ll sum=(p*c2(k)%M+2*c3(p/2)%M)%M;//整块答案直接统计
    map<int,vector<pii>>todo;
    rep(i,1,n){
        int l=read(),r=read();
        int LL=l/p,RR=r/p;
        if(LL!=RR){
            add(ans,sum*(RR-LL-1)%M);
            todo[LL].push_back({l%p,p-1});
            todo[RR].push_back({0,r%p});
        }else todo[LL].push_back({l%p,r%p});
    }
    for(auto& t:todo)solve(t.se);
    printf("%lld\n",ans%M);
    return 0;
}
```

