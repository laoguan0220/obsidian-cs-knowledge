![[Pasted image 20260517123132.png]]
# 树上 LCA 与倍增算法完整学习指南

## 一、必备基础概念与算法应用

### 1. 必须掌握的前置概念

- **树的基本概念**：根节点、父节点、子节点、深度（根到节点的边数 / 节点数）、祖先、后代、子树
- **LCA 定义**：两个节点 u 和 v 的**最近公共祖先**，是它们所有公共祖先中深度最大的那个节点
- **倍增思想**：二进制拆分，预处理 2^k 步能到达的位置，将线性时间的跳跃转化为对数时间
- **DFS/BFS**：用于预处理树的深度、父节点等信息

### 2. LCA 算法能解决的核心问题

- 直接求任意两个节点的最近公共祖先
- 计算**树上两点间最短距离**：`dist(u,v) = depth[u] + depth[v] - 2*depth[lca(u,v)]`
- 树上路径查询：路径上的最大值、最小值、和、异或和
- 树上差分：对路径上的所有节点进行加减操作，最后前缀和还原
- 图论问题转化：将有环图转化为生成树后，用 LCA 处理环相关问题
- 其他高级应用：虚树构建、树上莫队、树哈希等

## 二、主流 LCA 算法对比与实现

### 算法总览

表格

|算法|时间复杂度|空间复杂度|在线 / 离线|实现难度|适用场景|
|---|---|---|---|---|---|
|暴力法|预处理 O (n)，查询 O (n)|O(n)|在线|极低|树极小 (n≤1000) 或查询极少|
|倍增法|预处理 O (nlogn)，查询 O (logn)|O(nlogn)|在线|中等|绝大多数竞赛场景，最常用|
|Tarjan 离线法|预处理 O (n+q)，查询 O (1)|O(n+q)|离线|中等|查询量极大且可离线处理|
|树链剖分法|预处理 O (n)，查询 O (logn)|O(n)|在线|较高|已实现树链剖分的场景|
|欧拉序 + RMQ|预处理 O (nlogn)，查询 O (1)|O(nlogn)|在线|中等|查询量极大且需在线处理|

---

### 1. 暴力法（理解 LCA 本质）

#### 原理

1. 预处理每个节点的深度和直接父节点
2. 将深度较大的节点向上跳，直到两个节点深度相同
3. 两个节点同时向上跳，直到相遇，相遇点即为 LCA

#### 实现模板

cpp

运行

```
const int MAXN=1e5+5;
vector<int> G[MAXN];
int depth[MAXN], fa[MAXN];

// 预处理深度和父节点
void dfs(int u, int father) {
    fa[u]=father;
    depth[u]=depth[father]+1;
    for(int v:G[u]) {
        if(v!=father) dfs(v,u);
    }
}

// 暴力求LCA
int lca(int u, int v) {
    // 让u成为深度更大的节点
    if(depth[u]<depth[v]) swap(u,v);
    // u向上跳到和v同深度
    while(depth[u]>depth[v]) u=fa[u];
    // 同时向上跳直到相遇
    while(u!=v) {
        u=fa[u];
        v=fa[v];
    }
    return u;
}
```

---

### 2. 倍增法（重点掌握，竞赛首选）

#### 原理

- 预处理每个节点的**2^k 级祖先**：`up[k][u]`表示节点 u 向上跳 2^k 步到达的节点
- 递推关系：`up[k][u] = up[k-1][up[k-1][u]]`（跳 2^(k-1) 步再跳 2^(k-1) 步）
- 查询时，先将两个节点跳到同一深度，再从大到小枚举 k，若跳 2^k 步后不相遇则跳，最后父节点即为 LCA
- ==需要注意的点==  如果一个节点是另一个节点的子节点 那么当他们深度相同时 这两个节点相等
所以得判断一下是否相等 如果他们是两个不同的子树 需要让他们处在祖先的节点的下一层
然后再跳一步到达祖先节点

#### 实现模板

cpp

运行

```
const int MAXN=1e5+5;
const int LOG=20; // 2^20>1e6，足够大多数情况
vector<int> G[MAXN];
int depth[MAXN], up[LOG][MAXN];

// 预处理倍增数组
void dfs(int u, int father) {
    up[0][u]=father;
    depth[u]=depth[father]+1;
    // 预处理2^1到2^(LOG-1)级祖先
    for(int k=1;k<LOG;k++) {
        up[k][u]=up[k-1][up[k-1][u]];
    }
    for(int v:G[u]) {
        if(v!=father) dfs(v,u);
    }
}

// 倍增求LCA
int lca(int u, int v) {
    // 让u成为深度更大的节点
    if(depth[u]<depth[v]) swap(u,v);
    // u向上跳到和v同深度
    int diff=depth[u]-depth[v];
    for(int k=0;k<LOG;k++) {
        if(diff&(1<<k)) {
            u=up[k][u];
        }
    }
    if(u==v) return u;
    // 从大到小枚举k，跳尽可能大的步长
    for(int k=LOG-1;k>=0;k--) {
        if(up[k][u]!=up[k][v]) {
            u=up[k][u];
            v=up[k][v];
        }
    }
    // 最后父节点就是LCA
    return up[0][u];
}
```

#### 注意事项

- LOG 的取值：`LOG=ceil(log2(MAXN))`，一般取 20 足够处理 1e6 以内的节点
- 根节点的父节点设为 0 或自己，避免越界
- 可以在 DFS 中同时预处理路径上的最值、和等信息，扩展功能

---

### 3. Tarjan 离线算法

#### 原理
![[Pasted image 20260517230843.png]]
- 基于**并查集**和**DFS**，需要提前知道所有查询
- 遍历树时，将已访问的子树合并到父节点的集合中
- 当处理到节点 u 时，所有与 u 相关的查询中，已访问过的节点 v 的 LCA 就是 find (v)
可以保证 fa$[x]$的含义是从v节点到当前dfs路径上的最近公共祖先 这一实时性
而你遍历x的时候 当前这个链上的前置节点都是它的祖先 

#### 实现模板

cpp

运行

```
const int MAXN=1e5+5;
vector<int> G[MAXN];
vector<pair<int,int>> query[MAXN]; // query[u]存与u相关的查询(v, idx)
int ans[MAXN]; // 存储每个查询的答案
int fa[MAXN], vis[MAXN];

// 并查集
int find(int x) {
    return fa[x]==x?x:fa[x]=find(fa[x]);
}

void tarjan(int u, int father) {
    vis[u]=1;
    for(int v:G[u]) {
        if(v!=father) {
            tarjan(v,u);
            fa[v]=u; // 合并子树到父节点
        }
    }
    // 处理所有与u相关的查询
    for(auto &p:query[u]) {
        int v=p.first, idx=p.second;
        if(vis[v]) {
            ans[idx]=find(v);
        }
    }
}

// 使用方法
// 1. 初始化并查集：for(int i=1;i<=n;i++) fa[i]=i;
// 2. 添加查询：query[u].emplace_back(v, i); query[v].emplace_back(u, i);
// 3. 调用tarjan(root, 0);
```

---

### 4. 树链剖分法

#### 原理

- 将树剖分成若干条重链，保证任意一条路径最多经过 logn 条重链
- 求 LCA 时，每次将所在重链顶端深度较大的节点跳到重链顶端的父节点
- 当两个节点在同一条重链上时，深度较小的节点即为 LCA

#### 核心代码片段

cpp

运行

```
// 树链剖分求LCA
int lca(int u, int v) {
    while(top[u]!=top[v]) { // 不在同一条重链上
        if(depth[top[u]]<depth[top[v]]) swap(u,v);
        u=fa[top[u]]; // 跳到重链顶端的父节点
    }
    return depth[u]<depth[v]?u:v;
}
```

---

### 5. 欧拉序 + RMQ 法

#### 原理

- DFS 遍历树，记录每个节点第一次出现的位置和深度，得到欧拉序数组
- 两个节点 u 和 v 的 LCA 就是它们在欧拉序中第一次出现位置之间深度最小的节点
- 用 ST 表预处理 RMQ，实现 O (1) 查询

## 三、学习建议

1. **先学暴力法**：理解 LCA 的本质和基本求解思路
2. **重点攻克倍增法**：这是竞赛中最常用的算法，实现简单且效率足够
3. **了解其他算法**：知道它们的适用场景和优缺点，遇到特定问题时能选择最优解
4. **多做练习题**：从基础的 LCA 查询开始，逐步练习树上距离、树上差分等扩展应用

需要我给你一份**倍增法 LCA 的完整可运行代码**（包含输入输出和样例）吗？


```cpp迭代法求lca
#include<bits/stdc++.h>
using namespace std;

const int N = 3e5 + 10;
const int LOG = 20; // 2^19 > 3e5，所以 0~19 足够

struct Edge {
    int to;
    int w;
};

vector<Edge> g[N];

int n, m;
int depth[N];
int fa[N][LOG];
long long distRoot[N]; // 如果不需要距离，可以删掉

void build_lca_iterative(int root = 1)
{
    vector<int> order;
    order.reserve(n);

    stack<int> st;
    st.push(root);

    fa[root][0] = 0;
    depth[root] = 1;
    distRoot[root] = 0;

    while (!st.empty())
    {
        int u = st.top();
        st.pop();

        order.push_back(u);

        for (auto e : g[u])
        {
            int v = e.to;
            int w = e.w;

            if (v == fa[u][0]) continue;

            fa[v][0] = u;
            depth[v] = depth[u] + 1;
            distRoot[v] = distRoot[u] + w;

            st.push(v);
        }
    }

    for (int j = 1; j < LOG; j++)
    {
        for (int i = 1; i <= n; i++)
        {
            fa[i][j] = fa[fa[i][j - 1]][j - 1];
        }
    }
}

int lca(int x, int y)
{
    if (depth[x] < depth[y])
    {
        swap(x, y);
    }

    int diff = depth[x] - depth[y];

    for (int j = LOG - 1; j >= 0; j--)
    {
        if ((diff >> j) & 1)
        {
            x = fa[x][j];
        }
    }

    if (x == y) return x;

    for (int j = LOG - 1; j >= 0; j--)
    {
        if (fa[x][j] != fa[y][j])
        {
            x = fa[x][j];
            y = fa[y][j];
        }
    }

    return fa[x][0];
}

long long get_dist(int u, int v)
{
    int g = lca(u, v);
    return distRoot[u] + distRoot[v] - 2LL * distRoot[g];
}
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> m;

    for (int i = 1; i < n; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;

        g[u].push_back({v, w});
        g[v].push_back({u, w});
    }

    build_lca_iterative(1);

    while (m--)
    {
        int u, v;
        cin >> u >> v;

        cout << lca(u, v) << '\n';
        // 如果要求距离：
        // cout << get_dist(u, v) << '\n';
    }

    return 0;
}
```
```cpp
#include<bits/stdc++.h>
using namespace std;

const int N = 3e5 + 10;

struct Edge {
    int to;
    int w;
};

struct Query {
    int to;
    int id;
};

struct Event {
    int type;
    int u;
    int v;
    // type = 0: 进入节点 u，父亲是 v
    // type = 1: 合并 u 和它的儿子 v
    // type = 2: 退出节点 u，处理询问
};

vector<Edge> g[N];
vector<Query> q[N];

int n, m;

int dsu[N];
int ancestor[N];
bool vis[N];

int parentTree[N];
long long distRoot[N];

int ansLca[N];

int findset(int x)
{
    int root = x;

    while (dsu[root] != root)
    {
        root = dsu[root];
    }

    while (dsu[x] != x)
    {
        int t = dsu[x];
        dsu[x] = root;
        x = t;
    }

    return root;
}

void tarjan_lca_iterative(int root = 1)
{
    stack<Event> st;

    parentTree[root] = 0;
    distRoot[root] = 0;

    st.push({0, root, 0});

    while (!st.empty())
    {
        Event cur = st.top();
        st.pop();

        int type = cur.type;
        int u = cur.u;
        int v = cur.v;

        if (type == 0)
        {
            // 进入节点 u
            parentTree[u] = v;
            dsu[u] = u;
            ancestor[u] = u;

            // 退出事件最后执行
            st.push({2, u, 0});

            // 倒序压栈，让遍历顺序接近递归 DFS
            for (int i = (int)g[u].size() - 1; i >= 0; i--)
            {
                int to = g[u][i].to;
                int w = g[u][i].w;

                if (to == v) continue;

                distRoot[to] = distRoot[u] + w;

                // 先压合并，再压进入
                // 因为栈后进先出，所以会先处理子树，再合并
                st.push({1, u, to});
                st.push({0, to, u});
            }
        }
        else if (type == 1)
        {
            // 子树 v 已经处理完，合并 u 和 v
            int fu = findset(u);
            int fv = findset(v);

            dsu[fv] = fu;
            ancestor[fu] = u;
        }
        else
        {
            // 退出节点 u，标记为访问完成
            vis[u] = true;

            for (auto item : q[u])
            {
                int other = item.to;
                int id = item.id;

                if (vis[other])
                {
                    ansLca[id] = ancestor[findset(other)];
                }
            }
        }
    }
}

long long get_dist_by_lca(int u, int v, int lca)
{
    return distRoot[u] + distRoot[v] - 2LL * distRoot[lca];
}
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> m;

    for (int i = 1; i < n; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;

        g[u].push_back({v, w});
        g[v].push_back({u, w});
    }

    vector<pair<int, int>> ask(m + 1);

    for (int i = 1; i <= m; i++)
    {
        int u, v;
        cin >> u >> v;

        ask[i] = {u, v};

        q[u].push_back({v, i});
        q[v].push_back({u, i});
    }

    tarjan_lca_iterative(1);

    for (int i = 1; i <= m; i++)
    {
        int u = ask[i].first;
        int v = ask[i].second;
        int g = ansLca[i];

        cout << "LCA = " << g << '\n';

        // 如果要求路径距离：
        // cout << get_dist_by_lca(u, v, g) << '\n';
    }

    return 0;
}
```