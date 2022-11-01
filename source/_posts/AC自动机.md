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
2. fail指针
   定义:表示从根节点到该节点所组成字符序列的所有后缀和整个模式字符串集合即整个Trie树中所有前缀两者中的最长公共部分。fail指针的优化作用体现于可以减少不必要的重复匹配，类似于将KMP的border放在树上跳
## 经典问题
>给定 n 个模式串和一个文本串t，求有多少个不同的模式串在文本串里出现过。两个模式串不同当且仅当他们编号不同。
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
>有 N 个由小写字母组成的模式串以及一个文本串 T。每个模式串可能会在文本串中出现多次。你需要找出哪些模式串在文本串 T 中出现的次数最多。
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
> 一个文本串 S 和 n 个模式串 T ，请你分别求出每个模式串 T 在 S 中出现的次数。
```cpp
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