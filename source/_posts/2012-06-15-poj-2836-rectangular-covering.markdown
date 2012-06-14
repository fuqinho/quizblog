---
layout: post
title: "POJ 2836 - Rectangular Covering"
date: 2012-06-15 07:46
comments: true
categories: [POJ, BitDP]
---
## 概要
平面座標上の点がn個与えられる。いくつかの長方形でこれらの点を全て覆う時、長方形の面積の合計は最小でいくつか。長方形の各辺は座標軸と並行でなければならず、また1辺は1以上の整数でなければならない。([原文](http://poj.org/problem?id=2836))

## 制約
* n ≦ 15

## 解法
* まず、候補になる長方形をリストアップする。n個の点から2点選んで、その2点で作られる長方形にどの点がカバーされているかをチェックする。
* リストアップした長方形を組み合わせて全点をカバーする組み合わせを作る。
* dp[S]を、集合Sに含まれる点がカバーされている場合の最小の合計面積として、ビットDPする。

## コード
``` cpp
// Limit: 65536K  1000ms
// Used :   816K   157ms
#include <iostream>
#include <vector>
#include <cstdlib>
using namespace std;

const int INF = 1000000000;

struct Rect {
  int covering, area;
  Rect(int c, int a): covering(c), area(a) {}
};

bool isCovered(int p, int p1, int p2, vector<int>& x, vector<int>& y) {
  return (x[p]-x[p1])*(x[p]-x[p2]) <= 0 &&
         (y[p]-y[p1])*(y[p]-y[p2]) <= 0;
}

int solve(int n, vector<int>& x, vector<int>& y) {
  // list up rectangles
  vector<Rect> rectangles;
  for (int i = 0; i < n; i++) {
    for (int j = i+1; j < n; j++) {

      // make rectangle by i-th point and j-th point,
      // and check covering points and its area (width and height must not be 0)
      int area = max(1, abs(x[i]-x[j])) * max(1, abs(y[i]-y[j]));
      Rect rect = Rect((1 << i)|(1 << j), area);
      for (int k = 0; k < n; k++) {
        if (isCovered(k, i, j, x, y)) rect.covering |= (1 << k);
      }
      rectangles.push_back(rect);
    }
  }

  // DP for state S: which point are covered
  // dp[S]: minimum area to cover points in S
  vector<int> dp(1 << n, INF);
  dp[0] = 0;
  for (int i = 0; i < rectangles.size(); i++) {
    int covering = rectangles[i].covering;
    for (int s = 0; s < (1 << n); s++) {
      if (dp[s] != INF && (s|covering) != s) {
        dp[s|covering] = min(dp[s|covering], dp[s] + rectangles[i].area);
      }
    }
  }
  return dp[(1 << n) - 1];
}

int main() {
  cin.tie(0);
  ios::sync_with_stdio(false);

  while (true) {
    int n; cin >> n;
    if (n == 0) break;

    vector<int> x(n), y(n);
    for (int i = 0; i < n; i++) cin >> x[i] >> y[i];

    int ans = solve(n, x, y);
    cout << ans << endl;
  }
}
```
