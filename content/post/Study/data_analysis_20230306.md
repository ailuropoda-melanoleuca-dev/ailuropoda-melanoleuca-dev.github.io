+++
draft = false
thumbnail = ""
tags = ["Python" , "Data-Analysis"]
categories = ["Study" , "Data-Science"]
date = "2023-03-06T19:36:48+09:00"
title = "データ分析100本ノック(1-10)"
description = ""
+++

　なんで今更？と思うかもしれないが、家に積読されているのを発見したので、10本ずつノックしていく。
毎日10本ノックすれば10日で終わる(計画性なし)。

今回はデータの読み込み、データフレームの結合とかから、matplotlibでグラフを書くところまで。

### ノック1　データを読み込んでみよう
- read_csvをやるだけ
- head()で上5件見れる

```python3
import pandas as pd 
customer_master_df = pd.read_csv("../data/customer_master.csv")
customer_master_df.head()
```

### ノック2　データを結合してみよう
- 下にデータフレームを結合する
- concat([hoge, huga])でhogeの下にhugaをくっつける
- ignore_index = True でインデックスを振り直す

```python3
transaction_1_df = pd.read_csv("../data/transaction_1.csv")
transaction_2_df = pd.read_csv("../data/transaction_2.csv")
transaction_df = pd.concat([transaction_1_df, transaction_2_df], ignore_index = True)
transaction_df.head()


transaction_detail_1_df = pd.read_csv("../data/transaction_detail_1.csv")
transaction_detail_2_df = pd.read_csv("../data/transaction_detail_2.csv")
transaction_detail_df = pd.concat([transaction_detail_1_df, transaction_detail_2_df], ignore_index = True)
transaction_detail_df.head()
```

### ノック3　売り上げデータ同士を結合してみよう
- merge([hoge, huga])で横にくっつける
- on = "hugo"でhugoをジョインキーとしてくっつける
- howで右左どちらにくっつけるか決めれる

```python3
join_data = pd.merge(transaction_detail_df, transaction_df[["transaction_id",  "payment_date" , "customer_id"]], on = "transaction_id", how = "left")
```

### ノック4　マスターデータを結合してみよう
- 足りないデータ、共通データは何か、どちらのデータフレームが主軸になるかを考える

```python3
join_data= pd.merge(join_data, customer_master_df, on = "customer_id", how = "left")
item_master_df = pd.read_csv("../data/item_master.csv")
join_data = pd.merge(join_data, item_master_df, on = "item_id", how = "left")
```

### ノック5　必要なデータ列を作ろう
- hugo["a"] = hugo["b"] * hugo["c"]でhugoにaカラムが計算されて追加される

```python3
join_data["price"]= join_data["quantity"] * join_data["item_price"]
```

### ノック6　データ検算をしよう
- sum()メソッドで列ごとに総和

```python3
print(join_data["price"].sum())
```

### ノック7　各種統計量を把握しよう
- isnull()でnullならばTrueとなるデータフレームを返す
- それを列ごとにsum()すれば、そのカラムにいくつのnullがあるかが分かる
```python3
join_data.isnull().sum()
```
- describe()は基本統計量が計算できる

```python3
join_data.describe()
```

### ノック8　月別でデータ集計してみよう
- to_datetime()で時系列データに変更
- dt.strftime("%Y%m")で文字列として、年月値を作成
```python3
join_data["payment_date"] = pd.to_datetime(join_data["payment_date"])
join_data["payment_month"] = join_data["payment_date"].dt.strftime("%Y%m")
```
- groupby("グループ化キー").sum()["xxx"]でキーに沿ってグルーピングし、xxxカラムを総和する
```python3
join_data.groupby("payment_month").sum()["price"]
```

### ノック9　月別、商品別でデータを集計してみよう
- 上と同じ、月であり、アイテム名であるグループ化要素をしていして、量と価格を抜き出す

```python3
join_data.groupby(["payment_month", "item_name"]).sum()[["price", "quantity"]]
```

### ノック10　商品別の売上推移を可視化してみよう
- ピボットテーブルでテーブル作成

```python3
graph_data = pd.pivot_table(join_data, index = "payment_month", columns = "item_name" ,values = "price", aggfunc = "sum")
```
- 可視化
```python3
import matplotlib.pyplot as plt 
%matplotlib inline

plt.plot(list(graph_data.index) , graph_data["PC-A"] , label = "PC-A")
plt.plot(list(graph_data.index) , graph_data["PC-B"] , label = "PC-B")
plt.plot(list(graph_data.index) , graph_data["PC-C"] , label = "PC-C")
plt.plot(list(graph_data.index) , graph_data["PC-D"] , label = "PC-D")
plt.plot(list(graph_data.index) , graph_data["PC-E"] , label = "PC-E")
plt.legend()
```