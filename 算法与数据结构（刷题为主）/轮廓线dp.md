# P1879 与 P2435 的轮廓线 DP 深度研究与 C++ 实现

这两道题非常适合放在一起学。P1879 的边界状态只需要记录“这一列有没有种玉米”，属于非常轻量的轮廓线状态；P2435 则需要在轮廓线上直接记录“这一位置的颜色”，已经很接近竞赛里常说的插头 DP / 轮廓线 DP 染色模型。洛谷题面给出的约束也正好说明了这种分层：P1879 的网格只有 12×12；P2435 虽然有很大的纵向规模，但第一行和最后一行固定，而且数据分组让基于轮廓线的做法成为可行路线。 citeturn1view0turn1view1turn0search4turn7view0

## 轮廓线 DP 是什么

OI Wiki 在“插头 DP”里把这类方法描述为：在状态压缩 DP 中，不去保存整个已经处理完的区域，而是只保存“已处理区域”与“未处理区域”交界处那条边界上的信息。OI Wiki 还明确指出，阶段既可以按“整行”划分，也可以按“逐格”划分；后者通常就被称为轮廓线 DP。也就是说，按行状态压缩和逐格轮廓线，在本质上都是“只记边界，不记内部”的同一种思想。 citeturn4view2turn4view3

轮廓线 DP 的核心思想可以浓缩成四句话。第一，先固定一个扫描顺序，最常见的是从上到下、从左到右。第二，当前决策以后，真正还会影响未来的，只是扫描边界附近的信息，所以只保留边界状态。第三，在当前位置做一个局部决策时，只检查和当前位置相邻、已经确定下来的那些边界信息。第四，做完决策之后，把轮廓线往前“推”一格，于是状态就完成了一次更新。OI Wiki 在介绍骨牌覆盖时，专门把“逐行划分阶段”和“逐格划分阶段”并列展示，强调的正是这种“边界推进”的思想。 citeturn4view3

放到这两道题里，P1879 只要求上下左右不能相邻种植，因此未来只需要知道上一行哪些位置已经种了；它适合用“按行轮廓线”。P2435 要求相邻点颜色不同，而未来是否可行只和当前轮廓线上每个位置的颜色值有关，因此适合用“逐格轮廓线”。这两个解法看起来风格不同，但核心都是：**未来只依赖边界，内部已经决定好的部分可以被遗忘**。 citeturn1view0turn1view1turn4view2turn4view3

## P1879 的状态设计与转移

P1879 的题意是：给定一个 M 行 N 列的土地矩阵，1 表示肥沃、0 表示不能种；选择若干格种植玉米，要求任意两块被选中的格子不能共边，并统计方案数，对 10^8 取模。题面还明确说明“一个都不选”也算一种方案。由于 M、N 都不超过 12，这题非常适合做状态压缩；而从轮廓线角度看，处理到第 i 行时，真正还会影响后续的，只是“第 i 行哪些位置种了”，因此把“一整行的选取情况”当成边界状态就够了。 citeturn1view0turn4view3

先把每一行编码成一个二进制掩码。第 i 行的肥沃情况记成 rowMask[i]，其中第 j 位为 1 表示这一格可以种。然后枚举所有一行的状态 s，如果一行内部没有左右相邻的两个 1，那么它就是合法行状态。这个条件可以写成：

$$
(s \;\&\; (s \ll 1)) = 0
$$

除此之外，状态 s 还必须完全落在肥沃土地上，也就是：

$$
(s \;\&\; \sim rowMask[i]) = 0
$$

两行之间的约束也很简单：如果第 i 行状态是 s，第 i-1 行状态是 t，那么同一列不能上下同时种植，于是需要：

$$
(s \;\&\; t) = 0
$$

于是就得到经典的按行轮廓线转移。设：

$$
dp[i][s]
$$

表示处理到第 i 行，且第 i 行采用状态 s 的方案数。那么有：

$$
dp[i][s] = \sum dp[i-1][t]
$$

其中求和只在同时满足下面三个条件的 t 上进行：

$$
(s \;\&\; \sim rowMask[i]) = 0
$$

$$
(t \;\&\; \sim rowMask[i-1]) = 0
$$

$$
(s \;\&\; t) = 0
$$

第一行的初始化最直接：只要状态 s 本身合法且完全落在第一行的肥沃土地上，方案数就是 1。最后把最后一行所有状态的方案数相加即可。由于 N 最大只有 12，单行合法状态的数量其实很少，远远小于 2^12；因此这个解法在题目约束下完全足够。 citeturn1view0

这题虽然很多人把它归类为“朴素状压 DP”，但从 OI Wiki 对“按行划分阶段”和“逐格划分阶段”本质相同的描述来看，它同样可以被理解成一个最轻量的轮廓线 DP：轮廓线不是一格一格推进，而是一整行一整行推进。洛谷题单中也把 P1879 同时标成了“朴素状压DP/轮廓线DP”。 citeturn4view3turn0search4

## P1879 的 C++ 实现与逐行解析

```cpp
#include <bits/stdc++.h>                    // 万能头文件
using namespace std;                        // 使用标准命名空间

const int MOD = 100000000;                  // 题目要求对 1e8 取模

int main() {
    ios::sync_with_stdio(false);            // 关闭 iostream 和 stdio 的同步
    cin.tie(nullptr);                       // 解除 cin 与 cout 的绑定，加速输入输出

    int M, N;                               // M 行，N 列
    cin >> M >> N;                          // 读入网格规模

    vector<int> fertile(M + 1, 0);          // fertile[i]：第 i 行可种位置的二进制掩码

    for (int i = 1; i <= M; ++i) {          // 逐行读入土地
        for (int j = 0; j < N; ++j) {       // 逐列读入
            int x;                          // 当前格子是否肥沃
            cin >> x;                       // 0 表示贫瘠，1 表示肥沃
            if (x == 1) {                   // 如果这一格可以种
                fertile[i] |= (1 << j);     // 就把第 j 位设成 1
            }
        }
    }

    vector<int> states;                     // 保存所有“本行内部不相邻”的合法状态
    for (int s = 0; s < (1 << N); ++s) {    // 枚举一行的所有子集
        if ((s & (s << 1)) == 0) {          // 若不存在左右相邻的两个 1
            states.push_back(s);            // 则这是一个合法行状态
        }
    }

    vector<int> dpPrev(states.size(), 0);   // dpPrev：上一行的 DP 数组
    vector<int> dpCur(states.size(), 0);    // dpCur：当前行的 DP 数组

    for (int i = 0; i < (int)states.size(); ++i) {   // 初始化第一行
        if ((states[i] & ~fertile[1]) == 0) {        // 若状态完全落在第一行的肥沃土地内
            dpPrev[i] = 1;                           // 那么这一种选法对应 1 种方案
        }
    }

    for (int row = 2; row <= M; ++row) {             // 从第二行开始向下转移
        fill(dpCur.begin(), dpCur.end(), 0);         // 先把当前行答案清零

        for (int i = 0; i < (int)states.size(); ++i) {   // 枚举当前行状态 s
            int s = states[i];                           // 取出当前行状态
            if ((s & ~fertile[row]) != 0) continue;     // 如果 s 用到了贫瘠格，直接跳过

            for (int j = 0; j < (int)states.size(); ++j) {   // 枚举上一行状态 t
                int t = states[j];                           // 取出上一行状态
                if ((t & ~fertile[row - 1]) != 0) continue; // 若 t 不合法，同样跳过
                if ((s & t) != 0) continue;                 // 上下同列不能同时种

                dpCur[i] += dpPrev[j];                      // 累加合法转移
                if (dpCur[i] >= MOD) dpCur[i] -= MOD;      // 取模，防止数值过大
            }
        }

        dpPrev.swap(dpCur);                                 // 当前行滚动为下一轮的上一行
    }

    int ans = 0;                                            // 保存最终答案
    for (int v : dpPrev) {                                  // 最后一行的所有合法状态都要统计
        ans += v;                                           // 累加方案数
        if (ans >= MOD) ans -= MOD;                         // 取模
    }

    cout << ans << '\n';                                    // 输出答案
    return 0;                                               // 正常结束程序
}
```

这份代码里最关键的三段分别对应三件事。第一段是合法状态预处理，也就是 `states` 的枚举；它把所有“本行内部不相邻”的掩码先筛出来，后面 DP 就不用反复检查横向冲突了。第二段是第一行初始化；由于题目允许“一个都不种”，所以全 0 状态天然也会被保留下来。第三段是双层状态转移，检查的其实就是两类局部约束：本行自身不能左右相邻、相邻两行不能上下相邻。整个实现和前面的递推式是一一对应的。 citeturn1view0turn4view3

如果你把这份代码当成轮廓线 DP 来理解，那么 `dpPrev` 里存的就是“当前轮廓线”——也就是刚处理完那一整行之后，边界上每一列是否已经被占用。下一行只会和这个边界发生联系，因此更早之前的所有内部细节都已经被压缩掉了。对初学者来说，P1879 是把“轮廓线只记录必要信息”这一件事看得最清楚的入门题之一。 citeturn4view3turn0search4

## P2435 的状态设计与转移

P2435 的题意是：给一个 n 行 m 列的格点图，用 k 种颜色给每个点染色，要求任意相邻点颜色都不同；第一行和最后一行的颜色已经固定，问总方案数，对 376544743 取模。题目的分组非常关键：有一组数据允许 n 到 10^7、m 到 10^5，但只在 k≤2 时出现；而当 k 到 4 时，m 最大只有 8，n 最大只有 100，题面还特别说明 n、m、k 不会同时达到最大值。也就是说，这题必须按分情况的方法来做。 citeturn1view1

先说最容易漏掉的边界情况。若 n=1，那么第一行和最后一行其实是同一行，所以只有在“给定的两行完全相同且自身横向合法”时答案才是 1，否则答案是 0。若 n=2，那么中间根本没有自由行，只有在第一行、最后一行都横向合法，并且每一列上下颜色不同的时候答案才是 1，否则是 0。若 k=1，那么除了只有一个点的情况以外，只要图中存在一条边，就不可能合法染色。以上都属于题意本身直接推出的结论。 citeturn1view1

真正有意思的是 k=2 的情况。因为只有两种颜色，只要一行横向相邻点都不同，这一行就必然是交替颜色；再加上竖向相邻也要不同，所以下一行就一定是上一行的逐位取反。于是，当第一行固定以后，整个棋盘其实已经被完全确定了，答案只能是 0 或 1。形式化地说，对任意一列 j，都必须满足：

$$
row_{i+1}[j] = row_i[j] \oplus 1
$$

于是最后一行必须满足：

$$
last[j] = first[j] \oplus ((n-1)\bmod 2)
$$

只要检查这一条件是否成立即可。由于题目确实存在 n≤10^7、m≤10^5、k≤2 的数据组，这个特判不是“优化”，而是正解不可缺的一部分。 citeturn1view1

接下来才是你要求的轮廓线 DP 主体。对于 k=3 或 k=4，如果仍然把“整行颜色方案”当作状态，那么一行合法染色数最多会达到：

$$
4 \cdot 3^{7} = 8748
$$

若再做两行之间的整行兼容转移，复杂度近似是：

$$
O(n \cdot S^2)
$$

在 n≤100 的组别里，最坏估算会到 7.6×10^9 量级，明显不合适。逐格轮廓线 DP 的好处就在这里：它把“整行和整行之间的巨大转移”拆成了“每次只处理一个点的局部转移”。洛谷上关于 P2435 的题解也正是按这个思路使用 base-4 轮廓线状态来做。 citeturn1view1turn7view0

状态如何设计？因为 k≤4，所以轮廓线上的每个位置只需要 2 bit 就能表示它的颜色。我们令轮廓线长度为 m+1，用一个 base-4（也可以看成每位 2 bit）的整数来编码它。洛谷题解里采用的也是“m+1 位四进制”的编码方式：处理到第 i 行第 j 列时，轮廓线状态的第 j-1 位表示左侧相邻点的颜色，第 j 位表示上侧相邻点的颜色。这样一来，当前位置能否放颜色 c，只需要检查：

$$
c \ne left
$$

$$
c \ne up
$$

并且如果当前已经在倒数第二行，那么还要保证它和固定的最后一行不同：

$$
c \ne last[j]
$$

这就是一个典型的“只看边界邻居”的轮廓线决策。 citeturn7view0

为什么初始化只需要把第一行放进状态里？因为第一行已经固定好了，而我们真正要枚举的是第 2 行到第 n-1 行这些“中间行”。在进入某一中间行之前，把当前状态整体左移 2 bit，相当于在轮廓线最左边补出一个“空槽位”，于是对第 j 列来说，新的第 j 位正好对应“上方颜色”，新的第 j-1 位正好对应“左方颜色”。这是整份程序里最漂亮的一步：一条位运算，就把新一行开始时的轮廓位置全部摆正了。 citeturn7view0

若把当前位置记为 (i,j)，当前状态记为 state，则更新可以写成下面这个形式。先读出：

$$
L = state[j-1], \quad U = state[j]
$$

枚举可选颜色 c 以后，把这两个位置清空，再把 c 写回轮廓线：

$$
state[j-1] \leftarrow c
$$

$$
state[j] \leftarrow 
\begin{cases}
c, & j < m \\
0, & j = m
\end{cases}
$$

最后一个式子的含义是：如果还没到这一行的末尾，当前点会继续成为“下一格的左邻居 / 未来的边界颜色”；如果已经到行尾，那么这一位在下一次推进时自然会被挪走。由于题目里在 k=3,4 的相关组别中 m 很小，轮廓线状态总空间最多是：

$$
4^{m+1} \le 4^9 = 262144
$$

因此用活跃状态表加滚动数组来做是完全可行的。 citeturn1view1turn7view0

## P2435 的 C++ 实现与逐行解析

```cpp
#include <bits/stdc++.h>                          // 万能头文件
using namespace std;                              // 使用标准命名空间

static const int MOD = 376544743;                 // 题目给定模数
static const int MAX_STATE = 1 << 18;             // 4^9 = 2^18，足够覆盖 m<=8 时的所有轮廓线状态

int main() {
    ios::sync_with_stdio(false);                  // 关闭同步，加速输入
    cin.tie(nullptr);                             // 解绑 cin / cout

    int n, m, k;                                  // n 行，m 列，k 种颜色
    cin >> n >> m >> k;                           // 读入规模

    vector<int> first(m + 1), last(m + 1);        // first：第一行颜色；last：最后一行颜色
    for (int i = 1; i <= m; ++i) cin >> first[i]; // 读入第一行
    for (int i = 1; i <= m; ++i) cin >> last[i];  // 读入最后一行

    auto rowLegal = [&](const vector<int>& row) { // 判断一行自身是否横向合法
        for (int i = 1; i < m; ++i) {             // 枚举相邻两列
            if (row[i] == row[i + 1]) {           // 若相邻点颜色相同
                return false;                     // 则该行不合法
            }
        }
        return true;                              // 否则这一行横向合法
    };

    if (!rowLegal(first) || !rowLegal(last)) {    // 若首行或末行本身就不合法
        cout << 0 << '\n';                        // 直接无解
        return 0;                                 // 结束程序
    }

    if (n == 1) {                                 // 只有一行时
        cout << (first == last ? 1 : 0) << '\n';  // 首尾其实是同一行，必须完全相同
        return 0;                                 // 结束程序
    }

    if (k == 1) {                                 // 只有一种颜色时
        cout << 0 << '\n';                        // 只要图中存在边，就不可能合法
        return 0;                                 // 这里 n>=2，所以必定无解
    }

    if (n == 2) {                                 // 只有两行时，没有中间行需要枚举
        for (int j = 1; j <= m; ++j) {            // 枚举每一列
            if (first[j] == last[j]) {            // 若上下同色
                cout << 0 << '\n';                // 则不合法
                return 0;                         // 直接结束
            }
        }
        cout << 1 << '\n';                        // 否则唯一方案就是题面给定的这两行
        return 0;                                 // 结束程序
    }

    if (k == 2) {                                 // 两种颜色时，整张图被第一行唯一决定
        for (int j = 1; j <= m; ++j) {            // 枚举每一列
            int expected = first[j] ^ ((n - 1) & 1); // 第 n 行应当等于第一行按奇偶翻转后的颜色
            if (last[j] != expected) {            // 若与给定末行不一致
                cout << 0 << '\n';                // 则无解
                return 0;                         // 结束程序
            }
        }
        cout << 1 << '\n';                        // 否则整张图只有这一种方案
        return 0;                                 // 结束程序
    }

    int pw4[10];                                  // pw4[i] = 4^i，用于四进制编码
    pw4[0] = 1;                                   // 4^0 = 1
    for (int i = 1; i <= 9; ++i) {                // 预处理到 4^9 即可
        pw4[i] = pw4[i - 1] * 4;                  // 递推得到 4 的幂
    }

    auto encodeRow = [&](const vector<int>& row) { // 把一整行编码成 base-4 状态
        int state = 0;                             // 初始状态为 0
        for (int i = 1; i <= m; ++i) {             // 枚举每一列
            state |= row[i] << (2 * (i - 1));      // 每个颜色占 2 bit
        }
        return state;                              // 返回编码后的状态
    };

    static int buf0[MAX_STATE], buf1[MAX_STATE], buf2[MAX_STATE];
    int* cur = buf0;                               // cur：当前活跃状态对应的方案数
    int* nxt = buf1;                               // nxt：处理下一格后的方案数
    int* tmp = buf2;                               // tmp：进入新一行时“整体左移 2 bit”用的临时数组

    vector<int> curList, nxtList, tmpList;         // 只保存当前非零状态，避免整表枚举

    int startState = encodeRow(first);             // 初始轮廓线就是固定的第一行
    cur[startState] = 1;                           // 初态方案数为 1
    curList.push_back(startState);                 // 把初态加入活跃列表

    int ans = 0;                                   // 最终答案

    auto addState = [&](int state, int ways) {     // 向 nxt 中插入 / 累加一个状态
        if (nxt[state] == 0) {                     // 如果这个状态第一次出现
            nxtList.push_back(state);              // 就把它加入活跃列表
        }
        nxt[state] += ways;                        // 累加方案数
        if (nxt[state] >= MOD) nxt[state] -= MOD;  // 取模
    };

    for (int row = 2; row <= n - 1; ++row) {       // 逐行处理所有中间行
        tmpList.clear();                           // 临时状态列表清空

        for (int s : curList) {                    // 枚举当前所有活跃状态
            int ns = s << 2;                       // 整体左移 2 bit，给新一行左边界补一个空槽
            tmp[ns] = cur[s];                      // 方案数原样搬过去
            tmpList.push_back(ns);                 // 记录新的活跃状态
            cur[s] = 0;                            // 清空旧数组，方便下次复用
        }

        curList.swap(tmpList);                     // 当前活跃状态变成移位后的状态
        swap(cur, tmp);                            // cur 指向真正的“新一行起始状态表”

        for (int col = 1; col <= m; ++col) {       // 逐格处理这一行中的每个点
            nxtList.clear();                       // 下一轮状态列表清空

            for (int s : curList) {                // 枚举所有当前活跃状态
                int ways = cur[s];                 // 取出该状态的方案数

                int leftColor = (s >> (2 * (col - 1))) & 3; // 取出轮廓线第 col-1 位：左邻居颜色
                int upColor   = (s >> (2 * col)) & 3;       // 取出轮廓线第 col 位：上邻居颜色

                int baseState = s                              // 先从原状态开始
                              - leftColor * pw4[col - 1]       // 清掉第 col-1 位
                              - upColor   * pw4[col];          // 清掉第 col 位

                if (col == 1) leftColor = -1;                  // 第一列没有左邻居，用 -1 表示“空”

                for (int c = 0; c < k; ++c) {                  // 枚举当前位置染成哪一种颜色
                    if (c == leftColor || c == upColor) {      // 不能与左邻居或上邻居同色
                        continue;                              // 不合法，跳过
                    }

                    if (row == n - 1 && c == last[col]) {      // 若当前在倒数第二行
                        continue;                              // 还必须与固定的最后一行不同色
                    }

                    if (row == n - 1 && col == m) {            // 如果已经处理到最后一个自由点
                        ans += ways;                           // 直接把贡献加进答案
                        if (ans >= MOD) ans -= MOD;            // 取模
                    } else {
                        int ns = baseState + c * pw4[col - 1]; // 当前点会写回到第 col-1 位
                        if (col < m) {                         // 只要还没到行尾
                            ns += c * pw4[col];                // 当前点也会成为新的“上 / 边界”颜色
                        }
                        addState(ns, ways);                    // 把新状态加入下一轮
                    }
                }

                cur[s] = 0;                                    // 当前状态处理完，清零以复用数组
            }

            curList.swap(nxtList);                             // 进入下一格时，当前状态变成刚转移出的状态
            swap(cur, nxt);                                    // 交换 cur / nxt 的角色
        }
    }

    cout << ans << '\n';                                       // 输出答案
    return 0;                                                  // 正常结束
}
```

这份程序最难懂、但也最值得掌握的地方，一共有四个。第一，`startState = encodeRow(first)`：它表示轮廓线的初始信息根本不需要从空状态开始推，而是直接等于“固定的第一行颜色”。第二，进入新一行时的 `s << 2`：这一步等价于在轮廓线最左边塞入一个空槽，从而把“上一行的第 j 列颜色”自动对齐到“当前格子的上邻居位置”。第三，`baseState` 那两行减法：它们是在四进制意义下把轮廓线的第 `col-1` 位和第 `col` 位清空，方便把当前位置的新颜色写回去。第四，在 `row == n - 1 && col == m` 时不再入表而是直接加到答案里，因为这说明所有自由点都已经染完了。整个程序的所有位运算，本质上都在做一件事：**沿着轮廓线把左邻居和上邻居传给下一格**。 citeturn1view1turn7view0

如果你把它和 P1879 对比，会发现它们的差别并不在“是不是 DP”，而是在“边界上要存什么”。P1879 的边界位只有 0/1 两种含义；P2435 的边界位必须记录具体颜色值，所以每个位置需要 2 bit。也正因为如此，P2435 更能体现轮廓线 DP 的味道：你不是在做整行和整行的转移，而是在做“当前位置和它周围边界”的极局部决策。 citeturn4view3turn8view0turn1view1

## 两题放在一起看

如果把这两题放进同一张脑图里，最重要的结论是：**轮廓线 DP 不是某一种固定模板，而是一种状态设计原则**。当未来只需要知道“轮廓线上有没有被占用”时，像 P1879 这样按行记录一个二进制掩码就够了；当未来需要知道“轮廓线上具体是什么颜色”时，像 P2435 这样逐格推进、在轮廓线上存颜色值就更自然。OI Wiki 对“按行阶段”和“逐格阶段”的并列介绍，恰好说明了这种统一性。 citeturn4view3

再往前走一步看，这两题其实也说明了“要不要上更重的插头 DP”这个判断标准。OI Wiki 把插头 DP 描述成需要对连通性进行编码的状态压缩问题；但这两题都不要求维护连通块结构，所以根本不需要最小表示、括号匹配、连通块重标号这些更复杂的内容。P1879 只存占用；P2435 只存颜色；这已经足够决定未来是否合法。换句话说，**轮廓线 DP 的精髓不是把状态做复杂，而是把状态做到刚刚好**。 citeturn4view2turn8view0

最后给一个很实用的记忆方式。遇到网格 DP 时，先问自己三个问题。第一，扫描顺序固定以后，未来到底会依赖过去的哪些信息？第二，这些信息是不是只落在“已处理区和未处理区的边界”上？第三，这个边界上的每个位置，记录 1 bit、2 bit，还是需要记录更复杂的连通性编号？如果你能把这三个问题答清楚，轮廓线 DP 的状态通常就已经设计出来了。P1879 的答案是“记录是否种植”；P2435 的答案是“记录颜色”；这正是它们最适合作为一组对照题来学习的原因。 citeturn1view0turn1view1turn4view2turn4view3

![[Pasted image 20260609214239.png]]
```cpp
#include <bits/stdc++.h>

using namespace std;

#define endl "\n"

typedef long long ll;

const int N = 1e5 + 5;

const int M=1e8;

////////////

int m,n,tot;

int a[13];

int f[21][1<<12+5];

int st[1<<12+5];

//////

void build()

{

    for(int i=0;i<(1<<m);i++)

    {

        if(i&(i<<1))

        continue;

        st[++tot]=i; ///记录下来一行当中所有满足条件的状态

    }

}

//

/*

2 3

1 1 1

0 1 0

*/

//部分状态确定法 选择特征鲜明的状态对整体分类

int main()

{

    ios::sync_with_stdio(false);

    cin.tie(nullptr);

    cout.tie(nullptr);

    cin>>n>>m;

    for(int i=1;i<=n;i++)

    {

        for(int j=1;j<=m;j++)

        {

            int x;

            cin>>x;

            a[i]+=(x<<(m-j));

        }

    }

    build();

    for(int i=1;i<=tot;i++)

    {

        if(!((st[i]|a[1])==a[1]))

        continue;

        f[1][st[i]]=1;

    }

    for(int i=2;i<=n;i++)

    {

        for(int j=1;j<=tot;j++)

        {

            int cur=st[j];

            if(!((cur|a[i])==a[i]))

            continue;

            for(int k=1;k<=tot;k++)

            {

                int last=st[k];

                if(!((last|a[i-1])==a[i-1]))

                continue;

                if(cur&last)

                continue;

                f[i][cur]=(f[i][cur]+f[i-1][last])%M;

            }

        }

    }

    int ans=0;

    for(int i=1;i<=tot;i++)

    {

        int cur=st[i];

        ans=(ans+f[n][cur])%M;

    }

    cout<<ans<<endl;

}
版本2
#include <bits/stdc++.h>

using namespace std;

#define endl "\n"

typedef long long ll;

inline ll Read()

{

    ll x = 0, f = 1;

    char c = getchar();

    while (c != '-' && (c < '0' || c > '9')) c = getchar();

    if (c == '-') f = -f, c = getchar();

    while (c >= '0' && c <= '9') x = (x << 3) + (x << 1) + c - '0', c = getchar();

    return x * f;

}

const int N = 1e5 + 5;

const int M=1e8;    

////////////

int n,m;

int a[13][13];

int dp[13][13][1<<12+1];

//////

int setnum(int s,int j,int num)

{

    return num==0?(s&(~(1<<j))):(s|(1<<j));

}//将状态的某一位的值设置为1或者是0

int getnum(int s,int j)

{

    return (s>>j)&1;

}//得到状态的某一位数值

int dfs(int i,int j,int s)

{

    if(i==n+1)

    {

        return 1;

    }

    if(j==m+1)

    {

        return dfs(i+1,1,s);

    }

    if(dp[i][j][s]!=-1)

    {

        return dp[i][j][s];

    }

    int ans=dfs(i,j+1,setnum(s,j-1,0));//不选

//如果要选的话也必须满足相应的选择条件

    if(a[i][j]==1&&(j==1||getnum(s,j-2)==0)&&getnum(s,j-1)==0)

    {

        ans=(ans+dfs(i,j+1,setnum(s,j-1,1)))%M;

    }

    dp[i][j][s]=ans;

    return ans;

}

//s记录的是同一行(1,j-1)的状态 上一行（j,m)的状态

int main()

{

    ios::sync_with_stdio(false);

    cin.tie(nullptr);

    cout.tie(nullptr);

    cin>>n>>m;

    for(int i=1;i<=n;i++)

    {

        for(int j=1;j<=m;j++)

        {

            cin>>a[i][j];

        }

    }

    for(int i=1;i<=n;i++)

    {

        for(int j=1;j<=m;j++)

        {

            for(int k=0;k<(1<<m);k++)

            {

                dp[i][j][k]=-1;

            }

        }

    }

    cout<<dfs(1,1,0)<<endl;

}
版本3 空间压缩版本
#include <bits/stdc++.h>

using namespace std;

  

const int MAXN = 12;

const int MAXM = 12;

const int MOD = 100000000;

  

int n, m, maxs;

int grid[MAXN][MAXM];

int dp[MAXM + 1][1 << MAXM];

int prepare[1 << MAXM];

  

int get(int s, int j) {

    return (s >> j) & 1;

}

  

int setBit(int s, int j, int v) {

    return v == 0 ? (s & (~(1 << j))) : (s | (1 << j));

}

  

int compute() {

    for (int s = 0; s < maxs; s++) {

        prepare[s] = 1;

    }

  

    for (int i = n - 1; i >= 0; i--) {

        // j == m

        for (int s = 0; s < maxs; s++) {

            dp[m][s] = prepare[s];

        }

  

        // 普通位置

        for (int j = m - 1; j >= 0; j--) {

            for (int s = 0; s < maxs; s++) {

                // 不种草

                int ans = dp[j + 1][setBit(s, j, 0)];

  

                // 种草

                if (grid[i][j] == 1 &&

                    (j == 0 || get(s, j - 1) == 0) &&

                    get(s, j) == 0) {

                    ans = (ans + dp[j + 1][setBit(s, j, 1)]) % MOD;

                }

  

                dp[j][s] = ans;

            }

        }

  

        // 设置 prepare

        for (int s = 0; s < maxs; s++) {

            prepare[s] = dp[0][s];

        }

    }

  

    return dp[0][0];

}

  

int main() {

    ios::sync_with_stdio(false);

    cin.tie(nullptr);

  

    cin >> n >> m;

    maxs = 1 << m;

  

    for (int i = 0; i < n; i++) {

        for (int j = 0; j < m; j++) {

            cin >> grid[i][j];

        }

    }

  

    cout << compute() << '\n';

  

    return 0;

}
```
