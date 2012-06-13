---
layout: post
title: "SRM 545 - StrIIRec (Div1 275/Div2 550)"
date: 2012-06-11 20:30
comments: true
categories: [TopCoder, SRM, Greedy]
---

## 概要
n文字の文字列"abcd..."の各文字を並べ替えた文字列(例:"adbc")の転倒数について考える。ここで転倒数とは、はじめの文字列とは順番が入れ替わっている文字の組み合わせの数([Wikipedia](http://ja.wikipedia.org/wiki/%E8%BB%A2%E5%80%92%E6%95%B0))。

n文字の順列全てのうち、転倒数がminInv以上になっていて、かつ辞書順がminVal以降になっている文字列の中で最も辞書順が若いものを求めよ。なければ空文字列を返せ。

## 制約
n ≦ 20

## 解法
* 総当たりは20!なので無理
* 転倒数は、「i+1番目以降でi番目の文字より小さい文字の数」を全てのiについて足せば求まる。
* minValの時点で確定している転倒数があるので、もしそれが既にminInv以上になっていれば、未確定の文字は辞書順最小になるよう小さい順にならべて返せば良い。
* minValに満たなければ、文字列の末尾から順に、minValになるまで大きい文字を使うことで転倒数を増やす。

## コード
``` cpp
#include <vector>
#include <string>
#include <numeric>
using namespace std;

class StrIIRec {
public:
  int num_smaller(char c, vector<bool>& used) {
    int count = 0;
    for (int i = 0; i < c - 'a'; i++) {
      if (!used[i]) count++;
    }
    return count;
  }

  char find_kth_char(int k, vector<bool>& used) {
    int count = 0;
    for (int i = 0; i < used.size(); i++) {
      if (!used[i]) {
        if (count == k) return 'a' + i;
        else count++;
      }
    }
    return ' ';
  }

  string recover(vector<int>& inv_count) {
    string res = "";
    vector<bool> used(inv_count.size());
    for (int i = 0; i < inv_count.size(); i++) {
      char c = find_kth_char(inv_count[i], used);
      res += c;
      used[c - 'a'] = true;
    }
    return res;
  }

  string recovstr(int n, int minInv, string minStr) {
    // inv_count[i]: the number of chars smaller than i-th char after i-th
    // inv_couht = {2,0,1,0} <=> "cadb"
    vector<int> inv_count(n, 0);

    // calc assured inversions by minStr
    vector<bool> used(n, false);
    for (int i = 0; i < minStr.size(); i++) {
      char c = minStr[i];
      used[c-'a'] = true;
      inv_count[i] = num_smaller(c, used);
    }
    int inversion = accumulate(inv_count.begin(), inv_count.end(), 0);

    // determine inversion count for each characters from the tail
    for (int i = n-1; i >= 0; i--) {
      if (inversion < minInv) {
        int can_add = n-1 - i - inv_count[i];
        if (minInv - inversion >= can_add) {
          inv_count[i] += can_add;
          inversion += can_add;
        } else {
          inv_count[i] += minInv - inversion;
          inversion = minInv;
        }
      }
    }
    
    if (inversion >= minInv) {
      return recover(inv_count);
    } else {
      return "";
    }
  }
};
```

