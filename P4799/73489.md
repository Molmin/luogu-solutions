[P4799 【世界冰球锦标赛】](https://www.luogu.org/problem/P4799)

相信大家都是从@StudyingFather 神犇的题单过来的，那么这道题就应该用双向搜索解决了。而双向搜索往往也用来解决这种`N`等于三四十的搜索问题。

当你打爆搜的时候，你是否想过，多打$2^{40}$种情况，但是绝大部分都是没有用的？如果可以只搜索出一些结果，并且用这些结果来统计最优解那该多好？那么这就是折半搜索了。

1. 我们把40个数，分为前一半和后一半，算出他们能够组合出的价格并分别记录进数组里（当然已经超过`M`就算了），把其中一个数组排序。（这里是`ka`）
```cpp
M: 1000
ka: 0 100 500 600 (1500 1600 2000 2100)
kb: 0 500 1000 (1500)
```
2. 接下来后一半数里肯定是要选数的，我们从前往后遍历`kb`，就相当于遍历选后一半的所有情况，比如当`kb`遍历到`500`的时候，由于`M == 1000`，所以`ka`里小于等于500的数都可以贡献答案，因为`ka`排好序了，所以直接二分查找就可以确定小于等于`M - kb[i]`的数的个数。

```cpp
#include <bits/stdc++.h>
#define long long long
using namespace std;

int N, mid, ac, bc;
long M, ans, d[47];

vector <long> ka, kb;

void DFS(int L, int R, long sum, vector <long> &k)
{
	if (sum > M) return;
	if (L > R)
	{
		k.push_back(sum);//累计答案 
		return;
	}
	DFS(L+1, R, sum+d[L], k);
	DFS(L+1, R, sum, k);
}

int main()
{
	cin >> N >> M, mid = N >> 1;
	for (int i = 1; i <= N; i++)
		cin >> d[i];
	DFS(1, mid, 0, ka), DFS(mid+1, N, 0, kb);
	sort(ka.begin(), ka.end());
	for (int i = 0; i < kb.size(); i++)
		ans += upper_bound(ka.begin(), ka.end(), M-kb[i]) - ka.begin();
	//要注意upper_bound找的是严格大于M-kb[i]的第一个数，但是vector下标从0开始，所以刚刚好。 
	cout << ans << endl;
}
```