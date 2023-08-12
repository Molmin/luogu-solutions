# Codeforces 1355 A 题解

## Part1 前言

这道题是我在比赛的时候做的。当时我没想出来，就直接看了 B。然后同学说这道 A 题很水，我就回来看了看，根据他给的提示，完成了本题。

## Part2 题面

有一个数列 $a$，满足以下要求：
$$
a_{n+1}=a_{n}+\operatorname{minDigit}\left(a_{n}\right) \cdot \operatorname{maxDigit}\left(a_{n}\right)
$$
但是题目毕竟没有那么简单。它会给你一个 $a_1$ 和 $K$ ，满足 $1 \leq a_{1} \leq 10^{18}, 1 \leq K \leq 10^{16}$，让你求出 $a_K$ 的值。

## Part3 思路

这道题其实并不难，你可以先暴力看看，会发现一个神奇的结果，在不久后 $\operatorname{minDigit}\left(a_{n}\right)$ 就成了 $0$！

所以，当我们发现 $\operatorname{minDigit}\left(a_{n}\right)$ 到 $0$ 的时候，就立刻终止暴力循环，省掉后面不必要的操作。

其实就是当 $\operatorname{minDigit}\left(a_{n}\right)$ 为 $0$ 的时候 `break`，或者老老实实执行 $K$ 次。

虽然这种做法我没法证明会不会被卡掉，但至少它能 `Accepted`。

## Part4 代码

请自觉不要抄袭。

```c++
#include<iostream>
#define int long long
using namespace std;
int next(int x){
	int maxv=0,minv=10;
	int p=x;
	while(p){
		maxv=max(maxv,p%10);
		minv=min(minv,p%10);
		p/=10;
	}
	return x+maxv*minv;
}
signed main(){
	int t;
	cin>>t;
	while(t--){
		int n,k;
		cin>>n>>k;
		while(--k){
			int x=next(n);
			if(x==n)
				break;
			n=x;
		}
		cout<<n<<endl;
	}
}
```

## Part5 结语

自己连这么简单的题都不会，真是太菜了啊！

另外，这道题我曾经 `Wrong Answer ` 过一次，在此提醒：
$$
\texttt{十年 OI 一场空，不开 long long 见祖宗！}
$$