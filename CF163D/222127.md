首先题面看起来恐怖，其实爆搜解决一切。

但是太纯的爆搜应该是会死的很惨的，所以我们考虑剪枝:

首先，比较容易想的剪枝:

保证 $a \geqslant b \geqslant c$,这样可以减少一些重复的搜索量(下面没用)。

接下来考虑最优化剪枝:

$bc = \frac{V}{a}\leftarrow V = abc \rightarrow a = \frac{V}{bc}$

$ab+bc+ac = a(b+c)+bc = a(b+c)+\frac{V}{a} \geqslant 2a\sqrt{bc}+\frac{V}{a} = 2a\sqrt{\frac{V}{a}}+\frac{V}{a}$

如上述式子:

若存在 $ans \leqslant 2a\sqrt{\frac{V}{a}}+\frac{V}{a}$，
就可以剪掉。

再就是很基本的可行性剪枝:

$a^3 < V,ab^2 < V$

综上这道水题就被解决了。

~~~
# include <ctime>
# include <cmath>
# include <cstdio>
# include <cstdlib>
# include <cstring>
# include <algorithm>
# include <queue>
# include <iostream>
# define LL long long

LL ansa,ansb,ansc,ans,S,p[105][65],V = 1;
int sp[105],n;

void dfs_bc(const int cur,const LL a,const LL b)
{
    if((double)b*b > V/a) return;
    else if(cur > n)
    {
        const LL c = V/a/b;
        if(a*(b+c)+b*c < ans)
        {
            ans = a*(b+c)+b*c;
            ansa = a;
            ansb = b;
            ansc = c;
        }
    }
    else 
    {
        for(int j = sp[cur]; j >= 0 ; --j)
        {
            sp[cur] -= j;
            dfs_bc(cur+1,a,b*p[cur][j]);
            sp[cur] += j;
        }
    }
}

void dfs_a(int cur,const LL a)
{
    if((double)a*a > V/a) return;
    else if(cur > n)
    {
        const int x = V/a;
        if(ans <= 2*a*sqrt(x)+x) return;
        dfs_bc(1,a,1);
    }
    else 
    {
        for(int j = sp[cur]; j >= 0 ; --j)
        {
            sp[cur] -= j;
            dfs_a(cur+1,a*p[cur][j]);
            sp[cur] += j;
        }
    }
}

int main()
{
    int T;
    for(scanf("%d",&T); T ; T--)
    {
        scanf("%d",&n);V = 1;
        for(int i = 1; i <= n ; ++i) scanf("%lld%d",&p[i][1],sp+i);
        for(int i = 1; i <= n ; ++i)
        {
            p[i][0] = 1;
            for(int j = 2; j <= sp[i] ; ++j) p[i][j] = p[i][j-1]*p[i][1];
            V *= p[i][sp[i]];
        }
        ans = ((1LL)<<(62LL));
        dfs_a(1,1);
        printf("%lld %lld %lld %lld\n",ans*2,ansa,ansb,ansc);
    }
    return 0;
}

~~~