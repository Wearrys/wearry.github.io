---
title: NOI2016 网格
date: 2017-06-10 14:36:06
tags:
    - Graph Theory
---

### Description

> 给出一个 $ N \times M $ 的网格图和图上的 $ C $ 个障碍物, 求最少删去多少个点可使得原图空格不四连通.
$ N, M <= 10 ^ 9, C <= 10 ^ 5 $

<!--more-->

### Solution

我们可以发现, 可能的答案只有 $ \{ -1, 0, 1, 2 \} $ 几种.

考虑一些简单的情况:
答案等于 $ -1 $ 时, 点数小于 $2$ 或者恰好有两个相邻的点.
答案为 $ 0 $ 时, 显然原图不连通.

那么就只需知道答案是否为 $ 1 $, 发现答案等于 $ 1 $ 当且仅当原图存在割点, 暴力的话不难做到 $ O(n * m) $.

将到达每个障碍的曼哈顿距离不超过 $2$ 的空格给提出来, 然后在这些点中找出一个割点, 满足到达最近的障碍的距离不超过 $1$.  
这样的点就一定是原图中的割点.

### Code

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
typedef pair<int, int> pii;

#define fst first
#define snd second
#define pb push_back

template <typename T> bool chkmax(T& a, T b) { return a < b ? a = b, 1 : 0; }
template <typename T> bool chkmin(T& a, T b) { return a > b ? a = b, 1 : 0; }

const int maxn = 2.5e6 + 10;
const int oo = 0x3f3f3f3f;

template<typename T> T read() {
    T n = 0, f = 1;
    char ch = getchar();
    for( ;!isdigit(ch); ch = getchar()) if(ch == '-') f = -1;
    for( ; isdigit(ch); ch = getchar()) n = n * 10 + ch - 48;
    return n * f;
}

struct Hash_Map {
    static const int mod = 1666667;

    int cnt = 0;
    int st[mod], nxt[maxn], X[maxn], Y[maxn];

    inline void clear() {
        cnt = 0;
        memset(st, 0, sizeof st);
    }
    inline int idx(int x, int y) {
        return ((233LL*x + (y^888))%mod + mod)%mod;
    }
    int find(int x, int y) {
        int u = idx(x, y);
        for(int i = st[u]; i; i = nxt[i]) 
            if(X[i] == x && Y[i] == y) 
                return i;
        return -1;
    }
    int insert(int x, int y) {
        int u = idx(x, y);

        for(int i = st[u]; i; i = nxt[i]) 
            if(X[i] == x && Y[i] == y) 
                return i;

        ++ cnt;
        X[cnt] = x, Y[cnt] = y;
        nxt[cnt] = st[u]; st[u] = cnt;
        return cnt;
    }
}HM;

LL n, m, c;
const int dx[] = { 0, 1, 0, -1, 1, -1, 1, -1 };
const int dy[] = { 1, 0, -1, 0, 1, -1, -1, 1 };

int st[maxn], nxt[maxn << 3], to[maxn << 3], ecnt = 1;
void addedge(int x, int y) { 
    to[++ecnt] = y; nxt[ecnt] = st[x]; st[x] = ecnt;
    to[++ecnt] = x; nxt[ecnt] = st[y]; st[y] = ecnt;
}

int vis[maxn];
int mark[maxn], now = 0;

void init() {
    ecnt = 1;
    HM.clear();
    memset(st, 0, sizeof st);
}

int flood_fill(int u) {
    int res = 1;
    mark[u] = now;
    for(int i = st[u], v; i; i = nxt[i]) if(mark[v = to[i]] != now) 
        res += flood_fill(v);
    return res;
}

int area_count(int s) { return ++ now, flood_fill(s); }

int chk() {
    if(n*m-c < 2) return -1;
    int res = area_count(c + 1);
    if(n*m == 2 || (res == 2 && n*m-c == 2)) return -1;
    return -2;
}

bool flag;
int dfn[maxn], low[maxn], dfs_clock = 0;
bool dfs(int u, int fa, bool f = false) {
    low[u] = dfn[u] = ++dfs_clock;

    for(int i = st[u], v; i; i = nxt[i]) if((v = to[i]) ^ fa) {
        if(!dfn[v]) {
            if(dfs(v, u)) return true;

            if(vis[u] == 1 && (low[v] > dfn[u] || (!f && low[v] == dfn[u]))) {
                return true;
            }
            chkmin(low[u], low[v]);
        }else chkmin(low[u], dfn[v]);
    }
}

bool chk1() {
    flag = 0;
    memset(dfn, dfs_clock = 0, sizeof dfn);

    for(int i = c+1; i <= HM.cnt; i++) if(!dfn[i]) {
        if(dfs(i, 0, 1)) return 1;
    }
    return 0;
}

#define x(i) HM.X[i]
#define y(i) HM.Y[i]

//char ch[1000][1000];
int X[maxn], Y[maxn], idx[maxn];
bool chk0() {
    memset(idx, 0, sizeof idx);
    memset(vis, 0, sizeof vis);

    for(int i = 1; i <= c; i++) HM.insert(X[i], Y[i]);
    for(int v = 1; v <= c; v++) if(!vis[v]) {
        vector<int> V;
        static int q[maxn];
        int head = 0, tail = 0;

        ecnt = 1;
        vis[q[tail++] = v] = 3;
        while(head < tail) {
            int h = q[head++];
            for(int i = 0; i < 8; i++) {
                int nx = x(h) + dx[i];
                int ny = y(h) + dy[i];

                if(nx >= 1 && nx <= n && ny >= 1 && ny <= m) {
                    int Nxt = HM.insert(nx, ny);

                    st[Nxt] = 0;
                    if(Nxt <= c) {
                        if(!vis[Nxt]) {
                            vis[Nxt] = 3;
                            q[tail++] = Nxt;
                        }
                    }else if(idx[Nxt] != v) {
                        idx[Nxt] = v, V.pb(Nxt);
                    }
                }
            }
        }
        for(int i = 0; i < int(V.size()); i++) {
            int u = V[i];
            for(int j = 0; j < 2; j++) {
                int nx = x(u) + dx[j];
                int ny = y(u) + dy[j];

                if(nx >= 1 && nx <= n && ny >= 1 && ny <= m) {
                    int Nxt = HM.find(nx, ny);
                    if(idx[Nxt] == v) addedge(u, Nxt);
                }
            }
        }

        if(V.size() && area_count(V[0]) != int(V.size()))
            return true;
    }
    return false;
}


void build() {
    memset(vis, 0, sizeof vis);

    int tail = 0;
    static int q[maxn];

    for(int i = 1; i <= c; i++) 
        HM.insert(X[i], Y[i]), vis[i] = 3;

    for(int i = 1; i <= c; i++)
        for(int j = 0; j < 8; j++) {
            int nx = X[i] + dx[j];
            int ny = Y[i] + dy[j];

            if(nx >= 1 && nx <= n && ny >= 1 && ny <= m) {
                int Nxt = HM.insert(nx, ny);

                if(!vis[Nxt]) {
                    vis[Nxt] = 1;
                    q[tail++] = Nxt;
                }
            }
        }

    int lim = tail;
    for(int i = 0; i < lim; i++) {
        for(int j = 0; j < 8; j++) {
            int nx = x(q[i]) + dx[j];
            int ny = y(q[i]) + dy[j];

            if(nx >= 1 && nx <= n && ny >= 1 && ny <= m) {
                int Nxt = HM.insert(nx, ny);
                if(!vis[Nxt]) {
                    vis[Nxt] = 2;
                    q[tail++] = Nxt;
                }
            }
        }
    }

    for(int i = 0; i < tail; i++) {
        for(int j = 0; j < 2; j++) {
            int nx = x(q[i]) + dx[j];
            int ny = y(q[i]) + dy[j];

            if(nx >= 1 && nx <= n && ny >= 1 && ny <= m) {
                int Nxt = HM.find(nx, ny);
                if(Nxt > c) addedge(q[i], Nxt);
            }
        }
    }
}

int spe() {

    int res = 0;
    if(n == 1) {
        Y[++c] = 0; Y[++c] = m+1; sort(Y+1, Y+c+1);
        for(int i = 2; i <= c; i++) if(Y[i] - Y[i-1] > 1) ++ res;
    }else {
        X[++c] = 0; X[++c] = n+1; sort(X+1, X+c+1);
        for(int i = 2; i <= c; i++) if(X[i] - X[i-1] > 1) ++ res;
    }
    return res >= 2 ? 0 : 1;
}
int main() {
#ifndef ONLINE_JUDGE
    freopen("data.txt", "r", stdin);
    freopen("ans.txt", "w", stdout);
#endif

    for(int T = read<int>(); T--; ) {

        init();

        n = read<int>();
        m = read<int>();
        c = read<int>();

        for(int i = 1; i <= c; i++) {
            X[i] = read<int>();
            Y[i] = read<int>();
        }

        if(chk0()) {
            puts("0");
            continue;
        }

        init();
        build();

        static int ans;
        if((ans = chk()) != -2) { }
        else if(min(n, m) == 1) { ans = spe(); }
        else {
            ans = chk1() ? 1 : 2;
            if(c == 0) ans = min(n, m) == 1 ? 1 : 2;
        }
        printf("%d\n", ans);
    }
    return 0;
}
```
