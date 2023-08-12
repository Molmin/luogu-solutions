# 一个不用树链剖分和LCT的题解
~~其实是因为本蒟蒻不会树链剖分和LCT嘤嘤嘤~~

---

* **前半部分的想法和树链剖分是一样的**  
将所有操作按时间反转,将删边转化为加边。  
从最开始的图中随便取出一棵生成树,假设最开始树上所有边都是白边  
此后每添加一条非树边$(u,v)$,就将树上$u-v$的路径全部染黑  

* **然后不难发现,每次询问$(u,v)$,则$u-v$路径上的白边就是关键航线,$u-v$路径上的的白边数量就是答案**  
路径染色,查询路径上白边数量,不难想到用树链剖分+线段树来维护,时间复杂度$O(N\cdot log^2N)$

---

* 思考一下如何不用树链剖分  
发现每条边只用被染色一次~~黑的染多少次都是黑的~~  
**令白边的边权为1,黑边的边权为0**  
每次添加一条非树边$(u,v)$,就将$u-v$路径上所有未染黑的边边权都-1,并将这些边染黑   
**这样每次询问$(u,v)$的时候,答案就是$u-v$路径的边权和**

* ***如何保证每条边只被染黑一次？***  
每次染黑一条路径$u-v$的时候,用并查集将路径上所有边$(a,b)$的$father$设置为$u-v$的$LCA(u-v)$的$father$,这样下次再访问到$(a,b)$这条边的时候,直接跳过这条边,到$LCA(u-v)$处  
**由于每条边只会被缩一次，所以时间复杂度均摊$O(N)$**
* **每条边用较深的那个点来表示**  
例如,边$(a,b)$中,$a$为$b$的儿子  
$(a,b)$的$father$用$a$的$father$表示  
$(a,b)$的边权用$a$的点权表示

* **这部分有点复杂,详情可以见代码**

---
* 还剩一个小问题:如何不用树链剖分维护单点加,路径求和  
令$L(u)$表示$u$到根的边权和  
**$u-v$路径的边权和即为$L(u)+L(v)-2L(LCA(u,v))$**

* 不难发现,每次将点$u$的点权$+x$,则对于$u$子树中所有点$v$,$L(v)$会$+x$,子树外所有点$p$的$L(p)$都不变  
又因为,同一棵子树中的点的$DFS$序是连续的  
**问题就转化为了区间加,单点求值**

* 将正常的树状数组询问,修改时+,-的方向反过来,就得到了  
**资瓷区间加,单点求值的树状数组**  
(详情请见代码)

---

**时间复杂度$O(N\cdot logN)$,瓶颈在树状数组,倍增$LCA$,`std::map`上**

## 完结撒花~
```
#include<iostream>
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<map>
#define lb(x) (x&-x)//二进制最低位
#define pii pair<int,int>
#define fi first
#define se second
using namespace std;
int n,m,cnt,tme;//点数,边数,操作总数,DFS时间戳
int hd[30009],eg[60009],nxt[60009],tot;//存图
int dfn[30009],ed[30009],dep[30009];//依次为:DFS序,子树中DFS序最大值,节点深度
int c[30009];//树状数组
int fn[30009];//并查集数组
int f[30009][16];//倍增数组
pii e[100009],q[40009];//边集,询问
int del[100009],ont[100009];//边是否被删除,是否为生成树上的边
int typ[40009],ans[40009];//询问的种类,答案
map<pii,int> mp;//记录边的编号
void add(int x,int val)//树状数组[1,x]区间加val
{
    for(int i=x;i;i-=lb(i))
        c[i]+=val;
}
int ask(int x)//x单点求值
{
    int ans=0;
    for(int i=x;i<=n;i+=lb(i))
        ans+=c[i];
    return ans;
}
int fnd(int x)//并查集
{
    return x==fn[x]?x:fn[x]=fnd(fn[x]);
}
void ins(int a,int b)//加边
{
    eg[++tot]=b;
    nxt[tot]=hd[a];
    hd[a]=tot;
}
void dfs(int x,int fa)
{
    dep[x]=dep[fa]+1;
    ed[x]=dfn[x]=++tme;//标记DFS序
    f[x][0]=fa;//标记父亲
    for(int i=1;i<=15;i++)//预处理倍增数组
        f[x][i]=f[f[x][i-1]][i-1];
    for(int i=hd[x];i;i=nxt[i])
        if(eg[i]!=fa)//防止返祖访问
        {
            dfs(eg[i],x);
            ed[x]=max(ed[x],ed[eg[i]]);//更新ed
        }
}
int getlca(int x,int y)//倍增求LCA
{
    if(dep[x]<dep[y])
        swap(x,y);
    int l=dep[x]-dep[y];
    for(int i=0;i<=15;i++)
        if(l&(1<<i))
            x=f[x][i];
    if(x==y)
        return x;
    for(int i=15;i>=0;i--)
        if(f[x][i]!=f[y][i])
            x=f[x][i],y=f[y][i];
    return f[x][0];
}
void modify(int x,int y)//核心:并查集缩点(其实和树链剖分的修改操作很像)
{
    int fx=fnd(x),fy=fnd(y),lca=getlca(x,y);
    while(fx!=fy)
    {
        if(dep[fx]<dep[fy])//优先将深的点向上跳
            swap(fx,fy);
        if(fx!=lca)//将fx到fx父亲的边的边权-1,若fx为LCA则不用修改
        {
            add(ed[fx],-1);//树状数组区间修改
            add(dfn[fx]-1,1);//树状数组区间修改
        }
        x=fnd(f[fx][0]);//fx父亲所在集合的根
        fn[fx]=x;//将fx的根设置为fx父亲所在集合的根
        fx=x;//继续向上跳
    }
}//这里每条边只会被修改一次,所以时间复杂度均摊O(N)
int query(int x,int y)//询问u-v路径的边权和
{
    return ask(dfn[x])+ask(dfn[y])-2*ask(dfn[getlca(x,y)]);
}
void init()
{
    for(int i=1;i<=n;i++)//初始化并查集
        fn[i]=i;
    for(int i=1;i<=m;i++)//得到一棵生成树
        if(!del[i])
        {
            int a=e[i].fi,b=e[i].se;
            if(fnd(a)!=fnd(b))//若a,b不连通
            {
                ont[i]=1;//标记树边
                fn[fnd(a)]=fnd(b);//并查集合并
                ins(a,b);//加边
                ins(b,a);//加边
            }
        }
    dfs(1,0);
    for(int i=2;i<=n;i++)//将所有边的边权设为1
    {
        add(ed[i],1);//树状数组区间修改
        add(dfn[i]-1,-1);//树状数组区间修改
    }
    for(int i=1;i<=n;i++)//初始化并查集
        fn[i]=i;
    for(int i=1;i<=m;i++)//添加未被删除的非树边
        if(!del[i]&&!ont[i])
            modify(e[i].fi,e[i].se);
}
void solve()
{
    for(int i=cnt;i>=1;i--)//倒序处理操作
    {
        if(!typ[i])
            modify(q[i].fi,q[i].se);//添加非树边
        else
            ans[i]=query(q[i].fi,q[i].se);//处理询问
    }
    for(int i=1;i<=cnt;i++)//正序输出
        if(typ[i])
            printf("%d\n",ans[i]);
}
int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1,a,b;i<=m;i++)
    {
        scanf("%d%d",&a,&b);
        if(a>b)
            swap(a,b);
        e[i]=pii(a,b);
        mp[e[i]]=i;//用map记录边的编号
    }
    int c,a,b;
    scanf("%d",&c);
    while(c!=-1)
    {
        scanf("%d%d",&a,&b);
        if(a>b)
            swap(a,b);
        q[++cnt]=pii(a,b);
        typ[cnt]=c;//记录操作种类
        if(!c)
            del[mp[q[cnt]]]=1;//这条边已被删除
        scanf("%d",&c);
    }
    init();
    solve();
    return 0;
}
```