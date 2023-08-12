**题目大意：**

给定 $n$ 个点 $m$ 条边，其中 $n-1$ 条边为普通边，这些普通边构成了一棵树，剩下的边为特殊边。问图中有多少简单环恰包含两条特殊边。

数据范围：$1\le n\le 2\times10^5,1\le m\le 2\times 10^5,m\ge n-1,1\le a_i,b_i\le n$ .

> 知识储备：LCA

> 题目难度：省选/USACO Platinum

**解析：**

不难发现，简单环的形式是两条特殊边加上若干树上边。如果设这两条特殊边为  $(u_1,v_1),(u_2,v_2)$ ，当且仅当树上路径 $[u_1,v_1],[u_2,v_2]$ 有边重合时才能构造出简单环。

下面思考如何对重合的树上路径进行计数。先考虑一条链上的情况，即序列上计算重合区间的数量。可以扫一遍序列，对于当前区间 $[l_i,r_i]$ ，计算其他区间的左端点被该区间包含的数量即可。

树上路径也是类似的，我们将 $[u,v]$ 拆分为 $[u,L],[v,L]$ ，其中 $L$ 为 $u,v$ 的 LCA 。这样每个 $[u,v]$ 就被拆分为了两个链上的情况，类似处理序列的方法就可以计数。

然而这可能带来一定重复：

- $[u_i,v_i],[u_j,v_j]$ 在两条链上都有交：

考虑 $[u,L],[v,L]$ 链上 $L$ 的儿子 topx, topy ，那么两条链都有交等价于 $i,j$ 的 topx, topy 相同。用 map 存有相同 topx, topy 的 $[u,v]$ 数量 $k$ ，答案减去 $\binom{k}{2}$ 即可。

- 某条链上的起始点相同：

若这个起始点有 $k$ 条链，那么会算 $k^2$ 次，我们希望算 $\binom k2$ 次，减去算多的即可。

时间复杂度：$O(n\log n)$ .

```cpp
#include <bits/stdc++.h>
#define rep(i,a,b) for(int i=(a);i<=(b);++i)
#define per(i,a,b) for(int i=(a);i>=(b);--i)
#define pii pair<int,int>
#define fi first
#define se second
#define pb push_back
#define ll long long
using namespace std;
inline int read(){
    int x=0,f=1;char ch=getchar();
    while (!isdigit(ch)){if (ch=='-') f=-1;ch=getchar();}
    while (isdigit(ch)){x=x*10+ch-48;ch=getchar();}
    return x*f;
}
const int N=2e5+5;
int n,m,fa[N][20],Log[N],dep[N];
int a[N],b[N],L[N],siz[N],sum[N];
ll ans;
vector <int> G[N];
void dfs(int u,int f){
    dep[u]=dep[f]+1,fa[u][0]=f;
    for(int x:G[u])if(x!=f)dfs(x,u);
}
int lca(int x,int y) {
	if(dep[x]<dep[y])swap(x,y);
	while(dep[x]>dep[y]){
	    x=fa[x][Log[dep[x]-dep[y]]];
	}
    if(x==y)return x;
    per(i,18,0){
        if(fa[x][i]!=fa[y][i]){
            x=fa[x][i];
            y=fa[y][i];
        }
    }
    return fa[x][0];
}
int Top(int x,int anc){
    if(x==anc)return -1;
    per(i,18,0){
        if(dep[fa[x][i]]>dep[anc])x=fa[x][i];
    }
    return x;
}
map <pii,int> mp;
void dfs2(int u,int f,int now){
    siz[u]=now;
    for(int v:G[u])if(v!=f){
        dfs2(v,u,now+sum[v]);
    }
}
void init(){
    Log[0]=-1;
    rep(i,1,n)Log[i]=Log[i>>1]+1;
    rep(i,1,18){
        rep(j,1,n)fa[j][i]=fa[fa[j][i-1]][i-1];
    }
}
int main(){
    //freopen("disrupt.in","r",stdin);
    //freopen("disrupt.out","w",stdout);
    n=read(),m=read();
    rep(i,1,n-1){
        int u=read(),v=read();
        G[u].pb(v),G[v].pb(u);
    }
    dfs(1,0);
    init();
    rep(i,n,m){
        a[i]=read(),b[i]=read();
        L[i]=lca(a[i],b[i]);
    }
    rep(i,n,m){
        int topx=Top(a[i],L[i]),topy=Top(b[i],L[i]);
        if(topx!=-1){
            sum[topx]++;
            ans-=sum[topx];
        }
        if(topy!=-1){
            sum[topy]++;
            ans-=sum[topy];
        }
        if(topx!=-1&&topy!=-1){
            if(topx>topy)swap(topx,topy);
            ans-=mp[{topx,topy}];
            mp[{topx,topy}]++;
        }
        //cout<<i<<" "<<topx<<" "<<topy<<" "<<ans<<endl;
    }
    dfs2(1,0,0);
    rep(i,n,m){
        ans+=siz[a[i]]+siz[b[i]]-2*siz[L[i]];
    }
    printf("%lld\n",ans);
    return 0;
}
```

