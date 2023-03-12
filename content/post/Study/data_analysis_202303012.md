+++
draft = false
thumbnail = ""
tags = ["Python", "Data-Analysis"]
categories = ["Data-Science"]
date = "2023-03-12T20:16:06+09:00"
title = "データ分析100本ノック(16-20)"
description = ""
+++

### ノック16 顧客名の揺れを補正しよう
- str.replace()で消す。
```python3
kokyaku_data["顧客名"] = kokyaku_data["顧客名"].str.replace(" ", "") 
kokyaku_data["顧客名"] = kokyaku_data["顧客名"].str.replace("　", "") 
kokyaku_data["顧客名"].head()
```
### ノック17 日付の揺れを補正しよう
- excelの書式が整っていないので、登録日に変な値が入っているものがある
- とりあえずそのインデックスを計算
```python3
flg_is_serial = kokyaku_data["登録日"].astype("str").str.isdigit()
```

- 書き換え
- floatにした後に、基準日の19900101を足して計算
```python3
fromSerial = pd.to_timedelta(kokyaku_data.loc[flg_is_serial, "登録日"].astype("float"), unit = "D") + pd.to_datetime("1900/01/01")
```

- 最後にそれぞれ書式を揃えてconcat
```python3
fromString = pd.to_datetime(kokyaku_data.loc[~flg_is_serial , "登録日"])
kokyaku_data["登録日"] = pd.concat([fromSerial , fromString])
```
### ノック18　顧客名をキーに２つのデータを結合しよう
- もう何回もやったので省略
```python3
join_data = pd.merge(uriage_data, kokyaku_data, left_on="customer_name", right_on = "顧客名", how = "left")
join_data = join_data.drop("customer_name" , axis = 1)
join_data
```
### ノック19　クレンジングしたデータをダンプしよう
- データの保存
```python3
dump_data = join_data[["purchase_date", "purchase_month", "item_name" , "item_price" , "顧客名" , "かな" , "地域" , "メールアドレス" , "登録日"]]
dump_data.to_csv("dump_data.csv" , index = False)
```
### ノック20 データを集計しよう
- ピボットテーブルを使っての集計は既に何度かやったので、省略
```python3
byItem = dump_data.pivot_table(index = "purchase_month" , columns = "item_name" , aggfunc = "size" , fill_value = 0 )
byItem
```
