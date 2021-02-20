
## Ch. 02
## https://github.com/jukyellow/pyda100/blob/master/2%EC%9E%A5/%EC%A0%9C2%EC%9E%A5_answer.ipynb

### 엑셀 읽기
kokyaku_data = pd.read_excel("kokyaku_daicho.xlsx")

### 상품명 오류(공백/대소문자) 수정
```
print(len(pd.unique(uriage_data["item_name"])))

uriage_data["item_name"] = uriage_data["item_name"].str.upper()
uriage_data["item_name"] = uriage_data["item_name"].str.replace("　", "")
uriage_data["item_name"] = uriage_data["item_name"].str.replace(" ", "")

print(uriage_data.sort_values(by=["item_name"], ascending=True))
```

### 금액 결측치 수정(null인 값을 찾아서 유사상품중에 가장 큰값으로 세팅)
```
print(uriage_data.isnull().any(axis=0))
flg_is_null = uriage_data["item_price"].isnull()

# 금액 결측치가 있는(flag_is_null==True) 대상만 반복
for trg in list(uriage_data.loc[flg_is_null, "item_name"].unique()):
    # ~flg_is_null ==> flg_is_null == False
    # 상품명이 같은, 금액 결측치가 없 대상들중에서, 최대값을 구해서 -> 결측치가 있는 대상으로 업데이트
    price = uriage_data.loc[(~flg_is_null) & (uriage_data["item_name"] == trg), "item_price"].max()
    uriage_data["item_price"].loc[(flg_is_null) & (uriage_data["item_name"]==trg)] = price

uriage_data.head()

# 상품명 최고가 최저가
for trg in list(uriage_data["item_name"].sort_values().unique()):
    print(trg + "의최고가：" + str(uriage_data.loc[uriage_data["item_name"]==trg]["item_price"].max()) 
          + "의최저가：" + str(uriage_data.loc[uriage_data["item_name"]==trg]["item_price"].min(skipna=False)))
```

###  날짜오류 수정(숫자->날짜변환)
```
flg_is_serial = kokyaku_data["등록일"].astype("str").str.isdigit()
print(flg_is_serial.sum())

fromSerial = pd.to_timedelta(kokyaku_data.loc[flg_is_serial, "등록일"].astype("float"), unit="D") + pd.to_datetime("1900/01/01")
fromString = pd.to_datetime(kokyaku_data.loc[~flg_is_serial, "등록일"])

kokyaku_data["등록일"] = pd.concat([fromSerial, fromString])
print(kokyaku_data)

kokyaku_data["등록연월"] = kokyaku_data["등록일"].dt.strftime("%Y%m")
rslt = kokyaku_data.groupby("등록연월").count()["고객이름"]
print(rslt)
```

### 특정 키(이름)을 중심으로 두개의 데이터를 결합(조인)
```
join_data = pd.merge(uriage_data, kokyaku_data, left_on="customer_name", right_on="고객이름", how="left")
join_data = join_data.drop("customer_name", axis=1)
join_data
```

### 정제한 데이터를 덤프하자
```
dump_data = join_data[["purchase_date", "purchase_month", "item_name", "item_price", "고객이름", "지역", "등록일"]]
print(dump_data)
dump_data.to_csv("dump_data.csv", index=False)
```

### 데이터를 집계하자
- 월별 상품별 건수  
```
import_data = pd.read_csv("dump_data.csv")
import_data.head()
byItem = import_data.pivot_table(index="purchase_month", columns="item_name", aggfunc="size", fill_value=0)
print(byItem)
```
- 고객별 구매 건수  
```
byCustomer = import_data.pivot_table(index="purchase_month", columns="고객이름", aggfunc="size", fill_value=0)
byCustomer
```
- 도시별 구매건수 합  
```
byRegion = import_data.pivot_table(index="purchase_month", columns="지역", aggfunc="size", fill_value=0)
byRegion
```
- 결합 오류건? 확인?
```
away_data = pd.merge(uriage_data, kokyaku_data, left_on="customer_name", right_on="고객이름", how="right")
away_data[away_data["purchase_date"].isnull()]
```
