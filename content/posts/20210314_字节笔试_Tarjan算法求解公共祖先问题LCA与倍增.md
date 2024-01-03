---
title: 【算法-字节笔试-中等难度】Tarjan算法求解公共祖先问题LCA，并介绍倍增算法
date: 2021-03-14T14:08:14+08:00
tags:
  - 分类/知识库/计算机/算法/算法思考
categories:
  - 算法
---

今天字节笔试的第二题，详情由于保密协议不能上网，但是大意就是给一大堆节点，去求LCA。递归直接爆栈，用stack写递归有一个点，改进优化了一下有两个点……
我印象中这个算法挺简单的，就搜了一下，果然找到了。不是，现在校招算法题都这么丧病了吗。
由于保密协议，不能放代码。后面放Tarjan算法学习笔记。

LCA问题[参考资料](https://blog.csdn.net/Lean_Feather/article/details/113057714)，
Tarjan的时间复杂度为O((n+q)× 并查集的复杂度 )，而使用路径压缩和按秩合并的并查集复杂度为O(Alpha(n))。所以作为离线算法，Tarjan比倍增算法快很多。
但作为在线算法，倍增算法能实时得到解法。
[RMQ](https://blog.csdn.net/weixin_42001089/article/details/83590686)

复杂度介绍：
* Tarjan的复杂度为O(n+q)
* RMQ预处理为O(nlogn)，查询O(1)
* 倍增算法复杂度为O((n+q)logn)


参考资料：
* [Tarjan求解LCA](https://www.cnblogs.com/wkfvawl/p/9415280.html)，非常好的教学，很详细地列举了LCA的步骤。关键是有图，有逐步分解的图，非常好。

## 伪代码
```
Tarjan(u)//marge和find为并查集合并函数和查找函数
{
    for each(u,v)    //访问所有u子节点v
    {
        Tarjan(v);        //继续往下遍历
        marge(u,v);    //合并v到u上
        标记v被访问过;
    }
    for each(u,e)    //访问所有和u有询问关系的e
    {
        如果e被访问过;
        u,e的最近公共祖先为find(e);
    }
}
```

## 具体实现代码
```cpp
#include <iostream>
#include <vector>
#include <utility>
using namespace std;
int N = 5;
vector<vector<int> > lib;//假设lib为二维动态数组，lib[i]表示节点i的所有孩子vector
vector<int> parent(N,0);//并查集数组
vector<bool> isVisited(N,false);
vector<vector<int> > query;//query关系，双向的

int find(int e){
    if(parent[e] != e) return find(parent[e]);
    return e;
}

void Tarjan(int u){
    //访问所有的孩子
    for(int i =0;i<lib[u].size();i++){
        int v = lib[u][i];
        Tarjan(v);
        parent[v] = u;//merge
        isVisited[v] = true;
    }
    for(int i = 0;i<query[u].size();i++){
        int e = query[u][i];
        if(isVisited[e]){
            cout<<"u和e的共同祖先为"<<find(u,e)<<endl;
        }
    }
}

int main(void){
    for(int i = 0;i<N;i++){
        parent[i] = i;
    }
    Tarjan(0);
    cout<<"test"<<endl;
    return 0;
}
```
## 倍增算法
参考资料：
* [b站视频](https://www.bilibili.com/video/BV1N7411G7JD)
* [csdn](https://blog.csdn.net/Lean_Feather/article/details/113057714)

实例代码（有点问题）
```cpp
#include <cstring>
#include <algorithm>
const int maxn = 1000;//递归栈的最大深度，原则上等于点的数量.
const int maxm = 500;
const int maxh = 31;

//定义见前向星
int info[maxn];
int point[maxm];
int next[maxm];

int dep[maxn];//深度数组
int anc[maxn][maxh];
void dfs(int root){
    static int Stack[maxn];
    int top=0;//栈顶指针

    memset(dep,0,sizeof(dep));
    dep[root] = 1;
    for(int i = 0;i<maxh;i++){
        anc[root][i] = root;//根节点无论怎么跳，都是根节点
    }
    Stack[++top] = root;//Stack[1] = top;
    while(top){
        int x = Stack[top];
        if(x != root){
            for(int i = 1;i<maxh;i++){//按照方程更新数组
                int y = anc[x][i-1];
                anc[x][i] = anc[y][i-1];
            }
        }
        for(int &i = info[x];i;i=next[i]){
            int y = point[i];
            if(y!=anc[x][0]){
                dep[y] = dep[x] + 1;
                anc[y][0] = x;
                Stack[++top] = y;
            }
        }
        while(top && head[Stack[top]] == 0) top--;//清理叶子节点
    }
    void swim(int &x,int H){
        //目标是让x向上跳H步，使用二进制方式
        for(int i=0;H>0;i++){
            if(H&1) x = anc[x][i];//i相当于现在跳2^i步，当H%2==1时
            x /= 2;//相当于右移
        }
    }
    int lca(int x,int y){
        int i;
        if(dep[x]>dep[y]) swap(x,y);
        swim(y,dep[y]-dep[x]);
        if(x==y) return x;
        for(;;){
            for(i=0;anc[x][i]!=anc[y][i];i++); //同时起跳，寻找不重叠的最近的父节点
            if(i==0){//找不到，则显然上一个节点即为LCA
                return anc[x][0];
            }
            //起跳，因为anc[x][i] == anc[y][i]，所以只能跳到i-1
            x = anc[x][i-1];
            y = anc[y][i-1];
        }
        return -1;//有点问题，应该走不到这一步
    }
}
```
该代码有一些问题：
1. 为什么叶子节点在深搜中全部丢弃，这样它们的anc数组就没有办法更新了
2. 最后lca的时候，-1是到不了的，因为上面是死循环，没有对其进行判断。

OI wiki的代码，题目有些区别，不过思想是一样的
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <vector>
#define MXN 50007
using namespace std;
std::vector<int> v[MXN];//邻接表写法
std::vector<int> w[MXN];
//fa表示亲缘数组，cost表示每一跳的代价
int fa[MXN][31], cost[MXN][31], dep[MXN];
int n, m;
int a, b, c;
void dfs(int root, int fno) {//用显式DFS做，其实差不多
  fa[root][0] = fno;
  dep[root] = dep[fa[root][0]] + 1;
  for (int i = 1; i < 31; ++i) {
    fa[root][i] = fa[fa[root][i - 1]][i - 1];
    cost[root][i] = cost[fa[root][i - 1]][i - 1] + cost[root][i - 1];
  }
  int sz = v[root].size();
  for (int i = 0; i < sz; ++i) {
    if (v[root][i] == fno) continue;
    cost[v[root][i]][0] = w[root][i];
    dfs(v[root][i], root);
  }
}
int lca(int x, int y) {
  if (dep[x] > dep[y]) swap(x, y);
  int tmp = dep[y] - dep[x], ans = 0;
  for (int j = 0; tmp; ++j, tmp >>= 1)
    if (tmp & 1) ans += cost[y][j], y = fa[y][j];
  if (y == x) return ans;
  for (int j = 30; j >= 0 && y != x; --j) {
    if (fa[x][j] != fa[y][j]) {
      ans += cost[x][j] + cost[y][j];
      x = fa[x][j];
      y = fa[y][j];
    }
  }
  ans += cost[x][0] + cost[y][0];
  return ans;
}
int main() {
  memset(fa, 0, sizeof(fa));
  memset(cost, 0, sizeof(cost));
  memset(dep, 0, sizeof(dep));
  scanf("%d", &n);
  for (int i = 1; i < n; ++i) {
    scanf("%d %d %d", &a, &b, &c);
    ++a, ++b;
    v[a].push_back(b);
    v[b].push_back(a);
    w[a].push_back(c);
    w[b].push_back(c);
  }
  dfs(1, 0);
  scanf("%d", &m);
  for (int i = 0; i < m; ++i) {
    scanf("%d %d", &a, &b);
    ++a, ++b;
    printf("%d\n", lca(a, b));
  }
  return 0;
}
```