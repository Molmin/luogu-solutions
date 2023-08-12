### CF757E Bash Plays with Functions

容易发现 $f_0(x)=\sum_{d|n}[\gcd(n,\frac{n}{d})=1]$

如果要求一个数字的约数和他“剩下的”部分互质，那么说明某个质因子全部被 $d$ 包含，所以设 $n$ 有 $m$ 种质因子，每一种选或不选就是 $2^m$

$\large f_r(n)=\sum_{uv=n}\frac{f_{r-1}(u)+f_{r-1}(v)}{2}$

考虑到每个有序对 $(u,v)$ 会被算两次并且除了个 $2$ ，那么 $f_r(n)=\sum_{d|n}f_{r-1}(d)=(f_{r-1}*1)$

$f_0(x)$ 是积性的，它的卷积也是积性的，所以可以统计质因子

$\large f_r(x)=\prod\limits_{i=1}^mf_{r}(p_i^{a_i})$

$\large f_{r}(p_i^{a_i})=\sum_{d|p_i^{a_i}}f_{r-1}(d)=\sum_{k=1}^{a_i}f_{r-1}(p_i^{k})$

发现这玩意其实和 $p_i$ 是个啥没关系，只与质因子的次数之和有关，最后的 $f_0$ 只能是 $2$

进一步的，有 $f_r(p^{a_1})=f_r(p^{a_i-1})+f_{r-1}(p^{ai})$ ，就能根据幂次递推了

小trick是可以在线性筛的时候记录每个数字的最小质因子个数，就可以在$\log$的时间复杂度内计算质因子幂次了

```cpp
#include <iostream>
using namespace std;
const int N   = 1e6;
const int mod = 1e9 + 7;
int prime[N + 10], vis[N + 10], pcnt, mindiv[N + 10];
void pre() {
    mindiv[1] = 1;
    for (int i = 2; i <= N; i++) {
        if (!vis[i]) {
            prime[++pcnt] = i;
            mindiv[i]     = i;
        }
        for (int j = 1; j <= pcnt; j++) {
            if (i * prime[j] > N) {
                break;
            }
            vis[i * prime[j]]    = 1;
            mindiv[i * prime[j]] = prime[j];
            if (i % prime[j] == 0) {
                break;
            }
        }
    }
}
int f[N + 10][25];
int main() {
    pre();
    f[0][0] = 1;
    for (int i = 1; i <= 20; i++) {
        f[0][i] = 2;
    }
    for (int i = 1; i <= N; i++) {
        f[i][0] = f[i - 1][0] % mod;
        for (int j = 1; j <= 20; j++) {
            f[i][j] = (f[i][j] + (f[i][j - 1] + f[i - 1][j]) % mod) % mod;
        }
    }
    int T;
    cin >> T;
    while (T--) {
        long long ans = 1;
        int r, n;
        scanf("%d%d", &r, &n);
        while (n != 1) {
            int d = mindiv[n], cnt = 0;
            while (n % d == 0) {
                cnt++;
                n /= d;
            }
            ans = ans * f[r][cnt] % mod;
        }
        cout << ans << endl;
    }
    return 0;
}
```

