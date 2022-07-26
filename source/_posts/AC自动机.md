---
title: AC自动机
date: 2022-10-31 20:51:27
tags: 字符串
---
something about AC自动机
<!--more-->
# AC自动机
## 理解
1. 对于AC自动机的理解:
   + 粗浅的理解:trie树+KMP
   + AC自动机上的每个点是一个状态，每一条边则是一种状态向另一种状态的转移，所以AC自动机可以和DP优雅地结合
   + AC自动机是一张图，所以许多图上算法在AC自动机上任然使用，所以经常构建完以后就变成了图论题
   + 是一种离线数据结构，不能做到边插入边查询
2. fail指针
   + 定义:表示从根节点到该节点所组成字符序列的所有后缀和整个模式字符串集合即整个Trie树中所有前缀两者中的最长公共部分。
   + fail指针的优化作用体现于可以减少不必要的重复匹配，类似于将KMP的border放在树上跳。
   + 将所有的 fail 指针连成边，构成了一棵树
## 经典问题
+ 给定 n 个模式串和一个文本串t，求有多少个不同的模式串在文本串里出现过。两个模式串不同当且仅当他们编号不同。
```cpp
struct AC {
  int tr[MAXN][30], cnt[MAXN], fail[MAXN], tot = 0;
  queue<int> q;
  void insert(string s) { // Trie树的插入
    int u = 0, len = s.length();
    for (int v, i = 0; i < len; i++) {
      v = s[i] - 'a';
      if (!tr[u][v])
        tr[u][v] = ++tot;
      u = tr[u][v];
    }
    ++cnt[u];
  }
  inline void build() { // BFS构建fail指针———灵魂
    while (q.size())
      q.pop();
    for (int i = 0; i < 26; i++) //第一层的fail指向根节点
      if (tr[0][i])
        fail[tr[0][i]] = 0, q.push(tr[0][i]);
    while (!q.empty()) {
      int u = q.front();
      q.pop();
      for (int i = 0; i < 26; i++)
        if (tr[u][i])
          fail[tr[u][i]] = tr[fail[u]][i],
          q.push(tr[u][i]); // 儿子的fail是fail的儿子
        //(因为当前节点fail指向的节点是与其后缀有最长公共前缀的节点
        //显然如果当前模式串还没有匹配上的话，可以去匹配和他只有最后一个字符不同的模式串
        //这样就能省去再次重新匹配前缀的时间)
        else //如果当前节点不存在字符为c的儿子，那就考虑去匹配和当前字符只有最后一位不同的fail[u]是否有c这个儿子
          tr[u][i] = tr[fail[u]][i];
    }
  }
  inline int query(string s) { //查询有多少个不同的模式串在文本串里出现过
    int u = 0, ans = 0, len = s.length();
    for (int i = 0; i < len; i++) { // Trie树查询+跳fail
      u = tr[u][s[i] - 'a'];
      for (int pos = u; pos && ~cnt[pos]; pos = fail[pos])
        //如果cnt已经为-1了，那么就说明之前的再往上跳fail都已经被匹配过了。
        ans += cnt[pos],
            cnt[pos] = -1; //如果纳入统计了就标记为-1，这样可以防止反复统计
    }
    return ans;
  }
} AK;
int n;
string s;
inline void work(signed CASE = 1, bool FINAL_CASE = false) {
  cin >> n;
  for (int i = 1; i <= n; i++)
    cin >> s, AK.insert(s); //注意插入的是模式串
  cin >> s;
  AK.build(); //一定要构建
  cout << AK.query(s) << endl;
}
```
+ 有 N 个由小写字母组成的模式串以及一个文本串 T。每个模式串可能会在文本串中出现多次。你需要找出哪些模式串在文本串 T 中出现的次数最多。
```cpp
string T[MAXN], S;
struct AC {
  int tr[MAXN][30], fail[MAXN], tot = 0, end[MAXN];
  int ans[MAXN];
  queue<int> q;
  void clear() { mmst0(tr), mmst0(fail), mmst0(end), tot = 0; } //清空AC自动机
  void insert(string s, int idx) {
    int u = 0, len = s.length();
    for (int v, i = 0; i < len; i++) {
      v = s[i] - 'a';
      if (!tr[u][v])
        tr[u][v] = ++tot;
      u = tr[u][v];
    }
    end[u] = idx;
  }
  inline void build() {
    while (q.size())
      q.pop();
    for (int i = 0; i < 26; i++)
      if (tr[0][i])
        fail[tr[0][i]] = 0, q.push(tr[0][i]);
    while (!q.empty()) {
      int u = q.front();
      q.pop();
      for (int i = 0; i < 26; i++)
        if (tr[u][i])
          fail[tr[u][i]] = tr[fail[u]][i], q.push(tr[u][i]);
        else
          tr[u][i] = tr[fail[u]][i];
    }
  }
  inline void query(string s) {//简单改换一下查询方式即可
    mmst0(ans); //清空ans
    int u = 0, len = s.length();
    for (int i = 0; i < len; i++) {
      u = tr[u][s[i] - 'a'];
      for (int pos = u; pos; pos = fail[pos]) //查询出现次数
        ans[end[pos]]++;
    }
  }
} AK;
int n, p[MAXN];
inline void work(signed CASE = 1, bool FINAL_CASE = false) {
  while (cin >> n) {
    AK.clear();
    if (n == 0)
      break;
    for (int i = 1; i <= n; i++)
      cin >> T[i], AK.insert(T[i], i);
    cin >> S;
    AK.build();
    AK.query(S);
    for (int i = 1; i <= n; i++)
      p[i] = i;
    sort(p + 1, p + 1 + n, [](auto a, auto b) {
      return AK.ans[a] == AK.ans[b] ? a < b : AK.ans[a] > AK.ans[b];
    });
    cout << AK.ans[p[1]] << "\n" << T[p[1]] << "\n";
    for (int i = 2; i <= n; i++) {
      if (AK.ans[p[i]] == AK.ans[p[i - 1]])
        cout << T[p[i]] << "\n";
      else
        break;
    }
  }
}
```
+ 一个文本串 S 和 n 个模式串 T ，请你分别求出每个模式串 T 在 S 中出现的次数。
```cpp
//Sol1:暴力统计 O(nm)
string T[MAXN], S;
struct AC {
  int tr[MAXN][30], fail[MAXN], tot = 0;
  vector<int> end[MAXN];
  int ans[MAXN];
  queue<int> q;
  void clear() { mmst0(tr), mmst0(fail), mmst0(end), tot = 0; } //清空AC自动机
  void insert(string s, int idx) {
    int u = 0, len = s.length();
    for (int v, i = 0; i < len; i++) {
      v = s[i] - 'a';
      if (!tr[u][v])
        tr[u][v] = ++tot;
      u = tr[u][v];
    }
    end[u].push_back(idx); //因为可能模式串相同，所以把询问挂在节点上
  }
  inline void build() {
    while (q.size())
      q.pop();
    for (int i = 0; i < 26; i++)
      if (tr[0][i])
        fail[tr[0][i]] = 0, q.push(tr[0][i]);
    while (!q.empty()) {
      int u = q.front();
      q.pop();
      for (int i = 0; i < 26; i++)
        if (tr[u][i])
          fail[tr[u][i]] = tr[fail[u]][i], q.push(tr[u][i]);
        else
          tr[u][i] = tr[fail[u]][i];
    }
  }
  inline void query(string s) {
    mmst0(ans); //清空ans
    int u = 0, len = s.length();
    for (int i = 0; i < len; i++) {
      u = tr[u][s[i] - 'a'];
      for (int pos = u; pos; pos = fail[pos]) //查询出现次数
        for (auto v : end[pos])               //遍历节点上的询问
          ans[v]++;
    }
  }
} AK;
int n, p[MAXN];
inline void work(signed CASE = 1, bool FINAL_CASE = false) {
  cin >> n;
  for (int i = 1; i <= n; i++)
    cin >> T[i], AK.insert(T[i], i);
  cin >> S;
  AK.build();
  AK.query(S);
  for (int i = 1; i <= n; i++) {
    cout << AK.ans[i] << "\n";
  }
}
```
```cpp
//Sol2:构建fail树,在树上差分解决问题O(n+m)
//字符串问题->图上问题
string T[MAXN], S;
struct AC {
  int tr[MAXN][30], fail[MAXN], tot = 0, end[MAXN];
  int ans[MAXN];
  vector<int> F[MAXN]; // fail树
  queue<int> q;
  void clear() { mmst0(tr), mmst0(fail), mmst0(end), tot = 0; }
  void insert(string s, int idx) {
    int u = 0, len = s.length();
    for (int v, i = 0; i < len; i++) {
      v = s[i] - 'a';
      if (!tr[u][v])
        tr[u][v] = ++tot;
      u = tr[u][v];
    }
    end[idx] = u; //记录当前字符串结束节点
  }
  inline void build() {
    while (q.size())
      q.pop();
    for (int i = 0; i < 26; i++)
      if (tr[0][i])
        fail[tr[0][i]] = 0, q.push(tr[0][i]), F[0].push_back(tr[0][i]);//勿忘第一层Fail树指向根节点
    while (!q.empty()) {
      int u = q.front();
      q.pop();
      for (int i = 0; i < 26; i++)
        if (tr[u][i])
          fail[tr[u][i]] = tr[fail[u]][i], q.push(tr[u][i]),
          F[tr[fail[u]][i]].push_back(tr[u][i]); //构建Fail树
        else
          tr[u][i] = tr[fail[u]][i];
    }
  }
  inline void query(string s) {
    mmst0(ans);
    int u = 0, len = s.length();
    for (int i = 0; i < len; i++) {
      u = tr[u][s[i] - 'a'];
      ans[u]++;
    }
  }
  //树上差分计算答案
  inline void dfs(int u) {
    for (auto v : F[u]) {
      dfs(v);
      ans[u] += ans[v];
    }
  }
  inline void get() {
    for (int i = 0; i < 26; i++)
      if (tr[0][i])
        dfs(tr[0][i]);
  }
} AK;
int n, p[MAXN];
inline void work(signed CASE = 1, bool FINAL_CASE = false) {
  cin >> n;
  for (int i = 1; i <= n; i++)
    cin >> T[i], AK.insert(T[i], i);
  cin >> S;
  AK.build();
  AK.query(S);
  AK.get();
  for (int i = 1; i <= n; i++) {
    cout << AK.ans[AK.end[i]] << "\n";
  }
}
```
+ 初始给定一堆字符串组成的字典，然后需要多次执行以下操作
  1.添加一个字符串到字典中
  2.给定一个字符串s，查询字典中字符串，统共在s里面出现了多少次
例如字典{ab,xyz} s = abxyzab 统共出现了3次 （可以计算重复的）
```cpp
//Sol:将操作离线（AC自动机是离线数据结构），然后转化为子树和问题，在dfn序使用数据结构（树状数组）维护答案
string  S;
struct BIT {//树状数组
  int tr[MAXN];
  inline void add(int pos, int val) {
    for (; pos < MAXN; pos += pos & -pos)
      tr[pos] += val;
  }
  inline void add(int l, int r, int val) { add(l, val), add(r + 1, -val); }
  inline int query(int pos) {
    int ans = 0;
    for (; pos; pos -= pos & -pos)
      ans += tr[pos];
    return ans;
  }
  inline void init() { mmst0(tr); }
} bit;
struct AC {
  int tr[MAXN][30], fail[MAXN], tot = 0, end[MAXN], tim = 0, dfn[MAXN],
                                in[MAXN], out[MAXN];
  vector<int> F[MAXN];
  queue<int> q;
  void clear() {
    mmst0(tr), mmst0(fail), mmst0(end), tot = 0, bit.init();
    for (int i = 0; i < MAXN; i++)
      F[i].clear();
    tim = 0;
  }
  int insert(string s) {
    int u = 0, len = s.length();
    for (int v, i = 0; i < len; i++) {
      v = s[i] - 'a';
      if (!tr[u][v])
        tr[u][v] = ++tot;
      u = tr[u][v];
    }
    end[u] = 1;
    return u;
  }
  inline void build() {
    while (q.size())
      q.pop();
    for (int i = 0; i < 26; i++)
      if (tr[0][i])
        fail[tr[0][i]] = 0, q.push(tr[0][i]), F[0].push_back(tr[0][i]);
    while (!q.empty()) {
      int u = q.front();
      q.pop();
      for (int i = 0; i < 26; i++)
        if (tr[u][i])
          fail[tr[u][i]] = tr[fail[u]][i], q.push(tr[u][i]),
          F[tr[fail[u]][i]].push_back(tr[u][i]);
        else
          tr[u][i] = tr[fail[u]][i];
    }
  }
  inline int query(string s) {
    int u = 0, len = s.length(), res = 0;
    for (int i = 0; i < len; i++) {
      u = tr[u][s[i] - 'a'];
      res += bit.query(dfn[u]);
    }
    return res;
  }
  inline void dfs(int u) {//得到dfn序
    in[u] = dfn[u] = ++tim;
    for (auto v : F[u])
      dfs(v);
    out[u] = tim;
  }
  inline void add(int u) { bit.add(in[u], out[u], 1); }
} AK;
int n, m, cnt, id[MAXN];
struct Q {
  int opt;
  string s;
} q[MAXN];
inline void work(signed CASE = 1, bool FINAL_CASE = false) {
  AK.clear();
  cin >> n >> m;
  for (int i = 1; i <= n; i++)
    cin >> S, AK.insert(S);
  cnt = 0;
  for (int opt, i = 1; i <= m; i++) {
    cin >> opt >> S;
    if (opt == 1) {
      id[++cnt] = AK.insert(S);
      q[i] = {1, ""};
      AK.end[id[cnt]] = 0;
    } else {
      q[i] = {2, S};
    }
  }
  AK.build();
  AK.dfs(0);
  for (int i = 0; i < MAXN; i++) {
    if (AK.end[i])
      AK.add(i);
  }
  cnt = 0;
  for (int i = 1; i <= m; i++) {
    if (q[i].opt == 1)
      AK.add(id[++cnt]);
    else
      cout << AK.query(q[i].s) << "\n";
  }
}
```
+ 给出一个字典，至多包含60 个字典串，且字典串总长度不超过100。求所有长度为M($\le$ 10000) 的串中（共$26^M$ 个），有多少个串至少包含一个字典串。
```cpp
//直接统计不怎么好搞，于是考虑容斥，ans = 26^M - 一个字典串都不包含的串（记为P串）的个数
//定义状态dp[i][u]表示匹配前长度为i的串，在节点匹配到u时候的P串个数
//转移么，通常是通过枚举边转移的
/*
for (int i = 1; i <= m; i++)//枚举长度
  for (int j = 0; j <= tot; j++)//枚举当前节点
    for (int k = 0; k < 26; k++)//枚举下一个节点是哪个字符
    //通过边(j,tr[j][k])构成转移
*/
//然后我们考虑怎样的dp转移是合法的，也就是什么样的串才是需要计数的P串
//有一个BF想法:既然是要没有字典串被包含，那么就是在匹配的过程中跳到的节点不能是一个串的结尾
//那么不妨在作为每一个字典串结尾的点上打上标记，dp转移前先暴力地跳fail，检验一下过程中每一个节点是否有标记
//如果有标记则说明当前以节点结尾的那个字典串匹配上了当前枚举的串的前缀，所以是不合法的
//但是在该数据范围下BF肯定是会TLE
//于是考虑有优化:
//优化1:考虑优化跳fail的过程,不妨建出fail树使用倍增优化跳fail查找是否有标记的过程
//这样这一过程就有O(depth)->O(log depth)
//优化2:观察不合法串的性质，发现串的合法性也有和fail指针类似的性质
//儿子的合法性与fail指向的那个节点的儿子的合法性是相同
//感性理解：如果当前节点的儿子到根的代表串是N，fail指向的那个节点的儿子到根代表的串是S
//根据定义S肯定是N的子串，如果S不合法，那么就说明S会匹配上一个串，那么作为包含S的N也一定会匹配上一个串。
//于是可以仿照bfs构建fail的过程来构建记录合法性的数组
int n, m;
const int mod = 1e4 + 7;
string s;
int dp[MAXN][MAXN];
inline int qpow(int a, int b, int mod) {
  int res = 1ll;
  for (; b; b >>= 1, a = 1ll * a * a % mod)
    if (b & 1)
      res = 1ll * res * a % mod;
  return res % mod;
}
struct AC {
  int tr[MAXN][26], cnt[MAXN], fail[MAXN], tot;
  bool ok[MAXN];
  queue<int> q;
  inline void insert(string s) {
    int u = 0;
    for (auto c : s) {
      int v = c - 'A';
      if (!tr[u][v])
        tr[u][v] = ++tot;
      u = tr[u][v];
    }
    ok[u] = true;
  }
  inline void build() {
    while (q.size())
      q.pop();
    for (int i = 0; i < 26; i++)
      if (tr[0][i])
        fail[tr[0][i]] = 0, q.push(tr[0][i]);
    while (q.size()) {
      int u = q.front();
      q.pop();
      for (int i = 0; i < 26; i++)
        if (tr[u][i])
          fail[tr[u][i]] = tr[fail[u]][i], ok[tr[u][i]] |= ok[tr[fail[u]][i]],//here 在构建合法性数组ok
          q.push(tr[u][i]);
        else
          tr[u][i] = tr[fail[u]][i];
    }
  }
  inline int query() {
    dp[0][0] = 1;
    for (int i = 1; i <= m; i++)
      for (int j = 0; j <= tot; j++)
        for (int k = 0; k < 26; k++)
          if (!ok[tr[j][k]])
            dp[i][tr[j][k]] = (dp[i][tr[j][k]] + dp[i - 1][j]) % mod;
            //转移  代表的是j通过(j,tr[j][k])(对应事件:匹配了一位为k的字符)到达tr[j][k]时所带来的方案数
            //匹配长度为i时当前节点(tr[j][k])到达的合法方案数 sum(能转移到当前节点的节点合法方案数(dp[i-1][j])) 
    int ans = qpow(26, m, mod);
    for (int i = 0; i <= tot; i++)
      ans = (ans - dp[m][i] + mod) % mod;
    return ans;
  }
} AK;
```