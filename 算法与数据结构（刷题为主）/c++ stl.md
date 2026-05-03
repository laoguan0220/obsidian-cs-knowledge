
---

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