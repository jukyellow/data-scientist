# Pandas DataFrame 처리

### 1)csv읽기(한글파일)
```
import pandas as pd
pd = pd.read_csv('201912_purcon_gd.csv', encoding='EUC-KR')
```

### 2)row/col 접근
```
-row: display('df.iloc[0]:', df.iloc[0])  
-col: display('df.loc:', df.loc[:,'ISSUEDT'])  
-row/col : display(df.loc[0]['BYRPRTNUM'])  
```

### 3)정제
```
-중복제거: = df.drop_duplicates(subset='index', keep='last').set_index('index')  
-결측치 확인  
print(df.isna().sum())
print(df.isin([np.nan, np.inf, -np.inf]).any(1))
df[df.isin([np.nan, np.inf, -np.inf]).any(1)]
-결측치 제거
df = df.dropna(axis=0) #0:row, 1:col
print(df.isna().sum())
```

### 4)numpy array 변환
```
df.to_numpy()
```

### 5)col/row 제거
```
-col 제거
df = df.drop("A", axis=1) # df = df.drop(columns="A")
df = df.drop(["A", "C"], axis=1) # df = df.drop(columns=["A", "C"])
-row 제거
df = df.drop("c1") # df = df.drop("c1", axis=0) #df = df.drop(index="c1")
df = df.drop(["c1", "c3"]) # df = df.drop(["c1", "c3"], axis=0) # df = df.drop(index=["c1", "c3"])
```

### 6)순회
```
-col 순회
for col in df.columns:
    series = df[col]
print(type(series[0]))
```

### 7) index 세팅
```
df.set_index('Symbol', inplace=True)
```

### 8)그룹화
```
-단일
df.groupby(by=['상품번호'], as_index=False).min()
df.groupby(by=['상품번호'], as_index=False).max()
df.groupby(by=['상품번호'], as_index=False).count()
df.groupby(by=['상품번호'], as_index=False).sum()
df.groupby(by=['상품번호'], as_index=False).mean()
-멀티항목
df.groupby(by=['고객번호', '상품번호'], as_index=False).sum()
-그룹항목별 사이즈표시의 정렬방법
tr_group = train_csv.sort_values(by="Label", ascending=False).groupby(by=['Label'])
print(tr_group.size().sort_values(ascending=False)) # size()에서 sort_values적용!
```

### 9) 특정 조건의 DataFrame 복제
```
for idx in range(10):
    df_new = train_x_df.loc[train_x_df.loc[:, "coin_index"]==idx]
```
