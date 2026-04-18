离线的操作
什么情况适合离线的操作？
没有一边查询一边修改的情况，比较适合离线操作
离线其实有点像先解决子问题，再解决大的问题，是一种水到渠成的方法，也就是说前面比较固定

对输出进行离线处理，没有动态的修改可以用离线处理，处理子问题到处理整个问题，有的时候没必要非得维护所有的信息，有的对问题的答案没有帮助，本题，1334 查询l，r范围，对于一种种类很多的位置，那么最靠右边的那个位置如果被包含在内，左边的就没意义了，如果没包含在内，左边的也肯定不在里面，也就是我们维护最新出现的位置加1即可，原本的位置减去1 
# 区间不同元素个数查询（离线 + 树状数组）

## 问题描述

给定一个长度为 n 的数组，进行 m 次查询，每次查询区间 [l,r] 内**不同元素的个数**。

## 算法思路

1. **离线处理**：将所有查询按照**右端点 r** 从小到大排序，统一处理。
2. **树状数组**：维护区间内不同元素的数量。
3. **去重核心**：用数组记录每个数值**最后一次出现的位置**，遍历数组时，若当前数值已出现过，就删除旧位置的标记，在新位置添加标记，保证每个数值在树状数组中**只保留最右侧的一个标记**。
4. **查询答案**：对于排序后的查询，直接用树状数组查询区间和，即为区间内不同元素的数量。
#树状数组 #算法 #离线处理 
## 完整代码

cpp

运行

```
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+5;

int n,m;
int arr[N];          // 原数组
int brr[N];          // 记录某个数值出现的最右边的位置
int tree[N];         // 树状数组

// 树状数组lowbit操作
int lowbit(int i)
{
    return i&(-i);
}

// 树状数组前缀和查询
int sum(int x)
{
    int ans=0;
    for(int i=x;i>0;i=i-lowbit(i))
    {
        ans+=tree[i];
    }
    return ans;
}

// 树状数组单点更新
void add(int x,int v)
{
    for(int i=x;i<=n;i=i+lowbit(i))
    {
        tree[i]+=v;
    }
}

// 查询结构体：存储左端点、右端点、查询编号
struct que{
    int l;
    int r;
    int id;
}que[N];

// 排序规则：按查询的右端点从小到大排序
bool cmp(struct que a,struct que b)
{
    return a.r<b.r;
}

int res[N];  // 存储每个查询的答案

int main()
{
    // 输入数组长度和数组元素
    cin>>n;
    for(int i=1;i<=n;i++)
    {
        cin>>arr[i];
    }
    
    // 输入查询次数和查询区间
    cin>>m;
    for(int i=1;i<=m;i++)
    {
        cin>>que[i].l>>que[i].r;
        que[i].id=i;
    }
    
    // 离线排序查询
    sort(que+1,que+1+m,cmp);
    
    int cur=0;  // 当前遍历到的数组位置
    for(int i=1;i<=m;i++)
    {
        int nowr=que[i].r;
        int nid=que[i].id;
        int nl=que[i].l;
        
        // 遍历到当前查询的右端点
        while(cur<nowr)
        {
            cur++;
            // 该数字第一次出现，直接标记
            if(brr[arr[cur]]==0)
            {
                brr[arr[cur]]=cur;
                add(cur,1);
            }
            // 该数字已出现过，删除旧位置标记，更新为新位置
            else
            {
                add(brr[arr[cur]],-1);
                brr[arr[cur]]=cur;
                add(cur,1);
            }
        }
        
        // 计算区间[l,r]的不同元素个数
        res[nid]=sum(nowr)-sum(nl-1);
    }
    
    // 按原查询顺序输出答案
    for(int i=1;i<=m;i++)
    {
        cout<<res[i]<<endl;
    }
```