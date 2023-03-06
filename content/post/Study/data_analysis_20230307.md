+++
draft = false
thumbnail = ""
tags = ["Python" , "Data-Analysis"]
categories = ["Study" , "Data-Science"]
date = "2023-03-07T00:24:42+09:00"
title = "データ分析100本ノック(11-15)"
description = ""
+++

既に10本単位じゃなくなってるのウケるな。
今回は少しのデータクレンジングを体験する感じ。

### ノック11/12 データを読み込んでみよう/データの揺れを見てみよう
- 読み込み
```python3
import pandas as pd 
uriage_data = pd.read_csv("../data/uriage.csv")
uriage_data.head()
```
- excelデータの読み込み
```python3
kokyaku_data = pd.read_excel("../data/kokyaku_daicho.xlsx")
kokyaku_data.head()
```

- item_nameに表記揺れ
  - 大文字小文字、変なスペースがあったりする

### ノック13 データに揺れがあるまま集計してみよう
- 商品番号はA-Zまでのローマ字であるのに、小文字が混じっていたり、文字の間にスペースがある
```python3
uriage_data["purchase_date"] = pd.to_datetime(uriage_data["purchase_date"])
uriage_data["purchase_month"] = uriage_data["purchase_date"].dt.strftime("%Y%m")
res = uriage_data.pivot_table(index = "purchase_month", columns = "item_name", aggfunc = "size", fill_value = 0 )
```

### ノック14 商品名の揺れを補正しよう
- str.upper()で大文字に変換
- str.replace("a" , "b")で文字aを文字bに変換

```python3
uriage_data["item_name"] = uriage_data["item_name"].str.upper()
uriage_data["item_name"] = uriage_data["item_name"].str.replace(" " , "")
uriage_data["item_name"] = uriage_data["item_name"].str.replace("  " , "")
```
### ノック15 データを読み込んでみよう
- 欠損値、ここではnullチェック
  - nullが1つでもあるカラムを検索
```python3
uriage_data.isnull().any(axis = 0 )
```

- nullのある箇所を探す
- 同じアイテム名であり、nullでないものから、価格を抜き出す
- 抜き出した価格で修正
- locは複数要素抽出できる

```python3
flg_is_null = uriage_data["item_price"].isnull()

for trg in list(uriage_data.loc[flg_is_null, "item_name"].unique()):
    price = uriage_data.loc[(~flg_is_null) & (uriage_data["item_name"] == trg), "item_price" ].max()
    uriage_data["item_price"].loc[(flg_is_null) & (uriage_data["item_name"] == trg)] = price
```





