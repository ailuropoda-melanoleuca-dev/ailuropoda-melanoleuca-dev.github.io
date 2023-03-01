+++
draft = false
thumbnail = ""
tags = ["AtCoder" , "C++", "Pigeonhole principle"]
categories = ["AtCoder"]
date = "2023-03-01T23:55:47+09:00"
title = "ABC174-C"
description = ""
+++

Difficulty - 902

パッと解放が浮かんでくれてよかった

### 問題文
高橋君は$K$の倍数と$7$が好きです。

$7,77,777,…$という数列の中に初めて$K$の倍数が登場するのは何項目ですか？

存在しない場合は代わりに$-1$を出力してください。

### 制約
- $1≤K≤10^6$
- $K$は整数である。

### 解法
毎回単純に$7$の倍数を計算していくと、
7,77,777,...,7777777777777777777で多倍長整数でもオーバーフローする。  
でも、割った余りだけ保持して$10$倍+$7$としていき、$K$で割り切れるかどうかを確認していけば良い(次の項が条件を満たす場合、今の余りが10倍され、7を加えられたものも、条件を満たす)。

あとは「何回それを繰り返すのか」、だけども、鳩の巣原理で$K$の大きさ分繰り返せば良い(実装ではめんどくさいので$K$の制約より大きい固定値を設定している)

```cpp
#include<bits/stdc++.h>
using namespace std ;

const int max_cnt = 10010010 ; 

int main(){
    long long K ; 
    cin >> K ; 
    long long now = 7 ; 

    for(int i = 1; i < max_cnt ; i++){
        if(now % K == 0){
            cout << i << endl;
            return 0 ; 
        }
        now = now*10 + 7 ; 
        now %= K ; 
    }

    cout << -1 << endl;
}
```