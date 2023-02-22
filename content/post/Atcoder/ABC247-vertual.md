+++
draft = false
thumbnail = ""
tags = ["AtCoder", "C++"]
categories = ["AtCoder"]
date = "2023-02-23T01:00:38+09:00"
title = "ABC247-ABCD"
description = ""
+++

## A - Move Right
### 問題文
横一列に$4$つのマスが並んでいます。  

各文字が$0$または$1$である長さ$4$の文字列$S$が与えられます。  
$S$の$i$文字目が$1$であるとき、左から$i$番目のマスには$1$人の人がおり、$S$の$i$文字目が$0$であるとき、左から$i$番目のマスには人がいません。  

全ての人が一斉に、$1$つ右隣のマスへ移動します。この移動により、もともと右端のマスにいた人は消えます。  

移動後の各マスに人がいるかどうかを、$S$と同様のルールで文字列として出力してください。

### 制約
$S$は $0, 1$のみからなる長さ$4$の文字列

### 解法

```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    string S ;
    cin >> S ; 
    S.pop_back() ; 
    cout << 0 << S << endl;
}
```

## B - Unique Nicknames
### 問題文
人$1$, 人$2$, … 人$N$の$N$人の人がいます。人$i$の姓は$s_i$、名は$t_i$です。

$N$人の人すべてにあだ名をつけることを考えます。人$i$のあだ名$a_i$は以下の条件を満たす必要があります。
- $a_i$は人$i$の姓あるいは名と一致する。言い換えると、$a_i = s_i$または$a_i = t_i$の少なくとも一方が成り立つ。
- $a_i$は自分以外の人の姓および名のどちらとも一致しない。言い換えると、$1≤j≤N,i \neq j$を満たすすべての整数$j$について$a_i \neq s_j$ かつ$a_i \neq t_j$が成り立つ。

$N$人全員に条件を満たすあだ名をつけることは可能でしょうか。可能ならば Yes を、そうでないならば No を出力してください。

### 制約
 - $2≤N≤100$
 - $N$は整数である。
 - $s_i,t_i$は英小文字からなる$1$文字以上$10$文字以下の文字列である。

### 解法

```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int N ;
    cin >> N ; 
    vector<string> s(N) , t(N) ; 

    map<string , int> mp ; 

    for(int i = 0 ; i < N ; i++) cin >> s.at(i) >> t.at(i) ; 

    for(int i = 0 ; i < N ; i++){
        mp[s.at(i)]++ ; 
        mp[t.at(i)]++ ; 
    }

    for(int i = 0 ; i < N ; i++){
        string S = s.at(i) ;
        string T = t.at(i) ; 

        if(S != T && mp[S] >= 2 && mp[T] >= 2){
            cout << "No" << endl;
            return 0 ; 
        }
        else if(S == T && (mp[S] >= 3) ){
            cout << "No" << endl;
            return 0 ; 
        }
    }

    cout << "Yes" << endl;

}
```

## C - 1 2 1 3 1 2 1
### 問題文
列$S_n$を次のように定義します。

- $S_1$は$1$つの$1$からなる長さ$1$の列である。
- $S_n$($n$は$2$以上の整数)は$S_{n−1},n,S_{n−1}$をこの順につなげた列である。

たとえば$S_2,S_3$は次のような列です。

- $S_2$は$S_1,2,S_1$をこの順につなげた列なので$1,2,1$である。
- $S_3$は$S_2,3,S_2$をこの順につなげた列なので$1,2,1,3,1,2,1$である。

$N$が与えられるので、列$S_N$をすべて出力してください。

### 制約
- $N$は整数
- $1≤N≤16$


### 解法

```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int N ;
    cin >> N ; 

    vector<vector<int > > s(N+1 , vector<int> ()) ; 
    s.at(1).push_back(1) ;  

    for(int i = 2; i <= N ; i++){

        for(int j = 0 ; j < s.at(i-1).size() ; j++){
            s.at(i).push_back(s.at(i-1).at(j)) ; 
        }

        s.at(i).push_back(i) ; 

        for(int j = 0 ; j < s.at(i-1).size() ; j++){
            s.at(i).push_back(s.at(i-1).at(j)) ; 
        }

    }

    for(int i = 0 ; i < s.at(N).size() ; i++){
        cout << s.at(N).at(i) << " " ; 
    }
    cout << endl;
}
```

## D - Cylinder
### 問題文
空の筒があります。$Q$個のクエリが与えられるので順に処理してください。  
クエリは次の$2$種類のいずれかです。

- **1 x c** : 数$x$が書かれたボールを筒の右側から$c$個入れる
- **2 c** : 筒の左側からボールを$c$個取り出し、取り出したボールに書かれている数の合計を出力する

なお、筒の中でボールの順序が入れ替わることはないものとします。

### 制約
- $1≤Q≤2×10^5$
- $0≤x≤10^9$
- $1≤c≤10^9$
- **2 c**のクエリが与えられるとき、筒の中には$c$個以上のボールがある
- 入力に含まれる値は全て整数である

### 解法

```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int Q ;
    cin >> Q ; 

    deque<pair<long long , long long> > q ; 

    while(Q--){
        long long t , x , c; 
        cin >> t ;

        if(t == 1){
            cin >> x >> c ;
            q.push_back(make_pair(x,c)) ; 
        }
        else{
            cin >> c ; 
            long long now = 0 ; 

            while(c){
                pair<long long , long long> xc = q.front() ; 
                q.pop_front() ; 

                if(c >= xc.second){
                    c -= xc.second; 
                    now += xc.second*xc.first ; 
                }
                else{
                    now += xc.first*c ;
                    q.push_front(make_pair(xc.first , xc.second - c)) ; 
                    c = 0 ; 
                }
            }
            cout << now << endl;
        }
    }
}
```