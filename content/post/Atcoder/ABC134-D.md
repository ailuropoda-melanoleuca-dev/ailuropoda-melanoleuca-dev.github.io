+++
draft = false
thumbnail = ""
tags = ["AtCoder", "C++"]
categories = ["AtCoder"]
date = "2023-02-15T01:44:12+09:00"
title = "ABC134-D"
description = ""
+++


### D - Preparing Boxes (Diff-926)

$N$個の空の箱が横一列に並んでいます。 左から 
$i(1≤i≤N)$ 番目の箱には整数 
$i$が書かれています。  
すぬけさんは、それぞれの箱に対してボールを 
$1$個入れるか何も入れないかを選ぶことができます。  
ここで、以下の条件を満たすようなボールの入れ方を、いいボールの入れ方と定めます。
$1$以上 
$N$以下の任意の整数 
$i$について、
$i $の倍数が書かれた箱に入っているボールの個数の和を 
$2$で割った余りが 
$a_i$である。  
いいボールの入れ方は存在するでしょうか。存在するならば 
$1$つ求めてください。

## 解法

　ある箱にボールを入れるor入れない事による影響を考える。ボールを入れると、どこに影響が出るかと言うと、これは箱$i$にボールを入れるとした時、その$i$の倍数の番号がついた箱。これは$i$より必ず大きくなることがわかる。  
　ということは箱番号$j$が大きいものから順にボールを入れるか決めていき、小さいもので辻褄を合わせることで、必ずそのようなボールの入れ方は決めることができることがわかる。


### 提出
0-indexedだと頭がこんがらがるので、1-indexedとして実装した。  
$idx \times co$は箱$i$の倍数を表しているが、long longになり得るので、そこだけは気をつける点かなと思った。

```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int N ;
    cin >> N ; 
    vector<int> a(N+1) ;
    for(int i = 1 ; i <= N ; i++) cin >> a.at(i) ; 
    vector<int> ans ; 
    vector<int> b(N+1) ; 
    for(int i = N ; i > 0 ; i--){
        int now = 0 ; 
        long long idx = i ; 
        long long co = 1 ; 
        while(idx * co <= N){
            now += b.at(idx*co) ; 
            co++ ; 
        }

        if(now % 2 != a.at(i)){
            ans.push_back(i) ; 
            b.at(i) = 1 ;
        }
    }

    cout << ans.size() << endl;
    for(auto p : ans) cout << p << endl;

}

```