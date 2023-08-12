AC 自动机应用熟练后，自己兴冲冲地去学 SAM 。

结果被劝退了，鸽了大半年才出来。

总结了下学习上的问题，发现网上的 blog 普遍有以下问题：

- 定义繁琐，刚学时直接一个 $\operatorname{endpos}$ 糊你脸上，极其劝退。
- 一开始所谓压缩、合并解释突兀，无法循序渐进地推出结论。

因此就有了这篇文章。

前置芝士：
- 自动机的定义：  
  字典树拓展为 “字典图” 就是自动机。如下就是一个合法的自动机：  
  ![](https://cdn.luogu.com.cn/upload/image_hosting/9zk0zhon.png)  
  其中的结点称为**状态**，带权的有向边称为**转移**，  
  遍历的起点即**初始状态**，按照字符串走到的对应结点称为**目标状态**。  
  状态的数量既可以是有限的，也可以是无限的。  
  同一状态同一权值的出边既可以只有一个（**确定性**的），  
  也可以有多个（**非确定性**的）。  
  OI 一般只研究 有限状态确定性自动机（DFA）。
- [AC 自动机及其原理](https://www.luogu.com.cn/blog/wangrx/solution-p5357)
 
## 问题引入

> 给定一个字符串 $s$，求另外若干个字符串是否为 $s$ 的子串，强制在线。

对于这个问题，离线可以通过将 $s$ 以外的字符串建立 AC 自动机来解决。

但该问题要求在线，因此易想到建立一个关于 $s$ 的字典树，存储 $s$ 的所有子串。  

如字符串 $\texttt{cabab}$ 子串构成的字典树如下：

![](https://cdn.luogu.com.cn/upload/image_hosting/rmp42as9.png)

但显然，这种做法时空复杂度均为 $\Theta(|s|^2)$ ，  
因此需要一种快速构建的方法，于是就有了 SAM。

## 字符的加入与 fail 指针

字符串算法通常为 $\Theta(|s|)$，这启示我们考虑加入单个字符的情况。

考虑末尾加入一个字符 $\texttt{c}$，则显然，加入的子串有：  
```plain
cababc
 ababc
  babc
   abc
    bc
     c
```

可以发现，它们都是由 $s$ 的后缀（可以为空）添加一个字符 $\texttt{c}$ 而来。  
我们将 $s$ 的后缀在字典树上标记出来（包括空串）：  
![](https://cdn.luogu.com.cn/upload/image_hosting/haeq11h5.png)

这个时候，我们意识到：遍历 $s$ 的后缀的过程，不就是跳 AC 自动机的 fail 指针吗？  
因为 fail 指针的定义是：

> 字符串 $s$ 在字典树上出现了的最长真后缀（可以为空）。

于是我们标记上 $s$ 的后缀的 fail 指针，可知它们形成了一条链：  
![](https://cdn.luogu.com.cn/upload/image_hosting/58at5gps.png)  

于是只要顺着这条链往上跳，就可以遍历 $s$ 的所有后缀，添加结点了：    
![](https://cdn.luogu.com.cn/upload/image_hosting/gc3fm95y.png)

同时，我们也可以知道新增结点的 fail 指针：  
设状态 $u$ 边权为 $c$ 的出边为 $u\mathop{\rightarrow}\limits^c$，则在 AC 自动机上有：  
- $u\mathop{\rightarrow}\limits^c$ 为 $u$ 在字典树上的儿子，则 $fail_{u\mathop{\rightarrow}\limits^c}=fail_u\mathop{\rightarrow}\limits^c$（转移 1）
- 否则，$u\mathop{\rightarrow}\limits^c=fail_u\mathop{\rightarrow}\limits^c$（转移 2）

这个过程并没有优化复杂度，但却给我们以极大的启发：  
如果将多个状态压缩为一个，我们也可以通过维护 fail 指针来构建自动机。

## 结束位置与状态压缩

考虑抠除 字典树上的出边，只保留 fail 指针（也就是只看 fail 树）：    
![](https://cdn.luogu.com.cn/upload/image_hosting/0xeaa0j8.png)

看不清？那就整理一下：

![](https://cdn.luogu.com.cn/upload/image_hosting/p7cxlu99.png)

可以看到，fail 树上存在“单独”（除链底外均没有儿子）的链，  
比如 $4\rightarrow 8\rightarrow 11,5\rightarrow 9\rightarrow 12,7\rightarrow 10$  
压缩这些链，就能降低跳 fail 树的时空间成本（暂时先不管出边）。 

回到整理前的图，考虑这些链的意义，发现这和子串在 $s$ 中的结束位置有关：

- 对于子串 $a,b$，  
  $a,b$ 存在一个相同的结束位置 $\Leftrightarrow$ fail 树上其中一个为另一个的祖先。  
  这是显然的，因为相同的结束位置意味着其中一个为另一个的后缀。
- 在该 fail 树上，子串 $a$ 为子串 $b$ 父亲 $\Leftrightarrow a$ 恰好为 $b$ 去掉第一个字符。  
  由 fail 指针定义可知 $a$ 为 $b$ 在字典树上的最长真后缀，  
  至多为 $b$ 去掉第一个字符。  
  由于字典树上存储了 $s$ 的所有子串，因此 $b$ 的所有子串也在里头，  
  这个可能的最大值必然存在。

由此引出了 $\operatorname{endpos}$ 的定义：

$\operatorname{endpos}(t)$ 为子串 $t$ 在原串中所有结束位置构成的集合。于是可知：
- 该 fail 树上，$a$ 为 $b$ 的祖先 $\Leftrightarrow a$ 为 $b$ 的后缀 $\Leftrightarrow\operatorname{endpos}(a)\supseteq \operatorname{endpos}(b)$
- 同一条“单独”的链上的子串 $\Leftrightarrow\operatorname{endpos}$ 相同。
- $\operatorname{endpos}$ 相同的子串，其长度是连续的，且**出边的 $\operatorname{endpos}$ 相同**。  

因此，$\operatorname{endpos}$ 相同的结点就会被我们压缩为一个结点，其出边保持不变，  
且可以用最长串和最短串的长度得出压缩的点的数量。

## SAM 构建
说了这么多，终于到了 SAM 的构建。  
除了出边，我们还要维护以下两个数组：
- 后缀链接 $\operatorname{link}$："单独"的链 链顶的 fail 指针。
- $\operatorname{len}$：压缩的链中最长串的长度。 

以及一个额外变量：
- $last$：整个串的末尾所在结点。
  
初始时，我们只有自动机的起点，即空串。

每加入一个字符 $c$ 至末尾，新增一个结点 $v,\operatorname{len}_v=\operatorname{len}_{last}+1$，  
然后从 $last$ 往上跳后缀链接，设跳到的结点为 $u$，则：  
- 若 $u\mathop{\rightarrow}\limits^c$ 不存在，由 转移 2 可知，将 $u\mathop{\rightarrow}\limits^c$ 置为 $v$。
- 否则，由转移 1 可知，$\operatorname{link}_v$ 必在 $u\mathop{\rightarrow}\limits^c$ 中。
  - 若  $\operatorname{len}_{u\mathop{\rightarrow}\limits^c}=\operatorname{len}_u+1$，则直接将 $\operatorname{link}_v$ 置为 $u\mathop{\rightarrow}\limits^c$，退出；
  - 否则，将 $u\mathop{\rightarrow}\limits^c$ 分裂为两个结点。  
  
    具体地，从 $u\mathop{\rightarrow}\limits^c$ 复制出一个结点，  
    旧的称为 $old$，新的称为 $new$，二者出边相同，   
    然后将 $\operatorname{len}_{new}$ 置为 $\operatorname{len}_u+1$。  
    接下来，将 $\operatorname{link}_{new}$ 置为 $\operatorname{link}_{old}$，再将 $\operatorname{link}_{old}$ 置为 $new$。   
    从 $u$ 继续跳后缀链接，若 $u\mathop{\rightarrow}\limits^c=old$，则置 $u\mathop{\rightarrow}\limits^c$ 为 $new$，  
    直到跳完了根或者 $u\mathop{\rightarrow}\limits^c\not=old$。

    之后，将 $\operatorname{link}_v$ 置为 $u$，退出。
- 若跳完了根都未出现转移，则将 $\operatorname{link}_v$ 置为空串。

最后，记得将 $last$ 置为 $v$。

```cpp
typedef unsigned int word;
struct SAM{
	word next[1<<21][26];//换成 map 也是可以的！
	word link[1<<21],len[1<<21];
	word size,last;
	inline SAM(){//初始状态
		size=last=0;
		link[0]=0xffffffff;
	}
	#define split()					\
		len[++size]=len[last]+1;	\
		memcpy(next[size],next[old],sizeof(next[0]));	\
		link[size]=link[old],link[old]=size;			\
		for(;last!=0xffffffff;last=link[last])			\
			if(next[last][c]==old) next[last][c]=size;	\
			else break; //分裂结点
	inline void operator +=(char c){//添加末端字符
		register word old;
		len[++size]=len[last]+1;//新建结点
		for(;last!=0xffffffff;last=link[last])
			if((old=next[last][c])==0) next[last][c]=size;//转移 2
			else if(len[old]==len[last]+1)//转移 1 情况 1
				return link[size]=old,void(last=size);
			else{//转移 1 情况 2
				split();
				return link[size-1]=size,void(last=size-1);
			}
		return link[size]=0,void(last=size);
	}
	#undef split
};
```
## 时空间复杂度及其证明
求证：SAM 的结点数不超过 $2|s|-1$。   
证明：算法本身即可证明该命题。  
$\qquad\quad$每次添加字符时只添加一个或两个结点。  
$\qquad\quad$头两次添加字符只会添加一个结点，  
$\qquad\quad$算上起点，至多有 $2|s|-1$ 个。  
$\qquad\quad\texttt{abb}\cdots\texttt{bb}$ 达到了这个上界，  
$\qquad\quad$从第三次开始添加字符开始每一次添加字符都添加了两个结点。  
求证：SAM 的转移数不超过 $3|s|-4$（假设 $|s|\ge 3$）  
证明：[（不会）](https://oi-wiki.org/string/sam/#_11) 

求证：SAM 的操作次数为线性的。  
证明：[（不会）](https://oi-wiki.org/string/sam/#_7)

综上，若设字符集为 $\sum$
- 若使用数组存储出边，则 SAM 时间复杂度为 $\Theta(|s|)$，  
  空间复杂度为 $\Theta(|s|\times|\sum|)$
- 若使用 `std::map` 存储出边，则 SAM 时间复杂度为 $\Theta(|s|\log|\sum|)$，  
  空间复杂度为 $\Theta(|s|)$

## 广义 SAM 及其构建
刚才我们用 SAM 解决了单个字符串的子串匹配问题，实际上多个字符串也可以。  
对多个字符串的所有子串建立字典树，  
易发现其 fail 树满足仅有单个字符串子串时的所有性质。  
（具体已在章节 结束位置与状态压缩 时给出）

因此 SAM 便可以转移到字典树上，但添加字符时要特判：  
新建结点时 $last\mathop{\rightarrow}\limits^c$ 已经出现，则无需新建结点，直接跳到转移 1。
```cpp
typedef unsigned int word;
struct SAM{
	word next[1<<21][26];
	word link[1<<21],len[1<<21];
	word size,last;
	inline SAM(){
		size=last=0;
		link[0]=0xffffffff;
	}
	#define split()					\
		len[++size]=len[last]+1;	\
		memcpy(next[size],next[old],sizeof(next[0]));	\
		link[size]=link[old],link[old]=size;			\
		for(;last!=0xffffffff;last=link[last])			\
			if(next[last][c]==old) next[last][c]=size;	\
			else break;
	inline void operator +=(char c){
		register word old;
		if(old=next[last][c]){//结点已经出现，直接跳到转移 1
			if(len[old]==len[last]+1)
				return void(last=old);
			split();
			return void(last=size);
		}
		len[++size]=len[last]+1;
		for(;last!=0xffffffff;last=link[last])
			if((old=next[last][c])==0) next[last][c]=size;
			else if(len[old]==len[last]+1)
				return link[size]=old,void(last=size);
			else{
				split();
				return link[size-1]=size,void(last=size-1);
			}
		return link[size]=0,void(last=size);
	}
	#undef split
};
```
添加字符到某个串末尾时将 $last$ 设为该串的末尾所在结点即可。  
可以证明分裂结点时各串末尾所在的结点不变。  
建议写 SAM 时直接写广义的。

## SAM/广义 SAM 的其他运用

SAM/广义 SAM 还有大量其它运用，均是基于被压缩的 fail 树的：  
顺着后缀链接往上跳，跳到的点均为其后缀。

1. 查找子串的出现次数。  
   对于子串 $t$，
   出现次数其实就是结束位置的数量，也就是 $|\operatorname{endpos}(t)|$。  
   每插入一个字符，标记当时的 $last$，  
   最后这些 $last$ 统一往上跳，计算 $|\operatorname{endpos}(t)|$ 即可。
2. 查找所有不同子串的出现次数。  
   计算 $\displaystyle\sum_{\texttt{子串} t}|\operatorname{endpos}(t)|$ 即可，  
   同一个 $\operatorname{endpos}$ 包含的子串数量可用 $\operatorname{len}$ 计算出来。
3. 查找所有不同子串的总长。  
   同一个 $\operatorname{endpos}$ 包含的子串总长也可以用
$\operatorname{len}$ 计算出来。
4. 查找所有不同子串的 总长 $\times$ 出现次数  
   2. , 3. 已经给出，因此同上。
5. 查找两个串的最长公共后缀/前缀。  
   后缀其实就是求 fail 树上的 LCA，前缀对反串建 SAM 即可。

还有非常多的应用，这里就不给出了。

