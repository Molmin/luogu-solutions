本文起笔于```2022.01.08```。

[P7969 [KSN2021] Self Defence](https://www.luogu.com.cn/problem/P7969) 

首先很容易可以想到一个 $O(n^3)$ 的套路 $\text{dp}$，设 $f(i,j,k,0/1)$ 表示当前处理到第 $i$ 个字符，已经有了 $j$ 个单位的权值，第 $i$ 个字符为 $\text{A/B}$，且第 $i$ 个字符所在的极长连续段长度为 $k$ 时的方案数，转移很简单，略过。

前两维难以优化掉，那只好优化掉第三维，方法是通过增加决策量来减少状态。

为了方便 $\text{dp}$ 的转移我们设 $m=M-1$，则长度为 $l$ 的连续段（全部由同一种字符组成）的权值为 $\max\{l-m,0\}$。

设 $f(i,j,0/1)$ 表示当前处理到第 $i$ 个字符，已经有了 $j$ 个单位的权值，且第 $i$ 个字符所在的极长连续段由 $\text{A/B}$ 组成，则决策时枚举上一个极长连续段（由与目前字符相反的字符 $\text{B/A}$ 组成）的末尾位置，然后转移。
$$
f(i,j,fg)=\sum_{k=0}^{i-1}f(k,j-\max\{i-k-m,0\},fg\oplus 1)
$$
然而上面的转移不完全正确，我们还要特判一下字符串子段 $str[k+1,i]$ 是否能构成由全部由 $fg$ 对应字符组成的极长连续段，于是设 $s(i,fg)$ 为 $k$ 所能取到的最小值，即为满足 $str[k+1,i]$ 为由 $fg$ 对应字符组成的极长连续段的最小的 $k$，$s$ 是很容易预处理的。

还要注意 $j-\max\{i-k-m,0\}$ 不能为负数，即 $k\ge i-m-j$，于是正确的转移方程为
$$
f(i,j,fg)=\sum_{k=\max\{s(i,fg),i-m-j\}}^{i-1}f(k,j-\max\{i-k-m,0\},fg\oplus 1)
$$
然后根据 $\max$ 取左边的，还是取右边的进行分段
$$
\begin{aligned}
f(i,j,fg)&=\sum_{k=\max\{s(i,fg),i-m-j\}}^{i-m-1}f(k,k+(m+j-i),fg\oplus 1)\\
&+\sum_{k=\max\{s(i,fg),i-m\}}^{i-1}f(k,j,fg\oplus 1)\\
\end{aligned}
$$
显然，这个转移是可以通过前缀和优化的

设
$$
\begin{aligned}
&g(i,j,fg)=\sum_{k=0}^{\min\{i,j\}}f(i-k,j-k,fg)\\
&h(i,j,fg)=\sum_{k=0}^{i}f(k,j,fg)\\
\end{aligned}
$$
再设
$$
\begin{aligned}
&b_1=\max\{s(i,fg),i-m-j\},&t_1=i-m-1\\
&b_2=\max\{s(i,fg),i-m\},&t_2=i-1\\
&x=m+j-i&\\
\end{aligned}
$$
所以有
$$
\begin{aligned}
f(i,j,fg)&=\sum_{k=b_1}^{t_1}f(k,k+x,fg\oplus 1)\\
&+\sum_{k=b_2}^{t_2}f(k,j,fg\oplus 1)\\
&=g(t_1,t_1+x,fg\oplus1)-g(b_1-1,b_1-1+x,fg\oplus1)\\
&+h(t_2,j,fg\oplus1)-h(b_2,j,fg\oplus1)
\end{aligned}
$$
而当 $b_i>t_i$ 时，对应的求和式为 $0$，这个要特判一下。

边界条件为 
$$
\begin{aligned}
&f(0,j,fg)=g(0,j,fg)=h(0,j,fg)=[j=0]\\
&f(i,j,fg)=0\quad(j>\max\{i-m,0\})\\
\end{aligned}
$$
最终答案为
$$
ans = f(N,K,0)+f(N,K,1)
$$
代码

```cpp
#include <stdio.h>
#include <algorithm>
#include <string.h>
#include <iostream>
#include <assert.h>
using namespace std;

#define re register
#define sf scanf
#define pf printf
#define nl() putchar('\n')
#define ms(x, val) memset(x, val, sizeof(x))
#define ll long long
#define db double
#define ull unsigned long long
#define _for(i, a, b) for(re int (i) = (a); (i) < (b); ++(i))
#define _rfor(i, a, b) for(re int (i) = (a); (i) <= (b); ++(i))
#define _fev(p, u) for(re int (p) = head[(u)]; (p); (p) = nex[(p)])
#define inf 0x7fffffff
#define maxn 3005
#define mod 1000000007ll

template <class T>
void print(string name, T arr[], int n, int flag = 1){
	cout<<name<<":";
	_for(i, 0, n)cout<<" "<<arr[i+flag];
	cout<<endl;
}

int rdnt(){
	re int x = 0, sign = 1;
	re char c = getchar();
	while (c < '0' || c > '9') { if (c == '-') sign = -1; c = getchar(); }
	while (c >= '0' && c <= '9') x = (x<<3) + (x<<1) + (c ^ 48), c = getchar();
	return x * sign;
}

char str[maxn];
int s[maxn][2], f[maxn][maxn][2], g[maxn][maxn][2], h[maxn][maxn][2];

bool yes(char x, char c){
	return x == '?' || x == c;
}

int cg(re int b, re int t, re int k, re int fg){
	if (b > t) return 0;
	else return (g[t][t+k][fg]+(b>0?mod-g[b-1][b-1+k][fg]:0))%mod;
}

int ch(re int b, re int t, re int j, re int fg){
	if (b > t) return 0;
	else return (h[t][j][fg]+(b>0?mod-h[b-1][j][fg]:0))%mod;
}

void print(int arr[maxn][maxn][2], int n, int k){
	_rfor(i, 0, n){
		pf("(%d):", i);
		_rfor(j, 0, k) pf(" %d|%d", arr[i][j][0], arr[i][j][1]);
		nl();
	}nl();
}

int main(){
	#ifndef ONLINE_JUDGE
	freopen("sample.in", "r", stdin);
	//freopen("sample.out", "w", stdout);
	#endif
	
	re int n = rdnt(), m = rdnt()-1, K = rdnt();
	sf("%s", str+1);
	s[0][0] = s[0][1] = 0;
	_rfor(i, 1, n){
		if (yes(str[i], 'A')) s[i][0] = yes(str[i-1], 'A')?s[i-1][0]:i-1;
		if (yes(str[i], 'B')) s[i][1] = yes(str[i-1], 'B')?s[i-1][1]:i-1;
		if (str[i] == 'A') s[i][1] = i;
		if (str[i] == 'B') s[i][0] = i;
	}
	
	re int b1, b2, t1, t2;
	f[0][0][0] = f[0][0][1] = g[0][0][0] = g[0][0][1] = h[0][0][0] = h[0][0][1] = 1;
	_rfor(i, 1, n) _rfor(j, 0, K) _rfor(fg, 0, 1){
		b1 = max(s[i][fg], i-m-j); t1 = i-m-1;
		b2 = max(s[i][fg], i-m); t2 = i-1;
		
		if (j <= max(0, i-m)) f[i][j][fg] = (cg(b1, t1, m+j-i, fg^1) + ch(b2, t2, j, fg^1))%mod;
		else f[i][j][fg] = 0;
		g[i][j][fg] = (f[i][j][fg] + (j>0?g[i-1][j-1][fg]:0))%mod;
		h[i][j][fg] = (f[i][j][fg] + h[i-1][j][fg])%mod;
	}
	
//	print(f, n, K);
//	print(g, n, K);
//	print(h, n, K);
	
	re int ans = (f[n][K][0] + f[n][K][1])%mod;
	pf("%d\n", ans);

	return 0;
}
```





