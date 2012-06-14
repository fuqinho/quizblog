---
layout: post
title: "UVa 11990 - Dynamic Inversion"
date: 2012-06-14 01:12
comments: true
categories: [UVa, Bucket]
---

## 概要
1からNまでの数字を使った順列が与えられる。そこからQ個の数を順に削除する時、Q回のステップにおいて削除直前の転倒数を出力せよ。([原文](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3141))

## 制約
* N ≦ 20,000
* Q ≦ 10,000

## 解法
* xを追加/削除する時に変化する転倒数は、「xより前にあるxより大きい数」と「xより後にあるxより小さい数」の合計なので、これを効率良く数えたい。
* i番目の数A[i]を座標上の点(i, A[i])にプロットした場合を考えると、上記の数は「(i,A[i])より左下の領域にある点の数」と「(i,A[i])より右上の領域にある点の数」になる。
* 点の数はバケット法を使って数える。領域全体はN×Nのサイズだが、これをN個の√N×√N領域に分ける。
* (0,0)と(x,y)で作られる矩形領域は、完全に含まれているバケット内の数とはみ出し領域に分けられる。
    * バケットに含まれる方はprefix sumを使ってO(√N)。2次元BITを使うともっと早いかもしれない。
    * はみ出した方も、注目するxの幅又はyの幅は√NなのでO(√N)で数え上げられる
* (0,0)と(x,y)で作られる領域に含まれる点の数をS(x,y)とすると右上領域はS(N,y)-S(x,y),左下はS(x,N)-S(x,y)
* 結局、点の追加も削除も転倒数のカウントもO(√N)の定数倍になる。トータルでO((N+M)√N)。

## コード
``` cpp
// Run Time: 2.080
#include <cstdio>
#include <cstring>
using namespace std;

// constants
const int MAX_N = 200000;
const int MAX_M = 100000;
const int BLOCK_SIZE = 450;
const int MAX_BLOCK = (MAX_N-1)/BLOCK_SIZE + 1;

// input data
int N, M;
int A[MAX_N];
int Q[MAX_M];

// variables
int cnt[MAX_BLOCK][MAX_BLOCK];
int prefix_sum[MAX_BLOCK][MAX_BLOCK];
int xs[MAX_N];
int ys[MAX_N];

// by行bx列のブロック内の個数が更新された場合は、
// by行内bx列以降のprefix_sumを更新する
void update_prefix_sum(int bx, int by) {
  int sum = (bx > 0 ? prefix_sum[bx-1][by] : 0);
  for (int i = bx; i < MAX_BLOCK; i++) {
    sum += cnt[i][by];
    prefix_sum[i][by] = sum;
  }
}

// 座標(x, y)に点を加える
void add(int x, int y) {
  int bx = x / BLOCK_SIZE;
  int by = y / BLOCK_SIZE;

  ys[x] = y;
  xs[y] = x;
  cnt[bx][by]++;
  update_prefix_sum(bx, by);
}

// 座標(x, y)の点を削除する
void remove(int x, int y) {
  int bx = x / BLOCK_SIZE;
  int by = y / BLOCK_SIZE;
  
  ys[x] = -1;
  xs[y] = -1;
  cnt[bx][by]--;
  update_prefix_sum(bx, by);
}

// 座標(0,0)と(x,y)で作られる矩形領域内の点の数を合計する
int count_sum(int x, int y) {
  int block_w = (x + 1) / BLOCK_SIZE;
  int block_h = (y + 1) / BLOCK_SIZE;
  
  int res = 0;
  // 丸々含まれてるブロックの分
  for (int i = 0; i < block_h; i++) {
    if (block_w > 0) res += prefix_sum[block_w-1][i];
  }
  // はみ出し領域
  for (int i = block_w * BLOCK_SIZE; i <= x; i++) {
    if (ys[i] != -1 && ys[i] < block_h * BLOCK_SIZE) res++;
  }
  for (int i = block_h * BLOCK_SIZE; i <= y; i++) {
    if (xs[i] != -1 && xs[i] <= x) res++;
  }
  return res;
}

// (x,y)の右上の領域と左下の領域の合計(＝(x,y)に依存している転倒数)を返す
int count_inversion(int x, int y) {
  int res = 0;
  int intersection = count_sum(x, y);
  res += count_sum(x, N-1) - intersection;
  res += count_sum(N-1, y) - intersection;
  return res;
}

void solve() {
  memset(xs, -1, sizeof(xs));
  memset(ys, -1, sizeof(ys));
  memset(cnt, 0, sizeof(cnt));
  memset(prefix_sum, 0, sizeof(prefix_sum));

  long long inversion = 0;
  for (int i = 0; i < N; i++) {
    add(i, A[i]);
    inversion += count_inversion(i, A[i]);
  }
  for (int i = 0; i < M; i++) {
    printf("%lld\n", inversion);
    inversion -= count_inversion(xs[Q[i]], Q[i]);
    remove(xs[Q[i]], Q[i]);
  }
}

int main() {
  while (scanf("%d %d", &N, &M) == 2) {
    for (int i = 0; i < N; i++) {
      scanf("%d", &A[i]);
      A[i]--;
    }
    for (int i = 0; i < M; i++) {
      scanf("%d", &Q[i]);
      Q[i]--;
    }
    solve();
  }
}
```

