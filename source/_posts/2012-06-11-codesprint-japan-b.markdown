---
layout: post
title: "CodeSprint Japan - マトリックス"
date: 2012-06-11 21:03
comments: true
categories: CodeSprint Graph Kruskal
---

## 概要
N個の都市がN-1本の道路によって全域木の形で結ばれている。与えられたK個の都市が互いに切断されている状態になるように、道路を壊したい。その最小コストを求めよ。i番目の道路を壊すコストはTiで与えられる。([原文](https://csjapan.interviewstreet.com/challenges/dashboard/#problem/4f904e704e404))

## 制約
* 2 ≦ N ≦ 100,000
* 2 ≦ K ≦ N
* 1 ≦ Tn ≦ 1,000,000

## 解法
* 「壊す道路のコスト最小」⇔「繋がってる道路のコスト最大」なので、クラスカル法を逆向きに用いて道路を繋げて全域木を作る。
* ただし、K個の都市は互いに繋げてはいけないので、最初から繋がっていたことにしておく。
* 道全体のコストから繋げた道路のコストを引いたものが求める答え

## コード
``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// Structure for Union-Find
class DisjointSet {
public:
  DisjointSet(int n) {
    rank = vector<int>(n, 0);
    par = vector<int>(n);
    for (int i = 0; i < n; i++) par[i] = i;
  }
  int find(int x) {
    if (x == par[x]) return x;
    else return par[x] = find(par[x]);
  }
  void unite(int x, int y) {
    int rx = find(x), ry = find(y);
    if (rank[rx] < rank[ry]) par[rx] = ry;
    else {
      par[ry] = rx;
      if (rank[rx] == rank[ry]) rank[rx]++;
    }
  }
  bool same(int x, int y) {
    return find(x) == find(y);
  }
private:
  vector<int> par;
  vector<int> rank;
};

long long solve(int N, int K, vector<pair<int, pair<int, int> > >& edges,
                vector<int>& machine) {
  DisjointSet ds(N);

  // pre-connect cities with machines to avoid connection at following operation
  for (int i = 1; i < K; i++) {
    ds.unite(machine[0], machine[i]);
  }

  // find max sum of costs for connectable roads by inverted Kruscal's algorithm
  sort(edges.rbegin(), edges.rend());
  long long connected_cost = 0;
  long long total_cost = 0;
  for (int i = 0; i < edges.size(); i++) {
    int cost = edges[i].first;
    int city1 = edges[i].second.first;
    int city2 = edges[i].second.second;
    if (!ds.same(city1, city2)) {
      connected_cost += cost;
      ds.unite(city1, city2);
    }
    total_cost += cost;
  }

  return total_cost - connected_cost;
}

int main() {
  cin.tie(0);
  ios::sync_with_stdio(false);

  // read input data
  int N, K;
  cin >> N >> K;
  vector<pair<int, pair<int, int> > > edges(N-1);
  for (int i = 0; i < N-1; i++) {
    cin >> edges[i].second.first >> edges[i].second.second >> edges[i].first;
  }
  vector<int> machine(K);
  for (int i = 0; i < K; i++) cin >> machine[i];

  // solve it and print answer
  long long ans = solve(N, K, edges, machine);
  cout << ans << endl;
}
```
