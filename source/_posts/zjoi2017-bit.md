---
title: ZJOI2017 树状数组
date: 2017-06-12 20:04:19
tags: 
    - Datastructure
---

### Description
> 给你一个长度为 $N$ 的序列以及 $M$ 次操作, 每次等概率地修改 $[L_i, R_i]$ 区间中一个值的奇偶性, 或者对于询问$[L_i,R_i]$区间中的和的奇偶性, 判断给定的一种错误算法输出正确答案的概率是多少
$ N, M <= 10^5 $
<!--more-->

### Solution

考虑这个错误算法, 由于可以把 $ + lowbit $ 和 $ - lowbit $ 都看成在一棵树上向父亲节点跳, 根据这个性质可以知道错误算法求了一个后缀和, 所以当且仅当对于询问区间 $[L_i, R_i]$, 满足 $ a[L_i-1] = a[R_i] $ 时它会输出正确答案

那么我们得到了一个 $ O(nm) $ 的暴力算法, 即每一个询问都暴力扫之前的所有修改计算概率即可.

然而这样还不能通过所有数据, 考虑将区间 $[L_i, R_i]$ 表示成二维平面上的一个点 $ (L_i, R_i)$.

然后用一个二维线段树去维护每个点代表的两个端点的值不相等的概率.显然地, 这个概率是可以很方便的合并的.
这样对于每次修改$[L_i, R_i]$的操作, 相当于:
- 将 $ x \in [L_i, R_i], y \in [L_i, R_i] $ 中的点 $(x, y)$ 与 $ \frac{2}{R_i - L_i+1} $ 合并.
- 将 $ x \in [L_i, R_i], y \in (R_i, N] $ 以及 $ x \in [0, L_i), y \in [L_i, R_i] $ 与 $ \frac{1}{R_i - L_i+1} $ 合并.

### Hint
注意到对于 $ L_i = 1 $ 的情况, $ Find(0) $ 直接返回了 $0$ 而非 $0$ 的后缀和
这个时候就要特判一下了, 注意到每次不管如何修改, 最终的正确答案是 $R_i$ 的前缀和, 而题中所给的方法输出的答案是 $R_i$ 的后缀和.

- 当前修改次数为偶数,  则当且仅当 $R_i = 0$ 前后缀和相同, 相当于查询 $a[R_i] = 0$ 的概率.
- 当前修改次数为奇数,  则当且仅当 $R_i = 1$ 前后缀和相同, 相当于查询 $a[R_i] = 1$ 的概率.

因为 $a[0]$ 不可能被修改, 所以上述问题也可以转化为查询 $[L_i, R_i]$ 相等概率.

### Code 

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
typedef pair<int, int> pii;

#define fst first
#define snd second
#define pb push_back
#define getchar getchar_unlocked

template <typename T> inline bool chkmax(T& a, T b) { return a < b ? a = b, 1 : 0; }
template <typename T> inline bool chkmin(T& a, T b) { return a > b ? a = b, 1 : 0; }

const int oo = 0x3f3f3f3f;
const int maxn = 1e5 + 5;
const int mod = 998244353;

int read() {
    int n = 0, f = 1;
    char ch = getchar();
    for( ;!isdigit(ch); ch = getchar()) if(ch == '-') f = -1;
    for( ; isdigit(ch); ch = getchar()) n = n * 10 + ch - 48;
    return n * f;
}

inline int merge(int a, int b) {
    return (a * (1LL - b + mod) % mod + b * (1LL - a + mod) % mod) % mod;
}
int N, M, prob;
namespace Seg_Tree {
    const int SZ_X = (maxn << 2) + 5;
    const int SZ_Y = (maxn * 400) + 5;

    int rt[SZ_X];
    int val[SZ_Y];
    int lc[SZ_Y], rc[SZ_Y], cnt;

#define LC (u << 1) 
#define RC (LC | 1)
#define mid ((l+r) >> 1)

    void Add_Y(int& u, int l, int r, int x, int y) {
        if(!u) u = ++cnt;
        if(x <= l && r <= y) {
            val[u] = merge(val[u], prob);
            return;
        }
        if(x <= mid) Add_Y(lc[u], l, mid, x, y);
        if(y > mid) Add_Y(rc[u], mid+1, r, x, y);
    }
    void Add_X(int u, int l, int r, int x, int y, int x0, int y0) {
        if(x <= l && r <= y) {
            Add_Y(rt[u], 0, N, x0, y0);
            return;
        }
        if(x <= mid) Add_X(LC, l, mid, x, y, x0, y0);
        if(y > mid) Add_X(RC, mid+1, r, x, y, x0, y0);
    }
    int query_Y(int u, int l, int r, int x) {
        if(l == r) return val[u];
        return merge(val[u], 
                x <= mid ? query_Y(lc[u], l, mid, x) : query_Y(rc[u], mid+1, r, x));
    }
    int query_X(int u, int l, int r, int x, int x0) {
        if(l == r) return query_Y(rt[u], 0, N, x0);

        return merge(query_Y(rt[u], 0, N, x0), 
                x <= mid ? query_X(LC, l, mid, x, x0) : query_X(RC, mid+1, r, x, x0));
    }
}

int fpm(int base, int exp) {
    int res = 1;
    for(; exp > 0; exp >>= 1) {
        if(exp & 1) 
            res = 1LL * res * base % mod;
        base = 1LL * base * base % mod;
    }
    return res;
}

int tot = 0;
void solve() {
    N = read(); M = read();
    while(M--) {
        static int ty, l, r;
        ty = read(); l = read(); r = read();
        if(ty == 1) {
            static LL tmp; 
            tmp = fpm(r-l+1, mod-2);
            prob = 2*tmp; Seg_Tree::Add_X(1, 0, N, l, r, l, r); 
            prob = tmp; Seg_Tree::Add_X(1, 0, N, l, r, r+1, N);
            prob = tmp; Seg_Tree::Add_X(1, 0, N, 0, l-1, l, r);
            ++ tot;
        }else {
            static int ans;
            ans = (1-Seg_Tree::query_X(1, 0, N, l-1, r)+mod) % mod;
            if(!(l-1) && (tot & 1))
                ans = (1-ans+mod) % mod;
            printf("%d\n", ans);
        }
    }
}
int main() {
#ifndef ONLINE_JUDGE
    freopen("data.txt", "r", stdin);
    freopen("ans.txt", "w", stdout);
#endif

    solve();
    return 0;
}
```
