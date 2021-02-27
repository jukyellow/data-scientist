# 

## Ch 03.
## https://github.com/jukyellow/pyda100/blob/master/3%EC%9E%A5/%EC%A0%9C3%EC%9E%A5_answer.ipynb

### 최근 고객데이터를 집계
- 최근 데이터 추출  
```
customer_join["end_date"] = pd.to_datetime(customer_join["end_date"])
customer_newer = customer_join.loc[(customer_join["end_date"]>=pd.to_datetime("20190331"))|(customer_join["end_date"].isna())]
```
- 집계
```
print(len(customer_newer))
print(customer_newer["end_date"].unique())
print(customer_newer.groupby("class_name").count()["customer_id"])
print(customer_newer.groupby("campaign_name").count()["customer_id"])
print(customer_newer.groupby("gender").count()["customer_id"])
```
### 이용이력 집계
- 월별 이용이력  
```
uselog["usedate"] = pd.to_datetime(uselog["usedate"])
uselog["연월"] = uselog["usedate"].dt.strftime("%Y%m")
uselog_months = uselog.groupby(["연월","customer_id"],as_index=False).count()
print(uselog_months.head())

uselog_months.rename(columns={"log_id":"count"}, inplace=True)
del uselog_months["usedate"]
print(uselog_months.head())

uselog_customer = uselog_months.groupby("customer_id").agg(["mean", "median", "max", "min" ])["count"]
uselog_customer = uselog_customer.reset_index(drop=False)
uselog_customer.head()
```
- 요일별 이용이력  
```
# weekday(요일정보): 0~6 -> 월~일요일
uselog["weekday"] = uselog["usedate"].dt.weekday
uselog_weekday = uselog.groupby(["customer_id","연월","weekday"], 
                                as_index=False).count()[["customer_id","연월", "weekday","log_id"]]
uselog_weekday.rename(columns={"log_id":"count"}, inplace=True)
print(uselog_weekday.head())

uselog_weekday = uselog_weekday.groupby("customer_id",as_index=False).max()[["customer_id", "count"]]
uselog_weekday["routine_flg"] = 0
uselog_weekday["routine_flg"] = uselog_weekday["routine_flg"].where(uselog_weekday["count"]<4, 1)
uselog_weekday.head()
```
- 회원기간 계산 : 년월정보->월수를 추출   
```
from dateutil.relativedelta import relativedelta
customer_join["calc_date"] = customer_join["end_date"]
customer_join["calc_date"] = customer_join["calc_date"].fillna(pd.to_datetime("20190430"))
customer_join["membership_period"] = 0
for i in range(len(customer_join)):
    delta = relativedelta(customer_join["calc_date"].iloc[i], customer_join["start_date"].iloc[i])
    customer_join["membership_period"].iloc[i] = delta.years*12 + delta.months
customer_join.head()
```
- hist 그래프  
```
import matplotlib.pyplot as plt
%matplotlib inline
plt.hist(customer_join["membership_period"])
```
