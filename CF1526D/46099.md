### CF1526D Kill Anton

应该能发现交换次数是和逆序对有关的，具体来说是原串的字符在构造的串中下一次出现的位置的数组的逆序对

比如 $s=\mathrm{ANTON} \ t=\mathrm{NNOTA}$ 位置数组为 $P=[5,1,4,3,2]$ 逆序对数为 $7$ 需要交换七次

手玩或者口胡一下会发现如果相同的字符放在一起不会更劣

证明：

设我们现在找到的字符串长这样 $\mathrm{....AAA_x....A_yAA...}$ ($A_x,A_y$ 仅用于标记，这两个字符 $P$ 中分别位于 $i,j$)

考虑两种合并方式 $\mathrm{....AAA_xA_y...AA...}$ 和 $\mathrm{...AA...A_xA_yAA}$ 

考虑第一种产生/消除的逆序对数量和为 $D_1=\sum\limits_{k=i+1}^{j-1} \mathrm{sign}(P_k-P_i)$

其中 $\mathrm{sign}(x)=\dfrac{x}{|x|}$ 就是取符号

第二种为 $D_2=\sum\limits_{k=i+1}^{j-1}\mathrm{sign}(P_j-P_k)$

事实上不可能有 $D_1+D_2 < 0$

这样就意味着至少存在有某个 $k$ 使得 $\mathrm{sign}(P_k-P_i)+\mathrm{sign}(P_j-P_k) < 0$

即 $P_k < P_i $ 且 $P_k > P_j \Rightarrow P_j <P_i$ 矛盾

那么有 $D_1+D_2 \ge 0$ 意味着这两者中至少有一个大于等于 $0$ 

那么按同样的方法把所有的相同字符放在一起不会更劣

枚举放每种字符的顺序共 $4!$ 种，每种 $n\log n$ 求逆序对比较大小即可

复杂度 $O(n\log n)$

```cpp
#include <algorithm>
#include <cstring>
#include <iostream>
#include <string>
#include <vector>
using namespace std;
const int N = 100004;
string s;
int t;
int tmp[N];
long long qres;
int a[N];
void msort(int l, int r) {
    if (l >= r)
        return;
    int m = l + r >> 1;
    msort(l, m);
    msort(m + 1, r);
    int i = l, j = m + 1;
    int k = 0;
    while (i <= m && j <= r) {
        if (a[i] <= a[j])
            tmp[k++] = a[i++];
        else {
            tmp[k++] = a[j++];
            qres += 1ll * m - i + 1ll;
        }
    }
    while (i <= m)
        tmp[k++] = a[i++];
    while (j <= r)
        tmp[k++] = a[j++];
    for (i = l, k = 0; i <= r; i++, k++)
        a[i] = tmp[k];
}
int main() {
    cin >> t;
    auto turn = [&](char c) -> int {
        if (c == 'A') {
            return 1;
        }
        if (c == 'N') {
            return 2;
        }
        if (c == 'O') {
            return 3;
        }
        if (c == 'T') {
            return 4;
        }
    };
    while (t--) {
        cin >> s;
        vector<int> v[5];
        for (int i = 0; i < s.length(); i++) {
            v[turn(s[i])].push_back(i + 1);
        }
        char chr[]     = {'A', 'N', 'O', 'T'};
        string ans     = "";
        long long mmax = 0;
        do {
            qres       = 0;
            string res = "";
            int cnt    = 0;
            for (int i = 0; i < 4; i++) {
                for (auto p : v[turn(chr[i])]) {
                    res += chr[i];
                    a[++cnt] = p;
                }
            }
            msort(1, cnt);
            if (qres >= mmax) {
                mmax = qres;
                ans  = res;
            }
        } while (next_permutation(chr, chr + 4));
        cout << ans << endl;
    }
    return 0;
}
// Asusetic eru quionours
```

