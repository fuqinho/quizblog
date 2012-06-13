---
layout: post
title: "CodeSprint Japan - 嫌いな数値"
date: 2012-06-11 21:38
comments: true
categories: [CodeSprint, NumberTheory, Euclidean]
---

## 概要
整数Kの約数のうち、N個の整数A1,A2...のいずれの約数でも無いものの個数を求めよ ([原文](https://csjapan.interviewstreet.com/challenges/dashboard/#problem/4f7272a8b9d15))

## 制約
* 1 ≦ N ≦ 1,000,000
* 1 ≦ K ≦ 10,000,000,000,000
* 1 ≦ An ≦ 1,000,000,000,000,000,000

## 解法
1. a1,a2...それぞれについて、Kとの最大公約数をユークリッドの互除法で求めて、setに突っ込んでいく。O(N*logK)
2. Kの約数をリストアップする。試し割り＋再帰で実装した。
3. setに詰まってる数(個数は大した数にならない)の約数をリストアップする
4. 上記(2)の個数から(3)の個数を引く

## コード
``` cpp
#include <iostream>
#include <vector>
#include <set>
#include <algorithm>
using namespace std;

long long gcd(long long a, long long b) {
  if (a < b) swap(a, b);
  while (b) {
    swap(a, b);
    b %= a;
  }
  return a;
}

void expand_factors(long long n, set<long long>& out) {
  if (out.find(n) == out.end()) {
    out.insert(n);
    for (long long i = 2; i * i <= n; i++) {
      if (n % i == 0) {
        expand_factors(i, out);
        expand_factors(n / i, out);
      }
    }
  }
}

int solve(long long K, vector<long long>& A) {
  // A_divs: set of gcd(An, K)
  set<long long> A_divs;
  for (int i = 0; i < A.size(); i++) {
    A_divs.insert(gcd(A[i], K));
  }

  // factors: all factors of K
  set<long long> factors;
  factors.insert(1);
  expand_factors(K, factors);

  // toremove: all factors of A-divs
  set<long long> toremve;
  toremve.insert(1);
  for (set<long long>::iterator it = A_divs.begin(); it != A_divs.end(); it++) {
    expand_factors(*it, toremve);
  }
    
  return factors.size() - toremve.size();
}

int main() {
  cin.tie(0);
  ios::sync_with_stdio(false);

  // read input data
  int N;
  long long K;
  cin >> N >> K;
  vector<long long> A(N);
  for (int i = 0; i < N; i++) {
    cin >> A[i];
  }

  // solve and print answer
  int ans = solve(K, A);
  cout << ans << endl;
}
```



