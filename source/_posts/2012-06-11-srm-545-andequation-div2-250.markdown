---
layout: post
title: "SRM 545 - ANDEquation (Div2 250)"
date: 2012-06-11 20:00
comments: true
categories: TopCoder SRM
---

## 概要
N+1個の整数が与えられる。これをN個のXiと1個のYに分けて、X0 & X1 & ... & X_N-1 = Y とできるのであればYを、できなければ-1を返せ。

## 制約
1 ≦ N ≦ 49

## 解法
Nが小さいので総当たり。1つYとして選んで、実際にその他全部のANDをとってYと比較する。O(N^^2 )

## コード
``` cpp
#include <vector>
using namespace std;

class ANDEquation {
public:
  int restoreY(vector <int> A) {
    for (int i = 0; i < A.size(); i++) {
      int xs = ~0;
      for (int j = 0; j < A.size(); j++) {
        if (j != i) xs &= A[j];
      }
      if (A[i] == xs) return A[i];
    }
    return -1;
  }
};
```
