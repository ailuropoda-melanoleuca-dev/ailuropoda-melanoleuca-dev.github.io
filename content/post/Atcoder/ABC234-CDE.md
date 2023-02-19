+++
draft = false
thumbnail = ""
tags = ["Atcoder", "C++" ]
categories = ["AtCoder"]
date = "2023-02-20T03:02:37+09:00"
title = "ABC234-CDE"
description = ""
+++

## C - Happy New Year!
### 問題文
$10$進法で表記したときに$0,2$のみからなる正整数のうち、 
$K$番目に小さいものを求めてください。

### 制約
 - $K$は$1$以上$10^{18}$以下の整数

### 解法
問題が$0,1$のみからなる正整数だった時を考える。これはある正整数を$2$進数で書き直したものが正解となる。例えば、小さいのものから
- $0 = 0_2$  
- $1 = 1_2$
- $2 = 10_2$
- $3 = 11_2$  

というふうになる。  
この問題では、$K$を2進数で表した時に、現れる$1$を$2$に変えるとACする。  
私は基数を変換するライブラリがあったので、文字列として受け取り、文字列として$2$進数に変換し、その文字列中の$1$を$2$に変更した。

```cpp


#include<bits/stdc++.h>
using namespace std ;

class Radix {
private:
  const char* s;
  int a[128];
public:
  Radix(const char* s = "0123456789ABCDEF") : s(s) {
    int i;
    for(i = 0; s[i]; ++i)
      a[(int)s[i]] = i;
  }
  std::string to(long long p, int q) {
    int i;
    if(!p)
      return "0";
    char t[64] = { };
    for(i = 62; p; --i) {
      t[i] = s[p % q];
      p /= q;
    }
    return std::string(t + i + 1);
  }
  std::string to(const std::string& t, int p, int q) {
    return to(to(t, p), q);
  }
  long long to(const std::string& t, int p) {
    int i;
    long long sm = a[(int)t[0]];
    for(i = 1; i < (int)t.length(); ++i)
      sm = sm * p + a[(int)t[i]];
    return sm;
  }
};

/*
int main() {
    Radix r;
    // 10 進数を n 進文字列に
    std::cout << r.to(255, 10) << std::endl; // => "255"
    std::cout << r.to(255, 12) << std::endl; // => "193"
    std::cout << r.to(255, 16) << std::endl; // => "FF"

    // n 進文字列を 10 進数に
    std::cout << r.to("255", 10) << std::endl; // => 255
    std::cout << r.to("255", 14) << std::endl; // => 467

    // n 進文字列を m 進文字列に
    std::cout << r.to("255", 7, 11) << std::endl; // => "116"
    std::cout << r.to("255", 11, 7) << std::endl; // => "611"
    return 0;
}

*/

int main(){
  string K ;
  cin >> K ; 
  Radix r ; 
  string ans = r.to(K , 10 , 2) ; 

  for(int i = 0 ; i < (int)ans.size() ; i++){
    if(ans.at(i) == '1') cout << 2 ; 
    else cout << 0 ; 
  }

  cout << endl;

}
```

## D - Prefix K-th Max
### 問題文

$(1,2,…,N)$ の順列 
$P=(P_1,P_2,...,P_N)$、および正整数$K$が与えられます。  
$i=K,K+1,...,N$について、以下を求めてください。

 - $P$の先頭$i$項のうち、$K$番目に大きい値

### 制約
 - $1≤K≤N≤5×10^5$ 
 - $(P_1,P_2,…,P_N)$は$(1,2,…,N)$の並び替えによって得られる  
 - 入力はすべて整数

### 解法
最初にSetにK個目までの要素を突っ込む。大きいものからK番目に大きいものはSetの先頭( *set.begin() )である。  
ここから順に$K$以上の$P$の要素を突っ込んでいく。この時に今現在の( *set.begin() )よりも大きい値が入ってくると、大きいものから見て順番が$1$個ずれる。よってインクリメントしたものが答え。

 ```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    int N , K ;
    cin >> N >> K ; 
    vector<int> P(N) ; 
    for(int i = 0 ; i < N ; i++) cin >> P.at(i) ; 
    set<int> st ; 
    for(int i = 0 ; i < K ; i++) st.insert(P.at(i)) ; 

    auto itr = st.begin(); 
    cout << *itr<< endl;

    for(int i = K ; i < N ; i++){
        st.insert(P.at(i)) ; 
        if(*itr < P.at(i)) itr++ ; 
        cout << *itr << endl;
    }

}

 ```

## E - Arithmetic Number
### 問題文
以下の条件を満たす正の整数$n$を、 等差数 と呼びます。
 - ($n$を先頭に余計な$0$を付けずに$10$進法で表記した際)$n$の上から$i$桁目を$d_i$とする。このとき、$n$が 
$k$桁の整数であったとすると、$(d_2, −d_1)=(d_3 − d_2)=...=(d_k − d_{k-1} )$が成立する。
    - この条件は、「 数列$(d_1 , d_2 , ... , d_k)$が等差数列である」と言い換えることができる。
    - 但し、$n$が$1$桁の整数である時、$n$は等差数であるものとする。たとえば、$234,369,86420,17,95,8,11,777$は等差数ですが、$751,919,2022,246810,2356$は等差数ではありません。

等差数のうち、$X$以上で最小のものを求めてください。

### 制約
$X$は$1$以上$10^{17}$以下の整数である

### 解法
数列の項の値は必ず1桁でなければならない。なので、実は候補はそんなに多くない。
初項、公差を全探索し、出来上がった数列に対して$min$を取ればACする。

```cpp
#include<bits/stdc++.h>
using namespace std ;

int main(){
    long long x ;
    cin >> x ; 
    long long ans = 2e18 ; 
    for(int s = 1 ; s <= 9 ; s++){//初項
        for(int d = -9 ; d <= 8 ; d++){//公差
            long long a = 0 ; 
            for(int i = s  ; 0 <= i && i <= 9 ; i += d ){
                a = a*10 + i ; 
                if(a >= x){
                    ans = min(ans ,a) ; 
                    break ; 
                }
            }
        }
    }

    cout << ans << endl;
}

```

