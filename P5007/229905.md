# P5007 DDOSvoid 的疑惑

## Description

找出树中所有 **内部元素不存在祖先后代关系** 的集合，定义一个集合的权值为集合中所有元素的权值和，再算出所有集合的权值和。

## Solution

乍一看题面好像很迷，但是我们仔细思考一下，对于两棵不同的子树，其内部的集合如何选取是互不影响的。同时我们发现如果要明确每一个集合是什么再去计算答案十分困难，于是想到考虑每个元素的贡献。

设 $f_u$ 为以 $u$ 为根的子树中的答案， $g_u$ 为以 $u$ 为根的子树中的合法集合数量， $v$ 为 $u$ 的一个儿子，得到转移方程为：

$$
f_u = f_v * (g_u + 1) + f_u * (g_v + 1);
$$

$$
g_u = g_u + g_v * (g_u + 1)
$$

值得注意的是：
- $f_u$ 和 $g_u$ 在转移过程中并不严格是上述定义，因为一个点的子树的遍历是按次序进行的，所以 $f_u$ 和 $g_u$ 其实是 **已经遍历过的子树的答案/集合数** 具体的可以看代码
- 一个点本身的权值是在他的子节点全部遍历完后才加进去的，因为要保证集合的合法性，而一旦选中了自己，那么他的子树中的节点就一定不能选了

最终答案即为 $f_1$。

>Talk is cheap, show me your code.

```cpp
#include <cstdio>

const int N = 1000005, mod = 1e8 + 7;
int n, t;
int e_cnt, heads[N], f[N], g[N];

struct Edge {
  int v, nxt;
} e[N << 1];

inline void add(int u, int v) {
  e[++e_cnt].v = v, e[e_cnt].nxt = heads[u], heads[u] = e_cnt;
  e[++e_cnt].v = u, e[e_cnt].nxt = heads[v], heads[v] = e_cnt;
}

void solve(int u, int fa) {
  for (int i = heads[u], v; i; i = e[i].nxt) {
    if ((v = e[i].v) != fa) {
      solve(v, u);
      f[u] = 1ll * f[v] * (g[u] + 1) % mod + 1ll * f[u] * (g[v] + 1) % mod, f[u] %= mod;
      g[u] = g[u] + 1ll * (g[u] + 1) * g[v] % mod, g[u] %= mod;
    }
  }
  f[u] += t ? u : 1, f[u] %= mod, ++g[u];
}

int main() {
#ifndef ONLINE_JUDGE
#ifdef LOCAL
  freopen("testdata.in", "r", stdin);
  freopen("testdata.out", "w", stdout);
#else
  freopen("P5007 DDOSvoid 的疑惑.in", "r", stdin);
  freopen("P5007 DDOSvoid 的疑惑.out", "w", stdout);
#endif
#endif

  scanf("%d%d", &n, &t);
  for (int i = 1, u, v; i < n; ++i) {
    scanf("%d%d", &u, &v);
    add(u, v);
  }
  solve(1, 1);
  printf("%d", f[1]);
  return 0;
}
```