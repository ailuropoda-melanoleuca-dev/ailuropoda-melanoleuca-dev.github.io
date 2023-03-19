+++
draft = false
thumbnail = ""
tags = ["Python" , "Data-Analysis"]
categories = ["Data-Science" , "Study"]
date = "2023-03-20T01:50:15+09:00"
title = "データ分析100本ノック(21-30)"
description = ""
+++


### ノック21 
- 読み込んでデータ確認

```python3
import pandas as pd 
uselog = pd.read_csv("../data/use_log.csv")
customer = pd.read_csv("../data/customer_master.csv")
class_master = pd.read_csv("../data/class_master.csv")
campaign_master = pd.read_csv("../data/campaign_master.csv")
print(len(uselog))
uselog.head()
customer.head()
class_master.head()
campaign_master.head()
```

### ノック22
- それぞれどのキーを使えば良いか考えて、データをマージする
```python3
customer_join = pd.merge(customer, class_master, on = "class" , how = "left")
customer_join = pd.merge(customer_join , campaign_master, on = "campaign_id", how = "left")
customer_join
customer_join.isnull().sum()
```

### ノック23 
- データ集計
  - 会員クラス
  - 今までの会員-退会収支 
```python3
customer_join.groupby("class_name").count()["customer_id"]
customer_join["start_date"] = pd.to_datetime(customer_join["start_date"])
customer_start = customer_join.loc[customer_join["start_date"] > pd.to_datetime("20180401")]
print(len(customer_start))
```

### ノック24 
- 最新月で在入会中の人は、end_dateがNaNか、2019-03-31以降の人
  - locで抽出
```python3
customer_join["end_date"] = pd.to_datetime(customer_join["end_date"])
customer_newer = customer_join.loc[(customer_join["end_date"]>=pd.to_datetime("20190331"))|(customer_join["end_date"].isna())]
print(len(customer_newer))
customer_newer["end_date"].unique()
customer_newer.groupby("class_name").count()["customer_id"]
```

### ノック25 
- 顧客毎に月の利用回数を算出
  - datetime型に変換してからgroupbyで集計 
```python3
uselog["usedate"] = pd.to_datetime(uselog["usedate"])
uselog["年月"] = uselog["usedate"].dt.strftime("%Y%m")
uselog_months = uselog.groupby(["年月" , "customer_id"], as_index = False).count()
uselog_months.rename(columns={"log_id":"count"}, inplace=True)
del uselog_months["usedate"]
uselog_months.head()
```

- 集計したものについて、さらに基本統計量を算出

```python3
uselog_customer = uselog_months.groupby("customer_id").agg(["mean", "median", "max", "min" ])["count"]
uselog_customer = uselog_customer.reset_index(drop=False)
uselog_customer.head()
```

### ノック26 
- 上で集計した利用データから、定期的に利用しているか否かを算出
  - 曜日毎に集計する
```python3
uselog["weekday"] = uselog["usedate"].dt.weekday
uselog_weekday = uselog.groupby(["customer_id", "年月" , "weekday"], as_index = False).count()[["customer_id", "年月" , "weekday", "log_id"]]
uselog_weekday.rename(columns = {"log_id" : "count"}, inplace = True)
```
- 月のうち、同じ日に4回以上利用していると、フラグを立てる
```python3
uselog_weekday = uselog_weekday.groupby("customer_id",as_index=False).max()[["customer_id", "count"]]
uselog_weekday["routine_flg"] = 0
uselog_weekday["routine_flg"] = uselog_weekday["routine_flg"].where(uselog_weekday["count"] < 4, 1)
uselog_weekday.head()
```


### ノック27 
- マージ
```python3
customer_join = pd.merge(customer_join, uselog_customer, on="customer_id", how="left")
customer_join = pd.merge(customer_join, uselog_weekday[["customer_id", "routine_flg"]], on="customer_id", how="left")
customer_join.head()

```


### ノック28 
- 会員期間の算出
  - NaNをどうするか
  - 単位は月
  - relativedeltaは日付の比較に使うやつ
```python3
from dateutil.relativedelta import relativedelta

customer_join["calc_date"] = customer_join["end_date"]
customer_join["calc_date"] = customer_join["calc_date"].fillna(pd.to_datetime("20190430"))
customer_join["membership_period"] = 0
for i in range(len(customer_join)):
    delta = relativedelta(customer_join["calc_date"].iloc[i], customer_join["start_date"].iloc[i])
    customer_join["membership_period"].iloc[i] = delta.years*12 + delta.months
customer_join
```

### ノック29 
- 会員期間の分布をみる
```python3
import matplotlib.pyplot as plt 
plt.hist(customer_join["membership_period"])

```


### ノック30 
- 退会者、継続者でそれぞれdescribeする
  - 違いをみようぜ
```python3
customer_end = customer_join.loc[customer_join["is_deleted"] == 1]
customer_end.head()

```