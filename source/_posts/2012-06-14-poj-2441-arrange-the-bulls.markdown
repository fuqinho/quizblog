---
layout: post
title: "POJ 2441 - Arrange the Bulls"
date: 2012-06-14 04:30
comments: true
categories: POJ BitDP
---
## 概要
N頭の牛とM個の家畜小屋があり、それぞれの牛には1つ以上の好きな家畜小屋がある。家畜小屋には1頭しか入れないとすると、全ての牛を好きな小屋のどれかに入れる方法は何通りあるか。([原文](http://poj.org/problem?id=2441))

## 制約
* N ≦ 20
* M ≦ 20

## 解法
* dp(i, S)を、「1番目からi番目の牛が集合Sで示される小屋を使っている場合のパターン数」としてビットDPする。
* 集合Sは要素数の最大が20なのでintのビットで表現
* フルにメモリ確保すると足りないので、iが偶数の時と奇数の時の２つ用意して使いまわす。

## コード
``` cpp
// Limit: 65536K  4000ms
// Used : 12996K  1813ms
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
using namespace std;

int solve(int N, int M, vector<vector<int> >& fav) {
  // dp[i][S]: the number of patterns that cow 0-i use S(set) barns
  vector<vector<int> > dp(2, vector<int>(1 << M));
  dp[0][0] = 1;

  for (int i = 0; i < N; i++) {
    for (int j = 0; j < fav[i].size(); j++) {
      int barn_bit = (1 << fav[i][j]);
      for (int s = 0; s < (1 << M); s++) {
        if (!(s & barn_bit)) {
          dp[(i+1)&1][s|barn_bit] += dp[i&1][s];
        }
      }
    }
    fill(dp[i&1].begin(), dp[i&1].end(), 0);
  }
  return accumulate(dp[N&1].begin(), dp[N&1].end(), 0);
}

int main() {
  cin.tie(0);
  ios::sync_with_stdio(false);

  // read input data
  int N, M; cin >> N >> M;
  vector<vector<int> > fav(N);
  for (int i = 0; i < N; i++) {
    int c; cin >> c;
    for (int j = 0; j < c; j++) {
      int b; cin >> b;
      fav[i].push_back(b-1); // 0-base index
    }
  }
  // solve it and print answer
  int ans = solve(N, M, fav);
  cout << ans << endl;
}
```

