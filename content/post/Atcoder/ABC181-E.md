+++
draft = false
thumbnail = ""
tags = ["AtCoder", "C++", "Cumulative Sum"]
categories = ["AtCoder"]
date = "2023-03-04T16:53:24+09:00"
title = "ABC181-E"
description = ""
+++


### 問題文
$N$人の児童がおり、$i$番目の児童の身長は$H_i$です。$N$は奇数です。

今から、先生であるあなたを含めた$N+1$人で$2$人$1$組を$\frac{N+1}{2}$ペア組みます。

あなたの目標は、それぞれのペアの身長の差の合計を最小化することです。

すなわち、$i$番目のペアの身長の組を$(x_i,y_i)$としたとき、$\sum_{i=1}^{(N+1)/2} |x_i - y_i|$を最小化したいです。

あなたには$M$個の変身形態があり、$i$番目の変身形態での身長は$W_i$です。

あなたの変身形態とペアの組み方を工夫することで、それぞれのペアの身長の差の合計が最小でいくつにできるか求めてください。

### 制約
- 入力はすべて整数
- $1≤N,M≤2×10^5$
- $N$は奇数
- $1≤H_i ≤10^9$
- $1≤W_i≤10^9$

### 解法
- 先生の身長が固定であれば、児童の身長と合わせてソートして、
$0 \leq i \leq N-1$であり、偶数である$i$に対して、$i,i+1$をペアにすれば最適ペアが求まる。

- 先生の身長が可変である場合を考えると、どこに先生を挿入すれば良いかは二分探索でわかる。
    - 先生の走査に$\mathcal{O}(M)$
    - 挿入場所は$\mathcal{O}(logN)$

- ある場所に先生が挿入された時に、影響が出るのは、先生の前後のみである。
    - 前から、後ろからの累積差を求めておくと、$\mathcal{O}(1)$で差分計算可能

```cpp
#include<bits/stdc++.h>
using namespace std ; 

int main(){
    int N , M ; 
    cin >> N >> M ; 
    vector<long long> H(N) , W(M) ; 
    for(int i = 0 ; i < N ; i++) cin >> H.at(i) ; 
    for(int i = 0 ; i < M ; i++) cin >> W.at(i) ; 
    
    sort(H.begin() , H.end()) ; 

    vector<long long > left(N+1 , 0) , right(N+1 , 0) ;//累積差配列

    for(int i = 2 ; i < N ; i++ ){
        left.at(i) = left.at(i-2) +  H.at(i-1) - H.at(i-2) ; //左からの累積
        right.at(i) = right.at(i-2) + H.at(N-i+1) - H.at(N-i) ; //右からの累積
    }
    long long ans = 2LL<<60 ; 

    for(auto w : W){
        int i = lower_bound(H.begin() , H.end() , w) - H.begin() ; //iはwよりも小さい要素の個数

        if(i %2 == 0 ){ //何番目にwが挿入されるかで場合分け
            ans = min(ans , left.at(i) + right.at(N-i-1) + H.at(i) - w) ; 
        }
        else{
            ans = min(ans , left.at(i-1) + right.at(N-i) + w - H.at(i-1)) ; 
        }
    }

    cout <<ans << endl;


}
```