
---
c++里面对负数进行取模其结果还是负数

# 快速速查表

表格

|需求|代码|
|---|---|
|删除**最后一个元素**|`v.pop_back();`|
|删除**第一个元素**|`v.erase(v.begin());`|
priority_queue如何自定义排序规则
![[Pasted image 20260503193701.png]]
typedef pair<Int,int>pii; pair是两元组的意思 重命名为pii
pii(a,b);这个是一个二元组 而struct使用{a,b,c};
priority_queue它的堆顶是top() 而queue它的顶是front()
比较规则写成 pii a和pii b就行了
memset(ptr,value,num);
按照字节逐个赋值为value
# sort和 unique的配合使用

先看你代码：

cpp

运行

```
sort(b + 1, b + 1 + n);
int tot = unique(b + 1, b + 1 + n) - b - 1;
```

前提：数组 `b` 是**下标从 1 开始**。

---

## 举个具体例子

设：

`n = 5`

原数组：

plaintext

```
b[1]=5, b[2]=2, b[3]=5, b[4]=7, b[5]=2
```

### 第一步：`sort(b+1, b+1+n)`

排序后数组变成：

plaintext

```
b[1]=2, b[2]=2, b[3]=5, b[4]=5, b[5]=7
```

### 第二步：`unique(b+1, b+1+n)`

`unique` 作用：

把**相邻重复元素挤到后面**，返回**去重区域末尾的下一个地址**。

处理后数组逻辑：

plaintext

```
有效去重部分：2,5,7
后面剩下的是多余重复垃圾：2,5
```

`unique` 返回的指针 指向 **7 的下一个位置**。

### 第三步：`unique(...) - b - 1`

- `unique(...) - b` 得到这个指针对应的**数组下标**
- 再 `-1`，就是**去重后一共有多少个不同元素**

本例：

去重后不同元素是 `2,5,7`

`tot = 3`

---

## 公式拆解（记住就行）

数组从 **1 下标** 开始：

cpp

运行

```
sort(b+1, b+1+n);
int tot = unique(b+1, b+1+n) - b - 1;
```

- `tot` = **离散化后元素总个数**
- 之后用 `lower_bound(b+1, b+1+tot, x) - b` 就能拿到离散化编号

---

## 如果是 0 下标数组怎么写？

对比记一下：

cpp

运行

```
sort(b, b+n);
int tot = unique(b, b+n) - b;
```

---

## 一句话总结

1. 先 `sort` 让相同数挨在一起；
2. `unique` 把重复的扔后面，返回分界地址；
3. `- b - 1` 换算成**不同元素的总数**，给离散化用。