**题目大意：**

给一棵 $N$ 个点的树和 $M$ 条特殊边，特殊边有边权。希望求出去掉树上任意一条边后，加一条特殊边使图连通的最小代价。

数据范围：$2\le N\le 5\times 10^4,M\le 5\times10^4$ .

> 知识储备：并查集，倍增LCA

> 题目难度：省选/USACO Platinum

**解析：**

*（这道题稍为自然的想法是考虑特殊边能给哪些边的答案带来贡献，并用树剖+线段树维护最小值，这种做法的复杂度是 $O(N\log^2 N)$ 的，可以通过本题。 这里给出一个更为精妙的离线做法。）*

对特殊边按边权升序排序。考虑特殊边 $(u,v)$ 能成为哪些树上边的答案。去掉一条边加上 $(u,v)$ 后原图仍为树，不难发现去掉的这条边必须在 $[u,v]$ 的树上路径上。其次，当这条树边的答案已经确定后，不需要再考虑这条边断开的情况，于是可以用并查集合并两个端点。

实现比较简单，预处理倍增 LCA ，从小到大遍历特殊边，对树上边更新答案即可。

时间复杂度：$O(M\alpha(n))$ .

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
int n,m,fa[N][20],Log[N],pos[N],dep[N];
int dsu[N],ans[N];
int find(int x){return x==dsu[x]?x:dsu[x]=find(dsu[x]);}
struct Edge{
    int u,v,w;
    bool operator <(const Edge a)const{
        return w<a.w;
    }
}e[N];
vector <pii> G[N];
void dfs(int u,int f){
    dep[u]=dep[f]+1,fa[u][0]=f;
    for(auto x:G[u]){
        if(x.fi==f)continue;
        pos[x.se]=x.fi;
        dfs(x.fi,u);
    }
}
int lca(int x,int y) {
	if(dep[x]<dep[y])swap(x,y);
	while(dep[x]>dep[y]){
	    x=fa[x][Log[dep[x]-dep[y]]];
	}
    if(x==y)return x;
    for(int i=Log[dep[x]];i>=0;i--){
        if(fa[x][i]!=fa[y][i]){
            x=fa[x][i];
            y=fa[y][i];
        }
    }
    return fa[x][0];
}
void init(){
    Log[0]=-1;
    rep(i,1,n){
        Log[i]=Log[i>>1]+1;
        ans[i]=-1;
        dsu[i]=i;
    }
    rep(i,1,16){
        rep(j,1,n)fa[j][i]=fa[fa[j][i-1]][i-1];
    }
    sort(e+1,e+m+1);
}
int main(){
    //freopen("disrupt.in","r",stdin);
    //freopen("disrupt.out","w",stdout);
    n=read(),m=read();
    rep(i,1,n-1){
        int u=read(),v=read();
        G[u].pb({v,i}),G[v].pb({u,i});
    }
    dfs(1,0);
    rep(i,1,m){
        e[i]={read(),read(),read()};
    }
    init();
    rep(i,1,m){
        int u=e[i].u,v=e[i].v,w=e[i].w;
        int L=lca(u,v);
        for(u=find(u);dep[u]>dep[L];u=find(fa[u][0])){
            ans[u]=w,dsu[u]=fa[u][0];
        }
        for(v=find(v);dep[v]>dep[L];v=find(fa[v][0])){
            ans[v]=w,dsu[v]=fa[v][0];
        }
    }
    rep(i,1,n-1)printf("%d\n",ans[pos[i]]);
    return 0;
}
```

