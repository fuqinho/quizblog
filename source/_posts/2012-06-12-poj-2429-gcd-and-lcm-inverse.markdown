---
layout: post
title: "POJ 2429 - GCD &amp; LCM Inverse"
date: 2012-06-12 13:23
comments: true
categories: [POJ, NumberTheory, Euclidean, Eratosthenes, MillerRabin, PollardsRho]
---
## 概要
最小公倍数(LCM)と最大公約数(GCD)が与えられた時、それを満たす整数のペア(a,b)を求めよ。複数あり得る場合は、a+bが最も小さくなるペアを答えよ。([原文](http://poj.org/problem?id=2429))

## 制約
* a, b < 2^^63

## 解法
* LCM/GCDを素因数分解して、互いに素になるように２つに分けた後でGCDを掛ければ答えの候補になる。
* が、整数の範囲がめちゃくちゃ大きいので、素因数分解できない。
* 乱択アルゴリズムを使って確率的に求める。
* 素数判定は[ミラー-ラビン素数判定法](http://ja.wikipedia.org/wiki/%E3%83%9F%E3%83%A9%E3%83%BC-%E3%83%A9%E3%83%93%E3%83%B3%E7%B4%A0%E6%95%B0%E5%88%A4%E5%AE%9A%E6%B3%95)、合成数であることが分かれば[ポラード・ロー素因数分解法](http://ja.wikipedia.org/wiki/%E3%83%9D%E3%83%A9%E3%83%BC%E3%83%89%E3%83%BB%E3%83%AD%E3%83%BC%E7%B4%A0%E5%9B%A0%E6%95%B0%E5%88%86%E8%A7%A3%E6%B3%95)を使って約数を取り出して分解する。

## コード
``` cpp
// Limit: 65536K  2000ms
// Used :   908K   110ms	
#include <iostream>
#include <vector>
#include <map>
#include <cstdlib>
using namespace std;

typedef long long LL;

// return (a * b) % m
LL mod_mult(LL a, LL b, LL m) {
  LL res = 0;
  LL exp = a % m;
  while (b) {
    if (b & 1) {
      res += exp;
      if (res > m) res -= m;
    }
    exp <<= 1;
    if (exp > m) exp -= m;
    b >>= 1;
  }
  return res;
}

// return (a ^ b) % m
LL mod_exp(LL a, LL b, LL m) {
  LL res = 1;
  LL exp = a % m;
  while (b) {
    if (b & 1) res = mod_mult(res, exp, m);
    exp = mod_mult(exp, exp, m);
    b >>= 1;
  }
  return res;
}

// ミラー-ラビン素数判定法
bool miller_rabin(LL n, LL times) {
  if (n < 2) return false;
  if (n == 2) return true;
  if (!(n & 1)) return false;

  LL q = n-1;
  int k = 0;
  while (q % 2 == 0) {
    k++;
    q >>= 1;
  }
  // n - 1 = 2^k * q (qは奇素数)
  // nが素数であれば、下記のいずれかを満たす
  // (i) a^q ≡ 1 (mod n)
  // (ii) a^q, a^2q,..., a^(k-1)q のどれかがnを法として-1
  //
  // なので、逆に(i)(ii)いずれも満たしていない時は合成数と判定できる
  //
  for (int i = 0; i < times; i++) {
    LL a = rand() % (n-1) + 1; // 1,..,n-1からランダムに値を選ぶ
    LL x = mod_exp(a, q, n);
    // (i)をチェック
    if (x == 1) continue;
    // (ii)をチェック
    bool found = false;
    for (int j = 0; j < k; j++) {
      if (x == n-1) {
        found = true;
        break;
      }
      x = mod_mult(x, x, n);
    }
    if (found) continue;

    return false;
  }
  return true;
}

LL get_gcd(LL n, LL m) {
  if (n < m) swap(n, m);
  while (m) {
    swap(n, m);
    m %= n;
  }
  return n;
}

// ポラード・ロー素因数分解法
LL pollard_rho(LL n, int c) {
  LL x = 2;
  LL y = 2;
  LL d = 1;
  while (d == 1) {
    x = mod_mult(x, x, n) + c;
    y = mod_mult(y, y, n) + c;
    y = mod_mult(y, y, n) + c;
    d = get_gcd((x-y >= 0 ? x-y : y-x), n);
  }
  if (d == n) return pollard_rho(n, c+1);
  return d;
}

const int MAX_PRIME = 200000;
vector<int> primes;
vector<bool> is_prime;

// 小さい素数(MAX_PRIMEまで)は先に用意しとく
void init_primes() {
  is_prime = vector<bool>(MAX_PRIME + 1, true);
  is_prime[0] = is_prime[1] = false;
  for (int i = 2; i <= MAX_PRIME; i++) {
    if (is_prime[i]) {
      primes.push_back(i);
      for (int j = i*2; j <= MAX_PRIME; j+=i) {
        is_prime[j] = false;
      }
    }
  }
}

// 素数かどうか判定。大きければミラーラビンを使う
bool isPrime(LL n) {
  if (n <= MAX_PRIME) return is_prime[n];
  else return miller_rabin(n, 20);
}

// 素因数分解する。小さい数は用意した素数で試し割り、大きければポラード・ロー
void factorize(LL n, map<LL, int>& factors) {
  if (isPrime(n)) {
    factors[n]++;
  } else {
    for (int i = 0; i < primes.size(); i++) {
      int p = primes[i];
      while (n % p == 0) {
        factors[p]++;
        n /= p;
      }
    }
    if (n != 1) {
      if (isPrime(n)) {
        factors[n]++;
      } else {
        LL d = pollard_rho(n, 1);
        factorize(d, factors);
        factorize(n/d, factors);
      }
    }
  }
}

pair<LL, LL> solve(LL a, LL b) {
  LL c = b / a;
  map<LL, int> factors;
  factorize(c, factors);

  vector<LL> mult_factors;
  for (map<LL, int>::iterator it = factors.begin(); it != factors.end(); it++) {
    LL mul = 1;
    while (it->second) {
      mul *= it->first;
      it->second --;
    }
    mult_factors.push_back(mul);
  }
 
  LL best_sum = 1e18, best_x = 1, best_y = c;
  for (int mask = 0; mask < (1 << mult_factors.size()); mask++) {
    LL x = 1;
    for (int i = 0; i < mult_factors.size(); i++) {
      if (mask & (1 << i)) x *= mult_factors[i];
    }
    LL y = c / x;
    if (x < y && x + y < best_sum) {
      best_sum = x + y;
      best_x = x;
      best_y = y;
    }
  }
  return make_pair(best_x * a, best_y * a);
}
  
int main() {
  cin.tie(0);
  ios::sync_with_stdio(false);

  init_primes();
  LL a, b;
  while (cin >> a >> b) {
    pair<LL, LL> ans = solve(a, b);
    cout << ans.first << " " << ans.second << endl;
  }
}
```
