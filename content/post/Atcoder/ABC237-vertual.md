+++
draft = false
thumbnail = ""
tags = ["C++", "AtCoder"]
categories = ["AtCoder"]
date = "2023-02-17T03:16:18+09:00"
title = "ABC237(途中までバーチャル参加)"
description = ""
+++

## Result 
　50分でABCDの4完。
実際に参加するとパフォーマンスは大体1000程度。  
　Eはダイクストラ法でどうにかしたかったけど、コストが負の辺があるので、うーん、となったまま諦めた。解説を読むことにする。  
　解説は今度書く(今はとても眠い)

## A - Not Overflow
### 問題文
整数$N$が与えられます。$N$が$−2^{31}$以上かつ$2^{31}$未満ならば Yes を、そうでないならば No を出力してください。

### 制約
$−2^{63}≤N<2^{63}$
 
$N$は整数である。

### 提出
```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    long long N ;
    cin >> N ; 
    if( N < 2147483648 && N >= -2147483648) cout << "Yes" << endl;
    else cout << "No" << endl; 
}
```

## B - Matrix Transposition
### 問題文
$H$行$W$列の行列$A$が与えられます。
$A$の上から 
$i$行目、左から 
$j$列目の要素は 
$A_{i,j}$です。  
ここで、
$W$行$H$列の行列$B$を、上から$i$行目、左から 
$j$列目の要素が$A_{j,i}$と一致するような行列として定めます。
すなわち、$B$は$A$の転置行列です。$B$を出力してください。

### 制約
$1≤H,W≤10^5$  
$H×W≤10^5$  
$1≤A_{i,j}≤10^9$  
入力は全て整数である

### 提出
```cpp
#include<bits/stdc++.h>
using namespace std ; 

int main(){
    int H , W ;
    cin >> H >> W ; 
    vector<vector<int> > A(H , vector<int>(W)) ; 

    for(int i = 0 ; i < H ; i++){
        for(int j = 0 ; j < W ; j++){
            cin >> A.at(i).at(j) ; 
        }
    }

    for(int i = 0 ; i < W ; i++){
        for(int j = 0 ; j < H ; j++){
            cout << A.at(j).at(i) << " " ; 
        }
        cout << endl;
    }
}
```


## C - kasaka
### 問題文
英小文字からなる文字列$S$が与えられます。  
$S$の先頭に$a$をいくつか（$0$個でも良い）つけ加えて回文にすることができるか判定してください。
ただし、長さ$N$の文字列 
$A=A_1A_2…A_N$が回文であるとは、すべての$1≤i≤N$について$A_i=A_{N+1−i}$が成り立っていることをいいます。

### 制約
$1≤∣S∣≤10^6$  
$S$は英小文字のみからなる。

### 提出
```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    string S ;
    cin >> S ; 
    int N = S.size() ; 
    int back_a = 0 ; 
    for(int i = 0 ;  i < N ; i++){
        if(S.at(N-i-1) == 'a') back_a++ ; 
        else break ; 
    }

    int front_a = 0 ; 
    for(int i = 0 ; i < N ; i++){
        if(S.at(i) == 'a') front_a++ ; 
        else break ; 
    }

    if(front_a > back_a) cout << "No" << endl;
    else{
        reverse(S.begin() , S.end()) ; 
        for(int i = 0 ; i < back_a - front_a ; i++){
            S.push_back('a') ; 
        }
        reverse(S.begin() , S.end() ) ; 
        for(int i = 0 ; i < S.size() ; i++){
            if(S.at(i) != S.at(S.size() - 1 - i )){
                cout << "No" << endl;
                return 0 ; 
            }
        }

        cout << "Yes" << endl;
    }

}
```

## D - LR insertion
### 問題文
$1$個の$0$のみからなる数列$A=(0)$があります。  
また、$L$と$R$のみからなる長さ$N$の文字列 
$S=s_1s_2…s_N$が与えられます。

$i=1,2,…,N$の順番で、次の操作を行います。  
 - $s_i$が$L$のとき、$A$内にある$i−1$のすぐ左に$i$を挿入する  
 - $s_i$が$R$のとき、$A$内にある$i−1$のすぐ右に$i$を挿入する  

最終的な$A$を求めてください。

### 制約
$1≤N≤5×10^5$  
$N$は整数である
$∣S∣=N$  
$s_i$は$L$か$R$のいずれかである

### 提出
```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int N ;
    cin >> N ;
    string S ;
    cin >> S ;

    deque<int> q ; 
    q.push_back(N) ; 

    for(int i = N-1 ; i >= 0 ; i--){
        if(S.at(i) == 'L'){
            q.push_back(i) ; 
        }
        else{
            q.push_front(i) ; 
        }
    }

    for(int i = 0 ; i <= N ; i++){
        cout << q.at(i) << " " ; 
    }
    cout << endl;
}
```