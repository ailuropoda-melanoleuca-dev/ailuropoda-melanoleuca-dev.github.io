+++
draft = false
thumbnail = ""
tags = ["AtCoder", "C++"]
categories = ["AtCoder"]
date = "2023-02-21T00:57:39+09:00"
title = "ABC223-ABCD"
description = ""
+++

## A - Exact Price
### 問題文
高橋君の財布の中には$100$円硬貨が$1$枚以上入っており、それ以外には何も入っていません。
高橋君の財布の中の合計金額が$X$円である可能性はありますか？

### 制約
 - $0≤X≤1000$
 - 入力は全て整数

### 解法
$X$が$100$で割り切れるならばYes, そうでなければNo  
但し、$X$が$0$であればNo
```cpp

#include<bits/stdc++.h>
using namespace std ;

int main(){
    int X;
    cin >> X; 
    if(X%100 == 0 && X != 0) cout << "Yes" << endl;
    else cout << "No" << endl;
}
```

## B - String Shifting
### 問題文
　空でない文字列に対し、先頭の文字を末尾に移動する操作を左シフト、末尾の文字を先頭に移動する操作を右シフトと呼びます。  
　例えば、**abcde**に対して左シフトを1回行うと**bcdea**となり、右シフトを2回行うと**deabc**となります。  
英小文字からなる空でない文字列Sが与えられます。Sに対し左シフトと右シフトをそれぞれ好きな回数（0回でもよい）行って得られる文字列のうち、辞書順で最小のものと最大のものを求めてください。

### 制約
 - $S$は英小文字からなる。
 - $S$の長さは$1$以上$1000$以下である。

### 解法
Sの長さがそんなに大きくないため、１文字ずつ回転すれば良い。
```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    string S ;
    cin >> S ; 
    vector<string> s ; 
    for(int i = 0 ; i < S.size() ; i++){
        s.push_back(S) ;  
        rotate(S.begin() , S.begin()+1, S.end()) ; 
    }

    sort(s.begin(), s.end()) ; 
    cout << s.at(0) << endl;
    cout << s.back() << endl;

}
```

### C - Doukasen
## 問題文
$N$本の導火線を一直線に接着したものがあります。左から$i$本目の導火線は長さが$A_i$cmで、$1$秒あたり$B_i$cmの一定の速さで燃えます。  
この導火線の左端と右端から同時に火をつけるとき、$2$つの火がぶつかる場所が着火前の導火線の左端から何cmの地点か求めてください。

## 制約
 - $1≤N≤10^5$
 - $1≤A_i,B_i≤1000$
 - 入力は全て整数

## 解法

左からの火が右端まで進む時間は、$T = \sum_0^{N-1} \frac{A[i]}{B[i]}$と計算できる。  
右と左から火をつけると、それぞれの火は$T/2$だけ燃え進み、ぶつかる。あとは$T/2$でどれだけ火が進むのか、を計算すれば、それが答え。

```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int N ; 
    cin >> N ; 
    vector<long double > A(N) , B(N) ; 
    for(int i = 0 ; i < N ; i++) cin >> A.at(i) >> B.at(i);
    long double mx = 0.0 ; 

    for(int i = 0 ; i < N ; i++){
        mx += A.at(i)/B.at(i) ; 
    }
    mx /= 2.0 ; 
    cout << fixed << setprecision(10) ;
    long double now = 0.0; 

    for(int i = 0 ; i < N ; i++){
        if(mx - A.at(i)/B.at(i) > 0 ){
            mx -= A.at(i)/B.at(i) ; 
            now += A.at(i) ;
        }
        else{
            now += mx * B.at(i) ; 
            break ; 
        }
    }
    cout << now << endl;
}

```

### D - Restricted Permutation
## 問題文
$(1,2,…,N)$を並び替えて得られる数列$P$であって以下の条件を満たすもののうち、辞書順で最小のものを求めてください。
 - $i=1,…,M$に対し、$P$において$A_i$は$B_i$よりも先に現れる。  

ただし、そのような$P$が存在しない場合は$-1$と出力してください。

## 制約
 - $2≤N≤2×10^{5}$
 - $1≤M≤2×10^{5}$
 - $1≤A_i,B_i≤N$
 - $A_i \neq B_i$
 - 入力は全て整数である。

## 解法

これ系の問題とても好き。  
依存関係を有向グラフとして考えて、トポロジカルソートすれば良い。

1. グラフを隣接リストで持つ
2. 入次数をメモる
3. 入次数が元から0であればプライオリティキューに突っ込む
4. プライオリティキューが空になるまで、現在みている頂点の依存関係にある頂点の入次数を減らす
     - 途中で入次数が0になったものがあれば、プライオリティキューに突っ込む
5. プライオリティキューが空になった時点で、解答用配列の大きさが$N$でなければ、$-1$を出力、そうでなければ、配列を出力する
```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int N ,M ; 
    cin >> N >> M ; 
    vector<vector<int> > G(N, vector<int>()) ; 
    vector<int> deg(N) ; 

    for(int i = 0 ; i < M ; i++){
        int a , b;
        cin >> a >> b ;
        a-- ; 
        b-- ; 
        G.at(a).push_back(b) ; 
        deg.at(b)++ ; 
    }

    priority_queue<int , vector<int> , greater<int> > pq ; 
    for(int i = 0 ; i < N ; i++){
        if(deg.at(i) == 0) pq.push(i) ; 
    }
    vector<int> ans ;

    while(!pq.empty()){
        int now = pq.top() ;
        pq.pop() ;
        ans.push_back(now) ; 
        for(int nx : G.at(now)){
            deg.at(nx)-- ;
            if(deg.at(nx) == 0) pq.push(nx) ; 
        }
    }

    if(ans.size() == N){
        for(int i = 0 ; i < N ; i++) cout << ans.at(i)+ 1  << " "; 
        cout << endl;
    }
    else cout << -1 << endl;
    
}

```