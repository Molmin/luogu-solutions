47min 时过 C 降智 50min 做不出 D。果然晚上容易降智。

题意不想复述，好长。

[link to CF](https://codeforces.com/problemset/problem/1770/D) | [link to Luogu](https://www.luogu.com.cn/problem/CF1770D)

合理猜测留给后手的两个数字必须相等。证明为若不相等，则后手可以把最终的数列换掉一个数字，一定能打破排列的性质。

因此若 $a_i=b_i$，则 $c_i$ 可以随便取；否则 $c_i=a_i$ 或 $c_i=b_i$。则方案数等于在只看 $a_i \neq b_i$ 的选择中取出一个排列（预先取出 $a_i=b_i$ 的数字）的方案数，再乘上 $n^{\text{自由元的个数}}$。

注意到这个规则挺弱，可以交换任意两条的顺序，也可以交换 $a_i$ 和 $b_i$，可以建个无向图来显示这个性质。

然后赛场降智开始写大分讨。调不出来，存在解判不存在。

考虑观察图的性质，这是带自环基环树森林。单拎一个基环树出来看看，则需要把每条边（除自环）定向让每个点入度恰为 $1$。

对于非环上的边，定向方向是唯一的，即远离环的方向；而环上的边有两种定向方法，除非是自环。

至此，逐一看每个连通块，若非基环树则无解；否则，判断是否有自环，无自环则对答案贡献 $2$，否则贡献 $n$。

```cpp
#include <cstdio>
#include <vector>
using namespace std;
const int M = 1e5 + 5, mod = 998244353;
int fa[M], n, sm[M], ans, siz[M], ec[M];
void init(int n) {
    for(int i = 1; i <= n; i++)
        fa[i] = i, sm[i] = 0, siz[i] = 1, ec[i] = 0;
}
int find(int x) {return fa[x] == x ? x : fa[x] = find(fa[x]);}
void merge(int u, int v) {
    u = find(u); v = find(v);
    if(u != v) fa[u] = v, sm[v] += sm[u], siz[v] += siz[u];
}
int main() {
    int t; scanf("%d", &t);
    while(t--) {
        int n; scanf("%d", &n);
        vector<int> a(n), b(n);
        init(n);
        for(int i = 0; i < n; i++) scanf("%d", &a[i]);
        for(int i = 0; i < n; i++) scanf("%d", &b[i]);
        int cnt = 0, cnt2 = 0; 
        for(int i = 0; i < n; i++) {
            if(a[i] == b[i]) ++cnt, sm[a[i]] = 1;
        }
        for(int i = 0; i < n; i++) merge(a[i], b[i]);
        for(int i = 0; i < n; i++) {
            ++ec[find(a[i])];
        } 
        int ans = 1;
        for(int i = 1; i <= n; i++) {
            if(fa[i] != i) continue;
            if(ec[i] != siz[i]) ans = 0;
            else if(sm[i] > 0) ans = 1ll * ans * n % mod;
            else ans = 1ll * ans * 2 % mod;
        }
        printf("%d\n", ans);
    }
}
```