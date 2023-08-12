题意：给定长度为 $n+m$ 的排列 $p$，其中 $1-n$ 位置为白色，$n+1-n+m$ 位置为黑色，每次交换一个白色位置与一个黑色位置的数，求把 $p$ 变成升序的最少操作次数。

[link to atcoder](https://atcoder.jp/contests/arc124/tasks/arc124_d) | [link to Luogu](https://www.luogu.com.cn/problem/AT_arc124_d)

看到排列，首先想建图。看到变成升序，想到了 thupc2022 那道造计算机，不过那题钦定了 $m=2$。

连边 $i \to p_i$，则一次操作交换两个异色点的指向。容易发现如果二者不在一个环中，则合并至一个环；否则，原环分裂成两个。容易发现总可以把一个环上连续一段“弧”分裂出来，要求两端弧的左端点异色。

如果 $x$ 指向异色点 $y$，对 $x,y$ 做一次操作，$y$ 就会脱离环。非纯色环一定能找到这样的点，每次减少一个颜色多于另一色的颜色，就能保证可以把所有点分开。

那么，只要把所有纯色环合并，就能通过 $n+m-cnt_{\text{环数}}$ 次操作分开所有点。

让环数多一些是好的；同时，把一对纯异色环合并起来，一次操作能减少两个纯色环，也是很贪心的选择。

设原来有 $k$ 个连通块，$a$ 个大小超过 $1$ 的纯白环，$b$ 个大小超过 $1$ 的纯黑环，则开始的合并耗费 $\max\{a,b\}$ 次操作，剩余 $k-\max\{a,b\}$ 个环，最后一步操作 $n+m-(k - \min\{a,b\})$ 次，总共 $n+m-k+2\max\{a,b\}$ 次操作。

还有一些一遍合并一遍分裂的可能最优解，这些等效到先合并再分裂，而由上面的构造能看出这是先合并再分裂时的最优解。

官方题解证明了进行 $i$ 次操作后 $f(i) = i + n + m - k_i + 2 \max\{a_i,b_i\} \geq f(i-1)$，带下标的数字意为操作 $i$ 次后的值。

感觉“先合并再分裂”这里还是比较自然的，最大化连通块的想法也属自然，但是对最优解的证明略繁琐。

```cpp
#include <cstdio>
#include <iostream>
using namespace std;
const int M = 3e5 + 5;
int n, m, p[M], k, a, b; bool col[M], vis[M];
int main(){
    scanf("%d %d", &n, &m);
    for(int i = 1; i <= n+m; i++) scanf("%d", &p[i]);
    for(int i = 1; i <= n; i++) col[i] = 1;
    for(int i = 1; i <= n+m; i++) {
        if(vis[i]) continue;
        int j = i, len = 0, cnt = 0;
        do {
            cnt += col[j]; ++len;
            vis[j] = 1; j = p[j]; 
        } while(j != i);
        ++k;
        if(cnt == 0 && len > 1) ++a;
        else if(cnt == len && len > 1) ++b;
    }
    printf("%d\n", n + m - k + 2*max(a, b));
}
```