---
title: "Spark에서 Upsert구현 방법"
author: Kang HyunGu
date: 2022-08-23 21:43:00 +0900
categories: [Bigdata, Spark]
tags: [Bigdata]
toc: true
pin: false
comments: true
---

Spark를 활용해서 Upsert를 구현하는 아이디어는 아래와 같다.  
#### 1. 기존에 있던 데이터중 신규 데이터에는 없는 데이터 추출.  
A를 기준으로 left anti join 수행.
<p align="left"> <img src="{{site.url}}/img/posts/left_anti_joinA.png" width="200" height="140"></p>  

#### 2. 신규로 추가되는 테이블만 추출   
B를 기준으로 left anti join 수행.
<p align="left"> <img src="{{site.url}}/img/posts/left_anti_joinB.png" width="200" height="140"></p>  

#### 3. 공통된 테이블은 신규로 추가되는 테이블로 대체.  
키값을 기준으로 A와 B에 교집합인 데이터는 B의 데이터로 교체 작업 수행.
<p align="left"> <img src="{{site.url}}/img/posts/inner_join.png" width="200" height="140"></p>  

#### 4. 위의 절차를 코드로 구현.  
멀티 컬럼으로 조인할 경우를 대비하여 컬럼을 hash함수로 구현하여 조인.  
2개의 테이블 모두 만약 대용량 테이블이라면 처리시에 hash함수가 성능에 영향을 미칠 수 있기 때문에  
join 조건에 &&으로 비교하여 멀티 컬럼을 조인하는것이 좋을 수 있겠다.  
다만, 이는 대용량 테이블에는 테스트해보지않아서 테스트가 필요하다.  
```python
from pyspark.sql import SparkSession, DataFrame
from functools import reduce
from pyspark.sql.functions import col, hash

spark = SparkSession.builder.getOrCreate()

data_set1=[("a","A",1),("b","B",2),("c","C",3)]
data_set2=[("a","A",1),("b","B",3),("d","D",4)]
df_col=[“lower”,”upper”,”num"]
dfs=[]
df1=spark.createDataFrame(data=data_set1,schema=df_col)
df2=spark.createDataFrame(data=data_set2,schema=df_col)
df1 = df1.withColumn("hash_value", hash("lower", "upper"))
df2 = df2.withColumn("hash_value", hash("lower", "upper"))

&#35;2개의 테이블이 같은 스키마를 가진 테이블일 경우가 있어 테이블에 alias를 부여함.
inner_df=df1.alias("old_df").join(df2.alias("new_df"), df1["hash_value"]==df2["hash_value"], "inner")
&#35; 2개 테이블에 모두 있는 데이터는 새로운 테이블의 데이터로 교체.
inner_df=inner_df.select("new_df.*").drop("hash_value")
&#35; df1을 기준으로 left anti join 수행
df1_anti_df=df1.join(df2, df1["hash_value"]==df2["hash_value"], "left_anti").drop("hash_value")
&#35; df2을 기준으로 left anti join 수행
df2_anti_df=df2.join(df1, df1["hash_value"]==df2["hash_value"], "left_anti").drop("hash_value")
dfs.extend([df1_anti_df,df2_anti_df,inner_df])
result_df = reduce(DataFrame.unionAll, dfs)
```
