---
title: HNOI2017 Day1 Solution
date: 2017-04-23 11:36:09
tags: 
    - Solution
    - Datastructure
---

### T1 Splay

#### Analysis:
&emsp;&emsp;由于这题具有良好的性质 -- (只旋转最大或者最小值)，所以我们可以手动模拟一下观察它的性质，不难发现，在我们旋转最小或者最大值到根节点的过程中,最终树的形态发生的变化是很少的:
&emsp;&emsp;最后我们只是将最大或者最小值所在位置的另一个方向的儿子接到它的父亲下面，然后将原树的根节点接在这个点上,那么这样的话，我们就可以用一棵LCT实时地维护这个过程。对于要求的答案，就相当于求这个点到根节点的距离，在维护LCT的同时就能实现。

<!--more-->

#### Code: 

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long LL;
typedef pair<int, int> pii;

#define fst first
#define snd second

const int oo = 0x3f3f3f3f;
const int maxn = 200000 + 10;

template<typename T> bool chkmin(T& a, T b) { return a > b ? a = b, 1 : 0; }
template<typename T> bool chkmax(T& a, T b) { return a < b ? a = b, 1 : 0; }

int read() {
    int n = 0, f = 1;
    char ch = getchar();
    for(;!isdigit(ch); ch = getchar()) if(ch == '-') f = -1;
    for(; isdigit(ch); ch = getchar()) n = n * 10 + ch - 48;
    return n * f;
}

namespace LCT {
#define rt(u) (!u || (c[fa[u]][0] != u && c[fa[u]][1] != u))

    int sz[maxn];
    int fa[maxn], c[maxn][2];
    
    void push_up(int u) {
        sz[u] = 1;
        if(c[u][0]) sz[u] += sz[c[u][0]];
        if(c[u][1]) sz[u] += sz[c[u][1]];
    }
    void rotate(int u) {
        int v = fa[u], w = fa[v];
        bool t = (c[v][1] == u);

        if(!rt(v)) c[w][c[w][1] == v] = u;
        c[v][t] = c[u][t^1], fa[c[u][t^1]] = v;
        c[u][t^1] = v, fa[v] = u, fa[u] = w;

        push_up(v); 
    }
    void splay(int u) {
        while(!rt(u)) {
            int v = fa[u], w = fa[v];
            if(!rt(v)) 
                rotate((c[v][1] == u) == (c[w][1] == v) ? v : u);
            rotate(u);
        }push_up(u);
    }
    int access(int u) {
        int v = 0;
        while(u) {
            splay(u);
            c[u][1] = v;
            push_up(u);
            u = fa[v = u];
        }return v;
    }
    void link(int u, int v) { fa[u] = v; }
    int Grt(int u) {
        access(u); splay(u);
        while(c[u][0]) u = c[u][0];
        splay(u);
        return u;
    }
    int remove(int u, int ch) {
        if(ch) {
            access(ch);

            fa[ch] = 0;
            if(c[u][0]) {
                fa[c[u][0]] = ch;
                c[ch][0] = c[u][0];
            }else c[ch][0] = 0;

            c[u][0] = c[u][1] = fa[u] = 0;

            return ch;
        }else {
            int t = c[u][0];
            c[u][0] = fa[c[u][0]] = 0;
            c[u][0] = c[u][1] = fa[u] = 0;

            return t;
        }
    }
    int query(int u) {
        access(u), splay(u);
        return sz[u];
    }
}

int tot;
set <pii> S;
set <pii> :: iterator it1, it2;
int c[maxn][2], fa[maxn];

void insert(int x) {
    ++tot;

    it1 = S.lower_bound(pii(x, tot));
    it2 = S.upper_bound(pii(x, tot));

    if(it1 != S.begin()) {
        --it1;
        if(!c[(*it1).snd][1]) { 
            fa[tot] = (*it1).snd;
            c[(*it1).snd][1] = tot;
            LCT :: link(tot, (*it1).snd);
        } 
    }
    if(it2 != S.end()) {
        if(!c[(*it2).snd][0]) { 
            fa[tot] = (*it2).snd;
            c[(*it2).snd][0] = tot;
            LCT :: link(tot, (*it2).snd);
        } 
    }
    S.insert(pii(x, tot));
    printf("%d\n", LCT::query(tot));
}

int u, rt;
void splmin(bool t) {
    it1 = S.begin(); u = (*it1).snd;
    printf("%d\n", LCT::query(u));

    rt = LCT::Grt(LCT::remove(u, c[u][1]));

    if(c[u][1]) {
        fa[c[u][1]] = fa[u];
        c[fa[u]][0] = c[u][1];
    }else c[fa[u]][0] = 0;

    fa[u] = 0;
    if(!t) {
        c[u][0] = 0;
        if(rt) {
            fa[rt] = u;
            c[u][1] = rt;
            LCT::fa[rt] = u;
        }
    }else S.erase(it1);
}
void splmax(bool t) {
    it1 = S.end(); --it1; u = (*it1).snd;
    printf("%d\n", LCT::query(u));

    rt = LCT::Grt(LCT::remove(u, c[u][0]));

    if(c[u][0]) {
        fa[c[u][0]] = fa[u];
        c[fa[u]][1] = c[u][0];
    }else c[fa[u]][1] = 0;

    fa[u] = 0;
    if(!t) {
        c[u][1] = 0;
        if(rt) {
            fa[rt] = u;
            c[u][0] = rt;
            LCT::fa[rt] = u;
        }
    }else S.erase(it1);
}

int main() {
    freopen("splay.in", "r", stdin);
    freopen("splay.out", "w", stdout);
    
    int m = read();
    while(m--) {
        static int op;
        op = read();
        if(op == 1) insert(read());
        if(op == 2) splmin(0);
        if(op == 3) splmax(0);
        if(op == 4) splmin(1);
        if(op == 5) splmax(1);
    }
    return 0;
}
```

### T2 Sf

#### Analysis:
&emsp;&emsp;我们可以首先利用单调栈在$O(n)$时间内求出每个$a[i]$之前值大于的第一个点$L[i]$和之后的第一个点$R[i]$,然后对每一个$a[i]$做最大值时分别统计对答案的贡献，考虑它产生q1的贡献时, 当且仅当选择的区间为$ (L[i], R[i]) $; 当它产生q2的贡献时，选择的区间为 $(j, R[i]) | j \in (L[i], i)$ 或者 $(L[i], j) | j \in (i, R[i])$。
&emsp;&emsp;那么这样之后问题就转化为一个二位数点的经典问题:如果我们将所有的区间$(l, r)$都抽象成点$(l, r)$,满足条件的点对就在询问点对的右下角，稍微麻烦的的是产生q2贡献的是一条线段，从下到上从右到左分别用主席树或者扫描线+线段树都可做。

#### Code:
```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long LL;
typedef pair<int, int> pii;

const int maxn = 200000 + 10;

int read() {
    int n = 0, f = 1;
    char ch = getchar();
    for(;!isdigit(ch); ch = getchar()) if(ch == '-') f = -1;
    for(; isdigit(ch); ch = getchar()) n = n * 10 + ch - 48;
    return n * f;
}

#define fst first
#define snd second
#define pb push_back
#define SZ(a) int((a).size())

namespace Seg_T {
 
    LL val[maxn << 2], add[maxn << 2];
    
#define lc (u << 1)
#define rc (lc | 1)
#define mid ((l + r) >> 1)

    void init() {
        memset(val, 0, sizeof val);
        memset(add, 0, sizeof add);
    }

    void push_down(int u, int l, int r) {
        if(add[u]) {
            add[lc] += add[u]; val[lc] += LL(mid-l+1) * add[u];
            add[rc] += add[u]; val[rc] += LL(r-mid) * add[u];
            add[u] = 0;
        }
    }
    void update(int u, int l, int r, int x, int y, int v) {
        if(x > y) return;
        if(x <= l && r <= y) {
            add[u] += v;
            val[u] += LL(r-l+1) * v;
            return;
        }
        push_down(u, l, r);
        if(x <= mid) 
            update(lc, l, mid, x, y, v);
        if(y > mid) 
            update(rc, mid+1, r, x, y, v);

        val[u] = val[lc] + val[rc];
    }
    LL query(int u, int l, int r, int x, int y) {
        if(x > y) return 0;
        if(x <= l && r <= y) return val[u];
        push_down(u, l, r);

        LL ans = 0;
        if(x <= mid) 
            ans += query(lc, l, mid, x, y);
        if(y > mid) 
            ans += query(rc, mid+1, r, x, y);
        return ans;
    }
}

int n, q, p1, p2;
int a[maxn], L[maxn], R[maxn];

vector<int> A[maxn];
vector<pii> Q1[maxn], Q2[maxn], A1[maxn], A2[maxn];

void init() {
    static int top; 
    static int stk[maxn];

    top = 0;
    stk[top++] = 0;
    for(int i = 1; i <= n; i++) {
        while(top > 1 && a[stk[top-1]] < a[i]) --top;
        L[i] = stk[top-1]; stk[top++] = i;
    }

    top = 0;
    stk[top++] = n+1;
    for(int i = n; i >= 1; i--) {
        while(top > 1 && a[stk[top-1]] < a[i]) --top;
        R[i] = stk[top-1]; stk[top++] = i;
    }

    for(int i = 1; i <= n; i++) {
        if(L[i] > 0 && R[i] <= n) A[L[i]].pb(R[i]);
        if(L[i] > 0) A1[L[i]].pb(pii(i+1, R[i]-1)); 
        if(R[i]<= n) A2[R[i]].pb(pii(L[i]+1, i-1));
    }

}

LL ans[maxn];
int main() {
    freopen("sf.in", "r", stdin);
    freopen("sf.out", "w", stdout);

    n = read(); q = read(); p1 = read(); p2 = read();
    for(int i = 1; i <= n; i++) a[i] = read();

    init();
    for(int i = 0; i < q; i++) {
        static int x, y;
        x = read(), y = read();
        ans[i] += (y-x) * p1;
        Q1[x].pb(pii(y, i));
        Q2[y].pb(pii(x, i));
    }

    Seg_T :: init();
    for(int i = n; i >= 1; i--) {
        for(int j = 0; j < SZ(A1[i]); j++) 
            Seg_T :: update(1, 1, n, A1[i][j].fst, A1[i][j].snd, p2);
        for(int j = 0; j < SZ(A[i]); j++)
            Seg_T :: update(1, 1, n, A[i][j], A[i][j], p1);
        for(int j = 0; j < SZ(Q1[i]); j++) 
            ans[Q1[i][j].snd] += Seg_T :: query(1, 1, n, 1, Q1[i][j].fst);
    }

    Seg_T :: init();
    for(int i = 1; i <= n; i++) {
        for(int j = 0; j < SZ(A2[i]); j++) 
            Seg_T :: update(1, 1, n, A2[i][j].fst, A2[i][j].snd, p2);
        for(int j = 0; j < SZ(Q2[i]); j++) 
            ans[Q2[i][j].snd] += Seg_T :: query(1, 1, n, Q2[i][j].fst, n);
    }

    for(int i = 0; i < q; i++) printf("%lld\n", ans[i]);
    return 0;
}
```

### T3 Gift

#### Analysis:
&emsp;&emsp;本质是求一个旋转方式，最小化：

$$ 
\begin{align} 
\sum_{i=1}^{n} (x_i-y_i+c) ^ {2} &= \sum_{i=1}^{n} (x_i-y_i)^{2} + 2c\sum_{i=1}^{n} (x_i-y_i) + nc^2 \\
&= \sum_{i=1}^{n} x_i^2 + \sum_{i=1}^{n} y_i^2 - 2\sum_{i=1}^{n} x_iy_i + 2c\sum_{i=1}^{n} (x_i - y_i) + nc^2 
\end{align}
$$

&emsp;&emsp;后面与c相关的部分就是一个二次函数，前面的两项是常数，那么我们的目标就变成了最大化:
$$ k \in [0, n)\ \ \  \sum_{i=1}^{n} x_iy_{i+k} | y_i = y_{i\ mod\ n} $$

&emsp;&emsp; 我们将x对应的数列翻转,则原式变为：
$$ \sum_{i=1}^{n} x_{n-i+1}\ y_{i+k} $$
&emsp;&emsp; 不难发现对于一个给定的k，其中每一项x与y的下标之和模n同余，循环卷积即可。

#### Code:
```cpp
#include<bits/stdc++.h>
using namespace std;
const int oo = INT_MAX;
const int maxn = 200000 + 10;
const double PI = acos(-1.0);

int base, dis;
int rev[maxn];

int read() {
    int n = 0, f = 1;
    char ch = getchar();
    for(;!isdigit(ch); ch = getchar()) if(ch == '-') f = -1;
    for(; isdigit(ch); ch = getchar()) n = n * 10 + ch - 48;
    return n * f;
}

void init(int N) {
    for(base = 1; base <= N; base <<= 1) ++dis;
    for(int i = 0; i < base; i++) 
        rev[i] = (rev[i>>1] >> 1) | ((i&1) << (dis-1));
}

void DFT(complex<double> x[], int N, int type) {
    for(int i = 0; i < N; i++) 
        if(i < rev[i]) swap(x[i], x[rev[i]]);

    for(int l = 2; l <= N; l <<= 1) {
        complex<double> wn(cos(2*PI/l), sin(2*PI*type/l));
        for(int i = 0; i < N; i += l) {
            complex<double> w(1, 0);
            for(int j = 0; j < (l >> 1); j++, w *= wn) {
                complex<double> L = x[i + j];
                complex<double> R = x[i + j + (l >> 1)] * w;
                x[i + j] = L + R;
                x[i + j + (l >> 1)] = L - R;
            }
        }
    }
}

int n;
int a[maxn], b[maxn];
complex<double> da[maxn], db[maxn];

int main() {
    freopen("gift.in", "r", stdin);
    freopen("gift.out", "w", stdout);

    n = read(); read();
    init(2 * n);
     
    long long ans = 0, B = 0;
    for(int i = 0; i < n; i++) a[i] = read(), ans += 1LL*a[i]*a[i], B += a[i];
    for(int i = 0; i < n; i++) b[i] = read(), ans += 1LL*b[i]*b[i], B -= b[i];

    B *= 2;
    long long k = (long long) floor(double(-B)/(2*n) + 0.5);

    ans += (n * k * k + k * B);

    reverse(b, b + n);
    for(int i = 0; i < n; i++) da[i] = a[i], db[i] = b[i];

    DFT(da, base, 1);
    DFT(db, base, 1);

    for(int i = 0; i < base; i++) da[i] *= db[i];
    DFT(da, base, -1);
    
    int tmp = -oo;
    for(int i = 0; i < n; i++) da[i] += da[i+n];
    for(int i = 0; i < n; i++) tmp = max(tmp, int((da[i].real()/base) + 0.5));

    ans -= tmp * 2;
    printf("%lld\n", ans);
    return 0;
}
```
